> **原文路径**：`skills/egm-review/TEST.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 测试计划：EGM 评审与子智能体（EGM Review & Subagents）

Frontmatter 与链接检查在 `pytest tests/unit` 中运行（参见 [TESTING.md](../../TESTING.md)）。

## 协议验证清单

在聊天中交互式评估或探索 `egm-review` 时，请验证以下各项：

-   **第 1 步（输入采集，Intake）：** 收集 `TargetFile` 和可选的 `GoldfishContext`。在进入第一个接收 `GoldfishContext` 的阶段之前，自动添加位于工作目录内的链接文件，对解析到工作目录之外的链接或 URL 先征求批准，并将 `GoldfishContext` 中的每个 URL 保存为本地文件。
-   **第 2 步（预设选择，Preset Selection）：** 提供四种评审预设（交互式 `interactive`、综合报告 `consolidated`、仅特性规格 `feature_spec`、仅实现计划 `implementation`），或将显式 `Step` 参数映射到对应的预设或单个阶段子流程（`Stage 5`–`Stage 9`）。
-   **第 3 步（语义分割，Semantic Split）：** 发现高层特性规格（`FeatureRange`）与详细实现计划（`DetailedRange`）之间的语义分割边界行。
-   **阶段 5 / 第 5 步（初始战略评审，Stage 5: Initial Strategic / Zoom Out Review）：** 并行派生 `egm-zoom-out` 与 `egm-inventive-catalyst`，审计架构海拔高度与伪架构妥协（false architectural compromises）。
-   **五阶段子流程（Stages 5–9 Subflows）：** 正确委派阶段 5（初始战略/拉远视角评审）、阶段 6（可理解性测试：直接派发 `egm-comprehension`，不经过合成器，保留转述 paraphrase）、阶段 7（批判评审，作用于 `FeatureRange`）、阶段 8（中段拉远视角评审）与阶段 9（实现就绪度测试，作用于 `DetailedRange`）。
