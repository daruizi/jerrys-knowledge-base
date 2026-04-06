# Day 12 — Node.js 环境 & TypeScript 核心语法

> 目标：搭好 Node.js + TypeScript 开发环境，掌握 TS 类型系统核心，用 TS 完成数据转换脚本。

---

## 一、环境搭建

```bash
brew install node          # 安装 Node.js（含 npm）
node --version             # v20+
npm --version

npm install -g pnpm        # 安装 pnpm（比 npm 更快）
```

### 1.1 初始化 TypeScript 项目

```bash
mkdir day12 && cd day12
pnpm init                  # 生成 package.json
pnpm add -D typescript ts-node @types/node

# 生成 tsconfig.json
npx tsc --init
```

**`tsconfig.json` 关键配置：**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "CommonJS",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "./dist",
    "rootDir": "./src"
  }
}
```

### 1.2 运行方式

```bash
# 直接运行（开发阶段，无需编译）
npx ts-node src/index.ts

# 编译后运行
npx tsc && node dist/index.js

# 开发时自动重载
pnpm add -D ts-node-dev
npx ts-node-dev --respawn src/index.ts
```

---

## 二、TypeScript 类型系统

### 2.1 基本类型

```typescript
// 基本类型注解
const name: string = "Jerry"
const age: number = 30
const active: boolean = true
const nothing: null = null
const undef: undefined = undefined

// 数组
const nums: number[] = [1, 2, 3]
const strs: Array<string> = ["a", "b"]   // 等价写法

// 元组（固定长度和类型）
const point: [number, number] = [10, 20]
const entry: [string, number] = ["age", 30]

// any（避免使用）和 unknown（安全的 any）
let x: unknown = "hello"
if (typeof x === "string") {
  console.log(x.toUpperCase())   // 类型收窄后才能使用
}
```

### 2.2 联合类型与字面量类型

```typescript
// 联合类型：可以是多种类型之一
type ID = string | number
let userId: ID = "abc123"
userId = 42   // 也合法

// 字面量类型：限定为特定值
type Direction = "north" | "south" | "east" | "west"
type Status = "pending" | "done" | "failed"

let dir: Direction = "north"
// dir = "up"   // 编译错误！

// 可选类型（? 表示可以是该类型或 undefined）
function greet(name: string, greeting?: string): string {
  return `${greeting ?? "Hello"}, ${name}!`
}
```

### 2.3 interface 与 type

```typescript
// interface：描述对象结构（推荐用于对象/类）
interface User {
  id: number
  name: string
  email?: string         // 可选属性
  readonly createdAt: Date  // 只读属性
}

// type：更灵活（可以定义联合、交叉、基本类型别名）
type Point = { x: number; y: number }
type StringOrNumber = string | number
type WithTimestamp<T> = T & { createdAt: Date }  // 交叉类型

// interface 可以继承
interface AdminUser extends User {
  role: "admin" | "superadmin"
  permissions: string[]
}

// 使用
const user: User = { id: 1, name: "Alice", createdAt: new Date() }
```

### 2.4 泛型（Generics）

泛型让函数/类型支持多种类型，同时保持类型安全：

```typescript
// 没有泛型：只能处理一种类型
function firstItem(arr: number[]): number { return arr[0] }

// 有泛型：适用于任意类型，且返回类型与输入一致
function firstItem<T>(arr: T[]): T | undefined {
  return arr[0]
}

const n = firstItem([1, 2, 3])       // 类型推断为 number
const s = firstItem(["a", "b"])      // 类型推断为 string

// 泛型约束
function getLength<T extends { length: number }>(value: T): number {
  return value.length
}
getLength("hello")   // 5
getLength([1, 2, 3]) // 3

// 泛型接口
interface ApiResponse<T> {
  data: T
  status: number
  message: string
}

type UserResponse = ApiResponse<User>
type ListResponse = ApiResponse<User[]>
```

### 2.5 函数类型

```typescript
// 函数类型注解
function add(a: number, b: number): number {
  return a + b
}

// 箭头函数
const multiply = (a: number, b: number): number => a * b

// 函数类型别名
type Transformer<T, R> = (input: T) => R
type Predicate<T> = (item: T) => boolean

// 高阶函数
function filter<T>(arr: T[], predicate: Predicate<T>): T[] {
  return arr.filter(predicate)
}

const evens = filter([1, 2, 3, 4, 5], (n) => n % 2 === 0)
// 类型自动推断：evens: number[]
```

---

## 三、实操案例：数据转换流水线

### 目标

读取 JSON 数据，经过多步类型安全的转换，输出统计报告。

新建 `day12/src/data_transform.ts`：

```typescript
// ── 类型定义 ─────────────────────────────────
interface RawRecord {
  id: number
  name: string
  category: string
  price: string       // 原始数据是字符串
  in_stock: string    // "true" / "false"
  tags: string        // 逗号分隔
}

