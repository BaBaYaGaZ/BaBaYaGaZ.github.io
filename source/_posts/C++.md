---
title: C++
date: 2026-03-09 15:49:51
tags:
---

# 第一大单元：语言基础扩展

### **1. 引用**

```
int a = 5;
int &b = a;   // b是a的引用（别名）
b = 10;       // 改b就是改a，现在a也是10
```

- `int &b` 表示 `ref` 是一个 `int` 类型的引用。
- `b` 是 `a` 的别名，对 `b` 的操作会直接作用于 `a`（引用不占用额外内存，编译器直接操作所引用的对象）。
- 一旦引用被初始化为一个对象，就不能被指向到另一个对象。
- 引用必须在创建时被初始化，不能为 `null`。
- 引用必须绑定**变量**，**不能是临时值**。

```
int& r = 10; // 是非法的！
```

- 不支持多级间接访问（**不能有引用的引用**）。

可以创建数组的引用，例如：`int (&ref)[10] = arr;`



### **2. const**

`const` 变量 **必须初始化**

合法：

```
const int a = 10;
```

非法：

```
const int a;
```

**const与普通变量**

```
int a = 10;
const int b = a;

a = 20;
```

```
b 会变吗？不会！b的值仍然是10
```

**指向常量的指针**：指针指向的区域值**不能通过指针改变**，可以改变指针的指向

```
int a = 10;
int const * p = &a;
const int * p = &a;
// p指向的类型是int const（int常量），指的是常量，常量不可通过p修改
a = 20; // 是合法的，int const * p = &a只限制不能通过p修改a，但a本身不是常量，可以修改
```

**常量指针**：指针指向的区域值可以变，不能改变指针的指向

```
int * const p
// const p指向的类型是int，const p是常量指针，其指向不能改变，但int值可以改变
```

**const 引用是什么？**

```
const int& r = a;
// r 是一个引用，引用的是 const int
// 不能通过 r 修改 a
// r 只是限制“通过 r 修改”，并没有让 a 变成 const
```

- const 引用可以绑定**临时值**：

```
const int& r = 10; // 是合法的
```

因为编译器会生成一个隐藏变量：

```
int temp = 10;
const int& r = temp;
```

所以：r 绑定到这个临时对象

**const引用的用途**

```
void print(const string& s) { // 不复制，传地址
    cout << s << endl;
}

string name = "Tom";
print(name);

或者print("hello"); // const引用可以绑定临时对象"hello"
```

若用普通引用

```
void print(string& s)
print("hello"); // 会报错，引用s不能绑定临时对象
```

若用值传递

```
void print(string s) // 每次调用都会复制一个 string
```



### **3. new/delete**

允许程序**在运行时**（而不是编译时）分配内存

#### **new：申请堆内存**

```
int* p = (int*)malloc(sizeof(int));
int* p = new int;

// 做了两件事
 1.在堆上申请内存，堆！！！
 2.返回地址
```

| 特性             | malloc  | new          |
| ---------------- | ------- | ------------ |
| 所属语言         | C       | C++          |
| 返回类型         | `void*` | 指定类型指针 |
| 是否需要强转     | 需要    | 不需要       |
| 是否调用构造函数 | 不会    | 会           |
| 释放方式         | free    | delete       |

```
// 分配单个变量
int* p1 = new int;           // 分配内存，但未初始化，随机值
int* p2 = new int(10);      // 分配内存并初始化为10
int* p3 = new int();        // 分配内存并初始化为0

// 分配数组
int* arr = new int[10];      // 调用10次构造函数，分配10个int的数组
```



#### **delete：释放堆内存**（对指针/地址进行操作）

```
int* p = new int(10);
int* arr = new int[10];  // 调用10次构造函数，分配10个int的数组

// 释放单个变量的堆内存
delete p;

// 释放数组堆内存
delete[] arr;  // 调用10次析构函数

// 错误写法：
delete arr;  // ❌只调用第1个对象的析构函数，后两个对象的内存泄漏！
```

不释放堆内存会产生**内存泄漏**，但通常不会立即报错



#### **栈对象 vs 堆对象**

```
int a = 10; // a是栈对象，函数结束自动销毁
int* p = new int(10); // *p(p指向的int对象)是堆对象，delete手动销毁
```

`*p`（p指向的 int 对象）是堆对象，`p`是栈对象

| 特性         | 栈（Stack）      | 堆（Heap）                |
| :----------- | :--------------- | :------------------------ |
| **创建方式** | `Book b(...)`    | `Book* p = new Book(...)` |
| **内存分配** | 自动             | 手动（new）               |
| **释放方式** | 自动（出作用域） | 手动（delete）            |
| **速度**     | 快               | 慢                        |
| **大小**     | 小（MB级）       | 大（GB级）                |
| **生命周期** | 随函数结束而结束 | 直到手动释放              |



#### **构造函数和析构函数中的new和delete**

```
Book* p = new Book;  // new 会触发构造函数

1.分配内存
2.调用构造函数
```

```
delete p;    // delete 会触发析构函数

1.调用析构函数
2.释放内存
```

`malloc / free` 不会调用构造、析构函数（对象内部的资源无法被释放），所以在 C++里，对象必须用 `new`

delete nullptr（对空指针delete）是**安全操作**，不会报错
delete 悬空指针（已经被释放过的内存），是**未定义操作**，会程序崩溃或报错！

