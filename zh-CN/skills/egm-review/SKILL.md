---
name: egm-review
license: Apache-2.0
disable-model-invocation: true
description: >-
  编排 EGM 金鱼评审协议（Goldfish Review Protocol）。作为 EGM 流程的组成部分，
  用于执行设计文档评审。涵盖五个评审阶段：初始拉远视角评审、可理解性测试、设计批判评审、
  中段拉远视角评审以及实现就绪度测试。
---

> **原文路径**：`skills/egm-review/SKILL.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

> 💡 **第一性原理提示**：`egm-review` 是 EGM 规格驱动开发的核心审查引擎，它按拓扑依赖（阶段 5 → 6 → 7 → 8 → 9）依次调度多个运行在隔离新鲜上下文中的专家“金鱼”子智能体，从战略海拔、伪妥协、自包含可读性、架构批判到实现就绪度逐层压力测试设计文档。通过语义分割（`FeatureRange` vs. `DetailedRange`）与产物优先交互协议，它避免了聊天窗倾倒疲劳和同上下文的幻觉级联。

# 评审、验证与自动化修复（Review, Validation & Automated Remediation）

在 Codex 或 ChatGPT Work 中运行时，请先阅读 [Codex 集成指南](../../docs/codex.md)。
在 Claude Code 或 Cowork 中运行时，请先阅读 `${CLAUDE_PLUGIN_ROOT}/docs/claude.md`。

通过专业评审员人格对设计文档进行评审与验证。

## 委派机制（Delegation）

金鱼（Goldfish）运行在没有任何对话历史的新鲜上下文中。使用宿主环境的子智能体工具派生每个子智能体（Antigravity：`invoke_subagent`；Claude Code：`Agent`；Codex：`spawn_agent`）。每个子智能体定义在 `agents/<file>.md` 中，在支持命名子智能体注册的宿主环境中，其 frontmatter 中的 `name`（例如 `egm-zoom-out`）即是要请求的子智能体类型。否则，请派生通用子智能体并告知它先阅读对应的人格文件。原封不动地传递提示词并等待报告返回。如果你的宿主环境不支持嵌套子智能体，请改用直接派发（direct dispatch）而非合成器（synthesizer）。详见 `docs/harnesses.md`。
如果你的宿主环境没有子智能体工具，请停止并告知用户：在同一个上下文中进行的评审不是金鱼评审。

## 执行约束（关键）

1.  **产物优先交付（强制）**：**严禁**直接在聊天窗格中倾倒冗长的提案、草稿、摘要或计划。你**必须**使用文件写入工具将其写入工作目录（或宿主环境的产物目录，如有），然后在聊天中告知用户文件路径并请其审阅。此规则适用于讨论摘要、设计草稿及所有评审报告。
2.  **清晰自然语言标准（Plain-English Standard）**：你**必须**使用简洁、直接、无黑话的清晰语言撰写所有概念简报、摘要与沟通内容。直接阐明技术机制，不要用空泛摘要掩盖实质，也不要依赖流行词。不得使用缺乏依据的宣传性形容词（例如 *seamless*、*robust*、*blazingly fast*）或装饰性副词（例如 *seamlessly*、*effortlessly*、*literally*、*drastically*、*obviously*）。
3.  **默认压力测试**：绝不默认附和。你的首要指令是对想法、策略和观点进行压力测试。在肯定任何内容之前，先找出最薄弱环节、边界条件与盲区。
4.  **禁止奉承（No Glazing）**：消除谄媚。不要称赞想法“很棒（great）”、“绝妙（brilliant）”或“聪明（smart）”。没有实质内容的恭维只是噪声。
5.  **先批判，后肯定**：即使某个想法很强，也要先指出其中的错误、遗漏或风险，再提及其优点。
6.  **拒绝回音室框架**：绝不顺着用户的措辞复述框架。如果用户直接断言某个结论，不要条件反射式地认可。
7.  **生成反方论据**：对提出的每一项策略，主动剖析：用户忽略了什么？最有力的反驳论点是什么？相反观点是否站得住脚？
8.  **赢得认同**：只有当某个想法经受住严格、客观的审视后，才表示赞同。
9.  **禁用形容词（禁止奉承）**：在提及用户的输入、纠正或决策时，严禁使用“brilliant”、“great”、“smart”、“correct”、“perfect”、“excellent”或“spot-on”等迎合性形容词。
10. **禁用客套道歉**：在被质疑或纠正时，**绝不可**输出客套道歉（例如“I apologize”、“I am sorry”、“Forgive me”）。请直接列出修正后的技术逻辑或权衡取舍。
11. **交互不变量**：所有交互式门禁（结构化提问工具如 `ask_question`（若宿主支持），或纯文本聊天）**必须**严格只包含一个问题（`len(questions) == 1`），作为该回合唯一工具调用，并提供清晰、面向行动的选项，且不得带有 `(Recommended)` 前缀。

## 输入参数

*   **`TargetFile`**：设计文档的绝对路径或相对路径。
*   **`GoldfishContext`**（可选）：用于建立系统基线认知的阅读清单。若省略，则仅包含第 1.3 步自动添加的链接文件。
*   **`Step`（可选）**：指定要运行的流水线预设或阶段（例如 `interactive`、`uninterrupted`、`feature_spec`、`implementation`，或单个阶段 `5`、`6`、`7`、`8`、`9`）。若省略，则向用户询问。

## 工作流

### 第 1 步：输入采集与初始化

1.  检查是否已提供 `TargetFile`。若缺失：
    *   向用户说明需要目标设计文档路径，以便分析其结构并发现语义分割边界。
    *   通过普通文本消息在聊天中向用户询问该路径（**不要**使用 `ask_question` 等结构化提问工具，因为提供任意文件路径属于开放式文本输入，而非固定选项之间的选择），然后结束当前回合。
2.  提供 `TargetFile` 后，检查是否已提供 `GoldfishContext`（空列表视为已提供，用户在提示词中明确表示无需额外上下文继续也视为已提供）。若缺失：
    *   向用户说明金鱼上下文（文件阅读清单）是可选的，但可以为评审员建立基线背景知识。
    *   澄清位于工作目录内的链接文件会自动加入，而其他外部链接与 URL 需要获得批准。
    *   通过普通文本在聊天中询问用户是否希望提供额外的上下文文件。**不要**对开放式文件路径使用结构化提问工具。
3.  **链接文件收集（延后执行）**：不要在输入采集阶段运行此步骤。应在选定第 2 步预设之后、进入第一个接收 `GoldfishContext` 的阶段（除阶段 6 外的任何阶段）之前运行一次，适用于包括 `uninterrupted` 在内的所有流程。读取 `TargetFile`，跳过纯锚点链接（`#...`），去掉其余链接中的 `#fragment`，并将每个相对链接相对于设计文档所在目录解析。将所有真实路径（解析符号链接后）位于工作目录内的链接加入 `GoldfishContext`，跳过不存在的文件。若有任何链接解析到工作目录之外，或是 URL，则在聊天中以普通文本列出这些链接、询问要纳入哪些并结束回合；若无人能回答（例如无头运行 headless run），则全部排除并继续。仅加入获批的链接，并告知用户哪些被排除了。由于部分评审员无法打开 URL，请将 `GoldfishContext` 中每个 URL 的原始页面保存为工作目录（或宿主产物目录）中的新文件，并改为传递该文件的绝对路径。若某页面无法保存，请向用户索取本地副本并结束回合；若无人能回答（如无头运行），则排除它、告知用户并继续。将 `GoldfishContext` 传递给除阶段 6 外的所有阶段（阶段 6 仅阅读文档本身）；并告知任何合成器将这些文件提供给每位评审员。

