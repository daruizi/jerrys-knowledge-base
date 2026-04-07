# Day 04 — 标准库精华

> 目标：掌握 Python 最常用的标准库模块——pathlib、datetime、re、collections、typing、os/subprocess，能不查文档完成日常文件/数据处理任务。

---

## 一、pathlib — 现代路径操作

`pathlib.Path` 是 Python 3.4+ 的跨平台路径库，比字符串拼接更安全、更直观。

```python
from pathlib import Path

# 创建路径对象（不会真正创建文件）
p = Path("data/users.json")
home = Path.home()               # 用户主目录，如 /Users/jerry
cwd = Path.cwd()                 # 当前工作目录

# 路径拼接（用 / 运算符）
project = Path("/Users/jerry/projects")
config = project / "config" / "settings.json"
print(config)  # /Users/jerry/projects/config/settings.json

# 路径属性
p = Path("data/report.csv")
print(p.name)       # report.csv       （文件名含扩展名）
print(p.stem)       # report            （文件名不含扩展名）
print(p.suffix)     # .csv             （扩展名）
print(p.parent)     # data             （父目录）
print(p.parts)      # ('data', 'report.csv')

# 判断存在性
p = Path("data")
print(p.exists())    # 是否存在
print(p.is_file())   # 是否是文件
print(p.is_dir())    # 是否是目录

# 创建目录
Path("output/reports").mkdir(parents=True, exist_ok=True)
# parents=True：自动创建中间目录
# exist_ok=True：已存在不报错

# 读写文件（比 open() 更简洁）
p = Path("hello.txt")
p.write_text("Hello, World!", encoding="utf-8")      # 写入
content = p.read_text(encoding="utf-8")              # 读取
p.write_bytes(b"\x89PNG\r\n")                        # 写入二进制
data = p.read_bytes()                                # 读取二进制

# 遍历目录
for item in Path(".").iterdir():
    print(item.name, "dir" if item.is_dir() else "file")

# glob 模式搜索
for py_file in Path(".").glob("**/*.py"):   # ** 匹配任意层级
    print(py_file)

for md_file in Path(".").rglob("*.md"):     # rglob 等价于 glob("**/*.md")
    print(md_file)

# 其他常用操作
p = Path("old_name.txt")
p.rename("new_name.txt")           # 重命名/移动
p.unlink(missing_ok=True)          # 删除文件
p.stat().st_size                   # 文件大小（字节）
p.stat().st_mtime                  # 修改时间（Unix 时间戳）

# 绝对路径
print(Path("data/file.txt").resolve())  # 解析为绝对路径
```

---

## 二、datetime — 时间处理

```python
from datetime import datetime, date, time, timedelta
from zoneinfo import ZoneInfo   # Python 3.9+

# ── 创建 ────────────────────────────────────────
now = datetime.now()            # 当前本地时间
today = date.today()            # 当前日期
utc_now = datetime.now(ZoneInfo("UTC"))              # UTC 时间
shanghai = datetime.now(ZoneInfo("Asia/Shanghai"))   # 上海时间

# 手动指定
dt = datetime(2026, 4, 7, 14, 30, 0)  # 2026-04-07 14:30:00

# ── 格式化 ──────────────────────────────────────
# strftime：datetime → string
print(now.strftime("%Y-%m-%d %H:%M:%S"))    # 2026-04-07 14:30:00
print(now.strftime("%Y/%m/%d"))             # 2026/04/07
print(now.strftime("%B %d, %Y"))            # April 07, 2026

# strptime：string → datetime
dt = datetime.strptime("2026-04-07 14:30", "%Y-%m-%d %H:%M")

# 常用格式符
# %Y 四位年  %m 月(01-12)  %d 日(01-31)
# %H 时(00-23)  %M 分(00-59)  %S 秒(00-59)
# %A 星期全称  %a 星期缩写  %B 月份全称

# ── 属性访问 ────────────────────────────────────
print(now.year, now.month, now.day)
print(now.hour, now.minute, now.second)
print(now.weekday())   # 0=周一, 6=周日
print(now.isoformat()) # "2026-04-07T14:30:00"（ISO 8601 格式）

# ── timedelta — 时间差 ──────────────────────────
delta = timedelta(days=7, hours=3, minutes=30)
future = now + delta
past = now - timedelta(days=30)

# 计算两个日期之间的差
d1 = date(2026, 1, 1)
d2 = date(2026, 4, 7)
diff = d2 - d1
print(diff.days)  # 96（天数差）

# 判断是否超时
import time
start = datetime.now()
time.sleep(0.1)
elapsed = (datetime.now() - start).total_seconds()
print(f"Elapsed: {elapsed:.3f}s")

# ── 时区处理 ─────────────────────────────────────
# 带时区的 datetime
aware_dt = datetime(2026, 4, 7, 14, 30, tzinfo=ZoneInfo("Asia/Shanghai"))
# 转换时区
utc_dt = aware_dt.astimezone(ZoneInfo("UTC"))
print(utc_dt.strftime("%Y-%m-%d %H:%M %Z"))  # 2026-04-07 06:30 UTC
```

