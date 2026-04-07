# Day 01 — Python 环境 & 核心语法

> 目标：搭好环境，全面掌握 Python 内置数据类型和核心语法，完成第一个实用脚本。

---

## 一、环境搭建

### 1.1 安装 Python

```bash
brew install python
python3 --version   # 确认版本 >= 3.12
```

### 1.2 虚拟环境（venv）

**为什么需要虚拟环境？**  
每个项目依赖的第三方库版本不同，虚拟环境让每个项目有独立的"包空间"，互不干扰。

```bash
# 创建项目目录
mkdir 21days-python && cd 21days-python
mkdir day01 && cd day01

# 创建虚拟环境
python3 -m venv .venv

# 激活（每次开新终端都要激活）
source .venv/bin/activate

# 退出虚拟环境
deactivate
```

激活后终端提示符前会出现 `(.venv)` 标识。

### 1.3 pip 包管理基础

```bash
pip install httpx              # 安装第三方库
pip list                       # 查看已安装的包
pip freeze > requirements.txt  # 导出依赖列表
pip install -r requirements.txt  # 从依赖列表安装
```

---

## 二、变量与基本类型

### 2.1 动态类型与类型注解

Python 是动态类型语言，变量无需声明类型：

```python
# 基本类型
name = "Alice"          # str 字符串
age = 25                # int 整数
height = 1.75           # float 浮点数
is_active = True        # bool 布尔值
nothing = None          # NoneType 空值

# 查看类型
print(type(name))       # <class 'str'>
print(type(age))        # <class 'int'>
```

**类型注解（Type Hints）**——不强制，但强烈推荐，AI 辅助更准确：

```python
name: str = "Alice"
age: int = 25
score: float = 9.5
is_active: bool = True
```

### 2.2 类型转换

不同类型之间可以显式转换：

```python
# int ↔ float ↔ str
print(int(3.9))          # 3（截断，不是四舍五入）
print(float(3))          # 3.0
print(str(42))           # "42"
print(int("42"))         # 42
print(int("0xFF", 16))  # 255（指定进制）
print(int("0b1010", 2)) # 10

# bool 转换规则
print(bool(0))     # False
print(bool(42))    # True（非零为 True）
print(bool(""))    # False
print(bool("hi"))  # True（非空为 True）
print(bool([]))    # False
print(bool(None))  # False
```

### 2.3 Truthiness（真值判断）

Python 中以下值在布尔上下文中为 **False**，其余全部为 **True**：

| 为 False 的值 | 类型 |
|---------------|------|
| `None` | NoneType |
| `False` | bool |
| `0`, `0.0`, `0j` | 数字零 |
| `""` | 空字符串 |
| `[]` | 空列表 |
| `()` | 空元组 |
| `{}` | 空字典 |
| `set()` | 空集合 |

```python
# 实际应用：用 truthiness 简化判断
items = []
if not items:
    print("列表为空")  # 等价于 if len(items) == 0

name = ""
print(name or "匿名")  # "匿名"（短路求值：name 为 falsy，返回右侧）

name = "Jerry"
print(name or "匿名")  # "Jerry"（name 为 truthy，返回左侧）
```

---

## 三、数字运算

```python
# 基本运算
print(10 + 3)     # 13  加
print(10 - 3)     # 7   减
print(10 * 3)     # 30  乘
print(10 / 3)     # 3.3333...  除（始终返回 float）
print(10 // 3)    # 3   整除（向下取整）
print(10 % 3)     # 1   取余
print(2 ** 10)    # 1024 幂运算

# divmod 同时获取商和余数
quotient, remainder = divmod(17, 5)
print(quotient, remainder)  # 3 2

# 常用内置函数
print(abs(-42))       # 42
print(round(3.14159, 2))  # 3.14
print(max(3, 7, 1))  # 7
print(min(3, 7, 1))  # 1
print(sum([1, 2, 3, 4]))  # 10

# math 模块（更多数学运算）
import math
print(math.floor(3.9))   # 3   向下取整
print(math.ceil(3.1))    # 4   向上取整
print(math.sqrt(16))     # 4.0 平方根
print(math.pi)           # 3.141592653589793
print(math.inf)          # 无穷大（常用于算法初始值）

# float 精度问题
print(0.1 + 0.2)             # 0.30000000000000004（不是 0.3！）
print(0.1 + 0.2 == 0.3)      # False
print(math.isclose(0.1 + 0.2, 0.3))  # True（正确的比较方式）
```

