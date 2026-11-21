> **原文路径**：`docs/designs/plugin_listing_presentation.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 插件 0.1.1 上架展示与产物包呈现

- **作者：** @n3wth
- **状态：** 提案（Proposed）。维护者范围审批待定。
- **日期：** 2026-11-04
- **范围 Issue：** #122。原始候选：PR #302。已拆分为独立评审单元替代该 PR。
- **候选提交：** `b903f35`，在不改动产物包的前提下向 `53077c3d64c42fc906f18373892a8e63a1ce9d6e` 新增了中性输入采集测试。

## 概述

在各插件清单文件及其生成副本中使用简洁、一致的文案。在插件上架信息中使用发布者的 Google 隐私政策链接，并保留仓库许可证作为服务条款链接。保留现有的 GitHub 支持页面链接与官网 URL。为 0.1.1 版本准备启动提示词（starter prompts）、逐技能默认提示词、PNG 图标以及便携式清单 Schema 声明。

本设计仅覆盖上架展示信息，不代表发布者政策审批通过，也不代表原生应用工作流支持。

## 问题背景

当前的插件摘要使用了内部工作流术语且描述冗长。目录隐私链接与发布者隐私政策不一致。读者在安装前需要清晰了解插件的用途。

原始 PR #302 超出了小补丁（minor-patch）的范围限制，修改了全部七个权威技能描述及其生成副本。本提案提供一条独立的设计评审路径，不申请范围豁免。

## 目标

- 用通俗平实的语言描述构想发展、文档准备与独立评审。
- 在权威文件与生成文件中为每个技能统一使用一条简短描述。
- 将 OpenAI 目录分类设为 Productivity（效率工具）。
- 保留 `https://egm.dev` 作为官网目标链接。
- 保留仓库 Issues 页面作为支持目标链接。
- 将 `privacyPolicyUrl` 设为 `https://policies.google.com/privacy`，并保留 `https://github.com/google/elephant-goldfish-model/blob/main/LICENSE` 作为 `termsOfServiceUrl`。
- 保持清单文件符合 Schema 校验，且生成的 Claude 文件与打包输出完全一致。
- 新增三条插件启动提示词，以及七个技能各自的默认提示词。
- 保留带 EGM 前缀的技能名称与稳定的技能标识符。
- 打包七个 256×256 的 PNG 技能图标，在 OpenAI YAML 元数据中引用它们，同时保留 SVG 源文件。
- 在便携式清单中声明 Agent Plugins 1.0.0 Schema，并将各宿主版本号统一对齐到 0.1.1。
- 测试不完整提示词、无关请求以及带有用户自定义评审标准的非软件提案。

## 非目标

- 更改技能指令、评审员权限或执行逻辑。
- 新增宿主环境、执行器、工具或网络服务。
- 将目录上架等同于原生工作流支持的证明。
- 更改官网文档系统或托管方式。
- 宣称 Google 官方支持该项目。
- 用服务条款声明取代软件许可证。

## 待决问题

- 发布者必须核查所链接的政策页面是否适用于本插件。
- 维护者必须批准贡献范围与阶段标签。
- 评审员必须核查更短的描述是否仍足以支撑技能选择。

## 关键用户旅程 (CUJs)

### 理解插件上架信息

- **前置条件（Given）：** 读者打开插件上架页面。
- **操作（When）：** 读者阅读插件摘要与技能列表。
- **预期结果（Then）：** 读者无需理解内部评审员术语即可识别插件的主要任务。
- **预期结果（Then）：** 上架信息不会暗示超出既有执行器与宿主环境限制的支持能力。

### 查看政策与支持信息

- **前置条件（Given）：** 读者检查开发者与政策字段。
- **操作（When）：** 读者打开官网、支持、隐私或条款链接。
- **预期结果（Then）：** 每个字段均打开其声明的目标页面，不存在损坏的仓库路径。

## 技术方案

首先更新权威根清单（canonical root manifest）。使用上述固定候选提交中的元数据值，而非可变的 PR HEAD。该提交定义了上架字段、能力声明以及全部七条替换描述。仅在既有 Schema 支持对应字段的位置更新兼容性清单。保留平台专属字段以及本地执行器要求。不添加清单 Schema 拒绝的字段。

缩短七个权威技能文件中的 frontmatter 描述，保持每个技能的名称、工作流正文与相对引用路径不变。在每个技能的 `agents/openai.yaml` 中添加默认提示词与 PNG 图标路径。工作流正文保持不变；新元数据仅改变用户发现与调用这些工作流的方式。保留带 EGM 前缀的技能名称与现有品牌色。将七个 PNG 文件纳入打包构建器的资源清单，同时保留 SVG 源文件以供既有图形与配色检查使用。在 `.plugin/plugin.json` 中保留 Schema 声明，但为了兼容 `omp` 在根 `plugin.json` 中省略 `$schema`。将根清单以及 Claude、Codex、Cursor 和便携式宿主版本号统一对齐至 0.1.1。使用现有打包构建器重新生成 Claude 运行时，不要独立编辑生成的副本。

