---
title: 关于gcc对strlen优化的问题
date: 2017-08-22 10:13:04
tags:
categories: learning
---

#### 关于GCC对strlen优化的问题
代码优先：
```
const char *str = "abcdadkjadasdasdasdasdasdasdasdasdfgeghrthrthrhrthrthrthrthrthrthrthrthrthrthrthvdvdvscvscawdaadasadaefg";

#define BUFF_LEN	1000

int fun()
{
    char buf[BUFF_LEN] = {0};

    for (int i = 0; i < strlen(str); ++i)
    {
        buf[i] = str[i];
    }

    return 0;
}

#define LOOP    100000

int main()
{
    struct timeval start, end;
    gettimeofday(&start, NULL);
    for (int i = 0; i < LOOP; ++i)
        fun();
    gettimeofday(&end, NULL);
    printf("fun time: %d\n", (end.tv_sec - start.tv_sec) * 1000 + (end.tv_usec - start.tv_usec) / 1000);

    return 0;
}
```
os：centos 6.5 64位，编译器gcc4.4.7，分别使用优化等级0-2编译代码，运行时间如下
```
$ gcc -O0 -std=c99 stro.c
$ ./a.out
$ fun time: 105
$ gcc -O1 -std=c99 stro.c
$ ./a.out
$ fun time: 775
$ gcc -O2 -std=c99 stro.c
$ ./a.out
$ fun time: 0
```
然后添加编译选项`-fno-builtin-strlen`，运行时间：
```
$ gcc -O0 -std=c99 -fno-builtin-strlen stro.c
$ ./a.out
$ fun time: 95
$ gcc -O1 -std=c99 -fno-builtin-strlen stro.c
$ ./a.out
$ fun time: 72
$ gcc -O2 -std=c99 -fno-builtin-strlen stro.c
$ ./a.out
$ fun time: 0
```
发现在O1优化等级下加与不加`-fno-builtin-strlen`对程序影响很大，为了找出原因，从汇编代码入手分析。
```
O1不加-fno-builtin-strlen的汇编代码
0000000000400514 <fun>:
  400514:	ba 00 00 00 00       	mov    $0x0,%edx
  400519:	48 c7 c6 ff ff ff ff 	mov    $0xffffffffffffffff,%rsi
  400520:	b8 00 00 00 00       	mov    $0x0,%eax
  400525:	eb 03                	jmp    40052a <fun+0x16>
  400527:	83 c2 01             	add    $0x1,%edx
  40052a:	48 8b 3d e7 04 20 00 	mov    0x2004e7(%rip),%rdi        # 600a18 <str>
  400531:	48 89 f1             	mov    %rsi,%rcx
  400534:	f2 ae                	repnz scas %es:(%rdi),%al
  400536:	48 f7 d1             	not    %rcx
  400539:	48 83 e9 01          	sub    $0x1,%rcx
  40053d:	48 63 fa             	movslq %edx,%rdi
  400540:	48 39 cf             	cmp    %rcx,%rdi
  400543:	72 e2                	jb     400527 <fun+0x13>
  400545:	b8 00 00 00 00       	mov    $0x0,%eax
  40054a:	c3                   	retq   

O1加-fno-builtin-strlen的汇编代码
0000000000400554 <fun>:
  400554:	53                   	push   %rbx
  400555:	bb 00 00 00 00       	mov    $0x0,%ebx
  40055a:	eb 03                	jmp    40055f <fun+0xb>
  40055c:	83 c3 01             	add    $0x1,%ebx
  40055f:	48 8b 3d f2 04 20 00 	mov    0x2004f2(%rip),%rdi        # 600a58 <str>
  400566:	e8 f5 fe ff ff       	callq  400460 <strlen@plt>
  40056b:	48 63 d3             	movslq %ebx,%rdx
  40056e:	48 39 c2             	cmp    %rax,%rdx
  400571:	72 e9                	jb     40055c <fun+0x8>
  400573:	b8 00 00 00 00       	mov    $0x0,%eax
  400578:	5b                   	pop    %rbx
  400579:	c3                   	retq
```
通过汇编代码发现，如果不加`-fno-builtin-strlen`选项，gcc在O1优化等级下会把`strlen`编译成`repnz scas %es:(%rdi),%al`指令，通过查阅资料发现该指令是一条扫描指令(可能描述不准确)，目的是计算字符串长度，而使用`-fno-builtin-strlen`选项，就直接调用`strlen`库函数。

通过运行时间来看gcc在O1等级下对`strlen`的优化不如glibc的`strlen`函数，而`-fno-builtin-strlen`选项就是关闭gcc对strlen的优化，同样的`memcmp`好像有类似的情况，待以后分析。