# Groundhog

Windows **Unified Write Filter (UWF)** 的单机图形管理工具，目标是替代 `uwfmgr.exe` 的日常操作。名字取自《土拨鼠之日》：UWF 保护下的机器每次重启都醒在同一个早晨，只有例外列表里的东西带着记忆穿越循环。

- 目标平台：Windows 10 IoT Enterprise LTSC 2021（含中文版）
- 技术栈：.NET 8+ / Avalonia / WMI（`root\standardcimv2\embedded`）
- 需求全文：[docs/project-brief.md](docs/project-brief.md)

## 协作方式

本仓库按 **人类 Owner + AI Controller + AI 执行代理** 的多会话协作模式开发：

- 会话规则：[CLAUDE.md](CLAUDE.md)（每会话必加载）
- 完整方法论：[docs/methodology.md](docs/methodology.md)
- 派发 prompt 模板：[docs/templates/dispatch-prompt.md](docs/templates/dispatch-prompt.md)
- 追踪走 GitHub issues / milestones / PR body；规格与计划作为版本化工件放在 `docs/`。

## 许可证

MIT，见 [LICENSE](LICENSE)。
