# Bootstrap Skills

一套 AI 协作框架的脚手架 Skill，基于 [Context Anchoring](https://martinfowler.com/articles/reduce-friction-ai/context-anchoring.html) 最佳实践。

## 核心理念

> 代码记录**结果**，不记录**推理**。被拒绝的替代方案在代码中不可见。

Bootstrap Skill 通过引导用户建立两层上下文体系：

- **Priming Document**（`AGENTS.md`）— 项目级，告诉 AI "这个项目怎么运作"
- **Feature Documents**（`docs/`）— 特性级，告诉 AI "为什么这样决定、进展如何"

**检验标准**：能否无焦虑地关闭当前会话并开启新会话？

## 安装

```bash
npx skills add https://github.com/<your-username>/bootstrap-skills --skill bootstrap
```

## 前置依赖

```bash
# 头脑风暴（bootstrap 使用此 skill 引导用户）
npx skills add https://github.com/obra/superpowers --skill brainstorming

# 发现更多 skills
npx skills add https://github.com/vercel-labs/skills --skill find-skills
```

## 使用

在任意项目目录中，触发 `bootstrap` skill：

```
/bootstrap
```

Bootstrap 将通过头脑风暴引导你完成：

1. **Phase 1**: 头脑风暴 — 交互式梳理项目信息，生成 AGENTS.md
2. **Phase 2**: 脚手架 — 创建 .claude/.qoder/.cursor 目录结构
3. **Phase 3**: 生成 — 创建 Rules、Feature Documents、Agents
4. **Phase 4**: 安装 — 引导安装推荐的社区 Skills
5. **Phase 5**: 锚定 — 验证 Context Anchoring 是否就绪

## 生成的项目结构

```
<your-project>/
├── AGENTS.md                          # Priming Document
├── docs/
│   ├── project.md                     # 架构与约束
│   ├── progress.md                    # 实现状态
│   ├── decisions.md                   # 决策 + 推理 + 被拒绝方案
│   └── open-questions.md              # 待解决问题
├── .claude/                           # Claude Code
│   ├── agents/
│   ├── rules/
│   └── skills/
├── .qoder/                            # Qoder
│   ├── agents/
│   ├── rules/
│   └── skills/
└── .cursor/                           # Cursor
    ├── agents/
    ├── rules/
    └── skills/
```

## 四层协作架构

| 层级 | 职责 | 位置 |
|------|------|------|
| Rules | 强制约束 | `*/rules/*.md` |
| AGENTS.md | 项目知识（Priming） | 项目根目录 |
| Skills | 领域能力 SOP | `*/skills/<name>/SKILL.md` |
| Subagent | 独立任务执行 | `*/agents/<name>.md` |

## 推荐 Skills

安装 bootstrap 后，推荐补充：

```bash
npx skills add https://github.com/obra/superpowers --skill requesting-code-review
npx skills add https://github.com/obra/superpowers --skill test-driven-development
npx skills add https://github.com/obra/superpowers --skill verification-before-completion
npx skills add https://github.com/obra/superpowers --skill systematic-debugging
npx skills add https://github.com/obra/superpowers --skill writing-plans
```

## 本仓库结构

```
bootstrap-skills/
├── bootstrap/
│   └── SKILL.md              # 核心 Bootstrap Skill
├── templates/                 # 模板文件
│   ├── agents/
│   │   ├── code-reviewer.md
│   │   └── explore-agent.md
│   ├── rules/
│   │   └── collaboration.md
│   └── docs/
│       ├── project.md
│       ├── progress.md
│       ├── decisions.md
│       └── open-questions.md
├── adapters/                  # 工具适配说明
│   ├── claude.md
│   ├── qoder.md
│   └── cursor.md
└── README.md
```

## 参考

- [Context Anchoring - Martin Fowler](https://martinfowler.com/articles/reduce-friction-ai/context-anchoring.html)
- [Architecture Decision Records - Michael Nygard (2011)](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [superpowers - obra](https://github.com/obra/superpowers)
- [skills - vercel-labs](https://github.com/vercel-labs/skills)

## License

MIT
