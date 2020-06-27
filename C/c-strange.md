---
title: C语言中的小事
date: 2019-11-25 10:56:20
categories: C
tags: syntax
toc: true
# description: some description show on the top
# feature: http://blackshow.me/favicon.png
---
# 说说C语言中的小事

本文记录C语言中了一些你比较疑惑，或者奇怪的问题。


## printf("%s\n");会输出什么？

这看起来很奇怪不是么？
依你来看这就是一个错误的语法呀？

没错，但也错了，对于GCC编译器来说，这是一个warning。

我们看个例子：
```C
    #include <stdio.h>

    int main(int argc, char ** argv){
        char * test_str = "Hi Bro";
        printf("%s\n");

        printf("%s\n", test_str);
        printf("%s\n");

        return 0;
    }
```

输出：
```text
    乱码或者其他不可读的内容
    Hi Bro
    Hi Bro
```
这个例子需要看printf的实现源码才能知道为什么会这样。

TBD...



## 你考虑过C语言符号的可见性么？

符号的可见性，比如这样
```C
    struct list_head students = { &students, &students };
```
关于符号`students`，为什么还没有定义完成就可以被使用？

其实这可以从编译原理的角度分析：
由于students是编译时确定的，不论他是`global`还是`static`，已经在image中分配了它的空间，那么它的地址也就确定了，所以就有了，`&students`可以被使用的技巧。



## 关于结构体初始化和赋值的一些问题

初始化结构体操作，像这样：
```C
    struct list_head students = {
        .next = &students,
        .prev = &students,
    };
```
这没有问题，但像下面这样就会出现错误：
```C
    struct inner {
        int id;
    };
    
    struct outter {
        struct inner i;
    };
    
    {
        struct outter * out = malloc(sizeof(struct outter));
        out->i = {100};// 错误的
    }
```
根本原因有两个：
1. 类似`students`这样的静态结构体，它的存储空间是由编译器在编译的时候决定的，所以可以采用你看到的，带有花括号的综合赋值语法
2. 类似`outter`这样的赋值，之所以错误是因为在编译的时候，无法确定`out`变量的地址空间，因为它是通过`malloc`在运行时动态分配的

正确做法是：
```C
    {
        struct outter * out = malloc(sizeof(struct outter));
        out->i.id = 100;
    }
```


## 你使用过#include "xxx.c"这样的宏么？

在编译C语言的时候，你可能会遇到重复定义的问题，但我想说的并不是普通的重复定义问题。

我们知道`.h`和`.c`后缀，只不过是用来给编译器区分文件类型而已，本质上都是编译器的输入源文件，只不过人为的把它分成`头文件`和`源文件`。

也就是说我们可以这样使用`include`
```C
    #include "demo.c" // 就像包含.h文件一样。
```
而如果碰到使用这个技巧的源文件的时候，你又故作聪明的在编译的时候为Makfile加了源文件`demo.o` (demo.c),
这时候就会出现重复定义。

那么总结就是，如果看到有.c文件，但是在编译的时候没有在编译源文件列表中添加，那么肯定是使用了本文所说的这个技巧。




## 看C源代码的时候，你有这样的思维定式么？

在C语言编程的时候，与大多数语言一样，你要在不断的层次转换中知道自己所处的位置，
不过我想说的是，很多同学当基础知识不够扎实的时候，就会不知道自己在哪里。

在C语言中
```C
    dev->mii.dev = dev->net
```
就这样简单的一条语句:
符号'->'表示前面的操作数是指针，并且他结构中有mii成员，而成员mii是什么？需要后面的符号'.'来说明，
原来它是一个结构体成员，内嵌在dev指针指向的复合结构中，它包含dev成员，不过dev是指针还是内嵌成员我们并不知道。
在右值中'->'符号有同样的解释，不过这里之所以可以这样赋值，就说明前面的'mii.dev'中的dev不是一个符合结构体，而且'dev->net'同样也不是复合结构体，它们会是指针或者C语言内置的简单类型。


