# Day 05 — FastAPI 入门

> 目标：理解 Web 框架工作原理，用 FastAPI 搭建一个完整的 TODO REST API，并通过浏览器和 curl 验证。

---

## 一、Web 框架是什么？

当你在浏览器访问 `http://localhost:8000/todos` 时，发生了什么：

```
浏览器/curl
    │  HTTP GET /todos
    ▼
FastAPI（你写的代码）
    │  找到匹配的路由函数 get_todos()
    ▼
返回 JSON 响应
    │  {"todos": [...]}
    ▼
浏览器显示结果
```

Web 框架做的事：**接收 HTTP 请求 → 路由到对应函数 → 返回 HTTP 响应**。

---

## 二、FastAPI 核心概念

### 2.1 安装

```bash
pip install fastapi uvicorn
```

- `fastapi` — Web 框架本体
- `uvicorn` — ASGI 服务器，负责接收网络请求并交给 FastAPI 处理

### 2.2 最小示例

新建 `main.py`：

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def root():
    return {"message": "Hello, World!"}
```

运行：

```bash
uvicorn main:app --reload
# main    → main.py 文件
# app     → 文件中的 app 变量
# --reload → 代码修改后自动重载（开发模式）
```

访问 `http://localhost:8000/` 看到 JSON 响应。

### 2.3 自动文档（Swagger UI）

FastAPI 自动生成交互式 API 文档，无需额外配置：

- **Swagger UI**：`http://localhost:8000/docs` — 可以直接在浏览器里测试 API
- **ReDoc**：`http://localhost:8000/redoc` — 更美观的只读文档

**这是 FastAPI 最大的优势之一**，开发过程中直接用 docs 页面调试，不用写测试脚本。

---

## 三、路由与参数

### 3.1 路径参数

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}

# GET /users/42  →  {"user_id": 42}
# GET /users/abc →  自动返回 422 错误（类型不匹配）
```

FastAPI 根据类型注解**自动做类型转换和验证**。

### 3.2 查询参数

```python
@app.get("/items")
def list_items(skip: int = 0, limit: int = 10, q: str | None = None):
    return {"skip": skip, "limit": limit, "q": q}

# GET /items             →  skip=0, limit=10, q=None
# GET /items?limit=5&q=abc →  skip=0, limit=5, q="abc"
```

### 3.3 请求体（Pydantic 模型）

```python
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    price: float
    in_stock: bool = True


@app.post("/items")
def create_item(item: Item):
    return {"message": "created", "item": item}
```

FastAPI 自动：
1. 从请求 body 解析 JSON
2. 用 Pydantic 验证字段类型
3. 如果验证失败，返回 `422 Unprocessable Entity` 和详细的错误信息

---

## 四、Pydantic 模型详解

Pydantic 是 FastAPI 的数据验证库，比普通 dataclass 更强大：

```python
from pydantic import BaseModel, Field, EmailStr
from datetime import datetime


class UserCreate(BaseModel):
    name: str = Field(min_length=2, max_length=50)
    email: str
    age: int = Field(ge=0, le=150)   # ge=大于等于，le=小于等于


class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    created_at: datetime


# 分开定义"创建时的输入"和"返回给客户端的输出"是好习惯
# 避免把密码等敏感字段暴露在响应里
```

---

## 五、HTTP 状态码与错误响应

```python
from fastapi import FastAPI, HTTPException, status

app = FastAPI()

@app.get("/items/{item_id}")
def get_item(item_id: int):
    if item_id not in fake_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Item {item_id} not found"
        )
    return fake_db[item_id]
```

常用的 `status` 常量（比直接写数字更可读）：

| 常量 | 值 | 含义 |
|------|----|------|
| `HTTP_200_OK` | 200 | 请求成功 |
| `HTTP_201_CREATED` | 201 | 资源创建成功 |
| `HTTP_204_NO_CONTENT` | 204 | 成功，无返回内容（删除时常用） |
| `HTTP_404_NOT_FOUND` | 404 | 资源不存在 |
| `HTTP_422_UNPROCESSABLE_ENTITY` | 422 | 数据验证失败 |

---

## 六、实操案例：TODO REST API

### 目标

实现完整的 TODO API，支持：

| 方法 | 路径 | 功能 |
|------|------|------|
| `GET` | `/todos` | 获取所有 TODO（支持过滤未完成） |
| `GET` | `/todos/{id}` | 获取单个 TODO |
| `POST` | `/todos` | 创建 TODO |
| `PATCH` | `/todos/{id}/complete` | 标记完成 |
| `DELETE` | `/todos/{id}` | 删除 TODO |

### 完整代码

新建项目目录：

```bash
mkdir day05 && cd day05
python3 -m venv .venv && source .venv/bin/activate
pip install fastapi uvicorn
```

新建 `main.py`：

```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel, Field
from datetime import datetime

