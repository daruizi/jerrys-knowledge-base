# Day 06 — 动态页面 & 批量下载

> 目标：理解静态抓取的局限，掌握 Playwright 控制浏览器抓取 JS 渲染页面，用 asyncio 实现高效批量下载。

---

## 一、静态抓取的局限

Day 05 的方法只能抓取**服务器直接返回 HTML 中的数据**。但很多现代网站用 JavaScript 动态加载数据：

```
httpx.get(url)  →  拿到 HTML 骨架（数据是空的）
                    ↓
              浏览器执行 JavaScript
                    ↓
              JavaScript 再发请求获取数据
                    ↓
              填充到页面（此时才有内容）
```

**如何判断页面是否动态加载？**  
右键 → 查看页面源代码（不是"检查元素"），如果看不到目标数据，就是动态渲染。

**两种解决方案：**
1. **逆向 API**：找到 JS 发出的真实数据请求，直接调用（更高效，但需要分析）
2. **浏览器自动化**：用代码控制真实浏览器，等 JS 执行完再抓（更通用）

---

## 二、Playwright — 浏览器自动化

Playwright 是微软开发的浏览器自动化库，支持 Chrome、Firefox、Safari。

```bash
pip install playwright
playwright install chromium   # 下载 Chromium 浏览器
```

### 2.1 基本用法

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    # 启动浏览器（headless=False 可以看到浏览器窗口，调试时用）
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()

    # 导航到页面
    page.goto("https://example.com")

    # 等待某个元素出现（动态页面必须等待）
    page.wait_for_selector(".book-list")

    # 获取页面 HTML，交给 BeautifulSoup 解析
    html = page.content()

    browser.close()
```

### 2.2 常用操作

```python
# 点击按钮
page.click(".load-more-btn")

# 填写表单
page.fill("input[name='keyword']", "Python")
page.press("input[name='keyword']", "Enter")

# 等待页面加载
page.wait_for_load_state("networkidle")  # 等待网络请求静止
page.wait_for_selector(".results")       # 等待元素出现

# 滚动到页面底部（触发懒加载）
page.evaluate("window.scrollTo(0, document.body.scrollHeight)")
page.wait_for_timeout(2000)  # 等待 2 秒

# 直接提取文本（不用 BeautifulSoup）
title = page.locator("h1").text_content()
items = page.locator(".item").all_text_contents()  # 返回列表

# 截图（调试利器）
page.screenshot(path="debug.png")
```

### 2.3 设置 User-Agent，避免被识别

```python
browser = p.chromium.launch(headless=True)
context = browser.new_context(
    user_agent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
               "AppleWebKit/537.36 (KHTML, like Gecko) "
               "Chrome/120.0.0.0 Safari/537.36",
    viewport={"width": 1920, "height": 1080},
)
page = context.new_page()
```

---

## 三、asyncio — 异步并发

串行下载 100 个文件，每个等 2 秒 = 200 秒。  
并发下载 100 个文件，同时等 2 秒 = ~2 秒。

### 3.1 async/await 基础

```python
import asyncio

async def fetch(url: str) -> str:
    # await 会暂停当前函数，让其他任务趁机运行
    await asyncio.sleep(1)  # 模拟网络请求耗时
    return f"data from {url}"


async def main():
    # 串行：总共 3 秒
    # result1 = await fetch("url1")
    # result2 = await fetch("url2")
    # result3 = await fetch("url3")

    # 并发：总共 ~1 秒
    results = await asyncio.gather(
        fetch("url1"),
        fetch("url2"),
        fetch("url3"),
    )
    print(results)


asyncio.run(main())
```

### 3.2 异步 HTTP 请求（httpx 异步版）

```python
import asyncio
import httpx

async def download(client: httpx.AsyncClient, url: str) -> bytes:
    response = await client.get(url)
    return response.content


async def download_all(urls: list[str]) -> list[bytes]:
    async with httpx.AsyncClient() as client:
        tasks = [download(client, url) for url in urls]
        return await asyncio.gather(*tasks)


urls = ["https://example.com/file1.jpg", "https://example.com/file2.jpg"]
results = asyncio.run(download_all(urls))
```

### 3.3 控制并发数量（避免服务器压力过大）

`asyncio.Semaphore` 限制同时运行的任务数：

```python
import asyncio
import httpx

async def download(semaphore, client, url, output_path):
    async with semaphore:  # 同时最多 5 个任务
        response = await client.get(url)
        with open(output_path, "wb") as f:
            f.write(response.content)
        print(f"✓ {output_path}")


async def batch_download(urls: list[str], output_dir: str, concurrency: int = 5):
    semaphore = asyncio.Semaphore(concurrency)
    async with httpx.AsyncClient(timeout=30) as client:
        tasks = [
            download(semaphore, client, url, f"{output_dir}/{i:04d}.jpg")
            for i, url in enumerate(urls)
        ]
        await asyncio.gather(*tasks)
```

---

## 四、实操案例 1：动态页面抓取

### 目标

抓取一个用 JavaScript 渲染的页面。我们用 `quotes.toscrape.com/js/`（专门用于练习的网站，JS 渲染版）。

新建 `day06/dynamic_scraper.py`：

```python
from playwright.sync_api import sync_playwright
from bs4 import BeautifulSoup
from dataclasses import dataclass
import json


