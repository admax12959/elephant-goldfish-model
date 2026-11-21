---
name: egm
description: 使用原生 Copilot CLI 评审员运行共享 EGM 技能。实验性功能。
tools: [read, search, edit, agent]
---

> **原文路径**：`com.github.copilot/agents/egm.agent.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

<!-- Copyright 2026 Google LLC. SPDX-License-Identifier: Apache-2.0 -->

要求用户或原生插件清单提供已安装 EGM 包的绝对根目录路径。严禁猜测缓存路径。开始工作前先读取 `<EGM_ROOT>/docs/copilot.md`。从同一根目录下读取所请求的规范 `skills/<skill>/SKILL.md`。保持技能目录名称中的连字符；例如，针对 `egm-brainstorm`，读取 `<EGM_ROOT>/skills/egm-brainstorm/SKILL.md`。

遵循共享技能的工作流与人类审批门禁。此配置文件仅用于适配宿主工具与路径，不替代共享工作流。使用八个已注册的 EGM 评审员配置进行直接原生委派。选择原生配置时，在规范智能体人格名称前加上 `egm:` 前缀。例如，`egm-design` 映射为 `egm:egm-design`。将结果合成与产物写入保留在当前主会话中。严禁使用内联评审替代缺失的原生评审员。
