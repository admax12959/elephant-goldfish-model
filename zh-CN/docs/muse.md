<!--
Copyright 2026 Google LLC.
SPDX-License-Identifier: Apache-2.0
-->

> **原文路径**：`docs/muse.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# Muse 集成指南

Muse 通过其 Claude 兼容插件路径加载 EGM，无需单独的 Muse 原生清单。该插件通过验证，支持全部七个技能和全部十个智能体适配器：八个评审员、合成器以及引导初始化编排器。

共享技能会自动将 Codex 和 Claude 会话路由至各自的集成指南，但不会自动路由 Muse 会话。除非你主动指引，否则 Muse 会话不会读取本文件。

## 安装并定位插件

请从本地路径完整安装整个插件（包含其专家角色与文档标准）：

```sh
git clone https://github.com/google/elephant-goldfish-model.git
muse plugins install ./elephant-goldfish-model --scope user
muse plugins approve egm
```

在本仓库上运行 `marketplace add` 会发现零个插件：Muse 会优先读取 Codex 目录（`.agents/plugins/marketplace.json`）并跳过其中的条目，因此不会继续回退读取 Claude 目录。请改为从本地路径安装，并确保源码目录不包含符号链接（先删除 `.venv/` 等临时目录，安装器会拒绝符号链接条目）。只需批准一次：插件启用期间技能无需评审即可保持活跃，但十个智能体定义必须在执行 `approve` 后才会激活。

安装完成后启动一个新会话。技能将以 `plugin:egm:egm-*` 的形式呈现，且仅支持用户显式调用激活；可通过 `muse skills list --source plugin` 确认。该命令输出的是 `plugin://` URI 而非文件系统路径；请从 `muse plugins inspect egm` 的 `cache=` 字段获取已安装路径。切勿假定某个特定的缓存目录、版本号或检出位置。

从 `<plugin>/skills/<skill>/SKILL.md` 出发，向上三级父目录即为插件根目录。基于该根目录解析随包附带的 `agents/`、`resources/` 和 `docs/` 路径；基于当前活动技能目录解析 `references/`；基于用户工作目录解析用户文档与生成的产物。切勿用用户项目中同名的文件替代插件内置的专家角色或文档标准。

无需安装即可进行离线预检：`muse plugins validate <path>` 会报告该插件有效，包含 7 个受支持技能与 10 个受支持智能体。出现警告属于预期现象（`multiple-manifests` 会优先选用 Claude 清单而非 Codex 清单；Claude 专属字段如展示元数据、`color` 以及未知的智能体 frontmatter 键会被忽略）。仅仅安装或发现插件并不能证明端到端工作流已获支持。

## 新鲜上下文评审员

Muse 将十个 Claude 适配器导入为受支持的智能体定义（`egm/egm-design`、`egm/egm-comprehension` 等），但所有智能体的工具投影均不受支持：Claude 工具白名单不会限制 Muse 评审员，因此切勿假定评审员处于只读状态。派发前请检查可用的原生子智能体工具 schema；工具名称可能带有命名空间，且随 Muse 版本不同而变化。

当会话暴露具名评审员智能体时优先使用它，并传递专家角色、文档标准、目标文档及显式选定阅读清单的绝对路径。切勿传递父会话记录、内部推理过程或目标文档本身应当阐明的答案。当具名智能体不可用时，回退为先读取专家角色文件的通用子智能体。请按 `name` frontmatter 而非文件名进行匹配：`egm-design` 对应 `agents/design_critic.md`。在 Muse 1.4.14 中，具名 `egm/egm-*` 定义在插件清单中虽显示为 `trusted_enabled`，但派生时会被拒绝（`agent_definition_unavailable`），因此实际可用路径为通用子智能体回退方案。

要求评审员在返回评审意见前，必须读取其专家角色、目标文档以及所有提供的文档标准。遗漏读取或文件不可访问都意味着评审不完整，而绝非“无问题通过”。当宿主暴露原生工具调用结果时，请核验这些记录；父级自行撰写的完成摘要不能作为等效证据。

在汇总评审意见之前，必须等待评审员实际完成。请如实向上传播失败与超时；已启动的评审员并不等于已完成的评审。如果当前环境不支持嵌套派发，请从父级直接派发评审员，同时保持同等隔离要求。如果可用工具无法建立新鲜上下文，请中止金鱼评审并报告该限制。

新鲜对话上下文并不能隔离文件系统、已配置的记忆服务或项目指令。切勿检索历史对话或记忆来填补设计文档中缺失的上下文。隔离性测试应使用唯一的仅父级可见标记，并检查子智能体的实际返回结果，而非轻信父级的声明。

## 提问与产物

共享技能中的提问载荷描述的是预期交互语义，而非字面意义上的 Muse API。仅当结构化提问工具可用且当前模式允许使用时才调用它，并映射到其实际 schema；否则请提出一条纯文本问题并等待回答。请严格保留技能的就绪检查与分节审批门禁；工具不可用绝不等于获得批准。

请使用绝对路径 Markdown 链接来链接生成的产物，例如 `[design draft](/absolute/project/design_draft.md)`。对于包含空格的路径，请使用 `[design draft](</absolute/my project/design_draft.md>)`。请勿在 Muse 聊天窗中使用 `file://` 链接。

## 验证

运行 `pytest tests/unit` 执行离线打包检查。端到端工作流测试固件（通过已安装插件运行 brainstorm、draft 以及无中断 review）已于 2026-10-30 在 Muse 1.4.14 上通过上述通用评审员回退路径验证成功。该次测试的提示词显式指定了本指南，且以无头模式预先模拟审批运行，因此技能自动路由、交互式审批门禁与聊天链接渲染尚未经过测试；详情请参阅[宿主测试记录](https://github.com/google/elephant-goldfish-model/blob/main/TESTING.md#harness-test-record)。仅仅安装或发现插件并不能证明端到端工作流已获支持。
