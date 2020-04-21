---
title: AndroidStudio项目之CmakeLists解析
date: 2018-02-09 15:23:23
tags:
- Android
- NDK
categories:
- Android开发笔记
---

# 前言
我们在使用AndroidStudio 3.0开发NDK项目的时候CmakeLists.txt将是我们必须要用到的文件,如果你不懂怎么用CmakeLists配置NDK请先看之前的一篇博客:[AndroidStudio 3.0 NDK环境搭建](http://pvphero.github.io/2018/02/08/AS3NDKEnvironment/),如果已经了解CmakeLists配置NDK项目,ok,那我们接下来步入正题~
<!--more-->

# CmakeLists源码
`CMakeLists.txt`
```
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.

add_library( # Sets the name of the library.
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/native-lib.cpp )

# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
                       native-lib

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

源码很简单除了注释的代码外,核心的代码也就那么几句.

- `cmake_minimum_required(VERSION 3.4.1)`
>  Sets the minimum version of CMake required to build the native library.

 用来设置编译本地native library的时候需要的Cmake最小版本.这个是创建AndroidStudio项目的时候自动生成,不需要太在意.

-  `add_library()`

```
add_library( # Sets the name of the library.
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/native-lib.cpp )
```

**native-lib** : 设置本地lib的name
**SHARED** : 表示编译生成的是动态链接库
**src/main/cpp/native-lib.cpp** : 表示编译文件的相对路径,这里可以是一个文件的路径也可以是多个文件的路径

- `find_library()`

这个的作用是用来让我们加一些编译本地NDK库的时候所用的到一些依赖库.
**log-lib** 是这个库的别名,方便我们以后引用
**log** 是我们调试的时候打印log的一个库

- `target_link_libraries()`
这个的目的是用来关联我们本地的库跟第三方的库.这里就是把native-lib库和log库关联起来.

# 自定义NDK的配置
## 单个C/C++文件
这个在之前的博客里有提到,可以翻看[AndroidStudio 3.0 NDK环境搭建](http://pvphero.github.io/2018/02/08/AS3NDKEnvironment/)
## 多个C/C++文件
我们在实际项目中,C++文件可能不止一个,如果有多个C++文件,我们的CmakeLists应该怎么配置呢?其实前面说`add_library()` 的时候提到了,**路径可以是多个文件的路径**.
所以我们可以这么配置:
```
add_library( # Sets the name of the library.
               hello

               # Sets the library as a shared library.
               SHARED

               # Provides a relative path to your source file(s).
               src/main/cpp/Hello.cpp
               src/main/cpp/Hi.cpp
               )
```

只需要在路径处增加一个路径,就配置好了.可能只说这个,大家会有点迷茫,可以放在项目中来看下,我们基于上一篇文章的项目,实现这个多个NDK文件的配置过程.
先回忆一下创建NDK项目的步骤:

  1. 创建一个Java文件
  2. 在这个类里面写一个native方法
  3. 生成头文件(*.h)
  4. 创建c文件并实现头文件里面的方法
  5. Java文件里面加入静态方法块
  6. 配置grade
  7. 在Activity里面调用Jni
  8. 配置CmakeLists.txt文件

我们先创建一个Hi.java文件,并在Hi.java文件中写一个native方法,如下:
![](https://ws2.sinaimg.cn/large/006tNc79ly1foa151xv7vj30pa09ht9x.jpg)
生成Hi的头文件
```
$ cd app/src/main/java

```

```
$ javah -d ../cpp com.vv.ndk.Hi

```

创建一个Hi.cpp  c文件实现`com_vv_ndk_Hi.h` 头文件里面未实现的方法

![](https://ws2.sinaimg.cn/large/006tNc79ly1foa1dpjsjij30rs04nwg5.jpg)

``` c
#include "com_vv_ndk_Hi.h"
JNIEXPORT jstring JNICALL Java_com_vv_ndk_Hi_sayHi(JNIEnv *env, jclass jclass1){
    return env->NewStringUTF("sat Hi");
}
```

Hi.java文件中加入静态代码块
![](https://ws2.sinaimg.cn/large/006tNc79ly1foa1gb32v3j30mo09habc.jpg)

注意这个`System.loadLibrary` 加载的是你本地库的名字

配置CmakeLists.txt文件
```

cmake_minimum_required(VERSION 3.4.1)

add_library(
               hello
               SHARED
               src/main/cpp/Hello.cpp
               src/main/cpp/Hi.cpp
               )

find_library(
              log-lib
              log )
target_link_libraries(
                       hello
                       ${log-lib} )

```

activity里面调用
xml文件
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.vv.ndk.MainActivity">

    <TextView
        android:id="@+id/sample_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/textview"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
         />

</android.support.constraint.ConstraintLayout>

```

activity文件
```
public class MainActivity extends AppCompatActivity {

    private TextView textView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TextView tv = (TextView) findViewById(R.id.sample_text);
        tv.setText(Hello.sayHello());

        textView=findViewById(R.id.textview);
        textView.setText(Hi.sayHi());
    }
}
```
运行效果
![](https://ws1.sinaimg.cn/large/006tNc79ly1foa1p1d12yj308c0ett93.jpg)

可以看到调用Hi文件的`sayHi()` 方法已经被调用了.

## 编译多个SO库

编译多个so库的cpp目录结构
![](https://ws1.sinaimg.cn/large/006tNc79ly1foa6dkks6tj30jg0f9jvi.jpg)

`one` 文件夹内的`CmakeLists.txt` 配置如下:

```
add_library(one SHARED one.cpp)

target_link_libraries(one ${log-lib} )
```

`two` 文件夹内`CmakeLists.txt` 配置如下:

```
add_library(two SHARED two.cpp)

target_link_libraries(two ${log-lib} )
```

`app` 项目的`CmakeLists.txt` 配置如下:

```

cmake_minimum_required(VERSION 3.4.1)

add_library(hello
            SHARED
            src/main/cpp/Hello.cpp
            src/main/cpp/Hi.cpp)

find_library(log-lib log )
target_link_libraries(hello ${log-lib} )

ADD_SUBDIRECTORY(src/main/cpp/one)
ADD_SUBDIRECTORY(src/main/cpp/two)
```
CmakeLists.txt文件支持继承,所以我们只需要在子配置文件中写不同的配置项就可以完成相应的配置.最后需要在项目的CmakeLists.txt文件中增加子配置文件的路径.

然后我们用Make构建Module app生成字节码文件
![](https://ws3.sinaimg.cn/large/006tNc79ly1foa6nrer8qj30o30cuwip.jpg)
这样就可以在`/app/build/intermediates/cmake/debug/obj/arm64-v8a/` 路径下看到我们刚刚生成的so文件.
![](https://ws2.sinaimg.cn/large/006tNc79ly1foa6r2xq61j30an0cmjs0.jpg)

需要源码的同学可以直接从github上下载:[NDKLearnDemo](https://github.com/pvphero/NDKLearnDemo)








