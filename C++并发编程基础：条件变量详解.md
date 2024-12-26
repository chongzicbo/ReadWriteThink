# C++并发编程中的条件变量

## 1. 引言

### 1.1 什么是并发编程
并发编程是指在同一时间段内，多个任务或线程同时执行的编程方式。通过并发编程，可以提高程序的执行效率，特别是在多核处理器上，能够充分利用硬件资源。

### 1.2 C++并发编程的背景
C++11引入了标准库中的多线程支持，使得并发编程变得更加容易和安全。C++标准库提供了`std::thread`、`std::mutex`、`std::condition_variable`等工具，帮助开发者实现线程间的同步和通信。

### 1.3 条件变量在并发编程中的重要性
条件变量是C++并发编程中的一个重要工具，用于线程间的同步。它允许一个线程等待另一个线程发出信号，从而避免忙等待，提高程序的效率。条件变量通常与互斥锁（`std::mutex`）一起使用，以确保线程间的安全通信。

## 2. 条件变量概述

### 2.1 什么是条件变量

#### 2.1.1 定义与概念
条件变量（`std::condition_variable`）是C++标准库中的一个同步原语，用于线程间的通信。它允许一个线程等待另一个线程发出信号，从而唤醒等待的线程。

#### 2.1.2 条件变量的工作原理
条件变量的工作原理基于等待和通知机制。一个线程可以调用`wait()`方法进入等待状态，直到另一个线程调用`notify_one()`或`notify_all()`方法发出信号。在等待期间，线程会释放持有的互斥锁，从而允许其他线程访问共享资源。

### 2.2 为什么需要条件变量

#### 2.2.1 线程间的同步问题
在多线程编程中，线程间的同步是一个常见的问题。例如，一个线程可能需要等待另一个线程完成某个任务后才能继续执行。如果没有合适的同步机制，线程可能会进入忙等待状态，浪费CPU资源。

#### 2.2.2 条件变量的作用
条件变量提供了一种高效的同步机制，允许线程在特定条件下等待，并在条件满足时被唤醒。通过条件变量，线程可以避免忙等待，从而提高程序的效率。

### 2.3 条件变量解决的问题

#### 2.3.1 线程等待特定条件
条件变量允许线程在特定条件满足时才继续执行。例如，一个线程可以等待某个共享变量的值发生变化，或者等待另一个线程完成某个任务。

#### 2.3.2 避免忙等待
忙等待是指线程在等待某个条件时，不断检查该条件是否满足，从而浪费CPU资源。条件变量通过让线程进入睡眠状态，避免了忙等待，提高了程序的效率。

## 3. 条件变量的应用场景

### 3.1 生产者-消费者模型

#### 3.1.1 场景描述
生产者-消费者模型是并发编程中的经典问题。生产者线程负责生成数据并将其放入缓冲区，消费者线程负责从缓冲区中取出数据并进行处理。条件变量用于同步生产者和消费者线程，确保缓冲区在有数据时消费者才能消费，缓冲区在有空闲空间时生产者才能生产。

#### 3.1.2 代码示例

##### 3.1.2.1 生产者线程
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> buffer;
const int buffer_size = 10;

