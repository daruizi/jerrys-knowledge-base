# Day 03 — 面向对象编程（完整版）

> 目标：全面掌握 Python OOP——类、继承、魔术方法、@dataclass、ABC、Enum、Protocol，完成任务管理器。

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

### 1.2 `@classmethod` 和 `@staticmethod`

```python
class User:
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age

    # classmethod：第一个参数是类本身（cls），常用于工厂方法
    @classmethod
    def from_string(cls, data: str) -> "User":
        """从 'name,age' 格式字符串创建用户"""
        name, age_str = data.split(",")
        return cls(name.strip(), int(age_str.strip()))

    @classmethod
    def from_dict(cls, data: dict) -> "User":
        """从字典创建用户"""
        return cls(data["name"], data["age"])

    # staticmethod：与类相关但不需要 self/cls 的工具方法
    @staticmethod
    def is_valid_age(age: int) -> bool:
        return 0 <= age <= 150

    def __repr__(self) -> str:
        return f"User({self.name!r}, {self.age})"


# classmethod 作为工厂方法
u1 = User.from_string("Alice, 25")
u2 = User.from_dict({"name": "Bob", "age": 30})
print(u1, u2)  # User('Alice', 25) User('Bob', 30)

# staticmethod 作为工具方法
print(User.is_valid_age(25))   # True
print(User.is_valid_age(200))  # False
```

### 1.3 继承

子类继承父类的所有属性和方法，可以覆盖（override）或扩展：

```python
class Animal:
    def __init__(self, name: str) -> None:
        self.name = name

    def speak(self) -> str:
        return f"{self.name} makes a sound."


class Dog(Animal):
    def speak(self) -> str:
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

### 1.4 多重继承与 MRO

Python 支持多重继承，方法解析顺序（MRO）决定调用哪个父类方法：

```python
class A:
    def greet(self) -> str:
        return "Hello from A"

class B(A):
    def greet(self) -> str:
        return "Hello from B"

class C(A):
    def greet(self) -> str:
        return "Hello from C"

class D(B, C):  # 同时继承 B 和 C
    pass

d = D()
print(d.greet())  # "Hello from B"（按 MRO 顺序，B 在 C 前面）

# 查看 MRO 顺序
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)

# super() 按 MRO 链逐级调用
class Base:
    def __init__(self):
        print("Base init")

class Left(Base):
    def __init__(self):
        print("Left init")
        super().__init__()  # → 按 MRO 调用下一个

class Right(Base):
    def __init__(self):
        print("Right init")
        super().__init__()

class Child(Left, Right):
    def __init__(self):
        print("Child init")
        super().__init__()

Child()
# Child init → Left init → Right init → Base init
```

> **建议**：多重继承容易造成复杂性，优先使用组合（has-a）而非继承（is-a）。如需多重行为，用 Protocol 或 Mixin 模式。

### 1.5 `@property`

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
print(c.radius)        # 5（像属性一样访问，不用加括号）
print(f"{c.area:.2f}") # 78.54

c.radius = 10          # 触发 setter，会验证值
# c.radius = -1        # ValueError: 半径必须大于 0
```

---

## 二、魔术方法（Dunder Methods）

### 2.1 字符串表示

```python
class Product:
    def __init__(self, name: str, price: float):
        self.name = name
        self.price = price

    def __repr__(self) -> str:
        """给开发者看的精确表示（在控制台、列表中显示）"""
        return f"Product({self.name!r}, {self.price})"

    def __str__(self) -> str:
        """给用户看的友好表示（print 时调用）"""
        return f"{self.name} - ¥{self.price:.2f}"

p = Product("iPhone", 7999)
print(repr(p))  # Product('iPhone', 7999)
print(str(p))   # iPhone - ¥7999.00
print(p)        # iPhone - ¥7999.00（print 优先调用 __str__）
```

### 2.2 比较与排序

