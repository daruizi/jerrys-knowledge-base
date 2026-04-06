# Day 07 — 测试 & 项目结构

> 目标：掌握 pytest 基础，为 TODO API 写测试，并整理出可扩展的项目结构。这是 Python 阶段的收官日。

---

## 一、为什么要写测试？

- **防止回归**：修改代码时，测试确保旧功能没有被意外破坏
- **AI Coding 的护栏**：让 AI 生成代码后，用测试验证正确性
- **文档作用**：测试本身就是"这个函数应该怎么工作"的说明书

---

## 二、pytest 基础

### 2.1 安装

```bash
pip install pytest pytest-cov
```

### 2.2 第一个测试

pytest 约定：
- 测试文件命名为 `test_*.py` 或 `*_test.py`
- 测试函数命名以 `test_` 开头

```python
# test_math.py
def add(a: int, b: int) -> int:
    return a + b


def test_add_positive():
    assert add(1, 2) == 3


def test_add_negative():
    assert add(-1, -2) == -3


def test_add_zero():
    assert add(0, 0) == 0
```

运行：

```bash
pytest                     # 运行所有测试
pytest test_math.py        # 运行指定文件
pytest -v                  # 详细输出（显示每个测试的名字）
pytest -v -k "add"         # 只运行名字包含 "add" 的测试
```

### 2.3 assert 断言

```python
# 判断相等
assert result == expected

# 判断包含
assert "error" in message

# 判断类型
assert isinstance(result, list)

# 判断近似（浮点数）
assert abs(result - 3.14) < 0.01

# 判断抛出异常
import pytest
with pytest.raises(ValueError):
    int("not a number")
```

### 2.4 fixture — 测试前置准备

fixture 是共享的"测试环境准备"代码，避免在每个测试里重复写：

```python
import pytest


@pytest.fixture
def sample_list():
    """每个用到它的测试函数都会得到一个新列表"""
    return [1, 2, 3, 4, 5]


def test_length(sample_list):
    assert len(sample_list) == 5


def test_sum(sample_list):
    assert sum(sample_list) == 15
```

---

## 三、测试 FastAPI（TestClient）

FastAPI 提供 `TestClient`，让你在测试中直接发 HTTP 请求，不需要真正启动服务器：

```bash
pip install httpx   # TestClient 依赖 httpx
```

```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)


def test_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello, World!"}


def test_create_todo():
    response = client.post("/todos", json={"title": "Buy milk"})
    assert response.status_code == 201
    data = response.json()
    assert data["title"] == "Buy milk"
    assert data["done"] == False
    assert "id" in data
```

---

## 四、实操案例：为 TODO API 写完整测试

### 4.1 项目结构调整

先整理 Day 06 的项目结构：

```
day07/
├── app/
│   ├── __init__.py
│   ├── main.py          # FastAPI app
│   ├── models.py        # 数据模型
│   └── database.py      # 数据库配置
├── tests/
│   ├── __init__.py
│   └── test_todos.py    # 测试文件
├── requirements.txt
└── .gitignore
```

把 Day 06 的代码复制过来，调整 `database.py`，让测试使用独立的内存数据库（不影响真实数据）：

**`app/database.py`**：

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

**`app/main.py`** — 把 `app` 独立，便于测试时替换依赖：

```python
from fastapi import FastAPI, HTTPException, Depends, status
from sqlmodel import Session, select

from app.database import create_tables, get_session
from app.models import Todo, TodoCreate, TodoResponse

app = FastAPI(title="TODO API")


@app.on_event("startup")
def on_startup():
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


@app.post("/todos", response_model=TodoResponse, status_code=201)
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


@app.delete("/todos/{todo_id}", status_code=204)
def delete_todo(todo_id: int, session: Session = Depends(get_session)):
    todo = session.get(Todo, todo_id)
    if not todo:
        raise HTTPException(status_code=404, detail=f"TODO {todo_id} 不存在")
    session.delete(todo)
    session.commit()
```

### 4.2 测试文件

**`tests/test_todos.py`**：

