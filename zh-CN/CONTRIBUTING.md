> **原文路径**：`CONTRIBUTING.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 向大象-金鱼模型（EGM）插件贡献代码

我们欢迎社区贡献！当新的构想、技能与评审员视角在开源社区中共享时，所有人都会从中受益。

## 我的变更属于哪条轨道？

在开启 Issue 或 PR 之前，请先选择对应的贡献轨道。各轨道的完整规则见[贡献轨道与工作流](#contribution-tracks--workflow)；本节提供快速指引。

*   **在轨道 1 限制范围内的小型修复**：使用 [PR 模板](../.github/PULL_REQUEST_TEMPLATE.md) 直接开启 PR。
*   **新特性、架构变更，或任何超出轨道 1 限制的变更**：先开启一个[特性提案 Issue](https://github.com/google/elephant-goldfish-model/issues/new?template=feature_proposal.md) 并遵循轨道 2 的步骤。现有设计文档列表见 [docs/designs/README.md](docs/designs/README.md)。

**示例**：如果某个文档链接失效，请在分支上修复目标地址，按照 [TESTING.md 中的测试配置](TESTING.md) 准备好环境，并运行 `pytest tests/unit/test_link_integrity.py`。随后使用 [PR 模板](../.github/PULL_REQUEST_TEMPLATE.md) 开启一个轨道 1 PR 并记录测试结果。此类小型修复无需事先提交 Issue 或设计文档。

如有使用方面的问题，请前往 [Discussions Q&A](https://github.com/google/elephant-goldfish-model/discussions/categories/q-a)。

## 设计原则

以下通用原则指导着 EGM 框架的开发：

*   **人类对结果负责（Humans Own the Outcomes）**：AI 是探索与评审伙伴，但人类工程师始终对所有设计决策和代码负最终责任。
*   **产物优先交互协议（Artifact-First UX）**：大型提案、草稿和评审报告应当写入专用文件，而不是倾倒在聊天信息流中。
*   **活体规格（Living Specifications）**：代码与设计规格文档必须同步演进。

<a id="contribution-tracks--workflow"></a>

## 贡献轨道与工作流（Contribution Tracks & Workflow）

请撰写简短的 PR 标题，以动作动词开头，使用句首大写（sentence case）并描述具体变更。在不影响语义清晰的前提下省略多余词汇；为保证表意清楚，请保留必要的冠词（如 "a" 和 "the"）。
示例：`Explain ignored agents folder warning in Claude Code`。

请添加适用的标签：`docs` 表示文档、`site` 表示官网、`ci` 表示 GitHub Actions、`deps` 表示依赖项。如果你没有权限添加标签，请在 PR 描述中给出建议。`design-ready` 与 `pr-ready` 由维护者控制。PR 合并时才需要具备对应标签，你可以提前开启 PR。自动化流程负责管理 `design-needed` 与 `triage: passed` 标签。

所有贡献均须遵守我们的[AI 与自动化贡献政策](AI_POLICY.md)：每个 Pull Request 都必须有一位亲自评审并测试过该变更的人类作者，且作者应能独立解释这些改动。

贡献遵循 [`.github/PULL_REQUEST_TEMPLATE.md`](../.github/PULL_REQUEST_TEMPLATE.md) 中定义的双轨准入模型：

### 轨道 1：缺陷修复 / 小型补丁（非测试代码 `<= 100` 行，核心文件 `< 3` 个）
针对性修复、拼写纠错以及测试补充，只要不改变核心智能体行为且不触及 3 个及以上核心文件（`skills/`、`agents/`），均可直接作为 Pull Request 提交，无需事先提交 Issue 或设计文档。
若 PR 超出上述限制且未附带已批准的设计文档，将会收到提示性标签（`design-needed`），建议改走双阶段对齐流程。

### 轨道 2：特性 / 架构变更（双阶段对齐）
任何新增技能、智能体人格，或超出 `100` 行非测试代码（或触及 `skills/`、`agents/` 中 `>= 3` 个核心文件）的变更，均须遵循 EGM 规格驱动开发（SDD）流程：

1.  **沟通并达成对齐（`design-ready`）**：
    在投入大规模实现或编写完整规格前，先开启一个[特性 / 架构咨询 Issue](https://github.com/google/elephant-goldfish-model/issues/new?template=feature_proposal.md)（`.github/ISSUE_TEMPLATE/feature_proposal.md`），概述问题陈述以及拟议的 `Goals`（目标）与 `Non-Goals`（非目标）。我们推荐使用 `/egm-brainstorm` 来完成这一部分。维护者评审并在方向达成一致后，会贴上 `design-ready` 标签。

2.  **起草并评审设计规格**：
    在 `docs/designs/<name>.md` 下起草技术设计规格。我们推荐使用 `/egm-draft` 起草，并在 [`docs/designs/README.md`](docs/designs/README.md) 中登记。同时建议运行 `/egm-review docs/designs/<name>.md` 对提案进行压力测试。

3.  **提交设计文档 PR 并等待 `pr-ready`**：
    你可以在贴上 `design-ready` 标签前就开启此 PR，但须在贴标后才能合并。开启一个专门的 **Design Doc PR**，包含 `docs/designs/<name>.md` 与 `docs/designs/README.md`（使用 `Refs #<issue>` 而非 `Closes #<issue>` 关联已对齐的 Issue，以保持 Issue 处于开启状态）。维护者合并 Design Doc PR 后，规格文档即正式入库，随后维护者会手动将 Issue 标签切换为 `pr-ready`。

