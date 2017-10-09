---
title: c++内存模型
date: 2017-10-09 16:50:13
tags:
categories: cplusplus
---
### C++11六种内存模型
1. **memory_order_relaxed**: *Relaxed operation: there are no synchronization or ordering constraints imposed on other reads or writes, only this operation's atomicity is guaranteed*(松散操作，在别的读/写上没有强制同步或者顺序限制，仅仅保证该操作是原子的)
2. **memory_order_consume**: *A load operation with this memory order performs a consume operation on the affected memory location: no reads or writes in the current thread dependent on the value currently loaded can be reordered before this load. Writes to data-dependent variables in other threads that release the same atomic variable are visible in the current thread. On most platforms, this affects compiler optimizations only*(使用`memory_order_consume`的加载操作施加一个`consume`操作在相关的内存位置`memory location`: 当前线程，没有依赖当前值的读/写可以重排序在加载之前。在释放相同原子变量的线程中写入数据依赖的变量对当前线程是可见的。在大多数平台，这仅仅影响了编译器的优化)
3. **memory_order_acquire**: *A load operation with this memory order performs the acquire operation on the affected memory location: no reads or writes in the current thread can be reordered before this load. All writes in other threads that release the same atomic variable are visible in the current thread*(使用`memory_order_acquire`的加载操作施加一个`acquire`操作在相关的内存位置：当前线程没有读/写可以被重排序在加载之前。在释放相同原子变量的线程中所有的写对当前线程可见。)
4. **memory_order_release**: *A store operation with this memory order performs the release operation: no reads or writes in the current thread can be reordered after this store. All writes in the current thread are visible in other threads that acquire the same atomic variable and writes that carry a dependency into the atomic variable become visible in other threads that consume the same atomic*(使用`memory_order_release`的存储操作施加一个`release`操作：在当前线程，没有读/写可以重排序在存储之后。当前线程的所有写入对获取相同原子变量的线程可见，并且，写入携带依赖到一个原子变量对消费相同原子变量的线程可见)
5. **memory_order_acq_rel**: *A read-modify-write operation with this memory order is both an acquire operation and a release operation. No memory reads or writes in the current thread can be reordered before or after this store. All writes in other threads that release the same atomic variable are visible before the modification and the modification is visible in other threads that acquire the same atomic variable.*(`memory_order_acq_rel`使读改写操作具有`acquire`和`release`属性。当前线程，没有读可以重排到存储之前，没有写可以重排到存储之后。释放相同原子变量的线程的所有写入在修改之前是可见的，并且修改对获取相同原子变量的线程是可见的)
6. **memory_order_seq_cst**: *Any operation with this memory order is both an acquire operation and a release operation, plus a single total order exists in which all threads observe all modifications in the same order*(`memory_order_seq_cst`的任何操作都是`acquire`和`release`的，存在一个顺序，所有的修改对所有的线程有相同的顺序)

### 深入理解C++11中的解释

memory_order | described
---|---
memory_order_relaxed | 不对执行顺序做任何保证
memory_order_consume | 本线程中，所有后续的有关本原子类型的操作，必须在本条原子操作完成之后执行
memory_order_acquire | 本线程中，所有后续的读操作必须在本条原子操作完成之后执行
memory_order_release | 本线程中，所有之前的写操作完成后才能执行本条原子操作
memory_order_acq_rel | 同时包含`memory_order_acquire`和`memory_order_release`
memory_order_seq_cst | 全部存取都按顺序执行