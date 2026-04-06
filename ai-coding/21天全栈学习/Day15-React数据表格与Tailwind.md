# Day 15 — React 数据表格 & Tailwind CSS

> 目标：掌握 Tailwind CSS 基础，实现可搜索/过滤/排序的数据表格，用于展示爬取的数据。

---

## 一、安装 Tailwind CSS

```bash
cd day15   # 继续用 Vite + React + TS 项目
pnpm add -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

**`tailwind.config.js`：**

```js
export default {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"],
  theme: { extend: {} },
  plugins: [],
}
```

**`src/index.css`（替换内容）：**

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

---

## 二、Tailwind 核心思路

Tailwind 是**原子化 CSS**——每个 class 只做一件小事，组合使用：

```tsx
// 传统 CSS
<div className="card">...</div>
// .card { background: white; padding: 16px; border-radius: 8px; box-shadow: ... }

// Tailwind
<div className="bg-white p-4 rounded-lg shadow-md">...</div>
```

### 常用工具类速查

```
布局：  flex items-center justify-between gap-4
        grid grid-cols-3
        w-full h-screen max-w-4xl mx-auto p-4

背景：  bg-white bg-gray-100 bg-blue-500

文字：  text-sm text-lg text-xl font-bold text-gray-600 text-center

边框：  border border-gray-200 rounded rounded-lg rounded-full

间距：  p-4（padding all）  px-4（左右）  py-2（上下）
        m-4（margin all）   mt-2（上）   mb-4（下）

交互：  cursor-pointer hover:bg-gray-100 transition-colors
        focus:outline-none focus:ring-2 focus:ring-blue-500

响应式：sm:text-sm md:text-base lg:text-lg
```

---

## 三、实操案例：爬取数据展示表格

### 目标

展示从 API 获取的数据（模拟爬取结果），支持：搜索、按列排序、状态过滤、分页。

**`src/App.tsx`：**

```tsx
import { useState, useEffect, useMemo } from 'react'

// ── 类型定义 ─────────────────────────────────
interface ScrapedItem {
  id: number
  title: string
  author: string
  category: string
  price: number
  rating: number
  inStock: boolean
  url: string
}

type SortKey = keyof Pick<ScrapedItem, 'title' | 'price' | 'rating'>
type SortDir = 'asc' | 'desc'

// ── 模拟爬取数据（实际应从后端 API 获取）────────
const MOCK_DATA: ScrapedItem[] = Array.from({ length: 50 }, (_, i) => ({
  id: i + 1,
  title: `商品 ${i + 1} - ${["Python入门", "React实战", "TypeScript指南", "数据分析", "机器学习"][i % 5]}`,
  author: ["张三", "李四", "王五", "赵六", "钱七"][i % 5],
  category: ["图书", "电子", "服装", "食品", "工具"][i % 5],
  price: Math.round((Math.random() * 500 + 10) * 10) / 10,
  rating: Math.round((Math.random() * 2 + 3) * 10) / 10,
  inStock: Math.random() > 0.3,
  url: `https://example.com/item/${i + 1}`,
}))

// ── 子组件 ───────────────────────────────────
function SortIcon({ active, dir }: { active: boolean; dir: SortDir }) {
  return (
    <span className={`ml-1 text-xs ${active ? 'text-blue-500' : 'text-gray-400'}`}>
      {active ? (dir === 'asc' ? '↑' : '↓') : '↕'}
    </span>
  )
}

function Badge({ inStock }: { inStock: boolean }) {
  return (
    <span className={`px-2 py-0.5 rounded-full text-xs font-medium ${
      inStock ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'
    }`}>
      {inStock ? '有货' : '缺货'}
    </span>
  )
}

// ── 主组件 ───────────────────────────────────
const PAGE_SIZE = 10

