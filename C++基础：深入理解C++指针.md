---
title: '深入理解C++指针'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
  - c++基础
---



## 深入理解C++指针

#### 1. 引言

指针在C++中是一个非常重要的概念，它允许程序员直接操作内存地址，从而实现高效的内存管理和灵活的编程。指针的基本概念是存储变量的内存地址，通过解引用操作可以访问该地址上的数据。指针的使用不仅提高了程序的性能，还为复杂的数据结构和算法提供了基础支持。

#### 2. 指针的基本概念

##### 2.1 指针的定义和声明

指针是一个变量，它存储的是另一个变量的内存地址。指针的声明需要指定它所指向的数据类型。

```cpp
#include <iostream>

int main() {
    int var = 10;  // 定义一个整型变量
    int* ptr;      // 定义一个指向整型的指针

    ptr = &var;    // 将指针指向变量var的地址

    std::cout << "Value of var: " << var << std::endl;
    std::cout << "Address of var: " << &var << std::endl;
    std::cout << "Value of ptr: " << ptr << std::endl;

    return 0;
}
```

**代码解析：**
- `int var = 10;`：定义一个整型变量`var`，并赋值为10。
- `int* ptr;`：定义一个指向整型的指针`ptr`。
- `ptr = &var;`：将指针`ptr`指向变量`var`的地址。
- `std::cout`：输出变量的值、地址以及指针的值。

##### 2.2 指针的初始化

指针在声明时可以初始化，也可以在后续赋值。未初始化的指针可能会导致未定义行为。

```cpp
#include <iostream>

int main() {
    int var = 20;
    int* ptr = &var;  // 指针在声明时初始化

    std::cout << "Value of var: " << var << std::endl;
    std::cout << "Value of ptr: " << ptr << std::endl;

    return 0;
}
```

**代码解析：**
- `int* ptr = &var;`：指针在声明时直接初始化为`var`的地址。

##### 2.3 指针的解引用

解引用操作是通过指针访问它所指向的变量的值。

```cpp
#include <iostream>

int main() {
    int var = 30;
    int* ptr = &var;

    std::cout << "Value of var: " << var << std::endl;
    std::cout << "Value of *ptr: " << *ptr << std::endl;  // 解引用指针

    *ptr = 40;  // 通过指针修改var的值
    std::cout << "New value of var: " << var << std::endl;

    return 0;
}
```

**代码解析：**
- `*ptr`：解引用指针，访问`var`的值。
- `*ptr = 40;`：通过指针修改`var`的值。

#### 3. 指针与内存地址

##### 3.1 内存地址的概念

内存地址是计算机内存中每个字节的唯一标识符。指针存储的就是这些地址。

```cpp
#include <iostream>

int main() {
    int var = 50;
    int* ptr = &var;

    std::cout << "Address of var: " << &var << std::endl;
    std::cout << "Value of ptr: " << ptr << std::endl;

    return 0;
}
```

**代码解析：**
- `&var`：获取变量`var`的内存地址。
- `ptr`：存储`var`的地址。

##### 3.2 指针与内存地址的关系

指针存储的是变量的内存地址，通过指针可以访问和修改该地址上的数据。

```cpp
#include <iostream>

int main() {
    int var = 60;
    int* ptr = &var;

    std::cout << "Value of var: " << var << std::endl;
    std::cout << "Address of var: " << &var << std::endl;
    std::cout << "Value of ptr: " << ptr << std::endl;
    std::cout << "Value of *ptr: " << *ptr << std::endl;

    return 0;
}
```

**代码解析：**
- `ptr`：存储`var`的地址。
- `*ptr`：解引用指针，访问`var`的值。

##### 3.3 指针的地址操作符

指针本身也是一个变量，它也有自己的地址。可以使用`&`操作符获取指针的地址。

```cpp
#include <iostream>

int main() {
    int var = 70;
    int* ptr = &var;

    std::cout << "Address of var: " << &var << std::endl;
    std::cout << "Value of ptr: " << ptr << std::endl;
    std::cout << "Address of ptr: " << &ptr << std::endl;

    return 0;
}
```

**代码解析：**
- `&ptr`：获取指针`ptr`的地址。

#### 4. 指针的算术运算

##### 4.1 指针的加法和减法

指针的加法和减法操作是基于指针所指向的数据类型的大小进行的。