---

## 三、re — 正则表达式（重点）

正则是处理文本的核心工具，也是爬虫阶段必备技能。

### 3.1 基本函数

```python
import re

text = "My email is jerry@example.com and phone is 138-1234-5678"

# search：找第一个匹配，返回 Match 对象或 None
match = re.search(r"\d{3}-\d{4}-\d{4}", text)
if match:
    print(match.group())   # 138-1234-5678
    print(match.start())   # 匹配开始位置
    print(match.end())     # 匹配结束位置

# match：只在字符串开头匹配
re.match(r"\d+", "123abc")  # 匹配
re.match(r"\d+", "abc123")  # None

# findall：找所有匹配，返回字符串列表
emails = re.findall(r"[\w.+-]+@[\w-]+\.[a-z]{2,}", text)
print(emails)  # ['jerry@example.com']

# finditer：找所有匹配，返回 Match 对象迭代器（大文本时更高效）
for m in re.finditer(r"\d+", "a1 b22 c333"):
    print(m.group(), m.start())  # 1 1 / 22 4 / 333 8

# sub：替换
result = re.sub(r"\d{3}-\d{4}-\d{4}", "***-****-****", text)
print(result)  # 手机号被遮码

# compile：预编译（重复使用同一模式时更高效）
phone_pattern = re.compile(r"\d{3}-\d{4}-\d{4}")
match = phone_pattern.search(text)
```

### 3.2 分组（Capture Groups）

```python
# () 创建捕获组
date_str = "2026-04-07"
match = re.search(r"(\d{4})-(\d{2})-(\d{2})", date_str)
if match:
    print(match.group(0))  # "2026-04-07"（整体匹配）
    print(match.group(1))  # "2026"
    print(match.group(2))  # "04"
    print(match.group(3))  # "07"
    year, month, day = match.groups()  # 解包所有分组

# 命名分组 (?P<name>...)
match = re.search(r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})", date_str)
if match:
    print(match.group("year"))   # "2026"
    print(match.groupdict())     # {'year': '2026', 'month': '04', 'day': '07'}

# 非捕获组 (?:...)
re.findall(r"(?:https?|ftp)://(\S+)", "Visit https://example.com")
# ['example.com']（只捕获括号内的内容，不包括协议部分的 (?:...) ）
```

### 3.3 常用模式

```python
# 元字符
# .   任意字符（除换行）
# ^   行首  $  行尾
# \d  数字  \D  非数字
# \w  字母/数字/下划线  \W  非 \w
# \s  空白字符  \S  非空白
# \b  单词边界

# 量词
# *   0次或多次   +  1次或多次   ?  0次或1次
# {n}  恰好n次   {n,m}  n到m次
# 默认贪婪，加?变非贪婪：*? +? ??

# 常用模式汇总
patterns = {
    "email":    r"[\w.+-]+@[\w-]+\.[a-z]{2,}",
    "url":      r"https?://[^\s<>\"]+",
    "ip":       r"\b(?:\d{1,3}\.){3}\d{1,3}\b",
    "phone_cn": r"1[3-9]\d{9}",              # 手机号
    "phone_fmt": r"\d{3}[-\s]\d{4}[-\s]\d{4}",  # 138-1234-5678
    "date":     r"\d{4}-\d{2}-\d{2}",
    "chinese":  r"[\u4e00-\u9fff]+",
    "integer":  r"-?\d+",
    "float":    r"-?\d+\.?\d*",
}

# 标志（flags）
re.search(r"hello", "Hello World", re.IGNORECASE)   # 忽略大小写
re.findall(r"^\d+", "1\n2\n3", re.MULTILINE)        # ^ 匹配每行行首
re.search(r".+", "line1\nline2", re.DOTALL)          # . 匹配换行符
```

### 3.4 实用技巧

