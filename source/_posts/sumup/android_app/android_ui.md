---
title: Android ui组件总结
urlname: android_app_ui_sum
#date: 2017-04-10
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 10Android_App
tags:
    - Android
    - ui
    - 组件
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


## ACTION_CANCEL 请简述Android事件传递机制， ACTION_CANCEL事件何时触发？ 
关于第一个问题，不做任何解释。 
关于ACTION_CANCEL何时被触发，系统文档有这么一种使用场景：在设计设置页面的滑动开关时，如果不监听ACTION_CANCEL，在滑动到中间时，如果你手指上下移动，就是移动到开关控件之外，则此时会触发ACTION_CANCEL，而不是ACTION_UP，造成开关的按钮停顿在中间位置。 
意思是当滑动的时候就会触发，不知道大家搞没搞过微信的长按录音，有一种状态是“松开手指，取消发送”，这时候就会触发ACTION_CANCEL。

## 简述Android的View绘制流程，Android的wrap_content是如何计算的
https://blog.csdn.net/qinjuning/article/details/7110211
  绘制流程的三个步骤，即：
1、  measure过程 --- 测量过程
2、	layout 过程     --- 布局过程
3、	draw 过程      --- 绘制过程
https://blog.csdn.net/qinjuning/article/details/8051811
https://blog.csdn.net/qinjuning/article/details/8074262
https://www.cnblogs.com/duanweishi/p/4301742.html

## bundle的数据结构，如何存储，既然有了Intent.putExtra，为啥还要用bundle。
bundle的内部结构其实是Map，传递的数据可以是boolean、byte、int、long、float、double、string等基本类型或它们对应的数组，也可以是对象或对象数组。当Bundle传递的是对象或对象数组时，必须实现Serializable 或Parcelable接口。

## Binder 线程池
Binder线程池：每个Server进程在启动时会创建一个binder线程池，并向其中注册一个Binder线程；之后Server进程也可以向binder线程池注册新的线程，或者Binder驱动在探测到没有空闲binder线程时会主动向Server进程注册新的的binder线程。对于一个Server进程有一个最大Binder线程数限制，默认为16个binder线程，例如Android的system_server进程就存在16个线程。对于所有Client端进程的binder请求都是交由Server端进程的binder线程来处理的。


<!-- more -->


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

