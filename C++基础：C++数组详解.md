---
title: 'C++数组详解'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
  - c++数组
---



# C++数组详解

#### 1. 引言

在C++编程中，数组是一种非常重要的数据结构。数组允许我们存储多个相同类型的数据，并通过索引快速访问这些数据。数组在编程中的应用非常广泛，例如在处理大量数据、实现矩阵运算、存储字符串等场景中，数组都扮演着关键角色。

#### 2. C++数组的基本概念

##### 什么是数组？

数组是一种数据结构，用于存储相同类型的元素。数组中的每个元素都有一个唯一的索引，通过索引可以快速访问和修改数组中的元素。

##### 数组的声明和初始化

在C++中，数组的声明需要指定数组的类型、名称和大小。数组的大小在声明时必须是一个常量表达式，且在编译时确定。

```cpp
#include <iostream>
using namespace std;

int main() {
    // 声明一个包含5个整数的数组
    int arr[5];

    // 初始化数组
    arr[0] = 10;
    arr[1] = 20;
    arr[2] = 30;
    arr[3] = 40;
    arr[4] = 50;

    // 输出数组元素
    for (int i = 0; i < 5; i++) {
        cout << "arr[" << i << "] = " << arr[i] << endl;
    }

    return 0;
}
```

**代码解析**：
- `int arr[5];`：声明一个包含5个整数的数组。
- `arr[0] = 10;`：将数组的第一个元素（索引为0）赋值为10。
- `for (int i = 0; i < 5; i++)`：通过循环遍历数组并输出每个元素的值。

##### 数组的大小和索引

数组的大小是指数组中元素的数量，而索引是用来访问数组中特定元素的位置。C++中的数组索引从0开始，因此一个大小为`n`的数组，其索引范围是`0`到`n-1`。

#### 3. 一维数组

##### 一维数组的声明和初始化

一维数组是最简单的数组形式，它只包含一行元素。一维数组可以通过直接初始化或逐个赋值的方式进行初始化。

```cpp
#include <iostream>
using namespace std;

int main() {
    // 声明并初始化一维数组
    int arr[5] = {10, 20, 30, 40, 50};

    // 输出数组元素
    for (int i = 0; i < 5; i++) {
        cout << "arr[" << i << "] = " << arr[i] << endl;
    }

    return 0;
}
```

**代码解析**：
- `int arr[5] = {10, 20, 30, 40, 50};`：声明并初始化一个包含5个整数的数组。
- `for (int i = 0; i < 5; i++)`：通过循环遍历数组并输出每个元素的值。

##### 访问和修改数组元素

数组元素可以通过索引进行访问和修改。索引从0开始，依次递增。

```cpp
#include <iostream>
using namespace std;

int main() {
    // 声明并初始化一维数组
    int arr[5] = {10, 20, 30, 40, 50};

    // 访问并修改数组元素
    arr[2] = 35;  // 修改第三个元素的值

    // 输出数组元素
    for (int i = 0; i < 5; i++) {
        cout << "arr[" << i << "] = " << arr[i] << endl;
    }

    return 0;
}
```

**代码解析**：
- `arr[2] = 35;`：将数组的第三个元素（索引为2）的值修改为35。
- `for (int i = 0; i < 5; i++)`：通过循环遍历数组并输出每个元素的值。

##### 数组的遍历

数组的遍历是指依次访问数组中的每个元素。通常使用`for`循环来实现数组的遍历。

```cpp
#include <iostream>
using namespace std;

int main() {
    // 声明并初始化一维数组
    int arr[5] = {10, 20, 30, 40, 50};

    // 遍历数组并输出每个元素
    for (int i = 0; i < 5; i++) {
        cout << "arr[" << i << "] = " << arr[i] << endl;
    }

    return 0;
}
```

**代码解析**：
- `for (int i = 0; i < 5; i++)`：通过循环遍历数组并输出每个元素的值。

#### 4. 多维数组

##### 二维数组的声明和初始化

