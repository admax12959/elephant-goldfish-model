---
name: egm-review-loop
license: Apache-2.0
disable-model-invocation: true
description: >-
  通过自动化多智能体评审循环执行大象-金鱼模型（EGM）中的对抗式评审流程，对照定性标准与结构模板审计文档，
  并自动应用获批修复。适用于评审设计文档、README、特性规格、实现计划，或验证文档完备性与语气。
---

> **原文路径**：`skills/egm-review-loop/SKILL.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

> 💡 **第一性原理提示**：`egm-review-loop` 是驱动“金鱼并发审查 → 大象裁决修复 → 再次审查”多轮打磨循环的编排引擎。由于每次新鲜上下文金鱼的采样视角存在随机性，它不追求不切实际的“单轮零发现”，而是通过“已裁决（Settled）不重开、净轮次（Clean Round）连续两轮才算收敛（Converged）、检测修复振荡（Churn）与 5 轮上限”的确定性状态机，把多轮 AI 评审拉回可控轨道。

# 金鱼评审：循环编排驱动器（Goldfish Review: Loop）

在 Codex 或 ChatGPT Work 中运行时，请先阅读 [Codex 集成指南](../../docs/codex.md)。
在 Claude Code 或 Cowork 中运行时，请先阅读 `${CLAUDE_PLUGIN_ROOT}/docs/claude.md`。

## 概述

本技能充当驱动多智能体评审循环的中央编排引擎，协调两种截然不同的智能体记忆模型：

*   **金鱼（对抗式审计员，Adversarial Auditors）**：派生在隔离、新鲜上下文窗口中的子智能体，用于在不带历史偏见的情况下捕获未言明的假设。
*   **大象（持久化裁决者，Persistent Resolver）**：持有项目背景知识的主智能体上下文，负责评估发现并应用获批修复。

该驱动器管理参数采集、解析评审员与文档标准路径、在每轮迭代前无条件重新解析目标行号范围、按配置派生并发的专业金鱼审计员，并与大象上下文协同完成问题裁决。

## 委派机制（Delegation）

金鱼（Goldfish）运行在没有任何对话历史的新鲜上下文中。使用宿主环境的子智能体工具派生每个子智能体（Antigravity：`invoke_subagent`；Claude Code：`Agent`；Codex：`spawn_agent`）。每个子智能体定义在 `agents/<file>.md` 中，在支持命名子智能体注册的宿主环境中，其 frontmatter 中的 `name`（例如 `egm-zoom-out`）即是要请求的子智能体类型。否则，请派生通用子智能体并告知它先阅读对应的人格文件。原封不动地传递提示词并等待报告返回。如果你的宿主环境不支持嵌套子智能体，请改用直接派发（direct dispatch）而非合成器（synthesizer）。详见 `docs/harnesses.md`。
如果你的宿主环境没有子智能体工具，请停止并告知用户：在同一个上下文中进行的评审不是金鱼评审。

## 输入参数

调用方或用户需提供且仅提供以下参数：

*   **`TargetFile`**：目标文档的绝对路径或相对路径。
*   **`SemanticTarget`**：描述性定位字符串（例如 *“定位 SS-ID 为 'S1.12' 的 H4 区块”*），或 `full file`。
*   **`Reviewers`**：评审员名称列表（简称或路径）。
*   **`Standards`**：评审员名称到文档标准名称（简称或路径）的映射。
*   **`GoldfishContext`**：提供给评审员以建立基线认知的阅读清单（源文件、目录或文档 URL）。调用方传入空列表视为已提供；不要再次询问。由于部分评审员无法打开 URL，在第一轮开始前请将每个 URL 的原始页面保存为工作目录（或宿主产物目录）中的新文件，并改为传递其绝对路径。若某页面无法保存，请向用户索取本地副本并结束回合；若无人能回答（如无头运行），则排除它、告知用户并继续。

## 路径与标准解析逻辑

解析 `Reviewers` 与 `Standards` 中的名称时：

*   **评审员简称（Reviewer Shortnames）**：将评审员简称（例如 `design_critic`、`structural`、`style`、`implementation_readiness`、`codebase_details`、`comprehension`、`zoom_out`）映射到 `agents/{name}.md`。该人格文件定义了评审员的核心职责、审计维度与输出格式。
*   **标准简称（Standard Shortnames）**：将标准简称（例如 `feature-spec-standard`、`implementation-readiness-standard`、`zoom-out-standard`、`heilmeier-standard`、`re-implementation-standard`）映射到 `resources/document_standards/{name}.md`。
*   **自定义路径（Custom Paths）**：若名称包含斜杠（`/`），将其视为绝对路径或工作区相对路径并直接读取。

## 执行工作流

### 第 1 步：输入采集与配置体验

评估已提供的输入参数（`TargetFile`、`SemanticTarget`、`Reviewers`、`Standards`、`GoldfishContext`）。

1.  **识别缺失输入**：确定哪些必填参数尚未提供。
2.  **交互式引导**：
    *   若**全部**输入均已提供，直接进入第 2 步。
    *   若**部分或全部**输入缺失，通过聊天消息**仅针对缺失项**逐一交互式向用户询问。
3.  **评审员推荐**：
    *   *特性规格（Feature Specs）*：推荐 `design_critic`（映射到 `resources/document_standards/feature-spec-standard.md`）与 `structural`（映射到 `resources/document_standards/feature-spec-standard.md`）。
    *   *实现计划（Implementation Plans）*：推荐 `implementation_readiness`（映射到 `resources/document_standards/implementation-readiness-standard.md`）与 `codebase_details`。
4.  **确认配置**：展示汇总后的配置计划（目标、评审员及映射标准），并在继续前请用户确认。

### 第 2 步：打磨循环（迭代周期，The Sharpening Loop）

在持续循环中执行以下序列：

#### 2.1 无条件目标解析

在每轮迭代之前，无条件重新扫描并读取 `TargetFile`，利用稳定的 `SemanticTarget` 字符串（例如定位目标标题）动态确定更新后的绝对起止行号。即使文档在先前轮次中被编辑过，这也能保证行号绝对准确（若指定了 `full file` 模式，则确认全文件行号边界）。

#### 2.2 评审蜂群执行（Review Swarm Execution）

1.  **启动子智能体蜂群**：
    派生子智能体执行评审：
    *   **层级派发（合成器，Hierarchical Dispatch）**：派生中间合成器子智能体（`agents/synthesizer.md` / `egm-synthesizer`）。合成器并发扇出金鱼评审蜂群、对发现去重、写入报告产物，并仅返回一条汇总消息以避免根对话出现上下文污染。将下文的“失败评审员”规则与 `GoldfishContext` 一并传给合成器，并告知它将这些文件分发给每位评审员。其汇总必须为每位被请求的评审员给出状态：`completed` 或 `not run`。
    *   **直接派发（扁平蜂群，Direct Dispatch）**：或者直接派生 `Reviewers` 中请求的每一位评审员子智能体。指示每个子智能体先阅读 `agents/{name}.md` 中的人格定义，再对照 `resources/document_standards/` 中指定的标准审计 `TargetFile` 的已解析行号范围，并附上 `GoldfishContext`。
    *   每位评审员产出带严重级别（🔴 Blocker、🟡 Major、🟢 Nit）、行号、差异描述与理由的发现条目。
2.  **汇总发现**：将所有评审员的发现收集为单一扁平的 `FindingsList`。为每条发现分配稳定 ID（`<reviewer>-<round>-<n>`），并记录其评审员、严重级别与受影响范围。
3.  **失败评审员（Failed Reviewers）**：超时、失败或未返回报告的评审员均视为未完成评审。重试一次。若再次失败，在报告中将其记录为 **not run（未运行）**。绝不可将其计为零发现的评审员。若合成器汇总中缺少某位被请求评审员的状态，也将该评审员记录为 `not run`。

#### 2.3 聚合裁决交接（Aggregated Resolution Hand-off）

1.  将 `TargetFile`、当前有效行号范围以及聚合后的 `FindingsList` 传递给 **`egm-resolve`** 工作流。
2.  在主大象上下文中执行裁决工作流：
    *   将发现分类为 `CONTEST`（提出异议）与 `PROPOSE FIX`（建议修复）等裁决类别。
    *   使用文件写入工具保存 `goldfish_review_report.md`。
    *   呈现交互式决策门禁（Auto accept、Continue、Interactive breakdown）。
    *   使用文件编辑工具将获批的文本修复应用到 `TargetFile`。

#### 2.4 迭代检查与回合边界（Iteration Check & Turn Boundary）

由于每轮的新鲜上下文评审员采样视角不同，即使文档持续改进，出现零发现的轮次也十分罕见。请改用以下确定性状态机判定规则：

*   **已裁决发现（Settled findings）**：当一条发现被提出异议（`CONTEST`）或被记录为不予修复（已接受风险，accepted risk）时，即进入“已裁决（`Settled`）”状态。跨轮次持久化维护已裁决列表。若新一轮评审员再次提出同一条已裁决发现（相同的受影响范围、相同的问题），绝不要重新打开它。
*   **已修复发现（Fixed findings）**：已应用的修复（`Fixed`）**不会**使一条发现变为“已裁决（`Settled`）”。若后续轮次的评审员在同一受影响范围再次提出相同问题，说明此前的修复未奏效，或后续编辑重新引入了该问题——必须重新打开（re-open）该发现。
*   **净轮次（Clean Round）**：当没有任何评审员提出尚未裁决（not settled）的 `Blocker` 或 `Major` 级别发现时，该轮即判定为“净轮次（`Clean Round`）”。`Nit` 级别发现不构成阻塞。只要有任何一位被请求的评审员记录为 `not run`（未运行），该轮就**不是**净轮次。**必须在应用本轮修复之前**，仅凭本轮评审结果做出净轮次判定。
*   **已收敛（Converged）**：连续达成两个净轮次（two consecutive clean rounds）。单次净轮次不足以判定收敛，因为下一轮全新上下文的评审员仍可能在同一段文本中发现不同的缺口。
*   **轮次上限（Cap）**：最多 5 轮。
*   **修复振荡（Churn）**：一条新发现恰好出现在某项已应用修复的作用范围内，且只要回退（revert）那次修复即可解决该新发现。

当状态达到**已收敛（Converged）**时，进入第 3 步。当触及**轮次上限（Cap）**或检测到**修复振荡（Churn）**时，按严重级别列出简短的未决发现清单，随后向用户仅提出一个问题：接受现状并继续（accept and proceed）、突破上限继续迭代（continue past the cap），或将文档退回起草阶段（return the document to drafting）。

**关键回合交接要求（Crucial Turn-Taking Requirement）**：每轮裁决阶段完成后，循环驱动器**必须尊重 `egm-resolve` 中的回合交接检查点**。驱动器必须在人类回合期间保持空闲等待，仅在用户显式确认希望再跑一轮评审时，才循环回到第 2.1 步。

### 第 3 步：输出边界与最终检查点（Output Boundary & Final Checkpoint）

完成时（已收敛 `converged`、用户接受上限或修复振荡结果 `accepted at cap`，或手动停止 `halted`）：

*   **程序化调用方（Programmatic Callers）**：若作为 `egm-review` 的子流程运行，干净退出并将执行权交还父工作流。
*   **手动调用（Manual Invocations）**：在聊天中输出完成总结：*“评审循环已在 [N] 轮后完成（[converged | accepted at cap | halted]）。未决发现已记录在报告中。你是希望进入下一阶段，还是有其他最终问题？”*
