---
name: egm-bootstrap
license: Apache-2.0
disable-model-invocation: true
description: 为整个代码仓库子树引导生成符合规范的目录 README.md 契约文档。
---

> **原文路径**：`skills/egm-bootstrap/SKILL.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。
>
> 💡 **第一性原理提示**：该技能的核心设计思想是通过“花生与干草模型”（叶子节点代码契约 + 父级目录蓝图索引）为代码库建立自底向上的上下文地图，从而防止 AI 智能体在大型仓库中迷失或产生幻觉。通过广度优先搜索（BFS）增量扫描、15 目录安全门禁与只读金鱼侦察兵分块扫描机制，它严格控制了上下文窗口膨胀与并发开销，并确保所有文档声明都锚定在真实的代码符号之上。

# EGM 花生与干草引导技能（`egm-bootstrap`）

在 Codex 或 ChatGPT Work 中运行时，请先阅读 [Codex 集成指南](../../docs/codex.md)。
在 Claude Code 或 Cowork 中运行时，请先阅读 `${CLAUDE_PLUGIN_ROOT}/docs/claude.md`。

## 概述

本技能在目录树中自动创建、修正、格式化并校验符合大象-金鱼模型（EGM）规范的 `README.md` 文件，为人类开发者与 AI 助手建立清晰的架构契约（接口、不变量与副作用）。

*   **花生（Peanuts）**：叶子层级的组件包 README，包含具体的文件级契约。
*   **干草（Hay）**：父层级的仓库蓝图，将子级接口契约汇总为高海拔索引。

## 委派机制

金鱼（Goldfish）运行在没有对话历史的新鲜上下文中。请使用当前宿主环境的子智能体工具派生各个子智能体（Antigravity：`invoke_subagent`；Claude Code：`Agent`；Codex：`spawn_agent`）。每个子智能体均定义在 `agents/<file>.md` 中；若宿主环境支持注册命名子智能体，其 frontmatter 中的 `name`（例如 `egm-zoom-out`）即是要请求的子智能体类型。否则，请派生通用子智能体，并指示其先阅读对应的智能体人格文件。原样传入提示词并等待报告返回。若宿主环境不支持嵌套子智能体，请绕过合成器改用直接派发模式。详见 `docs/harnesses.md`。
如果你的宿主环境没有子智能体工具，请内联执行侦察步骤。

## 输入参数

*   **`path`**：要执行引导的目标根目录路径（必填）。
*   **`is_worker`**：内部参数。当作为工作子编排器（Sub-Orchestrator）执行时为 `true`（可选，默认：`false`）。
*   **`sub_tree_root`**：内部参数。被分配的子树分区的根路径（可选）。
*   **`target_filename`**：要生成或更新的目标契约文件名（可选，默认：`"README.md"`）。当不能修改既有的人工维护 README 时，支持指定替代蓝图文件名（例如 `"EGM.md"`）。

## 执行约束（关键）

1.  **禁止生成业务代码**：你的职责严格限定在文档范围内。不得编写或重构任何功能性源代码。
2.  **版本控制隔离（VCS Isolation）**：不得执行任何版本控制提交操作（`git commit`、`git add`）。所有文件系统变更必须使用标准文件写入工具写在本地。追踪与提交操作一律交由人类用户处理。
3.  **临时文件隔离**：每个工作节点必须将其状态、检查点和日志文件保存在工作区之外的独立目录中（宿主临时目录或 `mktemp -d`），绝不能使用固定或共享路径。如果两者均不可用，则跳过写入临时文件。
4.  **工具使用（关键）**：使用你的 Shell、文件读取、文件写入和文件编辑工具进行目录发现与契约/README 生成。在生成阶段直接遵循整洁的 Markdown 格式（标准标题、表格和围栏代码块）。如果仓库中显式配置了项目级 Markdown 格式化工具（例如 `npx prettier --write`），可选择性调用。
5.  **子智能体委派**：并行派生子编排器工作节点。
6.  **强制安全门禁（关键）**：如果发现的子目录数量超过 15 个，你**必须**立即请求用户批准（如果宿主环境提供结构化提问工具则使用该工具，否则在普通聊天中询问），然后才能继续全树执行或派生工作节点。

---

## 工作流

### 步骤 0：生态系统与验证命令发现

1.  检查仓库根目录下的包管理/构建清单文件：
    *   Node.js / TypeScript：`package.json` -> `npm test`（或 `pnpm test` / `yarn test`）
    *   Rust：`Cargo.toml` -> `cargo test`
    *   Go：`go.mod` -> `go test ./...`
    *   Python：`pyproject.toml` / `pytest.ini` / `setup.py` -> `pytest`
    *   .NET / C#：`*.sln` / `*.slnx` / `*.csproj` -> `dotnet test`
    *   C/C++ / Make：`Makefile` -> `make test`
    *   Bazel：`BUILD` / `BUILD.bazel` -> `bazel test //...`
    *   未找到：`<test-command>`