二维数组可以看作是一个表格，包含行和列。二维数组的声明需要指定行数和列数。

```cpp
#include <iostream>
using namespace std;

int main() {
    // 声明并初始化一个3行4列的二维数组
    int arr[3][4] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };

    // 输出二维数组元素
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 4; j++) {
            cout << "arr[" << i << "][" << j << "] = " << arr[i][j] << " ";
        }
        cout << endl;
    }

    return 0;
}
```

**代码解析**：
- `int arr[3][4]`：声明一个3行4列的二维数组。
- `for (int i = 0; i < 3; i++)`：外层循环遍历行。
- `for (int j = 0; j < 4; j++)`：内层循环遍历列。
- `cout << "arr[" << i << "][" << j << "] = " << arr[i][j] << " ";`：输出每个元素的值。

##### 访问和修改多维数组元素

二维数组的元素可以通过行索引和列索引进行访问和修改。

```cpp
#include <iostream>
using namespace std;

int main() {
    // 声明并初始化一个3行4列的二维数组
    int arr[3][4] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };

    // 访问并修改数组元素
    arr[1][2] = 70;  // 修改第二行第三列的元素

    // 输出二维数组元素
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 4; j++) {
            cout << "arr[" << i << "][" << j << "] = " << arr[i][j] << " ";
        }
        cout << endl;
    }

    return 0;
}
```

**代码解析**：
- `arr[1][2] = 70;`：将第二行第三列的元素（索引为`[1][2]`）的值修改为70。
- `for (int i = 0; i < 3; i++)`：外层循环遍历行。
- `for (int j = 0; j < 4; j++)`：内层循环遍历列。
- `cout << "arr[" << i << "][" << j << "] = " << arr[i][j] << " ";`：输出每个元素的值。

##### 多维数组的遍历

多维数组的遍历需要使用嵌套循环。对于二维数组，通常使用两层`for`循环来遍历行和列。

```cpp
#include <iostream>
using namespace std;

int main() {
    // 声明并初始化一个3行4列的二维数组
    int arr[3][4] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };

    // 遍历二维数组并输出每个元素
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 4; j++) {
            cout << "arr[" << i << "][" << j << "] = " << arr[i][j] << " ";
        }
        cout << endl;
    }

    return 0;
}
```

**代码解析**：
- `for (int i = 0; i < 3; i++)`：外层循环遍历行。
- `for (int j = 0; j < 4; j++)`：内层循环遍历列。
- `cout << "arr[" << i << "][" << j << "] = " << arr[i][j] << " ";`：输出每个元素的值。

#### 5. 动态数组

##### 动态数组的声明和初始化

动态数组是指在运行时动态分配内存的数组。C++中可以使用`new`关键字来动态分配数组内存，使用`delete`关键字来释放内存。

```cpp
#include <iostream>
using namespace std;

int main() {
    int size;
    cout << "Enter the size of the array: ";
    cin >> size;

    // 动态分配一维数组
    int* arr = new int[size];

    // 初始化数组
    for (int i = 0; i < size; i++) {
        arr[i] = i * 10;
    }

    // 输出数组元素
    for (int i = 0; i < size; i++) {
        cout << "arr[" << i << "] = " << arr[i] << endl;
    }

    // 释放动态分配的内存
    delete[] arr;

    return 0;
}
```

**代码解析**：
- `int* arr = new int[size];`：动态分配一个大小为`size`的整数数组。
- `for (int i = 0; i < size; i++)`：通过循环初始化数组元素。
- `delete[] arr;`：释放动态分配的数组内存。

##### 动态数组的使用场景

动态数组通常用于在编译时无法确定数组大小的情况下。例如，当数组的大小需要根据用户输入或程序运行时的计算结果来确定时，动态数组非常有用。

##### 动态数组的内存管理

动态数组的内存管理需要特别注意，因为动态分配的内存不会自动释放。如果不释放动态分配的内存，可能会导致内存泄漏。因此，在使用完动态数组后，必须使用`delete[]`来释放内存。

