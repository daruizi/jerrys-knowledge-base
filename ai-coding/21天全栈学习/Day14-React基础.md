# Day 14 — React 基础

> 目标：理解 React 核心思想，掌握组件、props、state、副作用，做出能展示数据的页面。

---

## 一、搭建项目

```bash
pnpm create vite@latest day14 -- --template react-ts
cd day14
pnpm install
pnpm dev   # 启动开发服务器，访问 http://localhost:5173
```

---

## 二、React 核心思想

**UI = f(state)**：界面是状态的函数。状态变了，React 自动重新渲染界面。

```
state（数据）
    ↓  React 自动渲染
UI（界面）
    ↓  用户交互（点击、输入）
state 更新
    ↓  重新渲染
UI 更新
```

你只需要：**描述"状态是什么时界面长什么样"**，不需要手动操作 DOM。

---

## 三、组件（Component）

React 组件就是一个**返回 JSX 的 TypeScript 函数**：

```tsx
// JSX：在 TypeScript 里写 HTML（其实是 JavaScript）
function Greeting({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>   // {} 里可以放任意 JS 表达式
}

// 使用组件（就像 HTML 标签）
<Greeting name="Jerry" />
```

**JSX 规则：**
- 组件名首字母大写（区分 HTML 标签）
- 必须有一个根元素（或用 `<>...</>` Fragment 包裹）
- 用 `className` 代替 `class`
- 用 `{}` 插入表达式

---

## 四、Props（属性）

Props 是父组件传给子组件的数据，只读，不能修改：

```tsx
interface CardProps {
  title: string
  description: string
  score: number
  tags?: string[]       // 可选
}

function Card({ title, description, score, tags = [] }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <p>{description}</p>
      <span>评分: {score.toFixed(1)}</span>
      <div>
        {tags.map(tag => (
          <span key={tag} className="tag">{tag}</span>
        ))}
      </div>
    </div>
  )
}

// 列表渲染必须加 key（帮助 React 区分每一项）
```

---

## 五、useState — 状态管理

```tsx
import { useState } from 'react'

function Counter() {
  // [当前值, 设置函数]
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <button onClick={() => setCount(c => c - 1)}>-1</button>  {/* 函数式更新 */}
      <button onClick={() => setCount(0)}>重置</button>
    </div>
  )
}
```

**重要：`setState` 不会立即改变 `state`，而是触发下一次渲染。**

```tsx
// 对象状态：必须创建新对象（不能直接修改）
const [user, setUser] = useState({ name: "Alice", age: 25 })

// ❌ 错误：直接修改
user.age = 26

// ✓ 正确：创建新对象
setUser({ ...user, age: 26 })
```

---

## 六、useEffect — 副作用

副作用指的是与 React 渲染无关的操作：请求数据、订阅事件、修改 document.title 等。

```tsx
import { useState, useEffect } from 'react'

function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    // 每次 userId 变化时执行
    let cancelled = false   // 防止组件卸载后设置状态

    setLoading(true)
    setError(null)

    fetch(`https://jsonplaceholder.typicode.com/users/${userId}`)
      .then(r => r.json())
      .then(data => {
        if (!cancelled) {
          setUser(data)
          setLoading(false)
        }
      })
      .catch(err => {
        if (!cancelled) {
          setError(err.message)
          setLoading(false)
        }
      })

    // 清理函数：组件卸载或 userId 变化前执行
    return () => { cancelled = true }
  }, [userId])   // 依赖数组：只有 userId 变化才重新执行

  if (loading) return <p>加载中...</p>
  if (error) return <p>错误: {error}</p>
  if (!user) return null

  return <div>{user.name}</div>
}
```

**依赖数组规则：**
- `[]` — 只在组件挂载时执行一次
- `[dep1, dep2]` — dep 变化时执行
- 不传 — 每次渲染都执行（通常不想要这个）

---

## 七、事件处理

```tsx
function Form() {
  const [value, setValue] = useState("")

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value)
  }

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()   // 阻止默认表单提交（刷新页面）
    console.log("提交:", value)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={value} onChange={handleChange} placeholder="输入内容" />
      <button type="submit">提交</button>
    </form>
  )
}
```

---

## 八、实操案例：数据展示页

### 目标

从 JSONPlaceholder API 获取用户列表，展示为卡片，可点击查看详情。

修改 `src/App.tsx`：

```tsx
import { useState, useEffect } from 'react'