```cpp
#include <iostream>

int main() {
    int arr[5] = {10, 20, 30, 40, 50};
    int* ptr = arr;

    std::cout << "Value of *ptr: " << *ptr << std::endl;
    ptr++;  // 指针加1
    std::cout << "Value of *ptr after increment: " << *ptr << std::endl;

    ptr--;  // 指针减1
    std::cout << "Value of *ptr after decrement: " << *ptr << std::endl;

    return 0;
}
```

**代码解析：**
- `ptr++`：指针加1，指向数组的下一个元素。
- `ptr--`：指针减1，指向数组的前一个元素。

##### 4.2 指针的比较运算

指针可以进行比较运算，通常用于判断指针是否指向同一个内存地址。

```cpp
#include <iostream>

int main() {
    int arr[5] = {10, 20, 30, 40, 50};
    int* ptr1 = arr;
    int* ptr2 = &arr[2];

    if (ptr1 < ptr2) {
        std::cout << "ptr1 is before ptr2" << std::endl;
    } else {
        std::cout << "ptr1 is after ptr2" << std::endl;
    }

    return 0;
}
```

**代码解析：**
- `ptr1 < ptr2`：比较两个指针的地址大小。

##### 4.3 指针与数组的关系

指针和数组在C++中密切相关，数组名本质上是一个指向数组首元素的指针。

```cpp
#include <iostream>

int main() {
    int arr[5] = {10, 20, 30, 40, 50};
    int* ptr = arr;

    for (int i = 0; i < 5; i++) {
        std::cout << "Value of arr[" << i << "]: " << *(ptr + i) << std::endl;
    }

    return 0;
}
```

**代码解析：**
- `*(ptr + i)`：通过指针访问数组元素。

#### 5. 指针与数组

##### 5.1 数组名作为指针

数组名可以作为指向数组首元素的指针使用。

```cpp
#include <iostream>

int main() {
    int arr[5] = {10, 20, 30, 40, 50};

    std::cout << "Value of arr[0]: " << *arr << std::endl;
    std::cout << "Value of arr[1]: " << *(arr + 1) << std::endl;

    return 0;
}
```

**代码解析：**
- `*arr`：数组名作为指针，访问首元素。
- `*(arr + 1)`：访问数组的第二个元素。

##### 5.2 指针数组

指针数组是一个数组，其中的每个元素都是一个指针。

```cpp
#include <iostream>

int main() {
    int var1 = 10, var2 = 20, var3 = 30;
    int* arr[3] = {&var1, &var2, &var3};

    for (int i = 0; i < 3; i++) {
        std::cout << "Value of arr[" << i << "]: " << *arr[i] << std::endl;
    }

    return 0;
}
```

**代码解析：**
- `int* arr[3]`：定义一个指针数组，存储三个整型变量的地址。
- `*arr[i]`：解引用指针数组中的元素。

##### 5.3 数组指针

数组指针是一个指向数组的指针。

```cpp
#include <iostream>

int main() {
    int arr[5] = {10, 20, 30, 40, 50};
    int (*ptr)[5] = &arr;

    for (int i = 0; i < 5; i++) {
        std::cout << "Value of arr[" << i << "]: " << (*ptr)[i] << std::endl;
    }

    return 0;
}
```

**代码解析：**
- `int (*ptr)[5]`：定义一个指向包含5个整型元素的数组的指针。
- `(*ptr)[i]`：通过数组指针访问数组元素。

#### 6. 指针与函数

##### 6.1 函数指针

函数指针是指向函数的指针，可以用来调用函数。

```cpp
#include <iostream>

void printHello() {
    std::cout << "Hello, World!" << std::endl;
}

int main() {
    void (*funcPtr)() = printHello;

    funcPtr();  // 通过函数指针调用函数

    return 0;
}
```

**代码解析：**
- `void (*funcPtr)()`：定义一个指向返回类型为`void`、无参数的函数的指针。
- `funcPtr()`：通过函数指针调用函数。

##### 6.2 指针作为函数参数

指针可以作为函数参数，用于修改函数外部的变量。

```cpp
#include <iostream>

void increment(int* ptr) {
    (*ptr)++;
}

int main() {
    int var = 10;
    increment(&var);

    std::cout << "Value of var after increment: " << var << std::endl;

    return 0;
}
```

