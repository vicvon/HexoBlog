---
title: c++编译器检查函数参数
date: 2019-08-27 22:29:00
tags:
categories: cplusplus
---
### C++编译期检查函数参数

最近在使用`asio`时，发现`asio`的会检查异步回调函数的参数，不符合的参数个数和参数类型都会在编译时检查出来，于是简单跟踪了`asio`的代码，发现`asio`使用的是`c++11`的`decltype`关键字的特性，`decltype`在编译期进行类型推导并且不会对表达式求值。

除了使用`decltype`外，还使用了逗号表达式。逗号表达式会按顺序执行逗号前面的表达式，例如：

```c++
auto d = (a = b, c);
```

这个表达式会先将`b`的值赋值给`a`，然后将`c`的值赋值给`d`。

将逗号表达式和`decltype`结合在一起就可以完成编译期的函数参数检查，例如：

```c++
template <typename Handler, typename Args>
auto testFuncArgs(Handler h, Args* a) -> decltype(h(*a), char(0)) {

}

template <typename Handler>
auto testFuncArgs(Handler h, ...) -> decltype(int(0)) {

}
```

测试代码：

```c++
auto f1 = [](int) {

};

auto f2 = [](std::string) {
    
};

testFuncArgs(f1, static_cast<const int*>(0)); // 1.匹配上第一个模板
testFuncArgs(f2, static_cast<const int*>(0)); // 2.匹配上第二个模板
```

第一个语句可以编译通过，但是第二个语句无法编译通过。通过利用`SFINAE`机制来进行模板匹配，当`f1`的参数是`int`类型，就匹配上了第一个模板，否则就匹配上了第二个模板。因为第一个模板中有一个`h(*a)`的表达式，第一个函数可以匹配上，而第二个函数不能匹配所以会与第二个模板匹配。

上面代码再加上静态断言就可以完成编译期函数参数检查

```c++
static_assert(sizeof(testFuncArgs(f1, static_cast<const int*>(0))) == 1, "param not int");  // 不触发断言
static_assert(sizeof(testFuncArgs(f2, static_cast<const int*>(0))) == 1, "param not int");  // 触发断言
static_assert(sizeof(testFuncArgs(f2, static_cast<const int*>(0))) == 4, "param not int");  // 不触发断言
```

没有第二个模板不会触发静态断言，而是编译错误。

类似的检查都可以使用逗号表达式和`decltype`结合来实现，例如，[编译器判断某个类是否存在某个成员函数](https://www.cnblogs.com/qicosmos/p/3753037.html)。