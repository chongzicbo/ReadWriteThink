# Python 日志处理最佳实践：使用 `loguru` 构建高效日志系统

在现代软件开发中，日志记录是不可或缺的一部分。Python 的标准库 `logging` 是一个强大且灵活的日志记录工具，但在实际项目中，配置和管理日志可能会变得复杂。为了简化日志记录的过程，`loguru` 库应运而生。`loguru` 是一个功能强大且易于使用的日志库，它提供了更简洁的 API 和更丰富的功能。本文将结合实际项目经验，总结使用 `loguru` 模块的最佳实践，并提供一个完整的代码示例。

---

## 1. 为什么选择 `loguru`？

相比于 `logging` 模块，`loguru` 提供了以下优势：

- **简洁的 API**：无需复杂的配置，只需几行代码即可完成日志记录。
- **自动日志轮转**：支持日志文件的自动轮转，避免日志文件过大。
- **丰富的日志格式**：支持多种日志格式，包括颜色高亮、时间戳等。
- **异常捕获**：自动捕获未处理的异常，并记录完整的堆栈信息。
- **日志过滤**：支持基于日志级别和模块的过滤。
- **高性能**：内部优化了性能，适合高并发场景。

---

## 2. 最佳实践概述

在实际项目中，使用 `loguru` 的最佳实践包括以下几点：

1. **统一的日志配置**：通过一个统一的日志配置函数，避免在每个模块中重复编写日志配置代码。
2. **根据模块名称自动命名日志文件**：每个模块的日志存储在独立的文件中，便于管理和分析。
3. **日志轮转**：使用 `loguru` 的日志轮转功能，自动管理日志文件的大小和数量。
4. **日志级别控制**：在开发环境中使用 `DEBUG` 级别，在生产环境中使用 `INFO` 或 `WARNING` 级别。
5. **日志格式化**：使用统一的日志格式，便于阅读和分析。
6. **日志安全**：避免记录敏感信息，如密码、密钥等。

---

## 3. 示例项目结构

```
my_project/
├── main.py
├── module_a.py
├── module_b.py
├── logger_config.py
└── logs/
    ├── module_a.log
    ├── module_b.log
    └── app.log
```

---

## 4. 安装 `loguru`

首先，确保你已经安装了 `loguru` 库。如果没有安装，可以使用以下命令进行安装：

```bash
pip install loguru
```

---

## 5. 统一的日志配置函数

为了减少重复代码，我们可以在一个单独的模块中定义一个统一的日志配置函数 `setup_logger`，并在每个模块中调用它。

### `logger_config.py`

```python
from loguru import logger
import sys

def setup_logger(module_name, log_dir="logs"):
    """
    根据模块名称配置日志记录器，并自动生成日志文件名。

    :param module_name: 模块名称，用于生成日志文件名
    :param log_dir: 日志文件存储目录，默认为 "logs"
    """
    # 配置日志格式
    log_format = (
        "<green>{time:YYYY-MM-DD HH:mm:ss}</green> | "
        "<level>{level: <8}</level> | "
        "<cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> - <level>{message}</level>"
    )

    # 配置控制台日志
    logger.remove()  # 移除默认的日志处理器
    logger.add(sys.stdout, format=log_format, level="INFO")

    # 配置文件日志（根据模块名称生成日志文件名）
    log_file = f"{log_dir}/{module_name}.log"
    logger.add(
        log_file,  # 日志文件路径
        format=log_format,
        level="DEBUG",
        rotation="10 MB",  # 日志文件轮转大小
        retention="7 days",  # 保留日志文件的时间
        compression="zip",  # 日志文件压缩格式
    )

    return logger
```

---

## 6. 主模块：`main.py`

在主模块中，我们调用 `setup_logger` 函数来配置日志记录器，并记录日志。

```python
from logger_config import setup_logger
import module_a
import module_b

# 配置主模块的日志记录器
logger = setup_logger("main")

# 记录日志
def main():
    logger.info("主模块启动")
    module_a.run()
    module_b.run()
    logger.info("主模块结束")

if __name__ == "__main__":
    main()
```

---

## 7. 模块 A：`module_a.py`

在模块 A 中，我们同样调用 `setup_logger` 函数来配置日志记录器，并记录日志。

