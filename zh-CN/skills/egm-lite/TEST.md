> **原文路径**：`skills/egm-lite/TEST.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 测试计划：EGM-Lite 技能

Frontmatter、链接检查与场景覆盖率在 `pytest tests/unit` 中运行（参见 [TESTING.md](../../TESTING.md)）。

## 协议验证清单

在聊天中交互式评估或探索 `egm-lite` 时，请验证以下各项：

-   **第 1 步（批判暂停，Critique Pause）：** 智能体扮演 **大象（批判者，Elephant - Critic）** 角色，以简明自然语言的 `> **BLUF:**` 引用块（后接 `---`）开头，说明方案是否已就绪可进入 `checklist` 还是存在阻塞；在不编辑仓库代码的前提下质疑假设；当存在多个可行选项时，在 `egm_lite_alternatives.md` 中记录竞争方案且不在聊天正文内联展开讨论（对单一路径或微不足道的任务则省略该文件）；并且仅提示输入 `checklist`，若收到 `exec` 则以状态建议（state advisory）回应。
-   **第 2 步（清单与规格门禁，Checklist & Spec Gating）：**
    -   *第 2A 节（清单生成，Checklist Generation）：* 收到 `checklist` 后，智能体扮演 **大象（规划者，Elephant - Planner）** 角色，采用 GFM 任务列表复选框（`- [ ]`）输出逐项列出的变更目标清单（将文件与引用规格格式化为可点击的文件与行号范围链接）、变更理由以及任何用户同意的评审前排除项；评估确定性复杂度条件（两个及以上生产代码文件、异步/并发、公共 API 或 `[⚠️ UNVERIFIED]`/`[⏳ ACK]` 标签）；输出 BLUF，对非平凡任务推荐先做规格评审（`goldfish` / `review`），对常规/文档任务说明直接执行合理；且在任何情况下都同时提供 `exec` 与 `goldfish` / `review` 选项。
    -   *第 2B 节（无状态规格评审，Stateless Spec Review）：* 在实现前收到 `goldfish` 或 `review` 时，智能体通过宿主环境的新鲜上下文子智能体工具扮演 **金鱼（规格评审员，Goldfish - Specification Reviewer）** 角色（若无可用工具则立即停止），注入清单 Markdown 与规格评审准则（跳过评审前排除项与已接受的 `CONTEST` 条目，且禁止 AST/diff 检查），并暂停等待子智能体响应。
-   **第 3 步（执行协议，Exec Protocol）：** 收到 `exec`（或点击 **Proceed**）后，智能体扮演 **大象（构建者，Elephant - Builder）** 角色。若紧接在已验证通过的规格之后，直接读取 `egm_lite_checklist.md` 而不重建；若为真正的快速路径，则先初始化清单。仅实现已批准的清单条目（涵盖代码、规格或设计文档），运行覆盖受影响目标的本地测试（纯文档编辑则为 N/A 或语法检查 linter），将已完成条目勾选为 `- [x]`；在上报裁决（escalation）、自我修复（self-repair）或多文件总结时以 `> **BLUF:**` 开头（干净的单目标运行则省略）；并在完成门禁处提示输入 `review` 或 `goldfish`。
-   **第 4 步（对抗式差异评审，Adversarial Diff Review）：** 在执行后（或无进行中任务但存在已有差异时）收到 `review` 或 `goldfish` 时，智能体扮演 **金鱼（子智能体评审员，Goldfish - Reviewer via Subagent）** 角色（通过宿主环境的新鲜上下文子智能体工具派生，无可用工具则立即停止），将发现限定在实质性问题并抑制风格类吹毛求疵，执行严格的二元评审响应（仅报告发现 vs. 零发现时显式回复 `[✅ LGTM]`），按差异评审准则评估（针对文档差异，将运行时检查替换为事实一致性与 Markdown 规范），并暂停等待子智能体响应。
-   **第 5 步（统一评审后裁决门禁，Unified Post-Review Resolution Gate）：** 收到子智能体发现（规格评审或差异评审）后，智能体扮演 **大象（裁决者，Elephant - Resolver）** 角色，保持分诊为纯展示型而不执行任何文件变更；以简明自然语言的 `> **BLUF:**` 引用块（后接 `---`）开头，冠以 **`[🛠️ X FIX | 🛡️ Y CONTEST]`**（或零发现通过时的 **`[✅ LGTM]`**）；将发现分诊为连续编号并带优先级标签的 `[X] [PY]` 条目（`P0`..`P3`）；根据返回响应的评审类型进行分支处理（实现前规格分诊提供 `checklist`、`exec` 或 `goldfish` 重审，并严禁过早关闭任务；实现后差异评审分诊提供 `checklist` 或 `exec`）；并将已接受的 `CONTEST` 理由持久化写入 `egm_lite_checklist.md`。
-   **顶层不变量与优雅恢复（Top-Level Invariants & Graceful Recovery）：** `goldfish` 与 `review` 是绑定到当前状态的互换别名；无效状态跃迁（例如无可审内容时请求差异评审、清单创建前请求规格评审）会输出优雅的 `> **BLUF:** [State Advisory]` 引用块；快速路径执行原生支持规格与设计文档的迭代更新。
-   **代码引用与非谄媚立场（Code References & Non-Sycophantic Posture）：** 引用的本地文件与行号范围使用可点击链接，内部运行指令绝不回显到用户聊天中，且回复优先指出技术风险或缺陷，不使用奉承性形容词或客套道歉。
