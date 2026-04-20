# Qoder 适配器

## 目录约定

```
.qoder/
├── agents/                # Subagent 定义
│   └── <name>.md
├── rules/                 # 强制约束
│   └── <name>.md
└── skills/                # 能力模块
    └── <name>/
        └── SKILL.md
```

## AGENTS.md 加载

Qoder 自动读取项目根目录的 `AGENTS.md`。

## Rules 格式

与 Claude Code 相同，使用 frontmatter `trigger: always_on`。

在 IDE 中确认：Qoder Settings → Rules → 确认类型为 Always Apply。

## Agent 格式

与 Claude Code 相同格式，额外支持 MCP 工具：

```markdown
---
name: <agent-name>
description: <何时使用>
tools:
- Bash
- Read
- mcp__quest__search_codebase
skills:
- <引用的 skill>
---
```

## Skills 安装

```bash
npx skills add <repo-url> --skill <name>
```

安装到 `.qoder/skills/<name>/SKILL.md`。
