---
title: C++
date: 2026-03-09 15:49:51
tags:
---

**访问说明符**

| 访问位置               | `public` | `protected` | `private`  |
| :--------------------- | :------- | :---------- | :--------- |
| **本类内部**           | ✅ 可访问 | ✅ 可访问    | ✅ 可访问   |
| **派生类内部**         | ✅ 可访问 | ✅ 可访问    | ❌ 不可访问 |
| **类外部（通过对象）** | ✅ 可访问 | ❌ 不可访问  | ❌ 不可访问 |

在C++的`class`中，如果不写访问说明符，**默认是 `private`**，即第一个访问说明符出现之前的所有成员，**默认是`private`**



在C++的`class`里，**变量和函数都可以放**：

```
class Student {
public:
    // 变量（属性）
    char name[20];
    int age;
    
    // 函数（方法）- 这是C++新增的！
    void introduce() {
        cout << "我叫" << name << "，今年" << age << "岁";
    }
};
```



**运算符重载**：让已有的运算符（如 `+`、`-`、`*`、`/`、`=`、`<<` 等）能够**对自定义类型（类对象）进行运算**。

**本质**：是一种特殊的**函数重载**

运算符重载的语法格式：

```
返回值类型 operator 运算符 (参数列表) {
    // 实现运算逻辑
}
```

例如：

```
class Complex {//类似于创建Complex结构体
public://表示公有，外部可访问
    double real, imag;
    
    // 重载 + 运算符
    Complex operator+(const Complex& other) {//在结构体内重载加法函数，Complex为返回类型，operator+为函数名
        Complex c;
        c.real = this->real + other.real;//this指向c1，this->real就是c1.real
        c.imag = this->imag + other.imag;
        return c;
    }
};

// 调用时
Complex c1, c2;
Complex c3 = c1 + c2; //c1是调用者（左操作数），c2是参数other
```

a + b 等价于 a.operator+(b)



**引用是什么？**

```
int a = 5;
int &b = a;   // b是a的引用（别名）
b = 10;       // 改b就是改a，现在a也是10
```

- `int &b` 表示 `ref` 是一个 `int` 类型的引用。
- `b` 是 `a` 的别名，对 `b` 的操作会直接作用于 `a`（引用不占用额外内存，编译器直接操作所引用的对象）。
- 一旦引用被初始化为一个对象，就不能被指向到另一个对象。
- 引用必须在创建时被初始化，不能为 `null`。
- 引用的对象必须是一个变量。
- 不支持多级间接访问（不能有引用的引用）。

可以创建数组的引用，例如：`int (&ref)[10] = arr;`



**this的核心概念**：

- `this`是一个**指针**，指向**当前对象自己**
- 哪个对象调用了这个函数，`this`就指向谁（左操作数为调用者）



`::` 叫**作用域解析运算符**，含义：“的”

**场景1：在类外面定义成员函数**

```
class Complex {
public:
    double real, imag;
    
    // 方式1：直接在类里面写完整函数（简短时用）
    void print() {
        cout << real << "+" << imag << "i";
    }
    
    // 方式2：只声明，后面再定义（函数长时用）
    Complex operator+(const Complex& other);
};

// 在类外面定义operator+函数
// 👇 这里必须用::告诉编译器：这个operator+是Complex类的
Complex Complex::operator+(const Complex& other) {
    Complex c;
    c.real = this->real + other.real;
    c.imag = this->imag + other.imag;
    return c;
}
```



**场景2：访问被隐藏的同名成员**

```
class A {
public:
    int value = 10;
    void func() { cout << "A::func" << endl; }
};

class B : public A {  // B继承A
public:
    int value = 20;   // 隐藏了A的value
    void func() {     // 隐藏了A的func
        cout << "B::func" << endl;
        
        // 想访问A的value怎么办？
        cout << A::value;  // 👈 用::指定是A的value
        
        // 想调用A的func怎么办？
        A::func();  // 👈 用::指定是A的func
    }
};
```



**场景3：访问静态成员**

静态成员属于类，不属于某个对象，所以用 `类名::` 访问：

```
class Math {
public:
    static double PI;  // 静态成员变量
    static double square(double x) { return x * x; }  // 静态函数
};

// 静态成员需要在类外面初始化
double Math::PI = 3.14159;

int main() {
    // 直接通过类名访问，不需要创建对象
    cout << Math::PI;        // 3.14159
    cout << Math::square(5); // 25
}
```



**场景4：命名空间**

