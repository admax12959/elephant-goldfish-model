> **原文路径**：`TESTING.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# EGM 测试手册

本文档提供一份分步开发者操作手册，用于在 EGM 技能套件中运行单元测试。

---

## 环境准备

请安装 Node.js 22 或更高版本，用于运行离线范围提示工作流测试（scope-advisory workflow tests）。这些测试会针对模拟的 GitHub API 执行工作流的 JavaScript 逻辑。

### A. Python 环境配置
创建并激活一个隔离的 Python 3.10+ 虚拟环境：

```bash
# 进入仓库根目录
cd /path/to/elephant-goldfish-model

# 检查 Python 版本（需 3.10 或更高版本）
python3 --version

# 创建虚拟环境
python3 -m venv .venv

# 激活虚拟环境
source .venv/bin/activate

# 依赖组（Dependency groups）需要 pip 25.1 或更高版本
python -m pip install --upgrade "pip>=25.1"
python -m pip install --group test
```

如果你的 Python 版本低于 3.10，请从 [python.org/downloads](https://www.python.org/downloads/) 或系统包管理器安装较新版本。打开一个新终端并在继续前验证更新后的版本。如果 `python3` 仍指向旧版本，请显式使用新版解释器（例如 `python3.12 -m venv .venv`）。

如果你此前使用旧版 Python 创建过 `.venv`，请将其删除并用更新后的版本重新创建。

在 Python 3.9 或更早版本中，`pytest` 会在测试收集阶段因 `TypeError: unsupported operand type(s) for |` 而中止。

---

## 单元测试

单元测试完全离线运行，无需网络访问、API 密钥或模型依赖。它们校验 YAML Frontmatter Schema 与相对链接完整性、可移植插件清单（portable plugin manifest）、共享元数据一致性以及插件市场/组件路径。

读取钩子（read-hook）测试需要 POSIX `sh`，工作流门禁（workflow-gate）测试需要 `PATH` 中存在 `bash` 以执行工作流的实际 Shell 步骤。在 Windows 上，请使用 Git for Windows 并在 Git Bash 中运行测试，或者将其 `bin` 目录（包含 `sh.exe`）添加到当前 PowerShell 会话的 `PATH` 中。使用 `python -m venv .venv` 创建环境，然后在 PowerShell 中执行 `.venv\Scripts\Activate.ps1` 或在 Git Bash 中执行 `source .venv/Scripts/activate` 激活，再按上述说明安装测试依赖。
即使启用了 Git 的 `core.autocrlf`，Shell 脚本也会通过 `.gitattributes` 强制保持 LF 换行符。CI 会在 Ubuntu 和 Windows 上运行这套离线套件；但这并不等同于确立对原生宿主工作流的支持。

`CI` 检查会等待两个单元测试作业完成；只要任一校验失败、被取消或被跳过，即判定失败。包含站点校验的工作流也必须将其列为 `CI` 的依赖项。同一 Pull Request 的新推送会取消先前的运行，而推送到 `main` 分支的运行则保持相互独立。

```bash
# 运行全部单元测试
pytest tests/unit

# 以详细输出模式运行
pytest tests/unit -v

# 运行指定的单元测试模块
pytest tests/unit/test_frontmatter.py -v
pytest tests/unit/test_link_integrity.py -v
```

---

## 代码检查（Lint）

CI 使用 `ruff` 对 Python 进行静态检查，使用 `actionlint` 检查 GitHub Actions 工作流。在本地同时运行两项检查：

```bash
python -m pip install --group lint && ruff check .
actionlint
```

---

## 原生 Claude 评审员探针（可选）

在已认证的 Claude Code 2.1.271+ 安装环境下，运行：

```bash
EGM_CLAUDE_LIVE=1 pytest tests/integration/test_claude.py
```

该测试会消耗账号的模型配额。它会构建 Claude 运行时并为其分配唯一的测试名称。两个探针会在目录级读取权限且无审批钩子的条件下调用评审技能，检查原生评审员是否只读取自身智能体人格、看不到父会话或项目 `CLAUDE.md` 上下文，以及在显式拒绝规则拦截其人格文件时是否会不作裁决而直接停止。这是一项聚焦型探针测试，并非完整评审工作流测试或插件市场安装测试。
详见 [Claude 集成指南](docs/claude.md)。

---

## 原生 Codex 集成测试（可选）

可选的集成测试通过临时 Codex 插件市场运行线上候选版本：

```sh
EGM_CODEX_LIVE=1 pytest tests/integration/test_codex.py -v
```

这些测试验证候选版本安装、零上下文（新鲜上下文）评审员隔离性以及工作流产物。每项测试均在隔离沙箱中使用已认证的 Codex CLI 运行。
详见 [Codex 集成指南](docs/codex.md)。

若要测试带标签的 OpenAI 发布压缩包而非当前检出代码，请从同一个 GitHub Release 下载压缩包与 `SHA256SUMS`，校验哈希值后传入压缩包路径运行同一套测试：

```sh
shasum -a 256 -c --ignore-missing SHA256SUMS
EGM_CODEX_RELEASE_ZIP=/absolute/path/to/egm-openai-0.1.0.zip \
  EGM_CODEX_LIVE=1 pytest tests/integration/test_codex.py -v
