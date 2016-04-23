---
layout: post
title: "由 minifilter 想到的"
description: ""
category: 
tags: [minifilter, sfilter, 驱动子系统]
---

(好习惯易放难捡啊，一转眼就月底了，好几周没写博客了，惭愧惭愧，慢慢再重新捡起。)

sfilter 和 minifilter 是 windows 驱动中最常用的两种文件系统过滤驱动框架，都是微软提供的。用 sfilter 写文件过滤驱动会很麻烦，要自己创建控制设备对象，过滤设备对象，符号链接，注册分发函数，而且获取文件名的操作相当之麻烦。用 minifilter 就简单多了，基本上只要填充好一个结构体 (`FLT_REGISTRATION`，其实最重要的成员是 `FLT_OPERATION_REGISTRATION` 结构体) 的值，然后将它注册到 minifilter manager 中即可，不用自己创建任何设备对象和符号链接，获取文件名只用一个 API 就搞定了。

突然想到，这和 Linux 中的驱动子系统不是像极了吗，实现一个结构体，然后注册到一个系统里。如此对比起来，sfilter 就像是 Linux 中没有使用驱动子系统的驱动，一切都要自己来实现。

再仔细一想想，这种实现一个结构体，注册到某个系统，在某个时机被系统主动调用的思想其实挺朴素的。像用 Windows SDK 写一个简单的窗口程序，也是先实现一个结构体 WNDCLASSEX，然后再调用 RegisterClassEx 注册。

MFC 的消息映射机制，Linux 中的信号机制，以及 Windows 驱动中的分发函数，虽然名字不一样，但本质思想也是差不多，但它们实现的不是结构体，而是直接将函数注册到了系统中。
