---
title: Jvm GC 垃圾回收
urlname: jvm_art_gc_analysis
#date: 2016-10-09
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 03JVM&ART
tags:
    - java
    - jvm
    - gc
    - oom
---

## 垃圾回收哪些内存

JVM运行时数据区中，线程私有的程序计数器、虚拟机栈、本地方法栈随线程的创建和退出而自动产生和销毁，不需要垃圾回收。JVM垃圾回收的是共享区域的内存，**主要是方法区和Java堆内存的回收**。

### **1.1、方法区**

方法区的垃圾收集主要是回收废弃常量和无用的类（卸载类）。回收废弃常量与下面将介绍的回收Java堆中的对象很相似，而判定“无用的类”需同时满足三个条件：

1、该类所有实例已被回收，即Java堆中无该类的任何实例。（下层）

2、该类对应的java.lang.Class对象没有在任何地方被引用，无在任何地方通过反射访问该类的方法。（下层）

3、加载该类的ClassLoader已被回收。（上层）

### 1.2、堆

Java堆里面存放着几乎所有的对象实例，垃圾收集器对堆进行回收前，首先要做的就是确定哪些对象可以作为垃圾回收。

JDK1.2后，Java对引用概念进行了扩充，将引用分为强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Soft Reference）、虚引用（Phantom Reference）。被强引用的对象（我们通常所用的对象就是强引用的）一定不会被回收，被其他引用的对象却可能被回收。

总的来说，回收的堆对象有两类：

1、有被引用的对象：即被软引用、弱引用、虚引用所引用的对象可能被当垃圾回收。

1、软引用：软引用对象在系统将要发生内存溢出前会被回收

2、弱引用：弱引用对象只能躲过一次垃圾回收，躲过一次后就被回收

3、虚引用：虚引用对象肯定会被回收，一个对象是否有虚引用完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例，对象被设置成虚引用关联的唯一作用是在对象被垃圾收集时收到一个系统通知。

2、无被引用的对象：需要检查哪些对象已死（没被引用），主要有两种算法检测（**3.1节详述具体实现**）：

**1、引用计数法**：对象有个计数器记录被引用的个数，为0者可被回收。采用此法的有微软的COM技术、使用ActionScript3的FlashPlayer、Python等。此法存在对象间循环引用的问题导致对象不能被回收。

**2、可达性分析法：**主流Java虚拟机采用此法。选取一系列GC Roots对象作为起点分析引用链确定不可达对象。采用此法的有Java、C#、Lisp等。对于不可达的对象，如果其覆盖了finalize()方法并且没被执行过，则在JVM对之回收前其有一次逃过被回收的机会。如下：JVM将该对象放入F-Queue队列，并由虚拟机自动建立的、低优先级的Finalizer线程稍后去执行它。若finalize()里此对象又被成员变量（类变量或实例变量）引用，则此对象复活，不会被回收；否则被回收。

copyfrom：http://www.cnblogs.com/z-sm/p/6243378.html


<!-- more -->

## 判断对象是否存活？

### GC的两种判定方法：引用计数与根搜索算法

- 引用计数： 给对象添加一个引用计数器，每当有一个地方引用该对象时，计数器值加1，当引用失效时，计数器值减1,。任何时候计数器都为0的对象就是不可能再被使用的。它很难解决对象之间相互循环引用问题。
- 根搜索算法（GC Roots Traceing）: 通过一系列名为“GC Roots”的对象作为起点，从这些节点开始向下搜索，搜索走过的路径成为引用链，当一个对象到GC Roots没有任何引用链相连时，则证明此对象不可用。

#### 1. 引用计数算法

优势：实现简单，效率高。
致命缺陷：无法解决对象相互引用的问题——会导致对象的引用虽然存在，但是已经不可能再被使用，却无法被回收。

#### 2. 可达性分析算法

对象到GC Roots没有引用链，则回收。

GC Roots包括：

- （1）Java虚拟机栈中引用的对象。
- （2）方法区中类静态属性引用的对象。
- （3）方法去中常量引用的对象。
- （4）本地方法栈中Native方法（JNI）引用的对象。

### 如何确定不可达对象

**1、引用计数法：（采用此法的有微软的COM计数、使用ActionScript3的FlashPlayer、Python等）**

**思想**：

堆中的每一个对象有一个引用计数，当一个对象被创建并把指向该对象的引用赋值给一个变量时，引用计数置为1，当再把这个引用赋值给其他变量时，引用计数加1，当一个对象的引用超过了生命周期或者被设置为新值时，对象的引用计数减1，任何引用计数为0的对象都可以被当成垃圾回收。当一个对象被回收时，它所引用的任何对象计数减1，这样，可能会导致其他对象也被当垃圾回收。

**存在的问题**：循环引用问题，示例如下。如果采用引用计数法，则两个实例没法被回收，但实际被回收了，说明JVM采用的不是引用计数法。

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class ReferenceCountingGC {
 2     public Object instance = null;
 3     private static final int _1MB = 1024 * 1024;
 4     /**
 5      * 这个成员属性的唯一意义就是占点内存,以便能在GC日志中看清楚是否被回收过
 6      */
 7     private byte[] bigSize = new byte[2 * _1MB];
 8     
 9     public static void testGC() {
10         // 定义两个对象
11         ReferenceCountingGC objA = new ReferenceCountingGC();
12         ReferenceCountingGC objB = new ReferenceCountingGC();
13         
14         // 给对象的成员赋值，即存在相互引用情况
15         objA.instance = objB;
16         objB.instance = objA;
17         
18         // 将引用设为空，即没有到堆对象的引用了
19         objA = null;
20         objB = null;
21         
22         // 进行垃圾回收
23         System.gc();    
24     }
25     
26     public static void main(String[] args) {
27         testGC();    
28     }
29 }
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**2、关于可达性分析法：（采用此法的有Java、C#、Lisp等）**

**思想**：选取一系列GCRoots对象作为起点，开始向下遍历搜索其他相关的对象，搜索所走过的路径成为引用链，遍历完成后，若一个对象到GCRoots对象没有任何引用链，则证明此对象是不可用的，可以被当做垃圾进行回收。

**可作为GC Roots的对象**：

