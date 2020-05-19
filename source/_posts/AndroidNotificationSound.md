---
title: 关于Android 7.0系统通知声音不能播放
date: 2017-07-31 15:18:56
tags:
- Android
categories:
- Android开发笔记
---

由于沉迷于撸(nong)码(yao),很久没有更新过博客了,甚是惭愧.公司的项目比较高大上,主要面对老外开发,所以要适配各种版本的Android机,项目里有个闹钟提醒患者吃药的功能,但是这个功能获取系统通知铃声在Android 6.0以下好好的,换了个7.0的手机却不能播放出声音了.Android的锅,我们不背,我们不背![这里写图片描述](http://img.blog.csdn.net/20170727174234464?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcHZwaGVyb3N6dw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)...但是能解决的还是解决下吧.

<!--more-->
# 问题现象及问题定位
NotificationCompat.Builder.setSound(URI)的时候,发现没有发出任何声音,但是却显示出了一个错误.
![这里写图片描述](http://img.blog.csdn.net/20170727214940263?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcHZwaGVyb3N6dw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
# 问题分析
> 将系统铃声设置为系统通知铃声需要两个操作

>> - 通过RingtoneManager.ACTION_RINGTONE_PICKER,获取"/system/media/audio/notifications"路径下的音乐的URI

>> - 调用RingtoneManager.setActualDefaultRingtoneUri()，传入相应的uri和需要设置的铃声类型即可。

如果你使用的是file: Uri,在targetSdkVersion>=24(Android 7.0以上)的时候是不适用的.因为Android 7.0的Uri会检查播放的声音是否是[file:Uri禁用值](https://commonsware.com/blog/2016/03/14/psa-file-scheme-ban-n-developer-preview.html)

# 问题的解决方法

只需要在调用声音的前面加一句**黑代码** 就可以完美解决

``` java
	grantUriPermission("com.android.systemui", soundUri,
                            Intent.FLAG_GRANT_READ_URI_PERMISSION);
```
 这里的soundUri就是你调取系统声音的Uri

# 贴码

说那么多没用的,不如撸一把代码.为了方便大家直接使用做了一个简单的demo,方便大家参考.

也可以从github上直接下载下来,[RingStom](https://github.com/pvphero/RingStom)

- 布局activity_main.xml文件
``` java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >

    <Button
        android:id="@+id/buttonRingtone"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="设置来电铃声"
        />
    <Button
        android:id="@+id/buttonAlarm"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="设置闹钟铃声"
        />
    <Button
        android:id="@+id/buttonNotification"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="设置通知铃声"
        />

</LinearLayout>

```

- 相关的java代码

``` java
public class MainActivity extends AppCompatActivity {

    /* 3个按钮 */
    private Button mButtonRingtone;
    private Button mButtonAlarm;
    private Button mButtonNotification;

    /* 自定义的类型 */
    public static final int CODE_RINGSTONE = 0;
    public static final int CODE_ALARM = 1;
    public static final int CODE_NOTIFICATION = 2;
    /**
     *  来电铃声文件夹
     *  /system/media/audio/ringtones       系统来电铃声
     *  /sdcard/music/ringtones             用户来电铃声
     */
    private String strRingtoneFolder = "/system/media/audio/ringtones";
//  private String strRingtoneFolder = "/sdcard/music/ringtones";
    /**
     *  闹钟铃声文件夹
     *  /system/media/audio/alarms          系统闹钟铃声
     *  /sdcard/music/alarms                用户闹钟铃声
     */
    private String strAlarmFolder = "/system/media/audio/alarms";
//  private String strAlarmFolder = "/sdcard/music/alarms ";
    /**
     *  闹钟铃声文件夹
     *  /system/media/audio/notifications       系统通知铃声
     *  /sdcard/music/notifications             用户通知铃声
     */
    private String strNotificationFolder = "/system/media/audio/notifications";
//  private String strNotificationFolder = "/sdcard/music/notifications";


    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mButtonRingtone = (Button) findViewById(R.id.buttonRingtone);
        mButtonAlarm = (Button) findViewById(R.id.buttonAlarm);
        mButtonNotification = (Button) findViewById(R.id.buttonNotification);
        mButtonRingtone.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                if (hasFolder(strRingtoneFolder)) {
                    // 打开系统铃声设置
                    Intent intent = new Intent(
                            RingtoneManager.ACTION_RINGTONE_PICKER);
                    // 类型为来电RINGTONE
                    intent.putExtra(RingtoneManager.EXTRA_RINGTONE_TYPE,
                            RingtoneManager.TYPE_RINGTONE);
                    // 设置显示的title
                    intent.putExtra(RingtoneManager.EXTRA_RINGTONE_TITLE,
                            "设置来电铃声");
                    // 当设置完成之后返回到当前的Activity
                    startActivityForResult(intent, CODE_RINGSTONE);
                }
            }
        });
        mButtonAlarm.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                if (hasFolder(strAlarmFolder)) {
                    // 打开系统铃声设置
                    Intent intent = new Intent(
                            RingtoneManager.ACTION_RINGTONE_PICKER);
                    // 设置铃声类型和title
                    intent.putExtra(RingtoneManager.EXTRA_RINGTONE_TYPE,
                            RingtoneManager.TYPE_ALARM);
                    intent.putExtra(RingtoneManager.EXTRA_RINGTONE_TITLE,
                            "设置闹钟铃声");
                    // 当设置完成之后返回到当前的Activity
                    startActivityForResult(intent, CODE_ALARM);
                }
            }
        });
        mButtonNotification.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                if (hasFolder(strNotificationFolder)) {
                    // 打开系统铃声设置
                    Intent intent = new Intent(
                            RingtoneManager.ACTION_RINGTONE_PICKER);
                    // 设置铃声类型和title
                    intent.putExtra(RingtoneManager.EXTRA_RINGTONE_TYPE,
                            RingtoneManager.TYPE_NOTIFICATION);
                    intent.putExtra(RingtoneManager.EXTRA_RINGTONE_TITLE,
                            " 设置通知铃声");
                    // 当设置完成之后返回到当前的Activity
                    startActivityForResult(intent, CODE_NOTIFICATION);
                }
            }
        });
    }
    /**
     * 当设置铃声之后的回调函数
     */
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode != RESULT_OK) {
            return;
        }
        // 得到我们选择的铃声
        Uri pickedUri = data
                .getParcelableExtra(RingtoneManager.EXTRA_RINGTONE_PICKED_URI);
        if (pickedUri != null) {
            switch (requestCode) {
                case CODE_RINGSTONE:
                    // 将我们选择的铃声设置成为默认来电铃声
                    RingtoneManager.setActualDefaultRingtoneUri(this,
                            RingtoneManager.TYPE_RINGTONE, pickedUri);
                    break;
                case CODE_ALARM:
                    // 将我们选择的铃声设置成为默认闹钟铃声
                    RingtoneManager.setActualDefaultRingtoneUri(this,
                            RingtoneManager.TYPE_ALARM, pickedUri);
                    break;
                case CODE_NOTIFICATION:
                    // 将我们选择的铃声设置成为默认通知铃声
                    /**
                     * 敲黑板:黑代码解决Android 7.0 调用系统通知无法播放声音的问题
                     */
                    grantUriPermission("com.android.systemui", pickedUri,
                            Intent.FLAG_GRANT_READ_URI_PERMISSION);
                    RingtoneManager.setActualDefaultRingtoneUri(this,
                            RingtoneManager.TYPE_NOTIFICATION, pickedUri);
                    break;
            }
        }
    }

    /**
     * 检测是否存在指定的文件夹,如果不存在则创建
     *
     * @param strFolder
     *            文件夹路径
     */
    private boolean hasFolder(String strFolder) {
        boolean btmp = false;
        File f = new File(strFolder);
        if (!f.exists()) {
            if (f.mkdirs()) {
                btmp = true;
            } else {
                btmp = false;
            }
        } else {
            btmp = true;
        }
        return btmp;
    }
}
```

希望这篇博客可以带你们入坑~2333333