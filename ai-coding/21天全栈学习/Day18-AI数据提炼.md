# Day 18 — AI + 数据提炼

> 目标：掌握 Claude API 基础用法，用 AI 清洗和结构化爬取的原始数据，实现智能数据提取管道。

---

## 一、为什么用 AI 处理爬取数据

爬虫拿到的原始数据往往：
- 格式不统一（"¥59.00" / "59元" / "59"）
- 信息杂乱（一段文本里混着作者/出版社/年份）
- 需要分类打标签（给文章自动归类）
- 需要摘要（长文章提取关键信息）

传统方法：写大量正则表达式，脆弱且难维护。  
AI 方法：用自然语言描述需求，让 Claude 直接输出结构化 JSON。

---

## 二、Anthropic SDK 基础

```bash
pip install anthropic
```

```python
import anthropic

client = anthropic.Anthropic(api_key="your-api-key")
# 或从环境变量读取：client = anthropic.Anthropic()
# 需设置 ANTHROPIC_API_KEY 环境变量
```

### 2.1 基本调用

```python
message = client.messages.create(
    model="claude-opus-4-6",          # 最新最强模型
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "用一句话介绍 Python"}
    ]
)
print(message.content[0].text)
```

### 2.2 结构化输出（让 AI 返回 JSON）

```python
import json

def extract_structured(text: str) -> dict:
    message = client.messages.create(
        model="claude-opus-4-6",
        max_tokens=1024,
        system="你是数据提取助手。只返回 JSON，不要解释。",
        messages=[{
            "role": "user",
            "content": f"""
从以下文本中提取信息，返回 JSON 格式：
{{
  "title": "书名",
  "author": "作者",
  "price": 数字（不含单位）,
  "category": "分类",
  "summary": "一句话摘要"
}}

文本：{text}
"""
        }]
    )
    raw = message.content[0].text
    # 提取 JSON（有时 AI 会加代码块标记）
    raw = raw.strip().strip("```json").strip("```").strip()
    return json.loads(raw)
```

### 2.3 批量处理（控制费用）

```python
import time

def batch_extract(texts: list[str], delay: float = 0.5) -> list[dict]:
    results = []
    for i, text in enumerate(texts):
        print(f"处理第 {i+1}/{len(texts)} 条...")
        try:
            result = extract_structured(text)
            results.append({"ok": True, "data": result})
        except Exception as e:
            results.append({"ok": False, "error": str(e), "raw": text})
        time.sleep(delay)   # 避免超过 API 速率限制
    return results
```

---

## 三、System Prompt 工程

好的 System Prompt 让 AI 输出更稳定、更可控：

```python
SYSTEM_PROMPT = """
你是数据提取专家。

规则：
1. 只返回合法的 JSON，不要任何解释文字
2. 如果某字段无法确定，填 null
3. price 只保留数字（整数或浮点数），去掉货币符号和单位
4. category 从以下选项中选择：技术/文学/历史/科学/其他
5. summary 不超过 50 字
"""
```

---

## 四、实操案例：AI 数据提炼管道

### 目标

1. 读取爬取的原始网页文本
2. 用 Claude 提取结构化信息
3. 存入数据库

新建 `day18/ai_extractor.py`：

