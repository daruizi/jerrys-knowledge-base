# Day 01 — Python 环境 & 核心语法

> 目标：搭好环境，掌握 Python 最常用的基础语法，完成第一个实用脚本。

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

---

## 二、核心语法

### 2.1 变量与基本类型

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
```

### 2.2 字符串与 f-string

```python
name = "Jerry"
age = 30

# 旧方式（不推荐）
print("Hello, " + name + "!")
print("Age: %d" % age)

# f-string（推荐，Python 3.6+）
print(f"Hello, {name}!")
print(f"Age: {age}, next year: {age + 1}")
print(f"Score: {9.5:.2f}")    # 保留2位小数 → 9.50
print(f"Upper: {name.upper()}")  # 可以调用方法 → JERRY
```

### 2.3 列表（List）

有序、可重复、可修改：

```python
fruits = ["apple", "banana", "cherry"]

# 访问
print(fruits[0])       # apple
print(fruits[-1])      # cherry（负索引从末尾算）
print(fruits[1:3])     # ['banana', 'cherry']（切片）

# 修改
fruits.append("mango")        # 末尾添加
fruits.insert(1, "blueberry") # 指定位置插入
fruits.remove("banana")       # 删除指定值
popped = fruits.pop()         # 弹出末尾元素

# 遍历
for fruit in fruits:
    print(fruit)

# 常用操作
print(len(fruits))     # 长度
print("apple" in fruits)  # True/False 判断包含
fruits.sort()          # 排序（原地）
```

**列表推导式**——用一行替代 for 循环生成新列表：

```python
numbers = [1, 2, 3, 4, 5]

# 普通写法
squares = []
for n in numbers:
    squares.append(n ** 2)

# 推导式写法（更 Pythonic）
squares = [n ** 2 for n in numbers]
print(squares)  # [1, 4, 9, 16, 25]

# 带条件过滤
evens = [n for n in numbers if n % 2 == 0]
print(evens)  # [2, 4]
```

### 2.4 字典（Dict）

键值对，无序（Python 3.7+ 保持插入顺序）：

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
user["age"] = 31
user["email"] = "jerry@example.com"   # 新增键

# 删除
del user["city"]

# 遍历
for key, value in user.items():
    print(f"{key}: {value}")

# 判断键是否存在
if "email" in user:
    print("Has email")
```

**字典推导式**：

```python
names = ["alice", "bob", "charlie"]

# 生成 {名字: 名字长度} 的字典
name_lengths = {name: len(name) for name in names}
print(name_lengths)  # {'alice': 5, 'bob': 3, 'charlie': 7}
```

### 2.5 条件与循环

```python
# 条件判断
score = 85
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "F"
print(grade)  # B

# 三元表达式
status = "pass" if score >= 60 else "fail"

# while 循环
count = 0
while count < 3:
    print(count)
    count += 1

# range() 生成数字序列
for i in range(5):        # 0,1,2,3,4
    print(i)

for i in range(1, 6):     # 1,2,3,4,5
    print(i)

for i in range(0, 10, 2): # 0,2,4,6,8（步长2）
    print(i)
```

---

## 三、实操案例：JSON 处理脚本

### 目标

读取一个包含用户信息的 JSON 文件，过滤出年龄 >= 18 的用户，按年龄排序后输出。

### 步骤 1：创建测试数据文件

新建 `users.json`：

```json
[
  {"name": "Alice", "age": 17, "city": "Beijing"},
  {"name": "Bob", "age": 25, "city": "Shanghai"},
  {"name": "Charlie", "age": 15, "city": "Guangzhou"},
  {"name": "Diana", "age": 30, "city": "Shenzhen"},
  {"name": "Eve", "age": 22, "city": "Beijing"}
]
```

### 步骤 2：编写脚本

新建 `json_processor.py`：

```python
import json

# 1. 读取 JSON 文件
with open("users.json", "r", encoding="utf-8") as f:
    users = json.load(f)

print(f"总用户数: {len(users)}")

# 2. 过滤：只保留年龄 >= 18 的用户
adults = [user for user in users if user["age"] >= 18]

print(f"成年用户数: {len(adults)}")

# 3. 按年龄升序排序
adults_sorted = sorted(adults, key=lambda user: user["age"])

# 4. 格式化输出
print("\n成年用户列表（按年龄排序）：")
for user in adults_sorted:
    print(f"  {user['name']:<10} {user['age']} 岁  {user['city']}")

# 5. 将结果写入新文件
output = {
    "total": len(users),
    "adults": adults_sorted
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
总用户数: 5
成年用户数: 3

成年用户列表（按年龄排序）：
  Eve        22 岁  Beijing
  Bob        25 岁  Shanghai
  Diana      30 岁  Shenzhen

结果已写入 adults.json
```

### 代码要点回顾

| 知识点 | 在代码中的体现 |
|--------|----------------|
| `with open()` | 自动关闭文件，推荐写法 |
| `json.load()` | 将 JSON 文件解析为 Python 列表/字典 |
| 列表推导式 + 条件 | `[user for user in users if ...]` |
| `sorted()` + `lambda` | 按指定字段排序 |
| `json.dump()` | 将 Python 对象写入 JSON 文件 |
| `ensure_ascii=False` | 允许中文正常输出 |

---

## 四、今日 Checklist

- [ ] `python3 --version` 输出 3.12+
- [ ] 成功创建并激活虚拟环境
- [ ] `json_processor.py` 运行无报错
- [ ] `adults.json` 文件被正确生成
- [ ] git commit：`git add . && git commit -m "day01: json processor"`

> 📅 最后更新：2026-04-06
