---
title: 单例模式double-check存在的问题
date: 2017-10-09 16:45:44
tags:
categories: cplusplus
---
### 单例模式双检索方式在多线程程序中存在的问题
先看单例模式双检索方式代码
```
class Singleton()
{
public:
    //...
    Singleton * getInstance()
    {
        if (instance == nullptr)
        {
            lock(); // 加锁
            if ( instance == nullptr)
            {
                instance = new Singleton();
            }
        }
        return instance;
    }
private:
    Singleton *instance;
}
```
双检索的目的是为了防止在第一次检查时对象还没有被创建，在等待锁的过程中其他地方创建了对象，所以在获得锁后再次检查对象是否被创建，否则可能会创建2个对象。

但是双检索的方式在多线程的环境下同样会出现问题，因为对象的创建不是原子操作。对象的创建要经历3个过程：
1. 创建一段可以容纳对象的内存空间；
2. 调用对象的构造函数；
3. 将该段内存地址赋值给指针变量；
而以上3步并不能保证是按1-2-3的顺序完成的，有可能是1-3-2的顺序完成，如果先把内存地址赋值给了指针变量但是对象还没有构造好，另一个线程在使用的过程中检查会不成立直接返回对象地址导致错误发生。