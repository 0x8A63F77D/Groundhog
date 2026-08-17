# 02 — 状态模型层

> 位置：UI 与 `IUwfProvider` 之间。全部核心逻辑是**纯函数**（无 IO、无时间、无线程），IO 由一个薄壳按纯函数吐出的 `Effect` 执行。这是方法论 §五.3 的"函数式核心从出生开始"。

## 1. Diff 与 pending（纯函数）

```csharp
public enum PendingArea { Filter, VolumeProtection, VolumeBinding, FileExclusion, RegistryExclusion, RegistryPersist, OverlayType, OverlaySize, Servicing }

public sealed record PendingChange(
    PendingArea Area,
    string Key,            // 卷盘符 / 排除路径 / 注册表键 / "" 
    string CurrentText,    // 展示用，来自 Field.Ok 的值；这里只放已知值
    string NextText);

public enum Comparison { Equal, Pending, Unknown }

/// 三值比较：任一侧 Unavailable → Unknown（既不是 pending 也不是 equal）
public static Comparison Compare<T>(Field<T> current, Field<T> next);

/// 整快照 diff。列表类字段按元素做集合差：Next 有 Current 无 → "重启后新增"；反之 → "重启后移除"
public static IReadOnlyList<PendingChange> Diff(UwfSnapshot s);
```

**规则**
- **P1**：`Unknown` 不进 pending 列表；UI 上该项显示"未知（某侧读取失败）"，不显示 pending 徽章。理由：把读失败当"没变化"会掩盖问题，当"有变化"会误导重启。
- **P2**：`Filter.CurrentEnabled ≠ NextEnabled` 是最高优先级的 pending（决定"重启后整个保护开/关"），汇总视图置顶。
- **P3**：`OverlayLive` 无 session 维度，不参与 diff。
- **P4**：`VolumeState.CommitPending`（文档 `[read]`）作为独立标记显示，不参与 diff（它是 UWF 自己的 pending 语义，不是 current/next 差异）。

**测试**：表驱动——对每个 `PendingArea` 至少 4 行：(Ok,Ok,相等)、(Ok,Ok,不等)、(Unavailable,Ok)、(Ok,Unavailable)；列表字段再加 (增)、(删)、(增删同时)。

## 2. 输入校验（纯函数）

```csharp
public static ValidationResult ValidateFileExclusionPath(string s);   // 必须以 '\' 开头、不含盘符、非空、无非法字符
public static ValidationResult ValidateRegistryKey(string s);         // 必须以 HKEY_LOCAL_MACHINE\ / HKEY_USERS\ ... 长格式开头
public static ValidationResult ValidateOverlaySize(OverlayType t, int mb);    // 类型同 MaximumSizeMb（SInt32），非 uint；mb ≥ 0 且 Disk ≥ 1024（文档）
public static ValidationResult ValidateThresholds(uint warning, uint critical, Field<int> maxMb); // warning < critical（严格小于，参考 §UWF_Overlay.SetWarningThreshold Remarks："The warning threshold must be lower than the critical threshold."）；均 ≤ max（若 max 已知）
```
格式规则只来自参考 §"Path/string formats"；文档没规定的（如是否允许尾随 `\`）不校验，交给 WMI 返回 HRESULT（级别 d）。

## 3. 应用状态机（纯 `Step`）

### 3.1 状态

```csharp
public sealed record AppState(
    Availability Availability,        // NotStarted | Probing | Available | NotInstalled(detail) | AccessDenied(detail) | QueryFailed(detail)
    UwfSnapshot? Snapshot,            // 最近一次成功快照；Available 态下也可能为 null（首次读尚未成功）
    string? LastReadFailure,          // 最近一次读失败的摘要；成功读后清空。派生：
                                      //   Snapshot!=null && LastReadFailure!=null → "数据可能过时"横幅（陈旧）
                                      //   Snapshot==null && LastReadFailure!=null → "无法读取 UWF 状态"空态面板（03 §1）
    RefreshPhase Refresh,             // Idle | InFlight(seq)
    int NextReadSeq,                  // 单调递增；每次发出 ReadSnapshot 时分配并写进效果
    CommandPhase Command,             // None | AwaitingConfirm(cmd) | Executing(cmd) | Failed(cmd, result)
    ImmutableList<LogEntry> Log,      // 操作日志区（含级别 c 的字段降级说明）
    bool AutoRefreshEnabled);
