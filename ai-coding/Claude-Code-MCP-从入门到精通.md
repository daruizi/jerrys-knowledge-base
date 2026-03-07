# Claude Code - MCP 从入门到精通

> Model Context Protocol (MCP) 是连接 AI 应用与外部系统的开放标准，本教程将带你从零开始掌握 MCP 的核心概念、配置使用和开发技巧。

---

## 📖 目录

1. [什么是 MCP？](#什么是-mcp)
2. [MCP 核心概念](#mcp-核心概念)
3. [MCP 架构解析](#mcp-架构解析)
4. [在 Claude Code 中使用 MCP](#在-claude-code-中使用-mcp)
5. [创建你的第一个 MCP Server](#创建你的第一个-mcp-server)
6. [实战案例：天气查询 MCP Server](#实战案例天气查询-mcp-server)
7. [MCP 最佳实践](#mcp-最佳实践)
8. [常见问题与排错](#常见问题与排错)

---

## 什么是 MCP？

### 一句话解释

**MCP 就像是 AI 的 "USB-C 接口"** —— 它提供了一个标准化的方式，让 AI 应用（如 Claude Code）能够连接到各种外部系统（数据库、API、文件系统等）。

### 为什么需要 MCP？

在 MCP 出现之前，如果你想让 Claude 访问你的数据库、查询 Jira 工单、或者操作 GitHub，你需要：

1. 为每个服务写专门的集成代码
2. 维护多套不同的 API 接口
3. 处理各种认证和授权问题

**MCP 解决了这些问题**：
- ✅ **一次开发，到处运行** —— 写一个 MCP Server，就能在 Claude Code、Claude Desktop、Cursor、VS Code 等支持 MCP 的客户端中使用
- ✅ **标准化协议** —— 统一的数据交换格式，降低学习成本
- ✅ **丰富的生态** —— 官方和社区提供了大量现成的 MCP Server

### MCP 能做什么？

| 场景 | 示例 |
|------|------|
| **访问数据库** | 查询 PostgreSQL、MySQL、MongoDB 等 |
| **操作代码仓库** | 创建 Issue、提交 PR、查看代码 |
| **管理任务** | 操作 Jira、Notion、Linear 等项目管理工具 |
| **数据分析** | 连接 Sentry、Statsig 等监控平台 |
| **文件操作** | 读写本地文件、操作云存储 |
| **API 集成** | 调用任意 HTTP API |

---

## MCP 核心概念

### 三大核心组件

MCP 定义了三个核心概念，理解它们是掌握 MCP 的关键：

#### 1. Tools（工具）

**工具**是 AI 可以主动调用的函数，用于执行具体操作。

```
类比：工具就像是 AI 的"手"，让它能够执行操作
```

**示例工具：**
- `query_database` - 执行 SQL 查询
- `create_issue` - 创建 GitHub Issue
- `send_email` - 发送邮件

#### 2. Resources（资源）

**资源**是 AI 可以读取的数据源，类似于文件。

```
类比：资源就像是 AI 的"书架"，让它能够查阅信息
```

**示例资源：**
- `file://config.json` - 配置文件
- `database://schema` - 数据库结构
- `api://users` - 用户列表

#### 3. Prompts（提示词模板）

**提示词**是预定义的交互模板，帮助用户快速执行常见任务。

```
类比：提示词就像是"快捷键"，一键执行复杂操作
```

**示例提示词：**
- `/review-pr` - 代码审查模板
- `/create-issue` - 创建 Issue 模板

### 传输方式

MCP 支持两种传输方式：

| 传输方式 | 适用场景 | 特点 |
|----------|----------|------|
| **Stdio** | 本地进程 | 性能最佳，适合本地工具 |
| **HTTP/SSE** | 远程服务 | 支持云服务，可跨网络 |

---

## MCP 架构解析

### 客户端-服务器模型

```
┌─────────────────────────────────────────────────────┐
│                   MCP Host                          │
│              (如 Claude Code)                       │
│                                                     │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐      │
│  │MCP Client │  │MCP Client │  │MCP Client │      │
│  │    (1)    │  │    (2)    │  │    (3)    │      │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘      │
└────────┼──────────────┼──────────────┼─────────────┘
         │              │              │
         ▼              ▼              ▼
   ┌──────────┐   ┌──────────┐   ┌──────────┐
   │MCP Server│   │MCP Server│   │MCP Server│
   │ (GitHub) │   │ (Sentry) │   │(Database)│
   └──────────┘   └──────────┘   └──────────┘
```

### 角色说明

| 角色 | 说明 | 示例 |
|------|------|------|
| **MCP Host** | 运行 AI 应用的主程序 | Claude Code、VS Code、Cursor |
| **MCP Client** | 管理与服务器的连接 | 由 Host 自动创建和管理 |
| **MCP Server** | 提供工具、资源、提示词 | GitHub Server、Sentry Server |

### 工作流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant Host as Claude Code
    participant Server as MCP Server

    User->>Host: 提问："帮我查看最近的错误"
    Host->>Server: 1. 初始化连接
    Server-->>Host: 返回支持的 capabilities
    Host->>Server: 2. 获取工具列表 (tools/list)
    Server-->>Host: 返回可用工具
    Host->>Server: 3. 调用工具 (tools/call)
    Server-->>Host: 返回结果
    Host-->>User: 展示分析结果
```

---

## 在 Claude Code 中使用 MCP

### 查看已配置的 MCP 服务器

在 Claude Code 会话中输入：

```text
/mcp
```

这会显示所有已配置的 MCP 服务器及其状态。

### 添加 MCP 服务器

#### 方式一：添加远程 HTTP 服务器（推荐）

```bash
# 基本语法
claude mcp add --transport http <名称> <URL>

# 示例：添加 GitHub MCP
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 示例：添加 Sentry MCP
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

#### 方式二：添加本地 Stdio 服务器

```bash
# 基本语法
claude mcp add --transport stdio <名称> -- <命令> [参数...]

# 示例：添加文件系统 MCP
claude mcp add --transport stdio filesystem -- npx -y @modelcontextprotocol/server-filesystem /path/to/allowed/dir

# 示例：添加数据库 MCP（带环境变量）
claude mcp add --transport stdio \
  --env DATABASE_URL="postgresql://user:pass@localhost:5432/mydb" \
  mydb -- npx -y @bytebase/dbhub
```

#### 方式三：使用 JSON 配置添加

```bash
# 从 JSON 配置添加服务器
claude mcp add-json my-server '{"type":"http","url":"https://api.example.com/mcp","headers":{"Authorization":"Bearer token"}}'
```

### 管理 MCP 服务器

```bash
# 列出所有已配置的服务器
claude mcp list

# 查看特定服务器详情
claude mcp get github

# 删除服务器
claude mcp remove github
```

### 配置作用域

MCP 服务器可以配置在不同的作用域：

| 作用域 | 命令参数 | 存储位置 | 用途 |
|--------|----------|----------|------|
| **local** | `--scope local` | `~/.claude.json` | 仅当前项目可用（默认） |
| **project** | `--scope project` | `.mcp.json` | 团队共享，提交到版本控制 |
| **user** | `--scope user` | `~/.claude.json` | 所有项目可用 |

```bash
# 添加项目级 MCP（团队共享）
claude mcp add --scope project --transport http github https://api.githubcopilot.com/mcp/

# 添加用户级 MCP（个人工具）
claude mcp add --scope user --transport http notion https://mcp.notion.com/mcp
```

### 项目级配置文件 (.mcp.json)

对于团队共享的 MCP 配置，可以在项目根目录创建 `.mcp.json`：

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "database": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@bytebase/dbhub"],
      "env": {
        "DATABASE_URL": "${DB_URL}"
      }
    }
  }
}
```

> **注意**：敏感信息（如 API Key）不要直接写在 `.mcp.json` 中，应该使用环境变量 `${VAR_NAME}` 的形式。

### 认证配置

许多 MCP 服务器需要认证。Claude Code 支持 OAuth 2.0 认证：

```bash
# 1. 添加需要认证的服务器
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. 在 Claude Code 会话中完成认证
/mcp
# 选择对应的服务器，按照提示完成浏览器认证
```

---

## 创建你的第一个 MCP Server

### 环境准备

确保你的系统已安装：
- Node.js 18+ 或 Python 3.10+
- npm 或 uv（Python 包管理器）

### 使用 TypeScript 创建 MCP Server

#### 步骤 1：创建项目

```bash
# 创建项目目录
mkdir my-first-mcp-server
cd my-first-mcp-server

# 初始化项目
npm init -y

# 安装依赖
npm install @modelcontextprotocol/sdk zod
npm install -D typescript @types/node

# 创建源码目录
mkdir src
```

#### 步骤 2：配置 TypeScript

创建 `tsconfig.json`：

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
```

#### 步骤 3：创建 MCP Server

创建 `src/index.ts`：

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// 创建 MCP Server 实例
const server = new McpServer({
  name: "my-first-mcp-server",
  version: "1.0.0",
});

// 定义一个简单的工具：加法计算器
server.tool(
  "add",                                          // 工具名称
  "Add two numbers together",                     // 工具描述
  {
    a: z.number().describe("First number"),       // 参数 a
    b: z.number().describe("Second number"),      // 参数 b
  },
  async ({ a, b }) => {
    const result = a + b;
    return {
      content: [
        {
          type: "text",
          text: `The sum of ${a} and ${b} is ${result}`,
        },
      ],
    };
  }
);

// 定义另一个工具：问候语生成
server.tool(
  "greet",
  "Generate a personalized greeting message",
  {
    name: z.string().describe("Name to greet"),
    language: z.enum(["english", "chinese", "spanish"]).optional().default("english").describe("Language for greeting"),
  },
  async ({ name, language }) => {
    const greetings = {
      english: `Hello, ${name}! Nice to meet you!`,
      chinese: `你好，${name}！很高兴认识你！`,
      spanish: `¡Hola, ${name}! ¡Mucho gusto!`,
    };
    return {
      content: [
        {
          type: "text",
          text: greetings[language],
        },
      ],
    };
  }
);

// 启动服务器
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("MCP Server started successfully!");
}

main().catch((error) => {
  console.error("Server error:", error);
  process.exit(1);
});
```

#### 步骤 4：更新 package.json

```json
{
  "name": "my-first-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/index.js",
  "bin": {
    "my-mcp-server": "dist/index.js"
  },
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

#### 步骤 5：构建并测试

```bash
# 编译 TypeScript
npm run build

# 本地测试
node dist/index.js
```

#### 步骤 6：在 Claude Code 中使用

```bash
# 添加到 Claude Code
claude mcp add --transport stdio my-first-mcp -- node /path/to/my-first-mcp-server/dist/index.js

# 验证已添加
claude mcp list
```

现在你可以在 Claude Code 中使用这些工具了：

```text
> 请使用 add 工具计算 123 + 456
> 请用中文问候张三
```

---

## 实战案例：天气查询 MCP Server

让我们创建一个实用的天气查询 MCP Server，它会调用免费的天气 API。

### 完整代码

创建 `src/weather-server.ts`：

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "weather-server",
  version: "1.0.0",
});

// 天气查询工具
server.tool(
  "get_weather",
  "Get current weather information for a city",
  {
    city: z.string().describe("City name (e.g., 'Beijing', 'Shanghai')"),
  },
  async ({ city }) => {
    try {
      // 使用免费的 wttr.in API
      const response = await fetch(
        `https://wttr.in/${encodeURIComponent(city)}?format=j1`
      );

      if (!response.ok) {
        throw new Error(`Weather API error: ${response.status}`);
      }

      const data = await response.json();
      const current = data.current_condition[0];

      const weatherInfo = {
        city: city,
        temperature: `${current.temp_C}°C / ${current.temp_F}°F`,
        description: current.weatherDesc[0].value,
        humidity: `${current.humidity}%`,
        wind: `${current.windspeedKmph} km/h ${current.winddir16Point}`,
        feelsLike: `${current.FeelsLikeC}°C`,
      };

      return {
        content: [
          {
            type: "text",
            text: JSON.stringify(weatherInfo, null, 2),
          },
        ],
      };
    } catch (error) {
      return {
        content: [
          {
            type: "text",
            text: `Error fetching weather: ${error instanceof Error ? error.message : 'Unknown error'}`,
          },
        ],
        isError: true,
      };
    }
  }
);

// 天气预报工具
server.tool(
  "get_forecast",
  "Get 3-day weather forecast for a city",
  {
    city: z.string().describe("City name"),
  },
  async ({ city }) => {
    try {
      const response = await fetch(
        `https://wttr.in/${encodeURIComponent(city)}?format=j1`
      );

      if (!response.ok) {
        throw new Error(`Weather API error: ${response.status}`);
      }

      const data = await response.json();
      const forecast = data.weather.slice(0, 3).map((day: any) => ({
        date: day.date,
        maxTemp: `${day.maxtempC}°C`,
        minTemp: `${day.mintempC}°C`,
        avgTemp: `${day.avgtempC}°C`,
        description: day.hourly[4].weatherDesc[0].value,
        chanceOfRain: `${day.hourly[4].chanceofrain}%`,
      }));

      return {
        content: [
          {
            type: "text",
            text: JSON.stringify({ city, forecast }, null, 2),
          },
        ],
      };
    } catch (error) {
      return {
        content: [
          {
            type: "text",
            text: `Error fetching forecast: ${error instanceof Error ? error.message : 'Unknown error'}`,
          },
        ],
        isError: true,
      };
    }
  }
);

// 启动服务器
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Weather MCP Server started!");
}

main().catch(console.error);
```

### 添加资源支持

MCP 服务器还可以暴露资源：

```typescript
// 添加资源：支持的城市列表
server.resource(
  "supported-cities",
  "weather://cities",
  { mimeType: "application/json" },
  async () => {
    const cities = [
      { name: "Beijing", country: "China" },
      { name: "Shanghai", country: "China" },
      { name: "New York", country: "USA" },
      { name: "London", country: "UK" },
      { name: "Tokyo", country: "Japan" },
    ];
    return {
      contents: [
        {
          uri: "weather://cities",
          mimeType: "application/json",
          text: JSON.stringify(cities, null, 2),
        },
      ],
    };
  }
);
```

### 使用提示词模板

```typescript
// 添加提示词模板
server.prompt(
  "weather-report",
  "Generate a weather report for a city",
  { city: z.string() },
  ({ city }) => ({
    messages: [
      {
        role: "user",
        content: {
          type: "text",
          text: `Please get the current weather for ${city} and provide a brief summary.`,
        },
      },
    ],
  })
);
```

---

## MCP 最佳实践

### 1. 工具设计原则

#### 好的工具设计

```typescript
// ✅ 好的设计：单一职责，清晰的描述
server.tool(
  "search_issues",
  "Search for issues in a GitHub repository by keywords",
  {
    query: z.string().describe("Search keywords"),
    repo: z.string().optional().describe("Repository in format 'owner/repo'"),
  },
  async ({ query, repo }) => { /* ... */ }
);
```

#### 避免的设计

```typescript
// ❌ 不好的设计：功能过于复杂，描述不清晰
server.tool(
  "do_something",
  "Do something useful",  // 描述太模糊
  {
    data: z.any().describe("Some data"),  // 类型不明确
  },
  async ({ data }) => { /* ... */ }
);
```

### 2. 错误处理

```typescript
server.tool(
  "safe_operation",
  "A tool with proper error handling",
  { input: z.string() },
  async ({ input }) => {
    try {
      // 执行操作
      const result = await performOperation(input);
      return {
        content: [{ type: "text", text: JSON.stringify(result) }],
      };
    } catch (error) {
      // 返回结构化的错误信息
      return {
        content: [{
          type: "text",
          text: `Operation failed: ${error instanceof Error ? error.message : 'Unknown error'}`,
        }],
        isError: true,
      };
    }
  }
);
```

### 3. 安全考虑

```typescript
// 使用环境变量存储敏感信息
server.tool(
  "api_call",
  "Make authenticated API call",
  { endpoint: z.string() },
  async ({ endpoint }) => {
    const apiKey = process.env.API_KEY;
    if (!apiKey) {
      return {
        content: [{ type: "text", text: "Error: API_KEY not configured" }],
        isError: true,
      };
    }
    // 使用 apiKey 进行 API 调用
  }
);
```

### 4. 资源设计

```typescript
// 为数据提供清晰的 URI 结构
server.resource(
  "user-profile",
  "users://profile",  // 清晰的资源标识
  { mimeType: "application/json" },
  async () => { /* ... */ }
);

server.resource(
  "user-settings",
  "users://settings",  // 一致的命名规范
  { mimeType: "application/json" },
  async () => { /* ... */ }
);
```

### 5. 配置文件规范

`.mcp.json` 文件的推荐格式：

```json
{
  "mcpServers": {
    "server-name": {
      "type": "http",
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

---

## 常见问题与排错

### Q1: MCP 服务器连接失败

**症状**：`/mcp` 显示服务器状态为 "Connection failed"

**解决方案**：
1. 检查服务器是否正在运行
2. 验证 URL 或命令路径是否正确
3. 查看日志：`claude mcp get <server-name>`

```bash
# 检查本地服务器是否能正常运行
node /path/to/server/dist/index.js

# 测试远程服务器连接
curl -X POST https://api.example.com/mcp
```

### Q2: 工具调用超时

**症状**：工具调用长时间无响应

**解决方案**：
```bash
# 增加超时时间（毫秒）
MCP_TIMEOUT=30000 claude
```

### Q3: Windows 上 npx 命令失败

**症状**：在 Windows 上出现 "Connection closed" 错误

**解决方案**：
```bash
# Windows 需要使用 cmd /c 包装
claude mcp add --transport stdio my-server -- cmd /c npx -y @some/package
```

### Q4: OAuth 认证失败

**症状**：认证后仍然无法使用服务器

**解决方案**：
1. 使用 `/mcp` 查看认证状态
2. 尝试 "Clear authentication" 重新认证
3. 检查浏览器是否正确重定向

### Q5: 工具输出被截断

**症状**：看到 "Output exceeds limit" 警告

**解决方案**：
```bash
# 增加输出限制
export MAX_MCP_OUTPUT_TOKENS=50000
claude
```

### Q6: 如何调试 MCP Server

**方法一：使用 MCP Inspector**

```bash
# 安装并运行 MCP Inspector
npx @modelcontextprotocol/inspector node dist/index.js
```

这会在浏览器中打开一个调试界面，可以：
- 查看服务器提供的工具
- 测试工具调用
- 查看通信日志

**方法二：添加日志**

```typescript
// 在服务器代码中添加日志（输出到 stderr）
console.error("Debug: Processing request for", input);
```

---

## 快速参考卡片

### 常用命令

```bash
# 查看 MCP 状态
/mcp

# 添加 HTTP 服务器
claude mcp add --transport http <name> <url>

# 添加本地服务器
claude mcp add --transport stdio <name> -- <command>

# 列出所有服务器
claude mcp list

# 查看服务器详情
claude mcp get <name>

# 删除服务器
claude mcp remove <name>
```

### 工具定义模板

```typescript
server.tool(
  "tool-name",                    // 工具名称
  "Tool description",             // 工具描述
  {                               // 参数定义
    param1: z.string().describe("Parameter description"),
  },
  async ({ param1 }) => {         // 处理函数
    return {
      content: [{ type: "text", text: "Result" }],
    };
  }
);
```

### 资源定义模板

```typescript
server.resource(
  "resource-name",
  "resource://uri",
  { mimeType: "application/json" },
  async () => ({
    contents: [{
      uri: "resource://uri",
      mimeType: "application/json",
      text: JSON.stringify(data),
    }],
  })
);
```

---

## 推荐资源

### 官方文档

- [MCP 官方网站](https://modelcontextprotocol.io/)
- [MCP 规范文档](https://modelcontextprotocol.io/specification/latest)
- [Claude Code MCP 文档](https://code.claude.com/docs/en/mcp)

### 示例代码

- [官方 MCP Server 示例](https://github.com/modelcontextprotocol/servers)
- [MCP SDK (TypeScript)](https://github.com/modelcontextprotocol/typescript-sdk)
- [MCP SDK (Python)](https://github.com/modelcontextprotocol/python-sdk)

### 社区资源

- [Awesome MCP](https://github.com/punkpeye/awesome-mcp-servers) - MCP 服务器资源汇总
- [MCP Inspector](https://github.com/modelcontextprotocol/inspector) - 调试工具

---

## 总结

MCP 是连接 AI 与外部系统的标准化协议，通过本教程，你已经学会了：

| 知识点 | 掌握程度 |
|--------|----------|
| MCP 核心概念（Tools、Resources、Prompts） | ✅ |
| 在 Claude Code 中配置和使用 MCP | ✅ |
| 使用 TypeScript 创建 MCP Server | ✅ |
| 错误处理和安全最佳实践 | ✅ |
| 常见问题的排查方法 | ✅ |

下一步，你可以：
1. 探索 [官方 MCP Server 仓库](https://github.com/modelcontextprotocol/servers) 中的示例
2. 为你常用的工具创建 MCP Server
3. 将你的 MCP Server 分享给社区

---

> 📅 最后更新：2026-03-07
> 📚 更多 AI Coding 相关内容请查看 [ai-coding 目录](./README.md)