```
delete p1;
delete p1; // 尝试释放已被释放的内存，程序崩溃或报错！
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





### **4. string**

```
// C
char s[100] = "hello";
char* s = "hello";

// C++
#include <string>
string s = "hello";
```

`string` **不是关键字**，和class的本质一样，而是标准库里的**类**

```
// C
char s[100];
s = "hello";   // 错误❌

// C++
string s1;
s1 = "world";  // 正确✅
```

`string` 可以**直接拼接**

```
string s1 = "hello";
string s2 = "world";
string s3 = s1 + s2; // s3 = "helloworld"
string s4 = s1 + " " + s2; // s4 = "hello world"
```

`string` **求长度length()**，length()是string对象的**成员函数**

```
string s = "hello";
s.length(); // 5
```

`string` 也能像**数组一样**访问字符

```
string s = "hello";
cout << s[0] << endl; // h
cout << s[1] << endl; // e
```



### 5. cin/cout

```
cin >> x;
cout << x << endl;
```

`<<` 叫 插入运算符
`>>`叫 提取运算符

**连续输入输出**

```
int a, b;
cin >> a >> b;
cout << a << " " << b << endl;
```



### **6. 函数重载（Overload）**

在**同一作用域**内，允许存在**多个同名函数**，只要它们的**参数列表不同**

```
// 函数名同名，C 不支持，但 C++ 支持
int add(int a, int b);
double add(double a, double b);
```

对于**参数列表不同**

情况1：参数**个数**不同

```
void f(int a);
void f(int a, int b);
```

情况2：参数**类型**不同

```
void f(int a);
void f(double a);
```

情况3：参数**顺序**不同（类型不同的前提下）

```
void f(int a, double b);
void f(double a, int b);
```

**仅仅返回值不同，不算重载**

```
int f(int a);
double f(int a);
// 返回值不同，但参数列表完全一样，不算重载
```

C++ **判断重载**时，不看返回值，**只看参数列表**



### **7. 默认参数**

在**函数声明或定义**中，为参数**指定默认值**；调用时如果**没传这个参数**，就**使用默认值**

```
void print(int x, int y = 10) {  // 为参数 y 指定默认值 10
    cout << x << " " << y << endl;
}

int main() {
    print(5);     // 只传了一个参数，所以 y = 10
    print(5, 20); // 传了第二个参数，所以 y = 20
    return 0;
}
```

默认参数必须**从右往左连续给出**，默认参数**只能放在**参数列表的**右边**

合法：

```
void f(int a, int b = 10, int c = 20);
```

非法：

```
void f(int a = 10, int b, int c = 20);
```

默认参数**和函数重载**有时候会**冲突**，造成歧义（二义性）

```
void f(int a);
void f(int a, int b = 10);

f(5); // 编译器会疑惑，调第一个，还是调第二个
```



### **8. bool**

C 语言里 **没有真正的布尔类型**

约定：0 表示 false；  非0 表示 true

C++提供了真正的布尔类型，只有两个值：**true、false**（true表示1，false表示0）

`bool` **占 1 B**

`bool` 和整数允许**互相转换**

```
bool a = 10; // 结果是：a = true（非0 → true）
bool a = true; int x = a; // x值为1
```



### **9.  inline内联函数**

**适合**：非常短的函数、频繁调用、逻辑简单

**不适合**：函数很长、递归函数、复杂逻辑

**建议**编译器**把函数展开**，而不是调用；**没有函数调用**，可以**减少开销**

```
inline int add(int a, int b) {
    return a + b;
}
x = add(3,4); // 内联函数可能被编译器直接展开为：x = 3 + 4;
```

inline **只是建议**，编译器可以接受或**拒绝**，函数很复杂时，编译器就不会展开

**类内定义的函数**默认 inline

```
class A {
public:
    void f() {
        cout << "hello";
    }
};
```





# **第二大单元：类与对象**

### **1. 类、对象、访问权限**

C++ 的类不仅可以存**成员变量**，还可以存**成员函数**

```
class Student {
public:
    int age;
    int score;

    void print() {
        cout << age << " " << score << endl;
    }
};

Student s1;
Student s2;
```

**成员函数**只有一份，存放在**代码区**；对象里只有成员变量
所以对象大小只取决于**成员变量**



**访问说明符**

| 访问位置               | `public` | `protected` | `private`  |
| :--------------------- | :------- | :---------- | :--------- |
| **本类内部**           | ✅ 可访问 | ✅ 可访问    | ✅ 可访问   |
| **派生类内部**         | ✅ 可访问 | ✅ 可访问    | ❌ 不可访问 |
| **类外部（通过对象）** | ✅ 可访问 | ❌ 不可访问  | ❌ 不可访问 |

在C++的`class`中，如果不写访问说明符，**默认是 `private`**，即第一个访问说明符出现之前的所有成员，**默认是`private`**



### **2. 构造函数**

**没有构造函数**的类，在**对象创建**后，成员变量**没有初始化**，成员变量的值是随机值

C++ 设计了**构造函数**，使**对象创建时自动执行初始化代码**

```
class Student {
public:
    int age;
    int score;