```

### 3.2 事件（封闭集合）

**不变量 C0（纯度）**：`Step(state, event) → (state, effects)` 的输入只有这两个参数。时间、序号、随机数、IO 结果——一切非确定性——**只能作为事件的字段进入**，或从 `state` 派生；`Step` 内部不读时钟、不生成 ID、不做 IO。为此**每个事件都带 `At: DateTimeOffset`**（薄壳在构造事件时打戳），`LogEntry.At` 一律取自触发它的事件的 `At`。实现时转移表按 **状态 × 事件 穷举生成**，文档未列出的组合默认"不变、无效果"，并有一条测试断言穷举表无遗漏——下面的表是"有行为的行"，不是全表。

```csharp
/// 封闭事件集合。字段即全部载荷：转移表只能引用这里声明的字段（C0）。
public abstract record AppEvent(DateTimeOffset At)
{
    public sealed record Started(DateTimeOffset At) : AppEvent(At);
    public sealed record ProbeCompleted(DateTimeOffset At, UwfAvailability Result) : AppEvent(At);
    public sealed record RefreshRequested(DateTimeOffset At, RefreshSource Source) : AppEvent(At);   // Manual | Timer
    public sealed record SnapshotArrived(DateTimeOffset At, int Seq, UwfSnapshot Snapshot) : AppEvent(At);
    public sealed record SnapshotFailed(DateTimeOffset At, int Seq, FailureInfo Failure) : AppEvent(At);
    public sealed record AutoRefreshToggled(DateTimeOffset At, bool On) : AppEvent(At);
    public sealed record CommandRequested(DateTimeOffset At, UwfCommand Command) : AppEvent(At);     // UwfCommand = 封闭 union，一项对应 IUwfProvider 的一个方法
    public sealed record CommandConfirmed(DateTimeOffset At) : AppEvent(At);                          // 仅危险命令经过
    public sealed record CommandCancelled(DateTimeOffset At) : AppEvent(At);                          // 取消确认 / 关闭失败详情
    public sealed record CommandCompleted(DateTimeOffset At, UwfCommandResult Result) : AppEvent(At);
    public sealed record CommandThrew(DateTimeOffset At, FailureInfo Failure) : AppEvent(At);         // 薄壳/调度层自身的 bug；provider 是 total 的（01 F5），WMI 异常不会到这里
}

