---
layout: post
title: "修改博客主题适配 Jekyll 3.0"
description: ""
category: 
tags: [Github pages, Jekyll 3, Scribble]
---

今天折腾了一天，把原来用的博客主题 [Scribble](https://github.com/chloerei/scribble) 适配成可以在 Jekyll 3.0 下正常使用了。

最近几次每次往 Github pages 上 push commit 后，都会收到 Github 发来的邮件，大意就是说，Github pages 已经升级到 Jekyll 3.0 了，2016/5/1 后将只支持使用 kramdown 作为 markdown 处理器，而我目前使用的是 redcarpet。

于是我按照邮件的指示修改配置 `_config.yml`，将 markdown processor 改成 kramdown。修改之后再 push commit，倒是不收到邮件了，但是博文里的代码语法高亮没有了。博文中使用的是 Github 官方推荐的语法高亮声明方法，比如。

    ```java
    public class User {
    }
    ```

刚好今天看到一本讲 [Jekyll 3.0 的中文电子书](http://jekyll-china.com/book/)，于是果断买之，看看能不能解决语法高亮的问题。

按照书中和 Github 官博中的介绍尝试了几种配置方法，比如

    highlighter: rouge
    markdown: kramdown
    kramdow:
      input: GFM
      syntax_highlighter: rouge

皆没有生效，甚是纳闷。

但是用 `$jekyll new [site-name]` 创建一个新的默认 site，使用默认配置，同样使用上面的语法高亮声明方式，却是可以工作的。对于我这个前端菜鸟来说，大概知道问题可能是出在 css 上，却也是无从下手。

最终想到了一种曲线救国的方式，用 `$jekyll new [site-name]` 的方式新建一个 site，然后从原来的 Scribble 主题中仅拷贝 layout (即 _include 和 _layout 目录) 和除代码高亮外的其它 css，保留默认的语法高亮的 css。最终问题解决。然后把原来的 _post 目录持贝到新 site 中，commit 后强推到远程仓库，覆盖原来的 site。 

```ruby
#See, now syntax highlighter works! 

def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
```

整个杂合过程可以 clone 本 [repo](https://github.com/baurine/baurine.github.io)，然后 checkout 后 `scribble_theme` tag 查看。

如果你想使用此 theme，也可以这么做，首先在本地安装 Jekyll 3.0，确保在本地的 `$jekyll new [site-name]` 可以正常工作，然后 clone 本 repo，checkout 到 `scribble_theme`，运行 `$jekyll serve`，查看效果，然后在此基本上创作新的博文。