## C++并发中线程池：从概念到实践

## 1. 什么是线程池？

### 1.1 线程池的基本概念

线程池是一种并发编程技术，它通过预先创建一组线程并将其放入池中，以便在需要时可以重复使用这些线程来执行任务。线程池的主要目的是减少线程创建和销毁的开销，提高程序的性能和资源利用率。

在传统的多线程编程中，每次任务到来时都会创建一个新的线程，任务完成后线程被销毁。这种方式虽然简单，但频繁的线程创建和销毁会带来较大的性能开销，尤其是在高并发场景下。线程池通过复用线程，避免了这些开销，从而提高了程序的效率。

### 1.2 线程池的工作原理

线程池的工作原理可以概括为以下几个步骤：

1. **初始化线程池**：在程序启动时，线程池会预先创建一定数量的线程，并将这些线程放入池中。
2. **任务提交**：当有任务需要执行时，任务会被提交到线程池的任务队列中。
3. **任务分配**：线程池中的线程会从任务队列中获取任务并执行。如果所有线程都在忙碌，新的任务会等待，直到有空闲线程可用。
4. **任务执行**：线程执行任务，完成后返回线程池，等待下一个任务。
5. **线程池关闭**：当所有任务完成后，线程池可以被关闭，线程被销毁。

## 2. 为什么使用线程池？

### 2.1 线程池的优势

1. **减少线程创建和销毁的开销**：线程池通过复用线程，避免了频繁的线程创建和销毁，从而提高了程序的性能。
2. **提高资源利用率**：线程池可以有效地管理线程资源，避免因线程过多导致的资源耗尽问题。
3. **任务调度灵活**：线程池可以根据任务的优先级、类型等进行灵活调度，确保任务能够高效执行。
4. **简化并发编程**：使用线程池可以简化并发编程的复杂性，开发者只需关注任务的提交和结果的获取，而不需要管理线程的生命周期。

### 2.2 线程池的适用场景

线程池适用于以下场景：

1. **高并发任务处理**：如Web服务器处理HTTP请求、数据库查询等。
2. **任务执行时间不确定**：如异步IO操作、网络请求等。
3. **需要控制线程数量**：如需要限制并发线程数量以避免资源耗尽的场景。

## 3. 如何使用线程池？

### 3.1 线程池的基本结构

一个典型的线程池通常包含以下几个组件：

1. **任务队列**：用于存储待执行的任务。
2. **工作线程**：线程池中的线程，负责从任务队列中获取任务并执行。
3. **线程池管理器**：负责线程池的初始化、任务的提交、线程的调度以及线程池的关闭。

### 3.2 创建线程池

#### 代码示例

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <future>

class ThreadPool {
public:
    ThreadPool(size_t threads) : stop(false) {
        for (size_t i = 0; i < threads; ++i) {
            workers.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(this->queue_mutex);
                        this->condition.wait(lock, [this] { return this->stop || !this->tasks.empty(); });
                        if (this->stop && this->tasks.empty())
                            return;
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    task();
                }
            });
        }
    }

    template <class F, class... Args>
    auto enqueue(F&& f, Args&&... args) -> std::future<typename std::result_of<F(Args...)>::type> {
        using return_type = typename std::result_of<F(Args...)>::type;
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...));
        std::future<return_type> res = task->get_future();
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            if (stop)
                throw std::runtime_error("enqueue on stopped ThreadPool");
            tasks.emplace([task]() { (*task)(); });
        }
        condition.notify_one();
        return res;
    }

    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            stop = true;
        }
        condition.notify_all();
        for (std::thread& worker : workers)
            worker.join();
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex queue_mutex;
    std::condition_variable condition;
    bool stop;
};

