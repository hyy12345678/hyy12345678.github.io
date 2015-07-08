---
layout: post
title:  A Python Script For Make Jekll Blog Easier
date:   2015-06-23 14:37:17
categories: clay_created
tags: clay_created
image: /assets/article_images/nightmare.JPG
---
> 从jekllForBootstrap转到原生jekyll,缺少了rake命令支持,每次新建文章都显得比较繁琐。于是自己用python写了个脚本，实现类似的功能,自己用着还行,代码如下：

<code>
    
    #!/usr/bin/env python
    #-*- coding: utf-8 -*-
    import time
    import os
    import sys
    def buildThemeDic():
        themeDic = {};
        themeDic["W"] = "/assets/article_images/desktop.JPG";
        themeDic["N"] = "/assets/article_images/nightmare.JPG";
        themeDic["D"] = "/assets/article_images/dawn.JPG";
        return themeDic;
    def checkEnv():
        basePath = os.path.abspath('.');
        checkPath = basePath + "/_posts"
        return os.path.exists(checkPath);
    def checkAndWrite(time,title,content):
        fileName = time+'-'+title+'.md';
        basePath = os.path.abspath('.');
        checkFile = basePath + "/_posts/"+fileName;
        if os.path.exists(checkFile):
            print "title you input is already exists,change a title!";
            sys.exit(0);
        else:
            f = open(checkFile,'w');
            f.write(content);
            f.close;
            print "Your file is created,path is :"+checkFile;
            print "~~~ Enjoy your trip and have fun ~~~";
    docMsg = '''---
    layout: post
    title:  %s
    date:   %s
    categories: clay_created
    tags: clay_created
    image: %s
    ---
    ''';
    if not checkEnv():
        print "you don't have _posts folder ,check the environment";
        sys.exit(0);
    themeDic = buildThemeDic();
    print "Enter your title:";
    title = raw_input().title();
    #print "your title is :"+title;

    while title == "":
        print "Please enter your article's title:"
        title = raw_input().title();

    print "Select a theme:('W' for work,'N' for nightmare,'D' for dawn)";
    theme = raw_input().upper();

    while not theme in themeDic:
        print "Select a theme:('W' for work,'N' for nightmare,'D' for dawn)";
        theme = raw_input().upper();


    myTime = time.time();
    fmtTime = time.strftime("%Y-%m-%d %H:%M:%S",time.gmtime(myTime));
    fileNameTime = time.strftime("%Y-%m-%d",time.gmtime(myTime));
    #print fmtTime;


    docMsg = docMsg % (title,fmtTime,themeDic.get(theme));
    print docMsg;

    fileNameTitle = "-".join(title.split());

    checkAndWrite(fileNameTime,fileNameTitle,docMsg);
</code>