> **原文路径**：`docs/harnesses.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 宿主环境（Harnesses）

EGM 只有一套统一的技能与专家角色。本仓库作为所有受支持宿主环境的单一插件源，任何宿主环境都不会分叉出一份独立的技能文本。

## 安装方式

| 宿主环境 | 安装方法 |
| --- | --- |
| ChatGPT 桌面端 | 安装[已发布的 EGM 插件](https://chatgpt.com/plugins/plugins_6aeb90f47ec88191a3c107e01d3eb357)（仅限桌面端列表）；个人测试请参阅[本地打包测试](codex.md#测试本地包) |
| Codex 桌面端 | 安装[已发布的 EGM 插件](https://chatgpt.com/plugins/plugins_6aeb90f47ec88191a3c107e01d3eb357)（仅限桌面端列表） |
| Antigravity CLI | `agy plugin install https://github.com/google/elephant-goldfish-model` |
| Antigravity App | 运行 `/plugin` 或参阅[安装说明](#antigravity-app--antigravity-for-ides) |
| Antigravity for IDEs | 运行 `/plugin` 或参阅[安装说明](#antigravity-app--antigravity-for-ides) |
| Antigravity IDE（独立版） | 暂不支持；参阅[安装说明](#antigravity-app--antigravity-for-ides) |
| Claude Code | `claude plugin marketplace add google/elephant-goldfish-model && claude plugin install egm@egm` |
| Claude Cowork | 从 claude.ai/plugins 或 Cowork 插件面板通过插件市场 `google/elephant-goldfish-model` 安装 |
| Codex CLI | `codex plugin marketplace add google/elephant-goldfish-model && codex plugin add egm@egm` |
| ChatGPT 工作区导入 | Admin > Plugins > Add > Import marketplace，仓库 URL 填 `https://github.com/google/elephant-goldfish-model`，路径填 `.agents/plugins`；云端评审限制仍然适用 |
| omp | `omp plugin marketplace add google/elephant-goldfish-model && omp plugin install egm@egm` |
| Muse | `git clone https://github.com/google/elephant-goldfish-model.git && muse plugins install ./elephant-goldfish-model --scope user && muse plugins approve egm` |

请完整安装整个插件。仅复制 `skills/` 目录的纯技能安装器会破坏 EGM，因为技能在运行时需要读取 `agents/` 与 `resources/document_standards/`。

<a id="antigravity-app--antigravity-for-ides"></a>

### Antigravity App 与 Antigravity for IDEs

- **Antigravity App**：运行 `/plugin`，让智能体执行 `Install https://github.com/google/elephant-goldfish-model`，或使用下方命令手动安装到 `~/.gemini/config/plugins/egm/`。
- **Antigravity for IDEs（Visual Studio Code 扩展）**：运行 `/plugin`，让智能体执行 `Install https://github.com/google/elephant-goldfish-model`，或使用下方命令手动安装到 `~/.gemini/config/plugins/egm/`。
- **Antigravity IDE（独立版）**：目前暂不支持。尽管插件与技能可以从 `~/.gemini/config/plugins/egm/` 加载，但独立版 Antigravity IDE（已在 macOS 与 Linux 上测试）未暴露金鱼评审（`/egm-brainstorm` 与 `/egm-review`）所需的新鲜上下文子智能体工具（`invoke_subagent`）。

Antigravity App 与 Antigravity for IDEs 共享全局插件目录 `~/.gemini/config/plugins/<name>/`。若要在没有 `agy` CLI 的情况下进行全新手动安装，请将完整运行时包复制到 `~/.gemini/config/plugins/egm/`（直接复制到已有插件目录可能会遗留过时文件，因此在更新时请先删除 `~/.gemini/config/plugins/egm`）：

```sh
git clone https://github.com/google/elephant-goldfish-model.git
cd elephant-goldfish-model
mkdir -p ~/.gemini/config/plugins/egm
cp -R plugin.json README.md LICENSE skills agents resources docs assets ~/.gemini/config/plugins/egm/
```

重新加载窗口（或重启 Antigravity App），然后确认 `egm` 出现在插件列表中，且其技能（`egm-brainstorm`、`egm-draft`、`egm-review`）可用。这些技能在 Antigravity App 与 Antigravity for IDEs 中的具体命令形式尚未经过验证。

## 清单文件（Manifests）

根目录的 `plugin.json` 是插件标识、版本、描述与发布者元数据的权威来源。它遵循可移植的 Agent Plugins 1.0.0 格式；之所以省略 `$schema` 键，是因为 omp 会将声明了 `$schema` 的清单路由至一个严格校验器，而该校验器会拒绝 `disable-model-invocation`。更新共享字段时请先修改根 `plugin.json`，再同步更新兼容性清单；单元测试会拦截元数据漂移。在安装测试证明可以移除之前，请保留各现有清单中的平台专属字段。

插件市场文件（Marketplace files）是指向本包的发现目录，而非其技能的副本。具体而言，`.agents/plugins/` 存放 Codex 目录，而 `agents/` 存放评审员专家角色。所有宿主共享同一份 `skills/`、`agents/` 与 `resources/` 内容。

| 宿主环境 | 清单文件 | 技能路径 | 子智能体 |
| --- | --- | --- | --- |
| Antigravity | `plugin.json` | `skills/`（默认） | `agents/`（默认） |
| Claude Code | `.claude-plugin/plugin.json`、`.claude-plugin/marketplace.json` | `skills/`（默认） | `plugin.json` 中 `agents` 列出的 Claude 适配器与编排器 |
| Claude Cowork | `.claude-plugin/plugin.json`、`.claude-plugin/marketplace.json` | `skills/`（默认） | 子智能体在 Cowork 内部运行；不适用 hooks |
| Codex | `.codex-plugin/plugin.json`、`.agents/plugins/marketplace.json` | `skills` 路径 | 不单独随包附带；通用子智能体直接读取专家角色文件 |
| ChatGPT | `plugin.json`、`.codex-plugin/plugin.json`；工作区导入使用 `.agents/plugins/marketplace.json` | `skills` 路径 | 检测当前会话中的原生工具；通用智能体直接读取专家角色 |
| omp | `.claude-plugin/plugin.json`、`.claude-plugin/marketplace.json`（Claude 兼容回退） | `skills/`（默认） | 来自 `agents/` / `.claude-plugin/agents/` 的专家角色注册为原生 `task` 智能体 |
| Muse | `.claude-plugin/plugin.json`（Claude 兼容回退） | `skills/`（默认） | 10 个适配器导入为智能体；不支持工具投影限制 |

专家角色提示词存放在 `agents/` 中，这也是 Antigravity 扫描子智能体的目录。每个专家角色都带有 `name` 与 `description` frontmatter，因此加载 `agents/*.md` 的宿主环境会将它们注册为具名子智能体（`egm-design`、`egm-zoom-out` 等）。

新增评审员专家角色时，请在 `.claude-plugin/agents/` 下添加对应的 Claude 适配器，并在 `.claude-plugin/plugin.json` 中列出。适配器负责读取共享专家角色，并配置 Claude 的只读工具与上下文隔离。这样可以避免将 Claude 的工具名称强加给其他宿主。在 Claude Code 中，`/plugin` > Errors 会提示默认 `agents/` 文件夹因清单设置了 `"agents"` 而被忽略；该警告符合预期且无害（请参阅 [Claude 集成](claude.md#claude-code-警告默认-agents-文件夹被忽略)）。

## 委派机制（Delegation）

技能文档中使用“派生一个子智能体（spawn a subagent）”并指定某个专家角色。每个宿主环境都会将该表述映射到自身对应的工具：

| 宿主环境 | 子智能体工具 | 具名专家角色 | 嵌套能力 |
| --- | --- | --- | --- |
| Antigravity | `invoke_subagent` | 支持 | 支持 |
| Claude Code | Agent (Task) 工具 | 支持 | 默认最多支持 3 层 |
| Claude Cowork | Cowork 子智能体 | 支持 | 子智能体在 Cowork 内部运行 |
| Codex | 原生 spawn/wait 工具；需检查可用 schema | 不支持 | 运行时检测；不支持时回退为直接派发 |
| ChatGPT | 当前 Work 会话暴露时的原生 spawn/wait 工具 | 不支持 | 运行时检测；不支持时回退为直接派发 |
| omp | `task` 工具 | 支持 | 支持 |
| Muse | 原生子智能体；需检查可用 schema | 1.4.14 中拒绝具名派生；已验证通用回退方案 | 运行时检测；不支持时回退为直接派发 |

若某个宿主环境不支持具名专家角色，技能会派生一个通用子智能体，并指示其首先读取 `agents/<file>.md`。如果宿主环境无法嵌套派生子智能体，则采用直接派发代替合成器（synthesizer）。

金鱼评审必须依赖新鲜上下文。如果宿主环境没有可用的子智能体工具，评审技能会立即中止，而不会在同一上下文中凑合执行。请检查实际的工具可用性，不要仅凭产品名称臆断。这一原则同样适用于独立版 Antigravity IDE 以及未暴露原生新鲜上下文委派能力的 ChatGPT 模式。

新鲜对话上下文并不意味着宿主会忽略其指令文件。Codex 的 `fork_context: false` 和 `fork_turns: "none"` 仅隔离父会话轮次，并不能证明项目级或用户级指令已被排除。请将稳定的工作区规范放在指令文件中，并将需要金鱼质疑的评审假设明确写为显式的评审准则。

## 产物目录（Artifact Directory）

技能默认将简报、草稿和报告写入工作目录，或在宿主拥有专属产物目录时写入该目录。在 Claude Code、Claude Cowork、Codex 和 ChatGPT 中，你可以指定另一个文件夹。只需在提示词中，或在宿主会加载的智能体指令文件（项目级或用户级的 `CLAUDE.md` 或 `AGENTS.md`）中加入这一行：

```text
EGM artifact directory: .dev/docs
```

相对路径将相对于工作目录解析。如果指令文件中设置的路径在跟随符号链接解析后落在了工作目录之外，EGM 会在创建文件夹或向其中写入之前请求你确认。如果你拒绝确认或无人可确认（例如无头运行模式），文件将写入工作目录。该确认结果不会被保存，因此新会话会再次询问。直接在提示词中给出的路径则无需额外确认。设计文档及其他项目文件的位置不会因此移动。Claude Code 仅在项目没有 `CLAUDE.md` 时才会读取项目级的 `AGENTS.md`。

请将已批准的规格文档提交到 `docs/designs/`，并将项目规则提交到 `AGENTS.md`，以确保团队成员与智能体使用相同的已提交文件。仅在希望于批准前避免将临时草稿提交到 Git 时，才将 `.dev/docs` 加入 `.gitignore`。

## 支持状态与贡献其他宿主环境

**Antigravity CLI**、**Claude Code** 和 **Codex** 均已通过自动化安装测试与端到端工作流测试固件（fixtures）。但评审员隔离性的验证证据范围更窄：父轮次隔离与宿主指令排除是两项独立的验证结果。

Claude 评审员适配器设置了 `omitClaudeMd: true`，详情见 [Claude 集成](claude.md#评审员隔离)。对于 Codex，Issue [#237](https://github.com/google/elephant-goldfish-model/issues/237) 报告称即使设置了 `fork_turns: "none"`，Codex CLI 0.166.0 仍会注入项目级与用户级 `AGENTS.md` 文件。对于 Antigravity CLI，同一 Issue 报告评审员上下文中存在项目级 `AGENTS.md` 与 `GEMINI.md`。这些报告反映的是宿主指令泄漏，而非父对话轮次泄漏的证据。Codex 测试与已记录的 Antigravity 运行仅检验了父轮次隔离，两者均未验证指令文件排除。此处尚未针对 Codex 或 Antigravity 验证任何宿主专属的指令排除控制项。

**Claude Cowork**（[#105](https://github.com/google/elephant-goldfish-model/issues/105)）和 **ChatGPT Work**（[#106](https://github.com/google/elephant-goldfish-model/issues/106)）具备可用的安装路径，但仍存在尚未补齐的云端追踪（cloud trace）验证缺口。README 中的前缀指南仅覆盖 Antigravity（agy CLI）、Claude Code 和 Codex。对于 ChatGPT 工作区导入，可通过 `@egm` 访问技能（参见 [ChatGPT](codex.md#chatgpt)）；Claude Cowork 中的命令形式尚未记录。

**omp**（[#186](https://github.com/google/elephant-goldfish-model/issues/186)）具备可用的安装路径。技能以 `/skill:egm-<command>` 形式调用（omp 为每个技能注册一条命令，没有 `/egm-*` 命名空间）。社区已在 omp v18.12.6 上验证：全部七个技能均可被发现和解析，全部九个评审员专家角色均成功注册为子智能体。omp 的端到端工作流测试固件尚未实现自动化。

**Muse**（1.4.14）具备可用的安装路径。`muse plugins validate` 报告该插件有效，支持全部七个技能和全部十个智能体适配器；但所有智能体的工具投影均不受支持，因此 Claude 的工具白名单不会限制 Muse 评审员。请从本地路径安装；在该仓库上运行 `marketplace add` 会发现零个插件，因为 Muse 会优先读取 Codex 目录。端到端工作流测试固件已于 2026-10-30 通过（包含 brainstorm、draft 以及 15 名新鲜上下文评审员参与的无中断 review）；由于该运行时中无法派生具名 `egm/egm-*` 智能体，评审员采用了先读取已安装专家角色的通用回退路径。请参阅 [Muse 指南](muse.md)。目前尚无技能自动将 Muse 会话路由至该指南（[#252](https://github.com/google/elephant-goldfish-model/issues/252)），因此请手动指示会话阅读该指南。

**Windsurf**（已于 2026-06-02 更名为 Devin Desktop；[#376](https://github.com/google/elephant-goldfish-model/issues/376)）具备实验性工作区路由支持。规则与工作流文件直接解析共享的 `skills/*/SKILL.md`、`agents/*.md` 和 `resources/document_standards/*.md` 文件，不复制技能文本。若无法提供新鲜上下文的只读子智能体委派，金鱼评审会立即停止，且评审员在宿主的只读子智能体配置下启动。2026-11-10 在 Devin CLI 3000.12.43 上的运行通过了路由、人类暂停、只读评审员与父标记隔离检查，但所有文件写入均被拒绝，因此没有任何工作流运行至最终产物阶段。Devin Desktop 与 Devin CLI 运行的是同一个智能体 Devin Local（取代了 Cascade）。Devin Desktop IDE 仍未经测试。CLI 不会将 `.windsurf/workflows/` 列为斜杠命令，`/egm-*` 请求通过规则进行路由。拒绝权限提示会导致宿主在无消息的情况下结束当前轮次，需发送后续消息以继续。相关证据请参阅[设计文档](https://github.com/google/elephant-goldfish-model/blob/main/docs/designs/windsurf_support.md#recorded-cli-evidence-2026-11-10)。

关于已测试的交互界面与限制，请参阅[宿主测试记录](https://github.com/google/elephant-goldfish-model/blob/main/TESTING.md#harness-test-record)。

我们欢迎希望协助验证并扩展宿主环境支持的贡献者。

### 贡献对新宿主环境的支持

只有当 `/egm-brainstorm`、`/egm-draft` 和 `/egm-review` 通过插件安装完成端到端运行后，该宿主环境才算正式受支持。贡献对新宿主环境的支持时，请按以下步骤进行：

1. 在仓库根目录添加相应的清单或配置文件（例如符合该宿主插件/扩展标准的文件）。
2. 在本指南（`docs/harnesses.md`）中记录安装步骤、清单映射与委派机制。
3. 验证 `/egm-brainstorm`、`/egm-draft` 和 `/egm-review` 能够端到端运行，并在 Pull Request 描述中附上验证记录或执行轨迹（如 [Pull Request 模板](https://github.com/google/elephant-goldfish-model/blob/main/.github/PULL_REQUEST_TEMPLATE.md) 中所述）。