    Student() {  // 构造函数
        age = 18;
        score = 0;
    }
};
```

特征1：**函数名**必须和**类名 ** **一样**

特征2：**没有返回类型**

```
Student()      // ✅正确的构造函数
void Student() // ❌如果写了 void，它就变成普通函数了
```

特征3：对象创建时**自动调用**，构造函数不需要手动调用

```
Student s;  // 自动执行Student()构造函数
```

**带参数的构造函数**

```
class Student {
public:
    int age;
    int score;

    Student(int a, int s) {
        age = a;
        score = s;
    }
};

Student s(20, 90);
```

**构造函数重载**（C++允许多个构造函数共存）

```
class Student {
public:
    int age;
    int score;

    Student() {
        age = 18;
        score = 0;
    }
    Student(int a, int s) {
        age = a;
        score = s;
    }
};

Student s1; 
Student s2(20, 90);
```

**默认构造函数**（没有参数的构造函数）

如果**没有写**任何**构造函数**，编译器会**自动生成A( )**

```
class A {
public:
    int x;
};

A a;  // 合法，因为有A()
```

```
class A {
public:
    int x;
    A(int a) {
    	x = a;
    }
};

A a;  // 非法，编译错误，没有A()
```

**对象创建的三种写法**

```
Student s;
// 调用：Student()
```

```
Student s(20,90);
// 调用：Student(int,int)
```

```
Student s = Student(20,90);
// 语义上等价于：Student s(20,90)
```



### **3. 析构函数**

```
class Student {
public:
    Student() {
        cout << "constructor" << endl;
    }

    ~Student() {
        cout << "destructor" << endl;
    }
};

int main() {
    Student s;
    cout << "main running" << endl;
    return 0;
}
```

运行结果：

```
constructor
main running
destructor
```

当程序运行到 `Student s` ，会创建对象并**自动调用构造函数**

当 `main` 函数结束时，对象生命周期结束会销毁对象，**自动调用析构函数**

特点1：**函数名前有 `~`**

```
~Student() // ~ 叫 析构符
```

特点2：**没有返回类型**

```
void ~Student() // ❌错误！不能写返回值
```

特点3：**不能有参数**，析构函数只能写成：

```
~Student()
```

```
~Student(int x) // ❌错误！
```

所以，析构函数 **不能重载**，一个类 **最多只有一个析构函数**



**析构函数的调用时机**

情况1：栈对象生命周期结束（main结束，对象a销毁，调用析构函数）

```
int main() {
    A a;
    return 0;
}
```

情况2：作用域结束（进入代码块，创建a，离开代码块，销毁a，调用析构函数）

```
int main() {
    {
        A a;  // 栈对象
    }         // 会调用~A()，但栈不归delete管，不会发生delete
    return 0;
}
```

情况3：`delete` **堆对象**（new A，创建对象，delete p，调用析构函数，释放内存）

```
int main() {
    A* p = new A;  // new出来的堆对象，不能自动销毁，只能手动delete
    delete p;      // delete 会触发析构函数 ~A()
    return 0;
}
```



**堆对象**的生命周期，**必须由 delete 结束**，由**delete调用析构函数**

如果没有delete，**析构函数不会被调用**

**析构函数**负责**清理对象内部资源**，**delete**负责**销毁对象**

```
class A
{
    int* data;
public:
    A() {
        data = new int[100];
    }
    ~A() {
        delete[] data; // 负责清理对象 A 的内部资源int[100]
    }
};
A* p = new A;
delete p;  // 负责销毁对象 A

// delete p 先调用析构函数清理A内部资源，再销毁对象A
```



**构造顺序**：先创建先构造；**析构顺序**：后创建先析构（栈思想）

```
class A {
public:
    A()  { cout << "A construct\n"; }
    ~A() { cout << "A destruct\n"; }
};

int main() {
    A a;
    A b;
    A c;
}
```

构造顺序：a, b, c（a最先进栈，c最后进栈）

析构顺序：c, b, a（c最先出栈，a最后出栈）

```
class B {
public:
    B()  { cout << "B construct\n"; }
    ~B() { cout << "B destruct\n"; }
};

class A {
    B b;
public:
    A()  { cout << "A construct\n"; }
    ~A() { cout << "A destruct\n"; }
};

A a;
```

构造顺序：B, A

析构顺序：A, B

```
class A {
    int x;
    int y;
    int z;
public:
    A()  { cout << "A construct\n"; }
    ~A() { cout << "A destruct\n"; }
};
```

成员变量构造顺序：x, y, z

成员变量析构顺序：z, y, x



### **4. this指针**

`this` 是一个指针，永远指向**当前对象**

**哪个对象**调用成员函数，this 就指向**哪个对象**

```
class Student {
public:
    int age;

    void setAge(int a) {
        this->age = a; // 当前发起调用的对象的 age
    }

    void print() {
        cout << age << endl;
    }
};

int main() {
    Student s1;
    Student s2;

    s1.setAge(20); // this = &s1
    s2.setAge(30); // this = &s2

    s1.print();
    s2.print();

    return 0;
}
```

很多时候， `this->` **可以省略**

```
void setAge(int a) {
    age = a;       // 编译器会自动理解为：this->age = a
}
```

但有一种情况**必须写** `this`：成员变量和参数**同名**

```
class Student {
public:
    int age;

