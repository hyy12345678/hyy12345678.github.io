---
layout: post
title:  解决离线使用AndroidStudio的办法，通过添加本地Maven依赖
date:   2015-09-25 14:39:32
categories: clay_created
tags: clay_created
image: /assets/article_images/nightmare.JPG
---

公司的网络虽然有http代理，但是通过AndroidStudio访问基本不可行，导致gradle在处理依赖的时候出现长时间卡在：
Resolve dependencies “XXXComplie”...
解决的方法有3种：
1.挂VPN
2.采用自建maven私服
一般上网搜一搜都能查到这两种方式，但是对于自己来说前者太贵，后者不够灵活。

于是想到了第3种方法，本地添加Maven资源，在gradle中使用
  ```
  repositories {
          mavenLocal()
  }
  ```

相关的依赖资源可以通过浏览器下载到本地，jcenter或者MavenCentral都可以。
注意，需要把相关资源的pom表示也拷贝出来，这个后面本地install时会用到。一般长这个样子：
  ```
  <dependency>
  <groupId>com.j256.ormlite</groupId>
  <artifactId>ormlite-core</artifactId>
  <version>4.48</version>
  </dependency>
 ``` 
 

设置好本地maven的setting，确保本地repository可用。没有创建.m2目录的，可以用mvn help:system初始化一下。
这里推荐使用全局的setting，即在用户目录下.m2文件夹下的那个，如果没有可以从M2_HOME/conf目录下拷贝一个过去。

添加本地maven资源的方法如下：

Maven 安装 JAR 包的命令是：
  ```
  mvn install:install-file -Dfile=jar包的位置 -DgroupId=上面的groupId -DartifactId=上面的artifactId -Dversion=上面的version -Dpackaging=jar
  ```

例如：
我下载的这个 jar 包是放到了 D:\mvn 目录下(D:\mvn\spring-context-support-3.1.0.RELEASE.jar)
那么我在 cmd 中敲入的命令就应该是：

    mvn install:install-file -Dfile=D:\mvn\spring-context-support-3.1.0.RELEASE.jar -DgroupId=org.springframework -DartifactId=spring-context-support -Dversion=3.1.0.RELEASE -Dpackaging=jar

最后，就可以在现有的网络环境下愉快的使用Android Studio了。

ps:补充一些关于加载本地aar包的方法的内容

项目中使用的基础类库是aar包，之前的配置是通过上传到Maven私服，然后在gradle的build里这样引用：
      ```
      compile 'com.xxx.yy.android:base-core-debug:1.0.0@aar'
      compile 'com.xxx.yy.android:base-ui-debug:1.0.0@aar'
      compile 'com.xxx.yy.android:base-net-debug:1.0.0@aar'
      compile 'com.xxx.yy.android:base-database-debug:1.0.0@aar'
      ```
但是由于是私服，回到家里便没有办法访问。于是有了下边的办法：
      在project级别的build.gradle这样配置一个repository(在buildscript和allprojects中都要添加):
       ```
       flatDir {
            dirs 'goldlibs'
        }
      ```
      这里的goldlibs是建立在项目工程下的一个目录，和src同级。
      
      然后在module级别的build.gradle这样引用：
      ```
      compile(name: 'core-debug', ext: 'aar')
      compile(name: 'database-debug', ext: 'aar')
      compile(name: 'net-debug', ext: 'aar')
      compile(name: 'ui-debug', ext: 'aar')
      ```
      
      刷新gradle，这样就可以引入本地的aar包了。


