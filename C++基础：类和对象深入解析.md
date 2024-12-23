---
title: 'C++类和对象深入解析'
categories:
  - [开发,cpp]
tags:
  - c++
  - cpp
  - c++基础
---



## C++类和对象深入解析

#### 1. 引言

在C++中，类（Class）和对象（Object）是面向对象编程（OOP）的核心概念。类是用户定义的数据类型，它封装了数据和操作数据的函数。对象是类的实例，通过对象可以访问类的成员。

面向对象编程的三大特性是封装、继承和多态。类和对象是实现这些特性的基础。通过类，我们可以将数据和操作数据的函数封装在一起，通过对象来实例化和使用这些数据和函数。

#### 2. 类的基本概念

##### 2.1 类的定义

类的定义使用关键字 `class`，后面跟着类的名称。类的主体包含在大括号 `{}` 中，并以分号 `;` 结尾。

```cpp
#include <iostream>

class MyClass {
public:
    int myData;  // 数据成员
    void myFunction() {  // 成员函数
        std::cout << "Hello from MyClass!" << std::endl;
    }
};

int main() {
    MyClass obj;  // 创建对象
    obj.myFunction();  // 调用成员函数
    return 0;
}
```

**代码解析：**
- `class MyClass`：定义了一个名为 `MyClass` 的类。
- `int myData`：类的数据成员。
- `void myFunction()`：类的成员函数，用于输出一条消息。
- `MyClass obj`：在 `main` 函数中创建了一个 `MyClass` 的对象 `obj`。
- `obj.myFunction()`：通过对象 `obj` 调用成员函数 `myFunction`。

##### 2.2 类的成员

###### 2.2.1 数据成员

数据成员是类的属性，它们存储对象的状态。

```cpp
class MyClass {
public:
    int myData;  // 数据成员
};

int main() {
    MyClass obj;
    obj.myData = 10;  // 访问数据成员
    std::cout << "Data: " << obj.myData << std::endl;
    return 0;
}
```

**代码解析：**
- `int myData`：定义了一个整型数据成员 `myData`。
- `obj.myData = 10`：通过对象 `obj` 访问并修改数据成员 `myData`。

###### 2.2.2 成员函数

成员函数是类的行为，它们操作类的数据成员。

```cpp
class MyClass {
public:
    int myData;
    void setData(int data) {  // 成员函数
        myData = data;
    }
    void displayData() {
        std::cout << "Data: " << myData << std::endl;
    }
};

int main() {
    MyClass obj;
    obj.setData(20);  // 调用成员函数设置数据
    obj.displayData();  // 调用成员函数显示数据
    return 0;
}
```

**代码解析：**
- `void setData(int data)`：定义了一个成员函数 `setData`，用于设置数据成员 `myData`。
- `void displayData()`：定义了一个成员函数 `displayData`，用于显示数据成员 `myData`。

##### 2.3 访问控制

C++ 提供了三种访问控制符：`public`、`private` 和 `protected`。

- `public`：成员可以被类的外部访问。
- `private`：成员只能在类的内部访问。
- `protected`：成员可以在类的内部和派生类中访问。

```cpp
class MyClass {
public:
    int publicData;
    void publicFunction() {
        std::cout << "Public Function" << std::endl;
    }

private:
    int privateData;
    void privateFunction() {
        std::cout << "Private Function" << std::endl;
    }

protected:
    int protectedData;
    void protectedFunction() {
        std::cout << "Protected Function" << std::endl;
    }
};

int main() {
    MyClass obj;
    obj.publicData = 10;  // 可以访问
    obj.publicFunction();  // 可以调用

    // obj.privateData = 20;  // 错误，不能访问
    // obj.privateFunction();  // 错误，不能调用

    // obj.protectedData = 30;  // 错误，不能访问
    // obj.protectedFunction();  // 错误，不能调用

    return 0;
}
```

**代码解析：**
- `publicData` 和 `publicFunction` 可以被类的外部访问。
- `privateData` 和 `privateFunction` 只能在类的内部访问。
- `protectedData` 和 `protectedFunction` 可以在类的内部和派生类中访问。