```python
import json
import time
import sqlite3
import logging
from pathlib import Path
from dataclasses import dataclass
import anthropic

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s",
                    datefmt="%H:%M:%S")
logger = logging.getLogger(__name__)

# ── 配置 ──────────────────────────────────────
MODEL = "claude-opus-4-6"
MAX_TOKENS = 512

SYSTEM_PROMPT = """
你是数据提取专家。只返回合法 JSON，不要任何解释。
如果信息不存在，对应字段填 null。
price 只保留数字，去掉单位。
sentiment 从 positive/neutral/negative 选一个。
"""

EXTRACT_TEMPLATE = """
从以下文本提取信息，返回 JSON：
{{
  "title": "标题",
  "author": "作者（无则null）",
  "price": 价格数字（无则null）,
  "category": "分类",
  "tags": ["标签1", "标签2"],
  "sentiment": "情感倾向",
  "summary": "30字以内摘要"
}}

文本：
{text}
"""


# ── 原始数据（模拟爬取结果）─────────────────────
RAW_TEXTS = [
    "《Python编程从入门到实践》作者：Eric Matthes，价格：￥89.00，适合初学者的Python入门书，内容由浅入深，包含大量实战项目。",
    "活着 余华著 定价：39元 茅盾文学奖获奖作品。通过福贵的一生，展现了中国半个世纪的历史变迁，充满了对生命的深刻思考。",
    "TypeScript Deep Dive - 免费在线书籍，作者 Basarat Ali Syed，全面介绍TS类型系统和高级特性，GitHub stars 18k+。",
    "三体 刘慈欣 中国科幻巅峰之作，雨果奖获奖。描述地球文明与三体文明的接触，探讨宇宙文明的生存法则。售价: 59",
]


# ── AI 提取 ───────────────────────────────────
@dataclass
class ExtractResult:
    raw_text: str
    extracted: dict | None
    error: str | None


def extract_one(client: anthropic.Anthropic, text: str) -> ExtractResult:
    try:
        message = client.messages.create(
            model=MODEL,
            max_tokens=MAX_TOKENS,
            system=SYSTEM_PROMPT,
            messages=[{"role": "user", "content": EXTRACT_TEMPLATE.format(text=text)}]
        )
        raw_json = message.content[0].text.strip().strip("```json").strip("```").strip()
        extracted = json.loads(raw_json)
        return ExtractResult(raw_text=text, extracted=extracted, error=None)
    except json.JSONDecodeError as e:
        logger.warning("JSON 解析失败: %s", e)
        return ExtractResult(raw_text=text, extracted=None, error=f"JSON解析失败: {e}")
    except Exception as e:
        logger.error("API 调用失败: %s", e)
        return ExtractResult(raw_text=text, extracted=None, error=str(e))


def batch_extract(texts: list[str], delay: float = 0.5) -> list[ExtractResult]:
    client = anthropic.Anthropic()
    results = []

    for i, text in enumerate(texts):
        logger.info("处理 %d/%d: %s...", i + 1, len(texts), text[:30])
        result = extract_one(client, text)

        if result.extracted:
            logger.info("  ✓ 提取成功: %s", result.extracted.get("title"))
        else:
            logger.warning("  ✗ 提取失败: %s", result.error)

        results.append(result)

        if i < len(texts) - 1:
            time.sleep(delay)

    return results


# ── 数据库存储 ────────────────────────────────
def save_results(results: list[ExtractResult], db_path: str = "data/extracted.db") -> None:
    Path(db_path).parent.mkdir(exist_ok=True)
    conn = sqlite3.connect(db_path)
    conn.execute("""
        CREATE TABLE IF NOT EXISTS extracted (
            id          INTEGER PRIMARY KEY AUTOINCREMENT,
            raw_text    TEXT,
            title       TEXT,
            author      TEXT,
            price       REAL,
            category    TEXT,
            tags        TEXT,
            sentiment   TEXT,
            summary     TEXT,
            extracted_at TEXT DEFAULT (datetime('now','localtime'))
        )
    """)

    for r in results:
        if not r.extracted:
            continue
        d = r.extracted
        conn.execute("""
            INSERT INTO extracted (raw_text, title, author, price, category, tags, sentiment, summary)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?)
        """, (
            r.raw_text,
            d.get("title"), d.get("author"), d.get("price"),
            d.get("category"), json.dumps(d.get("tags", []), ensure_ascii=False),
            d.get("sentiment"), d.get("summary"),
        ))

    conn.commit()
    count = conn.execute("SELECT COUNT(*) FROM extracted").fetchone()[0]
    conn.close()
    logger.info("数据库共 %d 条记录", count)


def main() -> None:
    print("=== AI 数据提炼管道 ===\n")
    results = batch_extract(RAW_TEXTS)

    # 打印结果
    ok = [r for r in results if r.extracted]
    print(f"\n成功提取: {len(ok)}/{len(results)} 条\n")
    for r in ok:
        d = r.extracted
        print(f"  📖 {d.get('title')}")
        print(f"     作者: {d.get('author')}  价格: {d.get('price')}  情感: {d.get('sentiment')}")
        print(f"     摘要: {d.get('summary')}")
        print()

    save_results(results)


if __name__ == "__main__":
    main()
```

### 运行

```bash
# 需要设置 API Key
export ANTHROPIC_API_KEY="your-key-here"
# 或在 .env 文件中设置，用 python-dotenv 加载

pip install anthropic
python3 ai_extractor.py
```

---

## 五、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `anthropic.Anthropic()` | 从环境变量自动读取 API Key |
| System Prompt | 约束输出格式和行为 |
| JSON 解析容错 | `strip("```json")` 去除代码块标记 |
| 批量处理限速 | `time.sleep(delay)` 避免超过速率限制 |
| 数据管道 | 原始文本 → AI 提取 → 结构化存储 |

---

## 六、今日 Checklist

- [ ] 设置 `ANTHROPIC_API_KEY` 环境变量
- [ ] `ai_extractor.py` 成功调用 API 并输出结果
- [ ] `data/extracted.db` 中有正确存储的数据
- [ ] 修改 `RAW_TEXTS` 加入自己的测试文本
- [ ] git commit：`git add . && git commit -m "day18: ai data extractor"`

> 📅 最后更新：2026-04-06