```python
from functools import total_ordering

@total_ordering  # 只需定义 __eq__ 和 __lt__，自动生成其余比较方法
class Student:
    def __init__(self, name: str, score: float):
        self.name = name
        self.score = score

    def __eq__(self, other) -> bool:
        if not isinstance(other, Student):
            return NotImplemented
        return self.score == other.score

    def __lt__(self, other) -> bool:
        if not isinstance(other, Student):
            return NotImplemented
        return self.score < other.score

    def __hash__(self) -> int:
        """定义了 __eq__ 就必须定义 __hash__，否则不能放入 set/dict"""
        return hash((self.name, self.score))

    def __repr__(self) -> str:
        return f"{self.name}({self.score})"

students = [Student("Alice", 85), Student("Bob", 92), Student("Charlie", 78)]
print(sorted(students))          # [Charlie(78), Alice(85), Bob(92)]
print(max(students))             # Bob(92)
print(Student("A", 85) == Student("B", 85))  # True（按 score 比较）
```

### 2.3 容器行为

```python
class Playlist:
    def __init__(self, name: str, songs: list[str] = None):
        self.name = name
        self._songs = songs or []

    def __len__(self) -> int:
        return len(self._songs)

    def __getitem__(self, index):
        return self._songs[index]

    def __setitem__(self, index, value):
        self._songs[index] = value

    def __contains__(self, song: str) -> bool:
        return song in self._songs

    def __iter__(self):
        return iter(self._songs)

    def __repr__(self) -> str:
        return f"Playlist({self.name!r}, {len(self)} songs)"

pl = Playlist("My Mix", ["Song A", "Song B", "Song C"])
print(len(pl))           # 3
print(pl[0])             # Song A
print(pl[1:])            # ['Song B', 'Song C']（切片也支持）
print("Song B" in pl)    # True
pl[0] = "New Song"       # 修改

for song in pl:           # 可迭代
    print(song)
```

### 2.4 运算符重载

```python
class Vector:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    def __add__(self, other: "Vector") -> "Vector":
        return Vector(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar: float) -> "Vector":
        return Vector(self.x * scalar, self.y * scalar)

    def __abs__(self) -> float:
        return (self.x ** 2 + self.y ** 2) ** 0.5

    def __repr__(self) -> str:
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)     # Vector(4, 6)
print(v1 * 3)      # Vector(3, 6)
print(abs(v2))      # 5.0
```

### 2.5 可调用对象

```python
class Formatter:
    """实现 __call__ 让实例像函数一样被调用"""
    def __init__(self, prefix: str):
        self.prefix = prefix

    def __call__(self, message: str) -> str:
        return f"[{self.prefix}] {message}"

error = Formatter("ERROR")
info = Formatter("INFO")
print(error("Something went wrong"))  # [ERROR] Something went wrong
print(info("Server started"))         # [INFO] Server started

# 可以像函数一样传递
messages = ["msg1", "msg2"]
print(list(map(error, messages)))  # ['[ERROR] msg1', '[ERROR] msg2']
```

---

## 三、`__slots__` — 内存优化

默认情况下每个实例都有一个 `__dict__` 字典存储属性，用 `__slots__` 可以固定属性列表、减少内存：

```python
import sys

class RegularPoint:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

class SlottedPoint:
    __slots__ = ("x", "y")  # 固定属性列表
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

# 内存对比
r = RegularPoint(1, 2)
s = SlottedPoint(1, 2)
print(sys.getsizeof(r.__dict__))  # ~104 字节（属性字典）
# s.__dict__                      # AttributeError! 没有 __dict__

# slots 阻止动态添加属性
# s.z = 3                         # AttributeError!
r.z = 3                           # OK（普通类允许）
```

> 大量实例（如数据记录）时用 `__slots__` 节省内存。一般业务类不需要。

---

## 四、`@dataclass`

自动生成 `__init__`、`__repr__`、`__eq__` 等方法，减少样板代码：

### 4.1 基本用法

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: float
    y: float

p1 = Point(1.0, 2.0)
print(p1)           # Point(x=1.0, y=2.0)（自动生成 __repr__）
p2 = Point(1.0, 2.0)
print(p1 == p2)     # True（自动生成 __eq__）
```

### 4.2 带默认值和方法

```python
from dataclasses import dataclass, field, asdict
from datetime import datetime

@dataclass
class Task:
    id: int
    title: str
    done: bool = False
    created_at: str = field(default_factory=lambda: datetime.now().strftime("%Y-%m-%d %H:%M"))
    tags: list[str] = field(default_factory=list)  # 可变类型必须用 field(default_factory=...)

    def complete(self) -> None:
        self.done = True

    def add_tag(self, tag: str) -> None:
        if tag not in self.tags:
            self.tags.append(tag)

