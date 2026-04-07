# Day 07 — 综合实战（Python 语言收官）

> 目标：把 Day 01-06 所有知识串联成一个完整工具，掌握模块/包结构和 sqlite3，完成 Python 语言阶段学习。

---

## 一、模块与包结构

### 1.1 模块 vs 包

- **模块**：一个 `.py` 文件
- **包**：一个含有 `__init__.py` 的目录

```
url_collector/              ← 包（package）
├── __init__.py             ← 让这个目录成为包（可以是空文件）
├── main.py                 ← 程序入口
├── fetcher.py              ← 负责网络请求
├── parser.py               ← 负责解析 HTML
├── storage.py              ← 负责数据库
└── cli.py                  ← 负责命令行界面
```

### 1.2 `__init__.py` 的作用

```python
# url_collector/__init__.py
# 可以为空，也可以暴露常用接口，让导入更简洁

from .fetcher import fetch_urls
from .storage import Database

# 外部就可以用：
# from url_collector import fetch_urls, Database
```

### 1.3 相对导入 vs 绝对导入

```python
# 在包内部，用相对导入（推荐）
from .fetcher import fetch_urls    # . 表示当前包
from ..utils import helper         # .. 表示上一层包

# 在包外部（如 main.py），用绝对导入
from url_collector.fetcher import fetch_urls
```

---

## 二、pyproject.toml — 现代项目配置

Python 生态正在从 `setup.py` / `requirements.txt` 迁移到 `pyproject.toml`：

```toml
# pyproject.toml（放在项目根目录）
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "url-collector"
version = "0.1.0"
description = "Async URL collector and title extractor"
requires-python = ">=3.12"
dependencies = [
    "httpx>=0.27",
    "beautifulsoup4>=4.12",
    "lxml>=5.0",
    "click>=8.1",
]

[project.scripts]
url-collector = "url_collector.cli:cli"  # 注册为命令行工具

[project.optional-dependencies]
dev = ["pytest", "ruff"]

[tool.ruff]
line-length = 100
```

安装为可编辑包（开发时推荐）：

```bash
pip install -e .          # 从 pyproject.toml 安装
pip install -e ".[dev]"   # 含开发依赖

# 安装后可以直接用命令
url-collector --help
url-collector fetch https://example.com
```

---

## 三、sqlite3 标准库

Python 内置 `sqlite3`，无需安装，适合工具脚本的轻量存储。

### 2.1 基本操作

```python
import sqlite3
from contextlib import contextmanager


@contextmanager
def get_connection(db_path: str):
    """用上下文管理器确保连接正确关闭"""
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row   # 让查询结果支持 result["column"] 访问
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()


# 使用
with get_connection("data.db") as conn:
    # 建表
    conn.execute("""
        CREATE TABLE IF NOT EXISTS urls (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            url TEXT UNIQUE NOT NULL,
            title TEXT,
            status INTEGER,
            fetched_at TEXT DEFAULT (datetime('now', 'localtime'))
        )
    """)

    # 插入（OR IGNORE 表示 url 重复时静默跳过）
    conn.execute(
        "INSERT OR IGNORE INTO urls (url, title, status) VALUES (?, ?, ?)",
        ("https://example.com", "Example Domain", 200)
    )

    # 查询
    rows = conn.execute("SELECT * FROM urls ORDER BY id DESC LIMIT 10").fetchall()
    for row in rows:
        print(dict(row))   # Row 对象可直接转 dict
```

### 2.2 批量插入（性能更好）

```python
data = [
    ("https://a.com", "Site A", 200),
    ("https://b.com", "Site B", 200),
    ("https://c.com", None, 404),
]

with get_connection("data.db") as conn:
    conn.executemany(
        "INSERT OR IGNORE INTO urls (url, title, status) VALUES (?, ?, ?)",
        data
    )
```

---

## 三、Click 子命令（完整版）

```python
import click

@click.group()
@click.option("--db", default="data.db", envvar="APP_DB", help="数据库路径")
@click.pass_context
def cli(ctx, db: str):
    """URL 收集工具"""
    ctx.ensure_object(dict)
    ctx.obj["db"] = db   # 通过 context 传递共享配置


@cli.command()
@click.argument("urls", nargs=-1)           # 接收任意数量的位置参数
@click.option("--file", "-f", type=click.Path(exists=True), help="从文件读取 URL")
@click.pass_context
def fetch(ctx, urls, file):
    """抓取 URL 并存入数据库"""
    db = ctx.obj["db"]
    # ...


@cli.command("list")
@click.option("--limit", "-n", default=20)
@click.option("--status", type=int, help="按状态码过滤")
@click.pass_context
def list_urls(ctx, limit, status):
    """列出数据库中的 URL"""
    db = ctx.obj["db"]
    # ...
```

---

## 四、实操案例：URL 收集工具

