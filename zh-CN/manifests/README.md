# 多宿主清单体系（Manifests & Thin Adapters）架构指南

> **原文路径**：`plugin.json`、`pyproject.toml`、`.claude-plugin/`、`.codex-plugin/`、`.cursor-plugin/`、`.plugin/`、`.windsurf/`、`.agents/` | 本文档为中文架构导读与零到一扩展实战手册，保留英文原文件以便对照。
>
> 💡 **第一性原理提示**：AI 编程宿主环境（Harness）的插件规范目前高度碎片化——Claude Code、Codex、Cursor、Windsurf、OpenCode、Antigravity 各有各的目录约定、权限字段与变量注入方式。如果为每个宿主复制一份完整的技能与人格文本，不出两周就会发生灾难性的**文档漂移（Drift）**。EGM 的解法是：**单一事实来源（Single Source of Truth, SSOT）+ 薄适配器（Thin Adapters）**——所有核心智慧只写一遍，各宿主目录只负责“路由指针 + 工具/上下文权限锁”。

---

## 1. 核心架构：单一事实来源 + 薄适配器

```mermaid
flowchart TB
    subgraph SSOT["单一事实来源 (Single Source of Truth)"]
        S1["skills/*/SKILL.md<br/>8 大规范工作流 + references/ + assets/"]
        S2["agents/*.md<br/>10 个规范智能体人格"]
        S3["resources/document_standards/*.md<br/>5 份质量评估准则"]
        S4["plugin.json<br/>可移植根元数据 + 版本权威基准"]
    end

    subgraph THIN["薄适配器与清单层 (Thin Adapters &amp; Host Manifests)"]
        H1[".claude-plugin/<br/>plugin.json + marketplace.json + 10 个薄 agent 壳"]
        H2[".codex-plugin/ &amp; .agents/<br/>plugin.json + marketplace.json"]
        H3[".cursor-plugin/<br/>plugin.json + marketplace.json + 8 个只读 agent 壳 + rules/egm.mdc"]
        H4[".plugin/<br/>agent-plugins.org 标准最小清单"]
        H5[".windsurf/<br/>rules/egm.md + 8 个零拷贝路由 workflow"]
        H6["adapters/opencode/install.py<br/>动态生成独立 opencode.json (见 adapters 指南)"]
    end

    H1 -->|${CLAUDE_PLUGIN_ROOT}| SSOT
    H2 -->|./skills/| SSOT
    H3 -->|EGM_ROOT 任务参数| SSOT
    H4 -->|共享根包| SSOT
    H5 -->|工作区根相对路径| SSOT
    H6 -->|静态快照 bundle/| SSOT
```

**薄适配器的铁律**：
1. **零正文复制**：宿主目录中的 Agent 壳或 Workflow 文件绝不拷贝 `agents/*.md` 或 `skills/*/SKILL.md` 的业务指令，只写 10~20 行“先读规范文件 + 本宿主权限约束 + 失效即阻断（Fail-Closed）兜底”。
2. **权限下沉到宿主原语**：真正锁死金鱼（零上下文评审员）只读且隔离的，是各宿主原生的声明字段（如 Claude 的 `tools: [Read, Grep, Glob]` + `omitClaudeMd: true`，Cursor 的 `readonly: true` + `permissionMode: readonly`），而不是单靠口头 Prompt 祈祷。
3. **版本强同步**：5 份发行清单（`plugin.json`、`.claude-plugin/plugin.json`、`.codex-plugin/plugin.json`、`.cursor-plugin/plugin.json`、`.plugin/plugin.json`）的 `version` 字段必须完全一致，并由 `scripts/prepare_release.py` 和 `tests/unit/test_harnesses.py` 强制校验。

---

## 2. 仓库根清单与工程配置

### 2.1 `plugin.json`（可移植根清单）
位于仓库根目录，定义全仓最核心的跨平台身份元数据（`name: "egm"`、`displayName`、`version: "0.1.1"`、`logo: "assets/icon.svg"`、`license: "Apache-2.0"`）。
- 同时通过 `extensions["com.openai"]` 携带 OpenAI/Codex 生态扩展字段（`requires_local_executor: true`、品牌色 `#1B6EF3`、默认引导 Prompt、商店图标 `./assets/icon.svg` 等），供 `scripts/pack_plugin.py` 交叉校验。

### 2.2 `pyproject.toml`（Python 工程与校验底座）
EGM 的插件运行时本身不需要安装 Python 包，但仓库的测试与代码规范依赖 `pyproject.toml` 统一管理：
- **`[dependency-groups]`**：
  - `lint`：锁定 `ruff==0.17.4`（规则 `E`, `F`, `I`，忽略 `E501`，按 Google 风格排序导入，排除 `site`、`distributions`、`.claude`）。
  - `test`：锁定 `pytest==9.0.3` 与 `pyyaml==6.0.3`。
