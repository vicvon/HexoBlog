---
title: 关于BOOST无锁队列的用法
date: 2017-10-09 16:46:56
tags:
categories: cplusplus
---
### 关于BOOST的无锁队列
queue有几个关键的模版参数：
1. `boost::lockfree::fixed_sized` 如果设置为true，则当push元素到队列容量时就会返回false，即队列满不在添加元素到队列；如果设置为false，则当push元素到队列容量时会调用系统的api分配空间存储期望加入队列的元素，这就不一定还是lockfree的了。
2. `boost::lockfree::capacity` 指定队列的大小，并且默认会开启`boost::lockfree::fixed_sized`为true，即队列满在无法添加元素会返回false。

queue的几种定义固定容量的无锁队列方式：
1. `boost::lockfree::queue<int, boost::lockfree::capacity<128> > q`
2. `boost::lockfree::queue<int, boost::lockfree::fixed_sized<true> > q(128)`