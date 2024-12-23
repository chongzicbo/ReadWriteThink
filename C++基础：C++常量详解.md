---
title: 'C++ 常量详解'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
  - c++基础
---



# C++ 常量详解

## 一、引言

在 C++ 编程中，常量是不可或缺的一部分。常量是指在程序运行期间其值不能被改变的量。使用常量可以提高代码的可读性、可维护性和安全性。通过定义常量，我们可以避免在代码中直接使用硬编码的数值或字符串，从而减少错误并使代码更易于理解。

## 二、常量的定义

### 常量的概念

常量是指在程序运行期间其值不能被改变的量。与变量不同，常量的值一旦被定义，就不能在程序的其他部分被修改。

### 常量的类型

常量可以分为两大类：字面常量和符号常量。

#### 1. 字面常量 (Literal Constants)

字面常量是指直接写在代码中的常量值。它们没有名称，直接表示一个固定的值。

- **整型常量 (Integer Constants)**：表示整数值，可以是十进制、八进制或十六进制。
- **浮点型常量 (Floating-point Constants)**：表示浮点数值，可以是小数形式或指数形式。
- **字符常量 (Character Constants)**：表示单个字符，用单引号括起来。
- **字符串常量 (String Constants)**：表示字符序列，用双引号括起来。

#### 2. 符号常量 (Symbolic Constants)

符号常量是指通过某种方式定义的具有名称的常量。它们有名称，可以在代码中多次使用。

- **使用 `#define` 预处理器指令定义的常量**：通过预处理器指令 `#define` 定义的常量。
- **使用 `const` 关键字定义的常量**：通过 `const` 关键字定义的常量，具有类型检查。
- **使用 `constexpr` 关键字定义的常量**：通过 `constexpr` 关键字定义的常量，要求在编译时求值。

## 三、常量的使用

### 1. 字面常量的使用

#### 整型常量的表示方法

整型常量可以用十进制、八进制或十六进制表示。

```cpp
#include <iostream>

int main() {
    // 十进制整型常量
    int decimal = 10;
    
    // 八进制整型常量（以0开头）
    int octal = 012; // 相当于十进制的10
    
    // 十六进制整型常量（以0x或0X开头）
    int hexadecimal = 0xA; // 相当于十进制的10

    std::cout << "Decimal: " << decimal << std::endl;
    std::cout << "Octal: " << octal << std::endl;
    std::cout << "Hexadecimal: " << hexadecimal << std::endl;

    return 0;
}
```

#### 浮点型常量的表示方法

浮点型常量可以用小数形式或指数形式表示。

```cpp
#include <iostream>

int main() {
    // 小数形式的浮点型常量
    double pi = 3.14159;
    
    // 指数形式的浮点型常量
    double e = 2.71828e0; // 相当于2.71828

    std::cout << "Pi: " << pi << std::endl;
    std::cout << "e: " << e << std::endl;

    return 0;
}
```

#### 字符常量的表示方法

字符常量用单引号括起来。

```cpp
#include <iostream>

int main() {
    // 字符常量
    char ch = 'A';

    std::cout << "Character: " << ch << std::endl;

    return 0;
}
```

#### 字符串常量的表示方法

字符串常量用双引号括起来。

```cpp
#include <iostream>

int main() {
    // 字符串常量
    std::string str = "Hello, World!";

    std::cout << "String: " << str << std::endl;

    return 0;
}
```

### 2. 符号常量的使用

#### 使用 `#define` 预处理器指令定义的常量

`#define` 是预处理器指令，用于定义符号常量。它的优点是简单易用，缺点是没有类型检查。

```cpp
#include <iostream>

// 使用 #define 定义常量
#define PI 3.14159

int main() {
    std::cout << "PI: " << PI << std::endl;

    return 0;
}
```

#### 使用 `const` 关键字定义的常量

`const` 关键字用于定义常量，具有类型检查，推荐在现代 C++ 中使用。

```cpp
#include <iostream>

int main() {
    // 使用 const 定义常量
    const double PI = 3.14159;

    std::cout << "PI: " << PI << std::endl;

    return 0;
}
```

#### 使用 `constexpr` 关键字定义的常量

`constexpr` 关键字用于定义编译时常量，要求在编译时求值。

```cpp
#include <iostream>

int main() {
    // 使用 constexpr 定义常量
    constexpr double PI = 3.14159;

    std::cout << "PI: " << PI << std::endl;

    return 0;
}
```

