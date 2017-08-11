---
title: 派生类this指针和基类this指针
date: 2017-08-11 14:26:10
categories: cplusplus
---
#### 关于派生类this指针和基类this指针的问题
```
class Base
{
public:
    Base()
    {
        cout << typeid(this).name() << endl;
        cout << this << endl;
    }
};

class Derived : public Base
{
public:
    Derived() : Base()
    {
        cout << typeid(this).name << endl;
        cout << this << endl;
    }
};
```
当创建`Derived`对象时，会先构造基类`Base`对象，这是基类中的`this`代表的是派生类，但是`typeid`还是基类。

构造派生类对象时会先构造基类对象，此时派生类对象还没有生成，`this`指向的是对象的首地址，当基类对象构造好后开始构造派生类对象，这时`this`指向的对象构造完成即是派生类对象。