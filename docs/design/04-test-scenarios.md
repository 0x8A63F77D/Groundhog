# 04 — 测试场景清单

> 三层：单元（纯函数，机器门禁）→ 契约（同一套测试跑 Mock 与 Wmi）→ VM 矩阵（闭环脚本，无人工干预）。测试框架 xUnit（.NET 默认，无特殊需求）。

## 1. 单元测试（里程碑 2 PR (a) / 里程碑 3）

| 编号 | 对象 | 场景 | 断言 |
|---|---|---|---|
| U1 | `Field<T>` 构造 | UInt32 CIM 值 0 / 1 / uint.MaxValue | 均为 `Ok`，原值 |
| U2 | `Field<T>` 构造 | CIM 值类型不符（如 string） | `Unavailable(WrongCimType)` |
| U3 | `Field<int>`（MaximumSize） | SInt32 值 -1 | `Unavailable(OutOfRange)`；**任何渲染路径拿不到 -1** |
| U4 | 列表归一化 | `GetExclusions` 返回 null | `Ok(空列表)` |
| U5 | `Compare` | (Ok a, Ok a) / (Ok a, Ok b) / (Unavailable, Ok) / (Ok, Unavailable) | Equal / Pending / Unknown / Unknown |
| U6 | `Diff` | 每个 `PendingArea` 至少 4 行（见 02 §1） | 表驱动 |
| U7 | `Diff` 列表 | Next 多一项 / 少一项 / 增删同时 / 顺序不同内容相同 | 新增 / 移除 / 两条 / 无 pending |
| U8 | 校验函数 | 合法与非法路径、注册表键、Disk 上限 <1024、warning≥critical（含相等，参考文档"warning 必须严格小于 critical"） | 对应 `ValidationResult` |
| U9 | `Step` 转移表 | 02 §3.4 每一行 | 新状态 + 效果列表精确相等 |
| U10 | `Step` 并发不变量 C1 | Executing 中再来 CommandRequested | 状态不变、无效果 |
| U11 | `Step` 并发不变量 C3 | 发读 A(seq=1)、手动刷新发读 B(seq=2)、B 先到、A 后到 | 最终 Snapshot = B；A 到达时状态不变 |
| U11b | `Step` `CommandThrew` | Executing 中收到 `CommandThrew` | Command=Failed（合成结果）、效果含 `ShowFailureDetails` + `ReadSnapshot`；随后 `CommandCancelled` 回 None，可再发命令（不卡死） |
| U11c | `Step` 陈旧标记 | 有快照后 `SnapshotFailed(当前 seq)` | `LastReadFailure` 非空、Snapshot 保留；下一次 `SnapshotArrived` 后为 null |
| U11e | `Step` 首次读失败 | Probe 成功、Snapshot 仍 null 时 `SnapshotFailed(当前 seq)` | Availability 仍 Available、Snapshot null、`LastReadFailure` 非空（UI 空态面板条件成立）；`RefreshRequested(Manual)` 可再次发读 |
| U11f | `Step` 启动与重试 | `NotStarted` 收 `Started`；`NotInstalled` 收 `Started`；`Probing` 收 `Started` | 前两者 → Probing + 效果 `Probe`（Log 保留）；第三者不变无效果 |
| U11g | `Step` 迟到探测 | Available 态收 `ProbeCompleted(NotInstalled)` | 不变 |
| U11d | `Step` 自动刷新开关 | `AutoRefreshToggled(false)` / `(true)` | 效果 `StopTimer` / `StartTimer`；状态字段同步 |
| U12 | 危险集合 | 每个危险命令 | 先进 AwaitingConfirm；非危险直接 Executing |
| U13 | 渲染守卫 | 遍历所有数值渲染函数，输入 `Unavailable` | 输出为"不可用"文案，且不含 `-` 开头的数字（属性测试：任意 `Field<uint>`/`Field<int>` 输入，输出串不匹配 `-\d`） |
| U14 | 日志去重 | 同字段连续两次快照都 Unavailable | 只一条日志 |

## 2. 契约测试（同一测试类，两个 fixture）

`IUwfProviderContract` 抽象测试基类；`MockProviderTests : IUwfProviderContract`（CI 跑），`WmiProviderTests : IUwfProviderContract`（仅 VM 上、`[Trait("RequiresUwf")]`）。

| 编号 | 场景 | 断言 |
|---|---|---|
| K1 | `ProbeAsync` 在正常环境 | `Available` |
| K2 | `ReadSnapshotAsync` | 每卷成对；`Filter` 两字段 `Ok`；`OverlayLive` 四字段 `Ok` 且 `uint` |
| K3 | `AddFileExclusionAsync` 后重读 | Next 列表含该路径，Current 不含（未重启） |
| K4 | `RemoveFileExclusionAsync` | 对称 |
| K5 | `AddRegistryExclusionAsync` / Remove | 同 K3/K4 |
| K6 | `SetOverlayTypeAsync` 在 Current 已启用时 | `Succeeded=false`，非零 HRESULT（Mock 按 `EnforceDocumentedPreconditions`；Wmi 真值 `[待 VM 确认]`——若 WMI 实际允许，此行改为记录事实并升级） |
| K7 | `SetWarningThresholdAsync(x)` 后重读 | `WarningThresholdMb == x`（阈值是否需重启 `[待 VM 确认]`——若需，断言改到重启后） |
| K8 | `ProtectVolumeAsync` / `Unprotect` | Next.Protected 变化，Current 不变 |
| K9 | **配置类**写路径必为 Next 实例（`CommitFile*`/`CommitRegistry*` 不适用——D3 已排除，见 01 §4 备注：Mock 立即作用于 Current） | 任何配置类命令后 Current 侧字段不变（Mock 可直接断言；Wmi 靠 K3/K8 覆盖） |
| K10 | 命令 HRESULT 原样 | Mock 注入 0x80070005 → `HResult == unchecked((int)0x80070005)` |

