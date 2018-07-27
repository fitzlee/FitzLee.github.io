---
title: Jvm Params配置调优参数
urlname: jvm_art_params_analysis
#date: 2016-10-09
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 03JVM&ART
tags:
    - java
    - jvm
    - params
    - gc
---

# JVM参数类型和调整

所有线程共享的内存主要有两块：堆内存和方法区。

其中堆内存分为两块：新生代Young generation（Eden区、From Survivor区、To Survivor区）、老年代Tenured generation。

方法区有人也称之“永久代”，但是它们并不等同。方法区是JVM的规范，而永久代是该规范的一种实现方式。从jdk1.7开始已经逐步去除“永久代”，在JDK8中取而代之的是“元空间”（Metaspace）。

元空间与永久代之间最大的区别在于：**元空间并不在虚拟机中，而是使用本地内存**。因此，默认情况下，元空间的大小仅受本地内存限制

下面是JVM的一些主要参数：

### 1. 基本参数

| 参数  | 描述 |
| ----- | ---- |
| -XX:+ | 打开 |
| -XX:- | 关闭 |

### 2. 内存大小配置参数

| 参数                    | 描述                                         |
| ----------------------- | -------------------------------------------- |
| -Xms                    | 初始堆内存大小                               |
| -Xmx                    | 最大堆内存大小                               |
| -Xmn                    | 年轻代内存大小                               |
| -Xss                    | 线程私有的虚拟机栈大小                       |
| -XX:MaxPermSize=64m     | 永久代最大值                                 |
| -XX:PermSize            | 永久代初始值                                 |
| -XX:MetaspaceSize       | 元空间初始大小                               |
| -XX:MaxMetaspaceSize    | 元空间最大值                                 |
| -XX:MaxDirectMemorySize | 直接内存大小，默认与Java堆最大值（-Xmx）一样 |

### 3. JVM调试参数

| 参数                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| -verbose:gc                                                  | 记录GC运行及运行时间                                         |
| -XX:+PrintGCDetails                                          | 记录GC运行时的详细数据信息，以及在进程结束时打印当前的内存各区域分配情况。 |
| -XX:+PrintGCTimeStamps                                       | 打印垃圾收集时间戳                                           |
| -Xloggc:{gcLogPath}                                          | gc日志存放路径                                               |
| -XX:+HeapDumpOnOutOfMemoryError                              | 在内存溢出的时候生成Heap dump文件                            |
| -verbose:class、-XX:+TraceClassLoading                       | 查看类加载信息(要求Product版虚拟机)                          |
| -XX:+TraceClassUnLoading                                     | 查看类卸载信息(要求FastDebug版虚拟机)                        |
| -Xdebug -Xnoagent -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8000 | 用于远程调试                                                 |

### 4. 垃圾收集器

| 参数                    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| -XX:+UseSerialG         | 使用Serial+Serial Old的收集器组合进行内存回收。              |
| -XX:+UseParNewGC        | 使用ParNew+Serial Old的收集器组合进行内存回收。              |
| -XX:+UseConcMarkSweepGC | 使用ParNew+CMS+Serial Old的收集器组合进行内存回收。Serial Old作为出现Concurrent Mode Failure失败后的后备收集器使用。 |
| -XX:+UseParallelGC      | 使用Parallel Scavenge+Serial Old(PS Mark Sweep)收集器组合进行内存回收。 |
| -XX:+UseParallelOldGC   | 使用Parallel Scavenge+Parallel Old收集器组合进行内存回收。   |

### 5. JVM调优参数

| 参数                                  | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| -XX:SurvivorRatio                     | 新生代中Eden区域和Survivor区域（单个Survivor）的容量比值，默认为8 |
| -XX:NewRatio                          | 堆内存中老生代和年轻代的容量比值。例：NewRatio=2，表明old：new=2:1 |
| -XX:PretenureSizeThreshold            | 直接晋升到老年代的对象大小，大于该值的对象直接在老年代分配。 |
| -XX:MaxTenuringThreshold              | 对象在新生代中能存活的最大年龄。                             |
| -XX:+UseAdaptiveSizePolicy            | 动态调整Java堆中各个区域的大小以及进入老年代的年龄（限Parallel Scaverge收集器） |
| -XX:+HandlePromotionFailure           | 允许老年代分配担保失败，开启后可以冒险YGC。                  |
| -XX:ParallelGCThreads                 | 设置并行GC时进行内存回收的线程数                             |
| -XX:GCTimeRatio                       | 默认为99，即允许1%的GC时间。GC时间占总时间的比例由公式1/(1+GCTimeRatio)得出（限Parallel Scaverge收集器） |
| -XX:MaxGCPauseMillis                  | 设置GC的最大停顿时间（限Parallel Scaverge收集器）            |
| -XX:+CMSInitialingOccupancyFraction   | 设置CMS收集器在老年代空间被使用多少后触发Full GC。默认值是68，即68%。（限CMS收集器） |
| -XX:+UseCMSCompactionAtFullCollection | 设置CMS在完成垃圾收集后进行一次内存碎片整理。（限CMS收集器） |
| -XX:+CMSFullGCsBeforeCompaction       | 设置CMS执行多少次GC后，下次GC时进行一次内存碎片整理，默认为0。即每次都整理。 |
| -Xnoclassgc                           | 不回收无用类                                                 |





参考：http://www.cnblogs.com/z-sm/p/6253335.html