长版上架描述不得作为执行限制的唯一记录。安装文档仍保留隔离评审员要求、云端部分支持以及宿主权限边界。项目支持免责声明仍保留在仓库与官网中。合并前必须由发布者评审措辞与政策链接。

## 备选方案考量

### 仅修改兼容性清单

该选项能快速改变可见的上架信息，但会导致权威元数据不一致。现有的一致性测试会拒绝该状态。

### 保留现有的长版技能描述

该选项保留了详细的触发词表述，但在面向用户的目录中暴露了内部阶段名称。选定候选方案采用简短描述，同时在各技能正文内完整保留工作流文本。

### 将每个技能描述拆分为单独的小 PR

该选项人为制造范围边界，使文案一致性更难评审。应将全部七条描述保留在同一个上架与提示词 PR 中。

### 发布上架文案但不含提示词或 PNG 资产

该选项能缩小产物包差异，但会把发现体验与首次调用留作单独的评审决策。本次发布将上架文案与提示词、PNG 资产、发布者政策链接拆分为独立的评审单元，每个单元均包含其测试与生成文件，且特性单元保留阶段 2 门禁。默认提示词只是暴露现有工作流，不会绕过缺失输入检查，也不会授予编辑文档的权限。

### 仅保留 SVG 图标引用

该选项可避免新增七个源资产。候选方案转而为宿主选择器提供固定尺寸的 PNG 资产，同时保留权威 SVG 图形。PNG 文件只是提供另一种宿主资产格式，并不证明宿主修复了通用图标问题。将原生图标渲染保留为明确的验证缺口。

便携式 Schema 声明与对齐后的版本号描述的是同一个产物包候选。应一并检查 Schema 与生成的资源清单，而不引入独立的打包系统。

## 非功能性需求

- 在计划内的元数据更新和新增七个 PNG 资产之外，保持清单 Schema 有效性及产物包内容不变。
- 不新增运行时网络请求或依赖项。
- 保留无需仓库写权限即可公开访问的政策与支持链接。
- 若发布者拒绝某些元数据字段，则回退这些字段并重新生成运行时。

## 详细实现

固定候选提交共修改 55 个文件：31 个源码、宿主与测试文件，以及 24 个生成的 Claude 文件。其中 14 个文件是 PNG 资产：7 个权威文件与 7 个生成副本。七个权威技能正文保持不变。

### 源码、宿主与测试文件（31）

- `.claude-plugin/marketplace.json`。
- `.claude-plugin/plugin.json`。
- `.codex-plugin/plugin.json`。
- `.cursor-plugin/plugin.json`。
- `.plugin/plugin.json`。
- `README.md`。
- `plugin.json`。
- `scripts/pack_plugin.py`。
- `skills/egm-bootstrap/SKILL.md`。
- `skills/egm-bootstrap/agents/openai.yaml`。
- `skills/egm-bootstrap/assets/icon.png`。
- `skills/egm-brainstorm/SKILL.md`。
- `skills/egm-brainstorm/agents/openai.yaml`。
- `skills/egm-brainstorm/assets/icon.png`。
- `skills/egm-draft/SKILL.md`。
- `skills/egm-draft/agents/openai.yaml`。
- `skills/egm-draft/assets/icon.png`。
- `skills/egm-drift-report/SKILL.md`。
- `skills/egm-drift-report/agents/openai.yaml`。
- `skills/egm-drift-report/assets/icon.png`。
- `skills/egm-resolve/SKILL.md`。
- `skills/egm-resolve/agents/openai.yaml`。
- `skills/egm-resolve/assets/icon.png`。
- `skills/egm-review-loop/SKILL.md`。
- `skills/egm-review-loop/agents/openai.yaml`。
- `skills/egm-review-loop/assets/icon.png`。
- `skills/egm-review/SKILL.md`。
- `skills/egm-review/agents/openai.yaml`。
- `skills/egm-review/assets/icon.png`。
- `tests/integration/test_codex.py`。
- `tests/unit/test_packaging.py`。

各清单文件定义了上架文案、政策链接、支持的提示词、Schema 声明以及 0.1.1 版本号。README 区分了软件工作流与面向其他文档的可配置评审。七个 `SKILL.md` 文件仅更新 frontmatter 描述。七个 YAML 文件在保留技能名称的前提下添加了默认提示词与 PNG 路径。打包构建器纳入新的 PNG 文件。单元测试校验 Schema 声明、提示词标识符、技能名称、PNG 尺寸与资源一致性。原生测试覆盖不完整启动提示词、无关请求与非软件提案。

