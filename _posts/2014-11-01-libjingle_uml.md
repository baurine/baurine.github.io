---
layout: post
title: "libjingle 线程和网络模型 UML 图"
description: ""
category: 
tags: [libjingle]
---

抽空画了一下 libjingle 的线程和网络模型的 UML 图，以便同事更好地理解。

libjingle 是 webRTC 的一部分，位于 webRTC 底层，用于 P2P 通信，但它也实现了完整的线程和网络模型，可以抽取出来单独使用。

libjingle 的线程模型和 chrome base 的线程模型很类似，每个线程都有自己的消息循环 (一般的框架都是只有 UI 线程才有消息循环，如 MFC 和 Android)。而 libjingle 更特殊一点的在于，它的线程和 socket 绑得比较紧，socket 是线程的一部分。当线程处理完消息循环里的所有消息后，就开始自动处理 socket 上的网络事件。

![libjingle_thread_socket]({{site.img_url}}/libjingle_thread_socket.png)

![libjingle_tcp_udp]({{site.img_url}}/libjingle_tcp_udp.png)
