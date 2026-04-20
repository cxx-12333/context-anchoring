# Cursor 适配器

## 目录约定

```
.cursor/
├── agents/                # 自定义 agent
│   └── <name>.md
├── rules/                 # 强制约束
│   └── <name>.md
└── skills/                # 能力模块
    └── <name>/
        └── SKILL.md
```

## AGENTS.md 加载

Cursor 自动读取项目根目录的 `AGENTS.md` 作为项目上下文。

## Rules 格式

Cursor 支持 `.cursorrules` 文件和 `.cursor/rules/` 目录两种方式。

使用目录方式时，格式与 Claude Code / Qoder 一致：

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

Cursor Agent 使用 `.cursor/agents/<name>.md`，格式与其他工具一致。

## Skills 安装

```bash
npx skills add <repo-url> --skill <name>
```

安装到 `.cursor/skills/<name>/SKILL.md`。

## 注意事项

- Cursor 的 Context Indexing 会自动索引代码库，但不能替代 Feature Documents（推理在代码中不可见）
- `.cursorrules` 是旧格式，推荐迁移到 `.cursor/rules/` 目录
