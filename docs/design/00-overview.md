# Groundhog 设计文档 — 总览与决策项（里程碑 1）

> 状态：**草案，待 Owner 验收**。本目录是 brief"工作方式 1"要求的第一个交付物：接口定义、状态模型、UI 线框、测试场景清单。
> 事实来源：[docs/project-brief.md](../project-brief.md)（需求）与 [docs/reference/uwf-wmi-reference.md](../reference/uwf-wmi-reference.md)（Microsoft Learn 誊写，每条带 URL）。
> 凡参考文件标 `[文档未说明]` 的行为，本设计一律标 **`[待 VM 确认]`**，并汇总在 §4 的"实验 0"里——不猜。

## 0. 文件索引

| 文件 | 内容 | 对应 brief 交付物 |
|---|---|---|
| [01-provider-interface.md](01-provider-interface.md) | `IUwfProvider` 接口、数据模型（`Field<T>`）、错误四级、Mock 故障注入 | IUwfProvider 接口定义 |
| [02-state-model.md](02-state-model.md) | 快照 / diff / pending 纯函数、刷新与命令状态机（转移表） | 状态模型 |
| [03-ui-wireframes.md](03-ui-wireframes.md) | 页面清单、双状态并排布局、错误呈现、危险操作确认 | UI 线框 |
| [04-test-scenarios.md](04-test-scenarios.md) | 单元 / 契约 / VM 矩阵场景，场景数据文件格式 | 测试场景清单 |

## 1. 三个设计不变量（一切细节从这里推出）

1. **失败住在数据模型里，不住在渲染层。** 每个从 WMI 读出的叶子值都是 `Field<T> = Ok(value) | Unavailable(reason)`。渲染层拿不到"原始数字 + 一个错误标记"，只拿得到 `Field<T>`——所以"把 -1 渲染进数值位"在类型上不可能发生。这是 brief 死法 2、3 的结构性解法（方法论 §三.6："让不变量住进数据模型"）。
2. **双 session 是数据模型的一等维度。** 快照里每个可配置项都成对出现（`Current` / `Next`），diff 是纯函数 `Diff(snapshot) → PendingChange[]`。UI 只是把这个对渲染出来；pending 标记不是 UI 状态，是 diff 的输出。
3. **provider 是唯一的 WMI 边界。** 所有 WMI 特有的怪癖（每卷两实例、`GetExclusions` 返回 null 表示空、`CurrentSession=false` 才能写、`RestartSystem` 文档矛盾）在 `WmiUwfProvider` 内部吸收；接口以上的代码不知道 WMI 存在。测试 CLI 与 GUI 共用这个 provider（brief 要求）。

## 2. 关键裁决（已定，可在验收时推翻）

| # | 裁决 | 依据 |
|---|---|---|
| D1 | 实例枚举用"枚举整类 + 在 C# 里按类型化 key 过滤"，**不用 WQL 字符串过滤**（`UWF_OverlayConfig` 也不用）。 | 官方示例除 OverlayConfig 外全是客户端过滤；WQL 拼字符串是 locale/转义风险面，而枚举结果本来就只有 2 个实例。 |
| D2 | 写操作一律调**方法**（`Protect()`、`AddExclusion()`…），只有 `PersistDomainSecretKey` / `PersistTSCAL` 没有对应方法，才走属性写。 | 参考 §"Cross-cutting facts"：所有官方示例在有方法时都用方法；属性写是否生效 `[待 VM 确认]`。 |
| D3 | 写操作只针对 `CurrentSession=false` 实例；provider 在调用前断言这一点（不是靠调用方记住）。 | brief 已知坑 1；对 `CurrentSession=true` 写的行为 `[文档未说明]`——所以结构上禁止。 |
| D4 | `GetExclusions` 返回 null → provider 归一化为空列表；接口以上永远不见 null。 | 参考 §C：文档明写"设为 null"。 |
| D5 | `UWF_OverlayConfig.MaximumSize` 文档为 SInt32；provider 读到负值 → `Unavailable(OutOfRange)`，不渲染。 | brief"任何位置不出现负数 MB"；负值语义 `[文档未说明]`。 |
| D6 | overlay 分段条分母 = `OverlayConsumption + AvailableSpace`（同一类同一次读取），`MaximumSize` 只做旁注对照。 | 两者关系 `[文档未说明]`；用同源数据画图避免跨类不一致产生的负数段。 |
| D7 | 安全重启：首选 `UWF_Filter.RestartSystem()`；`[待 VM 确认]` 它在 WMI 上是否可调（文档 B7 自相矛盾）。若不可调，按 brief 例外用 `shutdown.exe /r /t 0`（不解析输出）。安全关机用 `UWF_Filter.ShutdownSystem()`（文档无矛盾）。 | 参考 B7。 |
| D8 | 错误四级映射见 01 §4；HRESULT **不做 UWF 专属解释**（文档没有表），只呈现原始 HRESULT + 方法名 + 类名 + `Win32Exception` 系统消息（系统消息本身是本地化文本，只展示、不参与逻辑）。 | 参考 §C："No UWF-specific HRESULT table"。 |
| D9 | 状态模型的刷新/命令流程写成纯 `Step(state, event) → (state, effects)`，转移表测试；UI 只订阅 state。 | 方法论 §五.3。 |
| D10 | 里程碑 2 拆成 3 个 PR：(a) 数据模型 + `IUwfProvider` + `MockUwfProvider` + 契约测试；(b) `WmiUwfProvider` + 同一套契约测试对真机跑的 CLI 入口；(c) 实验 0 脚本与结果入库。 | 方法论 §七.3。 |