#### 3. 对象的创建与使用

##### 3.1 对象的声明与实例化

对象的声明和实例化可以通过以下方式进行：

```cpp
class MyClass {
public:
    int myData;
};

int main() {
    MyClass obj;  // 声明并实例化对象
    obj.myData = 10;  // 访问数据成员
    std::cout << "Data: " << obj.myData << std::endl;
    return 0;
}
```

**代码解析：**
- `MyClass obj`：声明并实例化了一个 `MyClass` 的对象 `obj`。

##### 3.2 对象的成员访问

对象的成员可以通过点运算符 `.` 进行访问。

```cpp
class MyClass {
public:
    int myData;
    void setData(int data) {
        myData = data;
    }
    void displayData() {
        std::cout << "Data: " << myData << std::endl;
    }
};

int main() {
    MyClass obj;
    obj.setData(20);  // 调用成员函数设置数据
    obj.displayData();  // 调用成员函数显示数据
    return 0;
}
```

**代码解析：**
- `obj.setData(20)`：通过对象 `obj` 调用成员函数 `setData`。
- `obj.displayData()`：通过对象 `obj` 调用成员函数 `displayData`。

##### 3.3 对象的初始化

###### 3.3.1 构造函数

构造函数是类的特殊成员函数，用于初始化对象。

```cpp
class MyClass {
public:
    int myData;
    MyClass() {  // 构造函数
        myData = 0;
        std::cout << "Constructor called" << std::endl;
    }
};

int main() {
    MyClass obj;  // 调用构造函数
    std::cout << "Data: " << obj.myData << std::endl;
    return 0;
}
```

**代码解析：**
- `MyClass()`：定义了一个构造函数，用于初始化 `myData` 并输出一条消息。

###### 3.3.2 默认构造函数

如果没有显式定义构造函数，编译器会自动生成一个默认构造函数。

```cpp
class MyClass {
public:
    int myData;
};

int main() {
    MyClass obj;  // 调用默认构造函数
    std::cout << "Data: " << obj.myData << std::endl;
    return 0;
}
```

**代码解析：**
- `MyClass obj`：调用默认构造函数。

###### 3.3.3 带参数的构造函数

构造函数可以带有参数，用于初始化对象时传递参数。

```cpp
class MyClass {
public:
    int myData;
    MyClass(int data) {  // 带参数的构造函数
        myData = data;
        std::cout << "Parameterized Constructor called" << std::endl;
    }
};

int main() {
    MyClass obj(10);  // 调用带参数的构造函数
    std::cout << "Data: " << obj.myData << std::endl;
    return 0;
}
```

**代码解析：**
- `MyClass(int data)`：定义了一个带参数的构造函数，用于初始化 `myData`。

###### 3.3.4 拷贝构造函数

拷贝构造函数用于创建对象的副本。

```cpp
class MyClass {
public:
    int myData;
    MyClass(int data) : myData(data) {  // 带参数的构造函数
        std::cout << "Parameterized Constructor called" << std::endl;
    }
    MyClass(const MyClass &obj) : myData(obj.myData) {  // 拷贝构造函数
        std::cout << "Copy Constructor called" << std::endl;
    }
};

int main() {
    MyClass obj1(10);  // 调用带参数的构造函数
    MyClass obj2 = obj1;  // 调用拷贝构造函数
    std::cout << "Data in obj2: " << obj2.myData << std::endl;
    return 0;
}
```

**代码解析：**
- `MyClass(const MyClass &obj)`：定义了一个拷贝构造函数，用于创建对象的副本。

##### 3.4 对象的销毁

析构函数是类的特殊成员函数，用于在对象销毁时执行清理操作。

```cpp
class MyClass {
public:
    ~MyClass() {  // 析构函数
        std::cout << "Destructor called" << std::endl;
    }
};

int main() {
    MyClass obj;  // 创建对象
    return 0;  // 对象销毁时调用析构函数
}
```

**代码解析：**
- `~MyClass()`：定义了一个析构函数，用于在对象销毁时输出一条消息。

