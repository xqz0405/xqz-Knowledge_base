# Claude Memory

Claude Code 全局记忆系统 — 用户画像、习惯分析、项目进度、自我反思，跨项目、跨机器持久化。

## 架构

```
GitHub (claude-memory)        ← 云端源，换机器 clone 即恢复
       ↕ git push/pull
~/.claude/memory/             ← Claude Code 自动读取（autoMemoryDirectory）
       ↕ sync-memory.mjs
Obsidian Vault/Claude记忆/    ← 可视化浏览、编辑、图谱
```

## 目录结构

```
~/.claude/memory/
├── MEMORY.md                  ← 全局索引（Claude 自动加载）
├── user_profile.md            ← 用户画像（我是谁、仓库、基础设施）
├── setup.mjs                  ← 新机器部署脚本
├── sync-memory.mjs            ← 双向同步 + git push
├── skills/                    ← 自定义斜杠命令
│   ├── startup.md             ← /startup  读取记忆，分析状态
│   ├── reflect.md             ← /reflect  反思更新，写入记忆
│   ├── learn.md               ← /learn    快速记录发现
│   └── status.md              ← /status   项目状态总览
├── 习惯分析/
│   ├── habit_communication.md ← 沟通风格
│   ├── habit_work_pattern.md  ← 工作模式
│   └── habit_decision_pattern.md ← 决策偏好
├── 项目进度/
│   ├── project_xqz_knowledge_base.md
│   ├── project_xqz_astro_site.md
│   ├── project_python_progress.md
│   ├── project_nodejs_complete.md
│   ├── project_site_bugfixes.md
│   ├── project_coc_admin.md
│   └── project_backend_error_handling.md
├── 用户偏好/
│   ├── feedback_content_depth.md
│   ├── feedback_ui_preferences.md
│   └── feedback_agent_usage.md
├── 自我反思/
│   ├── reflection_latest.md   ← 每次会话更新
│   ├── lesson_learned.md      ← 持续追加
│   └── improvement_log.md     ← 版本化追踪
└── 外部参考/
    └── reference_mcp_playwright.md
```

## 快速开始

### 新机器部署

```bash
# 1. clone 记忆仓库
git clone git@github.com:xqz0405/claude-memory.git ~/.claude/memory

# 2. 运行部署脚本
cd ~/.claude/memory
node setup.mjs

# 自动完成：
#   - 记忆文件 → ~/.claude/memory/
#   - Skills   → ~/.claude/skills/
#   - 同步脚本 → ~/claude-memory-sync.mjs
#   - Hook    → ~/.claude/settings.json (SessionStart + Stop 提醒)
#   - autoMemoryDirectory → ~/.claude/settings.json
```

### 日常使用

```bash
# 同步 Claude 记忆 ↔ Obsidian ↔ GitHub
node sync-memory.mjs
# 或
node ~/claude-memory-sync.mjs
```

同步脚本会：
1. `git pull` 拉取远程更新
2. 双向比较 mtime，较新的覆盖较旧的
3. `git add + commit + push` 提交变更

### 斜杠命令

| 命令 | 用途 |
|------|------|
| `/startup` | 读取全局记忆，分析当前状态，输出协作准备摘要 |
| `/reflect` | 回顾本次会话，更新习惯/教训/改进日志 |
| `/learn [内容]` | 快速记录一个新发现 |
| `/status` | 查看所有项目状态总览 |

### Hooks

| 事件 | 行为 |
|------|------|
| `SessionStart` | 提示可用的记忆命令 |
| `Stop` | 提醒 `/reflect` 更新记忆 |

## 记忆类型

| 类型 | 目录 | 何时更新 |
|------|------|----------|
| user | 根目录 | 身份/基础设施变更时 |
| habit | 习惯分析/ | 发现新的用户行为模式时 |
| project | 项目进度/ | 项目进展时 |
| feedback | 用户偏好/ | 被纠正或确认偏好时 |
| reflection | 自我反思/ | 每次会话结束时 |
| reference | 外部参考/ | 发现新工具/配置时 |

## 记忆文件格式

```markdown
---
name: short-kebab-case-slug
description: 一行摘要
metadata:
  type: habit | project | feedback | reflection | reference | user
---

正文内容

**Why:** 为什么记录这条
**How to apply:** Claude 如何应用
```

## Obsidian 可视化

记忆文件同步到 `Claude记忆/` 目录后，在 Obsidian 中可以：

- 打开 `_MOC-记忆索引.md` 导航所有记忆
- 用图谱视图查看 `[[双链]]` 关联
- 直接编辑记忆文件，同步回 Claude
- 用模板创建新的习惯/反思记忆

模板位于 Obsidian vault 的 `templates/` 目录：
- `_模板_记忆-习惯.md`
- `_模板_记忆-反思.md`

## 自增长机制

每次会话中 Claude 应主动：

1. **发现新习惯** → 更新 `习惯分析/habit_*.md`
2. **项目进展** → 更新 `项目进度/project_*.md`
3. **被纠正** → 追加 `自我反思/lesson_learned.md`，更新 `reflection_latest.md`
4. **周期性** → 更新 `自我反思/improvement_log.md` 版本号
5. **新项目** → 创建 `项目进度/project_*.md`，更新 `MEMORY.md`

## 账号

- GitHub: xqz0405
- Gitee: xqz_0405