```python
# split：用正则分割
parts = re.split(r"[,;|\s]+", "a, b;c|d  e")
print(parts)  # ['a', 'b', 'c', 'd', 'e']

# sub 用函数做替换
def mask_phone(match):
    return match.group()[:3] + "****" + match.group()[-4:]

result = re.sub(r"1[3-9]\d{9}", mask_phone, "联系 13812345678")
print(result)  # 联系 138****5678
```

---

## 四、collections — 专用容器

### 4.1 Counter — 计数器

```python
from collections import Counter

# 创建
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]
c = Counter(words)
print(c)               # Counter({'apple': 3, 'banana': 2, 'cherry': 1})

# 最常见的 N 个
print(c.most_common(2))  # [('apple', 3), ('banana', 2)]

# 访问（不存在的键返回 0，不报 KeyError）
print(c["apple"])   # 3
print(c["mango"])   # 0

# 运算
c2 = Counter(["banana", "cherry", "mango"])
print(c + c2)       # 合并计数
print(c - c2)       # 减去（结果 <= 0 的自动删除）
print(c & c2)       # 取最小值
print(c | c2)       # 取最大值

# 统计字符
text = "hello world"
char_count = Counter(text)
print(char_count.most_common(3))  # [('l', 3), ('o', 2), ('h', 1)]
```

### 4.2 defaultdict — 带默认值的字典

```python
from collections import defaultdict

# 普通字典：访问不存在的键会 KeyError
# defaultdict：访问不存在的键时自动创建默认值

# 分组
records = [("Engineering", "Alice"), ("Marketing", "Bob"), ("Engineering", "Charlie")]
groups = defaultdict(list)
for dept, name in records:
    groups[dept].append(name)  # 第一次访问 dept 时自动创建空列表
print(dict(groups))
# {'Engineering': ['Alice', 'Charlie'], 'Marketing': ['Bob']}

# 计数（Counter 的底层原理）
word_count = defaultdict(int)  # 默认值是 int() = 0
for word in "hello world hello".split():
    word_count[word] += 1

# 嵌套字典
nested = defaultdict(lambda: defaultdict(int))
nested["Beijing"]["male"] += 10
nested["Beijing"]["female"] += 15
print(dict(nested))  # {'Beijing': defaultdict(<int>, {'male': 10, 'female': 15})}
```

### 4.3 namedtuple — 命名元组

```python
from collections import namedtuple

# 创建带名字字段的元组（比普通元组更可读）
Point = namedtuple("Point", ["x", "y"])
LogEntry = namedtuple("LogEntry", ["ip", "method", "path", "status", "size"])

p = Point(10, 20)
print(p.x, p.y)    # 10 20
print(p[0], p[1])  # 10 20（也支持索引访问）
print(p)           # Point(x=10, y=20)

# 解包
x, y = p

log = LogEntry("192.168.1.1", "GET", "/index.html", 200, 1024)
print(f"{log.ip} {log.method} {log.path} → {log.status}")

# _asdict()：转为普通字典
print(log._asdict())
# OrderedDict([('ip', '192.168.1.1'), ...])

# typing.NamedTuple（更现代的写法，支持类型注解和默认值）
from typing import NamedTuple

class User(NamedTuple):
    name: str
    age: int
    city: str = "Beijing"  # 默认值

u = User("Alice", 25)
print(u)  # User(name='Alice', age=25, city='Beijing')
```

### 4.4 deque — 双端队列

```python
from collections import deque

# deque 在两端插入/删除都是 O(1)，列表左端操作是 O(n)
dq = deque([1, 2, 3])

dq.append(4)         # 右端添加
dq.appendleft(0)     # 左端添加
right = dq.pop()     # 右端弹出
left = dq.popleft()  # 左端弹出

# maxlen：固定大小的滑动窗口
recent = deque(maxlen=5)
for i in range(10):
    recent.append(i)
print(recent)  # deque([5, 6, 7, 8, 9], maxlen=5)
```

---

## 五、typing — 类型系统

Day 01-03 用了基础的类型注解，这里补全完整的 typing 模块：

### 5.1 基础类型

```python
from typing import Optional, Union

# Optional[X] 等价于 Union[X, None]
def find_user(user_id: int) -> Optional[dict]:
    ...  # 可能返回 dict 或 None

# Python 3.10+ 可以用 X | None 替代 Optional[X]
def find_user(user_id: int) -> dict | None:
    ...

# Union：可以是多种类型之一
def parse_id(value: Union[str, int]) -> int:
    return int(value)

# Python 3.10+：用 | 替代 Union
def parse_id(value: str | int) -> int:
    return int(value)
```

