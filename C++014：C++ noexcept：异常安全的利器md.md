---
title: 'C++ noexcept：异常安全的利器'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
---



# C++ noexcept：异常安全的利器

在C++编程中，异常处理是一个非常重要的主题。C++11引入了一个新的关键字 `noexcept`，它可以帮助我们更好地控制代码的异常安全性。本文将带你从基础到深入，逐步理解 `noexcept` 的作用、原理以及如何使用它来提升代码的健壮性。

## 1. 什么是 `noexcept`？

`noexcept` 是C++11引入的一个关键字，用于声明一个函数是否可能抛出异常。它的主要作用是告诉编译器和程序员，某个函数不会抛出异常，从而允许编译器进行更激进的优化，并且帮助我们编写更安全的代码。

### 1.1 `noexcept` 的基本语法

`noexcept` 有两种形式：

1. **`noexcept`**：表示函数不会抛出任何异常。
2. **`noexcept(bool)`**：表示函数是否抛出异常取决于括号中的表达式。如果表达式为 `true`，则函数不会抛出异常；如果为 `false`，则函数可能抛出异常。

```cpp
void func1() noexcept;          // func1 不会抛出异常
void func2() noexcept(true);    // func2 不会抛出异常
void func3() noexcept(false);   // func3 可能抛出异常
```

### 1.2 `noexcept` 的作用

`noexcept` 的主要作用是：

- **优化代码**：编译器知道函数不会抛出异常后，可以进行更激进的优化，例如消除异常处理的开销。
- **提高代码安全性**：通过明确声明函数是否抛出异常，可以帮助程序员更好地理解代码的行为，避免潜在的异常处理问题。

## 2. `noexcept` 的基础使用

我们先来看一个简单的例子，理解 `noexcept` 的基本用法。

```cpp
#include <iostream>

void may_throw() {
    throw std::runtime_error("An error occurred");
}

void no_throw() noexcept {
    // 这个函数不会抛出异常
    std::cout << "This function does not throw." << std::endl;
}

int main() {
    try {
        may_throw();
    } catch (const std::exception& e) {
        std::cout << "Caught exception: " << e.what() << std::endl;
    }

    no_throw();

    return 0;
}
```

### 2.1 代码解析

- `may_throw()` 函数会抛出一个 `std::runtime_error` 异常。
- `no_throw()` 函数使用 `noexcept` 声明，表示它不会抛出任何异常。
- 在 `main()` 函数中，我们尝试捕获 `may_throw()` 抛出的异常，并调用 `no_throw()` 函数。

### 2.2 运行结果

```
Caught exception: An error occurred
This function does not throw.
```

### 2.3 为什么使用 `noexcept`？

在这个例子中，`no_throw()` 函数明确声明了它不会抛出异常。这不仅帮助我们理解代码的行为，还允许编译器进行优化。如果 `no_throw()` 函数意外抛出了异常，程序会立即调用 `std::terminate()` 终止运行，而不是进入异常处理流程。

## 3. `noexcept` 的进阶使用

接下来，我们来看一个更复杂的例子，涉及到移动构造函数和移动赋值运算符。

### 3.1 移动语义与 `noexcept`

在C++11中，移动语义是一个非常重要的特性。移动构造函数和移动赋值运算符通常被声明为 `noexcept`，因为它们通常不会抛出异常。这使得标准库容器（如 `std::vector`）在重新分配内存时能够安全地使用移动操作，而不是复制操作。

```cpp
#include <iostream>
#include <vector>

class MyClass {
public:
    MyClass() {
        std::cout << "Default constructor" << std::endl;
    }

    // 移动构造函数，声明为 noexcept
    MyClass(MyClass&& other) noexcept {
        std::cout << "Move constructor" << std::endl;
    }

    // 移动赋值运算符，声明为 noexcept
    MyClass& operator=(MyClass&& other) noexcept {
        std::cout << "Move assignment operator" << std::endl;
        return *this;
    }

    // 复制构造函数
    MyClass(const MyClass& other) {
        std::cout << "Copy constructor" << std::endl;
    }

    // 复制赋值运算符
    MyClass& operator=(const MyClass& other) {
        std::cout << "Copy assignment operator" << std::endl;
        return *this;
    }

    ~MyClass() {
        std::cout << "Destructor" << std::endl;
    }
};

int main() {
    std::vector<MyClass> vec;

    // 预留足够的空间，避免多次重新分配
    vec.reserve(10);

    std::cout << "Pushing first element..." << std::endl;
    vec.push_back(MyClass());

    std::cout << "Pushing second element..." << std::endl;
    vec.push_back(MyClass());

    return 0;
}
```

