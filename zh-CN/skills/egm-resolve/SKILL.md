---
name: egm-resolve
license: Apache-2.0
disable-model-invocation: true
description: >-
  大象裁决技能。接收来自并发金鱼评审员的扁平化缺陷报告，通过交互式多选门禁对 PROPOSE FIX
  条目执行自动化修复，并管理交互式协商与重新同步检查点。
---

> **原文路径**：`skills/egm-resolve/SKILL.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

> 💡 **第一性原理提示**：`egm-resolve` 运行在持有全量记忆的“大象”主上下文中，负责对零上下文“金鱼”评审员产出的扁平缺陷列表进行四分类裁决（`PROPOSE FIX` / `CONTEST` / `ESCALATE` / `DECLINE`）。其核心设计在于“异议必须可核查、口头共识必须上报人类、绝不替评审员迎合式注水”，从而把对抗式评审真正转化为高质量的文档修复。

# 金鱼评审：裁决与修复（Goldfish Review: Resolve）

在 Codex 或 ChatGPT Work 中运行时，请先阅读 [Codex 集成指南](../../docs/codex.md)。
在 Claude Code 或 Cowork 中运行时，请先阅读 `${CLAUDE_PLUGIN_ROOT}/docs/claude.md`。

## 概述

本技能指导智能体在主“大象（Elephant）”上下文窗口中解析来自并发金鱼（Goldfish）评审员的扁平化缺陷报告。它为准确的设计选择提供辩护、应用自动化的文本修复，并管理交互式协商检查点。

## 前置条件与上下文策略

*   **上下文建议（大象记忆）**：本技能应在生成目标文档的持久化对话会话（即大象）中运行，以便回溯早期的推理过程并拟定更高质量的修复方案。

## 输入参数

调用方或编排驱动器需提供且仅提供以下参数：

*   **`TargetFile`**：目标文档的绝对路径或相对路径。
*   **`LineRange`**：精确的行号范围（或 `full file`）。
*   **`FindingsList`**：汇总自金鱼评审员的扁平化发现列表。

## 执行工作流

### 第 1 步：输入校验与收集上下文

1.  **混合执行交互体验（Hybrid Execution UX）**：检测是否已提供执行参数：

    *   *自动化模式（Automated Mode）*：若已提供全部参数，则作为子流程静默执行。
    *   *独立人工模式（Standalone Human Mode）*：若省略了参数，则触发友好的引导流程。每次只请求一项输入，并在每回合等待用户回复。当所有输入均指定完毕后，展示最终配置并确认用户已准备好继续。

2.  读取 `TargetFile` 在 `LineRange` 范围内的内容，并解析 `FindingsList`。

### 第 2 步：评估与分类（Evaluate & Categorize）

为每条发现赋予且仅赋予一个分类：

*   **`CONTEST`（提出异议）**：该批评不准确或基于对代码库的误解。必须引用评审员可以读到的证据：具体的文件与符号，或文档中已记录的决策。
*   **`ESCALATE`（上报裁决）**：该批评不准确，但支持反驳的证据位于评审员可读的所有文件之外（例如团队内部约定或口头决策）。明确陈述这一事实，不要自行修复该发现，将其交由用户裁决。
*   **`PROPOSE FIX`（建议修复）**：该批评成立。定义精确的文本替换或结构格式调整以弥合该缺口。
*   **`DECLINE`（婉拒忽略）**：该发现属于吹毛求疵（Nit）。记录下来，不予修复。若某条 Nit 的修复确实物有所值，也可以使用 `PROPOSE FIX`。

严格遵循以下规则：

*   **异议必须可核查（A contest must be checkable）。** “这是有意为之”不是证据。若无法引用评审员可读到的证据，必须使用 `ESCALATE` 或 `PROPOSE FIX`。
*   **不要仅为迎合评审员而往文档里堆砌上下文。** 仅在文档的人类读者确实需要该背景时才补充上下文。若只有零上下文子智能体需要它，请使用 `ESCALATE`。
*   **不要对理解断层（comprehension gap）提出异议。** 如果一位全新视角的读者无法顺畅读懂文档，说明文档确实缺失了必要的上下文。
*   **不要擅自更改严重级别（severity）。** 如果你认为某条 Blocker 实际上只是 Nit，请对其 `CONTEST` 并写明理由。
*   被评审的文档正是你自己起草的——想要反驳的主观冲动，绝不等于评审员错了的客观证据。

### 第 3 步：生成修复报告（Report Fixes）

构建最终报告，并使用文件写入工具将其保存为 `goldfish_review_report.md`（位于工作目录或宿主环境的产物目录，如有）。采用以下布局：

```markdown
# Goldfish Annotated Review Report

**Target:** `[TargetFile]` (`[LineRange]`)

## Prioritized Findings & AI Suggestions