interface Product {
  id: number
  name: string
  category: string
  price: number
  inStock: boolean
  tags: string[]
}

interface CategoryStats {
  category: string
  count: number
  totalRevenue: number
  avgPrice: number
  inStockCount: number
}

// ── 原始测试数据 ──────────────────────────────
const rawData: RawRecord[] = [
  { id: 1, name: "MacBook Pro", category: "Electronics", price: "12999", in_stock: "true", tags: "laptop,apple,premium" },
  { id: 2, name: "Python Book", category: "Books", price: "89", in_stock: "true", tags: "python,programming" },
  { id: 3, name: "iPhone 15", category: "Electronics", price: "7999", in_stock: "false", tags: "phone,apple" },
  { id: 4, name: "TypeScript Book", category: "Books", price: "99", in_stock: "true", tags: "typescript,programming" },
  { id: 5, name: "AirPods Pro", category: "Electronics", price: "1999", in_stock: "true", tags: "audio,apple,wireless" },
  { id: 6, name: "React Guide", category: "Books", price: "79", in_stock: "false", tags: "react,frontend" },
]

// ── 转换函数 ──────────────────────────────────
function parseProduct(raw: RawRecord): Product {
  return {
    id: raw.id,
    name: raw.name,
    category: raw.category,
    price: parseFloat(raw.price),
    inStock: raw.in_stock === "true",
    tags: raw.tags.split(",").map(t => t.trim()),
  }
}

function groupBy<T>(items: T[], keyFn: (item: T) => string): Map<string, T[]> {
  return items.reduce((map, item) => {
    const key = keyFn(item)
    const group = map.get(key) ?? []
    group.push(item)
    map.set(key, group)
    return map
  }, new Map<string, T[]>())
}

function calcStats(category: string, products: Product[]): CategoryStats {
  const total = products.reduce((sum, p) => sum + p.price, 0)
  return {
    category,
    count: products.length,
    totalRevenue: total,
    avgPrice: total / products.length,
    inStockCount: products.filter(p => p.inStock).length,
  }
}

// ── 主流程 ────────────────────────────────────
function main(): void {
  // Step 1: 解析
  const products = rawData.map(parseProduct)
  console.log(`解析完成，共 ${products.length} 个商品\n`)

  // Step 2: 过滤（只看有库存的）
  const inStock = products.filter(p => p.inStock)
  console.log(`有库存: ${inStock.length} 个`)

  // Step 3: 按类别分组
  const grouped = groupBy(products, p => p.category)

  // Step 4: 统计每个类别
  const stats: CategoryStats[] = []
  grouped.forEach((items, category) => {
    stats.push(calcStats(category, items))
  })

  // Step 5: 排序 + 输出报告
  stats.sort((a, b) => b.totalRevenue - a.totalRevenue)

  console.log("\n── 分类统计报告 ──────────────────")
  console.log(`${"类别".<10} ${"数量".padStart(4)} ${"均价".padStart(8)} ${"总营收".padStart(10)} ${"有货".padStart(4)}`)
  console.log("─".repeat(45))
  for (const s of stats) {
    console.log(
      `${s.category.padEnd(12)} ${String(s.count).padStart(4)} ` +
      `${s.avgPrice.toFixed(0).padStart(8)} ` +
      `${s.totalRevenue.toFixed(0).padStart(10)} ` +
      `${s.inStockCount}/${s.count}`
    )
  }

  // Step 6: 找出所有独特标签
  const allTags = products.flatMap(p => p.tags)
  const tagCounts = allTags.reduce((acc, tag) => {
    acc[tag] = (acc[tag] ?? 0) + 1
    return acc
  }, {} as Record<string, number>)

  const topTags = Object.entries(tagCounts)
    .sort(([, a], [, b]) => b - a)
    .slice(0, 5)

  console.log("\nTop 5 标签:")
  topTags.forEach(([tag, count]) => console.log(`  #${tag}: ${count} 次`))
}

main()
```

### 运行

```bash
npx ts-node src/data_transform.ts
```

---

## 四、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `interface` 定义对象结构 | `RawRecord`, `Product`, `CategoryStats` |
| 泛型 `<T>` | `groupBy<T>()` 适用于任意类型分组 |
| 联合类型 `T \| U` | `T \| undefined` 返回值 |
| `Array.reduce()` | 累加、分组、计数 |
| `Array.flatMap()` | 扁平化嵌套数组（标签合并） |
| 类型推断 | `products.map(parseProduct)` 自动推断为 `Product[]` |

---

## 五、今日 Checklist

- [ ] `node --version` 和 `npx tsc --version` 正常
- [ ] `data_transform.ts` 运行输出正确报告
- [ ] 理解 `interface` 和 `type` 的区别
- [ ] 理解泛型 `<T>` 的作用
- [ ] git commit：`git add . && git commit -m "day12: typescript data transform"`

> 📅 最后更新：2026-04-06