### 5.2 泛型（TypeVar / Generic）

```python
from typing import TypeVar, Generic

T = TypeVar("T")

# 泛型函数：返回类型与输入类型一致
def first(items: list[T]) -> T | None:
    return items[0] if items else None

n = first([1, 2, 3])     # n 的类型是 int
s = first(["a", "b"])    # s 的类型是 str

# 泛型类
class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

    def __len__(self) -> int:
        return len(self._items)

int_stack: Stack[int] = Stack()
int_stack.push(1)
int_stack.push(2)
```

### 5.3 TypeAlias / Literal / TypedDict

```python
from typing import TypeAlias, Literal, TypedDict

# TypeAlias：给复杂类型起别名
Vector: TypeAlias = list[float]
Matrix: TypeAlias = list[list[float]]

def dot(a: Vector, b: Vector) -> float:
    return sum(x * y for x, y in zip(a, b))

# Literal：限制为特定值
def set_log_level(level: Literal["DEBUG", "INFO", "WARNING", "ERROR"]) -> None:
    ...

set_log_level("INFO")     # OK
# set_log_level("VERBOSE")  # 类型检查器报错

# TypedDict：给字典定义类型
class UserDict(TypedDict):
    name: str
    age: int
    email: str

class PartialUser(TypedDict, total=False):  # total=False 所有字段可选
    name: str
    email: str

def process_user(user: UserDict) -> str:
    return f"{user['name']}, {user['age']}"
```

### 5.4 Annotated — 元数据注解

```python
from typing import Annotated

# 在类型上附加元数据（供框架/工具使用）
type Age = Annotated[int, "must be >= 0 and <= 150"]
type Email = Annotated[str, "must contain @"]

def create_account(name: str, age: Age, email: Email) -> None:
    ...

# FastAPI、Pydantic 大量使用 Annotated 来做数据校验
```

---

## 六、json / csv 进阶

### 6.1 json 进阶

```python
import json
from datetime import datetime
from dataclasses import dataclass, asdict

# 默认 json.dumps 不能序列化 datetime、dataclass 等
@dataclass
class Event:
    name: str
    timestamp: datetime

# ❌ json.dumps(asdict(Event("launch", datetime.now())))  # TypeError

# ✅ 自定义序列化器
class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

event = {"name": "launch", "timestamp": datetime.now()}
json_str = json.dumps(event, cls=CustomEncoder, ensure_ascii=False, indent=2)

# 或用 default 参数（更简洁）
def json_serializer(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Not serializable: {type(obj)}")

json_str = json.dumps(event, default=json_serializer)

# 从字符串/字节读取
data = json.loads('{"name": "Alice", "age": 25}')
data = json.loads(b'{"key": "value"}')  # 也接受 bytes
```

### 6.2 csv 进阶

```python
import csv
from pathlib import Path

# 读取（DictReader 把每行变成字典）
with open("data.csv", "r", encoding="utf-8-sig", newline="") as f:
    # utf-8-sig 自动处理 Excel 生成的 BOM 字符
    reader = csv.DictReader(f)
    for row in reader:
        print(row)  # {'name': 'Alice', 'age': '25'}

# 写入（DictWriter）
fieldnames = ["name", "age", "city"]
with open("output.csv", "w", encoding="utf-8", newline="") as f:
    # newline="" 防止 Windows 上出现额外空行
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows([
        {"name": "Alice", "age": 25, "city": "Beijing"},
        {"name": "Bob", "age": 30, "city": "Shanghai"},
    ])

# 处理含逗号的字段（csv 模块自动加引号）
writer.writerow({"name": "O'Brien, Tom", "age": 35, "city": "New York"})
```

---

## 七、os / subprocess — 系统交互

