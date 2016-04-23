---
layout: post
title: "在一台电脑上同时使用 GitHub 和 Git@OSC"
description: ""
category: 
tags: [git, github, gitosc]
---

在一台电脑上同时使用 GitHub 和 Git@OSC，本质其实是如何在一台电脑上同时使用多个 git 的公钥私钥对。

事情的起因是，我在 GitHub 上写博客，同时又在 Git@OSC 托管了一些工程项目。当我想在同一台电脑上，使用 ssh 方式管理 GitHub 和 Git@OSC 上的项目时，问题来了。使用 `ssh-keygen` 生成公钥私钥对时，私钥默认存放在 `~/.ssh/id_rsa`，那么后生成的就会覆盖原来的。如果保存时取一个别的名字，那么 git 又读不到它。

很快在网上搜索到了解决办法 ([链接](http://www.cnblogs.com/BeginMan/p/3548139.html))，写一个配置文件，在配置文件里指定某个账号对应哪个私钥文件。但网上的文章对配置文件里各项的内容如何填写稍微有些不够详情，所以我来做一些补充。

第一步，先用 `ssh-kengen` 为 GitHub 和 Git@OSC 产生公钥私钥对，分别保存为 `id_rsa_github` 和 `id_rsa_gitosc`，将相应的公钥添加到各自的网站。

```bash
$ ssh-keygen -t rsa -C "xxx@email.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bao/.ssh/id_rsa): id_rsa_github
...

$ ssh-keygen -t rsa -C "xxx@email.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bao/.ssh/id_rsa): id_rsa_gitosc
...
```

第二步，使用 `ssh-add` 命令将新的 ssh 私钥添加到 ssh agent 中，因为默认只识别 `id_rsa`。

```bash
$ ssh-add ~/.ssh/id_rsa_github
$ ssh-add ~/.ssh/id_ras_gitosc
```

如果提示 `Could not open a connection to your authentication agent` 错误，那么先使用 `ssh-agent bash`，再使用 `ssh-add`。

```bash
$ ssh-agent bash
bash-3.1$ ssh-add ~/.ssh/id_rsa_github
...
```

第三步，配置 ~/.ssh/config 文件，如果此文件不存在，则新建一个。

那么这个 config 文件的内容该怎么来填写呢。

一个 GitHub 工程的 ssh 地址格式是：git@github.com:username/proj.git，举个例子，我的博客项目地址是：git@github.com:baurine/baurine.github.io.git。

那么 config 就应该这么写：

```
Host github                          // 这个名字随便取，用来取代上面地址中的 git@github.com
  HostName github.com                // @ 与 : 之间的内容
  User git                           // @ 之前的内容
  IdentityFile ~/.ssh/id_rsa_github  // 对应的私钥文件
```

config 中后三行的空格其实是可以去掉的，但加上更能表示层次结构。

这样配置好 config 后，就可以用 github 来替代 git@github.com 进行各种操作，实际也必须这么做，否则是会出错的。如下所示。

```bash
$ ssh -T git@github.com              // wrong!
Permission denied (publickey).

$ ssh -T github                      // ok
Hi baurine! You've successfully authenticated, ...
```

```bash
$ ssh clone git@github.com:baurine/baurine.github.io.git   // wrong!
...
Permission denied (publickey).
fatal: Could not read from remote repository.

$ ssh clone github:baurine/baurine.github.io.git           // ok
...
Checking conectivity... done.
```

对于 Git@OSC 来说，一个工程的 ssh 地址格式是：git@git.oschina.net:username/proj.git，比如：`git@git.oschina.net:spark/ruby_rails_study.git`，那么 config 中的配置如下：

```
Host gitosc                          // 这个名字随便取，用来取代上面地址中的 git@git.oschina.net
  HostName git.oschina.net           // @ 与 : 之间的内容
  User git                           // @ 之前的内容
  IdentityFile ~/.ssh/id_rsa_gitosc  // 对应的私钥文件
```

配置好以后，就用 gitosc 替代 git@git.oschina.net 进行各种操作，比如：

```bash
$ ssh -T git@git.oschina.net         // wrong!
Permission denied (publickey).

$ ssh -T gitosc                      // ok
Welcome to Git@OSC, Baurine!
```

--------------------

#### 2015/8/12 update

Silly!  
今天突然想到，我为什么要对 GitHub 和 Git@OSC 使用不同的公钥私钥对呢，直接用一样的就行啦。用 `ssh-keygen -t rsa -C your.email` 生成公钥私钥对，然后把公钥同时添加到 GitHub 和 Git@OSC 的网站上。