#### 4. 类的继承

##### 4.1 继承的基本概念

继承允许一个类继承另一个类的属性和行为。

```cpp
class Base {
public:
    int baseData;
    void baseFunction() {
        std::cout << "Base Function" << std::endl;
    }
};

class Derived : public Base {
public:
    int derivedData;
    void derivedFunction() {
        std::cout << "Derived Function" << std::endl;
    }
};

int main() {
    Derived obj;
    obj.baseData = 10;  // 访问基类的数据成员
    obj.baseFunction();  // 调用基类的成员函数
    obj.derivedData = 20;  // 访问派生类的数据成员
    obj.derivedFunction();  // 调用派生类的成员函数
    return 0;
}
```

**代码解析：**
- `class Derived : public Base`：定义了一个派生类 `Derived`，继承自基类 `Base`。
- `obj.baseData` 和 `obj.baseFunction`：通过派生类对象 `obj` 访问基类的成员。

##### 4.2 继承的类型

C++ 支持三种继承类型：公有继承、私有继承和保护继承。

###### 4.2.1 公有继承

```cpp
class Base {
public:
    int baseData;
    void baseFunction() {
        std::cout << "Base Function" << std::endl;
    }
};

class Derived : public Base {
public:
    int derivedData;
    void derivedFunction() {
        std::cout << "Derived Function" << std::endl;
    }
};

int main() {
    Derived obj;
    obj.baseData = 10;  // 可以访问
    obj.baseFunction();  // 可以调用
    return 0;
}
```

**代码解析：**
- `class Derived : public Base`：公有继承，基类的公有成员在派生类中仍然是公有的。

###### 4.2.2 私有继承

```cpp
class Base {
public:
    int baseData;
    void baseFunction() {
        std::cout << "Base Function" << std::endl;
    }
};

class Derived : private Base {
public:
    int derivedData;
    void derivedFunction() {
        std::cout << "Derived Function" << std::endl;
    }
};

int main() {
    Derived obj;
    // obj.baseData = 10;  // 错误，不能访问
    // obj.baseFunction();  // 错误，不能调用
    return 0;
}
```

**代码解析：**
- `class Derived : private Base`：私有继承，基类的公有成员在派生类中变为私有的。

###### 4.2.3 保护继承

```cpp
class Base {
public:
    int baseData;
    void baseFunction() {
        std::cout << "Base Function" << std::endl;
    }
};

class Derived : protected Base {
public:
    int derivedData;
    void derivedFunction() {
        std::cout << "Derived Function" << std::endl;
    }
};

int main() {
    Derived obj;
    // obj.baseData = 10;  // 错误，不能访问
    // obj.baseFunction();  // 错误，不能调用
    return 0;
}
```

**代码解析：**
- `class Derived : protected Base`：保护继承，基类的公有成员在派生类中变为保护的。

##### 4.3 继承中的构造函数与析构函数

派生类的构造函数和析构函数会调用基类的构造函数和析构函数。

```cpp
class Base {
public:
    Base() {
        std::cout << "Base Constructor" << std::endl;
    }
    ~Base() {
        std::cout << "Base Destructor" << std::endl;
    }
};

class Derived : public Base {
public:
    Derived() {
        std::cout << "Derived Constructor" << std::endl;
    }
    ~Derived() {
        std::cout << "Derived Destructor" << std::endl;
    }
};

int main() {
    Derived obj;
    return 0;
}
```

**代码解析：**
- `Derived obj`：创建派生类对象时，首先调用基类的构造函数，然后调用派生类的构造函数。
- 对象销毁时，首先调用派生类的析构函数，然后调用基类的析构函数。

##### 4.4 继承中的访问控制

继承中的访问控制取决于继承的类型。

```cpp
class Base {
public:
    int publicData;
private:
    int privateData;
protected:
    int protectedData;
};

class Derived : public Base {
public:
    void accessBaseMembers() {
        publicData = 10;  // 可以访问
        // privateData = 20;  // 错误，不能访问
        protectedData = 30;  // 可以访问
    }
};

int main() {
    Derived obj;
    obj.accessBaseMembers();
    return 0;
}
```

