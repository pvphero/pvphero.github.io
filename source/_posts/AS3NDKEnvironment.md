---
title: AndroidStudio 3.0 NDK环境搭建
date: 2018-02-08 18:59:33
tags:
- Android
- NDK
categories:
- Android开发笔记
---

# 前言
网上关于NDK开发环境配置的相关博客已经很多,但是很少有关于NDK在AdnroidStduio 3.0以上的环境搭建相关的博客.所以特此记录下.
首先是对NDK的介绍,先对NDK有个初步的印象:
NDK(Native Development kit)是一个工作集,为了能让开发者可以更加直接的接触Android硬件资源和方便地使用传统的C/C++语言编写代码，NDK应运而生。在NDK公布以前，Android平台的第三方应用程序的编写只能依靠基于Java的Dalvik虚拟机进行开发。有了NDK后，开发者就可以更加方便的用传统的C/C++语言编写程序，并在程序封装文件（.apk）中直接嵌入
<!--more-->
- NDK提供了一系列的工具，帮助开发者快速开发C（或C++）的动态库，并能自动将so和java应用一起打包成apk。这些工具对开发者的帮助是巨大的。
- NDK集成了交叉编译器，并提供了相应的mk文件隔离CPU、平台、ABI等差异，开发人员只需要简单修改mk文件（指出“哪些文件需要编译”、“编译特性要求”等），就可以创建出so。
- NDK可以自动地将so和Java应用一起打包，极大地减轻了开发人员的打包工作。

# 下载工具
1. NDK: [NDK Downloads](https://developer.android.com/ndk/downloads/index.html)
2.  CMake:一个外部构建工具(AndroidStudio 3.0以上自带)
3.  LLDB:用于调试本地代码

**这些工具都可以使用SDM Manager下载**

**当前博客所用的列子的环境如下**

| 名称     |    版本号|
| :--------: | :--------:|
| AndroidStudio  | 3.0|
| JDK     |   jdk1.8.0_101 |
| NDk      |  16.1.4479499|
|compileSdkVersion|26|
|buildToolsVersion|26.1.0|
|minSdkVersion|21|
|targetSdkVersion|26|

# 创建一个NDK项目

## new一个项目,并勾选include c++ support
![20180208164209411](http://paynnyvep.bkt.clouddn.com/20180208164209411.png)

AndroidStudio 3.0上创建NDK项目的时候,记得勾选include c++ support,这样会很方便.接着一路next 最后点击finish.

![20180208164502140](http://paynnyvep.bkt.clouddn.com/20180208164502140.png)

这样一个NDK项目就已经创建好了,目录结构以及代码如下

![20180208164857742](http://paynnyvep.bkt.clouddn.com/20180208164857742.png)


可以看出来AS3.0勾选了include c++ support会比正常的项目多出来cpp文件夹跟CmakeLists.txt文件.这些将是我们在AS3.0上学习NDK环境开发的**重点**

这样一个NDK的项目就完全建好了,并且可以运行,我们来看下运行的效果
![20180208165533804](http://paynnyvep.bkt.clouddn.com/20180208165533804.png)

哈哈,是不是很方便?如果之前没有勾选include c++ support 就会很麻烦,这个可以在以后的博客里说下旧的NDK项目怎么在3.0上运行,3.0已经不同于2.0,现在先享受下AS给我们带来的方便吧~

我们的项目里面Jni文件肯定不止一个,如果需要新的Jni文件的话,请按下面的步骤来

## NDK自定义配置过程
### 创建一个Java文件
![20180208171048688](http://paynnyvep.bkt.clouddn.com/20180208171048688.png)

### 在这个类里面写一个native方法
![20180208171842166](http://paynnyvep.bkt.clouddn.com/20180208171842166.png)

### 生成头文件(*.h)
打开Terminal,给刚刚创建的类创建头文件.先cd到app/src/main/java目录下
```
cd app/src/main/java

```
然后使用javah命令

``` java
javah -d ../cpp com.vv.ndk.Hello

```

**javah** 执行java命令生成头文件(*.h)
**-d**  在当前目录下创建一个文件夹,文件夹名字是cpp
**com.vv.ndk.Hello** 包名.类名指定要生成那个java类文件的头文件
所以这个命令的目的是在cpp的上一层目录下创建一个cpp文件夹,并对`com.vv.ndk.Hello` 生成一个头文件,如下图所示:

![20180208174446624](http://paynnyvep.bkt.clouddn.com/20180208174446624.png)

这个命令输入完以后在app/src/main/cpp/文件夹下多了一个`com_vv_ndk_Hello.h`文件,并且这个文件里面有一个未实现的方法`Java_com_vv_ndk_Hello_sayHello`,这个方法就是Hello.java方法里面对应的**sayHello()方法**

![20180208174928653](http://paynnyvep.bkt.clouddn.com/20180208174928653.png)

### 创建c文件并实现头文件里面的方法
`Java_com_vv_ndk_Hello_sayHello` 这个文件是一个抽象的方法,我们需要创建一个*.c文件去实现这个方法.

-  我们在cpp文件夹下创建一个C++ Source File,命名为Hello.cpp
![20180208183405701](http://paynnyvep.bkt.clouddn.com/20180208183405701.png)

- 引入`com_vv_ndk_Hello.h` 头文件,并实现头文件里面的`Java_com_vv_ndk_Hello_sayHello` 方法

![20180208183623312](http://paynnyvep.bkt.clouddn.com/20180208183623312.png)

- 返回一个我们想要得到的值,代码如下

```java
#include "com_vv_ndk_Hello.h"
JNIEXPORT jstring JNICALL Java_com_vv_ndk_Hello_sayHello(JNIEnv *env, jclass jclass1){
    return env->NewStringUTF("say Hello");
}
```

### Java文件里面加入静态方法块

![20180208184000363](http://paynnyvep.bkt.clouddn.com/20180208184000363.png)

`System.loadLibrary("hello")` 是NDK的**moduleName**

### 配置grade
在`app.gradle` 文件的defaultConfig里面加上 ndk 的moduleName
```
defaultConfig {
        ...
        ndk{
            moduleName "hello"
        }
    }
```

### 在Activity里面调用Jni
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TextView tv = (TextView) findViewById(R.id.sample_text);
        tv.setText(Hello.sayHello());
    }
}
```

运行以后发现崩溃,不要慌

![20180208184821441](http://paynnyvep.bkt.clouddn.com/20180208184821441.png)

崩溃是因为你还没有配置最关键的CmakeLists.txt文件

### 配置CmakeLists.txt文件
```
add_library( # Sets the name of the library.
             hello

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/jni/Hello.cpp
             src/main/jni/Hi.cpp
             )
```

```
target_link_libraries( hello ${log-lib} )
```

### ok,运行
效果如下

![20180208185518666](http://paynnyvep.bkt.clouddn.com/20180208185518666.png)

这样调用Hello.sayHello()方法就显示出来了.

## 总结
以上是创建NDK项目的过程,现在我们来总结下创建的整个步骤.

 1. 创建一个Java文件
 2. 在这个类里面写一个native方法
 3. 生成头文件(*.h)
 4. 创建c文件并实现头文件里面的方法
 5. Java文件里面加入静态方法块
 6. 配置grade
 7. 在Activity里面调用Jni
 8. 配置CmakeLists.txt文件

博客就到这里吧,下面将会讲下CMakeLists的解析,看看这个文件到底是个啥.

