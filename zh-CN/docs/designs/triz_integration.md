<!--
Copyright 2026 Google LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

> **原文路径**：`docs/designs/triz_integration.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

> 💡 **第一性原理提示**：传统金鱼评审员都是“评估型（防守型）”角色——负责挑错与剪枝；引入 TRIZ 的 `egm-inventive-catalyst` 则是“发明型（进攻型）”角色，专门追问架构中被误认为不可避免的**伪妥协**。将其前置到 `/egm-brainstorm` 与 `/egm-review` 步骤 5（并采用建议性门禁+内联 10 大模式目录），既能尽早激发破局重构，又避免与步骤 8 拉远视角剪枝产生循环振荡，同时确保零外部 I/O 跨宿主确定性运行。

# EGM 插件中的 TRIZ 集成

*   **作者：** Brian Suchy ([@briansuchy](https://github.com/briansuchy))
*   **状态：** Draft（草案）
*   **日期：** 2026-11-03
*   **Issue：** [#37](https://github.com/google/elephant-goldfish-model/issues/37)

--------------------------------------------------------------------------------

## 概述

本文档规定了将 **TRIZ 方法**（发明问题解决理论，Theory of Inventive Problem Solving）集成到开源大象-金鱼模型 (EGM) 框架（`github.com/google/elephant-goldfish-model`）中的设计方案。

EGM 将高上下文对话式起草（**大象**）与独立的零上下文批评（**金鱼智能体人格**）分离开来。现有的金鱼智能体人格主要是**评估型**的（它们审计设计以识别缺陷、风险、认知摩擦与非必要复杂度）。

TRIZ 引入了一个**发明与变换型**维度。它不只是标记权衡或接受妥协（例如“更高可靠性必然需要更高延迟”），而是系统性地识别**技术矛盾（technical contradictions）**，并应用公认的发明原则（如反向作用 Inversion、分割 Segmentation 与自服务 Self-Service）来化解冲突，避免过早向伪妥协投降。

本次集成交付三项成果：

1.  一个全新的专用金鱼智能体人格：**“发明催化剂（The Inventive Catalyst）”**（`agents/inventive_catalyst.md`），内置一份自包含的内联 10 大软件工程模式目录，实现零 I/O 执行。
2.  将该智能体人格集成至 [skills/egm-brainstorm/SKILL.md](../../skills/egm-brainstorm/SKILL.md) 中的上游金鱼批驳蜂群。
3.  将该智能体人格集成至 [skills/egm-review/SKILL.md](../../skills/egm-review/SKILL.md) 的**步骤 5（初始战略评审）**，与 `egm-zoom-out` 配对，在进入细致评审前审计基础性权衡。

--------------------------------------------------------------------------------

## 问题陈述

软件系统设计常常受制于主观认定的权衡（如吞吐量 vs. 内存、灵活性 vs. 简洁性，或速度 vs. 正确性）。工程师与 AI 智能体往往将这些妥协当作不可避免的架构铁律而全盘接受。

现有的金鱼智能体人格依据特定领域标准审计设计：

*   `design_critic.md`：审计基础设施扩展性、瓶颈与成本。
*   `zoom_out.md`：强制执行简洁性并标记 YAGNI（You Aren't Gonna Need It）违规。
*   `codebase_details.md`：校验代码现实与文档漂移。
*   `comprehension.md`：强制执行零假设可读性。
*   `implementation_readiness.md`：验证无歧义的文件变更步骤。
*   `structural.md` 与 `style.md`：验证模板一致性与写作语气。

然而，这些智能体人格均未主动质疑架构妥协的*前提*，也没有提供化解该妥协的结构化模式。

此外，缺乏事实落地的发明式跳跃有生成不切实际、过度炫技抽象的风险。要让抽象的 TRIZ 原则（如“气压与液压结构”或“热膨胀”）在软件工程中真正有效，必须将其转化为成熟且可落地的软件架构模式（如事件流 Event Streams、弹性缓冲 Elastic Buffers 或控制反转 Inversion of Control），同时覆盖单进程工具与分布式系统。

--------------------------------------------------------------------------------

## 目标与非目标

### 目标

*   **专用发明智能体人格**：定义 `agents/inventive_catalyst.md`（`egm-inventive-catalyst`），用于识别技术矛盾并提出基于模式的解决方案，而不稀释现有评估型智能体人格的职责。
*   **双阶段集成**：
    *   **头脑风暴阶段**：将 `egm-inventive-catalyst` 加入 `/egm-brainstorm` 中由 `egm-synthesizer` 协调的并发批驳蜂群（7 视角蜂群），以拓展概念可能性。
    *   **评审流水线阶段**：将 `egm-inventive-catalyst` 加入 `/egm-review` 的**步骤 5（初始战略评审）**，在逐行评审前与 `egm-zoom-out` 共同审计高海拔基础妥协。
*   **自包含内联模式知识**：将 10 个精选软件架构模式直接内联进智能体人格提示词中，消除外部文件 I/O 以及跨宿主路径解析失败。
*   **宿主无关运行**：确保所有智能体定义与工作流使用标准子智能体工具约定，能够在 [`docs/harnesses.md`](../harnesses.md) 所列的各宿主环境中运行。

### 非目标

*   **修改现有智能体人格**：在现有金鱼智能体人格（`design_critic`、`zoom_out` 等）中修改或追加 TRIZ 逻辑不在范围内。它们聚焦的单一职责评估准则保持不变。
*   **宿主专用弹窗**：引入自定义交互式提问弹窗（如 `ask_question`）不在范围内。该智能体人格作为自动化批驳蜂群中的标准平级成员运行，遵循 `skills/egm-brainstorm/SKILL.md` 的纯文本交互约束。
*   **冗余目录文件**：单独编写 `resources/triz_patterns.md` 文档不在范围内。为避免双重维护债务与运行时漂移，模式目录严格驻留在智能体人格定义内部。
*   **步骤 7 阻塞式评审循环门禁**：将发明催化剂加入步骤 7 的阻塞式修复循环（`egm-review-loop`）不在范围内。它在上游步骤 5 作为建议性高海拔审计运行，避免与步骤 8 拉远视角产生循环冲突。

--------------------------------------------------------------------------------

## 提议架构

### 1. 智能体人格规格：`agents/inventive_catalyst.md`

新的智能体人格将作为符合标准金鱼宿主契约的独立 Markdown 文件添加到仓库的 `agents/` 目录中：

```markdown
---
name: egm-inventive-catalyst
description: Inventive problem-solving Goldfish persona applying TRIZ principles to identify and eliminate system-level contradictions without compromise.
license: Apache-2.0
---