    void setAge(int age) {
        age = age; ❌ 
        //正确写法：this->age = age;✅
    }
};
```



`this` 的类型是Student*，`*this`是当前对象本身

```
return *this; // 意为：返回当前对象
```

`this`  是隐含参数，每个**成员函数都有**

```
void setAge(int a)
在底层类似于：
void setAge(Student* this, int a)
```



### 5. 类内声明、类外定义、`::`

`::` 叫 **作用域解析运算符**，含义：“的”

```
Student::print
```

意为：Student 类的 print 函数

`::` 的另一个常见用法：**访问全局作用域**（本质仍然是 “从哪个作用域里找名字” ）

```
int value = 100;

int main()
{
    int value = 10;
    cout << value << endl;    // 局部变量value，10
    cout << ::value << endl;  // ::value表示全局作用域下的value，100
    return 0;
}
```



**类内声明、类外定义的函数**

```
class Student {
public:
    int age;
    void print();       // 类内只写声明
};

void Student::print() {   // 类外进行定义
    cout << age << endl;  // 本质仍然是：this->age = a;
    // 虽然在类外定义，但本质仍是成员函数，有成员访问权限，可以直接使用age成员
}

int main() {
    Student s;
    s.age = 20;
    s.print();
    return 0;
}
```

不仅普通成员函数可以，**构造函数、析构函数也可以**类内声明、类外定义

```
class Student {
public:
    int age;
    Student();
    ~Student();
};

Student::Student() {
    cout << "constructor" << endl;
}
Student::~Student() {
    cout << "destructor" << endl;
}
```





# 第三大单元：对象复制机制

### **1. 对象复制（拷贝构造函数）**

```
class Student {
public:
    int age;

    Student(int a) {
        age = a;
    }
    
    Student(const Student& other)
    {
        age = other.age;
    }
};

int main() {
    Student s1(20);
    Student s2 = s1; // s1.age == s2.age
    return 0;
}
```

实际上这**不是赋值**，而**是初始化**，并且调用了一个特殊函数：**拷贝构造函数**

**拷贝构造函数标准形式：**

```
Student(const Student& other)
// other是一个引用，引用的类型是const Student，不能通过other修改实参
```

意为：用一个已有对象，创建一个新对象

**拷贝构造函数的三个特点：**

特点1：函数名和类名一样

特点2：参数是**同类对象的引用**（必须写引用，防止复制对象再次调用拷贝构造，导致无限递归）

特点3：参数通常加 `const`（拷贝时不应该修改原对象）

**调用拷贝构造函数的时机：**

情况1：对象**初始化**

```
Student s2 = s1;
Student s2(s1);
// 两种写法完全一样
```

情况2：函数**参数传值**（调用 func(x)，参数 a 是传值，复制 x，调用拷贝构造）

```
class A {
public:
    A() {
        cout << "constructor" << endl;
    }
    A(const A& other) {
        cout << "copy constructor" << endl;
    }
};

void func(A a){
}

int main() {
    A x;
    func(x);
    return 0;
}
```

情况3：函数**返回对象**

```
A func() {
    A a;
    return a; // 也会产生对象复制，会调用拷贝构造函数
}
```



如果不写拷贝构造函数，编译器会生成一个**默认**拷贝构造函数，进行**逐成员复制**

**double free 经典bug：**

当类里面有 **指针成员** 时：

```
class Book {
public:
    int* pages;
};
```

默认复制会变成 pages **指针地址复制**

结果是：两个对象，指向同一块内存，当两个对象析构时，会delete 同一块内存



### 2. 浅拷贝 vs 深拷贝（对新变量进行初始化）

**浅拷贝**：**只复制**成员变量的**值**，而**不复制**成员变量**指向的资源**

```
class Book {
public:
    int* pages;

    Book(int p) {
        pages = new int; // 对象内部成员page指向的资源在堆上
        *pages = p;
    }
    ~Book() {
        delete pages;
    }
};

int main() {
    Book b1(100); // 对象本身在栈上
    Book b2 = b1;

    cout << *b1.pages << endl;
    cout << *b2.pages << endl;

    return 0;
}
```

当程序结束时，很可能会出现 程序崩溃 或 报错：

```
double free
```

编译器使用**默认版本拷贝构造函数**（**浅拷贝**），逐成员复制

复制行为是： `b2.pages = b1.pages`（只是复制了地址，两个对象共享**同一块内存**）

析构时会 **重复释放同一块内存（double free）**



**深拷贝**：复制对象时，为新对象**重新申请一块内存**，并**复制数据**

实现深拷贝，需要自己写 **拷贝构造函数**

```
class Book {
public:
    int* pages;

    Book(int p) {
        pages = new int;
        *pages = p;
    }
    // 拷贝构造函数
    Book(const Book& other) {
        pages = new int;          // 新申请内存
        *pages = *other.pages;    // 复制数据
    }
    ~Book() {
        delete pages;
    }
};
Book b2 = b1;
```

程序结束后不会崩溃或报错

调用：`Book(const Book& other)`

执行过程：

1. 为 b2.pages 申请新的堆内存
2. 把 b1.pages 的值复制过去

**深拷贝的标准写法**

当类里**有指针成员**时，拷贝构造函数通常写成：

```
ClassName(const ClassName& other) {
    pointer = new Type;
    *pointer = *other.pointer;
}
```



### 3. 赋值运算符重载`operator=`（对已有变量进行赋值）

```
class Book {
public:
    int* pages;

