# Day 06 — 装饰器、HTTP 与异步编程

> 目标：掌握装饰器的完整用法，理解 HTTP 协议，用 httpx 同步/异步请求 API，掌握 asyncio 并发编程。

---

## 一、装饰器（Decorator）

### 1.1 本质：函数包裹函数

装饰器就是一个**接收函数、返回新函数**的高阶函数。`@decorator` 语法是语法糖：

```python
@my_decorator
def say_hello():
    print("Hello")

# 等价于：
say_hello = my_decorator(say_hello)
```

### 1.2 手动实现：计时装饰器

```python
import time
from functools import wraps


def timer(func):
    @wraps(func)        # 保留原函数的 __name__、__doc__ 等元信息
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)       # 调用原函数
        elapsed = time.perf_counter() - start
        print(f"[timer] {func.__name__} 耗时 {elapsed:.4f}s")
        return result
    return wrapper


@timer
def slow_function(n: int) -> int:
    time.sleep(0.1)
    return n * 2

print(slow_function(5))
# [timer] slow_function 耗时 0.1002s
# 10

# 验证 @wraps 的作用
print(slow_function.__name__)  # slow_function（不是 wrapper）
```

### 1.3 带参数的装饰器

需要在外面再套一层（工厂函数）：

```python
def retry(max_times: int = 3, delay: float = 1.0):
    """重试装饰器：失败时自动重试"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_times + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_times:
                        raise
                    print(f"[retry] 第 {attempt} 次失败: {e}，{delay}s 后重试")
                    time.sleep(delay)
        return wrapper
    return decorator


@retry(max_times=3, delay=0.5)
def unstable_request(url: str) -> str:
    import random
    if random.random() < 0.7:   # 70% 概率失败
        raise ConnectionError("网络错误")
    return f"成功获取: {url}"
```

### 1.4 类装饰器

用类实现装饰器，通过 `__call__` 方法：

```python
class RateLimit:
    """限流装饰器：每隔 interval 秒最多调用一次"""
    def __init__(self, interval: float):
        self.interval = interval
        self._last_called = 0.0

    def __call__(self, func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            elapsed = time.time() - self._last_called
            if elapsed < self.interval:
                time.sleep(self.interval - elapsed)
            self._last_called = time.time()
            return func(*args, **kwargs)
        return wrapper


@RateLimit(interval=1.0)  # 每秒最多调用一次
def fetch_api(url: str) -> str:
    return f"data from {url}"


# 类装饰器也可以直接作为实例，适合需要保持状态的场景
class Counter:
    """统计函数调用次数"""
    def __init__(self, func):
        wraps(func)(self)   # 复制元信息到 self
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        return self.func(*args, **kwargs)


@Counter
def greet(name: str) -> str:
    return f"Hello, {name}!"

greet("Alice")
greet("Bob")
print(greet.count)  # 2
```

### 1.5 装饰器堆叠

多个装饰器叠加时，**从下到上**应用（离函数最近的先执行）：

```python
@timer          # 第二个应用（最外层）
@retry(max_times=3)  # 第一个应用（内层）
def fetch(url: str) -> str:
    ...

# 等价于：
fetch = timer(retry(max_times=3)(fetch))
# 调用顺序：timer.wrapper → retry.wrapper → fetch
```

### 1.6 实用装饰器集锦

```python
# 缓存（记忆化）
from functools import lru_cache, cache

@lru_cache(maxsize=128)  # LRU 缓存，最多保留 128 个结果
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

@cache  # Python 3.9+，等价于 lru_cache(maxsize=None)
def expensive_calc(n: int) -> int:
    ...

# 日志装饰器
import logging

def log_call(func):
    logger = logging.getLogger(func.__module__)
    @wraps(func)
    def wrapper(*args, **kwargs):
        logger.debug("→ %s(%s)", func.__name__, args)
        result = func(*args, **kwargs)
        logger.debug("← %s = %s", func.__name__, result)
        return result
    return wrapper

# 属性验证装饰器
def validate_positive(func):
    @wraps(func)
    def wrapper(self, value):
        if value <= 0:
            raise ValueError(f"{func.__name__} 需要正数，得到 {value}")
        return func(self, value)
    return wrapper

class Circle:
    @validate_positive
    def set_radius(self, value: float) -> None:
        self._radius = value
```

