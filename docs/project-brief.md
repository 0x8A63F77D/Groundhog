# Groundhog — 项目提示词

> Owner 交付的设计包（2026-08-17），原样保存。本文件是需求的**唯一权威出处**；与本文件冲突的情况停下来报告冲突本身，不静默变通（见"工作方式"第 5 条）。

你要开发 Groundhog:一个 Windows Unified Write Filter (UWF) 的单机图形管理工具,目标是替代 `uwfmgr.exe` 的日常操作。名字取自《土拨鼠之日》:UWF 保护下的机器每次重启都醒在同一个早晨,只有例外列表里的东西带着记忆穿越循环。

## 背景:为什么要重造这个轮子

现有工具(包括几个开源 GUI)在真实环境里的死法已经查明,它们直接构成本项目的硬性约束:

1. 在中文 Windows 上崩溃。 有工具报 `decode WMI row: field 'Protected' has the wrong type: illegal byte sequence`——因为它经由文本中介(shell 出去调 wmic/PowerShell 再解析 stdout)读 WMI,中文系统的 GBK 输出打死了 UTF-8 解码。目标用户的机器就是 Windows 10 IoT Enterprise LTSC 2021 中文版。
2. 把错误码当数据渲染。 有工具显示"当前覆盖层消耗: -1 MB"。`UWF_Overlay.OverlayConsumption` 是 UInt32,-1 只能来自工具自己的错误路径(哨兵值或 0xFFFFFFFF 按有符号解释)。
3. 单字段失败导致整个面板灰掉,没有按字段降级。
4. 双状态语义被 UI 掩盖。 UWF 绝大多数配置只写入 next session,current session 是只读快照。不标 pending 的 UI 让用户以为"改了没生效"。

## 硬性约束(不可协商)

* 全程通过 WMI 对象接口访问 UWF:命名空间 `root\standardcimv2\embedded`,用 `System.Management` 或 `Microsoft.Management.Infrastructure` 拿类型化 CIM 值。绝对禁止 shell 出去调 `uwfmgr.exe` / `wmic` / PowerShell 再解析文本输出——那是上面第 1 类死法的根源。唯一例外:安全关机/重启可以调系统 API 或 `shutdown.exe`(不解析其输出)。
* locale 无关:所有逻辑不得依赖任何本地化字符串。测试矩阵必须包含 zh-CN 和 en-US。
* 错误分四级,分别呈现:(a) UWF 可选功能未安装(WMI 命名空间/类不存在);(b) 权限不足;(c) 单个属性读取失败;(d) 方法调用返回非零 HRESULT。(a)(b) 给引导页,(c) 只降级对应字段并在日志区说明,(d) 弹操作失败详情。任何情况下不得把哨兵值或错误码渲染进数值位。
* UI 第一等公民是双状态对照:current session(只读)与 next session(可编辑)并排,diff 出的项显式标 pending reboot。
* 提权:manifest `requireAdministrator`。
* 发布:.NET 8+,单文件 self-contained exe。
* UI 框架:Avalonia(仅跑 Windows,但沿用团队既有经验)。

## 架构分层

```
UI (Avalonia, MVVM)
  ↓
状态模型层:current/next 快照、diff、pending 计算、刷新调度
  ↓
IUwfProvider 接口
  ├── WmiUwfProvider(真实现)
  └── MockUwfProvider(内存实现,含可注入的故障模式)
```

MockUwfProvider 不是可选项。 UI 开发全程对着 mock 跑,真机/VM 只用于验收 WMI 层。mock 必须能模拟:双 session 状态、重启(next→current 迁移)、功能未安装、单字段读失败、方法返回错误——这样错误分级的 UI 路径不依赖真机就能开发和测试。

## WMI 层已知的坑(前人踩过,直接绕开)

