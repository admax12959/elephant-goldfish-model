---
title: 安装 EGM
description: 选择你的 AI 应用程序并安装插件。
---

> **原文路径**：`docs/site/install.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

## ChatGPT 与 Codex 桌面端

在 ChatGPT 或 Codex 桌面端中打开 [EGM 插件页面](https://chatgpt.com/plugins/plugins_6aeb90f47ec88191a3c107e01d3eb357)，选择 **Install**（安装）。该页面仅支持桌面端。

## Claude Cowork

打开插件面板并添加以下插件市场：

```text
google/elephant-goldfish-model
```

## 命令行

根据你使用的应用程序运行相应命令。

### Claude Code

```sh
claude plugin marketplace add google/elephant-goldfish-model
claude plugin install egm@egm
```

### Codex CLI

```sh
codex plugin marketplace add google/elephant-goldfish-model
codex plugin add egm@egm
```

### Antigravity

```sh
agy plugin install https://github.com/google/elephant-goldfish-model
```

## 在首次评审之前

请安装完整插件，而不仅仅是单独的技能文件。评审环节需要多个在新鲜对话上下文中运行的独立 AI 评审员。

云端评审支持存在一定限制。详情请参阅[应用程序支持状态](https://github.com/google/elephant-goldfish-model/blob/main/docs/harnesses.md#support-status--contributing-other-harnesses)。

## 下一步

按照[快速开始](./getting-started.md)的步骤打磨一个想法或评审一份文档。

需要帮助？请查阅[安装指南](https://github.com/google/elephant-goldfish-model/blob/main/docs/harnesses.md#install)或[提交 Issue](https://github.com/google/elephant-goldfish-model/issues)。
