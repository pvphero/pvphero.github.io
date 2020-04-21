---
title: 使用命令行给Git项目加上tag
date: 2016-07-04 14:53:04
tags:
- Android Studio
- Git 
categories:
- Git 
---

我们在提交git项目的时候,有时候需要给项目的版本加上标签,我们需要用到以下命令给代码加上标签.
<!--more-->
```
$ git tag -a "V2.3.3" -m "change version name" 
$ git push --tags
```
**运行结果**
```
Counting objects: 1, done.
Writing objects: 100% (1/1), 168 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@123.57.148.160:/home/git/android/GradleCommonTeacher.git
 * [new tag]         V2.3.3 -> V2.3.3

```
**然后我们看一下提交工作流**
```
$ gitk
```
![这里写图片描述](http://img.blog.csdn.net/20160704145003965)

ok~可以看到tag已经提交上去了~

