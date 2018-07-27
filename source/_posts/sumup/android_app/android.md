---
title: Android 架构和性能优化
urlname: android_app_art_opt_sum
#date: 2017-04-10
top: true
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 10Android_App
tags:
    - Android
    - 架构
    - 性能优化
	- 框架分析
	- 插件机制
---

# Android应用

## 框架原理
https://github.com/android-cn/android-open-project-analysis
https://github.com/guoxiaoxing/android-open-framework-analysis
https://github.com/android-cn/android-open-project-analysis
https://github.com/BaronZ88/Blog

依赖注入 Dagger2
https://toutiao.io/posts/5a3fp5/preview
AOP面向切面编程 Aspectj
http://blog.csdn.net/xwh_1230/article/details/78213160
http://blog.csdn.net/xwh_1230/article/details/78225258
事件驱动 EventBus otto EventPoster
http://blog.csdn.net/android2me/article/details/66973037
RxJava数据异步
https://www.jianshu.com/p/5e93c9101dc5
Http Restful请求框架Retrofit
https://segmentfault.com/a/1190000005638577
框架列表
https://www.ctolib.com/cheatsheets-Android-ch.html
常用的Util
https://github.com/Blankj/AndroidUtilCode/blob/master/utilcode/README-CN.md
Android开源项目源码解析
https://github.com/android-cn/android-open-project-analysis



## 架构分层

分层架构 MVC MVVM MVP
https://developer.android.com/topic/libraries/architecture/guide.html
https://github.com/googlesamples/android-architecture
https://github.com/googlesamples/android-architecture-components

组件化 模块化 容器化 Atlas small
https://yq.aliyun.com/articles/7239


## 性能优化
### 性能指标
- 流畅**更快** ```卡顿，启动速度，页面显示响应速度```
- 稳定**更稳** ```Crash， ANR```
- 节省**更省** ```内存，CPU，安装包大小，存储，功耗电量，网络```


<!-- more -->


### 更快-流畅性
#### 启动速度
https://developer.android.com/topic/performance/launch-time.html
https://juejin.im/post/5874bff0128fe1006b443fa0

Google给出的优化方法：
1. 利用提前展示出来的Window，快速展示出来一个界面，给用户快速反馈的体验；
2. 避免在启动时做密集沉重的初始化（Heavy app initialization）；
考虑异步初始化三方组件，不阻塞主线程；
延迟部分三方组件的初始化；实际上我们粗粒度的把所有三方组件都放到异步任务里，可能会出现WorkThread中尚未初始化完毕但MainThread中已经使用的错误，因此这种情况建议延迟到使用前再去初始化；
而如何开启WorkThread同样也有讲究，这个话题在下文详谈。
项目优化：
将友盟、Bugly、听云、GrowingIO、BlockCanary等组件放在WorkThread中初始化；
延迟地图定位、ImageLoader、自有统计等组件的初始化：地图及自有统计延迟4秒，此时应用已经打开；而ImageLoader
因为调用关系不能异步以及过久延迟，初始化从Application延迟到SplashActivity；而EventBus因为再Activity中使用所以必须在Application中初始化。

3. 定位问题：避免I/O操作、反序列化、网络操作、布局嵌套等。
通过对traceview的详细跟踪以及代码的详细比对，我发现卡顿发生在：
部分数据库及IO的操作发生在首屏Activity主线程；
Application中创建了线程池；
首屏Activity网络请求密集；
工作线程使用未设置优先级；
信息未缓存，重复获取同样信息；
流程问题：例如闪屏图每次下载，当次使用；

以及其它细节问题：
执行无用老代码；
执行开发阶段使用的代码；
执行重复逻辑；
调用三方SDK里或者Demo里的多余代码；

项目修改：
1数据库及IO操作都移到工作线程，并且设置线程优先级为THREAD_PRIORITY_BACKGROUND，这样工作线程最多能获取到10%的时间片，优先保证主线程执行。
2流程梳理，延后执行；
实际上，这一步对项目启动加速最有效果。通过流程梳理发现部分流程调用时机偏早、失误等，例如：
更新等操作无需在首屏尚未展示就调用，造成资源竞争；
调用了IOS为了规避审核而做的开关，造成网络请求密集；
自有统计在Application的调用里创建数量固定为5的线程池，造成资源竞争，在上图traceview功能说明图中最后一行可以看到编号12执行5次，耗时排名前列；此处线程池的创建是必要但可以延后的。
修改广告闪屏逻辑为下次生效。


#### 页面响应速度优化
https://www.zhihu.com/question/47702122
http://www.cnblogs.com/lzl-sml/p/5223704.html
应用UI问题分析：https://blog.csdn.net/yanbober/article/details/48394201

卡顿原因：
```
人为在UI线程中做轻微耗时操作，导致UI线程卡顿；
布局Layout过于复杂，无法在16ms内完成渲染；
同一时间动画执行的次数过多，导致CPU或GPU负载过重；
View过度绘制，导致某些像素在同一帧时间内被绘制多次，从而使CPU或GPU负载过重；
View频繁的触发measure、layout，导致measure、layout累计耗时过多及整个View频繁的重新渲染；
内存频繁触发GC过多（同一帧中频繁创建内存），导致暂时阻塞渲染操作；
冗余资源及逻辑等导致加载和执行缓慢；
臭名昭著的ANR；
```

