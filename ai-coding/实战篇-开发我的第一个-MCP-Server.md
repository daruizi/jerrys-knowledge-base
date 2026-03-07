# 实战篇：开发你的第一个 MCP Server (本地灵感备忘录)

> **导读**
> 
> 你可能已经用过了 Claude Code 或 OpenClaw，也体验过官方的 MCP Server（如 GitHub、Google Drive）。但通用工具永远无法满足极其个性化的本地需求！
> 
> 今天，我们将手把手带你用 TypeScript 开发一个名为 **`my-notes-mcp`** 的极简 Server。
> 
> **它的能力**：让 AI 拥有读写你本地特定文件夹的能力。当你在终端或钉钉里对 AI 说：“刚才想到了一个关于短视频脚本的灵感，帮我记下来”，AI 会自动将这段话排版成优雅的 Markdown 并保存到你的本地硬盘。

---

## 一、 环境准备 (武器库搭建)

即使你是前端小白，跟着这三步走也能搞定。

### 1. 安装基础环境
确保你的电脑安装了 **Node.js** (推荐 v18 或以上版本)。在终端输入 `node -v` 检查。

### 2. 初始化项目结构
在你喜欢的目录下，打开系统终端抓取我们需要的依赖包：

```bash
# 创建并进入文件夹
mkdir my-notes-mcp && cd my-notes-mcp

# 初始化 package.json (-y 表示全采用默认选项)
npm init -y

# 安装最新的官方 SDK 和 TypeScript 相关工具
npm install @modelcontextprotocol/sdk
npm install -D typescript @types/node ts-node
```

### 3. 配置 TypeScript
为了让代码跑起来，我们需要快速初始化一份 TypeScript 配置文件：

```bash
npx tsc --init
```

打开刚刚生成的 `tsconfig.json`文件，确保里面这两行没有被注释掉：
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "moduleResolution": "node",
    // 其他配置保持默认即可
  }
}
```

---

## 二、 核心代码拆解 (造物主的工作)

在项目根目录下新建一个名为 `index.ts` 的文件，这是我们的所有逻辑。

为了让 AI 帮你记笔记，我们需要给它两个工具（Tools）：
1. `add_note`：让 AI 用来写文件。
2. `list_notes`：让 AI 看看你以前都记了什么。

打开 `index.ts`，分段开始复制：

### 1. 引入护甲与武器库

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";
import * as fs from "fs/promises";
import * as path from "path";
import * as os from "os";

// 定义我们笔记的专属存储路径：保存在你家目录下的 .my-notes 文件夹中
const NOTES_DIR = path.join(os.homedir(), ".my-notes");

// 实例化我们的 MCP Server
const server = new Server(
  {
    name: "my-notes-mcp", // 你的插件大名
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {}, // 声明我们要使用工具能力
    },
  }
);
```

### 2. 告诉 AI “我会干什么” (注册工具表)

当 AI 连接上你的 Server 时，它会问：“你能帮我做什么？” 我们得把工具的使用说明书告诉它。

```typescript
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "add_note", // AI 需要调用的确切函数名
        description: "保存一个新的灵感或备忘录到用户的本地笔记库中。",
        inputSchema: {     // 告诉 AI 调用时必须按什么格式传参数 (JSON Schema)
          type: "object",
          properties: {
            title: {
              type: "string",
              description: "笔记的标题，例如 '短视频脚本灵感'",
            },
            content: {
              type: "string",
              description: "笔记的具体内容（支持 Markdown 语法）",
            },
          },
          required: ["title", "content"], // 这俩参数缺一不可
        },
      },
      {
        name: "list_notes",
        description: "获取用户之前保存的所有笔记的文件名列表。",
        inputSchema: {
          type: "object",
          properties: {}, // 这个接口不需要参数
        },
      },
    ],
  };
});
```

### 3. 给 AI “打工” (执行业务逻辑)

当 AI 在对话中决定帮你记笔记时，它就会正式调用你的工具。这里的代码是真正帮它落地的动作。