### 生成的 Claude 文件（24）

使用现有打包构建器重新生成这些文件，不要独立编辑它们。

- `distributions/claude/.claude-plugin/plugin.json`。
- `distributions/claude/README.md`。
- `distributions/claude/plugin.json`。
- `distributions/claude/skills/egm-bootstrap/SKILL.md`。
- `distributions/claude/skills/egm-bootstrap/agents/openai.yaml`。
- `distributions/claude/skills/egm-bootstrap/assets/icon.png`。
- `distributions/claude/skills/egm-brainstorm/SKILL.md`。
- `distributions/claude/skills/egm-brainstorm/agents/openai.yaml`。
- `distributions/claude/skills/egm-brainstorm/assets/icon.png`。
- `distributions/claude/skills/egm-draft/SKILL.md`。
- `distributions/claude/skills/egm-draft/agents/openai.yaml`。
- `distributions/claude/skills/egm-draft/assets/icon.png`。
- `distributions/claude/skills/egm-drift-report/SKILL.md`。
- `distributions/claude/skills/egm-drift-report/agents/openai.yaml`。
- `distributions/claude/skills/egm-drift-report/assets/icon.png`。
- `distributions/claude/skills/egm-resolve/SKILL.md`。
- `distributions/claude/skills/egm-resolve/agents/openai.yaml`。
- `distributions/claude/skills/egm-resolve/assets/icon.png`。
- `distributions/claude/skills/egm-review-loop/SKILL.md`。
- `distributions/claude/skills/egm-review-loop/agents/openai.yaml`。
- `distributions/claude/skills/egm-review-loop/assets/icon.png`。
- `distributions/claude/skills/egm-review/SKILL.md`。
- `distributions/claude/skills/egm-review/agents/openai.yaml`。
- `distributions/claude/skills/egm-review/assets/icon.png`。

新增上述 14 个 PNG 资产，不删除任何源文件，不更改任何依赖。

### 评审单元与合并顺序

- PR #314 在各清单中更新发布者隐私链接，同时保留仓库许可证作为服务条款。目标分支为 `main`，需发布者审批。
- PR #316 包含上架文案、技能描述、插件启动提示词、Schema 元数据、版本号以及 Cursor 清单检查。目标分支为 `main`。
- PR #317 包含逐技能 OpenAI 默认提示词及其打包与原生提示词测试。目标分支为 #316 的分支。

政策 PR 彼此独立，维护者可在不阻塞其他单元的情况下拒绝它。特性 PR 在本设计合并且 Issue #122 获得实现审批前保持门禁锁定。须先将 #316 合并到 `main`，再合并 #317。在 #316 合并后，将 #317 变基（rebase）到 `main` 上、重新指定其目标分支，并重跑检查与评审。不要将 #317 合并到临时父分支。上架评审已在上架文案中恢复了既有的支持免责声明与云端部分支持说明。实证检查表明 PNG 技能图标并未改变宿主选择器的显示效果，因此实现保留 SVG 技能图标。此拆分遵循维护者关于单一目的与更小 Diff 的要求，并不构成该特性的小补丁豁免。

### 必需审批流程

1. 为阶段 1 获取维护者范围批准及 `design-ready` Issue 标签。
2. 合并本设计，并在实现合并前获取 `pr-ready` Issue 标签。
3. 更新权威元数据并跨清单比对受支持字段。
4. 重新生成 Claude 运行时并检视精确 Diff。
5. 运行单元测试、为两个目标构建产物包并评审上架文案。
6. 完成独立人类评审以及发布者对政策链接的评审。

这些标签名称遵循仓库当前的 PR 模板。具备评审就绪状态不代表已获得范围批准或允许合并替代特性 PR。Issue #122 仍是上架发布的范围参考，本设计并不完成其全部上架要求。

## 必需测试