![img](https://images0.cnblogs.com/blog/441423/201309/10202340-ec2dbe809c0b4987826c96c9fcd1764c.png) 



- HOME键的执行顺序：onPause->onStop->onRestart->onStart->onResume

- BACK键的顺序： onPause->onStop->onDestroy->onCreate->onStart->onResume

- onPause不要做太耗时的工作

- 情形一、一个单独的Activity的正常的生命过程是这样的：onCreate->onStart->onPause->onStop->onDestroy。例如：运行一个Activity，进行了一些简单操作（不涉及页面的跳转等），然后按返回键结束

- 情形二、有两个Activity（a和b），一开始显示a，然后由a启动b，然后在由b回到a，这时候a的生命过程应该是怎么样的呢（a被b完全遮盖）？a经历的过程为onCreate->onStart->onResume->onPause->onStop->onRestart->onStart->onResume。这个过程说明了图中，如果Activity完全被其他界面遮挡时，进入后台，并没有完全销毁,而是停留在onStop状态，当再次进入a时，onRestart->onStart->onResume，又重新恢复

- 情形三、基本情形同二一样，不过此时a被b部分遮盖（比如给b添加个对话框主题 android:theme="@android:style/Theme.Dialog"）， a经历的过程是：onCreate->onStart->onResume->onPause->onResume， 所以当Activity被部分遮挡时，Activity进入onPause，并没有进入onStop，从Activity2返回后，执行了onResume

- 情形四、 打开程序，启动a，点击a，启动AlertDialog，按返回键从AlertDialog返回。a经历的过程是：onCreate->onStart->onResume，当启动和退出Dialog时，Activity的状态始终未变，可见，Dialog实际上属于Acitivity内部的界面，不会影响Acitivty的生命周期。

- OnNewIntent执行时机：

  OnNewIntent被调用的前提是:ActivityA已经启动过,处于当前应用的Activity堆栈中;  当ActivityA的LaunchMode为**SingleTop**时，如果ActivityA在栈顶,且现在要再启动ActivityA，这时会调用onNewIntent()方法 。   当ActivityA的LaunchMode为**SingleInstance,SingleTask**时,如果已经ActivityA已经在堆栈中，那么此时会调用onNewIntent()方法  当ActivityA的LaunchMode为**Standard**时，由于每次启动ActivityA都是启动新的实例，和原来启动的没关系，所以**不会**调用原来ActivityA的onNewIntent方法 

  

  ## Activity模式

  **启动模式允许开发者定义一个activity的新实例如何与当前的Task关联。可以定义使用俩种方法来定义。**

  如果Activity A开启Activity B, Activity B就可以在它的manifest文件中定义它与当前的task如何关联，Activity A也可以要求activity B应该如何与当前的task关联。**如果两个activity都定义了Activity B应该如何与一个task关联，Activity A的要求（在intent中定义的）将会覆盖Activity B中要求（在manifest文件中定义的）。**

  注意：一些在manifest中的登录模式在intent中不再可用，同样地，一些在intent中定义的标志也可能没有在manifest中未定义。

  ##### Using the manifest file

  当在manifest文件中声明activity时，可以指定这个activity开启时如何与当前task关联。

  <activity>标签的launchMode属性可以设置为四种不同的模式：

  ​    “standard”（默认模式）

  ​    “singleTop”

  ​    “singleTask”

  ​    “singleInstance”

  ​    这几种模式的区别体现以下四点上：

  ​    **1）当这个activity被激活的时候，会放入哪个任务栈。**

  ​    对于“standard”和“singleTop”模式，这个新被激活的activity会放入和之前的activity相同的任务栈中――除非Intent对象包含FLAG_ACTIVITY_NEW_TASK标志。

  ​    “singleTask”并不会每次都新启动一个task。如果已经存在一个task与新activity亲和度（taskAffinity）一样，则activity将启动到该task。如果不是，才启动一个新task。同一个application里面，每个activity的taskAffinity默认都是一样的。

  ​    “singleInstance”模式则表示这个新被激活的activity会重新开启一个任务栈，并作为这个新的任务栈的唯一的activity。

  ​    **2）是否可以存在这个activity类型的多个实例。**

  ​    对于“standard”和“singleTop”模式，可以有多个实例，并且这些实例可以属于不同的任务栈，每个任务栈也可以包含有这个activity类型的多个实例。 

    “singleTop"要求如果创建intent的时候栈顶已经有要创建 的Activity的实例，则将intent发送给该实例，而不发送给新的实例。 

    “singleTask”和”singleInstance”则限制只生成一个实例。

    **3）包含此activity的任务栈是否可以包含其它的activity。**

  ​    “singleInstance”模式表示包含此activity的任务栈不可以包含其它的activity。若此activity启动了另一个activity组件，那么无论那个activity组件的启动模式是什么或是Intent对象中是否包含了FLAG_ACTIVITY_NEW_TASK标志，它都会被放入另外的任务栈。在其它方面“singleInstance”模式和“singleTask”模式是一样的。

  ​    其余三种启动模式则允许包含此activity的任务栈包含其它的activity。

  ​    **4）是否每次都生成新实例**

  ​    对于默认的“standard”模式，每当响应一个Intent对象，都会创建一个这种activity类型的新的实例。即每一个activity实例处理一个intent。

  ​    对于“singleTop”模式，只有当这个activity的实例当前处于任务栈的栈顶位置，则它会被重复利用来处理新到达的intent对象。否则就和“standard”模式的行为一样。

  ​    “singleInstance”是其所在栈的唯一activity，它会每次都被重用。

  ​     对于“singleTask”模式的acitvity，在其上面可能存在其它的activity组件，所以它的位置并不是栈顶，在这种情况下，intent对象会被丢弃。（虽然会被丢弃，但是这个intent对象会使这个任务栈切换到前台）

  ​    **注意：**

  ​    当已经存在的activity实例处理新的intent时候，会调用onNewIntent()方法 
  若为了处理一个新到达的intent对象而创建了一个activity实例，则用户按下“BACK”键就会退到之前的那个activity。但若这个新到达的intent对象是由一个已经存在的activity组件来处理的，那么用户按下“BACK” 键就不会回退到处理这个新intent对象之前的状态了。

  ##### Using Intent flags

  当开启一个activity时，可以通过在intent中包含标志来修改activity的默认的与当前task的关联，然后将该intent传递给startActivity().可以修改的默认的标志为：

  - FLAG_ACTIVITY_NEW_TASK
    在一个新的task中开启一个activity。如果包含该activity的task已经运行，该task就回到前台，activity通过onNewIntent()接受处理该intent。
    这是与"singleTask"登录模式相同的行为。
  - FLAG_ACTIVITY_SINGLE_TOP
    如果要被开启的activity是当前的activity（在返回栈的顶部），已经存在的实例通过onNewIntent()接收一个调用，然后处理该intent，而非重新创建一个新的实例。
    这与"singleTop"登录模式有相同的行为。
  - [FLAG_ACTIVITY_CLEAR_TOP](http://hi.baidu.com/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_TOP)
    如果要被开启的activity已经在当前的task中运行，系统不会生成该activity的一个新的实例，在该栈顶部的所有其他的activity会被销毁，这个intent通过 onNewIntent()被传递给该重新运行的activity的实例（现在在栈顶部）。
    manifest中没有相对应的属性。
    FLAG_ACTIVITY_CLEAR_TOP经常和FLAG_ACTIVITY_NEW_TASK一起使用.当一起使用时，这些标志可以确定一个存在的activity在另一个task中的位置，并且将其放置于可以响应intent的位置（FLAG_ACTIVITY_NEW_TASK确定该activity，然后FLAG_ACTIVITY_CLEAR_TOP销毁顶部其他的activity）。如果指定的activity的登录模式是"standard"，也会被从栈中移除，一个新的实例也会被登录到它的位置来处理到来的intent。那是因为当登录模式为 "standard"时，一个新的实例总是被创建

   

   注意： 其实以上的解释一起用非常复杂，比如一般系统默认activity是 standard，但当我activity代码设置为FLAG_ACTIVITY_NEW_TASK时仍然还是生成新的activity，当设置FLAG_ACTIVITY_CLEAR_TOP 时也是生成新的activity，当FLAG_ACTIVITY_CLEAR_TOP和FLAG_ACTIVITY_NEW_TASK时也是生成新的activity，或许这句好是经典“登录模式为 "standard"时，一个新的实例总是被创建”，以下的两种方式是我经常用的。

  参考： [Activity的启动模式](http://www.cnblogs.com/blueofsky/archive/2011/12/19/2293575.html)



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


## startActivity流程
http://gityuan.com/2016/03/12/start-activity/
![image](http://upload-images.jianshu.io/upload_images/11010834-a90b6a06c1f9cf66.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
启动流程：
点击桌面App图标，Launcher进程采用Binder IPC向system_server进程发起startActivity请求；
system_server进程接收到请求后，向zygote进程发送创建进程的请求；
Zygote进程fork出新的子进程，即App进程；
App进程，通过Binder IPC向sytem_server进程发起attachApplication请求；
system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送scheduleLaunchActivity请求；
App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息；
主线程在收到Message后，通过发射机制创建目标Activity，并回调Activity.onCreate()等方法。
到此，App便正式启动，开始进入Activity生命周期，执行完onCreate/onStart/onResume方法，UI渲染结束后便可以看到App的主界面。 启动Activity较为复杂，后续计划再进一步讲解生命周期过程与系统是如何交互，以及UI渲染过程，敬请期待。





## ListView原理学习与优化总结

1. **ListVeiw:  用来展示列表的View。**
2. **适配器 : 用来把数据映射到ListView上**
3. **他首先用getCount（）函数得到要绘制的这个列表的长度，然后开始绘制第一行，怎么绘制呢？调用getView()函数** 。**ListView 针对每个item，要求 adapter “返回一个视图” (getView)，也就是说ListView在开始绘制的时候，系统首先调用getCount（）函数，根据他的返回值得到ListView的长度，然后根据这个长度，调用getView（）一行一行的绘制ListView的每一项** 
4. ![img](http://img1.51cto.com/attachment/201206/102950574.jpg) 

1. **如果你有几千几万甚至更多的选项(item)时，其中只有可见的项目存在内存（内存内存哦，说的优化就是说在内存中的优化！！！）中，其他的在Recycler中**
2. **ListView先请求一个type1视图(getView)然后请求其他可见的项目。convertView在getView中是空(null)的**
3. **当item1滚出屏幕，并且一个新的项目从屏幕低端上来时，ListView再请求一个type1视图。convertView此时不是空值了，它的值是item1。你只需设定新的数据然后返回convertView，不必重新创建一个视图**

```java
     public View getView(int position, View convertView, ViewGroup parent) { 
            System.out.println("getView " + position + " " + convertView); 
            ViewHolder holder = null; 
            if (convertView == null) { 
                convertView = mInflater.inflate(R.layout.item1, null); 
                holder = new ViewHolder(); 
                holder.textView = (TextView)convertView.findViewById(R.id.text); 
                convertView.setTag(holder); 
            } else { 
                holder = (ViewHolder)convertView.getTag(); 
            } 
            holder.textView.setText(mData.get(position)); 
            return convertView; 
        } 
```

https://blog.csdn.net/hhq163/article/details/8132723



ListView真正意义上的优化:

1.  **ViewHolder   Tag 必不可少，这个不多说！**
2. **如果自定义Item中有涉及到图片等等的，一定要狠狠的处理图片，图片占的内存是ListView项中最恶心的，处理图片的方法大致有以下几种：2.1：不要直接拿个路径就去循环decodeFile();这是找死….用Option保存图片大小、不要加载图片到内存去;2.2:  拿到的图片一定要经过边界压缩2.3:在ListView中取图片时也不要直接拿个路径去取图片，而是以WeakReference（使用WeakReference代替强引用。比如可以使        用WeakReference mContextRef）、SoftReference、WeakHashMap等的来存储图片信息，是图片信息不是图片哦！2.4:在getView中做图片转换时，产生的中间变量一定及时释放，用以下形式：**
3. **尽量避免在BaseAdapter中使用static 来定义全局静态变量，我以为这个没影响 ，这个影响很大，static是Java中的一个关键字，当用它来修饰成员变量时，那么该变量就属于该类，而不是该类的实例。所以用static修饰的变量，它的生命周期是很长的，如果用它来引用一些资源耗费过多的实例（比如Context的情况最多），这时就要尽量避免使用了..**
4. **如果为了满足需求下必须使用Context的话：Context尽量使用Application Context，因为Application的Context的生命周期比较长，引用它不会出现内存泄露的问题**
5. **尽量避免在ListView适配器中使用线程，因为线程产生内存泄露的主要原因在于线程生命周期的不可控制**



## HashMap put和get操作

当使用`HashMap`的`put`方法的时候,有两个问题要解决：

1、长度为16的数组中，元素存储在哪个位置 
2、如果`key`出现`hash`冲突,如何解决

**第一个问题**,`HashMap` 是使用下面的算法来计算元素的存放位置的。

```
 int hash = hash(key);
 int i = indexFor(hash, table.length);12
```

首先先`hash`,之后结合数组的长度进行一个**&**操作得到得到数组的下标。

**第二个问题** 则利用`Entry`类的**next**变量来实现链表,把最新的元素放到链表头,旧的数据则被最新的元素的`next`变量引用着。 
举个例子,假设元素`Entry<"1","1">`通过`hash`算法算出存到下标为0的位置上,后面又添加一个`Entry<"2","2">`, 
假设`Entry<"2","2">`通过hash算法算出也需要存到下标为0的数组中,那么此时链表是下面这个样子的:

> Entry<”2”,”2”> –> Entry<”1”,”1”>

也即是说,当`key`出现`hash`冲突的时候,**链表中的第一个元素都是后面最新添加进来的那个,之前的则被next变量引用着**。虽然这里是插入的动作,但是由于使用了链表,所以无需像数组的插入那样,进行数组拷贝。

------

**HashMap get操作**

------

这个操作的原理就比较简单,只需要根据`key`的`hashcode`算出元素在数组中的下标,之后遍历`Entry`对象链表,直到找到元素为止。

```
int hash = (key == null) ? 0 : hash(key);
for (Entry<K,V> e = table[indexFor(hash, table.length)];e != null;e = e.next) {
    Object k;
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        return e;
}
123456789
```

这里有两个注意点: 
1、这里利用`key`的`hashcode`方法和`equals`方法,所以在使用`HashMap`的时候,如果使用对象作为key,最好覆写`key`的`hashcode`和`equals`方法 
不然可能出`put`到`HashMap`的时候,成功了,但是`get`的时候却没有找到数据 
2、如果`key` `hash`冲突太多,会造成链表过长,在链表中查找元素的时候,会比较慢



## shareprefs 

commit是同步 apply是异步

可以多进程访问，但会出现冲突，需要采用进程冲突机制，比如使用一个service进行保存



## onMeasure的过程 



## 四个模式应用场景：

**singleTop适合接收通知启动的内容显示页面。例如，某个新闻客户端的新闻内容页面，如果收到10个新闻推送，每次都打开一个新闻内容页面是很烦人的。

singleTask适合作为程序入口点。例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。之前打开过的页面，打开之前的页面就ok，不再新建。

singleInstance适合需要与程序分离开的页面。例如闹铃提醒，将闹铃提醒与闹铃设置分离。singleInstance不要用于中间页面，如果用于中间页面，跳转会有问题，比如：A -> B (singleInstance) -> C，完全退出后，在此启动，首先打开的是B。

这四种模式中的Standard模式是最普通的一种，没有什么特别注意。而SingleInstance模式是整个系统的单例模式，在我们的应用中一般不会应用到。所以，这里就具体解说 **SingleTop** 和 **SingleTask**模式的运用场景：

### **1. SingleTask模式的运用场景**

最常见的应用场景就是保持我们应用开启后仅仅有一个Activity的实例。最典型的样例就是应用中展示的主页（Home页）。

假设用户在主页跳转到其他页面，运行多次操作后想返回到主页，假设不使用SingleTask模式，在点击返回的过程中会多次看到主页，这明显就是设计不合理了。

------

------

------

### **2. SingleTop模式的运用场景**

假设你在当前的Activity中又要启动同类型的Activity，此时建议将此类型Activity的启动模式指定为SingleTop，能够降低Activity的创建，节省内存！





## 自定义控件有哪几种实现方式？

在实现自定义控件时需要重写哪几个构造方法？  组合控件，继承已有控件改写功能，继承view自己实现。  需要重写onMeasure、onDraw、onLayout；自定义ViewGroup如果不使用现成的LayoutParams还需要重写getDefaultLayoutParamas等方法。 



## Multidex 

Multidex了解吗？它是用来解决什么问题的？一个工程有10W个方法，分几个dex文件？每个dex文件的方法数是多少？   Multidex用来解决单个dex文件方法数65535上限导致打包不成功的问题。一个10w方法的app打包应该有两个dex文件。其中一个方法数达到上限，另一个为100000-65535=34465。

https://www.cnblogs.com/butterfly-clover/p/5161150.html

## multidex和热修复：

https://blog.csdn.net/hp910315/article/details/51681710

## multidex分包技术定制

https://neyoufan.github.io/2017/01/20/android/Multidex%E5%8A%A0%E9%80%9F/

https://tech.meituan.com/mt_android_auto_split_dex.html