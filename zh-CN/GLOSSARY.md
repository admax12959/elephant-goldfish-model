# 大象-金鱼模型（EGM）统一中英文术语表（Glossary）

> 本文档为 `zh-CN/` 全仓库中文翻译与学习指南的权威术语对照表。所有翻译文件均严格遵循此表，方便你中英对照阅读源码、Prompt 与设计文档。

| 英文原词 (English Term) | 推荐中文译名 (Chinese) | 核心含义与语境说明 |
| :--- | :--- | :--- |
| **Elephant-Goldfish Model (EGM)** | 大象-金鱼模型 (EGM) | 全仓核心方法论名称：大象起草、金鱼评审、人类拍板 |
| **Elephant** | 大象（高上下文构建者） | 持有全量历史、代码地图与业务背景的主智能体 |
| **Goldfish** | 金鱼（零上下文评审员） | 运行在隔离新鲜上下文中的只读子智能体 |
| **Spec-Driven Development (SDD)** | 规格驱动开发 (SDD) | 先对齐并审查完备规格文档，再编写代码的工程范式 |
| **Harness** | 宿主环境 / 运行时宿主 | 指 Antigravity、Claude Code、Codex、Cursor、Copilot、OpenCode、Windsurf 等 AI 编程客户端底座 |
| **Skill** | 技能 | `skills/*/SKILL.md` 定义的可调用工作流单元 |
| **Subagent** | 子智能体 | 由宿主工具派生出的独立执行线程 |
| **Persona** | 智能体人格 / 专家角色 | `agents/*.md` 中定义的子智能体系统指令设定 |
| **Document Standard / Rubric** | 文档标准 / 评估准则 | `resources/document_standards/*.md` 中的质量标尺 |
| **Context Window** | 上下文窗口 | LLM 单次对话或子智能体持有的记忆边界 |
| **Fresh Context / Zero-Context** | 新鲜上下文 / 零上下文 | 剥离父会话对话历史与作者主观辩解的隔离状态 |
| **Context Pollution** | 上下文污染 | 早期错误假设或冗余日志堆积导致推理质量下降 |
| **Hallucination Cascade** | 幻觉级联 | 错误假设在单一长对话中被模型自我强化、滚雪球放大 |
| **Weeds Syndrome / Altitude Drop** | 杂草综合征 / 海拔下沉 | 高层设计阶段过早陷入底层函数、循环变量等实现细节 |
| **Anti-Weeds (Altitude Control)** | 反杂草约束（海拔控制） | 强制架构文档停留在组件边界与数据流层面 |
| **Zoom Out** | 拉远视角（高海拔审查） | 跳出局部代码，审视根因、YAGNI 与现有基础设施复用 |
| **Artifact-First UX** | 产物优先交互协议 (Artifact-First) | 禁止向聊天窗倾倒长文本，一律写入持久化文件或侧边栏产物 |
| **Chat Dump Fatigue** | 聊天倾倒疲劳 | 冗长方案或评审日志直接刷屏聊天窗造成的阅读与上下文退化 |
| **Living Specification** | 活体规格 | 随代码演进并受漂移检测监控的受控设计文档 |
| **Drift / Documentation Drift** | 漂移 / 文档漂移 | 设计文档或 README 描述与仓库现行代码实现产生偏离 |
| **Synthesizer** | 合成器 / 蜂群合成器 (`egm-synthesizer`) | 协调并发金鱼扇出、去重并生成结构化报告的中间协调智能体 |
| **Fan-Out / Fan-In** | 扇出 / 扇入 | 并发派生多个子智能体执行并回收汇总结果 |
| **TRIZ (Theory of Inventive Problem Solving)** | TRIZ（发明问题解决理论） | `egm-inventive-catalyst` 用于消除系统矛盾的方法论 |
| **False Compromise** | 伪妥协 | 设计中被误认为不可避免、实可通过架构模式消除的技术权衡 |
| **Ideal Final Result (IFR)** | 理想最终解 (IFR) | TRIZ 概念，在不引入额外运维负担下实现核心功能的理想状态 |
| **Heilmeier Catechism** | 海尔迈耶教义问答 | 评估项目立项价值、风险与验收考题的八问框架 |
| **Critical User Journey (CUJ)** | 关键用户旅程 (CUJ) | 以 Given-When-Then 描述的端到端核心业务路径 |
| **Semantic Split / Boundary** | 语义分割 / 语义边界 | 划分高层特性规格 (`FeatureRange`) 与详细实现计划 (`DetailedRange`) 的分界线 |
| **Zero-Decision Rule** | 零决策规则 | 实现就绪标准核心原则：不把任何架构或设计选择留给实现者 |
| **Zero-Assumption Rule** | 零假设规则 | 重实现标准核心原则：文档须完备到无需猜测默认值或异常逻辑 |
| **Target Scope & File Boundary** | 目标范围与文件边界 | 经评审批准的阶段五代码修改白名单 |
| **Peanuts & Hay** | 花生与干草模型 | `egm-bootstrap` 中叶子组件契约（花生）与高层索引蓝图（干草）的隐喻 |
| **Checkpoint Gate / Consensus Gate** | 检查点门禁 / 共识门禁 | 必须暂停执行并等待人类明确确认或选择的操作关卡 |
| **PROPOSE FIX / CONTEST / ESCALATE / DECLINE** | 建议修复 / 提出异议 / 上报裁决 / 婉拒忽略 | `egm-resolve` 对金鱼评审意见的四大裁决分类（保留英文标签辅以中文释义） |
| **Settled / Clean Round / Converged / Churn** | 已裁决 / 净轮次 / 已收敛 / 修复振荡 | `egm-review-loop` 控制多轮迭代终止的核心状态机判定词 |
| **No Glazing / Sycophancy** | 禁止奉承 / 消除谄媚 | 严禁 AI 使用空洞赞美词或无原则迎合用户，必须优先质疑风险 |
