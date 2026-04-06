# Day 17 — 前后端联调

> 目标：让 Next.js 前端连接 Python 爬虫后端，理解 CORS、环境变量和跨域请求，完成真正的全栈联通。

---

## 一、架构图

```
浏览器
  │  GET http://localhost:3000            (Next.js 前端)
  │  GET http://localhost:3000/api/proxy  (Next.js API 代理)
  │                  │
  │                  │  GET http://localhost:8000/items
  │                  ▼
  │           Python FastAPI              (爬虫数据后端)
  │                  │
  │                  ▼
  │              SQLite DB
  └── 或直接 GET http://localhost:8000/items (需要 CORS)
```

**两种联调方式：**
1. **Next.js API 代理**（推荐）：前端只请求 Next.js，Next.js 转发给 Python 后端，无需处理 CORS
2. **直接跨域请求**：前端直接请求 Python，需要在 Python 端配置 CORS

---

## 二、Python FastAPI 后端（读取爬虫数据库）

先搭一个简单的 FastAPI，读取 Day 11 的 SQLite 数据库：

新建 `day17/backend/main.py`：

```python
from fastapi import FastAPI, Query
from fastapi.middleware.cors import CORSMiddleware
import sqlite3
from contextlib import contextmanager
from pathlib import Path

app = FastAPI()

# CORS 配置（允许 Next.js 直接访问）
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # 生产环境换成真实域名
    allow_methods=["GET"],
    allow_headers=["*"],
)

DB_PATH = "../../day11/auto_scraper/data/scraper.db"  # 指向 Day11 的数据库


@contextmanager
def get_db():
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row
    try:
        yield conn
    finally:
        conn.close()


@app.get("/items")
def list_items(
    q: str = Query(default="", description="搜索关键词"),
    page: int = Query(default=1, ge=1),
    page_size: int = Query(default=10, ge=1, le=100),
):
    with get_db() as conn:
        base_query = "SELECT * FROM quotes"
        params = []

        if q:
            base_query += " WHERE author LIKE ? OR text LIKE ?"
            params = [f"%{q}%", f"%{q}%"]

        total = conn.execute(
            f"SELECT COUNT(*) FROM ({base_query})", params
        ).fetchone()[0]

        rows = conn.execute(
            f"{base_query} ORDER BY id DESC LIMIT ? OFFSET ?",
            params + [page_size, (page - 1) * page_size]
        ).fetchall()

    return {
        "items": [dict(r) for r in rows],
        "total": total,
        "page": page,
        "page_size": page_size,
    }


@app.get("/stats")
def get_stats():
    if not Path(DB_PATH).exists():
        return {"total": 0, "authors": 0, "last_scraped": None}

    with get_db() as conn:
        total = conn.execute("SELECT COUNT(*) FROM quotes").fetchone()[0]
        authors = conn.execute("SELECT COUNT(DISTINCT author) FROM quotes").fetchone()[0]
        last = conn.execute(
            "SELECT scraped_at FROM quotes ORDER BY id DESC LIMIT 1"
        ).fetchone()

    return {
        "total": total,
        "authors": authors,
        "last_scraped": last["scraped_at"] if last else None,
    }
```

```bash
cd day17/backend
pip install fastapi uvicorn
uvicorn main:app --reload --port 8000
# 访问 http://localhost:8000/docs 验证
```

---

## 三、环境变量

不要把后端 URL 硬编码在前端代码里，用环境变量管理：

**`.env.local`**（不提交到 git）：

```
BACKEND_URL=http://localhost:8000
NEXT_PUBLIC_APP_NAME=数据抓取平台
```

**使用规则：**
- `NEXT_PUBLIC_` 前缀：在浏览器端也可以访问
- 无前缀：只在服务器端（Server Component / API Route）可以访问
- 绝对不要在 `NEXT_PUBLIC_` 变量里放 API Token、密码等敏感信息

```typescript
// 服务器端（Server Component / Route Handler）
const backendUrl = process.env.BACKEND_URL   // 可以读取

// 浏览器端（Client Component）
const appName = process.env.NEXT_PUBLIC_APP_NAME  // 可以读取
// process.env.BACKEND_URL  // undefined！服务器专用变量在浏览器不可见
```

---

## 四、Next.js API 代理（推荐方式）

在 Next.js 的 API Route 里转发请求给 Python，前端只需要请求 `/api/*`：

**`src/app/api/proxy/items/route.ts`**：

```typescript
import { NextRequest, NextResponse } from 'next/server'

const BACKEND = process.env.BACKEND_URL ?? 'http://localhost:8000'

export async function GET(request: NextRequest) {
  // 透传查询参数
  const { searchParams } = new URL(request.url)
  const backendUrl = `${BACKEND}/items?${searchParams.toString()}`

  try {
    const resp = await fetch(backendUrl, {
      headers: { 'Accept': 'application/json' },
      cache: 'no-store',
    })

    if (!resp.ok) {
      return NextResponse.json({ error: 'Backend error' }, { status: resp.status })
    }

    const data = await resp.json()
    return NextResponse.json(data)
  } catch {
    return NextResponse.json({ error: 'Backend unavailable' }, { status: 503 })
  }
}
```

