# 21 天 Python + TypeScript 全栈学习方案

> 适用人群：计算机科学专业背景，精通计算机和网络软硬件，前后端开发仅停留在了解阶段。  
> 目标：每天 1 小时，21 天建立全栈开发基础，以便更好地使用 AI Coding 辅助开发。  
> 学习模式：每天前 30 分钟学概念，后 30 分钟动手练。  
> 环境：macOS，已安装 Claude Code。

---

## 第一阶段：Python 后端基础（Day 1-7）

### Day 1 — Python 环境 & 核心语法
- **概念（30min）**：安装 Python 3.12+（`brew install python`），了解 `venv` 虚拟环境，学习变量、类型、f-string、列表/字典推导式
- **实操（30min）**：创建项目目录，建虚拟环境，写一个脚本处理 JSON 文件（读取、过滤、输出）
- **交付物**：`day01/json_processor.py`

### Day 2 — 函数、模块、错误处理
- **概念（30min）**：函数定义与类型标注（type hints）、`*args/**kwargs`、模块/包导入机制、`try/except` 异常处理
- **实操（30min）**：写一个 CLI 工具，接收命令行参数，读取 CSV 文件并统计数据
- **交付物**：`day02/csv_analyzer.py`

### Day 3 — 面向对象 & 数据类
- **概念（30min）**：class、`__init__`、继承、`@dataclass`、`@property`、协议（Protocol）
- **实操（30min）**：用 dataclass 建模一个简单的任务管理器（增删改查 tasks）
- **交付物**：`day03/task_manager.py`

### Day 4 — 包管理 & HTTP 基础
- **概念（30min）**：`pip`、`pyproject.toml`、`requirements.txt`；HTTP 方法、状态码、JSON API 概念
- **实操（30min）**：用 `httpx` 调用一个公开 API（如 GitHub API），获取并格式化展示数据
- **交付物**：`day04/github_fetcher.py`

### Day 5 — FastAPI 入门
- **概念（30min）**：FastAPI 基本概念、路由、请求/响应模型（Pydantic）、自动文档（Swagger UI）
- **实操（30min）**：搭建一个 TODO API（GET/POST/DELETE），用浏览器和 curl 测试
- **交付物**：`day05/todo_api/`

### Day 6 — 数据库 & ORM
- **概念（30min）**：SQLite 基础、SQLAlchemy / SQLModel ORM 概念、数据迁移
- **实操（30min）**：给 Day 5 的 TODO API 加上 SQLite 持久化存储
- **交付物**：`day06/todo_api_db/`

### Day 7 — Python 阶段回顾 & 小项目
- **概念（30min）**：回顾 Day 1-6 要点，学习 pytest 基础、项目结构最佳实践
- **实操（30min）**：给 TODO API 写 3-5 个测试用例，确保 CRUD 正常
- **交付物**：`day07/tests/`

---

## 第二阶段：TypeScript 前端基础（Day 8-14）

### Day 8 — Node.js 环境 & TypeScript 核心语法
- **概念（30min）**：安装 Node.js（`brew install node`），`npm/pnpm`，TypeScript 编译流程，基本类型（string/number/boolean/array/object）、interface、type
- **实操（30min）**：初始化 TS 项目，写脚本处理数组数据（map/filter/reduce）
- **交付物**：`day08/data_transform.ts`

### Day 9 — 函数、泛型、异步
- **概念（30min）**：箭头函数、泛型（`<T>`）、Promise、async/await、fetch API
- **实操（30min）**：用 TS 写一个异步脚本，调用 API 并用泛型定义响应类型
- **交付物**：`day09/api_client.ts`

### Day 10 — React 核心概念
- **概念（30min）**：JSX、组件（函数组件）、props、useState、useEffect、事件处理
- **实操（30min）**：用 Vite 创建 React+TS 项目（`npm create vite@latest`），做一个计数器 + 列表组件
- **交付物**：`day10/react-app/`

### Day 11 — React 状态管理 & 表单
- **概念（30min）**：受控组件、表单处理、useRef、组件间通信（props drilling vs context）
- **实操（30min）**：做一个 TODO 前端页面（添加、删除、标记完成）
- **交付物**：`day11/todo-frontend/`

### Day 12 — 样式 & 布局
- **概念（30min）**：CSS Modules / Tailwind CSS 基础、Flexbox、Grid 布局、响应式设计
- **实操（30min）**：给 TODO 前端加上 Tailwind 样式，做到美观可用
- **交付物**：`day12/todo-styled/`

