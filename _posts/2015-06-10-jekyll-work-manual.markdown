---
layout: post
title:  "Jekyll工作手册"
date:   2015-06-10 11:50:25
categories: 工作手册
tags: work_manual
image: /assets/article_images/desktop.JPG
---
# 写在前面的话

> 本文主要对jekyll的使用方法做一些摘录，目的是能够在日常使用中不用翻阅[官方文档](http://jekyllrb.com/docs/home/)。

# Jekyll是什么? 

> Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过 Markdown （或者 Textile） 以及 Liquid 转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。

# Jekyll目录结构

一个基本的 Jekyll 网站的目录结构一般是像这样的：
{% highlight java %}
.
├── _config.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _data
|   └── members.yml
├── _site
└── index.html
{% endhighlight %}

# 头信息及常用变量

**头信息的定义**:
任何只要包含 YAML 头信息的文件在 Jekyll 中都能被当做一个特殊的文件来处理。
头信息必须在文件的开始部分，并且需要按照 YAML 的格式写在两行三虚线之间。
{% highlight ruby %}
---
layout: post
title: Blogging Like a Hacker
---
{% endhighlight %}
在这两行的三虚线之间，你可以设置一些预定义的变量,或者甚至创建一个你自己定义的变量。
这样在接下来的文件和任意模板中或者在包含这些页面或博客的模板中都可以通过使用 Liquid 标签来访问这些变量。


**头信息的使用**:
> 使用模式："｛｛ 对象.属性 ｝｝"

在头信息中没有预先定义的任何变量都会在数据转换中通过 Liquid 模板被调用。例如，在头信息中你设置一个 title，然后就可以在你的模板中使用这个 title 变量来设置页面的 title属性 ：
{% highlight html %}
<!DOCTYPE HTML>
<html>
  <head>
    <title>{{page.title}}</title>
  </head>
  <body>
    ...
{% endhighlight %}