**代码解析：**
- `increment(&var)`：将变量`var`的地址传递给函数。
- `(*ptr)++`：通过指针修改`var`的值。

##### 6.3 返回指针的函数

函数可以返回指针，用于动态分配内存或返回数组。

```cpp
#include <iostream>

int* createArray(int size) {
    int* arr = new int[size];
    for (int i = 0; i < size; i++) {
        arr[i] = i * 10;
    }
    return arr;
}

int main() {
    int* arr = createArray(5);

    for (int i = 0; i < 5; i++) {
        std::cout << "Value of arr[" << i << "]: " << arr[i] << std::endl;
    }

    delete[] arr;  // 释放动态分配的内存

    return 0;
}
```

**代码解析：**
- `int* createArray(int size)`：定义一个返回整型指针的函数。
- `new int[size]`：动态分配数组。
- `delete[] arr`：释放动态分配的内存。

#### 7. 指针与动态内存分配

##### 7.1 `new`和`delete`操作符

`new`用于动态分配内存，`delete`用于释放内存。

```cpp
#include <iostream>

int main() {
    int* ptr = new int;
    *ptr = 100;

    std::cout << "Value of *ptr: " << *ptr << std::endl;

    delete ptr;  // 释放内存

    return 0;
}
```

**代码解析：**
- `new int`：动态分配一个整型变量的内存。
- `delete ptr`：释放动态分配的内存。

##### 7.2 动态数组的创建和销毁

动态数组可以通过`new`创建，通过`delete[]`销毁。

```cpp
#include <iostream>

int main() {
    int* arr = new int[5];

    for (int i = 0; i < 5; i++) {
        arr[i] = i * 10;
    }

    for (int i = 0; i < 5; i++) {
        std::cout << "Value of arr[" << i << "]: " << arr[i] << std::endl;
    }

    delete[] arr;  // 释放动态数组的内存

    return 0;
}
```

**代码解析：**
- `new int[5]`：动态分配一个包含5个整型元素的数组。
- `delete[] arr`：释放动态数组的内存。

##### 7.3 内存泄漏与避免方法

内存泄漏是指动态分配的内存未被释放，导致内存占用不断增加。

```cpp
#include <iostream>

int main() {
    int* ptr = new int;
    *ptr = 200;

    // 忘记释放内存
    // delete ptr;

    return 0;
}
```

**代码解析：**
- 忘记调用`delete ptr`，导致内存泄漏。

**避免方法：**
- 确保每次使用`new`分配内存后，都调用`delete`释放内存。

#### 8. 指针与结构体

##### 8.1 结构体指针

结构体指针是指向结构体变量的指针。

```cpp
#include <iostream>

struct Point {
    int x;
    int y;
};

int main() {
    Point p = {10, 20};
    Point* ptr = &p;

    std::cout << "Value of x: " << ptr->x << std::endl;
    std::cout << "Value of y: " << ptr->y << std::endl;

    return 0;
}
```

**代码解析：**
- `Point* ptr = &p`：定义一个指向结构体`Point`的指针。
- `ptr->x`：通过指针访问结构体成员。

##### 8.2 指针访问结构体成员

通过指针可以访问和修改结构体成员。

```cpp
#include <iostream>

struct Point {
    int x;
    int y;
};

int main() {
    Point p = {10, 20};
    Point* ptr = &p;

    ptr->x = 30;
    ptr->y = 40;

    std::cout << "Value of x: " << p.x << std::endl;
    std::cout << "Value of y: " << p.y << std::endl;

    return 0;
}
```

**代码解析：**
- `ptr->x = 30`：通过指针修改结构体成员`x`的值。
- `ptr->y = 40`：通过指针修改结构体成员`y`的值。

##### 8.3 结构体数组与指针

结构体数组可以通过指针进行遍历和访问。

```cpp
#include <iostream>

struct Point {
    int x;
    int y;
};

int main() {
    Point arr[3] = {{10, 20}, {30, 40}, {50, 60}};
    Point* ptr = arr;

    for (int i = 0; i < 3; i++) {
        std::cout << "Value of x: " << (ptr + i)->x << ", y: " << (ptr + i)->y << std::endl;
    }

    return 0;
}
```

**代码解析：**
- `Point* ptr = arr`：定义一个指向结构体数组的指针。
- `(ptr + i)->x`：通过指针访问结构体数组中的元素。

