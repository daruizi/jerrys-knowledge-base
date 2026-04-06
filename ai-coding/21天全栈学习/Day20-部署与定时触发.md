# Day 20 — 部署 & 定时触发

> 目标：把前端部署到 Vercel，用 GitHub Actions 定时运行爬虫，实现无人值守的自动化数据收集。

---

## 一、部署架构

```
GitHub Actions（cron 定时触发）
        ↓
  运行 Python 爬虫
        ↓
  数据写入云数据库（或推送到 GitHub）
        ↓
  Vercel（Next.js 前端）← 用户访问
        ↓
  Railway / Render（Python API）
```

---

## 二、Vercel 部署前端

### 2.1 安装 Vercel CLI

```bash
npm install -g vercel
vercel login
```

### 2.2 配置环境变量

在 Vercel Dashboard → Settings → Environment Variables 添加：

```
BACKEND_URL = https://your-python-api.railway.app
```

### 2.3 部署

```bash
cd day20/frontend
vercel          # 按提示操作，首次部署
vercel --prod   # 之后每次发布到生产环境
```

**`vercel.json`**（可选，配置重写规则）：

```json
{
  "rewrites": [
    { "source": "/api/backend/:path*", "destination": "https://your-api.railway.app/:path*" }
  ]
}
```

---

## 三、Railway 部署 Python API

Railway 是最简单的 Python 后端托管平台：

### 3.1 准备文件

**`Procfile`**（告诉 Railway 如何启动）：

```
web: uvicorn api:app --host 0.0.0.0 --port $PORT
```

**`requirements.txt`**：

```
fastapi
uvicorn
httpx
beautifulsoup4
lxml
anthropic
```

### 3.2 部署

```bash
npm install -g @railway/cli
railway login
railway init
railway up
```

---

## 四、GitHub Actions — 定时运行爬虫

### 4.1 基础结构

```yaml
# .github/workflows/scraper.yml
name: 定时爬虫

on:
  schedule:
    - cron: '0 2 * * *'    # 每天 UTC 02:00（北京时间 10:00）
  workflow_dispatch:         # 允许手动触发

jobs:
  scrape:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install -r requirements.txt
      - run: python pipeline.py scrape
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### 4.2 完整工作流：爬取 + AI 处理 + 推送数据

```yaml
# .github/workflows/auto_pipeline.yml
name: 自动化数据管道

on:
  schedule:
    - cron: '0 2 * * *'   # 每天 UTC 02:00
  workflow_dispatch:
    inputs:
      pages:
        description: '抓取页数'
        default: '3'
        required: false

jobs:
  pipeline:
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      # 1. 检出代码
      - name: Checkout
        uses: actions/checkout@v4

      # 2. 安装 Python
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: 'pip'

      # 3. 安装依赖
      - name: Install dependencies
        run: |
          cd day19/backend
          pip install -r requirements.txt

      # 4. 运行爬虫
      - name: Run scraper
        run: |
          cd day19/backend
          python pipeline.py scrape
        env:
          SCRAPE_PAGES: ${{ github.event.inputs.pages || '3' }}

      # 5. AI 提炼（可选，消耗 API 费用）
      - name: AI extraction
        run: |
          cd day19/backend
          python pipeline.py process
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

      # 6. 把数据文件提交回仓库（静态数据方案，无需数据库服务器）
      - name: Commit data
        run: |
          git config user.name "GitHub Actions Bot"
          git config user.email "actions@github.com"
          git add day19/backend/data/
          git diff --staged --quiet || git commit -m "chore: auto update scraped data $(date +%Y-%m-%d)"
          git push
```

### 4.3 设置 Secrets

在 GitHub → Settings → Secrets and variables → Actions → New repository secret：

```
ANTHROPIC_API_KEY = sk-ant-...
```

---

## 五、静态数据方案（简化部署）

如果不想维护 Python API 服务器，可以把爬取的数据保存为 JSON 文件，直接部署到 Vercel：

```python
# pipeline.py 最后一步：导出为 JSON
import json
from pathlib import Path

def export_json(db_path: str, output: str = "public/data/items.json") -> None:
    conn = sqlite3.connect(db_path)
    rows = conn.execute("SELECT * FROM items ORDER BY id DESC LIMIT 1000").fetchall()
    items = [dict(zip([d[0] for d in conn.execute("PRAGMA table_info(items)").fetchall()], row)) for row in rows]
    conn.close()

    Path(output).parent.mkdir(parents=True, exist_ok=True)
    Path(output).write_text(json.dumps(items, ensure_ascii=False, indent=2))
    print(f"导出 {len(items)} 条到 {output}")
```

前端直接读取 JSON 文件（无需 API）：

```typescript
// src/app/items/page.tsx（Server Component）
export default async function ItemsPage() {
  const res = await fetch('/data/items.json')
  const items = await res.json()
  return <ItemsTable initialItems={items} />
}
```

---

## 六、Cron 语法速查

```
┌─ 分钟 (0-59)
│  ┌─ 小时 (0-23, UTC)
│  │  ┌─ 日 (1-31)
│  │  │  ┌─ 月 (1-12)
│  │  │  │  ┌─ 星期 (0-7, 0和7都是周日)
│  │  │  │  │
*  *  *  *  *

0 2 * * *     每天 UTC 02:00（北京 10:00）
0 */6 * * *   每 6 小时
0 8 * * 1     每周一 UTC 08:00
0 0 1 * *     每月 1 号 00:00
```

---

## 七、今日 Checklist

- [ ] 创建 `.github/workflows/scraper.yml`
- [ ] 在 GitHub Secrets 设置 `ANTHROPIC_API_KEY`
- [ ] 手动触发 workflow，验证运行成功
- [ ] （可选）把 Next.js 前端部署到 Vercel
- [ ] git commit：`git add . && git commit -m "day20: deploy and github actions cron"`

> 📅 最后更新：2026-04-06
