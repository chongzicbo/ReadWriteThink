## 1.函数执行时间统计装饰器

### 功能：

用于统计函数执行的时间，常用于性能优化。

### 示例代码：

```python
from time import perf_counter
from functools import wraps
from typing import List


def timeit(loop: int = 1):
    """
    函数执行失败时，重试

    :param loop: 循环执行次数
    :return:
    """

    # 校验参数，参数值不正确时使用默认参数
    if loop < 1:
        loop = 1

    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            sum_time: float = 0.0
            for i in range(loop):
                start_time: float = perf_counter()
                ret = func(*args, **kwargs)
                end_time: float = perf_counter()
                sum_time += end_time - start_time

            print(
                f"函数({func.__name__})共执行{loop}次，平均执行时间 {(sum_time/loop):.3f} 秒"
            )
            return ret

        return wrapper

    return decorator

```

在 Python 开发中，工具函数（Utility Functions）是提高代码效率和可维护性的重要组成部分。这些函数可以帮助我们完成常见的任务，如时间统计、类型检查、字符串处理、文件操作等。以下是一些常用的工具函数及其使用场景。

---

## 2. 类型检查装饰器

### 功能：
用于检查函数参数的类型是否符合预期。

### 示例代码：

```python
from functools import wraps

def type_check(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        sig = inspect.signature(func)
        params = sig.parameters
        for i, (arg_name, arg_value) in enumerate(zip(params, args)):
            param = params[arg_name]
            if param.annotation != inspect.Parameter.empty and not isinstance(arg_value, param.annotation):
                raise TypeError(f"参数 {arg_name} 的类型应为 {param.annotation}，但传入的是 {type(arg_value)}")
        return func(*args, **kwargs)
    return wrapper

@type_check
def add(a: int, b: int) -> int:
    return a + b

print(add(1, 2))  # 正常
print(add(1, "2"))  # 抛出 TypeError
```

---

## 3. 重试装饰器

### 功能：
用于在函数执行失败时自动重试。

### 示例代码：

```python
from functools import wraps

def retry(max_retries=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            retries = 0
            while retries < max_retries:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    retries += 1
                    print(f"重试 {retries}/{max_retries}: {e}")
                    time.sleep(delay)
            raise Exception(f"达到最大重试次数 {max_retries}")
        return wrapper
    return decorator

@retry(max_retries=3, delay=1)
def flaky_function():
    if random.randint(0, 1) == 0:
        raise Exception("随机失败")
    return "成功"

print(flaky_function())
```

---

## 4. 批量处理函数

### 功能：
用于将一个列表或迭代器分批处理。

### 示例代码：

```python
def batch_process(data, batch_size=10):
    for i in range(0, len(data), batch_size):
        yield data[i:i + batch_size]

data = list(range(1, 101))
for batch in batch_process(data, batch_size=20):
    print(batch)
```

### 输出：
```
[1, 2, 3, ..., 20]
[21, 22, 33, ..., 40]
...
[81, 82, 83, ..., 100]
```

---

## 5. 文件操作工具函数

### 功能：
用于读取、写入和处理文件。

### 示例代码：

```python
def read_file(file_path):
    with open(file_path, 'r', encoding='utf-8') as f:
        return f.read()

def write_file(file_path, content):
    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(content)

def append_file(file_path, content):
    with open(file_path, 'a', encoding='utf-8') as f:
        f.write(content)

# 示例
write_file("example.txt", "Hello, World!\n")
append_file("example.txt", "This is a new line.\n")
print(read_file("example.txt"))
```

---

## 6. 字符串处理工具函数

### 功能：
用于字符串的格式化、分割、替换等操作。

### 示例代码：

```python
def format_string(template, **kwargs):
    return template.format(**kwargs)

def split_string(text, delimiter=","):
    return text.split(delimiter)

def replace_string(text, old, new):
    return text.replace(old, new)

# 示例
template = "My name is {name} and I am {age} years old."
print(format_string(template, name="Alice", age=30))

text = "apple,banana,cherry"
print(split_string(text))

text = "Hello, World!"
print(replace_string(text, "World", "Python"))
```

---

## 7. 日志记录工具函数

### 功能：
用于记录程序的运行日志。

### 示例代码：

```python
import logging
from logging.handlers import RotatingFileHandler

def setup_logger(module_name, log_dir="logs"):
    """
    根据模块名称配置日志记录器，并自动生成日志文件名。

    :param module_name: 模块名称，用于生成日志文件名
    :param log_dir: 日志文件存储目录，默认为 "logs"
    :return: 配置好的日志记录器
    """
    # 创建日志记录器
    logger = logging.getLogger(module_name)
    logger.setLevel(logging.DEBUG)  # 设置全局日志级别为 DEBUG

    # 配置日志格式化
    formatter = logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")

    # 配置控制台处理器（输出到终端）
    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.INFO)  # 控制台日志级别为 INFO
    console_handler.setFormatter(formatter)
    logger.addHandler(console_handler)

    # 配置文件处理器（根据模块名称生成日志文件名）
    log_file = f"{log_dir}/{module_name}.log"
    file_handler = RotatingFileHandler(
        log_file,  # 日志文件路径
        maxBytes=1024 * 1024,  # 每个日志文件的最大大小（1MB）
        backupCount=5  # 保留的旧日志文件数量
    )
    file_handler.setLevel(logging.DEBUG)  # 文件日志级别为 DEBUG
    file_handler.setFormatter(formatter)
    logger.addHandler(file_handler)

    return logger
```

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

## 8. 缓存工具函数

### 功能：
用于缓存函数的返回值，避免重复计算。

### 示例代码：

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(10))  # 计算并缓存
print(fibonacci(10))  # 直接从缓存中获取
```

---

## 9. 随机数据生成工具函数

### 功能：
用于生成随机数据，如随机字符串、随机数等。

### 示例代码：

```python
import random
import string

def random_string(length=10):
    return ''.join(random.choices(string.ascii_letters + string.digits, k=length))

def random_number(start=1, end=100):
    return random.randint(start, end)

# 示例
print(random_string(15))
print(random_number(1, 10))
```

---

## 10. 并发执行工具函数

### 功能：
用于并发执行多个任务。

### 示例代码：

```python
import concurrent.futures

def task(n):
    return n * n

def parallel_execute(tasks):
    with concurrent.futures.ThreadPoolExecutor() as executor:
        results = list(executor.map(task, tasks))
    return results

# 示例
tasks = [1, 2, 3, 4, 5]
print(parallel_execute(tasks))
```

---

## 总结

以上是 Python 中常用的一些工具函数，涵盖了时间统计、类型检查、重试机制、文件操作、字符串处理、日志记录、缓存、随机数据生成和并发执行等多个方面。这些工具函数可以帮助我们提高代码的可读性、可维护性和性能。

根据具体需求，你可以选择合适的工具函数，或者将它们组合起来，构建更复杂的工具链。