---

## 五、实操案例：完整前后端联调

### 目标

前端从真实的 Python 后端读取爬虫数据库，展示在 Next.js 页面。

**启动后端（终端 1）：**

```bash
cd day17/backend
uvicorn main:app --reload --port 8000
```

**启动前端（终端 2）：**

```bash
cd day17/frontend
npm run dev
```

**修改 `src/app/items/ItemsTable.tsx`**，改为请求真实 API：

```tsx
"use client"
import { useState, useEffect } from 'react'

interface Quote {
  id: number
  text: string
  author: string
  tags: string
  scraped_at: string
}

export default function ItemsTable() {
  const [items, setItems] = useState<Quote[]>([])
  const [total, setTotal] = useState(0)
  const [search, setSearch] = useState('')
  const [page, setPage] = useState(1)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    setLoading(true)
    setError(null)
    const params = new URLSearchParams({ q: search, page: String(page) })

    fetch(`/api/proxy/items?${params}`)
      .then(r => {
        if (!r.ok) throw new Error(`请求失败: ${r.status}`)
        return r.json()
      })
      .then(data => { setItems(data.items); setTotal(data.total) })
      .catch(err => setError(err.message))
      .finally(() => setLoading(false))
  }, [search, page])

  useEffect(() => setPage(1), [search])

  const totalPages = Math.ceil(total / 10) || 1

  return (
    <div className="bg-white rounded-xl shadow-sm overflow-hidden">
      <div className="p-4 border-b border-gray-100 flex gap-3 items-center">
        <input
          value={search} onChange={e => setSearch(e.target.value)}
          placeholder="搜索名言或作者..."
          className="border border-gray-300 rounded-lg px-3 py-2 text-sm w-64 focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <span className="text-sm text-gray-500">共 {total} 条</span>
      </div>

      {error && (
        <div className="p-4 text-red-500 text-sm">
          ⚠️ {error}（请确认 Python 后端已在 8000 端口运行）
        </div>
      )}

      {loading ? (
        <div className="p-8 text-center text-gray-400">加载中...</div>
      ) : (
        <div className="divide-y divide-gray-100">
          {items.map(item => (
            <div key={item.id} className="p-4 hover:bg-gray-50">
              <p className="text-gray-800 mb-1">"{item.text}"</p>
              <p className="text-blue-600 text-sm font-medium">— {item.author}</p>
              {item.tags && (
                <div className="flex gap-1 mt-2">
                  {item.tags.split(',').map(tag => (
                    <span key={tag} className="bg-gray-100 text-gray-500 px-2 py-0.5 rounded text-xs">
                      #{tag}
                    </span>
                  ))}
                </div>
              )}
              <p className="text-gray-300 text-xs mt-2">{item.scraped_at}</p>
            </div>
          ))}
          {items.length === 0 && !loading && (
            <p className="p-8 text-center text-gray-400">
              暂无数据。请先运行 Day 11 的爬虫抓取数据。
            </p>
          )}
        </div>
      )}

      <div className="flex justify-between items-center px-4 py-3 border-t border-gray-100">
        <span className="text-sm text-gray-500">第 {page}/{totalPages} 页</span>
        <div className="flex gap-2">
          <button onClick={() => setPage(p => Math.max(1, p-1))} disabled={page===1}
            className="px-3 py-1 border rounded text-sm disabled:opacity-40 hover:bg-gray-50">上一页</button>
          <button onClick={() => setPage(p => Math.min(totalPages, p+1))} disabled={page===totalPages}
            className="px-3 py-1 border rounded text-sm disabled:opacity-40 hover:bg-gray-50">下一页</button>
        </div>
      </div>
    </div>
  )
}
```

---

## 六、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| CORS 配置 | FastAPI `CORSMiddleware` |
| 环境变量 | `.env.local` + `process.env.BACKEND_URL` |
| API 代理 | Next.js Route Handler 转发请求，绕过浏览器跨域限制 |
| 错误状态处理 | `setError`，友好提示后端未启动 |
| `cache: 'no-store'` | 禁用 Next.js 缓存，获取实时数据 |

---

## 七、今日 Checklist

- [ ] Python 后端启动，`/docs` 页面正常
- [ ] `curl http://localhost:8000/items` 返回数据
- [ ] Next.js 前端通过代理拿到数据并展示
- [ ] 在浏览器 Network 面板观察请求链路（前端→Next.js API→Python）
- [ ] git commit：`git add . && git commit -m "day17: fullstack connect frontend to python backend"`

> 📅 最后更新：2026-04-06
