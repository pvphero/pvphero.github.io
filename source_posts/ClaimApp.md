---
title: Mac下关于百度开发者中心认领应用
date: 2016-07-11 17:18:21
tags:
- Android 
categories:
- Android开发笔记
---

# 问题的产生
发布一款应用有时因为签名问题,难免会遇到让开发者认领应用的情况.比如说我们公司发布的一款应用发布到360应用市场上,但是却被百度抓包,在我们把应用提交到百度应用平台之前,已经发布...这个时候就需要我们去找回应用了.认领应用其实很简单,无非就是给空包签名.但是有很琐碎,干脆记录下来,以后在遇到直接cv~~
<!--more-->
# 怎么认领
- 提交需认领应用的PackageName。
- 下载应用中心提供的未签名包，并将与待认领应用一致的签名写入该包中。
- 上传完成签名的安装包进行校验。

# 怎么签名

```java
jarsigner -verbose -keystore your_keystore -signedjar path_of_signed_apk  path_of_unsigned_apk your_alias
```
**更改上面引用中的值**

 - your_keystore 你的这个应用签名的keystore的位置
 - path_of_signed_apk  签名好的空包存放位置
 - path_of_unsigned_apk 代签名的空包的位置
 - your_alias 你的keystore的别名
 
 # 举例
 **比如我的一个应用**
``` 
jarsigner -verbose -keystore /Users/seekkou/Desktop/android_studio_key.jks -signedjar /Users/seekkou/Desktop/Baidu_Claim_signed.apk /Users/seekkou/Desktop/Baidu_Claim_unsigned.apk key
```

参考:[应用认领那些事](http://droidyue.com/blog/2014/12/14/android-yingyong-renling/?utm_source=tuicool&utm_medium=referral)
