---
title: 'C++ 联合体(union) 详解'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
  - c++基础
---



## C++ 联合体(union) 详解

---

### **一、引言**

在C++中，联合体（`union`）是一种特殊的数据结构，它允许在同一块内存空间中存储不同类型的数据。与结构体（`struct`）不同，联合体的所有成员共享同一块内存，因此联合体的大小取决于其最大成员的大小。

**联合体的用途：**

- 节省内存空间：当需要存储不同类型的数据，但一次只使用其中一种时，联合体可以显著减少内存占用。
- 类型双关（Type Punning）：通过联合体可以访问不同类型的数据，尽管这种做法需要谨慎，因为它可能导致未定义行为。

**与结构体的对比：**

- 结构体的成员各自独立分配内存，而联合体的成员共享同一块内存。
- 结构体可以同时访问所有成员，而联合体一次只能使用一个成员。

**联合体的优势：**
- 在某些场景下，联合体可以显著节省内存空间，尤其是在嵌入式系统或需要高效利用内存的场景中。

---

### **二、联合体的基本语法**

#### **1. 定义联合体**
联合体使用关键字 `union` 来定义，后跟联合体的名称。联合体的定义与结构体类似，但所有成员共享同一块内存。

```cpp
union MyUnion {
    int i;
    float f;
    char c;
};
```

#### **2. 联合体成员**
联合体的成员可以是任何数据类型，包括基本类型、数组、指针等。所有成员共享同一块内存空间。

```cpp
union MyUnion {
    int i;       // 4 bytes
    float f;     // 4 bytes
    char c[4];   // 4 bytes
};
```

#### **3. 联合体大小**
联合体的大小取决于其最大成员的大小。在上面的例子中，`MyUnion` 的大小为 4 字节，因为 `int`、`float` 和 `char[4]` 都是 4 字节。

#### **4. 访问联合体成员**
联合体成员的访问方式与结构体相同，使用成员访问运算符 `.` 或 `->`。

```cpp
MyUnion u;
u.i = 10;       // 访问 int 成员
u.f = 3.14f;    // 访问 float 成员
u.c[0] = 'A';   // 访问 char 成员
```

---

### **三、联合体的内存布局**

#### **1. 共享内存空间**
联合体的所有成员共享同一块内存空间。这意味着当你为一个成员赋值时，其他成员的值也会受到影响。

```cpp
union MyUnion {
    int i;
    float f;
    char c;
};

MyUnion u;
u.i = 10;       // 现在 u.i 是 10
u.f = 3.14f;    // 现在 u.f 是 3.14f，u.i 的值被覆盖
u.c = 'A';      // 现在 u.c 是 'A'，u.f 的值被覆盖
```

#### **2. 不同成员类型的内存存储**
由于联合体成员共享内存，不同类型的成员在内存中的存储方式可能不同。例如，`int` 和 `float` 在内存中的表示方式不同，因此当你从一个成员切换到另一个成员时，可能会看到不同的值。

```cpp
union MyUnion {
    int i;
    float f;
};

MyUnion u;
u.i = 10;       // 现在 u.i 是 10
std::cout << u.f;  // 输出可能是一个非预期的浮点值，因为 u.f 的内存被 u.i 覆盖
```

#### **3. 只能使用一个成员**
联合体的成员不能同时有效，只能使用其中一个。如果你尝试同时使用多个成员，可能会导致未定义行为。

---

### **四、联合体与结构体的区别**

| **特性**     | **联合体**                                     | **结构体**                         |
| ------------ | ---------------------------------------------- | ---------------------------------- |
| **内存分配** | 成员共享同一块内存                             | 成员各自独立分配内存               |
| **成员访问** | 一次只能使用一个成员                           | 可以同时访问所有成员               |
| **大小计算** | 大小取决于最大成员                             | 大小是所有成员大小之和（考虑对齐） |
| **使用场景** | 节省内存或类型双关                             | 存储多个相关数据项                 |
| **限制**     | 不能包含带有构造函数、析构函数或虚函数的类类型 | 没有此限制                         |

---

### **五、联合体的使用场景**

#### **1. 节省内存空间**
联合体适用于需要存储不同类型的数据，但一次只使用其中一种的场景。例如，在嵌入式系统中，内存资源有限，联合体可以显著减少内存占用。

```cpp
union Data {
    int i;
    float f;
    char c[4];
};

Data d;
d.i = 10;  // 使用 int 类型
d.f = 3.14f;  // 使用 float 类型
```

