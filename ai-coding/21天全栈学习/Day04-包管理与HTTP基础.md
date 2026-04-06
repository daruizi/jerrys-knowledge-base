# Day 04 — 包管理 & HTTP 基础

> 目标：掌握 Python 包管理体系，理解 HTTP 协议，用 `httpx` 调用真实 API。

---

## 一、包管理

### 1.1 pip — 包安装工具

```bash
pip install httpx           # 安装包
pip install httpx==0.27.0   # 安装指定版本
pip uninstall httpx         # 卸载
pip list                    # 列出已安装的包
pip show httpx              # 查看某个包的详情
pip install --upgrade httpx # 升级到最新版
```

**始终在虚拟环境中安装包**，避免污染全局环境。

### 1.2 `requirements.txt` — 依赖清单

用于记录项目依赖，方便他人复现环境：

```bash
# 导出当前环境所有依赖
pip freeze > requirements.txt

# 根据 requirements.txt 安装依赖
pip install -r requirements.txt
```

`requirements.txt` 内容示例：

```
httpx==0.27.0
pydantic==2.7.0
python-dotenv==1.0.1
```

### 1.3 `pyproject.toml` — 现代项目配置（了解即可）

比 `requirements.txt` 更完整，是新项目的推荐格式：

```toml
[project]
name = "my-project"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "httpx>=0.27.0",
    "pydantic>=2.7.0",
]
```

---

## 二、HTTP 协议基础

理解 HTTP 是前后端开发的地基，不理解 HTTP 就看不懂 API 报错。

### 2.1 请求方法（Method）

| 方法 | 语义 | 示例 |
|------|------|------|
| `GET` | 获取资源 | 获取用户列表 |
| `POST` | 创建资源 | 新建一篇文章 |
| `PUT` | 全量更新资源 | 更新整个用户信息 |
| `PATCH` | 部分更新资源 | 只修改用户的邮箱 |
| `DELETE` | 删除资源 | 删除一篇文章 |

### 2.2 状态码（Status Code）

| 范围 | 含义 | 常见码 |
|------|------|--------|
| 2xx | 成功 | `200 OK`、`201 Created`、`204 No Content` |
| 3xx | 重定向 | `301 Moved Permanently`、`302 Found` |
| 4xx | 客户端错误 | `400 Bad Request`、`401 Unauthorized`、`403 Forbidden`、`404 Not Found` |
| 5xx | 服务器错误 | `500 Internal Server Error`、`503 Service Unavailable` |

### 2.3 请求与响应结构

**请求（Request）：**

```
GET /users/123 HTTP/1.1
Host: api.example.com
Authorization: Bearer <token>
Content-Type: application/json

{"name": "Alice"}     ← Body（GET 请求通常无 body）
```

**响应（Response）：**

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com"
}
```

### 2.4 URL 结构

```
https://api.github.com/users/octocat/repos?per_page=10&page=1
│       │               │              │   └─ Query 参数
│       │               │              └──── Path 参数
│       │               └──────────────────── 路径
│       └──────────────────────────────────── 主机名
└──────────────────────────────────────────── 协议
```

---

## 三、httpx — 现代 HTTP 客户端

`httpx` 是 Python 最推荐的 HTTP 客户端，支持同步和异步两种模式。

### 3.1 安装

```bash
pip install httpx
```

### 3.2 基本用法

```python
import httpx

# GET 请求
response = httpx.get("https://httpbin.org/get")
print(response.status_code)   # 200
print(response.json())        # 将响应体解析为字典

# 带参数的 GET
response = httpx.get(
    "https://api.github.com/users/octocat/repos",
    params={"per_page": 5, "sort": "updated"}
)

# POST 请求
response = httpx.post(
    "https://httpbin.org/post",
    json={"name": "Alice", "age": 25}    # 自动设置 Content-Type: application/json
)

# 带请求头
response = httpx.get(
    "https://api.github.com/user",
    headers={"Authorization": "Bearer YOUR_TOKEN"}
)
```

### 3.3 异常处理

```python
import httpx

try:
    response = httpx.get("https://api.example.com/data", timeout=10.0)
    response.raise_for_status()  # 4xx/5xx 时抛出异常
    data = response.json()
except httpx.TimeoutException:
    print("请求超时")
except httpx.HTTPStatusError as e:
    print(f"HTTP 错误: {e.response.status_code}")
except httpx.RequestError as e:
    print(f"请求错误: {e}")
```

### 3.4 环境变量管理（存放 Token）

**永远不要把 API Token 硬编码在代码里！** 使用 `.env` 文件：

```bash
pip install python-dotenv
```

创建 `.env` 文件（**加入 .gitignore，不要提交到 git**）：

```
GITHUB_TOKEN=ghp_your_token_here
```

在代码中读取：

```python
from dotenv import load_dotenv
import os

