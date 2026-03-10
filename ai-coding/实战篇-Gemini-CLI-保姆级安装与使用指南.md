# 实战篇 - Gemini CLI 保姆级安装与使用指南

## 1. 简介

Gemini CLI 是 Google 推出的原生 AI 辅助开发工具，集成了强大的 Gemini 2.0 系列模型。它不仅是一个聊天机器人，更是一个拥有完整终端权限的"虚拟工程师"，能够深度理解项目架构、自动编写代码、运行测试并修复 Bug。

**核心优势：**
- 开源免费，代码托管在 GitHub
- 原生支持 Gemini 2.0 Flash/Pro 模型
- 支持 MCP (Model Context Protocol) 扩展
- 自动读取项目上下文，理解代码库
- 支持文件读写、终端命令执行

---

## 2. 安装与环境配置

### 2.1 系统要求

| 环境 | 最低版本 | 推荐版本 |
| :--- | :--- | :--- |
| Node.js | 18.x | 20.x 或更高 |
| npm | 8.x | 10.x 或更高 |
| 操作系统 | - | Windows 10+、macOS 12+、Ubuntu 20.04+ |

**检查当前环境：**
```bash
# 检查 Node.js 版本
node --version

# 检查 npm 版本
npm --version
```

### 2.2 安装步骤

打开终端（PowerShell 或 Bash），运行：

```bash
# 全局安装 Gemini CLI
npm install -g @google/gemini-cli
```

**国内用户加速安装：**
```bash
# 使用淘宝镜像
npm install -g @google/gemini-cli --registry=https://registry.npmmirror.com
```

### 2.3 安装验证

```bash
# 验证安装成功
gemini --version

# 查看帮助信息
gemini --help
```

### 2.4 卸载方法

```bash
# 如需卸载
npm uninstall -g @google/gemini-cli
```

### 2.5 常见安装问题

