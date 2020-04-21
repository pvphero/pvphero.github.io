---
title: 'Android IllegalStateException: Fragment not attached to Activity' 
date: 2016-07-04 15:22:29
tags:
- Android 
- fragment
categories:
- Android开发笔记
---

``` java
java.lang.IllegalStateException
Fragment QuestionCollectSimpleFragment{42283040} not attached to Activity
```
<!--more-->
## 问题的产生
项目中,加载一个fragment,然后迅速点击系统自带的返回或者自带的返回按钮弹出提示时自动退出.
## 异常分析
定位代码发现,该问题的产生的原因是在调用资源文件getResource()时发生的crash.

该问题产生的原因是因为fragment加载的时候还没有Attach到他所被管理的activity上就去加载Resource导致的.
## 解决方法
在调用getResource()方法时判断下改fragment是否attach到他所管理的activity上.使用isAdded() 方法.
``` java
if(isAdded()){
    getResources().getString(R.string.xxx);
}
```
