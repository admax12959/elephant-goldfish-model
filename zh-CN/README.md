# 大象-金鱼模型（Elephant-Goldfish Model）中文学习宝库总索引

> 本目录 (`zh-CN/`) 是 `elephant-goldfish-model` 全仓库的**中文镜像翻译与深度学习导航中心**。
> 所有英文原文件均保持原样未修改，方便你随时中英对照、同步上游更新，且 100% 兼容原仓库所有自动化测试。

---

## 🌟 从这里开始（强烈推荐）

1. **[《从 0 到 1 第一性原理深度学习手册》](LEARNING_GUIDE.md)** —— 必读！彻底讲透大象-金鱼模型的第一性原理、四层正交架构、Prompt 工程 7 条黄金法则，以及手把手带你从 0 到 1 打造同类系统的完整蓝图。
2. **[《统一中英文术语表（Glossary）》](GLOSSARY.md)** —— 36+ 核心术语中英对照，统一全仓翻译口径。
3. **[项目原版 README 中文版](README_ORIG.md)** —— 快速认识 EGM 的定位、安装方式与 8 个核心技能。

---

## 📚 全仓库中文翻译与架构导读导航

### 1. 仓库顶层规范与治理文档
- [README_ORIG.md](README_ORIG.md) — 项目主介绍（对应根 `README.md`）
- [AGENTS.md](AGENTS.md) — 仓库内 AI 助手贡献边界与 PR 协作规则
- [CONTRIBUTING.md](CONTRIBUTING.md) — 人类与 AI 贡献指南
- [TESTING.md](TESTING.md) — 测试体系、验证门禁与运行说明
- [ACCESSIBILITY.md](ACCESSIBILITY.md) — 无障碍与可访问性规范
- [AI_POLICY.md](AI_POLICY.md) — AI 辅助开发政策

### 2. 运行时规则与文档标准（`zh-CN/resources/`）
> 整个系统的“宪法”与“尺子”——决定大象怎么写、金鱼怎么审。
- [resources/README.md](resources/README.md) — 运行时权威资源总索引
- [resources/execution_instructions.md](resources/execution_instructions.md) — 大象起草阶段硬性执行规范
- [resources/document_standards/zoom-out-standard.md](resources/document_standards/zoom-out-standard.md) — 拉远视角（高海拔 / YAGNI / 反杂草）审查标准
- [resources/document_standards/feature-spec-standard.md](resources/document_standards/feature-spec-standard.md) — 高层特性规格标准（备选方案规则）
- [resources/document_standards/implementation-readiness-standard.md](resources/document_standards/implementation-readiness-standard.md) — 实现就绪标准（零决策规则 & 文件写白名单）
- [resources/document_standards/re-implementation-standard.md](resources/document_standards/re-implementation-standard.md) — 重实现标准（零假设规则）
- [resources/document_standards/heilmeier-standard.md](resources/document_standards/heilmeier-standard.md) — 海尔迈耶教义问答（立项 8 问）标准

### 3. 子智能体人格层（`zh-CN/agents/`，共 10 个）
> 零上下文只读“金鱼”评审团与编排器源码级 Prompt 翻译（每篇附第一性原理导读）。
- [agents/synthesizer.md](agents/synthesizer.md) — 蜂群编排与输出合成器 (`egm-synthesizer`)
- [agents/zoom_out.md](agents/zoom_out.md) — 怀疑论架构师 (`egm-zoom-out`)
- [agents/inventive_catalyst.md](agents/inventive_catalyst.md) — TRIZ 发明催化师 (`egm-inventive-catalyst`)
- [agents/comprehension.md](agents/comprehension.md) — 零先验新手读者 (`egm-comprehension`)
- [agents/design_critic.md](agents/design_critic.md) — 敌对式设计批评者 (`egm-design`)
- [agents/structural.md](agents/structural.md) — 结构布局验证器 (`egm-structural`)
- [agents/style.md](agents/style.md) — 技术文案编辑 (`egm-style`)
- [agents/implementation_readiness.md](agents/implementation_readiness.md) — 务实技术负责人 (`egm-implementation-readiness`)
- [agents/codebase_details.md](agents/codebase_details.md) — 迂腐级代码库审计员 (`egm-codebase-details`)
- [agents/bootstrap_orchestrator.md](agents/bootstrap_orchestrator.md) — 代码库初始化并行编排器 (`egm-bootstrap-orchestrator`)