1、方法区中静态属性引用的对象、常量引用的对象

2、虚拟机栈（栈中的本地变量表）中引用的对象

3、本地方法栈中JNI（Native方法）引用的对象

**具体实现**（确定GC Roots节点），以HotSpot的为例：

问题——枚举寻找出可作为GC Roots的节点耗时：可作为GCRoots的结点主要在全局性的引用（如常量或类静态属性）与执行上下文（如栈帧中的局部变量表）中，很多应用仅方法区就数百兆，逐个检查变量看是否是GC Roots很耗时。

方法（寻找GC Roots节点）：采用一组称为**OopMap**的数据结构，在特定代码位置记录下哪些变量是引用，这样GC时利用该结构得知哪些是GC Roots从而避免逐个检查变量。

#### 3. 关于引用

JDK1.2之后，Java对引用进行了扩充。

- （1）强引用：Object obj = new Object()，不会被jvm回收
- （2）软引用：在内存溢出异常发生之前才被强制回收。
- （3）弱引用：延迟到下次垃圾回收之前再被回收。
- （4）虚引用：仅为了在被回收时收到一个系统通知。

Java中提供这四种引用类型主要有两个目的：第一是可以让程序员通过代码的方式决定某些对象的生命周期；第二是有利于JVM进行垃圾回收。

- 强引用：程序代码中的普通引用。如Object obj = new Object(),只要强引用存在，垃圾回收器就不会回收。在不使用对象时应及时将引用设置为null，便于垃圾回收。
- 软引用：描述一些有用但并非必须的对象。对于软引用关联的对象在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收。SoftRefence
- 弱引用：描述非必须对象，比软引用弱一些。被弱引用关联的对象只能生存到下一次垃圾收集发生之前。无论当前内存是否足够，都会回收掉只被弱引用关联的对象。WeakRefence
- 虚引用：最弱的引用，不管是否有虚引用存在，完全不会对对象生存时间构成影响，也无法通过虚引用来取得一个对象实例。唯一目的是希望能够在这个对象被垃圾回收器之前收到系统通知。PhantomReference

相关参考：[Java 如何有效地避免OOM：善于利用软引用和弱引用](https://www.cnblogs.com/dolphin0520/p/3784171.html)

#### 4. finalize方法：

在对象被JVM回收之前，有一个低优先级的线程去执行。只有覆盖了finalize方法，才会执行，且只会执行一次。

#### 5. 回收方法区：

永久代的垃圾收集主要回收两部分内容：废弃常量和无用的类。

## 垃圾收集算法

垃圾回收算法（概述）：

1. 标记-清除算法：时间效率低、空间碎片
2. 复制算法：时间效率较高、空间利用率低。分配担保。适合于每次回收时大量对象死去只有少量存活的情况（年轻代）。
3. 标记-整理算法。适合对象存活率高的情况（老年代）。
4. 分代收集算法：就是上面几种用在堆的不同的分代区域中。年轻代：复制算法；老年代：标记-清除算法、标记-整理算法。

下面详述。

#### 1. 标记-清除算法（Mark-Sweep）

首先标记出所有需要回收的对象，使用可达性分析算法判断一个对象是否为可回收，在标记完成后统一回收所有被标记的对象。示例：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170713133044525-1749754709.png)

说明：1、效率问题，标记和清除两个阶段的效率都不高。2、空间问题，标记清除后会产生大量不连续的内存碎片，以后需要给大对象分配内存时，会提前触发一次垃圾回收动作。

#### 2. 复制算法

将内存分为两等块，每次使用其中一块。当这一块内存用完后，就将还存活的对象复制到另外一个块上，然后再把已使用过的内存空间一次清理掉。示例：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170713133054775-540568584.png)

说明：适合于每次回收时大量对象死去只有少量存活的情况，如年轻代中。1、无内存碎片问题，实现简单，时间效率高。2、空间利用率低，可用内存缩小为原来的一半。

将内存分为两个半区，将区A中的存活对象全部复制到B区的连续空间，然后清理A中所有空间。

缺点：内存实际空间减半。在对象存活率较高时需要进行较多的复制操作。

实际应用：将堆内存分为新生代和老年代。由于新生代中的对象98%都是可回收的，故将新生代又划分为Eden空间和两块较小的Survivor空间，默认Eden:Survivor（单个）=8:1

这样每次垃圾收集时，将Eden和S1中的存活对象复制到S2，然后清空Eden和S1区。

为了防止S2中空间不足以存储Eden和S1的所有剩余存活对象，提供老年代作为保障（Handle Promotion：分配担保）。

#### 3. 标记-整理算法（将存活对象移动到一起）

标记过程与标记 - 清除算法一样，但之后让所有存活的对象移向一端，然后直接清理掉边界以外的内存。示例：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170713133101400-240030919.png)

说明：适合对象存活率高的情况，如老年代中。无需考虑内存碎片问题。

#### 4. 分代收集

- 新生代：存活率低，使用复制算法
- 老年代：存活率高，使用“标记-整理”或“标记-清除”算法

根据对象存活周期将堆分为新生代和老年代，然后根据各年代特点选择适当的回收算法。

1. 新生代基本上对象都是朝生暮死的，生存时间很短暂，因此可采用复制算法，只需要复制少量的对象就可以完成收集。
2. 老年代中的对象存活率高，也没有额外的空间进行分配担保，因此必须使用标记 - 整理或者标记 - 清除算法进行回收。

## 垃圾收集器

### 概述：

HotSpot目前共7种垃圾收集器，如下（图中存在连线表示可以搭配使用）：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170713173455790-1147244880.png)

总结：

（1）

| **收集器**                        | **适用的代** | **采用的收集算法** | **适用的模式** | **串行、并行、并发**                     | **目标**       |
| --------------------------------- | ------------ | ------------------ | -------------- | ---------------------------------------- | -------------- |
| Serial                            | Young        | 复制               | Client         | 用户线程停止，收集线程串行               | GC时停顿时间少 |
| ParNew（Parallel New Generation） | Young        | 复制               | Server         | 用户线程停止，收集线程并行               | GC时停顿时间少 |
| Parallel Scavenge                 | Young        | 复制               |                | 用户线程停止、收集线程并行               | 吞吐量尽可能大 |
| CMS                               | Old          | 标记-清除          |                | 用户线程和收集线程均在执行，前者不用停止 | GC时停顿时间少 |
| Serial Old（MSC）                 | Old          | 标记-整理          | Client、Server | 用户线程停止，收集线程串行               | GC时停顿时间少 |
| Parallel Old                      | Old          | 标记-整理          |                | 用户线程停止，收集线程并行               | 吞吐量尽可能大 |
| G1                                | Young、Old   | 标记-整理          | Server         | 用户线程和收集线程均在执行，前者不用停止 | GC时停顿时间少 |