4.  **提交实现 PR**：
    开启关联 `docs/designs/<name>.md` 的实现 PR（使用 `Closes #<issue>`）。该 PR 须在规格文档合并且 Issue 获得 `pr-ready` 标签后方可合并。若想提前开工，可将其作为草稿 PR 堆叠在 Design Doc PR 之上。

### 堆叠 Pull Request 与 PR 规模

*   **堆叠（Stack）**：当一个 PR 依赖另一个 PR 时，请使用 GitHub [堆叠 Pull Request](https://docs.github.com/en/pull-requests/how-tos/stacked-pull-requests)。Design Doc PR 以 `main` 为目标分支；实现 PR 以设计分支为目标分支，并在设计合并前保持草稿状态。使用 `gh stack submit`，或对已开启的 PR 使用 `gh stack link <PR> <PR>`。GitHub 不会自动合并堆叠 PR。此功能需要上游仓库的分支权限。Fork 贡献者可以在本地准备好堆叠分支，待 Design Doc PR 合并后再开启实现 PR。
*   **规模（Size）**：每个 PR 只聚焦单一目标。评审者需要阅读的变更行数力争控制在约 200 行左右，超过约 400 行时应进行拆分（参阅 Google 的[小型变更](https://google.github.io/eng-practices/review/developer/small-cls.html)指南）。重命名、移动和删除操作应单独放在一个 PR 中。每个 PR 必须在底层 PR 之上通过全部测试。
*   **关联（Links）**：一个特性对应一个 Issue。堆叠中较低层的 PR 使用 `Refs #<issue>`，最顶层的 PR 使用 `Closes #<issue>`。若小型补丁有关联 Issue，则使用 `Fixes #<issue>`。后续工作请另开新 Issue。
*   **限制（Limits）**：轨道限制适用于整个堆叠。严禁为了规避设计流程而将完整特性拆分成多个小型补丁。

## 自动化校验与测试

*   **技能定义**：确保所有 `SKILL.md` 文件包含有效的 YAML Frontmatter（`name`、`description` 以及可选的 `disable-model-invocation`）。
*   **评估套件**：每项技能必须包含同目录的行为评估套件 `skills/<skill_name>/test_scenarios.yaml` 以及验证说明 `skills/<skill_name>/TEST.md`。运行静态检查（`pytest tests/unit`）的方法请参阅 [TESTING.md](TESTING.md)。
*   **格式规范**：使用标准格式化工具（如 Prettier 或 markdownlint）保持 Markdown 文件整洁规范。

## 贡献者许可协议（CLA）

对本项目的贡献必须附带[贡献者许可协议](https://cla.developers.google.com/about)（CLA）。你（或你的雇主）仍保留贡献内容的版权；签署 CLA 仅是授予我们将你的贡献作为项目一部分进行使用和分发的许可。

如果你或你当前的雇主已经签署过 Google CLA（即使是针对其他项目），通常无需再次签署。

访问 <https://cla.developers.google.com/> 查看你当前的协议状态或签署新协议。

## 社区行为准则

请参阅我们的[无障碍声明](ACCESSIBILITY.md)了解无障碍目标、当前限制以及如何反馈无障碍问题。

本项目遵循 [Google 开源社区行为准则](https://opensource.google/conduct/)。
