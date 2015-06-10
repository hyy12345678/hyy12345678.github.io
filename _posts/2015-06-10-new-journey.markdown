---
layout: post
title:  "jekyll工作手册"
date:   2015-06-10 11:50:25
categories: 工作手册
tags: work_manual
image: /assets/article_images/dawn.JPG
---

# 写在前面的话
本文主要对jekyll的使用方法做一些摘录，目的是能够在日常使用中不用翻阅[官方文档](http://jekyllrb.com/docs/home/)。

# Jekyll是什么? 
Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过 Markdown （或者 Textile） 以及 Liquid 转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。

# Jekyll目录结构
一个基本的 Jekyll 网站的目录结构一般是像这样的：


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
 
 来看看这些都有什么用：
 
| 文件 / 目录 | Second Header  Third Header |
| ------------ | ------------------------- |
| _config.yml | 保存配置数据。很多配置选项都会直接从命令行中进行设置，但是如果你把那些配置写在这儿，你就不用非要去记住那些命令了。 |
| _drafts | drafts 是未发布的文章。这些文件的格式中都没有 title.MARKUP 数据。学习如何使用 drafts. |
| _includes | 你可以加载这些包含部分到你的布局或者文章中以方便重用。可以用这个标签  {% include file.ext %} 来把文件 _includes/file.ext 包含进来。 |
| _layouts | layouts 是包裹在文章外部的模板。布局可以在 YAML 头信息中根据不同文章进行选择。 这将在下一个部分进行介绍。标签 {{ content }} 可以将content插入页面中。 |
| _posts | 这里放的就是你的文章了。文件格式很重要，必须要符合: YEAR-MONTH-DAY-title.MARKUP。 The permalinks 可以在文章中自己定制，但是数据和标记语言都是根据文件名来确定的。 |
| _data | Well-formatted site data should be placed here. The jekyll engine will autoload all yaml files (ends with .yml or .yaml) in this directory. If there's a file members.yml under the directory, then you can access contents of the file through site.data.members. |
| _site | 一旦 Jekyll 完成转换，就会将生成的页面放在这里（默认）。最好将这个目录放进你的 .gitignore 文件中。 |
| index.html | 如果这些文件中包含 YAML 头信息 部分，Jekyll 就会自动将它们进行转换。当然，其他的如 .html， .markdown，  .md，或者 .textile 等在你的站点根目录下或者不是以上提到的目录中的文件也会被转换。其他一些未被提及的目录和文件如  css 还有 images 文件夹， favicon.ico 等文件都将被完全拷贝到生成的 site 中。 |


 