- **`[tool.pytest.ini_options]`**：默认仅扫描 `tests/unit`（`addopts = "-ra -q --import-mode=importlib"`），确保离线单测快速纯净，不误触发需要真实宿主的 `tests/integration`。

---

## 3. 六大宿主清单目录逐一拆解

| 宿主目录 | 包含文件 | 宿主发现与路由机制 | 金鱼隔离与只读锁实现 | 路径解析机制 |
| :--- | :--- | :--- | :--- | :--- |
| **`.claude-plugin/`** | `plugin.json`、`marketplace.json`、`icon.png`、`agents/*.md`（10 个） | Claude Code 通过 `marketplace.json`（`source: "./"`）发现插件，按 `plugin.json` 的 `agents` 数组加载全部 10 个子智能体 | Frontmatter 声明 `tools: [Read, Grep, Glob]`、`omitClaudeMd: true`（剥离项目 CLAUDE.md 防上下文污染） | 使用宿主变量 `${CLAUDE_PLUGIN_ROOT}` 定位 `agents/*.md` 与 `resources/` |
| **`.codex-plugin/`** | `plugin.json` | 声明 `"skills": "./skills/"` 与 `"requires_local_executor": true`，配置 Codex 商店 `interface` 与图标 | 依赖本地执行器与隔离的子智能体派生；同时在 `skills/*/agents/openai.yaml` 提供技能级元数据 | 相对包根目录 `./skills/` 与 `./assets/icon.svg` |
| **`.agents/`** | `plugins/marketplace.json` | 面向 Codex / 通用 Agent 生态的插件市场索引（`source: "./"`、`policy.installation: "AVAILABLE"`、`authentication: "ON_INSTALL"`） | 指向仓库根共享包，由具体宿主清单接管权限 | 相对路径 `"./"` |
| **`.cursor-plugin/`** | `plugin.json`、`marketplace.json`、`agents/*.md`（8 个评审员）、`rules/egm.mdc`、`assets/` | `plugin.json` 指向 `./skills/`、`./.cursor-plugin/agents/`、`./.cursor-plugin/rules/` | 8 个薄智能体壳声明 `readonly: true`、`permissionMode: readonly`、`is_background: false`；`rules/egm.mdc` 强制直派评审员、禁止复用会话、遇阻即停（严禁父会话代演金鱼） | **不依赖 Markdown 变量替换**：由 `rules/egm.mdc` 指示父智能体从 `SKILL.md` 路径向上回溯三级确定绝对路径 `EGM_ROOT`，作为任务文本参数传给子智能体 |
| **`.plugin/`** | `plugin.json` | 遵循开放规范 `https://agent-plugins.org/schemas/1.0.0/plugin.schema.json` 的最小标准清单（如 Copilot 等兼容生态） | 与根包共享技能与人格定义，具体隔离规则见对应文档与适配器测试 | 相对仓库根 |
| **`.windsurf/`** | `rules/egm.md`、`workflows/egm-*.md`（8 个） | `rules/egm.md`（`trigger: model_decision`）作为全局路由契约；8 个 `workflows/` 文件作为轻量斜杠入口 | 要求使用宿主只读子智能体配置（如 Devin CLI 中的 `subagent_explore`）；**若宿主无法派生新鲜上下文只读子智能体，立即中止并报告“同上下文评审不是金鱼评审”** | 要求工作区根目录检出，直接引用根 `skills/<name>/SKILL.md` |

### 关键对比：薄适配器到底长什么样？

以 `design_critic` 金鱼评审员为例，对比 Claude 与 Cursor 的薄壳写法，可以直观感受到 EGM 的克制与精准：

- **Claude 薄壳（`.claude-plugin/agents/design_critic.md`，仅 18 行）**：
  - Frontmatter 锁死 `tools: [Read, Grep, Glob]` 与 `omitClaudeMd: true`。
  - 正文仅 7 行：指示先读 `${CLAUDE_PLUGIN_ROOT}/agents/design_critic.md`，其余标准相对 `${CLAUDE_PLUGIN_ROOT}` 解析，保持只读，读不到必需文件时报告不完整评审而非放行。
- **Cursor 薄壳（`.cursor-plugin/agents/design_critic.md`，仅 19 行）**：
  - Frontmatter 锁死 `readonly: true`、`permissionMode: readonly`、`is_background: false`。
  - 正文要求任务中必须提供绝对路径 `EGM_ROOT`（明确注明它是一个任务文本值而非环境变量），先读 `EGM_ROOT/agents/design_critic.md`，禁止编辑、Shell、MCP 和嵌套委派。
