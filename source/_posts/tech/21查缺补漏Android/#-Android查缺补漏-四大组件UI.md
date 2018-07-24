---
title: Android查缺补漏-四大组件UI
urlname: android_review_four_component_ui
date: 2016-11-01
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 21Android_review
tags:
    - Android
    - 查缺补漏
    - 四大组件
    - UI
---

## Service onStartCommand返回参数说明
方法onStartCommand()的返回值为int类型，主要的作用是当Service进程被意外kill掉时，Service服务下一步要做哪些行为，主要有3种值。
START_STICKY：Service被异外终止时不调用onDestroy()回调，并且终止后自动重启Service服务，只执行Service对象的onCreate()生命周期方法。
START_NOT_STICKY：Service被异外终止时不调用onDestroy()回调，并且不自动重启服务。
START_REDELIVER_INTENT：Service被异外终止时不调用onDestroy()回调，并且终止后自动重启Service服务，还要执行Service对象的onCreate()和onStartCommand()生命周期方法，并且从Intent中能取到值。
http://book.51cto.com/art/201211/363296.htm


## Service stopSelf
一直知道stopself是停掉Service的方法，但是却不知道什么时候停止。以为调用了stopself就会马上停止，实际上我错了.在onStartCommond方法里面调用stopself方法时，不会马上停止，而是onStartCommond方法执行结束才会停止。
还有一点，调用stopself方法之后，service会执行onDestory方法。
另外，如果onStartCommond中启动一个线程，调用stopself，线程也不会被杀死。
来自 <http://blog.csdn.net/hello0370/article/details/46781523> 

当调用finish方法时，onCreate方法会继续执行，之后调用onDestory方法。
最后，总结一下，Service的stopself方法的功能是，当完成所有功能之后，将service停掉，而不是等着系统回收。同样finish方法，是当系统执行完onCreate方法之后，调用onDestory方法销毁Activity。
来自 <http://blog.csdn.net/hello0370/article/details/46781523> 


## ACTION_CANCEL 请简述Android事件传递机制， ACTION_CANCEL事件何时触发？ 
关于第一个问题，不做任何解释。 
关于ACTION_CANCEL何时被触发，系统文档有这么一种使用场景：在设计设置页面的滑动开关时，如果不监听ACTION_CANCEL，在滑动到中间时，如果你手指上下移动，就是移动到开关控件之外，则此时会触发ACTION_CANCEL，而不是ACTION_UP，造成开关的按钮停顿在中间位置。 
意思是当滑动的时候就会触发，不知道大家搞没搞过微信的长按录音，有一种状态是“松开手指，取消发送”，这时候就会触发ACTION_CANCEL。

## 简述Android的View绘制流程，Android的wrap_content是如何计算的
https://blog.csdn.net/qinjuning/article/details/8051811
https://blog.csdn.net/qinjuning/article/details/8074262
https://www.cnblogs.com/duanweishi/p/4301742.html

## bundle的数据结构，如何存储，既然有了Intent.putExtra，为啥还要用bundle。
bundle的内部结构其实是Map，传递的数据可以是boolean、byte、int、long、float、double、string等基本类型或它们对应的数组，也可以是对象或对象数组。当Bundle传递的是对象或对象数组时，必须实现Serializable 或Parcelable接口。

## LocalBroadcastmanager使用
首先获取LocalBroadcastManager实例：
LocalBroadcastManager lbm = LocalBroadcastManager.getInstance(this);
然后通过函数 registerReceiver方法来注册监听器：
lbm.registerReceiver(new BroadcastReceiver(){
 @Override
 public void onReceive(Context arg0, Intent arg1) {
  // TODO Auto-generated method stub
 }  
}, new IntentFilter("com.android.action.PRIVATE_BROADCAST"));
最后调用sendBroadcast方法发送广播：
Intent intent=new Intent("com.android.action.PRIVATE_BROADCAST");
intent.putExtra(KEY, SENSITIVE_DATA);
lbm.sendBroadcast(intent);

