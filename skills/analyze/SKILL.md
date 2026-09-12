---
name: analyze
description: 交叉校验 spec.md、plan.md、tasks.md 三份产物的不一致、重复、歧义与遗漏。触发词：analyze、一致性分析、校验规格、分析 spec/plan/tasks。
---

对 spec.md、plan.md、tasks.md 三份核心产物做非破坏性的交叉一致性分析（用户输入可为空，若非空则先纳入考虑）。

用户输入：

$ARGUMENTS

目标：在实现前识别三份产物（spec.md、plan.md、tasks.md）之间的不一致、重复、歧义与未充分定义项。本命令必须仅在 `/tasks` 成功产出完整 tasks.md 之后运行。

严格只读：不得修改任何文件，输出结构化分析报告，可选提供补救计划（后续编辑命令须经用户明确批准后手动执行）。

宪章权威：项目宪章（`.specify/memory/constitution.md`）在本分析范围内不可协商。与宪章冲突自动判定为 CRITICAL，需要调整 spec/plan/tasks，而非稀释、重新解释或静默忽略。若某原则本身需修改，须在 `/analyze` 之外单独更新宪章。

执行步骤：

1. 在仓库根目录运行一次 `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks`，解析 JSON 得到 FEATURE_DIR 与 AVAILABLE_DOCS，推导绝对路径：SPEC = FEATURE_DIR/spec.md，PLAN = FEATURE_DIR/plan.md，TASKS = FEATURE_DIR/tasks.md。任一必需文件缺失即报错中止（提示用户运行缺失的前置命令）。

2. 加载产物：解析 spec.md（概述/上下文、功能需求、非功能需求、用户故事、边界情况）、plan.md（架构/技术栈、数据模型引用、阶段、技术约束）、tasks.md（任务 ID、描述、阶段分组、并行标记 [P]、文件路径引用）、宪章 `.specify/memory/constitution.md`。

3. 建立内部语义模型：需求清单（每个功能/非功能需求用稳定 key，如 "User can upload file" → `user-can-upload-file`）、用户故事/动作清单、任务覆盖映射、宪章规则集。

4. 检测：A 重复、B 歧义（模糊形容词、未解决占位符）、C 未充分定义、D 宪章对齐、E 覆盖缺口、F 不一致（术语漂移、数据实体不一致、任务排序矛盾、需求冲突）。

5. 严重级启发式：CRITICAL / HIGH / MEDIUM / LOW。

6. 生成 Markdown 报告（不写文件），含 Specification Analysis Report 表格（ID/Category/Severity/Location/Summary/Recommendation）、Coverage Summary、Constitution Alignment Issues、Unmapped Tasks、Metrics（需求总数、任务总数、覆盖率、歧义数、重复数、CRITICAL 数）。

7. 报告末尾输出 Next Actions：存在 CRITICAL 建议先解决再 `/implement`；仅 LOW/MEDIUM 可继续并给改进建议；给出明确命令建议。

8. 询问用户是否要对前 N 个问题给出具体补救编辑（不得自动应用）。

行为规则：绝不修改文件；绝不臆造缺失章节；保持确定性（重跑结果一致）；主表最多 50 条，其余聚合说明；零问题时输出含覆盖统计的成功报告。

Context: $ARGUMENTS
