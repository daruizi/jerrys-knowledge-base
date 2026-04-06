# Day 05 — 错误处理 & 健壮性

> 目标：掌握 Python 完整的错误处理体系，学会用 logging 取代 print，写出生产级别的健壮代码。

---

## 一、异常体系回顾与深入

### 1.1 Python 异常层级

```
BaseException
├── SystemExit          # sys.exit() 触发
├── KeyboardInterrupt   # Ctrl+C 触发
└── Exception           # ← 通常只捕获这一层及其子类
    ├── ValueError      # 值不合法（int("abc")）
    ├── TypeError       # 类型错误（1 + "a"）
    ├── KeyError        # 字典键不存在
    ├── IndexError      # 列表索引越界
    ├── FileNotFoundError  # 文件不存在（OSError 子类）
    ├── AttributeError  # 对象无此属性
    └── ...
```

**原则：捕获尽量具体的异常，而不是用 `except Exception` 一网打尽。**

### 1.2 完整的 try/except/else/finally

```python
def read_number(filepath: str) -> int | None:
    try:
        with open(filepath) as f:
            content = f.read().strip()
            return int(content)
    except FileNotFoundError:
        print(f"文件不存在: {filepath}")
        return None
    except ValueError:
        print(f"文件内容不是数字: {content!r}")
        return None
    except PermissionError:
        print(f"没有读取权限: {filepath}")
        return None
    else:
        # 仅当 try 块没有发生任何异常时执行
        print("读取成功")
    finally:
        # 无论是否发生异常都执行（清理资源）
        print("函数结束")
```

### 1.3 异常链（保留原始上下文）

```python
def load_config(path: str) -> dict:
    try:
        with open(path) as f:
            import json
            return json.load(f)
    except FileNotFoundError as e:
        # raise ... from e 保留原始异常信息
        raise RuntimeError(f"配置文件 {path} 加载失败") from e
```

---

## 二、自定义异常

为项目定义语义清晰的异常类，而不是复用通用的 `ValueError` / `RuntimeError`：

```python
# exceptions.py

class AppError(Exception):
    """项目基础异常，所有自定义异常继承此类"""
    pass


class ConfigError(AppError):
    """配置相关错误"""
    def __init__(self, key: str, reason: str) -> None:
        self.key = key
        self.reason = reason
        super().__init__(f"配置错误 [{key}]: {reason}")


class DataProcessError(AppError):
    """数据处理错误"""
    def __init__(self, row: int, message: str) -> None:
        self.row = row
        super().__init__(f"第 {row} 行处理失败: {message}")


# 使用
try:
    raise ConfigError("DATABASE_URL", "不能为空")
except ConfigError as e:
    print(e)          # 配置错误 [DATABASE_URL]: 不能为空
    print(e.key)      # DATABASE_URL
```

**好处：**
- 调用方可以精确捕获特定错误类型
- 可以携带结构化数据（`e.key`、`e.row`）
- 文档化了项目可能出现哪些错误

---

## 三、logging — 替代 print 的正确方式

`print` 的问题：无法控制级别、没有时间戳、无法写入文件、生产环境难以关闭。

### 3.1 基本用法

```python
import logging

# 配置（通常在程序入口处配置一次）
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logger = logging.getLogger(__name__)  # 每个模块一个 logger，名字用模块名

# 五个日志级别（从低到高）
logger.debug("调试信息，生产环境通常不显示")
logger.info("正常流程信息")
logger.warning("警告，程序仍可运行")
logger.error("错误，某个操作失败")
logger.critical("严重错误，程序可能无法继续")
```

### 3.2 同时输出到控制台和文件

