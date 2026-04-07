# Day 02 — 函数与函数式编程

> 目标：全面掌握 Python 函数定义、函数式编程范式（lambda/map/filter/生成器/闭包），以及 functools/itertools 标准库。

---

## 一、函数基础

### 1.1 定义与类型注解

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"

print(greet("Jerry"))  # Hello, Jerry!
```

**格式说明：**
- `name: str` — 参数类型注解
- `-> str` — 返回值类型注解
- 类型注解不影响运行，但能让编辑器和 AI 提供更好的补全与检查

### 1.2 默认参数

```python
def create_user(name: str, age: int = 18, city: str = "Beijing") -> dict:
    return {"name": name, "age": age, "city": city}

print(create_user("Alice"))                        # age=18, city=Beijing
print(create_user("Bob", 25))                      # age=25, city=Beijing
print(create_user("Charlie", city="Shanghai"))     # 关键字传参跳过 age
```

**规则：有默认值的参数必须放在无默认值参数的后面。**

**陷阱：可变对象做默认值**

```python
# ❌ 错误写法：所有调用共享同一个列表
def add_item(item, items=[]):
    items.append(item)
    return items

print(add_item("a"))  # ['a']
print(add_item("b"))  # ['a', 'b'] ← 不是 ['b']！

# ✅ 正确写法：用 None 做哨兵值
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

### 1.3 `*args` 和 `**kwargs`

`*args` — 接收任意数量的**位置参数**，打包为元组：

```python
def total(*numbers: int) -> int:
    return sum(numbers)

print(total(1, 2, 3))         # 6
print(total(10, 20, 30, 40))  # 100
```

`**kwargs` — 接收任意数量的**关键字参数**，打包为字典：

```python
def print_info(**kwargs) -> None:
    for key, value in kwargs.items():
        print(f"  {key}: {value}")

print_info(name="Alice", age=25, city="Beijing")
```

组合使用（顺序固定：普通参数 → `*args` → 关键字参数 → `**kwargs`）：

```python
def log(level: str, *messages: str, sep: str = " ", **context) -> None:
    text = sep.join(messages)
    print(f"[{level}] {text}")
    if context:
        print(f"  Context: {context}")

log("INFO", "Server", "started", port=8080, host="localhost")
# [INFO] Server started
#   Context: {'port': 8080, 'host': 'localhost'}
```

**解包调用（`*` 和 `**` 用于调用侧）：**

```python
def add(a: int, b: int, c: int) -> int:
    return a + b + c

args = [1, 2, 3]
print(add(*args))       # 6（列表解包为位置参数）

kwargs = {"a": 1, "b": 2, "c": 3}
print(add(**kwargs))    # 6（字典解包为关键字参数）
```

### 1.4 返回多个值

Python 函数可以返回元组，实现"多返回值"：

```python
def min_max(numbers: list[int]) -> tuple[int, int]:
    return min(numbers), max(numbers)

low, high = min_max([3, 1, 4, 1, 5, 9])
print(f"min={low}, max={high}")  # min=1, max=9
```

### 1.5 仅关键字参数（Keyword-Only）

`*` 之后的参数必须用关键字传递：

```python
def connect(host: str, port: int, *, timeout: int = 30, ssl: bool = True):
    print(f"Connecting to {host}:{port} (timeout={timeout}, ssl={ssl})")

connect("localhost", 8080)                    # OK
connect("localhost", 8080, timeout=10)        # OK
# connect("localhost", 8080, 10)             # TypeError! timeout 必须用关键字
```

### 1.6 仅位置参数（Positional-Only，Python 3.8+）

`/` 之前的参数只能按位置传递：

```python
def divide(a: float, b: float, /) -> float:
    return a / b

print(divide(10, 3))     # OK
# print(divide(a=10, b=3))  # TypeError! 只能按位置传
```

---

## 二、Lambda 函数

匿名函数，用于需要一个简短函数的场景：