| 问题 | 解决方案 |
| :--- | :--- |
| `EACCES permission denied` | 使用 `sudo npm install -g @google/gemini-cli` (macOS/Linux) 或以管理员身份运行 PowerShell |
| `node: command not found` | 先安装 Node.js，推荐使用 [nvm](https://github.com/nvm-sh/nvm) 管理 |
| 网络超时 | 使用国内镜像或配置代理 |

---

## 3. 身份认证

### 3.1 认证方式

Gemini CLI 支持两种认证方式：

#### 方式一：OAuth 登录（推荐个人用户）

```bash
# 启动登录流程（将自动打开浏览器）
gemini auth login

# 查看当前认证状态
gemini auth status

# 退出登录
gemini auth logout
```

#### 方式二：API Key 认证（适合 CI/CD 或无浏览器环境）

1. 访问 [Google AI Studio](https://aistudio.google.com/apikey) 获取 API Key
2. 配置环境变量：

```bash
# macOS / Linux
export GEMINI_API_KEY="your-api-key-here"

# Windows PowerShell
$env:GEMINI_API_KEY="your-api-key-here"

# Windows CMD
set GEMINI_API_KEY=your-api-key-here
```

### 3.2 免费额度说明

| 模型 | 免费额度 | 说明 |
| :--- | :--- | :--- |
| Gemini 2.0 Flash | 1500 次/天 | 快速响应，适合日常开发 |
| Gemini 2.0 Pro | 50 次/天 | 更强推理能力，适合复杂任务 |

> 额度以 Google AI Studio 官方为准，可能随时调整。

### 3.3 查看配额使用情况

登录 [Google AI Studio](https://aistudio.google.com/)，在 **Settings > Plan & Billing** 中查看实时配额使用情况。

---

## 4. 交互式核心命令 (Slash Commands)

在 Gemini CLI 的对话框中，以下斜杠命令用于管理会话，不消耗 Token。

### 4.1 会话管理命令

| 命令 | 功能描述 | 最佳实践 |
| :--- | :--- | :--- |
| **`/help`** | 显示帮助信息 | 忘记指令时随时调用 |
| **`/clear`** | 重置对话记忆 | **高频使用：** 开启新任务前必须执行，避免上下文污染 |
| **`/exit`** | 退出程序 | 安全关闭 CLI 界面 |
| **`/bug`** | 反馈问题 | 遇到工具运行异常时向 Google 提交报告 |
| **`/settings`** | 查看配置 | 确认当前使用的模型及工作目录 |
| **`/restore`** | 恢复历史会话 | 继续之前的对话上下文 |

### 4.2 模型切换命令

| 命令 | 功能描述 |
| :--- | :--- |
| **`/model`** | 查看和切换当前使用的模型 |
| **`/model gemini-2.0-flash`** | 切换到 Flash 模型（更快） |
| **`/model gemini-2.0-pro`** | 切换到 Pro 模型（更强） |

### 4.3 工具与扩展命令

| 命令 | 功能描述 |
| :--- | :--- |
| **`/tools`** | 查看当前可用的工具和扩展 |
| **`/mcp`** | 管理 MCP 服务器连接 |
| **`/context`** | 管理上下文文件（添加/移除文件到对话） |

---

## 5. 启动命令行参数 (Startup Flags)

在系统 Shell 中启动 `gemini` 时可携带参数进行初始化：

| 参数 | 描述 | 示例 |
| :--- | :--- | :--- |
| **`--dir <path>`** | 指定 AI 的工作目录（默认为当前目录） | `gemini --dir ./my-app` |
| **`--model <id>`** | 强制指定使用的 Gemini 模型版本 | `gemini --model gemini-2.0-pro` |
| **`--verbose`** | 显示所有后台运行日志（调试用） | `gemini --verbose` |
| **`--no-history`** | 启动时不加载上次对话的历史上下文 | `gemini --no-history` |
| **`--no-tools`** | 禁用所有工具扩展 | `gemini --no-tools` |

---

## 6. 配置文件详解

### 6.1 `GEMINI.md` 规范文件

这是控制 AI 行为的"最高宪法"。在项目根目录创建该文件，写下您的编码偏好。AI 每次任务前都会读取此文件，确保产出的代码符合您的预期。

**完整模板示例：**

```markdown
# 项目开发规范

## 代码风格
- 使用 TypeScript 严格模式
- 缩进：2 空格，不使用 Tab
- 引号：优先使用单引号
- 分号：不使用分号
- 最大行宽：100 字符

## 文件组织
- 组件放在 `src/components/` 目录
- 工具函数放在 `src/utils/` 目录
- 类型定义放在 `src/types/` 目录
- API 接口放在 `src/api/` 目录

## 命名约定
- 组件文件：PascalCase（如 `UserProfile.tsx`）
- 工具函数：camelCase（如 `formatDate.ts`）
- 常量：UPPER_SNAKE_CASE（如 `MAX_RETRY_COUNT`）
- CSS 类名：kebab-case（如 `user-profile-container`）

## 测试要求
- 新功能必须包含单元测试
- 测试覆盖率目标：80% 以上
- 修改代码后必须运行：`npm run test`

## Git 提交规范
- feat: 新功能
- fix: 修复 Bug
- docs: 文档更新
- refactor: 重构代码
- test: 测试相关

## 禁止事项
- 不要修改 `.env` 文件中的敏感信息
- 不要提交 `dist/`、`node_modules/` 目录
- 不要在组件中直接调用底层 API
- 不要使用 `any` 类型（除非必要）
```

### 6.2 `.geminiignore` 忽略配置

类似 `.gitignore`，用于排除不需要 AI 分析的文件，减少 Token 消耗。

**推荐配置：**

```gitignore
# 依赖目录
node_modules/
vendor/
venv/

# 构建产物
dist/
build/
out/
*.min.js
*.min.css

# 敏感文件
.env
.env.local
*.pem
*.key
credentials/
secrets/

# 大型文件
*.zip
*.tar.gz
*.log
*.bak

# IDE 配置
.idea/
.vscode/
*.swp

# 测试覆盖率报告
coverage/
.nyc_output/

# 缓存文件
.cache/
.parcel-cache/
```

### 6.3 高级配置文件

在 `~/.gemini/` 目录下可创建配置文件：

**`~/.gemini/config.json` 配置示例：**

```json
{
  "defaultModel": "gemini-2.0-flash",
  "maxTokens": 8192,
  "temperature": 0.7,
  "autoSave": true,
  "locale": "zh-CN"
}
```

**环境变量配置：**

| 变量 | 说明 | 示例 |
| :--- | :--- | :--- |
| `GEMINI_API_KEY` | 直接使用 API Key 认证 | `export GEMINI_API_KEY=xxx` |
| `GEMINI_MODEL` | 默认使用的模型 | `export GEMINI_MODEL=gemini-2.0-flash` |
| `GEMINI_PROXY` | 代理服务器地址 | `export GEMINI_PROXY=http://localhost:7890` |

---

## 7. MCP 服务器集成

MCP (Model Context Protocol) 是 Anthropic 推出的开放协议，Gemini CLI 支持 MCP 扩展，可连接外部工具和数据源。

### 7.1 配置 MCP 服务器

在项目目录创建 `.gemini/mcp.json`：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/dir"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-github-token"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost:5432/mydb"
      }
    }
  }
}
```

### 7.2 常用 MCP 服务器

| MCP 服务器 | 功能 | 安装命令 |
| :--- | :--- | :--- |
| `server-filesystem` | 文件系统读写 | `npx -y @modelcontextprotocol/server-filesystem` |
| `server-github` | GitHub 操作（Issue、PR） | `npx -y @modelcontextprotocol/server-github` |
| `server-postgres` | PostgreSQL 数据库查询 | `npx -y @modelcontextprotocol/server-postgres` |
| `server-sqlite` | SQLite 数据库操作 | `npx -y @modelcontextprotocol/server-sqlite` |
| `server-puppeteer` | 网页抓取与截图 | `npx -y @modelcontextprotocol/server-puppeteer` |

### 7.3 MCP 命令

```bash
# 在 Gemini CLI 中管理 MCP
/mcp              # 查看已配置的 MCP 服务器
/mcp connect      # 连接所有 MCP 服务器
/mcp disconnect   # 断开 MCP 连接
```

---

## 8. 专家子代理 (Expert Sub-agents)

Gemini CLI 内置多个专家代理，系统自动识别任务类型并调用：

### 8.1 Codebase Investigator（代码库侦探）

**适用场景：** 分析项目结构、追溯代码调用链、理解复杂逻辑

**示例提示词：**
- "分析这个项目的目录结构"
- "`login` 函数被哪些地方调用？"
- "解释 `AuthService` 的完整工作流程"
- "找到所有使用 `useState` 的组件"

### 8.2 Generalist（全才代理）

**适用场景：** 批量重构、添加注释、代码清理、重复性任务

**示例提示词：**
- "给所有 TypeScript 文件添加 JSDoc 注释"
- "修复项目中所有的 ESLint 警告"
- "将所有 `var` 替换为 `const`"
- "给全项目加 License 头"

### 8.3 CLI Help（官方文档专家）

**适用场景：** 咨询工具功能、配置方法、最佳实践

**示例提示词：**
- "Gemini CLI 如何配置代理？"
- "如何使用 MCP 服务器？"
- "最新的 Gemini 模型版本是什么？"
- "如何切换模型？"

### 8.4 Debug Agent（调试代理）

**适用场景：** 分析错误日志、定位 Bug、修复问题

**示例提示词：**
- "测试失败了，帮我分析原因"
- "这个错误是什么意思？如何修复？"
- "运行 `npm run build` 失败，帮我排查"

---

## 9. 实战案例演示

### 9.1 案例一：代码审查

```
用户：请审查 src/auth/login.ts 文件的安全性问题

