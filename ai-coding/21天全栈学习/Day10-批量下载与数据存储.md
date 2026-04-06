# Day 10 — 批量下载 & 数据存储

> 目标：用 asyncio 并发批量下载文件，掌握 SQLite 增量存储与去重，支持断点续传。

---

## 一、并发下载原理

```
串行下载 10 个文件（每个 1 秒）：
  文件1: ──下载──
  文件2:          ──下载──
  ...
  总耗时：10 秒

并发下载（同时 5 个）：
  文件1+2+3+4+5: ──下载──
  文件6+7+8+9+10:          ──下载──
  总耗时：~2 秒
```

核心工具：`asyncio` + `httpx.AsyncClient` + `asyncio.Semaphore`（Day 06 已学）

---

## 二、文件下载关键技巧

### 2.1 流式下载大文件

小文件可以一次性 `response.content`，大文件需要流式写入，避免内存溢出：

```python
async with client.stream("GET", url) as response:
    response.raise_for_status()
    with open(filepath, "wb") as f:
        async for chunk in response.aiter_bytes(chunk_size=8192):
            f.write(chunk)
```

### 2.2 从 URL 推断文件名

```python
from pathlib import Path
from urllib.parse import urlparse, unquote

def url_to_filename(url: str) -> str:
    path = urlparse(url).path
    name = unquote(Path(path).name)    # 解码 %20 等 URL 编码
    return name or "download"

print(url_to_filename("https://example.com/images/photo%201.jpg"))
# → photo 1.jpg
```

### 2.3 检查文件是否已下载（断点续传基础）

```python
def should_download(filepath: Path, expected_size: int | None = None) -> bool:
    if not filepath.exists():
        return True
    if expected_size and filepath.stat().st_size != expected_size:
        return True   # 文件不完整，重新下载
    return False
```

---

## 三、SQLite 增量存储

增量存储的核心：**第二次运行时不重复下载已有数据**。

```python
import sqlite3
from contextlib import contextmanager


@contextmanager
def get_conn(db_path: str):
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
            CREATE TABLE IF NOT EXISTS downloads (
                id          INTEGER PRIMARY KEY AUTOINCREMENT,
                url         TEXT UNIQUE NOT NULL,    -- UNIQUE 防止重复
                filename    TEXT,
                size        INTEGER,
                status      TEXT DEFAULT 'pending',  -- pending/done/failed
                downloaded_at TEXT
            )
        """)


def is_downloaded(db_path: str, url: str) -> bool:
    with get_conn(db_path) as conn:
        row = conn.execute(
            "SELECT status FROM downloads WHERE url=?", (url,)
        ).fetchone()
        return row is not None and row["status"] == "done"


def mark_done(db_path: str, url: str, filename: str, size: int) -> None:
    with get_conn(db_path) as conn:
        conn.execute("""
            INSERT INTO downloads (url, filename, size, status, downloaded_at)
            VALUES (?, ?, ?, 'done', datetime('now', 'localtime'))
            ON CONFLICT(url) DO UPDATE SET
              status='done', size=excluded.size,
              downloaded_at=excluded.downloaded_at
        """, (url, filename, size))


def mark_failed(db_path: str, url: str, error: str) -> None:
    with get_conn(db_path) as conn:
        conn.execute("""
            INSERT INTO downloads (url, status)
            VALUES (?, 'failed')
            ON CONFLICT(url) DO UPDATE SET status='failed'
        """, (url,))
```

---

## 四、实操案例：并发图片批量下载器

### 目标

- 从 URL 列表并发下载文件
- 已下载的自动跳过（断点续传）
- 结果存入 SQLite
- 显示进度

新建 `day10/batch_downloader.py`：

