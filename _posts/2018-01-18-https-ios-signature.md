---
layout: post
title: "由 HTTPS 理解 iOS 代码签名"
description: ""
category:
tags: [https, ios, code signature]
---

参考：

- [一个故事讲完 https](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665513779&idx=1&sn=a1de58690ad4f95111e013254a026ca2&chksm=80d67b70b7a1f26697fa1626b3e9830dbdf4857d7a9528d22662f2e43af149265c4fd1b60024&scene=21)
- [iOS App 签名的原理](http://blog.cnbang.net/tech/3386/)

平时在做 Android 开发的同时，偶尔做做 iOS 开发，同样是代码签名，Android 并不需要向 Google 申请什么证书，但对于 iOS APP 来说，需要向 Apple 申请证书，为了生成证书，又要生成公钥和私钥，我对这个流程一直搞不清，为什么需要这些证书，这些证书是用来做什么的?

直到无意间看了刘欣的公众号上的一篇文章 [一个故事讲完 https](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665513779&idx=1&sn=a1de58690ad4f95111e013254a026ca2&chksm=80d67b70b7a1f26697fa1626b3e9830dbdf4857d7a9528d22662f2e43af149265c4fd1b60024&scene=21)，明白了 HTTPS 的工作机制后，恍然大悟，原来 iOS 的这一套流程和 HTTPS 的流程是几乎一样的。

先来看一下对比图：

![https ios sign compare]({{site.img_url}}/https-website-ios-sign-compare.png)

在 HTTPS 体系中：

1. 网站为了布署 HTTPS，需要生成公钥和私钥，然后向 CA 申请证书。
1. CA 用自己的私钥为网站的公钥及其它信息进行数字签名，将网站的公钥及其它信息，和数字签名打包成数字证书，网站将数字证书存在服务器上。
1. 浏览器首次访问 https 网站时得到其数字证书，用内置在浏览器中 CA 公钥进证书进行验证，验证 OK 说明证书中包含的网站公钥是信赖的，于是用网站公钥加密随机生成的对称密钥，发送给服务器。验证失败就提示警告信息，禁止继续访问。
1. 服务器用私钥解密，得到对称密钥，随后双方用对称密钥进行对称加密通信。

而在 iOS 的体系中：

- Apple 扮演的正是 CA 的角色
- iOS APP 正如网站 - Website
- iOS 设备 (如 iPhone) 正如浏览器，内置了 Apple 的公钥

流程是这样的：

1. 开发者首先为 iOS APP 在本地生成公钥私钥对，然后向 Apple 申请证书。
1. Apple 用自己的私钥为 APP 的公钥及其它信息进行数字签名，然后 APP 的公钥及其它信息，和数字签名打包成数字证书，数字证书将会在 APP 编译打包时，把证书也打包进去。
1. iOS APP 在打包时，同时将 APP 数据进行 Hash 后，用自己的私钥对 Hash 值进行数字签名，这个签名就是真正的代码签名，这个签名也会打包进 APP，它是用来在安装时校验 APP 数据的完整性。所以这里有双重签名。
1. iOS 设备在安装 APP 时，首先用内置在设备中的 Apple 公钥，对 APP 中的证书进行验证，验证 OK 说明证书中包含的 APP 公钥是信赖的。验证失败，则不允许安装。
1. iOS 设备取出证书中的 APP 公钥，用这个 APP 公钥继续对第二重签名，即代码签名进行验证，验证 OK 说明这个 APP 是完整的，没有被人篡改过。验证失败，则不允许安装。

来自 [iOS App 签名的原理](http://blog.cnbang.net/tech/3386/) 的一张流程图：

![ios sign]({{site.img_url}}/https-ios-sign.png)

HTTPS 和 iOS 代码签名的流程，在最后一步有较大的区别：

- HTTPS 中，浏览器用网站的公钥加密对称密钥，服务端用网站的私钥解密。
- iOS 代码签名中，APP 用自己的私钥对 APP Hash 值签名，iOS 设备用 APP 的公钥验证代码签名。

可见：

- 签名用私钥，加密用公钥。
- 签名的主要目的是为了验证其正确性，并非加密，因为它可以用公钥解密，而公钥人人皆知。

另外，HTTPS 的普及，让曾经在对安全要求比较高的网站 (比如银行网站) 上大行其道的各种安全插件就此下岗了。

对于 Android 的代码签名来说，由于在这个体系中，并没有 CA 的角色，因此它只有一重签名，即代码签名。

![android sign]({{site.img_url}}/https-android-sign.png)
