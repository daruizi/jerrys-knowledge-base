# Day 08 — 静态页面抓取

> 目标：掌握 HTML 解析与 CSS 选择器，用 BeautifulSoup 完成第一个真实爬虫，学会应对基本反爬策略。

---

## 一、抓取原理

```
你的脚本
  │  httpx.get(url) → HTTP 请求
  ▼
服务器返回 HTML 文本
  │
  ▼  BeautifulSoup 解析
提取目标数据（书名、价格、链接……）
  │
  ▼
保存为 CSV / JSON / 数据库
```

**判断页面是否静态**：浏览器右键 → 查看页面源代码，若目标数据直接在 HTML 里就是静态页面，适合今天的方法。

---

## 二、HTML 结构速览

```html
<div class="product-list">
  <article class="product_pod">
    <h3><a href="/catalogue/book1.html" title="A Light in the Attic">A Light...</a></h3>
    <p class="price_color">£51.77</p>
    <p class="star-rating Three"></p>
    <p class="instock availability">In stock</p>
  </article>
</div>
```

**在浏览器中定位元素**：右键目标文字 → 检查（Inspect）→ 查看对应 HTML 标签和 class。

---

## 三、CSS 选择器速查

| 选择器 | 含义 | 例子 |
|--------|------|------|
| `div` | 所有 div 标签 | |
| `.class` | 指定 class | `.price_color` |
| `#id` | 指定 id | `#main` |
| `div.pod` | div 且 class=pod | |
| `div .title` | div **内部**的 .title | |
| `div > h3` | div **直接子级** h3 | |
| `a[href]` | 有 href 属性的 a | |
| `a[href^="/catalogue"]` | href 以 /catalogue 开头 | |

---

## 四、BeautifulSoup 核心用法

```bash
pip install httpx beautifulsoup4 lxml
```

```python
from bs4 import BeautifulSoup
import httpx

html = httpx.get("https://books.toscrape.com").text
soup = BeautifulSoup(html, "lxml")

# ── 查找元素 ──────────────────────────────
# 查找第一个匹配
tag = soup.select_one("h3 a")
tag = soup.find("h3")                         # 等价写法

# 查找所有匹配（返回列表）
tags = soup.select("article.product_pod")
tags = soup.find_all("article", class_="product_pod")  # 等价

# ── 提取内容 ──────────────────────────────
tag.get_text()                  # 标签内所有文本
tag.get_text(strip=True)        # 去除首尾空白
tag["href"]                     # 获取属性（KeyError if not exist）
tag.get("href", "")             # 安全获取属性

# ── 向上/向下导航 ─────────────────────────
tag.parent                      # 父节点
tag.find("span")                # 在 tag 内部再查找
list(tag.children)              # 直接子节点
```

---

## 五、反爬策略与应对

### 5.1 设置请求头

```python
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
}
```

### 5.2 随机延时

```python
import time, random

# 每次请求后等待 1~3 秒
time.sleep(random.uniform(1, 3))
```

### 5.3 失败重试（复用 Day 06 装饰器）

```python
from functools import wraps
import time

def retry(times=3, delay=1.0):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for i in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if i == times:
                        raise
                    time.sleep(delay * i)
        return wrapper
    return decorator
```

---

## 六、实操案例：books.toscrape.com 书单爬虫

`books.toscrape.com` 是专为练习爬虫设计的网站，无需担心封禁。

### 目标

抓取前 3 页书单，提取书名、价格、星级、链接，保存为 CSV。

新建 `day08/book_scraper.py`：

