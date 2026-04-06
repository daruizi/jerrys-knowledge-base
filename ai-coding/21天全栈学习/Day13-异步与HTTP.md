# Day 13 — 异步 & HTTP

> 目标：掌握 TypeScript 中 Promise 和 async/await，用泛型定义 API 响应类型，完成异步批量请求脚本。

---

## 一、Promise

Promise 代表一个**将来会有结果的操作**，有三种状态：pending（等待）→ fulfilled（成功）/ rejected（失败）。

```typescript
// 创建 Promise
const p = new Promise<number>((resolve, reject) => {
  setTimeout(() => {
    if (Math.random() > 0.5) resolve(42)
    else reject(new Error("随机失败"))
  }, 1000)
})

// 消费 Promise（链式调用）
p
  .then(value => console.log("成功:", value))
  .catch(err => console.error("失败:", err.message))
  .finally(() => console.log("完成"))
```

### 并发执行

```typescript
const p1 = fetch("https://api.example.com/a")
const p2 = fetch("https://api.example.com/b")
const p3 = fetch("https://api.example.com/c")

// 全部成功才返回，任一失败立即 reject
const [r1, r2, r3] = await Promise.all([p1, p2, p3])

// 全部完成（无论成功失败），适合需要知道每个结果的场景
const results = await Promise.allSettled([p1, p2, p3])
results.forEach(r => {
  if (r.status === "fulfilled") console.log("✓", r.value)
  else console.log("✗", r.reason)
})
```

---

## 二、async/await

`async/await` 是 Promise 的语法糖，让异步代码看起来像同步：

```typescript
// Promise 链式写法
function fetchUser(id: number) {
  return fetch(`/api/users/${id}`)
    .then(r => r.json())
    .then(data => data as User)
    .catch(err => { throw new Error(`获取用户失败: ${err}`) })
}

// async/await 写法（更清晰）
async function fetchUser(id: number): Promise<User> {
  try {
    const response = await fetch(`/api/users/${id}`)
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`)
    }
    return response.json() as Promise<User>
  } catch (err) {
    throw new Error(`获取用户失败: ${err}`)
  }
}
```

**重要规则：**
- `await` 只能在 `async` 函数内使用
- `async` 函数**总是**返回 `Promise`，即使你写 `return 42`，实际是 `Promise<42>`
- 顶层 `await`：Node.js 模块（`.mts` 或 `"type": "module"`）支持

---

## 三、用泛型定义 API 响应类型

```typescript
// 通用 API 响应结构
interface ApiResponse<T> {
  data: T
  status: number
  message: string
}

interface PagedResponse<T> {
  items: T[]
  total: number
  page: number
  pageSize: number
}

// 具体业务类型
interface Article {
  id: number
  title: string
  content: string
  author: string
  publishedAt: string
}