### 3.2 代码解析

- `MyClass` 类定义了移动构造函数和移动赋值运算符，并且都声明为 `noexcept`。
- 在 `main()` 函数中，我们创建了一个 `std::vector<MyClass>`，并尝试向其中添加两个元素。
- 由于 `MyClass` 的移动构造函数和移动赋值运算符都被声明为 `noexcept`，`std::vector` 在重新分配内存时会优先使用移动操作，而不是复制操作。

### 3.3 运行结果

```
Pushing first element...
Default constructor
Move constructor
Destructor
Pushing second element...
Default constructor
Move constructor
Destructor
Destructor
Destructor
```

### 3.4 为什么移动操作要声明为 `noexcept`？

在这个例子中，`std::vector` 在添加元素时，如果当前容量不足，会重新分配内存并将现有元素移动到新的内存位置。由于 `MyClass` 的移动构造函数和移动赋值运算符都被声明为 `noexcept`，`std::vector` 可以安全地使用移动操作，而不必担心异常的发生。

如果移动操作没有声明为 `noexcept`，`std::vector` 将不得不使用复制操作，这会导致性能下降。

## 4. `noexcept` 的原理与实现

### 4.1 `noexcept` 的编译器行为

当一个函数被声明为 `noexcept` 时，编译器会假设这个函数不会抛出异常。如果函数内部确实抛出了异常，程序会立即调用 `std::terminate()` 终止运行，而不是进入异常处理流程。

### 4.2 `noexcept` 的表达式

`noexcept` 还可以接受一个布尔表达式，用于动态判断函数是否可能抛出异常。例如：

```cpp
void func() noexcept(sizeof(int) == 4);
```

在这个例子中，`func()` 是否声明为 `noexcept` 取决于 `sizeof(int) == 4` 的结果。如果 `int` 的大小是4字节，则 `func()` 不会抛出异常；否则，`func()` 可能抛出异常。

### 4.3 `noexcept` 与析构函数

在C++中，类的析构函数默认是 `noexcept(true)`，即不会抛出异常。这是因为如果析构函数抛出异常，程序将无法安全地处理，可能导致未定义行为。

```cpp
class MyClass {
public:
    ~MyClass() noexcept {
        // 析构函数默认是 noexcept
    }
};
```

## 5. `noexcept` 的常见应用场景

### 5.1 标准库中的 `noexcept`

C++标准库中的许多函数和操作符都使用了 `noexcept`。例如，`std::swap` 在C++11中被声明为 `noexcept`，前提是交换的两个对象的移动操作都是 `noexcept`。

```cpp
template <typename T>
void swap(T& a, T& b) noexcept(noexcept(T(std::move(a))) && noexcept(a.operator=(std::move(b))));
```

### 5.2 自定义容器中的 `noexcept`

在自定义容器类中，移动构造函数和移动赋值运算符通常被声明为 `noexcept`，以确保容器在重新分配内存时能够安全地使用移动操作。

```cpp
class MyContainer {
public:
    MyContainer(MyContainer&& other) noexcept {
        // 移动构造函数
    }

    MyContainer& operator=(MyContainer&& other) noexcept {
        // 移动赋值运算符
        return *this;
    }
};
```

## 6. 总结

`noexcept` 是C++11引入的一个重要特性，它帮助我们更好地控制函数的异常行为，提升代码的性能和安全性。通过明确声明函数是否抛出异常，我们可以让编译器进行更激进的优化，并且避免潜在的异常处理问题。

在实际编程中，`noexcept` 尤其在移动语义和标准库容器中扮演着重要角色。合理使用 `noexcept`，可以让我们的代码更加高效、健壮。



文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)