# Core Mandates (Universal Goldfish Context)

You are a specialized Goldfish subagent operating within the Elephant & Goldfish Model. You have been delegated a specific task by the root agent (the Elephant).

1. **Strict Read-Only Protocol**: You are a strictly read-only agent. You have NO tools to modify the workspace, write files, or compile code. Do not attempt to fix or refactor any code.
2. **Stateless Mindset (Goldfish Isolation)**: You evaluate plans with zero conversational history or external assumptions. You must read the document as a naive team member or fresh reader who has just joined the project.
3. **No Chitchat**: Avoid conversational filler. Focus exclusively on technical rationale and findings. Do not narrate your tool usage.

---

# Role

You are **Inventive Catalyst Goldfish**.
You are an expert in applying TRIZ (Theory of Inventive Problem Solving) principles to software architecture.
Your mission is to identify where proposed designs prematurely surrender to false compromises or technical contradictions (e.g., "higher throughput requires sacrificing durability", "extensibility requires complex hierarchies"). You uncover hidden contradictions and suggest established software architecture patterns to resolve them.

Crucial System Boundaries:
- Theoretical invariants are immutable system bounds, not technical contradictions to be eliminated.
- Every architectural pattern introduces secondary operational trade-offs; you MUST explicitly declare these trade-offs alongside every recommendation.