    Book(int p) {
        pages = new int;
        *pages = p;
    }

    ~Book() {
        delete pages;
    }
};
int main() {
    Book b1(100);
    Book b2(200);

    b2 = b1;
    return 0;
}
```

默认赋值行为：`b2.pages = b1.pages`，原来 b2.pages 指向的**内存丢失**，两个对象**共享同一块内存**

程序结束时，再次产生**double free**



**正确做法：**自己写：`Book& operator=(const Book& other)`

```
class Book {
public:
    int* pages;

    Book(int p) {
        pages = new int;
        *pages = p;
    }
    ~Book() {
        delete pages;
    }
    
    // 拷贝构造函数（深拷贝）
    Book(const Book& other) {
        pages = new int;
        *pages = *other.pages;
    }
    
    // 重载 operator=
    Book& operator=(const Book& other) { // 返回类型是 Book&，返回引用，避免产生额外对象复制
        if (this == &other)              // 防止自赋值
        	return *this;
        
        delete pages;                    // 释放当前对象原本的资源（b2=b1，释放b2的资源）
        pages = new int;                 // 新申请内存
        *pages = *other.pages;           // 复制数据把b1的内容复制到b2的新内存
        return *this;                    // 返回b2对象本身（this为指向b2的指针，*this为b2）
    }    
};

int main() {
    Book b1(100);
    Book b2(200);

    b2 = b1; // b2调用operator=，参数为b1，等价于b2.operator=(b1)
    return 0;
}
```



### 4. Rule of Three（三法则）

如果一个类需要自己管理资源（例如 `new` 出来的内存），通常需要同时实现：

- 析构函数
- 拷贝构造函数
- 赋值运算符

| 场景                                     | 调用函数  |
| ---------------------------------------- | --------- |
| 初始化：`Book b2 = b1;` / `Book b2(b1);` | 拷贝构造  |
| 赋值：`b2 = b1;`                         | operator= |



# 第四大单元：类的补充

### 1. `const` 成员函数

声明： `int getAge() const;`

```
class Student {
public:
    int age;

    void print() const { // 成员函数
        cout << age << endl;
    }
};
```

成员函数：这个成员函数**不会修改对象的成员变量**，在这个函数内部，**对象是只读的**

```
class Student {
public:
    int age;

    void print() const {
        age = 30;   // ❌尝试修改成员，编译会报错
    }
};
```

const 成员函数本质是：**this 指针**变成了 **const**，不能通过 this 修改成员变量

| 函数                                   | 编译器理解为                      |
| -------------------------------------- | --------------------------------- |
| 普通成员函数：     `void print()`      | `void print(Student* this)`       |
| `const` 成员函数：`void print() const` | `void print(const Student* this)` |

**const对象只能调用const函数**

编译器**不允许** `const` 对象调用非 `const` 成员函数

但是，非 `const` 对象能调用 `const` 成员函数

```
class Student {
public:
    int age;

    void print() {
        // 无论函数内容是什么
    }
};

int main() {
    const Student s;
    s.print();      // ❌报错！无论函数内容是什么，const对象只要调用非const函数，就报错！
}
```

如果成员变量是指针 `int* p`，那么 `const` 成员函数 **不能改变指针p本身的指向**

但可以改变指针指向的数据 `*p`（因为 const 修饰的是 this 指针，不是指针指向的数据）



### 2. `static` 成员变量、函数

`static` 成员变量：**整个类只有一份**

```
class Student {
public:
    static int count;   // 静态成员变量声明（类内声明）

    Student() {
        count++;
    }
};
int Student::count = 0; // 静态成员变量定义（类外定义）
```

所有 Student 对象共享同一个 count，Student对象的大小**不包含 static 成员**

```
class A {
public:
    int x;
    static int y;
};
```

`sizeof(A)` = 4，而不是8

**访问 static 成员变量**

方式1，通过对象：`s1.count`

方式2，通过类名（推荐）：`Student::count`



`static` 成员函数：不属于某个对象，属于整个类；**不创建对象也能调用**

```
class Student {
public:
    static int count;   // static 成员变量

    static void printCount() {   // static 成员函数
        cout << count << endl;
    }
};

int Student::count = 0;

int main() {
    Student::printCount();   // 调用static 成员函数，不创建对象也能调用

    return 0;
}
```

普通成员函数有 this 指针；

而**静态成员函数没有 this**，所以**不能访问普通成员变量**，只能访问：static 成员变量、static 成员函数



### 3. 友元（friend）

#### 友元函数

**friend 允许外部函数访问 private**

如果函数 `void printX(A a)` 必须访问 `private`，可以**在类中声明**：

```
friend void printX(A a);
```

注意：`friend` **不是成员函数**，仍然是普通函数，只是拥有访问权限

```
class A {
private:
    int x;

public:
    A(int v) {
        x = v;
    }
    friend void printX(A a); // 友元声明：printX 是 A 的朋友，可以访问 A 的 private 成员
};

void printX(A a) {
    cout << a.x << endl;
}

int main() {
    A a(10);
    printX(a);

    return 0;
}
```

`friend `函数**不属于类**，**没有 this 指针**

调用方式仍然是：`printX(a)` ；而不是：`a.printX()`



#### 友元类

```
class A {
private:
    int x;

public:
    A(int v) {
        x = v;
    }
    
