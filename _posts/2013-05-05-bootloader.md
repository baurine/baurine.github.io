---
layout: post
title: "实现了一个简单的 Bootloader"
description: ""
category: 
tags: []
---

实现了 uboot 的大部分主要功能，比如 go, nand read/write/erase, 环境变量的增删改查及保存，tftp 下载，自动启动，输入可以退格(但没有命令的历史功能)。实现了 arp。当然还有最主要的 bootm 命令，可以正确地引导 Linux 内核。

另外，加了一条 talktopc 的命令，然后在 PC 上用 Socket 写了一个客户端，两者可以对话，自己觉得还蛮有趣的。

![mybootloader.png]({{site.img_url}}/mybootloader.png)

通过练习这个小项目，真正明白了网络协议的原理，以及 Linux 内核是如何启动起来的，很有成就感。