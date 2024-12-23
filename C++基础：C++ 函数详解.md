---
title: 'C++ 函数详解'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
  - c++基础
---



# C++ 函数详解：从基础到进阶

## 一、引言

C++ 作为一种强大的编程语言，广泛应用于系统编程、游戏开发、嵌入式系统等领域。其高效的性能和灵活的语法使其成为许多开发者的首选语言。在 C++ 编程中，函数是构建程序的基本单元，它们不仅提供了代码复用的机制，还帮助我们将程序模块化，提高代码的可读性和可维护性。

本文将深入探讨 C++ 函数的各个方面，从基础的函数定义、声明、调用，到进阶的函数重载、默认参数、内联函数、递归函数以及变参函数，最后结合面向对象编程中的成员函数、静态函数和友元函数，帮助读者全面理解和掌握 C++ 函数的使用。

## 二、C++ 函数基础

### 2.1 什么是函数？

函数是一段可重复使用的代码块，用于执行特定的任务。通过将代码封装在函数中，我们可以避免重复编写相同的代码，提高代码的复用性。此外，函数还可以将程序分解为多个模块，使得代码结构更加清晰，便于维护和调试。

函数的优势包括：
- **代码复用**：相同的代码只需编写一次，多次调用。
- **模块化**：将程序分解为多个功能模块，便于管理和扩展。
- **可读性**：通过函数名和注释，可以更清晰地表达代码的意图。
- **可维护性**：修改函数时，只需修改一处代码，所有调用该函数的地方都会自动更新。

### 2.2 函数的组成部分

一个函数通常由以下几个部分组成：
- **返回类型**：函数执行完毕后返回的数据类型。如果函数不返回任何值，则使用 `void`。
- **函数名**：函数的名称，用于调用函数。
- **参数列表**：函数接受的输入参数，可以为空。
- **函数体**：函数执行的具体代码，包含在一对大括号 `{}` 中。

下面是一个简单的函数示例：

```cpp
#include <iostream>

// 函数定义
int add(int a, int b) {
    return a + b;  // 返回两个整数的和
}

int main() {
    int result = add(3, 5);  // 调用函数 add，传入参数 3 和 5
    std::cout << "3 + 5 = " << result << std::endl;  // 输出结果
    return 0;
}
```

**代码解析：**
- `int add(int a, int b)`：定义了一个名为 `add` 的函数，接受两个 `int` 类型的参数 `a` 和 `b`，并返回它们的和。
- `return a + b;`：函数体中的代码计算 `a` 和 `b` 的和，并返回结果。
- `int result = add(3, 5);`：在 `main` 函数中调用 `add` 函数，传入参数 `3` 和 `5`，并将返回值赋给 `result`。

### 2.3 函数的声明与定义

在 C++ 中，函数的声明和定义是两个不同的概念：
- **函数声明**：告诉编译器函数的存在，但不提供具体的实现。通常用于在函数定义之前使用函数。
- **函数定义**：提供函数的具体实现，包含函数体。

**函数声明和定义的区别：**
- 函数声明只需要提供函数的返回类型、函数名和参数列表，不需要函数体。
- 函数定义则需要完整的函数体，用于实现函数的功能。

**代码示例：**

```cpp
#include <iostream>

// 函数声明
int multiply(int a, int b);

int main() {
    int result = multiply(4, 6);  // 调用函数 multiply
    std::cout << "4 * 6 = " << result << std::endl;
    return 0;
}

// 函数定义
int multiply(int a, int b) {
    return a * b;  // 返回两个整数的乘积
}
```

**代码解析：**
- `int multiply(int a, int b);`：函数声明，告诉编译器 `multiply` 函数的存在。
- `int result = multiply(4, 6);`：在 `main` 函数中调用 `multiply` 函数。
- `int multiply(int a, int b) { return a * b; }`：函数定义，提供 `multiply` 函数的具体实现。

### 2.4 函数的调用

函数的调用是通过函数名和参数列表来实现的。C++ 支持三种参数传递方式：
- **值传递**：将参数的值复制一份传递给函数，函数内部对参数的修改不会影响原始值。
- **引用传递**：将参数的引用传递给函数，函数内部对参数的修改会影响原始值。
- **指针传递**：将参数的地址传递给函数，函数内部通过指针访问和修改原始值。

**代码示例：**