    friend class B; // 声明B类是 A 的朋友，B 的所有函数都 可以访问 A 的 private 成员
};
```

意为：B 类的**所有成员函数**，**都可以访问** A 的 **private 成员**



友元关系**不是继承**，而是**单向授权**

A 把 B 设为 `friend`，并不意味着 B 也把 A 当 `friend`，`friend` **不是对称的**



### 4. 对象数组 / 对象指针 / 对象引用

**对象数组**：`Student arr[3];`

对象数组的构造顺序（入栈）：

- s[0] 构造
- s[1] 构造
- s[2] 构造

对象数组的析构顺序（出栈）：

- s[2] 析构
- s[1] 析构
- s[0] 析构

对象数组**初始化**（如果类有构造函数）：

```
class Student {
public:
    int age;

	Student(int a) {
    	age = a;
	}
};
```

两种初始化方式：

```
Student s[3] = { Student(10), Student(20), Student(30) };
```

```
Student s[3] = {10,20,30};
```

**对象指针**：`Student s;  Student* p = &s;`

访问成员：`p->x;`  等价于：`*(p).x;`

**对象引用**：`Student s;  Student& r = s;`

r 是 s 的别名，r 和 s 共享同一块内存，修改 r 就是修改 s 

最常见**函数参数写法**：`void func(const Student& s)`
避免对象复制，保证函数不会修改对象



## 第五大单元：运算符重载

运算符重载：为**自定义类型**  **定义运算符**的行为

运算符本质是**函数**

运算符函数的基本形式：`Type  operator符号 (参数)`，常见参数写法：`const T&`

例如： `Complex operator+(const Complex& other);`



**运算符重载**有两种写法：

**方式1：成员函数**

```
Complex operator+(const Complex& other)
```

调用方式：

```
c1 + c2
```

变成：

```
c1.operator+(c2)
```

------

**方式2：友元函数**

```
friend Complex operator+(const Complex& a, const Complex& b);
```

调用方式：

```
operator+(a,b)
```



运算符重载的**限制**：不能创造新运算符，只能重载已有的，不能改变运算符参数数量，不能改变运算符优先级

考试最常见的可重载运算符是：

```
+
=
[]
<<
>>
```



### 1. `+` 重载

####  写法一：成员函数版本

`a + b `编译器理解为：`operator+(a, b)` ；如果是成员函数版本：`a.operator+(b)`

```
class Complex {
public:
    int real;
    int imag;

    Complex(int r, int i) {
        real = r;
        imag = i;
    }

    Complex operator+(const Complex& other) {
        return Complex(real + other.real, imag + other.imag);
        // 返回新对象
    }
};

int main() {
    Complex c1(1,2);
    Complex c2(3,4);

    Complex c3 = c1 + c2; // c3.real = 4   c3.imag = 6
}
```

`c1 + c2`  编译器会转化为  `c1.operator+(c2)`，返回新的 Complex 对象给 c3

#### 写法二：friend函数版本

```
class Complex {
public:
    int real;
    int imag;

    Complex(int r,int i) {
        real = r;
        imag = i;
    }

    friend Complex operator+(const Complex& a, const Complex& b);
};

Complex operator+(const Complex& a, const Complex& b) {
    return Complex(a.real + b.real, a.imag + b.imag);
}
```

| 写法       | 形式               |
| ---------- | ------------------ |
| 成员函数   | `c1.operator+(c2)` |
| friend函数 | `operator+(c1,c2)` |





### 2. = 运算符重载

初始化：`Book b2 = b1;   ` 调用**拷贝构造函数**

赋值：`Book b2;  b2 = b1;`  调用赋值重载函数operator=

```
class Book{
public:
    int* pages;

    Book(int p){
        pages = new int(p);
    }

    Book(const Book& other){
        pages = new int(*other.pages);
    }

    Book& operator=(const Book& other){  // 函数返回类型为：Book&，返回引用避免拷贝构造
        if(this == &other)               // 检测自赋值，防止访问到已经释放的内存，造成程序崩溃
            return *this;
            
        delete pages;                    // 先释放旧资源
        pages = new int(*other.pages);   // 再申请新资源
        return *this;
    }

    ~Book(){
        delete pages;
    }
};
```

赋值表达式是函数调用，当写：`b2 = b1`

编译器理解为：`b2.operator=(b1)`  ，返回值必须是：Book&，**返回引用**，避免拷贝构造

`return *this`以支持**链式赋值** `a = b = c`

表达式从右向左：`b = c`，调用：`b.operator=(c)`，返回：`b`；

然后`a = b`，调用：`a.operator=(b)`，返回：当前对象`a`

**要点：**

- 返回值必须是：Type&，**返回引用**，避免拷贝构造
- 必须检测自赋值：this == &other
- 必须先delete旧资源，再new新资源
- 支持链式赋值



### 3.`[]` 运算符重载（必须是成员函数）

**让   自定义类   像数组一样   使用 `[]` 访问元素**

```
class Array {
private:
    int data[5];

public:
    int& operator[](int index) { // 获取data[i]的引用
        return data[index];
    }
};

int main() {
    Array arr;

    arr[0] = 10; // arr[0]是在调用operator[]，以返回data[0]的引用，再将10赋值给data[10]的引用
    arr[1] = 20;
}