### Day 13 — 前后端联调
- **概念（30min）**：CORS、fetch/axios 调用后端 API、环境变量、错误状态处理
- **实操（30min）**：把 Day 11-12 的前端连接到 Day 6 的 Python 后端 API
- **交付物**：`day13/fullstack-todo/`

### Day 14 — TypeScript 阶段回顾
- **概念（30min）**：回顾 TS 和 React 要点，学习 React Router 基础（多页面）
- **实操（30min）**：给 TODO 应用加一个"已完成"页面，用 React Router 实现导航
- **交付物**：`day14/todo-with-routing/`

---

## 第三阶段：全栈整合 & AI Coding 实战（Day 15-21）

### Day 15 — Next.js 入门
- **概念（30min）**：Next.js App Router、文件系统路由、Server Components vs Client Components
- **实操（30min）**：用 `npx create-next-app@latest` 创建项目，实现 2-3 个页面
- **交付物**：`day15/nextjs-app/`

### Day 16 — API Routes & 全栈 Next.js
- **概念（30min）**：Next.js Route Handlers（API 路由）、Server Actions、数据获取模式
- **实操（30min）**：在 Next.js 中实现一个书签收藏 API + 前端页面
- **交付物**：`day16/bookmark-app/`

### Day 17 — 数据库集成（Prisma/Drizzle）
- **概念（30min）**：TypeScript ORM（Prisma 或 Drizzle）、Schema 定义、Migration
- **实操（30min）**：给 Day 16 的书签应用加上 SQLite 数据库持久化
- **交付物**：`day17/bookmark-app-db/`

### Day 18 — AI API 集成
- **概念（30min）**：Claude API / Anthropic SDK 基础、消息格式、流式响应、Tool Use 概念
- **实操（30min）**：写一个 Python 脚本调用 Claude API，实现简单的对话功能
- **交付物**：`day18/chat_cli.py`

### Day 19 — AI 全栈应用
- **概念（30min）**：AI SDK（Vercel AI SDK）、流式 UI、聊天界面设计模式
- **实操（30min）**：在 Next.js 中构建一个简单的 AI 聊天界面，后端调用 Claude API
- **交付物**：`day19/ai-chat-app/`

### Day 20 — 部署 & DevOps 基础
- **概念（30min）**：Docker 基础、环境变量管理、Vercel/Railway 部署流程
- **实操（30min）**：把 Day 19 的项目部署到 Vercel（前端）或写一个 Dockerfile（后端）
- **交付物**：部署成功的 URL 或 `Dockerfile`

### Day 21 — 总结 & 个人项目启动
- **概念（30min）**：回顾 21 天学习路线，整理个人知识清单（知道什么、还差什么）
- **实操（30min）**：用 Claude Code 辅助启动一个你感兴趣的个人项目，体验 AI Coding 全流程
- **交付物**：个人项目初始 repo + CLAUDE.md

---

## 每日学习流程

```
[0:00-0:05]  快速回顾昨天内容（看昨天的交付物代码）
[0:05-0:30]  学习今天的概念（阅读 + Claude Code 答疑）
[0:30-0:55]  动手编码（先自己写，卡住时用 Claude Code 辅助）
[0:55-1:00]  git commit 今天的成果，写简短学习笔记
```

## 学习资源

| 资源 | 用途 |
|------|------|
| **Claude Code** | 随时提问、解释代码、辅助调试 |
| [Python 官方教程](https://docs.python.org/3/tutorial/) | Python 语法参考 |
| [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/) | TS 语法参考 |
| [React 官方文档](https://react.dev) | React 学习（新文档，质量高） |
| [FastAPI 官方文档](https://fastapi.tiangolo.com) | FastAPI 教程 |
| [Next.js 官方文档](https://nextjs.org/docs) | Next.js 学习 |

## 验证方式

- **每天**：确保交付物代码能运行，`git commit` 记录进度
- **Day 7**：Python TODO API + 测试全部通过
- **Day 14**：前端能正确连接后端，全栈 TODO 应用可用
- **Day 21**：能独立用 Claude Code 辅助搭建一个包含 AI 功能的全栈应用

## AI Coding 使用建议

- Day 1-7：尽量自己写，只在卡住时问 Claude Code「这段代码为什么报错」
- Day 8-14：可以让 Claude Code 生成脚手架代码，但要逐行理解
- Day 15-21：主动用 Claude Code 加速开发，重点学习如何写好 prompt 让 AI 生成高质量代码

> 📅 最后更新：2026-04-06