```typescript
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  // 确保目标文件夹存在，如果没有就自动创建一个
  await fs.mkdir(NOTES_DIR, { recursive: true });

  // 判断 AI 到底要用你的哪一个工具
  switch (request.params.name) {
    
    // --- 场景 1：AI 想写笔记 ---
    case "add_note": {
      // 从 AI 传过来的参数里把标题和内容提取出来
      const { title, content } = request.params.arguments as {
        title: string;
        content: string;
      };

      // 为了防止非法文件名，简单替换掉空格和特殊字符
      const safeTitle = title.replace(/[^a-zA-Z0-9\u4e00-\u9fa5]/g, "-");
      const filePath = path.join(NOTES_DIR, `${safeTitle}.md`);

      // 真正写入硬盘的时刻
      await fs.writeFile(filePath, content, "utf-8");

      // 返回执行结果给 AI，让它知道自己干成功了
      return {
        content: [
          {
            type: "text",
            text: `✅ 笔记已成功存入本地盘：${filePath}`,
          },
        ],
      };
    }

    // --- 场景 2：AI 想看看以前的笔记 ---
    case "list_notes": {
      try {
        const files = await fs.readdir(NOTES_DIR);
        const mdFiles = files.filter(f => f.endsWith('.md'));
        
        return {
          content: [
            {
              type: "text",
              text: mdFiles.length > 0 
                ? `现有的笔记有：\n${mdFiles.join('\n')}`
                : "当前笔记库空空如也~",
            },
          ],
        };
      } catch (e) {
         // 处理极端情况：比如文件夹压根不存在
         return {
          content: [{ type: "text", text: "当前笔记库空空如也~" }],
        };
      }
    }

    // --- 无效场景 ---
    default:
      throw new Error(`未知工具：${request.params.name}`);
  }
});
```

### 4. 点火发射 (启动 Server)

在文件极末尾，加入启动监听。标准流水线（Stdio）模式是如今绝大部分端程序的首选对接方式。

```typescript
async function run() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("🚀 my-notes-mcp 已成功在 Stdio 总线上启动！");
}

run().catch((error) => {
  console.error("启动崩溃:", error);
  process.exit(1);
});
```

---

## 三、 本地封神 (将神器接入系统)

代码写好了，如何让大模型承认它的存在？接下来我们编译它，并喂给你的终端 AI。

### 1. 编译并拿到执行绝对路径
在项目终端运行：

```bash
# 将 TypeScript 编译为在 Node.js 上能跑的 JavaScript
npx tsc

# 假设你当前路径在 /Users/Jerry/my-notes-mcp (Mac) 或 C:\my-notes-mcp (Win)
# 那么你的执行文件就是： /Users/Jerry/my-notes-mcp/index.js
```
*请务必记下编译后 `index.js` 所在的**绝对路径**。*

### 2. 方案 A：配给 Claude Code (官方终端)

执行 `claude mcp add` 命令，将你的新武器挂载上去（替换为你真实的绝对路径）：

```bash
claude mcp add myNotes node /Users/Jerry/my-notes-mcp/index.js
```

### 3. 方案 B：配给 OpenClaw (你的 24 小时钉钉助理)

编辑 `~/.openclaw/openclaw.json` (或你指定的配置文件)，新增一个 provider：

```json
{
  "mcp": {
    "providers": {
      "myNotes": {
        "command": "node",
        "args": ["/Users/Jerry/my-notes-mcp/index.js"]
      }
    }
  }
}
```

重启 OpenClaw 后，钉钉里的 AI 就拥有了这个武器。

---

## 四、 见证奇迹时刻

打开 Claude Code 或在钉钉中对 OpenClaw 说话：

> **👨 你的原话：**
> “AI，我刚才灵光一现。帮我记一下：未来的 LLM 应用开发绝不是天天拼 Prompt，而是谁能写出更棒的 MCP Server，这就像是从造枪转向了造子弹。请用优雅的 Markdown 帮我把这段话保存入库。”

> **🤖 AI 的回答 (Claude/OpenClaw 将在思考后做出)：**
> *(后台自动调用 `add_note` 工具...)*
> “报告主人，灵感已收录！我已经将其排版存储在 `~/.my-notes/未来的-LLM-应用开发.md` 中。”

现在打开你的本地文件夹看看吧，一份崭新由 AI 理解并排版生成的 `.md` 文件已经静静地躺在那里了。

🎉 **恭喜！你刚刚亲手打破了大模型的“沙盒”，让它获得了操纵你物理硬盘的能力，并完成了从“用工具”到“造工具”的技术跨越。**
