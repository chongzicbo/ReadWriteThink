---
    title: 'std::visit详解'
    categories:
      - [开发,cpp]
    tags:
      - c++
      - cpp
---



`std::visit` 是 C++17 引入的一个非常强大的工具，用于访问 `std::variant` 中存储的值。它的设计目的是为了简化对 `std::variant` 中不同类型值的处理，同时保持类型安全。下面我们将详细解释 `std::visit` 的原理、用法以及一些高级技巧。

---

## 一、`std::visit` 的基本概念

### 1.1 什么是 `std::visit`？

`std::visit` 是一个函数模板，用于对 `std::variant` 中存储的值进行操作。它的核心功能是根据 `std::variant` 中当前存储的类型，动态地调用相应的处理函数。

`std::visit` 的定义大致如下：

```cpp
template <class Visitor, class... Variants>
constexpr decltype(auto) visit(Visitor&& vis, Variants&&... vars);
```

- **`Visitor`**：一个可调用对象（如函数、lambda 表达式、函数对象等），用于处理 `std::variant` 中的值。
- **`Variants`**：一个或多个 `std::variant` 对象。

### 1.2 `std::visit` 的作用

`std::visit` 的主要作用是根据 `std::variant` 中存储的实际类型，动态地调用相应的处理逻辑。它避免了手动编写大量的 `if-else` 或 `switch` 语句来处理不同类型的情况，从而使代码更加简洁和类型安全。

---

## 二、`std::visit` 的基本用法

### 2.1 访问单个 `std::variant`

我们先来看一个简单的例子，展示如何使用 `std::visit` 访问单个 `std::variant`。

```cpp
#include <iostream>
#include <variant>
#include <string>

int main() {
    // 定义一个 variant，可以存储 int, float, std::string
    std::variant<int, float, std::string> var = 42;

    // 使用 std::visit 访问 variant 中的值
    std::visit([](auto&& arg) {
        using T = std::decay_t<decltype(arg)>;
        if constexpr (std::is_same_v<T, int>) {
            std::cout << "Integer: " << arg << std::endl;
        } else if constexpr (std::is_same_v<T, float>) {
            std::cout << "Float: " << arg << std::endl;
        } else if constexpr (std::is_same_v<T, std::string>) {
            std::cout << "String: " << arg << std::endl;
        }
    }, var);

    return 0;
}
```

#### 代码解析：

1. **`std::visit` 的参数**：
   - 第一个参数是一个 lambda 表达式，用于处理 `std::variant` 中的值。
   - 第二个参数是 `std::variant` 对象 `var`。

2. **Lambda 表达式**：
   - `auto&& arg`：表示 `std::variant` 中存储的值。
   - `using T = std::decay_t<decltype(arg)>`：获取 `arg` 的实际类型（去除引用和 cv 限定符）。
   - `if constexpr`：编译时判断 `arg` 的类型，并执行相应的逻辑。

3. **输出结果**：
   - 如果 `var` 中存储的是 `int`，输出 `"Integer: 42"`。
   - 如果 `var` 中存储的是 `float`，输出 `"Float: 3.14"`。
   - 如果 `var` 中存储的是 `std::string`，输出 `"String: Hello"`。

---

### 2.2 访问多个 `std::variant`

`std::visit` 不仅可以访问单个 `std::variant`，还可以同时访问多个 `std::variant`。它的行为类似于元组解包，会根据所有 `std::variant` 的当前类型组合，调用相应的处理逻辑。

```cpp
#include <iostream>
#include <variant>
#include <string>

int main() {
    // 定义两个 variant，分别存储 int 和 std::string
    std::variant<int, std::string> var1 = 42;
    std::variant<int, std::string> var2 = "Hello";

    // 使用 std::visit 访问两个 variant 中的值
    std::visit([](auto&& arg1, auto&& arg2) {
        using T1 = std::decay_t<decltype(arg1)>;
        using T2 = std::decay_t<decltype(arg2)>;

        if constexpr (std::is_same_v<T1, int> && std::is_same_v<T2, std::string>) {
            std::cout << "Integer and String: " << arg1 << ", " << arg2 << std::endl;
        } else if constexpr (std::is_same_v<T1, std::string> && std::is_same_v<T2, int>) {
            std::cout << "String and Integer: " << arg1 << ", " << arg2 << std::endl;
        } else {
            std::cout << "Other combination: " << arg1 << ", " << arg2 << std::endl;
        }
    }, var1, var2);

    return 0;
}
```