load_dotenv()  # 读取 .env 文件

token = os.getenv("GITHUB_TOKEN")
if not token:
    raise ValueError("缺少 GITHUB_TOKEN 环境变量")
```

---

## 四、实操案例：GitHub 仓库查询工具

### 目标

输入 GitHub 用户名，查询其公开仓库，按 Star 数排序展示。（**无需 Token，公开 API 可直接调用**）

### 完整代码

新建 `github_fetcher.py`：

```python
import sys
import httpx

BASE_URL = "https://api.github.com"


def get_user_info(username: str) -> dict:
    """获取用户基本信息"""
    url = f"{BASE_URL}/users/{username}"
    response = httpx.get(url, headers={"Accept": "application/vnd.github+json"})
    response.raise_for_status()
    return response.json()


def get_repos(username: str, max_count: int = 20) -> list[dict]:
    """获取用户的公开仓库列表"""
    url = f"{BASE_URL}/users/{username}/repos"
    response = httpx.get(
        url,
        params={"per_page": max_count, "sort": "updated"},
        headers={"Accept": "application/vnd.github+json"},
    )
    response.raise_for_status()
    return response.json()


def print_user(user: dict) -> None:
    print(f"\n👤 {user['login']}")
    if user.get("name"):
        print(f"   姓名: {user['name']}")
    if user.get("bio"):
        print(f"   简介: {user['bio']}")
    print(f"   粉丝: {user['followers']}  关注: {user['following']}")
    print(f"   公开仓库: {user['public_repos']}")


def print_repos(repos: list[dict]) -> None:
    # 按 Star 数降序排列
    sorted_repos = sorted(repos, key=lambda r: r["stargazers_count"], reverse=True)

    print(f"\n📦 仓库列表（Top {len(sorted_repos)}，按 Star 数排序）\n")
    print(f"  {'仓库名':<30} {'Stars':>6}  {'语言':<15} {'描述'}")
    print("  " + "-" * 75)

    for repo in sorted_repos:
        name = repo["name"][:29]
        stars = repo["stargazers_count"]
        lang = (repo.get("language") or "-")[:14]
        desc = (repo.get("description") or "")[:40]
        print(f"  {name:<30} {stars:>6}⭐  {lang:<15} {desc}")


def main() -> None:
    if len(sys.argv) < 2:
        print("用法: python3 github_fetcher.py <github用户名>")
        print("示例: python3 github_fetcher.py torvalds")
        sys.exit(1)

    username = sys.argv[1]
    print(f"正在查询 {username} 的 GitHub 信息...")

    try:
        user = get_user_info(username)
        repos = get_repos(username)

        print_user(user)
        print_repos(repos)

    except httpx.HTTPStatusError as e:
        if e.response.status_code == 404:
            print(f"用户 '{username}' 不存在")
        elif e.response.status_code == 403:
            print("API 请求频率超限，请稍后重试（或使用 Token）")
        else:
            print(f"HTTP 错误: {e.response.status_code}")
    except httpx.RequestError:
        print("网络连接失败，请检查网络")


if __name__ == "__main__":
    main()
```

### 运行

```bash
python3 github_fetcher.py torvalds
python3 github_fetcher.py daruizi
```

预期输出示例：

```
正在查询 torvalds 的 GitHub 信息...

👤 torvalds
   姓名: Linus Torvalds
   简介: Linux and git
   粉丝: 230000  关注: 0
   公开仓库: 8

📦 仓库列表（Top 8，按 Star 数排序）

  仓库名                          Stars  语言            描述
  ---------------------------------------------------------------------------
  linux                          230000⭐  C               Linux kernel source tree
  ...
```

### 代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `httpx.get()` + `params` | 构建带查询参数的 GET 请求 |
| `response.raise_for_status()` | 自动处理 4xx/5xx 错误 |
| `response.json()` | 将 JSON 响应解析为字典/列表 |
| `sorted()` + `reverse=True` | 降序排列 |
| 分层函数设计 | 获取数据 / 格式化输出 / 入口逻辑 分离 |
| 环境变量（`.env`）| Token 不写死在代码里 |

---

## 五、今日 Checklist

- [ ] 理解 GET/POST/PUT/DELETE 的区别
- [ ] 理解 2xx/4xx/5xx 状态码的含义
- [ ] `github_fetcher.py` 能正常查询并输出
- [ ] 测试用户不存在时的错误提示
- [ ] git commit：`git add . && git commit -m "day04: github fetcher"`

> 📅 最后更新：2026-04-06
