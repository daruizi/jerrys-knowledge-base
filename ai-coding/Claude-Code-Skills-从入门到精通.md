# Claude Code - Skills 从入门到精通

> Skills 是扩展 Claude Code 能力的核心机制，本教程将带你从零开始掌握 Skills 的概念、配置使用和开发技巧。

---

## 📖 目录

1. [什么是 Skills？](#什么是-skills)
2. [Skills 核心概念](#skills-核心概念)
3. [内置 Skills 介绍](#内置-skills-介绍)
4. [创建你的第一个 Skill](#创建你的第一个-skill)
5. [Skill 高级配置](#skill-高级配置)
6. [高级技巧与模式](#高级技巧与模式)
7. [分享与分发 Skills](#分享与分发-skills)
8. [常见问题与排错](#常见问题与排错)

---

## 什么是 Skills？

### 一句话解释

**Skills 就像是给 Claude 写的"操作手册"** —— 当你创建一个 `SKILL.md` 文件，Claude 就会把它加入到工具箱中，在合适的时机自动使用，或者你可以直接用 `/skill-name` 调用它。

### 为什么需要 Skills？

在日常开发中，你可能有一些特定的需求：

- 🎯 **代码审查**：每次提交前自动检查代码质量
- 📝 **文档生成**：按照团队规范生成 API 文档
- 🚀 **部署流程**：执行标准化的部署步骤
- 🔍 **问题诊断**：按照特定流程排查问题

**Skills 解决的问题：**

| 问题 | Skills 的解决方案 |
|------|-------------------|
| 每次都要重复输入相同的指令 | 创建 Skill，一键调用 |
| Claude 不知道团队的编码规范 | 在 Skill 中定义规范，Claude 自动遵循 |
| 想要 Claude 执行多步骤流程 | 在 Skill 中定义完整流程 |
| 需要动态获取上下文信息 | 使用命令注入，实时获取数据 |

### Skills vs MCP vs Commands

很多人会混淆这三个概念，让我们来区分一下：

| 特性 | Skills | MCP | Commands |
|------|--------|-----|----------|
| **用途** | 定义行为和指令 | 连接外部工具和数据源 | 内置的快捷操作 |
| **创建方式** | 写 SKILL.md 文件 | 开发 MCP Server | 无法自定义 |
| **触发方式** | 自动或手动 | 通过工具调用 | 手动输入 |
| **复杂度** | 简单（纯文本） | 中等（需要代码） | N/A |
| **典型场景** | 代码审查、文档生成 | 数据库查询、API 调用 | 清屏、压缩对话 |

---

## Skills 核心概念

### 基本结构

每个 Skill 是一个目录，核心是 `SKILL.md` 文件：

```
my-skill/
├── SKILL.md           # 主指令文件（必需）
├── template.md        # 模板文件（可选）
├── examples.md        # 示例文件（可选）
└── scripts/           # 脚本文件（可选）
    └── helper.sh
```

### SKILL.md 文件结构

```yaml
---
# YAML frontmatter（配置部分）
name: my-skill
description: 这个 skill 的作用说明
---

# Markdown 内容（指令部分）

这里是给 Claude 的具体指令...
```

**两个核心部分：**

1. **Frontmatter（前置配置）**：YAML 格式，定义 Skill 的元数据和行为
2. **Markdown 内容**：实际的指令内容，告诉 Claude 该做什么

### Skills 存储位置

Skills 可以存放在不同位置，决定了它的作用范围：

| 位置 | 路径 | 适用范围 |
|------|------|----------|
| **个人级** | `~/.claude/skills/<skill-name>/SKILL.md` | 所有项目可用 |
| **项目级** | `.claude/skills/<skill-name>/SKILL.md` | 仅当前项目 |
| **插件级** | `<plugin>/skills/<skill-name>/SKILL.md` | 插件启用时可用 |
| **企业级** | 托管设置目录 | 组织内所有用户 |

**优先级**：企业级 > 个人级 > 项目级 > 插件级

---

## 内置 Skills 介绍

Claude Code 自带了几个强大的内置 Skills：

### /simplify - 代码简化

**用途**：审查最近修改的文件，检查代码重用、质量和效率问题，然后自动修复。

```text
# 基本用法
/simplify

# 指定关注点
/simplify focus on memory efficiency
```

**工作流程**：
1. 启动三个并行审查代理（代码重用、代码质量、效率）
2. 汇总发现的问题
3. 自动应用修复

### /batch - 批量操作

**用途**：在代码库中并行执行大规模修改。

```text
# 示例：将 Solid 组件迁移到 React
/batch migrate src/ from Solid to React
```

**工作流程**：
1. 研究代码库
2. 将工作分解为 5-30 个独立单元
3. 展示计划供你审批
4. 每个单元启动一个后台代理执行
5. 运行测试并创建 PR

### /debug - 会话调试

**用途**：排查当前 Claude Code 会话的问题。

```text
# 基本用法
/debug

# 指定问题描述
/debug the tool call keeps timing out
```

### /loop - 定时循环

**用途**：按间隔重复执行某个提示。

```text
# 每 5 分钟检查部署状态
/loop 5m check if the deploy finished

# 每小时检查 PR 状态
/loop 1h check PR status
```

### /claude-api - API 参考

**用途**：加载 Claude API 参考材料，覆盖 Python、TypeScript、Java、Go 等语言。

```text
/claude-api
```

当你代码中导入 `anthropic`、`@anthropic-ai/sdk` 或 `claude_agent_sdk` 时，这个 Skill 会自动激活。

---

## 创建你的第一个 Skill

### 示例：代码解释 Skill

让我们创建一个能生成可视化图表和类比解释的 Skill。

#### 步骤 1：创建 Skill 目录

```bash
# 创建个人 Skill 目录
mkdir -p ~/.claude/skills/explain-code
```

#### 步骤 2：编写 SKILL.md

创建文件 `~/.claude/skills/explain-code/SKILL.md`：

```yaml
---
name: explain-code
description: 使用可视化图表和类比解释代码。当用户询问"这个怎么工作"或需要理解代码时使用。
---

# 代码解释指南

当解释代码时，请始终包含以下内容：

## 1. 从类比开始

用一个生活中的例子来比喻代码的工作原理：

```
这个认证系统就像是一个酒店的前台：
- 用户名和密码是你的身份证
- JWT Token 是房卡
- Token 过期时间就是退房时间
```

## 2. 绘制流程图

使用 ASCII 艺术展示流程或结构：

```
┌─────────┐    ┌─────────┐    ┌─────────┐
│  请求   │───▶│  认证   │───▶│  处理   │
└─────────┘    └─────────┘    └─────────┘
     │              │              │
     ▼              ▼              ▼
  用户输入      验证 Token      业务逻辑
```

## 3. 逐步讲解

用简单的语言，一行一行解释代码做什么。

## 4. 指出常见陷阱

提醒读者可能遇到的坑：

> ⚠️ 注意：Token 存储在 localStorage 中，可能存在 XSS 攻击风险。

---

保持解释的对话风格，对于复杂概念，使用多个类比来帮助理解。
```

#### 步骤 3：测试 Skill

**方式一：自动触发**

直接问 Claude 一个符合 description 的问题：

```text
这个登录函数是怎么工作的？
```

Claude 会自动加载这个 Skill，并按照其中的指南来解释代码。

**方式二：手动调用**

```text
/explain-code src/auth/login.ts
```

### 示例：代码审查 Skill

创建一个项目级的代码审查 Skill：

```bash
# 创建项目 Skill 目录
mkdir -p .claude/skills/review
```

创建 `.claude/skills/review/SKILL.md`：

```yaml
---
name: review
description: 按照团队规范审查代码变更
disable-model-invocation: true
---

# 代码审查清单

审查以下文件的变更：$ARGUMENTS

## 审查要点

### 1. 代码质量
- [ ] 代码是否清晰易读？
- [ ] 变量和函数命名是否语义化？
- [ ] 是否有重复代码可以抽取？
- [ ] 注释是否充分且有意义？

### 2. 安全性
- [ ] 是否有 SQL 注入风险？
- [ ] 是否有 XSS 风险？
- [ ] 敏感数据是否正确处理？
- [ ] 权限检查是否完善？

### 3. 性能
- [ ] 是否有 N+1 查询问题？
- [ ] 循环中是否有不必要的操作？
- [ ] 缓存策略是否合理？

### 4. 测试
- [ ] 新功能是否有测试覆盖？
- [ ] 边界情况是否测试？
- [ ] 测试是否有意义？

## 输出格式

请按以下格式输出审查结果：

```
## 📋 审查结果

### ✅ 通过项
- 列出做得好的地方

### ⚠️ 需要关注
- 列出需要讨论的地方

### 🔴 必须修改
- 列出必须修复的问题

### 💡 建议
- 可选的改进建议
```
```

**使用方式**：

```text
/review src/
```

---

## Skill 高级配置

### Frontmatter 字段详解

```yaml
---
# 基本信息
name: my-skill                    # Skill 名称（用于 /name 调用）
description: 这个 skill 做什么    # 描述（用于 Claude 判断何时使用）
argument-hint: [文件名]          # 参数提示（显示在自动补全中）

# 调用控制
disable-model-invocation: true   # 禁止 Claude 自动调用
user-invocable: false            # 从 / 菜单隐藏

# 执行配置
allowed-tools: Read, Grep        # 限制可用工具
model: claude-sonnet-4-6         # 指定使用的模型
context: fork                    # 在子代理中运行
agent: Explore                   # 使用特定代理类型

# 钩子
hooks:
  PreToolUse:
    - command: "echo 'About to use tool'"
---
```

### 关键字段说明

#### disable-model-invocation

控制 Claude 是否可以自动调用这个 Skill。

```yaml
# ❌ Claude 可以自动调用（默认）
disable-model-invocation: false

# ✅ 只能手动调用（适合有副作用的操作）
disable-model-invocation: true
```

**使用场景**：
- `/deploy` - 不希望 Claude 自动部署
- `/send-email` - 不希望 Claude 自动发邮件
- `/commit` - 不希望 Claude 自动提交

#### user-invocable

控制 Skill 是否出现在 `/` 菜单中。

```yaml
# ✅ 显示在菜单中（默认）
user-invocable: true

# ❌ 从菜单隐藏（适合背景知识）
user-invocable: false
```

#### allowed-tools

限制 Skill 可以使用的工具：

```yaml
# 只读模式
allowed-tools: Read, Grep, Glob

# 允许所有工具
# （不设置 allowed-tools 字段）
```

### 参数处理

Skills 支持接收参数：

#### 基本参数

```yaml
---
name: fix-issue
description: 修复 GitHub Issue
---

修复 GitHub Issue #$ARGUMENTS

步骤：
1. 读取 Issue 描述
2. 理解需求
3. 实现修复
4. 编写测试
```

**使用**：`/fix-issue 123`

#### 位置参数

```yaml
---
name: migrate
description: 迁移组件
---

将 $ARGUMENTS[0] 组件从 $ARGUMENTS[1] 迁移到 $ARGUMENTS[2]。

# 或者使用简写
将 $0 组件从 $1 迁移到 $2。
```

**使用**：`/migrate Header React Vue`

---

## 高级技巧与模式

### 动态上下文注入

使用 `!`command`` 语法在 Skill 执行前运行命令，并将输出插入到内容中：

```yaml
---
name: pr-summary
description: 总结 Pull Request 变更
---

# PR 上下文

- PR 标题: !`gh pr view --json title -q .title`
- PR 描述: !`gh pr view --json body -q .body`
- 变更文件: !`gh pr diff --name-only`
- Diff 统计: !`gh pr diff --stat`

## 任务

请总结这个 Pull Request：
1. 主要改动是什么？
2. 影响了哪些模块？
3. 有什么需要注意的点？
```

**工作原理**：
1. 先执行所有 `!`command`` 命令
2. 将输出替换到对应位置
3. Claude 接收到完整的、包含实时数据的提示

### 在子代理中运行

使用 `context: fork` 让 Skill 在独立的子代理中执行：

```yaml
---
name: deep-research
description: 深度研究代码库
context: fork
agent: Explore
---

研究主题: $ARGUMENTS

请执行以下研究：
1. 使用 Glob 和 Grep 查找相关文件
2. 阅读并分析代码
3. 总结发现，包含具体的文件引用
```

**优点**：
- 独立的上下文，不会污染主对话
- 可以使用专门的代理类型（如 Explore）
- 适合长时间运行的任务

### 添加支持文件

对于复杂的 Skill，可以将详细内容拆分到多个文件：

```
my-skill/
├── SKILL.md           # 主指令（保持精简）
├── reference.md       # 详细参考文档
├── examples.md        # 使用示例
└── templates/
    └── component.md   # 代码模板
```

在 `SKILL.md` 中引用这些文件：

```yaml
---
name: api-docs
description: 生成 API 文档
---

# API 文档生成器

根据路由文件生成 API 文档。

## 参考资料

- 完整 API 规范: 见 [reference.md](reference.md)
- 文档示例: 见 [examples.md](examples.md)

## 模板

使用 [templates/component.md](templates/component.md) 作为文档模板。
```

### 可视化输出 Skill

Skills 可以生成 HTML 文件并在浏览器中打开，实现可视化输出：

```yaml
---
name: visualize-codebase
description: 可视化代码库结构
allowed-tools: Bash(python *)
---

# 代码库可视化

运行可视化脚本：

```bash
python ~/.claude/skills/visualize-codebase/scripts/generate.py .
```

这会生成一个交互式的 HTML 文件，展示：
- 可折叠的目录树
- 文件大小
- 文件类型分布
```

---

## 分享与分发 Skills

### 项目级分享

将 `.claude/skills/` 目录提交到版本控制：

```bash
# 添加到 Git
git add .claude/skills/
git commit -m "Add team code review skill"
git push
```

团队成员克隆仓库后，Skills 自动可用。

### 插件分发

将 Skills 打包到插件中：

```
my-plugin/
├── plugin.json
└── skills/
    └── my-skill/
        └── SKILL.md
```

插件安装后，Skills 会以 `plugin-name:skill-name` 的命名空间可用。

### 企业级分发

通过托管设置在企业内统一分发：

- **macOS**: `/Library/Application Support/ClaudeCode/`
- **Linux**: `/etc/claude-code/`
- **Windows**: `C:\Program Files\ClaudeCode\`

---

## 常见问题与排错

### Q1: Skill 没有被触发

**症状**：Claude 没有在预期的情况下使用我的 Skill

**解决方案**：

1. 检查 `description` 是否包含用户自然会说的话
2. 使用 `/` 查看 Skill 是否在列表中
3. 尝试手动调用：`/skill-name`
4. 检查 `disable-model-invocation` 设置

### Q2: Skill 被触发了太多次

**症状**：Claude 在不需要的时候也使用了我的 Skill

**解决方案**：

1. 让 `description` 更具体
2. 设置 `disable-model-invocation: true`
3. 调整描述中的关键词

### Q3: Claude 看不到我的所有 Skills

**症状**：有些 Skills 没有出现在列表中

**解决方案**：

Skills 描述有字符预算限制（约 2% 上下文窗口）。如果 Skills 太多，可能会超出预算。

```bash
# 查看上下文使用情况
/context

# 增加预算
export SLASH_COMMAND_TOOL_CHAR_BUDGET=32000
```

### Q4: Skill 中的命令执行失败

**症状**：`!`command`` 命令没有正确执行

**解决方案**：

1. 确保命令在终端中可以正常运行
2. 检查命令路径是否正确
3. 确认有必要的权限

### Q5: 如何调试 Skill

**方法一：直接调用**

```text
/skill-name
```

观察 Claude 的响应是否符合预期。

**方法二：检查加载**

```text
/context
```

查看 Skill 是否被正确加载。

**方法三：简化测试**

创建一个最小化的 Skill 来测试：

```yaml
---
name: test
description: 测试 Skill
---

这是一个测试 Skill。参数: $ARGUMENTS
```

---

## 快速参考卡片

### 创建 Skill

```bash
# 创建个人 Skill
mkdir -p ~/.claude/skills/my-skill

# 创建项目 Skill
mkdir -p .claude/skills/my-skill
```

### SKILL.md 模板

```yaml
---
name: skill-name
description: 这个 skill 做什么，何时使用
---

# 标题

具体指令内容...

## 参数

$ARGUMENTS
```

### 常用 Frontmatter

```yaml
# 只允许手动调用
disable-model-invocation: true

# 只读模式
allowed-tools: Read, Grep, Glob

# 子代理执行
context: fork
agent: Explore
```

### 调用方式

```text
# 手动调用
/skill-name

# 带参数调用
/skill-name arg1 arg2

# 让 Claude 自动判断何时使用
# （确保 description 足够清晰）
```

---

## 推荐资源

### 官方文档

- [Claude Code Skills 文档](https://code.claude.com/docs/en/skills)
- [Agent Skills 开放标准](https://agentskills.io)

### 示例 Skills

- [官方 Skills 示例](https://github.com/anthropics/claude-code/tree/main/skills)
- [社区 Skills 收集](https://github.com/topics/claude-code-skills)

### 相关功能

- **[MCP](./Claude-Code-MCP-从入门到精通.md)**：连接外部工具和数据源
- **[CLAUDE.md](./claude-code-guide.md)**：项目级别的持久上下文
- **Hooks**：自动化工作流

---

## 总结

Skills 是扩展 Claude Code 能力的核心机制，通过本教程，你已经学会了：

| 知识点 | 掌握程度 |
|--------|----------|
| Skills 核心概念和结构 | ✅ |
| 创建和配置 Skills | ✅ |
| 参数处理和动态上下文注入 | ✅ |
| 子代理执行和可视化输出 | ✅ |
| 分享和分发 Skills | ✅ |
| 常见问题的排查方法 | ✅ |

### 下一步建议

1. **创建你的第一个实用 Skill**：从简单的开始，如代码格式化、提交信息生成
2. **探索内置 Skills**：尝试 `/simplify` 和 `/batch`，理解它们的工作方式
3. **组合使用**：Skills + MCP + CLAUDE.md 可以构建强大的工作流
4. **分享给团队**：将好用的 Skills 提交到项目中，让团队受益

---

> 📅 最后更新：2026-03-07
> 📚 更多 AI Coding 相关内容请查看 [ai-coding 目录](./README.md)