```python
# 语法：lambda 参数: 表达式
square = lambda x: x ** 2
print(square(5))  # 25

# 多参数
add = lambda a, b: a + b
print(add(3, 4))  # 7

# 最常见用法：作为 sorted/map/filter 的 key/func 参数
users = [
    {"name": "Charlie", "age": 30},
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 28},
]

# 按 age 排序
sorted_users = sorted(users, key=lambda u: u["age"])
print([u["name"] for u in sorted_users])  # ['Alice', 'Bob', 'Charlie']

# 按 name 长度降序
sorted_users = sorted(users, key=lambda u: -len(u["name"]))

# lambda 适合一行能写完的简单逻辑
# 复杂逻辑请用 def 定义命名函数
```

---

## 三、高阶函数：map / filter / sorted / reduce

高阶函数 = 接受函数作为参数的函数。

### 3.1 map — 映射

对每个元素应用函数，返回迭代器：

```python
numbers = [1, 2, 3, 4, 5]

# map + lambda
squares = list(map(lambda x: x ** 2, numbers))
print(squares)  # [1, 4, 9, 16, 25]

# map + 命名函数
def fahrenheit_to_celsius(f: float) -> float:
    return round((f - 32) * 5 / 9, 1)

temps_f = [32, 68, 100, 212]
temps_c = list(map(fahrenheit_to_celsius, temps_f))
print(temps_c)  # [0.0, 20.0, 37.8, 100.0]

# 等价的推导式写法（通常更 Pythonic）
temps_c = [fahrenheit_to_celsius(f) for f in temps_f]
```

### 3.2 filter — 过滤

保留返回 True 的元素：

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8]

evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4, 6, 8]

# 等价推导式
evens = [x for x in numbers if x % 2 == 0]

# filter(None, ...) 过滤 falsy 值
mixed = [0, 1, "", "hello", None, True, [], [1]]
truthy = list(filter(None, mixed))
print(truthy)  # [1, 'hello', True, [1]]
```

### 3.3 sorted — 排序（进阶）

```python
# 多级排序：先按 age 升序，age 相同按 name 字母序
users = [
    {"name": "Charlie", "age": 25},
    {"name": "Alice", "age": 30},
    {"name": "Bob", "age": 25},
]
result = sorted(users, key=lambda u: (u["age"], u["name"]))
# [Bob(25), Charlie(25), Alice(30)]

# 使用 operator 模块替代 lambda（更高效）
from operator import itemgetter
result = sorted(users, key=itemgetter("age", "name"))
```

### 3.4 reduce — 累积

将序列归约为单个值：

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

# 求乘积：1 * 2 * 3 * 4 * 5
product = reduce(lambda acc, x: acc * x, numbers)
print(product)  # 120

# 执行过程：
# step 1: acc=1, x=2 → 2
# step 2: acc=2, x=3 → 6
# step 3: acc=6, x=4 → 24
# step 4: acc=24, x=5 → 120

# 带初始值
total = reduce(lambda acc, x: acc + x, numbers, 0)
print(total)  # 15

# 实际应用：把嵌套列表拍平
nested = [[1, 2], [3, 4], [5, 6]]
flat = reduce(lambda acc, lst: acc + lst, nested, [])
print(flat)  # [1, 2, 3, 4, 5, 6]
```

**何时用 map/filter vs 推导式？**
- 简单转换/过滤 → 推导式更 Pythonic，可读性更好
- 已有现成函数 → map/filter 更简洁：`list(map(str.upper, words))`
- 链式操作多步 → 考虑生成器管道（见下文）

---

## 四、生成器（Generator）

### 4.1 生成器函数（yield）

生成器是**惰性迭代器**，一次只产生一个值，节省内存：

```python
def count_up(start: int, end: int):
    """生成从 start 到 end 的数字"""
    current = start
    while current <= end:
        yield current       # 暂停函数，产出一个值
        current += 1        # 下次 next() 时从这里继续

# 使用
gen = count_up(1, 5)
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3

# 通常在 for 循环中使用
for n in count_up(1, 5):
    print(n)  # 1, 2, 3, 4, 5
```

