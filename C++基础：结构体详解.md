---
title: 'C++ 结构体详解'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
  - c++基础
---



# C++ 结构体详解：从基础到进阶

## 引言

在C++编程中，结构体（Struct）是一个非常重要的概念。它允许我们将不同类型的数据组合在一起，形成一个新的数据类型。结构体在处理复杂数据时非常有用，比如表示一个学生的信息、一个点的坐标、或者一个复杂的数据记录。本文将从基础开始，逐步深入，带你全面了解C++中的结构体。

## 1. 什么是结构体？

### 1.1 结构体的定义

结构体是一种用户自定义的数据类型，它允许我们将多个不同类型的数据组合在一起。结构体的定义使用关键字 `struct`，后面跟着结构体的名称，然后在大括号内定义结构体的成员变量。

```cpp
struct Student {
    string name;  // 学生姓名
    int age;      // 学生年龄
    float gpa;    // 学生平均成绩
};
```

在这个例子中，我们定义了一个名为 `Student` 的结构体，它有三个成员变量：`name`、`age` 和 `gpa`。

### 1.2 结构体的使用

定义了结构体之后，我们可以像使用其他数据类型一样使用它。例如，我们可以声明一个 `Student` 类型的变量，并为其成员变量赋值。

```cpp
int main() {
    Student s1;  // 声明一个 Student 类型的变量 s1
    s1.name = "Alice";  // 为 s1 的 name 成员赋值
    s1.age = 20;        // 为 s1 的 age 成员赋值
    s1.gpa = 3.8;       // 为 s1 的 gpa 成员赋值

    cout << "Name: " << s1.name << ", Age: " << s1.age << ", GPA: " << s1.gpa << endl;
    return 0;
}
```

在这个例子中，我们声明了一个 `Student` 类型的变量 `s1`，并为其成员变量赋值。然后，我们通过 `.` 操作符访问结构体的成员变量，并输出它们的值。

### 1.3 结构体的内存布局

结构体的成员变量在内存中是连续存储的。例如，上面的 `Student` 结构体在内存中的布局如下：

```
| name (string) | age (int) | gpa (float) |
```

每个成员变量占据一定的内存空间，具体大小取决于变量的类型。例如，`string` 类型的大小取决于字符串的长度，`int` 类型通常占据 4 个字节，`float` 类型通常占据 4 个字节。

## 2. 结构体的初始化与赋值

### 2.1 结构体的初始化

结构体可以在声明时进行初始化。初始化的方式有多种，最常见的是使用大括号 `{}` 进行初始化。

```cpp
int main() {
    Student s1 = {"Alice", 20, 3.8};  // 初始化结构体变量 s1
    cout << "Name: " << s1.name << ", Age: " << s1.age << ", GPA: " << s1.gpa << endl;
    return 0;
}
```

在这个例子中，我们使用大括号 `{}` 对 `s1` 进行初始化，依次为 `name`、`age` 和 `gpa` 赋值。

### 2.2 结构体的赋值

结构体变量之间可以直接进行赋值操作。赋值操作会将一个结构体变量的所有成员变量的值复制到另一个结构体变量中。

```cpp
int main() {
    Student s1 = {"Alice", 20, 3.8};
    Student s2;
    s2 = s1;  // 将 s1 的值赋给 s2
    cout << "Name: " << s2.name << ", Age: " << s2.age << ", GPA: " << s2.gpa << endl;
    return 0;
}
```

在这个例子中，我们将 `s1` 的值赋给了 `s2`，`s2` 的成员变量将具有与 `s1` 相同的值。

## 3. 结构体的嵌套

### 3.1 嵌套结构体的定义

结构体可以嵌套定义，即一个结构体的成员变量可以是另一个结构体。例如，我们可以定义一个 `Address` 结构体，然后在 `Student` 结构体中嵌套使用它。

```cpp
struct Address {
    string city;  // 城市
    string street; // 街道
    int houseNumber; // 门牌号
};

struct Student {
    string name;  // 学生姓名
    int age;      // 学生年龄
    float gpa;    // 学生平均成绩
    Address addr; // 学生地址
};
```

在这个例子中，`Student` 结构体中嵌套了一个 `Address` 结构体，用于表示学生的地址信息。

### 3.2 嵌套结构体的使用

嵌套结构体的使用与普通结构体类似，只需通过多个 `.` 操作符来访问嵌套的成员变量。