---

## 二、HTTP 基础

在用 httpx 之前，先理解 HTTP 协议结构。

### 2.1 HTTP 请求结构

```
GET /api/users?page=1 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGc...
Accept: application/json
User-Agent: python-httpx/0.27

（Body 为空，GET 通常没有 Body）
```

**关键组成：**

| 组成部分 | 说明 | 示例 |
|----------|------|------|
| 方法（Method） | 操作类型 | GET、POST、PUT、PATCH、DELETE |
| URL | 资源地址 | `https://api.example.com/users/1` |
| Headers | 元信息 | Content-Type、Authorization、Cookie |
| Body | 请求体（可选） | JSON 数据、表单数据 |

### 2.2 HTTP 方法语义

| 方法 | 用途 | 是否有 Body | 幂等 |
|------|------|------------|------|
| GET | 获取资源 | 否 | ✓ |
| POST | 创建资源 | 是 | ✗ |
| PUT | 完整替换 | 是 | ✓ |
| PATCH | 部分更新 | 是 | ✗ |
| DELETE | 删除资源 | 否 | ✓ |

### 2.3 HTTP 状态码

| 范围 | 含义 | 常见状态码 |
|------|------|----------|
| 2xx | 成功 | 200 OK、201 Created、204 No Content |
| 3xx | 重定向 | 301 永久、302 临时、304 Not Modified |
| 4xx | 客户端错误 | 400 Bad Request、401 未认证、403 禁止、404 Not Found、429 限流 |
| 5xx | 服务端错误 | 500 内部错误、502 Bad Gateway、503 服务不可用 |

---

## 三、httpx — 现代 HTTP 客户端

`httpx` 同时支持同步和异步，是 `requests` 的现代替代品。

```bash
pip install httpx
```

### 3.1 同步请求基础

```python
import httpx

# GET 请求
resp = httpx.get("https://httpbin.org/get")
print(resp.status_code)    # 200
print(resp.headers)         # 响应头
print(resp.text)            # 响应体（字符串）
print(resp.json())          # 解析为 JSON（dict）

resp.raise_for_status()     # 状态码 >= 400 时抛出异常

# 带参数
resp = httpx.get(
    "https://httpbin.org/get",
    params={"page": 1, "limit": 20},  # 自动编码为 ?page=1&limit=20
)

# POST JSON
resp = httpx.post(
    "https://httpbin.org/post",
    json={"name": "Alice", "age": 25},  # 自动设置 Content-Type: application/json
)

# 带请求头（如 Authorization）
resp = httpx.get(
    "https://api.example.com/users",
    headers={"Authorization": "Bearer my-token"},
)
```

### 3.2 Client（复用连接）

频繁请求同一服务器时，用 `Client` 复用 TCP 连接（性能更好）：

```python
with httpx.Client(
    base_url="https://httpbin.org",
    headers={"User-Agent": "my-scraper/1.0"},
    timeout=10.0,
    follow_redirects=True,
) as client:
    resp1 = client.get("/get")
    resp2 = client.post("/post", json={"key": "value"})
    resp3 = client.get("/json")
    # 三个请求共享同一个 TCP 连接
```

### 3.3 错误处理

```python
import httpx

def safe_get(url: str) -> dict | None:
    try:
        with httpx.Client(timeout=10) as client:
            resp = client.get(url)
            resp.raise_for_status()
            return resp.json()
    except httpx.TimeoutException:
        print(f"请求超时: {url}")
    except httpx.HTTPStatusError as e:
        print(f"HTTP 错误 {e.response.status_code}: {url}")
    except httpx.RequestError as e:
        print(f"连接错误: {e}")
    return None
```

---

## 四、asyncio 异步编程

### 4.1 为什么需要异步？

```
同步（串行）：
  请求1: ──发送──[等待 1s]──处理──
  请求2:                    ──发送──[等待 1s]──处理──
  总耗时 = 1s + 1s = 2s

异步（并发）：
  请求1: ──发送──[等待 1s]──处理──
  请求2: ──发送──[等待 1s]──处理──  ← 同时等待
  总耗时 ≈ 1s
```

