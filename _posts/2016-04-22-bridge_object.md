---
layout: post
title: "编程中的桥梁对象"
description: ""
category: 
tags: []
---

1. Python

   (String) ---- str.decode() ----> [Unicode] ---- unicode.encode() ----> (String)

   Unicode 是字符在内存中的统一格式，而 utf8 / utf16 / utf32 是一种存储或传输格式，当它们被加载到内存中时，统一被解析成 Unicode 格式，而 Unicode 字符转存到外部存储器或是在网上进行传输时，可以选择转存为 utf8 / utf16/ utf32 等格式。

   - utf32 意味着所有的每个字符都用 4 个字节表示，比较浪费存储空间；
   - utf16 意味着大部分字符 (并不是全部) 用 2 个字节，少数可能需要用 3 个字节或 4 个字节表示；
   - utf8 有点像哈夫曼编码，先用 1 个字符来表示，不够了再用 2 个字节，再不够就用 3 个字符，最后再用 4 个字符。

1. Java

   (String) ---- SimpleDateFormat.parse(str) ----> [Date] ---- SimpleDateFormat.format(date) ----> (String)

1. Android

   (jpg/png) ---- BitmapFactory.decode() ----> [Bitmap] ---- Bitmap.compress() ----> (jpg/png)