### 3. 常量的命名规范

常量的命名应遵循一定的规范，以提高代码的可读性。

- **使用全大写字母和下划线命名常量**：例如 `MAX_VALUE`。
- **避免使用保留字和关键字作为常量名**：例如不要使用 `int` 或 `const` 作为常量名。

## 四、常量的应用场景

### 1. 定义程序中不变的值

常量常用于定义程序中不变的值，例如数学常数、程序配置参数和错误代码。

```cpp
#include <iostream>

// 定义数学常数
constexpr double PI = 3.14159;
constexpr double E = 2.71828;

// 定义程序配置参数
const int MAX_CONNECTIONS = 100;

// 定义错误代码
const int ERROR_CODE_FILE_NOT_FOUND = 404;

int main() {
    std::cout << "PI: " << PI << std::endl;
    std::cout << "E: " << E << std::endl;
    std::cout << "Max Connections: " << MAX_CONNECTIONS << std::endl;
    std::cout << "Error Code: " << ERROR_CODE_FILE_NOT_FOUND << std::endl;

    return 0;
}
```

### 2. 提高代码的可读性和可维护性

使用常量可以避免在代码中直接使用魔法数字，从而提高代码的可读性和可维护性。

```cpp
#include <iostream>

// 使用常量代替魔法数字
const int DAYS_IN_WEEK = 7;

int main() {
    std::cout << "Days in a week: " << DAYS_IN_WEEK << std::endl;

    return 0;
}
```

### 3. 提高程序的安全性

使用 `const` 关键字可以防止变量被意外修改，从而提高程序的安全性。

```cpp
#include <iostream>

void printValue(const int value) {
    // value = 10; // 错误：不能修改 const 变量
    std::cout << "Value: " << value << std::endl;
}

int main() {
    const int value = 42;
    printValue(value);

    return 0;
}
```

## 五、常量的注意事项

### 1. `#define` 预处理器指令定义的常量没有类型检查

```cpp
#include <iostream>

#define VALUE 42

int main() {
    double value = VALUE; // 没有类型检查，可能会导致错误
    std::cout << "Value: " << value << std::endl;

    return 0;
}
```

### 

### 2. `constexpr` 关键字定义的常量必须能够在编译时求值

```cpp
#include <iostream>

constexpr int getValue() {
    return 42; // 必须在编译时求值
}

int main() {
    constexpr int value = getValue();
    std::cout << "Value: " << value << std::endl;

    return 0;
}
```

### 3. 避免过度使用常量，影响代码的可读性

```cpp
#include <iostream>

const int A = 1;
const int B = 2;
const int C = 3;

int main() {
    // 过度使用常量可能会影响代码的可读性
    int result = A + B + C;
    std::cout << "Result: " << result << std::endl;

    return 0;
}
```

## 六、总结

本文详细介绍了 C++ 中常量的定义、使用和应用场景。常量在提高代码质量方面起着重要作用，能够提高代码的可读性、可维护性和安全性。通过合理使用常量，我们可以避免硬编码的数值和字符串，从而使代码更加清晰和易于维护。

## 七、参考资料

- [C++ Primer](https://www.amazon.com/C-Primer-5th-Stanley-Lippman/dp/0321714113)
- [cppreference.com](https://en.cppreference.com/w/)

## 八、附录

### 常见常量示例代码

```cpp
#include <iostream>

// 使用 #define 定义常量
#define MAX_VALUE 100

// 使用 const 定义常量
const int MIN_VALUE = 0;

// 使用 constexpr 定义常量
constexpr double PI = 3.14159;

int main() {
    std::cout << "Max Value: " << MAX_VALUE << std::endl;
    std::cout << "Min Value: " << MIN_VALUE << std::endl;
    std::cout << "PI: " << PI << std::endl;

    return 0;
}
```

### 常量相关的常见问题解答

1. **问：`const` 和 `constexpr` 有什么区别？**
   - 答：`const` 用于定义运行时常量，而 `constexpr` 用于定义编译时常量，要求在编译时求值。

2. **问：为什么推荐使用 `const` 而不是 `#define`？**
   - 答：`const` 具有类型检查，更安全且更易于调试，而 `#define` 没有类型检查，容易出错。

3. **问：`constexpr` 函数有什么要求？**
   - 答：`constexpr` 函数必须在编译时求值，且不能包含复杂的逻辑或运行时才能确定的值。



文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)