解决思路：
使用HierarchyViewer分析UI性能
使用GPU过度绘制分析UI性能
使用Lint进行资源及冗余UI布局等优化
使用Memory监测及GC打印与Allocation Tracker进行UI卡顿分析
使用Traceview和dmtracedump进行分析优化
使用Systrace进行分析优化
使用traces.txt文件进行ANR分析优化

一个是响应速度，要保证界面之间跳转的时候没有延迟，也就是说要保证onClick之后， 
1. 此Activity/Fragment的pause不会占用主线程太多时间 
2. onClick事件里面不要写耗时的操作 
3. 新的Activity的create、start、resume等生命周期函数不要占用太多时间

1. 布局优化，使用incluede、merge、viewstub标签 
2. 内存优化，该释放的东西一定要释放，图片一定要压缩防止OOM，资源不用滥用，service之类的一定要释放。 
3. UI线程除了展示其他事情尽量不做。但是其他线程一定要管理好。如果任务多，可以试试AsyncTask。 
4. 不要生成大量的变量，能复用的尽量复用，不能复用的尽量干掉。 
5. 可以缓存的数据可以缓存。网络请求的次数要减少。比如一个页面有3个请求。可以设计一下，看看能不能合并为请求一次，把3次需要的数据都返回过来。

编码过程原则：
第一：先UI，后数据。比如页面跳转，就要采用这种原则，获取数据的过程UI可以是一个读取状态的展示。
第二：异步计算存储。比如持久化数据，各种压缩算法，都采用异步的方式。
第三：精简View处理。对于列表类型的View，注意layout的计算和draw时的计算。

调试过程原则：
第一：注意内存使用。感觉对流畅度影响比较大是内存抖动。
第二：开发者工具布局边界的使用，尽量减少布局层级
第三：开发者工具GPU过度绘制，控制背景以及重叠区域的过渡绘制
第四：开发者工具GPU呈现模式分析，这里是帧率最好的体现，这里可以结合Monitors和内存CPU的trace文件来分析开销。



### 更稳-稳定性
Android Bug收集解决方案： https://www.jianshu.com/p/3b66f15babfb
Android 平台 Native 代码的崩溃捕获机制及实现： https://zhuanlan.zhihu.com/p/27834417
1. java crash: UncaughtExceptionHandler
2. native crash： coffeecatch或监控logcat日志
int sigaction(int signum,const struct sigaction *act,struct sigaction *oldact));
3. 代码review检测：findbugs/codex等检测工具
4. 测试monkey和beta：稳定性保证
5. 代码上线后，使用工具收集bug，上报大数据平台。

anr原因和分析：

ANR分析主要是Input、Broadcast、Service三种ANR， 对应的时间主要有以下几种：其中后台时间相对长一些，broadcast可以达到60s，但是前台一般最高10s，后台service是20s。 
类型                  前台        后台
Input                   8s          8s
Forground Broadcast      10s         20s
Background Broadcast     10s         60s
Service                 10s         20s

定位问题思路是，首先查看anr的进程，类型，cpu占用，iowait等指标
activity、broadcast、还是service，是前台还是后台。
cpu占用是否过高，iowait是否正常，是否是由于整机问题影响

查看线程调用栈，main主线程是由于什么原因导致的anr， 等锁lock，是否死锁？ 是否binder卡住了？ 是否no focus windows可能是由于进程被杀，搜索是否有window change查看界面切换的影响？wait和notifiy导致的死锁，查看相关代码？其他卡住的地方File、database、sharepreference可以怀疑文件系统等。

引起ANR问题的根本原因，总的来说可以归纳为两类：
	• 应用进程自身引起的，例如：
主线程阻塞、挂起、死循环
应用进程的其他线程的CPU占用率高，使得主线程无法抢占到CPU时间片
	• 其他进程间接引起的，例如：
当前应用进程进行进程间通信请求其他进程，其他进程的操作长时间没有反馈
其他进程的CPU占用率高，使得当前应用进程无法抢占到CPU时间片
分析ANR问题时，以上述可能的几种原因为线索，通过分析各种日志信息，大多数情况下你就可以很容易找到问题所在了。
http://www.cnblogs.com/zl1991/p/5947157.html

crash异常：
java exception未捕获
native crash fatal error或内存溢出等。

