---
layout: post
title: "疑难杂症系列"
description: ""
category: 
tags: [ultraiso, TtaoHoverTimer, jekyll, failed to build gem native extension]
---

### 一
为了用 U 盘安装 Ubuntu 13.04，按照网上教程下载了 UltraISO，将 Ubuntu 13.04 的 iso 镜像烧写到 U 盘上，但是每次烧写到中途，就会弹出出错的警告框 (以上操作在 Win8 64 上进行)。

![出错警告框]({{site.img_url}}/ultraiso.png)

真是郁闷坏了。于是用关键词「TtaoHoverTimer」进行 google，看来有不少受害者，但是没有一篇确切的文章提出真正的解决办法，都是在论坛发贴子问。茫茫帖子中，偶见就只有那么一个人回帖说「要不试试用管理员运行 UltraISO」，于是乎马上试了一下，果然是这个原因。哎，早该想到了，警告框里已经提示了拒绝访问，说明很有可能是权限的问题。但是很奇怪，网上的教程几乎没有提到要用管理员权限来操作这个要点。我想要么这些内容是拷贝来拷贝去的，要么他们都是在 XP 上操作的。

UltraISO 的软件也设计得不好，没有任何地方有提示说这个操作需要管理员权限的。

### 二
重装了 Ubuntu，因为我有一个 GitHub Pages 要管理，所以要重装 ruby，jekyll。在使用以下命令安装 jekyll 时，出现了错误：

    $ sudo gem install jekyll
    ...
    failed to build gem native extension
    ...

用「failed to build gem native extension」作为关键词 google，这次倒是有很多解决方法的文章，但众说纷纭，有说 ruby 版本不够新，要升级，可我这已经是用 `sudo apt-get install ruby` 刚刚安装上的，有说是没装 rvm (我想应该是 ruby virtual machine 吧)，试了，连 rvm 都装不上。不过这些大部分说的是在 Mac OS 上的解决办法。

然后在 stackoverflow 上，有一个回帖说，要不试试 `sudo apt-get install ruby1.9.1-dev` 吧。我用 `sudo apt-get install ruby`，默认安装的是 ruby1.9.1，而一般像 xxx-dev 的安装包，都是提供给开发人员使用的，会带有更多的组件。于是尝试之，嗯，果然可以了。

所以如果你也在 Ubuntu 上遇到这个问题，不妨用这个方法试一下。当然并不一定 100% 有效。