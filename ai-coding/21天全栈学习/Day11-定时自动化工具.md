# Day 11 — 定时自动化工具

> 目标：用 APScheduler 实现定时调度，用 Click 封装成完整 CLI，完成 Python 阶段最终工具。

---

## 一、APScheduler 定时调度

```bash
pip install apscheduler
```

### 1.1 三种触发器

```python
from apscheduler.schedulers.blocking import BlockingScheduler

scheduler = BlockingScheduler()

# ① interval — 固定间隔执行
scheduler.add_job(my_func, "interval", seconds=30)
scheduler.add_job(my_func, "interval", hours=1, minutes=30)

# ② cron — 类似 Linux crontab（指定时间点）
scheduler.add_job(my_func, "cron", hour=8, minute=0)          # 每天 08:00
scheduler.add_job(my_func, "cron", day_of_week="mon-fri", hour=9)  # 工作日 09:00
scheduler.add_job(my_func, "cron", hour="*/2")                 # 每两小时

# ③ date — 一次性在指定时间执行
scheduler.add_job(my_func, "date", run_date="2026-05-01 10:00:00")

scheduler.start()   # 阻塞运行，Ctrl+C 退出
```

### 1.2 后台运行（非阻塞）

```python
from apscheduler.schedulers.background import BackgroundScheduler

scheduler = BackgroundScheduler()
scheduler.add_job(my_func, "interval", minutes=5)
scheduler.start()

# 程序继续运行其他代码，调度在后台执行
# 程序退出前记得：scheduler.shutdown()
```

### 1.3 给任务传参数

```python
def scrape_job(pages: int, db_path: str) -> None:
    print(f"抓取 {pages} 页，存入 {db_path}")

scheduler.add_job(
    scrape_job,
    "interval",
    hours=6,
    args=[3, "data/scraper.db"],        # 位置参数
    # kwargs={"pages": 3, "db_path": "data/scraper.db"}  # 关键字参数
)
```

---

## 二、Click 子命令完整模式

回顾 Day 07 的 Click，这次加上 `pass_context` 共享配置：

```python
import click

@click.group()
@click.option("--db", default="data/scraper.db", envvar="SCRAPER_DB",
              show_default=True, help="数据库路径")
@click.pass_context
def cli(ctx, db):
    """自动化爬虫工具"""
    ctx.ensure_object(dict)
    ctx.obj["db"] = db        # 所有子命令都能访问 ctx.obj["db"]

@cli.command()
@click.pass_context
def scrape(ctx):
    db = ctx.obj["db"]
    # ...

@cli.command()
@click.pass_context  
def schedule(ctx):
    db = ctx.obj["db"]
    # ...
```

---

## 三、项目结构最佳实践

```
day11/auto_scraper/
├── __init__.py
├── main.py           ← python3 -m auto_scraper 入口
├── cli.py            ← Click 命令定义
├── scraper.py        ← 爬虫逻辑（来自 Day 08）
├── storage.py        ← 数据库操作
└── scheduler.py      ← 定时调度逻辑
```

**职责分离原则**：每个文件只做一件事，互相通过函数调用。

---

## 四、实操案例：完整自动化爬虫 CLI

### 支持命令

```
python3 -m auto_scraper scrape [--pages N]      立即抓取
python3 -m auto_scraper list [--limit N]        查询数据
python3 -m auto_scraper stats                  统计信息
python3 -m auto_scraper schedule [--interval N] 定时运行
```

**`storage.py`**：

