# Day 09 — 动态页面抓取

> 目标：理解为什么静态抓取会失效，掌握 Playwright 控制浏览器抓取 JavaScript 渲染的页面。

---

## 一、为什么静态抓取会失效

用 `httpx.get(url)` 只能拿到**服务器返回的原始 HTML**，但很多现代网站这样工作：

```
服务器返回 HTML 骨架（几乎没有数据）
        ↓
浏览器加载并执行 JavaScript
        ↓
JavaScript 再发 AJAX/Fetch 请求获取真正数据
        ↓
把数据填充到 DOM（此时你才能在页面上看到内容）
```

**判断是否动态渲染**：
1. 右键 → 查看页面源代码（不是"检查元素"）
2. `Ctrl+F` 搜索你想要的文字
3. 找不到 → 动态渲染，需要 Playwright

---

## 二、两种解决方案

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **逆向 API** | 速度快，无需浏览器 | 需要分析网络请求，可能有签名验证 | API 接口明确、参数简单 |
| **Playwright** | 通用，所见即所得 | 速度慢，资源消耗大 | 无法逆向或渲染复杂 |

**优先尝试逆向 API**，实在不行再用 Playwright。

---

## 三、如何逆向 API（先学这个）

在浏览器里找到真实的数据请求：

1. 打开目标网页，按 `F12` 打开开发者工具
2. 切换到 **Network** 标签页
3. 刷新页面，筛选 **Fetch/XHR**
4. 找到返回 JSON 的请求，点击查看 **Response**
5. 如果是纯 JSON，直接用 `httpx` 调用这个 URL

```python
# 发现真实 API 是 https://api.example.com/items?page=1
import httpx

response = httpx.get(
    "https://api.example.com/items",
    params={"page": 1, "size": 20},
    headers={"Referer": "https://www.example.com/"}  # 有时需要带 Referer
)
data = response.json()
```

---

## 四、Playwright 核心用法

```bash
pip install playwright
playwright install chromium
```

### 4.1 基本流程

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)   # headless=False 可以看到浏览器窗口
    page = browser.new_page()

    page.goto("https://example.com")
    page.wait_for_selector(".data-item")         # 等待目标元素出现

    html = page.content()                        # 获取渲染后的完整 HTML
    browser.close()
```

### 4.2 等待策略（重点）

等待是动态抓取的核心，等不够数据没加载，等太久效率低：

```python
# 等待特定元素出现
page.wait_for_selector(".product-list")

# 等待网络请求静止（适合数据加载完成后）
page.wait_for_load_state("networkidle")

# 等待特定请求完成
with page.expect_response("**/api/items**") as resp:
    page.click(".load-more")
data = resp.value.json()

# 固定等待（不推荐，但调试时有用）
page.wait_for_timeout(2000)  # 毫秒
```

### 4.3 常用操作

```python
# 点击
page.click(".load-more-btn")
page.click("text=下一页")         # 按文字内容点击

# 填写表单
page.fill("input[name='q']", "Python")
page.press("input[name='q']", "Enter")

# 滚动（触发懒加载）
page.evaluate("window.scrollTo(0, document.body.scrollHeight)")

# 直接提取数据（不用 BeautifulSoup）
title = page.locator("h1").text_content()
all_titles = page.locator(".title").all_text_contents()   # 返回列表

# 截图（调试利器！）
page.screenshot(path="debug.png", full_page=True)
```

### 4.4 设置 User-Agent 和 Cookie

```python
context = browser.new_context(
    user_agent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
               "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    viewport={"width": 1920, "height": 1080},
)
page = context.new_page()

# 设置 Cookie（登录后的会话）
context.add_cookies([
    {"name": "session", "value": "xxx", "domain": "example.com", "path": "/"}
])
```

---

## 五、实操案例：抓取 JS 渲染的名言网站

`quotes.toscrape.com/js/` 是 JS 渲染版，静态 `httpx` 抓不到数据。

新建 `day09/dynamic_scraper.py`：

```python
import json
import time
import logging
from pathlib import Path
from dataclasses import dataclass, asdict
from bs4 import BeautifulSoup
from playwright.sync_api import sync_playwright, Page

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s",
                    datefmt="%H:%M:%S")
