# Day 03 — 面向对象 & 数据类

> 目标：理解 Python OOP 核心概念，学会用 `@dataclass` 高效建模，完成任务管理器。

---

## 一、类（Class）基础

### 1.1 定义与实例化

```python
class User:
    # 类变量（所有实例共享）
    count: int = 0

    # __init__ 是构造方法，创建实例时自动调用
    def __init__(self, name: str, age: int) -> None:
        # 实例变量（每个实例独立）
        self.name = name
        self.age = age
        User.count += 1

    # 实例方法（第一个参数必须是 self）
    def greet(self) -> str:
        return f"Hi, I'm {self.name}, {self.age} years old."

    # __repr__：在 print() 和控制台显示对象时调用
    def __repr__(self) -> str:
        return f"User(name={self.name!r}, age={self.age})"


# 实例化
u1 = User("Alice", 25)
u2 = User("Bob", 30)

print(u1.greet())     # Hi, I'm Alice, 25 years old.
print(u2)             # User(name='Bob', age=30)
print(User.count)     # 2
```

### 1.2 继承

子类继承父类的所有属性和方法，可以覆盖（override）或扩展：

```python
class Animal:
    def __init__(self, name: str) -> None:
        self.name = name

    def speak(self) -> str:
        return f"{self.name} makes a sound."


class Dog(Animal):
    def speak(self) -> str:
        # 覆盖父类方法
        return f"{self.name} says: Woof!"

    def fetch(self) -> str:
        return f"{self.name} fetches the ball!"


class GuideDog(Dog):
    def __init__(self, name: str, owner: str) -> None:
        super().__init__(name)   # 调用父类 __init__
        self.owner = owner

    def guide(self) -> str:
        return f"{self.name} guides {self.owner} safely."


dog = Dog("Rex")
print(dog.speak())    # Rex says: Woof!

guide = GuideDog("Buddy", "Jerry")
print(guide.speak())  # Buddy says: Woof!（继承自 Dog）
print(guide.guide())  # Buddy guides Jerry safely.
```

### 1.3 `@property`

将方法伪装成属性，实现计算属性和访问控制：

```python
class Circle:
    def __init__(self, radius: float) -> None:
        self._radius = radius   # 下划线表示"内部变量，勿直接访问"

    @property
    def radius(self) -> float:
        return self._radius

    @radius.setter
    def radius(self, value: float) -> None:
        if value <= 0:
            raise ValueError("半径必须大于 0")
        self._radius = value

    @property
    def area(self) -> float:
        import math
        return math.pi * self._radius ** 2


c = Circle(5)
print(c.radius)   # 5（像属性一样访问，不用加括号）
print(f"{c.area:.2f}")  # 78.54

c.radius = 10     # 触发 setter，会验证值
print(c.radius)   # 10

c.radius = -1     # ValueError: 半径必须大于 0
```

---

## 二、`@dataclass`

手写 `__init__`、`__repr__` 很繁琐。`@dataclass` 自动生成这些方法：

### 2.1 基本用法

```python
from dataclasses import dataclass, field


@dataclass
class Point:
    x: float
    y: float


p = Point(1.0, 2.0)
print(p)           # Point(x=1.0, y=2.0)（自动生成 __repr__）
print(p.x)         # 1.0
p1 = Point(1.0, 2.0)
p2 = Point(1.0, 2.0)
print(p1 == p2)    # True（自动生成 __eq__）
```

### 2.2 带默认值和方法

```python
from dataclasses import dataclass, field
from datetime import datetime


@dataclass
class Task:
    id: int
    title: str
    done: bool = False
    created_at: datetime = field(default_factory=datetime.now)
    tags: list[str] = field(default_factory=list)

    def complete(self) -> None:
        self.done = True

    def add_tag(self, tag: str) -> None:
        if tag not in self.tags:
            self.tags.append(tag)

    def __repr__(self) -> str:
        status = "✓" if self.done else "○"
        return f"[{status}] #{self.id} {self.title}"
```

> **注意**：可变类型（list、dict）的默认值必须用 `field(default_factory=...)`，不能直接写 `tags: list = []`，否则所有实例会共享同一个列表。

---

## 三、Protocol（接口/鸭子类型）

Python 没有 interface 关键字，用 `Protocol` 表达"只要有这些方法就行"：

```python
from typing import Protocol


class Printable(Protocol):
    def to_string(self) -> str:
        ...


class Task:
    def __init__(self, title: str) -> None:
        self.title = title

    def to_string(self) -> str:
        return f"Task: {self.title}"


class Note:
    def __init__(self, content: str) -> None:
        self.content = content

    def to_string(self) -> str:
        return f"Note: {self.content}"


def print_item(item: Printable) -> None:
    print(item.to_string())


print_item(Task("Buy milk"))    # Task: Buy milk
print_item(Note("Meeting at 3pm"))  # Note: Meeting at 3pm
```

---

## 四、实操案例：任务管理器

### 目标

实现一个命令行任务管理器，支持增删改查，数据存储到 JSON 文件。

### 完整代码

新建 `task_manager.py`：

