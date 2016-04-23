---
layout: post
title: "Mac 风扇狂转的一种解决办法"
description: ""
category: 
tags: []
---

不知从什么时候起，Mac 的风扇开始时不时狂转起来，噪声很大。很苦恼，打开「Activity Manager」，杀掉占用 CPU 和 内存多的进程，不管用；同事推荐了「HDD Fan Control」这款软件，也完全不管用。

直到最近，我突然发现「Activity Manager」里有一栏「Energy」，从名字上来看，是指各个进程占用能量的情况，于是尝试把占用能量最高的应用关掉 (令我惊讶的是，Safari 经常是占用能量最高的应用，超过 Android Studio)，果然一会后世界又清净了。

![mac_activity_manager_energy]({{site.img_url}}/mac_activity_manager_energy.png)
