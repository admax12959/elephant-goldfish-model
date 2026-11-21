<!--
Copyright 2026 Google LLC.
SPDX-License-Identifier: Apache-2.0
-->

> **原文路径**：`docs/cursor.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# Cursor 集成指南（实验性）

本适配器面向 Cursor 本地 Agent。安装、工作流与隔离性均尚未验证，暂勿将 Cursor 加入受支持宿主列表。详情请参阅[设计文档](designs/additional-harnesses.md)与 [Issue #145](https://github.com/google/elephant-goldfish-model/issues/145)。

## 接口与版本

选定的 CLI 参考版本为 `2026.10.01-e373342`（检查日期：2026-10-24）。其帮助信息列出了 `--plugin-dir`、`--model`、`--workspace` 和 `--output-format`。目前仅运行了帮助、版本检查及原生离线加载器探针，尚未运行任何模型或评审员。该版本仅是已检查版本，并非经验证的最低版本。

首次运行版本命令时在锁定的 macOS 钥匙串（keychain）处停止；通过启动器的单命令环境变量 `AGENT_CLI_CREDENTIAL_STORE=file` 完成了帮助与版本检查，未修改任何账号配置。

原生清单为 `.cursor-plugin/plugin.json`，目录描述文件 `.cursor-plugin/marketplace.json` 指向该清单。原生清单选择了共享的 `skills/`、位于 `.cursor-plugin/agents/` 的 Cursor 适配器以及位于 `.cursor-plugin/rules/` 的 Cursor 规则。显式指定智能体路径排除了原始共享专家角色。每个评审员均声明了 `readonly: true`、`permissionMode: readonly`、`model: inherit` 和 `is_background: false`。所选 CLI 的插件加载器读取的是 `permissionMode` 而非 `readonly`，而文档中记录的 `readonly` 字段保留给本地 Agent 界面使用。原生加载器探针确认全部八个评审员均注册为只读模式，但这并不证明运行时强制执行或前台行为。在未来的实时运行中，请显式选择模型，并从任务记录中记下其实际标识符。

参考来源：[插件参考](https://cursor.com/docs/reference/plugins)、[子智能体配置与上下文](https://cursor.com/docs/subagents) 以及 [本地插件安装](https://cursor.com/docs/plugins#test-plugins-locally)。这些来源描述的是宿主能力，而非 EGM 验证结果。

## 安装候选包

请使用干净且未安装其他 EGM 版本的 Cursor 配置档案。宿主策略必须允许本地导入。切勿禁用组织策略或覆盖现有插件包。

1. 在干净的 EGM 检出目录中切换至候选提交。
2. 在该检出目录中运行以下命令：

   ```sh
   git diff --exit-code HEAD
   egm_candidate="$HOME/.cursor/plugins/local/egm"
   mkdir -p "$(dirname "$egm_candidate")"
   mkdir "$egm_candidate" && {
     cp -R .cursor-plugin skills agents resources docs LICENSE "$egm_candidate/" &&
     git rev-parse HEAD > "$egm_candidate/.egm-source-sha"
   }
   ```

3. 若任一命令失败请立即停止，重试前先检查可能残留的部分副本。
4. 重新加载 Cursor 并在 Customize 中检查该插件。
5. 确认包含七个技能、八个 EGM 评审员以及一条 EGM 运行时规则。
6. 记录绝对包路径以及已安装文件的校验和。

该副本保留了所有共享运行时目录及其相对结构，排除了可移植根目录清单以显式选用 Cursor 格式，不包含指向外部检出目录的符号链接，且共享源文件保持原样未改动。

所检查的 CLI 还支持通过 `agent --plugin-dir /absolute/path/to/cursor-package` 加载临时候选包。该命令会启动智能体并可能消耗模型额度，请仅在获得实时测试明确授权后运行，且切勿使用强制模式或自动 MCP 审批。

## 调用工作流

在执行第一个技能之前，向本地 Agent 提供已安装的根目录：

```text
EGM_ROOT: /absolute/path/to/cursor-package
Read /absolute/path/to/cursor-package/docs/cursor.md.
Run /egm-brainstorm for the requested feature.
```

从 Cursor 的斜杠菜单中选择已安装的 EGM 技能。如果某个技能未显示，请读取其绝对路径 `skills/<directory>/SKILL.md` 并在父上下文中遵循其指令，切勿自造斜杠命令命名空间。对 `egm-draft`、`egm-review` 以及工作流子技能均采用相同流程。执行前必须确保适配器规则处于激活状态。

基于所选包根目录解析 `agents/`、`resources/` 和 `docs/`；基于对应技能目录解析该技能的 `references/`。切勿假定 Markdown 会展开 `${CURSOR_PLUGIN_ROOT}` 或 `${CLAUDE_PLUGIN_ROOT}`。针对 Claude 与 Codex 的宿主专属指令不会选中 Cursor 运行时。

将合成器的编排留在父级执行，由父级直接派发评审员。需要时请读取 `agents/synthesizer.md` 获取权威综合指令。切勿创建无限制的合成器子智能体。引导初始化（Bootstrap）子智能体委派不在本实验性适配器的覆盖范围内；如所请求的操作需要该能力，请中止执行。

每次评审均须启动对应已注册专家角色类型的新原生 Task。向其传递包根目录、专家角色、目标文档、范围及必需文档标准的绝对路径。仅传递显式的金鱼上下文，不含父级讨论或历史记录。仅当权威阶段明确要求时才包含先前报告。在进入下一阶段前收集每个子任务的完成结果，切勿复用评审员上下文。在派发下一阶段前写出每个已完成阶段的报告。第 8 步须向评审员传递现有 `zoom_out_report.md` 的绝对路径；若该报告缺失则中止。如果派发、必需读取或完成环节失败，请报告阶段不完整并中止，切勿用线程内模拟替代原生评审员。

父级将产物写入用户的工作目录。遵循用户提示词或指令文件中的 `EGM artifact directory: <path>`，将相对覆盖路径相对于工作目录解析并在必要时创建。切勿将报告写入已安装的插件目录中。严格保留权威审批暂停门禁。

## 验证门禁

使用 `pytest tests/unit/test_cursor.py` 或 `pytest tests/unit` 运行离线检查。这些测试会验证元数据、原生配置以及路径中包含空格和 Unicode 字符的重定位包体。在没有本地包体时，可选的原生检查会自动跳过。

在 macOS 或 Linux 上针对所检查的 arm64 CLI 包体运行原生加载器探针：

```sh
EGM_CURSOR_BUNDLE="$HOME/.local/share/cursor-agent/versions/2026.10.01-e373342/index.js" \
  pytest tests/unit/test_cursor.py