```cpp
#include <iostream>
using namespace std;

int main() {
    int size;
    cout << "Enter the size of the array: ";
    cin >> size;

    // 动态分配一维数组
    int* arr = new int[size];

    // 初始化数组
    for (int i = 0; i < size; i++) {
        arr[i] = i * 10;
    }

    // 输出数组元素
    for (int i = 0; i < size; i++) {
        cout << "arr[" << i << "] = " << arr[i] << endl;
    }

    // 释放动态分配的内存
    delete[] arr;

    return 0;
}
```

**代码解析**：
- `int* arr = new int[size];`：动态分配一个大小为`size`的整数数组。
- `for (int i = 0; i < size; i++)`：通过循环初始化数组元素。
- `delete[] arr;`：释放动态分配的数组内存。

#### 6. 数组与指针

##### 数组与指针的关系

在C++中，数组和指针有着密切的关系。数组名本质上是一个指向数组第一个元素的常量指针。也就是说，数组名可以被当作指针来使用，但它不能被重新赋值。

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[5] = {10, 20, 30, 40, 50};

    // 数组名 arr 是一个指向数组第一个元素的指针
    cout << "arr[0] = " << *arr << endl;  // 输出 10

    // 通过指针访问数组元素
    int* ptr = arr;  // ptr 指向数组的第一个元素
    cout << "ptr[0] = " << *ptr << endl;  // 输出 10

    return 0;
}
```

**代码解析**：
- `arr` 是数组名，它是一个指向数组第一个元素的指针。
- `*arr` 解引用指针，访问数组的第一个元素。
- `int* ptr = arr;` 将指针 `ptr` 指向数组的第一个元素。

##### 通过指针访问数组元素

指针可以用来访问数组中的元素，通过指针的移动可以遍历整个数组。

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[5] = {10, 20, 30, 40, 50};
    int* ptr = arr;  // ptr 指向数组的第一个元素

    // 通过指针访问数组元素
    for (int i = 0; i < 5; i++) {
        cout << "Element " << i << " = " << *(ptr + i) << endl;
    }

    return 0;
}
```

**代码解析**：
- `*(ptr + i)` 通过指针的偏移量访问数组中的元素。
- `ptr + i` 表示指针移动 `i` 个位置，`*` 解引用指针获取元素值。

##### 数组指针的移动和操作

指针可以通过自增或自减操作来移动，从而遍历数组。

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[5] = {10, 20, 30, 40, 50};
    int* ptr = arr;  // ptr 指向数组的第一个元素

    // 通过指针移动访问数组元素
    for (int i = 0; i < 5; i++) {
        cout << "Element " << i << " = " << *ptr << endl;
        ptr++;  // 指针移动到下一个元素
    }

    return 0;
}
```

**代码解析**：
- `ptr++` 将指针移动到下一个元素。
- `*ptr` 解引用指针获取当前元素的值。

#### 7. 数组的常见操作

##### 数组的排序

数组的排序是常见的操作之一。可以使用标准库中的 `std::sort` 函数对数组进行排序。

```cpp
#include <iostream>
#include <algorithm>  // std::sort
using namespace std;

int main() {
    int arr[5] = {30, 10, 50, 20, 40};

    // 使用 std::sort 对数组进行排序
    sort(arr, arr + 5);

    // 输出排序后的数组
    for (int i = 0; i < 5; i++) {
        cout << "arr[" << i << "] = " << arr[i] << endl;
    }

    return 0;
}
```

**代码解析**：
- `sort(arr, arr + 5);` 对数组 `arr` 进行升序排序。
- `arr + 5` 表示数组的末尾（不包括最后一个元素）。

##### 数组的查找

数组的查找可以通过线性查找或二分查找实现。线性查找适用于未排序的数组，而二分查找适用于已排序的数组。

```cpp
#include <iostream>
using namespace std;

