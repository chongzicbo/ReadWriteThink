# C++并发编程基础：并发与并行、多线程与多进程

## 一、引言

在现代计算机系统中，多核处理器的普及使得并发编程变得越来越重要。并发编程允许我们充分利用多核处理器的优势，提高程序的执行效率和响应速度。掌握并发编程技术不仅可以帮助我们编写更高效的代码，还能提升系统的资源利用率。

本文将重点介绍C++并发编程的基础知识，包括并发与并行的概念、多线程与多进程的区别，以及如何使用C++标准库进行多线程和多进程编程。

## 二、并发与并行

### 2.1 概念解析

#### 并发 (Concurrency)

**定义：** 并发是指两个或多个任务在同一时间段内交替执行，但在任意时刻只有一个任务在执行。

**举例：** 在单核处理器上，操作系统通过时间片轮转的方式，让多个任务交替执行，从而实现并发。

#### 并行 (Parallelism)

**定义：** 并行是指两个或多个任务在同一时刻同时执行。

**举例：** 在多核处理器上，多个任务可以同时在不同的核心上执行，从而实现并行。

### 2.2 区别与联系

- **并发关注的是任务的逻辑结构，而并行关注的是任务的实际执行。**
- **并行是并发的一种特殊形式，但并发不一定意味着并行。**
- **并发可以提高系统的响应速度和资源利用率，而并行可以提高系统的吞吐量和处理能力。**

## 三、多线程与多进程

### 3.1 概念解析

#### 线程 (Thread)

**定义：** 线程是操作系统能够进行运算调度的最小单位。它被包含在进程中，是进程中的实际执行单元。一个进程可以包含多个线程，这些线程共享进程的地址空间和资源。

**特点：**

- **轻量级：** 线程的创建和销毁开销较小，切换速度快。
- **共享资源：** 线程之间共享进程的内存空间、文件描述符等资源。
- **独立执行：** 每个线程有自己的栈空间和程序计数器，可以独立执行不同的任务。
- **通信便捷：** 由于共享内存，线程之间的通信和数据共享非常方便，但也容易引发数据竞争和同步问题。

#### 进程 (Process)

**定义：** 进程是操作系统中资源分配的基本单位。每个进程都有独立的地址空间、内存、文件资源等。一个进程可以包含多个线程，这些线程在进程的地址空间内并发执行。

**特点：**

- **资源隔离：** 每个进程拥有独立的地址空间和资源，一个进程的崩溃不会直接影响其他进程。
- **创建开销大：** 进程的创建和销毁开销较大，切换速度较慢。
- **通信复杂：** 进程之间的通信需要借助特定的机制，如管道、消息队列、共享内存等，通信效率较低。
- **独立执行：** 每个进程有自己的地址空间和资源，可以独立执行不同的任务。

#### 线程与进程的区别及联系

**区别：**

1. **资源分配：**
   - **进程：** 拥有独立的地址空间和资源，是操作系统中资源分配的基本单位。
   - **线程：** 共享进程的地址空间和资源，是进程中的实际执行单元。
2. **创建和销毁开销：**
   - **进程：** 创建和销毁开销较大，切换速度较慢。
   - **线程：** 创建和销毁开销较小，切换速度快。
3. **通信方式：**
   - **进程：** 进程之间的通信需要借助特定的机制，如管道、消息队列、共享内存等，通信效率较低。
   - **线程：** 线程之间共享内存，通信和数据共享非常方便，但也容易引发数据竞争和同步问题。
4. **隔离性：**
   - **进程：** 进程之间拥有独立的地址空间和资源，一个进程的崩溃不会影响其他进程，隔离性较好。
   - **线程：** 线程之间共享进程的资源，一个线程的崩溃可能会影响整个进程，隔离性较差。

**联系：**

- **包含关系：** 线程是进程的一部分，一个进程可以包含多个线程。
- **并发执行：** 无论是进程还是线程，都可以在操作系统中并发执行，充分利用多核处理器的优势。
- **资源共享：** 线程之间共享进程的资源，而进程之间拥有独立的资源。

#### 多线程 (Multithreading)

**定义：** 多线程是指在一个进程内创建多个线程，每个线程执行不同的任务。

**特点：**
- **线程之间共享进程的地址空间和资源。**
- **线程之间的切换开销较小。**
- **需要考虑线程同步和数据竞争问题。**

#### 多进程 (Multiprocessing)

**定义：** 多进程是指创建多个进程，每个进程执行不同的任务。

**特点：**

- **进程之间拥有独立的地址空间和资源。**
- **进程之间的切换开销较大。**
- **进程间通信需要借助特定的机制，例如管道、消息队列等。**

### 3.2 区别与选择

- **资源占用：** 多进程比多线程占用更多的系统资源。
- **通信效率：** 多线程之间的通信效率高于多进程。
- **隔离性：** 多进程比多线程具有更好的隔离性，一个进程的崩溃不会影响其他进程。
- **选择依据：**
  - 如果需要高并发、低延迟的应用，例如Web服务器，可以选择多线程。
  - 如果需要高并行、高吞吐量的应用，例如科学计算，可以选择多进程。
  - 如果需要兼顾并发和并行，并且对隔离性要求不高，可以选择多线程+多进程的混合模式。

## 四、C++并发编程基础

### 4.1 C++标准库中的并发支持

C++11引入了标准库中的并发支持，主要包括以下几个库：

- **thread 库:** 用于创建和管理线程。
- **mutex 库:** 用于线程同步，避免数据竞争。
- **condition_variable 库:** 用于线程间通信。
- **future 库:** 用于异步编程，获取线程执行结果。

### 4.2 多线程编程示例

#### 4.2.1 使用 `std::thread` 创建线程

