# Claude Code — 从入门到精通

> Anthropic 出品的终端 AI 编程助手，让你在命令行中与 AI 结对编程。

---

## 目录

1. [简介](#简介)
2. [安装与配置](#安装与配置)
3. [快速上手](#快速上手)
4. [核心功能详解](#核心功能详解)
5. [CLAUDE.md — 项目记忆](#claudemd--项目记忆)
6. [Slash Commands — 自定义命令](#slash-commands--自定义命令)
7. [高级用法](#高级用法)
8. [实战技巧与最佳实践](#实战技巧与最佳实践)
9. [常见问题（FAQ）](#常见问题faq)
10. [延伸资源](#延伸资源)

---

## 简介

### 什么是 Claude Code？

Claude Code 是 Anthropic 推出的 **命令行 AI 编程工具**，可以直接在终端中理解你的代码库、编辑文件、运行命令、管理 Git 工作流，让 AI 成为你真正的编程伙伴。

### 核心定位

| 特点 | 说明 |
|------|------|
| 🖥️ 终端原生 | 在你最熟悉的终端里工作，无需切换到浏览器 |
| 🧠 深度理解代码 | 自动分析项目结构、代码逻辑和依赖关系 |
| 🤖 全自主代理 | 不只是建议——能直接编辑文件、运行测试、提交代码 |
| 🔄 多模型支持 | 可在 Opus、Sonnet、Haiku 之间灵活切换 |

### 与其他 AI 编程工具的区别

- **vs GitHub Copilot**：Copilot 主要做行级/块级代码补全；Claude Code 是全局性的代理，能跨文件操作整个项目
- **vs Cursor**：Cursor 是基于 VS Code 的 IDE；Claude Code 是终端工具，更轻量、更灵活
- **vs ChatGPT**：ChatGPT 需要手动粘贴代码；Claude Code 直接读取你的本地项目

---

## 安装与配置

### 前置要求

- **操作系统**：macOS、Linux 或 Windows
- **Windows 用户**：需要安装 Git for Windows
- **订阅计划**：Claude Pro、Max、Teams 或 Enterprise 订阅，或者 Anthropic Console API 账号

### 安装方式

#### 方式一：原生安装器（⭐ 推荐）

这是官方推荐的方式，无需预装 Node.js，且支持自动更新。

**macOS / Linux：**

```bash
curl -fsSL https://claude.ai/install.sh | sh
```

**Windows（PowerShell 管理员模式）：**

```powershell
irm https://claude.ai/install.ps1 | iex
```

#### 方式二：通过 WSL2 安装（Windows 备选）

如果你习惯 Linux 环境，可以在 WSL2 中使用：

```bash
# 1. 安装 WSL（PowerShell 管理员模式）
wsl --install

# 2. 在 WSL 中安装 nvm 和 Node.js
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install --lts
nvm use --lts

# 3. 安装 Claude Code
npm install -g @anthropic-ai/claude-code
```

#### 方式三：npm 安装（⚠️ 已废弃）

```bash
npm install -g @anthropic-ai/claude-code
```

> ⚠️ **注意**：npm 安装方式已被官方标记为废弃（deprecated），推荐使用原生安装器。

### 首次认证

```bash
# 启动 Claude Code
claude

# 按提示在浏览器中登录你的 Anthropic 账号完成认证
```

### VS Code 集成

除了纯命令行使用外，还可以安装官方 VS Code 扩展，获得图形化界面体验：

1. 在 VS Code 扩展商店搜索 **"Claude Code"**
2. 安装后，会在 VS Code 侧边栏出现 Claude Code 面板
3. 支持 diff 预览、Git 集成、上下文感知等功能

---

## 快速上手

### 基本流程

```bash
# 1. 进入你的项目目录
cd my-project

# 2. 启动 Claude Code 交互会话
claude

# 3. 用自然语言描述你的需求
> 帮我分析一下这个项目的整体架构

# 4. Claude 会自动读取代码、分析结构，给你回答
```

### 常用交互方式

| 操作 | 语法 | 示例 |
|------|------|------|
| 自然语言对话 | 直接输入 | `帮我写一个用户登录功能` |
| 引用特定文件 | `@文件名` | `@src/auth.ts 这个文件有什么问题` |
| 执行 Shell 命令 | `!命令` | `!npm test` |
| 使用 Slash 命令 | `/命令名` | `/review` |

### 第一个实用示例

```
你: 帮我看看 src/utils.ts 里有没有可以优化的地方

Claude: [自动读取文件，分析代码]
我发现了以下几个可优化点：
1. 第 23 行的循环可以用 Array.map() 替代...
2. parseDate 函数没有做错误处理...

需要我帮你修复吗？

你: 好的，帮我修复

Claude: [直接编辑文件，展示改动 diff]
已修改 src/utils.ts，以下是变更内容...
```

---

## 核心功能详解

### 1. 代码理解与分析 🔍

Claude Code 能深度理解你的整个项目：

- **项目结构分析**：自动识别目录结构、技术栈、依赖关系
- **代码解释**：用中文解释复杂的代码逻辑
- **依赖追踪**：跨文件追踪函数调用链和数据流
- **Bug 诊断**：定位问题根源并给出修复方案

```
> 分析一下这个项目用了哪些设计模式
> 解释一下 handlePayment 函数的完整执行流程
> 这个错误是什么原因导致的？给我修复方案
```

### 2. 代码编辑与重构 ✏️

Claude Code 可以直接修改你的代码文件：

- **跨文件编辑**：同时修改多个相关文件
- **智能重构**：提取函数、重命名变量、调整架构
- **代码生成**：基于需求描述生成完整的功能模块

```
> 把 UserService 类拆分成 UserAuthService 和 UserProfileService
> 给所有的 API 端点加上统一的错误处理中间件
> 创建一个 React 组件，实现一个可搜索的下拉选择框
```

### 3. Git 工作流管理 🔀

无需记忆 Git 命令，用自然语言操作：

- **提交管理**：自动生成有意义的 commit message
- **分支操作**：创建分支、合并、解决冲突
- **PR 创建**：生成 Pull Request 描述和变更摘要

```
> 帮我提交这些改动，写一个清晰的 commit message
> 创建一个 feature/user-auth 分支并切过去
> 帮我整理这个 PR 的描述
```

### 4. 测试支持 🧪

- **写测试**：自动生成单元测试和集成测试
- **运行测试**：执行测试并分析失败原因
- **修复测试**：根据失败信息自动修复代码或测试

```
> 给 UserService 类编写单元测试，覆盖所有边界情况
> 运行测试，如果有失败的帮我修复
```

### 5. 模型切换 🔄

在不同任务中使用最合适的模型：

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| **Opus** | 最强大，推理能力最强 | 复杂架构设计、难点调试 |
| **Sonnet** | 均衡，速度与质量兼顾 | 日常开发、代码编写 |
| **Haiku** | 最快速，成本最低 | 简单问答、快速查询 |

```
/model opus    # 切换到 Opus
/model sonnet  # 切换到 Sonnet
/model haiku   # 切换到 Haiku
```

---

## CLAUDE.md — 项目记忆

### 什么是 CLAUDE.md？

`CLAUDE.md` 是放在项目根目录的 Markdown 文件，Claude Code 每次启动会自动读取它。相当于给 AI 一份项目说明书，让它快速了解你的项目约定。

### 为什么需要它？

Claude Code 是**无状态**的——每次新会话都从零开始。`CLAUDE.md` 提供了持久化的项目上下文，避免每次都重复说明。

### 创建方式

```bash
# 方法一：使用 /init 命令自动生成
claude
> /init

# 方法二：手动创建
# 在项目根目录创建 CLAUDE.md 文件
```

### 编写模板

```markdown
# 项目名称

## 技术栈
- 语言：TypeScript 5.x
- 框架：Next.js 14 (App Router)
- 数据库：PostgreSQL + Prisma ORM
- 样式：Tailwind CSS

## 项目结构
- src/app/         → 页面路由
- src/components/  → 可复用组件
- src/lib/         → 工具函数和数据库客户端
- src/types/       → TypeScript 类型定义

## 开发规范
- 使用中文注释
- 组件采用函数式写法 + hooks
- 使用 Zod 进行数据校验
- 所有 API 返回统一格式 { success, data, error }

## 常用命令
- `npm run dev`       → 启动开发服务器
- `npm run test`      → 运行测试
- `npm run lint`      → 代码检查
- `npx prisma studio` → 打开数据库可视化工具

## 注意事项
- 不要直接修改 generated/ 目录下的文件
- 环境变量存放在 .env.local 中
- 数据库迁移必须通过 prisma migrate 管理
```

### CLAUDE.md 最佳实践

| ✅ 推荐做法 | ❌ 避免做法 |
|------------|------------|
| 控制在 200 行以内 | 写成超长文档 |
| 只写 AI 无法自行推断的信息 | 重复 README 里的内容 |
| 包含验证步骤（如何运行测试） | 只写规范不提供验证方法 |
| 定期精简和更新 | 写完就不管了 |
| 用 `@imports` 引用子文档分模块管理 | 全塞在一个文件里 |

### 层级结构

`CLAUDE.md` 支持多级存在，范围从宽到窄：

```
~/.claude/CLAUDE.md          → 全局配置（所有项目生效）
项目根目录/CLAUDE.md          → 项目级配置
项目根目录/src/CLAUDE.md     → 子目录级配置（更具体）
```

更具体的配置会覆盖更宽的配置。

### 使用 @imports 拆分

当项目说明较多时，可以用 `@imports` 引用子文件：

```markdown
# 项目配置

@docs/coding-standards.md
@docs/api-conventions.md
@docs/deployment-guide.md
```

这样主文件保持精简，详细文档按需加载。

---

## Slash Commands — 自定义命令

### 内置 Slash 命令

| 命令 | 功能 |
|------|------|
| `/init` | 生成 CLAUDE.md 项目配置文件 |
| `/clear` | 清空当前会话上下文 |
| `/compact` | 压缩对话历史，节省 token |
| `/review` | 对当前代码进行 code review |
| `/model` | 切换 AI 模型 |

### 创建自定义命令

自定义命令本质上是 Markdown 文件，存放在特定目录中。

#### 命令作用域

| 类型 | 目录 | 调用方式 | 说明 |
|------|------|----------|------|
| 项目级 | `.claude/commands/` | `/project:命令名` | 可 Git 管理，团队共享 |
| 用户级 | `~/.claude/commands/` | `/user:命令名` | 个人全局使用 |

#### 示例：创建一个"新建 API 端点"命令

**文件路径**：`.claude/commands/new-api.md`

```markdown
---
description: 创建一个新的 API 端点
argument-hint: <端点路径> <功能描述>
allowed-tools: write, edit
---

请在 src/app/api/ 目录下创建一个新的 API 端点。

路径：$1
功能要求：$2

要求：
1. 使用 TypeScript 编写
2. 包含完整的错误处理
3. 添加输入验证（使用 Zod）
4. 同时生成对应的单元测试
5. 遵循项目现有的 API 返回格式
```

**使用方式**:

```
/project:new-api /users/profile 获取和更新用户个人资料
```

#### 示例：创建一个"代码审查"命令

**文件路径**：`.claude/commands/review-pr.md`

```markdown
---
description: 对当前分支的改动做全面代码审查
---

请对当前分支与 main 分支的差异做全面的代码审查。

审查要点：
1. 代码质量和可读性
2. 潜在的 Bug 和边界情况
3. 安全性问题
4. 性能影响
5. 测试覆盖率

请用中文输出审查报告，按严重程度排序。
```

#### 高级：使用子目录组织命令

```
.claude/commands/
├── api/
│   ├── new.md          → /project:api:new
│   └── test.md         → /project:api:test
├── db/
│   ├── migrate.md      → /project:db:migrate
│   └── seed.md         → /project:db:seed
└── review.md           → /project:review
```

---

## 高级用法

### 1. 非交互模式（SDK 调用）

用 `-p` 参数可以直接传入 prompt，适合脚本调用：

```bash
# 单次查询
claude -p "分析 src/index.ts 的主要功能"

# 配合管道使用
cat error.log | claude -p "分析这个错误日志，找出根本原因"

# 在 CI/CD 中使用
claude -p "检查代码是否符合项目编码规范" --output-format json
```

### 2. Plan Mode（规划模式）

适合复杂任务，让 Claude 先出方案再执行：

```
> /plan 重构整个认证模块，将 JWT 换成 OAuth2

Claude: 以下是我的重构计划：
第一阶段：分析当前认证流程...
第二阶段：设计 OAuth2 集成方案...
第三阶段：逐步替换...

你确认这个方案吗？
```

### 3. 图片支持

Claude Code 支持粘贴图片并描述：

```
> [粘贴 UI 截图] 按照这个设计稿实现这个页面
> [粘贴错误截图] 帮我分析这个错误
```

### 4. Multi-Agent（多代理协作）

Claude Code 支持子代理（Subagent），可以把任务拆分给多个 AI 并行处理：

```
> 帮我同时完成以下任务：
> 1. 给所有组件加上 TypeScript 类型
> 2. 编写对应的单元测试
> 3. 更新 API 文档
```

### 5. MCP（Model Context Protocol）集成

通过 MCP，Claude Code 可以连接外部工具和服务：

- **Jira**：自动从 Jira 拉取任务描述
- **Notion**：读取 Notion 中的项目文档
- **数据库**：直接查询数据库获取上下文
- **AWS**：管理云资源

---

## 实战技巧与最佳实践

### 🎯 Prompt 编写技巧

#### 1. 明确且具体

```
❌ "帮我改一下代码"
✅ "在 src/api/users.ts 中，给 getUserById 函数添加缓存逻辑，
   使用 Redis，缓存时间 5 分钟，未命中时查数据库"
```

#### 2. 分步执行复杂任务

```
❌ "帮我做一个完整的电商系统"
✅ "第一步：帮我设计数据库模型，包含商品、订单、用户三个表"
   "第二步：基于这些模型生成 Prisma schema"
   "第三步：创建商品的 CRUD API"
```

#### 3. 利用上下文

```
✅ "@package.json @src/config.ts 基于当前的项目配置，帮我添加 Redis 缓存支持"
```

### 🛡️ 安全与控制

| 实践 | 说明 |
|------|------|
| 审查修改 | Claude 编辑文件前会展示 diff，确认后才应用 |
| 使用 `.claudeignore` | 类似 `.gitignore`，排除敏感文件 |
| 定期清理上下文 | 用 `/clear` 或 `/compact` 避免上下文污染 |
| 注意成本 | Opus 模型消耗更多 token，简单任务用 Haiku |

### ⚡ 效率提升技巧

1. **善用 `/compact`**：长会话中定期压缩，保持响应质量
2. **合理选模型**：日常用 Sonnet，难题用 Opus，简单查询用 Haiku
3. **活用 `@` 引用**：直接指定文件，减少 Claude 搜索时间
4. **编写好 CLAUDE.md**：投入时间写好项目配置，之后每次会话都能省时间
5. **自定义 Slash 命令**：把重复的工作流程封装成命令

### 📋 典型工作流

```
# 1. 开始新功能开发
claude
> /init                              # 初始化项目记忆（仅首次）
> 创建 feature/search 分支

# 2. 实现功能
> 实现全文搜索功能，使用 Elasticsearch
> @src/services/ 在这个目录下创建 SearchService

# 3. 测试
> 给 SearchService 写单元测试
> !npm test                          # 运行测试

# 4. 代码审查
> /review                           # 让 Claude 审查代码

# 5. 提交
> 帮我提交并推送，写清楚 commit message
```

---

## 常见问题（FAQ）

### Q: Claude Code 免费吗？

不免费。需要 Claude Pro（$20/月）、Max（$100/$200/月）、Teams 或 Enterprise 订阅。也可以使用 Anthropic Console 的 API 额度。

### Q: Windows 上怎么安装？

推荐使用原生安装器（PowerShell 管理员运行 `irm https://claude.ai/install.ps1 | iex`）。也可以在 WSL2 中安装使用。

### Q: 支持哪些编程语言？

Claude Code 不限语言，Python、JavaScript/TypeScript、Java、Go、Rust、C++……都可以。它理解的是代码本身，而非特定语言。

### Q: 我的代码安全吗？

代码会发送到 Anthropic 的服务器进行处理。可以通过 `.claudeignore` 排除敏感文件（如 `.env`、密钥文件等）。

### Q: 和 Cursor 哪个好？

视场景而定。如果你喜欢在 IDE 中工作且需要实时代码补全，选 Cursor。如果你偏爱终端工作流、需要跨文件操作和自动化能力，选 Claude Code。两者也可以配合使用。

### Q: 怎么控制花费？

- 简单任务使用 Haiku 或 Sonnet 模型
- 用 `/compact` 定期压缩会话
- 用 `@` 引用文件代替让 Claude 全局搜索
- 在 Anthropic Console 设置用量上限

---

## 延伸资源

| 资源 | 链接 |
|------|------|
| 官方文档 | https://docs.anthropic.com/en/docs/claude-code |
| GitHub 仓库 | https://github.com/anthropics/claude-code |
| VS Code 扩展 | VS Code 扩展商店搜索 "Claude Code" |
| 社区 Discord | Anthropic 官方 Discord |
| API 文档 | https://docs.anthropic.com/en/docs |

---

> 📅 最后更新：2026-03-07
>
> 📝 本教程持续更新中，欢迎补充和改进！
