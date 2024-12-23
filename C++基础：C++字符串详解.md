---
title: 'C++字符串详解'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
  - c++基础
---



# C++字符串详解

## 1. 引言

### C++字符串的基本概念
在C++中，字符串是用来存储和操作文本数据的序列。字符串可以包含字母、数字、符号等字符。C++提供了两种主要的字符串类型：C风格字符串和C++标准库字符串（`std::string`）。

### C++中字符串的两种主要类型
1. **C风格字符串**：本质上是字符数组，以空字符（`'\0'`）结尾。例如：`char str[] = "Hello";`
2. **C++标准库字符串**（`std::string`）：是一个类，提供了丰富的成员函数来操作字符串，使用起来更加方便和安全。

### 为什么学习C++字符串很重要
字符串是编程中非常常见的数据类型，几乎所有的应用程序都会涉及到字符串的操作。掌握C++字符串的使用，能够帮助我们更高效地处理文本数据，避免常见的错误和陷阱。

---

## 2. C风格字符串

### 2.1 定义和初始化

C风格字符串是一个字符数组，以空字符（`'\0'`）结尾。定义和初始化C风格字符串的方式有多种。

```cpp
#include <iostream>
using namespace std;

int main() {
    // 定义并初始化C风格字符串
    char str1[] = "Hello";  // 编译器自动添加'\0'
    char str2[] = {'W', 'o', 'r', 'l', 'd', '\0'};  // 手动添加'\0'

    cout << "str1: " << str1 << endl;
    cout << "str2: " << str2 << endl;

    return 0;
}
```

**代码解析**：
- `str1` 是一个C风格字符串，编译器会自动在末尾添加空字符 `'\0'`。
- `str2` 是手动初始化的字符数组，必须显式添加 `'\0'` 以表示字符串的结束。

### 2.2 字符串操作

#### 2.2.1 字符串长度

C风格字符串的长度可以通过 `strlen` 函数获取。

```cpp
#include <iostream>
#include <cstring>  // 包含strlen函数的头文件
using namespace std;

int main() {
    char str[] = "Hello";
    int length = strlen(str);  // 获取字符串长度

    cout << "Length of str: " << length << endl;

    return 0;
}
```

**代码解析**：
- `strlen(str)` 返回字符串的长度，不包括末尾的 `'\0'`。

#### 2.2.2 字符串连接

C风格字符串的连接可以使用 `strcat` 函数。

```cpp
#include <iostream>
#include <cstring>  // 包含strcat函数的头文件
using namespace std;

int main() {
    char str1[20] = "Hello";
    char str2[] = " World";

    strcat(str1, str2);  // 将str2连接到str1

    cout << "Concatenated string: " << str1 << endl;

    return 0;
}
```

**代码解析**：
- `strcat(str1, str2)` 将 `str2` 连接到 `str1` 的末尾。注意，`str1` 必须有足够的空间来存储连接后的字符串。

#### 2.2.3 字符串比较

C风格字符串的比较可以使用 `strcmp` 函数。

```cpp
#include <iostream>
#include <cstring>  // 包含strcmp函数的头文件
using namespace std;

int main() {
    char str1[] = "apple";
    char str2[] = "banana";

    int result = strcmp(str1, str2);  // 比较两个字符串

    if (result == 0) {
        cout << "Strings are equal" << endl;
    } else if (result < 0) {
        cout << "str1 is less than str2" << endl;
    } else {
        cout << "str1 is greater than str2" << endl;
    }

    return 0;
}
```

**代码解析**：
- `strcmp(str1, str2)` 返回一个整数，表示两个字符串的比较结果：
  - `0`：字符串相等。
  - `<0`：`str1` 小于 `str2`。
  - `>0`：`str1` 大于 `str2`。

### 2.3 字符串的内存管理

C风格字符串的内存管理需要手动处理，容易出现内存泄漏或越界问题。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

