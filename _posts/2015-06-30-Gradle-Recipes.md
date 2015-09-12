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
理论上它可以做任何groovy语言可以做的事情。但更多的时候，人们用它来完成构建。


## 单纯的gradle

## java && gradle

## android && gradle

###  打包签名

gradle 默认输出 release apk 是没有签名的，那么我们需要签名的很简单，只需要在android{}里面补充加上加上即可。
    
    signingConfigs {
       myConfig{
         storeFile file("gradle.keystore")
            storePassword "gradle"
            keyAlias "gradle"
            keyPassword "gradle"
        }
    }
        
       buildTypes{
         release {
            signingConfig  signingConfigs.myConfig
         } 
       }

然后，运行gradle clean gradle build ,这次在build/apk 你看到了多了一个[项目名]-release-unaligned， 从字面上面我就可以知道，这个只是没有进行zipAlign 优化的版本而已。而[项目名]-release 就是我们签名，并且zipAlign 的apk包了。
打混淆包的话，只需要在原来的基础上，添加上proguard相关的代码即可。

    buildTypes{
       release {
       signingConfig  signingConfigs.myConfig
         runProguard true
         proguardFile 'proguard-android.txt'
       }
    }

### ProjectFlavors
在各个版本改动不大，有时仅仅是替换一下applicationId，或者Mainfest中meta－data的值的时候，推荐使用这种方式。
不过这种方式能做的却不仅仅如此，它可以为每个flavor指定不同的sourceSet和dataSet，在compile的时候和main进行整合，得到功能完全不同的多个flavor的apk。

    android {
        compileSdkVersion 22
        buildToolsVersion "22.0.0"
        defaultConfig {
            applicationId "hyy.com.servicedemo"
            minSdkVersion 16
            targetSdkVersion 22
            versionCode 1
            versionName "1.0"
        }
        buildTypes {
            release {
                minifyEnabled false
                proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            }
        }
        productFlavors {
            flavor2 {
                applicationId "hyy.com.servicedemo.flavor2"
                versionCode 4
            };
            flavor1 {
                applicationId "hyy.com.servicedemo.flavor1"
                versionCode 20
            }
        }
    }

如上，productFlavors中定义了两个Flavor，他们可以复写defaultConfig中对项目的配置。
本例指定了应用的minSdkVersion分别为为4，和20（当然还可以配置更多的属性，具体可参考相关文档）。与此同时，Gradle还会为该flavor关联对应的sourceSet，默认位置为src/<flavorName>目录，对应到本例就是src/flavor1和src/flavor1。

接下来，要做的就是根据具体的需求在build.gradle文件中配置flavor，并添加必要的代码和资源文件。以flavor1为例，运行gradle assembleFlavor1命令既可生成所需的适配包。

##适配的场景：##
- 使用不同的包名
- 控制 BuildConfig 参数
- 使用不同的应用名
- 使用第三方sdk

[参考链接](http://tech.meituan.com/mt-apk-adaptation.html)