int main() {
    ThreadPool pool(4);
    std::vector<std::future<int>> results;

    for (int i = 0; i < 8; ++i) {
        results.emplace_back(
            pool.enqueue([i] {
                std::cout << "hello " << i << std::endl;
                std::this_thread::sleep_for(std::chrono::seconds(1));
                std::cout << "world " << i << std::endl;
                return i * i;
            })
        );
    }

    for (auto&& result : results)
        std::cout << result.get() << ' ';
    std::cout << std::endl;

    return 0;
}
```

#### 代码解析

1. **线程池类定义**：
   ```cpp
   class ThreadPool {
   public:
       ThreadPool(size_t threads) : stop(false) {
           for (size_t i = 0; i < threads; ++i) {
               workers.emplace_back([this] {
                   while (true) {
                       std::function<void()> task;
                       {
                           std::unique_lock<std::mutex> lock(this->queue_mutex);
                           this->condition.wait(lock, [this] { return this->stop || !this->tasks.empty(); });
                           if (this->stop && this->tasks.empty())
                               return;
                           task = std::move(this->tasks.front());
                           this->tasks.pop();
                       }
                       task();
                   }
               });
           }
       }
   ```
   - `ThreadPool`类是线程池的核心类，它包含一个构造函数，用于初始化线程池。
   - `workers`是一个`std::vector<std::thread>`，用于存储工作线程。
   - 构造函数中，通过循环创建指定数量的线程，并将它们放入`workers`中。
   - 每个线程的执行逻辑是：不断从任务队列中获取任务并执行，直到线程池被关闭且任务队列为空。

2. **任务提交**：
   ```cpp
   template <class F, class... Args>
   auto enqueue(F&& f, Args&&... args) -> std::future<typename std::result_of<F(Args...)>::type> {
       using return_type = typename std::result_of<F(Args...)>::type;
       auto task = std::make_shared<std::packaged_task<return_type()>>(
           std::bind(std::forward<F>(f), std::forward<Args>(args)...));
       std::future<return_type> res = task->get_future();
       {
           std::unique_lock<std::mutex> lock(queue_mutex);
           if (stop)
               throw std::runtime_error("enqueue on stopped ThreadPool");
           tasks.emplace([task]() { (*task)(); });
       }
       condition.notify_one();
       return res;
   }
   ```
   - `enqueue`函数用于向线程池提交任务。
   - 它接受一个可调用对象`f`和它的参数`args`，并返回一个`std::future`对象，用于获取任务的执行结果。
   - 任务被封装为一个`std::packaged_task`，并放入任务队列`tasks`中。
   - 最后，通过`condition.notify_one()`通知一个工作线程来执行任务。

3. **线程池关闭**：
   ```cpp
   ~ThreadPool() {
       {
           std::unique_lock<std::mutex> lock(queue_mutex);
           stop = true;
       }
       condition.notify_all();
       for (std::thread& worker : workers)
           worker.join();
   }
   ```
   - 析构函数用于关闭线程池。
   - 首先，将`stop`标志设置为`true`，并通过`condition.notify_all()`通知所有工作线程。
   - 然后，等待所有工作线程完成任务并退出。

### 3.3 提交任务到线程池

#### 代码示例

```cpp
int main() {
    ThreadPool pool(4);
    std::vector<std::future<int>> results;

    for (int i = 0; i < 8; ++i) {
        results.emplace_back(
            pool.enqueue([i] {
                std::cout << "hello " << i << std::endl;
                std::this_thread::sleep_for(std::chrono::seconds(1));
                std::cout << "world " << i << std::endl;
                return i * i;
            })
        );
    }

    for (auto&& result : results)
        std::cout << result.get() << ' ';
    std::cout << std::endl;

    return 0;
}
```

#### 代码解析

1. **创建线程池**：
   ```cpp
   ThreadPool pool(4);
   ```
   - 创建一个包含4个工作线程的线程池。

2. **提交任务**：
   ```cpp
   for (int i = 0; i < 8; ++i) {
       results.emplace_back(
           pool.enqueue([i] {
               std::cout << "hello " << i << std::endl;
               std::this_thread::sleep_for(std::chrono::seconds(1));
               std::cout << "world " << i << std::endl;
               return i * i;
           })
       );
   }
   ```
   - 通过循环向线程池提交8个任务。
   - 每个任务是一个lambda函数，它打印`hello`和`world`，并返回`i * i`。

3. **获取任务结果**：
   ```cpp
   for (auto&& result : results)
       std::cout << result.get() << ' ';
   std::cout << std::endl;
   ```
   - 通过`std::future::get()`获取每个任务的执行结果，并打印出来。

### 3.4 关闭线程池

#### 代码示例

```cpp
~ThreadPool() {
    {
        std::unique_lock<std::mutex> lock(queue_mutex);
        stop = true;
    }
    condition.notify_all();
    for (std::thread& worker : workers)
        worker.join();
}
```

#### 代码解析

1. **设置停止标志**：
   ```cpp
   {
       std::unique_lock<std::mutex> lock(queue_mutex);
       stop = true;
   }
   ```
   - 通过`std::unique_lock`锁定任务队列，并将`stop`标志设置为`true`，表示线程池即将关闭。

2. **通知所有线程**：
   ```cpp
   condition.notify_all();
   ```
   - 通过`condition.notify_all()`通知所有工作线程，告知它们线程池即将关闭。

3. **等待所有线程完成**：
   ```cpp
   for (std::thread& worker : workers)
       worker.join();
   ```
   - 遍历所有工作线程，并通过`join()`等待它们完成当前任务并退出。

## 4. 任务队列与调度

### 4.1 任务队列的作用

任务队列是线程池的核心组件之一，它的主要作用是存储待执行的任务。任务队列通常是一个先进先出（FIFO）的数据结构，任务按照提交的顺序依次被线程池中的工作线程执行。任务队列的设计和实现直接影响到线程池的性能和稳定性。

任务队列的主要作用包括：

1. **任务存储**：任务队列用于存储提交到线程池的任务，确保任务不会丢失。
2. **任务调度**：任务队列是任务调度的基础，线程池中的工作线程从任务队列中获取任务并执行。
3. **线程安全**：任务队列通常需要支持多线程并发访问，因此需要使用锁或其他同步机制来保证线程安全。

### 4.2 任务队列的实现

#### 代码示例

```cpp
#include <iostream>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <functional>

class TaskQueue {
public:
    void enqueue(std::function<void()> task) {
        std::unique_lock<std::mutex> lock(mutex_);
        tasks_.push(task);
        condition_.notify_one();  // 通知一个等待的线程
    }

    std::function<void()> dequeue() {
        std::unique_lock<std::mutex> lock(mutex_);
        condition_.wait(lock, [this] { return !tasks_.empty(); });  // 等待任务
        auto task = tasks_.front();
        tasks_.pop();
        return task;
    }

private:
    std::queue<std::function<void()>> tasks_;
    std::mutex mutex_;
    std::condition_variable condition_;
};

int main() {
    TaskQueue taskQueue;

    // 生产者线程：提交任务
    std::thread producer([&taskQueue]() {
        for (int i = 0; i < 5; ++i) {
            taskQueue.enqueue([i]() {
                std::cout << "Task " << i << " is running" << std::endl;
            });
        }
    });

    // 消费者线程：执行任务
    std::thread consumer([&taskQueue]() {
        for (int i = 0; i < 5; ++i) {
            auto task = taskQueue.dequeue();
            task();
        }
    });

    producer.join();
    consumer.join();

    return 0;
}
```

好的！以下是对您提供的代码的详细解析。

---

#### 代码解析

##### 1. 任务队列类定义

```cpp
class TaskQueue {
public:
    void enqueue(std::function<void()> task, int priority) {
        std::unique_lock<std::mutex> lock(mutex_);
        tasks_.emplace_back(std::move(task), priority);
        condition_.notify_one();
    }

    std::function<void()> dequeue() {
        std::unique_lock<std::mutex> lock(mutex_);
        condition_.wait(lock, [this] { return !tasks_.empty(); });

        // 找到优先级最高的任务
        auto it = tasks_.begin();
        for (auto i = tasks_.begin(); i != tasks_.end(); ++i) {
            if (i->second > it->second) {
                it = i;
            }
        }

        // 移除并返回最高优先级的任务
        std::function<void()> highest_task = std::move(it->first);
        tasks_.erase(it);
        return highest_task;
    }

private:
    std::list<std::pair<std::function<void()>, int>> tasks_;
    std::mutex mutex_;
    std::condition_variable condition_;
};
```

###### 1.1 `enqueue` 方法

```cpp
void enqueue(std::function<void()> task, int priority) {
    std::unique_lock<std::mutex> lock(mutex_);
    tasks_.emplace_back(std::move(task), priority);
    condition_.notify_one();
}
```

- **功能**：将任务和优先级加入任务队列。
- **参数**：
  - `task`：一个可调用对象（如 lambda 函数），表示要执行的任务。
  - `priority`：任务的优先级，值越大优先级越高。
- **实现细节**：
  - 使用 `std::unique_lock<std::mutex>` 锁定互斥锁 `mutex_`，确保线程安全。
  - 使用 `tasks_.emplace_back(std::move(task), priority)` 将任务和优先级打包成 `std::pair`，并添加到 `tasks_` 的末尾。
  - 使用 `condition_.notify_one()` 通知一个等待的线程，表示有新任务加入队列。

###### 1.2 `dequeue` 方法

```cpp
std::function<void()> dequeue() {
    std::unique_lock<std::mutex> lock(mutex_);
    condition_.wait(lock, [this] { return !tasks_.empty(); });

    // 找到优先级最高的任务
    auto it = tasks_.begin();
    for (auto i = tasks_.begin(); i != tasks_.end(); ++i) {
        if (i->second > it->second) {
            it = i;
        }
    }

    // 移除并返回最高优先级的任务
    std::function<void()> highest_task = std::move(it->first);
    tasks_.erase(it);
    return highest_task;
}
```

- **功能**：从任务队列中取出优先级最高的任务。
- **实现细节**：
  - 使用 `std::unique_lock<std::mutex>` 锁定互斥锁 `mutex_`，确保线程安全。
  - 使用 `condition_.wait(lock, [this] { return !tasks_.empty(); })` 等待任务队列非空。
  - 遍历 `tasks_`，找到优先级最高的任务（即 `priority` 最大的任务）。
  - 使用 `std::move(it->first)` 将任务的函数对象移动到 `highest_task`。
  - 使用 `tasks_.erase(it)` 从任务队列中移除该任务。
  - 返回 `highest_task`，供消费者线程执行。

###### 1.3 成员变量

```cpp
std::list<std::pair<std::function<void()>, int>> tasks_;
std::mutex mutex_;
std::condition_variable condition_;
```

- **`tasks_`**：任务队列，使用 `std::list` 存储任务和优先级的 `std::pair`。
  - 使用 `std::list` 而不是 `std::vector` 的原因是：`std::list` 的插入和删除操作是常数时间复杂度，适合频繁的插入和删除操作。
- **`mutex_`**：互斥锁，用于保护任务队列的线程安全。
- **`condition_`**：条件变量，用于线程间的等待和通知。

---

##### 2. 生产者线程

```cpp
std::thread producer([&taskQueue]() {
    taskQueue.enqueue([]() { std::cout << "Task 1 (Priority 1) is running" << std::endl; }, 1);
    taskQueue.enqueue([]() { std::cout << "Task 2 (Priority 3) is running" << std::endl; }, 3);
    taskQueue.enqueue([]() { std::cout << "Task 3 (Priority 2) is running" << std::endl; }, 2);
});
```

- **功能**：向任务队列中提交任务。
- **实现细节**：
  - 创建一个 lambda 函数，作为生产者线程的执行逻辑。
  - 调用 `taskQueue.enqueue` 方法，向任务队列中添加三个任务，每个任务具有不同的优先级。

---

##### 3. 消费者线程

```cpp
std::thread consumer([&taskQueue]() {
    for (int i = 0; i < 3; ++i) {
        auto task = taskQueue.dequeue();
        task();
    }
});
```

- **功能**：从任务队列中取出任务并执行。
- **实现细节**：
  - 创建一个 lambda 函数，作为消费者线程的执行逻辑。
  - 使用 `for` 循环，从任务队列中取出三个任务。
  - 调用 `taskQueue.dequeue()` 方法，取出优先级最高的任务。
  - 调用 `task()` 执行任务。

---

##### 4. 主函数

```cpp
int main() {
    TaskQueue taskQueue;

    // 生产者线程：提交任务
    std::thread producer([&taskQueue]() {
        taskQueue.enqueue([]() { std::cout << "Task 1 (Priority 1) is running" << std::endl; }, 1);
        taskQueue.enqueue([]() { std::cout << "Task 2 (Priority 3) is running" << std::endl; }, 3);
        taskQueue.enqueue([]() { std::cout << "Task 3 (Priority 2) is running" << std::endl; }, 2);
    });

    // 消费者线程：执行任务
    std::thread consumer([&taskQueue]() {
        for (int i = 0; i < 3; ++i) {
            auto task = taskQueue.dequeue();
            task();
        }
    });

    producer.join();
    consumer.join();

    return 0;
}
```

- **功能**：创建任务队列、生产者线程和消费者线程，并等待它们完成。
- **实现细节**：
  - 创建 `TaskQueue` 对象 `taskQueue`。
  - 创建生产者线程 `producer`，向任务队列中提交任务。
  - 创建消费者线程 `consumer`，从任务队列中取出任务并执行。
  - 使用 `join()` 等待生产者和消费者线程完成。

---

##### 5. 运行结果

运行上述代码后，输出结果如下：

```
Task 2 (Priority 3) is running
Task 3 (Priority 2) is running
Task 1 (Priority 1) is running
```

- **解释**：
  - 任务按照优先级顺序执行，优先级最高的任务最先执行。
  - 任务 2 的优先级最高（3），因此最先执行。
  - 任务 3 的优先级次之（2），第二个执行。
  - 任务 1 的优先级最低（1），最后执行。

---

##### 总结

这段代码实现了一个支持优先级调度的任务队列，使用 `std::list` 存储任务和优先级，并通过 `std::mutex` 和 `std::condition_variable` 确保线程安全。生产者线程向任务队列中提交任务，消费者线程从任务队列中取出优先级最高的任务并执行。通过合理的设计，任务队列可以高效地支持多线程并发操作。

### 4.3 任务调度策略

任务调度策略决定了任务如何被分配给线程池中的工作线程。常见的调度策略包括：

1. **FIFO（先进先出）**：任务按照提交的顺序依次执行，这是最简单的调度策略。
2. **优先级调度**：任务根据优先级进行调度，优先级高的任务优先执行。
3. **公平调度**：确保每个任务都有机会被执行，避免某些任务长时间等待。

好的！以下是分别实现 **FIFO（先进先出）**、**优先级调度** 和 **公平调度** 三种任务调度策略的代码，并对每种策略进行详细解析。

---

#### 4.3.1  FIFO（先进先出）调度策略

##### 代码示例

```cpp
#include <iostream>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <thread>

class TaskQueueFIFO {
public:
    void enqueue(std::function<void()> task) {
        std::unique_lock<std::mutex> lock(mutex_);
        tasks_.push(task);
        condition_.notify_one();
    }

    std::function<void()> dequeue() {
        std::unique_lock<std::mutex> lock(mutex_);
        condition_.wait(lock, [this] { return !tasks_.empty(); });

        // FIFO 调度：直接取出队首任务
        std::function<void()> task = tasks_.front();
        tasks_.pop();
        return task;
    }

private:
    std::queue<std::function<void()>> tasks_;
    std::mutex mutex_;
    std::condition_variable condition_;
};

int main() {
    TaskQueueFIFO taskQueue;

    // 生产者线程：提交任务
    std::thread producer([&taskQueue]() {
        taskQueue.enqueue([]() { std::cout << "Task 1 is running" << std::endl; });
        taskQueue.enqueue([]() { std::cout << "Task 2 is running" << std::endl; });
        taskQueue.enqueue([]() { std::cout << "Task 3 is running" << std::endl; });
    });

    // 消费者线程：执行任务
    std::thread consumer([&taskQueue]() {
        for (int i = 0; i < 3; ++i) {
            auto task = taskQueue.dequeue();
            task();
        }
    });

    producer.join();
    consumer.join();

    return 0;
}
```

##### 代码解析

1. **`enqueue` 方法**：
   - 使用 `std::queue` 存储任务，任务按照先进先出的顺序排列。
   - 使用 `std::unique_lock` 锁定互斥锁，确保线程安全。
   - 将任务加入队列，并通过 `condition_.notify_one()` 通知消费者线程。

2. **`dequeue` 方法**：
   - 使用 `std::unique_lock` 锁定互斥锁，确保线程安全。
   - 使用 `condition_.wait` 等待任务队列非空。
   - 直接取出队首任务（FIFO 调度），并从队列中移除。

3. **运行结果**：
   - 任务按照提交顺序执行：Task 1 -> Task 2 -> Task 3。

---

#### 4.3.2 优先级调度策略

##### 代码示例

```cpp
#include <iostream>
#include <list>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <thread>

class TaskQueuePriority {
public:
    void enqueue(std::function<void()> task, int priority) {
        std::unique_lock<std::mutex> lock(mutex_);
        tasks_.emplace_back(std::move(task), priority);
        condition_.notify_one();
    }

    std::function<void()> dequeue() {
        std::unique_lock<std::mutex> lock(mutex_);
        condition_.wait(lock, [this] { return !tasks_.empty(); });

        // 优先级调度：找到优先级最高的任务
        auto it = tasks_.begin();
        for (auto i = tasks_.begin(); i != tasks_.end(); ++i) {
            if (i->second > it->second) {
                it = i;
            }
        }

        // 移除并返回最高优先级的任务
        std::function<void()> highest_task = std::move(it->first);
        tasks_.erase(it);
        return highest_task;
    }

private:
    std::list<std::pair<std::function<void()>, int>> tasks_;
    std::mutex mutex_;
    std::condition_variable condition_;
};

int main() {
    TaskQueuePriority taskQueue;

    // 生产者线程：提交任务
    std::thread producer([&taskQueue]() {
        taskQueue.enqueue([]() { std::cout << "Task 1 (Priority 1) is running" << std::endl; }, 1);
        taskQueue.enqueue([]() { std::cout << "Task 2 (Priority 3) is running" << std::endl; }, 3);
        taskQueue.enqueue([]() { std::cout << "Task 3 (Priority 2) is running" << std::endl; }, 2);
    });

    // 消费者线程：执行任务
    std::thread consumer([&taskQueue]() {
        for (int i = 0; i < 3; ++i) {
            auto task = taskQueue.dequeue();
            task();
        }
    });

    producer.join();
    consumer.join();

    return 0;
}
```

##### 代码解析

1. **`enqueue` 方法**：
   - 使用 `std::list` 存储任务和优先级。
   - 将任务和优先级打包成 `std::pair`，并加入任务队列。
   - 使用 `condition_.notify_one()` 通知消费者线程。

2. **`dequeue` 方法**：
   - 使用 `condition_.wait` 等待任务队列非空。
   - 遍历任务队列，找到优先级最高的任务。
   - 移除并返回最高优先级的任务。

3. **运行结果**：
   - 任务按照优先级顺序执行：Task 2 (Priority 3) -> Task 3 (Priority 2) -> Task 1 (Priority 1)。

---

#### 4.3.3 公平调度策略

##### 代码示例

```cpp
#include <iostream>
#include <deque>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <thread>

class TaskQueueFair {
public:
    void enqueue(std::function<void()> task) {
        std::unique_lock<std::mutex> lock(mutex_);
        tasks_.push_back(task);
        condition_.notify_one();
    }

    std::function<void()> dequeue() {
        std::unique_lock<std::mutex> lock(mutex_);
        condition_.wait(lock, [this] { return !tasks_.empty(); });

        // 公平调度：轮询取出任务
        std::function<void()> task = tasks_.front();
        tasks_.pop_front();
        return task;
    }

private:
    std::deque<std::function<void()>> tasks_;
    std::mutex mutex_;
    std::condition_variable condition_;
};

int main() {
    TaskQueueFair taskQueue;

    // 生产者线程：提交任务
    std::thread producer([&taskQueue]() {
        taskQueue.enqueue([]() { std::cout << "Task 1 is running" << std::endl; });
        taskQueue.enqueue([]() { std::cout << "Task 2 is running" << std::endl; });
        taskQueue.enqueue([]() { std::cout << "Task 3 is running" << std::endl; });
    });

    // 消费者线程：执行任务
    std::thread consumer([&taskQueue]() {
        for (int i = 0; i < 3; ++i) {
            auto task = taskQueue.dequeue();
            task();
        }
    });

    producer.join();
    consumer.join();

    return 0;
}
```

##### 代码解析

1. **`enqueue` 方法**：
   - 使用 `std::deque` 存储任务，支持双端操作。
   - 将任务加入队列的末尾。
   - 使用 `condition_.notify_one()` 通知消费者线程。

2. **`dequeue` 方法**：
   - 使用 `condition_.wait` 等待任务队列非空。
   - 轮询取出队首任务（公平调度），并从队列中移除。

3. **运行结果**：
   - 任务按照提交顺序执行：Task 1 -> Task 2 -> Task 3。

---

#### 总结

以下是三种调度策略的对比：

| 调度策略   | 实现方式     | 任务执行顺序                           |
| ---------- | ------------ | -------------------------------------- |
| FIFO       | `std::queue` | 先进先出                               |
| 优先级调度 | `std::list`  | 优先级最高的任务最先执行               |
| 公平调度   | `std::deque` | 轮询取出任务，确保每个任务都有机会执行 |

通过合理选择调度策略，可以满足不同场景的需求。例如：
- **FIFO** 适合任务顺序固定的场景。
- **优先级调度** 适合需要优先处理高优先级任务的场景。
- **公平调度** 适合需要确保每个任务都有机会执行的场景。

## 5. 线程池的实现

### 5.1 使用C++11标准库实现线程池

#### 代码示例

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <future>

class ThreadPool {
public:
    ThreadPool(size_t threads) : stop(false) {
        for (size_t i = 0; i < threads; ++i) {
            workers.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(this->queue_mutex);
                        this->condition.wait(lock, [this] { return this->stop || !this->tasks.empty(); });
                        if (this->stop && this->tasks.empty())
                            return;
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    task();
                }
            });
        }
    }

    template <class F, class... Args>
    auto enqueue(F&& f, Args&&... args) -> std::future<typename std::result_of<F(Args...)>::type> {
        using return_type = typename std::result_of<F(Args...)>::type;
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...));
        std::future<return_type> res = task->get_future();
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            if (stop)
                throw std::runtime_error("enqueue on stopped ThreadPool");
            tasks.emplace([task]() { (*task)(); });
        }
        condition.notify_one();
        return res;
    }

    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            stop = true;
        }
        condition.notify_all();
        for (std::thread& worker : workers)
            worker.join();
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex queue_mutex;
    std::condition_variable condition;
    bool stop;
};

int main() {
    ThreadPool pool(4);
    std::vector<std::future<int>> results;

    for (int i = 0; i < 8; ++i) {
        results.emplace_back(
            pool.enqueue([i] {
                std::cout << "hello " << i << std::endl;
                std::this_thread::sleep_for(std::chrono::seconds(1));
                std::cout << "world " << i << std::endl;
                return i * i;
            })
        );
    }

    for (auto&& result : results)
        std::cout << result.get() << ' ';
    std::cout << std::endl;

    return 0;
}
```

#### 代码解析

1. **线程池类定义**：
   ```cpp
   class ThreadPool {
   public:
       ThreadPool(size_t threads) : stop(false) {
           for (size_t i = 0; i < threads; ++i) {
               workers.emplace_back([this] {
                   while (true) {
                       std::function<void()> task;
                       {
                           std::unique_lock<std::mutex> lock(this->queue_mutex);
                           this->condition.wait(lock, [this] { return this->stop || !this->tasks.empty(); });
                           if (this->stop && this->tasks.empty())
                               return;
                           task = std::move(this->tasks.front());
                           this->tasks.pop();
                       }
                       task();
                   }
               });
           }
       }
   ```
   - `ThreadPool`类是线程池的核心类，它包含一个构造函数，用于初始化线程池。
   - `workers`是一个`std::vector<std::thread>`，用于存储工作线程。
   - 构造函数中，通过循环创建指定数量的线程，并将它们放入`workers`中。
   - 每个线程的执行逻辑是：不断从任务队列中获取任务并执行，直到线程池被关闭且任务队列为空。

2. **任务提交**：
   ```cpp
   template <class F, class... Args>
   auto enqueue(F&& f, Args&&... args) -> std::future<typename std::result_of<F(Args...)>::type> {
       using return_type = typename std::result_of<F(Args...)>::type;
       auto task = std::make_shared<std::packaged_task<return_type()>>(
           std::bind(std::forward<F>(f), std::forward<Args>(args)...));
       std::future<return_type> res = task->get_future();
       {
           std::unique_lock<std::mutex> lock(queue_mutex);
           if (stop)
               throw std::runtime_error("enqueue on stopped ThreadPool");
           tasks.emplace([task]() { (*task)(); });
       }
       condition.notify_one();
       return res;
   }
   ```
   - `enqueue`函数用于向线程池提交任务。
   - 它接受一个可调用对象`f`和它的参数`args`，并返回一个`std::future`对象，用于获取任务的执行结果。
   - 任务被封装为一个`std::packaged_task`，并放入任务队列`tasks`中。
   - 最后，通过`condition.notify_one()`通知一个工作线程来执行任务。

3. **线程池关闭**：
   ```cpp
   ~ThreadPool() {
       {
           std::unique_lock<std::mutex> lock(queue_mutex);
           stop = true;
       }
       condition.notify_all();
       for (std::thread& worker : workers)
           worker.join();
   }
   ```
   - 析构函数用于关闭线程池。
   - 首先，将`stop`标志设置为`true`，并通过`condition.notify_all()`通知所有工作线程。
   - 然后，等待所有工作线程完成任务并退出。

### 5.2 使用第三方库实现线程池

#### 代码示例

```cpp
#include <iostream>
#include <thread>
#include <future>
#include <vector>
#include "ctpl_stl.h"  // 使用ctpl线程池库

int main() {
    ctpl::thread_pool pool(4);  // 创建一个包含4个线程的线程池
    std::vector<std::future<int>> results;

    for (int i = 0; i < 8; ++i) {
        results.emplace_back(
            pool.push([i](int) {
                std::cout << "hello " << i << std::endl;
                std::this_thread::sleep_for(std::chrono::seconds(1));
                std::cout << "world " << i << std::endl;
                return i * i;
            })
        );
    }

    for (auto&& result : results)
        std::cout << result.get() << ' ';
    std::cout << std::endl;

    return 0;
}
```

#### 代码解析

1. **引入第三方库**：
   ```cpp
   #include "ctpl_stl.h"
   ```
   - 使用`ctpl`线程池库，它是一个轻量级的C++线程池库，提供了简单易用的接口。

2. **创建线程池**：
   ```cpp
   ctpl::thread_pool pool(4);
   ```
   - 创建一个包含4个线程的线程池。

3. **提交任务**：
   ```cpp
   for (int i = 0; i < 8; ++i) {
       results.emplace_back(
           pool.push([i](int) {
               std::cout << "hello " << i << std::endl;
               std::this_thread::sleep_for(std::chrono::seconds(1));
               std::cout << "world " << i << std::endl;
               return i * i;
           })
       );
   }
   ```
   - 通过`pool.push`方法向线程池提交任务。
   - 每个任务是一个lambda函数，它打印`hello`和`world`，并返回`i * i`。

4. **获取任务结果**：
   ```cpp
   for (auto&& result : results)
       std::cout << result.get() << ' ';
   std::cout << std::endl;
   ```
   - 通过`std::future::get()`获取每个任务的执行结果，并打印出来。

## 6. 线程池的应用场景

线程池在现代软件开发中有着广泛的应用，尤其是在需要高效处理大量并发任务的场景中。以下是几个典型的应用场景。

### 6.1 并发处理大量任务

在需要处理大量任务的场景中，线程池可以显著提高任务处理的效率。通过将任务分配给线程池中的多个线程，可以避免为每个任务创建和销毁线程的开销。

#### 代码示例

```cpp
#include <iostream>
#include <vector>
#include <thread>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <future>

class ThreadPool {
public:
    ThreadPool(size_t threads) : stop(false) {
        for(size_t i = 0; i < threads; ++i)
            workers.emplace_back([this] {
                for(;;) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(this->queue_mutex);
                        this->condition.wait(lock, [this]{ return this->stop || !this->tasks.empty(); });
                        if(this->stop && this->tasks.empty())
                            return;
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    task();
                }
            });
    }

    template<class F, class... Args>
    auto enqueue(F&& f, Args&&... args) -> std::future<typename std::result_of<F(Args...)>::type> {
        using return_type = typename std::result_of<F(Args...)>::type;
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        std::future<return_type> res = task->get_future();
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            if(stop)
                throw std::runtime_error("enqueue on stopped ThreadPool");
            tasks.emplace([task](){ (*task)(); });
        }
        condition.notify_one();
        return res;
    }

    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            stop = true;
        }
        condition.notify_all();
        for(std::thread &worker: workers)
            worker.join();
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex queue_mutex;
    std::condition_variable condition;
    bool stop;
};

int main() {
    ThreadPool pool(4);
    std::vector<std::future<int>> results;

    for(int i = 0; i < 8; ++i) {
        results.emplace_back(
            pool.enqueue([i] {
                std::cout << "task " << i << " is running\n";
                std::this_thread::sleep_for(std::chrono::seconds(1));
                std::cout << "task " << i << " is done\n";
                return i * i;
            })
        );
    }

    for(auto && result: results)
        std::cout << result.get() << ' ';

    return 0;
}
```

#### 代码解析

1. **ThreadPool类定义**：
   - `ThreadPool(size_t threads)`：构造函数，创建指定数量的线程，并将它们放入线程池中。
   - `enqueue`方法：用于将任务加入任务队列，并返回一个`std::future`对象，以便获取任务的执行结果。
   - `~ThreadPool()`：析构函数，确保所有线程在销毁线程池时正确退出。

2. **任务处理逻辑**：
   - 每个线程在创建后进入一个无限循环，等待任务队列中的任务。
   - 当有任务加入队列时，线程会从队列中取出任务并执行。

3. **main函数**：
   - 创建一个包含4个线程的线程池。
   - 将8个任务加入线程池，并获取每个任务的执行结果。
   - 最后，输出每个任务的执行结果。

### 6.2 网络服务器处理请求

在网络服务器中，线程池可以用于处理客户端的并发请求。通过将每个请求分配给线程池中的一个线程，可以避免为每个请求创建新线程的开销。

#### 代码示例

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <functional>
#include <future>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

class ThreadPool {
public:
    ThreadPool(size_t threads) : stop(false) {
        for(size_t i = 0; i < threads; ++i)
            workers.emplace_back([this] {
                for(;;) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(this->queue_mutex);
                        this->condition.wait(lock, [this]{ return this->stop || !this->tasks.empty(); });
                        if(this->stop && this->tasks.empty())
                            return;
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    task();
                }
            });
    }

    template<class F, class... Args>
    auto enqueue(F&& f, Args&&... args) -> std::future<typename std::result_of<F(Args...)>::type> {
        using return_type = typename std::result_of<F(Args...)>::type;
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        std::future<return_type> res = task->get_future();
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            if(stop)
                throw std::runtime_error("enqueue on stopped ThreadPool");
            tasks.emplace([task](){ (*task)(); });
        }
        condition.notify_one();
        return res;
    }

    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            stop = true;
        }
        condition.notify_all();
        for(std::thread &worker: workers)
            worker.join();
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex queue_mutex;
    std::condition_variable condition;
    bool stop;
};

