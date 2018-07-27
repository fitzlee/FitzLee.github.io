---
title: Android 一些简单的安全总结
urlname: android_app_security_sum
#date: 2017-04-10
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 10Android_App
tags:
    - Android
    - secutiry
    - broadcast
	- provider
---

## provider权限
http://blog.csdn.net/flowingflying/article/details/17412609> 
**全局权限**
<provider
                android:name="com.xxx.ExampleProvider "
                android:authorities="com.xxx.provider "
                android:exported="true"
                android:readPermission="com.xxx.permission.READ"
                android:writePermission="com.xxx.permission.WRITE" />

<uses-permission android:name="com.xxx.permission.READ" />

**uri权限**
<provider android:name=".PrivProvider" 
    android:authorities="xxx.xxx.flowingflying.propermission.PrivProvider" 
    android:readPermission="xxx.permission.READ_CONTENTPROVIDER" 
    android:exported="true" > 
    <path-permission android:pathPrefix="/hello" android:readPermission="READ_HELLO_CONTENTPROVIDER" />
</provider> 


全局Uri granting权限传递
<provider android:name=".PrivProvider" 
     android:authorities="xxx.xxx.flowingflying.propermission.PrivProvider" 
     android:readPermission="xxx.permission.READ_CONTENTPROVIDER" 
     android:grantUriPermissions="true" 
     android:exported="true" /> 

Intent intent = new Intent(this,ReadProvider.class); 
intent.setClassName("com.example.propermissiongrant", "com.example.propermissiongrant.MainActivity");
intent.setData(Uri.parse("content://xxx.xxx.flowingflying.propermission.PrivProvider/world/1"));
intent.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);  //传递权限 
startActivity(intent); 

**部分uri权限传递**
<provider android:name=".PrivProvider" 
     android:authorities="xxx.xxx.flowingflying.propermission.PrivProvider" 
     android:readPermission="xxx.permission.READ_CONTENTPROVIDER" 
     android:exported="true" >  
     <grant-uri-permission android:pathPrefix="/hello" />
</provider> 


<!-- more -->


## android:exported说明

Activity 是否可由其他应用的组件启动 —“true”表示可以，“false”表示不可以。若为“false”，则 Activity 只能由同一应用的组件或使用同一用户 ID 的不同应用启动。

默认值取决于 Activity 是否包含 Intent 过滤器。没有任何过滤器意味着 Activity 只能通过指定其确切的类名称进行调用。 这意味着 Activity 专供应用内部使用（因为其他应用不知晓其类名称）。 因此，在这种情况下，默认值为“false”。另一方面，至少存在一个过滤器意味着 Activity 专供外部使用，因此默认值为“true”。

该属性并非限制 Activity 对其他应用开放度的唯一手段。 您还可以利用权限来限制哪些外部实体可以调用 Activity（请参阅 permission 属性）。

https://developer.android.com/guide/topics/manifest/activity-element.html?hl=zh-cn#exported


## Broadcast权限
https://www.cnblogs.com/whoislcj/p/5497409.html
https://blog.csdn.net/javensun/article/details/7334230

**在Android应用开发中，有时会遇到以下两种情况**
1. 一些敏感的广播并不想让第三方的应用收到 ；
2. 要限制自己的Receiver接收某广播来源，避免被恶意的同样的ACTION的广播所干扰。

在这些场景下就需要用到广播的权限限制。

**第一种场景： 谁有权收我的广播**
在这种情况下，可以在自己应用发广播时添加参数声明Receiver所需的权限。
首先，在Androidmanifest.xml中定义新的权限RECV_XXX，例如：
	1. <permission android:name = "com.android.permission.RECV_XXX"/>  
然后，在Sender app发送广播时将此权限作为参数传入，如下：
	1. sendBroadcast("com.android.XXX_ACTION", "com.android.permission.RECV_XXX");  
这样做之后就使得只有具有RECV_XXX权限的Receiver才能接收此广播要接收该广播，在Receiver应用的AndroidManifest.xml中要添加对应的RECV_XXX权限。
例如：
	1. <uses-permission android:name="com.android.permission.RECV_XXX"></uses-permission>  

**第二种场景： 谁有权给我发广播？**
在这种情况下，需要在Receiver app的<receiver> tag中声明一下Sender app应该具有的权限。
首先同上，在AndroidManifest.xml中定义新的权限SEND_XXX，例如：
	1. <permission android:name="com.android.SEND_XXX"/>  
然后，在Receiver app的Androidmanifest.xml中的<receiver>tag里添加权限SEND_XXX的声明，如下：
	1. <receiver android:name=".XXXReceiver"   
	2.           android:permission="com.android.permission.SEND_XXX">   
	3.     <intent-filter>  
	4.          <action android:name="com.android.XXX_ACTION" />   
	5.     </intent-filter>  
	6. </receiver>  
这样一来，该Receiver便只能接收来自具有该SEND_XXX权限的应用发出的广播。
要发送这种广播，需要在Sender app的AndroidManifest.xml中也声明使用该权限即可，如下：
	1. <uses-permission android:name="com.android.permission.SEND_XXX"></uses-permission>  
如此，可以用来对广播的来源与去处进行简单的控制。
同样，对Activity 和 ContentProvider的访问权限控制也类似。


**Protected-broadcast**
只有系统应用才能发送。
• Process.SYSTEM_UID
• Process.PHONE_UID
• Process.SHELL_UID
• Process.BLUETOOTH_UID
• or 0
• 其他应用不能发送

参考：
https://crashpost.wordpress.com/2014/01/11/android-protected-broadcasts/
http://blog.csdn.net/u013553529/article/details/78409382


## Android存储
https://bboyfeiyu.gitbooks.io/android-tech-frontier/content/others/%E6%B8%85%E6%99%B0%E7%9A%84%E8%BD%AF%E4%BB%B6%E6%9E%B6%E6%9E%84/readme.html
https://github.com/Blankj/AndroidStandardDevelop
https://github.com/googlesamples/android-architecture
http://kernel.meizu.com/android-m-external-storage.html
http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2013/0923/1557.html
http://blog.csdn.net/liuxu0703/article/details/53897702
http://blog.csdn.net/qq_23940659/article/details/50639787

相信很多新手对于Android的一些系统默认路径不太了解，在这里以5.1的Nexus5为例来介绍一下，希望对新手有点帮助，当然我也是新手啦。
Environment.getDataDirectory().getPath()=/data
Environment.getDownloadCacheDirectory().getPath()=/cache
Environment.getExternalStorageDirectory()=/storage/emulated/0    /sdcard/  /storage/self/primary   /mnt/user/0/primary
Environment.getRootDirectory().getPath()=/system
//警报的铃声
Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_ALARMS).getPath()=/storage/emulated/0/Alarms
//相机拍摄的图片和视频保存的位置
Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DCIM).getPath()=/storage/emulated/0/DCIM
//下载文件保存的位置
Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS).getPath()=/storage/emulated/0/Download
//电影保存的位置
Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_MOVIES).getPath()=/storage/emulated/0/Movies
//音乐保存的位置
Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_MUSIC).getPath()=/storage/emulated/0/Music
//通知音保存的位置
Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_NOTIFICATIONS).getPath()=/storage/emulated/0/Notifications
//下载的图片保存的位置
Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES).getPath()=/storage/emulated/0/Pictures
//用于保存podcast（博客）的音频文件
Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PODCASTS).getPath()=/storage/emulated/0/Podcasts
//保存铃声的位置
Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_RINGTONES).getPath()=/storage/emulated/0/Ringtones

