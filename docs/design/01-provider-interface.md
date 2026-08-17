# 01 — `IUwfProvider` 接口与数据模型

> 设计层伪代码（C#），不是最终源码；命名与形状在里程碑 2 PR (a) 落地时以此为准，偏离需在 PR 里说明。
> 事实出处：`docs/reference/uwf-wmi-reference.md`（下文用 `[ref: 类名]` 指向对应节）。

## 1. 叶子值：`Field<T>`

```csharp
/// 一个从 WMI 读出的值。要么有值，要么带原因不可用。永远不含哨兵。
public abstract record Field<T>
{
    public sealed record Ok(T Value) : Field<T>;
    public sealed record Unavailable(FieldFailure Reason) : Field<T>;
}

public sealed record FieldFailure(
    FieldFailureKind Kind,      // ReadThrew | WrongCimType | Null | OutOfRange | NotExposed
    string ClassName,           // "UWF_Overlay"
    string PropertyOrMethod,    // "OverlayConsumption" / "GetExclusions"
    int? HResult,               // 若有
    string? Detail);            // 系统消息原文（本地化文本，只展示不判断）
```

- **不变量 F1**：provider 读每个属性都单独 try；一个属性抛异常只让**它自己**变 `Unavailable`（错误级别 c）。
- **不变量 F2**：CIM 值先按文档类型强转（`UInt32`/`Boolean`/`String`/`SInt32`），类型不符 → `Unavailable(WrongCimType)`。**不经文本**。
- **不变量 F3**：数值字段的 `Ok` 只可能包含文档类型的合法值：`UInt32` 直接进 `uint`；`SInt32` 字段（仅 `UWF_OverlayConfig.MaximumSize`）负值 → `Unavailable(OutOfRange)`（裁决 D5）。
- **不变量 F4**：列表字段的 `Ok` 永远是非 null 列表（`GetExclusions` 的 null 归一化为空，裁决 D4）。

## 2. 快照

```csharp
public enum UwfSession { Current, Next }

/// 一次完整读取的不可变结果。所有可配置项成对出现。
public sealed record UwfSnapshot(
    DateTimeOffset TakenAt,
    FilterState Filter,
    IReadOnlyList<VolumePair> Volumes,
    SessionPair<RegistryFilterState> Registry,
    SessionPair<OverlayConfigState> OverlayConfig,
    OverlayLiveState OverlayLive,          // 单例，无 session 维度 [ref: UWF_Overlay]
    SessionPair<ServicingState> Servicing,
    Field<bool> HormEnabled);              // [待 VM 确认] 见 00 §3 Q1；不暴露则 Unavailable(NotExposed)

public sealed record SessionPair<T>(T Current, T Next);

public sealed record FilterState(                    // [ref: UWF_Filter]
    Field<bool> CurrentEnabled,
    Field<bool> NextEnabled);

public sealed record VolumeKey(string DriveLetter, string VolumeName);   // 两个 key 一起带，UI 暴露绑定方式

public sealed record VolumePair(VolumeKey Key, VolumeState Current, VolumeState Next);

public sealed record VolumeState(                    // [ref: UWF_Volume]
    Field<bool> Protected,
    Field<bool> BindByDriveLetter,                   // true=按盘符（松），false=按卷名（紧）
    Field<bool> CommitPending,
    Field<IReadOnlyList<string>> FileExclusions);    // 经 GetExclusions()，非 WQL

public sealed record RegistryFilterState(            // [ref: UWF_RegistryFilter]
    Field<bool> PersistDomainSecretKey,
    Field<bool> PersistTSCAL,
    Field<IReadOnlyList<string>> Exclusions);        // 经 GetExclusions()

public enum OverlayType { Ram = 0, Disk = 1 }        // [ref: UWF_OverlayConfig.Type]

public sealed record OverlayConfigState(
    Field<OverlayType> Type,
    Field<int> MaximumSizeMb);                       // 文档 SInt32；负值 → Unavailable(OutOfRange)

public sealed record OverlayLiveState(               // [ref: UWF_Overlay]，四个 UInt32/MB
    Field<uint> ConsumptionMb,
    Field<uint> AvailableSpaceMb,
    Field<uint> WarningThresholdMb,
    Field<uint> CriticalThresholdMb);

public sealed record ServicingState(                 // [ref: UWF_Servicing]
    Field<bool> Enabled);                            // 属性真名 ServicingEnabled/ServiceEnabled [待 VM 确认]（B3）
```

**卷配对规则**：按 `(DriveLetter, VolumeName)` 把 `CurrentSession=true/false` 两个实例配成 `VolumePair`。只有一侧存在时，另一侧的所有字段为 `Unavailable(Null)`（不丢卷、不猜）。

## 3. 可用性探测（错误级别 a / b）

