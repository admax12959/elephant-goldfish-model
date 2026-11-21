> **原文路径**：`docs/designs/README.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# EGM 设计规格文档

本目录包含定义大象-金鱼模型 (EGM) 框架、技能、评审员智能体人格及运行护栏的权威技术设计规格文档。

| 文档 | 作者 | 主要组件 |
| :------- | :----- | :---------------- |
| [site_documentation.md](site_documentation.md)<br>提议的多页面指南、技能参考手册与共享站点界面。 | [@n3wth](https://github.com/n3wth) | 网站与文档 |
| [lite.md](lite.md)<br>快速临时工程协议，采用解耦的 5 阶段状态机并结合大象与金鱼智能体人格切换，适用于小型快速迭代、规格修订、代码重构及缺陷修复。 | [@njeunje](https://github.com/njeunje) | [`egm-lite`](../../skills/egm-lite/SKILL.md) |
| [triz_integration.md](triz_integration.md)<br>苏联 TRIZ（发明问题解决理论）方法集成草案，引入发明催化剂（Inventive Catalyst）智能体人格，以消除 `/egm-brainstorm` 和 `/egm-review` 步骤 5 中的架构伪妥协。 | [@briansuchy](https://github.com/briansuchy) | [`egm-brainstorm`](../../skills/egm-brainstorm/SKILL.md)、[`egm-review`](../../skills/egm-review/SKILL.md) (#37) |
| [ask.md](ask.md)<br>只读命令草案，用于回答关于 EGM 的问题并根据工作区产物推荐下一个技能。 | [@smituparmar01](https://github.com/smituparmar01) | `egm-ask` (#175) |
| [skill_icons.md](skill_icons.md)<br>供插件、README 和网站共享的权威技能图标资产。 | [@n3wth](https://github.com/n3wth) | 技能展示 |
| [runtime_packaging.md](runtime_packaging.md)<br>已实现的运行时打包边界、权限与校验机制。 | [@n3wth](https://github.com/n3wth) | 运行时打包实现 (#136) |
| [releases.md](releases.md)<br>已实现的版本化归档、校验和及手动发布流程。 | [@n3wth](https://github.com/n3wth) | 发布实现 (#137) |
| [claude_release_branch_distribution.md](claude_release_branch_distribution.md)<br>将经过认证的 Claude 运行时发布到生成分支以供目录提交的草案。 | [@n3wth](https://github.com/n3wth) | Claude 目录发布 (#121) |
| [architecture.md](architecture.md)<br>EGM 框架的系统架构，涵盖组件模型、端到端数据流及智能体生命周期。 | [@quinnjim](https://github.com/quinnjim)、[@eewing314](https://github.com/eewing314) | [EGM 框架架构](../../README.md) |
| [brainstorm.md](brainstorm.md)<br>快速零文档构思工具，使用 7 视角金鱼蜂群在起草前审计并压力测试早期概念。 | [@briansuchy](https://github.com/briansuchy) | [`egm-brainstorm`](../../skills/egm-brainstorm/SKILL.md) |
| [draft.md](draft.md)<br>逐节迭代的规格起草协议与语义边界定义。 | [@quinnjim](https://github.com/quinnjim)、[@eewing314](https://github.com/eewing314) | [`egm-draft`](../../skills/egm-draft/SKILL.md) |
| [drift_report.md](drift_report.md)<br>自动化扫描器，用于检测并报告已签入的设计规格与现行代码库实现之间的偏离（漂移）。 | [@quinnjim](https://github.com/quinnjim) | [`egm-drift-report`](../../skills/egm-drift-report/SKILL.md) |
| [bootstrap.md](bootstrap.md)<br>跨代码包与仓库树的自底向上递归引导及哨兵块更新插入（Upsert）文档机制。 | [@briansuchy](https://github.com/briansuchy) | [`egm-bootstrap`](../../skills/egm-bootstrap/SKILL.md) |
| [review.md](review.md)<br>使用隔离金鱼评审智能体人格的多阶段验证协议与自动化裁决引擎。 | [@tearseb](https://github.com/tearseb)、[@briansuchy](https://github.com/briansuchy) | [`egm-review`](../../skills/egm-review/SKILL.md) |
| [synthesizer.md](synthesizer.md)<br>中间编排子智能体，用于协调蜂群扇出、去重评审意见并规范化链接引用。 | [@briansuchy](https://github.com/briansuchy) | [`synthesizer`](../../agents/synthesizer.md) |
| [additional-harnesses.md](additional-harnesses.md)<br>提议对 Cursor、OpenCode 和 Copilot CLI 提供支持。维护者范围审批待定。 | [@n3wth](https://github.com/n3wth) | [宿主环境集成](../harnesses.md) |
| [windsurf_support.md](windsurf_support.md)<br>提议的实验性 Windsurf (Devin Desktop) 工作区路由与金鱼隔离边界。 | [@n3wth](https://github.com/n3wth) | [宿主环境集成](../harnesses.md) (#376) |
| [multi_harness_support.md](multi_harness_support.md)<br>提议的 Hermes Agent、OpenClaw 和 Delta 集成需求与验证门禁；原生支持尚未验证。 | [@Seburan](https://github.com/Seburan)、[@yathamsriram243-blip](https://github.com/yathamsriram243-blip) | [宿主环境架构](../harnesses.md) |
| [plugin_listing_presentation.md](plugin_listing_presentation.md)<br>提议的商店列表文案、起始提示词与打包技能资产（不修改工作流正文）。 | [@n3wth](https://github.com/n3wth) | 插件展示 (#122) |