interface User {
  id: number
  name: string
  email: string
  phone: string
  website: string
  company: { name: string }
  address: { city: string }
}

// ── 子组件 ───────────────────────────────────
function UserCard({ user, onSelect }: { user: User; onSelect: (u: User) => void }) {
  return (
    <div
      onClick={() => onSelect(user)}
      style={{
        border: "1px solid #ddd", borderRadius: 8, padding: 16,
        cursor: "pointer", marginBottom: 12,
        transition: "box-shadow 0.2s",
      }}
      onMouseEnter={e => (e.currentTarget.style.boxShadow = "0 2px 8px rgba(0,0,0,0.15)")}
      onMouseLeave={e => (e.currentTarget.style.boxShadow = "")}
    >
      <h3 style={{ margin: "0 0 8px" }}>{user.name}</h3>
      <p style={{ margin: "4px 0", color: "#666" }}>📧 {user.email}</p>
      <p style={{ margin: "4px 0", color: "#666" }}>📍 {user.address.city}</p>
      <p style={{ margin: "4px 0", color: "#888", fontSize: 14 }}>🏢 {user.company.name}</p>
    </div>
  )
}

function UserDetail({ user, onClose }: { user: User; onClose: () => void }) {
  return (
    <div style={{
      position: "fixed", inset: 0, background: "rgba(0,0,0,0.5)",
      display: "flex", alignItems: "center", justifyContent: "center",
    }}>
      <div style={{ background: "#fff", borderRadius: 12, padding: 32, maxWidth: 400, width: "90%" }}>
        <h2>{user.name}</h2>
        <p>📧 {user.email}</p>
        <p>📞 {user.phone}</p>
        <p>🌐 {user.website}</p>
        <p>🏢 {user.company.name}</p>
        <p>📍 {user.address.city}</p>
        <button onClick={onClose} style={{ marginTop: 16, padding: "8px 24px" }}>
          关闭
        </button>
      </div>
    </div>
  )
}

// ── 主组件 ───────────────────────────────────
export default function App() {
  const [users, setUsers] = useState<User[]>([])
  const [loading, setLoading] = useState(true)
  const [search, setSearch] = useState("")
  const [selected, setSelected] = useState<User | null>(null)

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/users")
      .then(r => r.json())
      .then(data => {
        setUsers(data)
        setLoading(false)
      })
  }, [])

  const filtered = users.filter(u =>
    u.name.toLowerCase().includes(search.toLowerCase()) ||
    u.email.toLowerCase().includes(search.toLowerCase())
  )

  return (
    <div style={{ maxWidth: 600, margin: "40px auto", padding: "0 16px" }}>
      <h1>用户列表</h1>

      <input
        value={search}
        onChange={e => setSearch(e.target.value)}
        placeholder="搜索姓名或邮箱..."
        style={{ width: "100%", padding: "8px 12px", fontSize: 16, marginBottom: 20, boxSizing: "border-box" }}
      />

      {loading ? (
        <p>加载中...</p>
      ) : (
        <>
          <p style={{ color: "#888" }}>共 {filtered.length} 位用户</p>
          {filtered.map(user => (
            <UserCard key={user.id} user={user} onSelect={setSelected} />
          ))}
          {filtered.length === 0 && <p>没有匹配的用户</p>}
        </>
      )}

      {selected && <UserDetail user={selected} onClose={() => setSelected(null)} />}
    </div>
  )
}
```

### 运行

```bash
pnpm dev
```

浏览器打开 `http://localhost:5173`，可以看到用户列表、搜索框和点击详情弹窗。

---

## 九、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| 组件拆分 | `UserCard`, `UserDetail`, `App` 各司其职 |
| Props 类型 | `{ user: User; onSelect: (u: User) => void }` |
| `useState` | `users`, `loading`, `search`, `selected` |
| `useEffect` | 组件挂载时请求 API，`[]` 只执行一次 |
| 条件渲染 | `loading ? <p>加载中</p> : <列表>` |
| 列表渲染 | `users.map(u => <UserCard key={u.id} />)` |
| 事件处理 | `onChange`, `onClick` |

---

## 十、今日 Checklist

- [ ] 项目启动，页面正常显示用户列表
- [ ] 搜索框能正确过滤
- [ ] 点击卡片能打开详情弹窗
- [ ] 理解为什么列表渲染需要 `key`
- [ ] 理解 `useEffect` 依赖数组的作用
- [ ] git commit：`git add . && git commit -m "day14: react user list"`

> 📅 最后更新：2026-04-06