```python
import asyncio
import logging
from pathlib import Path
from urllib.parse import urlparse, unquote
from datetime import datetime
import httpx

from db import init_db, is_downloaded, mark_done, mark_failed

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s",
                    datefmt="%H:%M:%S")
logger = logging.getLogger(__name__)


def url_to_filename(url: str, index: int) -> str:
    path = urlparse(url).path
    name = unquote(Path(path).name)
    return name if name and "." in name else f"file_{index:04d}"


async def download_one(
    client: httpx.AsyncClient,
    sem: asyncio.Semaphore,
    url: str,
    filepath: Path,
    db_path: str,
) -> dict:
    async with sem:
        # 检查是否已下载
        if is_downloaded(db_path, url) and filepath.exists():
            logger.info("跳过（已下载）: %s", filepath.name)
            return {"url": url, "status": "skipped"}

        try:
            async with client.stream("GET", url, follow_redirects=True) as resp:
                resp.raise_for_status()
                filepath.parent.mkdir(parents=True, exist_ok=True)
                size = 0
                with open(filepath, "wb") as f:
                    async for chunk in resp.aiter_bytes(8192):
                        f.write(chunk)
                        size += len(chunk)

            mark_done(db_path, url, filepath.name, size)
            logger.info("✓ %s  (%.1f KB)", filepath.name, size / 1024)
            return {"url": url, "status": "done", "size": size}

        except Exception as e:
            mark_failed(db_path, url, str(e))
            logger.error("✗ %s  %s", url, e)
            return {"url": url, "status": "failed", "error": str(e)}


async def batch_download(
    urls: list[str],
    output_dir: str = "downloads",
    db_path: str = "data/downloads.db",
    concurrency: int = 5,
) -> list[dict]:
    init_db(db_path)
    output = Path(output_dir)
    output.mkdir(parents=True, exist_ok=True)

    sem = asyncio.Semaphore(concurrency)
    tasks = []
    async with httpx.AsyncClient(timeout=30) as client:
        for i, url in enumerate(urls):
            filename = url_to_filename(url, i)
            filepath = output / filename
            tasks.append(download_one(client, sem, url, filepath, db_path))
        results = await asyncio.gather(*tasks)

    # 统计
    done = sum(1 for r in results if r["status"] == "done")
    skipped = sum(1 for r in results if r["status"] == "skipped")
    failed = sum(1 for r in results if r["status"] == "failed")
    total_size = sum(r.get("size", 0) for r in results)

    print(f"\n下载完成：✓ {done} 个  ⏭ {skipped} 个跳过  ✗ {failed} 个失败")
    print(f"总大小: {total_size / 1024:.1f} KB")
    print(f"文件保存在: {output.absolute()}")

    return results


def main() -> None:
    # 用 Lorem Picsum 测试（每次返回不同随机图片，稳定可访问）
    urls = [f"https://picsum.photos/seed/{i}/800/600" for i in range(1, 21)]

    print(f"准备下载 {len(urls)} 张图片（并发: 5）")
    asyncio.run(batch_download(
        urls,
        output_dir="downloads/images",
        db_path="data/downloads.db",
        concurrency=5,
    ))

    print("\n--- 再次运行，验证断点续传 ---")
    asyncio.run(batch_download(
        urls[:5],   # 只请求前 5 个，应该全部跳过
        output_dir="downloads/images",
        db_path="data/downloads.db",
    ))


if __name__ == "__main__":
    main()
```

新建 `day10/db.py`（把数据库操作单独放一个模块）：

```python
import sqlite3
from contextlib import contextmanager


@contextmanager
def get_conn(db_path: str):
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
    import os
    os.makedirs(os.path.dirname(db_path) or ".", exist_ok=True)
    with get_conn(db_path) as conn:
        conn.execute("""
            CREATE TABLE IF NOT EXISTS downloads (
                id            INTEGER PRIMARY KEY AUTOINCREMENT,
                url           TEXT UNIQUE NOT NULL,
                filename      TEXT,
                size          INTEGER,
                status        TEXT DEFAULT 'pending',
                downloaded_at TEXT
            )
        """)


def is_downloaded(db_path: str, url: str) -> bool:
    with get_conn(db_path) as conn:
        row = conn.execute(
            "SELECT status FROM downloads WHERE url=?", (url,)
        ).fetchone()
        return row is not None and row["status"] == "done"


def mark_done(db_path: str, url: str, filename: str, size: int) -> None:
    with get_conn(db_path) as conn:
        conn.execute("""
            INSERT INTO downloads (url, filename, size, status, downloaded_at)
            VALUES (?, ?, ?, 'done', datetime('now', 'localtime'))
            ON CONFLICT(url) DO UPDATE SET
              status='done', size=excluded.size,
              downloaded_at=excluded.downloaded_at
        """, (url, filename, size))


def mark_failed(db_path: str, url: str, error: str = "") -> None:
    with get_conn(db_path) as conn:
        conn.execute("""
            INSERT INTO downloads (url, status) VALUES (?, 'failed')
            ON CONFLICT(url) DO UPDATE SET status='failed'
        """, (url,))
```

### 运行

```bash
cd day10
pip install httpx
python3 batch_downloader.py
```

---

## 五、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `client.stream()` | 流式下载大文件，节省内存 |
| `aiter_bytes()` | 异步逐块读取响应体 |
| `asyncio.Semaphore` | 限制并发数（Day 06 复用） |
| `ON CONFLICT ... DO UPDATE` | SQLite upsert，实现增量存储 |
| `is_downloaded()` 检查 | 断点续传：已完成的任务直接跳过 |
| `url_to_filename()` | 从 URL 解析文件名，处理 URL 编码 |

---

## 六、今日 Checklist

- [ ] 第一次运行，20 张图片全部下载
- [ ] 第二次运行，前 5 张全部显示"跳过"
- [ ] 查看 `data/downloads.db`，确认记录正确
- [ ] 理解 `ON CONFLICT DO UPDATE` 的作用
- [ ] git commit：`git add . && git commit -m "day10: async batch downloader with sqlite"`

> 📅 最后更新：2026-04-06