**关键规则**：`await` 只能在 `async def` 函数内使用。

### 4.2 基本用法

```python
import asyncio

async def fetch(url: str) -> str:
    await asyncio.sleep(1)   # 模拟 1 秒网络请求（await 让出控制权）
    return f"data from {url}"

async def main():
    result = await fetch("https://example.com")
    print(result)

asyncio.run(main())   # 启动事件循环（程序入口）
```

### 4.3 并发：gather

```python
async def task(name: str, delay: float) -> str:
    await asyncio.sleep(delay)
    return f"{name} done in {delay}s"

async def main():
    # 并发执行，等全部完成（类似 Promise.all）
    results = await asyncio.gather(
        task("A", 1),
        task("B", 2),
        task("C", 3),
    )
    print(results)  # 约 3 秒后输出，不是 6 秒

    # return_exceptions=True：某个失败不影响其他
    results = await asyncio.gather(
        task("A", 1),
        task("B", 2),
        return_exceptions=True
    )
```

### 4.4 TaskGroup（Python 3.11+）

比 `gather` 更安全：任意任务失败时自动取消其他任务：

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(task("A", 1))
        t2 = tg.create_task(task("B", 2))
        t3 = tg.create_task(task("C", 3))
    # 所有任务完成后继续
    print(t1.result(), t2.result(), t3.result())
```

### 4.5 Semaphore — 控制并发数

```python
async def worker(sem: asyncio.Semaphore, name: str) -> str:
    async with sem:           # 同时最多 N 个任务进入
        print(f"{name} 开始")
        await asyncio.sleep(1)
        return name

async def main():
    sem = asyncio.Semaphore(3)   # 并发上限 3
    tasks = [worker(sem, f"任务{i}") for i in range(10)]
    results = await asyncio.gather(*tasks)
```

### 4.6 Queue — 生产者消费者

```python
async def producer(queue: asyncio.Queue, urls: list[str]) -> None:
    for url in urls:
        await queue.put(url)
    await queue.put(None)  # 哨兵值，通知消费者结束

async def consumer(queue: asyncio.Queue, client: httpx.AsyncClient) -> list:
    results = []
    while True:
        url = await queue.get()
        if url is None:
            break
        resp = await client.get(url)
        results.append({"url": url, "status": resp.status_code})
        queue.task_done()
    return results

async def main():
    queue = asyncio.Queue(maxsize=10)
    urls = ["https://httpbin.org/get"] * 5

    async with httpx.AsyncClient() as client:
        prod = asyncio.create_task(producer(queue, urls))
        cons = asyncio.create_task(consumer(queue, client))
        await prod
        results = await cons
    print(results)
```

### 4.7 异步 httpx

```python
import asyncio
import httpx

async def fetch_url(client: httpx.AsyncClient, url: str) -> dict:
    try:
        resp = await client.get(url, timeout=10, follow_redirects=True)
        resp.raise_for_status()
        return {"url": url, "status": resp.status_code, "ok": True}
    except Exception as e:
        return {"url": url, "error": str(e), "ok": False}

async def fetch_all(urls: list[str], concurrency: int = 5) -> list[dict]:
    sem = asyncio.Semaphore(concurrency)
    async with httpx.AsyncClient() as client:
        async def limited_fetch(url):
            async with sem:
                return await fetch_url(client, url)
        return await asyncio.gather(*[limited_fetch(u) for u in urls])
```

---

## 五、环境变量（python-dotenv）

API Key 等敏感信息不应硬编码在代码里，用 `.env` 文件管理：

```bash
pip install python-dotenv
```

`.env` 文件（添加到 `.gitignore`，不要提交到 git！）：

```
API_KEY=sk-abc123
BASE_URL=https://api.example.com
MAX_RETRIES=3
```

```python
import os
from dotenv import load_dotenv

load_dotenv()  # 读取 .env 文件，加载到环境变量

api_key = os.getenv("API_KEY")
base_url = os.getenv("BASE_URL", "https://default.example.com")
max_retries = int(os.getenv("MAX_RETRIES", "3"))

