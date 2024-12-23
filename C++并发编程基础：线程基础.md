## C++并发编程基础之线程基础

---

#### 1. 引言

##### 1.1 什么是并发编程
并发编程是指在同一时间段内，多个任务同时执行的编程方式。并发编程可以提高程序的执行效率，尤其是在多核处理器上，能够充分利用硬件资源。并发编程的核心在于如何管理多个任务的执行顺序、资源共享和同步问题。

##### 1.2 C++中的并发支持
C++11引入了标准库中的并发支持，提供了`std::thread`、`std::mutex`、`std::condition_variable`等工具，使得开发者可以方便地进行多线程编程。C++的并发库设计简洁且功能强大，能够满足大多数并发编程的需求。

##### 1.3 线程在并发编程中的重要性
线程是并发编程的基本单位，每个线程代表一个独立的执行流。通过创建多个线程，程序可以在同一时间内执行多个任务，从而提高程序的响应速度和处理能力。线程的管理、同步和资源共享是并发编程中的关键问题。

---

#### 2. 线程的创建与销毁

##### 2.1 `std::thread`简介

###### 2.1.1 `std::thread`的基本概念
`std::thread`是C++标准库中用于表示线程的类。通过`std::thread`，开发者可以创建、管理和销毁线程。每个`std::thread`对象代表一个独立的执行流。

###### 2.1.2 `std::thread`的构造函数
`std::thread`的构造函数允许传入一个可调用对象（如函数、Lambda表达式、函数对象等），并可选地传递参数给该可调用对象。

###### 2.1.3 `std::thread`的默认构造函数
`std::thread`的默认构造函数创建一个未关联任何线程的`std::thread`对象。这种线程对象是不可执行的，直到通过`std::thread::operator=`或移动构造函数关联到一个实际的线程。

##### 2.2 线程的创建

###### 2.2.1 创建一个简单的线程

```cpp
#include <iostream>
#include <thread>

void threadFunction() {
    std::cout << "Hello from thread!" << std::endl;
}

int main() {
    // 创建一个线程，执行threadFunction
    std::thread t(threadFunction);

    // 等待线程执行完毕
    t.join();

    std::cout << "Thread finished execution." << std::endl;
    return 0;
}
```

**代码解析：**
- `std::thread t(threadFunction);`：创建一个线程对象`t`，并关联到`threadFunction`函数。
- `t.join();`：主线程等待`t`线程执行完毕后再继续执行。

###### 2.2.2 传递参数给线程函数

```cpp
#include <iostream>
#include <thread>

void threadFunction(int x, const std::string& str) {
    std::cout << "Parameter 1: " << x << ", Parameter 2: " << str << std::endl;
}

int main() {
    // 创建一个线程，传递参数给threadFunction
    std::thread t(threadFunction, 42, "Hello");

    // 等待线程执行完毕
    t.join();

    std::cout << "Thread finished execution." << std::endl;
    return 0;
}
```

**代码解析：**
- `std::thread t(threadFunction, 42, "Hello");`：创建线程时，传递两个参数`42`和`"Hello"`给`threadFunction`。

###### 2.2.3 使用Lambda表达式创建线程

```cpp
#include <iostream>
#include <thread>

int main() {
    // 使用Lambda表达式创建线程
    std::thread t([]() {
        std::cout << "Hello from Lambda thread!" << std::endl;
    });

    // 等待线程执行完毕
    t.join();

    std::cout << "Thread finished execution." << std::endl;
    return 0;
}
```

**代码解析：**
- `std::thread t([]() { ... });`：使用Lambda表达式作为线程的执行体。

##### 2.3 线程的销毁

###### 2.3.1 线程的析构函数
当`std::thread`对象被销毁时，如果线程还未结束，程序会调用`std::terminate()`终止程序。因此，必须确保线程在销毁前已经结束或被正确管理。

###### 2.3.2 线程的资源管理
线程的资源管理通常通过`join()`或`detach()`来实现。`join()`会阻塞主线程直到子线程结束，而`detach()`则允许子线程独立运行。

###### 2.3.3 线程的异常安全
在异常情况下，线程的资源管理尤为重要。可以使用RAII（资源获取即初始化）技术来确保线程资源的安全释放。

###### 2.3.4 代码示例和解析

```cpp
#include <iostream>
#include <thread>

void threadFunction() {
    std::cout << "Thread is running." << std::endl;
}

int main() {
    std::thread t(threadFunction);

    try {
        // 模拟异常情况
        throw std::runtime_error("An error occurred!");
    } catch (...) {
        // 确保线程在异常情况下也能正确结束
        if (t.joinable()) {
            t.join();
        }
        throw; // 重新抛出异常
    }

    return 0;
}
```

**代码解析：**
- 在异常处理块中，检查线程是否可连接（`joinable()`），如果可连接则调用`join()`，确保线程资源被正确释放。

---

#### 3. 线程的基本操作

##### 3.1 线程的启动

###### 3.1.1 线程的自动启动
当`std::thread`对象创建时，关联的线程会自动启动。

###### 3.1.2 线程的启动时机
线程的启动时机由操作系统调度器决定，开发者无法精确控制线程的启动时间。

##### 3.2 线程的等待

###### 3.2.1 `join()`方法

```cpp
#include <iostream>
#include <thread>

void threadFunction() {
    std::cout << "Thread is running." << std::endl;
}

int main() {
    std::thread t(threadFunction);

    // 等待线程执行完毕
    t.join();

    std::cout << "Thread finished execution." << std::endl;
    return 0;
}
```

**代码解析：**
- `t.join();`：主线程等待`t`线程执行完毕后再继续执行。