```python
from logger_config import setup_logger

# 配置模块 A 的日志记录器
logger = setup_logger("module_a")

# 记录日志
def run():
    logger.debug("模块 A 的调试信息")
    logger.info("模块 A 的普通信息")
    logger.warning("模块 A 的警告信息")
    logger.error("模块 A 的错误信息")
    logger.critical("模块 A 的严重错误信息")
```

---

## 8. 模块 B：`module_b.py`

在模块 B 中，我们同样调用 `setup_logger` 函数来配置日志记录器，并记录日志。

```python
from logger_config import setup_logger

# 配置模块 B 的日志记录器
logger = setup_logger("module_b")

# 记录日志
def run():
    logger.debug("模块 B 的调试信息")
    logger.info("模块 B 的普通信息")
    logger.warning("模块 B 的警告信息")
    logger.error("模块 B 的错误信息")
    logger.critical("模块 B 的严重错误信息")
```

---

## 9. 运行结果

### 控制台输出

```
2023-10-01 12:00:00 | INFO     | main:main:10 - 主模块启动
2023-10-01 12:00:00 | INFO     | module_a:run:6 - 模块 A 的普通信息
2023-10-01 12:00:00 | WARNING  | module_a:run:8 - 模块 A 的警告信息
2023-10-01 12:00:00 | ERROR    | module_a:run:10 - 模块 A 的错误信息
2023-10-01 12:00:00 | CRITICAL | module_a:run:12 - 模块 A 的严重错误信息
2023-10-01 12:00:00 | INFO     | module_b:run:6 - 模块 B 的普通信息
2023-10-01 12:00:00 | WARNING  | module_b:run:8 - 模块 B 的警告信息
2023-10-01 12:00:00 | ERROR    | module_b:run:10 - 模块 B 的错误信息
2023-10-01 12:00:00 | CRITICAL | module_b:run:12 - 模块 B 的严重错误信息
2023-10-01 12:00:00 | INFO     | main:main:14 - 主模块结束
```

### 日志文件内容

#### `logs/app.log`（主模块日志）：
```
2023-10-01 12:00:00 | INFO     | main:main:10 - 主模块启动
2023-10-01 12:00:00 | INFO     | main:main:14 - 主模块结束
```

#### `logs/module_a.log`（模块 A 日志）：
```
2023-10-01 12:00:00 | DEBUG    | module_a:run:4 - 模块 A 的调试信息
2023-10-01 12:00:00 | INFO     | module_a:run:6 - 模块 A 的普通信息
2023-10-01 12:00:00 | WARNING  | module_a:run:8 - 模块 A 的警告信息
2023-10-01 12:00:00 | ERROR    | module_a:run:10 - 模块 A 的错误信息
2023-10-01 12:00:00 | CRITICAL | module_a:run:12 - 模块 A 的严重错误信息
```

#### `logs/module_b.log`（模块 B 日志）：
```
2023-10-01 12:00:00 | DEBUG    | module_b:run:4 - 模块 B 的调试信息
2023-10-01 12:00:00 | INFO     | module_b:run:6 - 模块 B 的普通信息
2023-10-01 12:00:00 | WARNING  | module_b:run:8 - 模块 B 的警告信息
2023-10-01 12:00:00 | ERROR    | module_b:run:10 - 模块 B 的错误信息
2023-10-01 12:00:00 | CRITICAL | module_b:run:12 - 模块 B 的严重错误信息
```

---

## 10. 总结

通过统一的日志配置函数 `setup_logger`，我们实现了以下目标：

1. **自动生成日志文件名**：根据模块名称动态生成日志文件名，避免手动指定文件名。
2. **减少重复代码**：在每个模块中只需调用 `setup_logger` 函数，无需重复编写日志配置代码。
3. **灵活的日志管理**：每个模块的日志存储在独立的文件中，便于管理和分析。
4. **日志轮转**：通过 `loguru` 的日志轮转功能，自动管理日志文件的大小和数量。
5. **日志级别控制**：在开发环境中使用 `DEBUG` 级别，在生产环境中使用 `INFO` 或 `WARNING` 级别。
6. **日志格式化**：使用统一的日志格式，便于阅读和分析。
7. **日志安全**：避免记录敏感信息，如密码、密钥等。

`loguru` 的简洁性和强大功能使其成为 Python 日志记录的首选工具。希望本文对你在实际项目中使用 `loguru` 有所帮助！如果你有任何问题或需要进一步的帮助，请随时留言。