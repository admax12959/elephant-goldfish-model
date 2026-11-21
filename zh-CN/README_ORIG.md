> **原文路径**：`README.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

<p align="center">
  <a href="https://egm.dev"><img src="../docs/assets/readme-header.svg" width="480" height="116" alt="大象-金鱼模型 (Elephant-Goldfish Model)"></a>
</p>

<p align="center">大象起草，金鱼评审。</p>

<p align="center">
  <a href="#安装">安装</a> &ensp;·&ensp;
  <a href="#尝试你的第一个设计">快速上手</a> &ensp;·&ensp;
  <a href="docs/getting_started.md">文档</a> &ensp;·&ensp;
  <a href="https://egm.dev">官网</a>
</p>

<br>

带着完整上下文构思方案，用全新视角评审文档。EGM（大象-金鱼模型）是一款面向 AI 编程助手的开源插件。

**大象（Elephant，高上下文构建者）** 掌握你的项目上下文并起草规格文档。**金鱼（Goldfish，零上下文评审员）** 在剥离前期对话历史的隔离上下文中审视草稿与引用的源码材料，帮助暴露隐含假设与盲区。**你（人类）** 在真正实现代码之前做出架构权衡并批准规格文档。

内置的起草与设计评审工作流面向软件规格文档。对于其他类型的文档，你可以使用 `/egm-review-loop` 自由选择评审员和文档标准。初始化引导（Bootstrap）与漂移报告（Drift reporting）适用于软件代码库。

https://github.com/user-attachments/assets/93c799a2-10a3-4266-af02-eb79dd33a9a2

## 安装

选择你的 AI 编程智能体。请安装完整插件，以便各项技能能够读取评审员提示词与文档标准。

### ChatGPT 与 Codex 桌面端

在桌面应用中打开[已发布的 EGM 插件](https://chatgpt.com/plugins/plugins_6aeb90f47ec88191a3c107e01d3eb357)并安装。此插件商店页面仅限桌面端使用。如需在 Codex CLI 中使用，请执行下方命令。

### Antigravity CLI

```sh
agy plugin install https://github.com/google/elephant-goldfish-model
```

对于 **Antigravity App** 与 **Antigravity for IDEs**（Visual Studio Code 扩展），请运行 `/plugin` 或参阅[安装说明](docs/harnesses.md#antigravity-app-and-antigravity-for-ides)。

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

安装完成后请开启一个新会话。关于 **Claude Cowork** 与 **ChatGPT Work** 的配置，请参阅[环境配置指南](docs/harnesses.md)。

## 在现有代码上体验 60 秒快速评审

对照线上现行代码，评审现有的设计文档、README 或 PR 计划，无需从头编写新规格。下方命令针对 Antigravity（agy CLI）与 Claude Code 使用了 `/egm:` 前缀。在 Codex 中，请从技能选择器中选择对应的 EGM 技能。

<img src="../skills/egm-review/assets/icon.svg" width="20" height="20" align="top" alt=""> **评审现有文档**

```text
/egm:egm-review docs/designs/webhooks.md
```

传入你的设计文档路径，或保存为 Markdown 的 PR 计划路径。零上下文的新鲜评审员将检查设计合理性、表述清晰度、与代码库的契合度以及实现计划。

<img src="../skills/egm-drift-report/assets/icon.svg" width="20" height="20" align="top" alt=""> **检查文档漂移**

```text
/egm:egm-drift-report docs/designs/
```

智能体将把该目录中的每份文档与当前源代码进行比对，并报告二者的偏离之处。由你决定是更新文档还是修改代码。

## 尝试你的第一个设计

在你的 AI 编程智能体中打开项目，选择一个需要做设计决策的特性。本示例为项目添加 Webhook 投递功能：当投递失败，或同一事件重复到达时，系统应该如何处理？

下方命令展示时不带前缀。在 Antigravity（agy CLI）与 Claude Code 中，请加上 `/egm:` 前缀，例如 `/egm:egm-brainstorm`。在 agy CLI 1.4.10 中，无前缀名称会自动补全为带前缀的命令（[#220](https://github.com/google/elephant-goldfish-model/issues/220)），但 1.4.8 版本不支持（[#196](https://github.com/google/elephant-goldfish-model/issues/196)），因此请保留前缀。在 Codex 中，请从技能选择器中选择对应的 EGM 技能。头脑风暴与评审均要求会话具备创建新鲜子智能体的能力。

<img src="../skills/egm-brainstorm/assets/icon.svg" width="20" height="20" align="top" alt=""> **探索构想**

```text
/egm-brainstorm Add webhook delivery to this project.
```

与智能体讨论需求与权衡。新鲜上下文评审员会对方案发起质疑，智能体随后将达成一致的方向保存至 `concepts_brief.md`。

<img src="../skills/egm-draft/assets/icon.svg" width="20" height="20" align="top" alt=""> **起草规格文档**

```text
/egm-draft
```

智能体将概念简报转化为 `design_draft.md`，并在每完成一个章节后暂停，等待你在聊天中提供反馈。

<img src="../skills/egm-review/assets/icon.svg" width="20" height="20" align="top" alt=""> **评审草稿**

```text
/egm-review design_draft.md
```

请使用智能体返回的草稿文件路径。

新鲜上下文评审员将检查设计本身、文档清晰度、与现行代码的契合度以及实现计划。由你决定要处理哪些评审发现。当你批准设计后，即可让你的 AI 编程智能体开始实现。

完整流程（包括将已批准的规格文档保存至 `docs/designs/`）请参阅[工作流指南](docs/getting_started.md)。

## 工作原理

<table>
  <thead><tr><th align="left">角色</th><th align="left">上下文</th><th align="left">职责</th></tr></thead>
  <tbody>
    <tr valign="top"><td><strong>大象（Elephant）</strong></td><td>你的项目、需求与前期决策</td><td>探索问题并起草规格文档。</td></tr>
    <tr valign="top"><td><strong>金鱼（Goldfish）</strong></td><td>仅限文档本身及其引用的源码材料，剥离前期对话历史</td><td>发现缺失的假设、含糊的需求与实现风险。</td></tr>
    <tr valign="top"><td><strong>你（人类）</strong></td><td>真正重要的目标与约束条件</td><td>解决架构权衡、批准设计并决定要构建什么。</td></tr>
  </tbody>
</table>

金鱼评审需要新鲜的子智能体上下文。如果你的智能体无法创建新鲜子智能体，EGM 会中止评审。评审不能替代测试或人类判断。请参阅[受支持的宿主环境与验证边界](docs/harnesses.md#support-status--contributing-other-harnesses)。

## 选择一项技能

你可以按需单独使用任意技能，不必严格走完整个流程。

<table>
  <thead><tr><th align="left">技能</th><th align="left">用途…</th></tr></thead>
  <tbody>
    <tr valign="top"><td width="160" valign="top"><img src="../skills/egm-brainstorm/assets/icon.svg" width="20" height="20" alt="" align="top">&nbsp;<a href="skills/egm-brainstorm/SKILL.md">Brainstorm（头脑风暴）</a></td><td>检验构想并撰写概念简报。</td></tr>
    <tr valign="top"><td width="160" valign="top"><img src="../skills/egm-draft/assets/icon.svg" width="20" height="20" alt="" align="top">&nbsp;<a href="skills/egm-draft/SKILL.md">Draft（起草）</a></td><td>将概念简报或现有笔记转化为设计文档。</td></tr>
    <tr valign="top"><td width="160" valign="top"><img src="../skills/egm-resolve/assets/icon.svg" width="20" height="20" alt="" align="top">&nbsp;<a href="skills/egm-resolve/SKILL.md">Resolve（裁决修订）</a></td><td>逐条处理评审发现并就修改达成共识。</td></tr>
    <tr valign="top"><td width="160" valign="top"><img src="../skills/egm-bootstrap/assets/icon.svg" width="20" height="20" alt="" align="top">&nbsp;<a href="skills/egm-bootstrap/SKILL.md">Bootstrap（引导建档）</a></td><td>为现有代码库创建或更新文档。</td></tr>
    <tr valign="top"><td width="160" valign="top"><img src="../skills/egm-lite/assets/icon.svg" width="20" height="20" alt="" align="top">&nbsp;<a href="skills/egm-lite/SKILL.md">Lite（轻量快轨）</a></td><td>快速推进小型迭代、规格修订、重构与缺陷修复。</td></tr>
    <tr valign="top"><td width="160" valign="top"><img src="../skills/egm-review/assets/icon.svg" width="20" height="20" alt="" align="top">&nbsp;<a href="skills/egm-review/SKILL.md">Review（五阶段评审）</a></td><td>使用新鲜上下文智能体分五个阶段评审设计文档。</td></tr>
    <tr valign="top"><td width="160" valign="top"><img src="../skills/egm-review-loop/assets/icon.svg" width="20" height="20" alt="" align="top">&nbsp;<a href="skills/egm-review-loop/SKILL.md">Review&nbsp;loop（评审循环）</a></td><td>针对特定评审任务运行可配置的评审员组合。</td></tr>
    <tr valign="top"><td width="160" valign="top"><img src="../skills/egm-drift-report/assets/icon.svg" width="20" height="20" alt="" align="top">&nbsp;<a href="skills/egm-drift-report/SKILL.md">Drift&nbsp;report（漂移报告）</a></td><td>将已入库文档与当前源代码进行比对。</td></tr>
  </tbody>
</table>

## 延伸阅读

- [快速入门](docs/getting_started.md)：完整工作流与项目目录布局。
- [环境配置](docs/harnesses.md)：安装方式、智能体能力与已知限制。
- [架构设计](docs/designs/architecture.md)：技能与评审员如何协同工作。
- [测试手册](TESTING.md)：自动化检查、执行记录与验证边界。
- [Dave Rensin 的原始文章](https://medium.com/@drensin/elephants-goldfish-and-the-new-golden-age-of-software-engineering-c33641a48874)：EGM 背后的核心思想。

## 参与贡献

请参阅[无障碍声明](ACCESSIBILITY.md)了解无障碍目标、已知限制以及如何反馈使用障碍。

提交 PR 请阅读[贡献指南](CONTRIBUTING.md)，报告缺陷请前往 [Issues](https://github.com/google/elephant-goldfish-model/issues)，提问交流请前往 [Discussions](https://github.com/google/elephant-goldfish-model/discussions)。

## 许可证

[Apache 2.0](../LICENSE)。本项目并非 Google 官方支持的产品。
