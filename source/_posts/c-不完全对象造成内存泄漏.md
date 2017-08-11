---
title: c++不完全对象造成内存泄漏
date: 2017-08-11 14:21:49
categories: cplusplus
---
##### C++类前置声明会造成内存泄漏的问题总结

先上代码
```
class A;

class B
{
public:
    B()
    {
        cout << "B()" << endl;
    }
    ~B()
    {
        delete a;
        cout << "~B()" << endl;
    }
    A *a;
};

class A
{
public:
    A()
    {
        cout << "A()" << endl;
    }
    ~A()
    {
        cout << "~A()" endl;
    }
};

int main()
{
    A *aa = new A();
    B *bb = new B();
    bb->a = aa;
    delete bb;
}
```
这种情况会导致A无法析构，造成内存泄漏。

再上一份代码
```
class A
{
public:
    A()
    {
        cout << "A()" << endl;
    }
    ~A()
    {
        cout << "~A()" endl;
    }
};

class B
{
public:
    B()
    {
        cout << "B()" << endl;
    }
    ~B()
    {
        delete a;
        cout << "~B()" << endl;
    }
    A *a;
};

int main()
{
    A *aa = new A();
    B *bb = new B();
    bb->a = aa;
    delete bb;
}
```
这样A就可以正常析构了。
造成内存泄漏的原因是前置声明的类用delete属于未定义的行为。