---
title: 深度理解Android技术总结
urlname: android_app_tech_summup
date: 2017-03-03
comments: true
top: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 10Android_App
tags:
    - 书籍推荐
    - Android
    - APP
---

#深度理解Android技术总结

##Android基础

###开发文档    
https://developer.android.com/guide/index.html?hl=zh-cn 

###阿里巴巴Android开发规范  
https://github.com/Blankj/AndroidStandardDevelop

###框架层知识点 
分层架构 MVC MVVM MVP
https://developer.android.com/topic/libraries/architecture/guide.html
https://github.com/googlesamples/android-architecture
https://github.com/googlesamples/android-architecture-components
组件化 模块化 容器化 Atlas small
https://yq.aliyun.com/articles/7239
依赖注入 Dagger2
https://toutiao.io/posts/5a3fp5/preview
AOP面向切面编程 Aspectj
http://blog.csdn.net/xwh_1230/article/details/78213160
http://blog.csdn.net/xwh_1230/article/details/78225258
事件驱动 EventBus otto EventPoster
http://blog.csdn.net/android2me/article/details/66973037
RxJava数据异步   
http://blog.csdn.net/caihongdao123/article/details/51897793
https://www.jianshu.com/p/5e93c9101dc5
Http Restful请求框架Retrofit
https://segmentfault.com/a/1190000005638577
框架列表
https://www.ctolib.com/cheatsheets-Android-ch.html
常用的Util
https://github.com/Blankj/AndroidUtilCode/blob/master/utilcode/README-CN.md
Android开源项目源码解析
https://github.com/android-cn/android-open-project-analysis


## Android中间件
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


<!-- more -->

## Android性能优化

### 性能指标
- 流畅**更快** ```卡顿，启动速度，页面显示速度，响应速度```
- 稳定**更稳** ```Crash， ANR```
- 节省**更省** ```内存，CPU，安装包大小，存储，功耗电量，网络```

https://github.com/openthos/openthos/wiki/understand-android
https://www.kancloud.cn/kancloud/android-performance/53238
https://developer.android.com/topic/performance/index.html
https://developer.android.com/studio/profile/index.html

### 流畅性
#### 启动速度
https://developer.android.com/topic/performance/launch-time.html
https://juejin.im/post/5874bff0128fe1006b443fa0

#### 页面响应速度优化
https://www.zhihu.com/question/47702122
http://www.cnblogs.com/lzl-sml/p/5223704.html

#### UI渲染显示速度提升
https://blog.csdn.net/yanbober/article/details/48394201

### 稳定性
- 稳定性问题 http://gityuan.com/2016/06/19/stability_summary/
- ANR分析策略  http://rayleeya.iteye.com/blog/1955652
- anr google https://developer.android.com/topic/performance/vitals/anr.html#fix_the_problems
- ANR超时源码解析 http://gityuan.com/2016/07/02/android-anr/
- ANR信息收集 http://gityuan.com/2016/12/02/app-not-response/  
- ANR input  http://gityuan.com/2017/01/01/input-anr/
- app crash  http://gityuan.com/2016/06/24/app-crash/
- native crash http://gityu an.com/2016/06/25/android-native-crash/

#### 稳定性分析及工具
- Android Profile 工具集 https://developer.android.com/studio/profile/index.html
- Device Monitor工具集 https://developer.android.com/studio/profile/monitor.html#usage
- 性能优化 http://rayleeya.iteye.com/blog/1961005

- traceview 
http://gityuan.com/2017/07/11/android_debug/
https://bxbxbai.github.io/2014/10/25/use-trace-view/
https://developer.android.com/studio/profile/traceview.html#overview
https://blog.csdn.net/innost/article/details/9008691 
https://blog.csdn.net/u011240877/article/details/54347396#2%E4%BD%BF%E7%94%A8-android-studio-%E7%94%9F%E6%88%90-trace-%E6%96%87%E4%BB%B6

- systrace 
http://gityuan.com/2016/01/17/systrace/
https://developer.android.com/studio/command-line/systrace.html

- dumpsys 
http://gityuan.com/2015/08/22/tool-dumpsys/
https://developer.android.com/studio/command-line/dumpsys.html

- dmtracedump
https://developer.android.com/studio/command-line/dmtracedump.html

Android Device Monitor  

Android Device Monitor is a standalone tool that provides a UI for several Android app debugging and analysis tools.However, most components of the Android Device Monitor are **deprecated** in favor of updated tools available in Android Studio 3.0 and higher. The table below helps you decide which developer tools you should use. 

Android Device Monitor component	What you should use
* Dalvik Debug Monitor Server (DDMS)	
This tool is deprecated. Instead, use Android Profiler in Android Studio 3.0 and higher to profile your app's CPU, memory, and network usage.
If you want to perform other debugging tasks, such as sending commands to a connected device to set up port-forwarding, transfer files, or take screenshots, then use the Android Debug Bridge (adb), Android Emulator, Device File Explorer, or Debugger window.

* Traceview	
If you want to inspect existing .trace files, or ones you've captured by instrumenting your app with the Debug class, keep using Traceview.
If you want to record new method traces and inspect realtime CPU usage of your app's processes, use Android Studio's CPU profiler.

* Systrace	
If you need to inspect native system processes and address UI jank caused by dropped frames, use systrace from the command line.
Otherwise, use Android Studio's CPU profiler to profile your app's processes.

* Tracer for OpenGL ES	Use the Graphics API Debugger.
* Hierarchy Viewer	
If you want to inspect your app's view hierarchy at runtime, use Layout Inspector.

If you want to profile the rendering speed of your app's layout, use Window.OnFrameMetricsAvailableListener as described in this blog post.

Pixel Perfect	Use Layout Inspector.


**ANR分析套路**
http://maoao530.github.io/2017/02/21/anr-analyse/
https://blog.csdn.net/sinat_34157462/article/details/78651870
https://blog.csdn.net/wei_lei/article/details/70311702
http://www.cnblogs.com/purediy/p/3225060.html

#### crash - 内存泄露OutOfMemoryError
https://www.jianshu.com/u/b8dad3885e05
http://rayleeya.iteye.com/blog/1956059
http://www.voidcn.com/article/p-txoxuyet-bqt.html 
https://juejin.im/entry/59f7ea06f265da43143ffee4
http://blog.csdn.net/qyf2010qyf/article/details/52852000

###节省性

#### 减少apk安装包大小
https://developer.android.com/topic/performance/reduce-apk-size.html#apk-structure
**避免使用枚举**
https://zhuanlan.zhihu.com/p/25865835
http://www.10tiao.com/html/330/201704/2653578978/2.html
https://blog.csdn.net/hp910315/article/details/48975655
https://blog.csdn.net/my_truelove/article/details/70519234

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