```csharp
public abstract record UwfAvailability
{
    public sealed record Available : UwfAvailability;
    public sealed record FeatureNotInstalled(string Detail) : UwfAvailability;   // 命名空间/类不存在
    public sealed record AccessDenied(string Detail) : UwfAvailability;          // 权限不足
    public sealed record QueryFailed(int HResult, string Detail) : UwfAvailability; // 其他：区别于"未安装"
}
```

- 探测顺序：连接命名空间 → 枚举 `UWF_Filter` 类定义 → 读一个实例。三步各自失败映射到上面三种。
- **具体异常/HRESULT 形状 `[待 VM 确认]`**（实验 0 步骤 4、5）。设计只定接口，映射表在实验后填。
- 启动时探测一次；`QueryFailed` 提供"重试"；`FeatureNotInstalled` / `AccessDenied` 进引导页（03 §2）。

## 4. 命令与结果（错误级别 d）

```csharp
public sealed record UwfCommandResult(
    bool Succeeded,
    int HResult,                 // 方法返回值原样；0 = S_OK
    string ClassName,
    string MethodName,           // 失败时 = 失败的那一步的 WMI 方法名
    string? SystemMessage,       // 本地化文本，只展示
    IReadOnlyList<string> CompletedSteps);   // 复合命令里失败前已成功的 WMI 方法名（单步命令恒为空）

public interface IUwfProvider
{
    Task<UwfAvailability> ProbeAsync(CancellationToken ct);
    Task<UwfSnapshot> ReadSnapshotAsync(CancellationToken ct);

    // 全局 [ref: UWF_Filter]
    Task<UwfCommandResult> EnableFilterAsync(CancellationToken ct);           // 生效需重启
    Task<UwfCommandResult> DisableFilterAsync(CancellationToken ct);          // 生效需重启
    Task<UwfCommandResult> ShutdownSystemAsync(CancellationToken ct);         // 立即
    Task<UwfCommandResult> RestartSystemAsync(CancellationToken ct);          // 裁决 D7，[待 VM 确认]

    // 卷 [ref: UWF_Volume] —— 配置类：作用于 CurrentSession=false 实例（裁决 D3）
    Task<UwfCommandResult> ProtectVolumeAsync(VolumeKey v, CancellationToken ct);
    Task<UwfCommandResult> UnprotectVolumeAsync(VolumeKey v, CancellationToken ct);
    Task<UwfCommandResult> SetBindByDriveLetterAsync(VolumeKey v, bool byDriveLetter, CancellationToken ct); // 参数名/语义 B2 [待 VM 确认]
    Task<UwfCommandResult> AddFileExclusionAsync(VolumeKey v, string volumeRelativePath, CancellationToken ct);
    Task<UwfCommandResult> RemoveFileExclusionAsync(VolumeKey v, string volumeRelativePath, CancellationToken ct);
    // 卷 —— 提交类：立即写穿到底层卷，不产生 pending；作用于哪个实例文档未说明 [待 VM 确认]（实验 0 步骤 8），
    // 不受 D3 约束。Mock 按"作用于 Current 实例、立即生效"实现并在代码注释标出。
    Task<UwfCommandResult> CommitFileAsync(VolumeKey v, string volumeRelativePath, CancellationToken ct);
    Task<UwfCommandResult> CommitFileDeletionAsync(VolumeKey v, string volumeRelativePath, CancellationToken ct);

    // 注册表 [ref: UWF_RegistryFilter]
    Task<UwfCommandResult> AddRegistryExclusionAsync(string key, CancellationToken ct);
    Task<UwfCommandResult> RemoveRegistryExclusionAsync(string key, CancellationToken ct);
    Task<UwfCommandResult> CommitRegistryAsync(string key, string? valueName, CancellationToken ct);
    Task<UwfCommandResult> CommitRegistryDeletionAsync(string key, string? valueName, CancellationToken ct);
    Task<UwfCommandResult> SetPersistDomainSecretKeyAsync(bool on, CancellationToken ct);  // 属性写 [待 VM 确认]
    Task<UwfCommandResult> SetPersistTscalAsync(bool on, CancellationToken ct);            // 属性写 [待 VM 确认]

    // Overlay 配置 [ref: UWF_OverlayConfig]，文档前置条件：当前 session UWF 必须已禁用
    Task<UwfCommandResult> SetOverlayTypeAsync(OverlayType type, CancellationToken ct);
    Task<UwfCommandResult> SetOverlayMaximumSizeAsync(int sizeMb, CancellationToken ct);   // 类型与 MaximumSizeMb（SInt32）一致，非 uint，避免溢出；非负 且 Disk 型 ≥1024

    // Overlay 阈值 [ref: UWF_Overlay] —— 复合方法：一次设置两个阈值。WMI 只有 SetWarningThreshold / SetCriticalThreshold 两个单独方法，
    // 且文档要求 warning < critical；provider 内部按安全顺序调两次（两者都升 → 先 critical 后 warning；都降 → 先 warning 后 critical；
    // 一升一降 → 任意），第一次失败即停止并返回该次的结果（MethodName 指明是哪一个，CompletedSteps 列出已成功的那一步）。
    // **部分失败不回滚**：回滚本身也是一次可能失败的 WMI 调用，且命令后必刷新会把真实状态带回 UI；诚实呈现优于制造第二个失败点。
    // 顺序规则是 WMI 怪癖，按不变量 3 住在 provider 里，
    // 状态模型只见一个命令（满足 C1 单飞）。是否需重启 [待 VM 确认]。
    Task<UwfCommandResult> SetOverlayThresholdsAsync(uint warningMb, uint criticalMb, CancellationToken ct);

    // Servicing [ref: UWF_Servicing]
    Task<UwfCommandResult> EnableServicingAsync(CancellationToken ct);
    Task<UwfCommandResult> DisableServicingAsync(CancellationToken ct);
}
```

