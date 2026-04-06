# Day 19 — 完整自动化平台

> 目标：整合爬虫 + AI 提炼 + 数据库 + Next.js 前端，打通"抓取 → 处理 → 展示"完整链路。

---

## 一、整体架构

```
[定时触发 / 手动触发]
        ↓
  Python 爬虫（Day 08-11）
        ↓
  原始数据 → SQLite（raw_data）
        ↓
  AI 提炼（Day 18）
        ↓
  结构化数据 → SQLite（processed_data）
        ↓
  FastAPI 提供 API
        ↓
  Next.js 前端展示
        ↓
     用户浏览
```

---

## 二、统一数据库设计

新建 `day19/backend/schema.sql`：

```sql
-- 爬取的原始数据
CREATE TABLE IF NOT EXISTS raw_items (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    source_url  TEXT NOT NULL,
    raw_text    TEXT NOT NULL,
    status      TEXT DEFAULT 'pending',   -- pending/processed/failed
    scraped_at  TEXT DEFAULT (datetime('now','localtime'))
);

-- AI 提炼后的结构化数据
CREATE TABLE IF NOT EXISTS items (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    raw_id      INTEGER REFERENCES raw_items(id),
    title       TEXT,
    author      TEXT,
    category    TEXT,
    price       REAL,
    tags        TEXT,   -- JSON 数组字符串
    sentiment   TEXT,
    summary     TEXT,
    processed_at TEXT DEFAULT (datetime('now','localtime'))
);

-- 任务运行日志
CREATE TABLE IF NOT EXISTS run_logs (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    task        TEXT,   -- scrape/process/all
    status      TEXT,   -- running/done/failed
    scraped     INTEGER DEFAULT 0,
    processed   INTEGER DEFAULT 0,
    run_at      TEXT DEFAULT (datetime('now','localtime')),
    finished_at TEXT
);
```

---

## 三、Pipeline 脚本

新建 `day19/backend/pipeline.py`：

```python
"""
完整数据管道：抓取 → AI 提炼 → 入库
可单独运行各阶段，也可全流程运行
"""

import json
import time
import sqlite3
import logging
import httpx
import anthropic
from bs4 import BeautifulSoup
from pathlib import Path
from contextlib import contextmanager

logging.basicConfig(level=logging.INFO,
                    format="%(asctime)s [%(levelname)s] %(message)s",
                    datefmt="%H:%M:%S")
logger = logging.getLogger(__name__)

DB_PATH = "data/platform.db"
MODEL = "claude-opus-4-6"


# ── 数据库 ────────────────────────────────────
@contextmanager
def get_db():
    Path(DB_PATH).parent.mkdir(exist_ok=True)
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row
    conn.executescript(Path("schema.sql").read_text())
    try:
        yield conn
        conn.commit()
    except:
        conn.rollback()
        raise
    finally:
        conn.close()


# ── 阶段 1：抓取 ──────────────────────────────
def run_scrape(pages: int = 2) -> int:
    """抓取名言，存入 raw_items，返回新增数量"""
    logger.info("开始抓取（%d 页）", pages)
    new_count = 0

    with get_db() as conn:
        for page in range(1, pages + 1):
            url = f"https://quotes.toscrape.com/page/{page}/"
            try:
                resp = httpx.get(url, timeout=15,
                                 headers={"User-Agent": "Mozilla/5.0"})
                resp.raise_for_status()
            except Exception as e:
                logger.error("抓取失败 page=%d: %s", page, e)
                continue

            soup = BeautifulSoup(resp.text, "lxml")
            for q in soup.select(".quote"):
                text = q.select_one(".text").get_text(strip=True).strip('""')
                author = q.select_one(".author").get_text(strip=True)
                raw_text = f'"{text}" — {author}'

                try:
                    conn.execute(
                        "INSERT INTO raw_items (source_url, raw_text) VALUES (?, ?)",
                        (url, raw_text)
                    )
                    new_count += 1
                except sqlite3.IntegrityError:
                    pass

            logger.info("  第 %d 页完成", page)
            time.sleep(1)

    logger.info("抓取完成，新增 %d 条", new_count)
    return new_count


# ── 阶段 2：AI 提炼 ───────────────────────────
SYSTEM = "数据提取专家。只返回JSON，无解释。字段缺失填null。"
PROMPT_TPL = """提取信息返回JSON：
{{"quote": "名言文本", "author": "作者", "theme": "主题词(1-3个字)", "sentiment": "positive/neutral/negative", "insight": "15字内启示"}}

文本：{text}"""


def process_one(client: anthropic.Anthropic, raw_id: int, raw_text: str) -> dict | None:
    try:
        msg = client.messages.create(
            model=MODEL, max_tokens=256,
            system=SYSTEM,
            messages=[{"role": "user", "content": PROMPT_TPL.format(text=raw_text)}]
        )
        raw = msg.content[0].text.strip().strip("```json").strip("```").strip()
        return json.loads(raw)
    except Exception as e:
        logger.warning("处理 raw_id=%d 失败: %s", raw_id, e)
        return None


def run_process(limit: int = 20) -> int:
    """对 pending 状态的原始数据做 AI 提炼，返回处理数量"""
    logger.info("开始 AI 提炼（最多 %d 条）", limit)
    client = anthropic.Anthropic()
    processed = 0

    with get_db() as conn:
        pending = conn.execute(
            "SELECT id, raw_text FROM raw_items WHERE status='pending' LIMIT ?",
            (limit,)
        ).fetchall()

        logger.info("待处理: %d 条", len(pending))

        for row in pending:
            raw_id, raw_text = row["id"], row["raw_text"]
            result = process_one(client, raw_id, raw_text)

            if result:
                conn.execute("""
                    INSERT INTO items (raw_id, title, author, category, tags, sentiment, summary)
                    VALUES (?, ?, ?, ?, ?, ?, ?)
                """, (
                    raw_id,
                    result.get("quote", "")[:200],
                    result.get("author"),
                    result.get("theme"),
                    json.dumps([result.get("theme")] if result.get("theme") else [], ensure_ascii=False),
                    result.get("sentiment"),
                    result.get("insight"),
                ))
                conn.execute("UPDATE raw_items SET status='processed' WHERE id=?", (raw_id,))
                processed += 1
            else:
                conn.execute("UPDATE raw_items SET status='failed' WHERE id=?", (raw_id,))

            time.sleep(0.3)

    logger.info("AI 提炼完成，处理 %d 条", processed)
    return processed


# ── 主入口 ────────────────────────────────────
def run_pipeline(scrape_pages: int = 2, process_limit: int = 20) -> None:
    logger.info("=== 开始完整管道 ===")
    scraped = run_scrape(scrape_pages)
    processed = run_process(process_limit)
    logger.info("=== 管道完成：抓取 %d，处理 %d ===", scraped, processed)


if __name__ == "__main__":
    import sys
    cmd = sys.argv[1] if len(sys.argv) > 1 else "all"
    if cmd == "scrape": run_scrape()
    elif cmd == "process": run_process()
    else: run_pipeline()
```