```cpp
#include <iostream>

// 值传递
void incrementByValue(int a) {
    a++;  // 修改参数的值
    std::cout << "Inside incrementByValue: " << a << std::endl;
}

// 引用传递
void incrementByReference(int& a) {
    a++;  // 修改参数的值
    std::cout << "Inside incrementByReference: " << a << std::endl;
}

// 指针传递
void incrementByPointer(int* a) {
    (*a)++;  // 修改参数的值
    std::cout << "Inside incrementByPointer: " << *a << std::endl;
}

int main() {
    int num = 10;

    incrementByValue(num);  // 值传递
    std::cout << "After incrementByValue: " << num << std::endl;

    incrementByReference(num);  // 引用传递
    std::cout << "After incrementByReference: " << num << std::endl;

    incrementByPointer(&num);  // 指针传递
    std::cout << "After incrementByPointer: " << num << std::endl;

    return 0;
}
```

**代码解析：**
- `incrementByValue(int a)`：值传递，函数内部对 `a` 的修改不会影响 `num`。
- `incrementByReference(int& a)`：引用传递，函数内部对 `a` 的修改会影响 `num`。
- `incrementByPointer(int* a)`：指针传递，函数内部通过指针修改 `num` 的值。

## 三、C++ 函数进阶

### 3.1 函数重载

函数重载是指在同一个作用域内，定义多个同名函数，但它们的参数列表不同。通过函数重载，我们可以使用相同的函数名处理不同类型的数据，提高代码的可读性。

**注意事项：**
- 返回类型不能作为函数重载的依据，只有参数列表的不同才能区分重载函数。

**代码示例：**

```cpp
#include <iostream>

// 函数重载：两个整数相加
int add(int a, int b) {
    return a + b;
}

// 函数重载：三个整数相加
int add(int a, int b, int c) {
    return a + b + c;
}

// 函数重载：两个浮点数相加
double add(double a, double b) {
    return a + b;
}

int main() {
    std::cout << "add(1, 2) = " << add(1, 2) << std::endl;
    std::cout << "add(1, 2, 3) = " << add(1, 2, 3) << std::endl;
    std::cout << "add(1.5, 2.5) = " << add(1.5, 2.5) << std::endl;
    return 0;
}
```

**代码解析：**
- `int add(int a, int b)`：接受两个整数参数，返回它们的和。
- `int add(int a, int b, int c)`：接受三个整数参数，返回它们的和。
- `double add(double a, double b)`：接受两个浮点数参数，返回它们的和。

### 3.2 默认参数

默认参数是指在函数声明或定义时为参数指定默认值。调用函数时，如果没有提供该参数的值，则使用默认值。

**注意事项：**
- 默认参数必须从右向左依次指定，不能跳过中间的参数。

**代码示例：**

```cpp
#include <iostream>

// 函数定义：带有默认参数
void greet(std::string name = "Guest") {
    std::cout << "Hello, " << name << "!" << std::endl;
}

int main() {
    greet();  // 使用默认参数 "Guest"
    greet("Alice");  // 传入参数 "Alice"
    return 0;
}
```

**代码解析：**
- `void greet(std::string name = "Guest")`：定义了一个带有默认参数的函数，默认参数为 `"Guest"`。
- `greet()`：调用函数时未传入参数，使用默认参数 `"Guest"`。
- `greet("Alice")`：调用函数时传入参数 `"Alice"`，覆盖默认参数。

### 3.3 内联函数

内联函数是 C++ 中的一种优化机制，编译器在编译时将函数调用替换为函数体代码，从而减少函数调用的开销，提高程序的执行效率。

**适用场景：**
- 函数体较小且频繁调用的函数。

**代码示例：**

```cpp
#include <iostream>

// 内联函数
inline int square(int x) {
    return x * x;
}

int main() {
    int result = square(5);  // 调用内联函数
    std::cout << "square(5) = " << result << std::endl;
    return 0;
}
```

**代码解析：**
- `inline int square(int x)`：定义了一个内联函数 `square`，返回 `x` 的平方。
- `int result = square(5)`：调用内联函数 `square`，编译器会将 `square(5)` 替换为 `5 * 5`。

### 3.4 递归函数

递归函数是指函数直接或间接调用自身。递归函数通常用于解决具有递归结构的问题，例如计算阶乘、斐波那契数列等。

**注意事项：**
- 递归函数必须设置递归终止条件，否则会导致无限递归，最终导致栈溢出。

**代码示例：**

