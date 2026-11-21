---
name: egm-draft
license: Apache-2.0
disable-model-invocation: true
description: >-
  基于 egm-brainstorm 产出的概念简报（或用户笔记）逐节起草设计文档。
  这是大象-金鱼模型（EGM）特性开发流程的第二步。在完成需求与方案对齐、
  准备编写设计文档时使用。
---

> **原文路径**：`skills/egm-draft/SKILL.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。
>
> 💡 **第一性原理提示**：`egm-draft` 是由高上下文“大象”主导的规格构建器：它强制逐节撰写、逐节过人类审批门禁，并坚守“作者决定架构”与“禁止编写业务代码”原则，防止 AI 擅自拍板架构权衡或陷入底层杂草。最后一步的“前置上下文对账（Reconcile）”而非自评自审，则严格捍卫了“起草者不能给自己打分”的大象-金鱼职责分离铁律。

# 设计起草（文档构建）

在 Codex 或 ChatGPT Work 中运行时，请先阅读 [Codex 集成指南](../../docs/codex.md)。
在 Claude Code 或 Cowork 中运行时，请先阅读 `${CLAUDE_PLUGIN_ROOT}/docs/claude.md`。

本技能引导用户基于头脑风暴阶段已对齐的问题与需求，起草一份技术设计文档。

## 执行约束（关键）

1.  你**必须**阅读并遵循 `resources/execution_instructions.md` 中的执行说明。
2.  **产物优先交付（强制）**：你被**严格禁止**在聊天窗格中直接输出长篇提案、草稿、摘要或计划。你**必须**使用文件写入工具将其直接写入工作目录（或宿主环境的产物目录，如有）（`design_draft.md` 或 `./design_draft.md`），并在聊天中告知用户文件路径、请其评审。该规则适用于讨论摘要、设计草稿以及所有评审报告。
3.  **平实直白表达标准（Plain-English Standard）**：你**必须**使用简洁、直接、无行话的平实语言编写所有概念简报、摘要及沟通内容。直接阐释技术机制，不得用空泛总结掩盖实质内容或堆砌流行词。禁止使用无根据的宣传性形容词（例如 *seamless* / 无缝、*robust* / 健壮、*blazingly fast* / 极速）或修饰性副词（例如 *seamlessly*、*effortlessly*、*literally*、*drastically*、*obviously*）。
4.  **默认压力测试**：绝不默认赞同。你的首要指令是对想法、策略和观点进行压力测试。在肯定任何观点之前，先找出最薄弱环节、边界情况与盲点。
5.  **禁止奉承（No Glazing）**：消除谄媚。不要称赞想法“很棒（great）”、“绝妙（brilliant）”或“聪明（smart）”。没有实质内容的恭维全是噪音。
6.  **批判优先**：即使想法很出色，也必须先指出其中的错误、缺失或风险，再提及优点。
7.  **拒绝回音室框架**：绝不顺着用户的框架复读。如果用户断言某个结论，不要条件反射式地表示认可。
8.  **生成反方论点**：对提出的每一项策略，主动列出：用户忽略了什么？最有力的反方论点是什么？反对观点是否站得住脚？
9.  **以审慎换取认同**：只有当想法经受住严格、客观的推敲后，才可表示同意。
10. **禁用形容词（禁止阿谀）**：在提及用户的输入、修正或决策时，严禁使用“brilliant（绝妙）”、“great（很棒）”、“smart（聪明）”、“correct（正确）”、“perfect（完美）”、“excellent（极佳）”或“spot-on（切中要害）”等奉承性形容词。
11. **禁用道歉语**：在受到质疑或纠正时，你**绝不能**输出礼貌性道歉（例如“I apologize”、“I am sorry”、“Forgive me”）。相反，请直接陈述修正后的技术逻辑或权衡。
12. **交互不变量**：所有交互式门禁（如果宿主环境提供 `ask_question` 等结构化提问工具则使用之，否则使用普通聊天）**必须**严格只包含一个问题（`len(questions) == 1`），作为该轮次的唯一工具调用，并提供清晰、面向行动的选项，且不带 `(Recommended)` 前缀。
13. **作者决定架构（Author Decides Architecture）**：当某项设计决策存在多个站得住脚的答案时（例如数据模型、组件边界、存储选型、协议），给出最多三个选项，分别说明其机制、权衡以及失效条件，并通过单选门禁请作者选择。不要对选项排序、不要标记默认项，也不要替作者做决定。如果作者让你代选（例如“你随便选一个吧”），重申各项权衡并再次呈现同一个门禁。仅当机械性选择只有唯一合理答案时（例如文件命名、排版格式），才可自行决定。

## 输入参数

*   **`Context File`**（可选）：前置文档的文件路径（例如 `docs/designs/<feature_name>.md`、`concepts_brief.md` 或 `discussion_summary.md`）。如果省略，本技能将默认自动读取前置上下文（例如 `concepts_brief.md` 或 `discussion_summary.md`）。

## 执行工作流

按顺序执行以下步骤。严格按序推进，但允许用户在遗漏内容时回退至先前阶段。在步骤之间，未经用户明确达成共识确认，不得跳步。随着对话在各章节间推进，迭代式地将内容写入文档。

### 章节审批门禁（步骤 2–5）

使用文件写入工具在 `design_draft.md` 中写入或更新每个章节后，必须在同一回合内严格按以下生成顺序呈现审批门禁：

1.  **可见聊天摘要与链接（第一步）**：你**必须**在提问门禁之前输出对话式 Markdown 文本。说明起草或更新了哪个章节，提炼 2–3 个关键决策或新增点，并提供指向 `design_draft.md` 的链接或路径（例如在 Codex 中使用 `[design draft](/absolute/project/design_draft.md)`）。
2.  **审批门禁（第二步）**：在同一回合内呈现单选审批选项（如果宿主环境提供 `ask_question` 等结构化提问工具，则使用 `len(questions) == 1` 且 `is_multi_select: false` 的调用；否则使用普通聊天）：

> [!CAUTION] **负向约束（禁止空聊天气泡）**：绝不要在响应文本为空的情况下发出结构化提问工具调用。在调用提问门禁之前，务必先生成可见的聊天说明与文件链接。

```json
{
  "questions": [
    {
      "question": "Drafted [Section Name] in design_draft.md. Please review. How would you like to proceed?",
      "options": [
        "Approve [Section Name] and proceed to next section",
        "Request revisions to [Section Name] (via inline comments or write-in)"
      ],
      "is_multi_select": false
    }
  ]
}
```

*   **选项 0（`Approve [Section Name] and proceed to next section` / 批准该章节并进入下一节）**：
    *   如果本回合上下文中同时包含针对 `design_draft.md` 的行内评论（inline comments），先将这些行内修改应用到 `design_draft.md`，然后再推进到下一章节。
    *   否则，立即开始起草下一章节。
*   **选项 1（`Request revisions to [Section Name] (via inline comments or write-in)` / 请求修改该章节）或带行内评论跳过弹窗**：
    *   **存在行内评论和/或手写修改指令时**：同时检查手写回复与留在 `design_draft.md` 上的行内评论（尊重保留高亮内容的正向标注，以及修改或删除内容的负向标注）。使用文件写入工具将所有修订应用到 `design_draft.md`，通知用户，并重新呈现审批门禁。
    *   **空手写输入守卫（关键）**：如果用户选择了该选项但既未提供手写文本**也**未在回合上下文中留下任何行内评论，**切勿**死循环或报错。请在聊天中提示：`"Please specify the adjustments or additions needed for [Section Name] (either via inline comments on design_draft.md or here in chat) so I can revise the draft."`（请说明对 [章节名] 需要做的调整或补充（可通过 `design_draft.md` 的行内评论或直接在此聊天中说明），以便我修订草稿。）并交出执行权，等待用户在聊天中澄清。

### 步骤 1：导入前置上下文（概念简报或笔记）

1.  **上下文文件**：如果指定了 `Context File`，直接读取并解析。
2.  **多态自动导入（Polymorphic Intake）**：如果未指定，按严格优先级顺序探测上下文来源：
    *   **优先级 1：现代构思简报（`concepts_brief.md`）**：检查 `./concepts_brief.md`、会话产物目录或 `scratch/concepts_brief.md` 中是否存在 `concepts_brief.md`。若找到，则导入并解析该简报。
    *   **优先级 2：讨论摘要（`discussion_summary.md`）**：若未找到 `concepts_brief.md`，则检查 `./discussion_summary.md`、会话产物目录或 `scratch/discussion_summary.md` 中是否存在 `discussion_summary.md`。若找到，则导入并解析该摘要。**切勿**提示用户启动 `/egm-brainstorm` 或发出弃用警告。
    *   **优先级 3：交互式聊天询问**：若上述两个文件均不存在，则在聊天中向用户询问该特性的问题陈述、目标和技术背景，或建议用户使用宿主环境的命令形式启动 `egm-brainstorm` 技能（例如在 Claude Code 和 agy CLI 中使用 `/egm:egm-brainstorm`，或在 Codex 中使用技能选择器）。
3.  **校验状态**：解析文档，向用户展示简短确认，以确保我们基于正确的上下文开始起草。
4.  **达成共识**：获取开始起草文档的明确批准。
5.  **解析作者**：若导入的简报中包含 `Author`，直接使用之。否则依次检查 `git config user.name` 和 `GIT_AUTHOR_NAME`。若两者均未设置，则在聊天中询问用户，并将其记录在文档头部。

### 步骤 2：起草概述、目标与问题陈述

1.  **回顾上下文**：利用导入的信息框定业务问题与用户痛点。
2.  **起草内容**：遵循设计文档模板（`references/design_doc.md`），用清晰平实的 Markdown 编写“问题陈述（The Problem）”、“概述（Overview）”、“目标（Goals）”和“非目标（Non-Goals）”。
3.  **聚焦重点**：强调 *为什么（why）*（用户需求与业务价值），而非 *怎么做（how）*（技术实现）。
4.  **写入草稿产物（强制）**：使用文件写入工具将这份初始草稿保存至工作目录（或宿主环境的产物目录，如有）中的 `design_draft.md`。
5.  **通知与校验（禁止聊天倾倒）**：你被**严格禁止**在聊天中打印草稿内容。相反，请在对话式聊天文本中通知用户“概述与问题陈述”已起草完毕，提供指向 `design_draft.md` 的可点击链接或路径，并呈现针对 `Overview & Goals` 的章节审批门禁。

### 步骤 3：起草技术方案

1.  **综合架构**：审视导入上下文中提出的高层架构与解决方案。
2.  **起草方案**：用清晰的散文描述拟议变更。指明新增组件、修改文件与 API 变更，并附上文本块图（例如 ASCII 或 Mermaid 图）以阐明数据流。
3.  **更新草稿产物（强制）**：使用文件写入工具更新 `design_draft.md`，追加/插入“技术方案（Technical Plan）”章节。
4.  **通知与校验（禁止聊天倾倒）**：你被**严格禁止**在聊天中打印技术方案。请在对话式聊天文本中通知用户“技术方案”已添加，提供指向 `design_draft.md` 的可点击链接或路径，并呈现针对 `Technical Plan` 的章节审批门禁。

### 步骤 4：起草已考虑的替代方案

1.  **核查替代方案**：从导入上下文的 `Alternatives Weighed` 章节中读取头脑风暴期间否决的替代方案。不要依赖记忆：起草阶段通常运行在新会话中。如果该章节缺失，询问用户否决了哪些替代方案及其原因。
2.  **起草内容**：用平实直白的 Markdown 编写“已考虑的替代方案（Alternatives Considered）”章节。
3.  **详述否决原因**：对每一个替代方案，明确说明*为何*被否决（例如延迟过高、扩展性受限、团队不熟悉）。将作者的决策记录为 `Rejected by author: [technical rationale]`（作者否决：[技术理由]）。除非直接引用上游维护者的 Issue、Discussion 或 PR URL，否则**切勿**写为 `Rejected by maintainer`。
4.  **更新草稿产物（强制）**：使用文件写入工具更新 `design_draft.md`，追加/插入“已考虑的替代方案（Alternatives Considered）”章节。
5.  **通知与校验（禁止聊天倾倒）**：你被**严格禁止**在聊天中打印替代方案内容。请在对话式聊天文本中通知用户“替代方案”章节已添加，提供指向 `design_draft.md` 的可点击链接或路径，并呈现针对 `Alternatives Considered` 的章节审批门禁。

### 步骤 5：起草详细实现计划

起草一份聚焦于“如何实现该特性”的详细实现计划。该计划必须足够详尽。你应当主动提问并补充足够细节，使得仅凭此计划即可完成实现，而无需在编码期做出任何有影响力的设计决策（零决策规则）。遵循以下准则：

1.  **单一实现清单**：在一个统一的章节中列出所有需要执行的事项。
2.  **穷举所有文件**：列出**每一个**将被修改、新建或删除的文件，包括所有需要更新的文档、Schema、配置、Proto 或测试文件。你**必须**涵盖计划触及的每一个文件引用。
3.  **变更理由**：每个文件都必须附带理由，说明*将修改什么*以及*为什么需要该修改*。
4.  **链接格式**：如已知则提供完整文件路径，或使用清晰的基名；对于包内及兄弟文档链接，使用相对路径（例如 `./sub/README.md` 或 `../sibling/README.md`）。
5.  **禁止编写实现代码（NO CODE）**：不要在这份设计草稿中生成实际实现代码。允许使用伪代码、API 定义或 Schema。
6.  **更新草稿产物（强制）**：使用文件写入工具更新 `design_draft.md`，追加/插入“详细实现计划（Detailed Implementation）”章节。
7.  **通知与校验（禁止聊天倾倒）**：你被**严格禁止**在聊天中打印实现计划。请在对话式聊天文本中通知用户“详细实现计划”已添加，提供指向 `design_draft.md` 的可点击链接或路径，并呈现针对 `Detailed Implementation` 的章节审批门禁。

### 步骤 6：设计文档定稿写入

当用户要求写入或保存设计文档时，严格遵循以下规则：

1.  **与前置上下文对账（Reconcile against the Predecessor Context）**：此处**不要**对照“拉远视角（Zoom Out）”标准给草稿打分。起草该草稿的会话天然认同它自己；`/egm-review` 阶段 5 会在新鲜上下文中运行那项检查。相反，重新阅读步骤 1 导入的上下文，并将其与当前草稿进行比对。没有任何金鱼能做这项检查，因为金鱼不会阅读前置简报。
    *   **矛盾（Contradiction）**：简报敲定了某个问题，但草稿做出了不同的决定。必须报告每一处矛盾。
    *   **新增（Addition）**：草稿决定了简报未曾提及的问题。仅当它改变了架构、外部接口或依赖项时才需报告。

    优先列出所有矛盾，每条占一行，同时引用简报原文与草稿原文。请用户逐一裁决。不要擅自修改草稿去迁就简报：草稿也可能是对的，因为起草阶段阅读了头脑风暴阶段未曾读过的代码。若没有任何偏离，用一句话说明即可。若步骤 1 未发现前置文件，跳过此步骤。

    如果存在未决矛盾，必须停下来等待用户裁决。在进入步骤 6.2 之前，将每一项裁决结果应用到 `design_draft.md`。

2.  **目标目录**：如果用户指定了目标目录或文件，直接使用。否则，探测工作区中既有的设计文档目录（依次检查 `Context File` 所在目录、`docs/designs/` 或 `<project>/docs/designs/`），绑定为 `[DiscoveredPath]`（默认为 `docs/designs/`），并呈现单选门禁（如果宿主环境提供 `ask_question` 等结构化提问工具，则使用 `len(questions) == 1` 且 `is_multi_select: false` 的调用；否则使用普通聊天）：

    ```json
    {
      "questions": [
        {
          "question": "Where should the design be saved?",
          "options": [
            "Save to detected directory: [DiscoveredPath]",
            "Specify custom target directory"
          ],
          "is_multi_select": false
        }
      ]
    }
    ```

    *   **选项 0（`Save to detected directory: [DiscoveredPath]` / 保存至探测到的目录）**：将 `<DESIGN_DIR>` 设为 `[DiscoveredPath]`。
    *   **选项 1（`Specify custom target directory` / 指定自定义目标目录）**：如果手写文本中提供了自定义路径，将 `<DESIGN_DIR>` 设为该路径。如果手写文本为空，则在聊天中提示：`"Please specify the custom target directory path where you would like to save the design document."`（请指定你希望保存设计文档的自定义目标目录路径。）并交出执行权。

3.  **更新站点地图（Sitemap）**：如果 `<DESIGN_DIR>/SITEMAP.md` 存在，在其中注册新设计文档（Link、Date、Author、Components、Description）。

4.  **写入文档**：定稿后，使用文件写入工具将 `design_draft.md` 的内容写入目标文件 `<DESIGN_DIR>/<FEATURE>.md`。如需调整最终结构，遵循 [设计文档模板](references/design_doc.md)。

5.  **整洁排版**：确保符合标准的 Markdown 格式、合法的表格语法以及正确闭合的代码块围栏。

6.  **交接**：提供指向定稿设计文档的可点击链接，并建议用户使用宿主环境的命令形式对 `<DESIGN_PATH>` 启动 `egm-review` 技能进行评审（例如在 Claude Code 和 agy CLI 中使用 `/egm:egm-review <DESIGN_PATH>`，或在 Codex 中使用技能选择器）。
