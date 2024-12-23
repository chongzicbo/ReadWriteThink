---
title: '现代C++的特性'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
---

现代C++特性列表如下

### C++11
- **自动类型推导 (`auto` 和 `decltype`)**: 允许编译器推导变量类型，简化代码编写。
- **lambda表达式**: 匿名函数，可以捕获作用域内的变量，方便编写内联函数。
- **智能指针 (`std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`)**: 管理动态内存，防止内存泄漏。
- **移动语义和右值引用**: 优化资源管理，通过移动而不是复制来提高性能。
- **线程支持库**: 提供多线程编程的支持，包括线程、互斥量等。
- **范围`for`循环**: 简化容器的遍历操作。
- **`constexpr`**: 允许在编译时进行计算，提高性能。
- **`std::initializer_list`**: 用于初始化容器和聚合类型。
- **`std::tuple` 和 `std::pair`**: 通用的元组和配对类型。
- **`std::function` 和 `std::bind`**: 类型擦除的函数对象和参数绑定。
- **`noexcept` 规定**: 指定函数不抛出异常，增强性能和安全性。
- **`override` 和 `final` 规定**: 增强继承的安全性。
- **`nullptr`**: 空指针常量，替代NULL。

### C++14
- **Lambda函数的返回类型推导**: 允许编译器推导lambda函数的返回类型。
- **泛型lambda**: 支持更多类型的参数和返回值。
- **`constexpr`扩展**: 允许在编译时执行更多计算。
- **`decltype(auto)`**: 自动推导返回类型，并保持其类型。

### C++17
- **结构化绑定**: 简化元组和结构体的访问。
- **`std::optional` 和 `std::variant`**: 提供更安全和灵活的容器类型。
- **折叠表达式**: 简化模板参数的展开。
- **内联变量**: 允许在头文件中定义内联变量。
- **`std::filesystem`**: 提供文件系统操作的支持。
- **`std::string_view`**: 轻量级字符串引用，提高字符串操作效率。

### C++20
- **概念（Concepts）**: 为模板编程提供更强大的约束机制。
- **模块（Modules）**: 提供更现代的模块系统，替代头文件。
- **协程（Coroutines）**: 支持异步编程，提高程序效率。
- **三向比较操作符（Spaceship Operator `<=>`）**: 提供统一的比较机制。
- **范围库（Ranges Library）**: 提供更强大的数据操作工具。
- **`std::format`**: 提供格式化输出功能，类似于Python的格式化字符串。

### C++23
- **模式匹配（Pattern Matching）**: 提供更灵活的数据匹配和处理机制。
- **`std::atomic_ref`**: 提供对原子操作的更灵活支持。
- **更多文件系统功能**: 进一步增强文件系统库的功能。

这些特性使得现代C++更加高效、安全和易用，满足了日益复杂的软件开发需求。



文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)