2.  将其记录为 `VerificationCommand`，用于填充生成的 README 契约文档。

---

### 步骤 1：启动分支控制

1.  检查 `is_worker` 输入参数。
2.  如果 `is_worker` 为 `true`，直接跳转至**步骤 4：子编排器工作流**，处理分配的 `sub_tree_root`。
3.  如果 `is_worker` 为 `false` 或未设置，则作为根编排器（Root Orchestrator）继续执行**步骤 2：增量范围探测与安全门禁**。

---

### 步骤 2：增量范围探测与安全门禁（根编排器）

1.  **增量 BFS 发现**：使用目录队列对目标 `path` 树执行逐层广度优先遍历（BFS），每次仅列出单个目录层级。（关键：绝不运行 `find` 或 `ls -R` 等递归搜索命令。）

    *   保留中间文件夹节点。
    *   忽略符号链接（symbolic links），防止无限递归。
    *   忽略隐藏/系统目录（`.git`、`.antigravity`、`node_modules`、`venv`、`dist`、`build`、`third_party`）。
    *   读取 `.gitignore` 和 `.egmignore`（若存在），忽略匹配的 glob 模式。
    *   在发现映射中保留包与构建清单文件（`package.json`、`Cargo.toml`、`go.mod`、`pyproject.toml`、`*.sln`、`*.csproj`、`Makefile`、`BUILD`），以便后续提取验证目标。

2.  **安全门禁与派生审批（关键）**：在 BFS 循环期间，如果累计发现的目录总数超过 **15** 个，必须立即暂停发现过程并请求用户批准（若宿主环境支持结构化提问工具则使用该工具，否则在聊天中询问），警告用户并征求全树执行的许可：

    *   `question`："The target directory tree contains more than 15 subdirectories. This is a large execution scope: up to 5 parallel workers, and at least one scout per code directory (one per ~1,500 lines of source). Do you want to proceed with full-tree bootstrapping?"（目标目录树包含超过 15 个子目录。本次执行范围较大：最多派生 5 个并行工作节点，且每个代码目录至少派生 1 个侦察兵（约每 1,500 行源码 1 个）。是否继续执行全树引导？）
    *   `options`：`["Yes, proceed with full tree bootstrapping", "No, abort execution"]`（`["是，继续执行全树引导", "否，中止执行"]`）
    *   （关键：请求批准后，你**必须**等待用户回复，然后才能继续或派生任何工作节点）。
    *   如果用户选择中止，立即终止执行。

