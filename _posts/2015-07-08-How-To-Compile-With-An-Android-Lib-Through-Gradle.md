---
layout: post
title:  How To Compile With An Android Lib Through Gradle
date:   2015-07-08 03:48:58
categories: clay_created
tags: clay_created
image: /assets/article_images/nightmare.JPG
---
# 通过Gradle怎样创建并编译一个Android Lib工程
<br/>
### 怎样创建一个Android Lib（库项目)
创建一个Android Lib工程和创建一个Android Application工程没有太大的不同。但还是有一点小区别的。
他们使用了不同的Plugin，但是在内部这些plugin共享了大部分的代码，并且他们都是由相同的com.android.tools.build.gradle.jar提供的。

    apply plugin: 'com.android.library'
    android {
        compileSdkVersion 22
        buildToolsVersion "22.0.0"
        defaultConfig {
            minSdkVersion 15
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
    }
    dependencies {
        compile fileTree(dir: 'libs', include: ['*.jar'])
        compile 'com.android.support:appcompat-v7:22.1.1'
    }


这里创建了一个使用API－22编译SourceSet的库项目。

### 普通项目和库项目的区别
一个库项目的输出是一个.aar包，它打包了编译代码（例如jar包或者本地so文件）和资源（mainifest，res，asset），一个库项目也可独立于应用程序声称一个APK。
标识Task同样适用于库项目（assembleDebug,assembleRelease）.

### 引用一个库项目的方法
引用一个库项目和引用一个普通项目的方法是一样的。
{% highlight groovy %}

    dependencies {
        compile fileTree(dir: 'libs', include: ['*.jar'])
        compile 'com.android.support:appcompat-v7:22.1.1'
        compile(project(':libraries:hyy-android-common-lib')) {
            exclude group: 'com.android.support', module: 'appcompat-v7'
    }
{% endhighlight %}

### 库项目的发布
一般情况下一个库只会发布它的release variant版本。这个版本会被所有引用它的项目使用。