print(api_key)  # sk-abc123
```

---

## 六、实操案例：带装饰器的异步批量请求器

新建 `async_fetcher.py`：

```python
import asyncio
import time
import httpx
from functools import wraps
from pathlib import Path

# ── 装饰器 ────────────────────────────────────

def async_timer(func):
    """计时装饰器（async 版本）"""
    @wraps(func)
    async def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = await func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"[耗时] {func.__name__}: {elapsed:.2f}s")
        return result
    return wrapper


class RateLimitedClient:
    """带限流的 httpx 客户端（类装饰器风格）"""
    def __init__(self, concurrency: int = 5):
        self.sem = asyncio.Semaphore(concurrency)

    def __call__(self, func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            async with self.sem:
                return await func(*args, **kwargs)
        return wrapper


rate_limit = RateLimitedClient(concurrency=5)


# ── 核心请求逻辑 ──────────────────────────────

@rate_limit
async def fetch_one(client: httpx.AsyncClient, url: str, max_retries: int = 2) -> dict:
    """带重试的单次请求"""
    for attempt in range(1, max_retries + 2):
        try:
            resp = await client.get(url, timeout=10, follow_redirects=True)
            resp.raise_for_status()
            return {"url": url, "status": resp.status_code, "size": len(resp.content), "ok": True}
        except (httpx.HTTPStatusError, httpx.RequestError) as e:
            if attempt <= max_retries:
                await asyncio.sleep(0.5 * attempt)
            else:
                return {"url": url, "error": str(e), "ok": False}


@async_timer
async def fetch_all(urls: list[str]) -> list[dict]:
    async with httpx.AsyncClient() as client:
        tasks = [fetch_one(client, url) for url in urls]
        return await asyncio.gather(*tasks)


# ── 输出报告 ──────────────────────────────────

def print_report(results: list[dict]) -> None:
    ok = [r for r in results if r.get("ok")]
    fail = [r for r in results if not r.get("ok")]

    print(f"\n✓ 成功 {len(ok)} 个")
    for r in ok:
        print(f"  {r['status']}  {r['size']:>8} bytes  {r['url']}")

    if fail:
        print(f"\n✗ 失败 {len(fail)} 个")
        for r in fail:
            print(f"  {r['url']}  {r.get('error', '')}")


def main() -> None:
    urls = [
        "https://httpbin.org/get",
        "https://httpbin.org/json",
        "https://httpbin.org/uuid",
        "https://httpbin.org/user-agent",
        "https://httpbin.org/headers",
        "https://httpbin.org/status/404",   # 触发错误
        "https://httpbin.org/delay/1",
        "https://httpbin.org/ip",
    ]

    print(f"并发请求 {len(urls)} 个 URL（限流: 5）\n")
    results = asyncio.run(fetch_all(urls))
    print_report(results)


if __name__ == "__main__":
    main()
```

### 运行

```bash
pip install httpx
python3 async_fetcher.py
```

---

## 七、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `@wraps(func)` | 保留原函数 `__name__` / `__doc__` |
| 带参数装饰器 | `retry(max_times=3)` 三层嵌套 |
| 类装饰器 | `RateLimitedClient.__call__` |
| 装饰器堆叠 | `@rate_limit` + `@async_timer` 叠加 |
| HTTP 方法和状态码 | `raise_for_status()` 处理 4xx/5xx |
| `httpx.Client` | 复用连接，设置 base_url / timeout |
| `asyncio.gather` | 并发执行多个协程 |
| `asyncio.Semaphore` | 限制并发数量 |
| 异步装饰器 | `async def wrapper` 包裹 `async def func` |

---

## 八、今日 Checklist

- [ ] 能不看笔记手写一个计时装饰器（带 `@wraps`）
- [ ] 理解带参数装饰器的三层嵌套结构
- [ ] 理解 HTTP 方法语义和状态码范围
- [ ] 能用 `httpx.Client` 发 GET / POST 请求
- [ ] 理解 `async/await` 和 `asyncio.gather` 的并发原理
- [ ] `async_fetcher.py` 运行成功，观察并发速度提升
- [ ] 对比：把 `gather` 改为串行 `await`，观察耗时差异
- [ ] git commit：`git add . && git commit -m "day06: async fetcher with decorators"`

> 📅 最后更新：2026-04-07