void producer(int id) {
    for (int i = 0; i < 20; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return buffer.size() < buffer_size; }); // 等待缓冲区有空闲空间
        buffer.push(i);
        std::cout << "Producer " << id << " produced " << i << std::endl;
        cv.notify_all(); // 通知消费者线程
    }
}
```

##### 3.1.2.2 消费者线程
```cpp
void consumer(int id) {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return !buffer.empty(); }); // 等待缓冲区有数据
        int value = buffer.front();
        buffer.pop();
        std::cout << "Consumer " << id << " consumed " << value << std::endl;
        cv.notify_all(); // 通知生产者线程
    }
}
```

#### 3.1.3 代码解析
- **生产者线程**：生产者线程在生产数据前，首先获取互斥锁，然后检查缓冲区是否已满。如果缓冲区已满，生产者线程会调用`cv.wait()`进入等待状态，直到消费者线程消费数据并释放空间。生产者线程生产数据后，调用`cv.notify_all()`通知所有等待的消费者线程。
- **消费者线程**：消费者线程在消费数据前，首先获取互斥锁，然后检查缓冲区是否为空。如果缓冲区为空，消费者线程会调用`cv.wait()`进入等待状态，直到生产者线程生产数据。消费者线程消费数据后，调用`cv.notify_all()`通知所有等待的生产者线程。



#### 3.1.4 完整代码

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> buffer;
const int buffer_size = 10;

void producer(int id) {
    for (int i = 0; i < 20; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return buffer.size() < buffer_size; }); // 等待缓冲区有空闲空间
        buffer.push(i);
        std::cout << "Producer " << id << " produced " << i << std::endl;
        cv.notify_all(); // 通知消费者线程
    }
}

void consumer(int id) {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return !buffer.empty(); }); // 等待缓冲区有数据
        int value = buffer.front();
        buffer.pop();
        std::cout << "Consumer " << id << " consumed " << value << std::endl;
        cv.notify_all(); // 通知生产者线程
    }
}

int main() {
    std::thread prod1(producer, 1);
    std::thread prod2(producer, 2);
    std::thread cons1(consumer, 1);
    std::thread cons2(consumer, 2);

    prod1.join();
    prod2.join();
    cons1.join();
    cons2.join();

    return 0;
}
```



### 3.2 线程池中的任务调度

#### 3.2.1 场景描述
线程池是一种常见的并发编程模式，用于管理一组线程，以便它们可以并发地执行任务。条件变量用于同步任务的添加和执行，确保线程池中的线程在有任务时才执行，没有任务时进入等待状态。

#### 3.2.2 代码示例

##### 3.2.2.1 任务添加
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <functional>

std::mutex mtx;
std::condition_variable cv;
std::queue<std::function<void()>> tasks;

void add_task(std::function<void()> task) {
    std::unique_lock<std::mutex> lock(mtx);
    tasks.push(task);
    cv.notify_one(); // 通知一个工作线程
}
```

##### 3.2.2.2 任务执行
```cpp
void worker() {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return !tasks.empty(); }); // 等待任务队列非空
        auto task = tasks.front();
        tasks.pop();
        lock.unlock();
        task(); // 执行任务
    }
}
```

#### 3.2.3 代码解析
- **任务添加**：当有新任务时，调用`add_task()`函数将任务添加到任务队列中，并调用`cv.notify_one()`通知一个工作线程。
- **任务执行**：工作线程在执行任务前，首先获取互斥锁，然后检查任务队列是否为空。如果任务队列为空，工作线程会调用`cv.wait()`进入等待状态，直到有新任务被添加。工作线程执行任务后，继续等待下一个任务。

#### 3.2.4 完整代码

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <functional>
#include <vector>

std::mutex mtx;
std::condition_variable cv;
std::queue<std::function<void()>> tasks;

void add_task(std::function<void()> task) {
    std::unique_lock<std::mutex> lock(mtx);
    tasks.push(task);
    cv.notify_one(); // 通知一个工作线程
}

void worker() {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [] { return !tasks.empty(); }); // 等待任务队列非空
        auto task = tasks.front();
        tasks.pop();
        lock.unlock();
        task(); // 执行任务
    }
}

int main() {
    // 创建 4 个工作线程
    std::vector<std::thread> workers;
    for (int i = 0; i < 4; ++i) {
        workers.emplace_back(worker);
    }

    // 添加任务
    for (int i = 0; i < 10; ++i) {
        add_task([i] {
            std::cout << "Task " << i << " is being executed by thread " << std::this_thread::get_id() << std::endl;
            std::this_thread::sleep_for(std::chrono::milliseconds(500)); // 模拟任务执行时间
        });
    }

    // 等待所有任务完成
    for (auto& worker : workers) {
        worker.join();
    }

    return 0;
}
```



### 3.3 多线程事件处理

