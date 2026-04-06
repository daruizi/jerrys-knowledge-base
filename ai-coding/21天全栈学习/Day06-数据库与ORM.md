# Day 06 — 数据库 & ORM

> 目标：理解关系型数据库核心概念，用 SQLModel 给 Day 05 的 TODO API 加上持久化存储。

---

## 一、为什么需要数据库？

Day 05 的 API 用字典存数据，重启服务后数据全丢失。数据库解决两个问题：

1. **持久化**：数据写入磁盘，重启后还在
2. **并发安全**：多个请求同时读写时不会数据损坏

---

## 二、关系型数据库基础

### 2.1 核心概念

| 概念 | 类比 | 说明 |
|------|------|------|
| 数据库（Database） | Excel 文件 | 多个表的容器 |
| 表（Table） | Excel Sheet | 存储同类数据的结构 |
| 行（Row） | Excel 行 | 一条记录 |
| 列（Column） | Excel 列 | 一个字段 |
| 主键（Primary Key） | 唯一 ID | 每行的唯一标识，不可重复 |

### 2.2 SQLite

SQLite 是一个**文件型数据库**，整个数据库就是一个 `.db` 文件：

- 无需安装数据库服务器
- Python 内置支持（`import sqlite3`）
- 适合开发/小项目；生产环境换 PostgreSQL

### 2.3 SQL 基础语句

虽然我们用 ORM，但要看懂底层 SQL：

```sql
-- 创建表
CREATE TABLE todos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    done BOOLEAN DEFAULT FALSE,
    created_at TEXT
);

-- 插入
INSERT INTO todos (title, done, created_at) VALUES ('Buy milk', FALSE, '2026-04-06');

-- 查询
SELECT * FROM todos;
SELECT * FROM todos WHERE done = FALSE;
SELECT * FROM todos ORDER BY id DESC LIMIT 10;

-- 更新
UPDATE todos SET done = TRUE WHERE id = 1;

-- 删除
DELETE FROM todos WHERE id = 1;
```

---

## 三、ORM — 用 Python 操作数据库

ORM（Object-Relational Mapping）让你用 Python 类操作数据库，无需手写 SQL。

**核心思想：**

```
Python 类  ←→  数据库表
类的属性   ←→  表的列
类的实例   ←→  表的一行
```

### 3.1 SQLModel 简介

SQLModel = SQLAlchemy（成熟的 ORM 底层）+ Pydantic（数据验证）。  
由 FastAPI 作者开发，与 FastAPI 天然兼容。

```bash
pip install sqlmodel
```

### 3.2 定义模型

```python
from sqlmodel import SQLModel, Field
from datetime import datetime


class Todo(SQLModel, table=True):
    # table=True 表示这是一个数据库表模型
    id: int | None = Field(default=None, primary_key=True)
    title: str = Field(max_length=200)
    done: bool = Field(default=False)
    created_at: str = Field(default_factory=lambda: datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
```

### 3.3 创建表和 Session

```python
from sqlmodel import create_engine, Session, SQLModel

# 数据库文件路径
DATABASE_URL = "sqlite:///todos.db"

# 创建引擎（连接数据库的对象）
engine = create_engine(DATABASE_URL, echo=True)
# echo=True 会打印执行的 SQL 语句，调试时很有用


def create_tables():
    """创建所有表（已存在则跳过）"""
    SQLModel.metadata.create_all(engine)


def get_session():
    """获取数据库 Session（每次请求用完关闭）"""
    with Session(engine) as session:
        yield session
```

### 3.4 CRUD 操作

```python
from sqlmodel import Session, select


# 创建
def create_todo(session: Session, title: str) -> Todo:
    todo = Todo(title=title)
    session.add(todo)
    session.commit()
    session.refresh(todo)  # 刷新，获取数据库生成的 id
    return todo


# 查询所有
def list_todos(session: Session) -> list[Todo]:
    statement = select(Todo)
    return session.exec(statement).all()


# 查询单个
def get_todo(session: Session, todo_id: int) -> Todo | None:
    return session.get(Todo, todo_id)


# 更新
def complete_todo(session: Session, todo_id: int) -> Todo | None:
    todo = session.get(Todo, todo_id)
    if todo:
        todo.done = True
        session.commit()
        session.refresh(todo)
    return todo


# 删除
def delete_todo(session: Session, todo_id: int) -> bool:
    todo = session.get(Todo, todo_id)
    if todo:
        session.delete(todo)
        session.commit()
        return True
    return False
```

---

## 四、实操案例：为 TODO API 添加数据库

### 完整代码

新建 `day06/` 目录，创建以下文件：

**`database.py`** — 数据库配置：

```python
from sqlmodel import create_engine, Session, SQLModel

DATABASE_URL = "sqlite:///todos.db"
engine = create_engine(DATABASE_URL, echo=False)


def create_tables() -> None:
    SQLModel.metadata.create_all(engine)


def get_session():
    with Session(engine) as session:
        yield session
```