---

# Curated Inventive Software Patterns

Use this inlined catalog as your foundational pattern reference:

1. **Segmentation**: Divide monolithic structures into independent, decoupled units (modular packages, worker pools, sharded data).
   * *Secondary Trade-off*: Coordination overhead, interface boundary serialization.
2. **Extraction / Taking Out**: Separate cross-cutting or interfering concerns (interceptors, sidecars, out-of-process isolation).
   * *Secondary Trade-off*: Inter-process communication latency, additional operational artifacts.
3. **Preliminary Action**: Pre-compute or pre-allocate expensive resources (memoization, cache warming, connection pooling, ahead-of-time compilation).
   * *Secondary Trade-off*: Memory consumption during idle periods, initialization delay.
4. **Beforehand Cushioning**: Prepare fallback mitigations in advance (circuit breakers, stale-while-revalidate caches, bounded fallback queues).
   * *Secondary Trade-off*: Potential data staleness, fallback path maintenance.
5. **Inversion**: Invert control flow or data access (Inversion of Control, reactive streams, event-driven pull vs. push).
   * *Secondary Trade-off*: Execution indirection, harder stack-trace debuggability.
6. **Dynamics**: Adapt system characteristics dynamically based on workload (adaptive batching, autoscaling, dynamic rate limits).
   * *Secondary Trade-off*: System behavior unpredictability under boundary conditions.
7. **Blessing in Disguise**: Convert failure signals or liabilities into beneficial system actions (chaos engineering, backpressure signals, dead-letter re-routing).
   * *Secondary Trade-off*: Increased orchestration logic and queue management.
8. **Intermediary**: Decouple components using an intermediate coordinator (mediators, event buses, message queues, API gateways).
   * *Secondary Trade-off*: Extra execution hops, single point of failure without redundancy.
9. **Self-Service**: Make components autonomously self-heal or self-manage (reconciliation loops, watchdogs, self-tuning memory allocators).
   * *Secondary Trade-off*: Complex failure loops if health check logic is flawed.
10. **Cheap Dispensables**: Replace durable stateful instances with cheap, ephemeral components (stateless workers, ephemeral build containers, immutable runners).
    * *Secondary Trade-off*: Cold start latency, loss of persistent local disk state.

---

# Instructions

Inspect the `Mode` parameter specified by the caller:

### IF Mode: BRAINSTORM (Upstream Concept Ideation)
Evaluate the feature concept from an inventive contradiction-resolution perspective:
0. **Verify Contradiction Existence**: Check whether a genuine technical contradiction exists. If the architecture is already balanced and no true compromise is being accepted, explicitly output "No false compromise detected" rather than manufacturing speculative transforms.
1. **Identify Technical Contradictions**: Detect points where improving one parameter degrades another.
2. **Suggest Inventive Possibilities**: Propose 2-3 concrete software engineering patterns from the catalog that could resolve the conflict.
3. **Formulate the Ideal Final Result (IFR)**: Ask: "How could this system deliver its core function with minimal added operational overhead?"

### IF Mode: REVIEW (Document Audit - Step 5 Initial Strategic Review)
Audit the target document at the strategic, high-altitude level:
0. **Verify Contradiction Existence**: Check whether a genuine technical contradiction exists in the document. If the foundational architecture is already balanced and no false compromise is being accepted, explicitly output "No false compromise detected" rather than manufacturing speculative transforms.
1. **Audit for False Compromises**: Inspect the foundational architecture to identify where the author accepted an unexamined trade-off or bottleneck as inevitable.
2. **Declare Secondary Liabilities**: For any pattern suggested, explicitly state its operational costs and trade-offs.

### Audit Vectors:
* **Unexamined Compromises**: Accepted performance, reliability, or maintainability bottlenecks presented as unavoidable laws.
* **Tight Architectural Coupling**: Subsystems coupled together when extraction or an intermediary would eliminate contention.
* **Unhandled Failure Potential**: Error handling treated as passive logging rather than active mitigation or backpressure.