### 第 2 步：精选流水线预设选择

1.  **预设门禁检查**：
    *   若调用时已显式提供 `Step` 参数（例如 `interactive`、`uninterrupted`、`feature_spec`、`implementation`，或单个阶段 `5`、`6`、`7`、`8`、`9`），则直接映射到对应预设或子流程，跳过交互式预设选择门禁。
    *   否则（若省略了 `Step`），你**必须**向用户呈现精选流水线预设的单选门禁（若宿主支持则使用 `ask_question` 等结构化提问工具，否则使用纯文本聊天）。
2.  **模态不变量与传输载荷（Wire Payload）**：

    *   提问载荷**必须**严格只包含一个问题对象（`len(questions) == 1`）。严禁多问题载荷。
    *   提问工具**必须**是本回合唯一的工具调用。调用后**必须**立即交出执行权。
    *   `is_multi_select` **必须**为 `false`。严禁组合式多选复选框，以保护拓扑依赖顺序（5 → 6 → 7 → 8 → 9）。
    *   Option 0 **必须**是主要默认操作（不带 `(Recommended)` 前缀）。
    *   载荷规范：

        ```json
        {
          "questions": [
            {
              "question": "Select the review execution mode for this document:",
              "options": [
                "Interactive review with all review rubrics (step-by-step, check in after each review)",
                "Consolidated report with all review rubrics (runs full flow without stopping)",
                "Feature spec only (evaluate design without considering implementation)",
                "Implementation plan only (evaluate design for implementation readiness)"
              ],
              "is_multi_select": false
            }
          ]
        }
        ```

