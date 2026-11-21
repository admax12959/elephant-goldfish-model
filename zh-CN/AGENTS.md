> **原文路径**：`AGENTS.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 大象-金鱼模型（EGM）智能体开发指南

你正在协助开发 EGM 代码仓库。

## 贡献边界

* **禁止自主提交 PR（No Autonomous PRs）**：未经明确的人类评审、测试与批准，严禁自动开启 Pull Request。
* **缺陷修复不写设计文档（No Design Docs for Bug Fixes）**：严禁为 `Bug Fix / Minor Patch`（缺陷修复 / 小型补丁）在 `docs/designs/` 下创建文件。新的设计文档属于 `Feature / Architectural Change`（特性 / 架构变更）轨道的第 1 阶段 PR。
* **分诊分类规范（Triage Taxonomy）**：确保人类操作者在 [`.github/PULL_REQUEST_TEMPLATE.md`](../.github/PULL_REQUEST_TEMPLATE.md) 中选择了正确的轨道：
  * `Bug Fix / Minor Patch`（非测试代码差异 `<= 100` 行，且涉及 `skills/` 或 `agents/` 中的核心文件数 `< 3`；无需新建设计文档，但若适用请更新现有设计文档）。
  * `Feature / Architectural Change — Stage 1: Design Doc PR`（需要一个已对齐且带有 `design-ready` 标签的 Issue，使用 `Refs #<issue>`，并新增 `docs/designs/<name>.md` 以及更新 `docs/designs/README.md`）。
  * `Feature / Architectural Change — Stage 2: Implementation PR`（需要一个已对齐且带有 `pr-ready` 标签的 Issue，以及已合并的 `docs/designs/<name>.md`）。
* **堆叠 PR 与标签（Stacks and Labels）**：在准备工作开始前，不必等待 `design-ready` 或 `pr-ready` 标签。PR 仅在合并时才需要对应标签。你可以先准备设计分支，再在其之上准备实现分支。经人类批准后，将其作为 GitHub 堆叠 PR（`gh stack submit` 或 `gh stack link`）提交，并将实现 PR 设为草稿（draft）。每个 PR 应只服务于单一目标，变更行数控制在约 200 至 400 行。底层 PR 使用 `Refs #<issue>`，顶层 PR 使用 `Closes #<issue>`。
  GitHub 堆叠 PR 需要上游仓库的分支权限。Fork 贡献者应在本地准备好堆叠分支，待设计文档 PR 合并后再开启实现 PR。
* **禁止人为拆分 PR（No Artificial PR Splitting）**：严禁为了规避 100 行范围阈值，将内聚的特性或重构人为拆分成多个较小的 Pull Request。
* **PR 描述模板（PR Description Template）**：每个 Pull Request 的描述都必须完整填写 [`.github/PULL_REQUEST_TEMPLATE.md`](../.github/PULL_REQUEST_TEMPLATE.md)。`gh pr create --body` 会跳过模板，因此请直接从该文件复制内容。由人类作者勾选 Accountability（问责确认）复选框。
* **PR 标题（PR Title）**：标题须以动作动词开头并采用句首大写格式（sentence case），例如 `Explain ignored agents folder warning in Claude Code`。不要使用 `type(scope):` 前缀。本规则优先于个人或全局智能体指令中的标题约定。
* **PR 标签（PR Labels）**：添加或建议适用的标签：`docs`、`site`、`ci` 或 `deps`。`design-ready` 与 `pr-ready` 由维护者控制，严禁自行添加这两个标签。
* **宿主环境验证证据（Harness Evidence）**：对于宿主环境支持相关的变更，请在 Verification（验证）章节中记录宿主名称与版本、安装结果，以及 `egm-brainstorm`、`egm-draft` 和 `egm-review` 的执行证据。未运行的检查项请填写 `not run`。
* **清晰的变更摘要（Clear Change Summary）**：在每个 Pull Request 中，明确列出所做的所有关键变更及其原因。对于特性提案，须在入库的 EGM 设计文档（`docs/designs/<name>.md`）中定义清晰的范围内目标（Goals）与范围外非目标（Non-Goals）。
