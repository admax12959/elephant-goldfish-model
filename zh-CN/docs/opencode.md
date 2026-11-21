<!-- Copyright 2026 Google LLC. SPDX-License-Identifier: Apache-2.0 -->

> **原文路径**：`docs/opencode.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# OpenCode V1 集成指南（实验性）

本适配器面向 OpenCode CLI **1.18.29**，基于 V1 的 `task` 和 `permission` 接口，不面向 V2 的 `subagent` 或 `permissions` 接口。安装检查并不代表工作流或隔离性已获支持。维护者审批、实时工作流与隔离性验证仍在 [#146](https://github.com/google/elephant-goldfish-model/issues/146) 中等待推进。提议的架构见[附加宿主环境设计文档](designs/additional-harnesses.md)。

## 安装

需要 Python 3.10+ 以及支持符号链接的 POSIX 文件系统。安装前请选定一个固定的候选提交，并在安装后保持该安装位于原始路径不变。安装器会复制完整的权威资源目录树，不会修改全局或项目配置，且会拒绝写入已存在的目标路径。每次测试候选版本请使用全新的目标目录。

```sh
git rev-parse HEAD
python3 adapters/opencode/install.py /absolute/egm-opencode-candidate
opencode --version
```

启动前请确认版本为 `1.18.29`。如有必要，可按如下方式安装该精确版本：

```sh
npm install --prefix /absolute/opencode-v1 opencode-ai@1.18.29
/absolute/opencode-v1/node_modules/.bin/opencode --version
```

使用前请检查生成的 `opencode.json` 与 `installation.json`。后者记录了包体标识以及所有随包文件的 SHA-256 哈希。原生配置包含七个命令、八个评审员以及 `egm` 主智能体。原生技能别名沿用权威 frontmatter 名称，并链接至未经修改的技能目录。

使用候选配置在项目目录中启动 OpenCode：

```sh
OPENCODE_CONFIG=/absolute/egm-opencode-candidate/opencode.json opencode --agent egm
```

模型调用需要单独的用户授权；安装与版本检查不涉及模型调用。使用原生技能前请先选择 `egm`，其他主智能体不会收到本适配器的派发指令。提交评审文本时，请使用普通消息而非斜杠命令，例如：

```text
Use the egm-review skill to review /absolute/path/design.md.
```

请将待评审材料粘贴到普通消息中，或保存为文件供智能体读取。生成的 `/egm-brainstorm`、`/egm-draft` 和 `/egm-review` 命令保留了 OpenCode 的原生命令语法，其参数并非纯字面文本：在智能体工具权限生效之前，OpenCode 会先求值 Shell 标记（感叹号后跟反引号包裹的命令）。切勿将不受信任的评审材料粘贴到斜杠命令参数中；Markdown 代码围栏无法禁用这一预处理机制。

如需卸载，请停止相关会话，仅删除该候选目录并移除其环境变量覆盖即可。

## 路径与派发

在生成的提示词中使用绝对包体根目录：基于该根目录解析 `agents/`、`resources/` 和 `docs/`；在解析技能别名后，基于权威技能目录解析 `references/`；基于项目目录解析用户文件。切勿用项目文件替代插件内置的专家角色。

通过 V1 `task` 工具从 `egm` 直接派发评审员。将 `subagent_type` 设为专家角色的权威 frontmatter 名称，提供 `description` 和 `prompt`，并省略 `task_id` 以创建全新会话。使用前台任务，切勿启用后台派发或复用评审员会话。主智能体负责执行综合汇总并写出报告，不安装嵌套合成器适配器。引导初始化（Bootstrap）工作子智能体与侦察智能体（scout）委派不在本适配器的覆盖范围内：尽管全部七个技能都可被发现，但任务白名单仅包含八个评审员类型。如果引导初始化需要编排器或侦察智能体，请立即中止，切勿用评审员替代或模拟缺失的工作智能体。

请传递专家角色、目标文档、文档标准以及选定源文件的绝对路径。切勿传递父历史记录、内部推理过程或记忆。仅当权威阶段明确要求时才提供先前报告的路径，例如第 8 步（中期拉远视角，Intermediate Zoom Out）所需的 `zoom_out_report.md`。仅在显式提供且为权威阶段所必需时才读取先前报告，切勿主动检索未经请求的历史记录、记忆或早期评审发现。综合汇总前必须确认所有指定读取均已完成且任务实际结束。若出现启动缺失、完成缺失、必需读取缺失或结果畸形，须报告为不完整。若原生委派不可用则中止执行，切勿在父对话中模拟金鱼评审员。

当 brainstorm 先于 draft 执行时，须先完成全部七名构思评审员的评审，将其发现汇总为 `concepts_brief.md` 后再开始起草。在 review 阶段，须写出当前已完成阶段的报告后再启动下一阶段。进入第 8 步前，须验证产物目录中存在 `zoom_out_report.md` 并向评审员传递其绝对路径；若该报告缺失则中止。

评审员拒绝除 `read` 和 `glob` 之外的所有工具。之所以拒绝 `grep`，是因为其搜索会绕过宿主的逐文件 `read` 规则（包括 `.env` 排除规则）；请使用受允许的文件读取获取评审上下文。V1 会将 MCP 资源读取与资源列表映射至 `read` 权限：评审员的 `read` 规则允许本地读取，随后拒绝 `mcp:*`，从而拦截 `mcp:<server>:<uri>` 读取以及 `mcp:<server>:*` 资源或模板列表。MCP 资源工具名可能仍可见，但其资源访问会在权限检查处被拒绝。适配器的外部目录例外仅覆盖已安装的包体；OpenCode 也会在解析后的智能体配置中添加对其工具输出目录的访问权限。Shell 命令、编辑、网络工具、MCP 工具、技能调用与委派均保持拒绝。主智能体在执行写入等其他工具时会请求权限。如需评审外部目标文件，请将测试夹具移入项目目录，或单独审查一项最小化权限变更。

遵循用户提示词或已加载指令中的 `EGM artifact directory: <path>`，将相对输出目录相对于项目目录解析；否则保留权威输出文件名与默认工作目录。严格保留每一个就绪、分节与阶段审批门禁。当原生提问工具不可用时，提出一条纯文本问题并等待用户回答；工具缺失或时间流逝均不构成批准。

## 验证限制

OpenCode 会合并自定义、全局、项目与受管配置。运行前请检查最终生效的配置，因为后续配置可能会覆盖这些权限。请使用干净的配置档案与一次性项目进行验证，并分别检查继承的指令、插件、MCP 服务、记忆与共享文件。只读工具配置不等于进程或文件系统隔离，新鲜子会话本身也不足以证明金鱼隔离。

运行离线检查：

```sh
pytest tests/unit/test_opencode_adapter.py
pytest tests/unit
```

这些测试校验安装过程、资源一致性、原生配置以及拒绝覆盖已有文件的行为；它们不模拟宿主完成记录，也不证明宿主层面的强制执行。常规 CI 中不会运行实时模型测试。

限定范围的 OpenCode 原生探针使用脚本化本地模型而非实时模型，用于检验原生子智能体派发、单个父级标记以及对合成 `.env` 读取的拒绝。它需要固定版本的 OpenCode 1.18.29 Linux x64 可执行文件：

```sh
npm ci --ignore-scripts --prefix tests/integration/opencode-ci
python tests/integration/opencode_native_probe.py \
  --source . \
  --opencode tests/integration/opencode-ci/node_modules/opencode-linux-x64/bin/opencode \
  --output /tmp/egm-opencode-probe-new