（2）

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170713173413150-700459069.png)

**注：**

1. 串行/并行是说垃圾收集线程是否能并行执行（此时用户线程得停止），并发则指垃圾收集时用户线程可以并发执行不用停下。
2. Parallel Scavenge和Parallel Old收集器以提高吞吐量为目标，其他收集器以减少停顿时间（单次GC所用时间）为目标。（吞吐量=运行用户代码时间 /（运行用户代码时间 + 垃圾回收时间）。停顿时间越短越适合与用户交互的程序以提高响应速度和用户体验；而高吞吐量可高效利用CPU以尽快完成程序运算任务，适合后台运算而不需要太多交互的任务。GC停顿时间缩短通常以牺牲吞吐量和新生代空间为代价换来。）
3. **虚拟机运行在Client 模式下时默认用 Serial + Serial Old 收集器组合，两个分别用于年轻代、老年代。**
4. **虚拟机运行在Server模式下时默认用 Parallel Scavenge + Serial Old收 集器组合，两个分别用于年轻代、老年代。**
5. 如果机器只有一个核的话，采用并行回收器可能得不偿失，因为多个回收线程会争抢CPU资源，反而造成更大的消耗，这时，就最好采用串行回收器。

 

下面详述： 

### **3.3.1、Serial收集器**

采用复制算法。

Serial收集器为单线程收集器，在进行垃圾收集时，必须要暂停其他所有的工作线程，直到它收集结束。运行过程如下图所示：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170713201524025-856073738.png)

说明：

1、需要STW（Stop The World），停顿时间长。

2、简单高效，对于单个CPU环境而言，Serial收集器由于没有线程交互开销，可以获取最高的单线程收集效率。

3、虚拟机运行在Client模式下的默认年轻代收集器。

 

### **3.3.2、ParNew收集器**

采用复制算法。

ParNew是Serial的多线程版本，除了使用多线程进行垃圾收集外，其他行为与Serial完全一样，运行过程如下图所示：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170713201605759-288856566.png)

说明：虚拟机运行在Server模式下的默认新生代收集器。

 

### **3.3.3、Parallel Scavenge收集器**

采用复制算法。

Parallel Scavenge收集器的目标是达到一个可控制的吞吐量，吞吐量 = 运行用户代码时间 / (运行用户代码时间 + 垃圾收集时间)，高吞吐量可以高效率地利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

提供了两个精确控制吞吐量的参数：

-XX:MaxGCPauseMillis：设置最大GC停顿毫秒数，收集器尽可能保证每次回收所用时间不超过该值。GC停顿时间缩短通常以牺牲吞吐量和新生代空间换来。

-XX:GCTimeRatio：设置用户代码运行时间与最大垃圾回收时间的比例，相当于吞吐量的倒数。

提供了自适应调整策略，-XX:+UseAdaptiveSizePolicy，这样虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数（新生代大小、Eden与Survivor比例、晋升老年代的对象大小阈值等）以提供最合适的停顿时间或者最大的吞吐量，这种调节方式称为GC自适应调节策略。

Parallel Scavenge的运行过程与ParNew相似。

 

### **3.3.4、Serial Old收集器**

采用标记 - 整理算法。

老年代的单线程收集器，运行过程在之前的Serial收集器已经给出。不再赘述。

常作为下面介绍的CMS收集器的后备预案，在并发收集发生Concurrent Mode Failure时使用。

 

### **3.3.5、Parallel Old收集器**

采用标记-整理算法。

老年代的多线程收集器，吞吐量优先，运行过程如下图所示：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170713202540915-187784083.png)

 

### **3.3.6、CMS收集器**

采用标记-清除算法。

CMS（Conrrurent Mark Sweep）收集器是以获取最短回收停顿时间为目标的收集器。使用标记 - 清除算法，收集过程分为如下四步：

1. 初始标记，标记GCRoots能直接关联到的对象（找到GC Roots节点），时间很短。
2. 并发标记，进行GCRoots Tracing（可达性分析）过程，时间很长。
3. 重新标记，修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，时间较长。
4. 并发清除，回收内存空间，时间很长。
5. 重置线程

其中，初始标记、重新标记仍需要GC停顿（Stop The World），并发标记与并发清除两个阶段耗时最长，但是可以与用户线程并发执行。运行过程如下图所示：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170713202621806-60214271.png)

缺点：1：内存碎片；2：由于并发故需要占用CPU、内存、且无法处理浮动垃圾

1、内存碎片。由于采用的标记 - 清除算法，会产生大量的内存碎片，不利于大对象的分配，可能会提前触发一次Full GC。虚拟机提供了-XX:+UseCMSCompactAtFullCollection参数来进行碎片的合并整理过程，这样会使得停顿时间变长，虚拟机还提供了一个参数配置-XX:+CMSFullGCsBeforeCompaction，用于设置执行多少次不压缩的Full GC后，接着来一次带压缩的GC。

2、多线程并发，耗费CPU。对CPU资源非常敏感，并发阶段虽不会导致用户线程停顿但会占用一部分线程（CPU资源）从而导致应用程序变慢，总吞吐率下降。（*不过总比让用户线程停顿好...*）

3、因用户线程并发执行，故需要预留一部分老年代空间提供并发收集时程序运行使用，从而需要额外占用内存空间。预留内存无法满足程序需要时就会出现一次Concurrent Mode Failure，此时启动后备预案即Serial Old来收集老年代垃圾。提供了参数-XX:CMSInitialtingOccupancyFraction来设置内存占用阈值，达到阈值CMS就进行垃圾回收。

