# Day 08 — 静态页面抓取

> 目标：掌握 HTML 解析与 CSS 选择器，用 BeautifulSoup 完成真实爬虫，理解反爬应对策略，输出结构化数据。

---

## 一、抓取原理

```
你的脚本
  │  httpx.get(url) → HTTP GET 请求
  ▼
服务器返回 HTML 文本
  │
  ▼  BeautifulSoup 解析 DOM 树
提取目标数据（书名、价格、链接……）
  │
  ├─→ CSV（表格工具打开）
  └─→ JSON（程序处理）/ SQLite（持久存储）
```

**判断页面是否静态的方法**：
- 右键 → "查看页面源代码"（不是 Inspect），若目标数据已在 HTML 里 → 静态，适合今天的方法
- 若源代码里只有 `<div id="root"></div>` 或空 `<body>` → 动态，需要 Playwright（Day 09）

---

## 二、HTML 结构速览

```html
<!DOCTYPE html>
<html lang="zh">
<head>
  <title>图书列表</title>
</head>
<body>
  <div id="main" class="container">
    <article class="product_pod" data-id="42">
      <div class="thumbnail">
        <img src="/images/cover.jpg" alt="封面">
      </div>
      <h3>
        <a href="/catalogue/book.html" title="完整书名">简短书名...</a>
      </h3>
      <p class="price_color">£51.77</p>
      <p class="star-rating Three"></p>
      <p class="instock availability">
        <i class="icon-ok"></i> In stock
      </p>
    </article>
  </div>
</body>
</html>
```

**在浏览器中定位元素**：右键目标文字 → 检查（Inspect）→ 查看对应 HTML 标签、class、属性。

---

## 三、CSS 选择器速查

| 选择器 | 含义 | 示例 |
|--------|------|------|
| `div` | 所有 div 标签 | `soup.select("div")` |
| `.class` | class 包含指定值 | `.price_color` |
| `#id` | id 精确匹配 | `#main` |
| `div.pod` | div 且 class=pod | `article.product_pod` |
| `div .title` | div **后代**的 .title（任意层级） | `article .price_color` |
| `div > h3` | div **直接子级** h3 | `ul > li` |
| `a[href]` | 有 href 属性的 a | |
| `a[href^="/catalogue"]` | href 以 /catalogue 开头 | |
| `a[href$=".html"]` | href 以 .html 结尾 | |
| `a[href*="book"]` | href 包含 book | |
| `li:first-child` | 父元素的第一个 li | |
| `li:last-child` | 父元素的最后一个 li | |
| `li:nth-child(2)` | 父元素的第 2 个子元素 | |
| `h1, h2, h3` | 多个选择器（OR） | |

---

## 四、BeautifulSoup 核心用法

```bash
pip install httpx beautifulsoup4 lxml
```

```python
import httpx
from bs4 import BeautifulSoup

html = httpx.get("https://books.toscrape.com").text
soup = BeautifulSoup(html, "lxml")   # lxml 解析器速度最快

# ── 查找元素 ──────────────────────────────────

# select_one：CSS 选择器，返回第一个匹配（或 None）
tag = soup.select_one("article.product_pod h3 a")

# select：CSS 选择器，返回所有匹配（列表，可能为空）
tags = soup.select("article.product_pod")

# find：标签名 + 属性过滤（等价于 select）
tag = soup.find("p", class_="price_color")
tags = soup.find_all("article", class_="product_pod")
tags = soup.find_all("a", href=True)  # 所有有 href 的 a

# ── 提取内容 ──────────────────────────────────

tag.get_text()               # 标签内所有文本（含子标签文本）
tag.get_text(strip=True)     # 去除首尾空白
tag.get_text(separator=" ")  # 子标签文本用空格分隔

tag["href"]                  # 获取属性（KeyError if missing）
tag.get("href", "")          # 安全获取，不存在返回默认值
tag.attrs                    # 所有属性字典：{'class': ['price_color'], ...}

# ── 导航 ──────────────────────────────────────

tag.parent                   # 父节点
tag.find_parent("div")       # 向上找第一个 div 祖先

list(tag.children)           # 直接子节点（含文本节点）
list(tag.descendants)        # 所有后代节点

tag.next_sibling             # 下一个兄弟节点（可能是文本节点）
tag.find_next_sibling("p")   # 下一个 p 兄弟标签

# ── 常用提取技巧 ──────────────────────────────

# 提取所有链接
links = [(a.get_text(strip=True), a["href"]) for a in soup.select("a[href]")]

# 提取表格数据
rows = []
for tr in soup.select("table tr"):
    cells = [td.get_text(strip=True) for td in tr.select("td, th")]
    if cells:
        rows.append(cells)

# 提取所有图片 src
imgs = [img.get("src", "") for img in soup.select("img[src]")]
```