```

该探针按平台与包体 SHA-256 固定 Cursor CLI `2026.10.01-e373342`：

- macOS arm64：`f6bd8dece34b56431ee74f1ac827e032d054381085e7da7088e85abdd1008330`。
- Linux arm64：`90ec23826cd3089232b0b243b8954ac72fab0454dd86d7d71ddae29d39c75cd2`。

Linux 包体来自 Cursor 的[版本化下载包](https://downloads.cursor.com/lab/2026.10.01-e373342/linux/arm64/agent-cli-package.tar.gz)，其下载归档的 SHA-256 为 `785c5f6bf2a60eb1121e27ed8c14f5ee07ed1b5b6692324f2d9a997238245eb5`。这些哈希仅用于标识所检查的文件，并非厂商签名。探针直接调用已安装的插件加载器而不经过 CLI 入口点，并在探针内部阻断网络连接和子进程启动，既不修补已安装的 CLI，也不启动模型会话。该私有加载器导出项与特定版本绑定，修改固定版本前须重新审计。

探针在重定位包体中发现了七个技能、七个只读评审员注册项以及一条激活规则。最初仅设置 `readonly: true` 时注册的是默认权限，补充 `permissionMode` 字段后修正了该结果。这些仅是原生解析结果，并非交互式安装或运行时证据。本历史记录未标识所测试的 EGM 提交或已安装包的校验和；在将其用于某个固定候选版本前，请针对该候选版本重新运行发现检查。

在进行实时运行前，须获得明确的模型调用批准。使用干净的工作区和不含无关 MCP 服务或记忆的配置档案。记录候选提交 SHA、包体校验和、宿主版本、交互面、模型及日期，并将以下三项结果分别记录：

| 门禁 | 所需证据 | 当前结果 |
| --- | --- | --- |
| 安装 | 交互式发现，以及对已安装专家角色、文档标准与指南的实际读取 | 未验证；离线原生发现已通过 |
| 工作流 | brainstorm、draft、review、产物文件以及真实的人类审批暂停 | 未验证 |
| 隔离性 | 新鲜子任务记录、必需读取记录、只读写入拒绝以及所有评审员完成记录 | 未验证 |

验证隔离性时，仅在父对话中放置一个随机标记，并检查子任务的输入、工具记录及其回答。针对一次性测试夹具探测写入拒绝，并验证其字节内容保持不变。原生只读模式仅限制写入，并非文件系统边界。提示词虽也禁止使用 Shell 和 MCP，但这些并非经证明的工具白名单。请分别评估注入的规则、仓库指令、记忆、MCP 访问以及共享文件；一个全新的任务并不会自动移除这些潜在的信息来源。

请记录原生评审员的启动与完成情况，而非仅记录父级的摘要。缺少记录会导致对应门禁保持未验证状态。对于 CLI `2026.10.01-e373342`，`tests/integration/cursor_child_reads.py` 可检查单个子任务的 SQLite 存储库，以核对匹配的 `ReadFile` 调用与成功结果：

```sh
python tests/integration/cursor_child_reads.py /absolute/path/to/child/store.db \
  --required /absolute/path/to/persona.md