### 4.2 生成器 vs 列表：内存对比

```python
import sys

# 列表：一次性加载所有元素到内存
nums_list = [i for i in range(1_000_000)]
print(sys.getsizeof(nums_list))  # ~8,000,000 字节（约 8MB）

# 生成器：几乎不占内存
nums_gen = (i for i in range(1_000_000))
print(sys.getsizeof(nums_gen))   # ~200 字节

# 生成器表达式：把 [] 换成 ()
squares_gen = (x ** 2 for x in range(1_000_000))
# 可以像列表一样遍历，但不能索引或重复遍历
```

### 4.3 实用生成器示例

```python
def read_large_file(filepath: str):
    """逐行读取大文件，不会一次加载到内存"""
    with open(filepath, "r") as f:
        for line in f:
            yield line.strip()

def fibonacci():
    """无限斐波那契数列"""
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# 取前 10 个斐波那契数
from itertools import islice
fib_10 = list(islice(fibonacci(), 10))
print(fib_10)  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

def batch(iterable, size: int):
    """将可迭代对象分批，每批 size 个"""
    batch = []
    for item in iterable:
        batch.append(item)
        if len(batch) == size:
            yield batch
            batch = []
    if batch:  # 剩余不足一批的
        yield batch

for b in batch(range(10), 3):
    print(b)  # [0,1,2], [3,4,5], [6,7,8], [9]
```

### 4.4 yield from（委托生成器）

```python
def flatten(nested_list):
    """递归展平嵌套列表"""
    for item in nested_list:
        if isinstance(item, list):
            yield from flatten(item)  # 委托给子生成器
        else:
            yield item

data = [1, [2, 3], [4, [5, 6]], 7]
print(list(flatten(data)))  # [1, 2, 3, 4, 5, 6, 7]
```

---

## 五、迭代器协议

理解迭代器的底层原理——for 循环的本质：

```python
# 迭代器协议：实现 __iter__ 和 __next__ 两个方法
class Countdown:
    """从 n 倒数到 1"""
    def __init__(self, n: int):
        self.n = n

    def __iter__(self):
        return self  # 迭代器返回自身

    def __next__(self) -> int:
        if self.n <= 0:
            raise StopIteration   # 告诉 for 循环停止
        value = self.n
        self.n -= 1
        return value

# 用在 for 循环中
for i in Countdown(5):
    print(i)  # 5, 4, 3, 2, 1

# for 循环的本质等价于：
it = iter(Countdown(3))   # 调用 __iter__
while True:
    try:
        value = next(it)  # 调用 __next__
        print(value)
    except StopIteration:
        break
```

> 生成器函数自动实现了迭代器协议，所以不需要手写类。大多数时候用生成器就够了。

---

## 六、闭包（Closure）

闭包 = 函数 + 它引用的外部变量。内层函数"记住"了外层函数的局部变量：

```python
# 基本闭包
def make_multiplier(factor: int):
    """返回一个乘以 factor 的函数"""
    def multiply(x: int) -> int:
        return x * factor  # factor 是外层变量，被"捕获"
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)
print(double(5))   # 10
print(triple(5))   # 15

# 实用示例：计数器工厂
def make_counter(start: int = 0):
    count = start
    def counter():
        nonlocal count    # 声明要修改外层变量（不是创建新变量）
        count += 1
        return count
    return counter

c1 = make_counter()
print(c1())  # 1
print(c1())  # 2
print(c1())  # 3

c2 = make_counter(10)  # 独立的计数器
print(c2())  # 11

# 实用示例：可配置的日志器
def make_logger(prefix: str):
    def log(message: str) -> None:
        from datetime import datetime
        now = datetime.now().strftime("%H:%M:%S")
        print(f"[{now}] {prefix}: {message}")
    return log

db_log = make_logger("DB")
api_log = make_logger("API")

db_log("Connection established")  # [14:30:00] DB: Connection established
api_log("Request received")       # [14:30:00] API: Request received
```