---

## 五、robots.txt — 爬虫道德规范

在爬取前检查网站是否允许抓取：

```python
from urllib.robotparser import RobotFileParser
from urllib.parse import urljoin

def can_fetch(base_url: str, path: str, user_agent: str = "*") -> bool:
    """检查 robots.txt 是否允许抓取指定路径"""
    rp = RobotFileParser()
    rp.set_url(urljoin(base_url, "/robots.txt"))
    try:
        rp.read()
        return rp.can_fetch(user_agent, urljoin(base_url, path))
    except Exception:
        return True  # 读取失败则默认允许

# 使用
if can_fetch("https://books.toscrape.com", "/catalogue/"):
    print("允许抓取")
else:
    print("robots.txt 禁止，停止爬取")
```

**robots.txt 示例**：

```
User-agent: *
Disallow: /admin/
Disallow: /private/
Allow: /catalogue/

Crawl-delay: 1    # 建议爬取间隔（秒）
```

> `books.toscrape.com` 是专为练习设计的，robots.txt 全部允许。

---

## 六、反爬策略与应对

### 6.1 设置请求头（最基本）

```python
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Accept-Encoding": "gzip, deflate, br",
    "Referer": "https://books.toscrape.com/",  # 模拟从网站内部跳转
}
```

### 6.2 随机延时（避免触发限速）

```python
import time, random

def polite_sleep(min_s: float = 1.0, max_s: float = 3.0) -> None:
    delay = random.uniform(min_s, max_s)
    time.sleep(delay)
```

### 6.3 失败重试（指数退避）

```python
from functools import wraps
import time

def retry(times: int = 3, delay: float = 1.0, backoff: float = 2.0):
    """指数退避重试装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            current_delay = delay
            for attempt in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == times:
                        raise
                    time.sleep(current_delay)
                    current_delay *= backoff  # 每次翻倍：1s → 2s → 4s
        return wrapper
    return decorator
```

### 6.4 httpx Client（复用连接 + Session）

```python
import httpx

# 使用 Client 复用 TCP 连接，并自动处理 Cookie
with httpx.Client(headers=HEADERS, timeout=15, follow_redirects=True) as client:
    resp1 = client.get("https://books.toscrape.com/catalogue/page-1.html")
    resp2 = client.get("https://books.toscrape.com/catalogue/page-2.html")
    # 两次请求复用同一个 TCP 连接，Cookie 自动传递
```

---

## 七、数据清洗（结合 Day 04 regex）

从 HTML 提取的数据通常需要清洗：

```python
import re

def clean_price(raw: str) -> float:
    """'£51.77' → 51.77"""
    match = re.search(r"[\d.]+", raw)
    return float(match.group()) if match else 0.0

def clean_text(raw: str) -> str:
    """去除多余空白和不可见字符"""
    return re.sub(r"\s+", " ", raw).strip()

def extract_rating(star_classes: list[str]) -> int:
    """['star-rating', 'Three'] → 3"""
    star_map = {"One": 1, "Two": 2, "Three": 3, "Four": 4, "Five": 5}
    for cls in star_classes:
        if cls in star_map:
            return star_map[cls]
    return 0

def normalize_url(base: str, relative: str) -> str:
    """相对路径 → 绝对路径"""
    from urllib.parse import urljoin
    return urljoin(base, relative)

# 测试
print(clean_price("Â£51.77"))      # 51.77
print(clean_price("Price: $9.99")) # 9.99
print(clean_text("  Hello   World  \n"))  # "Hello World"
```

---

## 八、翻页处理

```python
from bs4 import BeautifulSoup

def get_next_url(soup: BeautifulSoup, current_url: str) -> str | None:
    """从页面中提取下一页 URL"""
    next_btn = soup.select_one("li.next a")
    if not next_btn:
        return None
    from urllib.parse import urljoin
    return urljoin(current_url, next_btn["href"])

def get_all_page_urls(start_url: str, max_pages: int = 50) -> list[str]:
    """预先收集所有分页 URL（有些网站分页 URL 有规律）"""
    # 方式1：规律 URL（如 ?page=N 或 /page-N.html）
    import re
    urls = []
    for i in range(1, max_pages + 1):
        url = re.sub(r"page-\d+", f"page-{i}", start_url)
        urls.append(url)
    return urls

    # 方式2：跟随"下一页"链接（见主流程中的翻页逻辑）
```