```cpp
#include <iostream>

// 递归函数：计算阶乘
int factorial(int n) {
    if (n == 0 || n == 1) {
        return 1;  // 递归终止条件
    }
    return n * factorial(n - 1);  // 递归调用
}

int main() {
    int result = factorial(5);  // 计算 5 的阶乘
    std::cout << "5! = " << result << std::endl;
    return 0;
}
```

**代码解析：**
- `int factorial(int n)`：定义了一个递归函数 `factorial`，计算 `n` 的阶乘。
- `if (n == 0 || n == 1) { return 1; }`：递归终止条件，当 `n` 为 `0` 或 `1` 时，返回 `1`。
- `return n * factorial(n - 1)`：递归调用 `factorial(n - 1)`，并将结果乘以 `n`。

### 3.5 变参函数

变参函数是指可以接受可变数量参数的函数。C++ 提供了多种实现变参函数的方式，包括 `std::initializer_list` 和可变参数模板。

#### 3.5.1 使用 `std::initializer_list` 实现变参函数

`std::initializer_list` 是一种轻量级的变参函数实现方式，适用于参数类型相同的情况。

**代码示例：**

```cpp
#include <iostream>
#include <initializer_list>

// 使用 std::initializer_list 实现变参函数
void printNumbers(std::initializer_list<int> numbers) {
    for (int num : numbers) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
}

int main() {
    printNumbers({1, 2, 3, 4, 5});  // 调用变参函数
    return 0;
}
```

**代码解析：**
- `void printNumbers(std::initializer_list<int> numbers)`：定义了一个变参函数 `printNumbers`，接受一个 `std::initializer_list<int>` 类型的参数。
- `for (int num : numbers)`：遍历 `numbers` 中的每个元素，并输出。

#### 3.5.2 使用可变参数模板实现变参函数

可变参数模板是一种更强大的变参函数实现方式，适用于参数类型不同的情况。

**代码示例：**

```cpp
#include <iostream>

// 使用可变参数模板实现变参函数
void printArgs() {
    std::cout << std::endl;
}

template <typename T, typename... Args>
void printArgs(T first, Args... rest) {
    std::cout << first << " ";
    printArgs(rest...);  // 递归调用
}

int main() {
    printArgs(1, 2.5, "Hello", true);  // 调用变参函数
    return 0;
}
```

**代码解析：**
- `void printArgs()`：递归终止条件，当没有参数时，输出换行符。
- `template <typename T, typename... Args> void printArgs(T first, Args... rest)`：定义了一个可变参数模板函数 `printArgs`，接受任意数量的参数。
- `std::cout << first << " ";`：输出第一个参数。
- `printArgs(rest...)`：递归调用 `printArgs`，处理剩余的参数。

## 四、C++ 函数与面向对象编程

### 4.1 成员函数

成员函数是定义在类内部的函数，用于操作类的成员变量。成员函数可以通过类的对象调用，并且可以访问类的私有成员。

**代码示例：**

```cpp
#include <iostream>

class Rectangle {
private:
    int width, height;

public:
    // 构造函数
    Rectangle(int w, int h) : width(w), height(h) {}

    // 成员函数：计算面积
    int area() {
        return width * height;
    }
};

int main() {
    Rectangle rect(5, 3);  // 创建 Rectangle 对象
    std::cout << "Area: " << rect.area() << std::endl;  // 调用成员函数
    return 0;
}
```

**代码解析：**
- `class Rectangle`：定义了一个 `Rectangle` 类，包含两个私有成员变量 `width` 和 `height`。
- `Rectangle(int w, int h) : width(w), height(h) {}`：构造函数，初始化 `width` 和 `height`。
- `int area()`：成员函数，计算矩形的面积。
- `rect.area()`：通过对象 `rect` 调用成员函数 `area`。

### 4.2 静态函数

静态函数是使用 `static` 关键字修饰的函数，属于类本身，而不是类的对象。静态函数通常用于操作静态成员变量或实现工具函数。

**代码示例：**

```cpp
#include <iostream>

class MathUtils {
public:
    // 静态函数：计算平方
    static int square(int x) {
        return x * x;
    }
};

int main() {
    std::cout << "square(5) = " << MathUtils::square(5) << std::endl;  // 调用静态函数
    return 0;
}
```

**代码解析：**
- `static int square(int x)`：定义了一个静态函数 `square`，计算 `x` 的平方。
- `MathUtils::square(5)`：通过类名 `MathUtils` 调用静态函数 `square`。

