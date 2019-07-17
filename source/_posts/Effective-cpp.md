---
title: Effective cpp
date: 2019-02-26 23:16:59
tags: 
categories: cplusplus
---
## Effective C++

本文旨在回顾和记录《Effective C++》一书中的关于C++的55条建议，及每一条简要说明，结构同书中目录。所有的建议并非完全适用，但可以作为指导。

#### 让自己习惯C++

##### *1. 视C+为一个语言联邦*

C++由四个部分组成，`c with class`，`Object-Oriented C++`, `Template C++`, `STL`。

##### *2. 尽量以`const`,`enum`,`inline`替换`#define`*

`#define`只是单纯的字符串替换，因此不做类型检查，而且定义宏时最好加上小括号，防止调用宏后出现非预期结果。

对于单纯的常量，最好以`const`对象或`enum`替换`#define`

对于形似函数的红，最好改用`inline`函数替换`#define`

##### *3. 尽可能使用`const`*

将某些东西声明为`const`可帮助编译器侦测出错误用法。`const`可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体。

编译器强制实施`bitwise constness`，但你编写程序时应该使用“概念上的常量性”

当`const`和`non-const`成员函数有着实质等价的实现时，令`non-const`版本调用`const`版本可避免代码重复

##### *4. 确定对象被使用前先被初始化*

为内置型对象进行手工初始化，因为C++不保证初始化它们（modern c++行为是否一致？）

构造函数最好使用成员初始化列表，而不要在构造函数本体内使用赋值操作。（并非一定遵守）

为免除“跨编译单元之初始化次序”问题，用`local static`对象(函数内的static对象)替换`non-local static`对象(非函数内的stat对象)。

```c++
// a.hpp
class A {
public:
    void funA();
};
extern A a;

// b.hpp
class B {
public:
    B() {
        a.funA();
    }
};

B b(); //有可能构造B的时候A还没有被初始化
```

有可能构造B的时候A还没有被初始化。

``` c++
A & genA() {
    static A a;
    return a;
}

class B {
public:
    B() {
        genA().funA();
    }
}

B & tempB() {
    static B b;
    return b;
}
```

此时调用就保证了A对象被初始化。