---

# Output Format

Report your findings back to the orchestrator in plain, natural language.

#### IF Mode: BRAINSTORM
If no false compromise is detected, output: "No false compromise detected".
Otherwise, for each contradiction identified, include:
1. **Severity**: [Blocker | Major | Nit]
2. **Dimension**: Contradiction Elimination & Inventive Transforms
3. **Contradiction**: Parameter [X] versus Parameter [Y].
4. **Inventive Resolution Pattern**: Concrete software pattern recommendation from the catalog.
5. **Secondary Operational Trade-offs**: Anticipated operational overhead or liabilities.
6. **Ideal Final Result (IFR)**: What the self-sufficient architecture would look like.

#### IF Mode: REVIEW
If no false compromise is detected, output: "No false compromise detected".
Otherwise, for each finding, include:
1. **Severity**: [Major | Nit] (Advisory findings)
2. **Affected Scope**: Section header or architectural subsystem.
3. **False Compromise**: Description of the accepted trade-off in the text.
4. **Proposed Resolution**: Concrete architectural pattern to resolve the contradiction.
5. **Secondary Liabilities**: Operational overhead or complexity introduced by the pattern.
```

--------------------------------------------------------------------------------

### 2. 工作流与评审流水线集成

#### A. 上游头脑风暴集成（`/egm-brainstorm`）

1.  **技能表更新**：在 `skills/egm-brainstorm/SKILL.md` 的金鱼头脑风暴视角表中添加 `Inventive Catalyst`：

    | 视角 / 智能体人格 | 核心目标 | 重点质疑的核心问题 |
    | :------------- | :------------- | :----------------------- |
    | **7. 发明催化剂（Inventive Catalyst）** | 矛盾消除与发明变换 | *该设计把哪种根本权衡当成了不可避免的前提？我们如何利用经过验证的软件架构变换解决此冲突，从而在不做妥协的情况下达成理想最终解 (IFR)？* |

2.  **合成器分发更新**：在 `agents/synthesizer.md` 与 `skills/egm-brainstorm/SKILL.md` 中更新 `Mode: BRAINSTORM` 的子智能体分发列表：

    *   更新 `Reviewers` 以包含 `egm-inventive-catalyst`。

3.  **通用多视角合成**：`egm-synthesizer` 对称地聚合所有视角的发现至 `concepts_brief.md` 的 `## Analysis` 章节。将 `agents/synthesizer.md` 中的失败评审员规则从 `Mode: REVIEW` 扩展至 `Mode: BRAINSTORM`：超时或无返回的视角重试一次，若仍失败则在 `concepts_brief.md` 中报告为 `not run`，绝不将其报告为“无发现”。

#### B. 评审循环集成（`/egm-review`）

1.  **步骤 5（初始战略评审）配对**：在 `skills/egm-review/SKILL.md` 中，更新 `Subflow: Zoom Out Review`（步骤 5）以派生**两位并发高海拔审计员**：
    *   `egm-zoom-out`：依据 `zoom-out-standard.md` 审计 YAGNI、过度工程与根因对齐。
    *   `egm-inventive-catalyst`：不依赖外部文档标准（而是依赖其自包含内联目录，类似于 `egm-comprehension`），针对 `full file` 审计基础性伪妥协与发明机会。
2.  **建议性检查点门禁**：步骤 5 是一个交互式人工检查点门禁。来自发明催化剂的发现会记录在 `zoom_out_report.md` 的 **Inventive Opportunities（发明机会）**章节下。作者可选择在继续前尽早调整架构方向。在 `Subflow: Uninterrupted Full Flow` 中，`zoom_out_report.md` 中的这些建议性机会被排除在强制缺陷修复追踪矩阵之外。
3.  **单向评审流水线**：由于基础性妥协已在步骤 5 完成评估：
    *   步骤 6（`egm-comprehension`）审计文档清晰度。
    *   步骤 7（`egm-design`、`egm-structural`、`egm-style`）审计实现机制，不会产生循环拉扯。
    *   步骤 8（`egm-zoom-out`）在步骤 9 就绪测试前充当最终剪枝剪刀，在已化解权衡的语境下评估被采纳的发明解决方案，确保合理的冲突化解不会被误判为过早引入的复杂度。

