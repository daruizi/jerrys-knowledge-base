# Day 07 — 自动化 & 定时调度

> 目标：把爬虫封装成专业 CLI 工具，加上定时调度和数据库存储，完成 Python 阶段的完整自动化工具。

---

## 一、Click — 构建专业 CLI

Day 02 用 `sys.argv` 处理命令行参数，功能有限。`Click` 是构建 CLI 工具的标准库，自动生成帮助文档，参数处理更优雅。

```bash
pip install click
```

### 1.1 基本用法

```python
import click


@click.command()
@click.option("--name", "-n", default="World", help="要问候的名字")
@click.option("--count", "-c", default=1, type=int, help="重复次数")
def greet(name: str, count: int) -> None:
    """简单的问候程序"""
    for _ in range(count):
        click.echo(f"Hello, {name}!")


if __name__ == "__main__":
    greet()
```

运行：

```bash
python3 greet.py --help          # 自动生成帮助
python3 greet.py --name Jerry    # Hello, Jerry!
python3 greet.py -n Jerry -c 3   # 重复3次
```

### 1.2 子命令（多功能 CLI）

```python
import click


@click.group()
def cli():
    """自动化工具集"""
    pass


@cli.command()
@click.option("--pages", default=2, help="抓取页数")
@click.option("--output", default="data/output.csv", help="输出文件路径")
def scrape(pages: int, output: str) -> None:
    """抓取数据"""
    click.echo(f"开始抓取 {pages} 页，保存到 {output}")


@cli.command()
@click.argument("filepath")
def analyze(filepath: str) -> None:
    """分析已抓取的数据"""
    click.echo(f"分析文件: {filepath}")


if __name__ == "__main__":
    cli()
```

运行：

```bash
python3 tool.py --help
python3 tool.py scrape --pages 5
python3 tool.py analyze data/output.csv
```

---

## 二、APScheduler — 定时任务

把爬虫定时运行，实现"每天自动抓取"。

```bash
pip install apscheduler
```

### 2.1 一次性定时

```python
from apscheduler.schedulers.blocking import BlockingScheduler
from datetime import datetime


def my_job():
    print(f"[{datetime.now():%H:%M:%S}] 任务执行中...")


scheduler = BlockingScheduler()

# 每 10 秒执行一次
scheduler.add_job(my_job, "interval", seconds=10)

# 每天早上 8:00 执行
scheduler.add_job(my_job, "cron", hour=8, minute=0)

# 每周一早上 9:00 执行
scheduler.add_job(my_job, "cron", day_of_week="mon", hour=9)

print("调度器启动，Ctrl+C 退出")
scheduler.start()
```

### 2.2 常用触发器

| 触发器 | 用途 | 示例 |
|--------|------|------|
| `interval` | 固定间隔 | `seconds=30`, `hours=1` |
| `cron` | 类似 Linux crontab | `hour=8, minute=0` |
| `date` | 一次性在指定时间 | `run_date="2026-04-07 10:00:00"` |

---

## 三、SQLite 存储爬取结果

用数据库代替 CSV，支持查询、去重、增量更新。

```python
import sqlite3
from pathlib import Path


def init_db(db_path: str = "scraper.db") -> sqlite3.Connection:
    conn = sqlite3.connect(db_path)
    conn.execute("""
        CREATE TABLE IF NOT EXISTS books (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            author TEXT,
            rating REAL,
            url TEXT UNIQUE,       -- UNIQUE 防止重复插入
            scraped_at TEXT DEFAULT (datetime('now', 'localtime'))
        )
    """)
    conn.commit()
    return conn


def insert_book(conn: sqlite3.Connection, book: dict) -> bool:
    """插入一条记录，若 url 已存在则跳过，返回是否新增"""
    try:
        conn.execute(
            "INSERT INTO books (title, author, rating, url) VALUES (?, ?, ?, ?)",
            (book["title"], book["author"], book["rating"], book["url"])
        )
        conn.commit()
        return True
    except sqlite3.IntegrityError:
        # UNIQUE 约束冲突，说明已存在
        return False


def query_books(conn: sqlite3.Connection, min_rating: float = 0) -> list[dict]:
    cursor = conn.execute(
        "SELECT title, author, rating, scraped_at FROM books WHERE rating >= ? ORDER BY rating DESC",
        (min_rating,)
    )
    columns = [desc[0] for desc in cursor.description]
    return [dict(zip(columns, row)) for row in cursor.fetchall()]
```

