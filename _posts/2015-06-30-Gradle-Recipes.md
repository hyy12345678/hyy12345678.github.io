---
layout: post
title:  Gradle Recipes
date:   2015-06-30 16:08:36
categories: clay_created
tags: clay_created
image: /assets/article_images/desktop.JPG
---
# Gradle学习笔记
> 转向AS的时候遇到的最大困难其实是gradle的使用，为了学习AS有必要先对gradle有一个比较深入的理解，不然就变成了只知其然，不知其所以然，显然这不是一个负责任的态度。

## gradle是什么
不能免俗，也要先写两句，这里只写我自己的理解。
gradle是什么？从2011年开始很火的一个开源的构建框架，用groovy写的，备受业界推崇，连Google也把它作为Android官方IDE中的构建工具，取代了之前ADT中的Ant。
提起构建，从shell脚本，make，ant，maven到现在的gradle，一切似乎是个循环，从最开始的随便写（也可以认为是可编程），到后来的系统化，xml规范化，中央仓库解决依赖重复，这时人们又似乎被束缚了手脚，又开始羡慕洪荒时代那种像写代码一样做配置的快感，于是gradle出现了。

## gradle能做什么
理论上它可以做任何groovy语言可以做的事情。但更多的，人们用它来完成构建。
