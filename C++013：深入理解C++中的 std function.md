---
title: '深入理解C++中的std::function'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
---

# 深入理解C++中的std::function：从入门到精通

在C++编程中，函数是一等公民，这意味着函数可以像变量一样被传递、存储和使用。为了实现这一特性，C++标准库提供了一个非常强大的工具——`std::function`。本文将带你从基础到深入，全面理解`std::function`的用法和原理。

## 1. 什么是`std::function`？

`std::function`是C++11引入的一个模板类，它用于封装可调用对象（Callable Objects）。可调用对象包括普通函数、函数指针、lambda表达式、函数对象（Functors）以及带有`operator()`的类的实例。

`std::function`的主要作用是提供一种统一的方式来存储、复制和调用这些可调用对象。无论你传递的是哪种类型的可调用对象，`std::function`都能正确地调用它。

## 2. 基本用法

我们先从一个简单的例子开始，了解`std::function`的基本用法。

### 示例1：封装普通函数

```cpp
#include <iostream>
#include <functional>

// 定义一个普通函数
void printHello() {
    std::cout << "Hello, World!" << std::endl;
}

int main() {
    // 使用std::function封装普通函数
    std::function<void()> func = printHello;

    // 调用封装的函数
    func();  // 输出: Hello, World!

    return 0;
}
```

**代码解析：**
- `std::function<void()>`表示一个返回类型为`void`，不接受任何参数的函数。
- `func = printHello;`将普通函数`printHello`封装到`std::function`对象`func`中。
- `func();`调用封装的函数，输出`Hello, World!`。

### 示例2：封装lambda表达式

```cpp
#include <iostream>
#include <functional>

int main() {
    // 使用std::function封装lambda表达式
    std::function<int(int, int)> add = [](int a, int b) {
        return a + b;
    };

    // 调用封装的lambda表达式
    std::cout << "Sum: " << add(3, 4) << std::endl;  // 输出: Sum: 7

    return 0;
}
```

**代码解析：**
- `std::function<int(int, int)>`表示一个返回类型为`int`，接受两个`int`参数的函数。
- `add = [](int a, int b) { return a + b; };`将lambda表达式封装到`std::function`对象`add`中。
- `add(3, 4);`调用封装的lambda表达式，输出`Sum: 7`。

## 3. 深入理解`std::function`

### 3.1 `std::function`的模板参数

`std::function`的模板参数是一个函数类型，用于指定被封装的可调用对象的签名。例如：

- `std::function<void()>`：返回类型为`void`，不接受参数的函数。
- `std::function<int(int, int)>`：返回类型为`int`，接受两个`int`参数的函数。

### 3.2 `std::function`的构造和赋值

`std::function`可以通过多种方式进行构造和赋值：

- 从普通函数、函数指针、lambda表达式、函数对象等可调用对象构造。
- 通过赋值操作符`=`将可调用对象赋值给`std::function`对象。

### 3.3 `std::function`的调用

`std::function`对象可以通过函数调用操作符`()`来调用封装的可调用对象。调用时，参数和返回值必须与模板参数指定的函数类型一致。

### 3.4 `std::function`的空状态

`std::function`可以处于空状态（即没有封装任何可调用对象）。你可以使用`std::function::operator bool()`来检查`std::function`对象是否为空。

```cpp
#include <iostream>
#include <functional>

int main() {
    std::function<void()> func;

    if (func) {
        std::cout << "Function is not empty." << std::endl;
    } else {
        std::cout << "Function is empty." << std::endl;  // 输出: Function is empty.
    }

    return 0;
}
```

## 4. 复杂示例：结合函数对象和多态

为了更好地理解`std::function`的强大之处，我们来看一个更复杂的例子，结合函数对象和多态。

### 示例3：函数对象与多态

```cpp
#include <iostream>
#include <functional>
#include <vector>

// 定义一个基类
class Base {
public:
    virtual void print() const = 0;  // 纯虚函数
};

// 定义一个派生类
class Derived : public Base {
public:
    void print() const override {
        std::cout << "Derived class" << std::endl;
    }
};

// 定义一个函数对象
class Printer {
public:
    void operator()(const Base& obj) const {
        obj.print();
    }
};

int main() {
    // 创建一个Derived对象
    Derived d;

    // 使用std::function封装函数对象
    std::function<void(const Base&)> func = Printer();

    // 调用封装的函数对象
    func(d);  // 输出: Derived class

    // 使用std::vector存储std::function对象
    std::vector<std::function<void(const Base&)>> functions;
    functions.push_back(Printer());
    functions.push_back([](const Base& obj) { obj.print(); });

    // 遍历并调用所有封装的函数
    for (const auto& f : functions) {
        f(d);  // 输出: Derived class
    }

    return 0;
}
```

**代码解析：**
- `Base`是一个抽象基类，`Derived`是其派生类。
- `Printer`是一个函数对象，它重载了`operator()`，接受一个`const Base&`参数并调用其`print`方法。
- `std::function<void(const Base&)>`封装了`Printer`对象。
- 使用`std::vector`存储多个`std::function`对象，遍历并调用它们。

## 5. `std::function`的原理

### 5.1 可调用对象的存储

`std::function`内部使用了一种称为“类型擦除”（Type Erasure）的技术来存储和调用可调用对象。类型擦除允许`std::function`在不了解具体类型的情况下，存储和调用不同类型的可调用对象。

### 5.2 动态分配与内存管理

`std::function`内部可能会使用动态内存分配来存储较大的可调用对象（如lambda表达式捕获的变量）。因此，`std::function`的构造和复制可能会涉及内存分配和释放，这可能会带来一定的性能开销。

### 5.3 调用机制

`std::function`通过内部的`vtable`（虚函数表）机制来调用封装的可调用对象。无论封装的是哪种可调用对象，`std::function`都能通过统一的接口进行调用。

## 6. 常见问题与注意事项

### 6.1 `std::function`的性能

由于`std::function`涉及类型擦除和可能的动态内存分配，它的性能可能不如直接调用可调用对象。因此，在性能敏感的场景中，应谨慎使用`std::function`。

### 6.2 `std::function`的空状态

在使用`std::function`之前，应检查其是否为空，以避免调用空函数导致的未定义行为。

### 6.3 `std::function`与`std::bind`

`std::function`经常与`std::bind`一起使用，用于绑定函数的参数。例如：

```cpp
#include <iostream>
#include <functional>

void printSum(int a, int b) {
    std::cout << "Sum: " << a + b << std::endl;
}

int main() {
    // 使用std::bind绑定函数参数
    std::function<void()> func = std::bind(printSum, 3, 4);

    // 调用封装的函数
    func();  // 输出: Sum: 7

    return 0;
}
```

## 7. 总结

`std::function`是C++中一个非常强大的工具，它提供了一种统一的方式来存储、复制和调用各种可调用对象。通过本文的讲解，你应该已经掌握了`std::function`的基本用法和原理，并能够在实际编程中灵活运用它。



文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)

