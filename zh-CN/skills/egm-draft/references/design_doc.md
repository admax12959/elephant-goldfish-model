> **原文路径**：`skills/egm-draft/references/design_doc.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# [特性/项目名称]

*   **Author:** [作者姓名或 GitHub Handle]
*   **Status:** [Draft | Proposed | In Review | Approved]
*   **Date:** [YYYY-MM-DD]

--------------------------------------------------------------------------------

## Overview

<!-- 提供特性的高层摘要、它解决的业务或技术问题及其在系统中的上下文。
保持简洁，以便人类和智能体都能快速理解主要目标。
-->

## The Problem

[在此总结问题与拟议解决方案。]

## Goals

<!--
该特性将达成什么目标？要求具体、可衡量且清晰。
这些目标代表成功的绝对验收标准。
-->

*   目标 1
*   目标 2

## Non-Goals

<!--
明确说明哪些内容在范围之外。这对于防止需求蔓延以及为实现智能体划定严格边界至关重要。
-->

*   非目标 1
*   非目标 2

## Open Questions

<!--
列出未解决的设计问题、技术未知项或需要调研的依赖项。未回答的问题会跨起草会话保留在此，
以保存对话记忆并避免后续会话重复询问已经讨论过的问题。问题解决后将其移除。
-->

*   **Question 1**: [技术未知项或开放问题描述]
*   **Question 2**: [技术未知项或开放问题描述]

## Critical User Journeys (CUJs)

<!--
定义本设计支持的端到端关键用户旅程。每个 CUJ 必须包含：

1. Given：前置条件、初始状态与环境配置。
2. When：触发条件、用户动作或系统事件。
3. Then：预期结果、验证标准与后置条件。
-->

### CUJ 1: [旅程标题]

*   **Given**: [前置条件]
*   **When**: [触发动作]
*   **Then**: [预期结果]

### CUJ 2: [旅程标题]

*   **Given**: [前置条件]
*   **When**: [触发动作]
*   **Then**: [预期结果]

## Technical Plan (Architecture)

<!-- 详述高层设计、架构选型、系统数据流与组件交互。
附上 ASCII 或 Mermaid 图表以清晰建模各组件间关系。
-->

[在此解释架构选型、数据结构与系统组件映射。]

```text
[可选 ASCII 架构图]
```

## Alternatives Considered

<!-- 记录曾评估并否决的其他架构方法或设计。
说明优缺点，以及所选方案为何更优的技术推理。
-->

### Alternative A: [替代方案标题]

*   **Pros**: [列出优点]
*   **Cons**: [列出缺点]
*   **Reason for Rejection**: [未选择该方案的原因（例如 "Rejected by author: technical rationale"）。除非链接到维护者的 Issue、Discussion 或 PR，否则不得将否决归因于维护者。]

--------------------------------------------------------------------------------

## Non-Functional Requirements

<!-- 仅当非功能性考量构成本设计的关键风险时进行分析：

- 安全与隐私（Security & Privacy）：（数据敏感性、认证/授权、密钥管理）。
- 性能与延迟（Performance & Latency）：（执行耗时、存储占用、LLM Token 开销）。
- 发布与回滚（Rollout & Rollback）：（特性开关、向后兼容性、回滚命令）。
-->

[在此详述任何关键非功能性需求。如果没有关键风险，本节可省略或填写 "None"。]

--------------------------------------------------------------------------------

### Additional Resources

<!-- 注明设计讨论期间使用的外部资源与上下文。可选。若未提供则省略本节。-->

*   引用的重要项目文档
*   引用的重要 `docs/` 文档
*   相关工作

### Future Improvements

<!-- 在此说明未来有改进空间的领域。若未提供则省略本节。-->

*   **想法**：哪些方面可以改进？

--------------------------------------------------------------------------------

## Detailed Implementation Plan

<!-- 精确指定必须新建或修改哪些文件。不要使用含糊措辞或 TBD。
-->

### Target Files to Create/Modify

#### 1. `[文件路径 1]` ([Create | Modify])

*   **Type / Scope**: [例如 Python 服务、C++ 库、TypeScript 模块、构建配置]
*   **Proposed Changes**: [在此描述新增 API、类/函数签名或行为逻辑。聚焦实现所需的最小必要细节。]

#### 2. `[文件路径 2]` ([Create | Modify])

*   **Type / Scope**: [例如单元测试、端到端测试驱动]
*   **Proposed Changes**: [在此描述测试覆盖范围、预期测试用例或 Mock 断言。]

### Verification Plan

<!-- 指明必须通过哪些单元测试、集成测试或手动测试以验证实现。
应衡量哪些指标来验证成功？
-->

*   **Compilation/Build**: [例如 `npm run build`、`cargo build`、`make`]
*   **Unit Tests**: [例如 `pytest`、`npm test`、`cargo test`]
*   **Static Analysis & Linting**: [例如 `eslint`、`ruff check`、`golangci-lint`]
*   **Test Coverage Requirement**: [例如 85%+ 分支覆盖率]
*   **E2E/Integration**: [例如集成测试命令或自动化评估运行器]

--------------------------------------------------------------------------------

<!-- Provenance: Indicates initial structural drafting via EGM. -->

*Drafted with EGM*
