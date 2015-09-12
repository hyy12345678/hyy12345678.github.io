---
layout: post
title:  Android Multi Themes Implement
date:   2015-07-16 11:37:44
categories: clay_created
tags: clay_created
image: /assets/article_images/nightmare.JPG
---
# Android中如何使用主题属性

> 另外一种资源值允许你引用当前主题中的属性的值。这个属性值只能在样式资源和XML属性中使用；它允许你通过将它们改变为当前主题提供的标准变化来改变UI元素的外观，而不是提供具体的值。

## 属性的引用
- @表示引用资源,声明这是一个资源引用—随后的文本是以@[package:]type/name形式提供的资源名。 
- @android:string表明引用的系统的(android.*)资源 
- @string表示引用应用内部资源 
- 对于id, 可以用@+id表明创建一个id 
- ?表示引用属性   
    “？”引用主题属性，当您使用这个标记，你所提供的资源名必须能够在主题属性中找到，因为资源工具认为这个资源属性是被期望得到的，您不需要明确的指出它的类型   

举个例子： 

    <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
             android:layout_width="fill_parent"
             android:layout_height="wrap_content"
             android:clickable="false"
             android:background="?listview_item_background_selector"
             android:id="@+id/listview_root">   

这里我们使用了android:background="?listview_item_background_selector"来获取background的资源。但并没有给出资源的具体位置和类型。



## 属性的定义
在theme_attr.xml中，定义了需要使用的主题属性。
举个例子：

    <?xml version="1.0" encoding="utf-8"?>
    <resources>
    <attr name="account" format="reference"/>
    <attr name="account_management" format="reference"/>
    ...
    <attr name="listview_item_background_selector" format="reference"/>
    <attr name="listview_item_refresh_animation" format="reference"/>
    <attr name="line_drawable" format="reference"/>
    ...
    </resources>

如上，之前通过“?listview_item_background_selector”获取的属性的定义就是在这里给出的，不过也可以看出它的format为reference，所以需要到当前加载的属性中去查找相对应的具体值。  
这里没有使用扩展attr时常用的类似如下的写法： 
    
    <declare-styleable name="MyView">  
        <attr name="textColor" format="color" />  
        <attr name="textSize" format="dimension" />  
    </declare-styleable> 

是因为这样才可以在赋值时使用“?listview_item_background_selector”这种写法。   

declare-styleable下定义的属性，我们在获取它定义的属性是需要使用这种写法“R.sytleable.MyView_textColor”

## 属性的声明
在theme_xxx.xml中，声明了相应属性的具体取值。自定义style需要继承自系统包下原有的style。
举个例子：

    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <!--if you want to modify holo theme, you should modify PrivateThemeHoloOnlyForInherit style instead of AppTheme_Dark style-->
        <style name="AppTheme_Dark" parent="@style/PrivateThemeHoloOnlyForInherit">
            <item name="account">@drawable/account_light</item>
            <item name="account_management">@drawable/account_management_light</item>
            ...
            <item name="listview_item_background_selector">@drawable/listview_item_background_selector_dark</item>
            <item name="listview_item_refresh_animation">@drawable/refresh_light</item>
            ...
        </style>
    </resources>

至此，才真正指明了listview_item_background_selector属性的值：“@drawable/listview_item_background_selector_dark”。

##主题的加载和切换
资源工具会在当前加载的主题中自动寻找？xxx指定的属性的值，但是如何加载指定的主题为当前使用的主题呢？   

第一种，在manifest中application，activity中指定theme，比如这样   

    <application
            android:label="@string/app_name"
            android:icon="@drawable/ic_launcher"
            android:theme="@style/AppTheme_Light"

第二种，通过Context.setTheme()来设置：   
注意，这个方法的调用要在所有view在context中初始化之前，比如要早于Activity.setContentView
和LayoutInflater.inflate()之前。
一般可以在oncreate（）方法中，在调用super.oncreate（）方法之前调用setTheme(theme)。

## 参考资料
[Andriod中Style/Theme原理以及Activity界面文件选取过程浅析](http://www.apkbus.com/android-139495-1-1.html)
