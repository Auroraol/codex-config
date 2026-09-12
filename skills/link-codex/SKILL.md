---
name: link-codex
description: 把本仓库（codex-config）的 config.toml / AGENTS.md / skills 链接到 Codex 主目录 ~/.codex，让改动实时生效，不再手动复制。触发词：link-codex、链接配置、同步 skills、skills 不更新、配置不生效、junction、符号链接。
---

把 `D:\Github\xxx\codex-config` 挂到 `C:\Users\<你>\.codex`。**核心：文件用符号链接，目录用 junction。**

| 目标 | 方式 | 为什么 |
|---|---|---|
| `~/.codex/config.toml` | 符号链接（SymbolicLink） | 单文件，跨盘只能这样 |
| `~/.codex/AGENTS.md` | 符号链接 | 同上 |
| `~/.codex/skills/` | **junction**（`/J`） | 跨盘可用，且**不需要管理员/开发者模式** |

符号链接（含目录）在 Windows 上要开发者模式或管理员权限；junction 不要。所以目录一律用 junction。

## 前置检查

先确认现状，别盲目覆盖：

```bash
# 看是不是已经是链接（LinkType 为空 = 真实目录，没链上）
powershell -NoProfile -Command "Get-Item 'C:\Users\16658\.codex\skills' -Force | Select Name, LinkType, Target"

# 枚举差异：哪些目录只存在于 ~/.codex、哪些只存在于仓库
ls "C:/Users/16658/.codex/skills" > /tmp/a.txt
ls "D:/Github/1/codex-config/skills" > /tmp/b.txt
comm -23 <(sort /tmp/a.txt) <(sort /tmp/b.txt)   # 只在 ~/.codex —— 必须搬走
comm -13 <(sort /tmp/a.txt) <(sort /tmp/b.txt)   # 只在仓库 —— 链接后自动可见
```

## ⚠️ 建 junction 前必须做的一步

**junction 会完全遮蔽目标目录里原有的东西。** 凡是只在 `~/.codex/skills/` 里的目录（本仓库常见：`.system`、`xd-doc`），链接后全部消失。

所以顺序永远是：**先把独有的搬进仓库 → 再改名备份 → 最后建链接**。

`.system/` 是 Codex CLI 自己写入并随版本重建的系统技能目录，搬进仓库后**必须加进 `.gitignore`**。

## 执行

```bash
set -e
S="C:/Users/16658/.codex/skills"
R="D:/Github/1/codex-config/skills"

# 1. 独有的搬进仓库
mv "$S/xd-doc"  "$R/"
mv "$S/.system" "$R/"

# 2. 旧目录改名备份（不要 rm，留退路）
mv "$S" "C:/Users/16658/.codex/skills.bak"

# 3. 建 junction
powershell -NoProfile -Command "New-Item -ItemType Junction -Path 'C:\Users\16658\.codex\skills' -Target 'D:\Github\1\codex-config\skills'"
```

单文件（`config.toml`、`AGENTS.md`）用符号链接。**必须用 cmd 的 `mklink`，不要用 PowerShell 的 `New-Item`**（原因见下方踩坑）：

```bash
# 先备份，再建链接（mklink 不会覆盖已存在的文件）
mv "C:/Users/16658/.codex/AGENTS.md" "C:/Users/16658/.codex/AGENTS.md.bak"

cmd //c "cd /d C:\Users\16658\.codex && mklink AGENTS.md D:\Github\1\codex-config\AGENTS.md"

# 校验
powershell -NoProfile -Command "\$i = Get-Item 'C:\Users\16658\.codex\AGENTS.md' -Force; \$i.LinkType + ' -> ' + \$i.Target"
```

**链接前先 diff**，确认 `~/.codex` 里那份没有仓库缺失的改动——有的话先合并回仓库，否则一链就覆盖没了：

```bash
diff "C:/Users/16658/.codex/AGENTS.md" "D:/Github/1/codex-config/AGENTS.md" && echo "一致，可安全替换"
```

## 验证（必做，别只看"链接建成了"）

写一个探针文件，确认双向实时：

```bash
echo probe > "D:/Github/1/codex-config/skills/debug/__sync_probe.txt" && sleep 1
cat "C:/Users/16658/.codex/skills/debug/__sync_probe.txt" && echo "REALTIME OK"
rm -f "D:/Github/1/codex-config/skills/debug/__sync_probe.txt"
```

## 踩过的坑

- **PowerShell 的 `New-Item -ItemType SymbolicLink` 会报「此操作需要管理员权限」，但 cmd 的 `mklink` 直接成功。** 同一个非提权会话里，两者权限行为不一样——**用 `mklink`**。目录的 junction 则是两个都能建。
- **`mklink` 不覆盖已存在的文件**，报「文件已存在」。必须先 `mv` 备份走。
- **验证符号链接不要用 `diff 链接路径 vs 源路径`** —— 那是在拿文件和自己比，永远返回「一致」，等于没验证。要看 `LinkType` 是不是 `SymbolicLink` / `Junction`。
- **Git Bash 里 `cmd //c 'mklink /J "..." "..."'` 会报「语法不正确」**（引号被 shell 吞了）。改用 PowerShell 的 `New-Item -ItemType Junction`，一次成功。
- **Git Bash 对 junction 的 `ls | wc -l` 会返回 1**（列出不完整），是 Git Bash 的 bug，别被吓到。用 `cmd //c "dir /b <路径>"` 看才准。
- **重命名仓库里的目录可能 Permission denied** —— IDE（JetBrains）在索引时锁着。关掉 IDE 再改；不改名通常不影响功能，Codex 认的是 `SKILL.md` 里的 `name:`。
- **改完要重启 codex CLI**，它启动时才扫描 skills 目录。
- 确认无误后再删备份：`rm -rf "C:/Users/16658/.codex/skills.bak"`。