#### **2. 类型双关**
联合体可以用于类型双关，即通过联合体访问不同类型的数据。尽管这种做法需要谨慎，因为它可能导致未定义行为。

```cpp
union MyUnion {
    int i;
    float f;
};

MyUnion u;
u.f = 3.14f;
std::cout << u.i;  // 输出可能是一个非预期的整数值
```

#### **3. 实现特定数据结构**
联合体可以用于实现 tagged union，即一种可以根据标签（tag）选择不同类型的数据结构。

```cpp
enum Type { INT, FLOAT, CHAR };

union Data {
    int i;
    float f;
    char c;
};

struct TaggedUnion {
    Type tag;
    Data data;
};

TaggedUnion tu;
tu.tag = INT;
tu.data.i = 10;
```

---

### **六、联合体的注意事项**

#### **1. 不能包含带有构造函数、析构函数或虚函数的类类型**
联合体不能包含带有构造函数、析构函数或虚函数的类类型，因为联合体无法自动管理这些类型的生命周期。

#### **2. 联合体不能作为基类使用**
联合体不能被继承，也不能作为基类使用。

#### **3. 确保访问的成员是最近写入的**
使用联合体时，必须确保访问的成员是最近写入的，否则可能导致未定义行为。

#### **4. 使用 `std::variant` 替代联合体**
C++17 引入了 `std::variant`，它提供了更安全、更易用的类型双关机制，可以替代联合体在某些场景下的使用。

---

### **七、示例代码**

#### **1. 存储不同类型数据的联合体**
```cpp
union MyUnion {
    int i;
    float f;
    char c;
};

MyUnion u;
u.i = 10;
std::cout << u.i << std::endl;  // 输出 10
u.f = 3.14f;
std::cout << u.f << std::endl;  // 输出 3.14
u.c = 'A';
std::cout << u.c << std::endl;  // 输出 'A'
```

#### **2. 使用联合体实现类型双关**
```cpp
union MyUnion {
    int i;
    float f;
};

MyUnion u;
u.f = 3.14f;
std::cout << u.i << std::endl;  // 输出可能是一个非预期的整数值
```

#### **3. 使用联合体实现 tagged union**
```cpp
enum Type { INT, FLOAT, CHAR };

union Data {
    int i;
    float f;
    char c;
};

struct TaggedUnion {
    Type tag;
    Data data;
};

TaggedUnion tu;
tu.tag = INT;
tu.data.i = 10;
std::cout << tu.data.i << std::endl;  // 输出 10
```

---

### **八、C++11 及之后版本对联合体的扩展**

#### **1. 匿名联合体**
C++11 引入了匿名联合体，无需命名联合体类型，直接定义联合体变量。

```cpp
union {
    int i;
    float f;
} u;

u.i = 10;
std::cout << u.i << std::endl;  // 输出 10
```

#### **2. 联合体中的非平凡类型**
C++11 允许联合体包含带有构造函数、析构函数或虚函数的类类型，但需要手动管理其生命周期。

```cpp
class MyClass {
public:
    MyClass() { std::cout << "Constructed" << std::endl; }
    ~MyClass() { std::cout << "Destructed" << std::endl; }
};

union MyUnion {
    MyClass c;
    int i;
};

MyUnion u;
new (&u.c) MyClass();  // 手动调用构造函数
u.c.~MyClass();        // 手动调用析构函数
```



#### **3. 联合体与 `std::variant` 的对比**

在C++17中，标准库引入了 `std::variant`，它是一种类型安全的联合体替代方案。与联合体不同，`std::variant` 提供了更安全、更易用的机制来处理不同类型的数据，同时避免了联合体可能导致的未定义行为。

---

##### **`std::variant` 的用法介绍**

`std::variant` 是一个可存储多种类型值的类模板，类似于联合体，但它提供了类型安全的访问机制。`std::variant` 的每个实例可以存储其模板参数列表中指定的一种类型。

###### **1）. 定义 `std::variant`**

`std::variant` 的定义方式与联合体类似，但它是类型安全的。

```cpp
#include <variant>

std::variant<int, float, char> var;
```

在上面的例子中，`var` 可以存储 `int`、`float` 或 `char` 类型的值。

###### **2）. 访问 `std::variant` 的值**

`std::variant` 提供了多种方式来访问其存储的值：

- **`std::get`：** 通过索引或类型获取值。
- **`std::visit`：** 使用访问者模式处理 `std::variant` 中的值。
- **`std::holds_alternative`：** 检查 `std::variant` 是否包含某种类型的值。

**示例 1：使用 `std::get`**

