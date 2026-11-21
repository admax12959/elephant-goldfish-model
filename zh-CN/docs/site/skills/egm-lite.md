---
title: 轻量模式
description: 快速推进小型迭代、规格修订、代码重构与缺陷修复。
---

> **原文路径**：`docs/site/skills/egm-lite.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

## 功能说明

助手针对小型变更执行一套快速、解耦的 5 阶段工程协议，免去完整设计规格生命周期的开销：

1. **批判性暂停（Critique Pause）**：在触碰代码之前，先探查代码不变量并标记未经验证的假设。
2. **检查清单（Checklist）**：在 `egm_lite_checklist.md` 中生成轻量级任务清单；对于非平凡的变更，建议在编辑任何文件之前由隔离的子智能体执行一次规格评审（`goldfish` 或 `review`）。用户可以选择跳过该步骤。
3. **执行实现（Execution）**：落实已批准的变更并验证本地测试全部通过。
4. **对抗性评审（Adversarial Review）**：派生一个隔离的金鱼子智能体，围绕竞态条件、边界情况和缺失测试对代码差异（diff）进行审计。当当前没有进行中的任务时，`review` 或 `goldfish` 也可以直接评审你工作区中已有的变更，或你指定的某份 diff 或 Pull Request。
5. **裁决门禁（Resolution Gate）**：结合上下文对规格评审或代码差异评审的发现进行分流处理，划分为建议修复或基于证据的异议。规格评审无问题时提示进入 `exec`；代码差异评审无问题时关闭任务。

## 适用场景

适用于无需撰写完整多文档规格的小型快速迭代、规格修订、代码重构或缺陷修复。

## 产出结果

一份经过测试的代码变更，附带 `egm_lite_checklist.md` 中的执行检查清单以及一轮对抗性评审。

## 使用示例

```text
/egm-lite Fix lookup latency in RequestHandler by caching validated sessions
```

## 完整指令

助手遵循仓库中的[轻量模式技能指令](../../../skills/egm-lite/SKILL.md)。