---

## 九、实操案例：书单爬虫（全站 + 双格式输出）

### 目标

抓取 `books.toscrape.com` 前 N 页，提取书名/价格/星级/库存/链接，同时输出 CSV 和 JSON。

新建 `book_scraper.py`：

```python
import csv
import json
import time
import random
import logging
import httpx
from bs4 import BeautifulSoup
from dataclasses import dataclass, fields, astuple, asdict
from pathlib import Path
from functools import wraps
from urllib.parse import urljoin
from urllib.robotparser import RobotFileParser
import re

# ── 日志配置 ──────────────────────────────────
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    datefmt="%H:%M:%S",
)
logger = logging.getLogger(__name__)

# ── 常量 ──────────────────────────────────────
BASE_URL = "https://books.toscrape.com"
START_URL = f"{BASE_URL}/catalogue/page-1.html"
HEADERS = {
    "User-Agent": "Mozilla/5.0 (compatible; BookScraper/1.0)",
    "Accept-Language": "en-US,en;q=0.9",
}
STAR_MAP = {"One": 1, "Two": 2, "Three": 3, "Four": 4, "Five": 5}


# ── 数据模型 ──────────────────────────────────
@dataclass
class Book:
    title: str
    price: float
    stars: int
    in_stock: bool
    category: str
    url: str


# ── 装饰器：重试 ──────────────────────────────
def retry(times: int = 3, delay: float = 1.0):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == times:
                        raise
                    wait = delay * attempt
                    logger.warning("第 %d 次失败，%.1fs 后重试: %s", attempt, wait, e)
                    time.sleep(wait)
        return wrapper
    return decorator


# ── 数据清洗函数 ──────────────────────────────
def clean_price(raw: str) -> float:
    """'£51.77' → 51.77"""
    match = re.search(r"[\d.]+", raw)
    return float(match.group()) if match else 0.0


def extract_stars(classes: list[str]) -> int:
    """['star-rating', 'Three'] → 3"""
    for cls in classes:
        if cls in STAR_MAP:
            return STAR_MAP[cls]
    return 0


# ── HTTP 请求 ─────────────────────────────────
@retry(times=3, delay=1.0)
def fetch(client: httpx.Client, url: str) -> BeautifulSoup:
    resp = client.get(url, timeout=15)
    resp.raise_for_status()
    return BeautifulSoup(resp.text, "lxml")


# ── 解析单页 ──────────────────────────────────
def parse_books(soup: BeautifulSoup, category: str = "Unknown") -> list[Book]:
    books = []
    for article in soup.select("article.product_pod"):
        a_tag = article.select_one("h3 a")
        title = a_tag["title"]
        url = urljoin(f"{BASE_URL}/catalogue/", a_tag["href"].lstrip("../"))
        price = clean_price(article.select_one(".price_color").get_text())
        stars = extract_stars(article.select_one(".star-rating")["class"])
        in_stock = "In stock" in article.select_one(".availability").get_text()
        books.append(Book(title=title, price=price, stars=stars,
                          in_stock=in_stock, category=category, url=url))
    return books


def get_next_url(soup: BeautifulSoup, current_url: str) -> str | None:
    btn = soup.select_one("li.next a")
    return urljoin(current_url, btn["href"]) if btn else None


# ── 输出：CSV + JSON ──────────────────────────
def save_csv(books: list[Book], path: str) -> None:
    Path(path).parent.mkdir(parents=True, exist_ok=True)
    with open(path, "w", newline="", encoding="utf-8-sig") as f:
        writer = csv.writer(f)
        writer.writerow([fd.name for fd in fields(Book)])
        writer.writerows(astuple(b) for b in books)
    logger.info("CSV 已保存: %s (%d 条)", path, len(books))


def save_json(books: list[Book], path: str) -> None:
    Path(path).parent.mkdir(parents=True, exist_ok=True)
    with open(path, "w", encoding="utf-8") as f:
        json.dump([asdict(b) for b in books], f, ensure_ascii=False, indent=2)
    logger.info("JSON 已保存: %s (%d 条)", path, len(books))


# ── 统计报告 ──────────────────────────────────
def print_summary(books: list[Book]) -> None:
    from collections import Counter
    print(f"\n{'='*50}")
    print(f"  共抓取: {len(books)} 本")
    print(f"  平均价格: £{sum(b.price for b in books) / len(books):.2f}")
    print(f"  平均星级: {sum(b.stars for b in books) / len(books):.1f}")
    print(f"  有货: {sum(b.in_stock for b in books)} 本")

    star_dist = Counter(b.stars for b in books)
    print("\n  星级分布:")
    for s in range(5, 0, -1):
        bar = "★" * star_dist.get(s, 0)
        print(f"    {s}星: {star_dist.get(s, 0):3d}  {bar}")

    print("\n  最贵 Top 5:")
    for b in sorted(books, key=lambda x: -x.price)[:5]:
        print(f"    £{b.price:.2f}  {b.title[:45]}")
    print(f"{'='*50}")


# ── 主流程 ────────────────────────────────────
def scrape(max_pages: int = 5, output_dir: str = "data") -> None:
    # 检查 robots.txt
    rp = RobotFileParser()
    rp.set_url(f"{BASE_URL}/robots.txt")
    try:
        rp.read()
    except Exception:
        pass  # 读不到就继续

    all_books: list[Book] = []
    url = START_URL
    page_num = 0

    with httpx.Client(headers=HEADERS, follow_redirects=True) as client:
        while url and page_num < max_pages:
            page_num += 1

            if not rp.can_fetch("*", url):
                logger.warning("robots.txt 不允许抓取: %s，停止", url)
                break

            logger.info("第 %d 页: %s", page_num, url)
            soup = fetch(client, url)

            # 获取当前分类（从面包屑）
            breadcrumb = soup.select("ul.breadcrumb li")
            category = breadcrumb[-2].get_text(strip=True) if len(breadcrumb) >= 2 else "All"

            books = parse_books(soup, category=category)
            all_books.extend(books)
            logger.info("  本页 %d 本，累计 %d 本", len(books), len(all_books))

            url = get_next_url(soup, url)

            if url and page_num < max_pages:
                delay = random.uniform(1.0, 2.0)
                logger.info("  等待 %.1fs...", delay)
                time.sleep(delay)

    # 保存两种格式
    save_csv(all_books, f"{output_dir}/books.csv")
    save_json(all_books, f"{output_dir}/books.json")

    print_summary(all_books)


if __name__ == "__main__":
    import sys
    pages = int(sys.argv[1]) if len(sys.argv) > 1 else 3
    scrape(max_pages=pages)
```