```cpp
#include <iostream>
#include <thread>

void threadFunction() {
    std::cout << "Hello from thread!" << std::endl;
}

int main() {
    // 创建一个线程，执行 threadFunction
    std::thread t(threadFunction);

    // 主线程等待子线程完成
    t.join();

    std::cout << "Thread joined" << std::endl;
    return 0;
}
```

**代码解析：**
- `std::thread t(threadFunction);`：创建一个线程 `t`，并执行 `threadFunction` 函数。
- `t.join();`：主线程等待子线程 `t` 完成后再继续执行。

#### 4.2.2 使用 `std::mutex` 保护共享数据

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;
int sharedData = 0;

void increment() {
    for (int i = 0; i < 100000; ++i) {
        std::lock_guard<std::mutex> lock(mtx); // 自动加锁和解锁
        ++sharedData;
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "Shared data: " << sharedData << std::endl;
    return 0;
}
```

**代码解析：**
- `std::mutex mtx;`：定义一个互斥锁 `mtx`。
- `std::lock_guard<std::mutex> lock(mtx);`：使用 `std::lock_guard` 自动管理互斥锁的加锁和解锁，确保在 `increment` 函数中对 `sharedData` 的访问是线程安全的。

#### 4.2.3 使用 `std::condition_variable` 实现线程间通信

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void worker() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, []{ return ready; }); // 等待条件变量
    std::cout << "Worker thread is processing data" << std::endl;
}

void trigger() {
    std::unique_lock<std::mutex> lock(mtx);
    ready = true;
    cv.notify_one(); // 通知等待的线程
    std::cout << "Trigger thread signals data is ready" << std::endl;
}

int main() {
    std::thread t1(worker);
    std::thread t2(trigger);

    t1.join();
    t2.join();

    return 0;
}
```

**代码解析：**
- `std::condition_variable cv;`：定义一个条件变量 `cv`。
- `cv.wait(lock, []{ return ready; });`：线程 `t1` 等待条件变量 `cv`，直到 `ready` 变为 `true`。
- `cv.notify_one();`：线程 `t2` 通知等待的线程 `t1`，条件变量 `cv` 已满足。

#### 4.2.4 使用 `std::future` 获取异步任务结果

```cpp
#include <iostream>
#include <thread>
#include <future>

int asyncFunction() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    return 42;
}

int main() {
    // 创建一个异步任务
    std::future<int> result = std::async(std::launch::async, asyncFunction);

    std::cout << "Waiting for result..." << std::endl;

    // 获取异步任务的结果
    int value = result.get();

    std::cout << "Result: " << value << std::endl;
    return 0;
}
```

**代码解析：**
- `std::future<int> result = std::async(std::launch::async, asyncFunction);`：创建一个异步任务，并返回一个 `std::future` 对象 `result`。
- `result.get();`：主线程等待异步任务完成，并获取其返回值。

### 4.3 多进程编程示例

#### 4.3.1 使用 `fork()` 创建子进程

```cpp
#include <iostream>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        // 子进程
        std::cout << "Child process is running" << std::endl;
    } else if (pid > 0) {
        // 父进程
        std::cout << "Parent process is running, child PID: " << pid << std::endl;
    } else {
        // 创建进程失败
        std::cerr << "Fork failed" << std::endl;
    }

    return 0;
}
```

**代码解析：**
- `pid_t pid = fork();`：创建一个子进程，`fork()` 返回的 `pid` 在父进程中为子进程的 PID，在子进程中为 `0`。

#### 4.3.2 使用 `pipe()` 实现进程间通信

```cpp
#include <iostream>
#include <unistd.h>
#include <string.h>

int main() {
    int pipefd[2];
    char buffer[256];

    // 创建管道
    if (pipe(pipefd) == -1) {
        std::cerr << "Pipe creation failed" << std::endl;
        return 1;
    }

    pid_t pid = fork();

    if (pid == 0) {
        // 子进程：关闭写端，读取数据
        close(pipefd[1]);
        read(pipefd[0], buffer, sizeof(buffer));
        std::cout << "Child received: " << buffer << std::endl;
        close(pipefd[0]);
    } else if (pid > 0) {
        // 父进程：关闭读端，写入数据
        close(pipefd[0]);
        const char* message = "Hello from parent!";
        write(pipefd[1], message, strlen(message) + 1);
        close(pipefd[1]);
    } else {
        std::cerr << "Fork failed" << std::endl;
    }

    return 0;
}
```

**代码解析：**
- `pipe(pipefd);`：创建一个管道，`pipefd[0]` 为读端，`pipefd[1]` 为写端。
- 子进程关闭写端，读取父进程发送的数据；父进程关闭读端，向子进程发送数据。

#### 4.3.3 使用 `wait()` 等待子进程结束

```cpp
#include <iostream>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        // 子进程
        std::cout << "Child process is running" << std::endl;
        sleep(2);
        std::cout << "Child process is done" << std::endl;
    } else if (pid > 0) {
        // 父进程
        std::cout << "Parent process is waiting for child" << std::endl;
        wait(NULL); // 等待子进程结束
        std::cout << "Parent process is done" << std::endl;
    } else {
        std::cerr << "Fork failed" << std::endl;
    }

    return 0;
}
```

**代码解析：**
- `wait(NULL);`：父进程等待子进程结束，确保子进程在父进程之前完成。

## 五、总结

本文详细介绍了并发与并行、多线程与多进程的概念和区别，并通过代码示例展示了如何使用C++标准库进行多线程和多进程编程。C++标准库为并发编程提供了强大的支持，掌握这些工具可以显著提高程序的性能和响应速度。

并发编程是一个复杂的领域，涉及线程同步、数据竞争、死锁等问题。希望读者通过本文的学习，能够对C++并发编程有一个初步的了解，并将其应用到实际项目中。