**代码解析：**
- `publicData`：在公有继承中，基类的公有成员在派生类中仍然是公有的。
- `privateData`：基类的私有成员在派生类中不能访问。
- `protectedData`：在公有继承中，基类的保护成员在派生类中仍然是保护的。

#### 5. 多态性

##### 5.1 多态的基本概念

多态性允许使用基类指针或引用来调用派生类的成员函数。

```cpp
class Base {
public:
    virtual void display() {
        std::cout << "Base Class" << std::endl;
    }
};

class Derived : public Base {
public:
    void display() override {
        std::cout << "Derived Class" << std::endl;
    }
};

int main() {
    Base *basePtr;
    Derived derivedObj;
    basePtr = &derivedObj;
    basePtr->display();  // 调用派生类的成员函数
    return 0;
}
```

**代码解析：**
- `virtual void display()`：基类的虚函数，允许派生类重写。
- `void display() override`：派生类重写了基类的虚函数。
- `basePtr->display()`：通过基类指针调用派生类的成员函数。

##### 5.2 虚函数

虚函数是实现多态性的关键。

```cpp
class Base {
public:
    virtual void display() {
        std::cout << "Base Class" << std::endl;
    }
};

class Derived : public Base {
public:
    void display() override {
        std::cout << "Derived Class" << std::endl;
    }
};

int main() {
    Base *basePtr;
    Derived derivedObj;
    basePtr = &derivedObj;
    basePtr->display();  // 调用派生类的成员函数
    return 0;
}
```

**代码解析：**
- `virtual void display()`：基类的虚函数，允许派生类重写。
- `void display() override`：派生类重写了基类的虚函数。
- `basePtr->display()`：通过基类指针调用派生类的成员函数。

##### 5.3 纯虚函数与抽象类

纯虚函数是没有实现的虚函数，包含纯虚函数的类是抽象类。

```cpp
class Base {
public:
    virtual void display() = 0;  // 纯虚函数
};

class Derived : public Base {
public:
    void display() override {
        std::cout << "Derived Class" << std::endl;
    }
};

int main() {
    // Base baseObj;  // 错误，不能实例化抽象类
    Derived derivedObj;
    derivedObj.display();  // 调用派生类的成员函数
    return 0;
}
```

**代码解析：**
- `virtual void display() = 0`：定义了一个纯虚函数，使 `Base` 类成为抽象类。
- `void display() override`：派生类必须实现纯虚函数。

#### 6. 友元

##### 6.1 友元函数

友元函数可以访问类的私有和保护成员。

```cpp
class MyClass {
private:
    int privateData;
public:
    MyClass(int data) : privateData(data) {}
    friend void friendFunction(MyClass &obj);  // 友元函数声明
};

void friendFunction(MyClass &obj) {
    std::cout << "Private Data: " << obj.privateData << std::endl;  // 访问私有成员
}

int main() {
    MyClass obj(10);
    friendFunction(obj);  // 调用友元函数
    return 0;
}
```

**代码解析：**
- `friend void friendFunction(MyClass &obj)`：声明 `friendFunction` 为 `MyClass` 的友元函数。
- `friendFunction(MyClass &obj)`：友元函数可以访问 `MyClass` 的私有成员。

##### 6.2 友元类

友元类可以访问另一个类的私有和保护成员。

```cpp
class MyClass {
private:
    int privateData;
public:
    MyClass(int data) : privateData(data) {}
    friend class FriendClass;  // 友元类声明
};

class FriendClass {
public:
    void displayData(MyClass &obj) {
        std::cout << "Private Data: " << obj.privateData << std::endl;  // 访问私有成员
    }
};

int main() {
    MyClass obj(10);
    FriendClass friendObj;
    friendObj.displayData(obj);  // 调用友元类的成员函数
    return 0;
}
```

**代码解析：**
- `friend class FriendClass`：声明 `FriendClass` 为 `MyClass` 的友元类。
- `friendObj.displayData(obj)`：友元类的成员函数可以访问 `MyClass` 的私有成员。