---

## 四、字符串（str）

### 4.1 创建与 f-string

```python
# 三种引号
s1 = "hello"          # 双引号
s2 = 'hello'          # 单引号（等价）
s3 = """多行
字符串"""               # 三引号（保留换行）

# raw string（忽略转义）
path = r"C:\new\test"  # 不会把 \n 当换行符
print(path)            # C:\new\test

# f-string（推荐的格式化方式，Python 3.6+）
name = "Jerry"
age = 30
print(f"Hello, {name}!")
print(f"Age: {age}, next year: {age + 1}")
print(f"Score: {9.5:.2f}")         # 保留2位小数 → 9.50
print(f"{'Jerry':>10}")            # 右对齐，宽度10
print(f"{'Jerry':<10}")            # 左对齐，宽度10
print(f"{'Jerry':^10}")            # 居中对齐
print(f"{1000000:,}")              # 千分位 → 1,000,000
print(f"{0.75:.0%}")               # 百分比 → 75%
```

### 4.2 常用字符串方法

**查找与判断：**

```python
s = "Hello, World!"

# 查找
print(s.find("World"))      # 7（返回索引，找不到返回 -1）
print(s.index("World"))     # 7（找不到抛异常 ValueError）
print(s.count("l"))         # 3（出现次数）
print(s.startswith("Hello"))  # True
print(s.endswith("!"))        # True
print("World" in s)           # True（推荐用 in 做判断）
```

**转换：**

```python
s = "  Hello, World!  "

print(s.strip())        # "Hello, World!"（去两端空白）
print(s.lstrip())       # "Hello, World!  "（只去左端）
print(s.rstrip())       # "  Hello, World!"（只去右端）

s2 = "hello world"
print(s2.upper())       # "HELLO WORLD"
print(s2.lower())       # "hello world"
print(s2.title())       # "Hello World"
print(s2.capitalize())  # "Hello world"（只首字母大写）
print(s2.replace("world", "Python"))  # "hello Python"
```

**分割与拼接：**

```python
# split：字符串 → 列表
csv_line = "apple,banana,cherry"
fruits = csv_line.split(",")       # ["apple", "banana", "cherry"]
print("a b  c".split())           # ["a", "b", "c"]（默认按空白分割，自动去多余空格）
print("a.b.c".split(".", 1))      # ["a", "b.c"]（maxsplit=1 只分割一次）

# join：列表 → 字符串（与 split 相反）
words = ["Hello", "World"]
print(" ".join(words))       # "Hello World"
print(", ".join(words))      # "Hello, World"
print("\n".join(words))      # 逐行输出

# splitlines：按行分割
text = "line1\nline2\nline3"
print(text.splitlines())    # ["line1", "line2", "line3"]
```

**类型检测：**

```python
print("123".isdigit())    # True（全数字）
print("abc".isalpha())    # True（全字母）
print("abc123".isalnum()) # True（全字母或数字）
print("   ".isspace())    # True（全空白字符）
```

### 4.3 字符串是不可变的

```python
s = "hello"
# s[0] = "H"  # TypeError! 字符串不可变
s = "H" + s[1:]  # 只能创建新字符串 → "Hello"
```

---

## 五、列表（list）

有序、可重复、可修改：

### 5.1 基本操作

```python
fruits = ["apple", "banana", "cherry"]

# 访问
print(fruits[0])       # apple
print(fruits[-1])      # cherry（负索引从末尾算）
print(fruits[1:3])     # ['banana', 'cherry']（切片，左闭右开）
print(fruits[::2])     # ['apple', 'cherry']（步长为2）
print(fruits[::-1])    # ['cherry', 'banana', 'apple']（反转）

# 修改
fruits.append("mango")            # 末尾添加一个
fruits.extend(["kiwi", "grape"])   # 末尾添加多个（合并列表）
fruits.insert(1, "blueberry")     # 指定位置插入

# 删除
fruits.remove("banana")  # 删除第一个匹配的值（不存在则 ValueError）
popped = fruits.pop()    # 弹出末尾元素并返回
popped2 = fruits.pop(0)  # 弹出指定索引的元素
del fruits[0]            # 按索引删除，不返回值

# 查找
print(fruits.index("cherry"))  # 返回第一次出现的索引
print(fruits.count("apple"))   # 统计出现次数
print("apple" in fruits)       # True/False 判断包含

# 排序
numbers = [3, 1, 4, 1, 5]
numbers.sort()                  # 原地排序（修改原列表）
numbers.sort(reverse=True)      # 降序
sorted_nums = sorted(numbers)   # 返回新列表，不修改原列表

# 常用
print(len(fruits))       # 长度
```