Gemini：我来审查这个文件的安全性...

发现以下潜在问题：

1. 第 45 行：密码明文传输
   风险：中间人攻击
   建议：使用 HTTPS 并在前端进行密码哈希

2. 第 78 行：缺少 CSRF 保护
   风险：跨站请求伪造攻击
   建议：添加 CSRF Token 验证

3. 第 102 行：Session 过期时间过长（7天）
   风险：会话劫持风险增加
   建议：缩短为 24 小时，添加刷新机制

是否需要我帮您修复这些问题？
```

### 9.2 案例二：项目初始化

```
用户：帮我创建一个新的 Express + TypeScript 项目

Gemini：好的，我来创建项目结构...

[执行以下操作]
✓ 创建项目目录结构
✓ 初始化 package.json
✓ 配置 tsconfig.json（严格模式）
✓ 安装依赖：express, typescript, @types/express
✓ 创建 src/index.ts 入口文件
✓ 配置 ESLint 和 Prettier
✓ 创建 .gitignore

项目已准备就绪！运行 `npm run dev` 启动开发服务器。
```

### 9.3 案例三：Bug 修复

```
用户：测试失败了，帮我看看哪里出了问题

Gemini：让我分析测试输出...

问题定位：src/utils/parser.ts 第 23 行
原因：正则表达式无法匹配包含特殊字符的边界情况

错误输入示例：
  parseEmail("user+test@example.com")  // 返回 null

问题代码：
  const emailRegex = /^[\w]+@[\w]+\.[\w]+$/

建议修复：
  const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/

是否需要我应用此修复并重新运行测试？
```

### 9.4 案例四：代码重构

```
用户：把这个组件从 JavaScript 重构为 TypeScript

Gemini：我来重构这个组件...

[分析组件依赖]
- 导入 3 个外部依赖
- 使用 2 个内部模块
- 组件 props 包含 5 个属性

[重构步骤]
1. 创建类型定义 interfaces.ts
2. 转换组件文件为 .tsx
3. 添加 Props 类型注解
4. 修复类型错误（发现 2 处）