#### 3.3.1 场景描述
在多线程事件处理中，多个线程可能需要等待某个事件的发生，并在事件发生时进行处理。条件变量用于同步事件的通知和处理，确保线程在事件发生时才被唤醒。

#### 3.3.2 代码示例

##### 3.3.2.1 事件通知
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool event_occurred = false;

void notify_event() {
    std::unique_lock<std::mutex> lock(mtx);
    event_occurred = true;
    cv.notify_all(); // 通知所有等待的线程
}
```

##### 3.3.2.2 事件处理
```cpp
void handle_event(int id) {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return event_occurred; }); // 等待事件发生
    std::cout << "Thread " << id << " handling event" << std::endl;
}
```

#### 3.3.3 代码解析
- **事件通知**：当事件发生时，调用`notify_event()`函数，将`event_occurred`标志设置为`true`，并调用`cv.notify_all()`通知所有等待的线程。
- **事件处理**：线程在处理事件前，首先获取互斥锁，然后检查`event_occurred`标志是否为`true`。如果标志为`false`，线程会调用`cv.wait()`进入等待状态，直到事件发生。线程在事件发生时被唤醒，并进行事件处理。

#### 3.3.4 完整代码

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool event_occurred = false;

void notify_event() {
    std::unique_lock<std::mutex> lock(mtx);
    event_occurred = true;
    cv.notify_all(); // 通知所有等待的线程
}

void handle_event(int id) {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return event_occurred; }); // 等待事件发生
    std::cout << "Thread " << id << " handling event" << std::endl;
}

int main() {
    // 创建 3 个等待事件的线程
    std::thread t1(handle_event, 1);
    std::thread t2(handle_event, 2);
    std::thread t3(handle_event, 3);

    // 模拟事件发生
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 等待 2 秒
    notify_event();

    // 等待所有线程完成
    t1.join();
    t2.join();
    t3.join();

    return 0;
}
```



## 4. 条件变量的使用细节

### 4.1 条件变量的基本操作

条件变量的核心操作包括`wait()`、`notify_one()`和`notify_all()`。这些操作是线程间同步的基础。

#### 4.1.1 `wait()` 方法

`wait()`方法用于让当前线程进入等待状态，直到另一个线程调用`notify_one()`或`notify_all()`唤醒它。在调用`wait()`时，线程会自动释放持有的互斥锁，并在被唤醒后重新获取锁。

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
    cv.wait(lock, [] { return ready; }); // 等待条件变量
    std::cout << "Worker thread is processing data..." << std::endl;
}

void trigger() {
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 模拟延迟
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true; // 设置条件为真
    }
    cv.notify_one(); // 唤醒一个等待的线程
}

int main() {
    std::thread t1(worker);
    std::thread t2(trigger);

    t1.join();
    t2.join();

    return 0;
}
```

**代码解析**：
- `worker()`线程调用`cv.wait(lock, [] { return ready; })`，进入等待状态，直到`ready`变量为`true`。
- `trigger()`线程在延迟2秒后，设置`ready`为`true`，并调用`cv.notify_one()`唤醒`worker()`线程。
- `wait()`方法会自动释放互斥锁，并在被唤醒后重新获取锁。

#### 4.1.2 `notify_one()` 方法

`notify_one()`方法用于唤醒一个等待在条件变量上的线程。如果有多个线程在等待，只有一个线程会被唤醒。

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void worker(int id) {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return ready; });
    std::cout << "Worker " << id << " is processing data..." << std::endl;
}

void trigger() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one(); // 唤醒一个线程
}

int main() {
    std::thread t1(worker, 1);
    std::thread t2(worker, 2);
    std::thread t3(trigger);

    t1.join();
    t2.join();
    t3.join();

    return 0;
}
```

**代码解析**：
- `worker(int id)`线程等待`ready`变量为`true`。
- `trigger()`线程设置`ready`为`true`，并调用`cv.notify_one()`唤醒一个等待的线程。
- 由于`notify_one()`只唤醒一个线程，因此只有一个`worker`线程会被唤醒。