### 5.2 浅拷贝 vs 赋值

```python
# 赋值只是创建引用，两个变量指向同一列表
a = [1, 2, 3]
b = a           # b 和 a 是同一个列表
b.append(4)
print(a)        # [1, 2, 3, 4] ← a 也变了！

# 浅拷贝：创建新列表（内层对象仍共享）
a = [1, 2, 3]
b = a.copy()    # 或 b = a[:] 或 b = list(a)
b.append(4)
print(a)        # [1, 2, 3] ← a 不变

# 嵌套列表的浅拷贝陷阱
a = [[1, 2], [3, 4]]
b = a.copy()
b[0].append(99)
print(a)        # [[1, 2, 99], [3, 4]] ← 内层列表仍共享！

# 深拷贝：完全独立
import copy
a = [[1, 2], [3, 4]]
b = copy.deepcopy(a)
b[0].append(99)
print(a)        # [[1, 2], [3, 4]] ← 完全不受影响
```

### 5.3 解包（Unpacking）

```python
# 基本解包
first, second, third = [1, 2, 3]

# 星号解包：收集剩余元素
first, *rest = [1, 2, 3, 4, 5]
print(first)  # 1
print(rest)   # [2, 3, 4, 5]

*head, last = [1, 2, 3, 4, 5]
print(head)   # [1, 2, 3, 4]
print(last)   # 5

first, *middle, last = [1, 2, 3, 4, 5]
print(middle)  # [2, 3, 4]

# _ 忽略不需要的值
_, second, _ = [1, 2, 3]
print(second)  # 2
```

---

## 六、元组（tuple）

有序、可重复、**不可变**：

```python
# 创建
point = (10, 20)
single = (42,)          # 单元素元组必须加逗号！
empty = ()
from_list = tuple([1, 2, 3])

# 访问（与列表相同）
print(point[0])    # 10
print(point[-1])   # 20

# 不可修改
# point[0] = 99    # TypeError!

# 解包（最常用的场景）
x, y = point
print(x, y)        # 10 20

# 函数返回多个值本质是返回元组
def min_max(nums):
    return min(nums), max(nums)

low, high = min_max([3, 1, 4, 1, 5])
print(low, high)   # 1 5

# 元组作为字典的 key（列表不行，因为列表可变）
locations = {
    (39.9, 116.4): "Beijing",
    (31.2, 121.5): "Shanghai",
}
print(locations[(39.9, 116.4)])  # Beijing

# 常用方法
t = (1, 2, 2, 3, 3, 3)
print(t.count(3))  # 3
print(t.index(2))  # 1（第一次出现的索引）
```

**什么时候用元组而不是列表？**
- 数据不应被修改时（坐标、RGB 颜色、数据库记录行）
- 作为字典的 key 时
- 函数返回多个值时

---

## 七、字典（dict）

键值对，Python 3.7+ 保持插入顺序：

### 7.1 基本操作

```python
user = {
    "name": "Jerry",
    "age": 30,
    "city": "Beijing"
}

# 访问
print(user["name"])                    # Jerry
print(user.get("email", "N/A"))        # 安全访问，键不存在返回默认值

# 修改
user["age"] = 31                       # 修改
user["email"] = "jerry@example.com"    # 新增

# 删除
del user["city"]                       # 删除键（不存在则 KeyError）
removed = user.pop("email", None)      # 删除并返回值（不存在返回默认值）

# 遍历
for key, value in user.items():        # 键值对
    print(f"{key}: {value}")

for key in user:                       # 只遍历键
    print(key)

for value in user.values():            # 只遍历值
    print(value)

# 判断键是否存在
if "name" in user:
    print("Has name")
```

### 7.2 进阶操作

```python
# update：批量更新/合并
user = {"name": "Jerry", "age": 30}
user.update({"age": 31, "city": "Beijing"})
print(user)  # {'name': 'Jerry', 'age': 31, 'city': 'Beijing'}

# | 合并运算符（Python 3.9+）
defaults = {"theme": "dark", "lang": "zh"}
custom = {"lang": "en", "font_size": 14}
config = defaults | custom
print(config)  # {'theme': 'dark', 'lang': 'en', 'font_size': 14}

# setdefault：键不存在时设置默认值，存在则不修改
user.setdefault("email", "unknown@example.com")

# 用 dict() 构造函数
d = dict(name="Alice", age=25)     # 键名是合法标识符时可以这样写
d = dict([("a", 1), ("b", 2)])     # 从键值对列表构建

# 嵌套字典
users = {
    "u001": {"name": "Alice", "age": 25},
    "u002": {"name": "Bob", "age": 30},
}
print(users["u001"]["name"])  # Alice
```