4、并发收集，无法处理浮动垃圾。因为在并发清理阶段用户线程还在运行，自然就会产生新的垃圾，而在此次收集中无法收集他们，只能留到下次收集，这部分垃圾为浮动垃圾

 

### **3.3.7、G1收集器**

采用标记-整理算法。

G1（Garbage First）收集器具有如下特点：

1. 并行和并发。使用多个CPU来缩短Stop The World停顿时间，与用户线程并发执行。
2. 分代收集。独立管理整个堆，但是能够采用不同的方式去处理新创建对象和已经存活了一段时间、熬过多次GC的旧对象，以获取更好的收集效果。
3. 空间整合。基于标记 - 整理算法，无内存碎片产生，收集后能提供规整的可用内存。
4. 可预测的停顿。能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。

　　使用G1收集器时，Java堆会被划分为多个大小相等的独立区域（Region），虽然还保留新生代和老年代的概念，但两者已经不是物理隔离了，都是一部分Region（不需要连续）的集合。G1收集器中，Region之间的对象引用以及其他收集器的新生代和老年代之间的对象引用，虚拟机都使用Remembered Set来避免全堆扫描的。每个Region对应一个Remembered Set,虚拟机发现程序在对Reference类型的数据进行写操作时，会产生一个Write Barrier暂时中断写操作，检查Reference引用的对象是否处于不同的Region之中（在分代的例子中就是检查老年代的对象是否引用了新生代的对象），如果是，则通过CardTable把相关引用信息记录到被引用对象所属的Region的Remembered Set之中，当进行内存回收时，在GC根节点的枚举范围中加入Remembered Set即可保证不对全堆扫描也不会遗漏。

如果不计算维护Remembered Set的操作，G1收集器的运作可以分为如下几步：

1. 初始并发，标记GCRoots能直接关联到的对象；修改TAMS（Next Top At Mark Start）,使得下一阶段程序并发时，能够在可用的Region中创建新对象，需停顿线程，耗时很短。
2. 并发标记，从GCRoots开始进行可达性分析，与用户程序并发执行，耗时很长。
3. 最终标记，修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，变动的记录将被记录在Remembered Set Logs中，此阶段会把其整合到Remembered Set中，需要停顿线程，与用户程序并行执行，耗时较短。
4. 筛选回收，对各个Region的回收价值和成本进行排序，根据用户期望的GC时间进行回收，与用户程序并发执行，时间用户可控。

**G1收集器运行过程的前3阶段与CMS的类似**，具体示意图如下：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170713202635790-579277699.png) 

## 垃圾收集器组合及配置参数

以下是虚拟机的一些非稳定运行参数，用来配置使用的收集器组合，通过 -XX:参数 配置。

（可以使用参数-XX:+PrintCommandLineFlags让JVM打印出那些已经被用户或者JVM设置过的详细的XX参数的名称和值）

| **参　　数**                   | **描　　述**                                                 |
| ------------------------------ | ------------------------------------------------------------ |
| UseSerialGC                    | **虚拟机运行在Client 模式下的默认值**，打开此开关后，使用Serial + Serial Old 收集器组合进行内存回收 |
| UseParNewGC                    | 打开此开关后，使用ParNew + Serial Old 收集器组合进行内存回收 |
| UseConcMarkSweepGC             | 打开此开关后，使用ParNew + CMS + Serial Old 收集器组合进行内存回收。Serial Old 收集器将作为CMS 收集器出现Concurrent Mode Failure失败后的后备收集器使用 |
| UseParallelGC                  | **虚拟机运行在Server 模式下的默认值**，打开此开关后，使用Parallel Scavenge + Serial Old（PS MarkSweep）的收集器组合进行内存回收 |
| UseParallelOldGC               | 打开此开关后，使用Parallel Scavenge + Parallel Old 的收集器组合进行内存回收 |
| SurvivorRatio                  | 新生代中Eden 区域与Survivor 区域的容量比值， 默认为8， 代表Eden ：Survivor=8∶1 |
| PretenureSizeThreshold         | 直接晋升到老年代的对象大小，设置这个参数后，大于这个参数的对象将直接在老年代分配 |
| MaxTenuringThreshold           | 晋升到老年代的对象年龄。每个对象在坚持过一次Minor GC 之后，年龄就加1，当超过这个参数值时就进入老年代 |
| UseAdaptiveSizePolicy          | 动态调整Java堆中各个区域的大小以及进入老年代的年龄           |
| HandlePromotionFailure         | 是否允许分配担保失败，即老年代的剩余空间不足以应付新生代的整个Eden 和Survivor 区的所有对象都存活的极端情况 |
| ParallelGCThreads              | 设置并行GC 时进行内存回收的线程数                            |
| GCTimeRatio                    | GC 时间占总时间的比率，默认值为99，即允许1% 的GC 时间。仅在使用Parallel Scavenge 收集器时生效 |
| MaxGCPauseMillis               | 设置GC 的最大停顿时间。仅在使用Parallel Scavenge 收集器时生效 |
| CMSInitiatingOccupancyFraction | 设置CMS 收集器在老年代空间被使用多少后触发垃圾收集。默认值为68%，仅在使用CMS 收集器时生效 |
| UseCMSCompactAtFullCollection  | 设置CMS 收集器在完成垃圾收集后是否要进行一次内存碎片整理。仅在使用CMS 收集器时生效 |
| CMSFullGCsBeforeCompaction     | 设置CMS 收集器在进行若干次垃圾收集后再启动一次内存碎片整理。仅在使用CMS 收集器时生效 |



## 垃圾收集GC触发的时机

Minor GC = Young Gc 
Major GC = Full Gc 
https://blog.csdn.net/yhyr_ycy/article/details/52566105
https://blog.csdn.net/qq_33808060/article/details/60478794
要准确理解Java的垃圾回收机制，就要从：“什么时候”，“对什么东西”，“做了什么”三个方面来具体分析。

第一：“什么时候”即就是GC触发的条件。GC触发的条件有两种。（1）程序调用System.gc时可以触发；（2）系统自身来决定GC触发的时机。
系统判断GC触发的依据：根据Eden区和From Space区的内存大小来决定。当内存大小不足时，则会启动GC线程并停止应用线程。