---

## 四、实操案例：完整自动化爬虫工具

### 目标

把前几天的爬虫整合成一个完整的 CLI 工具，支持：
- `scrape` — 立即抓取并存入数据库
- `list` — 查询数据库中的数据
- `schedule` — 定时自动运行

### 项目结构

```
day07/auto_scraper/
├── main.py          # CLI 入口
├── scraper.py       # 爬虫逻辑
├── database.py      # 数据库操作
└── scheduler.py     # 定时调度
```

**`database.py`**：

```python
import sqlite3


def init_db(db_path: str = "scraper.db") -> sqlite3.Connection:
    conn = sqlite3.connect(db_path)
    conn.execute("""
        CREATE TABLE IF NOT EXISTS quotes (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            text TEXT NOT NULL,
            author TEXT NOT NULL,
            tags TEXT,
            url TEXT,
            scraped_at TEXT DEFAULT (datetime('now', 'localtime'))
        )
    """)
    conn.commit()
    return conn


def insert_quote(conn: sqlite3.Connection, quote: dict) -> bool:
    try:
        conn.execute(
            "INSERT INTO quotes (text, author, tags) VALUES (?, ?, ?)",
            (quote["text"], quote["author"], ",".join(quote.get("tags", [])))
        )
        conn.commit()
        return True
    except sqlite3.Error:
        return False


def count_quotes(conn: sqlite3.Connection) -> int:
    return conn.execute("SELECT COUNT(*) FROM quotes").fetchone()[0]


def query_quotes(conn: sqlite3.Connection, author: str | None = None, limit: int = 10) -> list[dict]:
    if author:
        cursor = conn.execute(
            "SELECT text, author, tags, scraped_at FROM quotes WHERE author LIKE ? LIMIT ?",
            (f"%{author}%", limit)
        )
    else:
        cursor = conn.execute(
            "SELECT text, author, tags, scraped_at FROM quotes ORDER BY id DESC LIMIT ?",
            (limit,)
        )
    cols = [d[0] for d in cursor.description]
    return [dict(zip(cols, row)) for row in cursor.fetchall()]
```

**`scraper.py`**：

```python
import httpx
from bs4 import BeautifulSoup


HEADERS = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
}


def scrape_quotes(pages: int = 2) -> list[dict]:
    """抓取 quotes.toscrape.com（静态版，无需 Playwright）"""
    all_quotes = []

    for page in range(1, pages + 1):
        url = f"https://quotes.toscrape.com/page/{page}/"
        try:
            response = httpx.get(url, headers=HEADERS, timeout=15)
            response.raise_for_status()
        except httpx.HTTPError as e:
            print(f"第 {page} 页请求失败: {e}")
            continue

        soup = BeautifulSoup(response.text, "lxml")
        for q in soup.select(".quote"):
            text = q.select_one(".text").get_text(strip=True).strip('\u201c\u201d"')
            author = q.select_one(".author").get_text(strip=True)
            tags = [t.get_text(strip=True) for t in q.select(".tag")]
            all_quotes.append({"text": text, "author": author, "tags": tags})

    return all_quotes
```

**`scheduler.py`**：

```python
from apscheduler.schedulers.blocking import BlockingScheduler
from datetime import datetime


def run_scheduled_scrape(pages: int, db_path: str) -> None:
    """定时任务回调函数"""
    from scraper import scrape_quotes
    from database import init_db, insert_quote, count_quotes

    print(f"\n[{datetime.now():%Y-%m-%d %H:%M:%S}] 定时任务触发")
    conn = init_db(db_path)
    before = count_quotes(conn)

    quotes = scrape_quotes(pages)
    new_count = sum(1 for q in quotes if insert_quote(conn, q))
    conn.close()

    print(f"  抓取 {len(quotes)} 条，新增 {new_count} 条（库中共 {before + new_count} 条）")


def start_scheduler(interval_minutes: int, pages: int, db_path: str) -> None:
    scheduler = BlockingScheduler()
    scheduler.add_job(
        run_scheduled_scrape,
        "interval",
        minutes=interval_minutes,
        args=[pages, db_path],
    )
    print(f"定时任务已启动，每 {interval_minutes} 分钟执行一次，Ctrl+C 退出")
    # 立即执行一次
    run_scheduled_scrape(pages, db_path)
    scheduler.start()
```

**`main.py`** — CLI 入口：

