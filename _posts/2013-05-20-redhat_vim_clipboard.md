---
layout: post
title: "开启 Redhat 上的 Vim 的系统剪切板功能"
description: ""
category: 
tags: [Vim, Redhat, 剪切板]
---

经常要在位于不同 terminal 的 vim 窗口之间拷贝文本，必须用鼠标框选，右键复制 (或者快捷键：ctrl + shift + c)，然后到目标窗口的插入模式下点击鼠标右键，选择粘贴 (或者快捷键：ctrl + shift + v)。很麻烦，首先是脱离不了鼠标，其次使用快捷键粘贴时必须在插入模式下。

另外还有一个严重的问题，粘贴时，格式可能会发生变化，vim 可能会对文本进行缩进，惨不忍睹。这个问题的解决办法是在粘贴前先用 :set paste 命令，使粘贴时完全遵照原来的格式，拷贝完以后还要 :set nopaste 回去。因为前者会让一些自己设定的快捷键失效。比如我设置了在任何模式下按一下 F3 就保存一次(即 `<ESC>:w<CR>`)，:set paste 后，我在插入模式下再按 F3，就变成输入了文本 `<F3>`。

从网上了解到在 vim 里是有专门的方式来处理系统剪切板里的内容的。vim 里有很多小块的单独的 buffer，vim 称之为寄存器 (别和 cpu 的寄存器搞混了，不是一个概念，你就理解为这里的一个寄存器就对应一块 buffer 就行了)，通过 :reg 命令可以看到这些寄存器，比如：

    :reg
    --- 寄存器 ---
    ""   ^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J
    "0   $ svn co svn://192.168.1.123/code ~/svn_code^J
    "1   ^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J^J
    "2   ^J
    "3   ---------------------------------------^J
    "4   http://www.cnblogs.com/JeffreyZhao/archive/2009/04/01/1424028.html^J
    "5   -------------------------------^J
    "6   -------------------------------^J
    "7   ^J
    "8   ^J
    "9   ^J
    "-   层
    "*   Tags：Vim，Redhat，剪切板
    "+   Tags：Vim，Redhat，剪切板
    "/   touch

你看，存在「0-9，-，/」这些寄存器。在这些寄存器里，记住「+」这个寄存器就行了，它用来存放系统剪切板的内容，可以和本 vim 外的其它窗口进行相互拷贝粘贴 (「*」寄存器其实也可以，它用来存放系统缓冲区的内容，就是你用鼠标框选但还没有点复制的内容，因为我容易把它和「+」搞混，我就决定弃用它)，而其它寄存器只能在本 vim 窗口内使用。

接下来面临两个问题，一是「+」寄存器不像「0-9，-，/」这些寄存器默认就有的，如何开启它；二是如何使用这些寄存器。

### 如何使用
使用的方法网上已经很多了，我就再简短地描述一下，使用「"」加上寄存器名，表示选中这个寄存器，后面再加上操作方法，比如 y 进行复制，p 进行粘贴。「+」寄存器一般是这么用：

* 复制  
首先在某个 vim 窗口中，在块选择模式下选中要复制的一段内容，按下 __"+y__ 将这段内容复制到了「+」寄存器即系统剪切板中。然后就可以在其它任何地方进行粘贴。

* 粘贴  
首先你在任何地方都可以复制一段文本，比如在浏览器里用 ctrl + c 复制一段，然后在 vim 里的命令模式下，按下 __"+p__ 粘贴。

一下要按三个键，也很麻烦，所以我做了以下按键映射：

    vmap <F5> "+y
    map <F6> "+p

### 如何开启
如果你在 vim 运行 :reg 命令没有看到「+」寄存器，说明当前 vim 不带系统剪切板功能。运行以下命令也可以查看：
  
    []# vim --version | grep clipboard
    +clientserver -clipboard +cmdline_compl +cmdline_hist 
    +cmdline_info +comments +xsmp_interact -xterm_clipboard 
    -xterm_save

看，clipboard 和 xterm_clipboard 前面是个「-」号，说明不支持。

在 ubuntu 下，这个问题很好解决，安装个 vim-gnome 就行了：

    []# sudo apt-get install vim-gnome

但在 redhat 上，没有 vim-gnome 这货啊！google「vim redhat 剪切板」，搜索到的结果没有一个靠谱。于是关键词换成「vim redhat clipboard」，搜到一篇英文结果 (当时没把网址记下来，现在找不着了...)，说在 redhat 要想使用系统剪切板的功能，必须安装 vim-X11，类似 vim-gnome 一样的。OK，运行 

    []# yum install vim-X11

装上了，一开 vim，运行 :reg，还是没有「+」寄存器，回头再把文章看一遍，原来 vim-X11 安装后的执行文件是 gvim，和 vim 是独立的，又和 ubuntu 不一样，我马上运行 gvim，出来一个图形界面窗口，好丑，难道以后就要用这货。运行 :reg，还是没有，怒了。只好耐心把文章重新阅读，原来还少了选项，要运行 gvim -v 才行的，而且还是非图形界面，太好了。在 gvim -v 运行的窗口里运行 :reg，果然有「+」寄存器了。然后用 "+y 复制了一段内容，跑到另外窗口进行 "+p，结果，不行，wholly shit！我心都凉了，咋回事，后来发现是在一个 gvim -v 和一个 vim 窗口之前相互拷贝...于是配置 ~/.bashrc：

    alias vim='gvim -v'

注：该文中的 redhat 版本是 5.8，全称 RedHat Enterprise Linux 5.8