第二：“对什么东西”笼统的认为是Java对象并没有错。但是准确来讲，GC操作的对象分为：通过可达性分析法无法搜索到的对象和可以搜索到的对象。对于搜索不到的方法进行标记。

第三：“做了什么”最浅显的理解为释放对象。但是从GC的底层机制可以看出，对于可以搜索到的对象进行复制操作，对于搜索不到的对象，调用finalize()方法进行释放。

具体过程：当GC线程启动时，会通过可达性分析法把Eden区和From Space区的存活对象复制到To Space区，然后把Eden Space和From Space区的对象释放掉。当GC轮训扫描To Space区一定次数后，把依然存活的对象复制到老年代，然后释放To Space区的对象。

GC将堆里的对象分为新生代，老年代，持久代。新生代有一个Eden区，两个survivor区。新生成的对象直接放在Eden区，Eden区满了就放进survivor1，当survivor1满了就会出发一次Minor GC：将存活的对象放入survivor2，然后清空Eden和survivor1，再将survivor区的交换，保证survivor2为空。当survivor2不足以存放Eden和survivor1的存活对象时，就会放入老年区。较大的对象和长期存活的对象直接进入老年区。当即将进入老年区的对象超过老年区剩余大小时，触发一次full GC。 
频率上说，Minor GC较频繁，full GC不频繁。 
持久代会存放一些静态值和方法。

Minor GC ，Full GC 触发条件
Minor GC触发条件：当Eden区满时，触发Minor GC。
Full GC触发条件：
（1）调用System.gc时，系统建议执行Full GC，但是不必然执行
（2）老年代空间不足
（3）方法去空间不足
（4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存
（5）由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小



### 何时执行GC（GC停顿）

（何时执行 即 考虑GC线程与用户线程间的协调）

JVM采用一组称为**OopMap**的数据结构，在特定代码位置记录下哪些变量是引用，这样GC时利用该结构得知哪些是GC Roots从而避免逐个检查变量。

GC停顿：每条指令都可能使引用关系发生变化，但若为每条指令生成OopMap空间成本高，因此让在特定位置生成OopMap，程序执行到这些位置时才会停顿进行GC。此特定位置包括两种：

**安全点**（Safe Point），面向执行着的线程，线程响应GC中断请求到安全点时暂停，以让垃圾收集器进行GC。安全点太少会导致GC等待时间长，太多会增大运行时负荷；其选取以“具有让程序长时间执行的特征”为标准，如方法调用、循环跳转、异常跳转等指令会产生安全点。GC时让线程中断有抢先式（让所有线程中断，若有线程不在安全点则恢复之以让之执行到安全点后停下）和主动式（在安全点设置标志，线程执行到安全点时检查标志看是否要中断挂起）两种。

**安全区域**（Safe Region），面向未在执行（如Sleep或Blocked状态）的线程，未在执行的线程（没被分配CPU时间）不会响应线程中断。安全区域是指一段代码片段之中，引用关系不会发生变化，在这个区域的任何地方开始GC都是安全的，安全区域可以看成是对安全点的扩展。当线程执行到安全区域中的代码时，首先标识自己已进入安全区域，则在这段时间里JVM发起GC时就不用管标识自己为安全区域状态的线程；在线程要离开安全区域时，它要检查系统是否已经完成了根节点枚举（或者整个GC过程），若完成，线程继续执行，否则，它必须等待直到收到可以安全离开安全区域的信号。



## GC类型

MinorGC与MajorGC（FullGC）的区别：

1. 新生代 GC（Minor GC）：指发生在新生代的垃圾收集动作，在Eden剩余空间不足以分配新对象时触发。因为 Java 对象大多都具备朝生夕灭的特性，所以 Minor GC 非常频繁，一般回收速度也比较快。
2. 老年代 GC（Major GC / Full GC）：指发生在老年代的 GC，在老年代剩余空间不足以容纳新对象时触发。出现了 Major GC，经常会伴随至少一次的 Minor GC（但非绝对的，在 ParallelScavenge 收集器的收集策略里 就有直接进行 Major GC 的策略选择过程） 。MajorGC 的速度一般会比 Minor GC 慢 10 倍以上。



## 理解GC日志

GC日志格式与所用JVM版本及收集器有关，但有一定的共性。这里以JDK1.8.0_45为例。

打印GC日志的参数：

```
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
```