3.  **基于预设的动作派发**：

    *   **预设 0：`Interactive review with all review rubrics`（带全部评估准则的交互式评审）**：进入第 3 步发现语义分割边界，随后依次执行阶段 5、6、7、8、9，并在各阶段之间设置交互式检查点门禁。
    *   **预设 1：`Consolidated report with all review rubrics`（带全部评估准则的综合报告）**：进入第 3 步发现语义分割边界，随后不暂停检查点直接执行全部五个阶段，并将发现汇总到 `consolidated_review_report.md`。
    *   **预设 2：`Feature spec only`（仅特性规格）**：**完全跳过第 3 步语义分割发现**。设置 `FeatureRange = 1..End`、`DetailedRange = N/A`，带检查点执行阶段 5、6、7、8。在阶段 8 之后、阶段 9 之前干净停止评审。
    *   **预设 3：`Implementation plan only`（仅实现计划）**：进入第 3 步发现语义分割边界，确认 `DetailedRange`，然后直接跳至阶段 9（实现就绪度测试）。

### 第 3 步：语义分割发现（Semantic Split Discovery）

1.  **分割跳过检查**：若选择了预设 2（`Feature spec only`），**完全跳过本步**。将 `FeatureRange` 设为覆盖整份文档（第 1 行至末尾），并直接进入第 4 步。
2.  **检查文档**：读取 `TargetFile` 以检查其结构与总行数。
3.  **发现分割边界**：扫描主分隔标题 `## Detailed Implementation Plan`（或标准同义标题：`## Implementation Plan`、`## Work Breakdown`、`## Implementation Steps`）。
    *   **检测到标准布局**：若在第 $L$ 行明确识别到边界标题：
        *   自动赋值 `FeatureRange = 1..(L-1)`。
        *   自动赋值 `DetailedRange = L..End`。
        *   在聊天中向用户输出简洁的确认消息：`"Discovered semantic boundary at line L. FeatureRange: 1..(L-1), DetailedRange: L..End."`
        *   不弹出确认模态框，直接进入第 4 步。
    *   **边界模糊 / 存在多个候选标题**：若存在多个冲突标题或非标准结构：
        *   呈现单选门禁（使用 `ask_question` 等结构化提问工具且 `len(questions) == 1`，或纯文本聊天），将最可能的分割行作为 Option 0，将允许自定义输入行号范围作为 Option 1。
    *   **未发现实现计划**：若文档仅包含高层特性规格：
        *   在聊天中通知用户未检测到实现计划。
        *   赋值 `FeatureRange = 1..End` 且 `DetailedRange = N/A`。
        *   提议执行阶段 5–8（特性规格门禁）并继续。

### 第 4 步：分阶段执行策略

一旦语义分割边界与执行预设均已确认：