```

`arr[0] = 10`  编译器理解为  `(arr.operator[](0)) = 10`
意为：arr调用operator[] ()函数，参数为0，返回data[0]的引用，再将10赋值给该引用

重载函数 `int& operator[](int index)` 的返回类型为`int&` ，
如果函数返回的是 `int`，返回的是临时值 10，不能被赋值，编译错误
如果函数返回的是引用 `int&`，返回的是 `data[0]` 的引用，再给 `data[0]` 的引用赋值 10

大多数运算符既可以是成员函数，也可以是friend函数，但**`[]`必须是成员函数**
因为**左操作数必须是对象**，`arr[i]`必须是`arr.operator[](i)`（成员函数），而非`operator[](arr,i)`（friend函数）



### 4. `<<` 与 `>>` 运算符重载（写成友元函数）

#### cout：

```
cout << c1;
```

`<<` 不是语言关键字，它本质是一个函数调用：

```
operator<<(cout, c1);
或
cout.operator<<(c1);
```

**`cout` 是一个 `ostream` 对象**

如果将`<<` 运算符重载写成成员函数：
`c1.operator<<(cout)` 也就是 `c1 << cout` 而不是 `cout << c1`

因此`<<`  运算符重载，通常写成`friend`函数：
`operator<<(cout, c1)` 也就是 `cout << c1`

`<<`重载函数声明：

```
friend ostream& operator<<(ostream& os,const Complex& c);
```

返回`ostream&`，是为了支持链式输出：`cout << c1 << c2`，相当于`operator<<(operator<<(cout, c1), c2)`，`operator<<(cout, c1)` 执行完后返回`cout`，继续执行 `operator<<(cout, c2)`



#### cin：

```
cin >> c2;
```

本质上是：

```
cin.operator>>(c);
```

**`cin` 是一个 `istream` 对象**

`>>`重载函数声明：

```
friend istream& operator>>(istream& is, Complex& c);
```

返回`istream&`，是为了支持链式输入：`cin >> c1 >> c2`，相当于`operator>>(operator>>(cin, c1), c2)`

第二个参数为`Complex& c`是因为：输入操作需要修改对象 `c` 的内容，不能加`const`



```
class Complex {
private:
    int real;
    int imag;

public:
    Complex(int r,int i) {
        real = r;
        imag = i;
    }
    friend ostream& operator<<(ostream& os,const Complex& c); 
    // 返回值类型是 ostream&，第一个参数是输出流对象 os，第二个参数是要输出的复数对象 c
    // 第二个参数const Complex& c，const为了不修改对象，Complex&为了不拷贝对象
    
    friend istream& operator>>(istream& is, Complex& c);
};

ostream& operator<<(ostream& os,const Complex& c) { 
    os << c.real << "+" << c.imag << "i";
    // 把 c.real 和 c.imag 的内容写入 os
    return os;
    // 把 os 本身返回
}

istream& operator>>(istream& is, Complex& c) {
    is >> c.real >> c.imag;
    // 从输入流里读两个整数,分别存入 c.real 和 c.imag
    return is;
    // 把 is 本身返回
}

int main() {
    Complex c(3,4);
    cout << c << endl;  //输出3+4i
}
```



### 5. 前置 `++` 与 后置 `++` 运算符重载

```
class Counter {
public:
    int value;

    Counter(int v) {
        value = v;
    }
    // 前置++
    Counter& operator++() {   // 返回引用
        value++;
        return *this;
    }
    
    // 后置++
    Counter operator++(int) { // 返回旧值，需要返回temp对象，而非原对象的引用
        Counter temp = *this; // 保存旧对象
        value++;              // value++
        return temp;          // 返回旧对象
    }
};

int main() {
    Counter c(5);
    ++c;                     // 编译器理解为：c.operator++();
    cout << c.value << endl; // 6
}
```

| 运算符 | 对应函数          |
| ------ | ----------------- |
| `++a`  | `operator++()`    |
| `a++`  | `operator++(int)` |

 





# 第六大单元：继承与多态

### 1. 继承基本语法

`class 子类 : 继承方式 父类`

子类 又名 **派生类**；父类 又名 **基类**

**继承的本质**：`B` 继承了 `A`，意味着：

- `B` 自动拥有了 `A` 的所有成员（变量+函数）
- `B` 可以添加自己的新成员
- `B` 可以修改（重写）从 `A` 继承来的某些函数

```
class Person{
public:
    string name;
    int age;

    void print() {
        cout << name << " " << age << endl;
    }
};

class Student : public Person {
public:
    int id;
};

class Teacher : public Person {
public:
    int salary;
};
```

Student、Teacher 自动拥有 Person 的成员（父类成员成为子类的一部分）

**子类对父类的访问权限：**

```
class Person {
public:
    int a;

protected:
    int b;

private:
    int c;
};
```

子类可访问：a  (**public**)，b  (**protected**)

子类不可访问：c  (private)



### 2. 三种继承方式与权限变化

继承方式 **不会改变父类成员本身的权限**，只影响**父类成员在子类中的访问级别**

`class 子类 : 继承方式 父类`

| 继承方式    | 效果                                | 最常用                     |
| :---------- | :---------------------------------- | :------------------------- |
| `public`    | 保持父类中原有的访问权限            | ✅ 最常用（表示"is-a"关系） |
| `protected` | 把父类的`public`成员变成`protected` | ❌ 较少用                   |
| `private`   | 把父类的所有成员变成`private`       | ❌ 默认方式                 |



### 3. 继承中的构造函数与析构函数顺序

构造顺序必须是：先构造父类，再构造子类

析构顺序必须是：先析构子类，再析构父类

```
class A {
public:
    A() {
        cout << "A 构造" << endl;
    }
    ~A() {
        cout << "A 析构" << endl;
    }
};