典型GC日志示例如下：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
0.080: [GC (System.gc()) [PSYoungGen: 1147K->616K(9216K)] 1147K->624K(19456K), 0.0006699 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
0.081: [Full GC (System.gc()) [PSYoungGen: 616K->0K(9216K)] [ParOldGen: 8K->508K(10240K)] 624K->508K(19456K), [Metaspace: 2555K->2555K(1056768K)], 0.0034445 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 PSYoungGen      total 9216K, used 410K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 5% used [0x00000000ff600000,0x00000000ff666800,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
  to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
 ParOldGen       total 10240K, used 508K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 4% used [0x00000000fec00000,0x00000000fec7f1c0,0x00000000ff600000)
 Metaspace       used 2562K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 275K, capacity 386K, committed 512K, reserved 1048576K
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

由PSYoungGen和ParOldGen得知JVM的垃圾收集器组合为Parallel Scavenge(新生代) + Parallel Old(老年代)。

1. 最前面的数字0.080、0.081表示GC发生的时间，为JVM启动以来经过的秒数。
2. [GC (System.gc())]与[Full GC (System.gc())]，表示此次垃圾收集的停顿类型，而不是用来区分新生代GC和老年代GC的，如果有Full，则表示此次GC发生了*Stop The World*。
3. PSYoungGen: 1147K->616K(9216K)，表示，“新生代区域：该内存区域GC前已使用容量 -> 该内存区域GC后已使用容量（该内存区域总容量）”。区域名称与所用收集器名称有关：若为Serial收集器则名为DefNew，Default New Generation；若为ParNew收集器则名为ParNew，Parallel New Generation；若为Parallel Scavenge收集器则名为PSYoungGenn。老年代和元空间与此类似。
4. 1147K->624K(19456K)，, 0.0006699 secs，表示，“GC前Java堆已使用的容量 -> GC后Java堆已使用的容量（Java堆总容量），GC所用秒数”。
5. [Times: user=0.00 sys=0.00, real=0.00 secs]，表示GC的更具体的时间，user代表用户态消耗的CPU时间，sys代表内核态消耗的CPU时间，real代表操作从开始到结束所经过的墙钟时间。CPU时间与墙钟时间的区别是，墙钟时间包括各种非运算的等待耗时，如等待磁盘IO，等待线程阻塞，CPU时间则不包含这些耗时。当系统有多CPU或者多核时，多线程操作会叠加这些CPU时间，所以读者看到user或者sys时间超过real时间也是很正常的。

 

4.3、Java程序的总运行时间 = 运行用户代码时间 + 非用户代码时间（即时编译时间 + 类加载验证时间 + 垃圾回收时间）。其中，非用户代码时间里的加载验证和即时编译一般集中在程序运行开始的一阶段，而垃圾回收则是伴随着程序运行的整个过程。

JDK提供的Visual VM工具里Visual GC中展示的即包括该三部分非用户代码时间（Compile Time、Class Loader Time、GC Time）；

前面说的吞吐量即这里的 运行用户代码时间/（运行用户代码时间 + 垃圾回收时间）。

此外：

类验证可以通过 -XVerify:none 禁用；

即时编译可以通过 -Xint 禁用，从而纯以解释执行的方式执行代码。



## 内存分配与回收策略

#### 1. 优先在Eden区分配（如果启动本地线程分配缓冲TLAB-Thread Local Allocation Buffer，则优先在TLAB）

如果Eden区满，则触发一次Minor GC(也称Young GC)

-XX:+PrintGCDetails

- 在JVM发生垃圾收集时打印内存回收日志
- 在进程退出时输出当前各区域的内存分配情况

在JDK8中，PermGen（永久代）被Metaspace（元空间）取代了。

#### 2. 大对象直接进入老年代

-XX:PretenureSizeThreshold：直接进入老年代的对象大小

#### 3. 长期存活的对象将进入老年代

-XX:MaxTenuringThreshold：设置对象在新生代中能存活的最大年龄，默认15

-XX:+PrintTenuringDistribution：打印老年代内的各年龄对象内存分配情况

#### 4. 动态对象年龄判定

若Survivor中相同年龄的所有对象大小总和超过Survivor的一半，则年龄大于或等于该年龄的对象就可以直接进入老年代。

#### 5. 空间分配担保

- （1）YGC发生前先检查“老年代最大可用的连续空间”是否大于新生代所有对象总空间。如果是，则触发YGC,无风险；如果否，进入（2）
- （2）再次判断“老年代最大可用的连续空间”是否大于历次平均晋升到老年代的对象大小总和。如果否，则直接执行FullGC；如果是，进入（3）
- （3）执行YGC，有风险。
- （4）如果3中的YGC失败，则再执行FullGC。

### 五、垃圾回收和GC实战

#### 1. YGC测试代码：

```
/**
 * @author zni.feng
 */
import java.lang.management.ManagementFactory;

public class YGCTest {
    /**
     * VM参数： -verbose:gc -XX:+UseSerialGC  -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8
     */
    private static final int _1MB= 1024 * 1024;
    
    public static void main(String[] args) {
        System.out.println(ManagementFactory.getRuntimeMXBean().getInputArguments());//打印JVM参数
        byte[] allocation1, allocation2, allocation3, allocation4;
        allocation1 = new byte[2*_1MB];
        allocation2 = new byte[2*_1MB];
        allocation3 = new byte[2*_1MB];
        allocation4 = new byte[4*_1MB]; //出现一次YGC
        System.out.println("exit");
    }

}
```

测试结果：

```
[-verbose:gc, -XX:+UseSerialGC, -Xms20M, -Xmx20M, -Xmn10M, -XX:+PrintGCDetails, -XX:SurvivorRatio=8, -Dfile.encoding=UTF-8]
[GC (Allocation Failure) [DefNew: 6815K->282K(9216K), 0.0077121 secs] 6815K->6426K(19456K), 0.0077785 secs] [Times: user=0.01 sys=0.01, real=0.01 secs] 
exit
Heap
 def new generation   total 9216K, used 4620K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  eden space 8192K,  52% used [0x00000007bec00000, 0x00000007bf03c8d8, 0x00000007bf400000)
  from space 1024K,  27% used [0x00000007bf500000, 0x00000007bf546800, 0x00000007bf600000)
  to   space 1024K,   0% used [0x00000007bf400000, 0x00000007bf400000, 0x00000007bf500000)
 tenured generation   total 10240K, used 6144K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
   the space 10240K,  60% used [0x00000007bf600000, 0x00000007bfc00030, 0x00000007bfc00200, 0x00000007c0000000)
 Metaspace       used 2695K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 294K, capacity 386K, committed 512K, reserved 1048576K
```

测试结果分析：

从结果可以看到，发生了一次YGC，GC后新生代（DefNew：Default New Generaion）从6815K降到了282K，即基本上全部回收，而整个堆内存的大小并没有显著减小（从6815K降到了6426K），这是因为allocation1、allocation2、allocation3对象仍存在（强引用），并没有被真正回收，只是从新生代进入了老年代（Survivor区不足以容纳6M的对象）。并且，从程序结束时的各区域内存分配情况可以看到，老年代占用了约6M（tenured generation total 10240K,used 6144K）；新生代Eden区占用了52%，约4M（total 8192K，即8M），是用来存放allocation4的对象。

#### 2. 大对象直接进入老年代

```
/**
 * @author zni.feng
 */
public class PretenureSizeThresholdTest {
    /**
     * VM参数: -verbose:gc -XX:+UseSerialGC  -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8
     * -XX:PretenureSizeThreshold=3145728
     * (3145728=3*1024*1024=3MB)
     */
    
    private static final int _1MB=1024*1024;
    public static void main(String[] args) {
        byte[] allocation;
        allocation = new byte[4*_1MB];
        
    }

}
```

测试结果：

```
Heap
 def new generation   total 9216K, used 835K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  eden space 8192K,  10% used [0x00000007bec00000, 0x00000007becd0f90, 0x00000007bf400000)
  from space 1024K,   0% used [0x00000007bf400000, 0x00000007bf400000, 0x00000007bf500000)
  to   space 1024K,   0% used [0x00000007bf500000, 0x00000007bf500000, 0x00000007bf600000)
 tenured generation   total 10240K, used 4096K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
   the space 10240K,  40% used [0x00000007bf600000, 0x00000007bfa00010, 0x00000007bfa00200, 0x00000007c0000000)
 Metaspace       used 2621K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 286K, capacity 386K, committed 512K, reserved 1048576K
```

测试结果分析：可以看到，allocation对象直接进入了老年代。



copy from : 

https://www.cnblogs.com/znicy/p/6918767.html



### GC区域Eden Survivor(from,to), Old Gen和Perm Gen

VM区域总体分两类，heap区和非heap区。 
heap区又分为： 
- Eden Space（伊甸园）、 
- Survivor Space(幸存者区)、 
- Old Gen（老年代）。

非heap区又分： 
- Code Cache(代码缓存区)； 
- Perm Gen（永久代）； 
- Jvm Stack(java虚拟机栈)； 
- Local Method Statck(本地方法栈)；

下面我们对每一个内存区域做详细介绍。 
**Eden Space**字面意思是伊甸园，对象被创建的时候首先放到这个区域，进行垃圾回收后，不能被回收的对象被放入到空的survivor区域。

**Survivor Space**幸存者区，用于保存在eden space内存区域中经过垃圾回收后没有被回收的对象。Survivor有两个，分别为To Survivor、 From Survivor，这个两个区域的空间大小是一样的。执行垃圾回收的时候Eden区域不能被回收的对象被放入到空的survivor（也就是To Survivor，同时Eden区域的内存会在垃圾回收的过程中全部释放），另一个survivor（即From Survivor）里不能被回收的对象也会被放入这个survivor（即To Survivor），然后To Survivor 和 From Survivor的标记会互换，始终保证一个survivor是空的。

Eden Space和Survivor Space都属于新生代，新生代中执行的垃圾回收被称之为Minor GC（因为是对新生代进行垃圾回收，所以又被称为Young GC），每一次Young GC后留下来的对象age加1。

注：GC为Garbage Collection，垃圾回收。

**Old Gen**老年代，用于存放新生代中经过多次垃圾回收仍然存活的对象，也有可能是新生代分配不了内存的大对象会直接进入老年代。经过多次垃圾回收都没有被回收的对象，这些对象的年代已经足够old了，就会放入到老年代。

当老年代被放满的之后，虚拟机会进行垃圾回收，称之为Major GC。由于Major GC除并发GC外均需对整个堆进行扫描和回收，因此又称为Full GC。

heap区即堆内存，整个堆大小=年轻代大小 + 老年代大小。堆内存默认为物理内存的1/64(<1GB)；默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制，可以通过MinHeapFreeRatio参数进行调整；默认空余堆内存大于70%时，JVM会减少堆直到-Xms的最小限制，可以通过MaxHeapFreeRatio参数进行调整。

下面我们来认识下非堆内存（非heap区） 
**Code Cache**代码缓存区，它主要用于存放JIT所编译的代码。CodeCache代码缓冲区的大小在client模式下默认最大是32m，在server模式下默认是48m，这个值也是可以设置的，它所对应的JVM参数为ReservedCodeCacheSize 和 InitialCodeCacheSize，可以通过如下的方式来为Java程序设置。

```
-XX:ReservedCodeCacheSize=128m
```

CodeCache缓存区是可能被充满的，当CodeCache满时，后台会收到CodeCache is full的警告信息，如下所示： 
“CompilerThread0” java.lang.OutOfMemoryError: requested 2854248 bytes for Chunk::new. Out of swap space?

注：JIT编译器是在程序运行期间，将Java字节码编译成平台相关的二进制代码。正因为此编译行为发生在程序运行期间，所以该编译器被称为Just-In-Time编译器。

**Perm Gen**全称是Permanent Generation space，是指内存的永久保存区域，因而称之为永久代。这个内存区域用于存放Class和Meta的信息，Class在被 Load的时候被放入这个区域。因为Perm里存储的东西永远不会被JVM垃圾回收的，所以如果你的应用程序LOAD很多CLASS的话，就很可能出现PermGen space错误。默认大小为物理内存的1/64。


https://blog.csdn.net/shiyong1949/article/details/52585256/



## Java最大线程数
https://www.cnblogs.com/princessd8251/articles/3914434.html
JVM中可以生成的最大数量由JVM的堆内存大小、Thread的Stack内存大小、系统最大可创建的线程数量（Java线程的实现是基于底层系统的线程机制来实现的，Windows下_beginthreadex，Linux下pthread_create）三个方面影响。具体数量可以根据Java进程可以访问的最大内存（32位系统上一般2G）、堆内存、Thread的Stack内存来估算。
ThreadStackSize      JVMMemory                    能创建的线程数
默认的325K             -Xms1024m -Xmx1024m    i = 2655
默认的325K               -Xms1224m -Xmx1224m    i = 2072
默认的325K             -Xms1324m -Xmx1324m    i = 1753
默认的325K             -Xms1424m -Xmx1424m    i = 1435
-Xss1024k             -Xms1424m -Xmx1424m    i = 452 
增大堆内存（-Xms，-Xmx）会减少可创建的线程数量，增大线程栈内存（-Xss，32位系统中此参数值最小为60K）也会减少可创建的线程数量


## JVM参数调优
https://blog.csdn.net/rickyit/article/details/53895060
1.  **堆大小设置** JVM 中最大堆大小有三方面限制：相关操作系统的数据模型（32-bt还是64-bit）限制；系统的可用虚拟内存限制；系统的可用物理内存限制。32位系统下，一般限制在1.5G~2G；64为操作系统对内存无限制。我在Windows Server 2003 系统，3.5G物理内存，JDK5.0下测试，最大可设置为1478m。
    **典型设置：**
    *   java **-Xmx3550m -Xms3550m -Xmn2g -Xss128k
        - Xmx3550m** ：设置JVM最大可用内存为3550M。
          **-Xms3550m** ：设置JVM促使内存为3550m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。
          **-Xmn2g** ：设置年轻代大小为2G。**整个堆大小=年轻代大小 + 年老代大小 + 持久代大小** 。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。
          **-Xss128k** ：设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。
    *   java -Xmx3550m -Xms3550m -Xss128k **-XX:NewRatio=4 -XX:SurvivorRatio=4 -XX:MaxPermSize=16m -XX:MaxTenuringThreshold=0
        -XX:NewRatio=4** :设置年轻代（包括Eden和两个Survivor区）与年老代的比值（除去持久代）。设置为4，则年轻代与年老代所占比值为1：4，年轻代占整个堆栈的1/5
        **-XX:SurvivorRatio=4** ：设置年轻代中Eden区与Survivor区的大小比值。设置为4，则两个Survivor区与一个Eden区的比值为2:4，一个Survivor区占整个年轻代的1/6
        **-XX:MaxPermSize=16m** :设置持久代大小为16m。
        **-XX:MaxTenuringThreshold=0** ：设置垃圾最大年龄。**如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代** 。对于年老代比较多的应用，可以提高效率。**如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间** ，增加在年轻代即被回收的概论。
2.  **回收器选择** JVM给了三种选择：**串行收集器、并行收集器、并发收集器** ，但是串行收集器只适用于小数据量的情况，所以这里的选择主要针对并行收集器和并发收集器。默认情况下，JDK5.0以前都是使用串行收集器，如果想使用其他收集器需要在启动时加入相应参数。JDK5.0以后，JVM会根据当前[系统配置](http://java.sun.com/j2se/1.5.0/docs/guide/vm/server-class.html) 进行判断。
    1.  **吞吐量优先** 的并行收集器
        如上文所述，并行收集器主要以到达一定的吞吐量为目标，适用于科学技术和后台处理等。
        **典型配置** ：
        *   java -Xmx3800m -Xms3800m -Xmn2g -Xss128k **-XX:+UseParallelGC -XX:ParallelGCThreads=20
            -XX:+UseParallelGC** ：选择垃圾收集器为并行收集器。 **此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。
            -XX:ParallelGCThreads=20** ：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。
        *   java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20 **-XX:+UseParallelOldGC
            -XX:+UseParallelOldGC** ：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。
        *   java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC **-XX:MaxGCPauseMillis=100
            -XX:MaxGCPauseMillis=100 :** 设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。
        *   java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100 **-XX:+UseAdaptiveSizePolicy
            -XX:+UseAdaptiveSizePolicy** ：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。
    2.  **响应时间优先** 的并发收集器
        如上文所述，并发收集器主要是保证系统的响应时间，减少垃圾收集时的停顿时间。适用于应用服务器、电信领域等。
        **典型配置** ：
        *   java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:ParallelGCThreads=20 **-XX:+UseConcMarkSweepGC -XX:+UseParNewGC
            -XX:+UseConcMarkSweepGC** ：设置年老代为并发收集。测试中配置这个以后，-XX:NewRatio=4的配置失效了，原因不明。所以，此时年轻代大小最好用-Xmn设置。
            **-XX:+UseParNewGC** :设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值。
        *   java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseConcMarkSweepGC **-XX:CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection
            -XX:CMSFullGCsBeforeCompaction** ：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。
            **-XX:+UseCMSCompactAtFullCollection** ：打开对年老代的压缩。可能会影响性能，但是可以消除碎片

## 堆栈溢出和内存泄露
- 永久方法区溢出
  因此持久带溢出有可能是运行时常量池溢出，也有可能是方法区中保存的class对象没有被及时回收掉或者class信息占用的内存超过了我们配置。
  当持久带溢出的时候抛出java.lang.OutOfMemoryError: PermGen space
```
List<String> list = new ArrayList<String>();    
                while(true){    
                        list.add(UUID.randomUUID().toString().intern());    
                }    
        }

作者：GhostStories
链接：https://www.jianshu.com/p/cd705f88cf2a
```

- 堆溢出
```
static class OOMError{} 
public static void main(String[] args) { 
List<OOMError> list = new ArrayList<OOMError>(); 
while (true) { list.add(new OOMError()); } 
}
```
如何解决发生在Java堆内存的OutOfMemoryError异常呢？

首先我们要分清楚产生OutOfMemoryError异常的原因是内存泄露还是内存溢出，如果内存中的对象确实都必须存活着而不像上面那样不断地创建对象实例却不使用该对象，则是内存溢出，而像上面代码中的情况则是内存泄露。

如果是内存泄露，我们可以通过一些内存查看工具来查看泄露对象到GC Roots的引用链，找到泄露对象是通过怎样的路径与GC Roots相关联并导致GC无法自动回收这些泄露对象，掌握了这些信息，我们就能比较准确地定位出泄露代码的位置。

如果不是内存泄露，也就是说内存中的对象确实都还必须存活，那么应该检查虚拟机的堆参数，看看是否还可以将机器物理内存调大，同时在代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况。

链接：http://www.imooc.com/article/18073


- 线程栈溢出
  栈（JVM Stack）存放主要是栈帧( 局部变量表, 操作数栈 , 动态链接 , 方法出口信息 )的地方。注意区分栈和栈帧：栈里包含栈帧。
  与线程栈相关的内存异常有两个：
  a）、StackOverflowError(方法调用层次太深，内存不够新建栈帧)
  b）、OutOfMemoryError（线程太多，内存不够新建线程）
```
	public void stackOverFlowMethod(){
		stackOverFlowMethod();
	}

```

http://www.imooc.com/article/18073
https://blog.csdn.net/hu1991die/article/details/43052281

OOM你遇到过哪些情况

- java.lang.OutOfMemoryError: Java heap space ------>java堆内存溢出，此种情况最常见，一般由于内存泄露或者堆的大小设置不当引起。
- java.lang.OutOfMemoryError: PermGen space ------>java永久代溢出，即方法区溢出了，一般出现于大量Class或者jsp页面，或者采用cglib等反射机制的情况，因为上述情况会产生大量的Class信息存储于方法区。
- java.lang.StackOverflowError ------> 不会抛OOM error，但也是比较常见的Java内存溢出。JAVA虚拟机栈溢出，一般是由于程序中存在死循环或者深度递归调用造成的，栈大小设置太小也会出现此种溢出。可以通过虚拟机参数**-Xss**来设置栈的大小。