```python
import os
import subprocess

# ── 环境变量 ──────────────────────────────────
# 读取环境变量
db_url = os.environ["DATABASE_URL"]        # 不存在抛 KeyError
api_key = os.getenv("API_KEY", "default")  # 不存在返回默认值

# 设置环境变量（只影响当前进程）
os.environ["MY_VAR"] = "value"

# 推荐用 python-dotenv 从 .env 文件加载
# from dotenv import load_dotenv
# load_dotenv()  # 加载 .env 文件中的变量

# ── 常用 os 函数 ──────────────────────────────
print(os.getcwd())             # 当前工作目录
os.chdir("/tmp")               # 切换目录
os.makedirs("a/b/c", exist_ok=True)  # 递归创建目录
os.remove("file.txt")          # 删除文件
os.rename("old.txt", "new.txt")  # 重命名
print(os.path.exists("file"))  # 路径是否存在（推荐用 pathlib 替代）

# ── subprocess — 执行外部命令 ────────────────
# run：运行命令，等待完成
result = subprocess.run(
    ["ls", "-la"],              # 命令和参数列表（避免 shell=True 的注入风险）
    capture_output=True,        # 捕获 stdout 和 stderr
    text=True,                  # 输出解码为字符串（而不是 bytes）
    check=True                  # 非零退出码时抛出 CalledProcessError
)
print(result.stdout)
print(result.returncode)   # 0 = 成功

# 简化写法：只需要输出时
output = subprocess.check_output(["git", "log", "--oneline", "-5"], text=True)
print(output)

# 带超时
try:
    result = subprocess.run(["sleep", "10"], timeout=3)
except subprocess.TimeoutExpired:
    print("命令超时")

# 传入 stdin
result = subprocess.run(
    ["wc", "-l"],
    input="line1\nline2\nline3",
    capture_output=True,
    text=True
)
print(result.stdout.strip())  # "3"
```

---

## 八、实操案例：Nginx 日志分析器

### 目标

综合运用 pathlib + re + collections + datetime，解析 Nginx 访问日志，统计各状态码、IP 访问频次、每小时流量分布。

### 步骤 1：创建示例日志

新建 `access.log`（Nginx 标准格式）：

```
192.168.1.1 - - [07/Apr/2026:10:00:01 +0800] "GET /index.html HTTP/1.1" 200 1234
192.168.1.2 - - [07/Apr/2026:10:00:15 +0800] "POST /api/login HTTP/1.1" 200 512
192.168.1.1 - - [07/Apr/2026:10:01:30 +0800] "GET /about.html HTTP/1.1" 200 2048
192.168.1.3 - - [07/Apr/2026:11:00:01 +0800] "GET /missing.html HTTP/1.1" 404 0
192.168.1.4 - - [07/Apr/2026:11:30:00 +0800] "GET /admin HTTP/1.1" 403 0
192.168.1.1 - - [07/Apr/2026:12:00:05 +0800] "GET /api/data HTTP/1.1" 500 0
192.168.1.2 - - [07/Apr/2026:12:01:00 +0800] "GET /index.html HTTP/1.1" 200 1234
192.168.1.5 - - [07/Apr/2026:12:05:00 +0800] "DELETE /api/user HTTP/1.1" 204 0
```

### 步骤 2：编写分析器

新建 `log_analyzer.py`：