- **命令方法是 total 的（不变量 F5）**：非零 HRESULT 走 `Succeeded=false`；WMI 调用抛出的**任何异常**也在 provider 内部捕获并转成 `Succeeded=false`（`HResult=ex.HResult`、`SystemMessage=ex.Message`、`MethodName`=实际抛出的那一步、`CompletedSteps`=之前已成功的步骤）。provider 的公开方法对调用方永不抛——所以状态模型的 `CommandThrew` 只对应薄壳自身的 bug，那时没有任何 WMI 调用进度可报，用命令的静态元数据合成结果就是全部真相。复合命令按步 try/catch，进度信息永远准确。
- **命令不自动刷新**：刷新由状态模型调度（02 §3），provider 保持无状态。
- 路径/键格式沿官方示例（参考 §"Path/string formats"）：文件路径卷相对、以 `\` 开头、无盘符；注册表键长格式 `HKEY_LOCAL_MACHINE\...`。格式校验在状态模型层做（纯函数），provider 不做二次校验。

## 5. `WmiUwfProvider` 实现要点

- 用 `Microsoft.Management.Infrastructure`（MMI）；理由：类型化 `CimInstance`/`CimMethodResult`，无 COM interop 遗留，且 .NET 8 单文件发布路径更干净。`System.Management` 作为退路，接口不变。**`[待里程碑 2 PR (a) 验证]`：MMI 在单文件 self-contained 下的可用性**（若不行换 `System.Management`，本设计不受影响——这正是把选择藏在 provider 后面的目的）。
- 每次 `ReadSnapshotAsync` = 枚举 6 个类 + 每卷/注册表调 `GetExclusions()`；不做增量。
- 实例选择：枚举 → 在 C# 里按 `CurrentSession`（bool）与 key 过滤（裁决 D1）。
- 方法调用：`InvokeMethod` → 返回值 `UInt32` 原样进 `HResult`（`unchecked((int)ret)`），不解释。
- 线程：MMI 调用同步阻塞 → 全部 `Task.Run` 到线程池；provider 不持锁（无共享可变状态）。

## 6. `MockUwfProvider` 与故障注入（brief：不是可选项）

```csharp
public sealed class MockUwfProvider : IUwfProvider
{
    public MockWorld World { get; }        // 可变的内存世界：Current/Next 两套配置 + overlay live 值
    public FaultPlan Faults { get; }       // 可随时改
    public void SimulateReboot();          // Next → Current 迁移；FileExclusions/Registry/OverlayConfig/Servicing/Filter 全部照文档"重启后生效"规则搬
}

public sealed class FaultPlan
{
    public UwfAvailability ProbeResult = Available;                        // 模拟 (a)(b)
    public HashSet<string> UnavailableFields = new();                      // 路径如 "OverlayLive.ConsumptionMb"、"Volumes[C:].Next.FileExclusions" → 该字段返回 Unavailable
    public Dictionary<string, int> MethodHResults = new();                 // "SetOverlayType" → 0x80070005 等（业务失败路径）
    public Dictionary<string, Exception> MethodThrows = new();             // "SetCriticalThreshold" → 抛该异常（F5 异常路径；键为 WMI 方法名，复合命令可指定第几步）
    public bool EnforceDocumentedPreconditions = true;                     // SetOverlayType/MaxSize 在 Current.Filter 启用时返回非零
    public TimeSpan Latency = TimeSpan.Zero;                               // 测刷新调度用
}
```

- Mock 的 `SimulateReboot` 迁移规则来自参考 §"takes effect after restart"，逐条对应；阈值是否需重启 `[待 VM 确认]`——Mock 先按"立即生效"实现并在代码注释里标出（实验 0 后对齐）。
- Mock 与 Wmi 共用一套**契约测试**（04 §2）：同一测试类，两个 fixture；WMI fixture 只在 VM 上跑。
