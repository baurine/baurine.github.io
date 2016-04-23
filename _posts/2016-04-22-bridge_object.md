---
layout: post
title: "编程中的桥梁对象"
description: ""
category: 
tags: [Github pages, Jekyll 3, Scribble]
---

Python:

(String) ---- str.decode() ----> [Unicdoe] ---- unicode.encode() ----> (String)

Java:

(String) ---- SimpleDateFormat.parse(str) ----> [Date] ---- SimpleDateFormat.format(date) ----> (String)

Android:

(jpg/png) ---- BitmapFactory.decode() ----> [Bitmap] ----> Bitmap.compress() ----> (jpg/png)