###### 3.2.2 `joinable()`方法

```cpp
#include <iostream>
#include <thread>

int main() {
    std::thread t; // 默认构造函数创建的线程对象

    if (t.joinable()) {
        std::cout << "Thread is joinable." << std::endl;
    } else {
        std::cout << "Thread is not joinable." << std::endl;
    }

    return 0;
}
```

**代码解析：**
- `t.joinable()`：检查线程是否可连接。

###### 3.2.3 线程等待的注意事项
在调用`join()`之前，必须确保线程是可连接的，否则会导致未定义行为。

##### 3.3 线程的分离

###### 3.3.1 `detach()`方法

```cpp
#include <iostream>
#include <thread>

void threadFunction() {
    std::cout << "Thread is running." << std::endl;
}

int main() {
    std::thread t(threadFunction);

    // 分离线程
    t.detach();

    std::cout << "Thread detached." << std::endl;
    return 0;
}
```

**代码解析：**
- `t.detach();`：分离线程，使其独立运行。

###### 3.3.2 分离线程的特性
分离线程独立于创建它的线程运行，主线程无需等待其结束。

###### 3.3.3 分离线程的资源管理
分离线程的资源由操作系统管理，开发者无需手动释放。

##### 3.4 线程的标识

###### 3.4.1 `get_id()`方法

```cpp
#include <iostream>
#include <thread>

void threadFunction() {
    std::cout << "Thread ID: " << std::this_thread::get_id() << std::endl;
}

int main() {
    std::thread t(threadFunction);

    // 获取主线程ID
    std::cout << "Main thread ID: " << std::this_thread::get_id() << std::endl;

    t.join();
    return 0;
}
```

**代码解析：**
- `std::this_thread::get_id()`：获取当前线程的唯一标识符。

###### 3.4.2 线程标识的作用
线程标识用于区分不同的线程，常用于调试和日志记录。

---

#### 4. 线程的生命周期管理

##### 4.1 线程的生命周期概述
线程的生命周期从创建开始，到执行结束或被销毁结束。

##### 4.2 线程的异常处理

###### 4.2.1 线程中的异常传播
线程中的异常不会自动传播到主线程，必须通过特定机制（如`std::future`）来捕获。

###### 4.2.2 线程异常处理的策略
在多线程程序中，异常处理需要特别小心，通常建议在每个线程中捕获并处理异常。

###### 4.2.3 代码示例和解析

```cpp
#include <iostream>
#include <thread>
#include <stdexcept>

void threadFunction() {
    try {
        throw std::runtime_error("An error occurred in thread!");
    } catch (const std::exception& e) {
        std::cerr << "Thread exception: " << e.what() << std::endl;
    }
}

int main() {
    std::thread t(threadFunction);
    t.join();
    return 0;
}
```

**代码解析：**
- 在子线程中捕获并处理异常，避免异常传播到主线程。

##### 4.3 线程的资源竞争

###### 4.3.1 资源竞争的概念
资源竞争是指多个线程同时访问共享资源，可能导致数据不一致或未定义行为。

###### 4.3.2 避免资源竞争的方法
使用互斥锁（`std::mutex`）或原子操作（`std::atomic`）来保护共享资源。

###### 4.3.3 代码示例和解析

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;
int shared_data = 0;

void threadFunction() {
    std::lock_guard<std::mutex> lock(mtx);
    shared_data++;
}

int main() {
    std::thread t1(threadFunction);
    std::thread t2(threadFunction);

    t1.join();
    t2.join();

    std::cout << "Shared data: " << shared_data << std::endl;
    return 0;
}
```

**代码解析：**
- `std::lock_guard<std::mutex> lock(mtx);`：使用互斥锁保护`shared_data`的访问，避免资源竞争。

##### 4.4 线程的同步与互斥

###### 4.4.1 同步与互斥的基本概念
同步是指协调多个线程的执行顺序，互斥是指保护共享资源不被多个线程同时访问。

###### 4.4.2 使用`std::mutex`进行线程同步

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void threadFunction() {
    std::lock_guard<std::mutex> lock(mtx);
    std::cout << "Thread is running." << std::endl;
}

int main() {
    std::thread t1(threadFunction);
    std::thread t2(threadFunction);

    t1.join();
    t2.join();

    return 0;
}
```

**代码解析：**
- `std::lock_guard<std::mutex> lock(mtx);`：确保同一时间只有一个线程可以访问临界区。

##### 4.5 线程的终止

###### 4.5.1 线程的正常终止
线程执行完其任务后会自动终止。

###### 4.5.2 线程的异常终止
线程中抛出未捕获的异常会导致线程异常终止。

###### 4.5.3 线程终止的资源清理
线程终止后，操作系统会回收其资源。

###### 4.5.4 代码示例和解析

```cpp
#include <iostream>
#include <thread>

void threadFunction() {
    std::cout << "Thread is running." << std::endl;
}

int main() {
    std::thread t(threadFunction);
    t.join(); // 等待线程正常终止
    std::cout << "Thread finished execution." << std::endl;
    return 0;
}
```

**代码解析：**
- `t.join();`：确保线程正常终止并释放资源。

---

#### 5. 总结

##### 5.1 线程基础知识的回顾
本文介绍了C++并发编程中的线程基础知识，包括线程的创建、销毁、基本操作、生命周期管理等。通过代码示例和详细解析，帮助读者理解线程的基本概念和使用方法。

##### 5.2 线程管理的最佳实践
- 使用`join()`或`detach()`管理线程资源。
- 使用互斥锁保护共享资源，避免资源竞争。
- 在异常情况下，确保线程资源的安全释放。
- 使用线程标识符进行调试和日志记录。

