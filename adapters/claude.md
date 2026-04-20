# Claude Code 适配器

## 目录约定

```
.claude/
├── settings.json          # 项目级设置（permissions, hooks）
├── agents/                # 自定义 agent 类型
│   └── <name>.md          # Agent 定义（frontmatter + 指令）
├── rules/                 # 强制约束
│   └── <name>.md          # Rule 文件（trigger: always_on）
└── skills/                # 能力模块
    └── <name>/
        └── SKILL.md       # Skill 定义
```

## AGENTS.md 加载

Claude Code 自动读取项目根目录的 `AGENTS.md`，无需额外配置。

## Rules 格式

```markdown
---
trigger: always_on
---

# Rule 标题

## 禁止
- ...

## 必须
- ...
```

## Agent 格式

```markdown
---
name: <agent-name>
description: <何时使用>
tools:
- Bash
- Read
skills:
- <引用的 skill>
---

# Agent 角色描述

<指令内容>
```

## Skills 安装

```bash
npx skills add <repo-url> --skill <name>
```

安装到 `.claude/skills/<name>/SKILL.md`。

## Hooks（settings.json）

```json
{
  "hooks": {
    "afterTaskComplete": "自动更新 docs/progress.md",
    "sessionStart": "读取 docs/progress.md"
  }
}
```

Hooks 通过 Claude Code 的 settings.json 配置，实现自动文档维护。
