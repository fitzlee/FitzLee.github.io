---
title: Android查缺补漏-常用工具
urlname: android_review_android_tools_adb
date: 2016-09-16
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 21Android_review
tags:
    - Android
    - 常用工具
---

## Proguard
useProguard minifyEnabled
https://developer.android.com/studio/build/shrink-code.html?hl=zh-cn

minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'

除了 minifyEnabled 属性外，还有用于定义 ProGuard 规则的 proguardFiles 属性：

getDefaultProguardFile('proguard-android.txt') 方法可从 Android SDK tools/proguard/ 文件夹获取默认的 ProGuard 设置。
提示：要想做进一步的代码压缩，请尝试使用位于同一位置的 proguard-android-optimize.txt 文件。它包括相同的 ProGuard 规则，但还包括其他在字节码一级（方法内和方法间）执行分析的优化，以进一步减小 APK 大小和帮助提高其运行速度。

proguard-rules.pro 文件用于添加自定义 ProGuard 规则。默认情况下，该文件位于模块根目录（build.gradle 文件旁）。

https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/introduction.html

## 资源压缩

请在 build.gradle 文件中将 shrinkResources 属性设置为 true
自定义要保留的资源
如果您有想要保留或舍弃的特定资源，请在您的项目中创建一个包含 <resources> 标记的 XML 文件，并在 tools:keep 属性中指定每个要保留的资源，在 tools:discard 属性中指定每个要舍弃的资源。这两个属性都接受逗号分隔的资源名称列表。您可以使用星号字符作为通配符。

例如：
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@layout/l_used*_c,@layout/l_used_a,@layout/l_used_b*"
    tools:discard="@layout/unused2" />
将该文件保存在项目资源中，例如，保存在 res/raw/keep.xml。构建不会将该文件打包到 APK 之中。

## Framework.jar 非dex的编译方式
in Android M, you can achieve that by using LOCAL_JACK_ENABLED = disabled variable in your makefile.
In Android N it is more tricky...it is broken and the solution to this is yet to come in AOSP mainline of Android N.
Solution is available in master.
Checked in 7.1.1 Android N.
You need to make use of javac-check target. This has been implemented for this purpose.
make javac-check-$(LOCAL_MODULE)
So, if your module name is "ABCD"
make javac-check-ABCD
This generates a classes-full-debug.jar, in the common\obj\JAVA_LIBRARIES\ABCD_intermediates.
This is the jar which you can use in your Studio environment.
Most important, this solution is not yet visible in any of the tags on Android N branch. It is part of the master hence you need to add this solution to your branch manually.
https://android.googlesource.com/platform/build/+/32bd0adf9c5dcd1560d77bdb886c7acc78496657
来自 <https://groups.google.com/forum/#!topic/android-building/gRuDxqEB1H8> 


## AM常见的用法

我们可以通过命令启动android中的Activity，Service，BroadcastReceiver 等组件
 
1. 拨打一个电话：
 
    am start -a android.intent.action.CALL -d tel:10086
 
    这里-a表示动作，-d表述传入的数据，还有-t表示传入的类型。
 
2. 打开一个网页：
 
    am start -a android.intent.action.VIEW -d  http://www.baidu.com （这里-d表示传入的data）
 
3. 打开音乐播放器：
 
    am start -a android.intent.action.MUSIC_PLAYER 或者
    am start -n com.android.music/om.android.music.MusicBrowserActivity
 
4. 启动一个服务：
 
    am startservice <服务名称>
 
    例如：am startservice -n com.android.music/com.android.music.MediaPlaybackService (这里-n表示组件)
    或者   am startservice -a com.smz.myservice (这里-a表示动作，就是你在Androidmanifest里定义的) 
 
5. 发送一个广播：
 
    am broadcast -a <广播动作>
 
    例如： am broadcast -a com.smz.mybroadcast
 
am还有很多的用法，有待研究。

来自 <http://blog.csdn.net/electricity/article/details/6409354> 



