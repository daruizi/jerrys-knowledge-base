# Claude Code - Skills 从入门到精通

> Skills 是扩展 Claude Code 能力的核心机制，本教程将带你从零开始掌握 Skills 的配置、开发技巧以及与项目级上下文的协同工作。
>
> **本教程基于 Claude Code 最新规范编写。**

---

## 📖 目录

1. [什么是 Skills？](#什么是-skills)
2. [Skills 核心概念](#skills-核心概念)
3. [内置 Skills 介绍](#内置-skills-介绍)
4. [创建你的第一个 Skill](#创建你的第一个-skill)
5. [Skill 高级配置与 Global Variables](#skill-高级配置与-global-variables)
6. [Hooks 系统：自动化拦截](#hooks-系统自动化拦截)
7. [高级模式：Implementation Plan 模式](#高级模式implementation-plan-模式)
8. [Skills 与 CLAUDE.md 的协同](#skills-与-claudemd-的协同)
9. [分享与分发 Skills](#分享与分发-skills)
10. [常见问题与排错](#常见问题与排错)

---

## 什么是 Skills？

### 一句话解释

**Skills 就像是给 Claude 写的"专业操作手册"** —— 当你定义一个 Skill，Claude 会将其能力加入工具箱。你可以通过 `/命令名` 直接调用，或者让 Claude 在符合场景时自动触发。

### Skills vs MCP vs CLAUDE.md

这是最容易混淆的三个概念，让我们通过下表明确它们的定位：

| 特性 | Skills | MCP (Protocol) | CLAUDE.md |
|------|--------|-----|----------|
| **本质** | **行为与指令** | **工具与数据源** | **静态上下文** |
| **主要功能** | 定义流程、规范输出、批量任务 | 连接数据库、API、外部文件 | 定义项目规范、索引、偏好 |
| **动态性** | 高（支持命令注入、Hooks） | 高（实时查询外部数据） | 低（仅作为参考信息） |
| **典型场景** | 代码审查流程、PR 总结、代码迁移 | 执行 SQL、查询 Jira、发送邮件 | 命名规范说明、架构设计速览 |

---

## Skills 核心概念

### 关键发现：Skill 的名称如何确定？

> [!IMPORTANT]
> 在 Claude Code 中，Skill 的手动调用命令名（如 `/my-skill`）是由该 Skill 所在的**文件夹名称**决定的，而 **不是** `SKILL.md` 文件内的 `name` 字段（该字段仅作为元数据）。

### 存储位置与作用域

| 类型 | 路径 | 适用范围 |
|------|------|----------|
| **个人级** | `~/.claude/skills/<skill-name>/SKILL.md` | 你在当前机器上打开的所有项目 |
| **项目级** | `.claude/skills/<skill-name>/SKILL.md` | 仅当前项目（推荐提交到 Git 共享） |

**优先级**：项目级配置会覆盖同名的个人级配置。

---

## 内置 Skills 介绍

Claude Code 预装了几个顶级 Skills，你可以直接在命令行使用：

- **`/simplify`**：并行启动多个代理来优化代码质量、减少重用并提升性能。
- **`/batch`**：大规模变更神器。它会先制定计划，获批后开启多个后台线程并发执行修改并自动运行测试。
- **`/debug`**：用于调试当前 Claude 会话本身（例如某个工具调用为何一直失败）。
- **`/loop [时间]`**：定时执行任务。例如 `/loop 5m run tests`。

---

## 创建你的第一个 Skill

### 场景：团队代码审查 (Code Review)

我们可以创建一个 Skill，让 Claude 按照特定的清单来审查代码。

#### 1. 创建目录
```bash
mkdir -p .claude/skills/cr
```

#### 2. 编写 SKILL.md
创建 `.claude/skills/cr/SKILL.md`：

````markdown
---
description: 按照团队质量标准审查指定代码。当用户需要 Review 代码或提交 PR 前使用。
---

# 🚀 团队代码审查指令

审查目标：$ARGUMENTS

## 审查重点
1. **安全性**：是否有未验证的输入？是否有敏感信息泄露？
2. **性能**：是否有 N+1 查询？循环中是否有昂贵操作？
3. **一致性**：是否遵循了项目根目录 `CLAUDE.md` 中的命名规范？

## 输出要求
请直接列出发现的问题，并为每个问题提供一条修复建议。
````

#### 3. 使用方式
直接在终端输入：`/cr src/auth/`
或者直接问：`帮我 Review 下权限部分的代码`（此时 Claude 会根据 `description` 自动匹配到该 Skill）。

---

## Skill 高级配置与 Global Variables

### 全局变量 (Global Variables)

除了常用的 `$ARGUMENTS`（或 `$0`, `$1`），Claude Code 还提供了一个非常强大的变量：

- **`$CORPUS_CONTENT`**：这是当前项目（语料库）的元数据摘要。它包含项目根目录结构、主要文件列表以及 Claude 对项目的初步理解。
- **用途**：当你编写一个需要理解全局架构的 Skill 时，引用此变量可以显著提升 Claude 的决策准确度。

### Description 编写指南（优化自动触发）

`description` 是 Claude 决定何时调用该 Skill 的唯一依据。
- **❌ 错误写法**：`description: 代码解释`（太模糊）
- **✅ 正确写法**：`description: 当用户询问特定函数的工作原理，或需要可视化展示逻辑流程时使用。`

---

## Hooks 系统：自动化拦截

Skills 支持钩子系统，让你可以在工具执行的前后插入逻辑。

### PreToolUse 钩子

在 Claude 使用任何工具（如 `replace_file_content`）之前执行。

**示例：自动备份钩子**
```yaml
---
name: auto-backup
description: 在修改文件前自动创建备份
hooks:
  PreToolUse:
    - command: "cp $ARGUMENTS[0] $ARGUMENTS[0].bak"
---
```

### PostToolUse 钩子

在工具执行成功后触发。常用于执行 Lint 检查或自动运行受影响的测试。

---

## 高级模式：Implementation Plan 模式

在进行复杂重构时，我们不希望 Claude 拿起键盘就改。你可以通过 Skill 强制其遵循**"先计划，后执行"**的模式。

````markdown
---
description: 执行复杂的代码重构。
disable-model-invocation: true
---

# 重构协议

1. **第一阶段：收集 context**。使用 Grep 和 Read 了解相关依赖。
2. **第二阶段：撰写 implementation_plan.md**。在项目根目录下生成此文件，列出所有待改动的点。
3. **第三阶段：请求确认**。停止执行，等待用户查看该文件并输入 "Proceed"。

除非用户明确说 "Proceed"，否则严禁直接调用修改工具。
````

---

## Skills 与 CLAUDE.md 的协同

这是进阶开发者的必杀技：

1.  **CLAUDE.md 定义规则**：在 `CLAUDE.md` 的 `## Coding Styles` 中定义缩进、命名规范。
2.  **Skill 引用规则**：在 `SKILL.md` 中写到："执行任务时，请务必参考根目录 `CLAUDE.md` 中的规范"。
3.  **优势**：你不需要在每个 Skill 里重复写规范，维护一份 `CLAUDE.md` 即可。

---

## 分享与分发 Skills

- **项目级分发**：直接将 `.claude/skills` 文件夹 `git add` 并提交。团队成员拉取代码后开箱即用。
- **npm 分发**：将 Skill 包装在插件中发布到 npm。其他用户只需执行 `npx -y your-plugin` 即可获得该 Skill 及其配套脚本。

---

## 常见问题与排错

### Q: 为什么我的图标/颜色在终端不显示？
**A**: Skill 的 Markdown 解析基于终端能力。建议在输出中使用标准的 Emoji（如 ✅, 🔴）以获得最佳跨平台支持。

### Q: 为什么 Claude 总是忽略我的 Skill？
**A**: 
1. 检查 `description` 是否包含触发场景。
2. 检查是否有 `disable-model-invocation: true`（如果有，Claude 绝不会自动调用它）。
3. 使用 `/context` 命令查看当前加载的所有 Skills 及其预算占用情况。

---

## 快速参考卡片

### 变量速查
- `$ARGUMENTS` (or `$0`, `$1`...)：用户输入的命令行参数。
- `$CORPUS_CONTENT`：项目全景摘要。

### 常用配置
- `disable-model-invocation: true`：禁止自动调用。
- `user-invocable: false`：在 `/` 菜单中隐藏，仅供背景知识或自动触发使用。
- `allowed-tools: [Read, Grep]`：安全策略，限制该 Skill 只能读取，不能修改。

---

> 📅 最后更新：2026-03-07
> ✍️ 作者：Jerry
> 📚 更多 AI Coding 相关内容请查看 [ai-coding 目录](../README.md)