@dataclass
class Quote:
    text: str
    author: str
    tags: list[str]


def scrape_quotes(pages: int = 3) -> list[Quote]:
    all_quotes: list[Quote] = []

    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        context = browser.new_context(
            user_agent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                       "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
        )
        page = context.new_page()

        for i in range(1, pages + 1):
            url = f"https://quotes.toscrape.com/js/page/{i}/"
            print(f"抓取第 {i} 页: {url}")

            page.goto(url)
            # 等待 JS 渲染完成（等待 quote 元素出现）
            page.wait_for_selector(".quote", timeout=10000)

            soup = BeautifulSoup(page.content(), "lxml")
            quotes = soup.select(".quote")

            for q in quotes:
                text = q.select_one(".text").get_text(strip=True).strip('""\u201c\u201d')
                author = q.select_one(".author").get_text(strip=True)
                tags = [tag.get_text(strip=True) for tag in q.select(".tag")]
                all_quotes.append(Quote(text=text, author=author, tags=tags))

            print(f"  获取到 {len(quotes)} 条名言")

        browser.close()

    return all_quotes


def main():
    quotes = scrape_quotes(pages=3)

    # 保存为 JSON
    data = [{"text": q.text, "author": q.author, "tags": q.tags} for q in quotes]
    with open("data/quotes.json", "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)

    print(f"\n共抓取 {len(quotes)} 条名言，已保存到 data/quotes.json")
    print("\n预览：")
    for q in quotes[:3]:
        print(f'  "{q.text[:50]}..." — {q.author}')


if __name__ == "__main__":
    import os
    os.makedirs("data", exist_ok=True)
    main()
```

---

## 五、实操案例 2：批量下载图片

### 目标

从一个 JSON 文件读取图片 URL 列表，并发批量下载到本地，显示进度。

新建 `day06/batch_downloader.py`：

```python
import asyncio
import httpx
from pathlib import Path
from dataclasses import dataclass


@dataclass
class DownloadTask:
    url: str
    filename: str


async def download_file(
    semaphore: asyncio.Semaphore,
    client: httpx.AsyncClient,
    task: DownloadTask,
    output_dir: Path,
) -> tuple[str, bool]:
    """下载单个文件，返回 (文件名, 是否成功)"""
    async with semaphore:
        try:
            response = await client.get(task.url, follow_redirects=True)
            response.raise_for_status()

            filepath = output_dir / task.filename
            filepath.write_bytes(response.content)
            return task.filename, True

        except Exception as e:
            print(f"  ✗ {task.filename}: {e}")
            return task.filename, False


async def batch_download(
    tasks: list[DownloadTask],
    output_dir: str = "downloads",
    concurrency: int = 5,
) -> None:
    output = Path(output_dir)
    output.mkdir(exist_ok=True)

    semaphore = asyncio.Semaphore(concurrency)
    success = 0
    failed = 0

    print(f"开始下载 {len(tasks)} 个文件（并发数: {concurrency}）")

    async with httpx.AsyncClient(timeout=30) as client:
        coros = [download_file(semaphore, client, task, output) for task in tasks]
        results = await asyncio.gather(*coros)

    for filename, ok in results:
        if ok:
            success += 1
            print(f"  ✓ {filename}")
        else:
            failed += 1

    print(f"\n完成：成功 {success} 个，失败 {failed} 个")
    print(f"文件保存在: {output.absolute()}")


def main():
    # 用 Lorem Picsum 的随机图片做示例（稳定可访问）
    tasks = [
        DownloadTask(
            url=f"https://picsum.photos/seed/{i}/400/300",
            filename=f"image_{i:03d}.jpg",
        )
        for i in range(1, 11)   # 下载 10 张图片
    ]

    asyncio.run(batch_download(tasks, output_dir="downloads", concurrency=5))


if __name__ == "__main__":
    main()
```

### 运行

```bash
cd day06
pip install playwright httpx beautifulsoup4 lxml
playwright install chromium

# 运行动态抓取
python3 dynamic_scraper.py

# 运行批量下载
python3 batch_downloader.py
```

---

## 六、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `page.wait_for_selector()` | 等待 JS 渲染完成再抓取 |
| `page.screenshot()` | 调试利器，看到浏览器看到什么 |
| `async/await` | 异步编程，不阻塞等待 |
| `asyncio.gather()` | 并发执行多个协程 |
| `asyncio.Semaphore` | 限制并发数，保护服务器 |
| `httpx.AsyncClient` | 异步 HTTP 客户端 |
| `follow_redirects=True` | 自动跟随重定向 |

---

## 七、今日 Checklist

- [ ] `dynamic_scraper.py` 成功抓取并生成 `data/quotes.json`
- [ ] `batch_downloader.py` 成功下载 10 张图片到 `downloads/`
- [ ] 对比串行和并发的耗时差异（在 batch_downloader 里加计时）
- [ ] 理解 `asyncio.Semaphore` 的作用
- [ ] git commit：`git add . && git commit -m "day06: playwright scraper + batch downloader"`

> 📅 最后更新：2026-04-06
