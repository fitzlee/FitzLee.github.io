---
title: Jvm 内存结构
urlname: jvm_art_mem_analysis
#date: 2016-10-09
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 03JVM&ART
tags:
    - java
    - jvm
    - mem
    - oom
---

## JVM虚拟机内存结构-运行数据区域

线程私有：栈区，本地方法栈，pc指针
线程共有：方法区，堆区

1. 程序计数器

当前线程所执行的字节码的行号指示器。

2. Java虚拟机栈

线程私有，与线程具有相同生命周期。用于存储**局部变量表、操作数栈、动态链表、方法出口等信息**。

局部变量表存放内容：

- 基本数据类型（boolean、byte、char、short、int、float、long、double）
- 对象引用（区别于符号引用，符号引用存放在常量池）
- returnAddress类型（指向一条字节码指令的地址）

64位长度的long和double类型数据占用2个局部变量空间（slot），其余占用1个slot。

两种异常：

- StackOverflowError：线程请求的栈深度>虚拟机允许的深度
- OutOfMemoryError: 动态扩展时无法申请到足够内存

3. 本地方法栈（Native Method Stack）

与虚拟机栈类似，区别是Native Method Stack服务于Native方法，而虚拟机栈服务于Java方法。

4. Java堆（Java Heap）

所有线程共享，存放**对象实例、数组**。

垃圾收集器管理的主要区域，也称“GC堆（Garbage Collected Heap）”

包含新生代（Eden空间、From Survivor空间、To Survivor空间）、老生代。

可划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer，TLAB）。

物理上可以不连续，逻辑上连续。

可扩展：-Xmx和-Xms控制。-Xmx最大堆内存大小，-Xms初始堆内存大小。

当堆中没有可用内存完成实例分配，并且也无法再扩展时——OutOfMemoryError

5. 方法区（别名Non-Heap）

也是所有线程共享，用于存储**已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据**。

也称“永久代（Permanent Generation）”，但本质上并不等价。

永久代有-XX:MaxPermSize的上限。

6. 运行时常量池（Runtime Constant Pool）

属于方法区的一部分。

7. 直接内存（Direct Memory）

JDK1.4新加入的NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O方式，它可使用Native函数库直接分配**堆外内存**。不受Java堆大小（-Xmx）限制，从而可能造成各个内存区域总和大于物理内存限制而造成动态扩展时出现OutOfMemoryError。

小结：

- 1、2、3三种内存区域是各个线程私有的
- 4、5是所有线程共有的
- 6是5的一部分
- 7不是虚拟机运行时数据区的一部分，属于虚拟机的内存区域外的其他物理内存

<!-- more -->

### Java内存结构