### 功能

- `fetch <url...>` — 异步批量请求 URL，提取标题，存入数据库
- `fetch -f urls.txt` — 从文件读取 URL 列表
- `list` — 查询并展示数据库中的记录
- `stats` — 显示统计数据

### 项目结构

```
day07/url_collector/
├── __init__.py
├── main.py       ← 入口（python3 -m url_collector）
├── cli.py        ← Click 命令定义
├── fetcher.py    ← 异步 HTTP + HTML 标题提取
└── storage.py    ← SQLite 操作
```

**`storage.py`**：

```python
import sqlite3
from contextlib import contextmanager
from pathlib import Path


@contextmanager
def get_connection(db_path: str):
    Path(db_path).parent.mkdir(exist_ok=True)
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()


def init_db(db_path: str) -> None:
    with get_connection(db_path) as conn:
        conn.execute("""
            CREATE TABLE IF NOT EXISTS urls (
                id       INTEGER PRIMARY KEY AUTOINCREMENT,
                url      TEXT UNIQUE NOT NULL,
                title    TEXT,
                status   INTEGER,
                size     INTEGER,
                fetched_at TEXT DEFAULT (datetime('now', 'localtime'))
            )
        """)


def save_result(db_path: str, result: dict) -> bool:
    """保存单条结果，返回是否新增（True=新增，False=已存在更新）"""
    with get_connection(db_path) as conn:
        cursor = conn.execute(
            """INSERT INTO urls (url, title, status, size)
               VALUES (:url, :title, :status, :size)
               ON CONFLICT(url) DO UPDATE SET
                 title=excluded.title, status=excluded.status,
                 size=excluded.size, fetched_at=datetime('now','localtime')""",
            result,
        )
        return cursor.lastrowid is not None


def query_urls(db_path: str, limit: int = 20, status: int | None = None) -> list[dict]:
    with get_connection(db_path) as conn:
        if status:
            rows = conn.execute(
                "SELECT * FROM urls WHERE status=? ORDER BY fetched_at DESC LIMIT ?",
                (status, limit)
            ).fetchall()
        else:
            rows = conn.execute(
                "SELECT * FROM urls ORDER BY fetched_at DESC LIMIT ?", (limit,)
            ).fetchall()
        return [dict(r) for r in rows]


def get_stats(db_path: str) -> dict:
    with get_connection(db_path) as conn:
        total = conn.execute("SELECT COUNT(*) FROM urls").fetchone()[0]
        ok = conn.execute("SELECT COUNT(*) FROM urls WHERE status=200").fetchone()[0]
        return {"total": total, "ok": ok, "fail": total - ok}
```

**`fetcher.py`**：

```python
import asyncio
import re
import httpx
from bs4 import BeautifulSoup

HEADERS = {"User-Agent": "Mozilla/5.0 (compatible; URLCollector/1.0)"}


async def fetch_one(
    client: httpx.AsyncClient,
    sem: asyncio.Semaphore,
    url: str,
) -> dict:
    async with sem:
        try:
            resp = await client.get(url, timeout=10, follow_redirects=True)
            # 从 HTML 提取 <title>
            title = None
            if "text/html" in resp.headers.get("content-type", ""):
                soup = BeautifulSoup(resp.text, "lxml")
                title_tag = soup.find("title")
                title = title_tag.get_text(strip=True)[:200] if title_tag else None
            return {
                "url": url,
                "title": title,
                "status": resp.status_code,
                "size": len(resp.content),
            }
        except Exception as e:
            return {"url": url, "title": None, "status": 0, "size": 0, "error": str(e)}


async def fetch_all(urls: list[str], concurrency: int = 5) -> list[dict]:
    sem = asyncio.Semaphore(concurrency)
    async with httpx.AsyncClient(headers=HEADERS) as client:
        tasks = [fetch_one(client, sem, url) for url in urls]
        return await asyncio.gather(*tasks)
```

**`cli.py`**：