内存泄露：
内存泄露从入门到精通三部曲之基础知识篇
https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=400674207&idx=1&sn=a9580ca0dffc62a6d7dbb8fd3d7a2ef1&scene=0&key=b410d3164f5f798e39485589b2e5250271c110cf11a7560f6fb949e56b993a0d85bc5f80af60383724e1fed3210be25c&ascene=0&uin=MjAyNzY1NTU%3D&devicetype=iMac+MacBookPro12%2C1+OSX+OSX+10.10.5+build(14F27)&version=11020201&pass_ticket=mM7MjOVn2d4S0ZkDKHWDkG9Cgdo%2FKiYCd%2BCFXjwe%2FCo%3D
内存泄露从入门到精通三部曲之排查方法篇
https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=400891536&idx=1&sn=0b6c629b0abe4a359d6552cd244c0c0c&scene=0&key=d4b25ade3662d643209f4b75e10fa26857f77b6a60d1e1c0b7361529f3dcdf8c1506749b0ae4397f4d74e8871e24f62e&ascene=0&uin=MjAyNzY1NTU%3D&devicetype=iMac+MacBookPro12%2C1+OSX+OSX+10.11.1+build(15B42)&version=11020201&pass_ticket=T1NGa3s1cwXUixFccOjWKBGBwZ3Vfeilv%2BMoVjLgOok%3D
内存泄露从入门到精通三部曲之常见原因与用户实践
https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=401107957&idx=2&sn=4b95bcfedd762b987ec57f60f80b1f94&scene=0&key=d72a47206eca0ea9acb409bf4ac5ee372a62883d23063e79d0457ed91b0bddd06972df28e34844fa96f1c1c39a6e2794&ascene=0&uin=MjAyNzY1NTU%3D&devicetype=iMac+MacBookPro12%2C1+OSX+OSX+10.11.1+build(15B42)&version=11020201&pass_ticket=tzHHVrqCnktj8H4OEQz2FQg1FtUowQPk7zymt5mui1s%3D



### 更省-节省性

#### 减少apk安装包大小
Android APK 瘦身实战-JOOX Music项目：https://zhuanlan.zhihu.com/p/26772897
https://developer.android.com/topic/performance/reduce-apk-size.html#apk-structure

1. 图片资源
删除无用资源
压缩PNG图片，并将能转成JPG的图片进行转换
压缩JPG图片
使用WebP文件格式
使用矢量图形

2. 代码方面（JAR包，so库等等）
删除不必要的生成代码、
避免使用枚举，
**避免使用枚举**
https://zhuanlan.zhihu.com/p/25865835
http://www.10tiao.com/html/330/201704/2653578978/2.html
https://blog.csdn.net/hp910315/article/details/48975655
https://blog.csdn.net/my_truelove/article/details/70519234
减少本机二进制文件的大小(删除调试符号而不是提取本机库)

3. 资源混淆
关于资源混淆，其核心就在于对APK中resources.arsc文件的修改，大家都知道，Android项目中res目录下每个资源都会有其对应的ID，而ID在R文件下关联着资源名称（如http://R.string.xxx），通过这些ID我们可以很方便的锁定某一项资源，而资源混淆的原理就在于修改ID对应的资源路径进行优化处理（如将res/drawable/xxx修改成res/drawable/a,或者修改为r/d/a），通过这个方式可以大大减小resources.arsc文件的内容，从而达到减包的目的。
资源混淆方面，JOOX采用的是微信提供的Android资源混淆打包工具。工具传送门：
安装包立减1M--微信Android资源混淆打包工具:https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=214472913&idx=1&sn=92b54b5fcd9bbab6513e46d92095a07f&scene=21#wechat_redirect



#### 减少过渡后台，android后台优化
https://developer.android.com/training/best-background.html

#### 减少view绘制过度
https://blog.csdn.net/qq_19711823/article/details/65627790
https://www.jianshu.com/p/2cc6d5842986
https://blog.csdn.net/yanbober/article/details/48394201
https://yq.aliyun.com/articles/82572

#### 减少流量消耗
https://developer.android.com/topic/performance/vitals/bg-network-usage.html#detect_the_problem
https://developer.android.com/training/basics/network-ops/data-saver.html#monitor-changes

#### 减少电量消耗
https://developer.android.com/topic/performance/power/index.html

#### 减少内存占用
https://developer.android.com/topic/performance/memory.html#code
https://developer.android.com/topic/performance/memory-overview.html



### 性能优化的建议
https://blog.csdn.net/carson_ho/article/details/79708444
https://juejin.im/post/5a0d30e151882546d71ee49e
https://www.ctolib.com/docs/sfile/notes-master/Android-Java/AndroidPerformancePatterns.html
https://github.com/Piasy/notes/blob/master/Android-Java/AndroidPerformancePatterns.md
https://www.jianshu.com/u/fdb392adfbed
https://segmentfault.com/a/1190000012413613
https://zhuanlan.zhihu.com/p/26364608

### Android优化checklist
https://aeli.gitbooks.io/android-training-course/performance/performance-tips.html




## 异常处理

## 中间层原理

###插件化
http://blog.csdn.net/ganyao939543405/article/details/76146760
###热修复 
https://github.com/Tencent/tinker/wiki
###Hook框架 
http://www.snowdream.tech/2016/09/02/android-install-xposed-framework/
https://jaq.alibaba.com/community/art/show?articleid=809
##VirtualXposed 和epic
https://github.com/android-hacker/VirtualXposed
https://github.com/tiann/epic
http://weishu.me/2017/12/02/non-root-xposed/


## 最佳实践




# Android系统

http://gityuan.com/android/
https://github.com/guoxiaoxing/android-open-source-project-analysis
http://weishu.me/archives/
http://androidxref.com/