#### 7. 静态成员

##### 7.1 静态数据成员

静态数据成员属于类，而不是类的对象。

```cpp
class MyClass {
public:
    static int staticData;  // 静态数据成员声明
    void setStaticData(int data) {
        staticData = data;
    }
    void displayStaticData() {
        std::cout << "Static Data: " << staticData << std::endl;
    }
};

int MyClass::staticData = 0;  // 静态数据成员定义

int main() {
    MyClass obj1, obj2;
    obj1.setStaticData(10);
    obj2.displayStaticData();  // 输出 10
    return 0;
}
```

**代码解析：**
- `static int staticData`：声明了一个静态数据成员。
- `int MyClass::staticData = 0`：定义并初始化静态数据成员。
- `obj1.setStaticData(10)`：通过对象 `obj1` 设置静态数据成员。
- `obj2.displayStaticData()`：通过对象 `obj2` 访问静态数据成员。

##### 7.2 静态成员函数

静态成员函数属于类，而不是类的对象。

```cpp
class MyClass {
public:
    static int staticData;  // 静态数据成员声明
    static void setStaticData(int data) {  // 静态成员函数
        staticData = data;
    }
    static void displayStaticData() {
        std::cout << "Static Data: " << staticData << std::endl;
    }
};

int MyClass::staticData = 0;  // 静态数据成员定义

int main() {
    MyClass::setStaticData(10);  // 调用静态成员函数
    MyClass::displayStaticData();  // 调用静态成员函数
    return 0;
}
```

**代码解析：**
- `static void setStaticData(int data)`：定义了一个静态成员函数，用于设置静态数据成员。
- `static void displayStaticData()`：定义了一个静态成员函数，用于显示静态数据成员。
- `MyClass::setStaticData(10)`：通过类名调用静态成员函数。

#### 8. 运算符重载

##### 8.1 运算符重载的基本概念

运算符重载允许用户定义的类型使用标准运算符。

```cpp
class MyClass {
public:
    int myData;
    MyClass(int data) : myData(data) {}
    MyClass operator+(const MyClass &obj) {  // 重载 + 运算符
        return MyClass(myData + obj.myData);
    }
};

int main() {
    MyClass obj1(10);
    MyClass obj2(20);
    MyClass obj3 = obj1 + obj2;  // 使用重载的 + 运算符
    std::cout << "Sum: " << obj3.myData << std::endl;
    return 0;
}
```

**代码解析：**
- `MyClass operator+(const MyClass &obj)`：重载了 `+` 运算符，用于两个 `MyClass` 对象的相加。
- `MyClass obj3 = obj1 + obj2`：使用重载的 `+` 运算符进行对象相加。

##### 8.2 常见运算符的重载

```cpp
class MyClass {
public:
    int myData;
    MyClass(int data) : myData(data) {}
    MyClass operator+(const MyClass &obj) {  // 重载 + 运算符
        return MyClass(myData + obj.myData);
    }
    MyClass operator-(const MyClass &obj) {  // 重载 - 运算符
        return MyClass(myData - obj.myData);
    }
    MyClass operator*(const MyClass &obj) {  // 重载 * 运算符
        return MyClass(myData * obj.myData);
    }
    MyClass operator/(const MyClass &obj) {  // 重载 / 运算符
        return MyClass(myData / obj.myData);
    }
};

int main() {
    MyClass obj1(10);
    MyClass obj2(20);
    MyClass obj3 = obj1 + obj2;  // 使用重载的 + 运算符
    MyClass obj4 = obj1 - obj2;  // 使用重载的 - 运算符
    MyClass obj5 = obj1 * obj2;  // 使用重载的 * 运算符
    MyClass obj6 = obj1 / obj2;  // 使用重载的 / 运算符
    std::cout << "Sum: " << obj3.myData << std::endl;
    std::cout << "Difference: " << obj4.myData << std::endl;
    std::cout << "Product: " << obj5.myData << std::endl;
    std::cout << "Quotient: " << obj6.myData << std::endl;
    return 0;
}
```

