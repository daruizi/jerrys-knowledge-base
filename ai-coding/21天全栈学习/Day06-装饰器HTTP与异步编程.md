# Day 06 — 装饰器 & asyncio 异步编程

> 目标：理解装饰器的本质并自己实现，掌握 async/await 异步编程模型，完成异步批量请求脚本。

---

## 一、装饰器（Decorator）

### 1.1 本质：函数包裹函数

装饰器就是一个**接收函数、返回新函数**的高阶函数。`@decorator` 语法是语法糖：

```python
@my_decorator
def say_hello():
    print("Hello")

# 等价于：
def say_hello():
    print("Hello")
say_hello = my_decorator(say_hello)
```

### 1.2 手动实现一个装饰器

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
```

### 1.3 带参数的装饰器

需要再包一层：

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
    if random.random() < 0.7:   # 70% 概率失败，模拟不稳定网络
        raise ConnectionError("网络错误")
    return f"成功获取: {url}"
```

### 1.4 实用装饰器集锦

**缓存（记忆化）：**

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(50))   # 瞬间完成（不缓存会递归爆炸）
```

**日志装饰器：**

```python
import logging

def log_call(func):
    logger = logging.getLogger(func.__module__)
    @wraps(func)
    def wrapper(*args, **kwargs):
        logger.info("调用 %s，参数: args=%s kwargs=%s", func.__name__, args, kwargs)
        result = func(*args, **kwargs)
        logger.info("%s 返回: %s", func.__name__, result)
        return result
    return wrapper
```

---

## 二、生成器进阶（配合 Day 02 复习）

生成器和异步编程紧密相关，先巩固一下：

```python
# yield 暂停函数，保存状态，下次 next() 继续
def countdown(n: int):
    while n > 0:
        yield n
        n -= 1

for i in countdown(3):
    print(i)   # 3, 2, 1

# 生成器表达式（节省内存）
squares = (x ** 2 for x in range(1_000_000))  # 不会立即计算
print(next(squares))   # 0

# yield from：委托给另一个可迭代对象
def chain(*iterables):
    for it in iterables:
        yield from it

print(list(chain([1, 2], [3, 4], [5])))  # [1, 2, 3, 4, 5]
```

---

## 三、asyncio 异步编程

### 3.1 为什么需要异步？

```
同步（串行）：
  任务1: 发请求 ──等待── 处理   # 等待时 CPU 闲着
  任务2:                发请求 ──等待── 处理
  总耗时 = 任务1 + 任务2

异步（并发）：
  任务1: 发请求 ──等待──
  任务2:        发请求 ──等待──   # 等待时做其他任务
  总耗时 ≈ max(任务1, 任务2)
```

**适用场景：大量 I/O 等待**（网络请求、文件读写）。CPU 密集型用多进程，不用异步。

### 3.2 核心概念

```python
import asyncio

# async def 定义一个协程函数
async def fetch(url: str) -> str:
    await asyncio.sleep(1)   # await 让出控制权，允许其他协程运行
    return f"data from {url}"

# 运行协程
async def main():
    result = await fetch("https://example.com")
    print(result)

asyncio.run(main())   # 程序入口，启动事件循环
```

### 3.3 并发执行多个任务

```python
import asyncio

async def task(name: str, delay: float) -> str:
    await asyncio.sleep(delay)
    return f"{name} 完成（{delay}s）"

async def main():
    # 串行：总共 1+2+3 = 6 秒
    # r1 = await task("A", 1)
    # r2 = await task("B", 2)
    # r3 = await task("C", 3)

    # 并发：总共约 3 秒
    results = await asyncio.gather(
        task("A", 1),
        task("B", 2),
        task("C", 3),
    )
    print(results)  # ['A 完成（1s）', 'B 完成（2s）', 'C 完成（3s）']

asyncio.run(main())
```

### 3.4 控制并发数（Semaphore）

```python
import asyncio

async def worker(sem: asyncio.Semaphore, name: str) -> str:
    async with sem:           # 同时最多 3 个任务进入
        print(f"{name} 开始")
        await asyncio.sleep(1)
        print(f"{name} 完成")
        return name

async def main():
    sem = asyncio.Semaphore(3)   # 并发上限 3
    tasks = [worker(sem, f"任务{i}") for i in range(10)]
    results = await asyncio.gather(*tasks)
    print(f"全部完成: {results}")

asyncio.run(main())
```

### 3.5 异步 HTTP 请求（httpx 异步版）

```python
import asyncio
import httpx

async def fetch_url(client: httpx.AsyncClient, url: str) -> dict:
    try:
        response = await client.get(url, timeout=10)
        response.raise_for_status()
        return {"url": url, "status": response.status_code, "ok": True}
    except Exception as e:
        return {"url": url, "error": str(e), "ok": False}

