> **原文路径**：`skills/egm-review/TEST.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 测试计划：EGM 评审与子智能体（EGM Review & Subagents）

Frontmatter 与链接检查在 `pytest tests/unit` 中运行（参见 [TESTING.md](../../TESTING.md)）。

## 协议验证清单

在聊天中交互式评估或探索 `egm-review` 时，请验证以下各项：

-   **第 1 步（输入采集）：** 收集 `TargetFile` 和可选的 `GoldfishContext`。在进入第一个接收 `GoldfishContext` 的阶段之前，自动添加位于工作目录内的链接文件，对解析到工作目录之外的链接或 URL 先征求批准，并将 `GoldfishContext` 中的每个 URL 保存为本地文件。
-   **第 2 步（预设选择）：** 提供四种评审预设（交互式、综合报告、仅特性规格、仅实现计划），或将显式 `Step` 参数映射到对应的预设或单个阶段子流程。
-   **第 3 步（语义分割）：** 发现特性规格与详细实现计划之间的分割行。
-   **第 5 步（初始战略评审）：** 并行派生 `egm-zoom-out` 与 `egm-inventive-catalyst`，审计海拔高度与伪架构妥协。
-   **子流程：** 正确委派初始战略评审、可理解性测试、批判评审、中段拉远视角与实现就绪度测试。