export default function App() {
  const [items] = useState<ScrapedItem[]>(MOCK_DATA)
  const [search, setSearch] = useState('')
  const [category, setCategory] = useState('全部')
  const [stockFilter, setStockFilter] = useState<'all' | 'in' | 'out'>('all')
  const [sortKey, setSortKey] = useState<SortKey>('id' as SortKey)
  const [sortDir, setSortDir] = useState<SortDir>('asc')
  const [page, setPage] = useState(1)

  const categories = ['全部', ...Array.from(new Set(items.map(i => i.category)))]

  // 过滤 + 排序（useMemo 避免每次渲染重复计算）
  const filtered = useMemo(() => {
    let result = items

    if (search) {
      const q = search.toLowerCase()
      result = result.filter(i =>
        i.title.toLowerCase().includes(q) ||
        i.author.toLowerCase().includes(q)
      )
    }
    if (category !== '全部') result = result.filter(i => i.category === category)
    if (stockFilter === 'in') result = result.filter(i => i.inStock)
    if (stockFilter === 'out') result = result.filter(i => !i.inStock)

    return [...result].sort((a, b) => {
      const av = a[sortKey as keyof ScrapedItem] as number | string
      const bv = b[sortKey as keyof ScrapedItem] as number | string
      const cmp = av < bv ? -1 : av > bv ? 1 : 0
      return sortDir === 'asc' ? cmp : -cmp
    })
  }, [items, search, category, stockFilter, sortKey, sortDir])

  const totalPages = Math.ceil(filtered.length / PAGE_SIZE)
  const pageItems = filtered.slice((page - 1) * PAGE_SIZE, page * PAGE_SIZE)

  // 切换排序
  const handleSort = (key: SortKey) => {
    if (sortKey === key) setSortDir(d => d === 'asc' ? 'desc' : 'asc')
    else { setSortKey(key); setSortDir('asc') }
    setPage(1)
  }

  // 过滤条件变化时重置到第一页
  useEffect(() => setPage(1), [search, category, stockFilter])

  return (
    <div className="min-h-screen bg-gray-50">
      <div className="max-w-6xl mx-auto px-4 py-8">

        {/* 标题 */}
        <h1 className="text-2xl font-bold text-gray-800 mb-6">爬取数据看板</h1>

        {/* 过滤栏 */}
        <div className="bg-white rounded-xl shadow-sm p-4 mb-6 flex flex-wrap gap-3 items-center">
          <input
            value={search}
            onChange={e => setSearch(e.target.value)}
            placeholder="搜索标题/作者..."
            className="border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 w-52"
          />
          <select
            value={category}
            onChange={e => setCategory(e.target.value)}
            className="border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            {categories.map(c => <option key={c}>{c}</option>)}
          </select>
          <div className="flex gap-2">
            {(['all', 'in', 'out'] as const).map(v => (
              <button
                key={v}
                onClick={() => setStockFilter(v)}
                className={`px-3 py-2 rounded-lg text-sm border transition-colors ${
                  stockFilter === v
                    ? 'bg-blue-500 text-white border-blue-500'
                    : 'border-gray-300 text-gray-600 hover:bg-gray-50'
                }`}
              >
                {v === 'all' ? '全部' : v === 'in' ? '有货' : '缺货'}
              </button>
            ))}
          </div>
          <span className="text-sm text-gray-500 ml-auto">共 {filtered.length} 条</span>
        </div>

        {/* 表格 */}
        <div className="bg-white rounded-xl shadow-sm overflow-hidden">
          <table className="w-full text-sm">
            <thead className="bg-gray-50 border-b border-gray-200">
              <tr>
                <th className="text-left px-4 py-3 text-gray-600 font-medium w-8">#</th>
                <th
                  className="text-left px-4 py-3 text-gray-600 font-medium cursor-pointer hover:text-gray-800"
                  onClick={() => handleSort('title')}
                >
                  标题 <SortIcon active={sortKey === 'title'} dir={sortDir} />
                </th>
                <th className="text-left px-4 py-3 text-gray-600 font-medium">分类</th>
                <th
                  className="text-right px-4 py-3 text-gray-600 font-medium cursor-pointer hover:text-gray-800"
                  onClick={() => handleSort('price')}
                >
                  价格 <SortIcon active={sortKey === 'price'} dir={sortDir} />
                </th>
                <th
                  className="text-right px-4 py-3 text-gray-600 font-medium cursor-pointer hover:text-gray-800"
                  onClick={() => handleSort('rating')}
                >
                  评分 <SortIcon active={sortKey === 'rating'} dir={sortDir} />
                </th>
                <th className="text-center px-4 py-3 text-gray-600 font-medium">库存</th>
              </tr>
            </thead>
            <tbody className="divide-y divide-gray-100">
              {pageItems.map(item => (
                <tr key={item.id} className="hover:bg-gray-50 transition-colors">
                  <td className="px-4 py-3 text-gray-400">{item.id}</td>
                  <td className="px-4 py-3">
                    <a href={item.url} target="_blank" rel="noreferrer"
                       className="text-blue-600 hover:underline font-medium">
                      {item.title}
                    </a>
                    <p className="text-gray-400 text-xs mt-0.5">{item.author}</p>
                  </td>
                  <td className="px-4 py-3 text-gray-500">{item.category}</td>
                  <td className="px-4 py-3 text-right font-medium">¥{item.price}</td>
                  <td className="px-4 py-3 text-right">
                    <span className="text-yellow-500">★</span> {item.rating}
                  </td>
                  <td className="px-4 py-3 text-center">
                    <Badge inStock={item.inStock} />
                  </td>
                </tr>
              ))}
            </tbody>
          </table>

          {/* 分页 */}
          <div className="flex items-center justify-between px-4 py-3 border-t border-gray-200">
            <span className="text-sm text-gray-500">
              第 {(page - 1) * PAGE_SIZE + 1}–{Math.min(page * PAGE_SIZE, filtered.length)} 条，共 {filtered.length} 条
            </span>
            <div className="flex gap-1">
              <button
                onClick={() => setPage(p => Math.max(1, p - 1))}
                disabled={page === 1}
                className="px-3 py-1 rounded border border-gray-300 text-sm disabled:opacity-40 hover:bg-gray-50"
              >上一页</button>
              <span className="px-3 py-1 text-sm text-gray-600">{page} / {totalPages}</span>
              <button
                onClick={() => setPage(p => Math.min(totalPages, p + 1))}
                disabled={page === totalPages}
                className="px-3 py-1 rounded border border-gray-300 text-sm disabled:opacity-40 hover:bg-gray-50"
              >下一页</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  )
}
```

### 运行

```bash
pnpm dev
```

---

## 四、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `useMemo` | 缓存过滤+排序结果，避免重复计算 |
| Tailwind 条件类 | `` `bg-${active ? 'blue' : 'gray'}-500` `` |
| 受控组件 | `search`, `category` 等状态驱动输入/选择框 |
| 组件拆分 | `SortIcon`, `Badge` 抽出可复用小组件 |
| 分页逻辑 | `slice((page-1)*PAGE_SIZE, page*PAGE_SIZE)` |

---

## 五、今日 Checklist

- [ ] Tailwind 样式生效（能看到蓝色按钮）
- [ ] 搜索、分类过滤、库存过滤均正常
- [ ] 点击表头能排序，再点一次反向排序
- [ ] 分页按钮正常跳转
- [ ] git commit：`git add . && git commit -m "day15: data table with tailwind"`

> 📅 最后更新：2026-04-06