void handle_client(int client_socket) {
    char buffer[1024] = {0};
    read(client_socket, buffer, 1024);
    std::cout << "Received: " << buffer << std::endl;
    std::string response = "HTTP/1.1 200 OK\nContent-Type: text/html\n\n<html><body><h1>Hello, World!</h1></body></html>";
    write(client_socket, response.c_str(), response.size());
    close(client_socket);
}

int main() {
    int server_fd, new_socket;
    struct sockaddr_in address;
    int addrlen = sizeof(address);

    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(8080);

    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }

    if (listen(server_fd, 3) < 0) {
        perror("listen");
        exit(EXIT_FAILURE);
    }

    ThreadPool pool(4);

    while (true) {
        if ((new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen)) < 0) {
            perror("accept");
            exit(EXIT_FAILURE);
        }
        pool.enqueue([new_socket] { handle_client(new_socket); });
    }

    return 0;
}
```

#### 代码解析

1. **ThreadPool类定义**：
   - 与前面的示例相同，定义了一个线程池类，用于管理多个线程。

2. **handle_client函数**：
   - 该函数用于处理客户端的请求。它从客户端读取数据，并发送一个简单的HTTP响应。

3. **main函数**：
   - 创建一个TCP服务器，监听8080端口。
   - 当有新的客户端连接时，将连接的套接字传递给线程池中的一个线程进行处理。

### 6.3 并行计算

在并行计算中，线程池可以用于将计算任务分配给多个线程，从而加速计算过程。

#### 代码示例

```cpp
#include <iostream>
#include <vector>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <functional>
#include <future>
#include <numeric>

