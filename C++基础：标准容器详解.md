### C++标准容器详解

#### 1. 引言

##### 1.1 C++标准容器概述
C++标准库提供了一系列的容器，这些容器是用于存储和组织数据的类模板。标准容器分为序列容器、关联容器、无序关联容器和容器适配器。每种容器都有其特定的用途和性能特性，开发者可以根据需求选择合适的容器。

##### 1.2 为什么使用标准容器
使用标准容器的好处包括：
- **高效性**：标准容器经过优化，能够提供高效的插入、删除和查找操作。
- **易用性**：标准容器提供了丰富的接口，使用方便。
- **可移植性**：标准容器是C++标准库的一部分，具有良好的可移植性。

##### 1.3 标准容器的基本分类
- **序列容器**：`std::vector`, `std::deque`, `std::list`, `std::forward_list`
- **关联容器**：`std::set`, `std::map`, `std::multiset`, `std::multimap`
- **无序关联容器**：`std::unordered_set`, `std::unordered_map`, `std::unordered_multiset`, `std::unordered_multimap`
- **容器适配器**：`std::stack`, `std::queue`, `std::priority_queue`

---

#### 2. 序列容器

##### 2.1 `std::vector`

###### 2.1.1 基本概念与特性
`std::vector` 是动态数组，支持随机访问。它的内存是连续的，因此在访问元素时非常高效。`std::vector` 会自动管理内存，当元素数量超过当前容量时，会自动重新分配内存。

###### 2.1.2 常用操作
- `push_back()`：在末尾插入元素。
- `pop_back()`：删除末尾元素。
- `size()`：返回元素数量。
- `capacity()`：返回当前容量。

###### 2.1.3 代码示例
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec;  // 创建一个空的vector

    // 插入元素
    vec.push_back(10);
    vec.push_back(20);
    vec.push_back(30);

    // 访问元素
    std::cout << "First element: " << vec[0] << std::endl;

    // 删除元素
    vec.pop_back();

    // 输出所有元素
    for (int i : vec) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

###### 2.1.4 代码解析
- `std::vector<int> vec;`：创建一个空的 `std::vector`。
- `vec.push_back(10);`：在 `vec` 的末尾插入元素 `10`。
- `vec.pop_back();`：删除 `vec` 的最后一个元素。
- `vec[0]`：通过下标访问 `vec` 的第一个元素。

---

##### 2.2 `std::deque`

###### 2.2.1 基本概念与特性
`std::deque`（双端队列）支持在两端高效地插入和删除元素。它的内存不是连续的，而是由多个块组成的。

###### 2.2.2 常用操作
- `push_front()`：在队列前端插入元素。
- `push_back()`：在队列后端插入元素。
- `pop_front()`：删除队列前端元素。
- `pop_back()`：删除队列后端元素。

###### 2.2.3 代码示例
```cpp
#include <iostream>
#include <deque>

int main() {
    std::deque<int> dq;  // 创建一个空的deque

    // 插入元素
    dq.push_front(10);
    dq.push_back(20);
    dq.push_front(5);

    // 访问元素
    std::cout << "First element: " << dq.front() << std::endl;

    // 删除元素
    dq.pop_front();

    // 输出所有元素
    for (int i : dq) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

###### 2.2.4 代码解析
- `std::deque<int> dq;`：创建一个空的 `std::deque`。
- `dq.push_front(10);`：在 `dq` 的前端插入元素 `10`。
- `dq.pop_front();`：删除 `dq` 的前端元素。
- `dq.front()`：访问 `dq` 的前端元素。

---

##### 2.3 `std::list`

###### 2.3.1 基本概念与特性
`std::list` 是双向链表，支持在任意位置高效地插入和删除元素。它的内存不是连续的，因此不支持随机访问。

###### 2.3.2 常用操作
- `push_back()`：在末尾插入元素。
- `push_front()`：在开头插入元素。
- `erase()`：删除指定位置的元素。

###### 2.3.3 代码示例
```cpp
#include <iostream>
#include <list>