```python
import csv
import time
import random
import logging
import httpx
from bs4 import BeautifulSoup
from dataclasses import dataclass, fields, astuple
from pathlib import Path
from functools import wraps

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s",
                    datefmt="%H:%M:%S")
logger = logging.getLogger(__name__)

BASE_URL = "https://books.toscrape.com/catalogue"
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
}

# 星级文字 → 数字映射
STAR_MAP = {"One": 1, "Two": 2, "Three": 3, "Four": 4, "Five": 5}


@dataclass
class Book:
    title: str
    price: str
    stars: int
    in_stock: bool
    url: str


# ── 带重试的请求函数 ─────────────────────────
def retry(times=3, delay=1.0):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for i in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if i == times:
                        raise
                    logger.warning("第 %d 次失败，%.1fs 后重试: %s", i, delay * i, e)
                    time.sleep(delay * i)
        return wrapper
    return decorator


@retry(times=3, delay=1.0)
def fetch_page(url: str) -> BeautifulSoup:
    resp = httpx.get(url, headers=HEADERS, timeout=15)
    resp.raise_for_status()
    return BeautifulSoup(resp.text, "lxml")


# ── 解析单页 ──────────────────────────────────
def parse_page(soup: BeautifulSoup) -> list[Book]:
    books = []
    for article in soup.select("article.product_pod"):
        # 标题（完整标题在 title 属性里）
        title_tag = article.select_one("h3 a")
        title = title_tag["title"]
        href = title_tag["href"].lstrip("../")
        url = f"{BASE_URL}/{href}"

        # 价格
        price = article.select_one(".price_color").get_text(strip=True)

        # 星级（class 是 "star-rating Three" 这样的格式）
        star_class = article.select_one(".star-rating")["class"]
        stars = STAR_MAP.get(star_class[1], 0)

        # 库存
        in_stock = "In stock" in article.select_one(".availability").get_text()

        books.append(Book(title=title, price=price, stars=stars,
                          in_stock=in_stock, url=url))
    return books


# ── 获取下一页 URL ────────────────────────────
def get_next_url(soup: BeautifulSoup, current_url: str) -> str | None:
    next_btn = soup.select_one("li.next a")
    if not next_btn:
        return None
    # 相对路径转绝对路径
    current_dir = current_url.rsplit("/", 1)[0]
    return f"{current_dir}/{next_btn['href']}"


# ── 保存结果 ──────────────────────────────────
def save_csv(books: list[Book], filepath: str) -> None:
    Path(filepath).parent.mkdir(exist_ok=True)
    with open(filepath, "w", newline="", encoding="utf-8-sig") as f:
        writer = csv.writer(f)
        writer.writerow([field.name for field in fields(Book)])
        writer.writerows([astuple(b) for b in books])
    logger.info("已保存 %d 条数据到 %s", len(books), filepath)


# ── 主流程 ────────────────────────────────────
def scrape(max_pages: int = 3, output: str = "data/books.csv") -> None:
    all_books: list[Book] = []
    url = "https://books.toscrape.com/catalogue/page-1.html"

    for page_num in range(1, max_pages + 1):
        logger.info("抓取第 %d 页: %s", page_num, url)
        soup = fetch_page(url)

        books = parse_page(soup)
        all_books.extend(books)
        logger.info("  本页 %d 本书，累计 %d 本", len(books), len(all_books))

        next_url = get_next_url(soup, url)
        if not next_url or page_num >= max_pages:
            break
        url = next_url

        delay = random.uniform(1, 2.5)
        logger.info("  等待 %.1fs...", delay)
        time.sleep(delay)

    save_csv(all_books, output)

    # 简单统计
    print(f"\n共抓取 {len(all_books)} 本书")
    avg_stars = sum(b.stars for b in all_books) / len(all_books) if all_books else 0
    print(f"平均星级: {avg_stars:.1f}")
    five_star = [b for b in all_books if b.stars == 5]
    print(f"五星图书: {len(five_star)} 本")
    print("\n五星图书列表:")
    for b in five_star[:5]:
        print(f"  {b.title[:40]:<40} {b.price}")


if __name__ == "__main__":
    scrape(max_pages=3)
```

### 运行

```bash
mkdir -p day08/data
cd day08
pip install httpx beautifulsoup4 lxml
python3 book_scraper.py
```

---

## 七、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `select` / `select_one` | CSS 选择器查找元素 |
| `tag["attr"]` / `.get_text()` | 提取属性和文本 |
| `@retry` 装饰器 | 复用 Day 06 知识，网络失败自动重试 |
| `@dataclass` + `astuple` | Day 03 知识，数据建模 + CSV 写入 |
| `logging` | Day 05 知识，替代 print |
| 翻页逻辑 | `get_next_url()` 通过解析"下一页"按钮获取 URL |

---

## 八、今日 Checklist

- [ ] 安装依赖并成功运行爬虫
- [ ] `data/books.csv` 正确生成
- [ ] 用浏览器"检查元素"找到一个新目标，修改选择器验证
- [ ] 测试 `max_pages=1` 和 `max_pages=5` 的差异
- [ ] git commit：`git add . && git commit -m "day08: books scraper"`

> 📅 最后更新：2026-04-06