**运行：**

```bash
python3 pipeline.py           # 全流程
python3 pipeline.py scrape    # 只抓取
python3 pipeline.py process   # 只提炼
```

---

## 四、API 服务

新建 `day19/backend/api.py`：

```python
from fastapi import FastAPI, Query
from fastapi.middleware.cors import CORSMiddleware
import sqlite3, json
from contextlib import contextmanager

app = FastAPI(title="自动化平台 API")
app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_methods=["*"], allow_headers=["*"])

DB_PATH = "data/platform.db"

@contextmanager
def db():
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row
    try:
        yield conn
    finally:
        conn.close()


@app.get("/api/stats")
def stats():
    with db() as conn:
        raw = conn.execute("SELECT COUNT(*) FROM raw_items").fetchone()[0]
        processed = conn.execute("SELECT COUNT(*) FROM items").fetchone()[0]
        pending = conn.execute("SELECT COUNT(*) FROM raw_items WHERE status='pending'").fetchone()[0]
    return {"raw": raw, "processed": processed, "pending": pending}


@app.get("/api/items")
def list_items(q: str = "", page: int = 1, page_size: int = 10):
    with db() as conn:
        base = "SELECT * FROM items"
        params: list = []
        if q:
            base += " WHERE title LIKE ? OR author LIKE ? OR summary LIKE ?"
            params = [f"%{q}%"] * 3
        total = conn.execute(f"SELECT COUNT(*) FROM ({base})", params).fetchone()[0]
        rows = conn.execute(
            f"{base} ORDER BY id DESC LIMIT ? OFFSET ?",
            params + [page_size, (page - 1) * page_size]
        ).fetchall()
    return {"items": [dict(r) for r in rows], "total": total, "page": page}


@app.post("/api/run")
def trigger_pipeline():
    """触发一次完整抓取+处理"""
    import threading
    from pipeline import run_pipeline
    threading.Thread(target=run_pipeline, daemon=True).start()
    return {"message": "管道已在后台启动"}
```

```bash
pip install fastapi uvicorn anthropic httpx beautifulsoup4 lxml
uvicorn api:app --reload --port 8000
```

---

## 五、前端触发按钮

在 Next.js 前端加一个"立即抓取"按钮，触发后端管道：

```tsx
"use client"
import { useState } from 'react'

export function TriggerButton() {
  const [status, setStatus] = useState<'idle' | 'running' | 'done'>('idle')

  const handleTrigger = async () => {
    setStatus('running')
    try {
      await fetch('/api/proxy/run', { method: 'POST' })
      setStatus('done')
      setTimeout(() => setStatus('idle'), 3000)
    } catch {
      setStatus('idle')
    }
  }

  return (
    <button
      onClick={handleTrigger}
      disabled={status === 'running'}
      className={`px-4 py-2 rounded-lg text-white text-sm font-medium transition-colors
        ${status === 'running' ? 'bg-gray-400 cursor-not-allowed' :
          status === 'done' ? 'bg-green-500' : 'bg-blue-500 hover:bg-blue-600'}`}
    >
      {status === 'running' ? '⏳ 抓取中...' : status === 'done' ? '✓ 完成' : '▶ 立即抓取'}
    </button>
  )
}
```

---

## 六、今日 Checklist

- [ ] `python3 pipeline.py scrape` 抓取成功
- [ ] `python3 pipeline.py process` AI 提炼成功（需要 API Key）
- [ ] FastAPI 启动，`/api/stats` 返回正确数字
- [ ] Next.js 前端展示 processed items
- [ ] git commit：`git add . && git commit -m "day19: complete automation platform"`

> 📅 最后更新：2026-04-06