### 4.3 友元函数

友元函数是使用 `friend` 关键字声明的函数，可以访问类的私有成员。友元函数通常用于重载运算符或实现特定功能。

**注意事项：**
- 友元函数会破坏类的封装性，应谨慎使用。

**代码示例：**

```cpp
#include <iostream>

class Point {
private:
    int x, y;

public:
    Point(int x, int y) : x(x), y(y) {}

    // 友元函数声明
    friend void printPoint(Point p);
};

// 友元函数定义
void printPoint(Point p) {
    std::cout << "Point: (" << p.x << ", " << p.y << ")" << std::endl;
}

int main() {
    Point p(3, 4);  // 创建 Point 对象
    printPoint(p);  // 调用友元函数
    return 0;
}
```

**代码解析：**
- `friend void printPoint(Point p)`：在 `Point` 类中声明 `printPoint` 为友元函数。
- `void printPoint(Point p)`：友元函数定义，可以访问 `Point` 类的私有成员 `x` 和 `y`。
- `printPoint(p)`：调用友元函数 `printPoint`，输出 `Point` 对象的坐标。

### 4.4 函数重写与函数重载的区别

函数重写（Override）和函数重载（Overload）是面向对象编程中的两个重要概念，但它们的含义和应用场景不同。

- **函数重载**：在同一个类中，定义多个同名函数，但参数列表不同。函数重载用于处理不同类型的数据，提高代码的可读性。
- **函数重写**：在派生类中重新定义基类中的虚函数，实现多态性。函数重写用于实现运行时多态。

**代码示例：**

```cpp
#include <iostream>

// 基类
class Base {
public:
    virtual void display() {
        std::cout << "Base class display" << std::endl;
    }
};

// 派生类
class Derived : public Base {
public:
    // 函数重写
    void display() override {
        std::cout << "Derived class display" << std::endl;
    }
};

int main() {
    Base* basePtr = new Derived();  // 基类指针指向派生类对象
    basePtr->display();  // 调用派生类的 display 函数
    delete basePtr;
    return 0;
}
```

**代码解析：**
- `virtual void display()`：基类中的虚函数 `display`，允许派生类重写。
- `void display() override`：派生类中重写了基类的 `display` 函数。
- `basePtr->display()`：通过基类指针调用 `display` 函数，实际调用的是派生类的 `display` 函数，实现了运行时多态。

## 五、C++ 函数模板

### 5.1 什么是函数模板？

**定义：** 函数模板是一种通用的函数描述，可以处理不同类型的数据。它允许我们编写一个通用的函数，而不需要为每种数据类型编写单独的函数。

**优势：** 提高代码复用性，避免重复编写相似的函数。

#### 代码示例：

```cpp
#include <iostream>

// 定义一个函数模板
template <typename T>
T add(T a, T b) {
    return a + b;
}

int main() {
    // 使用函数模板进行整数相加
    int intResult = add(3, 4);
    std::cout << "整数相加结果: " << intResult << std::endl;

    // 使用函数模板进行浮点数相加
    double doubleResult = add(3.5, 4.5);
    std::cout << "浮点数相加结果: " << doubleResult << std::endl;

    return 0;
}
```

**代码解析：**
- `template <typename T>` 定义了一个模板，`T` 是一个占位符，表示任意类型。
- `T add(T a, T b)` 是一个函数模板，接受两个类型为 `T` 的参数，并返回它们的和。
- 在 `main` 函数中，我们分别使用整数和浮点数调用 `add` 函数模板，编译器会根据实际参数类型生成具体的函数。

### 5.2 函数模板的定义与使用

**模板参数：** 模板参数可以是类型参数（如 `typename T`）或非类型参数（如 `int N`）。

**模板实例化：** 编译器根据实际参数类型生成具体的函数。

#### 代码示例：

```cpp
#include <iostream>

// 定义一个包含类型参数和非类型参数的函数模板
template <typename T, int N>
T multiply(T a) {
    return a * N;
}

int main() {
    // 使用函数模板进行整数乘法
    int intResult = multiply<int, 5>(3);
    std::cout << "整数乘法结果: " << intResult << std::endl;

    // 使用函数模板进行浮点数乘法
    double doubleResult = multiply<double, 2>(3.5);
    std::cout << "浮点数乘法结果: " << doubleResult << std::endl;

    return 0;
}
```