```cpp
#include <variant>
#include <iostream>

int main() {
    std::variant<int, float, char> var;
    var = 10;  // 存储 int 类型

    // 通过索引获取值
    std::cout << std::get<0>(var) << std::endl;  // 输出 10

    var = 3.14f;  // 存储 float 类型

    // 通过类型获取值
    std::cout << std::get<float>(var) << std::endl;  // 输出 3.14

    // 如果类型不匹配，会抛出 std::bad_variant_access 异常
    try {
        std::cout << std::get<int>(var) << std::endl;
    } catch (const std::bad_variant_access& e) {
        std::cerr << "Error: " << e.what() << std::endl;
    }
}
```

**示例 2：使用 `std::visit`**

`std::visit` 允许使用访问者模式处理 `std::variant` 中的值。

```cpp
#include <variant>
#include <iostream>

struct Visitor {
    void operator()(int i) const { std::cout << "int: " << i << std::endl; }
    void operator()(float f) const { std::cout << "float: " << f << std::endl; }
    void operator()(char c) const { std::cout << "char: " << c << std::endl; }
};

int main() {
    std::variant<int, float, char> var;
    var = 10;  // 存储 int 类型

    std::visit(Visitor{}, var);  // 输出 "int: 10"

    var = 3.14f;  // 存储 float 类型
    std::visit(Visitor{}, var);  // 输出 "float: 3.14"

    var = 'A';  // 存储 char 类型
    std::visit(Visitor{}, var);  // 输出 "char: A"
}
```

**示例 3：使用 `std::holds_alternative`**

`std::holds_alternative` 用于检查 `std::variant` 是否包含某种类型的值。

```cpp
#include <variant>
#include <iostream>

int main() {
    std::variant<int, float, char> var;
    var = 10;  // 存储 int 类型

    if (std::holds_alternative<int>(var)) {
        std::cout << "var contains int: " << std::get<int>(var) << std::endl;
    }

    var = 3.14f;  // 存储 float 类型

    if (std::holds_alternative<float>(var)) {
        std::cout << "var contains float: " << std::get<float>(var) << std::endl;
    }
}
```

---

##### **联合体与 `std::variant` 的对比**

| **特性**     | **联合体**                       | **std::variant**                     |
| ------------ | -------------------------------- | ------------------------------------ |
| **类型安全** | 不安全，可能导致未定义行为       | 安全，提供类型检查和异常处理         |
| **内存使用** | 内存共享，节省空间               | 内存不共享，但提供了更安全的访问机制 |
| **访问方式** | 直接访问成员，需手动管理类型切换 | 通过 `std::get` 或 `std::visit` 访问 |
| **易用性**   | 需要手动处理类型切换，容易出错   | 提供更简洁、易用的 API               |
| **性能**     | 内存共享，性能较高               | 类型安全机制可能带来轻微性能开销     |
| **适用场景** | 需要节省内存或实现类型双关的场景 | 需要类型安全的联合体替代方案         |

---

##### **总结**

- **联合体** 是一种高效的内存共享机制，适用于需要节省内存或实现类型双关的场景。然而，它的使用需要特别小心，避免未定义行为。
- **`std::variant`** 是 C++17 引入的类型安全联合体替代方案，提供了更安全、更易用的机制来处理不同类型的数据。
- 在现代 C++ 编程中，推荐优先使用 `std::variant`，尤其是在需要类型安全的场景下。但在某些特定场景（如嵌入式系统或需要极致内存优化的场景），联合体仍然具有不可替代的优势。

---

### **九、实际项目中的联合体应用案例**

#### **1. 嵌入式系统中的内存节省**
在嵌入式系统中，联合体可以用于节省内存空间，尤其是在需要存储不同类型的传感器数据时。

#### **2. 网络协议解析中的类型双关**
在网络协议解析中，联合体可以用于实现类型双关，将二进制数据解析为不同类型的数据。

#### **3. 游戏开发中的 tagged union**
在游戏开发中，联合体可以用于实现 tagged union，表示不同类型的游戏对象。

---

### **十、总结**

联合体是一种强大的工具，能够在特定场景下显著节省内存空间。然而，由于其成员共享同一块内存，使用时需要特别小心，避免未定义行为。在现代C++中，`std::variant` 提供了更安全、更易用的替代方案，但在某些特定场景下，联合体仍然是不可替代的。

---

### **十一、参考资料**

1. C++ Primer, 5th Edition
2. ISO/IEC 14882:2017 (C++17 Standard)
3. [cppreference.com - Union](https://en.cppreference.com/w/cpp/language/union)



文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)