Mock 专属（故障注入）：
| M1 | `Faults.ProbeResult = FeatureNotInstalled` | 应用进引导页态，不抛 |
| M2 | `Faults.UnavailableFields = {"OverlayLive.ConsumptionMb"}` | 快照其余全 `Ok`；状态机 Log 有一条级别 c |
| M3 | 逐个把每个字段路径设为 Unavailable（参数化遍历） | 每次只有该字段 Unavailable（brief 验收"注入任意单字段读取失败，其余正常"） |
| M4 | `SimulateReboot` | Next 值迁到 Current；`Diff` 为空 |
| M5 | `Faults.Latency = 2s` + 定时刷新 | 不并发两次读（用测试调度器计数，不用墙钟） |

## 3. UI 测试（里程碑 4，对 Mock，headless Avalonia）

| 编号 | 场景 | 断言 |
|---|---|---|
| G1 | 启动 + Mock 正常 | 顶栏两胶囊文案来自 Mock 值 |
| G2 | M1 | 引导页可见、主界面不可见 |
| G3 | M3 遍历 | 对应控件显示"不可用"，其它控件文案正常；页面不整体灰掉 |
| G4 | Next ≠ Current | 徽章可见；`Diff` 为空时导航计数不可见 |
| G5 | 危险命令 | 确认弹窗出现；取消后无 `Invoke` |
| G6 | 命令失败 | 失败详情弹窗含 HRESULT 十六进制与方法名 |
| G7 | 全界面文本扫描 | 渲染树所有文本节点不匹配 `-\d+\s*MB` |

只 settle 在期望文本或 fake 的实测调用上，无墙钟等待（方法论 §五.4）。视觉保真不写自动化断言。

## 4. VM 闭环矩阵（里程碑 5–6）

### 4.1 基础设施（沿 brief 协议）
- host：`Restore-VMCheckpoint` + `Start-VM` 包装脚本；SSH 可达轮询（短 burst，单次 ≤4 分钟）。
- VM：D: 为不受保护 VHDX，测试 CLI 与结果 JSON 落 D:。
- 测试 CLI = 同一个 `WmiUwfProvider` 的命令行壳：`groundhog-cli apply <scenario.json>` / `groundhog-cli assert <scenario.json> --out result.json` / `groundhog-cli probe --out probe.json`。

### 4.2 场景数据文件

```json
{
  "id": "disk-overlay-zh",
  "checkpoint": "zh-CN-uwf-clean",
  "apply": {
    "filter": "enable",
    "overlay": { "type": "Disk", "maxMb": 2048, "warningMb": 1024, "criticalMb": 1800 },
    "volumes": { "C:": { "protect": true, "fileExclusions": ["\\Logs", "\\Temp"] } },
    "registry": { "exclusions": ["HKEY_LOCAL_MACHINE\\SOFTWARE\\Groundhog\\Test"] },
    "servicing": false
  },
  "expectAfterReboot": {
    "filter": { "currentEnabled": true },
    "overlay": { "type": "Disk", "maxMb": 2048, "warningMb": 1024, "criticalMb": 1800 },
    "volumes": { "C:": { "protected": true, "fileExclusions": ["\\Logs", "\\Temp"] } },
    "registry": { "exclusions": ["HKEY_LOCAL_MACHINE\\SOFTWARE\\Groundhog\\Test"] },
    "servicing": false
  }
}
```

### 4.3 矩阵

| 维度 | 取值 | 来源 |
|---|---|---|
| overlay 类型 | RAM / Disk | brief |
| 语言 checkpoint | zh-CN / en-US | brief |
| persistent overlay | 开 / 关 | brief；**`[待 VM 确认]` WMI 是否暴露（00 §3 Q2）**——不暴露则此维删除 |

基础 4（或 8）个场景之外的**单点场景**：
| S1 | 未安装 UWF 的 checkpoint 上 `probe` | `FeatureNotInstalled` |
| S2 | 非管理员运行 `probe` | `AccessDenied`（形状 `[待 VM 确认]`） |
| S3 | `SetOverlayType` 在 filter 已启用时 | 非零 HRESULT（K6 的真机版） |
| S4 | 与 `uwfmgr get-config` 一致性 | 在 VM 上另存 `uwfmgr get-config` 输出（**仅作对照工件供人读**，测试代码不解析它，不违反硬约束）；断言只对 WMI 读回值 |
| S5 | 单场景回路无人工干预跑完 | 脚本退出码 0，结果 JSON 齐全 |

### 4.4 实验 0
见 [00-overview.md §4](00-overview.md)。它跑在矩阵之前，输出决定 `[待 VM 确认]` 项的收口。

## 5. 验收标准 ↔ 测试映射（brief"验收标准"）

| brief 验收 | 覆盖 |
|---|---|
| zh-CN VM 上 MVP 操作齐全且与 `uwfmgr get-config` 一致 | 矩阵 + S4 |
| 拔掉 UWF 启动得到引导页 | M1 / G2 / S1 |
| 任意单字段读取失败其余正常 | M3 / G3 |
| 界面任何位置无负数 MB | U3 / U13 / G7（类型层 + 属性测试 + 渲染树扫描三道） |
| 矩阵全绿无人工干预 | S5 |