#### 4.1.3 `notify_all()` 方法

`notify_all()`方法用于唤醒所有等待在条件变量上的线程。所有等待的线程都会被唤醒，并尝试重新获取互斥锁。

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void worker(int id) {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return ready; });
    std::cout << "Worker " << id << " is processing data..." << std::endl;
}

void trigger() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_all(); // 唤醒所有线程
}

int main() {
    std::thread t1(worker, 1);
    std::thread t2(worker, 2);
    std::thread t3(trigger);

    t1.join();
    t2.join();
    t3.join();

    return 0;
}
```

**代码解析**：
- `worker(int id)`线程等待`ready`变量为`true`。
- `trigger()`线程设置`ready`为`true`，并调用`cv.notify_all()`唤醒所有等待的线程。
- 所有`worker`线程都会被唤醒，并继续执行。

### 4.2 条件变量与互斥锁的配合使用

#### 4.2.1 互斥锁的作用

互斥锁（`std::mutex`）用于保护共享资源，确保同一时间只有一个线程可以访问共享资源。条件变量通常与互斥锁一起使用，以确保线程间的同步和数据安全。

#### 4.2.2 代码示例

##### 4.2.2.1 互斥锁与条件变量的结合

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
int shared_data = 0;
bool ready = false;

void producer() {
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 模拟生产数据
    {
        std::lock_guard<std::mutex> lock(mtx);
        shared_data = 42; // 生产数据
        ready = true;
    }
    cv.notify_one(); // 通知消费者
}

void consumer() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return ready; }); // 等待生产者生产数据
    std::cout << "Consumed data: " << shared_data << std::endl;
}

int main() {
    std::thread t1(producer);
    std::thread t2(consumer);

    t1.join();
    t2.join();

    return 0;
}
```

**代码解析**：
- `producer()`线程生产数据并设置`ready`为`true`，然后调用`cv.notify_one()`通知消费者。
- `consumer()`线程等待`ready`为`true`，然后消费数据。
- 互斥锁`mtx`确保在生产者和消费者之间对`shared_data`和`ready`的访问是线程安全的。

### 4.3 条件变量的虚假唤醒

#### 4.3.1 什么是虚假唤醒

虚假唤醒是指线程在没有被显式调用`notify_one()`或`notify_all()`的情况下被唤醒。这种情况可能会导致线程在条件未满足时继续执行，从而引发逻辑错误。

#### 4.3.2 如何避免虚假唤醒

为了避免虚假唤醒，通常会在`wait()`方法中传入一个谓词（predicate），用于检查条件是否真正满足。如果条件不满足，线程会继续等待。

#### 4.3.3 代码示例

##### 4.3.3.1 避免虚假唤醒的实现

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
int shared_data = 0;
bool ready = false;

void producer() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    {
        std::lock_guard<std::mutex> lock(mtx);
        shared_data = 42;
        ready = true;
    }
    cv.notify_one();
}

void consumer() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return ready; }); // 使用谓词避免虚假唤醒
    std::cout << "Consumed data: " << shared_data << std::endl;
}

