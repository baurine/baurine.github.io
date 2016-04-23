---
layout: post
title: "解决安装 GoAgent 时提示 \"unknown encoding: cp65001\" 的错误"
description: ""
category: 
tags: [goagent, cp65001]
---

在本人的 Windows 系统上运行 uploader.bat 往 GAE 上传 GoAgent 的代码时，一直提示

	File "python27.py", line 91, in <module>
	File “uploader.zip\__main__.py”, line 29, in <module>
	LookupError: unknown encoding: cp65001

从网上搜索后得知原来在 Windows 下 cp65001 就是 UTF-8，但 python 却不能识别 cp65001。 但是没有人给出很好的解决办法。

从搜索结果来看，这并不是一个普遍现象，因此可以想象这个问题在绝大多数人的电脑上是不存在的，为什么在我电脑上会出现呢。我回忆起我曾经按照网上一个把控制台字体修改成微软雅黑的教程操作，结果美容失败，把控制台整残了，也没找到回滚的办法，其结果是中文全显示成方块了。这其中有一个很重要的步骤：

	C:\\>chcp 65001

看来就是这一步引起的。尝试把控制台的编码改回去，从网上搜索得知 cp936 是简体中文的编码，于是：

	C:\\>chcp 936

再运行 uploader.bat 就一切 OK 了。
