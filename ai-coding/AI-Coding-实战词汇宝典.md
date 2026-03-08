# AI Coding 实战词汇宝典 (大模型应用开发必备黑话)

> **导读**
> 
> 在过去，AI 词汇表里堆满了“梯度下降”、“损失函数”等晦涩的算法黑话。但到了大模型时代，作为一名前端/后端或依靠 AI 辅助编程的开发者，你**完全不需要**关心底层是怎么“炼丹”的。
> 
> 本文档专为 **刚接触 AI 结对编程 (如 Claude Code、Cursor) 和 Agent 开发 (如 OpenClaw、MCP)** 的新手打造。我们摒弃了纯学术词汇，只留下在你日常与大模型“打交道”、“做应用”时，最高频、最实用的**工程黑话**。

---

## 一、 AI 结对编程核心交互词 (日常最高频)

在日常使用 Cursor 或 Claude Code 写代码时，你每天都会用到它们。

### 1. Prompt (提示词)
**释义**：你输入给 AI 的指令或问题。
**实战语境**：“这块代码生成的质量太差了，我得优化一下我的 Prompt，把需求描述得再详细点。”

### 2. System Prompt / Instructions (系统提示词 / 系统指令)
**释义**：隐藏在用户对话背后，用于给 AI 设定“人设”、“规矩”和“底线”的顶级指令。
**实战语境**：“在项目根目录建一个 `CLAUDE.md`，里面写满我们的 System Prompt，强制要求 AI 必须用 TypeScript 写代码，不能用 Any。”

### 3. Context (上下文)
**释义**：AI 此时此刻脑子里掌握的背景信息（包含了你们的聊天记录，以及它读取过的代码文件）。
**实战语境**：“不要一上来就让 AI 改 Bug，先让它把报错日志和相关的 3 个核心组件都读一遍，把 Context 喂饱了再提问。”

### 4. Artifact (交付物 / 独立组件)
**释义**：在 AI 编程中，指 AI 生成的、能独立预览或保存的**完整代码块、网页或文档**（而不是在对话框里挤牙膏式的文本）。
**实战语境**：“我用 Claude 查阅了一堆资料，最后让它直接生成一个完整的 Markdown Artifact 保存到本地。”

### 5. Session / Thread (会话 / 线程)
**释义**：你和 AI 的一次完整对话流程。不同 Session 之间的记忆是隔离的。
**实战语境**：“昨天那个 Session 的上下文太长了，AI 开始胡言乱语 (幻觉) 了，我们切一个新 Session 重新挑起话题吧。”

---

## 二、 API 与模型参数 (开发者必懂)

如果你在使用 OpenClaw，或者自己用 Python/Node 写脚本调用大模型，这些词必须烂熟于心。

### 1. Token (词元)
**释义**：大模型计费和处理文字的**最小计算/存储碎片**。不要把它等同于“一个字”。在现代大模型（如 Qwen, DeepSeek）优秀的中文分词器下，通常 1 个汉字 = 0.5~1 个 Token。
**实战语境**：“这哪怕开源模型再便宜，你一次性把几百万个 Token 的项目全塞给它阅读，API 账单也会爆炸的。”

### 2. Context Window (上下文窗口)
**释义**：一个模型一次性最多能记住多少个 Token。超出这个窗口的内容会被模型直接“遗忘”。
**实战语境**：“Claude 3.5 Sonnet 支持 200K 的 Context Window，这意味着你可以直接把一本半的书或者一个中型项目的源码一次性塞给它分析。”

### 3. Temperature (温度 / 创造力指数)
**释义**：控制 AI 回答“随机性”的参数。通常范围在 `0` 到 `1` 或 `2`。
**实战语境**：“我现在的需求是让 AI 帮我严格提取 JSON 数据，把 Temperature 设置为 `0`（绝对确定）；如果要写营销文案，设置为 `0.8`（更有创意）。”

### 4. Streaming (流式输出)
**释义**：让 AI 像打字机一样，想出一个字就吐出一个字的技术，而不是等了几十秒才一次性给出完整的长篇大论。
**实战语境**：“我们现在的聊天 UI 体验太差了，用户等得以为卡死了，赶紧把接口改成 Streaming 模式首字秒出。”

### 5. Prompt Caching (提示词/上下文缓存)
**释义**：长文本时代（2024年底兴起）的降本杀手锏。当你频繁向模型（如 Claude 或 DashScope）发送相同的超大 Context（如你的整个系统架构源码）时，API 平台会自动在内存中缓存这段阅读记忆。第二次提问时，不仅速度起飞，计费也会呈断崖式下跌（通常下降 50%~90%）。
**实战语境**：“我们把系统的核心知识库挂在提示词头部，千万别乱改里面的字，这样就能一直击中 Prompt Caching，省钱又极速响应。”

### 6. Rate Limit (并发拦截 / 速率限制)
**释义**：AI 厂商为了防刷接口，限制你每分钟能发几次请求 (RPM)，或每分钟能消耗多少 Token (TPM)。
**实战语境**：“程序崩溃了！原来是我们在循环里高频调用接口，触发了 Rate Limit 的 429 报错。”