int main() {
    std::thread t1(producer);
    std::thread t2(consumer);

    t1.join();
    t2.join();

    return 0;
}
```

**代码解析**：
- `consumer()`线程调用`cv.wait(lock, [] { return ready; })`，传入一个谓词`[] { return ready; }`。
- 如果`ready`为`false`，线程会继续等待，即使发生了虚假唤醒。
- 只有当`ready`为`true`时，线程才会继续执行。

## 5. 条件变量的缺点与局限性

### 5.1 性能开销

#### 5.1.1 上下文切换的成本

条件变量的使用会导致线程进入等待状态，从而引发上下文切换。上下文切换是操作系统将一个线程的执行状态保存并恢复另一个线程的过程，这个过程会带来一定的性能开销。

#### 5.1.2 条件变量的性能影响

频繁的上下文切换会降低程序的性能，尤其是在高并发场景下。因此，在使用条件变量时，应尽量减少不必要的等待和唤醒操作，以降低性能开销。

### 5.2 死锁风险

#### 5.2.1 死锁的成因

死锁是指两个或多个线程互相等待对方释放资源，导致所有线程都无法继续执行的情况。条件变量与互斥锁的结合使用可能会引发死锁，尤其是在多个线程之间存在复杂的依赖关系时。

#### 5.2.2 如何避免死锁

为了避免死锁，可以遵循以下原则：
- 确保所有线程以相同的顺序获取锁。
- 避免在持有锁的情况下调用外部代码。
- 使用超时机制，避免线程无限期等待。

### 5.3 复杂性增加

#### 5.3.1 代码的可读性问题

条件变量的使用会增加代码的复杂性，尤其是在多个线程之间存在复杂的同步逻辑时。复杂的同步逻辑可能会导致代码难以理解和维护。

#### 5.3.2 调试难度增加

由于条件变量的行为依赖于线程的调度，调试多线程程序可能会变得非常困难。特别是在出现虚假唤醒或死锁时，定位问题的原因可能需要花费大量时间。

## 6. 条件变量的替代方案

在并发编程中，条件变量并不是唯一的线程同步工具。根据不同的场景需求，还可以使用信号量、原子操作、`future`和`promise`等替代方案。

### 6.1 使用信号量

#### 6.1.1 信号量的概念

信号量（Semaphore）是一种用于控制多个线程对共享资源访问的同步工具。它维护一个计数器，线程可以通过`acquire()`方法减少计数器，或通过`release()`方法增加计数器。当计数器为0时，调用`acquire()`的线程会被阻塞，直到其他线程调用`release()`增加计数器。

#### 6.1.2 信号量与条件变量的对比

信号量和条件变量都可以用于线程间的同步，但它们的应用场景有所不同：
- **条件变量**：主要用于线程间的等待和通知，通常与互斥锁配合使用，适用于复杂的同步场景。
- **信号量**：主要用于控制对共享资源的访问，适用于资源有限的多线程场景。

##### 代码示例：使用信号量实现生产者-消费者模型

```cpp
#include <iostream>
#include <thread>
#include <semaphore>
#include <queue>

std::queue<int> buffer;
std::counting_semaphore<10> empty_slots(10); // 空闲槽位信号量
std::counting_semaphore<10> filled_slots(0); // 已填充槽位信号量

void producer() {
    for (int i = 0; i < 20; ++i) {
        empty_slots.acquire(); // 等待空闲槽位
        buffer.push(i);
        std::cout << "Produced: " << i << std::endl;
        filled_slots.release(); // 通知消费者有新数据
    }
}

void consumer() {
    while (true) {
        filled_slots.acquire(); // 等待已填充槽位
        int value = buffer.front();
        buffer.pop();
        std::cout << "Consumed: " << value << std::endl;
        empty_slots.release(); // 通知生产者有空闲槽位
    }
}

int main() {
    std::thread t1(producer);
    std::thread t2(consumer);

    t1.join();
    t2.join();

    return 0;
}
```

**代码解析**：
- `empty_slots`信号量表示缓冲区中的空闲槽位数量，初始值为10。
- `filled_slots`信号量表示缓冲区中已填充的槽位数量，初始值为0。
- 生产者线程在生产数据前，调用`empty_slots.acquire()`等待空闲槽位，生产数据后调用`filled_slots.release()`通知消费者。
- 消费者线程在消费数据前，调用`filled_slots.acquire()`等待已填充槽位，消费数据后调用`empty_slots.release()`通知生产者。

### 6.2 使用原子操作

#### 6.2.1 原子操作的概念

原子操作（Atomic Operation）是指在多线程环境中，不会被中断的操作。C++标准库提供了`std::atomic`类型，用于实现线程安全的原子操作。原子操作可以避免使用互斥锁，从而提高性能。

#### 6.2.2 原子操作与条件变量的对比

原子操作和条件变量都可以用于线程间的同步，但它们的应用场景有所不同：
- **条件变量**：适用于复杂的同步场景，通常与互斥锁配合使用。
- **原子操作**：适用于简单的状态标志或计数器，能够避免锁的开销。

##### 代码示例：使用原子操作实现线程同步

```cpp
#include <iostream>
#include <thread>
#include <atomic>