// 线性查找函数
int linearSearch(int arr[], int size, int target) {
    for (int i = 0; i < size; i++) {
        if (arr[i] == target) {
            return i;  // 返回目标元素的索引
        }
    }
    return -1;  // 未找到目标元素
}

int main() {
    int arr[5] = {30, 10, 50, 20, 40};
    int target = 20;

    // 线性查找
    int index = linearSearch(arr, 5, target);
    if (index != -1) {
        cout << "Element found at index: " << index << endl;
    } else {
        cout << "Element not found." << endl;
    }

    return 0;
}
```

**代码解析**：
- `linearSearch` 函数通过遍历数组查找目标元素。
- `return i;` 返回目标元素的索引。
- `return -1;` 表示未找到目标元素。

##### 数组的复制和合并

数组的复制可以通过逐个元素赋值实现，而数组的合并可以通过创建一个新数组并将两个数组的元素依次复制到新数组中。

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr1[3] = {1, 2, 3};
    int arr2[3] = {4, 5, 6};
    int mergedArr[6];

    // 复制 arr1 到 mergedArr
    for (int i = 0; i < 3; i++) {
        mergedArr[i] = arr1[i];
    }

    // 复制 arr2 到 mergedArr
    for (int i = 0; i < 3; i++) {
        mergedArr[i + 3] = arr2[i];
    }

    // 输出合并后的数组
    for (int i = 0; i < 6; i++) {
        cout << "mergedArr[" << i << "] = " << mergedArr[i] << endl;
    }

    return 0;
}
```

**代码解析**：
- `mergedArr[i] = arr1[i];` 将 `arr1` 的元素复制到 `mergedArr`。
- `mergedArr[i + 3] = arr2[i];` 将 `arr2` 的元素复制到 `mergedArr`。

#### 8. 数组的边界检查

##### 数组越界的危害

数组越界是指访问数组时超出了数组的合法索引范围。数组越界可能会导致程序崩溃、数据损坏或安全漏洞。

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[5] = {10, 20, 30, 40, 50};

    // 数组越界访问
    cout << arr[5] << endl;  // 访问非法索引

    return 0;
}
```

**代码解析**：
- `arr[5]` 访问了数组的第6个元素，超出了数组的合法索引范围（0到4）。

##### 如何进行有效的边界检查

为了避免数组越界，可以在访问数组元素之前进行边界检查。

```cpp
#include <iostream>
using namespace std;

int main() {
    int arr[5] = {10, 20, 30, 40, 50};
    int index;

    cout << "Enter an index: ";
    cin >> index;

    // 边界检查
    if (index >= 0 && index < 5) {
        cout << "arr[" << index << "] = " << arr[index] << endl;
    } else {
        cout << "Index out of bounds!" << endl;
    }

    return 0;
}
```

**代码解析**：
- `if (index >= 0 && index < 5)` 检查索引是否在合法范围内。
- `cout << "Index out of bounds!" << endl;` 提示用户索引越界。

#### 9. 字符数组

##### 字符数组的声明和初始化

字符数组是用来存储字符序列的数组，通常用于表示C风格字符串。

```cpp
#include <iostream>
using namespace std;

int main() {
    char str[6] = {'H', 'e', 'l', 'l', 'o', '\0'};  // 字符数组

    // 输出字符数组
    cout << "str = " << str << endl;

    return 0;
}
```

**代码解析**：
- `char str[6] = {'H', 'e', 'l', 'l', 'o', '\0'};` 声明并初始化一个字符数组。
- `'\0'` 是字符串的结束符，表示字符串的结尾。

##### 字符数组的常见操作

字符数组可以进行字符替换、字符查找等操作。

```cpp
#include <iostream>
using namespace std;

