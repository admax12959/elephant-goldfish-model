<!--
Copyright 2026 Google LLC.
SPDX-License-Identifier: Apache-2.0
-->

> **原文路径**：`skills/egm-bootstrap/references/templates.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# `egm-bootstrap` README 模板

以下为 `egm-bootstrap` 所编写契约文件的模板。`${target_filename}` 为配置的目标契约文件名（默认为 `README.md`）。

## 📄 组件包（花生 / Peanuts）README 模板

```markdown
# [组件名称]

## Overview
[以散文形式概述该组件的架构角色与目标]

## Interface
- [公共类、函数与符号]
- [子目录列表及指向其契约的相对链接（例如 `./subdir/README.md` 或 `./subdir/${target_filename}`）]

## Invariants
- [开发者与智能体必须维护的状态不变量和调用不变量]

## Side Effects
- [状态变更、网络调用、磁盘访问、锁或其他外部副作用]

## Verification
- Run: `<test-command>`
```

## 📄 仓库根目录（蓝图 / 干草 / Hay）README 模板

```markdown
# [仓库名称] Blueprint

## System Overview
[该子系统的高海拔架构概述]

[可选章节：如果根目录本身包含直接源代码文件]
## Local Interface & Invariants
- [根目录中文件的直接符号、不变量与副作用]

## Subdirectories & System Boundaries
- [子目录名称]: [指向既有子契约文件（如 `./subdir/README.md` 或 `./subdir/${target_filename}`）或 `./subdir/SKILL.md` 的相对链接] - [子契约/职责摘要]

## Global Invariants
- [跨越组件边界的系统级约束]

## Global Side Effects
- [受该子系统影响的共享资源、数据库或外部集成]

## System Verification
- Run: `<test-command>`
```

## 📄 中间蓝图（Intermediate Blueprint）README 模板

```markdown
# [目录名称]

## Overview
[对该目录路径下所归拢的子目录与系统的事实性概述]

## Subdirectories & Boundaries
- [子目录名称]: [指向既有子契约文件（如 `./subdir/README.md` 或 `./subdir/${target_filename}`）或 `./subdir/SKILL.md` 的相对链接] - [子契约/职责摘要]
```
