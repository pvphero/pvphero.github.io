---
title: Android使用Intent跳转的时候,如何清除堆栈里所有的Activity
date: 2016-04-27 11:27:17
tags:
- Android 
- Intent
- setFlags
categories:
- Android开发笔记
---

我在开发一块下单送货软件的时候遇到的这个问题.场景描述,用户从首页选择里订单,进入了订单确认页面,确认后进入了选择支付页面,支付成功以后需要返回首页.如何在进入首页的时候清除堆栈里所有的Activity?说说有效的方法吧.FLAG_ACTIVITY_CLEAR_TOP
<!--more-->
``` java
Intent intent = new Intent(A.this,B.class).setFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK | Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);

```

**看看里面的安卓源码**
``` java
 /**
     * If set in an Intent passed to {@link Context#startActivity Context.startActivity()},
     * this flag will cause any existing task that would be associated with the
     * activity to be cleared before the activity is started.  That is, the activity
     * becomes the new root of an otherwise empty task, and any old activities
     * are finished.  This can only be used in conjunction with {@link #FLAG_ACTIVITY_NEW_TASK}.
     */
    public static final int FLAG_ACTIVITY_CLEAR_TASK = 0X00008000;
```

源码中明确说明如果在startActivity的时候传递FLAG_ACTIVITY_CLEAR_TASK这个标志,那么这个标志将会清除之前所有已经打开的activity.然后将会变成另外一个空栈的root,然后其他的Activitys就都被关闭了.这个方法必须跟着{@link #FLAG_ACTIVITY_NEW_TASK}一起使用.


## 我的博客地址
[GitHub Blog](http://pvphero.github.io/)
[Coding Blog](http://shenzhenwei.coding.me/)
[CSDN Blog](http://blog.csdn.net/pvpheroszw)