3.  **自底向上后置剪枝（Bottom-up Post-Pruning Pass）**：映射完成（且已获批）后，自底向上（从叶子到根）遍历映射树，过滤掉非代码目录和纯测试目录：

    *   *源代码文件*：匹配公认的源代码、模式（Schema）与契约扩展名的文件：
        - 系统与编译型语言：`.c`, `.cc`, `.cpp`, `.cxx`, `.h`, `.hh`, `.hpp`, `.rs`, `.go`, `.zig`, `.lean`, `.f`, `.f90`, `.f95`, `.f03`, `.f08`, `.hs`
        - 托管与企业级语言：`.java`, `.kt`, `.scala`, `.cs`, `.razor`, `.cshtml`, `.fs`, `.vb`
        - 脚本与动态语言：`.py`, `.rb`, `.php`, `.lua`, `.sh`, `.bash`, `.ps1`, `.psm1`, `.psd1`, `.r`, `.R`, `.ex`, `.exs`
        - Web 与移动端：`.ts`, `.tsx`, `.js`, `.jsx`, `.mjs`, `.cjs`, `.swift`, `.m`, `.dart`
        - 数据与模式 / IDL：`.sql`, `.proto`, `.graphql`, `.gql`, `.thrift`, `.prisma`
        - 基础设施 / 声明式代码：`.tf`, `.hcl`, `.cue`
        （排除构建清单、锁文件、测试夹具、文档以及 `.csv`、`.tsv`、`.parquet` 等通用数据转储文件）。
    *   *测试目录排除*：排除仅包含测试代码的目录（例如命名为 `tests`、`testing`、`testdata`，或仅包含匹配 `*_test.*`、`*Test.*`、`test_*.*` 或 `*.spec.*` 文件的目录）。
    *   *非代码过滤*：排除既不包含源代码文件、也不包含任何含代码子目录的目录。仅当空中间目录拥有包含代码的子目录时才保留该中间目录。

4.  计算剪枝后的目录总数和最大深度。继续进入**步骤 3：执行策略选择**。

---

### 步骤 3：执行策略选择（根编排器）

1.  **直接执行策略（目录数 <= 5）**：

    *   如果剪枝后的目录总数为 5 个或更少，则在当前线程中直接执行自底向上的文档生成。
    *   跳转至**步骤 4：子编排器工作流**（将根编排器视为整个 `path` 的工作节点），然后继续执行**步骤 5：根目录汇总与终止**。

2.  **并行子智能体蜂群策略（目录数 > 5）**：

    *   （关键：如果目录数 > 15，你**必须**确认已获得用户的明确批准方可继续。如果尚未获批，必须立即请求批准并等待用户回复后再派生工作节点）。
    *   使用**自适应深度分区算法（Adaptive Depth Partitioning Algorithm）**将发现映射划分为独立的子树分支：
        1.  设置 `depth = 1`。
        2.  从发现映射中提取位于当前 `depth` 的目录节点。
        3.  如果当前 `depth` 的节点数在 3 到 5 之间：选择这些节点作为子树分区根。
        4.  如果当前 `depth` 的节点数 < 3：将 `depth` 加 1（最大不超过 3），并重复第 2 步。
        5.  如果当前 `depth` 的节点数 > 5：将最大的几个分支（按嵌套文件夹总数衡量）划分为 3-4 个独立分区，将剩余的小分支合并为一个兜底分区子树，并选择这些分区作为分区根。
        6.  回退策略：如果 `depth` 达到 3 时节点数仍然 < 3，则选择根节点的直接子目录（或回退到顺序执行）。
    *   **派生子编排器**：为每个分区子树根使用宿主环境的子智能体工具派生一个并行工作节点：
        *   子智能体：`egm-bootstrap-orchestrator`（`agents/bootstrap_orchestrator.md`）（或通用子智能体）
        *   `Prompt`："Run egm-bootstrap with path=`[path]`, is_worker=true, sub_tree_root=`[partition_root]`, target_filename=`[target_filename]`."（以 `path=[path]`、`is_worker=true`、`sub_tree_root=[partition_root]`、`target_filename=[target_filename]` 运行 `egm-bootstrap`。）
    *   **工作节点上限**：并行派生子编排器，总数绝不超过 5 个。
    *   进入空闲等待状态，待所有工作子智能体返回其 JSON 子树索引载荷后，跳转至**步骤 5：根目录汇总与终止**。

---

### 步骤 4：子编排器工作流