#### 9. 指针与类

##### 9.1 类成员指针

类成员指针是指向类成员变量或成员函数的指针。

```cpp
#include <iostream>

class MyClass {
public:
    int value;
    void printValue() {
        std::cout << "Value: " << value << std::endl;
    }
};

int main() {
    int MyClass::* ptr = &MyClass::value;
    void (MyClass::* funcPtr)() = &MyClass::printValue;

    MyClass obj;
    obj.*ptr = 100;
    (obj.*funcPtr)();

    return 0;
}
```

**代码解析：**
- `int MyClass::* ptr = &MyClass::value`：定义一个指向类成员变量的指针。
- `void (MyClass::* funcPtr)() = &MyClass::printValue`：定义一个指向类成员函数的指针。
- `obj.*ptr = 100`：通过指针访问类成员变量。
- `(obj.*funcPtr)()`：通过指针调用类成员函数。

##### 9.2 对象指针

对象指针是指向类对象的指针。

```cpp
#include <iostream>

class MyClass {
public:
    int value;
    void printValue() {
        std::cout << "Value: " << value << std::endl;
    }
};

int main() {
    MyClass obj;
    MyClass* ptr = &obj;

    ptr->value = 200;
    ptr->printValue();

    return 0;
}
```

**代码解析：**
- `MyClass* ptr = &obj`：定义一个指向类对象的指针。
- `ptr->value = 200`：通过指针访问类成员变量。
- `ptr->printValue()`：通过指针调用类成员函数。

##### 9.3 智能指针

智能指针是C++11引入的用于自动管理动态内存的指针。

```cpp
#include <iostream>
#include <memory>

class MyClass {
public:
    MyClass() { std::cout << "Constructor called" << std::endl; }
    ~MyClass() { std::cout << "Destructor called" << std::endl; }
};

int main() {
    std::unique_ptr<MyClass> ptr(new MyClass());

    // 智能指针会自动释放内存

    return 0;
}
```

**代码解析：**
- `std::unique_ptr<MyClass> ptr(new MyClass())`：定义一个智能指针，自动管理动态分配的内存。
- 智能指针在作用域结束时自动调用析构函数，释放内存。

#### 10. 指针的常见错误与调试

##### 10.1 空指针与野指针

空指针是指未初始化的指针，野指针是指指向已释放或未分配内存的指针。

```cpp
#include <iostream>

int main() {
    int* ptr1 = nullptr;  // 空指针
    int* ptr2;            // 野指针

    // std::cout << *ptr1 << std::endl;  // 会导致程序崩溃
    // std::cout << *ptr2 << std::endl;  // 会导致未定义行为

    return 0;
}
```

**代码解析：**
- `int* ptr1 = nullptr`：定义一个空指针。
- `int* ptr2`：定义一个野指针。
- 解引用空指针或野指针会导致程序崩溃或未定义行为。

##### 10.2 指针越界访问

指针越界访问是指访问了指针所指向内存范围之外的数据。

```cpp
#include <iostream>

int main() {
    int arr[5] = {10, 20, 30, 40, 50};
    int* ptr = arr;

    std::cout << "Value of arr[5]: " << *(ptr + 5) << std::endl;  // 越界访问

    return 0;
}
```

**代码解析：**
- `*(ptr + 5)`：越界访问数组元素，可能导致程序崩溃或未定义行为。

##### 10.3 指针调试技巧

使用调试工具（如GDB）可以帮助定位指针相关的问题。

```cpp
#include <iostream>

int main() {
    int* ptr = nullptr;

    // 调试时可以检查ptr是否为nullptr
    if (ptr == nullptr) {
        std::cout << "ptr is nullptr" << std::endl;
    }

    return 0;
}
```

**代码解析：**
- 使用条件判断检查指针是否为空指针。

#### 11. 总结

指针在C++中具有重要的作用，它提供了直接操作内存的能力，但也带来了一些复杂性和风险。指针的优点包括高效的内存管理和灵活的编程，缺点则包括容易引发内存泄漏、空指针和野指针等问题。指针在动态内存分配、数据结构实现、函数指针等方面有广泛的应用。进一步学习建议包括深入理解C++的内存模型、智能指针的使用以及指针在多线程编程中的应用。

文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)