* `UWF_Volume` 每卷两个实例,以 `CurrentSession` 布尔为 key 区分;只能修改 CurrentSession=false 的那个。
* 文件排除列表不能用 WQL 枚举 `UWF_ExcludedFile`(会返回空)——必须调 `UWF_Volume.GetExclusions()` 取嵌入对象。注册表排除同理走 `UWF_RegistryFilter` 的方法。
* `UWF_Overlay` 的 OverlayConsumption/AvailableSpace/两个阈值都是 UInt32、单位 MB;读失败或值异常时显示"不可用",不显示数字。
* 卷绑定方式(BindByDriveLetter vs volume name)要在 UI 暴露。
* 功能未启用时整个命名空间查不到类,启动时先探测并区分"未安装"与"查询失败"。

## MVP 功能范围

全局:filter 启用/禁用、servicing 模式、HORM 状态显示、安全关机/安全重启。 每卷:保护开关、文件 exclusion 增删查、commit file / commit deletion。 注册表:exclusion 增删查、commit registry key。 Overlay:类型(RAM/Disk)、最大大小、警告/严重阈值设置;consumption/available 实时显示,画分段条并在阈值处标色。 横切:所有 pending 改动的汇总视图 + 一键跳转重启;危险操作(取消保护、commit、disable filter)二次确认;操作日志区。

明确不做(MVP 之后再议):多机远程管理、profile 导入导出、overlay 文件明细可视化(GetOverlayFiles 按目录聚合)、多语言 UI。

## 测试闭环协议

开发机通过 SSH 操控一台 Hyper-V 测试 VM(Windows IoT Enterprise LTSC,已开 UWF 功能)。约定:

* 复位手段只信 checkpoint restore,不依赖 UWF 自己的丢弃语义——被测对象就是 UWF,测试会反复开关它。host 侧提供 `Restore-VMCheckpoint` + `Start-VM` 的包装脚本。
* VM 挂第二块不受保护的 VHDX 作为 D:,测试 CLI 和结果 JSON 都放 D:,避免受保护 C: 的丢弃语义和文件拷贝时序纠缠(scp 到 C: 的二进制重启后会消失,不要在这上面浪费迭代)。
* 单场景回路:restore → 启动 → 轮询 SSH 可达 → scp 测试 CLI 到 D: → 执行配置动作(写 next session)→ 重启 → 轮询 SSH 可达 → 跑断言(读 current session 验证生效)→ 收 JSON。
* 场景定义写成数据文件(overlay 类型、阈值、exclusion 列表、期望状态),矩阵至少覆盖:RAM/Disk overlay × zh-CN/en-US(两个 checkpoint)× persistent overlay 开关。
* 测试 CLI 与 GUI 共用同一个 WmiUwfProvider——测的就是产品代码。

## 工作方式

1. 设计先行。 第一个交付物是设计文档:IUwfProvider 接口定义、状态模型、UI 线框、测试场景清单。经批准后再写实现代码。
2. 里程碑顺序:设计文档 → WMI 层 + mock + 单元测试 → 状态模型 → UI(对 mock)→ 闭环脚本 → VM 上跑通测试矩阵 → 打包。每个里程碑结束停下来等验收。
3. 文档没写清楚的行为(比如某方法在特定状态下的返回值),先查 Microsoft Learn 的 UWF WMI provider reference;查不到就在真机/VM 上做最小实验确认;仍不确定就明确说不确定并把决定交回给我。不要编造 API 行为,不要用看起来合理的猜测填空。
4. 不要自行扩大需求范围。觉得某功能值得加,提出来,等确认。
5. 遇到与本文档冲突的情况(比如某约束在实际 API 面前不成立),停下来报告冲突本身,不要静默变通。

## 验收标准

* 在 zh-CN 的 IoT Enterprise LTSC 2021 VM 上,uwfmgr 能做的 MVP 范围内操作,Groundhog 都能做,且状态显示与 `uwfmgr get-config` 一致。
* 拔掉 UWF 功能后启动,得到引导页而不是崩溃或空面板。
* 注入任意单字段读取失败,面板其余部分正常工作。
* 界面上任何位置永远不出现负数的 MB 值。
* 测试矩阵全绿,且全程无人工干预跑完。