```

测试会围绕压缩包创建一个临时本地插件市场，逐字节校验每个已安装的归档文件，并在结束后清理临时插件与市场。该测试会消耗已认证的 Codex 模型用量。请分别记录发布标签、源码提交哈希、压缩包校验和、宿主与模型版本以及每项检查结果。工作流失败或缺少原生证据均不计为通过。

---

## 宿主环境测试记录

某个宿主环境只有在通过其插件安装并端到端跑通 `/egm-brainstorm`、`/egm-draft` 和 `/egm-review` 后，才计为受支持（参见[宿主环境指南](docs/harnesses.md)）。每次运行请新增一行记录。

| 宿主环境 | 版本 | 日期 | 安装 | Brainstorm | Draft | Review | 备注 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Antigravity CLI | 1.4.5 | 2026-10-22 | 通过；存在校验器警告 | 通过（测试夹具） | 通过（测试夹具） | 通过（测试夹具） | 使用临时 HOME，无钩子、无 MCP 服务器、无其他插件。父轮次隔离探针在原生子转录本上通过；未测试指令文件排除。Issue #237 报告了 CLI 1.4.0 上存在项目 `AGENTS.md` 和 `GEMINI.md` 注入问题。共运行 16 个原生评审员。阶段 5 至阶段 9 按时间戳顺序执行。所有必需读取均已发生。存在 5 份阶段报告。未编写任何代码。最终设计与草稿仅相差一行 `Status`。未测试项：交互式审批、`egm-bootstrap` 以及合成器嵌套。`egm-bootstrap` 技能超出了加载体积限制。 |
| Claude Code | 2.1.308 | 2026-10-22 | 通过（`--plugin-dir`） | 通过（测试夹具） | 通过（测试夹具） | 修复后通过（测试夹具） | 隔离探针通过。保留了初始阶段顺序失败记录；聚焦的 9 评审员重跑验证了顺序阶段执行与必需读取。进程关闭及父进程无 Shell 夹具限制保留在完整报告中。 |
| Claude Cowork | Opus 5.5 Medium，云端连接任务 | 2026-10-22 | 通过（ZIP） | 通过直接派发恢复完成 | 已完成 | 部分完成：阶段 5–7 | 用量上限中断了阶段 8。最终产物、阶段 9 以及实际隔离型子智能体输出仍未验证；见 #105。更早期的完整运行记录另行报告。 |
| OpenAI Codex | 0.163.0 | 2026-10-22 | 通过 | 通过（测试夹具） | 通过（测试夹具） | 通过（测试夹具） | 两项线上测试在 573.13 秒内通过；已验证父轮次隔离、15 个原生评审员、有序阶段及必需读取。未测试指令文件排除。Issue #237 报告了 CLI 0.166.0 上存在项目与用户级 `AGENTS.md` 注入问题。 |
| OpenAI Codex CLI（v0.1.0 OpenAI ZIP） | 0.164.0-alpha.11.1 | 2026-10-29 | 通过；已安装文件与 ZIP 一致 | 通过（测试夹具） | 通过（测试夹具） | 通过（测试夹具） | `gpt-6.1-sol`, medium。完整可选线上套件于 PR 提交 `8bd11c5` 在 482.30 秒内通过；源码 `b3a35956759bf7f024af2ff061b2e0b44b438e4c`；ZIP SHA-256 `5f17b3942eaccc657621dfa550ef9fcaf626ad7d170085a9ef98a0e9e8afcefb`。无路径工作流读取了全部三个已安装的 `SKILL.md` 文件并返回了 `docs/codex.md` 的内容；生成了所有要求的产物并完成了 15 个原生评审员执行。7 个离线回归用例覆盖了成功/失败读取、字面复合 `cat` 以及对 `&&` 和管道命令的保守拒绝。详见 [PR #217 中的完整运行报告](https://github.com/google/elephant-goldfish-model/pull/217)。 |
| ChatGPT Work | GPT-6.1 Sol, Light | 2026-10-22 | 通过；已验证安装哈希 | 已完成 | 已完成 | 9 个原生评审员均已完成 | 新鲜上下文探针返回 `UNKNOWN`。产物下载/最终审计以及完整的子智能体工具轨迹仍待补全；见 #106。 |
| omp | v18.12.6 | 2026-10-27 | 通过（社区安装） | 未运行 | 未运行 | 未运行 | 社区验证，非夹具运行：全部 7 项技能在新鲜无头会话中均可通过 `skill://` 发现并解析；全部 9 个评审员智能体人格均注册为原生 `task` 智能体。针对隐藏技能的 TUI `/skill:` 自动补全未验证；端到端夹具未运行。所需清单修复：连字符命名的技能目录（omp 校验器拒绝下划线目录）以及移除 `plugin.json` 中的 `$schema` 键（omp 严格的 agent-plugins 校验拒绝 `disable-model-invocation`）；见 #186。 |
| Muse | 1.4.14 | 2026-10-30 | 通过（本地路径安装 + 批准） | 通过（测试夹具） | 通过（测试夹具） | 通过（测试夹具） | 从无符号链接的工作树副本本地安装：插件通过 Claude 兼容清单校验有效；7/7 技能有效、零诊断错误且仅支持用户主动调用激活；9/9 智能体在批准后处于 `trusted_enabled` 状态；9 项工具投影不受支持（评审员未受工具限制）；29 条校验警告、0 条错误。添加插件市场仅发现 0 个插件（受 Codex 目录遮蔽影响）。端到端夹具（行数统计设计，`muse-wonder-1.0`，medium，在临时工作区运行无头 `muse exec`，仅使用已安装缓存；因尚无技能路由至该处，提示词直接指定了 `docs/muse.md`，见 [#252](https://github.com/google/elephant-goldfish-model/issues/252)；审批预先模拟，未测试交互式审批门禁）：头脑风暴、起草与不中断评审均已完成；共完成 15 个新鲜上下文评审员（6 个头脑风暴 + 阶段 5–9 的 9 个阶段评审员），阶段报告按时间戳顺序生成，最终设计与被评审草稿完全一致，未编写任何实现代码。隔离探针通过：新鲜子智能体对仅父会话可见的标记返回了 `UNKNOWN`。该运行时不支持派生具名评审员智能体（派生时因 `agent_definition_unavailable` 被拒），因此每位评审员均使用了文档记录的通用回退方案，先读取已安装的人格文件。针对夹具设计的评审发现属于设计问题，而非工作流执行故障。测试提交为 `91407cf`（技能、智能体与清单与夹具源码一致；指南与记录措辞在运行后有所扩展）。 |

### 解读原生测试结果

完整运行报告请保留在 [PR 描述](https://github.com/google/elephant-goldfish-model/pull/103)中。
上表仅为摘要。请在每份报告中记录被测提交、客户端版本、执行命令、原生证据与已知限制。解析 JSONL 时请严格按字面 LF（`\n`）切分；Python 的 `splitlines()` 还会误切分 JSON 字符串内部的合法 Unicode 字符。

云端验证缺口追踪见 [#105（Cowork）](https://github.com/google/elephant-goldfish-model/issues/105) 与 [#106（ChatGPT Work）](https://github.com/google/elephant-goldfish-model/issues/106)。

- 检查原生工具轨迹，核实评审员启动、完成情况以及预期的资源读取。
- 除产物是否存在外，还应验证阶段顺序执行情况以及对应的人格视角。
- 区分测试夹具本身的设计发现与评审工作流的执行故障。
- 将安装结果、执行结果与转录本隔离结果分开记录。

### 离线证据检查

运行 `pytest tests/unit/test_claude_traces.py` 以使用合成记录演练 Claude 探针的原生子智能体验证器。仅有人格读取请求并不足以证明读取成功：必须存在匹配的、无错误、非空的原生工具返回结果，且包含非空白文本（以字符串或原生文本块形式呈现）。即使子智能体回答了 `UNKNOWN`，缺失或无关的结果也会判定失败。这些测试不会调用模型，也不代表云端支持。此外，单凭成功的工具结果本身也不能证明整个资源已被完整读取。

对于 Cowork 与 ChatGPT Work，请为每位评审员单独保留以下证据：原生启动参数、规范的人格/目标/标准文件路径、实际读取结果与覆盖范围、完成状态以及阶段时间戳。子智能体自行写入的回执和父智能体的摘要不能替代缺失的工具记录。检查实际隔离型子智能体的回答；将宿主附加的上下文与父轮次隔离分开测试。含账号信息的原始记录请保持私密，仅公开脱敏后的发现与候选版本标识符。如果宿主平台无法暴露所需证据，请记录该边界并将受影响的检查项保持未关闭状态。