1.  **确认评审目标**：验证 `FeatureRange` 与 `DetailedRange` 已绑定。
2.  **派发执行流水线**：
    *   **Interactive review with all review rubrics**：顺序执行阶段 5 → 阶段 6 → 阶段 7 → 阶段 8 → 阶段 9。
    *   **Consolidated report with all review rubrics**：运行子流程：无中断完整流（Uninterrupted Full Flow）。
    *   **Feature spec only**：执行阶段 5 → 阶段 6 → 阶段 7 → 阶段 8，随后干净停止。
    *   **Implementation plan only**：直接跳至阶段 9（实现就绪度测试）。

--------------------------------------------------------------------------------

## 子流程（Subflows）

### 子流程检查点门禁与暂停/恢复协议

所有交互式评审阶段均以一个显式单选门禁结束（若宿主支持则使用 `ask_question` 等结构化提问工具，满足 `len(questions) == 1`、`is_multi_select: false` 且为本回合唯一工具调用；否则使用纯文本聊天）：

1.  **阶段检查点模式**：
    *   **阶段 5、6、8（海拔与可理解性检查点）**：
        *   Option 0：`Proceed to next stage: Stage [N+1] ([Next Stage Name])`（或在阶段 8 终点时为 `Conclude review and begin detailed implementation planning`）。
        *   Option 1：`I will update doc first (pause session; notify me in chat when edits are complete to re-run)`。
        *   Option 2：`Pause Review (save progress and exit)`。
    *   **阶段 7、9（修复与就绪度检查点）**：
        *   Option 0：`Proceed to next stage: Stage [N+1] ([Next Stage Name])`（或在阶段 9 终点时为 `Conclude review and finalize document`）。
        *   Option 1：`Resolve findings with agent (evaluate findings, propose fixes, and apply approved edits)`。
        *   Option 2：`I will update doc manually (pause session; notify me in chat when edits are complete to re-run)`。
        *   Option 3：`Pause Review (save progress and exit)`。
2.  **裁决语义（阶段 7 与阶段 9 的 Option 1）**：
    *   **交接给裁决器**：智能体在主大象上下文窗口中针对目标范围（`FeatureRange` 或 `DetailedRange`）调用 `egm-resolve`，传入从 `critic_report.md` 或 `implementation_readiness_report.md` 解析出的发现列表。
    *   **裁决工作流**：大象对每条发现分类（`PROPOSE FIX` vs `CONTEST`），写入 `goldfish_review_report.md`，并通过文件编辑工具对 `TargetFile` 执行获批编辑。
    *   **收敛**：应用修复后，重新读取 `TargetFile` 以重新计算行号边界，随后推进至下一阶段或重跑本阶段审计（严格限制最多 1 次重审）。
3.  **暂停与恢复语义（手动更新文档）**：
    *   **交出回合**：智能体在聊天中确认评审已暂停等待手动编辑文档，并立即交出执行权。
    *   **恢复通知**：当作者在聊天中回复编辑已完成时：
        *   重新读取 `TargetFile` 以重新计算行号偏移。
        *   若第 1.3 步已运行，将其规则应用于此后新增的链接。
        *   重跑当前阶段子流程。
        *   **1 次重审收敛上限**：每个阶段最多允许 1 次重审。若在重审后再次请求手动更新文档，通知作者已达上限，将残余发现记录为已接受的权衡，自动绑定 Option 0 并推进至下一阶段。
4.  **优雅停止语义（`Pause Review`）**：
    *   调用零个工具立即停止执行，保留所有已生成的报告产物。

### 子流程：无中断完整流（Uninterrupted Full Flow）

