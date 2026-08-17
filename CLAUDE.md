# Groundhog — 会话规则

> 本文件是每会话必加载的**精简操作规则**。完整方法论（含每条规则的出处与推理）在 [docs/methodology.md](docs/methodology.md)，冲突时以完整版为准。
> 需求的唯一权威出处是 Owner 交付的 [docs/project-brief.md](docs/project-brief.md)；与之冲突时停下报告冲突本身，不静默变通。

## 项目：Groundhog

Windows UWF（Unified Write Filter）单机图形管理工具，替代 `uwfmgr.exe` 日常操作。目标机：Windows 10 IoT Enterprise LTSC 2021 **中文版**。

**硬性约束（不可协商，细节见 brief）：**
- 只经 WMI 对象接口访问 UWF（`root\standardcimv2\embedded`，`System.Management` / MMI 取类型化 CIM 值）。**禁止** shell 调 `uwfmgr.exe` / `wmic` / PowerShell 解析文本。唯一例外：安全关机/重启可调系统 API 或 `shutdown.exe`（不解析输出）。
- locale 无关；测试矩阵含 zh-CN 与 en-US。
- 错误四级分别呈现：(a) 功能未安装 → 引导页；(b) 权限不足 → 引导页；(c) 单属性读失败 → 只降级该字段 + 日志；(d) 方法非零 HRESULT → 操作失败详情。**任何位置不得把哨兵值/错误码渲染进数值位**（永不出现负数 MB）。
- UI 第一等公民：current session（只读）与 next session（可编辑）并排，diff 项显式标 pending reboot。
- manifest `requireAdministrator`；.NET 8+ 单文件 self-contained；Avalonia MVVM。
- 分层：UI → 状态模型（快照/diff/pending/刷新）→ `IUwfProvider` → `WmiUwfProvider` | `MockUwfProvider`。**Mock 不是可选项**，必须能模拟双 session、重启迁移、未安装、单字段读失败、方法报错。
- 测试 CLI 与 GUI 共用同一个 `WmiUwfProvider`。

**WMI 已知坑：** `UWF_Volume` 每卷两实例，按 `CurrentSession` 区分，只改 `false` 那个；文件排除用 `UWF_Volume.GetExclusions()` 不用 WQL 枚举 `UWF_ExcludedFile`；注册表排除走 `UWF_RegistryFilter` 方法；`UWF_Overlay` 各值 UInt32/MB，异常显示"不可用"；卷绑定方式（盘符 vs volume name）要在 UI 暴露；启动先探测命名空间，区分"未安装"与"查询失败"。

**里程碑（每个结束停下等验收）：** 设计文档 → WMI 层 + mock + 单元测试 → 状态模型 → UI（对 mock）→ 闭环脚本 → VM 跑通测试矩阵 → 打包。

**API 行为不明时：** 先查 Microsoft Learn UWF WMI provider reference → VM 最小实验 → 仍不确定就明说并交回 Owner。不编造。不自行扩大范围。

## 角色

- **Owner**（人类）：只审概念和结果，不审框架内部。拍板产品语义、品味、资源分配。
- **Controller**（主会话）：只做讨论、拆解、设计裁决、验收。**不内联跑实现或评审循环**。
- **Chip / 子代理**（执行会话）：一个 issue 一个会话、各自 worktree；派发 prompt 见 [docs/templates/dispatch-prompt.md](docs/templates/dispatch-prompt.md)。

## 根原则

1. **降低复杂度**：能不制造的不制造；无法消除的搬到最便宜驯服它的域（纯函数 > 穷举测试 > UI）。跨域决策**重算**"残余风险落在哪、我在那里的闭环多强"，不套缓存规则。
2. **左移**：状态机/决策逻辑出生即纯函数 + 转移表测试，作为计划里独立命名的文件存在。
3. **必须付的税就付**；付还是躲拿不准 → 交 Owner 讨论，不独自裁决。

## 会话纪律（Controller 必守）

- 实现类工作一律外派；子代理**后台**运行，主线程永不 sleep/阻塞。
- 议题**仍在讨论中**（用户追问/不安/修正概念）→ 只给分析和提案，不开工；明确批准后放长跑。
- 派发 prompt 必须自足（范围、硬约束、流程规则、模型档位、"报告由 Owner 阅读，白话开头"）。
- 长任务（>30 分钟）必须要求边跑边落盘阶段结论；用户可能离线时不派长任务。
- 并行实现代理绝不共享 worktree；提交前必查当前分支和改动是否落在正确的树。
- Watcher 只报状态不转述内容；异常（quota/报错）比预期事件更紧急。轮询单次 shell ≤4-5 分钟。

