---
title: libevent最小堆源码分析
date: 2017-10-09 17:09:59
tags:
categories: algorithm
---
### libevent最小堆源码分析

libevent最小堆算法用于处理定时器超时任务


```
/* 最小堆数据结构 */
typedef struct min_heap
{
    struct event** p;  /* 保存的元素，使用数组模拟堆 */
    unsigned n, a;  /* n，当前保存的元素个数； a，数组的容量 */
} min_heap_t;

/* 比较元素大小 */
#define min_heap_elem_greater(a, b) \
    (evutil_timercmp(&(a)->ev_timeout, &(b)->ev_timeout, >))

/* 构造最小堆 */
void min_heap_ctor_(min_heap_t* s) { s->p = 0; s->n = 0; s->a = 0; }
/* 删除最小堆 */
void min_heap_dtor_(min_heap_t* s) { if (s->p) mm_free(s->p); }
/* 最小堆某元素初始化 */
void min_heap_elem_init_(struct event* e) { e->ev_timeout_pos.min_heap_idx = -1; }
/* 最小堆是否为空 */
int min_heap_empty_(min_heap_t* s) { return 0u == s->n; }
/* 最小堆大小 */
unsigned min_heap_size_(min_heap_t* s) { return s->n; }
/* 返回堆顶元素 */
struct event* min_heap_top_(min_heap_t* s) { return s->n ? *s->p : 0; }

/* 往最小堆中插入数据 */
int min_heap_push_(min_heap_t* s, struct event* e)
{
    /* 扩容 */
    if (min_heap_reserve_(s, s->n + 1))
        return -1;
    /* 元素位置调整 */
    min_heap_shift_up_(s, s->n++, e);
    return 0;
}

struct event* min_heap_pop_(min_heap_t* s)
{
    if (s->n)
    {
        struct event* e = *s->p;
        /* 删除堆顶元素，调整最小堆，子节点向上移或最后元素填补空缺 */
        min_heap_shift_down_(s, 0u, s->p[--s->n]);
        e->ev_timeout_pos.min_heap_idx = -1;
        return e;
    }
    return 0;
}

int min_heap_elt_is_top_(const struct event *e)
{
    /* 判断是否为堆顶元素 */
    return e->ev_timeout_pos.min_heap_idx == 0;
}

int min_heap_erase_(min_heap_t* s, struct event* e)
{
    if (-1 != e->ev_timeout_pos.min_heap_idx)
    {
        /* 元素个数减1，获取最后一个元素 */
        struct event *last = s->p[--s->n];
        /* 计算待删除节点的父节点 */
        unsigned parent = (e->ev_timeout_pos.min_heap_idx - 1) / 2;
        /* we replace e with the last element in the heap.  We might need to 
           shift it upward if it is less than its parent, or downward if it 
           is greater than one or both its children. Since the children are 
           known to be less than the parent, it can't need to shift both up 
           and down. */
        /* 用最后一个元素替换e，如果它比它父节点小，将它提升；如果它比它的孩
           子大，要将它降下。 */
        if (e->ev_timeout_pos.min_heap_idx > 0 && min_heap_elem_greater(s->p[parent], last))
            /* e不是根节点，并且last小于e的父节点，last替换e并提升位置 */
            min_heap_shift_up_unconditional_(s, e->ev_timeout_pos.min_heap_idx, last);
        else
            min_heap_shift_down_(s, e->ev_timeout_pos.min_heap_idx, last);
        e->ev_timeout_pos.min_heap_idx = -1;
        return 0;
    }
    return -1;
}

int min_heap_adjust_(min_heap_t *s, struct event *e)
{
    if (-1 == e->ev_timeout_pos.min_heap_idx) 
    {
        /* e是新数据，直接插入 */
        return min_heap_push_(s, e);
    } 
    else 
    {
        unsigned parent = (e->ev_timeout_pos.min_heap_idx - 1) / 2;
        /* The position of e has changed; we shift it up or down
         * as needed.  We can't need to do both. */
        if (e->ev_timeout_pos.min_heap_idx > 0 && min_heap_elem_greater(s->p[parent], e))
            min_heap_shift_up_unconditional_(s, e->ev_timeout_pos.min_heap_idx, e);
        else
            min_heap_shift_down_(s, e->ev_timeout_pos.min_heap_idx, e);
        return 0;
    }
}

int min_heap_reserve_(min_heap_t* s, unsigned n)
{
    if (s->a < n)
    {
        /* 元素个数超过容量 */
        struct event** p;
        /* 初始大小为8，之后翻倍增加 */
        unsigned a = s->a ? s->a * 2 : 8;
        /* 调整之后容量还不足，直接扩大到元素个数 */
        if (a < n)
            a = n;
        if (!(p = (struct event**)mm_realloc(s->p, a * sizeof *p)))
            return -1;
        s->p = p;
        s->a = a;
    }
    return 0;
}

void min_heap_shift_up_unconditional_(min_heap_t* s, unsigned hole_index, struct event* e)
{
    unsigned parent = (hole_index - 1) / 2;
    do
    {
	(s->p[hole_index] = s->p[parent])->ev_timeout_pos.min_heap_idx = hole_index;
	hole_index = parent;
	parent = (hole_index - 1) / 2;
    } while (hole_index && min_heap_elem_greater(s->p[parent], e));
    (s->p[hole_index] = e)->ev_timeout_pos.min_heap_idx = hole_index;
}

void min_heap_shift_up_(min_heap_t* s, unsigned hole_index, struct event* e)
{
    /* 计算出当前节点的父节点 */
    unsigned parent = (hole_index - 1) / 2;
    while (hole_index && min_heap_elem_greater(s->p[parent], e))
    {
        /* 父节点大于待插入元素，交换位置 */
        (s->p[hole_index] = s->p[parent])->ev_timeout_pos.min_heap_idx = hole_index;
        hole_index = parent;
        /* 继续计算父节点 */
        parent = (hole_index - 1) / 2;
    }
    /* 将插入元素保存到适当位置 */
    (s->p[hole_index] = e)->ev_timeout_pos.min_heap_idx = hole_index;
}

void min_heap_shift_down_(min_heap_t* s, unsigned hole_index, struct event* e)
{
    /* 计算当前节点的子节点位置 */
    unsigned min_child = 2 * (hole_index + 1);
    while (min_child <= s->n)
    {
        /* 找出最小的子节点，min_child == s->n表示当前节点只有一个子节点的情况 */
        min_child -= min_child == s->n || min_heap_elem_greater(s->p[min_child], s->p[min_child - 1]);
        /* 子节点大于元素值直接返回 */
        if (!(min_heap_elem_greater(e, s->p[min_child])))
            break;
        /* 交换位置 */
        (s->p[hole_index] = s->p[min_child])->ev_timeout_pos.min_heap_idx = hole_index;
        hole_index = min_child;
        /* 继续往子节点寻找 */
        min_child = 2 * (hole_index + 1);
    }
    /* 将插入元素保存到适当位置 */
    (s->p[hole_index] = e)->ev_timeout_pos.min_heap_idx = hole_index;
}

```