- 在合并技能描述变更前，先完成[发布分支设计](claude_release_branch_distribution.md)中的 Claude 分发切换。在发布器与市场路径通过验证且上架负责人确认切换分支前，保留生成目录。
- 针对每个通过描述发现技能的受支持宿主环境，在每次仅安装一个产物包的新鲜会话中，使用相同任务提示词对比当前版本与提议版本的产物包。为每个修改过的技能准备省略技能名称的匹配提示词，同时包含不应选中任何 EGM 技能的相近请求。
- 记录每个用例的宿主与模型版本、产物包修订版，以及观察到的技能加载或未加载结果。要求目标技能正常加载且无新增漏触发或误触发。将不受支持或未测试的宿主单独记录。显式调用启动提示词属于工作流检查。
- 运行完整的仓库单元测试套件。
- 使用 `scripts/pack_plugin.py` 构建 OpenAI 与 Claude 产物包。
- 检查清单 Schema 有效性以及共享元数据一致性。
- 检查生成的 Claude 文件与权威打包输出字节级一致。
- 检查所有宿主版本号均为 0.1.1，并检查声明的 Agent Plugins 1.0.0 Schema。
- 检查全部七条默认提示词均包含预期的技能标识符。
- 检查 PNG 文件头签名、256×256 尺寸、图标路径、保留的 SVG 源文件以及产物包资源一致性。
- 检查技能正文与既有引用路径未发生任何变化。
- 针对精确的 OpenAI ZIP 运行显式启用的原生测试，并保留其校验和与已安装文件比对记录。
- 检查不完整的启动提示词不会捏造文档或启动评审员。
- 直接调用实际的启动提示词，不附带关于缺失输入或审批的额外说明，要求回复主动向用户索取缺失输入。
- 检查无关的算术请求不会触发评审流程。
- 检查两名隔离评审员在读取非软件提案与评审标准时不修改提案。
- 实际打开官网、支持、隐私与条款链接目标。
- 获取发布者对政策适用性与分类选择的评审确认。

通过离线测试不代表原生安装或完整云端评审支持。自定义图标渲染仍未经验证；PNG 资源检查不能证明 Codex 会显示这些图标。在范围、设计、发布者与人类评审要求全部通过前，实现不得合并。

在已配置好文档化测试环境的仓库根目录运行以下命令：

```sh
python -m pytest tests/unit -q
python3 scripts/pack_plugin.py --target openai --output <temporary-directory>
python3 scripts/pack_plugin.py --target claude --output <temporary-directory>
git diff --check
git diff 0dfb5bf b903f35 -- skills/
```

使用 Claude 打包命令重新生成 Claude 运行时，随后用生成的 `egm-claude` 目录替换已签入的运行时。`tests/unit/test_frontmatter.py` 校验元数据结构，`tests/unit/test_packaging.py` 将生成的运行时与权威打包输出进行比对。逐一评审每个 `SKILL.md` 的 Diff，确认仅修改了 frontmatter 描述。YAML 元数据与 PNG 新增项属于技能目录内另行规划的变更。

仅在实现候选版本上通过显式启用运行付费原生测试：

```sh
EGM_CODEX_RELEASE_ZIP=/absolute/path/to/egm-openai-0.1.1.zip \
  EGM_CODEX_LIVE=1 python -m pytest tests/integration/test_codex.py \
  -k 'starter_prompt or nonsoftware or unrelated_prompt' -v
```

### 已报告的候选验证证据

修订后的启动测试单独调用上架提示词，不附带缺失输入或权限说明。在 `b903f35` 上，使用 Codex CLI 0.166.0 的三个原生启动用例在 50.28 秒内通过。最终回复要求提供缺失输入，未启动任何评审员，也未生成任何 Markdown 文档。单元测试套件通过 336 项测试，跳过两项可选原生探针。被测 ZIP 的 SHA256 为 `47178bb1b08d3b3ef4b663991bbd4b6542a64f91edfcaa30b9a7c2552d1c2df2`。全新构建的产物包与该归档一致。本次修订仅改动测试，因此产物包字节保持不变。

PR #302 的描述记录了 `53077c3d64c42fc906f18373892a8e63a1ce9d6e` 上的以下结果。本次设计更新仅记录该报告，未重跑测试也未核查原生追踪日志。

- 单元测试套件：336 项通过，2 项可选原生宿主探针跳过。
- 上述精确 ZIP 原生子集：5 项在 103.67 秒内通过。
- 3 个不完整提示词用例均未启动评审员或捏造文档。
- 算术请求未进入评审工作流。
- 两名新鲜上下文评审员读取提供的提案与标准，指出了缺失的负责人与财务信息，且未修改提案。
- 报告称 Schema 校验、归档完整性、校验和、接口一致性以及可复现产物比对均通过。
- 报告称最终的 OpenAI ZIP 与上述五项原生测试使用的归档一致。

报告还记录了更早候选版本上的两项软件工作流与隔离性测试（耗时 457.62 秒通过）。这些测试在 Schema 与标签变更后并未重跑，不能证明在精确 HEAD 上有七项原生测试全部通过。自定义图标渲染、发布者政策适用性、目录上架以及 Web 宿主支持均未被这些测试验证。

## 设计评审记录

一名新鲜上下文评审员对早期提案应用了 EGM 设计批评家（Design Critic）与实现就绪（Implementation Readiness）评估准则。本次修订将范围扩展至固定的 0.1.1 候选版本，并盘点了全部 55 个变更文件。早期评审不代表批准了扩展后的范围。针对本次修订的独立评审、发布者政策评审以及完整 EGM 协议仍待完成。