async def fetch_all(urls: list[str]) -> list[dict]:
    async with httpx.AsyncClient() as client:
        tasks = [fetch_url(client, url) for url in urls]
        return await asyncio.gather(*tasks)
```

---

## 四、实操案例：带重试装饰器的异步批量请求

### 目标

从 URL 列表文件异步批量请求，失败自动重试，输出结果统计。

新建 `day06/async_fetcher.py`：

```python
import asyncio
import time
import httpx
import json
from functools import wraps
from pathlib import Path

# ── 装饰器 ────────────────────────────────────

def timer(func):
    """计时装饰器（支持 async 函数）"""
    @wraps(func)
    async def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = await func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"\n[耗时] {func.__name__}: {elapsed:.2f}s")
        return result
    return wrapper


# ── 核心逻辑 ──────────────────────────────────

async def fetch_one(
    client: httpx.AsyncClient,
    sem: asyncio.Semaphore,
    url: str,
    max_retries: int = 2,
) -> dict:
    """带限流和重试的单次请求"""
    async with sem:
        for attempt in range(1, max_retries + 2):
            try:
                resp = await client.get(url, timeout=10, follow_redirects=True)
                resp.raise_for_status()
                return {
                    "url": url,
                    "status": resp.status_code,
                    "length": len(resp.content),
                    "ok": True,
                }
            except (httpx.HTTPError, httpx.RequestError) as e:
                if attempt <= max_retries:
                    await asyncio.sleep(0.5 * attempt)   # 指数退避
                else:
                    return {"url": url, "error": str(e), "ok": False}


@timer
async def fetch_all(urls: list[str], concurrency: int = 5) -> list[dict]:
    """并发批量请求"""
    sem = asyncio.Semaphore(concurrency)
    async with httpx.AsyncClient() as client:
        tasks = [fetch_one(client, sem, url) for url in urls]
        return await asyncio.gather(*tasks)


def print_report(results: list[dict]) -> None:
    ok = [r for r in results if r.get("ok")]
    fail = [r for r in results if not r.get("ok")]

    print(f"\n成功: {len(ok)} 个")
    for r in ok:
        print(f"  ✓ {r['url'][:50]:<50} {r['status']}  {r['length']:>8} bytes")

    if fail:
        print(f"\n失败: {len(fail)} 个")
        for r in fail:
            print(f"  ✗ {r['url'][:50]:<50} {r.get('error', '')}")


def main() -> None:
    # 测试用 URL 列表（httpbin.org 是专门用于测试 HTTP 请求的网站）
    urls = [
        "https://httpbin.org/get",
        "https://httpbin.org/status/200",
        "https://httpbin.org/status/404",   # 会触发错误
        "https://httpbin.org/delay/1",
        "https://httpbin.org/json",
        "https://httpbin.org/uuid",
        "https://httpbin.org/user-agent",
        "https://httpbin.org/headers",
    ]

    print(f"开始并发请求 {len(urls)} 个 URL（并发数: 5）")
    results = asyncio.run(fetch_all(urls, concurrency=5))
    print_report(results)


if __name__ == "__main__":
    main()
```

### 运行

```bash
pip install httpx
python3 async_fetcher.py
```

预期输出：

```
开始并发请求 8 个 URL（并发数: 5）

[耗时] fetch_all: 1.23s    ← 全部并发，不是 8 个串行

成功: 7 个
  ✓ https://httpbin.org/get                     200     382 bytes
  ...

失败: 1 个
  ✗ https://httpbin.org/status/404              HTTP Error 404
```

---

## 五、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `@wraps(func)` | 保留原函数元信息，避免装饰后 `__name__` 变成 `wrapper` |
| 带参数的装饰器 | 三层嵌套：`decorator_factory → decorator → wrapper` |
| `async def` / `await` | 定义协程，暂停等待 I/O |
| `asyncio.gather()` | 并发执行多个协程，等全部完成 |
| `asyncio.Semaphore` | 限制并发数，保护服务器和自身 |
| `httpx.AsyncClient` | 异步 HTTP 客户端（与 `asyncio` 配合） |
| 指数退避 | `sleep(0.5 * attempt)`：重试间隔递增 |

---

## 六、今日 Checklist

- [ ] 不看笔记，手写一个计时装饰器
- [ ] 理解 `@wraps` 的作用（去掉后 `print(fetch_all.__name__)` 输出什么？）
- [ ] `async_fetcher.py` 运行成功，观察并发带来的速度提升
- [ ] 对比：把 `asyncio.gather` 改成串行 `await`，耗时有何变化
- [ ] git commit：`git add . && git commit -m "day06: async fetcher with decorators"`

> 📅 最后更新：2026-04-06
