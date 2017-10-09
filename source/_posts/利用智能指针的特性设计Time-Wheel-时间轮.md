---
title: 利用智能指针的特性设计Time Wheel(时间轮)
date: 2017-10-09 16:47:51
tags:
categories: cplusplus
---
### 利用C++中的智能指针的特性设计Time Wheel
##### 适用场合
在需要做超时检测的场景中，不需要单独做一个超时检测线程也不用在业务流程中遍历所有待检测对象。

---
##### 时间轮原理
![image](/resources/time_wheel.jpg)

时间轮上每个槽(Bucket)都保存一系列的待检测对象，时间轮在每个检测周期转动一格，当转动到某一格后说明该位置所保存的对象已经到时间，触发这一格所保存对象的超时动作。

利用智能指针的特性可以很容易的实现对象超时后的自动处理工作。每一个Buchet都是一个待检测对象链，链中的每个元素都是一个智能指针对象，管理着需要做超时检测的对象。当时间轮转到某一位置会覆盖当前位置的元素，使原本保存的内容释放，智能指针也会释放内存调用所管理对象的析构函数，此时我们可以在析构函数中做超时处理部分的工作。使用一个入口类保存需要做超时检测的对象，当入口类析构时就可以在入口类的析构函数中调用超时对象的处理操作。

而每当我们需要更新某个对象的时间时，不需要从时间轮中找出该对象，直接将对象对应的入口类保存到时间轮的尾部，只有在智能指针的引用技术为0时才会调用入口类的析构函数，从而简化了超时检测的处理流程。

##### 时间轮的具体实现方法(非唯一)
```
class Entry;
class Object;

class Object
{
public:
    //...
    void processTimeOut()
    {
        //处理超时
    }
private:
    weak_ptr<Entry> context_;
};

class Entry
{
public:
    Entry(const weak_ptr<Object> & obj)
    : weakObj_(obj);
    {
    
    }
    ~Entry()
    {
        shared_ptr<Object> ptr = weakObj_.lock();
        if (ptr)
        {
            ptr->processTimeOut();
        }
    }
private:
    shared_ptr<Object> weakObj_;
};

typedef shared_ptr<Entry> EntryPtr;
typedef weak_ptr<Entry> WeakEntryPtr;
typedef unordered_set<EntryPtr> Bucket;
typedef boost::circular_buffer<Bucket> TimeWheel;
```

Entry入口类需要保存Object对象的强引用指针，而Object对象只需保存入口Entry的弱引用指针就可以，避免循环引用导致内存泄漏。

当Object对象被刷新的时候，使用context_构造Entry，加入到TimeWheel中，此时TimeWheel中就保存了多分EntryPtr了。