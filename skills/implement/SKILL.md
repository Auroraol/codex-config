---
name: implement
description: 执行 tasks.md 中定义的全部任务，按阶段、依赖、TDD 顺序实现。触发词：implement、执行实现计划、按 tasks 实现、开始实现。
---

执行实现计划，处理并执行 tasks.md 中定义的所有任务。

用户输入（若非空先纳入考虑）：

$ARGUMENTS

1. 在仓库根目录运行 `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks`，解析 FEATURE_DIR 与 AVAILABLE_DOCS，所有路径必须为绝对路径。

2. 加载并分析实现上下文：
   - 必读 tasks.md（完整任务清单与执行计划）
   - 必读 plan.md（技术栈、架构、文件结构）
   - 若存在读 data-model.md（实体与关系）、contracts/（API 规格与测试）、research.md（技术决策与约束）、quickstart.md（集成场景）

3. 解析 tasks.md：任务阶段（Setup/Tests/Core/Integration/Polish）、依赖（串行 vs 并行）、任务细节（ID、描述、文件路径、并行标记 [P]）、执行顺序。

4. 按计划执行：分阶段完成、遵守依赖、并行任务 [P] 可同跑、TDD 先写测试、同文件任务串行、每阶段校验。

5. 实现规则：Setup 先行、测试先于代码、核心开发、集成、打磨校验。

6. 进度与错误处理：每任务完成即报告；非并行任务失败即停；并行任务失败继续成功项并报告失败；给清晰报错；**已完成任务必须在 tasks 文件标记 [X]**。

7. 完成校验：验证全部任务完成、实现与规格一致、测试通过且覆盖达标、遵循技术计划、报告最终状态。

注意：本命令假设 tasks.md 已有完整任务拆解，若缺失建议先运行 `/tasks` 重新生成。