1.  **侦察兵注册**：使用只读的 `goldfish-explorer` 侦察兵。如果宿主环境支持在运行时定义子智能体，请按如下参数注册：

    *   `name`：`goldfish-explorer`
    *   `description`：`Read-only codebase explorer scout.`（只读代码库探索侦察兵。）
    *   `system_prompt`：>- You are a read-only codebase explorer. Your task is to analyze the source files in the target directory and return a concise, factual summary. Do not write or modify any files. Return a JSON object containing the directory's public symbols (interface), invariants, and side effects matching schema: {"interface": [], "invariants": [], "side_effects": []}. Never infer RPC methods, routes, status enums, or invariants from file or directory names alone; cite the exact `[file:symbol_or_flag]` for every entry, and output `["None"]` if no explicit invariants or side effects exist in code.
        （你是一名只读代码库探索者。你的任务是分析目标目录中的源代码文件，并返回简洁、基于事实的摘要。不要写入或修改任何文件。返回一个包含该目录公共符号（interface）、不变量（invariants）和副作用（side_effects）的 JSON 对象，结构需符合：`{"interface": [], "invariants": [], "side_effects": []}`。绝不要仅凭文件名或目录名推断 RPC 方法、路由、状态枚举或不变量；每一项都必须精确引用 `[file:symbol_or_flag]`，如果代码中不存在显式的不变量或副作用，请输出 `["None"]`。）
    *   `enable_write_tools`：`false`
    *   `enable_subagent_tools`：`false`