int main() {
    std::list<int> lst;  // 创建一个空的list

    // 插入元素
    lst.push_back(10);
    lst.push_front(5);
    lst.push_back(20);

    // 删除元素
    lst.erase(lst.begin());

    // 输出所有元素
    for (int i : lst) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

###### 2.3.4 代码解析
- `std::list<int> lst;`：创建一个空的 `std::list`。
- `lst.push_back(10);`：在 `lst` 的末尾插入元素 `10`。
- `lst.erase(lst.begin());`：删除 `lst` 的第一个元素。

---

##### 2.4 `std::forward_list`

###### 2.4.1 基本概念与特性
`std::forward_list` 是单向链表，只支持单向遍历。它的内存不是连续的，因此不支持随机访问。

###### 2.4.2 常用操作
- `push_front()`：在开头插入元素。
- `erase_after()`：删除指定位置后的元素。

###### 2.4.3 代码示例
```cpp
#include <iostream>
#include <forward_list>

int main() {
    std::forward_list<int> flst;  // 创建一个空的forward_list

    // 插入元素
    flst.push_front(10);
    flst.push_front(5);
    flst.push_front(20);

    // 删除元素
    flst.erase_after(flst.begin());

    // 输出所有元素
    for (int i : flst) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

###### 2.4.4 代码解析
- `std::forward_list<int> flst;`：创建一个空的 `std::forward_list`。
- `flst.push_front(10);`：在 `flst` 的开头插入元素 `10`。
- `flst.erase_after(flst.begin());`：删除 `flst` 的第二个元素。

---

#### 3. 关联容器

##### 3.1 `std::set`

###### 3.1.1 基本概念与特性
`std::set` 是基于红黑树的集合，存储唯一的元素，且元素自动排序。

###### 3.1.2 常用操作
- `insert()`：插入元素。
- `find()`：查找元素。
- `erase()`：删除元素。

###### 3.1.3 代码示例
```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> s;  // 创建一个空的set

    // 插入元素
    s.insert(10);
    s.insert(5);
    s.insert(20);

    // 查找元素
    if (s.find(5) != s.end()) {
        std::cout << "Element found!" << std::endl;
    }

    // 删除元素
    s.erase(10);

    // 输出所有元素
    for (int i : s) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

###### 3.1.4 代码解析
- `std::set<int> s;`：创建一个空的 `std::set`。
- `s.insert(10);`：插入元素 `10`。
- `s.find(5)`：查找元素 `5`。
- `s.erase(10);`：删除元素 `10`。

---

#### 4. 无序关联容器

##### 4.1 `std::unordered_set`

###### 4.1.1 基本概念与特性
`std::unordered_set` 是基于哈希表的集合，存储唯一的元素，但不保证顺序。

###### 4.1.2 常用操作
- `insert()`：插入元素。
- `find()`：查找元素。
- `erase()`：删除元素。

###### 4.1.3 代码示例
```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> us;  // 创建一个空的unordered_set

    // 插入元素
    us.insert(10);
    us.insert(5);
    us.insert(20);

    // 查找元素
    if (us.find(5) != us.end()) {
        std::cout << "Element found!" << std::endl;
    }

    // 删除元素
    us.erase(10);

    // 输出所有元素
    for (int i : us) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

###### 4.1.4 代码解析
- `std::unordered_set<int> us;`：创建一个空的 `std::unordered_set`。
- `us.insert(10);`：插入元素 `10`。
- `us.find(5)`：查找元素 `5`。
- `us.erase(10);`：删除元素 `10`。

---

#### 5. 容器适配器

##### 5.1 `std::stack`

###### 5.1.1 基本概念与特性
`std::stack` 是基于 `std::deque` 的栈，遵循 LIFO（后进先出）原则。

###### 5.1.2 常用操作
- `push()`：插入元素。
- `pop()`：删除栈顶元素。
- `top()`：访问栈顶元素。

###### 5.1.3 代码示例
```cpp
#include <iostream>
#include <stack>

int main() {
    std::stack<int> st;  // 创建一个空的stack

    // 插入元素
    st.push(10);
    st.push(20);
    st.push(30);

    // 访问栈顶元素
    std::cout << "Top element: " << st.top() << std::endl;

    // 删除栈顶元素
    st.pop();

    // 输出栈顶元素
    std::cout << "Top element after pop: " << st.top() << std::endl;

    return 0;
}
```

###### 5.1.4 代码解析
- `std::stack<int> st;`：创建一个空的 `std::stack`。
- `st.push(10);`：插入元素 `10`。
- `st.top()`：访问栈顶元素。
- `st.pop();`：删除栈顶元素。

---

#### 6. 容器的迭代器

##### 6.1 迭代器的基本概念
迭代器是用于遍历容器中元素的对象。它类似于指针，但提供了更多的功能。

##### 6.2 迭代器的分类
- **输入迭代器**：只读，单向移动。
- **输出迭代器**：只写，单向移动。
- **前向迭代器**：可读写，单向移动。
- **双向迭代器**：可读写，双向移动。
- **随机访问迭代器**：可读写，随机访问。

##### 6.3 迭代器的常用操作
- `begin()`：返回指向容器第一个元素的迭代器。
- `end()`：返回指向容器末尾的迭代器。
- `++`：移动到下一个元素。
- `--`：移动到上一个元素。

##### 6.4 代码示例
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {10, 20, 30};

    // 使用迭代器遍历vector
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

##### 6.5 代码解析
- `vec.begin()`：返回指向 `vec` 第一个元素的迭代器。
- `vec.end()`：返回指向 `vec` 末尾的迭代器。
- `*it`：解引用迭代器，获取元素值。

---

#### 7. 容器的性能分析

##### 7.1 时间复杂度概述
- `std::vector`：随机访问 O(1)，插入/删除 O(n)。
- `std::list`：插入/删除 O(1)，随机访问 O(n)。
- `std::set`：插入/删除/查找 O(log n)。
- `std::unordered_set`：插入/删除/查找 O(1)（平均情况）。

##### 7.2 不同容器的性能对比
- 如果需要频繁随机访问，选择 `std::vector`。
- 如果需要频繁插入/删除，选择 `std::list`。
- 如果需要自动排序，选择 `std::set`。
- 如果需要快速查找，选择 `std::unordered_set`。

##### 7.3 选择合适容器的指南
根据操作的频率和性能需求选择合适的容器。

---

#### 8. 容器的自定义比较函数与哈希函数

##### 8.1 自定义比较函数

###### 8.1.1 基本概念
自定义比较函数用于指定容器的排序规则。

###### 8.1.2 代码示例
```cpp
#include <iostream>
#include <set>

struct Compare {
    bool operator()(int a, int b) const {
        return a > b;  // 降序排列
    }
};

int main() {
    std::set<int, Compare> s;

    s.insert(10);
    s.insert(5);
    s.insert(20);

    for (int i : s) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

###### 8.1.3 代码解析
- `struct Compare`：定义一个比较函数对象。
- `std::set<int, Compare> s;`：使用自定义比较函数创建 `std::set`。

##### 8.2 自定义哈希函数

###### 8.2.1 基本概念
自定义哈希函数用于指定 `std::unordered_set` 的哈希规则。

###### 8.2.2 代码示例
```cpp
#include <iostream>
#include <unordered_set>

struct Hash {
    std::size_t operator()(int x) const {
        return x % 10;  // 简单哈希函数
    }
};

int main() {
    std::unordered_set<int, Hash> us;

    us.insert(10);
    us.insert(5);
    us.insert(20);

    for (int i : us) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

###### 8.2.3 代码解析
- `struct Hash`：定义一个哈希函数对象。
- `std::unordered_set<int, Hash> us;`：使用自定义哈希函数创建 `std::unordered_set`。

---

#### 9. 容器的线程安全与并发

##### 9.1 线程安全的基本概念

**线程安全**是指在多线程环境下，程序能够正确地处理共享资源，避免数据竞争（Data Race）和不一致的状态。在多线程编程中，多个线程可能同时访问和修改同一个数据结构，如果没有适当的同步机制，可能会导致不可预期的结果。

---

##### 9.2 标准容器的线程安全问题

C++标准库中的容器（如 `std::vector`, `std::map`, `std::unordered_set` 等）**不是线程安全的**。这意味着在多线程环境中，如果多个线程同时对同一个容器进行读写操作，可能会导致以下问题：

1. **数据竞争（Data Race）**：多个线程同时修改同一个容器，导致数据不一致。
2. **未定义行为（Undefined Behavior）**：C++标准未定义多线程对非线程安全容器的并发操作，可能导致程序崩溃或产生不可预期的结果。

例如，以下代码在多线程环境中是**不安全**的：

```cpp
#include <iostream>
#include <vector>
#include <thread>

std::vector<int> vec;

void add_to_vector(int value) {
    vec.push_back(value);  // 多线程同时写入，可能导致数据竞争
}

int main() {
    std::thread t1(add_to_vector, 10);
    std::thread t2(add_to_vector, 20);

    t1.join();
    t2.join();

    for (int i : vec) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**问题分析**：
- `vec.push_back(value);` 是线程不安全的操作，多个线程可能同时修改 `vec`，导致数据竞争。
- 运行结果可能是 `10 20`，也可能是 `20 10`，甚至可能导致程序崩溃。

---

##### 9.3 如何实现容器的线程安全

为了在多线程环境中安全地使用容器，可以采用以下方法：

1. **使用互斥锁（Mutex）**：通过加锁来保护临界区，确保同一时间只有一个线程可以访问容器。
2. **使用原子操作（Atomic Operations）**：对于简单的操作，可以使用 `std::atomic` 来避免加锁。
3. **使用并发容器**：C++标准库提供了一些线程安全的容器（如 `std::atomic` 和 `std::shared_mutex`），但标准容器本身不是线程安全的。

---

##### 9.4 使用互斥锁实现线程安全

**互斥锁（Mutex）** 是一种同步机制，用于保护共享资源。通过在访问容器之前加锁，可以确保同一时间只有一个线程可以修改容器。

##### 9.4.1 代码示例：使用 `std::mutex` 保护容器

```cpp
#include <iostream>
#include <vector>
#include <thread>
#include <mutex>

std::vector<int> vec;
std::mutex mtx;  // 创建一个互斥锁

void add_to_vector(int value) {
    std::lock_guard<std::mutex> lock(mtx);  // 加锁
    vec.push_back(value);  // 线程安全的操作
}

int main() {
    std::thread t1(add_to_vector, 10);
    std::thread t2(add_to_vector, 20);

    t1.join();
    t2.join();

    for (int i : vec) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

###### 9.4.2 代码解析

- `std::mutex mtx;`：创建一个互斥锁。
- `std::lock_guard<std::mutex> lock(mtx);`：在进入临界区之前加锁，离开临界区时自动解锁。
- `vec.push_back(value);`：在加锁的保护下，线程安全地向容器中插入元素。

**运行结果**：
- 无论运行多少次，结果都是 `10 20`，因为加锁确保了线程安全。

---

##### 9.5 使用原子操作实现线程安全

对于某些简单的操作，可以使用 `std::atomic` 来避免加锁。`std::atomic` 提供了线程安全的原子操作，适用于单个变量的读写。

###### 9.5.1 代码示例：使用 `std::atomic` 实现计数器

```cpp
#include <iostream>
#include <atomic>
#include <thread>

std::atomic<int> counter(0);  // 创建一个原子变量

void increment_counter() {
    for (int i = 0; i < 1000; ++i) {
        counter++;  // 线程安全的原子操作
    }
}

int main() {
    std::thread t1(increment_counter);
    std::thread t2(increment_counter);

    t1.join();
    t2.join();

    std::cout << "Counter: " << counter << std::endl;

    return 0;
}
```

###### 9.5.2 代码解析

- `std::atomic<int> counter(0);`：创建一个原子变量，初始值为 `0`。
- `counter++;`：线程安全的原子操作，确保多个线程同时修改 `counter` 时不会发生数据竞争。

**运行结果**：
- 无论运行多少次，结果都是 `2000`，因为 `std::atomic` 确保了线程安全。

---

##### 9.6 并发容器简介

C++标准库提供了一些并发容器，适用于多线程环境。这些容器在内部实现了线程安全的操作，开发者无需手动加锁。

###### 9.6.1 `std::atomic`

`std::atomic` 提供了对单个变量的线程安全操作，适用于简单的计数器或标志位。

###### 9.6.2 `std::shared_mutex`

`std::shared_mutex` 是一种读写锁，允许多个线程同时读取共享资源，但只允许一个线程写入。

###### 9.6.3 代码示例：使用 `std::shared_mutex` 实现读写锁

```cpp
#include <iostream>
#include <map>
#include <shared_mutex>
#include <thread>

std::map<int, int> data;
std::shared_mutex mtx;  // 创建一个读写锁

void write_to_map(int key, int value) {
    std::unique_lock<std::shared_mutex> lock(mtx);  // 写锁
    data[key] = value;
}

void read_from_map(int key) {
    std::shared_lock<std::shared_mutex> lock(mtx);  // 读锁
    if (data.find(key) != data.end()) {
        std::cout << "Value: " << data[key] << std::endl;
    } else {
        std::cout << "Key not found!" << std::endl;
    }
}

int main() {
    std::thread t1(write_to_map, 1, 10);
    std::thread t2(read_from_map, 1);

    t1.join();
    t2.join();

    return 0;
}
```

###### 9.6.4 代码解析

- `std::shared_mutex mtx;`：创建一个读写锁。
- `std::unique_lock<std::shared_mutex> lock(mtx);`：写锁，确保只有一个线程可以写入。
- `std::shared_lock<std::shared_mutex> lock(mtx);`：读锁，允许多个线程同时读取。

**运行结果**：
- 写线程和读线程可以并发执行，写锁确保写操作的线程安全，读锁允许多个线程同时读取。

---

##### 9.7 总结

在多线程环境中使用容器时，必须注意线程安全问题。可以通过以下方式实现线程安全：

1. **使用互斥锁（Mutex）**：通过加锁保护临界区。
2. **使用原子操作（Atomic）**：对于简单的操作，使用 `std::atomic`。
3. **使用并发容器**：C++标准库提供了一些并发容器，适用于多线程环境。

通过合理选择同步机制，可以确保多线程程序的正确性和性能。