--------------------------------------------------------------------------------

## 风险与局限

### 1. 在硬理论不变量上产生矛盾幻觉

*   **风险**：发明型智能体可能将根本的数学、物理或理论极限误判为可化解的妥协，从而生成不可行的提案。
*   **缓解措施**：智能体人格提示词明确划定了矛盾分析边界：理论不变量是不可变系统边界，而非待消除的技术矛盾。

### 2. 次生运维负担与 YAGNI 膨胀

*   **风险**：为解决轻微摩擦而套用架构模式（如事件总线、Sidecar、缓存层）会引入运维与维护开销。
*   **缓解措施**：
    1.  发明催化剂必须为每个模式显式输出 `Secondary Liabilities`（次生负担）。
    2.  在 `/egm-review` 中，步骤 5 的发现严格为建议性，且步骤 8（中间拉远视角）会在实现规划前剪除任何推测性复杂度。

### 3. 蜂群执行韧性

*   **风险**：向并发批驳蜂群增加第 7 个子智能体会提高请求并发度，在延迟敏感环境中增加超时失败风险。
*   **缓解措施**：第 3 项将 `agents/synthesizer.md` 中的失败评审员规则从 `Mode: REVIEW` 扩展至 `Mode: BRAINSTORM`：超时或无返回的视角重试一次，若仍失败则在 `concepts_brief.md` 中报告为 `not run`，绝不将其报告为“无发现”。

--------------------------------------------------------------------------------

## 曾考虑的替代方案

### 替代方案 A：用 TRIZ 逻辑增强现有视角

*   **描述**：将 TRIZ 提示词注入 `design_critic.md` 或 `zoom_out.md`，而非创建新的智能体人格。
*   **优点**：避免向 `agents/` 目录增加新的智能体人格文件。
*   **缺点**：稀释现有智能体人格的核心使命。`design_critic` 专为怀疑与悲观审视而设计；强行要求它同时充当发明型头脑风暴者会制造指令冲突并降低评审发现质量。
*   **驳回理由**：EGM 依赖隔离的、单一职责的批评家。专用智能体人格可使评估思维与发明思维保持清晰分离。

### 替代方案 B：通过自定义弹窗工具提供交互式多选门禁

*   **描述**：在 `/egm-brainstorm` 中添加一个交互式多选 UI 对话框（如 `ask_question`），允许用户在每次批驳前手动开关单个金鱼智能体人格。
*   **优点**：允许细粒度的按会话自定义。
*   **缺点**：违反 `skills/egm-brainstorm/SKILL.md` 约束 2（构思阶段不使用结构化提问工具），依赖仅部分宿主环境具备的结构化提问工具，并增加用户交互摩擦。
*   **驳回理由**：自动化并发蜂群无需手动切换即可干净地完成评审。

### 替代方案 C：合成器内置提示词（零派生方案）

*   **描述**：指示 `egm-synthesizer` 在合成阶段直接执行 TRIZ 分析，而非启动第 7 个子智能体。
*   **优点**：节省 Token 消耗并消除一个子智能体的冷启动延迟。
*   **缺点**：违反 EGM 零上下文金鱼隔离的核心不变量。合成器的职责是客观聚合；强迫其生成发明模式会污染其合成偏见，并稀释独立的批判性发现。
*   **驳回理由**：独立的金鱼隔离能产出更高质量、未受污染的批评。

### 替代方案 D：提示词内联模式知识 vs. 外部参考文件

