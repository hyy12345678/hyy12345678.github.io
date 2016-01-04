---
layout: post
title:  一个封装循环广告栏的例子
date:   2016-01-04 09:05:32
categories: clay_created
tags: clay_created
image: /assets/article_images/nightmare.JPG
---
# 一个moco的简明使用教程   

## moco简介
作为2013 年 Oracle Java 大赛的获奖作品 的Moco。他的作者是这样描述 Moco 的：   
Moco是一个简单搭建模拟服务器的程序库/工具，这个基于 Java 开发的开源项目已经在 Github 上获得了不少的关注。该项目的简介是这样描述自己的：Moco 是一个简单搭建 stub 的框架，主要用于测试和集成。这个框架的开发灵感来自 Mock 框架，如 Mockito 和 Playframework。
为什么要开发这个框架？
集成，尤其是基于 HTTP 协议的集成——web service、REST 等，在我们的项目开发中被广泛应用。以前，我们每次都要往 Jetty 或 Tomcat 等应用服务器上部署一个新的 WAR。大家都知道，开发部署一个 WAR 的过程是很枯燥的，即使在嵌入式服务器上也是如此。而且，每次我们做一点改动，整个 WAR 都要重新组装。
Moco 的出现，正是为了解决这些问题。开发团队只要根据自己的需要进行相应的配置，就会很方便得到一个模拟服务器。而且，由于 Moco 本身的灵活性，其用途已经不再局限于最初的集成测试，比如，Moco 可以用于移动开发，模拟尚未开发的服务；Moco 还可以用于前端开发，模拟一个完整的 Web 服务器，等等。

## 一个简单的开始
> 国际惯例，先从hello world开始我们的moco之旅吧。   

_首先，您需要：_   
1. 配置您的 Java 环境
    下载 Java 程序，并设置好系统环境变量 （PATH， JAVA_HOME）
2. 安装并配置 Gradle
    具体可以参考http://www.gradle.org/

_接下来，按照下面的步骤安装 Moco_   
1. 获取 Moco 源文件
    * 使用 git 命令，获取最新的代码
```
git clone https://github.com/dreamhead/moco.git
```
    * 也可以直接下载编译好的 Jar 文件，目前最新版本0.10.2
    