std::atomic<bool> ready(false);

void worker() {
    while (!ready.load()) { // 等待ready为true
        std::this_thread::yield(); // 让出CPU时间片
    }
    std::cout << "Worker thread is processing data..." << std::endl;
}

void trigger() {
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 模拟延迟
    ready.store(true); // 设置ready为true
}

int main() {
    std::thread t1(worker);
    std::thread t2(trigger);

    t1.join();
    t2.join();

    return 0;
}
```

**代码解析**：
- `ready`是一个原子布尔变量，用于表示线程是否可以继续执行。
- `worker()`线程通过`ready.load()`检查`ready`的值，如果为`false`，则让出CPU时间片，直到`ready`为`true`。
- `trigger()`线程在延迟2秒后，将`ready`设置为`true`，通知`worker()`线程继续执行。

### 6.3 使用future和promise

#### 6.3.1 future和promise的概念

`future`和`promise`是C++11引入的用于线程间通信的工具。`promise`用于设置值或异常，而`future`用于获取值或异常。`future`可以阻塞等待`promise`设置的值，从而实现线程间的同步。

#### 6.3.2 future和promise与条件变量的对比

`future`和`promise`与条件变量都可以用于线程间的同步，但它们的应用场景有所不同：
- **条件变量**：适用于复杂的同步场景，通常与互斥锁配合使用。
- **future和promise**：适用于单次的结果传递，能够简化线程间的通信。

##### 代码示例：使用future和promise实现线程同步

```cpp
#include <iostream>
#include <thread>
#include <future>

void worker(std::future<int>& fut) {
    int value = fut.get(); // 阻塞等待结果
    std::cout << "Worker thread received value: " << value << std::endl;
}

void trigger(std::promise<int>& prom) {
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 模拟延迟
    prom.set_value(42); // 设置结果
}

int main() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    std::thread t1(worker, std::ref(fut));
    std::thread t2(trigger, std::ref(prom));

    t1.join();
    t2.join();

    return 0;
}
```

**代码解析**：
- `prom`是一个`promise`对象，用于设置结果。
- `fut`是一个`future`对象，用于获取结果。
- `worker()`线程调用`fut.get()`阻塞等待结果，直到`trigger()`线程调用`prom.set_value()`设置结果。
- `trigger()`线程在延迟2秒后，设置结果为42。

## 7. 总结

### 7.1 条件变量的优势

条件变量是C++并发编程中的重要工具，具有以下优势：
- **灵活性**：条件变量可以用于复杂的线程同步场景，支持线程的等待和通知。
- **高效性**：条件变量避免了忙等待，能够显著提高程序的效率。
- **广泛适用**：条件变量适用于生产者-消费者模型、线程池、事件处理等多种场景。

### 7.2 条件变量的局限性

尽管条件变量功能强大，但也存在一些局限性：
- **性能开销**：条件变量的使用会导致上下文切换，可能带来性能开销。
- **死锁风险**：条件变量与互斥锁的结合使用可能引发死锁。
- **复杂性**：条件变量的使用会增加代码的复杂性，降低可读性和调试难度。

### 7.3 适用场景与最佳实践

- **适用场景**：条件变量适用于需要复杂同步逻辑的场景，如生产者-消费者模型、线程池等。
- **最佳实践**：
  - 避免不必要的等待和唤醒操作，减少性能开销。
  - 使用谓词避免虚假唤醒，确保线程在条件满足时才继续执行。
  - 遵循锁的获取顺序，避免死锁。
  - 在简单场景下，考虑使用原子操作或`future`和`promise`替代条件变量。

通过合理选择和使用条件变量，可以有效提高多线程程序的性能和可维护性。

## 