```python
import json
from dataclasses import dataclass, field, asdict
from datetime import datetime
from pathlib import Path

DATA_FILE = "tasks.json"


@dataclass
class Task:
    id: int
    title: str
    done: bool = False
    created_at: str = field(default_factory=lambda: datetime.now().strftime("%Y-%m-%d %H:%M"))

    def complete(self) -> None:
        self.done = True

    def __str__(self) -> str:
        status = "✓" if self.done else "○"
        return f"[{status}] #{self.id:03d} {self.title}  ({self.created_at})"


class TaskManager:
    def __init__(self) -> None:
        self.tasks: list[Task] = []
        self._next_id: int = 1
        self._load()

    # ── 持久化 ──────────────────────────
    def _load(self) -> None:
        """从 JSON 文件加载数据"""
        if not Path(DATA_FILE).exists():
            return
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            data = json.load(f)
        self.tasks = [Task(**item) for item in data]
        if self.tasks:
            self._next_id = max(t.id for t in self.tasks) + 1

    def _save(self) -> None:
        """将数据保存到 JSON 文件"""
        with open(DATA_FILE, "w", encoding="utf-8") as f:
            json.dump([asdict(t) for t in self.tasks], f, ensure_ascii=False, indent=2)

    # ── CRUD ────────────────────────────
    def add(self, title: str) -> Task:
        task = Task(id=self._next_id, title=title)
        self.tasks.append(task)
        self._next_id += 1
        self._save()
        return task

    def complete(self, task_id: int) -> bool:
        task = self._find(task_id)
        if task is None:
            return False
        task.complete()
        self._save()
        return True

    def delete(self, task_id: int) -> bool:
        task = self._find(task_id)
        if task is None:
            return False
        self.tasks.remove(task)
        self._save()
        return True

    def list_all(self, show_done: bool = True) -> list[Task]:
        if show_done:
            return self.tasks
        return [t for t in self.tasks if not t.done]

    def _find(self, task_id: int) -> "Task | None":
        return next((t for t in self.tasks if t.id == task_id), None)


# ── CLI ─────────────────────────────────
def print_help() -> None:
    print("""
命令列表：
  add <内容>      添加任务
  list            列出所有任务
  list pending    只列出未完成任务
  done <id>       标记任务完成
  del <id>        删除任务
  help            显示帮助
  quit            退出
""")


def main() -> None:
    manager = TaskManager()
    print("任务管理器已启动。输入 help 查看命令。")

    while True:
        try:
            raw = input("\n> ").strip()
        except (KeyboardInterrupt, EOFError):
            print("\n再见！")
            break

        if not raw:
            continue

        parts = raw.split(maxsplit=1)
        cmd = parts[0].lower()
        arg = parts[1] if len(parts) > 1 else ""

        if cmd == "quit":
            print("再见！")
            break

        elif cmd == "help":
            print_help()

        elif cmd == "add":
            if not arg:
                print("用法: add <任务内容>")
            else:
                task = manager.add(arg)
                print(f"已添加: {task}")

        elif cmd == "list":
            show_done = arg != "pending"
            tasks = manager.list_all(show_done)
            if not tasks:
                print("暂无任务")
            else:
                for task in tasks:
                    print(f"  {task}")

        elif cmd == "done":
            if not arg.isdigit():
                print("用法: done <任务ID>")
            elif manager.complete(int(arg)):
                print(f"任务 #{arg} 已完成")
            else:
                print(f"找不到任务 #{arg}")

        elif cmd == "del":
            if not arg.isdigit():
                print("用法: del <任务ID>")
            elif manager.delete(int(arg)):
                print(f"任务 #{arg} 已删除")
            else:
                print(f"找不到任务 #{arg}")

        else:
            print(f"未知命令: {cmd}，输入 help 查看帮助")


if __name__ == "__main__":
    main()
```

### 运行

```bash
python3 task_manager.py
```

操作示例：

```
> add 学习 Python Day 03
已添加: [○] #001 学习 Python Day 03  (2026-04-06 10:00)

> add 做俯卧撑 50 个
已添加: [○] #002 做俯卧撑 50 个  (2026-04-06 10:00)

> list
  [○] #001 学习 Python Day 03  (2026-04-06 10:00)
  [○] #002 做俯卧撑 50 个  (2026-04-06 10:00)

> done 1
任务 #1 已完成

> list pending
  [○] #002 做俯卧撑 50 个  (2026-04-06 10:00)

> del 2
任务 #2 已删除

> quit
```

### 代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `@dataclass` | 自动生成 `__init__`、`__repr__`、`__eq__` |
| `field(default_factory=...)` | 可变默认值的正确写法 |
| `asdict()` | 将 dataclass 转为字典，方便序列化 |
| `next(..., None)` | 安全地从列表中查找第一个匹配项 |
| 数据持久化 | load/save 方法操作 JSON 文件 |
| 职责分离 | `Task` 只管数据，`TaskManager` 管业务逻辑 |

---

## 五、今日 Checklist

- [ ] 理解 `self` 是什么
- [ ] 理解 `@dataclass` 省掉了哪些代码
- [ ] 任务管理器能正常增删改查
- [ ] 退出后重新运行，任务数据仍然存在（持久化验证）
- [ ] git commit：`git add . && git commit -m "day03: task manager"`

> 📅 最后更新：2026-04-06