```python
import re
from collections import Counter, defaultdict, namedtuple
from datetime import datetime
from pathlib import Path

# ── 数据模型 ──────────────────────────────────

LogEntry = namedtuple("LogEntry", [
    "ip", "method", "path", "status", "size", "timestamp"
])

# ── 正则模式 ──────────────────────────────────

LOG_PATTERN = re.compile(
    r'(?P<ip>\d+\.\d+\.\d+\.\d+)'          # IP 地址
    r' - - '
    r'\[(?P<time>[^\]]+)\]'                  # 时间戳
    r' "(?P<method>\w+) (?P<path>\S+) [^"]+"'  # 请求方法和路径
    r' (?P<status>\d{3})'                    # 状态码
    r' (?P<size>\d+)'                        # 响应大小
)

TIME_FORMAT = "%d/%b/%Y:%H:%M:%S %z"

# ── 解析函数 ──────────────────────────────────

def parse_log(filepath: str) -> list[LogEntry]:
    """解析 Nginx 日志文件，返回 LogEntry 列表"""
    path = Path(filepath)
    if not path.exists():
        raise FileNotFoundError(f"日志文件不存在: {filepath}")

    entries = []
    errors = 0

    for line_num, line in enumerate(path.read_text(encoding="utf-8").splitlines(), 1):
        line = line.strip()
        if not line:
            continue

        match = LOG_PATTERN.search(line)
        if not match:
            errors += 1
            continue

        d = match.groupdict()
        try:
            entry = LogEntry(
                ip=d["ip"],
                method=d["method"],
                path=d["path"],
                status=int(d["status"]),
                size=int(d["size"]),
                timestamp=datetime.strptime(d["time"], TIME_FORMAT),
            )
            entries.append(entry)
        except ValueError:
            errors += 1

    print(f"解析完成: {len(entries)} 条记录，{errors} 条解析失败\n")
    return entries

# ── 分析函数 ──────────────────────────────────

def analyze(entries: list[LogEntry]) -> None:
    if not entries:
        print("没有可分析的记录")
        return

    # 1. 状态码分布
    status_count = Counter(e.status for e in entries)
    print("── 状态码分布 ─────────────────")
    for status, count in sorted(status_count.items()):
        bar = "█" * count
        category = {2: "成功", 3: "重定向", 4: "客户端错误", 5: "服务端错误"}
        label = category.get(status // 100, "未知")
        print(f"  {status} ({label}): {count:3d}  {bar}")

    # 2. 访问频次最高的 IP
    ip_count = Counter(e.ip for e in entries)
    print("\n── Top 5 IP ──────────────────")
    for ip, count in ip_count.most_common(5):
        print(f"  {ip:<18} {count:3d} 次")

    # 3. 最热门的路径
    path_count = Counter(e.path for e in entries)
    print("\n── Top 5 路径 ────────────────")
    for path, count in path_count.most_common(5):
        print(f"  {path:<30} {count:3d} 次")

    # 4. 每小时流量
    hourly = defaultdict(int)
    for e in entries:
        hour = e.timestamp.strftime("%H:00")
        hourly[hour] += 1

    print("\n── 每小时流量 ────────────────")
    for hour in sorted(hourly):
        count = hourly[hour]
        bar = "█" * count
        print(f"  {hour}  {count:3d}  {bar}")

    # 5. 错误请求（4xx/5xx）
    errors = [e for e in entries if e.status >= 400]
    if errors:
        print(f"\n── 错误请求 ({len(errors)} 条) ──────────")
        for e in errors:
            print(f"  {e.ip:<18} {e.status}  {e.method} {e.path}")

    # 6. 总流量
    total_bytes = sum(e.size for e in entries)
    print(f"\n── 总计 ──────────────────────")
    print(f"  请求总数: {len(entries)}")
    print(f"  总流量:   {total_bytes:,} bytes ({total_bytes / 1024:.1f} KB)")
    print(f"  错误率:   {len(errors) / len(entries):.1%}")


if __name__ == "__main__":
    entries = parse_log("access.log")
    analyze(entries)
```

### 步骤 3：运行

```bash
python3 log_analyzer.py
```

预期输出：

```
解析完成: 8 条记录，0 条解析失败

── 状态码分布 ─────────────────
  200 (成功):   4  ████
  204 (成功):   1  █
  403 (客户端错误):   1  █
  404 (客户端错误):   1  █
  500 (服务端错误):   1  █

── Top 5 IP ──────────────────
  192.168.1.1          3 次
  192.168.1.2          2 次
  192.168.1.3          1 次
  192.168.1.4          1 次
  192.168.1.5          1 次

── 每小时流量 ────────────────
  10:00    3  ███
  11:00    2  ██
  12:00    3  ███

── 错误请求 (3 条) ──────────
  192.168.1.3        404  GET /missing.html
  192.168.1.4        403  GET /admin
  192.168.1.1        500  GET /api/data

── 总计 ──────────────────────
  请求总数: 8
  总流量:   5,028 bytes (4.9 KB)
  错误率:   37.5%
```

### 代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `pathlib.Path` | 文件读取、存在性检查 |
| `re.compile` + 命名分组 | 解析 Nginx 日志行 |
| `namedtuple` | LogEntry 结构化日志条目 |
| `Counter` | 状态码/IP/路径计数与排序 |
| `defaultdict(int)` | 按小时聚合流量 |
| `datetime.strptime` | 解析日志时间戳 |
| `strftime` | 格式化为小时 |

---

## 九、今日 Checklist

- [ ] 能用 `pathlib.Path` 做文件读写，不用 `open()` + 字符串拼接
- [ ] 能用 `datetime.strptime` 和 `strftime` 互相转换
- [ ] 能用 `re.search` / `findall` / `sub` 处理文本
- [ ] 能写带命名分组的正则表达式
- [ ] 理解 `Counter` / `defaultdict` / `namedtuple` / `deque` 的使用场景
- [ ] 了解 `Optional` / `Union` / `TypeVar` / `Literal` / `TypedDict` 的用法
- [ ] `log_analyzer.py` 运行输出正确分析报告
- [ ] git commit：`git add . && git commit -m "day04: log analyzer"`

> 📅 最后更新：2026-04-07
