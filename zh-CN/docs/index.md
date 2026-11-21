> **原文路径**：`docs/index.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 大象-金鱼模型 (EGM)

## 查找合适的文档

| 位置 | 用途 |
| --- | --- |
| [快速开始](getting_started.md) 与 [常见问题 (FAQ)](faq.md) | 安装指南与日常使用。 |
| [宿主环境](harnesses.md)、[Claude](claude.md) 与 [Codex](codex.md) | 针对特定宿主的安装与执行方式。 |
| [设计文档](https://github.com/google/elephant-goldfish-model/blob/main/docs/designs/README.md) | 架构、工作流原理与实现计划。 |
| [共享运行时资源](../resources/README.md) | 权威执行规则与评审标准。 |
| [测试手册](https://github.com/google/elephant-goldfish-model/blob/main/TESTING.md) | 命令与验证证据要求。 |
| [PR #103 验证报告](https://github.com/google/elephant-goldfish-model/pull/103) | 详细的宿主环境测试结果与已知验证缺口。 |

`docs/` 面向贡献者和用户解释整个系统。`resources/` 包含运行时直接读取的规则。在此处解释这些规则时请链接至对应文件，并将具体要求保留在它们的权威资源文件中。

AI 已经改变了软件开发。一切都已不同，我们都深知这一点。

**大象-金鱼模型**（Elephant-Goldfish Model，EGM）是团队适应这一变化的方式之一。它是一套用于快速交付高质量软件的协议，并借用一个易于记忆的隐喻来呈现。Dave Rensin 的文章 [Elephants, Goldfish, and the New Golden Age of Software Engineering](https://medium.com/@drensin/elephants-goldfish-and-the-new-golden-age-of-software-engineering-c33641a48874) 对此进行了深入探讨。如果你尚未读过那篇文章，建议从那里开始！

## 大象-金鱼哲学

### 大象（Elephant）

**大象**能记住一切。

有时我们希望大语言模型（LLM）了解关于我们系统的全部信息：系统的演进历史、用户旅程、逻辑组件等等。并且我们希望能自然地与它探讨这些部分如何协同工作。

仅靠阅读代码是无法做到这一点的。我们需要以一种 LLM 能够低成本导航的方式，将这些信息全部提供给它。做到这一点后，我们就可以快速提问、质疑假设并迫使自己为决策提供依据，从而让系统变更变得更加轻松。

当我们为 LLM 提供富含上下文的系统描述，并要求它探索我们的问题时，我们称之为**大象**（高上下文构建者）。

### 金鱼（Goldfish）

**金鱼**只有三秒记忆 [^not-really]。

LLM 具有非确定性。有时它们会产生幻觉，导致生成的计划或代码存在明显缺陷。在不检查幻觉的情况下就信任 LLM 去构建系统，注定会酿成麻烦。

最朴素的做法是让人类评审每一处变更。这确实有效，但成本高昂，且很快会成为瓶颈。另一种方法是让一个 LLM 去评审另一个 LLM 的产出。如果我们要求 LLM 在*无法访问*计划、代码或其他产物的生成过程的前提下对其进行评审，就能有效减少幻觉。

当我们以这种方式限制 LLM 所能访问的上下文，并要求它评审产出时，我们称之为**金鱼**（零上下文评审员）。

### 人类（Humans）

想象一下你不得不对经理说：*“抱歉！我的 AI 工具刚才把有问题的代码推到了生产环境……”* 闯祸的不是工具，而是你。

或者想象另一个场景：*“AI 帮我做完了整个项目！给我升职吧！”* 这同样荒谬。

AI 是工具，而不是决策者。开发者必须主动引导设计讨论、质疑 AI 的建议并亲自做出决策，而不是放任 AI 主导一切。

EGM 坚定秉持这一原则：**人类**对最终结果负责。

### 核心收益

EGM 强调学习如何将 LLM 作为工具来使用，并阐明了如何驾驭其优势、规避其弱点。

*   **效率提升：** 功能交付更快。将对齐工作前置到设计规格阶段，能显著减少下游返工与反复的评审意见修改循环。
*   **准确性：** 在前期设计讨论中的投入会沉淀为深入、经过验证的指导方针，从而降低 LLM 的幻觉。清晰、详尽地说明我们究竟要实现什么，能够为 AI 提供明确的对照基准。
*   **团队收益：** 项目交接、新人入职、问题分诊以及其他以往可能需要白板协作的人类流程都会变得更轻松。当 LLM 清楚功能位于何处、如何构建以及背后的意图时，人类也能快速了解并为系统做出贡献。

## 文档与指南

*   [快速开始](getting_started.md)：安装步骤与核心工作流。
*   [Codex](codex.md)：插件安装与原生工作流验证。
*   [Claude Code](claude.md)：插件资源与隔离评审员机制。
*   [常见问题 (FAQ)](faq.md)：常见问题与解答。
*   [框架架构](https://github.com/google/elephant-goldfish-model/blob/main/docs/designs/architecture.md)：详细的组件设计与生命周期。
*   [贡献指南](https://github.com/google/elephant-goldfish-model/blob/main/CONTRIBUTING.md)：如何贡献新的技能、专家角色与文档标准。

[^not-really]: 金鱼只有三秒记忆其实是个[谣言](https://www.biology.ox.ac.uk/article/goldfish-show-a-keen-ability-to-estimate-distances)。但较真的话就会破坏这个隐喻！
