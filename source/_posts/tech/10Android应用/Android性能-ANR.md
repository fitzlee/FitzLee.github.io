---
title: Android性能-ANR
urlname: android_app_perfermance_anr
date: 2017-04-20
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 10Android_App
tags:
    - 书籍推荐
    - Android
    - anr
    - 性能优化
---

## ANR基本分析定位方法

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


## 稳定性
- ANR原理分析  http://rayleeya.iteye.com/blog/1955652
- 稳定性问题 http://gityuan.com/2016/06/19/stability_summary/
- ANR超时源码解析 http://gityuan.com/2016/07/02/android-anr/
- ANR信息收集 http://gityuan.com/2016/12/02/app-not-response/  
- ANR input  http://gityuan.com/2017/01/01/input-anr/

### anr处理思路
**ANR原因**
Application Not Responding 三种类型，主线程事件长时间无法执行
 - KeyDispatchTimeout(input)
```
 static final int KEY_DISPATCHING_TIMEOUT = 5*1000; 
```
- BroadcastTimeout(broadcast)
```    
static final int BROADCAST_FG_TIMEOUT = 10*1000;  
static final int BROADCAST_BG_TIMEOUT = 60*1000;  
```
- ServiceTimeout(service)
```
static final int SERVICE_TIMEOUT = (20*1000);
static final int SERVICE_BACKGROUND_TIMEOUT = SERVICE_TIMEOUT * 10;
```


**查看日志**
1. ANR存储路径  /data/anr/ 
2. dropbox存储路径   /data/system/dropbox/ 
2. tombstones存储路径  /data/tombstones

Process: com.android.contacts 进程名字
Flags: 0x38d8be4d 应用安装标志，预装还是安装等
Package: com.android.contacts v24 (1.2) (报名和版本号)
Foreground: Yes (anr在前台还是后台)
Activity: com.android.contacts/com.mediatek.contacts.list.ContactListMultiChoiceActivity  
Subject: Input dispatching timed out (Waiting to send key event because the focused window has not finished processing all of the input events that were previously delivered to it.  Outbound queue length: 0.  Wait queue length: 1.)   （ANR类型信息）
Build: D-171117V33:user/release-keys



CPU usage from 74494ms to 0ms ago (2019-11-09 21:32:06.966 to 2019-11-09 21:33:21.460):
  104% 323/mobile_log_d: 17% user + 86% kernel / faults: 2622 minor
  56% 1041/system_server: 41% user + 14% kernel / faults: 30698 minor 36 major
  34% 1408/com.android.phone: 28% user + 6% kernel / faults: 17027 minor 35 major
  15% 5541/android.process.acore: 13% user + 2.3% kernel / faults: 10865 minor 49 major
  10% 248/servicemanager: 4.4% user + 6.5% kernel / faults: 63 minor
  10% 249/surfaceflinger: 5.1% user + 5.6% kernel / faults: 1645 minor 1 major
  8.6% 215/logd: 2.7% user + 5.8% kernel / faults: 310 minor 11 major
  5.6% 1848/com.android.contacts: 4.4% user + 1.1% kernel / faults: 9569 minor 677 major
  3.7% 1155/com.android.systemui: 2.9% user + 0.7% kernel / faults: 12145 minor 50 major
98% TOTAL: 54% user + 42% kernel + 0.1% iowait + 0% softirq
**单进程信息**
读取/proc/<pid>/task/<pid>/stat，utime，stime两次相差/两次采样周期的系统uptime；进程差值时间
56% 1041/system_server: 41% user + 14% kernel / faults: 30698 minor 36 major
cpu占用信息  进程信息: 用户态时间执行比 + kernel执行时间比 / 发生page faults的次数，一个minor，一个是major
**总TOTAL信息**
读取/proc/stat信息  http://www.linuxhowtos.org/System/procstat.htm
98% TOTAL: 54% user + 42% kernel + 0.1% iowait + 0% softirq
user: normal processes executing in user mode
nice: niced processes executing in user mode
system: processes executing in kernel mode
idle: twiddling thumbs
iowait: waiting for I/O to complete
irq: servicing interrupts
softirq: servicing softirqs




**ANR分析套路**
http://maoao530.github.io/2017/02/21/anr-analyse/
https://blog.csdn.net/sinat_34157462/article/details/78651870
https://blog.csdn.net/wei_lei/article/details/70311702
http://www.cnblogs.com/purediy/p/3225060.html

1. trace日志和logcat日志要结合起来看。
1. trace中的ANR原因，进程号，时间，首个调用栈等信息非常关键。
1. logcat日志中一方面查找对应时间的日志，检查应用的行为，检查具体导致SIG:3发出的原因。
3. ANR in XXX也是一个关键字，可以在logcat日志中检索。不过这句日志打印的时间通常要比ANR发生的时间晚一些。找到这部分log后，应该顺着向前继续查找ANR发生的日志。
4. 线程状态，锁，binder等。


查看main线程调用栈，确认ANR栈类型，不同栈类型问题分析方法有所差异
等锁 
Binder对端
wait栈
java标准接口 
NativePollOnce

ANR时间点确认：logcat里搜索“anr in”，或者在log_events里直接搜索anr。请注意logcat搜到的anr in时间很可能比anr时间滞后几秒。这么做是为了确认我们anr的时间范围。之前讲过了各种类型的anr时间。结合我们可以知道我们重点要关注的是什么时间段。例如后台广播在后台anr。那么我们重点关注日志范围是 （ANR时间-60秒）~ ANR时间
查看kmsgcat-log。确认线程是否D状态、是否多个线程Block
可以搜索卡住的线程是否被hungtask打印，hungtask发现D状态会主动触发打印

### crash处理思路
