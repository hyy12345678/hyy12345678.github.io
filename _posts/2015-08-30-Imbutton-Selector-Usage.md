---
layout: post
title:  Imbutton Selector Usage
date:   2015-08-30 12:20:50
categories: clay_created
tags: clay_created
image: /assets/article_images/dawn.JPG
---
# 图片按钮(ImageButton)可以根据当前按钮状态来显示不同的图片。 


二、代码要点 

1. 一般通过在<ImageButton>节点里设置android:src属性来进行显示设置图片源。 

2. 若想去掉原来按钮的背景，则通过设置图片背景为透明实现。(android:background="#00000000") 

main.xml 
 
    <?xml version="1.0" encoding="utf-8"?>    
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"    
        android:orientation="vertical"    
        android:layout_width="fill_parent"    
        android:layout_height="fill_parent"    
        >    
    <TextView      
        android:layout_width="fill_parent"     
        android:layout_height="wrap_content"     
        android:text="@string/hello"    
        />    
    <ImageButton    
        android:layout_width="wrap_content"     
        android:layout_height="wrap_content"    
        android:background="#00000000"     
        android:src ="@drawable/img_btn">    
    </ImageButton>    
    </LinearLayout>    


3. 为不同的状态设置不同的图片，通常的做法是定义一个XML(selector)。注意：<item>的排列是有序的，默认状态(default)的图片放在最后，它要在按下状态(btn_pressed)和焦点状态(btn_focused)都为False时，默认状态(default)才生效。 

img_btn.xml 
  
    <?xml version="1.0" encoding="utf-8"?>     
     <selector xmlns:android="http://schemas.android.com/apk/res/android">     
         <item android:state_pressed="true"     
               android:drawable="@drawable/btn_pressed" /> <!-- pressed -->     
         <item android:state_focused="true"     
               android:drawable="@drawable/btn_focused" /> <!-- focused -->     
         <item android:drawable="@drawable/btn_default" />  <!-- default -->     
     </selector>    
