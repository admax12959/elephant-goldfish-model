> **原文路径**：`AI_POLICY.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 人工智能与自动化贡献政策

我们鼓励在为本代码库开发变更时使用 AI 工具。但是，本仓库不接受纯粹由 AI 自主提交的贡献；人类贡献者必须审阅、理解并能够解释每一项提交的变更。

## 操作规则

1. **人类问责制（Human Accountability）**：每个 Pull Request 都必须有一位人类作者，亲自审阅、测试并验证过每一行变更代码。
2. **交互式代码评审（Interactive Code Review）**：贡献者必须在评审中亲自为自己的变更作出说明与辩护。禁止将维护者的评审意见直接复制给大语言模型，再把未经核实的模型回复粘贴回评审讨论串。
3. **EGM 设计要求与双阶段对齐（EGM Design Requirement & Two-Stage Alignment）**：计划提交特性或较大变更（非测试代码 `> 100` 行，或涉及 `skills/`、`agents/` 中 `>= 3` 个核心文件）的贡献者应当：
   * **a)** 首先通过[特性咨询 Issue](https://github.com/google/elephant-goldfish-model/issues/new?template=feature_proposal.md)（`.github/ISSUE_TEMPLATE/feature_proposal.md`）与维护者沟通对齐，以获取 `design-ready` 标签。
   * **b)** 提交并合并完整的 EGM 设计文档 PR（通过 `/egm-brainstorm`、`/egm-draft` 和 `/egm-review` 编写 `docs/designs/<name>.md`，并使用 `Refs #<issue>` 关联该 Issue），以便维护者为该特性咨询 Issue 贴上 `pr-ready` 标签。
   * **c)** 提交并合并经批准设计的实现 PR（使用 `Closes #<issue>`）。

   贡献者可以在贴上上述标签之前，将 (b) 和 (c) 作为堆叠 Pull Request 提前开启。每个 PR 在合并时仍须具备各自对应的标签。详见 [CONTRIBUTING.md](CONTRIBUTING.md#堆叠-pull-request-与-pr-规模)。
4. **禁止人为拆分 PR（No Artificial PR Splitting）**：切勿为了绕过 100 行范围阈值，将单一逻辑特性或重构人为拆分成多个较小的 Pull Request（例如拆成两个 60 行的 PR）。在同一 Issue 与同一份设计文档下的小型堆叠 Pull Request 不属于人为拆分。