```python
import logging
from pathlib import Path


def setup_logging(log_file: str = "app.log", level: int = logging.INFO) -> None:
    Path(log_file).parent.mkdir(exist_ok=True)

    logger = logging.getLogger()
    logger.setLevel(level)

    fmt = logging.Formatter(
        "%(asctime)s [%(levelname)-8s] %(name)s: %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S"
    )

    # 控制台输出
    console = logging.StreamHandler()
    console.setFormatter(fmt)
    logger.addHandler(console)

    # 文件输出
    file_handler = logging.FileHandler(log_file, encoding="utf-8")
    file_handler.setFormatter(fmt)
    logger.addHandler(file_handler)


# 调用后，所有模块的 logger 都会同时输出到控制台和文件
setup_logging("logs/app.log")
```

### 3.3 在异常处理中使用 logging

```python
logger = logging.getLogger(__name__)

try:
    result = risky_operation()
except Exception as e:
    logger.error("操作失败: %s", e)          # 推荐：% 格式化，不会提前计算
    logger.exception("操作失败（含堆栈）")    # 自动打印完整 traceback
    raise
```

---

## 四、防御性编程模式

### 4.1 类型检查与前置条件

```python
def calculate_average(numbers: list[float]) -> float:
    if not isinstance(numbers, list):
        raise TypeError(f"期望 list，得到 {type(numbers).__name__}")
    if not numbers:
        raise ValueError("列表不能为空")
    if not all(isinstance(n, (int, float)) for n in numbers):
        raise ValueError("列表元素必须都是数字")
    return sum(numbers) / len(numbers)
```

### 4.2 安全访问字典和列表

```python
data = {"user": {"name": "Alice"}}

# 危险（可能 KeyError）
name = data["user"]["email"]

# 安全
name = data.get("user", {}).get("email", "unknown")

# 对于列表，用默认值
items = [1, 2, 3]
value = items[10] if 10 < len(items) else None
```

### 4.3 输入验证

```python
from pathlib import Path


def process_file(filepath: str) -> None:
    path = Path(filepath)

    if not path.exists():
        raise FileNotFoundError(f"文件不存在: {filepath}")
    if not path.is_file():
        raise ValueError(f"不是文件: {filepath}")
    if path.suffix not in {".csv", ".json", ".txt"}:
        raise ValueError(f"不支持的文件类型: {path.suffix}")
    if path.stat().st_size == 0:
        raise ValueError(f"文件为空: {filepath}")

    # 通过验证，安全处理
    ...
```

---

## 五、实操案例：健壮的文件处理器

### 目标

读取一个包含数字的 CSV 文件，计算每列的统计数据。要求：完善的错误处理 + logging 输出。

新建 `day05/robust_processor.py`：

