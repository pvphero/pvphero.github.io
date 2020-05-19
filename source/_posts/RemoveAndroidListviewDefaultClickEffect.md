---
title: 怎么去除android listview的默认点击效果
date: 2016-05-12 17:18:21
tags:
- Android 
- ListView
categories:
- Android开发笔记
---

在做项目的时候会遇到listview点击item的时候出现黄色的默认的点击效果.但是项目中不要出现这个效果.贴上正确的方法,只需要在listview的布局文件中加上
<!--more-->
``` java
 android:listSelector="@android:color/transparent" 
```
做个笔记记下来~~
