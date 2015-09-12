---
layout: post
title:  Interview Topics
date:   2015-07-16 07:58:06
categories: clay_created
tags: clay_created
image: /assets/article_images/nightmare.JPG
---
# 面试杂谈   

> 纪录一些找工作时候被问到的问题，以及自己的理解。   
> 不积跬步，无以成千里。

- Android 相对坐标和绝对坐标,请解释一下他们的区别？    
    android的触摸事件一般会传递一个event参数，这个参数有两套获取触摸位置的方法（相对坐标和绝对坐标）： 

    getX(), getY()：取得当前触摸位置相对于当前调用事件的view的左上角的坐标。（相对坐标）    
    getRawX(), getRawY()：取得当前触摸位置相对于整个屏幕左上角的坐标。（绝对坐标）

- Android 声明周期，及每个生命周期都做哪些工作?   
    销毁时没有onfinish应该是onDestory

    有一个OnRestart方法，onStop后回到前台时调用，之前好多页面刷新其实应该写在这里。

    控件初始化setContentView在Oncreate中执行，Onresume中执行恢复在Onpause中被释放的资源，比如braodcastreciever，各种sensor like GPS，等等导致电量消耗的资源。

