# Day 02 — 函数、模块与错误处理

> 目标：掌握函数定义、模块拆分和异常处理，写出结构清晰、健壮的代码。

---

## 一、函数

### 1.1 基本定义

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
print(create_user("Charlie", city="Shanghai"))     # 关键字传参
```

**规则：有默认值的参数必须放在无默认值参数的后面。**

### 1.3 `*args` 和 `**kwargs`

`*args` — 接收任意数量的**位置参数**，打包为元组：

```python
def total(*numbers: int) -> int:
    return sum(numbers)

print(total(1, 2, 3))       # 6
print(total(10, 20, 30, 40))  # 100
```

`**kwargs` — 接收任意数量的**关键字参数**，打包为字典：

```python
def print_info(**kwargs) -> None:
    for key, value in kwargs.items():
        print(f"  {key}: {value}")

print_info(name="Alice", age=25, city="Beijing")
```

组合使用（顺序固定：普通参数 → `*args` → `**kwargs`）：

```python
def log(level: str, *messages: str, **context) -> None:
    print(f"[{level}] {' '.join(messages)}")
    if context:
        print(f"  Context: {context}")

log("INFO", "Server", "started", port=8080, host="localhost")
```

### 1.4 返回多个值

Python 函数可以返回元组，实现"多返回值"：

```python
def min_max(numbers: list[int]) -> tuple[int, int]:
    return min(numbers), max(numbers)

low, high = min_max([3, 1, 4, 1, 5, 9])
print(f"min={low}, max={high}")  # min=1, max=9
```

---

## 二、模块与导入

### 2.1 什么是模块？

每个 `.py` 文件就是一个模块。模块让代码按职责分文件，避免一个文件几千行。

**项目结构示例：**

```
day02/
├── main.py          # 程序入口
├── utils.py         # 工具函数
└── data/
    └── sales.csv    # 数据文件
```

### 2.2 导入方式

```python
# 导入整个模块
import os
print(os.getcwd())

# 导入模块中的特定内容（推荐，明确知道用什么）
from pathlib import Path
p = Path("data/sales.csv")

# 导入并重命名
import json as j
data = j.loads('{"key": "value"}')

# 从自己写的模块导入
# 假设 utils.py 里有 read_csv 函数
from utils import read_csv
```

### 2.3 `if __name__ == "__main__"`

每个 Python 文件既可以被直接运行，也可以被其他文件 import。
这行代码确保某些代码只在**直接运行时**执行，import 时不执行：

```python
# utils.py
def add(a: int, b: int) -> int:
    return a + b

if __name__ == "__main__":
    # 直接运行 utils.py 时才执行
    print(add(1, 2))  # 测试用
```

---

## 三、错误处理

### 3.1 异常基础

```python
# 不处理异常 → 程序崩溃
result = 10 / 0        # ZeroDivisionError
name = None
print(name.upper())    # AttributeError
```

### 3.2 `try / except`

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("除数不能为零")

# 捕获多种异常
try:
    value = int("abc")
except ValueError:
    print("无法转换为整数")
except TypeError:
    print("类型错误")

# 捕获所有异常（谨慎使用，会掩盖真实错误）
try:
    risky_operation()
except Exception as e:
    print(f"发生错误: {e}")
```

### 3.3 `else` 和 `finally`

```python
try:
    result = int("42")
except ValueError:
    print("转换失败")
else:
    # 没有异常时执行
    print(f"转换成功: {result}")
finally:
    # 无论如何都执行（常用于关闭文件/连接）
    print("清理完成")
```

### 3.4 文件操作中的异常处理

```python
from pathlib import Path

def read_file(filepath: str) -> str | None:
    try:
        with open(filepath, "r", encoding="utf-8") as f:
            return f.read()
    except FileNotFoundError:
        print(f"文件不存在: {filepath}")
        return None
    except PermissionError:
        print(f"没有读取权限: {filepath}")
        return None
```

---