重构完成！类型检查通过。
```

---

## 10. Token 使用监控

### 10.1 Token 消耗机制

| 消耗来源 | 说明 |
| :--- | :--- |
| 系统提示 | 每次请求固定消耗（约 500-1000 tokens） |
| 历史对话 | 累积增长，`/clear` 可重置 |
| 文件内容 | 根据文件大小计算 |
| GEMINI.md | 每次请求都会读取 |

### 10.2 Token 节省技巧

1. **及时清理上下文：** 切换话题时务必使用 `/clear`
2. **配置忽略文件：** 使用 `.geminiignore` 排除大型依赖和构建产物
3. **精简 GEMINI.md：** 只包含必要的规范，避免冗余
4. **按需添加文件：** 使用 `/context` 命令只添加相关文件
5. **选择合适模型：** 简单任务使用 Flash 模型

### 10.3 实时监控

```bash
# 每次大型任务后，CLI 会显示 Token 消耗
> 帮我重构这个项目
...
[完成] 消耗 Token: 15,420 (输入: 12,300, 输出: 3,120)
```

---

## 11. 常见问题排查

### Q1: 安装失败 "EACCES permission denied"

```bash
# 方案一：使用 sudo（macOS/Linux）
sudo npm install -g @google/gemini-cli

# 方案二：修复 npm 权限
sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}

# 方案三：使用 nvm 管理 Node.js（推荐）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install --lts
```

### Q2: 认证后仍提示 "Permission denied"

**排查步骤：**
1. 检查网络是否需要代理
2. 确认 Google 账号是否有 AI Studio 访问权限
3. 尝试 `gemini auth logout` 后重新登录
4. 检查 API Key 是否正确配置

### Q3: Token 消耗过快

**检查清单：**
- [ ] 是否忘记 `/clear` 切换话题
- [ ] `.geminiignore` 是否排除了大型文件
- [ ] GEMINI.md 是否过于冗长
- [ ] 是否不小心包含了大型二进制文件

### Q4: 响应缓慢或超时

**解决方案：**
1. 检查网络连接稳定性
2. 切换到更快的模型：`/model gemini-2.0-flash`
3. 使用 `/clear` 清理过多的上下文
4. 配置代理：`export GEMINI_PROXY=http://localhost:7890`

### Q5: 文件修改未生效

**可能原因：**
- 文件被 `.geminiignore` 排除
- 权限不足
- 文件被其他程序锁定

### Q6: MCP 服务器连接失败

**排查步骤：**
1. 确认 MCP 配置文件路径正确（`.gemini/mcp.json`）
2. 检查环境变量是否配置
3. 尝试手动运行 MCP 服务器命令测试

---

## 12. 与其他 AI CLI 工具对比

| 特性 | Gemini CLI | Claude Code | Cursor | GitHub Copilot CLI |
| :--- | :--- | :--- | :--- | :--- |
| **模型** | Gemini 2.0 | Claude 4.x | 多模型 | GPT-4 |
| **文件操作** | 支持 | 支持 | 支持 | 支持 |
| **终端命令** | 支持 | 支持 | 支持 | 支持 |
| **MCP 支持** | 支持 | 支持 | 支持 | 不支持 |
| **开源程度** | 开源 | 开源 | 闭源 | 闭源 |
| **免费额度** | 有 | 有 | 付费 | 需订阅 |
| **最佳场景** | Google 生态 | 通用开发 | IDE 集成 | GitHub 集成 |
| **中文支持** | 优秀 | 优秀 | 优秀 | 一般 |

---

## 13. 新手实战建议

### 13.1 入门三步走

1. **先问后动：** 刚开始可以先问"如何实现..."，让 AI 提供方案；满意后再要求"请执行刚才的方案"
2. **安全确认：** 当 AI 提示执行 `rm` 或 `git push` 等敏感命令时，务必审阅弹出的命令详情
3. **闭环验证：** 下达指令时加上："修复它，并运行相关测试确认已修复"

### 13.2 高效使用技巧

- **善用 `/clear`：** 每完成一个大任务，清理上下文后再开始新任务
- **配置 GEMINI.md：** 在项目初期就配置好，让 AI 了解你的编码规范
- **逐步细化：** 先让 AI 给出方案大纲，再逐步深入细节
- **利用历史：** 用 `/restore` 恢复之前的对话，避免重复说明背景

### 13.3 避免常见错误

- 不要一次性处理过多文件
- 不要忽略 AI 提示的风险警告
- 不要在未理解代码的情况下直接提交
- 不要忘记定期检查 Token 消耗

---

## 14. 参考资源

- [Gemini CLI GitHub 仓库](https://github.com/google-gemini/gemini-cli)
- [Google AI Studio](https://aistudio.google.com/)
- [Gemini API 文档](https://ai.google.dev/docs)
- [MCP 协议文档](https://modelcontextprotocol.io/)

---

> 最后更新：2026-03-10

祝您在 AI 辅助开发的旅程中如虎添翼！