# Day 16 — Next.js 入门 & API 路由

> 目标：理解 Next.js 的文件路由和服务端渲染优势，实现前后端一体的数据展示应用。

---

## 一、为什么用 Next.js

| | Vite + React | Next.js |
|--|--|--|
| 渲染 | 纯客户端（CSR） | 支持 SSR/SSG/CSR 混合 |
| API | 需要单独后端 | 内置 Route Handlers |
| SEO | 差（内容靠 JS 生成） | 好（服务端返回 HTML） |
| 部署 | 静态文件 | 需要 Node.js 环境或 Vercel |
| 适用场景 | 纯前端工具/管理后台 | 需要 SEO 的网站/全栈应用 |

**我们的场景**：爬取数据展示平台，需要 API 读取数据库 → 选 Next.js。

---

## 二、搭建项目

```bash
npx create-next-app@latest day16 --typescript --tailwind --eslint --app --src-dir
cd day16
npm run dev   # http://localhost:3000
```

---

## 三、App Router 文件结构

```
src/app/
├── layout.tsx          ← 根布局（所有页面共用的 HTML 骨架）
├── page.tsx            ← 首页 /
├── globals.css
├── dashboard/
│   └── page.tsx        ← /dashboard
├── items/
│   ├── page.tsx        ← /items
│   └── [id]/
│       └── page.tsx    ← /items/123（动态路由）
└── api/
    └── items/
        └── route.ts    ← API 路由 GET/POST /api/items
```

**规则：**
- 每个目录下的 `page.tsx` 就是该路径的页面
- `[param]` 目录是动态路由
- `api/*/route.ts` 是 API 端点

---

## 四、Server Components vs Client Components

Next.js App Router 默认所有组件是 **Server Components**（在服务器运行）：

```tsx
// Server Component（默认）：在服务器执行，可以直接 await fetch/数据库
// 不能用 useState、useEffect、事件处理、浏览器 API

export default async function ItemsPage() {
  // 直接在组件里 fetch！（服务器端执行）
  const items = await fetch('https://api.example.com/items').then(r => r.json())
  return <ul>{items.map(i => <li key={i.id}>{i.name}</li>)}</ul>
}
```

```tsx
// Client Component：加 "use client" 指令
// 可以用 useState、useEffect、事件处理

"use client"
import { useState } from 'react'

export default function SearchBox() {
  const [query, setQuery] = useState('')
  return <input value={query} onChange={e => setQuery(e.target.value)} />
}
```

**最佳实践：** 尽量用 Server Components 获取数据，只在需要交互的小组件上用 `"use client"`。

---

## 五、Route Handlers（API 路由）

在 `src/app/api/` 下创建 `route.ts`，处理 HTTP 请求：

```typescript
// src/app/api/items/route.ts
import { NextRequest, NextResponse } from 'next/server'

// 模拟数据库
const items = [
  { id: 1, title: "三体", author: "刘慈欣", price: 59 },
  { id: 2, title: "活着", author: "余华", price: 39 },
]

// GET /api/items
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url)
  const q = searchParams.get('q')

  const result = q
    ? items.filter(i => i.title.includes(q) || i.author.includes(q))
    : items

  return NextResponse.json({ data: result, total: result.length })
}

// POST /api/items
export async function POST(request: NextRequest) {
  const body = await request.json()

  if (!body.title) {
    return NextResponse.json({ error: 'title 必填' }, { status: 400 })
  }

  const newItem = { id: items.length + 1, ...body }
  items.push(newItem)
  return NextResponse.json(newItem, { status: 201 })
}
```

```typescript
// 动态路由 GET /api/items/[id]
// 文件：src/app/api/items/[id]/route.ts

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const item = items.find(i => i.id === parseInt(params.id))
  if (!item) return NextResponse.json({ error: 'Not found' }, { status: 404 })
  return NextResponse.json(item)
}
```

---

## 六、实操案例：爬取数据展示平台

### 目标

用 Next.js 构建：
- `/` 首页：统计概览
- `/items` 页：数据列表（Server Component 获取数据）
- `/api/items` API：返回爬取的数据

**`src/app/api/items/route.ts`**（模拟爬取数据）：

```typescript
import { NextRequest, NextResponse } from 'next/server'

// 模拟爬取的书籍数据
const scraped = Array.from({ length: 30 }, (_, i) => ({
  id: i + 1,
  title: `书名 ${i + 1}`,
  author: ["张三", "李四", "王五"][i % 3],
  price: Math.round(Math.random() * 200 + 20),
  rating: parseFloat((Math.random() * 2 + 3).toFixed(1)),
  category: ["技术", "文学", "历史"][i % 3],
  scrapedAt: new Date(Date.now() - Math.random() * 7 * 86400000).toISOString(),
}))

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url)
  const q = searchParams.get('q') ?? ''
  const category = searchParams.get('category') ?? ''
  const page = parseInt(searchParams.get('page') ?? '1')
  const pageSize = 10

  let result = scraped
  if (q) result = result.filter(i => i.title.includes(q) || i.author.includes(q))
  if (category) result = result.filter(i => i.category === category)

  const total = result.length
  const items = result.slice((page - 1) * pageSize, page * pageSize)

  return NextResponse.json({ items, total, page, pageSize })
}
```

**`src/app/page.tsx`**（首页，Server Component）：

