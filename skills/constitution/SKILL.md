---
name: constitution
description: 从交互或提供的原则输入创建或更新项目宪章，保持依赖模板同步。触发词：constitution、更新宪章、项目原则、治理原则。
---

从交互或提供的原则输入创建或更新项目宪章，确保所有依赖模板保持同步。

用户输入（若非空先纳入考虑）：

$ARGUMENTS

你正在更新 `.specify/memory/constitution.md` 的项目宪章。该文件是含方括号占位符（如 `[PROJECT_NAME]`、`[PRINCIPLE_1_NAME]`）的模板。任务是 (a) 收集/推导具体值，(b) 精确填充模板，(c) 将修订传播到依赖产物。

执行流程：

1. 加载 `.specify/memory/constitution.md` 现有模板，识别所有 `[ALL_CAPS_IDENTIFIER]` 占位符。用户可能要求比模板更多或更少的原则，按指定数量遵循。

2. 收集/推导占位符值：用户输入有则用之，否则从仓库上下文推断；RATIFICATION_DATE 用原始采用日期（未知则问或标 TODO），LAST_AMENDED_DATE 若有变更则为今天；CONSTITUTION_VERSION 按语义化版本递增（MAJOR 向后不兼容，MINOR 新增原则/段，PATCH 澄清/错字），版本类型有歧义时先给理由。

3. 起草更新内容：替换每个占位符，保留标题层级，每个原则段含简洁名称行 + 不可协商规则段落/要点 + 理由，Governance 段含修订流程、版本策略、合规评审预期。

4. 一致性传播清单：读 plan-template/spec-template/tasks-template 及 `.specify/templates/commands/*.md`，确认无过时引用（如仅 CLAUDE 的特定名），更新 README 等运行时文档中被改原则的引用。

5. 产出 Sync Impact Report（作为 HTML 注释置于宪章文件顶部）：版本变更、修改的原则、新增/删除的段、需更新模板（✅/⚠ 带路径）、延迟的 TODO。

6. 最终校验：无未解释的方括号 token、版本行与报告一致、日期 ISO 格式、原则声明式可测试无模糊语言。

7. 写回 `.specify/memory/constitution.md`（覆盖）。

8. 输出最终摘要：新版本与理由、需手动跟进的文件、建议提交信息。

格式与风格：Markdown 标题严格按模板；长行保持 <100 字符但不硬断；段间空行；无尾随空白。用户只给部分更新也照常校验与定版。关键信息缺失用 `TODO(<FIELD>): 说明` 并列入延迟项。始终操作现有文件，不新建模板。