/// 异常的结构化摘要（薄壳从 Exception 提取；Step 不见 Exception 类型）
public sealed record FailureInfo(int HResult, string Message, string ExceptionType);
// 注：没有单独的 RebootRequested 事件——"一键跳转重启"按钮直接发 CommandRequested(RestartSystem)，走危险确认（少一个概念）。
// 注：RefreshRequested 没有 AfterCommand 来源——命令后的刷新由 CommandCompleted/CommandThrew 行直接发 ReadSnapshot 效果，不经事件。
```

### 3.3 效果（薄壳执行）

```
Probe
ReadSnapshot(seq)                        // 薄壳完成后回投 SnapshotArrived(At, seq, snapshot) / SnapshotFailed(At, seq, FailureInfo)
Invoke(UwfCommand)
StartTimer(interval) / StopTimer
ShowFailureDetails(UwfCommandResult)     // 级别 d
```

### 3.4 转移表（节选；完整表即测试用例）

| 状态（Availability / Refresh / Command） | 事件 | 新状态 | 效果 |
|---|---|---|---|
| NotStarted / – / – | `Started` | Probing（其余字段初始值：Snapshot=null，Log 空，`AutoRefreshEnabled=true`，`NextReadSeq=0`） | `Probe` |
| NotInstalled \| AccessDenied \| QueryFailed / – / – | `Started`（引导页"重新检测"） | Probing；**保留** Log（重试历史可见），Snapshot 保持 null | `Probe` |
| Probing / – / – | `Started` | 不变（探测中忽略重复点击） | 无 |
| Probing / – / – | `ProbeCompleted(Available)` | Available / InFlight(s₀) / None，`NextReadSeq=s₀+1` | `ReadSnapshot(s₀)`, `StartTimer`（若 `AutoRefreshEnabled`） |
| Probing / – / – | `ProbeCompleted(NotInstalled\|AccessDenied)` | 对应引导态 | `StopTimer`（幂等） |
| Probing / – / – | `ProbeCompleted(QueryFailed)` | QueryFailed | `StopTimer`（幂等） |
| 非 Probing / – / – | `ProbeCompleted(…)` | 不变（迟到的探测结果丢弃） | 无 |
| Available / InFlight(s) / * | `SnapshotArrived(s, snap)`（seq 相等） | Refresh=Idle，Snapshot=snap，`LastReadFailure=null`；每个 `Unavailable` 字段追加一条 Log（级别 c，去重：同字段连续失败只记一次） | 无 |
| Available / * / * | `SnapshotArrived(s', …)`，s' ≠ 当前 InFlight 序号 | **不变**（旧读丢弃，C3） | 无 |
| Available / InFlight(s) / * | `SnapshotFailed(s, f)` | Refresh=Idle，`LastReadFailure=f.Message`（Snapshot 不动：有旧值则 UI 显示陈旧横幅，为 null 则显示空态面板，见 03 §1），Log 追加 | 无 |
| Available / * / * | `SnapshotFailed(s', …)`，s' ≠ 当前序号 | 不变 | 无 |
| Available / Idle / None | `RefreshRequested` | InFlight(s)，`NextReadSeq=s+1` | `ReadSnapshot(s)` |
| Available / InFlight(s) / * | `RefreshRequested(Manual)` | InFlight(s')，s'=当前 `NextReadSeq`，`NextReadSeq=s'+1`（分配规则同上；**取代**在飞读，旧读到达后按 C3 丢弃） | `ReadSnapshot(s')` |
| Available / InFlight(s) / * | `RefreshRequested(Timer)` | 不变（合并，不叠加） | 无 |
| Available / * / * | `AutoRefreshToggled(on)` | `AutoRefreshEnabled=on` | on → `StartTimer`；off → `StopTimer` |
| Available / * / None | `CommandRequested(危险)` | Command=AwaitingConfirm | 无 |
| Available / * / None | `CommandRequested(非危险)` | Command=Executing | `Invoke` |
| Available / * / AwaitingConfirm | `CommandConfirmed` | Executing | `Invoke` |
| Available / * / AwaitingConfirm | `CommandCancelled` | None | 无 |
| Available / * / Executing | `CommandCompleted(ok)` | None；Log 追加；Refresh=InFlight(s')，`NextReadSeq=s'+1`（s'=当前 `NextReadSeq`，同 96 行分配规则） | `ReadSnapshot(s')` |
| Available / * / Executing | `CommandCompleted(fail)` | Failed(cmd,result)；Log 追加；Refresh=InFlight(s')，`NextReadSeq=s'+1`（分配规则同上） | `ShowFailureDetails`（级别 d）, `ReadSnapshot(s')` |
| Available / * / Executing | `CommandThrew(f)` | Failed(cmd, 合成的 `UwfCommandResult{Succeeded=false, HResult=f.HResult, ClassName=cmd 的目标类, MethodName=cmd 的目标方法, SystemMessage=f.Message, CompletedSteps=[]}`)；Log 追加（标"内部错误"）；Refresh=InFlight(s')，`NextReadSeq=s'+1`（分配规则同上） | `ShowFailureDetails`, `ReadSnapshot(s')`——**不自动重试**（内部错误重试无意义，且违反 C1 的单飞原则） |
| Available / * / Executing | `CommandRequested(任意)` | 不变（拒绝并发命令） | 无 |
| * / * / Failed | 用户关闭详情 → `CommandCancelled` | None | 无 |
| * / * / Failed | `CommandRequested(任意)` | 不变（先关详情） | 无 |

**并发不变量**（写在这里，测试按它构造）：
- C1：任一时刻最多一个 `Invoke` 在飞（Executing 期间拒绝新命令）。
- C2：`ReadSnapshot` 与 `Invoke` 可以并行（读不改状态；WMI 读写并发安全性由 WMI 保证，provider 无共享可变状态）。但**命令完成后必刷新**（表中 `CommandCompleted` 行）。
- C3：`SnapshotArrived(seq)` 只在 `seq == 当前 InFlight 序号` 时被接受；其余丢弃——避免慢读覆盖新读。序号由 `Step` 分配（`NextReadSeq`），薄壳只负责回带，不产生。
- C4：所有 `Step` 在单一调度器上串行执行（Avalonia UI 线程或专用单线程 `SynchronizationContext`）；`Effect` 的完成以事件形式回投，不直接改状态。

**测试**：转移表逐行 = 一个测试；C3 用"先发读 A、再发读 B、B 先到、A 后到 → 状态是 B"构造；不等待墙钟，测试调度器复现排序语义（方法论 §五.4）。

## 4. 刷新调度

- 启动：`Probe` → 成功即 `ReadSnapshot`。
- 定时：`AutoRefreshEnabled` 时每 **5 秒** `RefreshRequested(Timer)`（overlay 消耗要"实时"）；in-flight 时合并。用户可关。
- 命令后：必刷新（表中已列）。
- 首版不做增量刷新（只读 overlay）；若 5 秒全量读在 VM 上可感知卡顿（`[待里程碑 3 实测]`），再拆"live 子快照"。

## 5. 命令元数据与危险集合（二次确认，brief 横切要求）

```csharp
/// 封闭 union：一项对应 IUwfProvider 的一个方法。元数据随命令走，Step / UI / 日志都从这里取，不各自判断。
public abstract record UwfCommand(
    string ClassName,            // WMI 目标类，如 "UWF_OverlayConfig"（复合命令取主类）
    string MethodName,           // WMI 目标方法，如 "SetType"（复合命令取第一步）
    bool IsDangerous,            // true → 走 AwaitingConfirm
    bool TakesEffectAfterReboot, // true → 成功日志附"重启后生效"
    string ProductDescription)   // 产品语言，用于确认弹窗与日志，如 "禁用 UWF 保护"
{
    public sealed record DisableFilter() : UwfCommand("UWF_Filter", "Disable", IsDangerous: true, TakesEffectAfterReboot: true, "禁用 UWF 保护");
    public sealed record SetOverlayThresholds(uint WarningMb, uint CriticalMb) : UwfCommand("UWF_Overlay", "SetWarningThreshold", false, false /*[待 VM 确认]*/, "设置 overlay 阈值");
    // …其余每个 IUwfProvider 方法一项，字段值来自参考文件"takes effect after restart"清单
}
```

危险（`IsDangerous=true`）：`DisableFilter`, `UnprotectVolume`, `CommitFile`, `CommitFileDeletion`, `CommitRegistry`, `CommitRegistryDeletion`, `ShutdownSystem`, `RestartSystem`, `SetOverlayType`, `SetOverlayMaximumSize`（后两者因文档前置条件"UWF 须已禁用"且影响下次启动）。是否危险是命令的属性，不是 UI 的判断。

## 6. 日志区条目

```csharp
public sealed record LogEntry(DateTimeOffset At, LogLevel Level, string Text, string? Detail);   // At = 触发事件的 At（C0），Step 不读时钟
```
- 级别 c：`"读取 UWF_Overlay.OverlayConsumption 失败：<Detail>；该字段显示为不可用"`。
- 级别 d：`"UWF_OverlayConfig.SetType 返回 0x80070005"` + Detail 为系统消息原文。
- 命令成功：`"已提交：<ProductDescription>；重启后生效"`（后半句仅当 `TakesEffectAfterReboot`，来自 §5 元数据，非猜测）。
