<!--
Copyright 2026 Google LLC.
SPDX-License-Identifier: Apache-2.0
-->

> **原文路径**：`docs/codex.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# Codex 集成指南

## 安装并定位插件

桌面端应用请安装[已发布的 EGM 插件](https://chatgpt.com/plugins/plugins_6aeb90f47ec88191a3c107e01d3eb357)。该列表仅限桌面端。

Codex CLI 请完整安装整个插件（包含其专家角色与文档标准）：

```sh
codex plugin marketplace add google/elephant-goldfish-model
codex plugin add egm@egm
```

安装完成后启动一个新会话，从可用技能列表中选择已安装的 `egm` 技能。请使用其被发现的绝对 `SKILL.md` 路径，不要假定某个特定的缓存目录、版本号或检出位置。

从 `<plugin>/skills/<skill>/SKILL.md` 出发，向上三级父目录即为插件根目录。请基于该根目录解析随包附带的 `agents/`、`resources/` 和 `docs/` 路径；基于当前活动技能目录解析 `references/`；基于用户的工作目录解析用户文档与生成的产物。切勿用用户项目中同名的文件替代插件内置的专家角色或文档标准。

如果用户的提示词或智能体指令文件（`AGENTS.md`）中包含 `EGM artifact directory: <path>` 这一行，则将简报、草稿与报告写入该文件夹。相对路径相对于工作目录解析。如果 `AGENTS.md` 中的路径在跟随符号链接解析后落在了工作目录之外，须在创建文件夹或写入前请用户确认。如果用户拒绝或无人可确认，则回退使用工作目录并说明原因。若目标文件夹尚不存在，则创建之。

在 ChatGPT Work 中，技能可能通过云端资源标识符暴露，而配套支撑文件存放在已安装的本地包体中。请使用技能清单或安装元数据提供的已安装路径来访问这些文件，切勿为任意支撑路径自行拼接云端资源标识符。如果无法获得已安装路径，请报告该能力缺失，不要猜测缓存位置。请从包体中读取评审员文件名及其 frontmatter：例如 `egm-design` 角色对应的是 `agents/design_critic.md`，而不是 `agents/design.md`。派发前请确认每个已解析的路径。

## 新鲜上下文评审员

检查当前可用的原生子智能体工具 schema。工具名称可能带有命名空间，且随 Codex 版本不同而变化。当 `spawn_agent` 暴露 `fork_context` 参数时，必须为每个金鱼显式设置 `fork_context: false`。当 schema 改为暴露 `fork_turns` 参数时，请使用 `fork_turns: "none"`。该要求同样适用于由合成器启动的评审员。使用通用智能体，并向其传递专家角色、文档标准、目标文档以及显式选定阅读清单的绝对路径。切勿传递父会话记录、内部推理过程或目标文档本身应当阐明的答案。

要求评审员在返回评审意见前，必须读取其专家角色、目标文档以及所有提供的文档标准。遗漏读取或文件不可访问都意味着评审不完整，而绝非“无问题通过”。当宿主暴露原生工具调用结果时，请核验这些记录；父级自行撰写的完成摘要不能作为等效证据。

在汇总评审意见之前，必须等待评审员实际完成。请如实向上传播失败与超时；已启动的评审员并不等于已完成的评审。如果当前环境不支持嵌套派发，请从父级直接派发评审员，同时保持同等隔离要求。如果可用工具无法建立新鲜上下文，请中止金鱼评审并报告该限制。

新鲜对话上下文并不能隔离文件系统、已配置的记忆服务或项目指令。切勿检索历史对话或记忆来填补设计文档中缺失的上下文。隔离性测试应使用唯一的仅父级可见标记（parent-only marker），并检查子智能体的实际返回结果，而非轻信父级的声明。

`fork_context: false` 与 `fork_turns: "none"` 选项仅针对父会话对话轮次，无法证明 Codex 排除了宿主指令文件。Issue [#237](https://github.com/google/elephant-goldfish-model/issues/237) 报告称，即使设置了 `fork_turns: "none"`，Codex CLI 0.166.0 仍会注入项目级和用户级 `AGENTS.md` 文件。除非有独立的子上下文探针针对所测试的宿主版本予以确认，否则应将“指令文件排除”视为未验证状态。

## 提问与产物

共享技能中的提问载荷（question payloads）描述的是预期交互语义，而非字面意义上的 Codex API。仅当结构化提问工具可用且当前模式允许使用时才调用它，并映射到其实际 schema；否则请提出一条纯文本问题并等待回答。请严格保留技能的就绪检查与分节审批门禁；工具不可用绝不等于获得批准。

请使用绝对路径 Markdown 链接来链接生成的产物，例如 `[design draft](/absolute/project/design_draft.md)`。对于包含空格的路径，请使用 `[design draft](</absolute/my project/design_draft.md>)`。请勿在 Codex 聊天窗中使用 `file://` 链接。

## 验证

运行 `pytest tests/unit` 执行离线打包检查。可选的原生工作流测试记录在 [TESTING.md](https://github.com/google/elephant-goldfish-model/blob/main/TESTING.md) 中，它们需要已认证的 Codex CLI 并会消耗模型额度。仅仅安装或发现插件并不能证明端到端工作流已获支持。

## ChatGPT

请在 ChatGPT 桌面端应用中安装[已发布的 EGM 插件](https://chatgpt.com/plugins/plugins_6aeb90f47ec88191a3c107e01d3eb357)。该列表仅限桌面端，并不代表支持云端 Work 会话。

### 测试本地包

个人测试时，在 ChatGPT 中打开 Plugins > Add > Upload plugin archive。从仓库根目录打包运行时文件：

```sh
python3 scripts/pack_plugin.py --target openai --output /tmp/egm-chatgpt
```

请使用全新的输出目录。上传 `/tmp/egm-chatgpt/egm-openai.zip` 并在 ChatGPT 桌面端应用中安装插件。在一个本地运行的新对话线程中使用 **Try in chat**。`TESTING.md` 中早期的云端 Work 测试早于本包体声明仅限桌面端支持。请勿将仓库插件市场目录和开发文件打入该归档包。OpenAI 应用目录包声明了本地执行模式，因为引导初始化（bootstrap）需要使用 Shell 命令。其列表信息使用可移植清单中的 OpenAI 扩展字段；Codex 兼容性清单则保留匹配的元数据以供原生安装使用。请将 `.claude-plugin/` 排除在 OpenAI 归档包之外，因为其智能体适配器专属于 Claude Code 集成。公共应用目录插件不能包含 hooks。

Enterprise、Business 与 Edu 工作区可通过 Admin > Plugins > Add > Import marketplace 导入本插件：将仓库 URL 设为 `https://github.com/google/elephant-goldfish-model`，将插件市场路径设为 `.agents/plugins`。随后可通过 `@egm` 直接调用技能。在启动金鱼评审前，请检查当前对话中的原生工具。部分 ChatGPT Work 会话会暴露带有 `fork_turns: "none"` 的原生 `spawn_agent`；当这些工具可用时，请遵循上文的新鲜上下文指令。若这些工具不可用，请立即中止并报告无法提供隔离环境；切勿在父上下文中模拟评审员。单个 Work 会话中工具可用，并不代表所有 ChatGPT 模式或账号均受支持。