```python
import csv
import logging
from pathlib import Path
from dataclasses import dataclass

# ── 自定义异常 ─────────────────────────────
class ProcessError(Exception):
    pass

class InvalidFileError(ProcessError):
    pass

class InvalidDataError(ProcessError):
    def __init__(self, row: int, col: str, value: str) -> None:
        self.row = row
        self.col = col
        self.value = value
        super().__init__(f"第 {row} 行 [{col}] 无效值: {value!r}")


# ── 日志配置 ───────────────────────────────
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)-8s] %(message)s",
    datefmt="%H:%M:%S",
)
logger = logging.getLogger(__name__)


# ── 数据模型 ───────────────────────────────
@dataclass
class ColumnStats:
    name: str
    count: int
    total: float
    minimum: float
    maximum: float

    @property
    def average(self) -> float:
        return self.total / self.count if self.count else 0.0


# ── 核心逻辑 ───────────────────────────────
def validate_file(filepath: str) -> Path:
    path = Path(filepath)
    if not path.exists():
        raise InvalidFileError(f"文件不存在: {filepath}")
    if path.suffix.lower() != ".csv":
        raise InvalidFileError(f"只支持 CSV 文件，得到: {path.suffix}")
    if path.stat().st_size == 0:
        raise InvalidFileError(f"文件为空: {filepath}")
    return path


def parse_csv(path: Path) -> list[dict]:
    records = []
    errors = 0

    with open(path, encoding="utf-8") as f:
        reader = csv.DictReader(f)
        for i, row in enumerate(reader, start=2):  # 从第2行开始（第1行是header）
            try:
                # 把所有值转换为浮点数
                parsed = {k: float(v) for k, v in row.items() if v.strip()}
                records.append(parsed)
            except ValueError as e:
                logger.warning("第 %d 行包含非数字数据，已跳过: %s", i, e)
                errors += 1

    if errors:
        logger.warning("共跳过 %d 行无效数据", errors)
    return records


def compute_stats(records: list[dict]) -> list[ColumnStats]:
    if not records:
        raise ProcessError("没有有效数据")

    columns = records[0].keys()
    stats = []

    for col in columns:
        values = [r[col] for r in records if col in r]
        if not values:
            logger.warning("列 [%s] 没有有效值，跳过", col)
            continue
        stats.append(ColumnStats(
            name=col,
            count=len(values),
            total=sum(values),
            minimum=min(values),
            maximum=max(values),
        ))
    return stats


def print_report(stats: list[ColumnStats]) -> None:
    print(f"\n{'列名':<15} {'数量':>6} {'平均值':>10} {'最小值':>10} {'最大值':>10}")
    print("-" * 55)
    for s in stats:
        print(f"{s.name:<15} {s.count:>6} {s.average:>10.2f} {s.minimum:>10.2f} {s.maximum:>10.2f}")


def main(filepath: str) -> None:
    logger.info("开始处理文件: %s", filepath)

    try:
        path = validate_file(filepath)
        records = parse_csv(path)
        logger.info("成功读取 %d 行数据", len(records))

        stats = compute_stats(records)
        print_report(stats)

    except InvalidFileError as e:
        logger.error("文件验证失败: %s", e)
    except ProcessError as e:
        logger.error("处理失败: %s", e)
    except Exception as e:
        logger.exception("未预期的错误")
        raise
    else:
        logger.info("处理完成")


if __name__ == "__main__":
    import sys
    if len(sys.argv) < 2:
        print("用法: python3 robust_processor.py <csv文件>")
        sys.exit(1)
    main(sys.argv[1])
```

### 测试数据

新建 `day05/data/scores.csv`：

```csv
math,english,science
85,92,78
90,88,95
invalid,76,82
72,0,
88,95,91
```

### 运行

```bash
python3 robust_processor.py data/scores.csv
```

输出：

```
10:30:01 [INFO    ] 开始处理文件: data/scores.csv
10:30:01 [WARNING ] 第 4 行包含非数字数据，已跳过: could not convert string to float: 'invalid'
10:30:01 [INFO    ] 成功读取 4 行数据
10:30:01 [INFO    ] 处理完成

列名              数量     平均值      最小值      最大值
-------------------------------------------------------
math               4      83.75      72.00      90.00
english            4      87.75      76.00      95.00
science            3      88.00      78.00      95.00
```

测试各种错误情况：

```bash
python3 robust_processor.py nofile.csv       # 文件不存在
python3 robust_processor.py data/scores.txt  # 类型错误
```

---

## 六、代码要点回顾

| 知识点 | 体现 |
|--------|------|
| 自定义异常层级 | `ProcessError` → `InvalidFileError` / `InvalidDataError` |
| `try/else` | `else` 块只在无异常时执行 |
| `logger.exception()` | 自动打印 traceback |
| 捕获具体异常 | 分别处理 `InvalidFileError` / `ProcessError` / `Exception` |
| `%s` 格式化 in logging | 延迟求值，性能更好 |
| 前置验证 | `validate_file()` 在处理前检查所有前提条件 |

---

## 七、今日 Checklist

- [ ] 理解 `try/except/else/finally` 各自的执行时机
- [ ] 写过自定义异常类
- [ ] 用 `logging` 替代 `print` 输出日志
- [ ] `robust_processor.py` 能正确处理有效/无效数据
- [ ] 测试各种错误场景，日志输出正确
- [ ] git commit：`git add . && git commit -m "day05: robust file processor with logging"`

> 📅 最后更新：2026-04-06