logger = logging.getLogger(__name__)


@dataclass
class Quote:
    text: str
    author: str
    tags: list[str]


def parse_quotes(html: str) -> list[Quote]:
    soup = BeautifulSoup(html, "lxml")
    quotes = []
    for q in soup.select(".quote"):
        text = q.select_one(".text").get_text(strip=True).strip('\u201c\u201d"')
        author = q.select_one(".author").get_text(strip=True)
        tags = [t.get_text(strip=True) for t in q.select(".tag")]
        quotes.append(Quote(text=text, author=author, tags=tags))
    return quotes


def has_next_page(page: Page) -> bool:
    return page.locator("li.next").count() > 0


def scrape_all_pages(max_pages: int = 5) -> list[Quote]:
    all_quotes: list[Quote] = []

    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        context = browser.new_context(
            user_agent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                       "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
        )
        page = context.new_page()

        url = "https://quotes.toscrape.com/js/page/1/"

        for page_num in range(1, max_pages + 1):
            logger.info("抓取第 %d 页", page_num)
            page.goto(url, wait_until="networkidle")

            # 等待名言内容加载
            page.wait_for_selector(".quote", timeout=15000)

            # 截图（调试用，正式运行可删除）
            # page.screenshot(path=f"debug_page{page_num}.png")

            quotes = parse_quotes(page.content())
            all_quotes.extend(quotes)
            logger.info("  获取 %d 条，累计 %d 条", len(quotes), len(all_quotes))

            if not has_next_page(page):
                logger.info("已到最后一页")
                break

            # 点击"下一页"
            page.click("li.next a")
            page.wait_for_selector(".quote", timeout=15000)
            url = page.url   # 更新当前 URL

            time.sleep(0.5)

        browser.close()

    return all_quotes


def main() -> None:
    quotes = scrape_all_pages(max_pages=5)

    # 保存为 JSON
    output = Path("data/quotes.json")
    output.parent.mkdir(exist_ok=True)
    output.write_text(
        json.dumps([asdict(q) for q in quotes], ensure_ascii=False, indent=2),
        encoding="utf-8"
    )

    # 统计
    print(f"\n共抓取 {len(quotes)} 条名言")

    from collections import Counter
    author_counts = Counter(q.author for q in quotes)
    print("\n名言最多的作者 Top 5：")
    for author, count in author_counts.most_common(5):
        print(f"  {author}: {count} 条")

    all_tags = [tag for q in quotes for tag in q.tags]
    tag_counts = Counter(all_tags)
    print("\n最热门标签 Top 5：")
    for tag, count in tag_counts.most_common(5):
        print(f"  #{tag}: {count} 次")

    logger.info("数据已保存到 %s", output)


if __name__ == "__main__":
    main()
```

### 运行

```bash
cd day09
pip install playwright httpx beautifulsoup4 lxml
playwright install chromium
mkdir -p data
python3 dynamic_scraper.py
```

### 对比验证（理解 JS 渲染差异）

```python
# 验证：httpx 直接请求 JS 版，拿不到数据
import httpx
from bs4 import BeautifulSoup

resp = httpx.get("https://quotes.toscrape.com/js/page/1/")
soup = BeautifulSoup(resp.text, "lxml")
print(len(soup.select(".quote")))   # 输出 0 ← 静态请求抓不到
```

---

## 六、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `wait_for_selector()` | 等待 JS 渲染完成再操作 |
| `wait_until="networkidle"` | 等待网络请求静止后再解析 |
| `page.click()` | 模拟点击翻页 |
| `page.content()` | 获取渲染后的 HTML |
| `page.screenshot()` | 调试利器，看到浏览器看到的内容 |
| `Counter` | Day 04 知识复用，统计频次 |

---

## 七、今日 Checklist

- [ ] 安装 Playwright 和 Chromium 成功
- [ ] `dynamic_scraper.py` 运行，`data/quotes.json` 正确生成
- [ ] 运行"对比验证"代码，确认静态请求确实拿不到数据
- [ ] 用 `headless=False` 运行一次，观察浏览器自动操作的过程
- [ ] git commit：`git add . && git commit -m "day09: playwright dynamic scraper"`

> 📅 最后更新：2026-04-06