## Sticky broadcast
首先点击next Activity从代码中可以看到receiver已经注册，但Log无输出，这是当然的了~~~因为没有广播发出自然就不会有人响应了。
按back后退到上图
下面分别点击send broadcast 和 send stickybroadcast按钮，随便点击几次，此时对应的receiver并没有注册，所以是不会有人响应这两条广播的。然后点击next activity，当打开新的activity后对应的receiver被注册，此时从日志中就能看出已经收到了send stickybroadcast发出的广播，但没有send broadcast发出的广播。这就是sendStickyBroadcast的特别之处，它将发出的广播保存起来，一旦发现有人注册这条广播，则立即能接收到。
日志打印为： action = com.android.action.sticky.broadcastand count = 4
从上面的日志信息可以看出sendStickyBroadcast只保留最后一条广播，并且一直保留下去，这样即使已经处理了这条广播但当再一次注册这条广播后依然可以收到它。
如果你只想处理一遍，removeStickyBroadcast方法可以帮你，处理完了后就将它删除吧。
http://blog.csdn.net/yihua0607/article/details/6890805

 要知道区别首先需要看一下Android Developers Reference， 它可是我们最好的老师了，sendBroadcast 大家应该都会用了我就不赘述了，下面来看看sendStickyBroadcast
google官方的解释是：
Perform a sendBroadcast(Intent) that is "sticky," meaning the Intent you are sending stays around after the broadcast is complete, so that others can quickly retrieve that data through the return value ofregisterReceiver(BroadcastReceiver, IntentFilter). In all other ways, this behaves the same as sendBroadcast(Intent).
You must hold the BROADCAST_STICKY permission in order to use this API. If you do not hold that permission,SecurityException will be thrown.
大概的意思是说： 发出的广播会一直滞留（等待），以便有人注册这则广播消息后能尽快的收到这条广播。其他功能与sendBroadcast相同。但是使用sendStickyBroadcast 发送广播需要获得BROADCAST_STICKY permission，如果没有这个permission则会抛出异常。

## Activity生命周期
http://www.cnblogs.com/zyw-205520/p/3313268.html
https://developer.android.com/guide/components/activities/activity-lifecycle.html#tba

## 按back键处理流程
我们在使用安卓手机时候，常常会用到back（返回）键来退出当前正在使用的Activity。这个事件的主要流程可以概括为两个步骤。
1. InputReader，InputDispatcher对输入事件的分发处理，包括调用PhoneWindowManager的interceptKeyBeforeDispatching和View，ViewGroup的dispatchTouchEvent等方法。
2.事件分发到该Activity后，Activity方法调用finish方法退出，销毁Activity。众多第三方软件常常设置连续两次按下back键退出应用程序的功能，或者是在各浏览器APP中返回上一页浏览过的网页的功能，就是通过重写这一步骤中涉及的方法而实现的。
本文分析的是第二步流程，第1步过程将在随后《android输入事件分发机制》一文中进行探讨。
由于大部分Activity不会重写onBackPressed()方法，因此它们在接受到back键输入事件后，最终会辗转调用到父类Activity.Java的onBackPressed()方法。
http://blog.csdn.net/jakioneplus/article/details/50034653


## Android事件分发机制
Activity ViewGroup View 
View是所有ViewGroup TextView LinearLayout等的基类
LinearLayout/RelativeLayout还有一些可以包含多个TextView、Button等的组装均继承自ViewGroup，因此也是一个view
Android事件分发主要是从Activity到ViewGroup到具体某个View响应的过程，事件都是以MotionEvent.ACTION_DOWN事件开始、UP事件结束，中间有无数的MOVE事件。
主要是dispatchTouchEvent  onTouchEvent onInterceptTouchEvent之间的传递和拦截处理。基本思路是从Activity->ViewGroup->View的dispatchTouchEvent一直往下执行，如果有一个返回false那么直接向上返回onTouchEvent执行，如果返回true那么直接消费掉结束，如果不返回那么继续下一层dispatchTouchEvent； 如果到最后一层dispatchTouchEvent还没有返回，那么执行onTouchEvent依次往上执行，如果onTouchEvent返回true，那么就消费掉结束，否则一直向上执行onTouchEvent，到最上层结束。
https://www.jianshu.com/p/38015afcdb58
查看解析步骤：
https://upload-images.jianshu.io/upload_images/944365-aea821bbb613c195.png?imageMogr2/auto-orient/