**闭包是装饰器的基础**——Day 06 会用闭包构建装饰器。

---

## 七、functools 精要

### 7.1 `partial` — 固定部分参数

```python
from functools import partial

def power(base: int, exp: int) -> int:
    return base ** exp

square = partial(power, exp=2)   # 固定 exp=2
cube = partial(power, exp=3)     # 固定 exp=3

print(square(5))  # 25
print(cube(3))    # 27

# 实际用途：简化重复的函数调用
import json
dump_pretty = partial(json.dumps, ensure_ascii=False, indent=2)
print(dump_pretty({"name": "你好"}))
```

### 7.2 `lru_cache` — 缓存计算结果

```python
from functools import lru_cache

@lru_cache(maxsize=128)  # 最多缓存 128 个结果
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# 不加缓存：fibonacci(35) 需要几秒
# 加了缓存：瞬间完成（重复子问题直接返回缓存值）
print(fibonacci(35))  # 9227465

# 查看缓存状态
print(fibonacci.cache_info())
# CacheInfo(hits=33, misses=36, maxsize=128, currsize=36)
```

### 7.3 `wraps` — 保留被装饰函数的信息

```python
from functools import wraps

# 没有 wraps 的问题：
def my_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def hello():
    """Say hello"""
    pass

print(hello.__name__)  # "wrapper" ← 丢失了原函数名！
print(hello.__doc__)   # None     ← 丢失了文档字符串！

# 加上 wraps 解决：
def my_decorator(func):
    @wraps(func)         # 把 func 的元信息复制到 wrapper
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

> `wraps` 在 Day 06 装饰器章节会详细使用。

---

## 八、itertools 精要

itertools 提供高效的迭代器工具，处理数据流非常强大：

### 8.1 组合工具

```python
from itertools import chain, islice, zip_longest

# chain：串联多个迭代器
list1 = [1, 2, 3]
list2 = [4, 5, 6]
list3 = [7, 8]
print(list(chain(list1, list2, list3)))  # [1, 2, 3, 4, 5, 6, 7, 8]

# islice：对迭代器切片（不支持负索引，但不需要加载全部数据）
from itertools import count  # 无限计数器
print(list(islice(count(1), 5)))      # [1, 2, 3, 4, 5]（取前5个）
print(list(islice(count(1), 3, 8)))   # [4, 5, 6, 7, 8]（跳过前3个，取到第8个）

# zip_longest：长度不同时用填充值（zip 默认截断到最短）
names = ["Alice", "Bob", "Charlie"]
scores = [90, 85]
print(list(zip_longest(names, scores, fillvalue=0)))
# [('Alice', 90), ('Bob', 85), ('Charlie', 0)]
```

### 8.2 排列组合

```python
from itertools import product, combinations, permutations

# product：笛卡尔积
colors = ["red", "blue"]
sizes = ["S", "M", "L"]
print(list(product(colors, sizes)))
# [('red','S'), ('red','M'), ('red','L'), ('blue','S'), ('blue','M'), ('blue','L')]

# combinations：组合（不考虑顺序）
print(list(combinations([1, 2, 3, 4], 2)))
# [(1,2), (1,3), (1,4), (2,3), (2,4), (3,4)]

# permutations：排列（考虑顺序）
print(list(permutations([1, 2, 3], 2)))
# [(1,2), (1,3), (2,1), (2,3), (3,1), (3,2)]
```

### 8.3 分组

```python
from itertools import groupby

# groupby：按 key 分组（数据必须先排序！）
data = [
    {"name": "Alice", "dept": "Engineering"},
    {"name": "Bob", "dept": "Engineering"},
    {"name": "Charlie", "dept": "Marketing"},
    {"name": "Diana", "dept": "Marketing"},
    {"name": "Eve", "dept": "Engineering"},
]

