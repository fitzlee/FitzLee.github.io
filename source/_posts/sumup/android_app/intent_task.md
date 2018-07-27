---
title: Android intent和启动模式
urlname: android_app_intent_start_mode
#date: 2017-04-10
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 10Android_App
tags:
    - Android
    - binder
    - intent
    - 启动模式
---

## taskAffinity、launchmode、Intent.FLAG_ACTIVITY_XXX 有关的堆栈和back键返回等
http://www.voidcn.com/blog/fengniuma/article/p-3214732.html
https://blog.piasy.com/2017/01/16/Android-Basics-Task-and-LaunchMode
http://blog.csdn.net/scnuxisan225/article/details/44930321
http://blog.csdn.net/tuhuolong/article/details/8844936

**taskAffinity解释：**
taskAffinity 标示以该Activity为root的所在task的名字。属性值推荐用.开头。 
可以通过<application>给所有activity设置taskAffinity。
<application  taskAffinity=".newtask" >
   <activity *** />
    ***
</application>
也可以单独设置activity 的taskAffinity
<activity  taskAffinity=".newtask"/>
如果<application>或<acitivity>都没有设置taskAffinity，则系统默认赋值为应用程序包名。也就是说taskAffinity要么是自己定义的值，要么去应用程序包名。 
值得注意的是，如果launcher activity使用默认taskAffinity，通过launcher activity启动的一个activity B，【B配置了taskAffinity属性，并且没有设置启动新的task, 那么其实B的taskAffinity就失效了】，因为task的名字是由root activity的taskAffinity决定的。
taskAffinity结合singleTask和SingleIntance和FLAG_ACTIVITY_NEW_TASK 一起使用会产生效果，单独的task只存在一个activity，该task上的activity清除等。

**Launchmode**
Standard  singleTop  singleTask singleInstance主要有这几种模式。
其中standard模式比较简单，直接在当前栈启动一个新的activity，而不管之前是否存在相同activity实例。
其中singleTop模式也比较简单，直接在当前栈查找是否存在该activity实例，如果存在直接调用onNewIntent方法，如果不存在那么直接创建改Activity。 不关注TaskAffinity。
其中singleTask模式较为麻烦，先寻找名字为taskAffnity指定的task，如果该task存在这个activity，那么清楚之上的所有activity，然后调用OnNewIntent方法。如果不存在该task，那么创建task，创建activity。
其中singleIntance模式，要求Activity实例独占task，所以先寻找是否存在该Activity，存在那么直接调用onNewIntent，如果不存在，那么创建该task和activity。 如果从该activity调用其他Activity那么会切换到其他task栈上，因为该activity独占task。
Intent.FLAG_ACTIVITY_XXX
FLAG_ACTIVITY_NEW_TASK 配合FLAG_ACTIVITY_CLEAR_TOP的时候，才会和“singleTask”行为一致–在已经存在目标activity的情况下，清除上层全部Activity.
FLAG_ACTIVITY_SINGLE_TOP  = singleTop
FLAG_ACTIVITY_CLEAR_TOP + FLAG_ACTIVITY_NEW_TASK = singleTask


## ActivityRecord、TaskRecord、ActivityStack之间的关系
ActivityStackSupervisor.mActivityDisplays-> ActivityDisplay.mStacks -> ActivityStack.mTaskHistory -> TaskRecord.mActivities -> ActivityRecord
简单来说就是一下类的关系图，ActivityStackSupervisor与ActivityDisplay都是系统唯一，ActivityDisplay负责管理很多ActivityStack，ActivityDisplay主要有Home Stack和App Stack这两个栈；一个ActivityStack负责管理很多TaskRecord，一个TaskRecord又包含很多ActivityRecord。一个activityRecord包含
ProcessRecord app //跑在哪个进程  TaskRecord task //跑在哪个task ActivityInfo info // Activity信息 int mActivityType //Activity类型  ActivityState state //Activity状态  ApplicationInfo appInfo //跑在哪个app
http://gityuan.com/2017/06/11/activity_record/
http://blog.csdn.net/itachi85/article/details/77542286
http://blog.csdn.net/kebelzc24/article/details/53747506
http://duanqz.github.io/2016-02-01-Activity-Maintenance#activitystack

## Intent-filter action category Data scheme匹配规则
http://blog.csdn.net/iispring/article/details/48481793

## intent匹配
https://developer.android.com/guide/components/intents-filters.html?hl=zh-cn#Resolution

## PendingIntent
如果Intent对象的Action是相同的，Data也是相同的，Categories也是相同的，Components也是相同的，Flags也是相同的），并且之前获取的PendingIntent对象还有效的话，那么该进程获取到的PendingItent对象将获得同一个对象的引用，可以通过cancel()方法来从系统中移除它。
PendingIntent对象由系统持有，因此不能通过设置不同的Extra来生成不同的PendingIntent对象，系统只通过刚才在上面提到的几个要素来判断PendingIntent对象是否是相同的。
PendingIntent有以下flag：
FLAG_CANCEL_CURRENT:如果当前系统中已经存在一个相同的PendingIntent对象，那么就将先将已有的PendingIntent取消，然后重新生成一个PendingIntent对象。
FLAG_NO_CREATE:如果当前系统中不存在相同的PendingIntent对象，系统将不会创建该PendingIntent对象而是直接返回null。
FLAG_ONE_SHOT:该PendingIntent只作用一次。
FLAG_UPDATE_CURRENT:如果系统中已存在该PendingIntent对象，那么系统将保留该PendingIntent对象，但是会使用新的Intent来更新之前PendingIntent中的Intent对象数据，例如更新Intent中的Extras。
创建PendingIntent方式：
PendingIntent.getActivity (context, requestCode, broadIntent, flags)
PendingIntent.getBroadcast(context,requestCode, broadIntent, flags)
PendingIntent.getService (context, requestCode, broadIntent, flags)
API文档里虽然未使用requestCode参数，但实际上是通过该参数来区别不同的Intent的
http://blog.csdn.net/yangwen123/article/details/8019739
https://my.oschina.net/youranhongcha/blog/196933

##  Launch Mode
先来说说在ActivityInfo.java中定义了4类Launch Mode：

LAUNCH_MULTIPLE(standard)：最常见的情形，每次启动Activity都是创建新的Activity;
LAUNCH_SINGLE_TOP: 当Task顶部存在同一个Activity则不再重新创建；其余情况同上；
LAUNCH_SINGLE_TASK：当Task栈存在同一个Activity(不在task顶部)，则不重新创建，而移除该Activity上面其他的Activity；其余情况同上；
LAUNCH_SINGLE_INSTANCE：每个Task只有一个Activity.
再来说说几个常见的flag含义：

FLAG_ACTIVITY_NEW_TASK：将Activity放入一个新启动的Task；
FLAG_ACTIVITY_CLEAR_TASK：启动Activity时，将目标Activity关联的Task清除，再启动新Task，将该Activity放入该Task。该flags跟FLAG_ACTIVITY_NEW_TASK配合使用。
FLAG_ACTIVITY_CLEAR_TOP：启动非栈顶Activity时，先清除该Activity之上的Activity。例如Task已有A、B、C3个Activity，启动A，则清除B，C。类似于SingleTop。
最后再说说：设置FLAG_ACTIVITY_NEW_TASK的几个情况：

调用者并不是Activity context；
调用者activity带有single instance；
目标activity带有single instance或者single task；
调用者处于finishing状态；