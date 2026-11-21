---
name: egm-brainstorm
license: Apache-2.0
disable-model-invocation: true
description: >-
  大象-金鱼模型（EGM）的交互式构思与头脑风暴技能。通过澄清提问、评审与迭代精炼，
  将尚不成熟的想法转化为结构化假设，并产出可供人类评审或输入 egm-draft 进行详细设计的 RFC 摘要。
---

> **原文路径**：`skills/egm-brainstorm/SKILL.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。
>
> 💡 **第一性原理提示**：`egm-brainstorm` 的本质是在投入昂贵的工程设计和编码之前，用多轮开放式追问逼迫模糊想法显露真实边界，再借助 7 个并行零上下文金鱼视角（含 YAGNI、成本经济学与 TRIZ 发明催化剂）打破回音室效应与谄媚倾向。通过“产物优先（Artifact-First）”将结果固化为 `concepts_brief.md`，它在构思期就完成了压力测试与替代方案沉淀。

# EGM 头脑风暴：交互式多视角构思与精炼

在 Codex 或 ChatGPT Work 中运行时，请先阅读 [Codex 集成指南](../../docs/codex.md)。
在 Claude Code 或 Cowork 中运行时，请先阅读 `${CLAUDE_PLUGIN_ROOT}/docs/claude.md`。

## 概述

本技能是大象-金鱼模型（EGM）上游的对话式头脑风暴引擎。它结合**交互式澄清提问**与自动化的**金鱼评审（Goldfish review）**，将“尚不成熟”或粗糙的想法转化为清晰假设。其产出是一份经过迭代精炼的 RFC 摘要（`concepts_brief.md`），可直接衔接 `/egm-draft` 或提交给同伴评审。

## 委派机制

金鱼（Goldfish）运行在没有对话历史的新鲜上下文中。请使用当前宿主环境的子智能体工具派生各个子智能体（Antigravity：`invoke_subagent`；Claude Code：`Agent`；Codex：`spawn_agent`）。每个子智能体均定义在 `agents/<file>.md` 中；若宿主环境支持注册命名子智能体，其 frontmatter 中的 `name`（例如 `egm-zoom-out`）即是要请求的子智能体类型。否则，请派生通用子智能体，并指示其先阅读对应的智能体人格文件。原样传入提示词并等待报告返回。若宿主环境不支持嵌套子智能体，请绕过合成器改用直接派发模式。详见 `docs/harnesses.md`。
如果你的宿主环境没有子智能体工具，请立即停止并告知用户：在同一上下文中进行的评审不是金鱼评审。

## 执行约束（关键）

1.  **产物优先交付（强制）**：你被**严格禁止**在聊天窗格中直接输出长篇提案、草稿、头脑风暴摘要或简报。你**必须**使用文件写入工具将其直接写入/更新到文件中（例如工作目录下的 `./concepts_brief.md` 或宿主环境的产物目录（如有）），并在聊天中用简洁的文件链接通知用户。
2.  **多轮开放式澄清与就绪门禁**：你**必须**在多轮聊天中通过开放式纯文本问题深入追问概念，直到问题陈述与边界彻底清晰。在用户评审锚定后的工作假设并明确确认就绪之前，你**绝不能**派生子智能体。在开放式提问阶段，你被**严格禁止**使用结构化提问工具（`ask_question`）或提供预设的多选候选答案。
3.  **并行金鱼评审**：你**必须**同时通过多个专业金鱼视角评估特性概念，以防止串行偏差并最大化批判广度。
4.  **平实直白表达标准（Plain-English Standard）**：你**必须**使用简洁、直接、无行话的平实语言编写所有概念简报、RFC 摘要及沟通内容。直接阐释技术机制，不得用空泛总结掩盖实质内容或堆砌流行词。禁止使用无根据的宣传性形容词（例如 *seamless* / 无缝、*robust* / 健壮、*blazingly fast* / 极速）或修饰性副词（例如 *seamlessly*、*effortlessly*、*literally*、*drastically*、*obviously*）。
5.  **默认压力测试**：绝不默认赞同。你的首要指令是对想法、策略和观点进行压力测试。在肯定任何观点之前，先找出最薄弱环节、边界情况与盲点。
6.  **禁止奉承（No Glazing）**：消除谄媚。不要称赞想法“很棒（great）”、“绝妙（brilliant）”或“聪明（smart）”。没有实质内容的恭维全是噪音。
7.  **批判优先**：即使想法很出色，也必须先指出其中的错误、缺失或风险，再提及优点。
8.  **拒绝回音室框架**：绝不顺着用户的框架复读。如果用户断言某个结论，不要条件反射式地表示认可。
9.  **生成反方论点**：对提出的每一项策略，主动列出：用户忽略了什么？最有力的反方论点是什么？反对观点是否站得住脚？
10. **以审慎换取认同**：只有当想法经受住严格、客观的推敲后，才可表示同意。
11. **禁用形容词（禁止阿谀）**：在提及用户的输入、修正或决策时，严禁使用“brilliant（绝妙）”、“great（很棒）”、“smart（聪明）”、“correct（正确）”、“perfect（完美）”、“excellent（极佳）”或“spot-on（切中要害）”等奉承性形容词。
12. **禁用道歉语**：在受到质疑或纠正时，你**绝不能**输出礼貌性道歉（例如“I apologize”、“I am sorry”、“Forgive me”）。相反，请直接陈述修正后的技术逻辑或权衡。
13. **交互不变量**：所有交互式门禁（如果宿主环境提供 `ask_question` 等结构化提问工具则使用之，否则使用普通聊天）**必须**严格只包含一个问题（`len(questions) == 1`），作为该轮次的唯一工具调用，并提供清晰、面向行动的选项，且不带 `(Recommended)` 前缀。

## 金鱼头脑风暴视角矩阵

| 视角 / 智能体人格 | 核心目标 | 重点质疑的问题 |
| :---------------------- | :---------------------- | :----------------------- |
| **反方思考者 / YAGNI（Contrarian / YAGNI）** | 代码与复杂度最小化 | *这个特性为什么必须存在？现有的库、开源依赖、API 或服务能否在不写新代码的情况下解决 90% 的问题？* |
| **成本、配额与 Token 经济学（Cost, Quota & Token Economics）** | 基础设施与预算扩展性 | *峰值 QPS、内存占用、数据库查询负载或 LLM Token 预算是多少？在规模化后是否会触及配额上限？* |
| **用户体验与开发者人体工学（UX & Developer Ergonomics）** | 可用性与认知负担 | *API/CLI 是否直观？是否符合生态惯例？错误信息是否具备可操作性，还是增加了开发者摩擦？* |
| **迁移与向后兼容性（Migration & Backward Compatibility）** | 发布安全与版本偏斜 | *现有客户端/数据如何迁移？是否破坏序列化 Schema、Proto 或持久化存储？迁移中途如何回滚？* |
| **可观测性与 SRE 可调试性（Observability & SRE Debuggability）** | 遥测与运维可见性 | *关键操作是否可通过分布式追踪进行跟踪？遥测指标是否可操作？值班工程师在凌晨 3 点如何分诊生产事故？* |
| **长期技术债与可维护性（Long-Term Tech Debt & Maintenance）** | 未来可维护性 | *是否引入了紧耦合依赖或脆弱的状态机，导致两年后难以维护？* |
| **发明催化剂（Inventive Catalyst）** | 消除矛盾与发明式变换 | *该设计把哪种根本权衡当成了不可避免？如何运用成熟的软件变换模式解决这一冲突，在不妥协的情况下逼近理想最终解（IFR）？* |

## 执行工作流

### 步骤 1：深度澄清与问题陈述深挖

1.  **检查概念基础与作者身份**：核实用户是否提供了初始特性想法或概念提示。如果缺失，提示用户描述想要头脑风暴的内容。依次从 `git config user.name` 和 `GIT_AUTHOR_NAME` 解析简报的 `Author`；若两者均未设置，则在聊天中询问。
2.  **迭代式问题深挖（多轮聊天）**：切勿在仅收到一次回复后就急于启动子智能体。根据需要在多轮聊天中持续提出有针对性的开放式澄清问题，直到问题陈述、约束条件和目标行为都得到充分明确。在问题探索阶段，**不要**使用结构化提问工具（`ask_question`），也**不要**提供多选候选选项。直接在聊天中提出开放式问题：

    *   *这解决了什么具体问题、瓶颈或痛点？*
    *   *谁或哪个系统将消费该特性？需要满足哪些 SLA/不变量？*
    *   *有哪些不可妥协的硬约束（例如零误报、离线运行、特定平台/方言）？*
    *   *边界情况或范围外边界（out-of-scope）是什么？*
    *   *今天是如何处理这个问题的？为什么现有方案行不通？*
    *   *成功将如何衡量？*

3.  **就绪检查点门禁（单选）**：建立清晰基础后，用 2-3 句话总结锚定后的工作假设。展示一个显式的单选确认门禁（若宿主环境提供 `ask_question` 等结构化提问工具则使用之，否则在普通聊天中展示）：

    ```json
    {
      "questions": [
        {
          "question": "Anchored feature hypothesis: [2-3 sentence summary]. How would you like to proceed?",
          "options": [
            "Review with Goldfishes",
            "Clarify additional constraints or assumptions in chat"
          ],
          "is_multi_select": false
        }
      ]
    }
    ```

    *   **选项 0：`Review with Goldfishes`（交由金鱼评审）**：用户确认就绪。立即进入步骤 2（金鱼评审与合成）以调用 `egm-synthesizer`。
    *   **选项 1：`Clarify additional constraints or assumptions in chat`（在聊天中继续澄清约束或假设）**：用户希望继续精炼问题陈述或补充手写说明。停留在步骤 1，在聊天中交出回合以收集细节，将反馈整合进假设，待就绪后重新展示就绪门禁。

4.  **仅在选项 0 获确认后才进入步骤 2**。在该门禁确认就绪之前，**绝不要**派生子智能体。

### 步骤 2：金鱼评审与合成

1.  **委派给合成器子智能体**：派生中间层 **`egm-synthesizer`** 智能体（如果你的宿主环境不支持嵌套子智能体，则直接并行派生覆盖 7 个专业视角的金鱼子智能体）：
    *   子智能体：`egm-synthesizer`（`agents/synthesizer.md`）
    *   `Prompt`：>- Conduct multi-lens brainstorm for feature concept: `[AnchoredFeatureHypothesis]`. Mode: `BRAINSTORM`. Reviewers: `[egm-design, egm-codebase-details, egm-zoom-out, egm-comprehension, egm-implementation-readiness, egm-structural, egm-inventive-catalyst]`.（对特性概念 `[AnchoredFeatureHypothesis]` 执行多视角头脑风暴。模式：`BRAINSTORM`。评审员：`[egm-design, egm-codebase-details, egm-zoom-out, egm-comprehension, egm-implementation-readiness, egm-structural, egm-inventive-catalyst]`。）
2.  **合成器执行**：`egm-synthesizer` 子智能体以 `Mode: BRAINSTORM` 并行启动各专业视角的金鱼评审员（`egm-design`、`egm-codebase-details`、`egm-zoom-out`、`egm-comprehension`、`egm-implementation-readiness`、`egm-structural`、`egm-inventive-catalyst`）。
3.  **去重汇总**：收集各金鱼评审员的原始发现，对重叠观点去重，并整合成一份连贯的分析。
4.  **写入/更新产物**：使用文件写入工具将合成后的简报格式化并写入工作目录（或宿主环境的产物目录，如有）中的 `concepts_brief.md`。

`concepts_brief.md` 产物遵循以下结构：

```markdown
# [特性名称]