app = FastAPI(title="TODO API", version="1.0.0")

# ── 数据模型 ─────────────────────────────────
class TodoCreate(BaseModel):
    title: str = Field(min_length=1, max_length=200)


class TodoResponse(BaseModel):
    id: int
    title: str
    done: bool
    created_at: str


# ── 内存数据库（Day 06 换成真实数据库）────────
todos: dict[int, dict] = {}
next_id: int = 1


# ── 路由 ─────────────────────────────────────
@app.get("/todos", response_model=list[TodoResponse])
def list_todos(done: bool | None = None):
    """获取所有 TODO，可通过 ?done=false 只获取未完成"""
    items = list(todos.values())
    if done is not None:
        items = [t for t in items if t["done"] == done]
    return items


@app.get("/todos/{todo_id}", response_model=TodoResponse)
def get_todo(todo_id: int):
    """获取单个 TODO"""
    if todo_id not in todos:
        raise HTTPException(status_code=404, detail=f"TODO {todo_id} 不存在")
    return todos[todo_id]


@app.post("/todos", response_model=TodoResponse, status_code=status.HTTP_201_CREATED)
def create_todo(body: TodoCreate):
    """创建新 TODO"""
    global next_id
    todo = {
        "id": next_id,
        "title": body.title,
        "done": False,
        "created_at": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
    }
    todos[next_id] = todo
    next_id += 1
    return todo


@app.patch("/todos/{todo_id}/complete", response_model=TodoResponse)
def complete_todo(todo_id: int):
    """标记 TODO 为已完成"""
    if todo_id not in todos:
        raise HTTPException(status_code=404, detail=f"TODO {todo_id} 不存在")
    todos[todo_id]["done"] = True
    return todos[todo_id]


@app.delete("/todos/{todo_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_todo(todo_id: int):
    """删除 TODO"""
    if todo_id not in todos:
        raise HTTPException(status_code=404, detail=f"TODO {todo_id} 不存在")
    del todos[todo_id]
```

### 运行与测试

**启动服务器：**

```bash
uvicorn main:app --reload
```

**方式一：Swagger UI（推荐）**

浏览器打开 `http://localhost:8000/docs`，直接在页面点击测试每个接口。

**方式二：curl 命令行**

```bash
# 创建 TODO
curl -X POST http://localhost:8000/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "学习 FastAPI"}'

# 获取所有 TODO
curl http://localhost:8000/todos

# 获取未完成的 TODO
curl "http://localhost:8000/todos?done=false"

# 获取单个 TODO
curl http://localhost:8000/todos/1

# 标记完成
curl -X PATCH http://localhost:8000/todos/1/complete

# 删除
curl -X DELETE http://localhost:8000/todos/1
```

**方式三：HTTPie（比 curl 更易读）**

```bash
pip install httpie
http POST localhost:8000/todos title="Buy milk"
http GET localhost:8000/todos
http PATCH localhost:8000/todos/1/complete
```

### 代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `@app.get/post/patch/delete` | 路由装饰器，绑定 URL 和函数 |
| `BaseModel` | 请求体验证 |
| `response_model=` | 控制响应字段，过滤掉不该返回的数据 |
| `status_code=201` | POST 创建资源返回 201 而非 200 |
| `204 No Content` | DELETE 成功无需返回 body |
| `HTTPException` | 抛出 HTTP 错误 |
| `global next_id` | 简单的内存自增 ID（临时用，Day 06 换掉） |

---

## 七、今日 Checklist

- [ ] Swagger UI 页面正常打开
- [ ] 通过 Swagger UI 或 curl 完成一次完整的 CRUD 操作
- [ ] 测试创建空标题时的 422 错误响应
- [ ] 测试访问不存在 ID 时的 404 响应
- [ ] git commit：`git add . && git commit -m "day05: todo rest api"`

> 📅 最后更新：2026-04-06
