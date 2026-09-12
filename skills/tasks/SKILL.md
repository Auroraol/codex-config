---
name: tasks
description: 基于设计产物生成依赖有序、可执行的 tasks.md。触发词：tasks、生成任务清单、拆解任务、任务规划。
---

基于可用设计产物生成依赖有序、可执行的 tasks.md。

用户输入（若非空先纳入考虑）：

$ARGUMENTS

1. 在仓库根目录运行 `.specify/scripts/bash/check-prerequisites.sh --json`，解析 FEATURE_DIR 与 AVAILABLE_DOCS，所有路径绝对。

2. 加载分析可用设计文档：必读 plan.md；若有读 data-model.md、contracts/、research.md、quickstart.md。并非所有项目都有全部文档（CLI 工具可能无 contracts/，简单库可能不需要 data-model.md），按可用内容生成。

3. 按模板生成任务：以 `.specify/templates/tasks-template.md` 为基础，替换为基于 Setup（项目初始化/依赖/linting）、Test [P]（每契约/集成场景一个）、Core（每实体/服务/命令/端点一个）、Integration（DB/中间件/日志）、Polish [P]（单测/性能/文档）的实际任务。

4. 生成规则：每契约文件→[P] 契约测试；每 data-model 实体→[P] 模型任务；每端点→实现任务（共享文件不并行）；每用户故事→[P] 集成测试；不同文件可 [P] 并行，同文件串行。

5. 按依赖排序：Setup 最先、测试先于实现（TDD）、模型先于服务、服务先于端点、核心先于集成、一切先于打磨。

6. 含并行执行示例：把可并行的 [P] 任务分组，给出实际 Task agent 命令。

7. 创建 FEATURE_DIR/tasks.md：正确特性名、编号任务（T001...）、清晰文件路径、依赖说明、并行指导。

Context: $ARGUMENTS

tasks.md 必须立即可执行，每个任务具体到 LLM 无需额外上下文即可完成。