#### 代码解析：

1. **访问多个 `std::variant`**：
   - `std::visit` 的第二个参数是两个 `std::variant` 对象 `var1` 和 `var2`。
   - Lambda 表达式接受两个参数 `arg1` 和 `arg2`，分别对应 `var1` 和 `var2` 中的值。

2. **类型判断**：
   - 使用 `if constexpr` 判断 `arg1` 和 `arg2` 的类型组合。
   - 例如，如果 `var1` 是 `int`，`var2` 是 `std::string`，则输出 `"Integer and String: 42, Hello"`。

3. **输出结果**：
   - 根据 `var1` 和 `var2` 的类型组合，输出不同的结果。

---

## 三、`std::visit` 的原理

### 3.1 动态分派

`std::visit` 的核心原理是**动态分派**（Dynamic Dispatch）。它会根据 `std::variant` 中存储的实际类型，动态地调用相应的处理函数。

例如，假设我们有一个 `std::variant<int, float, std::string>`，它的类型标签（index）会跟踪当前存储的类型。当调用 `std::visit` 时，它会根据类型标签找到对应的处理逻辑。

### 3.2 类型安全的分派

`std::visit` 的一个重要特性是**类型安全**。它确保了在编译时检查所有可能的类型组合，避免了手动编写 `if-else` 或 `switch` 语句时可能出现的遗漏或错误。

例如，如果我们忘记处理某个类型，编译器会报错，提示我们缺少相应的处理逻辑。

---

## 四、`std::visit` 的高级用法

### 4.1 使用 `std::variant` 的索引

除了使用 `std::visit`，我们还可以通过 `std::variant` 的索引来判断当前存储的类型。

```cpp
#include <iostream>
#include <variant>
#include <string>

int main() {
    std::variant<int, float, std::string> var = 42;

    // 获取当前存储的类型索引
    switch (var.index()) {
        case 0:  // int
            std::cout << "Integer: " << std::get<int>(var) << std::endl;
            break;
        case 1:  // float
            std::cout << "Float: " << std::get<float>(var) << std::endl;
            break;
        case 2:  // std::string
            std::cout << "String: " << std::get<std::string>(var) << std::endl;
            break;
    }

    return 0;
}
```

#### 代码解析：

1. **`var.index()`**：获取 `std::variant` 中当前存储的类型索引。
2. **`std::get<T>(var)`**：根据类型 `T` 获取 `std::variant` 中的值。

### 4.2 使用 `std::monostate` 处理空值

有时我们需要表示 `std::variant` 中没有存储任何值的情况。可以使用 `std::monostate` 来表示空值。

```cpp
#include <iostream>
#include <variant>
#include <string>

int main() {
    std::variant<std::monostate, int, std::string> var = std::monostate{};

    if (var.index() == 0) {
        std::cout << "Variant is empty." << std::endl;
    } else {
        std::visit([](auto&& arg) {
            std::cout << "Value: " << arg << std::endl;
        }, var);
    }

    return 0;
}
```

#### 代码解析：

1. **`std::monostate`**：表示 `std::variant` 中没有存储任何值。
2. **`var.index() == 0`**：判断 `std::variant` 是否为空。

---

## 五、总结

`std::visit` 是 C++17 中用于访问 `std::variant` 的强大工具，它通过动态分派和类型安全的特性，简化了多态类型的处理。通过 `std::visit`，我们可以避免手动编写大量的 `if-else` 或 `switch` 语句，使代码更加简洁和易于维护。

- **核心功能**：根据 `std::variant` 中存储的实际类型，动态调用相应的处理逻辑。
- **类型安全**：在编译时检查所有可能的类型组合，避免运行时错误。
- **灵活性**：支持访问单个或多个 `std::variant`，适用于复杂的多态场景。

文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)