### 4. 核心技能工作流层（`zh-CN/skills/`，共 8 大技能完整翻译）
每个技能目录均完整翻译了 `SKILL.md`、`TEST.md`、`test_scenarios.yaml`、`agents/openai.yaml` 及 `references/`：
- [egm-bootstrap](skills/egm-bootstrap/SKILL.md) — 自底向上代码库文档初始化（花生与干草模型）
- [egm-brainstorm](skills/egm-brainstorm/SKILL.md) — 交互式 7 棱镜头脑风暴
- [egm-draft](skills/egm-draft/SKILL.md) — 分节式规格编译与语义分割
- [egm-review](skills/egm-review/SKILL.md) — 五阶段金鱼隔离评审流水线
- [egm-review-loop](skills/egm-review-loop/SKILL.md) — 通用敌对式循环评审状态机
- [egm-resolve](skills/egm-resolve/SKILL.md) — 大象上下文裁决与人类审批闭环
- [egm-drift-report](skills/egm-drift-report/SKILL.md) — 文档与源码漂移巡检器
- [egm-lite](skills/egm-lite/SKILL.md) — 轻量级五步单回合工程协议

### 5. 文档中心与全部设计文档（`zh-CN/docs/`）
- **用户指南**：[index.md](docs/index.md) | [getting_started.md](docs/getting_started.md) | [faq.md](docs/faq.md) | [privacy.md](docs/privacy.md) | [releases.md](docs/releases.md) | [directory.md](docs/directory.md)
- **多宿主集成指南**：[harnesses.md](docs/harnesses.md) | [claude.md](docs/claude.md) | [codex.md](docs/codex.md) | [copilot.md](docs/copilot.md) | [cursor.md](docs/cursor.md) | [muse.md](docs/muse.md) | [opencode.md](docs/opencode.md)
- **验证报告**：[verification/skill-discovery-316.md](docs/verification/skill-discovery-316.md)
- **官网文档（`docs/site/`）**：[guide.md](docs/site/guide.md) | [install.md](docs/site/install.md) | [getting-started.md](docs/site/getting-started.md) | [workflow.md](docs/site/workflow.md) | [skills.md](docs/site/skills.md) 及 8 个技能专页
- **全部 20 篇架构与功能设计文档（`docs/designs/`）**：从 [docs/designs/README.md](docs/designs/README.md) 索引进入

### 6. GitHub Copilot 代理适配文件（`zh-CN/com.github.copilot/agents/`）
- 全部 9 个 `.agent.md` 适配定义文件的完整中文版（入口 [egm.agent.md](com.github.copilot/agents/egm.agent.md) 及 8 个评审代理）

### 7. 工程代码、多宿主清单、打包与测试体系中文架构导读
> 面向想读懂底层代码实现、从 0 到 1 复刻工程护城河的开发者：
- [manifests/README.md](manifests/README.md) — 单一权威来源 + 多宿主清单/薄适配器架构深度解析
- [adapters/README.md](adapters/README.md) — `adapters/opencode/install.py` 逐函数中文导读与安全隔离设计
- [scripts/README.md](scripts/README.md) — `pack_plugin.py` 与 `prepare_release.py` 确定性打包与发布源码解析
- [tests/README.md](tests/README.md) — 全部单元测试不变量、集成探针与 Fail-Closed Trace 校验器导读
- [site/README.md](site/README.md) — `egm.dev`（Astro + Starlight + `llms.txt`）文档站架构与脚本解析
