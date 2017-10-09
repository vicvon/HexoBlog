---
title: C++标准关于类的解释(POD、trivial、standard-layout)
date: 2017-10-09 16:48:52
tags:
categories: cplusplus
---
### trivially copyable类型(平凡拷贝类型，翻译不一定准确)定义：
1. has no not-trivial copy constructors(没有非平凡拷贝构造函数)
2. has no not-trivial move constructors(没有非平凡移动构造函数)
3. has no not-trivial copy assignment operators(没有非平凡赋值操作)
4. has no not-trivial move assignment operators(没有非平凡移动操作)
5. has a trivial destructor(有一个平凡析构函数)

一个trivial类是一个拥有默认构造函数,没有非trivial默认构造函数的trivial copyable类型

### standard-layout类型定义：
1. has no non-static data members of type non-standard-layout class(or array of such types) or reference(没有非静态成员是非标准布局的类型(或者数组)或者没有引用类型)
2. has no virtual functions and no virtual base classes(没有虚函数、没有虚基类)
3. has the same access control for all non-static data members(所有的非静态数据成员有相同的访问权限,要么都是public,要么都是private,要么都是protected)
4. has no not-standard-layout base classes(没有非标准布局基类)
5. either has no base classes with non-static data members, or has no non-static data members in the most derived class and at most one base class with non-static data members(没有包含非静态数据成员的基类或者派生类没有非静态数据成员而最多有一个基类有非静态数据成员)
6. has no base classes of the same type as the first non-static data member(没有与第一个非静态数据成员相同类型的基类)
7. has no two base class subobject of the same type(没有两个相同类型的基类)
8. has all non-static data members declared in the same class(either all in the derived or all in some base)(所有的非静态数据成员的声明在同一个类中)
9. 


### POD类型：
既是trivial类型也是standard-layout类型

### Aggregate
aggregate类型是一个数据或者是一个没有用户自定义的构造函数、没有私有和保护的非静态数据成员、没有基类、没有虚函数的类