## 四、实操案例：CSV 分析 CLI 工具

### 目标

从命令行传入 CSV 文件路径，读取销售数据，按类别统计金额并输出报告。

### 步骤 1：创建测试数据

新建 `data/sales.csv`：

```csv
date,product,category,amount
2026-01-01,iPhone,Electronics,9999
2026-01-02,T-Shirt,Clothing,199
2026-01-03,MacBook,Electronics,12999
2026-01-04,Jeans,Clothing,599
2026-01-05,AirPods,Electronics,1299
2026-01-06,Jacket,Clothing,899
2026-01-07,iPad,Electronics,4999
```

### 步骤 2：创建工具模块

新建 `utils.py`：

```python
import csv
from pathlib import Path


def read_csv(filepath: str) -> list[dict]:
    """读取 CSV 文件，返回字典列表"""
    path = Path(filepath)
    if not path.exists():
        raise FileNotFoundError(f"文件不存在: {filepath}")
    if path.suffix != ".csv":
        raise ValueError(f"不是 CSV 文件: {filepath}")

    with open(path, "r", encoding="utf-8") as f:
        reader = csv.DictReader(f)
        return list(reader)


def group_by_category(records: list[dict]) -> dict[str, float]:
    """按 category 统计 amount 总和"""
    result: dict[str, float] = {}
    for record in records:
        category = record["category"]
        amount = float(record["amount"])
        result[category] = result.get(category, 0) + amount
    return result


def print_report(stats: dict[str, float]) -> None:
    """格式化打印报告"""
    print("\n" + "=" * 35)
    print(f"{'类别':<15} {'销售额':>15}")
    print("-" * 35)
    total = 0.0
    for category, amount in sorted(stats.items(), key=lambda x: -x[1]):
        print(f"{category:<15} ¥{amount:>12,.2f}")
        total += amount
    print("=" * 35)
    print(f"{'合计':<15} ¥{total:>12,.2f}")
```

### 步骤 3：创建入口文件

新建 `main.py`：

```python
import sys
from utils import read_csv, group_by_category, print_report


def main() -> None:
    # 从命令行获取文件路径
    if len(sys.argv) < 2:
        print("用法: python3 main.py <csv文件路径>")
        print("示例: python3 main.py data/sales.csv")
        sys.exit(1)

    filepath = sys.argv[1]

    try:
        records = read_csv(filepath)
        print(f"读取成功，共 {len(records)} 条记录")

        stats = group_by_category(records)
        print_report(stats)

    except FileNotFoundError as e:
        print(f"错误: {e}")
        sys.exit(1)
    except ValueError as e:
        print(f"错误: {e}")
        sys.exit(1)


if __name__ == "__main__":
    main()
```

### 步骤 4：运行

```bash
python3 main.py data/sales.csv
```

预期输出：

```
读取成功，共 7 条记录

===================================
类别              销售额
-----------------------------------
Electronics      ¥     29,296.00
Clothing         ¥      1,697.00
===================================
合计             ¥     30,993.00
```

测试错误处理：

```bash
python3 main.py data/notexist.csv   # 文件不存在
python3 main.py                      # 没传参数
```

### 代码要点回顾

| 知识点 | 体现 |
|--------|------|
| `sys.argv` | 获取命令行参数 |
| `pathlib.Path` | 跨平台路径处理 |
| `csv.DictReader` | 将 CSV 每行读为字典 |
| 函数拆分到模块 | `utils.py` + `main.py` |
| 异常处理链 | `try/except` 分类处理不同错误 |
| `dict.get(key, default)` | 安全读取字典，避免 KeyError |

---

## 五、今日 Checklist

- [ ] 理解 `*args` 和 `**kwargs` 的区别
- [ ] `main.py` 运行后输出正确报告
- [ ] 测试文件不存在时的错误提示
- [ ] git commit：`git add . && git commit -m "day02: csv analyzer cli"`

> 📅 最后更新：2026-04-06