```cpp
int main() {
    Student s1 = {"Alice", 20, 3.8, {"New York", "5th Avenue", 123}};
    cout << "Name: " << s1.name << ", Age: " << s1.age << ", GPA: " << s1.gpa << endl;
    cout << "Address: " << s1.addr.city << ", " << s1.addr.street << ", " << s1.addr.houseNumber << endl;
    return 0;
}
```

在这个例子中，我们初始化了 `s1` 的 `addr` 成员变量，并通过 `.` 操作符访问嵌套的成员变量。

## 4. 结构体与函数

### 4.1 结构体作为函数参数

结构体可以作为函数的参数传递。当结构体作为参数传递时，通常是按值传递，即函数会创建结构体的一个副本。

```cpp
void printStudent(Student s) {
    cout << "Name: " << s.name << ", Age: " << s.age << ", GPA: " << s.gpa << endl;
}

int main() {
    Student s1 = {"Alice", 20, 3.8};
    printStudent(s1);  // 将 s1 作为参数传递给 printStudent 函数
    return 0;
}
```

在这个例子中，`printStudent` 函数接受一个 `Student` 类型的参数 `s`，并输出其成员变量的值。

### 4.2 结构体作为函数返回值

结构体也可以作为函数的返回值。函数可以返回一个结构体变量，调用者可以接收这个返回值。

```cpp
Student createStudent(string name, int age, float gpa) {
    Student s;
    s.name = name;
    s.age = age;
    s.gpa = gpa;
    return s;  // 返回一个 Student 类型的结构体
}

int main() {
    Student s1 = createStudent("Bob", 22, 3.5);
    printStudent(s1);  // 输出 s1 的信息
    return 0;
}
```

在这个例子中，`createStudent` 函数返回一个 `Student` 类型的结构体，`main` 函数接收这个返回值并输出其信息。

## 5. 结构体与指针

### 5.1 结构体指针

结构体指针是指向结构体变量的指针。我们可以使用 `->` 操作符来访问结构体指针所指向的成员变量。

```cpp
int main() {
    Student s1 = {"Alice", 20, 3.8};
    Student* ptr = &s1;  // 声明一个指向 s1 的指针
    cout << "Name: " << ptr->name << ", Age: " << ptr->age << ", GPA: " << ptr->gpa << endl;
    return 0;
}
```

在这个例子中，我们声明了一个指向 `s1` 的指针 `ptr`，并通过 `->` 操作符访问 `s1` 的成员变量。

### 5.2 动态分配结构体

我们可以使用 `new` 关键字动态分配结构体，并返回一个指向该结构体的指针。

```cpp
int main() {
    Student* ptr = new Student{"Alice", 20, 3.8};  // 动态分配一个 Student 结构体
    cout << "Name: " << ptr->name << ", Age: " << ptr->age << ", GPA: " << ptr->gpa << endl;
    delete ptr;  // 释放动态分配的内存
    return 0;
}
```

在这个例子中，我们使用 `new` 关键字动态分配了一个 `Student` 结构体，并返回一个指向该结构体的指针 `ptr`。最后，我们使用 `delete` 关键字释放了动态分配的内存。

## 6. 结构体与类

在C++中，结构体和类非常相似。事实上，结构体可以看作是类的简化版本。主要的区别在于：

- 结构体的成员变量默认是 `public` 的，而类的成员变量默认是 `private` 的。
- 结构体通常用于表示简单的数据结构，而类通常用于表示更复杂的数据和行为。

```cpp
struct Point {
    int x;  // 默认是 public
    int y;  // 默认是 public
};

class Circle {
    int radius;  // 默认是 private
public:
    void setRadius(int r) { radius = r; }
    int getRadius() { return radius; }
};
```

在这个例子中，`Point` 是一个结构体，其成员变量 `x` 和 `y` 默认是 `public` 的。而 `Circle` 是一个类，其成员变量 `radius` 默认是 `private` 的，我们通过 `setRadius` 和 `getRadius` 方法来访问它。

## 7. 总结

结构体是C++中非常重要的一个概念，它允许我们将不同类型的数据组合在一起，形成一个新的数据类型。通过本文的讲解，你应该已经掌握了结构体的基础知识，包括定义、初始化、赋值、嵌套、与函数的关系、指针的使用等。结构体在实际编程中非常有用，尤其是在处理复杂数据时。希望本文能帮助你更好地理解和使用C++中的结构体。

文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)