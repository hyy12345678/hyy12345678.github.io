---
layout: post
title:  Bug For Pulltorefresh List
date:   2015-09-12 05:43:38
categories: clay_created
tags: clay_created
image: /assets/article_images/nightmare.JPG
---
# Pulltorefresh List的一些bug
上个礼拜被一个已经被deprecated的代码库中的一两个bug搞的肾上腺素飙高，最后实在是不好定为，于是自己把相关的组件重写了一遍，回来这两天就在各种调试，发现的bug如下：

1. 不要在代码中调用“listViewContent.setMode(PullToRefreshBase.Mode.DISABLED)”方法后再想调用setMode(PullToRefreshBase.Mode.PULL_FROM_END)，会产生既不Disable，也不是pullFromEnd的状态。

所以如果要用这个库的话，老老实实的设定一次setMode在定义xml中或者代码中，不要变来变去，就不会触发bug了。想来作者也是觉得里面的状态写的太复杂了，搞不定所有情况，就放弃了。