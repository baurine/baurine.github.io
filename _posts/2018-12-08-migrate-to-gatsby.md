---
layout: post
title: "Migrate blog site to Gatsby"
tags: [gatsby]
---

坚持更新这个博客网站居然也有五年了，虽然产量不是很高但也没有完全中断过。从一开始就是用 Jekyll 搭建并布署在 Github Pages 上，最早用了 bootstrap 的主题，中途曾经尝试切换到 Hexo，但没多久又切回了 Jekyll，后来发现了一款 scribble 的简洁主题就一直沿用至今，后面就没再怎么折腾过切换其它建站方案和主题。

直到最近，听说了 [Gatsby](https://www.gatsbyjs.org/) 并进行了一些尝试，又让我有了折腾的冲动。主要是 Gatsby 是用 React 实现的，而我这两年都在写 React，Gatsby 让我可以掌控每个细节。

Gatsby 的 [tutorial](https://www.gatsbyjs.org/tutorial/) 写得很详细，照着一步一步操作，很快就能把一个博客网站搭建起来，原本我是想把新的网站也放到 Github Pages 上，覆盖用 Jekyll 搭建的网站，但是发现一个问题，因为之前我把一些博客文章的链接分享了出去，因此我要保证新的网站生成的文章链接，文章内的图片等链接也和旧的网站保持一致，但是做不到。

比如当前文章 `2018-12-08-migrate-to-gatsby.md` 在 Jekyll 下生成的路由是 `/2018/12/08/migrate-to-gatsby.html`，而在 Gatsby 下，我最多能保证生成 `/2018/12/08/migrate-to-gatsby/`，而这两者实际是不一样的地址。

因此我决定把新的博客网站布署到其它地方，旧网站继续保持，以避免以前分享到外部的链接失效，以后新的博客文章可能只发表在新网站上，旧网站不再更新。

除了生成的文章链接问题，Jekyll 和 Gatsby 还有些其它不同的地方：

1. Jekyll 把日期放在文件名中就够了，它可以从文件名中提取出日期并对文章排序，但 Gatsby 对文章进行排序是根据文件中的内容而不是文件名，因此需要把日期放到文件内容的头部，比如 `date: "2018-12-08"`
1. Jekyll 中，所有文章中用到的本地图片和文件，必须放到一个共同的目录中，不能使用相对路径，比如把每篇文章用到的图片放在当前文章的目录下，而 Gatsby 中可以。

因此迁移的时候，需要做一些额外的操作：

1. 把日期写到每一篇 markdown 文件的开头部位
1. 为每一篇文章生成一个目录，把文章和用到的图片或文件放到同一个目录中

鉴于文章数量有点多，手动操作了几篇后，效率有点低，因此写了一个 migrate.sh 的迁移脚本：

    # migrate.sh, run in macOS
    md_files=$(ls *.md)

    for file in $md_files
    do
      echo $file
      folder=$(echo $file | awk '{print substr($1, 1, 10)}')
      echo $folder
      sed -i .bak "/^layout:/d" $file
      sed -i .bak "/^description:/d" $file
      sed -i .bak "/^category:/d" $file
      sed -i .bak "/^date:/d" $file
      sed -i .bak "2a\\
    date: \"${folder}\"
    " $file

      mkdir $folder
      cp $file $folder
    done

    rm *.bak
    rm *.md

另外，新网站代码块的样式是直接从旧网站拷贝过去了。

新的网站 (by Gatsby)：

- [源码](https://github.com/baurine/gatsby-blog)
- [网站](https://baurine.netlify.com) (布署在 netlify)

旧网站 (by Jekyll)：

- [源码](https://github.com/baurine/baurine.github.io)
- [网站](http://baurine.github.io/) (布署在 Github Pages)