**`models.py`** — 数据模型：

```python
from sqlmodel import SQLModel, Field
from datetime import datetime


# 数据库表模型
class Todo(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    title: str = Field(max_length=200)
    done: bool = Field(default=False)
    created_at: str = Field(
        default_factory=lambda: datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    )


# 请求体模型（不含 id/created_at，由数据库生成）
class TodoCreate(SQLModel):
    title: str = Field(min_length=1, max_length=200)


# 响应模型（明确控制返回字段）
class TodoResponse(SQLModel):
    id: int
    title: str
    done: bool
    created_at: str
```

**`main.py`** — API 路由：

```python
from fastapi import FastAPI, HTTPException, Depends, status
from sqlmodel import Session, select

from database import create_tables, get_session
from models import Todo, TodoCreate, TodoResponse

app = FastAPI(title="TODO API v2（数据库版）")


@app.on_event("startup")
def on_startup():
    """应用启动时创建数据库表"""
    create_tables()


@app.get("/todos", response_model=list[TodoResponse])
def list_todos(done: bool | None = None, session: Session = Depends(get_session)):
    statement = select(Todo)
    if done is not None:
        statement = statement.where(Todo.done == done)
    return session.exec(statement).all()


@app.get("/todos/{todo_id}", response_model=TodoResponse)
def get_todo(todo_id: int, session: Session = Depends(get_session)):
    todo = session.get(Todo, todo_id)
    if not todo:
        raise HTTPException(status_code=404, detail=f"TODO {todo_id} 不存在")
    return todo


@app.post("/todos", response_model=TodoResponse, status_code=status.HTTP_201_CREATED)
def create_todo(body: TodoCreate, session: Session = Depends(get_session)):
    todo = Todo(title=body.title)
    session.add(todo)
    session.commit()
    session.refresh(todo)
    return todo


@app.patch("/todos/{todo_id}/complete", response_model=TodoResponse)
def complete_todo(todo_id: int, session: Session = Depends(get_session)):
    todo = session.get(Todo, todo_id)
    if not todo:
        raise HTTPException(status_code=404, detail=f"TODO {todo_id} 不存在")
    todo.done = True
    session.commit()
    session.refresh(todo)
    return todo


@app.delete("/todos/{todo_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_todo(todo_id: int, session: Session = Depends(get_session)):
    todo = session.get(Todo, todo_id)
    if not todo:
        raise HTTPException(status_code=404, detail=f"TODO {todo_id} 不存在")
    session.delete(todo)
    session.commit()
```

### 运行

```bash
cd day06
pip install fastapi uvicorn sqlmodel
uvicorn main:app --reload
```

验证持久化效果：

```bash
# 创建几个 TODO
curl -X POST http://localhost:8000/todos -H "Content-Type: application/json" -d '{"title":"Task 1"}'
curl -X POST http://localhost:8000/todos -H "Content-Type: application/json" -d '{"title":"Task 2"}'

# 停止服务器（Ctrl+C），再重新启动
uvicorn main:app --reload

# 数据仍然存在
curl http://localhost:8000/todos
```

### 查看数据库内容

```bash
# 安装 sqlite3 命令行工具（macOS 已内置）
sqlite3 todos.db

# 在 sqlite3 里执行：
.tables          -- 查看所有表
.schema todo     -- 查看表结构
SELECT * FROM todo;  -- 查询所有数据
.quit            -- 退出
```

### 依赖注入（Depends）原理

```python
# Depends(get_session) 的意思：
# "每次收到请求时，自动调用 get_session() 获取数据库连接，
#  请求处理完后自动关闭连接"

@app.get("/todos")
def list_todos(session: Session = Depends(get_session)):
    #                              ↑ FastAPI 自动注入，你不用手动传
    ...
```

这是 FastAPI 的**依赖注入**系统，让数据库连接的生命周期自动管理。

---

## 五、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `SQLModel` + `table=True` | 定义数据库表模型 |
| `create_engine` | 创建数据库连接 |
| `Session` | 数据库操作的单元，用完关闭 |
| `session.add/commit/refresh` | 插入数据的三步：加入→提交→刷新 |
| `select(Model).where(...)` | 构建查询语句 |
| `Depends(get_session)` | FastAPI 依赖注入，自动管理 session 生命周期 |
| 模型分层 | `Todo`（表）、`TodoCreate`（输入）、`TodoResponse`（输出）分开定义 |

---

## 六、今日 Checklist

- [ ] 运行 API 后能看到 `todos.db` 文件生成
- [ ] 重启服务器后数据仍然存在（持久化验证）
- [ ] 用 `sqlite3 todos.db` 命令行查看数据
- [ ] 理解 `Depends` 的作用
- [ ] git commit：`git add . && git commit -m "day06: todo api with sqlite"`

> 📅 最后更新：2026-04-06