```
#include <iostream>
using namespace std;  // std是命名空间

int main() {
    // 如果不写using namespace std，就要这样：
    std::cout << "hello";  // cout在std命名空间里
    
    // ::左边是命名空间名，右边是里面的东西
}
```



**继承就像"复制" + "扩展"**

```
class A { ... };        // 基类（父类）
class B : public A { ... }; // 派生类（子类），这里的public是继承方式
```

**继承的本质**：`B` 继承了 `A`，意味着：

- `B` 自动拥有了 `A` 的所有成员（变量+函数）
- `B` 可以添加自己的新成员
- `B` 可以修改（重写）从 `A` 继承来的某些函数

**三种继承方式：**

| 继承方式         | 效果                             | 最常用                     |
| :--------------- | :------------------------------- | :------------------------- |
| `public` 继承    | 保持A中原有的访问权限            | ✅ 最常用（表示"is-a"关系） |
| `protected` 继承 | 把A的`public`成员变成`protected` | ❌ 较少用                   |
| `private` 继承   | 把A的所有成员变成`private`       | ❌ 默认方式                 |



**一、构造与析构的顺序**

**规则：先父母，后自己（构造）；先自己，后父母（析构）**

```
class Base {
public:
    Base() { cout << "Base构造" << endl; }
    ~Base() { cout << "Base析构" << endl; }
};

class Derived : public Base {
public:
    Derived() { cout << "Derived构造" << endl; }
    ~Derived() { cout << "Derived析构" << endl; }
};

int main() {
    Derived d;
    // 输出顺序：
    // Base构造
    // Derived构造
    // Derived析构（d离开作用域时）
    // Base析构
}
```

- **构造函数** `Base()`：负责初始化（**和类名相同的函数**是构造函数，这是C++的规定），创建对象时**自动调用**，没有返回值（不写`void`），可以重载（可以有多个参数不同的构造函数）
- **析构函数** `~Base()`：销毁对象时**自动调用**，负责清理，**`~` 是取反符号，这里表示"析构"**



**`new`和`delete`**

允许程序**在运行时**（而不是编译时）分配内存

**new 的基本语法**

```
// 分配单个变量
int* p1 = new int;           // 分配内存，但未初始化
int* p2 = new int(10);      // 分配内存并初始化为10
int* p3 = new int();        // 分配内存并初始化为0

// 分配数组
int* arr = new int[10];      // 分配10个int的数组
```

**delete 的基本语法（对指针/地址进行操作）**

```
// 释放单个变量
delete p;    // 释放p指向的内存

// 释放数组
delete[] arr;    // 释放数组内存，注意要加[]
```

**new 的工作原理**

```
Book* book = new Book("C++教程", "张三", 500);

// 实际上发生了三件事：
// 1. 分配内存：operator new(sizeof(Book)) 分配足够的内存
// 2. 构造对象：在分配的内存上调用Book的构造函数
// 3. 返回指针：返回指向该内存的指针
```

**delete 的工作原理**

```
delete book;

// 实际上发生了两件事：
// 1. 调用析构函数：~Book() 清理对象资源
// 2. 释放内存：operator delete(book) 释放内存
```

**示例：**

```
class String {
private:
    char* str;  // 指针成员
    
public:
    String(const char* s) {  // 构造函数：分配内存 
        str = new char[strlen(s) + 1];
        strcpy(str, s);
    }
    
    ~String() {  // 析构函数：释放内存
        delete[] str;
    }
};

// 正确创建对象方式，用new
String* s = new String("hello");//创建对象的同时给s赋值"hello"

// 正确释放方式，用delete
delete s;  // ✅ 先调 析构函数 释放str，再释放s本身

// 如果用free释放
free(s);  // ❌ 析构函数 不会被调用！str指向的内存泄漏了！
```

**`new[]` 和 `delete[]` 的配对**

```
// 申请对象数组
String* arr = new String[3];  // 会调用3次构造函数！

// 释放数组
delete[] arr;  // 会调用3次析构函数！重要！

// 错误写法：
delete arr;  // ❌ 只调用第1个对象的析构函数，后两个对象的内存泄漏！
```



## **栈 vs 堆 对比表**

| 特性         | 栈（Stack）      | 堆（Heap）                |
| :----------- | :--------------- | :------------------------ |
| **创建方式** | `Book b(...)`    | `Book* p = new Book(...)` |
| **内存分配** | 自动             | 手动（new）               |
| **释放方式** | 自动（出作用域） | 手动（delete）            |
| **速度**     | 快               | 慢                        |
| **大小**     | 小（MB级）       | 大（GB级）                |
| **生命周期** | 随函数结束而结束 | 直到手动释放              |