**代码解析：**
- `template <typename T, int N>` 定义了一个模板，`T` 是类型参数，`N` 是非类型参数。
- `T multiply(T a)` 是一个函数模板，接受一个类型为 `T` 的参数，并返回它与非类型参数 `N` 的乘积。
- 在 `main` 函数中，我们分别使用整数和浮点数调用 `multiply` 函数模板，编译器会根据实际参数类型和非类型参数生成具体的函数。

### 5.3 函数模板的重载与特化

**函数模板重载：** 定义多个同名函数模板，但模板参数不同。

**函数模板特化：** 为特定类型提供专门的实现。

#### 代码示例：

```cpp
#include <iostream>

// 通用函数模板
template <typename T>
T add(T a, T b) {
    std::cout << "通用模板调用" << std::endl;
    return a + b;
}

// 针对 int 类型的特化模板
template <>
int add(int a, int b) {
    std::cout << "int 类型特化模板调用" << std::endl;
    return a + b;
}

int main() {
    // 使用通用模板进行浮点数相加
    double doubleResult = add(3.5, 4.5);
    std::cout << "浮点数相加结果: " << doubleResult << std::endl;

    // 使用特化模板进行整数相加
    int intResult = add(3, 4);
    std::cout << "整数相加结果: " << intResult << std::endl;

    return 0;
}
```

**代码解析：**
- `template <typename T>` 定义了一个通用函数模板。
- `template <> int add(int a, int b)` 是针对 `int` 类型的特化模板。
- 在 `main` 函数中，我们分别使用浮点数和整数调用 `add` 函数模板，编译器会根据实际参数类型选择合适的模板进行调用。

## 六、函数指针与回调函数

### 6.1 函数指针

**定义：** 函数指针是指向函数的指针，可以用来调用函数。

**应用场景：** 实现回调函数、动态函数调用。

#### 代码示例：

```cpp
#include <iostream>

// 定义一个普通函数
void sayHello() {
    std::cout << "Hello, World!" << std::endl;
}

int main() {
    // 声明一个函数指针
    void (*funcPtr)() = sayHello;

    // 通过函数指针调用函数
    funcPtr();

    return 0;
}
```

**代码解析：**
- `void (*funcPtr)()` 声明了一个函数指针 `funcPtr`，它指向一个不接受参数且返回类型为 `void` 的函数。
- `funcPtr = sayHello` 将函数指针 `funcPtr` 指向 `sayHello` 函数。
- `funcPtr()` 通过函数指针调用 `sayHello` 函数。

### 6.2 回调函数

**定义：** 回调函数是作为参数传递给其他函数的函数，在特定事件发生时被调用。

**应用场景：** 事件驱动编程、异步编程。

#### 代码示例：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

// 定义一个回调函数
bool compare(int a, int b) {
    return a < b;
}

// 定义一个接受回调函数的排序函数
void sortWithCallback(std::vector<int>& vec, bool (*cmp)(int, int)) {
    std::sort(vec.begin(), vec.end(), cmp);
}