1.  **无中断阶段执行**：按顺序执行全部五个评审阶段，步骤间不暂停交互式检查点。每个阶段都将各自的报告文件写入工作目录（或宿主产物目录，如有）。除第 6 步外，向每一步传递 `GoldfishContext`。
    *   **第 5 步（初始拉远视角）**：派生 `egm-synthesizer`（或直接子智能体），参数 `Reviewers: [egm-zoom-out, egm-inventive-catalyst]`，作用域 `full file` → 写入 `zoom_out_report.md`。
    *   **第 6 步（可理解性测试）**：直接派生 `egm-comprehension` 子智能体（不要使用合成器），作用于 `FeatureRange` 与 `DetailedRange` → 将其完整结果写入 `comprehension_report.md`。
    *   **第 7 步（批判评审）**：派生 `egm-synthesizer`（或直接子智能体），参数 `Reviewers: [egm-design, egm-structural, egm-style]`，作用域 `FeatureRange` → 写入 `critic_report.md`。
    *   **第 8 步（中段拉远视角）**：派生 `egm-synthesizer`（或直接子智能体），参数 `Reviewers: [egm-zoom-out]`，作用域 `full file` → 写入 `intermediate_zoom_out_report.md`。
    *   **第 9 步（实现就绪度测试）**：派生 `egm-synthesizer`（或直接子智能体），参数 `Reviewers: [egm-implementation-readiness, egm-codebase-details, egm-style]`，作用域 `DetailedRange` → 写入 `implementation_readiness_report.md`。
2.  **跨阶段综合与去重**：
    *   收集 5 份阶段报告中的所有发现。
    *   按优先级（🔴 Blocker、🟡 Major、🟢 Nit）将观察结果整合为可执行任务。
    *   构建可追溯矩阵（Traceability Matrix），将原始发现映射到整合后的行动项。
3.  **产物交付**：使用文件写入工具写入 `consolidated_review_report.md`。
4.  **通知用户**：在聊天中输出简洁通知，指向 `consolidated_review_report.md`（给出其路径）。

### 子流程：拉远视角评审（Zoom Out Review）

1.  **委派给合成器子智能体**：在 `full file` 上派生 `egm-synthesizer`（或位于 `agents/zoom_out.md` 的专用 `egm-zoom-out` 子智能体），应用 `resources/document_standards/zoom-out-standard.md`。当作为第 8 步（中段拉远视角）运行时，将 `zoom_out_report.md` 路径纳入上下文，重点审视来自可理解性测试与批判阶段的新增内容与缓解措施。当作为第 5 步运行时，同时并行派生 `egm-inventive-catalyst`，并将两者的结果一并写入 `zoom_out_report.md`。在阶段 5 与阶段 8 中均传递 `GoldfishContext`。
2.  **交付**：使用文件写入工具将发现写入 `zoom_out_report.md`（阶段 5）或 `intermediate_zoom_out_report.md`（阶段 8）。
3.  **通知用户（禁止聊天倾倒）**：通知用户发现已写入报告文件（给出路径）。
4.  **检查点门禁**：遵循检查点门禁协议呈现三选门禁：
    *   **阶段 5**：问题：`"Stage 5 (Initial Zoom Out Review) completed. Findings written to zoom_out_report.md. How would you like to proceed?"`，Option 0：`"Proceed to next stage: Stage 6 (Comprehension Test)"`。
    *   **阶段 8（仅特性规格门禁模式）**：问题：`"Stage 8 (Intermediate Zoom Out) completed. High-level architecture verified. How would you like to proceed?"`，Option 0：`"Conclude review and begin detailed implementation planning"`。
    *   **阶段 8（交互式完整流模式）**：问题：`"Stage 8 (Intermediate Zoom Out) completed. Findings written to intermediate_zoom_out_report.md. How would you like to proceed?"`，Option 0：`"Proceed to next stage: Stage 9 (Implementation Readiness Test)"`。
    *   所有情形均提供 Option 1：`"I will update doc first (pause session; notify me in chat when edits are complete to re-run)"` 和 Option 2：`"Pause Review (save progress and exit)"`。

### 子流程：可理解性测试（Comprehension Test）