class B : public A {
public:
    B() {
        cout << "B 构造" << endl;
    }

    ~B() {
        cout << "B 析构" << endl;
    }
};

int main() {
    B obj;
}
```

程序输出：
A 构造（父）
B 构造（子）
B 析构（子）
A 析构（父）

**父类构造函数有参数时，就没有默认构造函数**

必须要**构造函数初始化列表**

先调用父类构造函数，然后再执行子类构造函数体

```
class Person {
public:
    int age;

    Person(int a) { // 带参数的构造函数
        age = a;
    }
};

class Student : public Person {
public:
    int id;

    Student(int a,int i) : Person(a) { // 先调用父类构造函数
        id = i;
    }
};
```

`class Student : public Person`  其中  `: Person(a)`叫 **构造函数初始化列表**

意为：在Student对象构造时，用 a 来构造它的 Person 部分

```
int main() {
	Student s(18,1001);
}
```

第一步：构造父类部分，执行：Person(a)
第二步：执行子类构造函数体，执行：id = 1001;

**不能在函数体内调用父类构造函数**

```
Student(int a,int i)
{
    Person(a);  // ❌这其实是创建一个临时 Person 对象，而不是构造 Student 内部的 Person 部分
    id = i;
}
```



### 4. 同名隐藏

只要**子类出现同名函数**，不管参数是否相同，**父类**所有同名函数都会**被隐藏**

调用父类函数可以使用 **作用域解析符**：

```
b.A::func();
```

```
class A {
public:
    void func() {
        cout << "A func()" << endl;
    }
    void func(int) {
        cout<<"A func(int)"<<endl;
    }
};

class B : public A {
public:
    void func(double) {     //A 的 func() 和 func(int) 都被隐藏了，B只有 func(double)
        cout<<"B func(double)"<<endl;
    }
};

int main() {
    B b;
    b.func();    // ❌报错！fun()被class B隐藏，编译器无法找到
    b.A::func(); // ✅正确
}
```

如果要**恢复父类函数**（使同名父类函数在子类可见），使用 `using A::func;`

```
class B : public A {
public:
    using A::func;   // 使func()、func(int)在 B 类中可见

    void func(double) {
        cout<<"B func(double)"<<endl;
    }
};
```

| 类型     | 作用域 | 特征                 |
| -------- | ------ | -------------------- |
| 函数重载 | 同一类 | 同名**不同参数**     |
| 同名隐藏 | 父子类 | 子类**隐藏父类**函数 |



### 5. 虚函数与多态

**父类指针可以指向子类对象**，子类指针不能直接指向父类对象
父类指针指向的是父类部分成员，子类部分成员无法访问

```
class Base {
public:
    void show() {
        cout << "Base show" << endl;
    }
};

class Derived : public Base {
public:
    void show() {
        cout << "Derived show" << endl;
    }
};

int main() {
    Base* p;
    Derived d;
    p = &d;    // Base* 父类型的指针 p 指向 Derived 子类型的 d
    p->show(); // 输出：Base show
}
```

**静态绑定**，又名编译期绑定
`p->show();` 编译器只看 p 的类型，而非 p 所指的对象，p 是 Base*，调用 `Base::show()`



如果要调用 Derived 的函数，需要用 **`virtual`** 告诉编译器 ”这个函数可能被子类重写“
**虚函数的语法：**在 **父类函数前加 virtual**

```
class Base {
public:
    virtual void show() {   // 在父类函数前加 virtual
        cout << "Base show" << endl;
    }
};
p->show(); // 输出：Derived show
```

**动态绑定**，又名运行期绑定
编译器会在 **运行时判断对象真实类型**，p 指向 Derived 对象，调用`Derived::show()`



**多态**定义：同一接口，不同实现

```
class Animal {
public:
    virtual void speak() {
        cout << "Animal sound" << endl;
    }
};

class Dog : public Animal {
public:
    void speak() {
        cout << "Dog bark" << endl;
    }
};

class Cat : public Animal {
public:
    void speak() {
        cout << "Cat meow" << endl;
    }
};

int main() {
    Animal* p;

    Dog d;
    Cat c;

    p = &d;
    p->speak(); // Dog bark

    p = &c;
    p->speak(); // Cat meow
}
```

**C++的多态：同一个指针，指向不同对象，得到不同结果**
`virtual` 虚函数支持多态



发生**多态的三个条件**

- 继承关系
- 虚函数
- 基类指针或引用指向子类对象（父类指针指向子类对象）



**虚函数重写（override）：**子类重新实现父类的虚函数

条件：

- 父类函数必须是 `virtual`
- 子类函数与父类函数的 **函数名相同**
- **参数列表相同**
- **返回类型兼容**

```
class Base {
public:
    virtual void func();
};

class Derived : public Base {
public:
    void func(); 
    // 或 void func() override;
};
```



