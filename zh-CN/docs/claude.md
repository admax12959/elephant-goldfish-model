<!--
Copyright 2026 Google LLC.
SPDX-License-Identifier: Apache-2.0
-->

> **原文路径**：`docs/claude.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# Claude Code 集成指南

请使用 Claude Code 2.1.271 或更高版本以获得 `omitClaudeMd` 支持。更早的版本不得被视为经过验证的金鱼隔离环境。

## 调用方式与资源解析

```sh
claude plugin marketplace add google/elephant-goldfish-model
claude plugin install egm@egm
```

使用 `claude plugin enable egm@egm` 启用已安装的插件。
以 `/egm:egm-brainstorm`、`/egm:egm-draft` 或 `/egm:egm-review` 作为消息开头来触发对应工作流。
所有技能均保留 `disable-model-invocation: true`：它们不会出现在自动技能发现列表中，这是符合预期的行为。`Skill` 工具会拒绝调用这些技能，因此如需运行其他 EGM 技能（例如在评审第 7 阶段和第 9 阶段运行 `egm-review-loop` 或 `egm-resolve`），请直接读取对应的 `SKILL.md` 并在同一对话线程中遵循其指令。切勿对 EGM 技能使用 `Skill` 工具。请将起草与编排工作保留在主上下文中。
Claude Code 没有产物评论（artifact comments）功能。如需提出草稿修改意见，请直接编辑 `design_draft.md`、在其中添加 `<!-- -->` 注释，或在聊天窗中给出说明；模型会在每次修订前重新读取草稿。
请将简报、草稿与报告写入用户的工作目录。单会话暂存区（per-session scratchpad）不是产物目录：它仅属于当前会话，而交付物必须持久留存于会话之外。
如果用户的提示词或智能体指令文件（`CLAUDE.md` 或 `AGENTS.md`）中包含 `EGM artifact directory: <path>` 这一行，则改为写入该文件夹。相对路径相对于工作目录解析。如果指令文件中的路径在跟随符号链接解析后落在了工作目录之外，须在创建文件夹或写入前请用户确认。如果用户拒绝或无人可确认，则回退使用工作目录并说明原因。若目标文件夹尚不存在，则创建之。
在 Claude Code 中，`egm:egm-synthesizer` 可以派生评审员（默认允许最多 3 层嵌套），主上下文也可以直接派发评审员。在 Claude Cowork 中，子智能体无法继续派生子智能体，因此应直接从主上下文派发评审员，而不要启动合成器子智能体。在这两种情况下，均由主上下文汇总发现并编写报告。
评审的第 5、6、7、8、9 阶段必须按顺序执行，仅对当前阶段内的评审员进行并行派发。等待该阶段所有评审员完成后，写出本阶段报告并完成必需的修复，再派发下一阶段。无中断模式（Uninterrupted mode）仅跳过人类检查点，不跳过这些阶段依赖关系。

Claude 会在插件技能和智能体正文中替换 `${CLAUDE_PLUGIN_ROOT}`。请使用该已解析的绝对根目录来定位 `agents/`、`resources/` 和 `docs/`；使用当前被调用技能的目录来定位其 `references/`。本独立指南是作为普通文件读取的，因此文中的占位符仅用于说明：请直接使用在被调用技能中已解析出的根目录，切勿猜测缓存版本号或搜索其他已安装副本。
评审目标与生成的产物属于用户工作区，而非插件缓存目录。请向每位评审员传递资源和目标文件的绝对路径。

专家角色与文档标准位于项目外部的插件缓存中。EGM 不会安装任何权限自动审批钩子（hook）。请通过 Claude Code 的常规权限提示批准资源读取；当前版本可以在父会话中弹出后台评审员的权限提示。对于无头运行（headless runs），用户必须通过 Claude Code 的权限设置，为已解析插件的 `agents/`、`docs/`、`resources/` 和 `skills/` 文件夹授予读取权限。请将写入权限严格限制在项目目录、预期输出目录以及用于临时文件的会话或私有暂存目录内。`mktemp -d` 生成的路径无法在规则中提前命名，因此无头运行在没有其父文件夹规则的情况下无法使用该路径。切勿关闭权限检查或为所有已安装插件授予访问权限。如果评审员无法读取其专家角色或文档标准，必须如实报告该限制并中止，不得将其视为已完成评审。

例如，在确定实际已安装的插件目录后，仅将其资源文件夹添加到 `/permissions` 中经用户批准的读取规则：

```text
Read(//absolute/plugin/path/agents/**)
Read(//absolute/plugin/path/docs/**)
Read(//absolute/plugin/path/resources/**)
Read(//absolute/plugin/path/skills/**)
```

在 Windows 上，绝对路径需写成以 `//` 开头且盘符小写、不带冒号的形式，或使用原生路径格式：

- `Read(//c/Users/<you>/.claude/plugins/cache/egm/egm/<version>/agents/**)`
- `Read(C:\Users\<you>\.claude\plugins\cache\egm\egm\<version>\agents\**)`

`//C:/Users/...` 这种格式无法匹配，读取仍会被拒绝。

通过插件市场安装的内容位于版本目录中：`.../plugins/cache/<marketplace>/egm/<version>/`。写死单一版本号的规则在插件更新后会失效。仅将版本号段落替换为 `*` 即可在跨版本更新时保持匹配，同时仍然只覆盖本插件的文件夹。例如在 Windows 上：

```text
Read(//c/Users/<you>/.claude/plugins/cache/egm/egm/*/agents/**)
Read(//c/Users/<you>/.claude/plugins/cache/egm/egm/*/docs/**)
Read(//c/Users/<you>/.claude/plugins/cache/egm/egm/*/resources/**)
Read(//c/Users/<you>/.claude/plugins/cache/egm/egm/*/skills/**)
```

请替换示例中的路径，切勿直接复制猜测的缓存位置。这些读取规则也会尽最大努力覆盖对这些文件的 Grep 和 Glob 操作。使用 `--add-dir` 添加目录会同时扩展编辑权限，因此不能作为只读访问的替代方案。

## Claude Code 警告：默认 agents/ 文件夹被忽略

在 Claude Code（`/plugin` > Errors）中，你可能会看到如下提示性警告：

```text
Default agents/ folder is ignored because the manifest sets "agents"
```

在 EGM 显式列出适配器的配置下，这条特定提示是预期内的，并不意味着已注册的适配器加载失败；如出现其他错误请另行排查。

EGM 维护两个智能体目录：
1. `agents/`：存放权威、与宿主环境无关的评审员专家角色 Markdown 提示词，供 Antigravity 和其他宿主加载。
2. `.claude-plugin/agents/`：存放 Claude Code 适配器。评审员适配器使用 `omitClaudeMd: true` 和 `tools: [Read, Grep, Glob]`；合成器与引导初始化编排器（bootstrap orchestrator）拥有各自独立的工具列表，详见下文。

由于 `.claude-plugin/plugin.json` 在 `"agents"` 字段中显式列出了适配器，Claude Code 会跳过自动加载根目录的 `agents/` 文件夹。共享专家角色并未声明 Claude 的只读工具限制；如果改为自动加载它们，将会丢失适配器特有的限制（可用工具将仅受限于宿主全局权限）。同时注册两份还会导致评审员重名。请保留显式适配器列表，切勿为了消除这条提示而将其删除。

## 评审员隔离

每次独立评审都必须派生一个对应插件注册评审员类型的新原生 Agent，例如 `egm:egm-zoom-out`。切勿分叉（fork）当前对话或恢复（resume）之前的评审员。仅传递任务指令、评审目标、专家角色/文档标准路径以及显式提供的金鱼上下文；切勿复制父会话的讨论过程。

`.claude-plugin/agents/` 中的 Claude 适配器声明了 `tools: [Read, Grep, Glob]`、`model: inherit` 和 `omitClaudeMd: true`。每个适配器通过 `${CLAUDE_PLUGIN_ROOT}` 读取 `agents/` 中的权威专家角色。请勿将 Claude 工具名写入共享专家角色的 frontmatter 中：其他宿主可能会按自己的规则解析 `tools` 字段。评审员不能编辑文件或运行 Shell 命令，且未配置任何持久化 `memory`。合成器适配器声明 `tools: [Agent, Read, Grep, Glob, Write]`，使其能够派生评审员并写出报告，但不能运行 Shell 命令或使用当前会话的 MCP 连接器。插件子智能体的 `tools` 字段无法将 Write 限制在特定路径上，且 Claude Code 会忽略插件子智能体中的 `hooks` 与 `permissionMode`。因此，适配器中“仅写入所请求的报告或简报”的规则属于提示词指令，而非宿主强制限制；写入操作仍遵循当前会话的权限规则。派生 `egm:egm-synthesizer` 时，请在其任务说明中明确指明所使用的产物目录。否则它无法看到用户提示词中的覆盖行，且会忽略仅由智能体指令文件指定、但落在工作目录之外的文件夹。由于它没有 Shell 工具，它按字面路径检查目录，不跟随符号链接。
引导初始化编排器适配器仅允许使用 `Agent`、`Bash`、`Read`、`Grep`、`Glob`、`Write` 和 `Edit` 进行本地发现与文档编写。它将工作子智能体路由至引导初始化的 `SKILL.md`，使其返回 JSON 格式的 `subtree_index` 而非散文，并由父级处理修正。Bash 仍受常规权限规则约束，并非文件系统或网络沙箱。
评审员无法打开 URL。切勿在 `GoldfishContext` 中向 Claude 评审员传递文档 URL：请先将原始页面保存为工作区中的文件，再传递该文件路径。

普通子智能体会排除父会话对话历史。`omitClaudeMd` 还会额外排除用户级/项目级/本地指令文件，但受管策略（managed policies）仍然生效。这属于上下文隔离与工具隔离，而非文件系统隔离：评审员仍可读取有权限访问的文件。请显式传递所有必需的评审约束，并将无关的历史记录、日志和记忆排除在评审任务之外。

进行本地测试时，请使用 `claude --plugin-dir <copy>` 加载一个具有唯一名称的副本，以免与已安装的 `egm` 插件混淆。使用 `claude plugin validate <copy>/.claude-plugin/plugin.json` 验证该副本。

## Claude Cowork

Claude Cowork 使用与 Claude Code 相同的 `.claude-plugin` 插件包。
可从 claude.ai/plugins 或 Cowork 插件面板通过插件市场 `google/elephant-goldfish-model` 安装插件。
对于候选包，Customize > Plugins > Add plugin > Upload plugin 支持上传包含 Claude 清单、适配器与共享运行时文件的 ZIP 包。安装后选择 **Try in chat**。请记录生成的任务是在本地还是云端运行；已测试的桌面端交互面使用的是云端任务。请在实际的 Cowork 会话中验证资源解析与评审员工具是否生效。仅完成安装并不能证明 Claude Code 的子智能体设置或 `${CLAUDE_PLUGIN_ROOT}` 替换在该交互面上有效。
使用 `claude --plugin-dir` 进行本地测试仅在 Claude Code 中受支持。

参考资料：[插件路径](https://code.claude.com/docs/en/plugins-reference)、[技能调用](https://code.claude.com/docs/en/skills) 以及 [子智能体上下文与工具](https://code.claude.com/docs/en/sub-agents)。
