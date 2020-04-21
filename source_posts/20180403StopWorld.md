---
title: StopWorld组织成立
date: 2018-04-03 23:40:40
tags:
- Android
- Interview
---

[StopWorld](https://github.com/StopWorld)组织在今天成立了,成立的缘由是翔哥提出来要维护一套面试的项目.为啥起这个名字呢?翔哥说JVM有个很牛逼的函数,这个函数就叫StopWorld,听起来很酷炫,有种逆天的感觉,哈哈~

> 回到垃圾回收上，在开始学习GC之前你应该知道一个词：stop-the-world。不管选择哪种GC算法，stop-the-world都是不可避免的。Stop-the-world意味着从应用中停下来并进入到GC执行过程中去。一旦Stop-the-world发生，除了GC所需的线程外，其他线程都将停止工作，中断了的线程直到GC任务结束才继续它们的任务。GC调优通常就是为了改善stop-the-world的时间。

成立这个组织的最初目的是为了维护一套便于我们回顾一些知识的面试题,简单说下我们的项目成员:翔哥,浩哥,鑫鑫
<!--more-->

目前我们的项目初具规模

![](https://ws3.sinaimg.cn/large/006tKfTcly1fpzwpfxlt9j30rs0eoq53.jpg)

先贴出部分的Android的内容,以后会持续更新

# Android基础

<details>
<summary>目录</summary>


- [1.了解Fragment么,说说Fragmment的生命周期](#1-了解fragment么,说说fragmment的生命周期)
- [2.安卓的事件分发机制](#2-安卓的事件分发机制)
- [3.Android重要术语解释](#3-android重要术语解释)
- [4.Android启动模式](#4-android启动模式)
- [5.Android IPC:Binder原理](#5-android-ipc-binder原理)

</details>

## 知识点

### 1 了解Fragment么,说说Fragmment的生命周期
<details>
<summary>展开查看答案</summary>

一张图概括,详细请看博客链接[Fragment生命周期](http://pvphero.github.io/2018/03/13/20180313AndroidInterViewFragment/)
![图片](https://dn-coding-net-production-pp.qbox.me/de6215bc-2e4d-43db-bc39-45526fe33a01.png)
</details>

### 2 安卓的事件分发机制
<details>
<summary>展开查看答案</summary>

   - Android事件的基础知识：
   所有的Touch事件都封装到MotionEvent里面
   事件处理包括三种情况，分别为：`传递—-dispatchTouchEvent()函数`、`拦截—-
   onInterceptTouchEvent()函数`、`消费—-onTouchEvent()函数`和`OnTouchListener`
   事件类型分为`ACTION_DOWN`, `ACTION_UP`, `ACTION_MOVE`, `ACTION_POINTER_DOWN`,
   `ACTION_POINTER_UP`, `ACTION_CANCEL`等
   每个事件都是以`ACTION_DOWN`开始`ACTION_UP`结束

   - Android事件传递流程：
     1. 事件都是从`Activity.dispatchTouchEvent()`开始传递
     2. 事件由父View传递给子View，ViewGroup可以通过`onInterceptTouchEvent()`方法对事件拦截，
     停止其向子view传递
     3. 如果事件从上往下传递过程中一直没有被停止，且最底层子View没有消费事件，**事件会反向往上传递
     **,这时父View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会
     `Activityon.TouchEvent()`函数。
     4. 如果View没有对ACTION_DOWN进行消费，之后的其他事件不会传递过来，也就是说ACTION_DOWN必须
     返回true，之后的事件才会传递进来OnTouchListener优先于onTouchEvent()对事件进行消费

 - 三张效果图辅助理解
    **View不处理事件流程图（View没有消费事件)**

    ![](https://ws4.sinaimg.cn/large/006tKfTcly1fpzlduduzzj31ga0y8jxd.jpg)


    **View处理事件**

    ![](https://ws3.sinaimg.cn/large/006tKfTcly1fpzlek9w6cj31gs0xygrl.jpg)


    **事件拦截**

    ![](https://ws1.sinaimg.cn/large/006tKfTcly1fpzlexpyxjj31ge0xkn31.jpg)


    > [Android-三张图搞定Touch事件传递机制](http://hanhailong.com/2015/09/24/Android-三张图搞定Touch事件传递机制/)

    </details>

### 3 Android重要术语解释

<details>
<summary>展开查看详细</summary>

* 1.ActivityManagerServices，简称AMS，服务端对象，负责系统中所有Activity的生命周期
* 2.ActivityThread，App的真正入口。当开启App之后，会调用main()开始运行，开启消息循环队列，这就是传说中的UI线程或者叫主线程。与ActivityManagerServices配合，一起完成Activity的管理工作
* 3.ApplicationThread，用来实现ActivityManagerService与ActivityThread之间的交互。在ActivityManagerService需要管理相关Application中的Activity的生命周期时，通过ApplicationThread的代理对象与ActivityThread通讯。
* 4.ApplicationThreadProxy，是ApplicationThread在服务器端的代理，负责和客户端的ApplicationThread通讯。AMS就是通过该代理与ActivityThread进行通信的。
* 5.Instrumentation，每一个应用程序只有一个Instrumentation对象，每个Activity内都有一个对该对象的引用。Instrumentation可以理解为应用进程的管家，ActivityThread要创建或暂停某个Activity时，都需要通过Instrumentation来进行具体的操作。
* 6.ActivityStack，Activity在AMS的栈管理，用来记录已经启动的Activity的先后关系，状态信息等。通过ActivityStack决定是否需要启动新的进程。
* 7.ActivityRecord，ActivityStack的管理对象，每个Activity在AMS对应一个ActivityRecord，来记录Activity的状态以及其他的管理信息。其实就是服务器端的Activity对象的映像。
* 8.TaskRecord，AMS抽象出来的一个“任务”的概念，是记录ActivityRecord的栈，一个“Task”包含若干个ActivityRecord。AMS用TaskRecord确保Activity启动和退出的顺序。如果你清楚Activity的4种launchMode，那么对这个概念应该不陌生。

</details>

### 4 Android启动模式

<details>
<summary>展开查看答案</summary>

1. standard:默认标准模式，每启动一个都会创建一个实例
2. singleTop：栈顶复用，如果在栈顶就调用onNewIntent复用，从onResume()开始
3. singleTask：栈内复用，本栈内只要用该类型Activity就会调到栈顶复用，从onResume()开始
4. singleInstance：单例模式，除了3中特性，系统会单独给该Activity创建一个栈

</details>

### 5 Android IPC Binder原理

<details>
<summary>展开查看答案</summary>

1. 在Activity和Service进行通讯的时候，用到了Binder。
  1. 当属于同个进程我们可以继承Binder然后在Activity中对Service进行操作
  2. 当不属于同个进程，那么要用到AIDL让系统给我们创建一个Binder，然后在Activity中对远端的Service进行操作。
2. 系统给我们生成的Binder：
  1. Stub类中有:接口方法的id，有该Binder的标识，有asInterface(IBinder)(让我们在Activity中获取实现了Binder的接口，接口的实现在Service里，同进程时候返回Stub否则返回Proxy)，有onTransact()这个方法是在不同进程的时候让Proxy在Activity进行远端调用实现Activity操作Service
  2. Proxy类是代理，在Activity端，其中有:IBinder mRemote(这就是远端的Binder)，两个接口的实现方法不过是代理最终还是要在远端的onTransact()中进行实际操作。
3. 哪一端的Binder是副本，该端就可以被另一端进行操作，因为Binder本体在定义的时候可以操作本端的东西。所以可以在Activity端传入本端的Binder，让Service端对其进行操作称为Listener，可以用RemoteCallbackList这个容器来装Listener，防止Listener因为经历过序列化而产生的问题。
4. 当Activity端向远端进行调用的时候，当前线程会挂起，当方法处理完毕才会唤醒。
5. 如果一个AIDL就用一个Service太奢侈，所以可以使用Binder池的方式，建立一个AIDL其中的方法是返回IBinder，然后根据方法中传入的参数返回具体的AIDL。
6. IPC的方式有：Bundle（在Intent启动的时候传入，不过是一次性的），文件共享(对于SharedPreference是特例，因为其在内存中会有缓存)，使用Messenger(其底层用的也是AIDL，同理要操作哪端，就在哪端定义Messenger)，AIDL，ContentProvider(在本进程中继承实现一个ContentProvider，在增删改查方法中调用本进程的SQLite，在其他进程中查询)，Socket

</details>

