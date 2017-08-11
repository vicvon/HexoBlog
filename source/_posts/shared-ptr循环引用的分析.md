---
title: shared_ptr循环引用的分析
date: 2017-08-11 14:23:42
categories: cplusplus
---
[TOC]
#####  关于shared_ptr循环引用
智能指针如果被循环引用需要使用weak_ptr，但是什么叫循环引用？
```
class A
{
    //其它成员省略
    shared_ptr<B> bptr;
};

class B
{
    //其它成员省略
    shared_ptr<A> aptr;
};
```
定义两个类互相包含对方的智能指针。

```
int main()
{
    A a;
    B b;
    a.iptr = shared_ptr<B>(new B);
    b.iptr = shared_ptr<A>(new A)；
    reutrn 0;
}
```
**这段代码不会出现循环引用导致资源泄漏的情况。**

```
int main()
{
    shared_ptr<A> a = shared_ptr<A>(new A);
    shared_ptr<B> b = shared_ptr<B>(new B);
    a->iptr = b;
    b->iptr = a;
    return 0;
}
```
**这段代码会造成循环引用导致资源泄漏。**

根据这两段代码的现象，循环引用应该指的是两个类的成员互相包含对方类的智能指针，这两个类又被智能指针管理，这样就会造成循环引用。
在使用的时候，只要发现两个类的成员互相包含对方类的智能指针，就应该小心使用，如果要用智能指针管理两个类对象，就应该使用weak_ptr，或者将其中一个智能指针换成常规指针(好像有违常理)，如果只是定义局部变量或者使用new/delete管理就不会有循环引用的问题(这也有违常理)。