int main() {
    char *str = new char[20];  // 动态分配内存
    strcpy(str, "Hello");

    cout << "Dynamic string: " << str << endl;

    delete[] str;  // 释放内存

    return 0;
}
```

**代码解析**：
- `new char[20]` 动态分配了一个大小为20的字符数组。
- `strcpy(str, "Hello")` 将字符串复制到动态分配的内存中。
- `delete[] str` 释放动态分配的内存，避免内存泄漏。



### 2.4 字符串和字符数组的区别和联系

在C++中，字符串和字符数组是两个非常相似但又有所不同的概念。理解它们的区别和联系对于正确使用C风格字符串非常重要。

#### 2.4.1 字符串和字符数组的定义

- **字符数组**：字符数组是一个固定大小的数组，用于存储字符序列。它可以存储任意类型的字符数据，包括字母、数字、符号等。字符数组的大小在定义时必须明确指定。
- **字符串**：字符串是字符数组的一种特殊形式，它以空字符（`'\0'`）结尾。字符串通常用于存储文本数据，并且可以通过标准库函数进行操作。

#### 2.4.2 字符串和字符数组的区别

1. **结尾标志**：
   - 字符串必须以空字符（`'\0'`）结尾，这是字符串的标志。
   - 字符数组可以不以空字符结尾，它可以存储任意字符序列。

2. **操作方式**：
   - 字符串可以使用标准库函数（如 `strlen`、`strcpy`、`strcat` 等）进行操作。
   - 字符数组只能通过数组索引或指针操作，不能直接使用字符串操作函数。

3. **内存管理**：
   - 字符串的内存管理需要特别注意，必须确保有足够的空间存储空字符。
   - 字符数组的内存管理相对简单，但仍然需要注意数组越界问题。

#### 2.4.3 字符串和字符数组的联系

- 字符串本质上是一个以空字符结尾的字符数组。因此，字符串可以看作是字符数组的一种特殊形式。
- 字符数组可以通过添加空字符（`'\0'`）来转换为字符串，从而可以使用字符串操作函数。

#### 2.4.4 代码示例

以下代码展示了字符串和字符数组的区别和联系：

```cpp
#include <iostream>
#include <cstring>  // 包含字符串操作函数的头文件
using namespace std;

int main() {
    // 字符数组的定义
    char charArray[5] = {'H', 'e', 'l', 'l', 'o'};  // 没有空字符结尾

    // 字符串的定义
    char str[6] = "Hello";  // 自动添加空字符结尾

    // 输出字符数组
    cout << "Char Array: ";
    for (int i = 0; i < 5; i++) {
        cout << charArray[i];
    }
    cout << endl;

    // 输出字符串
    cout << "String: " << str << endl;

    // 字符数组转换为字符串
    charArray[5] = '\0';  // 手动添加空字符
    cout << "Char Array as String: " << charArray << endl;

    // 使用字符串操作函数
    int length = strlen(str);  // 获取字符串长度
    cout << "Length of string: " << length << endl;

    return 0;
}
```

**代码解析**：
1. `charArray` 是一个字符数组，没有空字符结尾，因此不能直接使用字符串操作函数。
2. `str` 是一个字符串，以空字符结尾，可以使用字符串操作函数。
3. 通过手动添加空字符（`'\0'`），可以将字符数组 `charArray` 转换为字符串，从而可以使用字符串操作函数。
4. `strlen(str)` 获取字符串的长度，不包括空字符。

### 2.4.5 总结

- 字符串是字符数组的一种特殊形式，以空字符结尾。
- 字符数组可以转换为字符串，通过添加空字符（`'\0'`）。
- 字符串可以使用标准库函数进行操作，而字符数组只能通过数组索引或指针操作。

---

## 3. C++标准库字符串 (`std::string`)

### 3.1 定义和初始化

`std::string` 是C++标准库提供的字符串类，使用起来更加方便和安全。

```cpp
#include <iostream>
#include <string>  // 包含std::string的头文件
using namespace std;

int main() {
    string str1 = "Hello";  // 直接初始化
    string str2("World");   // 构造函数初始化
    string str3(5, 'A');    // 初始化为5个'A'

    cout << "str1: " << str1 << endl;
    cout << "str2: " << str2 << endl;
    cout << "str3: " << str3 << endl;

    return 0;
}
```

**代码解析**：
- `string str1 = "Hello";` 直接初始化字符串。
- `string str2("World");` 使用构造函数初始化。
- `string str3(5, 'A');` 初始化为5个字符 `'A'`。

### 3.2 字符串操作

#### 3.2.1 字符串长度和容量

`std::string` 提供了 `length()` 和 `capacity()` 方法。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string str = "Hello";

    cout << "Length: " << str.length() << endl;
    cout << "Capacity: " << str.capacity() << endl;

    return 0;
}
```

**代码解析**：
- `str.length()` 返回字符串的实际长度。
- `str.capacity()` 返回字符串当前分配的内存大小。

#### 3.2.2 字符串连接和追加

`std::string` 提供了 `+` 运算符和 `append()` 方法。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string str1 = "Hello";
    string str2 = " World";

    string str3 = str1 + str2;  // 使用+运算符连接
    str1.append(str2);          // 使用append方法追加

    cout << "str3: " << str3 << endl;
    cout << "str1 after append: " << str1 << endl;

    return 0;
}
```

**代码解析**：
- `str1 + str2` 使用 `+` 运算符连接两个字符串。
- `str1.append(str2)` 将 `str2` 追加到 `str1` 的末尾。

#### 3.2.3 字符串比较

`std::string` 提供了 `compare()` 方法和 `==` 运算符。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string str1 = "apple";
    string str2 = "banana";

    if (str1 == str2) {
        cout << "Strings are equal" << endl;
    } else if (str1.compare(str2) < 0) {
        cout << "str1 is less than str2" << endl;
    } else {
        cout << "str1 is greater than str2" << endl;
    }

    return 0;
}
```