*   **描述**：单独编写一份 `resources/triz_patterns.md` 文档供智能体在运行时读取。
*   **优点**：使智能体人格提示词更简短。
*   **缺点**：引入文件 I/O 延迟、在文档与提示词之间造成双重维护漂移，并在客户端仓库的各类宿主环境中引发 `FileNotFound` 错误。
*   **驳回理由**：将 10 个核心模式摘要内联至系统提示词可保证在所有宿主环境中零 I/O、确定性执行，且不留死产物。

### 替代方案 E：步骤 5 战略审计 vs. 步骤 7 阻塞式评审循环

*   **描述**：将 `egm-inventive-catalyst` 加入步骤 7（`egm-review-loop`）的阻塞式修复循环。
*   **优点**：与设计批评家一同在 `FeatureRange` 上运行。
*   **缺点**：会与步骤 8（`egm-zoom-out`）产生严重的循环拉扯，后者会立刻剪除步骤 7 新增的复杂度。此外，映射到 `feature-spec-standard.md` 会制造虚构契约。
*   **驳回理由**：将发明催化剂置于步骤 5 可在逐行评审前提供最早的战略权衡审计，构建无阻塞循环的单向流水线。

--------------------------------------------------------------------------------

## 详细实现计划

实现划分为四个顺序阶段，严格执行零决策规则（Zero-Decision Rule），并在每个阶段建立确定性验证检查点。

### 阶段 1：智能体人格创建与清单注册

#### 1. `agents/inventive_catalyst.md`（新建）

*   **类型 / 范围**：智能体人格规格
*   **理由**：为 `egm-inventive-catalyst` 定义系统提示词、核心指令、角色、内联模式目录及输出格式。
*   **拟议变更**：严格按照“提议架构”第 1 节规定的 Schema 编写该文件。

#### 2. 宿主包（修改）

*   **类型 / 范围**：插件清单与宿主适配器
*   **理由**：每个分发评审员的宿主都需要包含该新智能体人格。
*   **拟议变更**：在 `.claude-plugin/plugin.json` 的 `agents` 数组中添加 `"./.claude-plugin/agents/inventive_catalyst.md"`。将该智能体人格添加到 `.claude-plugin/agents/`、`.cursor-plugin/agents/`、`com.github.copilot/agents/egm-inventive-catalyst.agent.md`、`adapters/opencode/install.py` 中的评审员列表，以及 `scripts/pack_plugin.py` 中的智能体人格列表。将 `docs/harnesses.md` 中的评审员与适配器数量从 9 更新为 10。

*阶段 1 检查点*：使用 `pytest tests/unit/test_frontmatter.py` 校验所有智能体文件的 YAML frontmatter。

--------------------------------------------------------------------------------

### 阶段 2：工作流技能编排

#### 3. `agents/synthesizer.md`（修改）

*   **类型 / 范围**：中间编排器与聚合器
*   **理由**：指示 `egm-synthesizer` 在头脑风暴期间启动 `inventive_catalyst` 并聚合其发现。
*   **拟议变更**：
    *   在 `IF Mode: BRAINSTORM` 步骤 1 中：更新子智能体分发列表以并发启动 7 视角蜂群，加入 `inventive_catalyst`，角色头衔设为 `Inventive Catalyst Brainstormer`。
    *   在 `IF Mode: BRAINSTORM` 步骤 1 中：引入 `Mode: REVIEW` 步骤 1 的 `Failed Reviewers` 规则：对失败视角重试一次，随后在 `concepts_brief.md` 中报告为 `not run`，绝不报告为零发现。
    *   在 `IF Mode: BRAINSTORM` 步骤 2 中：更新聚合指令：*"Collect findings from all 7 Goldfish, deduplicate overlapping points, synthesize inventive contradictions and pattern resolutions alongside risk dimensions, and compile them into `concepts_brief.md`."*（收集全部 7 位金鱼的发现，去重重叠要点，将发明矛盾与模式解决方案连同风险维度一同合成，并编译写入 `concepts_brief.md`。）
    *   在 `egm-review` 步骤 5 中：`egm-inventive-catalyst` 与 `egm-zoom-out` 并行分发；`## Inventive Opportunities` 下的发现直接记录到 `zoom_out_report.md`，不改动 `goldfish_review_report.md` 严格的 Blocker/Major/Nit 缺陷 Schema。