int main() {
    std::vector<int> vec = {3, 1, 4, 1, 5, 9, 2, 6};

    // 使用回调函数进行排序
    sortWithCallback(vec, compare);

    // 输出排序结果
    for (int i : vec) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**代码解析：**
- `bool compare(int a, int b)` 定义了一个回调函数，用于比较两个整数的大小。
- `void sortWithCallback(std::vector<int>& vec, bool (*cmp)(int, int))` 定义了一个接受回调函数的排序函数。
- 在 `main` 函数中，我们使用 `sortWithCallback` 函数对 `vec` 进行排序，并传入 `compare` 作为回调函数。

## 七、Lambda 表达式

### 7.1 什么是 Lambda 表达式？

**定义：** Lambda 表达式是一种匿名函数，可以在代码中直接定义和使用。

**优势：** 简化代码、提高可读性、支持闭包。

#### 代码示例：

```cpp
#include <iostream>

int main() {
    // 定义一个简单的 Lambda 表达式
    auto sayHello = []() {
        std::cout << "Hello, Lambda!" << std::endl;
    };

    // 调用 Lambda 表达式
    sayHello();

    return 0;
}
```

**代码解析：**
- `auto sayHello = []() { ... }` 定义了一个 Lambda 表达式，`[]` 是捕获列表，`()` 是参数列表，`{ ... }` 是函数体。
- `sayHello()` 调用 Lambda 表达式。

### 7.2 Lambda 表达式的语法

**捕获列表：** 指定 Lambda 表达式可以访问的外部变量。

**参数列表：** 指定 Lambda 表达式的参数。

**函数体：** 实现 Lambda 表达式的具体功能。

#### 代码示例：

```cpp
#include <iostream>

int main() {
    int x = 10;

    // 定义一个包含捕获列表、参数列表和函数体的 Lambda 表达式
    auto addX = [x](int a) {
        return a + x;
    };

    // 调用 Lambda 表达式
    std::cout << "结果: " << addX(5) << std::endl;

    return 0;
}
```

**代码解析：**
- `[x]` 捕获外部变量 `x`。
- `(int a)` 是参数列表，表示 Lambda 表达式接受一个整数参数 `a`。
- `{ return a + x; }` 是函数体，实现将参数 `a` 与捕获的变量 `x` 相加。
- `addX(5)` 调用 Lambda 表达式，传入参数 `5`。

### 7.3 Lambda 表达式的应用

**应用场景：** STL 算法、事件处理、多线程编程。

#### 代码示例：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {3, 1, 4, 1, 5, 9, 2, 6};

    // 使用 Lambda 表达式进行排序
    std::sort(vec.begin(), vec.end(), [](int a, int b) {
        return a < b;
    });

    // 输出排序结果
    for (int i : vec) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

**代码解析：**
- `std::sort(vec.begin(), vec.end(), [](int a, int b) { return a < b; })` 使用 Lambda 表达式作为排序规则，对 `vec` 进行升序排序。
- `for (int i : vec)` 遍历排序后的 `vec`，并输出每个元素。

## 八、std::function 与可调用对象

### 8.1 std::function

**定义：** `std::function` 是一种通用的可调用对象包装器，可以存储、复制和调用任何可调用对象。

**优势：** 提高代码灵活性、支持函数回调。

#### 代码示例：

```cpp
#include <iostream>
#include <functional>

// 定义一个普通函数
void sayHello() {
    std::cout << "Hello, World!" << std::endl;
}

int main() {
    // 使用 std::function 存储普通函数
    std::function<void()> func = sayHello;

    // 调用 std::function
    func();

    // 使用 std::function 存储 Lambda 表达式
    func = []() {
        std::cout << "Hello, Lambda!" << std::endl;
    };

    // 调用 std::function
    func();

    return 0;
}
```

**代码解析：**
- `std::function<void()> func = sayHello` 使用 `std::function` 存储普通函数 `sayHello`。
- `func()` 调用 `std::function` 存储的函数。
- `func = []() { ... }` 将 Lambda 表达式存储到 `std::function` 中。
- `func()` 调用 `std::function` 存储的 Lambda 表达式。

### 8.2 可调用对象

**定义：** 可调用对象是可以像函数一样被调用的对象，例如函数指针、Lambda 表达式、函数对象。

**应用场景：** 实现回调函数、事件处理。

#### 代码示例：

```cpp
#include <iostream>
#include <functional>

// 定义一个函数对象
struct Adder {
    int operator()(int a, int b) const {
        return a + b;
    }
};

int main() {
    // 使用 std::function 存储函数对象
    std::function<int(int, int)> func = Adder();

    // 调用 std::function
    std::cout << "结果: " << func(3, 4) << std::endl;

    return 0;
}
```

**代码解析：**
- `struct Adder` 定义了一个函数对象，重载了 `operator()`，使其可以像函数一样被调用。
- `std::function<int(int, int)> func = Adder()` 使用 `std::function` 存储函数对象 `Adder`。
- `func(3, 4)` 调用 `std::function` 存储的函数对象，并传入参数 `3` 和 `4`。

## 九、总结与展望

C++ 函数是 C++ 编程中的核心概念，涵盖了从普通函数到函数模板、函数指针、回调函数、Lambda 表达式以及 `std::function` 等多种形式。函数模板提高了代码的复用性，函数指针和回调函数增强了代码的灵活性，Lambda 表达式和 `std::function` 则进一步简化了代码结构。

随着 C++ 语言的不断发展，函数相关特性也在不断完善。未来，C++ 可能会引入更多高级特性，如更强大的模板元编程、更简洁的函数式编程支持等。鼓励读者持续学习，掌握 C++ 函数的最新特性，以应对日益复杂的编程需求。

文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)