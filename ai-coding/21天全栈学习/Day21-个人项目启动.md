# Day 21 — 个人项目启动

> 目标：回顾 21 天学习路线，整理技能清单，用 Claude Code 辅助启动一个真正属于你的自动化工具项目。

---

## 一、21 天技能地图

```
Python 核心（Day 1-7）
├── 语法基础：变量、列表、字典、推导式
├── 函数式：lambda、map/filter、生成器
├── OOP：class、魔术方法、dataclass、上下文管理器
├── 标准库：pathlib、datetime、re、collections、json、csv
├── 健壮性：自定义异常、logging、防御性编程
├── 异步：asyncio、async/await、httpx.AsyncClient
└── 工具：装饰器、sqlite3、click CLI

Python 爬虫（Day 8-11）
├── 静态抓取：httpx + BeautifulSoup + CSS 选择器
├── 动态抓取：Playwright 控制浏览器
├── 批量下载：asyncio 并发 + 断点续传
└── 自动化：APScheduler 定时 + Click 子命令

TypeScript（Day 12-17）
├── 核心：类型、interface、泛型、async/await
├── React：组件、props、useState、useEffect
├── Tailwind：原子化 CSS + 数据表格
├── Next.js：App Router、Server Component、Route Handler
└── 全栈：前后端联调、CORS、环境变量

全栈整合（Day 18-21）
├── Claude API：messages、system prompt、结构化输出
├── 数据管道：爬取 → AI 提炼 → 存储 → 展示
├── 部署：Vercel + Railway + GitHub Actions cron
└── 个人项目启动 ← 今天
```

---

## 二、技能自检清单

在开始个人项目前，诚实地评估自己。能独立完成打 ✓：

**Python 基础（必须全部 ✓）**
- [ ] 写一个带 type hints 的函数，接受列表返回过滤后的结果
- [ ] 用 `@dataclass` 定义数据模型，支持 `with` 语句（上下文管理器）
- [ ] 用正则表达式从字符串中提取 URL 列表
- [ ] 写一个异步函数，并发请求 5 个 URL

**爬虫（至少 2 项 ✓）**
- [ ] 给 `httpx.get` 加上请求头和重试装饰器
- [ ] 用 BeautifulSoup 提取某个网页的所有标题和链接
- [ ] 用 Playwright 截图一个需要 JS 渲染的页面
- [ ] 用 sqlite3 实现增量存储（已存在的 URL 跳过）

**TypeScript/React（至少 2 项 ✓）**
- [ ] 定义泛型接口 `ApiResponse<T>`
- [ ] 写一个有搜索过滤功能的 React 列表组件
- [ ] 在 Next.js 里创建 API Route，返回 JSON 数据

---

## 三、选择你的个人项目

根据你的实际需求选一个，下面是一些方向参考：

### 方向 A：价格监控工具
```
目标：定时抓取商品价格，价格下降时发通知
技术：httpx + BeautifulSoup + SQLite + APScheduler
扩展：接入微信推送 / 邮件通知
```

### 方向 B：RSS 聚合阅读器
```
目标：聚合多个 RSS 源，AI 摘要每篇文章，统一展示
技术：httpx + feedparser + Claude API + Next.js
扩展：收藏、标签、全文搜索
```

### 方向 C：数据看板
```
目标：定时爬取某类数据（股票/天气/GitHub 趋势），可视化展示
技术：Python 爬虫 + FastAPI + Next.js + Recharts 图表
扩展：历史对比、数据导出
```

### 方向 D：内容归档工具
```
目标：收藏网页/文章，AI 自动打标签和摘要，支持全文搜索
技术：Playwright 全页截图 + Claude API + SQLite + Next.js
扩展：浏览器插件一键收藏
```

### 方向 E：自定义爬虫平台
```
目标：在 UI 上配置爬取规则，无需改代码即可新增数据源
技术：FastAPI + SQLite + Next.js + APScheduler
扩展：支持 XPath / CSS 选择器可视化配置
```

---

## 四、用 Claude Code 启动项目

### 4.1 CLAUDE.md — 让 AI 了解你的项目

```markdown
# 项目：价格监控工具

## 目标
定时抓取指定商品页面的价格，存入数据库，价格下降时打印通知。

## 技术栈
- Python 3.12, httpx, BeautifulSoup, APScheduler, Click, SQLite

## 项目结构
- src/scraper.py  抓取逻辑
- src/storage.py  数据库操作
- src/notifier.py 通知逻辑
- src/cli.py      Click 命令

## 编码规范
- 所有函数必须有 type hints
- 使用 logging 而不是 print
- 数据库操作用 contextmanager

## 当前状态
Day 1 - 项目初始化
```

### 4.2 高效使用 Claude Code 的提问模板

**开始新功能：**
```
我要实现「用 BeautifulSoup 抓取某某网站的商品价格」
网页结构如下（粘贴 HTML 片段）
请生成抓取函数，遵循项目的编码规范
```

**调试报错：**
```
运行 python src/scraper.py 出现以下报错：
（粘贴完整错误信息）
相关代码在 src/scraper.py 第 42 行
```

**代码审查：**
```
请检查这段代码是否有问题，重点看：
1. 是否有资源泄漏
2. 错误处理是否完善
3. 是否符合项目规范
（粘贴代码）
```

### 4.3 今天要完成的实操

```bash
# 1. 创建项目目录
mkdir my-project && cd my-project
git init

# 2. 创建 CLAUDE.md（描述你的项目目标和规范）
# 3. 创建基础结构
mkdir src
touch src/__init__.py

# 4. 让 Claude Code 生成第一个功能模块
# 告诉它：项目目标 + 当前需要实现的具体功能

# 5. 运行，验证，迭代
# 6. Commit
git add . && git commit -m "init: project scaffold"
```

---

## 五、持续学习路线

21 天之后，继续深入的方向：

**爬虫进阶：**
- Scrapy 框架（大规模爬虫）
- 验证码处理（ddddocr）
- 代理池管理
- 分布式爬虫（Celery + Redis）

**Python 进阶：**
- 多进程（`multiprocessing`，CPU 密集型任务）
- 类型系统深入（Protocol、TypeVar、ParamSpec）
- 性能分析与优化（cProfile、memory_profiler）

**TypeScript/全栈进阶：**
- tRPC（类型安全的前后端通信）
- Prisma（TypeScript ORM）
- React Query（服务器状态管理）
- WebSocket（实时数据推送）

**AI 应用进阶：**
- Tool Use（让 AI 调用你的函数）
- 流式输出（stream responses）
- Embedding + 向量数据库（语义搜索）
- Agent 构建

---

## 六、今日任务

1. 完成「技能自检清单」，诚实标记
2. 选择个人项目方向，写好 CLAUDE.md
3. 创建项目仓库，完成基础脚手架
4. 用 Claude Code 辅助实现第一个核心功能
5. 推送到 GitHub

```bash
git add . && git commit -m "day21: 21天学习收官 + 个人项目启动"
```

---

## 七、祝贺你完成 21 天！

```
Day 01-07  ████████████████████  Python 核心语言 ✅
Day 08-11  ████████████████████  爬虫 & 自动化   ✅
Day 12-17  ████████████████████  TypeScript 全栈 ✅
Day 18-21  ████████████████████  AI + 部署       ✅
```

你已经能够独立构建：抓取任意网页、批量下载文件、AI 提炼数据、全栈展示、定时自动运行的完整工具。

**这不是终点，是起点。** 真正的掌握来自于用这些技能做出你自己的东西。

> 📅 最后更新：2026-04-06