**代码解析：**
- `MyClass operator+(const MyClass &obj)`：重载了 `+` 运算符。
- `MyClass operator-(const MyClass &obj)`：重载了 `-` 运算符。
- `MyClass operator*(const MyClass &obj)`：重载了 `*` 运算符。
- `MyClass operator/(const MyClass &obj)`：重载了 `/` 运算符。

#### 9. 模板

##### 9.1 函数模板

函数模板允许定义通用的函数。

```cpp
template <typename T>
T add(T a, T b) {
    return a + b;
}

int main() {
    int result1 = add(10, 20);  // 使用 int 类型
    double result2 = add(10.5, 20.5);  // 使用 double 类型
    std::cout << "Result1: " << result1 << std::endl;
    std::cout << "Result2: " << result2 << std::endl;
    return 0;
}
```

**代码解析：**
- `template <typename T>`：定义了一个模板，`T` 是类型参数。
- `T add(T a, T b)`：定义了一个通用的加法函数。
- `add(10, 20)`：使用 `int` 类型调用模板函数。
- `add(10.5, 20.5)`：使用 `double` 类型调用模板函数。

##### 9.2 类模板

类模板允许定义通用的类。

```cpp
template <typename T>
class MyClass {
public:
    T myData;
    MyClass(T data) : myData(data) {}
    T add(T a, T b) {
        return a + b;
    }
};

int main() {
    MyClass<int> obj1(10);  // 使用 int 类型
    MyClass<double> obj2(10.5);  // 使用 double 类型
    std::cout << "Sum: " << obj1.add(10, 20) << std::endl;
    std::cout << "Sum: " << obj2.add(10.5, 20.5) << std::endl;
    return 0;
}
```

**代码解析：**
- `template <typename T>`：定义了一个类模板，`T` 是类型参数。
- `MyClass<int> obj1(10)`：使用 `int` 类型实例化类模板。
- `MyClass<double> obj2(10.5)`：使用 `double` 类型实例化类模板。

#### 10. 异常处理

##### 10.1 异常处理的基本概念

异常处理用于处理程序中的错误。

```cpp
#include <iostream>
#include <stdexcept>

int divide(int a, int b) {
    if (b == 0) {
        throw std::runtime_error("Division by zero");
    }
    return a / b;
}

int main() {
    try {
        int result = divide(10, 0);
        std::cout << "Result: " << result << std::endl;
    } catch (const std::exception &e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

**代码解析：**
- `throw std::runtime_error("Division by zero")`：抛出一个异常。
- `try { ... } catch (const std::exception &e) { ... }`：捕获并处理异常。

##### 10.2 异常类的定义与使用

可以自定义异常类来处理特定的错误。

```cpp
#include <iostream>
#include <stdexcept>

class MyException : public std::exception {
public:
    const char* what() const noexcept override {
        return "My Custom Exception";
    }
};

int divide(int a, int b) {
    if (b == 0) {
        throw MyException();
    }
    return a / b;
}

int main() {
    try {
        int result = divide(10, 0);
        std::cout << "Result: " << result << std::endl;
    } catch (const MyException &e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

**代码解析：**
- `class MyException : public std::exception`：定义了一个自定义异常类。
- `throw MyException()`：抛出自定义异常。
- `catch (const MyException &e)`：捕获并处理自定义异常。

#### 11. 总结

C++ 中的类和对象是面向对象编程的核心概念。通过类，我们可以封装数据和操作数据的函数，通过对象来实例化和使用这些数据和函数。类和对象在实际编程中广泛应用于各种场景，如游戏开发、图形界面、网络编程等。

通过本文的学习，你应该掌握了 C++ 中类和对象的基本概念、构造函数和析构函数、继承、多态性、友元、静态成员、运算符重载、模板和异常处理等内容。

文章合集：[chongzicbo/ReadWriteThink: 博学而笃志，切问而近思 (github.com)](https://github.com/chongzicbo/ReadWriteThink/tree/main)

个人博客：[程博仕](https://chongzicbo.github.io/)

微信公众号：

![微信公众号](https://raw.githubusercontent.com/chongzicbo/images/main/picgo/%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)