```

请选择一个尚不存在的输出路径。探针通过并不代表实时模型行为、完整工作流或通用评审员隔离已获验证。

2026-10-24，OpenCode `1.18.29` 通过 `debug config` 与 `debug agent egm-style` 接受了生成的配置。历史上评审员工具集曾启用 `read`、`glob` 和 `grep`，当前候选版本已拒绝 `grep` 以维持 `.env` 读取限制。`debug skill` 命令从已安装的候选路径中发现了全部七个技能。这些命令未发起任何模型调用，该结果也不证明任务执行或运行时权限强制执行。独立的 XDG 目录并未阻止对主目录（home-directory）技能的发现，干净的验证配置档案也必须将这些位置纳入考量。本历史记录未标识所测试的 EGM 提交或已安装包的校验和；在将其用于某个固定候选版本前，请针对该候选版本重新运行发现检查。

在声明正式支持之前，须获得明确授权并完成以下检查：

1. 记录候选提交 SHA、已安装文件哈希、宿主版本、模型标识符、交互面及日期。
2. 运行 brainstorm、draft 和 review，并包含真实的人类审批暂停。
3. 保留原生评审员启动、完成、必需读取及生成产物的记录。
4. 针对一次性文件探测写入拒绝，并检查生效的子智能体权限。
5. 仅在父对话中放置一个随机标记，检查子智能体上下文与输出。
6. 分别报告安装、工作流与隔离性结果，将缺少证据的项目标为未验证。

针对 OpenCode 1.18.29，离线校验器可将导出的单个子会话与父 JSONL 轨迹进行比对。请传入每个必需的绝对 POSIX 文件路径：

```sh
opencode export <child-session-id> > child.json
python tests/integration/opencode_trace_verifier.py \
  --parent-trace parent.jsonl --child-export child.json \
  --require /absolute/path/to/agents/zoom_out.md \
  --require /absolute/path/to/target.md \
  --require /absolute/path/to/standard.md