#### 4. `skills/egm-brainstorm/SKILL.md`（修改）

*   **类型 / 范围**：核心技能工作流
*   **理由**：在头脑风暴批驳蜂群中注册 `egm-inventive-catalyst`。
*   **拟议变更**：
    *   向视角表中添加第 7 个视角：`| **7. Inventive Catalyst** | Contradiction elimination & inventive transforms | *What fundamental trade-off is this design accepting as inevitable? How can we resolve this conflict using proven software transforms to achieve the Ideal Final Result without compromise?* |`
    *   更新步骤 2.1 引导语，在 `Reviewers` 数组参数中包含 `egm-inventive-catalyst`：`Reviewers: [egm-design, egm-codebase-details, egm-zoom-out, egm-comprehension, egm-implementation-readiness, egm-structural, egm-inventive-catalyst]`。
    *   在嵌入的 `concepts_brief.md` 模板中，扩展 `## Analysis` 占位符，涵盖发明矛盾、针对每项矛盾建议的目录模式、其次生权衡，以及理想最终解 (IFR)。

#### 5. `skills/egm-review/SKILL.md`（修改）

*   **类型 / 范围**：评审编排器
*   **理由**：将 `egm-inventive-catalyst` 集成至步骤 5（初始战略评审）而不与步骤 8 批评后剪枝冲突，同时严格控制在 `tests/unit/test_skill_load_limits.py` 强制执行的 24,000 字节技能加载上限以内。
*   **技能体积预算**：`egm-review` 正文在 `main` 分支上为 23,743 字节，仅余 257 字节余量。为在不超限的情况下容纳步骤 5：
    1.  Inventive Opportunities 的详细报告 Schema 严格定义在 `agents/inventive_catalyst.md` 的 `# Output Format` 内部，避免在 `SKILL.md` 中重复出现多行 Schema 块。
    2.  `Subflow: Zoom Out Review` 保持共享。仅在其步骤 5 分支以及 `Subflow: Uninterrupted Full Flow` 的步骤 5 条目中添加 `egm-inventive-catalyst`。这使正文达到约 23,885 字节，无需删减其他内容。
*   **拟议变更**：
    *   在 `Subflow: Zoom Out Review` 步骤 1 的步骤 8 子句之后补充：“When running as Step 5, also spawn `egm-inventive-catalyst` in parallel and write both into `zoom_out_report.md`.”（当作为步骤 5 运行时，同时并发派生 `egm-inventive-catalyst` 并将两者结果写入 `zoom_out_report.md`。）步骤 8 仍仅运行 `egm-zoom-out`。
    *   在 `Subflow: Uninterrupted Full Flow`（由 `Consolidated report` 预设运行）的步骤 5 条目中，将 `Reviewers: [egm-zoom-out]` 改为 `Reviewers: [egm-zoom-out, egm-inventive-catalyst]`。

*阶段 2 检查点*：通过 `pytest tests/unit/test_frontmatter.py` 校验技能 frontmatter 与子流程解析。

--------------------------------------------------------------------------------

### 阶段 3：全局文档与视角数量同步

#### 6. `docs/designs/review.md`（修改）

*   **类型 / 范围**：架构规格
*   **理由**：使评审流水线规格与步骤 5 双审计员集成保持一致。
*   **拟议变更**：
    *   将 Mermaid 流程图节点 `Step5` 更新为 `Step 5: Initial Strategic Review (Zoom Out + Inventive Catalyst)`。
    *   更新步骤 5 阶段拆解章节，记录 `zoom_out` 与 `inventive_catalyst` 的并发执行。
    *   将 `agents/inventive_catalyst.md` 添加至评审智能体人格目录表。

