---
name: bootstrap
description: 在项目中初始化 AI 协作框架。通过头脑风暴引导用户创建 AGENTS.md（Priming Document）和 Feature Documents，搭建 .claude/.qoder/.cursor 多工具目录结构，建立 Context Anchoring 文档体系。核心目标：让 agent 的推理可持久化、可追溯、可自我学习。
---

# Bootstrap - AI 协作框架初始化

> 参考：[Context Anchoring - Martin Fowler](https://martinfowler.com/articles/reduce-friction-ai/context-anchoring.html)

代码记录**结果**，不记录**推理**。被拒绝的替代方案在代码中不可见。

Bootstrap 通过引导用户建立两层上下文体系，解决这个问题：

- **Priming Document**（`AGENTS.md`）— 项目级，稳定，告诉 AI "这个项目怎么运作"
- **Feature Documents**（`docs/`）— 特性级，活跃，告诉 AI "这个功能进展如何、为什么这样决定"

**开始时宣布：** "我正在使用 bootstrap 技能初始化 AI 协作框架。"

---

## 核心理念

### 为什么需要 Context Anchoring？

AI 的上下文窗口有上限。随着会话变长：
- **推理比决策先消失** — AI 记得"用 PostgreSQL"，但忘了"为什么选 PostgreSQL 而不是 MongoDB"
- **中间信息最先丢失** — "Lost in the Middle" 研究证实，位于上下文中部的信息召回率最低
- **压缩会丢弃推理** — 自动压缩保留"做了什么"，丢弃"为什么这样做"

**恶性循环**：开发者不敢关闭会话 → 会话越来越长 → AI 回忆早期决策的能力持续退化

**破解方式**：将决策上下文外化为文档，作为跨会话的权威参考。

### 检验标准（The Litmus Test）

> **能否无焦虑地关闭当前会话并开启新会话？**

如果答案是"是" — 上下文已正确锚定。
如果答案是"否" — 还有决策只存在于对话中，没有持久化。

### 适用范围

| 场景 | 需要锚定？ | 为什么 |
|------|-----------|--------|
| 快速问题、单文件工具 | 不需要 | 会话短，衰减无关紧要 |
| 单次会话功能（<1小时） | 轻量级 | 几个要点即可 |
| 跨天多会话功能 | 完整文档 | 丢失上下文的代价是小时级 |
| 多人协作功能 | 共享文档 | 协调独立 AI 会话间的决策 |

---

## 架构总览

初始化后，项目拥有两层上下文 + 三层协作配置：

```
<project-root>/
├── AGENTS.md                          # Priming Document（项目级上下文）
│                                      # "这个项目怎么运作"
│                                      # 稳定，季度级更新
│
├── docs/                              # Feature Documents（特性级上下文）
│   ├── project.md                     # 架构与约束
│   ├── progress.md                    # 实现状态（活文档）
│   ├── decisions.md                   # 决策 + 推理 + 被拒绝方案
│   └── open-questions.md              # 待解决问题
│                                      # "这个功能进展如何，为什么这样决定"
│                                      # 活跃，每次会话可能更新
│
├── .claude/                           # Claude Code 工具配置
│   ├── agents/ <name>.md             # Subagent 定义
│   ├── rules/ <name>.md             # 强制约束（Always Apply）
│   └── skills/ <name>/SKILL.md      # 能力模块（通过 npx 安装）
│
├── .qoder/                            # Qoder 配置（同构）
│   ├── agents/
│   ├── rules/
│   └── skills/
│
└── .cursor/                           # Cursor 配置（同构）
    ├── agents/
    ├── rules/
    └── skills/
```

### 文档的职责边界

| 文档 | 是什么 | 不是什么 | 类比 |
|------|--------|----------|------|
| `AGENTS.md` | 项目词汇表 — 技术栈、架构、命名约定 | 不写约束、不写决策 | 字典 |
| `docs/decisions.md` | 决策 + 推理 + 被拒绝方案 | 不写架构描述 | 审判记录 |
| `docs/progress.md` | 实现状态 — 完成了什么、还剩什么 | 不写为什么 | 清单 |
| `docs/project.md` | 架构设计、技术约束 | 不写协作流程 | 蓝图 |
| `docs/open-questions.md` | 待解决问题 | 不写已解决的问题（归 decisions） | 待办 |

### 四层协作架构

| 层级 | 职责 | 加载方式 | 比喻 |
|------|------|----------|------|
| **Rules** | 强制约束（禁止/必须） | Always Apply | 法律 |
| **AGENTS.md** | 项目知识（Priming） | 自动读取 | 宪法 |
| **Skills** | 领域能力 SOP | 自动识别 / 手动触发 | 手册 |
| **Subagent** | 独立任务执行 | Task tool | 专家 |

**冲突优先级：** Rules > AGENTS.md > Skills > Subagent

---

## 前置条件

在运行此 skill 之前，确保已安装：

```bash
# 头脑风暴（bootstrap 依赖此 skill 引导用户）
npx skills add https://github.com/obra/superpowers --skill brainstorming

# 发现更多 skills
npx skills add https://github.com/vercel-labs/skills --skill find-skills
```

---

## 初始化流程

### Phase 1: 头脑风暴 — 引导用户构建 AGENTS.md

**这是整个 bootstrap 最核心的阶段。AGENTS.md 不是模板填充，是通过引导式提问让用户自己梳理出项目知识。**

使用 `AskUserQuestion` 工具，按维度逐一与用户对话。每个维度先发散再收敛。

#### 维度 1：项目身份

提问方向：
- "这个项目解决什么问题？如果用一句话向新同事介绍，你会怎么说？"
- "项目的核心价值是什么？它最不可替代的部分是什么？"
- "项目现在处于什么阶段？（想法 / MVP / 迭代中 / 稳定维护）"

**输出 → AGENTS.md 的标题、描述、项目概述**

#### 维度 2：技术画像

提问方向：
- "用了什么语言和框架？为什么选它们？"
- "构建、测试、运行的命令是什么？"
- "有哪些关键第三方依赖？它们各自解决什么问题？"
- "项目的技术约束是什么？（比如必须纯 Go、不能用 CGO）"

提供 2-4 个选项帮助用户思考：
- 构建方式：Makefile / 脚本 / 直接 go build / npm scripts
- 测试方式：go test / pytest / jest / 无
- 包管理：go modules / npm / pip

**输出 → AGENTS.md 的技术栈表格、构建与运行、核心接口**

#### 维度 3：架构地图

提问方向：
- "项目的目录是怎么组织的？每个目录的职责是什么？"
- "有哪些核心接口或抽象？它们的契约是什么？"
- "数据怎么流动的？（从输入到输出的完整路径）"
- "项目用了什么设计模式？（插件、工厂、管道...）"

如果项目已存在，可以先用 `Glob` 和 `Read` 探索现有结构，然后向用户确认理解。

**输出 → AGENTS.md 的项目结构、核心接口、API 端点**

#### 维度 4：协作方式

提问方向：
- "你用哪些 AI 编程工具？（Claude Code / Qoder / Cursor）"
- "有哪些行为是绝对禁止的？为什么？"
- "有哪些流程是每次必须执行的？"
- "你希望 AI 在遇到不确定的事情时怎么做？"

**输出 → Rules 文件的约束内容、工具目录选择**

#### 维度 5：知识管理

提问方向：
- "项目里最容易被忘记的知识是什么？"
- "有哪些决策是经过反复讨论才确定的？"
- "有没有被否决但可能被重新考虑的方案？"
- "你希望下次会话时，AI 第一时间知道什么？"

**输出 → docs/ 文档的初始内容、AGENTS.md 的当前状态**

#### 收敛确认

每个维度完成后，向用户展示提炼出的内容并确认：
- "以上理解是否准确？"
- "有没有遗漏的重要信息？"

所有维度完成后，展示完整的 AGENTS.md 草稿，请用户最终确认。

---

### Phase 2: 脚手架 — 创建目录结构

根据 Phase 1 的结果，创建目录：

```bash
# Feature Documents（所有项目必须）
mkdir -p docs

# 只创建用户实际使用的工具目录
# 以下是全部三种，按需选择
mkdir -p .claude/agents .claude/rules .claude/skills
mkdir -p .qoder/agents .qoder/rules .qoder/skills
mkdir -p .cursor/agents .cursor/rules .cursor/skills
```

---

### Phase 3: 生成 — 创建核心文件

#### 3.1 AGENTS.md（Priming Document）

**从 Phase 1 头脑风暴结果生成。** 不是模板填充，是用户通过引导梳理出的项目知识。

结构：

```markdown
# <项目名>

<一句话描述 — 从维度 1 生成>

## 项目概述

**核心特性**：
- <特性 — 从维度 1、2 生成>

## Context Anchoring（跨会话上下文锚定）

本项目采用 Context Anchoring 实现跨会话上下文保持。

### 两层上下文

| 层级 | 文档 | 职责 | 更新频率 |
|------|------|------|----------|
| Priming | `AGENTS.md`（本文件） | 项目级上下文：技术栈、架构、约定 | 季度级 |
| Feature | `docs/*` | 特性级上下文：决策、进度、问题 | 每次会话 |

### Feature Documents

| 文档 | 职责 | 内容 |
|------|------|------|
| `docs/decisions.md` | Decisions | 决策、理由、**被拒绝的替代方案** |
| `docs/progress.md` | State | 实现进度、任务清单 |
| `docs/project.md` | Constraints | 技术约束、架构设计 |
| `docs/open-questions.md` | Open Questions | 待解决问题 |

**检验标准**：能否无焦虑地关闭当前会话并开启新会话？

## 构建与运行

<从维度 2 生成>

## 项目结构

<从维度 3 生成>

## 核心接口

<从维度 3 生成>

## 技术栈

<从维度 2 生成>

## 已安装 Skills

| Skill | 用途 | 安装来源 |
|-------|------|----------|

## 可用 Subagent

| Agent | 用途 | 引用 Skills |
|-------|------|-------------|

## 决策记录模板

### D0XX: [决策标题]

- **日期**: YYYY-MM-DD
- **决策**: [决策内容]
- **理由**: [为什么这样决定 — 这是防止推理丢失的关键]
- **被拒绝的替代方案**: [考虑过但未采用的方案 — 这是 Context Anchoring 的核心价值]
- **影响范围**: [受影响的文件/模块]
- **状态**: ✅ 已确认 / 🔄 讨论中 / ❌ 已废弃

## 当前状态

- **阶段**: <当前阶段>
- **里程碑**: <当前里程碑>
- **下一步**: <下一步计划>
```

**关键差异**：决策模板中把"原方案"和"变更原因"合并为"理由"，强调的是**推理的持久化**，而非决策本身。

#### 3.2 Rules 文件

为每个使用的工具创建 rules 文件。

**Rules 只写"约束"，不写知识。** 约束归 Rules，知识归 AGENTS.md。

文件：`<tool>/rules/<project-name>-collaboration.md`

```markdown
---
trigger: always_on
---

# <项目名> 强制约束

## 禁止

- <从维度 4 的"绝对禁止"生成>

## 必须

- 任务完成立即更新 `docs/progress.md`
- 技术决策立即记录 `docs/decisions.md`（必须包含推理和被拒绝方案）
- 会话开始先读 `docs/progress.md`
- <从维度 4 的"必须执行"生成>
```

#### 3.3 Feature Documents

**docs/project.md** — 从维度 2、3 生成（架构约束）：
```markdown
# <项目名> - 架构与约束

## 核心设计原则
<从维度 2、3 提取>

## 系统架构
<架构图>

## 技术选型与理由
| 技术 | 选型 | 为什么 | 被拒绝的替代 |
|------|------|--------|-------------|

## 核心模块
<模块说明>

## 目录结构
<目录树>
```

**docs/progress.md** — 初始化状态：
```markdown
# <项目名> - 实现状态

## 当前里程碑: <里程碑名>

### 完成项
- [x] 协作框架初始化

### 进行中
（暂无）

### 待开始
（从维度 5 的"下一步"生成）

## 里程碑

| 里程碑 | 目标 | 状态 |
|--------|------|------|
| M0: 协作框架 | Context Anchoring + 四层架构 | ✅ |
```

**docs/decisions.md** — 记录初始化决策：
```markdown
# <项目名> - 决策记录

> 代码记录结果，不记录推理。此文档持久化决策背后的推理。

## 决策

### D001: 采用 Context Anchoring 协作框架

- **日期**: YYYY-MM-DD
- **决策**: 采用两层上下文 + 四层协作架构
- **理由**: AI 上下文窗口有限，推理比决策先消失。将决策上下文外化为文档，作为跨会话权威参考。
- **被拒绝的替代方案**:
  - 仅 AGENTS.md：无法持久化特性级决策和推理
  - 仅 Memory：不稳定，跨工具不可用，内容不可审计
  - 不使用文档：推理只在对话中，会话结束即丢失
- **影响范围**: 全局协作流程
- **状态**: ✅ 已确认
```

**docs/open-questions.md** — 从维度 5 生成：
```markdown
# <项目名> - 待解决问题

## 待解决
（从维度 5 的"容易被忘记的知识"和"未确定事项"生成）

## 已解决
（暂无）
```

#### 3.4 Subagent（可选）

询问用户是否需要创建常用 subagent，生成到所有工具的 `agents/` 目录下。

推荐：
| Agent | 用途 |
|-------|------|
| `code-reviewer` | 独立上下文的代码审查 |
| `explore-agent` | 代码库探索与定位 |

---

### Phase 4: 安装 — 补充 Skills

展示推荐 Skills，让用户选择需要哪些：

| Skill | 用途 | 安装命令 |
|-------|------|----------|
| `brainstorming` | 结构化头脑风暴 | `npx skills add https://github.com/obra/superpowers --skill brainstorming` |
| `requesting-code-review` | 代码审查 SOP | `npx skills add https://github.com/obra/superpowers --skill requesting-code-review` |
| `test-driven-development` | TDD 红-绿-重构 | `npx skills add https://github.com/obra/superpowers --skill test-driven-development` |
| `verification-before-completion` | 完成前验证 | `npx skills add https://github.com/obra/superpowers --skill verification-before-completion` |
| `systematic-debugging` | 系统化调试 | `npx skills add https://github.com/obra/superpowers --skill systematic-debugging` |
| `writing-plans` | 编写实现计划 | `npx skills add https://github.com/obra/superpowers --skill writing-plans` |

安装后：
1. 根据项目语言适配 Skill 中的命令和示例
2. 更新 AGENTS.md 的"已安装 Skills"表格

也可以使用 `find-skills` skill 搜索和发现更多 Skills。

---

### Phase 5: 锚定 — 验证与沉淀

#### 5.1 验证清单

- [ ] `AGENTS.md` 存在 — Priming Document 就绪
- [ ] `docs/` 四份文档已创建 — Feature Documents 就绪
- [ ] 所选工具目录已创建（agents/rules/skills）
- [ ] Rules 文件已配置为 Always Apply
- [ ] brainstorming skill 已安装
- [ ] 决策 D001 已记录（含推理和被拒绝方案）

#### 5.2 Litmus Test

向用户确认：
> **"能否无焦虑地关闭当前会话并开启新会话？"**

- 如果"是" → 初始化完成。新会话将自动读取 AGENTS.md + docs/，30 秒恢复完整上下文。
- 如果"否" → 追问"担心丢失什么？"，将遗漏的内容补充到对应文档。

#### 5.3 使用指南

告知用户：
- **每次会话开始**：AI 自动读取 AGENTS.md 和 docs/，从文档状态开始工作
- **每次决策**：立即记录到 docs/decisions.md，必须包含推理
- **每个任务完成**：更新 docs/progress.md
- **每个新问题**：记录到 docs/open-questions.md
- **架构变更**：更新 docs/project.md 和 AGENTS.md
- **季度回顾**：检查 AGENTS.md 是否需要更新

---

## 边界原则

| 层级 | 只写什么 | 不写什么 |
|------|----------|----------|
| **Rules** | 约束（禁止/必须） | 知识（架构/接口） |
| **AGENTS.md** | 知识（技术栈/架构/约定） | 约束（禁止/必须） |
| **Skills** | SOP（如何做某类事） | 项目特定内容 |
| **Subagent** | 执行指令（做什么任务） | Skill 内容（通过 skills 字段引用） |

## 设计决策背后的推理

| 决策 | 理由 |
|------|------|
| AGENTS.md 通过头脑风暴生成而非模板填充 | 强制用户思考"我希望 AI 知道什么"，比模板更有价值 |
| 决策模板包含"被拒绝的替代方案" | 代码中不可见的信息，防止 AI 重新提出已被否决的方案 |
| Feature Documents 实时维护而非事后补充 | 推理在对话进行时最清晰，事后补充会丢失细节 |
| Rules 与 AGENTS.md 分离 | 约束需要 Always Apply 强制执行，知识只需要自动读取 |
| 多工具同构目录 | 同一份 agent/rule 内容适配不同工具格式，降低维护成本 |
