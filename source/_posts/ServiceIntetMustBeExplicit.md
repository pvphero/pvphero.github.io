---
title: Service Intent must be explicit的解决方案
date: 2017-09-14 15:11:43
tags:
- Android
categories:
- Android开发笔记
---

今天在学习AIDL的时候,通过以下步骤:

<!--more-->

- **在AndroidMenifest中声明service**


``` java

	<service
		android:name=".MyService"
        android:process=":remote"
        android:exported="true">
        <intent-filter >
        	<category android:name="android.intent.category.DEFAULT"></category>
            <action android:name="com.ihealth.learnaidl.MyService"></action>
        </intent-filter>
    </service>

```

- **在客户端中绑定service**


``` java

Intent intentService=new Intent();
                intentService.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                intentService.setAction(ACTION_BIND_SERVICE);
                MainActivity.this.bindService(intentService,mServiceConnection,BIND_AUTO_CREATE);


```

- **运行程序,结果报错了!**
java.lang.IllegalArgumentException: Service Intent must be explicit
![20170914154711498](http://paynnyvep.bkt.clouddn.com/20170914154711498.png)

经过查找资料[Stackoverflow.com](https://stackoverflow.com/questions/27842430/service-intent-must-be-explicit-intent),发现是需要将隐含意图转换为显示的意图，然后启动再服务。


所以综合Stackoverflow上给出的解决方案,现在大体上有两种解决的方法.

1.先说一个简单的办法
``` java
Intent mIntent = new Intent();
//自定义的Service的action
mIntent.setAction("XXX.XXX.XXX");
//自定义Service的包名
mIntent.setPackage(getPackageName());
context.startService(mIntent);
```

即只需要多加一句话**mIntent.setPackage(getPackageName());**就可以解决.

2.另外一个比较麻烦的方法
先通过一个函数将隐式调用转变为显示调用

``` java
/***
     * Android L (lollipop, API 21) introduced a new problem when trying to invoke implicit intent,
     * "java.lang.IllegalArgumentException: Service Intent must be explicit"
     *
     * If you are using an implicit intent, and know only 1 target would answer this intent,
     * This method will help you turn the implicit intent into the explicit form.
     *
     * Inspired from SO answer: http://stackoverflow.com/a/26318757/1446466
     * @param context
     * @param implicitIntent - The original implicit intent
     * @return Explicit Intent created from the implicit original intent
     */
    public static Intent createExplicitFromImplicitIntent(Context context, Intent implicitIntent) {
        // Retrieve all services that can match the given intent
        PackageManager pm = context.getPackageManager();
        List<ResolveInfo> resolveInfo = pm.queryIntentServices(implicitIntent, 0);

        // Make sure only one match was found
        if (resolveInfo == null || resolveInfo.size() != 1) {
            return null;
        }

        // Get component info and create ComponentName
        ResolveInfo serviceInfo = resolveInfo.get(0);
        String packageName = serviceInfo.serviceInfo.packageName;
        String className = serviceInfo.serviceInfo.name;
        ComponentName component = new ComponentName(packageName, className);

        // Create a new intent. Use the old one for extras and such reuse
        Intent explicitIntent = new Intent(implicitIntent);

        // Set the component to be explicit
        explicitIntent.setComponent(component);

        return explicitIntent;
    }

```

然后调用

```
Intent intent = new Intent();
                intent.setAction(ACTION_BIND_SERVICE);
                final Intent eintent = new Intent(createExplicitFromImplicitIntent(this,intent));
                bindService(eintent,mServiceConnection, Service.BIND_AUTO_CREATE);
```

两种办法都可以解决.记下来,免得以后忘了.
感谢Stackoverflow~[Google In-App billing, IllegalArgumentException: Service Intent must be explicit, after upgrading to Android L Dev Preview](https://stackoverflow.com/questions/24480069/google-in-app-billing-illegalargumentexception-service-intent-must-be-explicit/26318757#26318757)

[Service Intent must be explicit: Intent](https://stackoverflow.com/questions/27842430/service-intent-must-be-explicit-intent)

