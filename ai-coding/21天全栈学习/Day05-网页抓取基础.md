# Day 05 — 网页抓取基础（静态页面）

> 目标：理解网页结构，掌握用 `httpx` + `BeautifulSoup` 抓取静态页面，应对基本反爬策略，完成第一个真实爬虫。

---

## 一、抓取的本质

打开浏览器访问网页时，浏览器做了两件事：
1. 发送 HTTP GET 请求，拿回 HTML 文本
2. 渲染 HTML，呈现可视界面

爬虫做的事一样，只是**跳过第 2 步**：直接拿 HTML，然后用代码解析出需要的数据。

```
你的脚本
  │  HTTP GET https://example.com
  ▼
服务器返回 HTML 文本
  │
  ▼
BeautifulSoup 解析 HTML → 提取数据
  │
  ▼
保存为 CSV / JSON / 数据库
```

---

## 二、HTML 结构速览

抓取前要看懂 HTML，知道目标数据在哪个标签里：

```html
<html>
  <body>
    <div class="book-list">
      <div class="book-item">
        <h2 class="title">三体</h2>
        <span class="rating">9.3</span>
        <a href="/book/2567698" class="link">详情</a>
      </div>
      <div class="book-item">
        <h2 class="title">活着</h2>
        <span class="rating">9.4</span>
        <a href="/book/4913064" class="link">详情</a>
      </div>
    </div>
  </body>
</html>
```

**在浏览器中查看 HTML：** 在任意网页右键 → 检查元素（Inspect），找到目标数据对应的标签和 class。

---

## 三、CSS 选择器

BeautifulSoup 使用 CSS 选择器定位元素：

| 选择器 | 含义 | 示例 |
|--------|------|------|
| `div` | 所有 div 标签 | `soup.select("div")` |
| `.book-item` | class="book-item" | `soup.select(".book-item")` |
| `#main` | id="main" | `soup.select("#main")` |
| `div.book-item` | div 且 class=book-item | `soup.select("div.book-item")` |
| `div .title` | div 内部的 .title | `soup.select("div .title")` |
| `a[href]` | 有 href 属性的 a 标签 | `soup.select("a[href]")` |

---

## 四、BeautifulSoup 核心用法

```bash
pip install httpx beautifulsoup4 lxml
```

```python
from bs4 import BeautifulSoup

html = """
<div class="book-list">
  <div class="book-item">
    <h2 class="title">三体</h2>
    <span class="rating">9.3</span>
    <a href="/book/2567698">详情</a>
  </div>
  <div class="book-item">
    <h2 class="title">活着</h2>
    <span class="rating">9.4</span>
    <a href="/book/4913064">详情</a>
  </div>
</div>
"""

soup = BeautifulSoup(html, "lxml")

# 查找单个元素
title = soup.select_one(".title")
print(title.text)           # 三体
print(title.get_text(strip=True))  # 去除首尾空白

# 查找所有匹配元素
items = soup.select(".book-item")
for item in items:
    title = item.select_one(".title").text
    rating = item.select_one(".rating").text
    link = item.select_one("a")["href"]   # 获取属性值
    print(f"{title} - {rating} - {link}")
```

---

## 五、反爬策略与应对

网站常用的反爬手段和对应方法：

### 5.1 User-Agent 检测

服务器检查请求头中的 `User-Agent`，若是 Python 默认值则拒绝：

```python
import httpx

headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/120.0.0.0 Safari/537.36",
    "Accept-Language": "zh-CN,zh;q=0.9",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
}

response = httpx.get("https://example.com", headers=headers)
```

### 5.2 请求频率限制

快速连续请求会触发封禁，加随机延时模拟人工访问：

```python
import time
import random

for url in urls:
    response = httpx.get(url, headers=headers)
    # 随机等待 1-3 秒
    time.sleep(random.uniform(1, 3))
```

### 5.3 请求失败重试

网络不稳定或被临时封禁时，自动重试：

```python
import httpx
import time

def fetch_with_retry(url: str, max_retries: int = 3) -> httpx.Response | None:
    for attempt in range(max_retries):
        try:
            response = httpx.get(url, headers=headers, timeout=10)
            response.raise_for_status()
            return response
        except (httpx.HTTPError, httpx.RequestError) as e:
            print(f"第 {attempt + 1} 次请求失败: {e}")
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)  # 指数退避：1s, 2s, 4s
    return None
```

---

## 六、实操案例：豆瓣 Top250 书单爬虫

### 目标

抓取豆瓣图书 Top250 前两页，提取书名、评分、评价人数，保存为 CSV。

> **注意**：豆瓣有反爬限制，以下示例展示完整爬虫结构，实际运行需要加适当延时和正确的 Headers。如果豆瓣返回 403，可换用其他静态页面练习（如 `books.toscrape.com`，专门用于练习爬虫的网站）。

### 完整代码

新建 `day05/book_scraper.py`：