---

## 八、集合（set）

无序、不重复、可变。用于去重和集合运算：

```python
# 创建
s = {1, 2, 3}
s2 = set([1, 2, 2, 3, 3])  # 从列表去重 → {1, 2, 3}
empty = set()                # 空集合（不能用 {}，那是空字典）

# 添加/删除
s.add(4)            # 添加元素
s.discard(99)       # 删除元素（不存在也不报错，推荐）
s.remove(4)         # 删除元素（不存在则 KeyError）
s.pop()             # 弹出任意一个元素

# 集合运算
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

print(a | b)   # {1, 2, 3, 4, 5, 6}  并集
print(a & b)   # {3, 4}              交集
print(a - b)   # {1, 2}              差集（a 有 b 没有）
print(a ^ b)   # {1, 2, 5, 6}        对称差集（互相没有的）

# 子集/超集判断
print({1, 2}.issubset({1, 2, 3}))    # True  是子集
print({1, 2, 3}.issuperset({1, 2}))  # True  是超集

# 实用场景：去重
names = ["Alice", "Bob", "Alice", "Charlie", "Bob"]
unique = list(set(names))  # ["Alice", "Bob", "Charlie"]（顺序可能变）

# 实用场景：成员判断（比列表快得多）
valid_ids = {1001, 1002, 1003, 1004}
print(1002 in valid_ids)  # True（O(1) 时间复杂度）

# frozenset：不可变集合（可以作为字典的 key）
fs = frozenset([1, 2, 3])
```

---

## 九、推导式（Comprehensions）

用一行替代 for 循环生成新的数据结构：

### 9.1 列表推导式

```python
numbers = [1, 2, 3, 4, 5]

# 基本：[表达式 for 变量 in 可迭代对象]
squares = [n ** 2 for n in numbers]
print(squares)  # [1, 4, 9, 16, 25]

# 带条件过滤：[表达式 for 变量 in 可迭代对象 if 条件]
evens = [n for n in numbers if n % 2 == 0]
print(evens)  # [2, 4]

# 带 if-else（注意位置不同）
labels = ["even" if n % 2 == 0 else "odd" for n in numbers]
print(labels)  # ['odd', 'even', 'odd', 'even', 'odd']

# 嵌套推导
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [x for row in matrix for x in row]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 9.2 字典推导式

```python
names = ["alice", "bob", "charlie"]

# {键表达式: 值表达式 for 变量 in 可迭代对象}
name_lengths = {name: len(name) for name in names}
print(name_lengths)  # {'alice': 5, 'bob': 3, 'charlie': 7}

# 反转字典
d = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in d.items()}
print(inverted)  # {1: 'a', 2: 'b', 3: 'c'}
```

### 9.3 集合推导式

```python
words = ["hello", "world", "hello", "python"]
lengths = {len(w) for w in words}
print(lengths)  # {5, 6}（自动去重）
```

---

## 十、条件与循环

### 10.1 条件判断

```python
score = 85
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "F"

# 三元表达式
status = "pass" if score >= 60 else "fail"
```

### 10.2 循环

```python
# for 循环
for fruit in ["apple", "banana", "cherry"]:
    print(fruit)

# range() 生成数字序列
for i in range(5):          # 0,1,2,3,4
    print(i)
for i in range(1, 6):       # 1,2,3,4,5
    print(i)
for i in range(0, 10, 2):   # 0,2,4,6,8（步长2）
    print(i)

# enumerate：同时获取索引和值
fruits = ["apple", "banana", "cherry"]
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
# 0: apple
# 1: banana
# 2: cherry

# zip：并行遍历多个序列
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name}: {age}")

# while 循环
count = 0
while count < 3:
    print(count)
    count += 1

# break 和 continue
for n in range(10):
    if n == 3:
        continue   # 跳过本次
    if n == 7:
        break      # 终止循环
    print(n)       # 0, 1, 2, 4, 5, 6

# for-else（不常用但有用：循环正常结束时执行 else）
for n in [2, 3, 5, 7]:
    if n % 2 == 0 and n != 2:
        print(f"{n} is not prime")
        break