## 3. 需要 Owner 裁决的事项（每项三件套）

### Q1. HORM 状态显示——WMI 参考里没有任何 HORM 成员

brief MVP 要求"HORM 状态显示"，但官方 9 个类都没有 HORM 属性/方法（参考 A8）。

- **选项 A（推荐）：先做实验 0 再定。** 在 VM 上枚举 `UWF_Filter` 的真实 MOF；若存在未文档化的 HORM 属性，按 D2 原则只读它；若不存在，回到 B/C。
  - 你会经历：里程碑 2 开始前多一次 VM 实验（分钟级）；HORM 在设计里先占位为 `Field<bool> HormEnabled`，可能最终显示"不可用（此系统不暴露）"。
  - 最强反方：不该为一个"可能存在"的属性占位；占位本身是复杂度。
  - 什么证据能证明我错：实验 0 枚举结果里没有 HORM 成员，且你决定 MVP 必须显示 HORM——那 A 只是拖延了 B/C 的裁决。
- **选项 B：HORM 移出 MVP。** 只在"明确不做"里加一条。
  - 你会经历：界面上没有 HORM 一栏。
  - 最强正方：HORM 是 IoT 部署期一次性配置（`uwfmgr filter enable-horm`），不是日常操作；brief 的定位是"替代日常操作"。
  - 什么证据能证明这选择错：你的目标机器实际启用了 HORM，运维需要看它——问一下现场就知道。
- **选项 C：HORM 通过 `uwfmgr.exe` 只读探测。** 违反 brief 硬约束"禁止 shell 调 uwfmgr"，需要你明确开例外。
  - 你会经历：多一条例外；引入文本解析（本地化风险，正是死法 1）。
  - 最强正方：只读、只解析一个 on/off 关键字。
  - 什么证据能证明这选择错：中文系统上该行输出被本地化——一台 zh-CN VM 上跑一次 `uwfmgr filter get-config` 就能验。

### Q2. "persistent overlay 开关"（brief 测试矩阵的一个维度）——WMI 参考里同样没有

`UWF_OverlayConfig` 只有 `Type` 与 `MaximumSize`；参考里没有 persistent / passthrough 相关成员。

- **选项 A（推荐）：与 Q1 同批进实验 0。** 若真实 MOF 有该成员 → 进 MVP 与矩阵；若没有 → 矩阵去掉这一维（RAM/Disk × zh-CN/en-US = 4 个基础场景）。
- **选项 B：矩阵直接去掉这一维。**
- 三件套与 Q1 同构，不重复；可证伪把手都是"实验 0 的枚举结果"。

### Q3. 注册表排除的调用类——文档自相矛盾（参考 A3/B5）

`UWF_ExcludedRegistryKey` 页说用 `UWF_Volume.AddExclusion`，`UWF_RegistryFilter` 页说用 `UWF_RegistryFilter.AddExclusion`。brief 站 RegistryFilter。

- **裁决建议：按 brief（`UWF_RegistryFilter`），不需要你介入**——`UWF_Volume.AddExclusion` 的参数名是 `FileName`，语义就是文件；`UWF_ExcludedRegistryKey` 页大概率是文档笔误。列在这里只是让你知道有这条矛盾。实验 0 会顺手验证。

### Q4. 里程碑 5 的 VM 是否已就绪

不影响本文档，但决定实验 0 何时能跑（见 §4）。之前问过一次未答。

## 4. 实验 0（`[待 VM 确认]` 清单，一次跑完）

一个只读脚本，通过 WMI 对象接口（不是 uwfmgr）在 VM 上执行，输出 JSON：

1. 枚举 `root\standardcimv2\embedded` 下所有类名；确认 `UWF_ServicingHelper` 是否存在。
2. 对 9 个 UWF 类，导出真实 MOF（属性名/类型/限定符、方法签名）；对照参考文件的 B1–B12 逐条判定；重点：`ServicingEnabled` vs `ServiceEnabled`（B3）、`Id` vs `ID`（B4）、是否有 HORM / persistent 成员（Q1/Q2）。
3. 每类实例数与 key 值。
4. 在 UWF **未安装**的 checkpoint 上重复步骤 1：记录抛什么异常/HRESULT（区分"命名空间不存在"与"类不存在"）——这是错误级别 (a) 的探测依据（参考 A7）。
5. 以非管理员运行步骤 1–3：记录异常形状——错误级别 (b) 的探测依据。
6. `UWF_Filter.RestartSystem()` 是否可调（B7）；`UWF_Overlay.SetWarningThreshold` 是否需要重启才生效（文档未说明）。
7. `PersistTSCAL` 属性直接写是否生效（D2）。

结果文件入库到 `docs/reference/experiment-0-results.json`（里程碑 2 PR (c)），设计文档里所有 `[待 VM 确认]` 据此收口。

## 5. 明确不做（沿 brief）

多机远程、profile 导入导出、overlay 文件明细、多语言 UI、`UWF_Servicing.UpdateWindows`（brief 的"servicing 模式"只含开关）。
