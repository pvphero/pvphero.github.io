---
title: Fragment生命周期
date: 2018-03-13 13:47:49
tags:
- Android
- fragment
categories:
- AndroidInterview
---
整理一份安卓开发很基础的内容,但是很实用,帮助不了解Fragment在Activity里面生命周期的朋友.
Fragment从Android v3.0版本开始引用,已经算的上是很老的组件了.我们都知道安卓有最基本的四大组件:
`Activity`,`Service`,`ContentProvider`,`broadcastReceiver `.`Fragment`可以称为Android的第五大组件.肯定会有人问为什么第五大组件不是View?**大家可以发现,说的四大组件都有自己的生命周期,而View没有自己的生命周期,所以大家不要把这个概念给混淆了.**
<!--more-->
接下来我们就来谈谈Fragment的生命周期.为了方便大家理解跟记忆,我用一张图来说明,Fragment在Activity里面的生命周期是什么样的.如下图:

![图片](https://dn-coding-net-production-pp.qbox.me/de6215bc-2e4d-43db-bc39-45526fe33a01.png)