2.  **后序遍历（Post-Order Traversal）**：按后序（最深叶子优先）遍历分配的 `sub_tree_root`（或 `path`）。对于每个目录 `D`：

    *   **修正与修改策略**：检查 `target_filename`（默认：`README.md`）是否已存在于 `D` 中。

        *   如果 `target_filename` 已存在，**不要完全覆盖它**，而是读取其现有内容。
        *   *侦察扫描要求*：无论 `target_filename` 是否存在，**始终使用** `goldfish-explorer` 侦察兵扫描 `D` 的源代码，以构建真实的代码符号边界。
        *   *遗留文档提取与修正*：如果 `target_filename` 已存在，使用如下提示词调用推理 LLM：> "Analyze the following unstructured documentation content. Extract all public symbol names, classes, interfaces, invariants, and side effects. Format your response strictly as a JSON object matching the following schema: { \"interface\": string[], \"invariants\": string[], \"side_effects\": string[] }."（分析以下非结构化文档内容。提取所有公共符号名称、类、接口、不变量和副作用。将你的响应严格格式化为符合以下结构的 JSON 对象：`{ "interface": string[], "invariants": string[], "side_effects": string[] }`。）将这些提取出的符号与侦察兵的代码符号合并。使用文件编辑工具（或文件写入工具）修正并更新现有契约文件，使其符合 EGM 规范，同时保留开发者原有的注释和说明。

    *   **侦察兵派生与约束**：为 `D` 派生 `goldfish-explorer`。

        *   *侦察约束*：不设单目录或单文件字节上限；`D` 中的每个非测试文件都必须完整读取。改用约 **1,500 行**源码作为单次侦察调用的上限：先用搜索工具列出顶层定义（class/func/def/route/flag 所在行），沿这些边界将目录内的文件切分为多个分块（绝不在函数或类体中间截断；小文件可合并至同一分块），为每个分块并行派生一个 `goldfish-explorer`（并发上限 5 个，超出部分排队），最后合并各分块返回的 JSON。将文件的 imports 和顶层定义列表一并提供给每个分块，以便其判断可见性。在将 RPC 服务、模式或消息类型归属到本地文件之前，请先检查构建清单依赖和 `import` / `#include` 语句。按以下顺序读取非测试文件：
            1.  契约、模式与构建清单（`.h`、`.hh`、`.hpp`、`.proto`、`.graphql`、`.gql`、`.thrift`、`.prisma`、`.sql` DDL、接口类）；提取精确的 DDL 表名与导出的构建目标。
            2.  模块入口文件（`main.*`、`index.*`、`lib.*`、`mod.*`、`__init__.py` 或与目录同名的文件），随后按字母顺序读取生产源码/查询文件。
            3.  模板与配置文件：每个文件最多读取 200 行（纯配置目录最多 800 行）；排除 `.css`/`.scss`。
            4.  测试文件（`*_test.*`、`*Test.*`、`test_*.*`、`*.spec.*`）：完全忽略，包括混合目录中的测试文件。
        *   *子级容错*：如果侦察兵失败或超时，记录一条警告，使用占位值 `["(Details pending/documentation failed)"]`，并继续执行。

    *   **父级上下文传播**：在处理父目录 `P` 时，获取其直接子目录已汇总的接口摘要。

        *   *传播启发式规则*：
            1.  如果子目录是在本次运行中生成的文档，直接从内存执行缓存中获取其公共接口。
            2.  如果子目录是按修正策略更新过的，则使用与遗留文档提取结果合并后的符号集。
        *   在调用 `P` 的探索侦察兵时，将这些子级符号块作为提示词上下文传入。

    *   **验证命令目标查找**：使用已发现的 `VerificationCommand` 填充“验证（Verification）”章节。
        *   如果项目使用 Bazel/BUILD 文件：检查 `D` 或最近祖先目录中是否存在 `BUILD` 文件，并引用带有合法目标模式的 `bazel test` / `bazel build`（使用 `//[package_path]/...` 或 `//[package_path]:all`，绝不使用 `//[package_path]...` 或 `//[package_path]:...`）。对于交互式运行目标，使用 `bazel run`。
        *   如果项目使用其他清单文件（`package.json`、`Cargo.toml`、`go.mod`、`pyproject.toml`、`*.sln`、`*.csproj`、`Makefile`），填入对应的测试运行命令（.NET：对最近的 `*.csproj` 运行 `dotnet test`）。
        *   如果未识别出验证命令，则省略该项或使用占位符。

    *   **目录分类与模板生成**：根据 `D` 的分类，使用文件写入工具或文件编辑工具生成或修正 `target_filename`。

        *   **组件包（花生 / Peanut）**：`D` 包含直接源代码文件。使用**组件包 README 模板**。（如果同时包含子目录，请在 Interface 章节中加入指向子目录契约的相对链接）。
        *   **仓库蓝图（干草 / Hay）**：`D` 不包含直接源代码文件，但包含拥有代码契约的子目录。使用**中间蓝图 README 模板**。
        *   **链接格式策略（关键）**：包契约文件必须采用混合链接格式，以兼顾可移植性与清晰度：
            1.  **仅使用指向既有文件的相对链接**：对于同一代码库子系统内的子契约和兄弟文件引用，使用相对路径（例如 `./child/${target_filename}` 或 `../sibling/${target_filename}`）。（关键：仅当子目录存在于剪枝后的 `subtree_index` 中且确实包含 `${target_filename}` 时，才生成 `[Child](./child/${target_filename})` 链接。如果子目录包含 `SKILL.md` 或其他非契约资产，请直接链接到 `./child/SKILL.md` 或将其格式化为无链接的列表项。绝不输出指向不存在契约文件的猜测性链接）。
            2.  **绝对链接**：引用包所属子系统外部的文件时（例如跨包依赖），使用标准 URL 或仓库路径。

    *   **Markdown 格式化**：在 `D` 中写入或修正 `target_filename` 后，立即确保 Markdown 格式整洁（合法的标题、表格与代码块围栏）。如果仓库配置了项目级 Markdown 格式化工具（如 `npx prettier --write`），可对 `target_filename` 执行该工具。

3.  **工作节点报告**：分配的子树遍历完成后，整理 `subtree_index` 数组，包含每个已处理目录的 `path`、`summary`、`interface`、`invariants` 和 `side_effects`。

4.  如果 `is_worker` 为 `true`，请先读取相对于本技能目录的 [工作节点返回接口 Schema](../../../skills/egm-bootstrap/references/worker-return.schema.json)，然后向根编排器返回符合该 Schema 的 JSON 载荷并终止。如果 `is_worker` 为 `false`（顺序模式），直接进入**步骤 5：根目录汇总与终止**。

---

### 步骤 5：根目录汇总与终止（根编排器）