```

校验器会核对原生子会话与父会话标识、任务完成状态与时间窗口、评审员类型，以及每个工具调用的消息和会话 ID；要求是不带 `task_id` 的新任务；要求在子智能体最终回复之前，对每个传入路径都有非空且成功的文件读取记录，并将该回复与父任务结果匹配。缺失、冲突、畸形或不受支持的证据均会导致校验失败。父 JSONL 记录仅按字面 LF 拆分，不按 Unicode 分隔符拆分。除非调用方显式传入，否则该工具不会自动校验所有必需路径；它也不能证明整文件覆盖、会话记录真实性、上下文隔离或运行中工作流在缺少证据时确实中止。切勿将父级摘要或未出现标记视为充分的隔离证据。

## 接口参考来源

审阅日期：2026-10-24。以下版本化源码定义了本适配器的接口：

- [V1 智能体配置](https://github.com/anomalyco/opencode/blob/v1.18.29/packages/web/src/content/docs/agents.mdx)
- [V1 配置与合并顺序](https://github.com/anomalyco/opencode/blob/v1.18.29/packages/web/src/content/docs/config.mdx)
- [V1 技能发现](https://github.com/anomalyco/opencode/blob/v1.18.29/packages/web/src/content/docs/skills.mdx)
- [V1 命令语法](https://github.com/anomalyco/opencode/blob/v1.18.29/packages/web/src/content/docs/commands.mdx)
- [V1 命令预处理](https://github.com/anomalyco/opencode/blob/v1.18.29/packages/opencode/src/session/prompt.ts)
- [V1 task 实现](https://github.com/anomalyco/opencode/blob/v1.18.29/packages/opencode/src/tool/task.ts)
- [V1 权限求值器](https://github.com/anomalyco/opencode/blob/v1.18.29/packages/opencode/src/permission/index.ts)
- [V1 MCP 资源权限路径](https://github.com/anomalyco/opencode/blob/v1.18.29/packages/opencode/src/session/tools.ts)
- [当前官方智能体指南](https://opencode.ai/docs/agents/)