# 先排序
data.sort(key=lambda x: x["dept"])

# 再分组
for dept, members in groupby(data, key=lambda x: x["dept"]):
    names = [m["name"] for m in members]
    print(f"{dept}: {names}")
# Engineering: ['Alice', 'Bob', 'Eve']
# Marketing: ['Charlie', 'Diana']
```

---

## 九、高阶函数模式：函数作为参数和返回值

理解"函数是一等公民"——可以像普通值一样传递和返回：

```python
# 函数作为参数
def apply_operation(numbers: list[int], operation) -> list[int]:
    return [operation(n) for n in numbers]

print(apply_operation([1, 2, 3], lambda x: x ** 2))  # [1, 4, 9]
print(apply_operation([1, 2, 3], lambda x: x + 10))  # [11, 12, 13]
print(apply_operation([1, 2, 3], str))                # ['1', '2', '3']

# 函数作为返回值（闭包的应用）
def make_validator(min_val: int, max_val: int):
    """返回一个范围检查函数"""
    def validate(value: int) -> bool:
        return min_val <= value <= max_val
    return validate

is_valid_age = make_validator(0, 150)
is_valid_score = make_validator(0, 100)

print(is_valid_age(25))    # True
print(is_valid_age(200))   # False
print(is_valid_score(85))  # True

# 函数组合
def compose(*funcs):
    """将多个函数组合为一个：compose(f, g, h)(x) = f(g(h(x)))"""
    def composed(x):
        for f in reversed(funcs):
            x = f(x)
        return x
    return composed

process = compose(str.upper, str.strip, lambda s: s.replace(",", " "))
print(process("  hello, world  "))  # "HELLO  WORLD"
```

> 这个模式是装饰器的前置知识——装饰器本质就是"接受函数、返回函数"的高阶函数。

---

## 十、实操案例：数据处理管道

### 目标

用生成器 + 函数式编程构建一个数据处理管道：读取 CSV → 清洗 → 过滤 → 转换 → 分组统计。

### 步骤 1：创建测试数据

新建 `data/sales.csv`：

```csv
date,product,category,amount,region
2026-01-01,iPhone,Electronics,9999,North
2026-01-02,T-Shirt,Clothing,199,South
2026-01-03,MacBook,Electronics,12999,North
2026-01-04,Jeans,Clothing,599,East
2026-01-05,AirPods,Electronics,1299,South
2026-01-06,Jacket,Clothing,899,North
2026-01-07,iPad,Electronics,4999,East
2026-01-08,Sneakers,Clothing,1299,South
2026-01-09,Watch,Electronics,2999,North
2026-01-10,Hat,Clothing,99,East
```

### 步骤 2：编写管道

新建 `data_pipeline.py`：

```python
import csv
from functools import partial, reduce
from itertools import groupby
from operator import itemgetter

# ── 第一层：生成器读取（惰性，不一次加载全部） ──