```python
import csv
import time
import random
import httpx
from bs4 import BeautifulSoup
from dataclasses import dataclass, fields, astuple
from pathlib import Path


@dataclass
class Book:
    rank: int
    title: str
    author: str
    rating: str
    votes: str
    url: str


HEADERS = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/120.0.0.0 Safari/537.36",
    "Accept-Language": "zh-CN,zh;q=0.9",
}

BASE_URL = "https://book.douban.com/top250"


def fetch_page(url: str) -> BeautifulSoup | None:
    """获取页面并返回解析后的 BeautifulSoup 对象"""
    try:
        response = httpx.get(url, headers=HEADERS, timeout=15)
        response.raise_for_status()
        return BeautifulSoup(response.text, "lxml")
    except httpx.HTTPStatusError as e:
        print(f"HTTP 错误 {e.response.status_code}: {url}")
        return None
    except httpx.RequestError as e:
        print(f"请求错误: {e}")
        return None


def parse_books(soup: BeautifulSoup, start_rank: int) -> list[Book]:
    """从页面中解析书目信息"""
    books = []
    items = soup.select("tr.item")

    for i, item in enumerate(items):
        title_tag = item.select_one(".pl2 a")
        rating_tag = item.select_one(".rating_nums")
        votes_tag = item.select_one(".pl")
        author_tag = item.select_one(".pl2 .inq")  # 简介，部分页面有作者

        if not title_tag:
            continue

        title = title_tag.get("title") or title_tag.get_text(strip=True)
        url = title_tag.get("href", "")
        rating = rating_tag.get_text(strip=True) if rating_tag else "N/A"
        votes_text = votes_tag.get_text(strip=True) if votes_tag else ""
        # 提取括号内的数字，如 "(12345人评价)"
        votes = votes_text.strip("()").replace("人评价", "") if votes_text else "N/A"
        author = author_tag.get_text(strip=True) if author_tag else "N/A"

        books.append(Book(
            rank=start_rank + i,
            title=title,
            author=author,
            rating=rating,
            votes=votes,
            url=url
        ))

    return books


def save_to_csv(books: list[Book], filepath: str) -> None:
    """保存到 CSV 文件"""
    path = Path(filepath)
    path.parent.mkdir(exist_ok=True)

    with open(path, "w", newline="", encoding="utf-8-sig") as f:
        writer = csv.writer(f)
        # 写表头（从 dataclass 字段名自动获取）
        writer.writerow([f.name for f in fields(Book)])
        for book in books:
            writer.writerow(astuple(book))

    print(f"已保存 {len(books)} 条数据到 {filepath}")


def main(pages: int = 2) -> None:
    all_books: list[Book] = []

    for page in range(pages):
        start = page * 25
        url = f"{BASE_URL}?start={start}"
        print(f"正在抓取第 {page + 1} 页: {url}")

        soup = fetch_page(url)
        if soup is None:
            print(f"第 {page + 1} 页抓取失败，跳过")
            continue

        books = parse_books(soup, start_rank=start + 1)
        all_books.extend(books)
        print(f"  获取到 {len(books)} 本书")

        # 礼貌性延时，避免被封
        if page < pages - 1:
            delay = random.uniform(2, 4)
            print(f"  等待 {delay:.1f}s...")
            time.sleep(delay)

    if all_books:
        save_to_csv(all_books, "data/douban_top250.csv")
        print(f"\n共抓取 {len(all_books)} 本书")
        # 打印前5条预览
        print("\n预览（前5条）：")
        for book in all_books[:5]:
            print(f"  #{book.rank:3d} {book.title:<20} {book.rating}")
    else:
        print("未获取到数据")


if __name__ == "__main__":
    main(pages=2)
```

### 备用练习站点（无反爬）

如果豆瓣访问受限，用这个专门为练习爬虫设计的网站：

```python
# 把 BASE_URL 换成：
BASE_URL = "https://books.toscrape.com"
# 然后根据该网站 HTML 结构调整选择器
```

### 运行

```bash
mkdir -p day05/data
cd day05
pip install httpx beautifulsoup4 lxml
python3 book_scraper.py
```

### 代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `soup.select(".class")` | CSS 选择器查找所有匹配元素 |
| `soup.select_one(".class")` | 查找第一个匹配元素 |
| `tag.get_text(strip=True)` | 获取标签内文本，去除空白 |
| `tag["href"]` / `tag.get("href")` | 获取标签属性值 |
| 随机延时 | `time.sleep(random.uniform(1,3))` 模拟人工访问 |
| `dataclass` + `astuple` | 数据建模 + 方便写入 CSV |
| `csv.writer` | 写入 CSV 文件 |

---

## 七、今日 Checklist

- [ ] 理解 `select` 和 `select_one` 的区别
- [ ] 爬虫能运行并生成 CSV 文件
- [ ] 用浏览器「检查元素」找到目标数据对应的 CSS 选择器
- [ ] 理解为什么要加延时
- [ ] git commit：`git add . && git commit -m "day05: static page scraper"`

> 📅 最后更新：2026-04-06