### 运行

```bash
pip install httpx beautifulsoup4 lxml
python3 book_scraper.py        # 默认 3 页
python3 book_scraper.py 5      # 抓取 5 页
```

预期输出：

```
10:00:01 [INFO] 第 1 页: https://books.toscrape.com/catalogue/page-1.html
10:00:02 [INFO]   本页 20 本，累计 20 本
10:00:02 [INFO]   等待 1.4s...
10:00:03 [INFO] 第 2 页: https://books.toscrape.com/catalogue/page-2.html
...
10:00:07 [INFO] CSV 已保存: data/books.csv (60 条)
10:00:07 [INFO] JSON 已保存: data/books.json (60 条)

==================================================
  共抓取: 60 本
  平均价格: £35.42
  平均星级: 3.0
  有货: 55 本

  星级分布:
    5星:  14  ██████████████
    4星:  13  █████████████
    3星:  10  ██████████
    2星:  12  ████████████
    1星:  11  ███████████

  最贵 Top 5:
    £59.69  Set Me Free
    ...
==================================================
```

---

## 十、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `soup.select` / `select_one` | CSS 选择器提取元素 |
| `tag["attr"]` / `.get_text()` | 属性和文本提取 |
| `robots.txt` 检查 | `RobotFileParser` 爬取前验证 |
| `clean_price()` / `extract_stars()` | regex 清洗 HTML 原始数据 |
| `urljoin` | 相对路径安全转绝对路径 |
| 翻页逻辑 | `get_next_url()` 解析下一页按钮 |
| `@retry` 装饰器 | 复用 Day06，指数退避重试 |
| `@dataclass` + `astuple/asdict` | Day03，数据建模 + 序列化 |
| `logging` | Day05，替代 print |
| CSV + JSON 双输出 | 覆盖不同使用场景 |
| `Counter` 统计 | Day04，星级分布计数 |

---

## 十一、今日 Checklist

- [ ] 安装依赖并成功运行爬虫
- [ ] `data/books.csv` 和 `data/books.json` 均正确生成
- [ ] 用浏览器"检查元素"找到一个新目标，修改选择器验证
- [ ] 理解 `select` vs `find_all` 的关系（同一功能，不同写法）
- [ ] 理解 `urljoin` 为什么比字符串拼接更安全
- [ ] 测试 `max_pages=1` 和 `max_pages=50`（全站）的差异
- [ ] git commit：`git add . && git commit -m "day08: books scraper"`

> 📅 最后更新：2026-04-07