![image.png](https://upload-images.jianshu.io/upload_images/11010834-349018968e31c9d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.PC寄存器/程序计数器：

　　严格来说是一个数据结构，用于保存当前正在执行的程序的内存地址，由于Java是支持多线程执行的，所以程序执行的轨迹不可能一直都是线性执行。当有多个线程交叉执行时，被中断的线程的程序当前执行到哪条内存地址必然要保存下来，以便用于被中断的线程恢复执行时再按照被中断时的指令地址继续执行下去。为了线程切换后能恢复到正确的执行位置，每个线程都需要有一个独立的程序计数器，各个线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存,这在某种程度上有点类似于“ThreadLocal”，是线程安全的。

2.Java栈 Java Stack：
![image.png](https://upload-images.jianshu.io/upload_images/11010834-b2af65bb985fd1ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　Java栈总是与线程关联在一起的，每当创建一个线程，JVM就会为该线程创建对应的Java栈，在这个Java栈中又会包含多个栈帧(Stack Frame)，这些栈帧是与每个方法关联起来的，每运行一个方法就创建一个栈帧，每个栈帧会含有一些局部变量、操作栈和方法返回值等信息。每当一个方法执行完成时，该栈帧就会弹出栈帧的元素作为这个方法的返回值，并且清除这个栈帧，Java栈的栈顶的栈帧就是当前正在执行的活动栈，也就是当前正在执行的方法，PC寄存器也会指向该地址。只有这个活动的栈帧的本地变量可以被操作栈使用，当在这个栈帧中调用另外一个方法时，与之对应的一个新的栈帧被创建，这个新创建的栈帧被放到Java栈的栈顶，变为当前的活动栈。同样现在只有这个栈的本地变量才能被使用，当这个栈帧中所有指令都完成时，这个栈帧被移除Java栈，刚才的那个栈帧变为活动栈帧，前面栈帧的返回值变为这个栈帧的操作栈的一个操作数。

　　由于Java栈是与线程对应起来的，Java栈数据不是线程共有的，所以不需要关心其数据一致性，也不会存在同步锁的问题。

　　在Java虚拟机规范中，对这个区域规定了两种异常状况：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常；如果虚拟机可以动态扩展，如果扩展时无法申请到足够的内存，就会抛出OutOfMemoryError异常。在Hot Spot虚拟机中，可以使用-Xss参数来设置栈的大小。栈的大小直接决定了函数调用的可达深度。

3.堆 Heap:

　　堆是JVM所管理的内存中国最大的一块，是被所有Java线程锁共享的，不是线程安全的，在JVM启动时创建。堆是存储Java对象的地方，这一点Java虚拟机规范中描述是：所有的对象实例以及数组都要在堆上分配。Java堆是GC管理的主要区域，从内存回收的角度来看，由于现在GC基本都采用分代收集算法，所以Java堆还可以细分为：新生代和老年代；新生代再细致一点有Eden空间、From Survivor空间、To Survivor空间等。

4.方法区Method Area:

　　方法区存放了要加载的类的信息（名称、修饰符等）、类中的静态常量、类中定义为final类型的常量、类中的Field信息、类中的方法信息，当在程序中通过Class对象的getName.isInterface等方法来获取信息时，这些数据都来源于方法区。方法区是被Java线程锁共享的，不像Java堆中其他部分一样会频繁被GC回收，它存储的信息相对比较稳定，在一定条件下会被GC，当方法区要使用的内存超过其允许的大小时，会抛出OutOfMemory的错误信息。方法区也是堆中的一部分，就是我们通常所说的Java堆中的永久区 Permanet Generation，大小可以通过参数来设置,可以通过-XX:PermSize指定初始值，-XX:MaxPermSize指定最大值。

5.常量池Constant Pool:

　　常量池本身是方法区中的一个数据结构。常量池中存储了如字符串、final变量值、类名和方法名常量。常量池在编译期间就被确定，并保存在已编译的.class文件中。一般分为两类：字面量和应用量。字面量就是字符串、final变量等。类名和方法名属于引用量。引用量最常见的是在调用方法的时候，根据方法名找到方法的引用，并以此定为到函数体进行函数代码的执行。引用量包含：类和接口的权限定名、字段的名称和描述符，方法的名称和描述符。

6.本地方法栈Native Method Stack:

　　本地方法栈和Java栈所发挥的作用非常相似，区别不过是Java栈为JVM执行Java方法服务，而本地方法栈为JVM执行Native方法服务。本地方法栈也会抛出StackOverflowError和OutOfMemoryError异常。


![image](http://upload-images.jianshu.io/upload_images/11010834-1088bcf587b5dec1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image](http://upload-images.jianshu.io/upload_images/11010834-1a36f5b70b8e1ee2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



参考：https://www.zhihu.com/question/29833675



### 7个区域内存溢出

- 程序计数器：看做当前线程所执行的字节码行号指示器。是线程私有的内存，且唯一一块不报OutOfMemoryError异常的内存区域。

- Java虚拟机栈：用于描述java方法的内存模型：每个方法被执行时都会同时创建一个栈帧用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程就对应着一个栈帧在虚拟机中从入栈到出栈的过程。如果线程请求的栈深度大于虚拟机所允许的深度就报StackOverflowError, 如果虚拟 机栈可以动态扩展，当拓展时无法申请到足够的内存会抛出OutOfMemoryError. 是线程私有的。

- 本地方法栈：与虚拟机栈相似，不同的在于它是为虚拟机使用到Native方法服务的。会抛出StackOverflowError和OutOfMemoryError。是线程私有的。

- Java堆: 是所有线程共享的一块内存，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。如果堆上没有内存完成实例的分配就会报OutOfMemoryError.

- 方法区（永久代）：用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。当方法区无法满足内存分配需求时，会抛出OutOfMemoryError。是共享内存。

- 运行时常量池：用于存放编译器生成的各种字面量和符号引用，是方法区的一部分。无法申请内存时抛出OutOfMemoryError。

- 直接内存：不是虚拟机运行时数据的一部分，也不是java虚拟机规范中定义的区域，是计算机直接的内存空间。这部分也被频繁使用，如JAVA NIO的引入基于通道和缓存区的I/O使用native函数直接分配堆外内存。如果内存不足会报OutOfMemoryError。

  

## JMM模型

### JMM三性质

　　**原子性（Atomicity）**：一个操作不能被打断，要么全部执行完毕，要么不执行。在这点上有点类似于事务操作，要么全部执行成功，要么回退到执行该操作之前的状态。

　　基本类型数据的访问大都是原子操作，long 和double类型的变量是64位，但是在32位JVM中，32位的JVM会将64位数据的读写操作分为2次32位的读写操作来进行，这就导致了long、double类型的变量在32位虚拟机中是非原子操作，数据有可能会被破坏，也就意味着多个线程在并发访问的时候是线程非安全的。

　**可见性**：一个线程对共享变量做了修改之后，其他的线程立即能够看到（感知到）该变量这种修改（变化）。

　　Java内存模型是通过将在工作内存中的变量修改后的值同步到主内存，在读取变量前从主内存刷新最新值到工作内存中，这种依赖主内存的方式来实现可见性的。

无论是普通变量还是volatile变量都是如此，区别在于：volatile的特殊规则保证了volatile变量值修改后的新值立刻同步到主内存，每次使用volatile变量前立即从主内存中刷新，因此volatile保证了多线程之间的操作变量的可见性，而普通变量则不能保证这一点。

　　除了volatile关键字能实现可见性之外，还有synchronized,Lock，final也是可以的。

　　使用synchronized关键字，在同步方法/同步块开始时（Monitor Enter）,使用共享变量时会从主内存中刷新变量值到工作内存中（即从主内存中读取最新值到线程私有的工作内存中），在同步方法/同步块结束时(Monitor Exit),会将工作内存中的变量值同步到主内存中去（即将线程私有的工作内存中的值写入到主内存进行同步）。

　　使用Lock接口的最常用的实现ReentrantLock(重入锁)来实现可见性：当我们在方法的开始位置执行lock.lock()方法，这和synchronized开始位置（Monitor Enter）有相同的语义，即使用共享变量时会从主内存中刷新变量值到工作内存中（即从主内存中读取最新值到线程私有的工作内存中），在方法的最后finally块里执行lock.unlock()方法，和synchronized结束位置（Monitor Exit）有相同的语义,即会将工作内存中的变量值同步到主内存中去（即将线程私有的工作内存中的值写入到主内存进行同步）。

　　final关键字的可见性是指：被final修饰的变量，在构造函数数一旦初始化完成，并且在构造函数中并没有把“this”的引用传递出去（“this”引用逃逸是很危险的，其他的线程很可能通过该引用访问到只“初始化一半”的对象），那么其他线程就可以看到final变量的值。

　　**有序性**：对于一个线程的代码而言，我们总是以为代码的执行是从前往后的，依次执行的。这么说不能说完全不对，在单线程程序里，确实会这样执行；但是在多线程并发时，程序的执行就有可能出现乱序。用一句话可以总结为：在本线程内观察，操作都是有序的；如果在一个线程中观察另外一个线程，所有的操作都是无序的。前半句是指“线程内表现为串行语义（WithIn Thread As-if-Serial Semantics）”,后半句是指“指令重排”现象和“工作内存和主内存同步延迟”现象。

Java提供了两个关键字volatile和synchronized来保证多线程之间操作的有序性,volatile关键字本身通过加入内存屏障来禁止指令的重排序，而synchronized关键字通过一个变量在同一时间只允许有一个线程对其进行加锁的规则来实现，

在单线程程序中，不会发生“指令重排”和“工作内存和主内存同步延迟”现象，只在多线程程序中出现。

### happens-before原则

```
1、程序次序规则：在一个单独的线程中，按照程序代码的执行流顺序，（时间上）先执行的操作happen—before（时间上）后执行的操作。

2、管理锁定规则：一个unlock操作happen—before后面（时间上的先后顺序，下同）对同一个锁的lock操作。

3、volatile变量规则：对一个volatile变量的写操作happen—before后面对该变量的读操作。

4、线程启动规则：Thread对象的start（）方法happen—before此线程的每一个动作。

5、线程终止规则：线程的所有操作都happen—before对此线程的终止检测，可以通过Thread.join（）方法结束、Thread.isAlive（）的返回值等手段检测到线程已经终止执行。

6、线程中断规则：对线程interrupt（）方法的调用happen—before发生于被中断线程的代码检测到中断时事件的发生。

7、对象终结规则：一个对象的初始化完成（构造函数执行结束）happen—before它的finalize（）方法的开始。

8、传递性：如果操作A happen—before操作B，操作B happen—before操作C，那么可以得出A happen—before操作C。
```



原则分析：

https://blog.csdn.net/ns_code/article/details/17348313
https://www.cnblogs.com/lewis0077/p/5143268.html



## JVM内存分配

总的来说，JVM管理的内存包括堆内存和非堆内存。堆就是Java代码可及的内存，是留给开发人员使用的；非堆就是JVM留给自己用的，所以方法区、JVM内部处理或优化所需的内存(如JIT编译后的代码缓存)、每个类结构(如运行时常数池、字段和方法数据)以及方法和构造方法的代码都在非堆内存中。

因此这里所说的内存分配是指堆内存的分配，即我们程序中生成的对象的分配。**以下所述针对的是HotSpot虚拟机**。

### 1、Java堆结构

以HotSpot为例，如下图：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201701/568153-20170105140405722-1371693441.png)

#### 1.1、年轻代

年轻代（或称新生代，Young、New）通常用来放新生成的对象（但不绝对，如可以通过-XX:PretenureSizeThreadshold配置将大对象直接分配在老年代）。年轻代的目标就是尽可能快速地收集掉那些生命周期短的对象。

年轻代分3个区，1个Eden区，2个Survivor区（from 和 to），但每次只使用Eden和一个Survivor区，另一个Survivor区空着。空Survivor区用来放MinorGC时从Eden和在使用的Survivor区中复制来的活着的对象。

针对年轻代的GC为Minor GC或称Young GC，在Eden剩余空间不足以分配新对象时触发

#### 1.2、老年代

老年代（Old、Tenure）通常用来存放从年轻代复制过来的对象（也可直接在老年代分配对象）。老年代中存放的是一些生命周期较长的对象。

针对老年代的GC为Major GC或称Full GC，在老年代剩余空间不足以容纳新对象时触发。Major GC经常会伴随至少一次的Minor GC（但非绝对，如下面所说的HandlePromotionFailure策略，有可能直接进行Major GC；Parallel Scavenge收集器提供了直接进行Major GC的策略选择），Major GC一般比Minor GC慢10倍以上。

 

**永久代**

通常还可以听到**永久代（PermGen space）**的概念，其实它并不在Java堆。有时永久代又被称为元空间（Metaspace）或方法区，比较乱。它们的区别是：方法区为JVM的规范，永久代、元空间分别是方法区在HotSpot中的一种实现，对于其他类型的虚拟机实现，如 JRockit（BEA）、J9（IBM），其并没有这些概念。

### 2、堆内存分配过程　　

对象的内存分配，绝大部分都是在堆上分配（少数经过JIT编译后被拆散为标量类型并间接在栈上分配），这里介绍在堆上的分配。

生成对象（为对象分配内存）的过程如下：

1. 首先看Eden剩余空间是否足够分配该对象，若够则直接在Eden分配；
2. 否则进行MinorGC：将Eden和在使用的Survivor区中活着的对象复制到另一个Survivor区，并回收Eden和使用着的Survivor区。然后把对象分配到Eden，以后另一个Survivor成为使用的Survivor区；
3. 若另一个Survivor区不能完全容纳复制过来的对象，则能放下的放入该Survivor，把放不下的放到老年代（即进行分配担保）；
4. 若老年代剩余空间不够了则进行Full GC，
5. 若Full GC后仍不够则抛出OOM异常。

具体可以分为：

- 分配过程（优先在Eden分配）
- 分配担保（垃圾回收时Survivor放不下的存活对象移到老年代）
- 提前移动的配置（大对象、对象年龄、动态年龄）

以下示例所用JDK为1.8.0_45。

#### 2.1、对象优先在Eden区分配

对象优先分配在Eden区，如果Eden区没有足够的空间时，虚拟机执行一次Minor GC。

示例代码：

![点击全屏显示](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

GC日志：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[GC (Allocation Failure) [DefNew: 7291K->508K(9216K), 0.0035043 secs] 7291K->6652K(19456K), 0.0035385 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 4687K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  51% used [0x00000000fec00000, 0x00000000ff014930, 0x00000000ff400000)
  from space 1024K,  49% used [0x00000000ff500000, 0x00000000ff57f390, 0x00000000ff600000)
  to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
 tenured generation   total 10240K, used 6144K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  60% used [0x00000000ff600000, 0x00000000ffc00030, 0x00000000ffc00200, 0x0000000100000000)
 Metaspace       used 2559K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 274K, capacity 386K, committed 512K, reserved 1048576K
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

说明：新生代可用的空间为9M = 8M（Eden容量） + 1M（一个survivor容量），分配完allocation1、allocation2、allocation3之后，无法再分配allocation4，会发生分配失败，则需要进行一次Minor GC，survivor to区域的容量为1M，无法容纳总量为6M的三个对象，则会通过担保机制将allocation1、allocation2、allocation3转移到老年代，然后再将allocation4分配在Eden区。

#### 2.2、空间分配担保

空间分配担保（Handle Promotion，意译成**对象存储位置晋升**还差不多，分配担保...什么鬼）即让老年代空间来存放年轻代中的对象，其前提是老年代还有容纳的空间。

在发生Minor GC时，虚拟机会检查老年代连续的空闲区域是否大于新生代所有对象的总和，若成立，则说明Minor GC是安全的，否则，虚拟机需要查看HandlePromotionFailure的值，看是否允许担保失败，若允许，则虚拟机继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，若大于，将尝试进行一次Minor GC；若小于或者HandlePromotionFailure设置不运行冒险，那么此时将改成一次Full GC。

以上是JDK Update 24之前的策略，之后的策略改变了，只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行Minor GC，否则将进行Full GC。

冒险是指经过一次Minor GC后有大量对象存活，而新生代的survivor区很小，放不下这些大量存活的对象，所以需要老年代进行分配担保，把survivor区无法容纳的对象直接进入老年代。

该策略的意义之一在于减少不必要的MinorGC。具体的流程图如下：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201701/568153-20170105160932753-379862642.png)![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201701/568153-20170105152551456-892847674.png)

示例代码：

![点击全屏显示](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

GC日志（没在老JDK上跑，可能与期望的有点出入）：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[GC (Allocation Failure) [DefNew: 7291K->508K(9216K), 0.0042361 secs] 7291K->4604K(19456K), 0.0042750 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 6811K->508K(9216K), 0.0007200 secs] 10907K->4604K(19456K), 0.0007436 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 2638K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  26% used [0x00000000fec00000, 0x00000000fee14930, 0x00000000ff400000)
  from space 1024K,  49% used [0x00000000ff400000, 0x00000000ff47f0e0, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 4096K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  40% used [0x00000000ff600000, 0x00000000ffa00020, 0x00000000ffa00200, 0x0000000100000000)
 Metaspace       used 2560K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 274K, capacity 386K, committed 512K, reserved 1048576K
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

说明：

发生了两Minor次GC，第一次发生在给allocation4分配内存空间时，由于老年代的连续可用空间大于存活的对象总和，所以allocation2、allocation3将会进入老年代，allocation1的空间将被回收，allocation4分配在新生代；第二次发生在给allocation7分配内存空间时，此次GC将allocation4、allocation5、allocation6所占的内存全部回收。最后，allocation2、allocation3在老年代，allocation7在新生代。 

#### 2.3、大对象直接进入老年代

需要大量连续内存空间的Java对象称为大对象，大对象的出现会导致提前触发垃圾收集以获取更大的连续的空间来进行大对象的分配。虚拟机提供了-XX:PretenureSizeThreadshold（仅对Serial和ParNew收集器有效）参数来设置大对象的阈值，超过阈值的对象直接分配到老年代。这样做的目的是避免在Eden区和两个 Survivor区之间发生大量的内存拷贝（新生代采用复制算法收集内存）。

示例代码：

![点击全屏显示](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

GC日志：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Heap
 def new generation   total 9216K, used 1311K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  16% used [0x00000000fec00000, 0x00000000fed47f80, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 4096K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  40% used [0x00000000ff600000, 0x00000000ffa00010, 0x00000000ffa00200, 0x0000000100000000)
 Metaspace       used 2559K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 274K, capacity 386K, committed 512K, reserved 1048576K
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

说明：4MB对象超过PretenureSizeThreshold阈值（3MB），直接分配在了老年代。

#### 2.4、长期存活的对象进入老年代

每个对象有一个对象年龄计数器，与对象头标记字中的GC分代年龄对应。对象出生在Eden区、经过一次Minor GC后仍然存活，并能够被Survivor容纳，设置年龄为1，对象在Survivor区每次经过一次Minor GC，年龄就加1，当年龄达到一个阈值（默认15），下次触发Minor GC时就被移到老年代（即配置的是最多被在Survivor间移动几次，达到阈值后下一次GC时不移动而是直接晋升了）。虚拟机提供了-XX:MaxTenuringThreshold来进行设置。

示例代码：

![点击全屏显示](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

GC日志：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[GC (Allocation Failure) [DefNew Desired survivor size 524288 bytes, new threshold 1 (max 1)- age   1:     783256 bytes,     783256 total: 5499K->764K(9216K), 0.0024536 secs] 5499K->4860K(19456K), 0.0024878 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew Desired survivor size 524288 bytes, new threshold 1 (max 1): 4860K->0K(9216K), 0.0006581 secs] 8956K->4860K(19456K), 0.0006864 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 4178K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  51% used [0x00000000fec00000, 0x00000000ff014930, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 4860K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  47% used [0x00000000ff600000, 0x00000000ffabf120, 0x00000000ffabf200, 0x0000000100000000)
 Metaspace       used 2560K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 274K, capacity 386K, committed 512K, reserved 1048576K
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

说明：发生了两次Minor GC，第一次是在给allocation3进行分配的时候会出现一次Minor GC，此时survivor区域不能容纳allocation2，但是可以容纳allocation1，所以allocation1将会进入survivor区域并且年龄为1，达到了阈值，将在下一次GC时晋升到老年代，而allocation2则会通过担保机制进入老年代。第二次发生GC是在第二次给allocation3分配空间时，这时，allocation1的晋升到老年代，此次GC也可以清理出原来allocation3占据的4MB空间，将allocation3分配在Eden区。所以，最后的结果是allocation1、allocation2在老年代，allocation3在Eden区。

#### 2.5、动态判断对象年龄

对象的年龄到达了MaxTenuringThreshold可以进入老年代，同时，如果在survivor区中相同年龄所有对象大小的总和大于survivor区的一半，年龄大于等于该年龄的对象就可以直接进入老年代。无需等到MaxTenuringThreshold中要求的年龄。

示例代码：

![点击全屏显示](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

GC日志：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[GC (Allocation Failure) [DefNew Desired survivor size 524288 bytes, new threshold 1 (max 15)- age   1:    1045416 bytes,    1045416 total: 5755K->1020K(9216K), 0.0027553 secs] 5755K->5116K(19456K), 0.0028017 secs] [Times: user=0.00 sys=0.02, real=0.00 secs] 
[GC (Allocation Failure) [DefNew Desired survivor size 524288 bytes, new threshold 15 (max 15): 5116K->0K(9216K), 0.0009216 secs] 9212K->5116K(19456K), 0.0009399 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 4178K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  51% used [0x00000000fec00000, 0x00000000ff014930, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 5116K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  49% used [0x00000000ff600000, 0x00000000ffaff130, 0x00000000ffaff200, 0x0000000100000000)
 Metaspace       used 2559K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 274K, capacity 386K, committed 512K, reserved 1048576K
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

说明：发生了两次Minor GC，第一次发生在给allocation4分配内存时，此时allocation1、allocation2将会进入survivor区，而allocation3通过担保机制将会进入老年代。第二次发生在第二次给allocation4分配内存时，此时，survivor区的allocation1、allocation2达到了survivor区容量的一半，将会进入老年代，此次GC可以清理出allocation4原来的4MB空间，并将allocation4分配在Eden区。最终，allocation1、allocation2、allocation3在老年代，allocation4在Eden区。、

### 3、堆内存分配参数设置

见 [HotSpot JVM各区域内存分配参数设置-MarchOn](http://www.cnblogs.com/z-sm/p/6253335.html#autoid-2-4-0)

- -Xmx3550m：等价于-XX:MaxHeapSize，设置JVM最大堆内存为3550M。
- -Xms3550m：等价于-XX:InitialHeapSize，设置JVM初始堆内存为3550M。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。
- -Xmn2g：设置年轻代大小为2G。在整个堆内存大小确定的情况下，增大年轻代将会减小年老代，反之亦然。此值关系到JVM垃圾回收，对系统性能影响较大，官方推荐配置为整个堆大小的3/8。
- -XX:NewSize=1024m：设置年轻代初始值为1024M。
- -XX:MaxNewSize=1024m：设置年轻代最大值为1024M。
- -XX:SurvivorRatio=4：设置年轻代中Eden区与一个Survivor区的比值。表示Edgen为一个Survivor的4倍，即1个Survivor区占整个年轻代大小的1/6。默认为8。
- -XX:NewRatio=4：设置老年代与年轻代（包括1个Eden和2个Survivor区）的比值。表示老年代是年轻代的4倍。
- -XX:PretenureSizeThreadshold=1024：设置让大于此阈值的对象直接分配在老年代（只对Serial、ParNew收集器有效），单位为字节
- -XX:MaxTenuringThreshold=7：表示一个对象如果在Survivor区移动了7次那下次MinorGC时就进入老年代。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代，对于需要大量常驻内存的应用，这样做可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代存活时间，增加对象在年轻代被垃圾回收的概率，减少Full GC的频率，可以在某种程度上提高服务稳定性



copy from ：http://www.cnblogs.com/z-sm/p/6252245.html



## JVM对象创建和内存布局

### 1、对象创建的过程

　　1、类加载、解析、初始化：虚拟机遇到new时先检查此指令的参数是否能在常量池中找到类的符号引用，并检查符号引用代表的类是否被加载、解析、初始化，若没有则先进行类加载。

　　2、对象内存分配：类加载检查通过后，虚拟机为新生对象分配内存，对象所需内存大小在类加载完成后便可完全确定。分配内存的任务等同于从堆中分出一块确定大小的内存。（具体分配策略见：[JVM内存分配策略-MarchOn](http://www.cnblogs.com/z-sm/p/6252245.html)）

　　　　根据内存是否规整（即用的放一边，空闲的放另一边，是否如此与所使用的垃圾收集器是否带有压缩整理Compact功能有关），分配方式分为指针碰撞（Serial、ParNe等收集器）和空闲列表（CMS收集器等）两种

　　　　并发控制：可能多个对象同时从堆中分配内存因此需要同步，两种解决方案：虚拟机用CAS配上失败重试保证原子操作；把内存分配动作按线程划分在不同空间中进行，即每个线程预先分配一块线程本地缓冲区TLAB，各线程在各自TLAB为各自对象分配内存。

　　3、对象的初始化：对象头和对象实例数据的初始化

### 2、对象的内存布局

**布局：**

1. 对象头：标记字（32位虚拟机4B，64位虚拟机8B） + 类型指针（32位虚拟机4B，64位虚拟机8B）+ [数组长（对于数组对象才需要此部分信息）]
2. 实例数据
3. 对齐填充：对于64位虚拟机来说，对象大小必须是8B的整数倍，不够的话需要占位填充

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170710131157259-132221985.png)

- 对象头用于存储对象的元数据信息：
  - Mark Word 部分数据的长度在32位和64位虚拟机（未开启压缩指针）中分别为32bit和64bit，存储对象自身的运行时数据如哈希值等。Mark Word一般被设计为非固定的数据结构，以便存储更多的数据信息和复用自己的存储空间。
  - 类型指针 指向它的类元数据的指针，用于判断对象属于哪个类的实例。
- 实例数据存储的是真正有效数据，如各种字段内容，各字段的分配策略为longs/doubles、ints、shorts/chars、bytes/boolean、oops(ordinary object pointers)，相同宽度的字段总是被分配到一起，便于之后取数据。父类定义的变量会出现在子类定义的变量的前面。
- 对齐填充部分仅仅起到占位符的作用，并非必须。

**示例（以HashMap<Long,Long>为例）：**

其只有Key和Value是有效数据，共2*8B=16B，包装成Long对象后分别具有了8B标记字和8B的类型指针，共24B*2=48B；两个对象组成Map.Entry后多了16B对象头、一个8B的next字段、4B的int类型的hash字段，还必须添加4B的空白填充。共32B；最后还有对HashMap中对此Entry的8B的引用。所以空间利用率为 16B / (48B+32B+8B) ≈ 18%

 

**32位虚拟机 VS 64位虚拟机：**

由于指针膨胀和各种数据类型对齐补白的原因，运行于64位系统上的Java应用需要消耗更多的内存（通常比32位的增加10%~30%的内存开销） ；此外，64位虚拟机的运行速度比32位的大约有15%左右的性能差距。

不过，64位虚拟机也有它的优势：首先能管理更多的内存，32位最多4GB，实际上还受OS允许进程最大内存的限制（Windows下2GB）；其次，随着硬件技术的发展，计算机终究会完全过渡到64位，虚拟机也将过渡到64位。

 

### 3、对象的访问定位

对象的访问定位也取决于具体的虚拟机实现。当我们在堆上创建一个对象实例后，就要通过虚拟机栈中的reference类型数据来操作堆上的对象。现在主流的访问方式有两种（HotSpot虚拟机采用的是第二种）：

1. 使用句柄访问对象。即reference中存储的是对象句柄的地址，而句柄中包含了对象实例数据与类型数据的具体地址信息，相当于二级指针。
2. 直接指针访问对象。即reference中存储的就是对象地址，相当于一级指针**。**

　　两种方式有各自的优缺点。当垃圾回收移动对象时，对于方式一而言，reference中存储的地址是稳定的地址，不需要修改，仅需要修改对象句柄的地址；而对于方式二，则需要修改reference中存储的地址。从访问效率上看，方式二优于方式一，因为方式二只进行了一次指针定位，节省了时间开销，而这也是HotSpot采用的实现方式。下图是句柄访问与指针访问的示意图。

![点击全屏显示](https://images2015.cnblogs.com/blog/616953/201602/616953-20160226155344349-887482013.png)    ![点击全屏显示](https://images2015.cnblogs.com/blog/616953/201605/616953-20160518141733169-1486000631.png)