def read_csv(filepath: str):
    """逐行读取 CSV，返回字典生成器"""
    with open(filepath, "r", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for row in reader:
            yield row

# ── 第二层：清洗/转换函数 ──

def clean_record(record: dict) -> dict:
    """清洗单条记录：去空白、转类型"""
    return {
        "date": record["date"].strip(),
        "product": record["product"].strip(),
        "category": record["category"].strip(),
        "amount": float(record["amount"]),
        "region": record["region"].strip(),
    }

def make_filter(field: str, value: str):
    """闭包：生成字段过滤函数"""
    def filterer(record: dict) -> bool:
        return record[field] == value
    return filterer

# ── 第三层：管道组装 ──

def pipeline(filepath: str) -> None:
    # Step 1: 读取（生成器）
    raw_records = read_csv(filepath)

    # Step 2: 清洗（map）
    clean_records = map(clean_record, raw_records)

    # Step 3: 过滤高价商品（filter + lambda）
    high_value = filter(lambda r: r["amount"] >= 500, clean_records)

    # Step 4: 收集为列表（到这里才真正执行上面的生成器链）
    records = list(high_value)
    print(f"高价商品（≥500）: {len(records)} 条\n")

    # Step 5: 按类别分组统计（itertools.groupby）
    records.sort(key=itemgetter("category"))
    print("── 按类别统计 ──")
    for category, group in groupby(records, key=itemgetter("category")):
        items = list(group)
        total = reduce(lambda acc, r: acc + r["amount"], items, 0)
        avg = total / len(items)
        print(f"  {category:<15} {len(items)} 件  "
              f"总额 ¥{total:>10,.0f}  均价 ¥{avg:>8,.0f}")

    # Step 6: 按地区分组统计
    records.sort(key=itemgetter("region"))
    print("\n── 按地区统计 ──")
    for region, group in groupby(records, key=itemgetter("region")):
        items = list(group)
        total = sum(r["amount"] for r in items)
        products = ", ".join(r["product"] for r in items)
        print(f"  {region:<8} ¥{total:>10,.0f}  [{products}]")

    # Step 7: 使用闭包生成的过滤器
    is_electronics = make_filter("category", "Electronics")
    electronics = list(filter(is_electronics, records))

    # 找到最贵的产品
    most_expensive = max(electronics, key=lambda r: r["amount"])
    print(f"\n最贵电子产品: {most_expensive['product']} ¥{most_expensive['amount']:,.0f}")

    # Step 8: 用 partial 简化输出
    format_price = partial("{:>10,.0f}".format)
    total_revenue = sum(r["amount"] for r in records)
    print(f"高价商品总营收: ¥{format_price(total_revenue)}")


if __name__ == "__main__":
    pipeline("data/sales.csv")
```

### 步骤 3：运行

```bash
mkdir -p data
# （先创建上面的 CSV 文件）
python3 data_pipeline.py
```

预期输出：

```
高价商品（≥500）: 8 条

── 按类别统计 ──
  Clothing         3 件  总额 ¥     2,797  均价 ¥      932
  Electronics      5 件  总额 ¥    32,295  均价 ¥    6,459

── 按地区统计 ──
  East     ¥     5,598  [Jeans, iPad]
  North    ¥    26,896  [iPhone, MacBook, Jacket, Watch]
  South    ¥     2,598  [AirPods, Sneakers]

最贵电子产品: MacBook ¥12,999
高价商品总营收: ¥    35,092
```

### 代码要点回顾

| 知识点 | 体现 |
|--------|------|
| 生成器函数（yield） | `read_csv()` 逐行读取不占内存 |
| map + 命名函数 | `map(clean_record, raw_records)` |
| filter + lambda | `filter(lambda r: r["amount"] >= 500, ...)` |
| 闭包工厂 | `make_filter("category", "Electronics")` |
| reduce 累加 | `reduce(lambda acc, r: acc + r["amount"], ...)` |
| itertools.groupby | 按类别、按地区分组 |
| partial 固定参数 | `partial("{:>10,.0f}".format)` |
| operator.itemgetter | 替代 `lambda x: x["field"]` |

---

## 十一、今日 Checklist

- [ ] 理解 `*args` 和 `**kwargs` 的区别和使用场景
- [ ] 理解可变默认值陷阱（`def f(items=[])` 的问题）
- [ ] 能用 lambda 配合 sorted/map/filter
- [ ] 能写生成器函数（yield），理解与列表的内存差异
- [ ] 理解闭包的概念，能写出 make_xxx 工厂函数
- [ ] 了解 functools.partial 和 lru_cache 的用法
- [ ] 了解 itertools.chain/islice/groupby 的用法
- [ ] `data_pipeline.py` 运行输出正确
- [ ] git commit：`git add . && git commit -m "day02: data pipeline"`

> 📅 最后更新：2026-04-07