```python
import click
from database import init_db, insert_quote, count_quotes, query_quotes
from scraper import scrape_quotes


@click.group()
def cli():
    """自动化数据抓取工具"""
    pass


@cli.command()
@click.option("--pages", "-p", default=2, show_default=True, help="抓取页数")
@click.option("--db", default="scraper.db", show_default=True, help="数据库路径")
def scrape(pages: int, db: str) -> None:
    """立即抓取数据并存入数据库"""
    click.echo(f"开始抓取 {pages} 页...")

    conn = init_db(db)
    before = count_quotes(conn)
    quotes = scrape_quotes(pages)

    new_count = sum(1 for q in quotes if insert_quote(conn, q))
    conn.close()

    click.echo(f"完成：抓取 {len(quotes)} 条，新增 {new_count} 条，跳过重复 {len(quotes) - new_count} 条")
    click.echo(f"数据库共 {before + new_count} 条记录")


@cli.command("list")
@click.option("--author", "-a", default=None, help="按作者过滤")
@click.option("--limit", "-l", default=10, show_default=True, help="显示条数")
@click.option("--db", default="scraper.db", show_default=True, help="数据库路径")
def list_quotes(author: str | None, limit: int, db: str) -> None:
    """查询数据库中的数据"""
    conn = init_db(db)
    quotes = query_quotes(conn, author=author, limit=limit)
    conn.close()

    if not quotes:
        click.echo("没有找到数据")
        return

    click.echo(f"\n共找到 {len(quotes)} 条（最多显示 {limit} 条）\n")
    for q in quotes:
        click.echo(f"  [{q['author']}]")
        click.echo(f"  {q['text'][:80]}{'...' if len(q['text']) > 80 else ''}")
        click.echo(f"  标签: {q['tags']}  抓取时间: {q['scraped_at']}")
        click.echo()


@cli.command()
@click.option("--interval", "-i", default=60, show_default=True, help="间隔分钟数")
@click.option("--pages", "-p", default=2, show_default=True, help="每次抓取页数")
@click.option("--db", default="scraper.db", show_default=True, help="数据库路径")
def schedule(interval: int, pages: int, db: str) -> None:
    """启动定时抓取任务"""
    from scheduler import start_scheduler
    start_scheduler(interval_minutes=interval, pages=pages, db_path=db)


if __name__ == "__main__":
    cli()
```

### 运行

```bash
cd day07/auto_scraper
pip install click apscheduler httpx beautifulsoup4 lxml

# 查看帮助
python3 main.py --help

# 立即抓取
python3 main.py scrape --pages 3

# 查询数据
python3 main.py list --limit 5
python3 main.py list --author "Einstein"

# 启动定时任务（每 5 分钟一次，测试用）
python3 main.py schedule --interval 5 --pages 1
```

---

## 五、Python 阶段知识地图（更新版）

```
Week 1 Python 基础 + 数据抓取
│
├── Day 01  基础语法（变量、列表、字典、推导式）
├── Day 02  函数、模块、异常处理
├── Day 03  OOP、dataclass、数据持久化
├── Day 04  HTTP 协议、httpx、调用 API       → 抓取的基础
├── Day 05  BeautifulSoup、CSS 选择器        → 静态页面抓取
├── Day 06  Playwright、asyncio              → 动态页面 + 批量下载
└── Day 07  Click CLI、APScheduler、SQLite   → 自动化 + 定时运行
```

学完 Day 07，你已经能独立完成：
- 抓取静态/动态网页数据
- 批量并发下载文件
- 封装成 CLI 工具
- 定时自动运行并存数据库

---

## 六、今日 Checklist

- [ ] `python3 main.py --help` 显示正确帮助文档
- [ ] `scrape` 命令成功抓取并存入数据库
- [ ] `list` 命令能查询并格式化输出
- [ ] `schedule` 命令启动后按间隔自动执行
- [ ] 理解 `UNIQUE` 约束如何实现增量更新
- [ ] git commit：`git add . && git commit -m "day07: automated scraper cli with scheduler"`

---

## 七、进入 Day 08 前的自检

能回答以下问题，说明 Python 阶段掌握扎实：

1. `httpx` 和 `Playwright` 分别用于什么场景？
2. `asyncio.Semaphore` 的作用是什么？
3. 为什么数据库的 URL 字段要设 `UNIQUE` 约束？
4. `@click.command()` 和 `@click.group()` 的区别？
5. `APScheduler` 的 `interval` 和 `cron` 触发器分别适合什么需求？

> 📅 最后更新：2026-04-06
