<!-- Copyright 2026 Google LLC. SPDX-License-Identifier: Apache-2.0 -->

> **原文路径**：`docs/copilot.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# GitHub Copilot CLI 集成指南

面向 [Issue #147](https://github.com/google/elephant-goldfish-model/issues/147) 的实验性适配器。工作流执行与评审员隔离尚未经过验证。本指南不代表支持 VS Code 中的 Copilot 或云端智能体。

## 接口与安装

离线参考版本为 GitHub Copilot CLI **1.0.82**。
采用现有的 Agent Plugins 1.0 根目录 `plugin.json`。Copilot 专属配置文件位于 `com.github.copilot/agents/`。Issue [#375](https://github.com/google/elephant-goldfish-model/issues/375) 正在跟踪 VS Code 中 GitHub Copilot 的实验性工作区发现机制。`.vscode/settings.json` 使用了两项已弃用的兼容配置：`chat.agentSkillsLocations` 映射至 `skills`，`chat.agentFilesLocations` 映射至 `com.github.copilot/agents`。共享的技能、专家角色和文档标准仍保留在 `skills/`、`agents/` 和 `resources/` 中。

请使用固定候选提交上的完整仓库检出，并在整个会话期间保持该检出完整。切勿仅复制 `skills/` 或单独安装配置目录。根目录清单优先于 Claude 兼容性清单。无需配置 Copilot hooks、MCP 服务器或替代清单。

将 `EGM_ROOT` 设为候选检出的绝对路径：

```sh
EGM_ROOT=/absolute/path/to/elephant-goldfish-model
git -C "$EGM_ROOT" rev-parse HEAD
copilot --version
COPILOT_OFFLINE=true copilot --plugin-dir "$EGM_ROOT" plugin list
```

最后一条命令必须在外部插件列表下显示 `egm`。该命令仅检查注册情况，不验证评审员执行。如需使用独立配置档案，请将 `COPILOT_HOME` 设为新目录；注意这并不会禁用所有用户级技能或项目指令。在进行任何实时测试前，请先检查宿主清单。

在获得明确的模型调用授权后，于目标项目中启动 Copilot：

```sh
copilot --plugin-dir "$EGM_ROOT" --add-dir "$EGM_ROOT" --agent egm:egm
```

在第一条消息中提供包的绝对根目录与所需技能：

```text
EGM package root: /absolute/path/to/elephant-goldfish-model
Read that package's docs/copilot.md and run egm-brainstorm.
EGM artifact directory: .dev/docs
```

后续请求可使用 `egm-draft` 或 `egm-review`。评审时请提供目标文档。请务必使用 `egm:egm` 入口智能体，以便在执行技能前加载宿主映射。直接选择某个评审员作为主会话并不能建立新鲜的金鱼上下文。使用前请检查 `/agent` 是否存在命名冲突；个人配置档案可能会覆盖包内配置档案。

经验证的离线路径使用 `--plugin-dir`。通过 `copilot plugin install` 进行持久化安装尚未验证。1.0.82 版本的帮助文档未记录本地路径安装功能（尽管当前 GitHub 在线文档已有说明），请勿据此推断较新的命令接口在该版本上可用。

## 资源路径

从挂载的检出目录或原生插件清单中解析包根目录。对于已发现的 `skills/<skill>/SKILL.md`，向上回溯三级父目录即可得到包根目录。在委派之前，先从该根目录读取本指南与权威技能文件。

- 基于包根目录解析 `agents/`、`resources/` 和 `docs/`。
- 基于当前技能目录解析该技能的 `references/`。
- 基于项目工作目录解析用户文档。
- 向评审员传递绝对路径，包括包根目录、目标文档与文档标准。
- 保留路径中的空格与 Unicode 字符，切勿按空白符拆分路径。
- 如果任何必需资源不可用，立即中止；切勿用项目文件替代。

配置档案中的 `<EGM_ROOT>` 是由任务提供的路径，而非环境变量展开。本适配器不假定 Copilot 会在 Markdown 正文中自动展开变量。

## 原生委派

使用 Copilot 的原生自定义智能体委派工具，并检查其暴露的 schema。通过带命名空间的 ID 选择已注册配置档案，切勿编造工具参数。进行原生委派时，将每个权威专家角色名映射为 `egm:<name>`。入口配置档案允许使用 `agent` 进行委派及 `edit` 写入产物。每个评审员仅允许 `read` 和 `search`，评审员既不能继续委派，也不能写入报告。

| 配置档案 | 权威专家角色 |
| --- | --- |
| `egm:egm-design` | `agents/design_critic.md` |
| `egm:egm-codebase-details` | `agents/codebase_details.md` |
| `egm:egm-comprehension` | `agents/comprehension.md` |
| `egm:egm-implementation-readiness` | `agents/implementation_readiness.md` |
| `egm:egm-structural` | `agents/structural.md` |
| `egm:egm-style` | `agents/style.md` |
| `egm:egm-zoom-out` | `agents/zoom_out.md` |
| `egm:egm-inventive-catalyst` | `agents/inventive_catalyst.md` |

每次调用评审员（包括重复评审）都必须启动一个新的原生子智能体。任务内容仅包含所请求的模式、目标、行号范围、专家角色、文档标准以及显式阅读清单。切勿转发父对话、内部理由、记忆或目标文档本身应当解释的答案。切勿检索父历史记录或记忆。仅当权威工作流阶段明确要求且阅读清单中显式提供时，才读取先前的评审报告；切勿检索其他过往发现。例如，第 8 步（中期拉远视角，Intermediate Zoom Out）要求调用方在阅读清单中包含 `zoom_out_report.md` 的绝对路径，而不要转发其他先前的评审发现。

对于共享技能中的合成器步骤，请采用直接派发方式。读取权威的 `agents/synthesizer.md` 以获取评审员选择规则与报告格式，由主会话汇总结果并写入产物。切勿尝试嵌套委派或注册无权限限制的合成器。引导初始化工作子智能体（Bootstrap workers）不在本适配器的已验证范围内。

为评审员保持 `include-custom-instructions: false`。当前 GitHub 文档描述该设置适用于被委派的子智能体，而非作为主会话选中的智能体。本地版本是否强制执行该设置尚未验证。仅靠独立上下文窗口本身，并不足以证明指令、记忆或共享文件已完全隔离。

在进行综合汇总前，必须等待每个原生子智能体实际完成。在宿主提供记录的情况下，检查必需资源的读取情况。子智能体缺失、读取失败、超时或工具被拒都会导致该阶段不完整，必须中止并报告该限制。切勿模拟子智能体，也切勿将缺失评审意见视为通过结果。

## 人类门禁与产物

遵循权威技能的审批步骤与阶段顺序。当结构化提问工具不可用时，提出一条纯文本问题并等待回答。工具缺失不等于授予批准。切勿启用 autopilot 来绕过这些门禁。

除非用户提供了 `EGM artifact directory: <path>`，否则将产物写入工作目录。相对产物路径相对于该工作目录解析。保留技能规定的文件名并报告其最终路径。所有产物写入操作均留在主会话中执行；评审员仅返回评审意见。

## 验证与现存限制

离线运行 `pytest tests/unit/test_copilot_adapter.py` 和 `pytest tests/unit`。这些宿主专属测试检查配置档案、路径解析以及重定位包体中的权威资源完整性，不会调用模型。

在已安装 CLI 1.0.82 的环境下，可附带运行其离线原生发现检查：

```sh
EGM_COPILOT_OFFLINE=1 pytest tests/unit
```

2026-10-24，CLI 1.0.82 成功从挂载包中加载了全部八个配置档案与七个技能。该测试使用一次性的 `COPILOT_HOME` 和 `COPILOT_OFFLINE=true`，以 `--headless --stdio` 启动已安装的 CLI，随后在不发送提示词的情况下调用以下原生 SDK 协议操作：

1. `session.create`：使用本地拒绝请求的提供方创建一个会话。
2. `session.agent.list`：返回八条 `source: plugin` 记录及绝对配置档案路径。
3. `session.skills.list`：返回七个已启用技能及绝对权威路径。
4. `session.destroy`：销毁会话。

该测试校验精确的 ID、路径、工具列表、评审员的 `userInvocable: false` 以及空的智能体 MCP 定义。本地提供方收到 **0 次请求**。模型名 `gpt-4.1` 仅为会话元数据，并未执行任何模型调用。无需额外的清单映射。

所检查的 macOS arm64 CLI 二进制文件的 SHA-256 为 `948a5e69d924559fdfb2292d9cd8584b8fba28f8fd0349eeefc18711225e340f`。

代表性原生记录（其中本地根目录缩写为 `<package>`）：

```json
{"id":"egm:egm-design","source":"plugin","path":"<package>/com.github.copilot/agents/egm-design.agent.md","tools":["read","search"],"userInvocable":false,"mcpServers":{}}
{"name":"egm-review","commandName":"egm:egm-review","source":"plugin","path":"<package>/skills/egm-review/SKILL.md","enabled":true}
```

在该版本上，独立的 `skill list` 命令会遗漏挂载的技能，其输出并非会话加载器清单。本地 `plugins list --help` 也指出检查自定义智能体需要会话上下文。原生 SDK 清单无需运行模型即可填补这一发现缺口。

发现成功并不证明技能执行、写入拒绝或上下文隔离有效。RPC 清单不会暴露 `include-custom-instructions` 的强制执行情况。这条历史记录不标识所测试的 EGM 提交或已安装包的校验和；在将其用于某个固定候选版本前，请针对该候选版本重新运行发现检查。

在声明正式支持之前，必须在获得明确模型调用授权后，运行每个技能 `test_scenarios.yaml` 中现有的 brainstorm、draft 和 review 场景，并分别记录以下三类结果：

1. 安装：候选提交 SHA、包体标识、完整资源及已加载的配置档案。
2. 工作流：宿主版本、模型标识符、日期、原生子智能体启动与完成记录、必需读取记录、产物路径以及真实的人类审批暂停。
3. 隔离性：随机的仅父级可见标记、实际子上下文、注入的指令、记忆与 MCP 暴露情况，以及针对一次性测试夹具（fixture）的写入尝试。

必须检查工具调用记录以验证写入拒绝，评审员的口头承诺不等于强制执行。如果无法获取记录，则保持未验证状态。对共享文件的读取权限可能暴露先前评审发现；新鲜上下文并非文件系统沙箱。

## 官方参考资料

- [插件格式与优先级](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-plugin-reference)
- [自定义智能体与指令继承](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli)
- [智能体工具与别名](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [CLI 命令参考](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)
- [官方 SDK 协议操作](https://github.com/github/copilot-sdk/blob/main/python/copilot/generated/rpc.py)

这些参考资料描述的是宿主文档能力，而非 EGM 运行时测试证据。