```python
import sqlite3
from contextlib import contextmanager
from pathlib import Path


@contextmanager
def get_conn(db_path: str):
    Path(db_path).parent.mkdir(parents=True, exist_ok=True)
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row
    try:
        yield conn
        conn.commit()
    except:
        conn.rollback()
        raise
    finally:
        conn.close()


def init_db(db_path: str) -> None:
    with get_conn(db_path) as conn:
        conn.execute("""
            CREATE TABLE IF NOT EXISTS quotes (
                id         INTEGER PRIMARY KEY AUTOINCREMENT,
                text       TEXT NOT NULL,
                author     TEXT NOT NULL,
                tags       TEXT,
                scraped_at TEXT DEFAULT (datetime('now','localtime')),
                UNIQUE(text, author)
            )
        """)
        conn.execute("""
            CREATE TABLE IF NOT EXISTS scrape_log (
                id         INTEGER PRIMARY KEY AUTOINCREMENT,
                run_at     TEXT DEFAULT (datetime('now','localtime')),
                pages      INTEGER,
                fetched    INTEGER,
                new_count  INTEGER
            )
        """)


def insert_quote(db_path: str, quote: dict) -> bool:
    """返回 True 表示新增，False 表示已存在"""
    with get_conn(db_path) as conn:
        try:
            conn.execute(
                "INSERT INTO quotes (text, author, tags) VALUES (?,?,?)",
                (quote["text"], quote["author"], ",".join(quote.get("tags", [])))
            )
            return True
        except sqlite3.IntegrityError:
            return False


def log_run(db_path: str, pages: int, fetched: int, new_count: int) -> None:
    with get_conn(db_path) as conn:
        conn.execute(
            "INSERT INTO scrape_log (pages, fetched, new_count) VALUES (?,?,?)",
            (pages, fetched, new_count)
        )


def query_quotes(db_path: str, author: str | None, limit: int) -> list[dict]:
    with get_conn(db_path) as conn:
        if author:
            rows = conn.execute(
                "SELECT * FROM quotes WHERE author LIKE ? ORDER BY id DESC LIMIT ?",
                (f"%{author}%", limit)
            ).fetchall()
        else:
            rows = conn.execute(
                "SELECT * FROM quotes ORDER BY id DESC LIMIT ?", (limit,)
            ).fetchall()
        return [dict(r) for r in rows]


def get_stats(db_path: str) -> dict:
    with get_conn(db_path) as conn:
        total = conn.execute("SELECT COUNT(*) FROM quotes").fetchone()[0]
        authors = conn.execute("SELECT COUNT(DISTINCT author) FROM quotes").fetchone()[0]
        last_run = conn.execute(
            "SELECT run_at, new_count FROM scrape_log ORDER BY id DESC LIMIT 1"
        ).fetchone()
        return {
            "total": total,
            "authors": authors,
            "last_run": dict(last_run) if last_run else None,
        }
```

**`scraper.py`**：

```python
import httpx
from bs4 import BeautifulSoup

HEADERS = {"User-Agent": "Mozilla/5.0 (compatible; AutoScraper/1.0)"}


def scrape_quotes(pages: int = 2) -> list[dict]:
    all_quotes = []
    for page_num in range(1, pages + 1):
        url = f"https://quotes.toscrape.com/page/{page_num}/"
        try:
            resp = httpx.get(url, headers=HEADERS, timeout=15)
            resp.raise_for_status()
        except httpx.HTTPError:
            continue

        soup = BeautifulSoup(resp.text, "lxml")
        for q in soup.select(".quote"):
            all_quotes.append({
                "text": q.select_one(".text").get_text(strip=True).strip('\u201c\u201d"'),
                "author": q.select_one(".author").get_text(strip=True),
                "tags": [t.get_text(strip=True) for t in q.select(".tag")],
            })
    return all_quotes
```

**`scheduler.py`**：

```python
import logging
from datetime import datetime
from apscheduler.schedulers.blocking import BlockingScheduler
from .scraper import scrape_quotes
from .storage import init_db, insert_quote, log_run, get_stats

logger = logging.getLogger(__name__)


def run_scrape_job(db_path: str, pages: int) -> None:
    logger.info("[%s] 定时任务触发", datetime.now().strftime("%H:%M:%S"))
    init_db(db_path)
    quotes = scrape_quotes(pages)
    new_count = sum(1 for q in quotes if insert_quote(db_path, q))
    log_run(db_path, pages, len(quotes), new_count)
    logger.info("  抓取 %d 条，新增 %d 条", len(quotes), new_count)

    stats = get_stats(db_path)
    logger.info("  数据库共 %d 条，%d 位作者", stats["total"], stats["authors"])


def start(db_path: str, pages: int, interval_minutes: int) -> None:
    scheduler = BlockingScheduler()
    scheduler.add_job(
        run_scrape_job,
        "interval",
        minutes=interval_minutes,
        args=[db_path, pages],
    )
    logger.info("定时任务启动，每 %d 分钟执行一次", interval_minutes)
    run_scrape_job(db_path, pages)   # 立即执行一次
    try:
        scheduler.start()
    except KeyboardInterrupt:
        logger.info("调度器已停止")
```