else:
    print("All checked")  # 循环没被 break 时执行
```

---

## 十一、Walrus Operator `:=`（Python 3.8+）

海象运算符：在表达式中**同时赋值和使用**，减少重复计算：

```python
# 场景 1：避免重复调用
# 传统写法
data = input("Enter: ")
while data != "quit":
    print(f"You said: {data}")
    data = input("Enter: ")

# walrus 写法（更简洁）
while (data := input("Enter: ")) != "quit":
    print(f"You said: {data}")

# 场景 2：在条件中使用
import re
text = "My phone is 138-1234-5678"
if (match := re.search(r"\d{3}-\d{4}-\d{4}", text)):
    print(f"Found phone: {match.group()}")

# 场景 3：在推导式中避免重复计算
words = ["hello", "hi", "hey", "world", "wonderful"]
long_upper = [upper for w in words if len(upper := w.upper()) > 3]
print(long_upper)  # ['HELLO', 'WORLD', 'WONDERFUL']
```

---

## 十二、match/case 模式匹配（Python 3.10+）

比 if-elif 更强大的模式匹配，支持解构：

```python
# 基本值匹配
def describe_status(code: int) -> str:
    match code:
        case 200:
            return "OK"
        case 404:
            return "Not Found"
        case 500:
            return "Server Error"
        case _:                # _ 是通配符，匹配一切
            return f"Unknown: {code}"

print(describe_status(200))  # OK
print(describe_status(418))  # Unknown: 418

# 解构匹配（序列）
def describe_point(point):
    match point:
        case (0, 0):
            return "Origin"
        case (x, 0):
            return f"On X-axis at {x}"
        case (0, y):
            return f"On Y-axis at {y}"
        case (x, y):
            return f"Point({x}, {y})"

print(describe_point((3, 0)))   # On X-axis at 3
print(describe_point((0, 5)))   # On Y-axis at 5

# 解构匹配（字典）
def process_event(event: dict):
    match event:
        case {"type": "click", "x": x, "y": y}:
            print(f"Click at ({x}, {y})")
        case {"type": "keypress", "key": key}:
            print(f"Key pressed: {key}")
        case {"type": t}:
            print(f"Unknown event: {t}")

process_event({"type": "click", "x": 100, "y": 200})
# Click at (100, 200)

# guard 子句（附加条件）
def classify_age(age: int) -> str:
    match age:
        case n if n < 0:
            return "Invalid"
        case n if n < 18:
            return "Minor"
        case n if n < 65:
            return "Adult"
        case _:
            return "Senior"
```

---

## 十三、实操案例：JSON 数据处理器

### 目标

读取包含用户信息的 JSON 文件，清洗数据、过滤、排序、统计，综合运用今天学的核心语法。

### 步骤 1：创建测试数据文件

新建 `users.json`：

```json
[
  {"name": "  Alice ", "age": 17, "city": "Beijing", "tags": "student,music"},
  {"name": "Bob", "age": 25, "city": "Shanghai", "tags": "engineer,python"},
  {"name": "Charlie", "age": 15, "city": "Guangzhou", "tags": "student,art"},
  {"name": "Diana", "age": 30, "city": "Shenzhen", "tags": "engineer,go,python"},
  {"name": "Eve", "age": 22, "city": "Beijing", "tags": "designer,music"},
  {"name": "Frank", "age": 28, "city": "Shanghai", "tags": "engineer,typescript"}
]
```

### 步骤 2：编写脚本

新建 `json_processor.py`：

```python
import json

# ── 1. 读取 JSON 文件 ─────────────────────────
with open("users.json", "r", encoding="utf-8") as f:
    users = json.load(f)

print(f"总用户数: {len(users)}")

# ── 2. 数据清洗（字符串方法） ─────────────────
for user in users:
    user["name"] = user["name"].strip()        # 去除首尾空白
    user["tags"] = user["tags"].split(",")     # "a,b,c" → ["a", "b", "c"]

# ── 3. 过滤：只保留成年用户（列表推导式） ────
adults = [u for u in users if u["age"] >= 18]
print(f"成年用户数: {len(adults)}")

# ── 4. 按年龄排序 ─────────────────────────────
adults_sorted = sorted(adults, key=lambda u: u["age"])

