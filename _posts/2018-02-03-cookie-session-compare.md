---
layout: post
title: "我对 cookie 和 session 区别的理解"
description: ""
category:
tags: [cookie, session, rails]
---

摘自我对《Agile Web Development Rails 5》所做的笔记。

对于 cookie 和 session 的区别，人们往往张口就来，cookie 存储在客户端，session 存储在服务端，其实并非如此。

**Rails Sessions**

session 用来保持状态，不要和 cookie 混淆，它们只是有一点点交集而已。

**Session Storage**

session 的存储方式 `session_store`，在 rails 中多达 6 种：

- `:cookie_store`
- `:active_record_store`
- `:drb_store`
- `:mem_cache_store`
- `:memory_store`
- `:file_store`

这 6 种方式，除了第一种 `:cookie_store` 是存储在客户端的 cookie 中，其余都是在服务端进行状态的持久化。cookie 只是实现 session 的手段之一，而且在 rails 中这是默认的实现方式。

我认为，人们常说的 cookie 有广义和狭义之分，广义的 cookie，就是被浏览器客户端持久化的这些对象，会在每次请求时被浏览器自动携带，它包括狭义的 cookie 和 session。狭义的 cookie 就是所有的 cookie 中除去 session 数据的那部分。

如果采用 `:cookie_store` 方式的 session，数据失效很好处理，关闭浏览器，再打开浏览器，cookie 中的 session 部分就会被自动清空，所以这种方式 session 的有效周期维持在一个浏览器进程时长。这种方式将减轻服务端逻辑。但 `:cookie_store` 方式只适合存储比较小量的数据，比如只存一个 `user_id`。

如果采用服务端方式的 session，session 的生命周期是永久的，因此必须借助过期时间来清除过期的 session。另外，要注意，采用服务端方式的 session，并不是说就不需要往客户端的 cookie 中存储任何数据了，还是需要的，只不过只需要存储一个 `session_id`，然后通过 `session_id` 到服务端再去拿对应的其它数据。

正因为 session 在客户端的实现总是要借助 cookie，这就是为什么我们很容易把它和 cookie 混淆。

session 在 cookie 中存储的数据，一般都是以加密形式存在的，至少在 rails 中是这样的，在服务端用密钥 (一般是对称密钥) 加密，返回给客户端，客户端请求时带上 session，然后在服务端再进行解密。

而 cookie 中除了 session 的数据外，其它的一般来说是明文的。
