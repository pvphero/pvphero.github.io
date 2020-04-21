---
title: Anroid Intent跳转系统设置页面
date: 2018-01-18 17:20:01
tags:
- Android
categories:
- Android开发笔记
---

我们的app支持18种语言,为了测试能很快的跳转到语言的切换页面.所以专门扒拉了一下intent跳转系统所有设置页面的方法,记录下来,以后忘记了可以直接查看.
<!--more-->
就是这么漂,就是这么酷炫~
![](https://ws3.sinaimg.cn/large/006tNc79ly1fnks2wr9xzg308w06ox6p.gif)
# android.provider.Settings
``` java

1.   ACTION_ACCESSIBILITY_SETTINGS ：    // 跳转系统的辅助功能界面

           Intent intent =  new Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS);
           startActivity(intent);

2.    ACTION_ADD_ACCOUNT ：               // 显示添加帐户创建一个新的帐户屏幕。【测试跳转到微信登录界面】

           Intent intent =  new Intent(Settings.ACTION_ADD_ACCOUNT);
           startActivity(intent);

3.   ACTION_AIRPLANE_MODE_SETTINGS：       // 飞行模式，无线网和网络设置界面

           Intent intent =  new Intent(Settings.ACTION_AIRPLANE_MODE_SETTINGS);
           startActivity(intent);

        或者：

     ACTION_WIRELESS_SETTINGS  ：

                Intent intent =  new Intent(Settings.ACTION_WIFI_SETTINGS);
                startActivity(intent);

4.    ACTION_APN_SETTINGS：                 //  跳转 APN设置界面

           Intent intent =  new Intent(Settings.ACTION_APN_SETTINGS);
           startActivity(intent);

5.   【需要参数】 ACTION_APPLICATION_DETAILS_SETTINGS：   // 根据包名跳转到系统自带的应用程序信息界面

               Uri packageURI = Uri.parse("package:" + "com.tencent.WBlog");
               Intent intent =  new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS,packageURI);
               startActivity(intent);

6.    ACTION_APPLICATION_DEVELOPMENT_SETTINGS :  // 跳转开发人员选项界面

           Intent intent =  new Intent(Settings.ACTION_APPLICATION_DEVELOPMENT_SETTINGS);
           startActivity(intent);

7.    ACTION_APPLICATION_SETTINGS ：      // 跳转应用程序列表界面

           Intent intent =  new Intent(Settings.ACTION_APPLICATION_SETTINGS);
           startActivity(intent);

       或者：

      ACTION_MANAGE_ALL_APPLICATIONS_SETTINGS   // 跳转到应用程序界面【所有的】

             Intent intent =  new Intent(Settings.ACTION_MANAGE_ALL_APPLICATIONS_SETTINGS);
             startActivity(intent);

       或者：

       ACTION_MANAGE_APPLICATIONS_SETTINGS  ：//  跳转 应用程序列表界面【已安装的】

             Intent intent =  new Intent(Settings.ACTION_MANAGE_APPLICATIONS_SETTINGS);
             startActivity(intent);



8.    ACTION_BLUETOOTH_SETTINGS  ：      // 跳转系统的蓝牙设置界面

           Intent intent =  new Intent(Settings.ACTION_BLUETOOTH_SETTINGS);
           startActivity(intent);

9.    ACTION_DATA_ROAMING_SETTINGS ：   //  跳转到移动网络设置界面

           Intent intent =  new Intent(Settings.ACTION_DATA_ROAMING_SETTINGS);
           startActivity(intent);

10.    ACTION_DATE_SETTINGS ：           //  跳转日期时间设置界面

            Intent intent =  new Intent(Settings.ACTION_DATA_ROAMING_SETTINGS);
            startActivity(intent);

11.    ACTION_DEVICE_INFO_SETTINGS  ：    // 跳转手机状态界面

            Intent intent =  new Intent(Settings.ACTION_DEVICE_INFO_SETTINGS);
            startActivity(intent);

12.    ACTION_DISPLAY_SETTINGS  ：        // 跳转手机显示界面

            Intent intent =  new Intent(Settings.ACTION_DISPLAY_SETTINGS);
            startActivity(intent);

13.    ACTION_DREAM_SETTINGS     【API 18及以上 没测试】

            Intent intent =  new Intent(Settings.ACTION_DREAM_SETTINGS);
            startActivity(intent);


14.    ACTION_INPUT_METHOD_SETTINGS ：    // 跳转语言和输入设备

            Intent intent =  new Intent(Settings.ACTION_INPUT_METHOD_SETTINGS);
            startActivity(intent);

15.    ACTION_INPUT_METHOD_SUBTYPE_SETTINGS  【API 11及以上】  //  跳转 语言选择界面 【多国语言选择】

             Intent intent =  new Intent(Settings.ACTION_INPUT_METHOD_SUBTYPE_SETTINGS);
             startActivity(intent);

16.    ACTION_INTERNAL_STORAGE_SETTINGS         // 跳转存储设置界面【内部存储】

             Intent intent =  new Intent(Settings.ACTION_INTERNAL_STORAGE_SETTINGS);
             startActivity(intent);

      或者：

        ACTION_MEMORY_CARD_SETTINGS    ：   // 跳转 存储设置 【记忆卡存储】

             Intent intent =  new Intent(Settings.ACTION_MEMORY_CARD_SETTINGS);
             startActivity(intent);


17.    ACTION_LOCALE_SETTINGS  ：         // 跳转语言选择界面【仅有English 和 中文两种选择】

              Intent intent =  new Intent(Settings.ACTION_LOCALE_SETTINGS);
              startActivity(intent);


18.     ACTION_LOCATION_SOURCE_SETTINGS :    //  跳转位置服务界面【管理已安装的应用程序。】

              Intent intent =  new Intent(Settings.ACTION_LOCATION_SOURCE_SETTINGS);
              startActivity(intent);

19.    ACTION_NETWORK_OPERATOR_SETTINGS ： // 跳转到 显示设置选择网络运营商。

              Intent intent =  new Intent(Settings.ACTION_NETWORK_OPERATOR_SETTINGS);
              startActivity(intent);

20.    ACTION_NFCSHARING_SETTINGS  ：       // 显示NFC共享设置。 【API 14及以上】

              Intent intent =  new Intent(Settings.ACTION_NFCSHARING_SETTINGS);
              startActivity(intent);

21.    ACTION_NFC_SETTINGS  ：           // 显示NFC设置。这显示了用户界面,允许NFC打开或关闭。  【API 16及以上】

              Intent intent =  new Intent(Settings.ACTION_NFC_SETTINGS);
              startActivity(intent);

22.    ACTION_PRIVACY_SETTINGS ：       //  跳转到备份和重置界面

              Intent intent =  new Intent(Settings.ACTION_PRIVACY_SETTINGS);
              startActivity(intent);

23.    ACTION_QUICK_LAUNCH_SETTINGS  ： // 跳转快速启动设置界面

               Intent intent =  new Intent(Settings.ACTION_QUICK_LAUNCH_SETTINGS);
               startActivity(intent);

24.    ACTION_SEARCH_SETTINGS    ：    // 跳转到 搜索设置界面

               Intent intent =  new Intent(Settings.ACTION_SEARCH_SETTINGS);
               startActivity(intent);

25.    ACTION_SECURITY_SETTINGS  ：     // 跳转到安全设置界面

               Intent intent =  new Intent(Settings.ACTION_SECURITY_SETTINGS);
               startActivity(intent);

26.    ACTION_SETTINGS   ：                // 跳转到设置界面

                Intent intent =  new Intent(Settings.ACTION_SETTINGS);
                startActivity(intent);

27.   ACTION_SOUND_SETTINGS                // 跳转到声音设置界面

                 Intent intent =  new Intent(Settings.ACTION_SOUND_SETTINGS);
                 startActivity(intent);

28.   ACTION_SYNC_SETTINGS ：             // 跳转账户同步界面

                Intent intent =  new Intent(Settings.ACTION_SYNC_SETTINGS);
                startActivity(intent);

29.     ACTION_USER_DICTIONARY_SETTINGS ：  //  跳转用户字典界面

                Intent intent =  new Intent(Settings.ACTION_USER_DICTIONARY_SETTINGS);
                startActivity(intent);

30.     ACTION_WIFI_IP_SETTINGS  ：         // 跳转到IP设定界面

                 Intent intent =  new Intent(Settings.ACTION_WIFI_IP_SETTINGS);
                 startActivity(intent);

31.     ACTION_WIFI_SETTINGS  ：            //  跳转Wifi列表设置

                 Intent intent = new Intent(Settings.ACTION_WIFI_SETTINGS);
                 startActivity(intent);
```

# 跳转方式
``` java
    Intent intent = new Intent(Settings.*********);
    startActivity(intent);
```

# 举个例子
方法都贴完了怎么能不举个栗子,撸码爽爽

**MainActivity.java**
```java
import android.content.Intent;
import android.provider.Settings;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        findViewById(R.id.button).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(Settings.ACTION_LOCALE_SETTINGS);
                startActivity(intent);
            }
        });
    }
}
```

**activity_main**
```java
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">


    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="语言选择"/>
</android.support.constraint.ConstraintLayout>

```

# 实现效果
![](https://ws1.sinaimg.cn/large/006tNc79ly1fnkw9y61gdj30u01hc0ty.jpg)

