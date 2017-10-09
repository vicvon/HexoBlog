---
title: 自动获取tuple中某种类型的元素
date: 2017-10-09 16:44:12
tags:
categories: cpluspluc
---
###  自动获得tuple中某种类型的元素代码学习分析

```
#include <iostream>
#include <tuple>

using namespace std;

template <class T, std::size_t N, class... Args>
struct indexOf;

template <class T, std::size_t N, class... Args>
struct indexOf<T, N, T, Args...> : std::integral_constant<int, N>
{

};

template <class T, std::size_t N, class U, class... Args>
struct indexOf<T, N, U, Args...> : std::integral_constant<int, indexOf<T, N + 1, Args...>::value>
{

};

template <class T, std::size_t N>
struct indexOf<T, N> : std::integral_constant < int, -1 >
{

};

template <class T, class... Args>
T get_element_by_type(const std::tuple<Args...>& t)
{
    return std::get<indexOf<T, 0, Args...>::value>(t);
}
```

测试用例
```
tuple<int, double, char, short> tp = make_tuple(1, 2.3, 2, 1);
auto r = get_element_by_type<double>(tp);  //r = 2.3
```

模版实例化过程分析：
1.  当调用`get_element_by_type<double>(tp)`，模版被实例化为`double get_element_by_type(const tuple<int, double, char, short>& tp)`,然后调用`get<indexOf<double, 0, int, double, char, short>::value>(t)`

2.  实例化`indexOf`类，因为模版参数第一个和第三个不是一种类型，所以特化第二个模版，实例化为`indexOf<double, 0 + 1, double, char, short>`

3.  继续实例化模版类，因为第一个和第三个模版参数类型一样，所以实例化第一个模版，从而返回`indexOf::value`的值1

4.  最终实际调用的是`get<1>(tp)`，即返回`tuple`中第二个元素