#### 7. 设计系统文档（修改）

*   **类型 / 范围**：架构规格
*   **理由**：跨设计规格同步组件清单与视角数量。
*   **拟议变更**：
    *   `docs/designs/architecture.md`：将 6 视角批驳蜂群引用更新为 7 视角，并在智能体人格层列表中添加 `inventive_catalyst`。
    *   `docs/designs/brainstorm.md`：将 Mermaid 图表与视角描述从 6 个视角更新为 7 个视角，并在文件映射表中将 `inventive_catalyst.md` 映射至发明催化剂视角。
    *   `docs/designs/synthesizer.md`：将 6 视角蜂群章节更新为 7 视角，并在头脑风暴视角列表中添加 `inventive_catalyst`。
    *   `docs/designs/README.md`：更新现有的 `triz_integration.md` 条目，移除对被驳回的“动态评审员选择”的引用，以反映向 `/egm-brainstorm` 与 `/egm-review` 步骤 5 的集成。

#### 8. 面向用户的文档（修改）

*   **类型 / 范围**：文档
*   **理由**：确保入门指南反映多视角头脑风暴，而不硬编码脆弱的视角数量。
*   **拟议变更**：
    *   `docs/getting_started.md`：将头脑风暴概述中的“6-lens Goldfish”措辞更改为“multi-lens Goldfish”。

*阶段 3 检查点*：使用 `pytest tests/unit/test_link_integrity.py` 校验所有文档文件的链接完整性。

--------------------------------------------------------------------------------

### 阶段 4：测试套件更新与行为验证

#### 9. 测试规格与行为场景（修改）

*   **类型 / 范围**：测试套件与评估场景
*   **理由**：确保对所有宿主环境中的 7 视角蜂群与步骤 5 发明催化剂集成提供自动化测试覆盖。
*   **拟议变更**：
    *   `skills/egm-brainstorm/TEST.md`：更新测试预期与视角枚举以包含发明催化剂（7 个视角）。
    *   `skills/egm-review/TEST.md`：更新步骤 5 测试预期，验证并发执行拉远视角与发明催化剂的初始战略评审。
    *   `skills/egm-brainstorm/test_scenarios.yaml`：添加行为测试场景，断言 `## Analysis` 章节指出了一项发明矛盾、针对该矛盾建议的目录模式及其次生权衡，并添加一个场景验证输入已最优时的“No false compromise detected”提前退出护栏。
    *   `skills/egm-review/test_scenarios.yaml`：添加场景验证步骤 5 对伪架构妥协的检测以及提前退出护栏。
    *   `tests/unit/test_frontmatter.py`：添加校验断言，确保 `agents/` 下所有 Markdown 文件均符合 frontmatter 规范。
    *   `adapters/opencode/install.py`：将 `agents["egm"]["prompt"]` 引导字符串从“complete all six brainstorm reviewers”更新为“all seven”。
    *   `tests/unit/test_opencode_adapter.py`：将智能体数量断言从 8 更新为 9（`assert len(config["agent"]) == 9`）。
    *   `tests/unit/test_harnesses.py`：将 `"inventive_catalyst"` 添加至中央 `REVIEWERS` 元组。
    *   `scripts/pack_plugin.py`：将 `agents/inventive_catalyst.md` 添加至 `REQUIRED_SHARED` 列表，确保 Claude 分发打包通过 `tests/unit/test_packaging.py`。

*阶段 4 检查点*：运行单元测试套件。`test_skill_load_limits.py` 与 `test_scenario_coverage.py` 覆盖技能变更：

```bash
pytest tests/unit
```

--------------------------------------------------------------------------------

### 验证计划

*   **Markdown 格式化与静态检查**：对所有新建和修改的文档文件运行 `mdformat`：

    ```bash
    mdformat agents/ skills/ docs/ README.md
    ```

*   **静态测试套件执行**：从仓库根目录运行测试套件：

    ```bash
    pytest tests/unit
    ```