**`cli.py`**：

```python
import logging
import click
from .scraper import scrape_quotes
from .storage import init_db, insert_quote, log_run, query_quotes, get_stats

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    datefmt="%H:%M:%S",
)


@click.group()
@click.option("--db", default="data/scraper.db", envvar="SCRAPER_DB",
              show_default=True, help="数据库路径")
@click.pass_context
def cli(ctx, db: str) -> None:
    """自动化数据抓取工具"""
    ctx.ensure_object(dict)
    ctx.obj["db"] = db
    init_db(db)


@cli.command()
@click.option("--pages", "-p", default=2, show_default=True, help="抓取页数")
@click.pass_context
def scrape(ctx, pages: int) -> None:
    """立即抓取数据"""
    db = ctx.obj["db"]
    click.echo(f"开始抓取 {pages} 页...")

    quotes = scrape_quotes(pages)
    new_count = sum(1 for q in quotes if insert_quote(db, q))
    log_run(db, pages, len(quotes), new_count)

    click.echo(f"完成：抓取 {len(quotes)} 条，新增 {new_count} 条，"
               f"跳过重复 {len(quotes) - new_count} 条")


@cli.command("list")
@click.option("--limit", "-n", default=10, show_default=True)
@click.option("--author", "-a", default=None, help="按作者名过滤")
@click.pass_context
def list_quotes(ctx, limit: int, author: str | None) -> None:
    """列出数据库中的记录"""
    rows = query_quotes(ctx.obj["db"], author=author, limit=limit)
    if not rows:
        click.echo("暂无数据，请先运行 scrape 命令")
        return

    for r in rows:
        click.echo(f"\n[{r['author']}]")
        click.echo(f"  {r['text'][:80]}{'...' if len(r['text']) > 80 else ''}")
        if r["tags"]:
            click.echo(f"  标签: {r['tags']}")


@cli.command()
@click.pass_context
def stats(ctx) -> None:
    """显示统计信息"""
    s = get_stats(ctx.obj["db"])
    click.echo(f"数据库共 {s['total']} 条名言，{s['authors']} 位作者")
    if s["last_run"]:
        click.echo(f"上次运行: {s['last_run']['run_at']}，新增 {s['last_run']['new_count']} 条")


@cli.command()
@click.option("--interval", "-i", default=60, show_default=True, help="间隔分钟数")
@click.option("--pages", "-p", default=2, show_default=True)
@click.pass_context
def schedule(ctx, interval: int, pages: int) -> None:
    """启动定时抓取（Ctrl+C 停止）"""
    from .scheduler import start
    start(db_path=ctx.obj["db"], pages=pages, interval_minutes=interval)
```

**`main.py`**：

```python
from .cli import cli

if __name__ == "__main__":
    cli()
```

**`__init__.py`**（空文件）：

```
```

### 运行

```bash
cd day11
pip install httpx beautifulsoup4 lxml apscheduler click
python3 -m auto_scraper --help
python3 -m auto_scraper scrape --pages 3
python3 -m auto_scraper list --limit 5
python3 -m auto_scraper stats
python3 -m auto_scraper schedule --interval 5   # 每5分钟，Ctrl+C 停止
```

---

## 五、Python 爬虫阶段完成

```
Day 08  BeautifulSoup 静态抓取    → 能抓静态页面
Day 09  Playwright 动态抓取      → 能抓 JS 渲染页面
Day 10  asyncio 批量下载         → 能并发批量下载文件
Day 11  APScheduler + Click      → 能定时自动化 + CLI 封装
```

学完 Day 08-11，你已经能独立完成：
- 抓取任意静态/动态网页
- 批量下载文件（支持断点续传）
- 封装成专业 CLI 工具
- 定时自动运行，数据增量存库

---

## 六、今日 Checklist

- [ ] 所有 4 个命令正常运行
- [ ] `schedule` 命令启动后按时自动执行
- [ ] 运行两次 `scrape`，验证重复数据被跳过
- [ ] `stats` 显示正确统计
- [ ] git commit：`git add . && git commit -m "day11: complete auto scraper cli"`

> 📅 最后更新：2026-04-06