# ── 5. 格式化输出 ─────────────────────────────
print("\n成年用户列表（按年龄排序）：")
for user in adults_sorted:
    tags_str = ", ".join(user["tags"])         # 列表 → 字符串
    print(f"  {user['name']:<10} {user['age']} 岁  {user['city']:<10} [{tags_str}]")

# ── 6. 集合运算：统计 ─────────────────────────
all_tags = {tag for u in adults for tag in u["tags"]}  # 集合推导式
print(f"\n所有标签: {all_tags}")

cities = {u["city"] for u in adults}  # 唯一城市
print(f"城市分布: {cities}")

# 找出同时有 python 和 engineer 标签的用户
py_engineers = [
    u["name"] for u in adults
    if {"python", "engineer"}.issubset(set(u["tags"]))
]
print(f"Python 工程师: {py_engineers}")

# ── 7. 字典推导式：按城市分组 ─────────────────
city_groups = {}
for u in adults:
    city_groups.setdefault(u["city"], []).append(u["name"])
print(f"\n按城市分组: {city_groups}")

# ── 8. match/case 处理命令 ────────────────────
def handle_command(cmd: str, data: list[dict]) -> None:
    match cmd.split():
        case ["count"]:
            print(f"共 {len(data)} 人")
        case ["find", name]:
            results = [u for u in data if name.lower() in u["name"].lower()]
            for u in results:
                print(f"  Found: {u['name']}, {u['age']}")
        case ["city", city_name]:
            in_city = [u for u in data if u["city"] == city_name]
            print(f"  {city_name}: {[u['name'] for u in in_city]}")
        case _:
            print(f"  未知命令: {cmd}")

print("\n命令演示：")
handle_command("count", adults)
handle_command("find bob", adults)
handle_command("city Shanghai", adults)

# ── 9. 写入结果 ───────────────────────────────
output = {
    "total": len(users),
    "adult_count": len(adults),
    "adults": adults_sorted,
    "unique_cities": list(cities),
    "all_tags": list(all_tags),
}

with open("adults.json", "w", encoding="utf-8") as f:
    json.dump(output, f, ensure_ascii=False, indent=2)

print("\n结果已写入 adults.json")
```

### 步骤 3：运行

```bash
python3 json_processor.py
```

预期输出：

```
总用户数: 6
成年用户数: 4

成年用户列表（按年龄排序）：
  Eve        22 岁  Beijing    [designer, music]
  Bob        25 岁  Shanghai   [engineer, python]
  Frank      28 岁  Shanghai   [engineer, typescript]
  Diana      30 岁  Shenzhen   [engineer, go, python]

所有标签: {'designer', 'music', 'python', 'engineer', 'typescript', 'go'}
城市分布: {'Shanghai', 'Beijing', 'Shenzhen'}
Python 工程师: ['Bob', 'Diana']

按城市分组: {'Shanghai': ['Bob', 'Frank'], 'Beijing': ['Eve'], 'Shenzhen': ['Diana']}

命令演示：
共 4 人
  Found: Bob, 25
  city Shanghai: ['Bob', 'Frank']

结果已写入 adults.json
```

### 代码要点回顾

| 知识点 | 在代码中的体现 |
|--------|----------------|
| `with open()` | 自动关闭文件 |
| `json.load()` / `json.dump()` | JSON 读写 |
| `str.strip()` / `str.split()` / `str.join()` | 字符串清洗 |
| 列表推导式 + 条件 | `[u for u in users if ...]` |
| 集合推导式 | `{tag for u in adults for tag in u["tags"]}` |
| 集合运算 `issubset` | 找同时有多个标签的用户 |
| `sorted()` + `lambda` | 按指定字段排序 |
| `dict.setdefault()` | 安全地按 key 分组 |
| `match/case` + 序列解构 | 命令分发 |
| f-string 格式化 | `:<10` 左对齐、`:,` 千分位 |

---

## 十四、今日 Checklist

- [ ] `python3 --version` 输出 3.12+
- [ ] 成功创建并激活虚拟环境
- [ ] 能说出 6 种 falsy 值
- [ ] 掌握字符串 split/join/strip/replace/find
- [ ] 理解列表 copy vs 赋值（浅拷贝陷阱）
- [ ] 能用集合做去重和交集/并集运算
- [ ] 能用 walrus operator 简化 while 循环
- [ ] 能用 match/case 做模式匹配
- [ ] `json_processor.py` 运行无报错
- [ ] `adults.json` 文件被正确生成
- [ ] git commit：`git add . && git commit -m "day01: json processor"`

> 📅 最后更新：2026-04-07