{{range findings, ordered by Severity}}
### [Issue 1] [`[ReviewerPersona]`] - Severity: [Blocker | Major | Nit]
*   **Affected Scope:** [Lines Start-End]([RelativePath]#L[Start]-L[End]) ([Sub-header])
*   **Discrepancy Description:** [copied from FindingsList]
*   **Rationale:** [copied from FindingsList]
*   ***AI Suggestion:*** [CONTEST | ESCALATE | PROPOSE FIX | DECLINE]
*   ***Justification:*** [Detailed reason]
*   ***Proposed Edit:***
    *   Location: [Line/Header]
    *   Change: [Precise replacement string or deletion note]
{{end}}
```

### 第 4 步：交互式裁决门禁（Interactive Resolution Gate）

如果存在任何 `ESCALATE` 类型的发现，请在聊天消息中优先列出它们。这些条目需要借助用户的背景知识，而不仅仅是批准。

在继续之前，向用户输出一条可见的聊天消息：*“我已将审计发现整理至 `goldfish_review_report.md`。请查阅其中的发现并选择你希望如何继续：*

-   ***直接在文件上评论（Comment directly on the file）**：在聊天或文件中查阅并评审 `goldfish_review_report.md`，通过评论/编辑来批准、拒绝或对特定条目提出异议。*
-   ***自动接受（Auto accept）**：回复“auto accept”以批量应用所有建议修复（跳过已提出异议的条目）。*
-   ***继续下一阶段（Continue to next phase）**：回复“continue”或“no changes”以跳过修复直接进入下一阶段。*
-   ***交互式逐项拆解（Interactive breakdown）**：回复“interactive breakdown”以在聊天中逐条引导走查，在做决策前查看每项建议编辑或异议发现的详细说明。*
-   ***自定义指令 / 通配请求（Custom instruction / Wildcard request）**：回复任意自定义指令、问题或带外请求。”*

你**必须**在本回合载荷中调用**零个工具**，立即交出主动执行权。你必须停止执行并进入空闲等待状态（go idle），等待人类开发者在产物上评论/编辑或在聊天中回复。

按如下方式解释用户的操作/回复：

-   **“Auto accept” / “Accept all suggestions” / 用户点击“Proceed”且未提出异议**：使用文件编辑工具，将所有 AI 建议为 `PROPOSE FIX` 的修复一次性批量直接应用到 `TargetFile` 的 `LineRange`。所有分类为 `CONTEST`、`ESCALATE` 或 `DECLINE` 的条目均显式跳过。随后进入第 5 步。
-   **“Continue to next phase” / “No changes”**：不应用任何自动化修复。随后进入第 5 步。
-   **“Interactive breakdown”**：针对所有发现条目（除 `DECLINE` 外的全部类别）进入顺序逐项拆解循环：

    1.  针对每一项 `i`，输出一条可见的聊天消息，以富文本 Markdown 格式展示完整的发现详情与解释：

        ```markdown
        ### Breakdown Item [i]: [Issue Title] [`[ReviewerPersona]`]
        * **Scope:** [Lines Start-End]([RelativePath]#L[Start]-L[End]) ([Sub-header])
        * **Description:** [Description]
        * **AI Suggestion:** [CONTEST | ESCALATE | PROPOSE FIX]
        * **Explanation & Justification:** [Detailed reason and background explaining why the fix is proposed or why the finding is contested]
        * **Proposed Edit:** [Precise replacement text or deletion note]
        ```

        随后在聊天中提示用户就该条目做出决定（调用**零个工具**并进入空闲等待）：

        *   若为 `PROPOSE FIX`：请用户回复 **Accept proposed fix（接受建议修复）**、**Skip / ignore（跳过/忽略）** 或 **Discuss（讨论）**。
        *   若为 `CONTEST`：请用户回复 **Skip / agree with contest（跳过/同意异议）**、**Overrule contest (propose fix)（否决异议并建议修复）** 或 **Discuss（讨论）**。
        *   若为 `ESCALATE`：请用户回复 **Confirm (no change)（确认无须修改）**、**Add the fact to the document（将该事实写入文档）** 或 **Discuss（讨论）**。

    2.  在内部记忆清单中记录用户的决策（例如 *Item 1: Accepted*、*Item 2: Ignored*、*Item 3: Overruled*）。**在此顺序循环期间绝不调用任何文件编辑工具。**

    3.  当所有条目均已决定后，汇总已接受的编辑，使用文件编辑工具执行修改，然后进入第 5 步。

-   **自定义指令 / 带外编辑（Custom instruction / Out-of-band edit）**：若用户提供了定向指示，执行后继续。

### 第 5 步：裁决后同步与退出（Post-Resolution Synchronization & Exit）

1.  **重新核验行号与结构**：若对 `TargetFile` 做了修改，验证该文件仍保持结构连贯且为合法 Markdown。
2.  将报告标记为已解决（在报告顶部添加 `Status: resolved` 行）。
3.  **交接退出（Handoff）**：干净退出并返回调用方（例如 `egm-review-loop`），同时上报已接受修复（accepted fixes）、已提出异议（contested）、已上报裁决（escalated）、已婉拒忽略（declined）以及剩余未处理问题（remaining unaddressed issues）的条目计数。