1.  **接收工作节点载荷**：收集所有并行子编排器返回的 JSON `subtree_index` 载荷。

    *   **根级容错**：如果某个并行工作节点失败或超时，记录一条警告，在根蓝图的子目录索引中插入占位警告链接（`[Subtree Root] (Documentation failed / Subagent execution timeout)`），并完成其余汇总工作。（工作节点失败后残留的部分文件将被视为增量进度特性；修正策略支持在后续运行中断点续传）。

2.  **中间目录文档化**：识别位于根目录 `path` 与各分区子树根之间的所有中间目录。

    *   按自底向上的顺序串行处理这些中间目录。
    *   利用工作节点返回的汇总摘要，生成中间蓝图契约文件。
    *   确保每个中间蓝图契约文件格式整洁、相对链接合法。

3.  **根目录文档化**：本次执行的目标根路径 `path` *始终*被归类为**仓库根目录（蓝图 / 干草）**。

    *   检查 `path` 中是否已存在 `target_filename`。如果存在，应用**修正与修改策略**读取并保留根契约文件顶部的既有标题、元数据和架构概述。
    *   检查 `path` 自身是否包含直接源代码文件。如果存在，为 `path` 派生一个 `goldfish-explorer` 侦察兵以提取本地符号和不变量，并将其追加到可选的 `## Local Interface & Invariants` 章节中。
    *   使用**仓库根蓝图模板**编译最终的根目录 `target_filename`，汇总所有顶层子目录边界。确保任何根级架构图或时序图都严格匹配子级 `subtree_index` 载荷中的同步/异步边界、操作名称和存储后端，不虚构任何未经验证的节点。
    *   使用文件写入工具写出根目录 `target_filename`。

4.  **最终正确性验证（关键）**：在结束工作之前，你**必须**对 `subtree_index` 中所有新建和修改的契约文件（包括所有子包契约，例如 `pkg1/${target_filename}` 和 `pkg2/${target_filename}`）执行一次最终正确性核查。你**必须**显式读取每一个新建或修改的契约文件以验证其内容，不得跳过任何子契约文件。验证要点：

    *   **EGM 合规性与代码锚定**：确保 `Interface`、`Invariants` 和 `Side Effects` 章节均存在，引用来自侦察提取的真实代码符号/标志/表名，且不含任何猜测性或未经核实的声明。
    *   **Markdown 语法**：确保标题规范、表格合法、代码块围栏闭合正确。
    *   **有效且可解析的链接（关键）**：确保所有子目录和兄弟目录链接都使用合法的相对路径（`./subdir/${target_filename}` 或 `./subdir/SKILL.md`），并显式验证每个被引用的目标文件在文件系统上真实存在。移除或修正所有损坏的相对引用。
    *   **合法目标**：验证测试目标与项目测试配置一致，且使用合法目标语法（对于 Bazel，使用 `/...` 或 `:all`，绝不使用 `:...`）。

5.  **可选漂移评审（关键）**：验证完成后，询问用户（若宿主环境提供结构化提问工具则使用该工具，否则在聊天中询问）是否对本次生成的文档更新运行漂移报告：

    *   `question`："Do you want to run the EGM Drift Report on the generated contract updates to check for consistency with the codebase?"（是否要对本次生成的契约更新运行 EGM 漂移报告，以检查其与代码库的一致性？）
    *   `options`：`["(Recommended) Yes, run drift report on modified contracts", "No, skip drift review"]`（`["（推荐）是，对已修改的契约运行漂移报告", "否，跳过漂移评审"]`）
    *   如果用户选择“是（Yes）”，则针对已修改的契约文件运行 `egm-drift-report` 技能，同时核查遗漏的代码覆盖以及文档到代码的幻觉（将每一个文档记录的方法、路由、枚举、数据库表和不变量与本地源文件逐一比对，并将差异记录在漂移报告中供人类分诊）。（注：虽然 `egm-drift-report` 常用于设计文档，但也可作为变体应用于符合 EGM 规范的契约文件）。

6.  成功终止执行。

---

## 模板

在编写任何契约文件之前，请先阅读相对于本技能目录的 [README 模板](references/templates.md)。
该文件定义了上文引用的**组件包（花生）**、**仓库根目录（蓝图 / 干草）**和**中间蓝图**三种 README 模板。
