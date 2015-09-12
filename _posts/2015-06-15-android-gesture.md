---
layout: post
title:  "Android手势操作"
date:   2015-06-15 23:27:25
categories: Android
tags: Android
image: /assets/article_images/desktop.JPG
---
# Android手势操作
![](http://ww4.sinaimg.cn/bmiddle/aa397b7fjw1dzplsgpdw5j.jpg)
> 一盏灯， 一片昏黄； 一简书， 一杯淡茶。 守着那一份淡定， 品读属于自己的寂寞。 保持淡定， 才能欣赏到最美丽的风景！ 保持淡定， 人生从此不再寂寞。

## 前言

利用手势操作在现在的APP中越来越普及，大多数时候使用Fling，Scroll等Gesture能大幅度提高用户的操作体验，特别是大屏手机返回键程越来越大的现状下。
在Android系统下，手势识别是通过GestureDetector.OnGestureListener接口实现的，不过官方的文档可能觉得这部分太基础和简单了，所以官方的API文档中对手势的讲解描述的都很简单，API Demo中也没有提供一个清楚的例子，所以自己总结一下，其中还是涉及不少的基础知识和一些官方文档中说明不清的地方，如果不能好好掌握这些基础知识，做起事情来难免要吃一些苦头。言归正传，下面我们开始：

## 基础知识
我们先来明确一些概念，首先，Android的事件处理机制是基于Listener（监听器）来实现的，比我们今天所说的触摸屏相关的事件，就是通 过onTouchListener。其次，所有View的子类都可以通过setOnTouchListener()、 setOnKeyListener()等方法来添加对某一类事件的监听器。第三，Listener一般会以Interface（接口）的方式来提供，其中 包含一个或多个abstract（抽象）方法，我们需要实现这些方法来完成onTouch()、onKey()等等的操作。这样，当我们给某个view设置了事件Listener，并实现了其中的抽象方法以后，程序便可以在特定的事件被dispatch到该view的时候，通过callbakc函数给予适当的响应。
>这篇文章简单介绍了事件触发的过程：[Android事件分发图解](http://blog.csdn.net/yuyuanhuang/article/details/44627619)

## 实作
我们现在实作一个使用手势的例子。
我们给RelativeView的实例my_view设定了一个onTouchListener，因为GestureTest类实现了OnTouchListener 接口，所以简单的给一个this作为参数即可。onTouch方法则是实现了OnTouchListener中的抽象方法，我们只要在这里添加逻辑代码即 可在用户触摸屏幕时做出响应，

    public class MainActivity extends ActionBarActivity implements View.OnTouchListener,GestureDetector.OnGestureListener{
    GestureDetector gestureDetector = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        gestureDetector = new GestureDetector(this);
        gestureDetector.setIsLongpressEnabled(true);
        View v = findViewById(R.id.my_view);
        v.setOnTouchListener(this);
    }


    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

    @Override
    public boolean onTouch(View view, MotionEvent motionEvent) {
        Log.i("HYY","onTouch present");

        return gestureDetector.onTouchEvent(motionEvent);
    }

    @Override
    public boolean onDown(MotionEvent motionEvent) {
        Log.i("HYY","onDown present");
        return true;
    }

    @Override
    public void onShowPress(MotionEvent motionEvent) {
        Log.i("HYY","onShowPress present");
    }

    @Override
    public boolean onSingleTapUp(MotionEvent motionEvent) {
        Log.i("HYY","onSingleTapUp present");
        return false;
    }

    @Override
    public boolean onScroll(MotionEvent motionEvent, MotionEvent motionEvent2, float v, float v2) {
        Log.i("HYY","onScroll present");
        return false;
    }

    @Override
    public void onLongPress(MotionEvent motionEvent) {
        Log.i("HYY","onLongPress present");

    }

    @Override
    public boolean onFling(MotionEvent motionEvent, MotionEvent motionEvent2, float v, float v2) {
        Log.i("HYY","onFling present");
        return false;
    }
    }

## 特别提示
> 这里有一点需要特别注意:__要触发onScroll和onFling,必须让监听器的onDown的返回值设为true__

那么为什么如果不把onDown的返回值设为true的话，onScroll和onFiling就都不会被触发呢？我们来看一下源码中的介绍

<pre>
<code>
		/**
         * Notified when a scroll occurs with the initial on down {@link MotionEvent} and the
         * current move {@link MotionEvent}. The distance in x and y is also supplied for
         * convenience.
         *
         * @param e1 The first down motion event that started the scrolling.
         * @param e2 The move motion event that triggered the current onScroll.
         * @param distanceX The distance along the X axis that has been scrolled since the last
         *              call to onScroll. This is NOT the distance between {@code e1}
         *              and {@code e2}.
         * @param distanceY The distance along the Y axis that has been scrolled since the last
         *              call to onScroll. This is NOT the distance between {@code e1}
         *              and {@code e2}.
         * @return true if the event is consumed, else false
         */
        boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY);

		/**
         * Notified of a fling event when it occurs with the initial on down {@link MotionEvent}
         * and the matching up {@link MotionEvent}. The calculated velocity is supplied along
         * the x and y axis in pixels per second.
         *
         * @param e1 The first down motion event that started the fling.
         * @param e2 The move motion event that triggered the current onFling.
         * @param velocityX The velocity of this fling measured in pixels per second
         *              along the x axis.
         * @param velocityY The velocity of this fling measured in pixels per second
         *              along the y axis.
         * @return true if the event is consumed, else false
         */
        boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY);
</code>
</pre>
> theonScroll和onFling方法的注释上写的很清楚，要判定为onScroll或onFling必须要建立在onDown被消费的前提下（即onDown返回true）