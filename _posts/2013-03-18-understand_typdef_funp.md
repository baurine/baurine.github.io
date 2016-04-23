---
layout: post
title: "理解 typedef 定义的函数指针"
description: ""
category: 
tags: [typedef, 函数指针]
---

在 C 里面，可以用 typedef 来为一个已有的数据类型增加一个新的别名。比如：

    typedef int Length;

这种简单的形式，大家都能理解。可是遇到下面这种形式，不少人就傻眼了。比如：

    typedef int (*PFI)(int, const char *);

难道是把 int 数据类型定义成了 `(*PFI)(int, const char *)` 的数据类型，可是哪有这样的数据类型啊。即使被别人告知这是定义了一种函数指针类型，但却怎么也无法和 typedef int Length 这种形式关联起来，不是应该有一种已有的数据类型，一种新的数据类型吗？可是它们在哪呢？

我们把上面的表达式稍做改变，疑惑就解开了，如下：

    typedef int (int, const char *)  *   PFI;
            ~~~~~~~~~~~~~~~~~~~~~~~~~~   ~~~
                  原来的数据类型           别名

或者：

    typedef int (int, const char *)     *PFI;
            ~~~~~~~~~~~~~~~~~~~~~~~     ~~~~
                  原来的数据类型           别名

所以，这个表达式是定义了类型为 PFI 的函数指针类型，指向的函数，返回值是 int 型，形参是 int 和 `const char *`。所以，它的使用就很好理解了，比如：

    //测试代码，所以没有写得很严谨
    int str_chr(int index, const char *s)
    {
      return s[index];  
    }

    ...
    PFI funp = &str_chr;
    //对于函数指针，这里也直接写成 PFI funp = str_chr; 上面那种写法很少用
    int ret = (*funp)(5, "baurine");
    //对于函数指针，也可以直接写成 int ret = funp(5, "test");

也可以不将 PFI 定义成函数指针类型，而是直接定义成函数类型，如下：

    typedef int PFI(int, const char *);

那么它就要这么使用：

    PFI *funp = str_chr;

这就是 C 语言的灵活和复杂之处。

来看一个更复杂的吧 (与 typedef 无关了)：

    void (*signal(int signo, void (*func)(int)))(int);

是不是要疯了...至少第一次看到这种定义的时候，我是的。这是 Unix/Linux 里系统调用 signal 函数的原型。这样的定义怎么来理解呢，同样，我们来做一下调整，如下：

    void (int)  * signal(int signo, void (int) *func);
    ~~~~~~~~~~~~~ ~~~~~~ ~~~~~~~~~  ~~~~~~~~~~~~~~~~~
      返回值类型    函数名    形参一          形参二

所以，首先这个表达式是定义了一个函数，函数名是 signal，形参一是 int 型，返回值和形参二的类型是一样的，都是函数指针，指向的函数，返回值是 void，形参是 int 型。呃，是不是很绕。

signal 函数的作用就是为某个信号注册一个新的信号处理函数，同时返回原来的信号处理函数，以便在退出时进行恢复。

上面的定义形式实在是不是太复杂了，所以 man 手册上实际上是这么定义的：

    typedef void (*sighandler_t)(int);
    sighandler_t signal(int signum, sighandler_t handler);

啊，你问我为什么上面那些表达式可以做那样的调整呢，说实话，我也不知道，我只知道这样调整后就可以方便理解了。我也没查到哪里有这方面的文档，如果有人知道的话，能否告之。

**2013/06/08 补充：**  
是我孤陋寡闻了，后来仔细把《C++ Primer》第四版重新阅读了一遍，里面有些章节对上面的问题进行了解释，比如第 7.9 节--指向函数的指针。