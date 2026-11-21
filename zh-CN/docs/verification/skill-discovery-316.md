> **原文路径**：`docs/verification/skill-discovery-316.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# #316 技能发现对比验证

于 2026 年 11 月 6 日针对每个用例与修订版本分别运行一次全新会话。基线版本 `6a8001e` 与候选版本 `ce3e309` 使用不含 EGM 技能名称的相同提示词。随后从 `main` 分支的合并未修改两者的技能文件。下表为每个用例单次运行所观测到的技能选择结果。

| 任务 | Codex 基线版本 | Codex 候选版本 |
| --- | --- | --- |
| 搭建设计与评审工作区（Set up a design and review workspace） | Bootstrap | Bootstrap, Draft, Review, Review Loop |
| 探索离线同步权衡（Explore offline sync tradeoffs） | 无（None） | Brainstorm |
| 基于已达成一致的简报起草设计（Draft a design from an agreed brief） | 无（None） | Draft |
| 对比架构与实现（Compare architecture and implementation） | Drift Report | Drift Report |
| 分诊相互冲突的评审意见（Triage conflicting review findings） | 无（None） | Resolve |
| 组建自定义结构与风格评审组（Set up a custom structural and style panel） | Review Loop | 无（None） |
| 运行标准分阶段设计评审（Run a standard staged design review） | Review | Review |
| 解答简单算术题（Answer simple arithmetic） | 无（None） | 无（None） |
| 仅校对拼写（Proofread for spelling only） | 无（None） | 无（None） |

Codex CLI 0.166.0 每次仅加载一个临时 EGM 插件市场。只有成功读取已安装的技能文件才计为被选中。全局技能仍保持可用。每个提示词均要求只读操作且禁止启动评审员。候选版本改进了三个用例，但也在搭建工作区用例中加载了多余技能，并遗漏了自定义评审组用例。在决定是否保留更短的技能描述前，应重复验证这两个用例。

搭配 Sonnet 5.5 的 Claude Code 2.1.319 在每次会话中加载一个生成的 EGM 插件包。两个包体均列出了全部七个 EGM 技能，且在全部九个提示词中均未触发任何 EGM `Skill` 工具调用。这符合两个版本中的显式调用策略。该探针未测试显式调用或完整工作流。

仅修改列表展示的修订版（listing-only revision）逐字节保留了所有技能文件。上述结果适用于已撤回的描述文本变更。
