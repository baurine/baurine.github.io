---
layout: post
title: "如何在 Linux 下的 Firefox NPAPI 插件中绘制按钮等 GtkWidget"
description: ""
category: 
tags: [npapi, GtkWidget]
---

最近的项目是写一个 linux 下浏览器插件，用来播放从自己的设备上采集的视频，遇到一个难题，就是如何在插件里绘制 GTK Widget，比如按钮。研究了几天也没搞定，差点要绝望了。网上搜不到资料，到 google group 发帖，到 stackoverflow 提问，给 npapi-vlc 的作者发邮件，都没有回复。把 google group 的上百个帖子浏览了一番，发现有两三个人提了与我类似的问题，三四年了都没有人回复。

昨天突然有了灵感，今早到公司一试，居然试成功了。真是太兴奋了。又解决了一个 google 不到解决办法的问题。

哎，为了写这个插件，我这个以前没怎么写过 linux 程序的人，从 ffmpeg, opengl 一路看起，又看了 npapi, gtk, gdk, cairo, 今天发现还要看 xlib 才能看懂参考的 npapi-vlc 的代码。怎么感觉没有尽头呢...

解决起来其实也很简单，只是因为之前没有做过，又找不到资料，所以无从下手。

方法是首先要在 `NPP_GetValue()` 中使用这么一段代码：
    
    case NPPVpluginNeedsXEmbed: 
      *((bool *)value) = true; 
      return NPERR_NO_ERROR;

使用到了一种叫 XEmbed 的技术。
注意，这段代码不能写在 `NP_GetValue()` 函数，我之前就写在这个函数中了，导致怎么试都出来不结果。所以更稳妥的做法是在 `NP_GetValue()` 直接调用 `NPP_GetValue()`。

然后可以在 `NPP_SetWindow(NPWindow* window)` 函数中使用

    GtkWidget* parent = gtk_plug_new(window.window);

在插件窗口中生成一个相当于 `TOP_LEVEL_WINDOW` 的 GtkWidget，接着就可以在这个 GtkWidget 之上绘制按钮之类的 GtkWidget。

我在网上找到的关于 XEmbed 唯一的相关资料：  
<https://developer.mozilla.org/zh-CN/docs/XEmbed_Extension_for_Mozilla_Plugins>

一个关于 XEmbed 的简单 Demo：  
<http://multimedia.cx/diamondx/>