// 类型安全的 fetch 封装
async function apiFetch<T>(url: string): Promise<T> {
  const response = await fetch(url, {
    headers: { "Accept": "application/json" }
  })
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${url}`)
  }
  return response.json() as Promise<T>
}

// 使用：类型全程安全
const articles = await apiFetch<Article[]>("/api/articles")
articles.forEach(a => console.log(a.title))  // 自动补全
```

---

## 四、并发控制

TypeScript 没有 Python 的 `asyncio.Semaphore`，用 Promise 手动实现并发限制：

```typescript
async function limit<T>(
  tasks: (() => Promise<T>)[],
  concurrency: number
): Promise<T[]> {
  const results: T[] = []
  const chunks: (typeof tasks)[] = []

  // 把任务分批（每批 concurrency 个）
  for (let i = 0; i < tasks.length; i += concurrency) {
    chunks.push(tasks.slice(i, i + concurrency))
  }

  for (const chunk of chunks) {
    const chunkResults = await Promise.all(chunk.map(fn => fn()))
    results.push(...chunkResults)
  }

  return results
}

// 使用
const urls = ["url1", "url2", "url3", "url4", "url5"]
const results = await limit(
  urls.map(url => () => fetch(url).then(r => r.json())),
  3   // 每次最多 3 个并发
)
```

---

## 五、实操案例：异步批量 URL 请求

新建 `day13/src/batch_fetch.ts`：

```typescript
interface FetchResult {
  url: string
  status: number
  ok: boolean
  contentType: string
  size: number
  error?: string
}

async function fetchOne(url: string): Promise<FetchResult> {
  try {
    const response = await fetch(url, {
      signal: AbortSignal.timeout(10000),  // 10秒超时
    })
    const body = await response.text()
    return {
      url,
      status: response.status,
      ok: response.ok,
      contentType: response.headers.get("content-type") ?? "",
      size: new TextEncoder().encode(body).length,
    }
  } catch (err) {
    return {
      url,
      status: 0,
      ok: false,
      contentType: "",
      size: 0,
      error: err instanceof Error ? err.message : String(err),
    }
  }
}

async function batchFetch(urls: string[], concurrency = 3): Promise<FetchResult[]> {
  console.log(`开始请求 ${urls.length} 个 URL，并发数: ${concurrency}`)
  const start = Date.now()

  const results: FetchResult[] = []
  for (let i = 0; i < urls.length; i += concurrency) {
    const chunk = urls.slice(i, i + concurrency)
    const chunkResults = await Promise.all(chunk.map(fetchOne))
    results.push(...chunkResults)
    console.log(`  进度: ${Math.min(i + concurrency, urls.length)}/${urls.length}`)
  }

  const elapsed = ((Date.now() - start) / 1000).toFixed(2)
  console.log(`\n完成，耗时 ${elapsed}s`)
  return results
}

function printReport(results: FetchResult[]): void {
  const ok = results.filter(r => r.ok)
  const fail = results.filter(r => !r.ok)

  console.log(`\n✓ 成功: ${ok.length}  ✗ 失败: ${fail.length}`)
  console.log("\n详细结果:")
  console.log(`${"URL".padEnd(45)} ${"状态".padStart(4)} ${"大小".padStart(8)}`)
  console.log("─".repeat(62))

  for (const r of results) {
    const urlStr = r.url.replace("https://", "").slice(0, 44).padEnd(45)
    const status = r.ok ? `${r.status}` : (r.error ? "ERR" : `${r.status}`)
    const size = r.ok ? `${(r.size / 1024).toFixed(1)}KB` : "-"
    console.log(`${urlStr} ${status.padStart(4)} ${size.padStart(8)}`)
  }
}

// 测试用 URL 列表
const testUrls = [
  "https://httpbin.org/get",
  "https://httpbin.org/json",
  "https://httpbin.org/uuid",
  "https://httpbin.org/user-agent",
  "https://httpbin.org/status/404",
  "https://httpbin.org/delay/1",
]

const results = await batchFetch(testUrls, 3)
printReport(results)
```

### 运行（需要 `"type": "module"` 支持顶层 await）

```bash
# 在 package.json 中加 "type": "module"
# tsconfig.json 中 "module": "ESNext", "moduleResolution": "bundler"

npx ts-node --esm src/batch_fetch.ts

# 或者用更简单的方式：
npx tsx src/batch_fetch.ts   # tsx 工具更简单
pnpm add -D tsx
```

---

## 六、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `async/await` | 所有异步函数 |
| `Promise.all()` | 并发执行一批请求 |
| `Promise.allSettled()` | 不管成功失败都等全部完成 |
| `AbortSignal.timeout()` | 请求超时控制 |
| 泛型 `ApiResponse<T>` | 类型安全的 API 响应 |
| 并发分批 | `chunk` 切割控制并发数 |

---

## 七、今日 Checklist

- [ ] 理解 Promise 三种状态
- [ ] 理解 `async` 函数一定返回 `Promise`
- [ ] `batch_fetch.ts` 运行，输出正确报告
- [ ] 对比 `Promise.all` 和 `Promise.allSettled` 的区别
- [ ] git commit：`git add . && git commit -m "day13: async batch fetch"`

> 📅 最后更新：2026-04-06