int main() {
    char str[6] = {'H', 'e', 'l', 'l', 'o', '\0'};

    // 字符替换
    str[1] = 'a';  // 将 'e' 替换为 'a'

    // 输出字符数组
    cout << "str = " << str << endl;  // 输出 "Hallo"

    return 0;
}
```

**代码解析**：
- `str[1] = 'a';` 将字符数组中的第二个字符替换为 `'a'`。

##### 字符数组与C风格字符串的关系

字符数组可以用来表示C风格字符串，C风格字符串是以 `'\0'` 结尾的字符数组。

```cpp
#include <iostream>
using namespace std;

int main() {
    char str[] = "Hello";  // C风格字符串

    // 输出字符数组
    cout << "str = " << str << endl;

    return 0;
}
```

**代码解析**：
- `char str[] = "Hello";` 声明并初始化一个C风格字符串。
- 编译器会自动在字符串末尾添加 `'\0'`。

#### 10. C风格字符串

##### 什么是C风格字符串？

C风格字符串是以 `'\0'` 结尾的字符数组，通常用于表示文本数据。

##### C风格字符串的声明和初始化

C风格字符串可以通过字符数组或字符串字面量来声明和初始化。

```cpp
#include <iostream>
using namespace std;

int main() {
    char str1[] = "Hello";  // 字符串字面量
    char str2[6] = {'H', 'e', 'l', 'l', 'o', '\0'};  // 字符数组

    // 输出字符串
    cout << "str1 = " << str1 << endl;
    cout << "str2 = " << str2 << endl;

    return 0;
}
```

**代码解析**：
- `char str1[] = "Hello";` 声明并初始化一个C风格字符串。
- `char str2[6] = {'H', 'e', 'l', 'l', 'o', '\0'};` 声明并初始化一个字符数组。

##### C风格字符串的常见操作

C风格字符串可以使用 `<cstring>` 库中的函数进行字符串连接、字符串比较等操作。

```cpp
#include <iostream>
#include <cstring>  // C风格字符串库
using namespace std;

int main() {
    char str1[10] = "Hello";
    char str2[] = "World";

    // 字符串连接
    strcat(str1, str2);
    cout << "str1 = " << str1 << endl;  // 输出 "HelloWorld"

    // 字符串比较
    if (strcmp(str1, "HelloWorld") == 0) {
        cout << "Strings are equal." << endl;
    } else {
        cout << "Strings are not equal." << endl;
    }

    return 0;
}
```

**代码解析**：
- `strcat(str1, str2);` 将 `str2` 连接到 `str1` 的末尾。
- `strcmp(str1, "HelloWorld")` 比较两个字符串是否相等。

##### C风格字符串与标准库`<cstring>`的使用

`<cstring>` 库提供了许多用于处理C风格字符串的函数，如 `strlen`、`strcpy`、`strcat` 等。

```cpp
#include <iostream>
#include <cstring>  // C风格字符串库
using namespace std;

int main() {
    char str1[10] = "Hello";
    char str2[10];

    // 字符串复制
    strcpy(str2, str1);
    cout << "str2 = " << str2 << endl;  // 输出 "Hello"

    // 字符串长度
    cout << "Length of str1 = " << strlen(str1) << endl;  // 输出 5

    return 0;
}
```

**代码解析**：
- `strcpy(str2, str1);` 将 `str1` 复制到 `str2`。
- `strlen(str1)` 返回字符串 `str1` 的长度（不包括 `'\0'`）。

#### 11. 总结

本文详细介绍了C++数组的相关知识，包括数组的声明、初始化、访问、修改和遍历是学习C++编程的基础，数组与指针的关系、数组的常见操作、数组的边界检查、字符数组和C风格字符串的使用。通过代码示例和详细解析，读者可以更好地理解和应用C++数组。

在实际编程中，数组是非常基础且重要的数据结构，掌握数组的声明、初始化、访问、修改和遍历是学习C++编程的基础。建议读者在实际项目中多加练习，进一步巩固和应用所学知识。

进一步学习的方向可以包括：
- 动态内存分配的高级用法。
- 使用标准库中的容器（如 `std::vector` 和 `std::array`）替代传统数组。
- 深入学习字符串处理和字符编码的相关知识。

文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)