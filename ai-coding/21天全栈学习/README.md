# 21天全栈学习讲义

> 每天 1 小时，理论 + 案例实操。最终目标：独立构建数据抓取、批量下载、自动化工具，具备全栈开发能力。

- [00 — 学习计划总览](./00-学习计划总览.md)

## 第一阶段：Python 核心语言（Day 1-7）— 掌握 80% Python

| 讲义 | 核心内容 | 交付物 |
|------|----------|--------|
| [Day 01 — 环境 & 基础语法](./Day01-Python环境与核心语法.md) | venv、变量、列表/字典/集合、推导式 | `json_processor.py` |
| [Day 02 — 函数 & 函数式编程](./Day02-函数模块与错误处理.md) | type hints、*args/**kwargs、lambda、map/filter、生成器(yield) | `data_pipeline.py` |
| [Day 03 — 面向对象（完整版）](./Day03-面向对象与数据类.md) | 魔术方法、上下文管理器、@classmethod、@dataclass | `data_container.py` |
| [Day 04 — 标准库精华](./Day04-包管理与HTTP基础.md) | pathlib、datetime、re 正则、collections | `log_analyzer.py` |
| [Day 05 — 错误处理 & 健壮性](./Day05-错误处理与健壮性.md) | 自定义异常、logging、防御性编程 | `robust_processor.py` |
| [Day 06 — 装饰器 & asyncio](./Day06-装饰器与异步编程.md) | 自定义装饰器、async/await、asyncio.gather | `async_fetcher.py` |
| [Day 07 — 综合实战](./Day07-综合实战.md) | click CLI、sqlite3、模块结构、Day 1-6 串联 | `url_collector/` |

## 第二阶段：Python 爬虫 & 自动化（Day 8-11）

| 讲义 | 核心内容 | 交付物 |
|------|----------|--------|
| [Day 08 — 静态页面抓取](./Day08-静态页面抓取.md) | HTML/CSS选择器、BeautifulSoup、反爬策略 | `book_scraper.py` |
| [Day 09 — 动态页面抓取](./Day09-动态页面抓取.md) | Playwright、wait_for_selector、截图调试 | `dynamic_scraper.py` |
| [Day 10 — 批量下载 & 数据存储](./Day10-批量下载与数据存储.md) | asyncio 并发下载、SQLite 增量存储、去重 | `batch_downloader.py` |
| [Day 11 — 定时自动化工具](./Day11-定时自动化工具.md) | APScheduler、click 子命令、项目打包 | `auto_scraper/` |

## 第三阶段：TypeScript / Node.js（Day 12-17）

| 讲义 | 核心内容 | 交付物 |
|------|----------|--------|
| [Day 12 — TypeScript 核心语法](./Day12-TypeScript核心语法.md) | 类型系统、interface、泛型、函数类型 | `data_transform.ts` |
| [Day 13 — 异步与 HTTP](./Day13-异步与HTTP.md) | Promise、async/await、fetch、并发控制 | `batch_fetch.ts` |
| [Day 14 — React 基础](./Day14-React基础.md) | 组件、props、useState、useEffect | `user_list/` |
| [Day 15 — React 数据表格 & Tailwind](./Day15-React数据表格与Tailwind.md) | Tailwind CSS、useMemo、分页、排序 | `data_table/` |
| [Day 16 — Next.js 入门 & API 路由](./Day16-Nextjs入门与API路由.md) | App Router、Server/Client Component、Route Handler | `scraper_dashboard/` |
| [Day 17 — 前后端联调](./Day17-前后端联调.md) | CORS、环境变量、Next.js API 代理 | `fullstack_quotes/` |

## 第四阶段：全栈整合 + AI 辅助（Day 18-21）

| 讲义 | 核心内容 | 交付物 |
|------|----------|--------|
| [Day 18 — AI 数据提炼](./Day18-AI数据提炼.md) | Anthropic SDK、system prompt、结构化 JSON 输出 | `ai_extractor.py` |
| [Day 19 — 完整自动化平台](./Day19-完整自动化平台.md) | 爬虫 + AI + FastAPI + Next.js 全链路 | `platform/` |
| [Day 20 — 部署 & 定时触发](./Day20-部署与定时触发.md) | Vercel、Railway、GitHub Actions cron | 部署 URL / workflow.yml |
| [Day 21 — 个人项目启动](./Day21-个人项目启动.md) | 技能自检、项目方向选择、CLAUDE.md 模板 | 个人项目 repo |

> 📅 最后更新：2026-04-06