# asdict：转为字典（方便 JSON 序列化）
t = Task(1, "Learn Python")
print(asdict(t))
# {'id': 1, 'title': 'Learn Python', 'done': False, 'created_at': '...', 'tags': []}
```

### 4.3 不可变 dataclass（frozen）

```python
@dataclass(frozen=True)  # 不可修改
class Color:
    r: int
    g: int
    b: int

red = Color(255, 0, 0)
# red.r = 128  # FrozenInstanceError!

# frozen dataclass 可以作为 dict key 或放入 set
colors = {Color(255, 0, 0): "red", Color(0, 0, 255): "blue"}
```

### 4.4 排序支持（order）

```python
@dataclass(order=True)   # 自动生成比较方法
class Priority:
    level: int
    name: str

tasks = [Priority(3, "low"), Priority(1, "critical"), Priority(2, "high")]
print(sorted(tasks))  # [Priority(1, 'critical'), Priority(2, 'high'), Priority(3, 'low')]
```

---

## 五、Enum 枚举

用于定义一组命名常量：

```python
from enum import Enum, auto, StrEnum

class Status(Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    DONE = "done"
    CANCELLED = "cancelled"

# 访问
print(Status.PENDING)         # Status.PENDING
print(Status.PENDING.value)   # "pending"
print(Status.PENDING.name)    # "PENDING"

# 从值创建
s = Status("done")
print(s)                      # Status.DONE

# 比较
print(Status.DONE == Status.DONE)      # True
print(Status.DONE == "done")           # False（Enum 不等于原始值！）
print(Status.DONE is Status.DONE)      # True

# 遍历
for status in Status:
    print(f"{status.name}: {status.value}")

# 作为字典 key
counts = {status: 0 for status in Status}
counts[Status.DONE] = 5

# auto()：自动分配值
class Color(Enum):
    RED = auto()     # 1
    GREEN = auto()   # 2
    BLUE = auto()    # 3

# StrEnum（Python 3.11+）：值就是字符串，可以直接当字符串用
class Priority(StrEnum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"

print(Priority.HIGH == "high")  # True（StrEnum 可以和字符串比较）
print(f"Priority: {Priority.HIGH}")  # Priority: high
```

---

## 六、ABC（抽象基类）

强制子类实现特定方法：

```python
from abc import ABC, abstractmethod

class Storage(ABC):
    """存储后端的抽象接口"""

    @abstractmethod
    def save(self, key: str, data: dict) -> None:
        """保存数据"""
        ...

    @abstractmethod
    def load(self, key: str) -> dict | None:
        """加载数据"""
        ...

    def exists(self, key: str) -> bool:
        """非抽象方法：子类可以继承不重写"""
        return self.load(key) is not None

# s = Storage()  # TypeError! 不能实例化抽象类

class JSONStorage(Storage):
    """JSON 文件存储"""
    def __init__(self, directory: str = "."):
        self.directory = directory

    def save(self, key: str, data: dict) -> None:
        import json
        from pathlib import Path
        path = Path(self.directory) / f"{key}.json"
        path.write_text(json.dumps(data, ensure_ascii=False, indent=2))

    def load(self, key: str) -> dict | None:
        import json
        from pathlib import Path
        path = Path(self.directory) / f"{key}.json"
        if not path.exists():
            return None
        return json.loads(path.read_text())

class MemoryStorage(Storage):
    """内存存储（测试用）"""
    def __init__(self):
        self._data: dict[str, dict] = {}

    def save(self, key: str, data: dict) -> None:
        self._data[key] = data

    def load(self, key: str) -> dict | None:
        return self._data.get(key)

# 两种实现都可以当 Storage 使用
def backup(storage: Storage, key: str, data: dict) -> None:
    storage.save(key, data)
    print(f"Saved to {type(storage).__name__}")

backup(JSONStorage(), "users", {"name": "Alice"})
backup(MemoryStorage(), "users", {"name": "Alice"})
```

**ABC vs Protocol**：
- **ABC**：显式继承，运行时检查 `isinstance(obj, Storage)` → True
- **Protocol**：隐式满足（鸭子类型），不需要继承，更灵活

---

## 七、Protocol（结构化类型）

只要有对应的方法/属性就算满足接口，不需要继承：

```python
from typing import Protocol, runtime_checkable

@runtime_checkable  # 允许 isinstance 检查
class Printable(Protocol):
    def to_string(self) -> str: ...

class Task:
    def __init__(self, title: str):
        self.title = title
    def to_string(self) -> str:
        return f"Task: {self.title}"

class Note:
    def __init__(self, content: str):
        self.content = content
    def to_string(self) -> str:
        return f"Note: {self.content}"

def print_item(item: Printable) -> None:
    print(item.to_string())

print_item(Task("Buy milk"))         # Task: Buy milk
print_item(Note("Meeting at 3pm"))   # Note: Meeting at 3pm

# runtime_checkable 允许运行时检查
print(isinstance(Task("x"), Printable))  # True

# Protocol 也可以定义属性
class Named(Protocol):
    name: str

def greet(obj: Named) -> str:
    return f"Hello, {obj.name}"
```

---

## 八、实操案例：任务管理器

### 目标

实现一个命令行任务管理器，综合运用 Enum、@dataclass、ABC、魔术方法。

### 完整代码

新建 `task_manager.py`：

```python
import json
from abc import ABC, abstractmethod
from dataclasses import dataclass, field, asdict
from datetime import datetime
from enum import StrEnum
from pathlib import Path

# ── Enum 定义任务状态 ──

class Status(StrEnum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    DONE = "done"

# ── dataclass 定义任务 ──

@dataclass
class Task:
    id: int
    title: str
    priority: int = 3               # 1=最高, 5=最低
    status: Status = Status.PENDING
    created_at: str = field(default_factory=lambda: datetime.now().strftime("%Y-%m-%d %H:%M"))

    def complete(self) -> None:
        self.status = Status.DONE

    def start(self) -> None:
        self.status = Status.IN_PROGRESS

    def __lt__(self, other: "Task") -> bool:
        """支持按优先级排序（数字越小优先级越高）"""
        return self.priority < other.priority

    def __str__(self) -> str:
        icons = {Status.PENDING: "○", Status.IN_PROGRESS: "◑", Status.DONE: "✓"}
        icon = icons[self.status]
        return f"[{icon}] #{self.id:03d} P{self.priority} {self.title}  ({self.created_at})"

# ── ABC 定义存储接口 ──

class TaskStorage(ABC):
    @abstractmethod
    def save(self, tasks: list[Task]) -> None: ...
    @abstractmethod
    def load(self) -> list[Task]: ...

class JSONTaskStorage(TaskStorage):
    def __init__(self, filepath: str = "tasks.json"):
        self.filepath = Path(filepath)

    def save(self, tasks: list[Task]) -> None:
        data = [asdict(t) for t in tasks]
        self.filepath.write_text(
            json.dumps(data, ensure_ascii=False, indent=2), encoding="utf-8"
        )

    def load(self) -> list[Task]:
        if not self.filepath.exists():
            return []
        data = json.loads(self.filepath.read_text(encoding="utf-8"))
        return [Task(**item) for item in data]

# ── 管理器 ──

class TaskManager:
    def __init__(self, storage: TaskStorage) -> None:
        self.storage = storage
        self.tasks: list[Task] = storage.load()
        self._next_id: int = max((t.id for t in self.tasks), default=0) + 1

    def add(self, title: str, priority: int = 3) -> Task:
        task = Task(id=self._next_id, title=title, priority=priority)
        self.tasks.append(task)
        self._next_id += 1
        self.storage.save(self.tasks)
        return task

    def complete(self, task_id: int) -> bool:
        task = self._find(task_id)
        if task is None:
            return False
        task.complete()
        self.storage.save(self.tasks)
        return True

    def start(self, task_id: int) -> bool:
        task = self._find(task_id)
        if task is None:
            return False
        task.start()
        self.storage.save(self.tasks)
        return True

    def delete(self, task_id: int) -> bool:
        task = self._find(task_id)
        if task is None:
            return False
        self.tasks.remove(task)
        self.storage.save(self.tasks)
        return True

    def list_all(self, status: Status | None = None) -> list[Task]:
        tasks = self.tasks if status is None else [t for t in self.tasks if t.status == status]
        return sorted(tasks)  # 按优先级排序（使用 __lt__）

    def _find(self, task_id: int) -> Task | None:
        return next((t for t in self.tasks if t.id == task_id), None)

# ── CLI ──

def print_help() -> None:
    print("""
命令列表：
  add <内容> [p=优先级]   添加任务（p=1最高, p=5最低, 默认3）
  list [pending|done]    列出任务（可选按状态过滤）
  start <id>             标记任务进行中
  done <id>              标记任务完成
  del <id>               删除任务
  help                   显示帮助
  quit                   退出
""")

def main() -> None:
    storage = JSONTaskStorage()
    manager = TaskManager(storage)
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

        match cmd:
            case "quit":
                print("再见！")
                break

            case "help":
                print_help()

            case "add":
                if not arg:
                    print("用法: add <任务内容> [p=1~5]")
                    continue
                # 解析优先级 p=N
                priority = 3
                if " p=" in arg:
                    arg, p_str = arg.rsplit(" p=", 1)
                    if p_str.isdigit() and 1 <= int(p_str) <= 5:
                        priority = int(p_str)
                task = manager.add(arg, priority)
                print(f"已添加: {task}")

            case "list":
                status_filter = None
                if arg == "pending":
                    status_filter = Status.PENDING
                elif arg == "done":
                    status_filter = Status.DONE
                tasks = manager.list_all(status_filter)
                if not tasks:
                    print("暂无任务")
                else:
                    for task in tasks:
                        print(f"  {task}")

            case "start":
                if not arg.isdigit():
                    print("用法: start <任务ID>")
                elif manager.start(int(arg)):
                    print(f"任务 #{arg} 进行中")
                else:
                    print(f"找不到任务 #{arg}")

            case "done":
                if not arg.isdigit():
                    print("用法: done <任务ID>")
                elif manager.complete(int(arg)):
                    print(f"任务 #{arg} 已完成")
                else:
                    print(f"找不到任务 #{arg}")

            case "del":
                if not arg.isdigit():
                    print("用法: del <任务ID>")
                elif manager.delete(int(arg)):
                    print(f"任务 #{arg} 已删除")
                else:
                    print(f"找不到任务 #{arg}")

            case _:
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
> add 学习 Python Day 03 p=1
已添加: [○] #001 P1 学习 Python Day 03  (2026-04-07 10:00)

> add 做俯卧撑 50 个
已添加: [○] #002 P3 做俯卧撑 50 个  (2026-04-07 10:00)

> start 1
任务 #1 进行中

> list
  [◑] #001 P1 学习 Python Day 03  (2026-04-07 10:00)
  [○] #002 P3 做俯卧撑 50 个  (2026-04-07 10:00)

> done 1
任务 #1 已完成

> list pending
  [○] #002 P3 做俯卧撑 50 个  (2026-04-07 10:00)

> quit
```

### 代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `StrEnum` | Status 枚举，可直接当字符串用 |
| `@dataclass` + `field` | Task 自动生成 init/repr/eq |
| `__lt__` | 支持 `sorted(tasks)` 按优先级排序 |
| `__str__` | 友好的任务显示格式 |
| `ABC` + `@abstractmethod` | TaskStorage 接口强制实现 save/load |
| `@classmethod` 模式 | JSONTaskStorage 的工厂模式 |
| `match/case` | 命令分发替代 if-elif |
| 数据持久化 | JSON 文件存储 |
| 职责分离 | Task 管数据、TaskManager 管逻辑、TaskStorage 管存储 |

---

## 九、今日 Checklist

- [ ] 理解 `self` 和 `cls` 的区别
- [ ] 能写 `@classmethod` 工厂方法
- [ ] 理解 `@dataclass` 省掉了哪些代码
- [ ] 理解 `__repr__` vs `__str__` 的区别
- [ ] 能用 `__lt__` 让对象支持排序
- [ ] 理解 `__slots__` 的内存优化作用
- [ ] 能定义 Enum 并在代码中使用
- [ ] 理解 ABC 和 Protocol 的区别和使用场景
- [ ] 任务管理器能正常增删改查
- [ ] 退出后重新运行，任务数据仍然存在（持久化验证）
- [ ] git commit：`git add . && git commit -m "day03: task manager"`

> 📅 最后更新：2026-04-07
