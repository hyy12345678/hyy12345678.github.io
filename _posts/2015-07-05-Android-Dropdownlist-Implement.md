---
layout: post
title:  Android Dropdownlist Implement
date:   2015-07-05 17:09:52
categories: clay_created
tags: clay_created
image: /assets/article_images/dawn.JPG
---

# 一个自定义下拉刷新ListView的实现     
<b/>
> 原理很简单：通过对ListView添加了一个刷新layout，滚动中时不断改变header的高度和内容并记录一些状态，在用户手指离开屏幕时根据状态决定进行刷新还是放弃刷新。

主要是通过重写ListView的onTouchEvent和OnScrollListener的onScrollStateChanged、onScroll函数实现
先介绍下刷新状态共有四种，如下：
CLICK_TO_REFRESH 点击刷新状态，为初始状态
DROP_DOWN_TO_REFRESH 当刷新layout高度低于一定范围时，为此状态
RELEASE_TO_REFRESH 当刷新layout高度高于一定范围时，为此状态
REFRESHING 刷新中时，为此状态
 
### 2.1 onTouchEvent函数
public boolean onTouchEvent(MotionEvent event)根据用户在屏幕上的move事件，进行相应操作，如下：

- ACTION_DOWN表示用户手指刚接触屏幕，会记录用户此时touch的点的y坐标，在下面调整高度时使用
- ACTION_MOVE表示用户手指正在屏幕上移动，此时会不断调整header的高度，即下拉时刷新item部分高度的不断变化
- ACTION_UP表示用户手指离开屏幕，此时会根据当前状态决定是进行刷新还是放弃刷新，若刷新调用用户设置的OnRefreshListener接口。
 
### 2.2 onScrollStateChanged函数
public void onScrollStateChanged(AbsListView view, int scrollState)
记录listView当前的滚动状态到currentScrollState，包括三种状态：

- SCROLL_STATE_TOUCH_SCROLL ListView正在滚动中，并且手指尚未离开屏幕
- SCROLL_STATE_FLING ListView仍在滚动中，但用户手指已经离开屏幕
- SCROLL_STATE_IDLE ListView已经停止滚动
 
### 2.3 onScroll函数
public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount)
根据listView当前的滚动状态即currentScrollState和当前刷新的状态不断修改header内容显示和刷新状态，如下：

**ListView为SCROLL_STATE_TOUCH_SCROLL状态(按着不放滚动中)并且刷新状态不为REFRESHING**  

*  a. 刷新对应的item可见时，若刷新layout高度超出范围，则置刷新状态为RELEASE_TO_REFRESH；若刷新layout高度低于高度范围，则置刷新状态为DROP_DOWN_TO_REFRESH。
*  b. 刷新对应的item不可见，重置header
 
**ListView为SCROLL_STATE_FLING状态(松手滚动中)**  

- a. 若刷新对应的item可见并且刷新状态不为REFRESHING，设置position为1的(即第二个)item可见
- b. 若反弹回来，设置position为1的(即第二个)item可见