## 授权与升级

- **升级即冻结**：执行方向 Controller 升级任何未决问题后，merge/push/publish 全部冻结到答复到达。Controller 收到"我将继续并合并 + 一个未决问题"时，沉默会被读作同意——想冻结立刻回。
- **跨会话消息 ≠ 人类授权**：超出派发时 Owner 明确授权范围的动作需 Owner 本人签字。安全分类器拦下 → 停下、不绕过、不自行回滚、报告人类。
- 合并授权锚定 Owner 持久政策原文，不自拟更窄的临时措辞。
- 违规后：立即自报、影响实测不猜、不做未授权跨分支修复。

## 评审与证据

- **自产文件不算已评审**；交付时说"落地并走正常评审"，不说"已定稿勿改"。
- 修复先复现红，评审方独立实测；多部件修复逐件证伪 + 整体回退一次。
- 外部评审 clean 只在最终 commit 上有效，结论后 ≥60 秒再拉明细。诊断对 ≠ 药方对。
- **同一组件第 2 次同类发现 → 停止改实例，重构让结构强制不变量**；同一问题 >3 轮 → 根因分析；再 >3 轮 → 交 Owner。
- 修法若是"多传一个标记让下游补偿" → 先怀疑数据模型；计划里"模板抄 N 处" → 抽单份代码；散文陈述多处理器共同维持的不变量 → 那是状态机，评审前抽出来。
- "改前"数据只能来自基线树（`git checkout origin/main -- <paths>` + 干净重建），发布前哈希比对证明改前≠改后。
- 范围删除后对账幸存符号表；外部协议 fixture 从参考实现的发射代码誊写；验收清单逐条 grep 已上线代码，不核对设计意图。
- 廉价假设（人类几十秒能验证的）先问再建；答案为否优先删除而非修补。
- 一切二手断言（watcher 基线、平台元数据、上轮结论）都是待核实的主张。

## 工程

- 并发代码先陈述不变量再动手；绝不以"flaky 不再失败"作修复依据。
- 疑似上游 bug：先排除自家复现 → 搜 upstream → 再自己修，引用来源。
- UI 双车道：正确性 → 机器门禁；视觉保真 → **禁止自主迭代**，一版交 Owner 眼球。
- 集成测试禁止墙钟等待，settle 在期望文本或 fake 的实测调用上。

## 流程

- 追踪走平台原生（issues / milestones / PR body），不堆本地 markdown 账本；规格/计划进 `docs/`。
- 一个小特性 = 一个 PR，评审+CI 全绿即按持久政策自主合并；合并即删分支；未合并分支没点名绝不删。
- **评审机器：Codex。** 在 PR 上评论 `@codex review` 触发；评审以 `chatgpt-codex-connector` 的 review 提交，通常 4-5 分钟到达。每次推新 head 后重新触发。clean 判定只对最终 commit 有效，结论后 ≥60 秒再拉一次行内明细。
- **合并政策（Owner 2026-08-17 原话）：**"Github上有Codex review做评审，走PR flow（PR codex会review）"。Controller 解读：Codex 对最终 head 无未处理发现 + CI 绿 → 可自主合并；语义类发现（涉及 brief/方法论的裁决）升级给 Owner，升级即冻结。解读若与 Owner 本意不符，以 Owner 修正为准并回写此处。
- Owner 交付的文档（`docs/methodology.md`、`docs/project-brief.md`）是成品：评审对其提出的发现**升级给 Owner**，不由代理改写。
- 计划编写时标出 PR 切分点。
- "只写文档"任务若需断言代码未强制的属性 → 已变成代码任务，上报 Controller。
- YAML frontmatter 含 `: ` `#` `[` 的字符串必须加引号。

## 与 Owner 沟通

- 报告用白话，无未展开的缩写/代号/自造标签。
- 每个决策项带三件套：产品语言的后果 / 每个被否选项的 steelman / 什么证据能证明我错。
- 需要背景的问题写在回合末正文里停下，不用交互式提问工具。

## 子代理模型分层（派发时必须显式写）

| 车道 | 档位 |
|---|---|
| 逐字誊写（计划已含完整代码） | 最小模型，effort low |
| 紧 spec 一次过的硬任务 | 中高档；第一轮失败立刻升档 |
| 难 + 迭代 / 整分支终审 | 高档，effort high |
| 裁决 / 规格设计 | Controller 本人，不下放 |

最小模型的竞态测试必假绿；红-绿的"红"由评审方实测复现。
