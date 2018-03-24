---
layout: post
title: "也说 MVC"
description: ""
category:
tags: [mvc]
---

参考：

- [你真的理解了 MVC，MVP，MVVM 吗？](https://mp.weixin.qq.com/s/EzxfJLb5Hjxyw0_S5rThvg)
- [MVC，MVP 和 MVVM 的图示](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)

其实之前一直就很想写一篇博客，也来说说 MVC。为什么呢，虽然这个概念已经在网上讲烂了，但是每篇文章下面的评论还是争论不休，比如阮先生的这篇文章 - [MVC，MVP 和 MVVM 的图示](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)。大部分文章都是前端或客户端 (桌面/移动客户端) 的开发人员，以前端或客户端的角度讲 MVC，也可能是因为这个概念最早是从桌面客户端来的，但文章并没有说明这个 MVC 是在前端或客户端场景使用的，因此评论里的后端开发人员就首先不同意了。

后端开发人员也天天用着 MVC 呢，绝大部分 Web 框架都是 MVC 架构的，但 Web 框架的 MVC 工作模式和客户端的 MVC，很明显就不一样，主要体现在 View 上。客户端的 View 是对象，有方法，可以被 Controller/Model 调用，但 Web 中的 View，我们一般理解为是 HTML 模板，其实就是一些字符串，没有方法可以被 Controller/Model 被用，倒是在模板中可以调用 Model 的方法。

另外，我觉得，在前端/客户端中的 MVC，View 算是入口，它接收大部分外界的输入，然后将事件传递给 Controller 处理，View 可以理解成是主动式的。

但在后端的 MVC 中，HTTP 请求直接交给 Controller 处理，Controller 从 Model 中取得数据，然后生成 View (HTML/JSON/XML...)，返回给客户端，View 完全是被动式的，它也没有任何能力可以接受外界输入。

当看到刘欣的这篇文章 - [你真的理解了 MVC，MVP，MVVM 吗？](https://mp.weixin.qq.com/s/EzxfJLb5Hjxyw0_S5rThvg) 时，我觉得我想写的文章可以不用写了，在这篇文章中，很明确地区分了客户端/前端和 Web 后端场景下的 MVC，不至于让读者迷惑。

至于 MVP 和 MVVM，这两个一般只用于前端/客户端，不容易迷惑。