```python
import pytest
from fastapi.testclient import TestClient
from sqlmodel import create_engine, Session, SQLModel

from app.main import app
from app.database import get_session
from app.models import Todo

# 使用内存数据库，测试完全隔离，不影响真实数据
TEST_DATABASE_URL = "sqlite://"


@pytest.fixture(name="session")
def session_fixture():
    """每个测试函数使用独立的空数据库"""
    engine = create_engine(TEST_DATABASE_URL, connect_args={"check_same_thread": False})
    SQLModel.metadata.create_all(engine)
    with Session(engine) as session:
        yield session
    SQLModel.metadata.drop_all(engine)


@pytest.fixture(name="client")
def client_fixture(session: Session):
    """替换真实数据库连接为测试数据库"""
    def override_get_session():
        yield session

    app.dependency_overrides[get_session] = override_get_session
    with TestClient(app) as client:
        yield client
    app.dependency_overrides.clear()


# ── 测试用例 ─────────────────────────────────

def test_create_todo(client: TestClient):
    """测试创建 TODO"""
    response = client.post("/todos", json={"title": "Buy milk"})
    assert response.status_code == 201
    data = response.json()
    assert data["title"] == "Buy milk"
    assert data["done"] == False
    assert data["id"] == 1


def test_list_todos(client: TestClient):
    """测试获取 TODO 列表"""
    client.post("/todos", json={"title": "Task 1"})
    client.post("/todos", json={"title": "Task 2"})

    response = client.get("/todos")
    assert response.status_code == 200
    assert len(response.json()) == 2


def test_list_todos_filter_done(client: TestClient):
    """测试按完成状态过滤"""
    r1 = client.post("/todos", json={"title": "Task 1"})
    client.post("/todos", json={"title": "Task 2"})

    todo_id = r1.json()["id"]
    client.patch(f"/todos/{todo_id}/complete")

    response = client.get("/todos?done=false")
    assert len(response.json()) == 1
    assert response.json()[0]["title"] == "Task 2"


def test_get_todo(client: TestClient):
    """测试获取单个 TODO"""
    created = client.post("/todos", json={"title": "Test"}).json()
    response = client.get(f"/todos/{created['id']}")
    assert response.status_code == 200
    assert response.json()["title"] == "Test"


def test_get_todo_not_found(client: TestClient):
    """测试获取不存在的 TODO 返回 404"""
    response = client.get("/todos/999")
    assert response.status_code == 404


def test_complete_todo(client: TestClient):
    """测试标记完成"""
    created = client.post("/todos", json={"title": "Test"}).json()
    response = client.patch(f"/todos/{created['id']}/complete")
    assert response.status_code == 200
    assert response.json()["done"] == True


def test_delete_todo(client: TestClient):
    """测试删除 TODO"""
    created = client.post("/todos", json={"title": "To delete"}).json()
    response = client.delete(f"/todos/{created['id']}")
    assert response.status_code == 204

    # 确认已删除
    response = client.get(f"/todos/{created['id']}")
    assert response.status_code == 404


def test_create_todo_empty_title(client: TestClient):
    """测试空标题返回 422"""
    response = client.post("/todos", json={"title": ""})
    assert response.status_code == 422
```

### 4.3 运行测试

```bash
# 在 day07/ 目录下
pytest tests/ -v
```

预期输出：

```
tests/test_todos.py::test_create_todo              PASSED
tests/test_todos.py::test_list_todos               PASSED
tests/test_todos.py::test_list_todos_filter_done   PASSED
tests/test_todos.py::test_get_todo                 PASSED
tests/test_todos.py::test_get_todo_not_found       PASSED
tests/test_todos.py::test_complete_todo            PASSED
tests/test_todos.py::test_delete_todo              PASSED
tests/test_todos.py::test_create_todo_empty_title  PASSED

8 passed in 0.45s
```

查看测试覆盖率：

```bash
pytest tests/ -v --cov=app --cov-report=term-missing
```

---

## 五、Python 阶段项目结构最佳实践

```
my-project/
├── app/                    # 应用代码
│   ├── __init__.py
│   ├── main.py             # FastAPI app 和路由
│   ├── models.py           # 数据模型
│   ├── database.py         # 数据库配置
│   └── config.py           # 配置项（从环境变量读取）
├── tests/                  # 测试代码
│   ├── __init__.py
│   └── test_*.py
├── .env                    # 环境变量（不提交到 git）
├── .env.example            # 环境变量示例（提交到 git）
├── .gitignore
├── requirements.txt
└── README.md
```

**`.gitignore` 必须包含：**

```
.venv/
__pycache__/
*.pyc
.env
*.db
.coverage
```

---

## 六、Python 阶段知识地图

```
Week 1 Python 后端
│
├── Day 01  基础语法（变量、列表、字典、推导式）
├── Day 02  函数、模块、错误处理
├── Day 03  OOP、dataclass、持久化
├── Day 04  HTTP 概念、httpx、调用外部 API
├── Day 05  FastAPI、路由、Pydantic、REST API
├── Day 06  SQLite、SQLModel、ORM、依赖注入
└── Day 07  pytest、TestClient、测试隔离、项目结构
```

掌握以上内容，你已经能独立构建一个**带数据库的 REST API**，这是大多数后端应用的核心骨架。

---

## 七、今日 Checklist

- [ ] `pytest tests/ -v` 全部测试通过（8/8）
- [ ] 理解 `fixture` 的作用（测试隔离）
- [ ] 理解 `dependency_overrides` 如何替换真实数据库
- [ ] 整理项目目录结构，加好 `.gitignore`
- [ ] git commit：`git add . && git commit -m "day07: api tests and project structure"`

---

## 八、进入 Day 08 前的自检

回答以下问题（不看讲义），能答对说明掌握扎实：

1. `@dataclass` 和普通 `class` 的区别是什么？
2. `try/except/else/finally` 各自在什么情况下执行？
3. FastAPI 中 `Depends` 的作用是什么？
4. 为什么测试要用内存数据库而不是真实数据库？
5. `GET` 和 `POST` 的本质区别是什么？

> 📅 最后更新：2026-04-06