```tsx
async function getStats() {
  const res = await fetch('http://localhost:3000/api/items', { cache: 'no-store' })
  const data = await res.json()
  return data
}

export default async function Home() {
  const { total } = await getStats()

  return (
    <main className="max-w-4xl mx-auto px-4 py-12">
      <h1 className="text-3xl font-bold text-gray-800 mb-2">数据抓取平台</h1>
      <p className="text-gray-500 mb-8">自动化数据收集与展示</p>

      <div className="grid grid-cols-3 gap-6 mb-8">
        {[
          { label: "已抓取数据", value: total, color: "blue" },
          { label: "今日新增", value: 12, color: "green" },
          { label: "失败任务", value: 0, color: "red" },
        ].map(card => (
          <div key={card.label} className="bg-white rounded-xl shadow-sm p-6 border border-gray-100">
            <p className="text-gray-500 text-sm">{card.label}</p>
            <p className={`text-3xl font-bold text-${card.color}-600 mt-1`}>{card.value}</p>
          </div>
        ))}
      </div>

      <a href="/items"
         className="inline-block bg-blue-500 text-white px-6 py-3 rounded-lg hover:bg-blue-600 transition-colors">
        查看全部数据 →
      </a>
    </main>
  )
}
```

**`src/app/items/page.tsx`**（列表页，混合组件）：

```tsx
import ItemsTable from './ItemsTable'

export default function ItemsPage() {
  return (
    <main className="max-w-6xl mx-auto px-4 py-8">
      <div className="flex items-center justify-between mb-6">
        <h1 className="text-2xl font-bold text-gray-800">数据列表</h1>
        <a href="/" className="text-blue-500 hover:underline text-sm">← 返回首页</a>
      </div>
      <ItemsTable />
    </main>
  )
}
```

**`src/app/items/ItemsTable.tsx`**（Client Component，处理搜索交互）：

```tsx
"use client"
import { useState, useEffect } from 'react'

interface Item {
  id: number; title: string; author: string
  price: number; rating: number; category: string; scrapedAt: string
}

export default function ItemsTable() {
  const [items, setItems] = useState<Item[]>([])
  const [total, setTotal] = useState(0)
  const [search, setSearch] = useState('')
  const [page, setPage] = useState(1)
  const [loading, setLoading] = useState(false)

  useEffect(() => {
    setLoading(true)
    const params = new URLSearchParams({ q: search, page: String(page) })
    fetch(`/api/items?${params}`)
      .then(r => r.json())
      .then(data => { setItems(data.items); setTotal(data.total) })
      .finally(() => setLoading(false))
  }, [search, page])

  useEffect(() => setPage(1), [search])

  const totalPages = Math.ceil(total / 10)

  return (
    <div className="bg-white rounded-xl shadow-sm overflow-hidden">
      <div className="p-4 border-b border-gray-100">
        <input
          value={search} onChange={e => setSearch(e.target.value)}
          placeholder="搜索标题或作者..."
          className="border border-gray-300 rounded-lg px-3 py-2 text-sm w-64 focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <span className="ml-3 text-sm text-gray-500">共 {total} 条</span>
      </div>

      {loading ? (
        <div className="p-8 text-center text-gray-400">加载中...</div>
      ) : (
        <table className="w-full text-sm">
          <thead className="bg-gray-50">
            <tr>{["#","标题","作者","分类","价格","评分","抓取时间"].map(h => (
              <th key={h} className="px-4 py-3 text-left text-gray-600 font-medium">{h}</th>
            ))}</tr>
          </thead>
          <tbody className="divide-y divide-gray-100">
            {items.map(item => (
              <tr key={item.id} className="hover:bg-gray-50">
                <td className="px-4 py-3 text-gray-400">{item.id}</td>
                <td className="px-4 py-3 font-medium">{item.title}</td>
                <td className="px-4 py-3 text-gray-500">{item.author}</td>
                <td className="px-4 py-3"><span className="bg-blue-50 text-blue-600 px-2 py-0.5 rounded text-xs">{item.category}</span></td>
                <td className="px-4 py-3">¥{item.price}</td>
                <td className="px-4 py-3">⭐ {item.rating}</td>
                <td className="px-4 py-3 text-gray-400 text-xs">{new Date(item.scrapedAt).toLocaleDateString()}</td>
              </tr>
            ))}
          </tbody>
        </table>
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

### 运行

```bash
npm run dev
# 访问 http://localhost:3000
# 访问 http://localhost:3000/items
# 访问 http://localhost:3000/api/items
```

---

## 七、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| Server Component | `Home` 和 `ItemsPage` 直接在服务器 fetch |
| Client Component | `ItemsTable` 加 `"use client"` 处理搜索交互 |
| Route Handler | `/api/items/route.ts` 处理 GET 请求和查询参数 |
| 文件路由 | `app/items/page.tsx` → `/items` 页面 |
| `cache: 'no-store'` | 禁用缓存，每次请求都获取最新数据 |

---

## 八、今日 Checklist

- [ ] 首页 `/` 显示统计卡片
- [ ] `/items` 页面显示数据列表
- [ ] `/api/items?q=书名1` 返回过滤结果
- [ ] 理解 Server Component 和 Client Component 的区别
- [ ] git commit：`git add . && git commit -m "day16: nextjs scraper dashboard"`

> 📅 最后更新：2026-04-06
