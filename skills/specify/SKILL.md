---
name: specify
description: 从自然语言特性描述创建或更新特性规格（跑 create-new-feature 脚本写 spec）。触发词：specify、写规格、创建规格文档、feature spec。
---

从自然语言特性描述创建或更新特性规格。

用户输入（若非空先纳入考虑）：

$ARGUMENTS

触发消息中 `/specify` 之后的文本即特性描述，假设始终可用（即使 `$ARGUMENTS` 字面出现），除非命令为空否则不要用户重复。

基于该特性描述：

1. 在仓库根目录运行 `.specify/scripts/bash/create-new-feature.sh --json "$ARGUMENTS"`，解析 JSON 得到 BRANCH_NAME 与 SPEC_FILE，所有路径必须绝对。**只能运行此脚本一次**，始终以终端 JSON 输出为准。

2. 加载 `.specify/templates/spec-template.md` 了解必需章节。

3. 用模板结构把规格写到 SPEC_FILE，用特性描述推导的具体内容替换占位符，保持章节顺序与标题。

4. 报告完成：分支名、规格文件路径、下一阶段就绪状态。

注意：脚本会创建并切出新分支并初始化规格文件后再写入。