**代码解析**：
- `str1 == str2` 使用 `==` 运算符比较字符串。
- `str1.compare(str2)` 返回一个整数，表示比较结果。

#### 3.2.4 字符串查找和替换

`std::string` 提供了 `find()` 和 `replace()` 方法。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string str = "Hello World";

    size_t pos = str.find("World");  // 查找子字符串
    if (pos != string::npos) {
        str.replace(pos, 5, "C++");  // 替换子字符串
    }

    cout << "Modified string: " << str << endl;

    return 0;
}
```

**代码解析**：
- `str.find("World")` 查找子字符串 `"World"` 的位置。
- `str.replace(pos, 5, "C++")` 将 `"World"` 替换为 `"C++"`。

#### 3.2.5 字符串分割和子串

`std::string` 提供了 `substr()` 方法。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string str = "Hello World";

    string sub = str.substr(6, 5);  // 获取子字符串

    cout << "Substring: " << sub << endl;

    return 0;
}
```

**代码解析**：
- `str.substr(6, 5)` 从位置6开始，获取长度为5的子字符串。

### 3.3 字符串的输入输出

`std::string` 可以直接使用 `cin` 和 `cout` 进行输入输出。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string str;

    cout << "Enter a string: ";
    cin >> str;

    cout << "You entered: " << str << endl;

    return 0;
}
```

**代码解析**：
- `cin >> str` 从标准输入读取字符串。
- `cout << str` 输出字符串到标准输出。

---

## 4. C风格字符串与`std::string`的对比

### 4.1 内存管理的差异

C风格字符串需要手动管理内存，而 `std::string` 自动处理内存分配和释放。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    // C风格字符串
    char *cstr = new char[20];
    strcpy(cstr, "Hello");
    delete[] cstr;

    // std::string
    string str = "Hello";  // 自动管理内存

    return 0;
}
```

### 4.2 操作的便捷性

`std::string` 提供了丰富的成员函数，操作更加便捷。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string str = "Hello";
    str += " World";  // 直接使用+=运算符

    cout << str << endl;

    return 0;
}
```

### 4.3 性能考虑

在某些情况下，C风格字符串的性能可能更高，但需要权衡内存管理和安全性。

---

## 5. 高级字符串操作

### 5.1 正则表达式

C++11引入了正则表达式库 `std::regex`。

```cpp
#include <iostream>
#include <regex>
#include <string>
using namespace std;

int main() {
    string str = "Hello 123 World";
    regex pattern("\\d+");  // 匹配数字

    smatch match;
    if (regex_search(str, match, pattern)) {
        cout << "Matched: " << match.str() << endl;
    }

    return 0;
}
```

### 5.2 字符串转换

#### 5.2.1 字符串转数字

```cpp
#include <iostream>
#include <string>
#include <sstream>
using namespace std;

int main() {
    string str = "123";
    int num;

    stringstream(str) >> num;  // 字符串转数字

    cout << "Number: " << num << endl;

    return 0;
}
```

#### 5.2.2 数字转字符串

```cpp
#include <iostream>
#include <string>
#include <sstream>
using namespace std;

int main() {
    int num = 123;
    string str;

    stringstream ss;
    ss << num;
    str = ss.str();  // 数字转字符串

    cout << "String: " << str << endl;

    return 0;
}
```

### 5.3 国际化和本地化

C++提供了 `std::locale` 和 `std::wstring` 来处理国际化和本地化。

---

## 6. 常见错误和陷阱

### 6.1 字符串越界

```cpp
#include <iostream>
using namespace std;

int main() {
    char str[5] = "Hello";  // 越界错误
    cout << str << endl;

    return 0;
}
```

### 6.2 内存泄漏

```cpp
#include <iostream>
using namespace std;

int main() {
    char *str = new char[20];
    // 忘记释放内存

    return 0;
}
```

### 6.3 字符编码问题

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string str = "你好";  // 默认使用UTF-8编码
    cout << str << endl;

    return 0;
}
```

---

## 7. 总结

本文详细介绍了C++字符串的两种主要类型：C风格字符串和`std::string`，并通过代码示例展示了它们的定义、初始化、操作和常见问题。掌握这些知识，能够帮助我们更好地处理字符串数据，提升编程效率。

**推荐进一步学习的资源**：
- C++ Primer
- Effective C++
- C++标准库文档

文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)