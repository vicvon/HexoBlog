---
title: 关于top-level cv-qualifiers
date: 2017-10-09 16:50:56
tags:
categories: cplusplus
---
### top-level const 和low-level const
top-level const：const修饰的是自身
low-level const：const修饰的是别人

所谓的自身和别人：

- POD类型，类对象都是自身
- 指针可以是自身，也可以是别人
- 引用都是别人

通俗记忆`const char *`是`low-level const`，而`char *const`是`top-level const`。

`const char *`表示指针所指向的内容不可改变，指针本身是可以改变的，相对于`low-level const`修饰的是别人

`char * const`表示指针本身不可以改变，但是所指向的内容是可以变的，相对于`top-level const`修饰的自身