---
layout: post
title: 博客搭建指南：Jekyll + Markdown
date: 2015-04-21 19:23:00
digest: 博客搭建过程总结, Blogging like a hacker，Jekyll构建博客系统，Markdown写作指南。
tags: [jekyll]
category: lessons
---

这是我在本博客的第一篇文章，记录用 Jekyll + Markdown 书写博客的过程。推荐一篇文章： [Blogging Like a Hacker][BlogHacker]。

使用Jekyll搭建博客可以将文章放在 GitHub Pages上，这样可以使用Github上的在线编辑和多人协作模式，不过对于个人博客，放在主机上也可以。搭建可以参考[理想的写作环境：Git+Github+Markdown+Jekyll][lxxzhj]。

### Jekyll
使用[Jekyll]搭建博客可以参见主页上的快速搭建博客系统：

<pre><code class="bash">
~ $ gem install jekyll
~ $ jekyll new my-awesome-site
~ $ cd my-awesome-site
~/my-awesome-site $ jekyll serve
\# => Now browse to http://localhost:4000
</code></pre>

主目录下有 ：

- about.md      :   用于作者自我介绍
- _config.yml   :   博客系统配置选项文件
- feed.xml        :   构成RSS
- index.xml      :   博客主页
- css/main.scss : css 配置文件
- _layouts/*     :   页面默认布局、文章布局和页面布局
- _includes/*   :   html页面中header, footer, head部分
- _posts/*        :    用于存放博客文章
- _sass/*          :    CSS文件

还会有 \_site 文件夹，是 Jekyll serve 时生成的博客网站代码。此外，我们可以添加 \_drafts 文件夹，临时记录和草稿文件当可以写在这里。

Jekyll的文章(post)**分成两部分**，上半部分是博客的yaml配置选项， 下半部分是文章内容，由 Markdown 书写。

### Markdown
语法简单，布局简洁，可以专注写作。 

- [Markdown语法中译][MD语法]
- [献给写作者的 Markdown 新手指南][MD 指南]

推荐[平克写作八原则][pk8yz]，博客文章也非常有营养。




[BlogHacker]:  http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html
[lxxzhj]:      http://www.yangzhiping.com/tech/writing-space.html
[jekyll]:      http://jekyllrb.com/
[MD语法]:      http://www.markdown.cn
[MD 指南]:     http://www.jianshu.com/p/q81RER
[pk8yz]:       http://www.yangzhiping.com/psy/pinker.html