- **Windsurf 路由（`.windsurf/workflows/egm-review.md`，仅 10 行）**：
  - 只有 5 条编号指令：读根 `skills/egm-review/SKILL.md` → 执行 → 解析 `agents/` 与 `resources/` → 强制核验新鲜上下文只读子智能体能力 → 仅允许父会话在人类批准后写文件。

---

## 4. 从 0 到 1 接入一个全新宿主环境（New Harness Playbook）

如果你想为一款新的 AI 编程客户端（假设叫 `FooCode`）增加 EGM 支持，请严格按照以下 7 步工程路径推进：

### 第 1 步：确认贡献轨道与前置设计
- 根据 `AGENTS.md` 的分类规范，新增宿主通常涉及 `>= 3` 个核心文件或 `> 100` 行变更，属于 **Feature / Architectural Change**：
  1. 先提交 Stage 1 设计文档 PR（`docs/designs/<harness>.md` 并更新 `docs/designs/README.md`），明确 **Goals / Non-Goals**、该宿主的隔离能力边界、以及能否满足金鱼零上下文铁律。
  2. 设计对齐后再推进 Stage 2 实现 PR。

### 第 2 步：调研宿主隔离原语（最关键的技术决策）
在写一行配置之前，先核实新宿主是否支持：
1. **独立的子智能体线程**（不继承父会话历史，且能禁止会话复用/恢复）。
2. **声明式只读权限锁**（能否关闭文件写入、Shell 执行、MCP 写工具，以及是否需要像 OpenCode 那样防范 `grep` 绕过 `.env` 规则）。
3. **路径解析机制**（宿主是否注入类似 `${PLUGIN_ROOT}` 的变量？如果不支持，则参照 Cursor 的 `EGM_ROOT` 传递模式或 OpenCode 的静态 `bundle/` 方案）。
> 若宿主完全不支持隔离只读子智能体，必须在规则层采用 Windsurf 的 **Fail-Closed 策略**：允许单会话技能运行，但遇到金鱼蜂群工作流时明确拒绝降级为同上下文评审。

### 第 3 步：创建薄清单与薄适配器
- 在仓库根创建 `.<harness>-plugin/plugin.json`（或对应宿主约定的目录，如 `adapters/<harness>/`）：
  - `name`、`version`、`description`、`license`、`homepage`、`repository` 必须与根 `plugin.json` 严格一致。
  - 为需要注册的评审员编写薄壳文件，仅声明宿主权限锁 + 指向规范 `agents/*.md` 的路径，**严禁复制规范文本**。
  - 若宿主支持市场发现，补充对应的 `marketplace.json`（`source: "./"`）。

### 第 4 步：接入打包与版本同步检查
- 若新宿主需要分发独立 ZIP 运行时包：
  1. 在 `scripts/pack_plugin.py` 的 `HOST` 白名单字典中登记该宿主的专属文件模式与资产引用检查。
  2. 在 `scripts/prepare_release.py` 的 `HOST_MANIFESTS` 中加入新清单路径，确保发布时版本号强同步。
- 确保不引入任何 `hooks`、`mcpServers`、`mcp` 或 `apps` 依赖（否则会被打包安全防线拦截）。

### 第 5 步：编写离线契约单元测试与轨迹校验器
- 在 `tests/unit/test_<harness>.py` 中锁死离线不变量：
  - 清单字段与根 `plugin.json` 一致性。
  - 所有评审员薄壳均为只读且不拷贝规范正文。
  - 重定位/打包后所有规范资源与路径引用完整闭环。
- 如果宿主支持导出子会话轨迹，参照 `tests/integration/opencode_trace_verifier.py` 编写 **Fail-Closed 轨迹校验器**，并在 `tests/unit/` 中配套对抗性单测（覆盖缺失读取、会话复用、冲突错误、路径逃逸等场景）。

### 第 6 步：补充宿主指南与支持矩阵文档
- 新增 `docs/<harness>.md`，说明安装步骤、支持版本、已知限制与验证方式。
- 在 `docs/harnesses.md` 的支持矩阵中登记该宿主的状态（如 Experimental / Supported）。

### 第 7 步：收集实机宿主证据（Harness Evidence）
- 按照 `AGENTS.md` 要求，在 PR 的 **Verification** 区块如实记录：
  - 宿主名称与精确版本号
  - 安装结果
  - `egm-brainstorm`、`egm-draft`、`egm-review` 的实测证据（未运行的检查须明确标注 `not run`）。
