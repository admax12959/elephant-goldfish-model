> **原文路径**：`docs/designs/windsurf_support.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# Windsurf 工作区路由

- **作者：** [@n3wth](https://github.com/n3wth)
- **状态：** 提案（Proposal）。维护者范围审批与设计评审待定。
- **日期：** 2026-11-10
- **跟踪 Issue：** [#376](https://github.com/google/elephant-goldfish-model/issues/376)

## 问题陈述

Windsurf Cascade 从 `.windsurf/rules/*.md` 加载工作区规则，并从 `.windsurf/workflows/*.md` 加载斜杠工作流（slash workflows）。在 Windsurf 中打开 EGM 仓库的用户目前没有针对七个 EGM 工作流的原生入口。如果将技能指令直接复制到 `.windsurf/workflows/`，会导致与 `skills/*/SKILL.md` 发生漂移，并破坏金鱼新鲜上下文评审员隔离。

本规格定义了一个面向 Windsurf Cascade 的实验性工作区路由适配器。每个 Windsurf 入口直接路由至仓库根目录下的权威 `skills/`、`agents/` 与 `resources/` 文件。每当 Windsurf 缺乏隔离的只读子智能体委派能力时，金鱼蜂群工作流将立即中止。

## 范围

本节界定了 [#376](https://github.com/google/elephant-goldfish-model/issues/376) 下实验性 Windsurf 工作区路由的边界。

### 目标

该实验性适配器提供四项能力：

- 将全部七个 EGM 工作流暴露为不超过 15 行的 `.windsurf/workflows/egm-*.md` 斜杠命令，且不拷贝任何技能正文。
- 定义一条 `.windsurf/rules/egm.md` 工作区规则（`trigger: model_decision`），将 EGM 请求路由至根目录共享文件。
- 当新鲜上下文、只读子智能体不可用时，立即中止金鱼蜂群工作流（`egm-brainstorm`、`egm-review`、`egm-review-loop`）。
- 在 `tests/unit/test_windsurf.py` 中离线验证路由契约、技能覆盖率与行数限制。

### 非目标

本次变更排除以下四项内容：

- 在真实工作流执行、只读强制执行与评审员隔离性测试全部通过前，宣称已验证支持 Windsurf。
- 在 `.windsurf/` 内复制 `skills/*/SKILL.md` 或 `agents/*.md` 中的提示词规则。
- 支持非根目录安装或文件系统符号链接（这两者在 Windows 检出与多宿主插件校验器上均会失败）。
- 在阶段 1 构建自定义 Windsurf 扩展二进制文件或 MCP 子智能体服务端。

## 现有组件与权责划分

所有权威工作流行为均保留在仓库根目录的 `skills/`、`agents/` 与 `resources/` 中。当前宿主边界参见 [harnesses](../harnesses.md) 与 [TESTING.md](../../TESTING.md)。

Windsurf 适配器贡献者负责维护 `.windsurf/`、[harnesses](../harnesses.md) 中的状态条目以及 `tests/unit/test_windsurf.py`。仓库维护者审批特性范围，并决定何时依据真实测试记录将 Windsurf 从实验性状态升级。人类用户掌握所有设计决策与工作流审批门禁。

## 提议架构

该适配器要求工作区根目录检出，使 `.windsurf/` 与 `skills/`、`agents/` 和 `resources/` 并列。它新增了 `.windsurf/` 下的 8 个极简 Markdown 文件、[harnesses](../harnesses.md) 中的一段状态说明以及 1 个单元测试模块。

### 组件布局

下表列出适配器中的每个文件及其契约。

| 路径 | 角色 | 约束 |
| --- | --- | --- |
| `.windsurf/rules/egm.md` | 工作区路由与金鱼隔离规则 | 使用 `trigger: model_decision` frontmatter，不超过 20 行 |
| `.windsurf/workflows/egm-*.md` | 指向 `skills/egm-*/SKILL.md` 的七个斜杠命令指针 | 不超过 15 行，零拷贝技能正文 |
| `docs/harnesses.md` | 面向用户的宿主状态记录 | 将 Windsurf 标记为实验性，并注明缺少真实隔离验证证据 |
| `tests/unit/test_windsurf.py` | 离线契约测试 | 验证规则 frontmatter、技能一致性、目标路径以及隔离拆分 |

### 执行与隔离流

当用户在仓库根目录的 Windsurf Cascade 中运行 `/egm-<workflow>` 时，执行遵循以下规则：

1. Cascade 读取 `.windsurf/workflows/egm-<workflow>.md` 与 `.windsurf/rules/egm.md`。
2. Cascade 相对于工作区根目录解析 `skills/egm-<workflow>/SKILL.md`、`agents/*.md` 与 `resources/document_standards/*.md`。
3. 单会话及可内联工作流（`egm-draft`、`egm-resolve`、`egm-drift-report`、`egm-bootstrap`）在当前会话中运行。在 `egm-bootstrap` 中，当不存在子智能体工具时 Cascade 内联执行侦察步骤，与 [egm-bootstrap](../../skills/egm-bootstrap/SKILL.md) 保持一致。
4. 金鱼蜂群工作流（`egm-brainstorm`、`egm-review`、`egm-review-loop`）要求具备零父历史的隔离只读子智能体。在执行评审员提示词前，工作流包装器指示 Cascade：若原生只读子智能体委派不可用则立即停止，报告同上下文评审不属于金鱼评审，且不调用任何写工具或 Shell 工具。
5. 每名金鱼评审员均在宿主的只读子智能体配置（Devin CLI 中为 `subagent_explore`，不暴露任何写、编辑或 Shell 工具）上启动。规则与三个金鱼包装器禁止使用可写配置，且禁止恢复（resume）评审员会话。评审员以文本形式返回发现项。仅有父智能体可以写入文件：即技能分配给父智能体的产出，或在用户批准写入后用户请求的文件。

### 人类审批门禁

当没有结构化提问组件可用时，Windsurf Cascade 使用普通聊天回复。适配器保留权威技能中的每一处人类暂停门禁：

- [Brainstorm](../../skills/egm-brainstorm/SKILL.md)：在派发金鱼评审员前等待用户确认就绪，在交接前等待简报批准。
- [Draft](../../skills/egm-draft/SKILL.md)：等待输入采集签核、逐节审批与输出路径审批。
- [Review](../../skills/egm-review/SKILL.md)：在具备子智能体委派能力后，等待预设选择与阶段暂停选择。
- [Resolve](../../skills/egm-resolve/SKILL.md)：呈现建议修复与提出异议的条目，并在编辑文件前等待用户明确批准。

## 验证与发布门禁

验证将隔离的 CI 离线测试与真实 Windsurf Cascade 探针分开，与 [additional-harnesses.md](additional-harnesses.md) 和 [TESTING.md](../../TESTING.md) 保持一致。

### 离线测试

自动化 CI 在未安装 Windsurf 的 Linux 与 Windows 上运行 `tests/unit/test_windsurf.py`：

- 验证 `.windsurf/rules/egm.md` 包含合法 YAML frontmatter、设置 `trigger: model_decision` 并声明 `fresh-context` 规则。
- 验证 `.windsurf/workflows/*.md` 与 `EXPECTED_SKILLS` 一一对应，且每个文件不超过 15 行。
- 验证金鱼蜂群工作流（`egm-brainstorm`、`egm-review`、`egm-review-loop`）携带子智能体中止规则，而单会话及内联工作流（`egm-draft`、`egm-resolve`、`egm-drift-report`、`egm-bootstrap`）不会误中止。
- 验证规则与三个金鱼包装器均指明只读子智能体配置、禁止恢复评审员，并将文件写入保留给父智能体。

### 真实环境晋升门禁

在贡献者于固定 Windsurf 版本上记录以下三项门禁的真实证据前，Windsurf 保持实验性状态：

1. **工作流执行**：在干净工作区中运行 `egm-brainstorm`、`egm-draft` 与 `egm-review`，验证交互式人类暂停。
2. **只读强制执行**：尝试从金鱼评审员子智能体写入一次性工作区文件，验证宿主拒绝工具调用。
3. **新鲜上下文隔离**：在父会话中植入随机标记，检查 Windsurf 轨迹日志以确认子评审员既未收到父历史也未收到同级发现项。

门禁 2 需要两个探针。未提及 EGM 工作流的请求不会加载 `model_decision` 规则，因此不能用来测试适配器。探测宿主能力时，要求只读配置上的子智能体创建一个文件；探测适配器时，在 `/egm-review` 内部要求每名评审员自行保存笔记。

<a id="cli-2026-11-10"></a>

### CLI 实测证据（2026-11-10）

Cognition 于 2026-06-02 将 Windsurf 更名为 Devin Desktop。Devin Local 取代 Cascade 成为本地智能体，旧版 Cascade 智能体已于 2026-07-01 停用。Devin Desktop 与 Devin CLI 运行同一个 Devin Local 智能体。本次运行未安装 Devin Desktop；作者在 macOS 27.2 上通过 Muse 1.4.15 会话，以无头模式和 `tmux` 交互模式驱动 Devin CLI 3000.12.43，针对 [#379](https://github.com/google/elephant-goldfish-model/pull/379) 适配器进行了测试。该 CLI 将 `.windsurf/rules/egm.md` 列为 Windsurf 规则，但未将 `.windsurf/workflows/` 列为斜杠命令，`/egm-*` 请求通过该规则完成路由。由于所有文件写入均被拒绝，工作流未运行至产出最终产物。

| 门禁 | CLI 结果 | 证据 |
| --- | --- | --- |
| 工作流执行 | 路由与暂停通过 | `egm-brainstorm`、`egm-draft` 与 `egm-review` 均读取了各自的 `SKILL.md`。Draft 在写入前暂停三次；Brainstorm 在派发 7 名评审员前等待就绪确认；Review 在输入采集处暂停 |
| 只读强制执行 | 通过 | 只读配置下的子智能体没有任何写工具。在 EGM 工作流内，22 次评审员启动全部使用该配置且零恢复。用户要求的评审员笔记文件以父智能体写入请求的形式呈现给用户，并未实际写入 |
| 新鲜上下文隔离 | 通过 | 随机标记在 2 份评审员任务文本中出现 0 次，在宿主会话存储的 70 个子转录节点中出现 0 次 |

上述运行不满足晋升门禁要求：它们使用的是 CLI 而非 IDE，且在生成最终产物前即停止。

## 备选方案考量

下表从维护成本与隔离安全性两个维度评估各备选设计方案。

| 备选方案 | 决策 | 理由 |
| --- | --- | --- |
| 将 `SKILL.md` 正文拷贝到 `.windsurf/workflows/*.md` | 拒绝 | Windsurf 将每个工作流限制在 12,000 字符以内，且拷贝提示词每次编辑都会漂移 |
| 将 `.windsurf/workflows/` 符号链接到 `skills/` | 拒绝 | Windsurf 期望扁平的 `.md` 文件，且符号链接在 Windows 检出与打包器中均会失败 |
| 在单次对话中顺序模拟金鱼智能体人格 | 拒绝 | 共享对话历史违背 EGM 新鲜上下文评审员独立性 |
| 构建 MCP 服务端以派生外部评审员 | 推迟 | MCP 服务端引入运行时部署与 API 密钥管理，若 Windsurf 始终不提供原生子智能体 API，应另行提案 |
| 在真实隔离测试通过前暂缓提交所有 Windsurf 文件 | 拒绝 | 极简路由文件可让用户安全运行 `egm-draft` 与 `egm-resolve`，同时拦截模拟评审 |

## 风险与缓解措施

下表列出技术约束、安全风险与缓解措施。

| 风险 | 影响 | 缓解措施 |
| --- | --- | --- |
| Cascade 忽略自然语言规则并内联模拟评审 | 破坏金鱼隔离并暴露工作区写工具 | 保持 Windsurf 为实验性状态，在三个金鱼包装器中写入立即中止指令，并禁止在评审路径调用写或 Shell 工具 |
| Windsurf 对每个工作流文件强制执行 12,000 字符上限 | 大型工作流无法在 Cascade 中加载 | 在 `test_windsurf.py` 中将 `.windsurf/workflows/*.md` 限制在 15 行以内，完整提示词保留在 `skills/` |
| 用户仅复制 `.windsurf/` 而未包含仓库根目录 | 相对 `skills/` 路径无法打开 | 在 `.windsurf/rules/egm.md` 中声明工作区根目录布局要求，使缺失文件立即报错 |
| 未指定 EGM 工作流的请求要求启动评审员子智能体 | 宿主不加载规则，选中可写配置并提议替评审员写文件 | 超出适配器范围。该规则仅约束 EGM 工作流；上述门禁 2 探针在工作流内运行 |
| 用户在交互式会话中拒绝权限弹窗 | 宿主无提示结束当前轮次并丢弃批量工具调用 | 属于宿主行为。发送后续消息即可继续；将其作为限制记录在 [harnesses](../harnesses.md) |
| 个人会话钩子或 MCP 服务端注入指令 | 无头运行可能在读取技能前因工具调用被拒而中止 | 使用不含钩子的宿主配置运行真实探针 |
| 新增 EGM 技能时遗漏对应 Windsurf 工作流 | 斜杠命令列表落后于 `skills/` | `test_windsurf.py` 在每个 Pull Request 上断言与 `EXPECTED_SKILLS` 严格相等 |

## 发布与回滚

发布采用 [#376](https://github.com/google/elephant-goldfish-model/issues/376) 下的双 PR 栈：

1. 合并本阶段 1 规格文档及其在 [README.md](README.md) 中的索引行（`Refs #376`）。
2. 合并叠放在其上的阶段 2 实现 [#379](https://github.com/google/elephant-goldfish-model/pull/379)（`Refs #376`），在真实 Windsurf 验证通过前保持 [#376](https://github.com/google/elephant-goldfish-model/issues/376) 开启。

回滚只需单次回退 [#379](https://github.com/google/elephant-goldfish-model/pull/379)，即可移除 `.windsurf/`、[harnesses](../harnesses.md) 中的状态说明与 `tests/unit/test_windsurf.py`，不影响任何其他宿主环境。

## 待决事项与评审状态

- 在 [#379](https://github.com/google/elephant-goldfish-model/pull/379) 合并前，维护者必须评审 [#376](https://github.com/google/elephant-goldfish-model/issues/376) 下的阶段 1 范围。
- 作者于 2026-11-10 在 Devin CLI 上运行了子智能体委派、只读拒绝与会话存储核查（见[已记录的 CLI 证据](#cli-2026-11-10)）。尚无贡献者在 Windsurf IDE（Devin Desktop）中运行，也尚未运行至产出最终产物。
- 维护者需决定 Devin CLI 证据是否可计入晋升门禁，还是仅认可 IDE 证据（两者底层均运行 Devin Local）。
- 作者于 2026-11-10 对照特性规格标准运行了 EGM 结构化、风格与设计批判者金鱼评审员，并在本草案及 [#379](https://github.com/google/elephant-goldfish-model/pull/379) 中解决了全部 Blocker 与 Major 发现项。

## 调研参考资料

查阅于 2026-11-10。以下链接仅记录 Windsurf 平台能力，不构成 EGM 验证证据：

- [Windsurf is now Devin Desktop](https://devin.ai/blog/windsurf-is-now-devin-desktop)
- [Devin Desktop FAQ](https://docs.devin.ai/desktop/devin-desktop-faq)
- [Windsurf Cascade workflows](https://docs.windsurf.com/windsurf/cascade/workflows)
- [Windsurf Cascade memories and rules](https://docs.windsurf.com/windsurf/cascade/memories)
- [扩展编程智能体宿主环境支持](additional-harnesses.md)
- [宿主环境集成](../harnesses.md)
- [EGM 宿主测试记录](../../TESTING.md#harness-test-record)