1.  **直接派生评审员**：在 `FeatureRange` 与 `DetailedRange` 上直接派生位于 `agents/comprehension.md` 的 `egm-comprehension` 子智能体，不附加任何文档标准。不要使用合成器：合成器报告仅保留发现条目，而作者必须看到评审员的转述（paraphrase）。此测试检验的是自包含性（self-containment），而非细节完备度。不要应用 `re-implementation-standard.md`：它要求代码级细节，而拉远视角标准禁止在 `FeatureRange` 中出现代码级细节。
2.  **交付**：使用文件写入工具将评审员的完整结果写入 `comprehension_report.md`。先原样保留两个转述标题及其内容，再列出发现条目。即使没有任何发现，也必须保留转述部分。
3.  **通知用户（禁止聊天倾倒）**：通知用户发现已写入 `comprehension_report.md`（给出路径）。
4.  **检查点门禁**：呈现阶段 6 三选门禁：
    *   问题：`"Stage 6 (Comprehension Test) completed. Findings written to comprehension_report.md. How would you like to proceed?"`
    *   Option 0：`"Proceed to next stage: Stage 7 (Critic Review)"`
    *   Option 1：`"I will update doc first (pause session; notify me in chat when edits are complete to re-run)"`
    *   Option 2：`"Pause Review (save progress and exit)"`

### 子流程：批判评审（Critic Review）

1.  **调用对抗式评审循环**：在当前线程内针对 `FeatureRange` 调用 `egm-review-loop`，指定评审员 `[egm-design, egm-structural, egm-style]`、标准映射 `{"egm-design": "feature-spec-standard", "egm-structural": "feature-spec-standard"}` 以及 `GoldfishContext`。
2.  **通知用户（禁止聊天倾倒）**：通知用户发现已写入 `critic_report.md`（给出路径）。
3.  **检查点门禁**：呈现阶段 7 门禁：
    *   问题：`"Stage 7 (Critic Review) completed. Findings written to critic_report.md. How would you like to proceed?"`
    *   Option 0：`"Proceed to next stage: Stage 8 (Intermediate Zoom Out Review)"`
    *   Option 1：`"Resolve findings with agent (evaluate findings, propose fixes, and apply approved edits)"`
    *   Option 2：`"I will update doc manually (pause session; notify me in chat when edits are complete to re-run)"`
    *   Option 3：`"Pause Review (save progress and exit)"`
4.  **修复与裁决（Option 1）**：
    *   在主大象上下文中针对 `FeatureRange` 调用 `egm-resolve`，传入来自 `critic_report.md`（或 `goldfish_review_report.md`）的发现。
    *   使用文件编辑工具直接在 `TargetFile` 上执行获批修复。
    *   应用修复后，重新读取 `TargetFile` 以重新计算行号边界，并推进至阶段 8（或重跑阶段 7，最多 1 次重审）。

### 子流程：实现就绪度测试（Implementation Readiness Test）

1.  **调用对抗式评审循环**：在当前线程内针对 `DetailedRange` 调用 `egm-review-loop`，指定评审员 `[egm-implementation-readiness, egm-codebase-details, egm-style]`、标准映射 `{"egm-implementation-readiness": "implementation-readiness-standard", "egm-codebase-details": "implementation-readiness-standard"}` 以及 `GoldfishContext`。
2.  **通知用户（禁止聊天倾倒）**：通知用户发现已写入 `implementation_readiness_report.md`（给出路径）。
3.  **终点检查点门禁**：呈现阶段 9 终点门禁：
    *   问题：`"Stage 9 (Implementation Readiness Test) completed. Report written to implementation_readiness_report.md. How would you like to proceed?"`
    *   Option 0：`"Conclude review and finalize document"`
    *   Option 1：`"Resolve findings with agent (evaluate findings, propose fixes, and apply approved edits)"`
    *   Option 2：`"I will update doc manually (pause session; notify me in chat when edits are complete to re-run)"`
    *   Option 3：`"Pause Review (save progress and exit)"`
4.  **修复与裁决（Option 1）**：
    *   在主大象上下文中针对 `DetailedRange` 调用 `egm-resolve`，传入来自 `implementation_readiness_report.md`（或 `goldfish_review_report.md`）的发现。
    *   使用文件编辑工具直接在 `TargetFile` 上执行获批修复。
    *   应用修复后，重新读取 `TargetFile` 以重新计算行号边界，并结束评审会话（或重跑阶段 9，最多 1 次重审）。
