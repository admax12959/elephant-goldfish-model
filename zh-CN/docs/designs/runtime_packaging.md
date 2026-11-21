> **原文路径**：`docs/designs/runtime_packaging.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 运行时打包

状态：已通过合并的 #136 实现；原生宿主验证证据仍独立维护。
作者：@n3wth。日期：2026-10-24。

关联 Issue：#121 与 #122。实现 PR：#136。

## 问题与范围

用户需要仅包含各宿主环境所需资源的运行时产物包。完整的仓库归档包还会混入开发文件与官网资产。

本设计从同一棵源码树构建 Claude 与 OpenAI 运行时产物包，同时保留正常的用户可控权限。不新增宿主环境、不新增外部服务，也不添加自动权限审批。产物包校验并不等同于证明目录上架审核通过或运行时兼容性。

## 打包契约

使用在 #136 中合并的 `scripts/pack_plugin.py` 作为唯一构建器。`--target claude|openai` 用于选择目标宿主环境，`--output` 用于指定新的输出目标路径。构建产出为一个 `egm-TARGET` 目录和一个 `egm-TARGET.zip` 归档文件。

通过白名单（allowlist）纳入清单文件（manifests）、技能、参考文档、智能体人格、原生适配器、必需文档、许可证以及图像资产。纳入 `docs/assets` 下的 SVG 与 PNG 文件以供 README 图片使用。排除官网内容、测试、工作流、钩子（hooks）、开发用市场元数据以及无关文件。

目标文件清单明确如下：

- 共享文件：`plugin.json`、`LICENSE`、`README.md`、`skills/*/SKILL.md`、`skills/*/agents/openai.yaml`、技能资产与参考文件、`agents/*.md`、`resources/*.md` 以及 `resources/document_standards/*.md`。
- 共享指南：`docs/index.md`、`docs/getting_started.md`、`docs/harnesses.md`、`docs/claude.md`、`docs/codex.md`、`docs/faq.md` 以及 `docs/privacy.md`。同时纳入发布准备流程新增的 `docs/releases.md`。
- 共享图像：`assets/` 与 `docs/assets/` 下的 SVG 与 PNG 文件。
- Claude 专属：`.claude-plugin/plugin.json`、`.claude-plugin/icon.png` 以及 `.claude-plugin/agents/*.md`。
- OpenAI 专属：`.codex-plugin/plugin.json`。

构建器中的 `SHARED`、`HOST` 和 `REQUIRED_SHARED` 常量实现了这份清单。必需的引用文件必须存在；可选文件必须匹配这些路径模式。

在运行时包中保留安装指南、宿主指令、智能体人格、技能与评审标准。排除仓库设计文档、`TESTING.md`、`CONTRIBUTING.md`、`AI_POLICY.md` 以及 PR 模板。在打包的指南中通过 GitHub 链接指向这些开发文档。阅读开发文档需要网络访问，而执行打包后的技能不需要网络。在两个生成的运行时包中均检查相对 Markdown 链接的有效性。仅涉及开发文档的修改不得要求重新生成运行时内容。

拒绝符号链接、超出源码树范围的路径、缺失的必需文件、未打包或非字符串的清单引用、钩子（hooks）以及不受支持的 MCP 或应用依赖。拒绝写入已存在的输出目录或归档文件。失败的构建可能会留下不完整的输出；应排查失败原因，并在下次尝试时使用新的输出路径。

对归档条目进行排序，并使用固定的时间戳与文件权限模式。在相同构建环境下验证字节级完全一致。不对未经测试的工具链版本宣称可复现性。

不要将生成的 Claude 运行时提交到 `main`。Claude 市场目录（marketplace catalog）从仓库根目录安装。官方目录发布则使用生成的发布分支，参见[Claude 发布分支分发](claude_release_branch_distribution.md)。

## 权限与宿主边界

移除自动权限审批钩子。在 Claude 原生适配器上保留受限的工具列表。资源读取使用正常的宿主权限。被拒绝的读取操作必须中止受影响的评审员，绝不能捏造评审结论。

只读 Claude 评审员允许使用 `Read`、`Grep` 和 `Glob`。合成器（`egm-synthesizer`）额外允许 `Agent` 和 `Write` 以执行委派与报告写入。Bootstrap 允许 `Agent`、`Bash`、`Read`、`Grep`、`Glob`、`Write` 和 `Edit` 以生成文档。这些工具列表并不构成文件系统或网络沙箱。

将父上下文隔离性与文件系统访问、宿主附加指令或记忆功能分开测试。Claude CLI 探针结果不能代表 Cowork 的行为；OpenAI 打包结果也不能代表支持所有 ChatGPT 模式。

## 验证与发布

- 测试必需内容、图像纳入、排除规则、不安全路径、缺失引用、覆盖拒绝以及可复现性。
- 针对生成的运行时验证允许读取与拒绝读取资源的行为。
- 记录源码 SHA、已安装产物的身份标识、宿主/模型版本，并分别记录安装、工作流与隔离性结果。
- 在 [Cowork #105](https://github.com/google/elephant-goldfish-model/issues/105) 与 [ChatGPT Work #106](https://github.com/google/elephant-goldfish-model/issues/106) 中保留云端工作流与隔离性缺口记录。
- 设计与实现 #136 已合并；保留针对精确产物的宿主检查。
- 通过[版本发布设计](releases.md)协调带标签的发布产物。

#121 与 #122 中的目录上架发布仍独立推进。提交前须确认所属账号、政策链接、发布者权限以及平台方反馈。仅在验证已上架的目录条目后才关闭这两个 Issue。

## 备选方案与待决事项

分别手工维护各个产物包会导致漂移。完整仓库 ZIP 会夹带不必要的开发内容。共享白名单使产物包边界保持明确且可评审。

打包实现 #136 已合并。与该实现独立的事项包括 #121/#122 中的目录审批以及针对精确产物的宿主验证。
