---
layout: post
title:  Solution For Cant'T Stop Mysql Service On Mac10.10
date:   2015-07-29 18:07:56
categories: clay_created
tags: clay_created
image: /assets/article_images/desktop.JPG
---
# Mac10.10下Mysql服务不能停止的解决办法

## 先说一下这个问题的来源
十有八九是因为10.10之后mysql无法开机自动启动，于是网上有很多采用launchctl来添加自动重启的办法。网上的做法一般是这样的：   
在/Library/LaunchDaemons下，增加一个com.mysql.plist，代码如下：

    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
            <key>Label</key>
            <string>com.mysql</string>
            <key>ProgramArguments</key>
            <array>
                    <string>/usr/local/mysql/bin/mysqld_safe</string>
            </array>
            <key>RunAtLoad</key>
            <true/>
            <key>KeepAlive</key>
            <false/>
    </dict>
    </plist>

然后调用launchctl load -w com.mysql.plist这个来加载plist   

## 那么，重点来了，这样会有一个问题，就是没有办法停止mysql服务。  
解决的办法其实很简单，先launchctl list查看一下是否有mysql的自启动服务com.mysql.plist。  
如果有的话，调用launchctl unload －w com.mysql.plist卸载这个服务。  