class ThreadPool {
public:
    ThreadPool(size_t threads) : stop(false) {
        for(size_t i = 0; i < threads; ++i)
            workers.emplace_back([this] {
                for(;;) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(this->queue_mutex);
                        this->condition.wait(lock, [this]{ return this->stop || !this->tasks.empty(); });
                        if(this->stop && this->tasks.empty())
                            return;
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    task();
                }
            });
    }

    template<class F, class... Args>
    auto enqueue(F&& f, Args&&... args) -> std::future<typename std::result_of<F(Args...)>::type> {
        using return_type = typename std::result_of<F(Args...)>::type;
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        std::future<return_type> res = task->get_future();
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            if(stop)
                throw std::runtime_error("enqueue on stopped ThreadPool");
            tasks.emplace([task](){ (*task)(); });
        }
        condition.notify_one();
        return res;
    }

    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            stop = true;
        }
        condition.notify_all();
        for(std::thread &worker: workers)
            worker.join();
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex queue_mutex;
    std::condition_variable condition;
    bool stop;
};

int main() {
    ThreadPool pool(4);
    std::vector<std::future<int>> results;

    for(int i = 0; i < 8; ++i) {
        results.emplace_back(
            pool.enqueue([i] {
                std::cout << "Computing factorial of " << i << "\n";
                int factorial = 1;
                for(int j = 1; j <= i; ++j) {
                    factorial *= j;
                }
                std::cout << "Factorial of " << i << " is " << factorial << "\n";
                return factorial;
            })
        );
    }

    int total = 0;
    for(auto && result: results)
        total += result.get();

    std::cout << "Total factorial sum: " << total << std::endl;

    return 0;
}
```

#### 代码解析

1. **ThreadPool类定义**：
   - 与前面的示例相同，定义了一个线程池类，用于管理多个线程。

2. **main函数**：
   - 创建一个包含4个线程的线程池。
   - 将8个计算阶乘的任务加入线程池，并获取每个任务的执行结果。
   - 最后，输出所有阶乘结果的总和。

## 7. 线程池的优缺点

### 7.1 线程池的优点

- **提高性能**：通过重用线程，避免了频繁创建和销毁线程的开销。
- **资源管理**：限制了同时运行的线程数量，避免了系统资源被耗尽。
- **简化编程模型**：开发者只需将任务加入任务队列，而不需要手动管理线程的生命周期。

### 7.2 线程池的缺点

- **复杂性**：线程池的实现和管理相对复杂，需要处理线程同步、任务调度等问题。
- **调试困难**：多线程环境下的调试比单线程环境更加困难，容易出现死锁、竞态条件等问题。
- **灵活性有限**：线程池的线程数量是固定的，无法动态调整，可能在某些场景下不够灵活。

## 8. 总结与展望

### 8.1 线程池的未来发展

随着多核处理器的普及和并发编程需求的增加，线程池技术将继续发展。未来的线程池可能会更加智能化，能够根据系统负载动态调整线程数量，或者支持更复杂的任务调度策略。

### 8.2 进一步学习的资源推荐

- **书籍**：《C++并发编程实战》（C++ Concurrency in Action）
- **在线资源**：C++官方文档、Stack Overflow、GitHub上的开源项目
- **课程**：Coursera、Udemy等平台上的并发编程课程

