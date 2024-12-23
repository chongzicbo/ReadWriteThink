---
title: '深入理解C++函数指针'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
  - c++基础
---



# 深入理解C++函数指针

## 引言

函数指针是C++中强大且灵活的工具，允许我们将函数作为参数传递、存储在数据结构中，并动态调用。通过函数指针，我们可以实现回调机制、动态调用不同函数等功能，极大地增强了代码的灵活性和可扩展性。

本博客的目标是帮助读者全面理解函数指针的概念、语法和应用场景，并通过详细的代码示例加深理解。

## 一、函数指针基础

### 1. 什么是函数指针？

**函数指针**是指向函数的指针变量。与普通指针不同，函数指针指向的是函数的入口地址，而不是数据的地址。函数指针的类型由函数的返回类型和参数列表决定。

### 2. 声明函数指针

在C++中，声明函数指针的语法相对复杂，需要明确指定函数的返回类型和参数列表。

```cpp
// 声明一个指向返回类型为int，参数为两个int的函数的指针
int (*funcPtr)(int, int);
```

**代码解析：**
- `int (*funcPtr)(int, int);` 表示 `funcPtr` 是一个指向函数的指针，该函数返回 `int` 类型，并接受两个 `int` 类型的参数。
- 注意括号 `(*funcPtr)` 的重要性，它确保 `funcPtr` 是一个指针，而不是一个返回 `int*` 的函数。

### 3. 初始化函数指针

函数指针可以通过将函数名赋值给它来进行初始化。函数名本身就代表函数的地址。

```cpp
int add(int a, int b) { 
    return a + b; 
}

funcPtr = add;  // 将函数指针指向add函数
```

**代码解析：**
- `funcPtr = add;` 将 `funcPtr` 指向 `add` 函数。
- 函数名 `add` 实际上是函数的首地址，因此可以直接赋值给函数指针。

### 4. 调用函数指针

通过函数指针调用函数的方式与直接调用函数类似，只需使用指针变量即可。

```cpp
int result = funcPtr(3, 4);  // 通过函数指针调用add函数
std::cout << "Result: " << result << std::endl;  // 输出：Result: 7
```

**代码解析：**
- `funcPtr(3, 4)` 通过函数指针调用 `add` 函数，并传递参数 `3` 和 `4`。
- 与直接调用 `add(3, 4)` 相比，唯一的区别是调用方式是通过指针。

## 二、函数指针的高级应用

### 1. 函数指针作为函数参数

函数指针可以作为参数传递给另一个函数，这种机制常用于实现回调函数。

```cpp
void printResult(int (*func)(int, int), int a, int b) {
    std::cout << "Result: " << func(a, b) << std::endl;
}

printResult(add, 3, 4);  // 输出：Result: 7
```

**代码解析：**
- `printResult` 函数接受一个函数指针 `func` 作为参数，并调用该函数。
- `printResult(add, 3, 4)` 将 `add` 函数作为参数传递给 `printResult`，并在函数内部调用 `add` 函数。

### 2. 函数指针数组

函数指针数组可以存储多个函数指针，方便动态调用不同函数。

```cpp
int subtract(int a, int b) { 
    return a - b; 
}

int multiply(int a, int b) { 
    return a * b; 
}

int (*funcArray[])(int, int) = {add, subtract, multiply};

for (int i = 0; i < 3; i++) {
    std::cout << funcArray[i](6, 2) << std::endl;  // 输出：8, 4, 12
}
```

**代码解析：**
- `int (*funcArray[])(int, int) = {add, subtract, multiply};` 声明了一个函数指针数组 `funcArray`，其中存储了 `add`、`subtract` 和 `multiply` 三个函数的指针。
- `funcArray[i](6, 2)` 通过数组索引动态调用不同的函数。

### 3. 使用 `typedef` 简化函数指针声明

`typedef` 可以用来简化复杂的函数指针类型声明。

```cpp
typedef int (*FuncPtr)(int, int);

FuncPtr funcPtr = add;
printResult(funcPtr, 3, 4);  // 输出：Result: 7
```

**代码解析：**
- `typedef int (*FuncPtr)(int, int);` 定义了一个新的类型 `FuncPtr`，表示指向返回 `int` 类型、接受两个 `int` 类型参数的函数的指针。
- `FuncPtr funcPtr = add;` 使用 `FuncPtr` 类型声明 `funcPtr`，并将其指向 `add` 函数。

## 三、函数指针与C++11及更高版本

### 1. 使用 `std::function` 替代函数指针

C++11引入了 `std::function`，它是一个通用的函数包装器，可以存储、复制和调用任何可调用目标（函数、lambda表达式、bind表达式等）。

```cpp
#include <functional>

std::function<int(int, int)> func = add;
std::cout << func(3, 4) << std::endl;  // 输出：7
```

**代码解析：**
- `std::function<int(int, int)> func = add;` 使用 `std::function` 包装 `add` 函数。
- `func(3, 4)` 调用 `add` 函数，与使用函数指针类似，但 `std::function` 更加灵活和易用。

### 2. 使用lambda表达式

C++11引入了lambda表达式，可以方便地定义和使用匿名函数。

```cpp
std::function<int(int, int)> func = [](int a, int b) { return a + b; };
std::cout << func(3, 4) << std::endl;  // 输出：7
```

**代码解析：**
- `[](int a, int b) { return a + b; }` 是一个lambda表达式，定义了一个匿名函数。
- `std::function<int(int, int)> func = ...;` 将lambda表达式赋值给 `std::function` 对象。

## 四、总结与展望

函数指针是C++中非常强大的工具，能够实现回调机制、动态调用函数等功能。通过本博客的学习，读者应该对函数指针的概念、语法和应用场景有了深入的理解。

尽管随着C++标准的不断发展，函数指针的使用可能会逐渐减少，但理解其原理对于深入理解C++语言仍然至关重要。未来，随着C++11及更高版本中 `std::function` 和lambda表达式的引入，函数指针的使用可能会变得更加简洁和灵活。

文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)