### 7. Reasoning Model (推理模型 / 慢思考模型)
**释义**：2025年最新范式（如 OpenAI o1, DeepSeek R1）。区别于传统的“快答模型”，它能在给出最终答案前，生成一段内化的“思维链树 (Think Block)”。
> ⚠️ **实战大坑**：面对推理模型，**极其忌讳**再给它写那种繁琐的、充满 Few-shot 示例的保姆级 Prompt。你只需要告诉它**“最终目标(What)”**，怎么推演实现**“(How)”**让它自己去“慢思考”！
**实战语境**：“用 DeepSeek R1 解这种高阶算法题，不要去干涉它的解题流，丢一个干瘪的需求过去，等它在后台疯狂转圈慢慢 Reasoning 完就行了。”

---

## 三、 Agent 工程与高级开发 (核心干货)

这正是 OpenClaw、MCP 和自动化流水线（Skills）所解决的核心痛点。

### 1. Agent (智能体 / 数字助理)
**释义**：不再只是“一问一答”的聊天机器人。Agent 是带有目标，能**利用工具自己干活**的自动化闭环。
**实战语境**：“我们用 OpenClaw 部署了一个钉钉 Agent，你只要说句话，它就会自己去服务器上敲 Bash 命令重启 Nginx。”

### 2. Function Calling / Tool Use (函数调用 / 工具使用)
**释义**：赋予大模型“手”和“脚”的核心技术。模型自己无法执行代码、无法上网，但它能通过输出标准格式（如 JSON），指示宿主程序去调 API 办事。
**实战语境**：“这个模型很笨，不支持 Tool Use，如果用它，它就无法在 OpenClaw 里执行读写文件的操作。”

### 3. MCP (Model Context Protocol / 模型上下文协议)
**释义**：由 Anthropic (Claude) 首创的一种**标准级插件协议**。相当于给大模型装上了一个个“USB 接口”。
**实战语境**：“不要硬编码查天气的逻辑了！直接拉一个官方的天气 MCP Server，所有的 Agent 就能即插即用了。”

### 4. RAG (Retrieval-Augmented Generation / 检索增强生成)
**释义**：防止 AI “胡说八道 (幻觉)”的最佳方案。当 AI 遇到不懂的企业内部知识时，先让它去数据库里**搜 (Retrieve)** 资料，把搜到的资料和问题一起甩给大模型去**总结 (Generate)**。
**实战语境**：“客服 Agent 回答产品参数总是不对，我们需要给它挂载一个 RAG 知识库系统，让它照着产品说明书来回答。”

### 5. Setup / Sandbox (环境 / 沙箱)
**释义**：出于安全考虑，将 AI 隔离在一个有限权限的壳子里（如 Docker），防止它误删核心文件。
**实战语境**：“给 AI 开放 Bash 工具权限非常危险！一定要在 `openclaw.json` 里配置 Docker Sandbox，把它锁死在容器里玩泥巴。”

---

## 四、 AI 协同代码黑话 (Prompt 高效动词表)

在和 AI 结对编程时，不要用“请你帮我弄一下这个”这种模糊的词，请使用以下标准动词。

| 动词 (推荐) | 释义 | 实战 Prompt 示例 |
|:---|:---|:---|
| **Explain** | 解释逻辑 | "Explain the logic of this regex in pure Chinese." (用纯中文解释这段正则) |
| **Review** | 代码审查 | "Review this Pull Request and point out any potential security vulnerabilities." (审查这段代码并指出所有潜在的安全漏洞) |
| **Refactor** | 重构代码 | "Refactor this huge function into 3 smaller, independent helpers." (把这个屎山函数重构为3个独立的方法) |
| **Translate** | 语言迁移 | "Translate this old Python 2 script into a modern TypeScript Node.js module." (把这个旧的 Python 脚本翻译成现代 TS 模块) |
| **Scaffold** | 搭脚手架 | "Scaffold a standard React component with Tailwind CSS." (快速帮我搭一个带Tailwind的React组件架子) |
| **Debug** | 调试纠错 | "I'm getting a `TypeError: undefined` here. Help me debug this issue." (帮我调试一下这个未定义的报错) |
| **Optimize** | 优化性能 | "Optimize this sorting algorithm to run in O(n log n) time." (把这个排序算法的复杂度优化到 O(n log n)) |
| **Mock** | 造假数据 | "Mock a JSON payload with 10 random imaginary users for testing." (给我伪造10个测试用的 JSON 用户数据) |

---

## 五、 传统算法底层概念 (混圈子仅作了解)

虽然日常工程不需要懂，但如果你在知乎或推特上冲浪，大概率会看到：

1. **LLM (Large Language Model)**：大语言模型。一切的基础，比如 GPT-4, Claude 3.5, 或者是你要装在本地的 Ollama (Llama 3, Qwen)。
2. **Transformer**：2017 年 Google 提出的一种轰动世界的深度学习架构，是现在所有 LLM 能读懂人类语言的底层地基。
3. **Embedding (词向量嵌入)**：把人类懂的“语言”转换为机器能算距离的“多维坐标（密集数组）”。如果要在本地开发 RAG，这就是基础。
4. **Fine-tuning (微调 / SFT)**：买来一个聪明的大学生（预训练基座模型），给他疯狂做某一领域的五年高考三年模拟（专业语料喂养），把他变成专科医生。
5. **Hallucination (幻觉)**：大模型在一本正经地胡说八道。（比如你问林黛玉为什么要倒拔垂杨柳，它不仅不指正，还会给你编一个可歌可泣的故事）。

---

> 🚀 **下一步学习推荐**
> 掌握了黑话，是时候动手实操了！请前往 [实战篇-开发我的第一个-MCP-Server.md](./实战篇-开发我的第一个-MCP-Server.md) 感受大河的澎湃。
