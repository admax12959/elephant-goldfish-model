> **原文路径**：`skills/egm-resolve/TEST.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 测试计划：金鱼评审裁决技能（Goldfish Review Resolve Skill）

Frontmatter 与链接检查在 `pytest tests/unit` 中运行（参见 [TESTING.md](../../TESTING.md)）。

## 协议验证清单

在聊天中交互式评估或探索 `egm-resolve` 时，请验证以下各项：

-   **第 1 步（输入校验与上下文）：** 接收 `TargetFile`、`LineRange` 与 `FindingsList`。
-   **第 2 步（评估与分类）：** 使用可核查的证据为有效设计选择辩护（`CONTEST`），将无任何受审文件支持的反驳交由用户裁决（`ESCALATE`），起草具体修复方案（`PROPOSE FIX`），并记录无须修复的 Nit（`DECLINE`）。
-   **第 3 步（生成修复报告）：** 生成 `goldfish_review_report.md` 产物。
-   **第 4 步（裁决门禁）：** 在不使用模态弹窗的情况下向用户提供清晰的决策选项（Auto accept、Continue、Interactive breakdown），并执行经批准的文件修改。
