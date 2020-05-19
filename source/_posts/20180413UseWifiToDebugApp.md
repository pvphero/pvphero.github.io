---
title: AndroidStudio利用ADB WIFI调试程序
date: 2018-04-13 18:41:18
tags:
- Android
- Debug
categories:
- Android开发笔记
---

# 前言
手机的USB口被占用,想调试手机简直要崩溃.还好现在AndroidStudio支持WIFI调试,记录下WIFI调试程序的步骤.

<!--more-->

![](https://ws2.sinaimg.cn/large/006tNc79ly1fqb7808s37j308c0b53zh.jpg)

# 步骤

- 首先打开手机的USB调试选项,并通过USB连接手机

- 打开Terminal,输入`adb tcpip 5555`

如果没有出现错误则会出现`restarting in TCP mode port: 5555`则说明是正确的.

![](https://ws3.sinaimg.cn/large/006tNc79ly1fqb7dk65n6j30gr0143ym.jpg)

- 再输入`adb connect <手机的WLAN IP>:5555` 如果回显`connected to <手机的WLAN IP>:5555` 则说明连接成功.

- 断开USB连接

我们在Logcat中可以看到调试信息了
![](https://ws1.sinaimg.cn/large/006tNc79ly1fqb7oy5y4lj30s80ggqa9.jpg)


如果要切回USB模式,输入
```
$ adb usb
```

回显`restarting in USB mode`,说明切换成功.