```

根据匹配的父 Task 完成记录中的子任务 ID 选择对应的存储库。待子任务停止后，使用 SQLite 的备份 API 创建一个已关闭的快照。该命令会拒绝带有 WAL 文件或主文件仍在变化的快照；若某个必需路径缺少匹配的非空成功读取记录，命令将以非零状态退出。存储库与输出可能包含私密路径，请妥善保密。该命令仅检查文件读取，不验证派发、审批、阶段顺序、隔离性或产物。存储库 schema 属于私有且与版本强绑定；它能统计不透明的二进制大对象（blob），但无法解析其内容。目前未提供完整的 Cursor 工作流追踪解析器；在采纳任何解析器前，必须测试缺失启动记录、缺失完成记录、畸形记录以及 Unicode 行分隔符，并仅按字面 LF 拆分 JSONL。现有的 Claude 和 Codex 追踪测试不适用于验证 Cursor 证据。

## Grok Bot 评估

Grok Bot 属于独立的未验证交互面，当前不包含任何 Grok Bot 适配器。其[工作指南](https://cursor.com/docs/grok-bot/work)描述了持久保留记忆（retained memory）和共享计算机机制，这些特性可能会将先前评审发现暴露给另一位评审员。本地 Cursor 测试结果不能证明 Grok Bot 兼容性。

| 要求 | 评估 |
| --- | --- |
| 完整包导入与资源读取 | 未测试 |
| 新鲜评审员上下文 | 未测试；持久记忆需要单独探针验证 |
| 只读强制执行 | 未测试 |
| 原生启动与完成记录 | 未测试 |
| 人类审批暂停 | 未测试 |
| 记忆与共享文件中的先前发现 | 未测试；需分别独立检查 |

请先完成上述检查，再另行提议任何 Grok Bot 实现方案。

## 移除候选包

在 Cursor 中禁用本地 EGM 插件，仅删除你创建的候选文件夹，然后重新加载 Cursor。保留工作区产物与验证证据。