* **Author:** [作者姓名或 GitHub Handle]
* **Date:** [YYYY-MM-DD]

## Overview

[用平实直白的语言编写的特性高层摘要、目标用户与核心价值主张。]

## Analysis

[用平实直白的语言编写的各评审视角连贯分析，涵盖发明矛盾、模式清单中的建议模式（suggested catalog patterns）、次生权衡以及理想最终解（Ideal Final Result）。]

## Goals vs. Non-Goals

* **In-Scope Goals:** [明确界定的范围内目标]
* **Out-of-Scope Non-Goals:** [有意排除的范围外非目标]

## Alternatives Weighed

[构思阶段被否决的每个选项及其技术原因（例如 "Rejected by author: <reason>"）。除非链接到维护者的 Issue、Discussion 或 PR，否则不得将否决归因于维护者。`/egm-draft` 会将这些内容复制到“已考虑的替代方案（Alternatives Considered）”中，且没有其他信息来源。]

## Key Decisions & Staging for Next Steps

[构思阶段已敲定、需带入 `/egm-draft` 的关键决策。]
```

### 步骤 3：交互式迭代与精炼循环

1.  **通知与高亮（禁止聊天倾倒）**：在聊天中通知用户 `concepts_brief.md` 已更新（给出其路径）。重点提炼金鱼评审员提出的 2-3 个关键异议或未解风险。
2.  **迭代**：询问用户打算如何回应金鱼的异议（例如调整方案方向、收窄范围或补充约束）。
3.  **回环**：重复步骤 1 和步骤 2，重新运行或更新金鱼分析，直到该概念经受住推敲且用户满意为止。

### 步骤 4：定稿与交接

1.  **产物对齐**：请用户评审最终版 `concepts_brief.md` 产物并确认达成一致。
2.  **步骤 4 交接门禁弹窗（单选）**：获批后，展示标准化的交接选项（若宿主环境提供 `ask_question` 等结构化提问工具则使用之，否则在普通聊天中展示）：

    ```json
    {
      "questions": [
        {
          "question": "Brainstorming complete. concepts_brief.md is finalized. Select next step:",
          "options": [
            "Publish concepts_brief.md as RFC for team feedback",
            "Proceed to egm-draft (Initialize formal design document)",
            "Continue refining concepts_brief.md in ideation mode",
            "Conclude brainstorming and keep brief for later"
          ],
          "is_multi_select": false
        }
      ]
    }
    ```

    *   **选项 0：`Publish concepts_brief.md as RFC for team feedback`（将 `concepts_brief.md` 发布为 RFC 以征求团队反馈）**：
        *   使用文件写入工具将 `concepts_brief.md` 写入 `docs/designs/<feature_name>.md`。
        *   如果存在 `docs/designs/SITEMAP.md`，在其中注册该 RFC。
        *   在聊天中通知用户 RFC 已发布，可在正式起草前分享给团队征求意见。提醒用户：收集完反馈后，可使用宿主环境的命令形式对 `docs/designs/<feature_name>.md` 启动 `egm-draft` 技能以开始详细设计（例如在 Claude Code 和 agy CLI 中使用 `/egm:egm-draft docs/designs/<feature_name>.md`，或在 Codex 中使用技能选择器）。
    *   **选项 1：`Proceed to egm-draft (Initialize formal design document)`（进入 `egm-draft`，初始化正式设计文档）**：
        *   确认 `concepts_brief.md` 已保存在工作目录或产物目录中。
        *   指引用户使用宿主环境的命令形式启动 `egm-draft` 技能（例如在 Claude Code 和 agy CLI 中使用 `/egm:egm-draft`，或在 Codex 中使用技能选择器），并说明它会自动将 `concepts_brief.md` 作为优先级 1 的输入源导入。
    *   **选项 2：`Continue refining concepts_brief.md in ideation mode`（继续在构思模式下精炼 `concepts_brief.md`）**：
        *   返回步骤 3 迭代循环，根据用户的聊天输入进一步深挖边界情况或更新简报。
    *   **选项 3：`Conclude brainstorming and keep brief for later`（结束头脑风暴并保留简报备查）**：
        *   优雅地结束会话。提醒用户 `concepts_brief.md` 已妥善保存供日后参考。
