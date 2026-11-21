---
title: 快速开始
description: 体验该方法、选择应用程序，并启动你的第一个项目。
---

> **原文路径**：`docs/site/getting-started.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

## 从方法论开始

阅读[仓库配置指南](https://github.com/google/elephant-goldfish-model/blob/main/docs/getting_started.md)。该指南介绍了对话式协作方法，并附有奠基性文章的链接。你可以在安装插件之前先体验这套方法。

## 选择一个应用程序

该插件运行在受支持的 AI 应用程序（宿主环境）内部。在开始你的第一个项目之前，请先在你的应用程序中[安装 EGM](./install.md)。

安装 EGM 并不要求你一定要编写软件代码。评审工作流需要能够运行隔离的独立子智能体的应用程序。目前云端支持仍不完整，请查看安装指南了解当前的限制。

## 从一个软件想法开始

描述你要解决的问题、受影响的对象以及你已知的信息。例如：

```text
/egm-brainstorm
Help students book a shared study space.
Help me explore who needs access, how bookings work,
and what happens when two requests overlap.
```

助手会提出问题并撰写一份简短的简报。在继续进入[起草与评审](./workflow.md)之前，请先阅读这份简报。

## 评审其他类型的文档

将文档保存为文件，然后使用[自定义评审](./skills/egm-review-loop.md)为你的任务选择合适的评审员与文档标准。

```text
/egm-review-loop
Review study-space-plan.md.
Help me choose reviewers and standards
for the opening schedule and responsibilities.
```

检查评审配置，并在修改方案之前仔细核对评审发现的问题。