```python
import asyncio
import click
from pathlib import Path
from .fetcher import fetch_all
from .storage import init_db, save_result, query_urls, get_stats


@click.group()
@click.option("--db", default="data/urls.db", envvar="URL_COLLECTOR_DB",
              show_default=True, help="数据库路径")
@click.pass_context
def cli(ctx, db: str):
    """URL 收集与管理工具"""
    ctx.ensure_object(dict)
    ctx.obj["db"] = db
    init_db(db)


@cli.command()
@click.argument("urls", nargs=-1)
@click.option("--file", "-f", type=click.Path(exists=True), help="从文件读取 URL（每行一个）")
@click.option("--concurrency", "-c", default=5, show_default=True, help="并发数")
@click.pass_context
def fetch(ctx, urls, file, concurrency):
    """抓取 URL，提取标题，存入数据库"""
    db = ctx.obj["db"]

    all_urls = list(urls)
    if file:
        lines = Path(file).read_text(encoding="utf-8").splitlines()
        all_urls += [l.strip() for l in lines if l.strip() and not l.startswith("#")]

    if not all_urls:
        click.echo("请提供 URL 或用 -f 指定文件")
        return

    click.echo(f"开始抓取 {len(all_urls)} 个 URL（并发: {concurrency}）...")
    results = asyncio.run(fetch_all(all_urls, concurrency=concurrency))

    new = ok = fail = 0
    for r in results:
        if r.get("error"):
            click.echo(f"  ✗ {r['url'][:60]}  {r['error']}", err=True)
            fail += 1
        else:
            save_result(db, r)
            ok += 1
            click.echo(f"  ✓ [{r['status']}] {r['url'][:50]}  {r['title'] or '-'}")

    click.echo(f"\n完成：成功 {ok}，失败 {fail}")


@cli.command("list")
@click.option("--limit", "-n", default=20, show_default=True)
@click.option("--status", type=int, default=None, help="按状态码过滤")
@click.pass_context
def list_urls(ctx, limit, status):
    """列出数据库中的记录"""
    rows = query_urls(ctx.obj["db"], limit=limit, status=status)
    if not rows:
        click.echo("暂无数据")
        return
    click.echo(f"\n{'ID':>4}  {'状态':>4}  {'标题':<35}  URL")
    click.echo("-" * 80)
    for r in rows:
        title = (r["title"] or "-")[:34]
        url = r["url"][:45]
        click.echo(f"{r['id']:>4}  {r['status']:>4}  {title:<35}  {url}")


@cli.command()
@click.pass_context
def stats(ctx):
    """显示统计数据"""
    s = get_stats(ctx.obj["db"])
    click.echo(f"总计: {s['total']}  ✓ 成功: {s['ok']}  ✗ 失败: {s['fail']}")
```

**`main.py`**：

```python
from .cli import cli

if __name__ == "__main__":
    cli()
```

**`__init__.py`**（空文件即可）：

```python
```

### 运行

```bash
cd day07
pip install httpx beautifulsoup4 lxml click

# 作为模块运行
python3 -m url_collector --help
python3 -m url_collector fetch https://python.org https://github.com
python3 -m url_collector list
python3 -m url_collector stats

# 从文件读取
echo "https://python.org
https://fastapi.tiangolo.com
https://docs.python.org" > urls.txt
python3 -m url_collector fetch -f urls.txt
```

---

## 六、Python 80%+ 知识点完成清单

### 基础语法（Day 01）

| 知识点 | 内容 | 状态 |
|--------|------|------|
| 变量与类型 | str/int/float/bool/None，类型注解 | ✅ |
| 类型转换 | int()/str()/float()/bool()，边界情况 | ✅ |
| Truthiness | falsy 值：None/0/""/[]/{}等 | ✅ |
| 字符串方法 | split/join/strip/replace/find/startswith 等 | ✅ |
| 数字运算 | ///%/divmod/abs/round，math 模块 | ✅ |
| 列表 | 切片/解包/extend/copy/浅拷贝陷阱 | ✅ |
| 字典 | update/pop/setdefault，合并运算符 `|` | ✅ |
| 集合 | 并/交/差/对称差，frozenset | ✅ |
| 元组 | 解包，`*rest`，单元素 `(1,)`，作为 key | ✅ |
| 推导式 | 列表/字典/集合推导式，嵌套推导 | ✅ |
| 条件循环 | if/elif/else，for/while，enumerate/zip | ✅ |
| Walrus 运算符 | `:=`，在条件/循环/推导式中使用 | ✅ |
| match/case | 值匹配，解构，guard 子句，类型匹配 | ✅ |

### 函数与函数式（Day 02）

| 知识点 | 内容 | 状态 |
|--------|------|------|
| 函数定义 | 默认参数/可变默认陷阱/关键字参数 | ✅ |
| *args/**kwargs | 任意参数，解包调用，仅关键字参数 `/` `*` | ✅ |
| lambda | 匿名函数，配合 sorted/map/filter | ✅ |
| map/filter/reduce | 高阶函数，functools.reduce | ✅ |
| 生成器 | yield，惰性求值，内存对比，yield from | ✅ |
| 迭代器协议 | `__iter__`/`__next__`，StopIteration | ✅ |
| 闭包 | 自由变量，nonlocal，工厂函数 | ✅ |
| functools | partial/lru_cache/wraps/reduce | ✅ |
| itertools | chain/islice/zip_longest/groupby/product/combinations | ✅ |
| 高阶函数模式 | 函数作参数/返回值，函数组合 | ✅ |

### 面向对象（Day 03）

| 知识点 | 内容 | 状态 |
|--------|------|------|
| class/继承/super() | 实例/类变量，方法覆盖 | ✅ |
| @classmethod/@staticmethod | 工厂方法，工具方法 | ✅ |
| 多重继承/MRO | 菱形问题，`__mro__`，super()链 | ✅ |
| @property | getter/setter，计算属性 | ✅ |
| 魔术方法 | `__repr__/__str__/__eq__/__lt__/__hash__` | ✅ |
| 容器魔术方法 | `__len__/__getitem__/__setitem__/__contains__` | ✅ |
| `__call__` | 可调用对象 | ✅ |
| 运算符重载 | `__add__/__mul__/__abs__` | ✅ |
| `__slots__` | 内存优化，固定属性列表 | ✅ |
| @dataclass | 自动生成 init/repr/eq，field/asdict/frozen | ✅ |
| Enum/StrEnum | 命名常量，auto，迭代，作为 dict key | ✅ |
| ABC | abstractmethod，强制子类实现 | ✅ |
| Protocol | 结构化类型，runtime_checkable | ✅ |

### 标准库（Day 04）

| 知识点 | 内容 | 状态 |
|--------|------|------|
| pathlib | Path操作，glob，read_text/write_text | ✅ |
| datetime | now/strptime/strftime，timedelta，时区 | ✅ |
| re | search/findall/sub，分组，命名分组，flags | ✅ |
| collections | Counter/defaultdict/namedtuple/deque | ✅ |
| json/csv | 自定义序列化，DictWriter | ✅ |
| os/subprocess | environ/getenv，subprocess.run | ✅ |
| typing | Optional/Union/TypeVar/Generic/Literal/TypedDict | ✅ |

### 错误处理（Day 05）

| 知识点 | 内容 | 状态 |
|--------|------|------|
| try/except/else/finally | 完整结构，异常链 `raise from` | ✅ |
| 自定义异常 | 继承 Exception，携带结构化数据 | ✅ |
| logging | basicConfig，handler，logger.exception | ✅ |
| 防御性编程 | isinstance/前置验证/安全访问 | ✅ |
| assert | 内部不变量检查 vs 用户输入验证 | ✅ |
| warnings | warn，DeprecationWarning，filterwarnings | ✅ |
| contextlib | suppress/contextmanager/ExitStack | ✅ |
| ExceptionGroup | except*，并发任务多异常 | ✅ |

### 装饰器与异步（Day 06）

| 知识点 | 内容 | 状态 |
|--------|------|------|
| 装饰器原理 | 函数包裹函数，@wraps | ✅ |
| 带参数装饰器 | 三层嵌套工厂 | ✅ |
| 类装饰器 | `__call__` 实现，保持状态 | ✅ |
| 装饰器堆叠 | 多装饰器执行顺序 | ✅ |
| HTTP 基础 | 方法/状态码/请求结构 | ✅ |
| httpx 同步 | Client/get/post/raise_for_status | ✅ |
| async/await | 协程，asyncio.run，事件循环 | ✅ |
| asyncio.gather | 并发，return_exceptions | ✅ |
| asyncio.Semaphore | 并发数控制 | ✅ |
| asyncio.TaskGroup | Python 3.11+，任务失败自动取消 | ✅ |
| asyncio.Queue | 生产者消费者模式 | ✅ |
| httpx 异步 | AsyncClient，async with | ✅ |
| 环境变量 | dotenv，os.getenv | ✅ |

### 工程实践（Day 07）

| 知识点 | 内容 | 状态 |
|--------|------|------|
| 模块/包结构 | `__init__.py`，相对/绝对导入 | ✅ |
| pyproject.toml | 现代项目配置，脚本注册，可编辑安装 | ✅ |
| sqlite3 | CRUD，`OR IGNORE`/`ON CONFLICT`，Row 工厂 | ✅ |
| click CLI | group/command/argument/option/pass_context | ✅ |

**Python 核心已覆盖 ~87%。** 未覆盖的高级内容（元类、描述符、多进程/多线程、C 扩展）属于专项领域，日常开发很少直接用到。

---

## 七、今日 Checklist

- [ ] `python3 -m url_collector --help` 显示正确文档
- [ ] `fetch` 命令成功抓取并存库
- [ ] `list` 命令展示记录
- [ ] `stats` 命令显示统计
- [ ] 理解 `__init__.py` 的作用（删掉会怎样？）
- [ ] git commit：`git add . && git commit -m "day07: url collector cli tool"`

---

## 八、进入 Day 08 前的自检

能回答以下问题说明 Python 阶段掌握扎实：

1. `@wraps(func)` 不写会有什么问题？
2. `asyncio.gather` 和串行 `await` 的区别？
3. `sqlite3.Row` 和普通元组查询结果有什么区别？
4. `yield from` 和 `yield` 的区别？
5. 自定义异常继承 `Exception` 而不是 `BaseException` 的原因？

> 📅 最后更新：2026-04-07
