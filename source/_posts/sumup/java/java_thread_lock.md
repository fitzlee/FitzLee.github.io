---
title: java 线程原理及锁同步分析
urlname: java_jdk_base_thread_lock
#date: 2016-03-06
top: true
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 00Java&JDK
tags:
    - java
    - jdk
    - thread
    - lock
    - synchronize
---

## Java内存模型

内存模型：在特定的操作协议下，对特定的内存或高速缓存进行读写访问的过程抽象。 

Java虚拟机规范定义了**Java内存模型（Java Memory Model，JMM）**来屏蔽各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果（C/C++等则直接使用物理机和OS的内存模型，使得程序须针对特定平台编写），它在多线程的情况下尤其重要。

JMM的主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节。这里的变量是指共享变量，存在竞争问题的变量，如实例字段、静态字段、数组对象元素等，不包括线程私有的局部变量、方法参数等，因为私有变量不存在竞争问题。可以JMM认为包括内存划分、变量访问操作与规则两部分。

### 内存划分

物理机中的内存模型：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170717154306331-490912881.png)

Java内存划分如下所示（可与上述物理机中的内存模型作类比）：

 ![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170717160643128-1712699420.png)

分为主内存和工作内存，每个线程都有自己的工作内存，它们共享主内存。

主内存（Main Memory）存储所有共享变量的值。

工作内存（Working Memory）存储该线程使用到的共享变量在主内存的的值的副本拷贝。

- 线程对共享变量的所有读写操作都在自己的工作内存中进行，不能直接读写主内存中的变量（volatile变量也不例外，虽然它看起来如同直接访问主内存一般）。
- 不同线程间也无法直接访问对方工作内存中的变量，线程间变量值的传递必须通过主内存完成。

注：这种划分与Java内存区域中堆、栈、方法区等的划分是不同层次的划分，两者基本没有关系。硬要联系的话，大致上主内存对应Java堆中对象的实例数据部分、工作内存对应栈的部分区域；从更低层次上说，主内存对应物理硬件内存、工作内存对应寄存器和高速缓存。

 <!-- more -->

### 内存间交互的操作和规则

（这里介绍的访问操作及规则完全确定了Java程序中哪些内存访问在并发下是安全的，1.3节介绍与此等效的判断原则——先行发生原则）

**1.2.1、8个原子操作和执行规则**

**8个原子操作**：JMM定义了8个原子操作来完成工作内存和主内存间的交互：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170717165108347-304679706.png)

1、lock（锁定）：作用于主内存的变量，它把一个变量标示为一条线程独占的状态。
2、unlock（解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
3、read（读取）：作用于主内存的变量，它把一个变量的值从主内存传输到工作内存中，以便随后的load动作使用。
4、load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
5、use（使用）：作用于工作内存的变量，它把工作内存中的一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作。
6、assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
7、store（存储）：作用于工作内存的变量，它把工作内存中的一个变量的值传递到主内存中，以便随后的write操作使用。
8、write（写入）：作用于主内存的变量，它把store操作从工作内存中得到的变量值放入主内存的变量中。

**8个原子操作的执行规则**：JMM还规定了执行上述8中基本操作时需满足如下规则：

1、不允许read和load、store和write操作之一单独出现，即不允许一变量从主内存读取了但工作内存不接受，或从工作内存发起回写了但主内存不接受的情况。即要求read、load成对顺序出现，但不要求连续出现（中间可以插入其他指令），store、write亦然。
2、不允许一个线程丢弃它的最近的assign操作，即变量在工作内存中改变了之后必须把该变化同步回主内存。
3、不允许一个线程无原因地（没有发生过任何assign操作）把数据从线程的工作内存同步回主内存中。
4、一个新的变量只能从主内存中“诞生”，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量，换句话说就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。
5、一个变量在同一个时刻只允许一条线程对其执行lock操作，但lock操作可以被同一个条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。
6、如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行load或assign操作初始化变量的值。
7、如果一个变量实现没有被lock操作锁定，则不允许对它执行unlock操作，也不允许去unlock一个被其他线程锁定的变量。
8、对一个变量执行unlock操作之前，必须先把此变量同步回主内存（执行store和write操作）。

从上面可以看出，把变量从工作内存复制到主内存需要顺序执行read、load，从工作内存同步回主内存则需要顺序执行store、write。总结：

read、load必须成对顺序出现，但不要去连续出现。assign、store、write同之；

变量诞生和初始化：变量只能从主内存“诞生”，且须先初始化后才能使用，即在use/store前须先load/assign；

lock一个变量后会清空工作内存中该变量的值，使用前须先初始化；unlock前须将变量同步回主内存；

一个变量同一时刻只能被一线程lock，lock几次就须unlock几次；未被lock的变量不允许被执行unlock，一个线程不能去unlock其他线程lock的变量。

 

**1.2.2、volatile变量访问规则**

 volatile变量是JVM提供的最轻量级的同步机制。Java内存模型对volatile专门定义了一些特殊的访问规则，被volatile修饰的变量可以保证可见性、有序性，但不保证原子性。

**1、保证可见性**：即一个线程修改了一变量的值，其他线程立即可见该变量的新值。

原理：一个线程对变量修改后立即同步回主内存，即 assign、store、write须连续执行、其他线程对变量读取前立即从主内存刷新新值到工作内存即 read、load、use须连续执行。

**2、保证有序性**：禁止指令重排序。

原理：确保指令重排序时不会把后面指令排列到内存屏障（Memory Barrier，也称内存栅栏：在volatile变量上加lock前缀指令）之前的位置，并且不会把变量之前的指令排到内存屏障之后的位置，即在执行内存屏障这句话时，这前面的操作已全部完成。

**3、不保证原子性**

输出值不等于threadNum*countRange，说明volatile变量并没保证原子性。原因也很明了：race++不是原子操作，虽然线程拿到race值时是最新的，但执行完加一操作写回时可能race值已经被其他线程更新了，导致更新被覆盖。



**指令重排序**：

含义：只要程序的最终结果与它顺序化情况的结果相等，那么指令的执行顺序可以与代码顺序不一致。如java语言规范就是规定JVM线程内部维持顺序化语义。

类型：计算机在执行程序时，为了提高性能，编译器和处理器的常常会对指令做重排，一般分以下3种：

**编译器优化的重排**：编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。

**指令并行的重排**：现代处理器采用了指令级并行技术来将多条指令重叠执行。如果不存在数据依赖性(即后一个执行的语句无需依赖前面执行的语句的结果)，处理器可以改变语句对应的机器指令的执行顺序

**内存系统的重排**：由于处理器使用缓存和读写缓存冲区，这使得加载(load)和存储(store)操作看上去可能是在乱序执行，因为三级缓存的存在，导致内存与缓存的数据同步存在时间差。

其中编译器优化的重排属于编译期重排，指令并行的重排和内存系统的重排属于处理器重排，在多线程环境中，这些重排优化可能会导致程序出现内存可见性问题

指令重排序的意义：JVM能根据处理器特性（CPU多级缓存系统、多核处理器等）适当的对机器指令进行重排序，使机器指令能更符合CPU的执行特性，最大限度的发挥机器性能。

 

**1.2.3、long和double型变量的特殊规则（非原子性协定）**

Java内存模型要求前述8个操作具有原子性，但对于64位的数据类型long和double，在模型中特别定义了一条宽松的规定：允许虚拟机将没有被volatile修饰的64位数据的读写操作划分为两次32位的操作来进行。即未被volatile修饰时线程对其的读取不是原子操作，可能只读到“半个变量”值。虽然如此，商用虚拟机几乎都把64位数据的读写实现为原子操作，因此我们**可以忽略这个问题**。

 

### 先行发生原则

**先行发生（happens-before）原则**用来确定一个访问在并发环境下是否安全、数据是否存在竞争。**其与1.2节介绍的访问操作及规则等效**。

Java内存模型具备一些先天的“有序性”，即不需要通过任何同步手段（volatile、synchronized等）就能够得到保证的有序性，这个通常也称为happens-before原则。如果两个操作的执行次序不符合先行原则且无法从happens-before原则推导出来，那么它们就不能保证它们的有序性，虚拟机可以随意地对它们进行重排序。

这些规则包括：

1、程序次序规则（Program Order Rule）：一个线程内，按照代码顺序（虑到分支、循环等结构，准确地说应该是控制流顺序），书写在前面的操作先行发生于书写在后面的操作。
2、锁定规则（Monitor Lock Rule）：一个unLock操作先行发生于后面对同一个锁的lock操作。“后面”指时间上的先后顺序。
3、volatile变量规则（Volatile Variable Rule）：对一个变量的写操作先行发生于后面对这个变量的读操作。“后面”指时间上的先后顺序。
4、传递规则（Transitivity）：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C。
5、线程启动规则（Thread Start Rule）：Thread对象的start()方法先行发生于此线程的每个一个动作。
6、线程中断规则（Thread Interruption Rule）：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生（通过Thread.interrupted()检测）。
7、线程终止规则（Thread Termination Rule）：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行。
8、对象终结规则（Finaizer Rule）：一个对象的初始化完成（构造函数执行结束）先行发生于他的finalize()方法的开始。

注：先行发生与时间先后顺序之间没有必然联系，衡量并发安全问题必须以先行发生原则为准。如对于一个普通变量的不带同步的get、set方法，让时间上线程1、2先后调用get、set方法，线程2并不一定得到线程1设置的值。因为它们不符合先行原则也不能由之导出。

 参考：http://www.cnblogs.com/z-sm/p/7196347.html



## Java内存结构

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


## 工作内存和主存
![image.png](https://upload-images.jianshu.io/upload_images/11010834-26e89ce6bb3ce835.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

JMM与Java内存区域唯一相似点，都存在共享数据区域和私有数据区域，在JMM中主内存属于共享数据区域，从某个程度上讲应该包括了堆和方法区，而工作内存数据线程私有数据区域，从某个程度上讲则应该包括程序计数器、虚拟机栈以及本地方法栈。或许在某些地方，我们可能会看见主内存被描述为堆内存，工作内存被称为线程栈，实际上他们表达的都是同一个含义。关于JMM中的主内存和工作内存说明如下

主内存

主要存储的是Java实例对象，所有线程创建的实例对象都存放在主内存中，不管该实例对象是成员变量还是方法中的本地变量(也称局部变量)，当然也包括了共享的类信息、常量、静态变量。由于是共享数据区域，多条线程对同一个变量进行访问可能会发现线程安全问题。

工作内存

主要存储当前方法的所有本地变量信息(工作内存中存储着主内存中的变量副本拷贝)，每个线程只能访问自己的工作内存，即线程中的本地变量对其它线程是不可见的，就算是两个线程执行的是同一段代码，它们也会各自在自己的工作内存中创建属于当前线程的本地变量，当然也包括了字节码行号指示器、相关Native方法的信息。注意由于工作内存是每个线程的私有数据，线程间无法相互访问工作内存，因此存储在工作内存的数据不存在线程安全问题。

弄清楚主内存和工作内存后，接了解一下主内存与工作内存的数据存储类型以及操作方式，根据虚拟机规范，对于一个实例对象中的成员方法而言，如果方法中包含本地变量是基本数据类型（boolean,byte,short,char,int,long,float,double），将直接存储在工作内存的帧栈结构中，但倘若本地变量是引用类型，那么该变量的引用会存储在功能内存的帧栈中，而对象实例将存储在主内存(共享数据区域，堆)中。但对于实例对象的成员变量，不管它是基本数据类型或者包装类型(Integer、Double等)还是引用类型，都会被存储到堆区。至于static变量以及类本身相关信息将会存储在主内存中。需要注意的是，在主内存中的实例对象可以被多线程共享，倘若两个线程同时调用了同一个对象的同一个方法，那么两条线程会将要操作的数据拷贝一份到自己的工作内存中，执行完成操作后才刷新到主内存，简单示意图如下所示：

![image.png](https://upload-images.jianshu.io/upload_images/11010834-851feac06c421079.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

JMM的最初目的，就是为了能够支持多线程程序设计的，每个线程可以认为是和其他线程不同的CPU上运行，或者对于多处理器的机器而言，该模型需要实现的就是使得每一个线程就像运行在不同的机器、不同的CPU或者本身就不同的线程上一样，这种情况实际上在项目开发中是常见的。对于CPU本身而言，不能直接访问其他CPU的寄存器，模型必须通过某种定义规则来使得线程和线程在工作内存中进行相互调用而实现CPU本身对其他CPU、或者说线程对其他线程的内存中资源的访问，而表现这种规则的运行环境一般为运行该程序的运行宿主环境（操作系统、服务器、分布式系统等），而程序本身表现就依赖于编写该程序的语言特性，这里也就是说用Java编写的应用程序在内存管理中的实现就是遵循其部分原则，也就是前边提及到的JMM定义了Java语言针对内存的一些的相关规则。然而，虽然设计之初是为了能够更好支持多线程，但是该模型的应用和实现当然不局限于多处理器，而在JVM编译器编译Java编写的程序的时候以及运行期执行该程序的时候，对于单CPU的系统而言，这种规则也是有效的，这就是是上边提到的线程和线程之间的内存策略。JMM本身在描述过程没有提过具体的内存地址以及在实现该策略中的实现方法是由JVM的哪一个环节（编译器、处理器、缓存控制器、其他）提供的机制来实现的，甚至针对一个开发非常熟悉的程序员，也不一定能够了解它内部对于类、对象、方法以及相关内容的一些具体可见的物理结构。相反，JMM定义了一个线程与主存之间的抽象关系，其实从上边的图可以知道，每一个线程可以抽象成为一个工作内存（抽象的高速缓存和寄存器），其中存储了Java的一些值，该模型保证了Java里面的属性、方法、字段存在一定的数学特性，按照该特性，该模型存储了对应的一些内容，并且针对这些内容进行了一定的序列化以及存储排序操作，这样使得Java对象在工作内存里面被JVM顺利调用，（当然这是比较抽象的一种解释）既然如此，大多数JMM的规则在实现的时候，必须使得主存和工作内存之间的通信能够得以保证，而且不能违反内存模型本身的结构，这是语言在设计之处必须考虑到的针对内存的一种设计方法。这里需要知道的一点是，这一切的操作在Java语言里面都是依靠Java语言自身来操作的，因为Java针对开发人员而言，内存的管理在不需要手动操作的情况下本身存在内存的管理策略，这也是Java自己进行内存管理的一种优势。

其他硬件原理：https://blog.csdn.net/javazejian/article/details/72772461

　　

## JMM 原子性、可见性、有序性

JMM是围绕着在并发过程中（多线程环境下）如何处理原子性、可见性、有序性这3个特征来建立的。

总结：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170718121606740-135234070.png)

JMM就是一组规则，这组规则意在解决在并发编程可能出现的线程安全问题，并提供了内置解决方案（happen-before原则）及其外部可使用的同步手段(synchronized/volatile等)，确保了程序执行在多线程环境中的应有的原子性，可视性及其有序性。

### 原子性

含义：是指一个操作在同一时刻只能被一个线程执行，只有该线程执行完了其他线程才能执行该操作。该操作可以只有一个步骤也可以有多个步骤，后者可以称为组合操作。

JMM实现：

由JMM直接保证的原子性变量操作包括read、load、use、assign、store、write；基本数据类型的读写也是原子性的（long、double也可以当做原子性）。

由JMM的lock、unlock可实现更大范围的原子性保证，虽用户没法直接用之，但可用synchronized关键字来保证原子性。synchronized关键字编译后在同步块前后形成字节码指令monitorenter、monitorexit，这两个指令最终即利用了lock、unlock操作保证其间组合操作的原子性。

### 可见性

含义：是指当一个线程修改了共享变量的值，其他线程立即得知该修改。

JMM实现：

（volatile）变量值被一个线程修改后会立即同步回主内存、变量值被其他线程读取前立即从主内存刷新值到工作内存。即read、load、use三者连续顺序执行，assign、store、write连续顺序执行。

（synchronized）1.2.1节中原子操作执行规则8——“对一个变量执行unlock操作之前，必须先把此变量同步回主内存，即执行store、write”。

（final）final修饰的字段在构造器中一旦初始化完成，且构造器没有把“this”的引用传递出去，则其他线程可立即看到final字段的值。

### 有序性

含义：在一个线程内所有的操作都是有序的，表现为串行化语义（在一个线程观察另一个则操作不一定有序，是因为有“指令重排序”现象和“工作内存与主内存同步延迟”现象）。

JMM实现：

（volatile）禁止指令重排序

（synchronized）1.2.1节中原子操作执行规则5——“一个变量在同一个时刻只允许一条线程对其执行lock操作”，此规则决定了持有同一个锁的两个同步块只能串行进入。



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

## happens-before原则
    1、程序次序规则：在一个单独的线程中，按照程序代码的执行流顺序，（时间上）先执行的操作happen—before（时间上）后执行的操作。
    
    2、管理锁定规则：一个unlock操作happen—before后面（时间上的先后顺序，下同）对同一个锁的lock操作。
    
    3、volatile变量规则：对一个volatile变量的写操作happen—before后面对该变量的读操作。
    
    4、线程启动规则：Thread对象的start（）方法happen—before此线程的每一个动作。
    
    5、线程终止规则：线程的所有操作都happen—before对此线程的终止检测，可以通过Thread.join（）方法结束、Thread.isAlive（）的返回值等手段检测到线程已经终止执行。
    
    6、线程中断规则：对线程interrupt（）方法的调用happen—before发生于被中断线程的代码检测到中断时事件的发生。
    
    7、对象终结规则：一个对象的初始化完成（构造函数执行结束）happen—before它的finalize（）方法的开始。
    
    8、传递性：如果操作A happen—before操作B，操作B happen—before操作C，那么可以得出A happen—before操作C。

原则分析：https://blog.csdn.net/ns_code/article/details/17348313
https://www.cnblogs.com/lewis0077/p/5143268.html

## volatile原理

volatile保证线程可见性，禁止指令重排，但不保证原子性。

volatile关键字的两层语义

　　一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

　　1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

　　2）禁止进行指令重排序。

下面这段话摘自《深入理解Java虚拟机》：

　　“观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令”

　　lock前缀指令实际上相当于一个**内存屏障**（也成内存栅栏），内存屏障会提供3个功能：

　　1）它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；

　　2）它会强制将对缓存的修改操作立即写入主存；

　　3）如果是写操作，它会导致其他CPU中对应的缓存行无效。


## Mesi缓存一致性
如果一个变量在多个CPU中都存在缓存（一般在多线程编程时才会出现），那么就可能存在缓存不一致的问题。

　　为了解决缓存不一致性问题，通常来说有以下2种解决方法：

　　1）通过在总线加LOCK#锁的方式

　　2）通过缓存一致性协议

　　这2种方式都是硬件层面上提供的方式。

　　在早期的CPU当中，是通过在总线上加LOCK#锁的形式来解决缓存不一致的问题。因为CPU和其他部件进行通信都是通过总线来进行的，如果对总线加LOCK#锁的话，也就是说阻塞了其他CPU对其他部件访问（如内存），从而使得只能有一个CPU能使用这个变量的内存。比如上面例子中 如果一个线程在执行 i = i +1，如果在执行这段代码的过程中，在总线上发出了LCOK#锁的信号，那么只有等待这段代码完全执行完毕之后，其他CPU才能从变量i所在的内存读取变量，然后进行相应的操作。这样就解决了缓存不一致的问题。

　　但是上面的方式会有一个问题，由于在锁住总线期间，其他CPU无法访问内存，导致效率低下。

　　所以就出现了缓存一致性协议。最出名的就是Intel 的MESI协议，MESI协议保证了每个缓存中使用的共享变量的副本是一致的。它核心的思想是：当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。

![image.png](https://upload-images.jianshu.io/upload_images/11010834-18b9545819102efb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


以上内容参考自：
https://blog.csdn.net/javazejian/article/details/72772461#t16
https://blog.csdn.net/jjavaboy/article/details/77164474
https://www.cnblogs.com/lewis0077/p/5143268.html
https://blog.csdn.net/longfulong/article/details/78790955

## 锁的种类
http://ifeve.com/java_lock_see2/

可重入锁
可重入锁，也叫做递归锁，指的是同一线程 外层函数获得锁之后 ，内层递归函数仍然有获取该锁的代码，但不受影响。
在JAVA环境下 ReentrantLock 和synchronized 都是 可重入锁.
可重入锁最大的作用是避免死锁
```java
public class SpinLock1 {
	private AtomicReference<Thread> owner =new AtomicReference<>();
	private int count =0;
	public void lock(){
		Thread current = Thread.currentThread();
		if(current==owner.get()) {
			count++;
			return ;
		}

		while(!owner.compareAndSet(null, current)){

		}
	}
	public void unlock (){
		Thread current = Thread.currentThread();
		if(current==owner.get()){
			if(count!=0){
				count--;
			}else{
				owner.compareAndSet(current, null);
			}

		}

	}
}
```

## 公平锁/非公平锁

公平锁是指多个线程按照申请锁的顺序来获取锁。
非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。
对于Java `ReentrantLock`而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。
对于`Synchronized`而言，也是一种非公平锁。由于其并不像`ReentrantLock`是通过AQS的来实现线程调度，所以并没有任何办法使其变成公平锁。

## 可重入锁

可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。说的有点抽象，下面会有一个代码的示例。
对于Java `ReentrantLock`而言, 他的名字就可以看出是一个可重入锁，其名字是`Re entrant Lock`重新进入锁。
对于`Synchronized`而言,也是一个可重入锁。可重入锁的一个好处是可一定程度避免死锁。

```
synchronized void setA() throws Exception{
    Thread.sleep(1000);
    setB();
}

synchronized void setB() throws Exception{
    Thread.sleep(1000);
}
```

上面的代码就是一个可重入锁的一个特点，如果不是可重入锁的话，setB可能不会被当前线程执行，可能造成死锁。

## 独享锁/共享锁共享锁

独享锁是指该锁一次只能被一个线程所持有。
共享锁是指该锁可被多个线程所持有。

对于Java `ReentrantLock`而言，其是独享锁。但是对于Lock的另一个实现类`ReadWriteLock`，其读锁是共享锁，其写锁是独享锁。
读锁的共享锁可保证并发读是非常高效的，读写，写读 ，写写的过程是互斥的。
独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。
对于`Synchronized`而言，当然是独享锁。

## 互斥锁/读写锁

上面讲的独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。
互斥锁在Java中的具体实现就是`ReentrantLock`
读写锁在Java中的具体实现就是`ReadWriteLock`

## 乐观锁/悲观锁

乐观锁与悲观锁不是指具体的什么类型的锁，而是指看待并发同步的角度。
悲观锁认为对于同一个数据的并发操作，一定是会发生修改的，哪怕没有修改，也会认为修改。因此对于同一个数据的并发操作，悲观锁采取加锁的形式。悲观的认为，不加锁的并发操作一定会出问题。
乐观锁则认为对于同一个数据的并发操作，是不会发生修改的。在更新数据的时候，会采用尝试更新，不断重新的方式更新数据。乐观的认为，不加锁的并发操作是没有事情的。

从上面的描述我们可以看出，悲观锁适合写操作非常多的场景，乐观锁适合读操作非常多的场景，不加锁会带来大量的性能提升。
悲观锁在Java中的使用，就是利用各种锁。
乐观锁在Java中的使用，是无锁编程，常常采用的是CAS算法，典型的例子就是原子类，通过CAS自旋实现原子操作的更新。

## 分段锁

分段锁其实是一种锁的设计，并不是具体的一种锁，对于`ConcurrentHashMap`而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作。
我们以`ConcurrentHashMap`来说一下分段锁的含义以及设计思想，`ConcurrentHashMap`中的分段锁称为Segment，它即类似于HashMap（JDK7与JDK8中HashMap的实现）的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表；同时又是一个ReentrantLock（Segment继承了ReentrantLock)。
当需要put元素的时候，并不是对整个hashmap进行加锁，而是先通过hashcode来知道他要放在那一个分段中，然后对这个分段进行加锁，所以当多线程put的时候，只要不是放在一个分段中，就实现了真正的并行的插入。
但是，在统计size的时候，可就是获取hashmap全局信息的时候，就需要获取所有的分段锁才能统计。
分段锁的设计目的是细化锁的粒度，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作。

## 偏向锁/轻量级锁/重量级锁

这三种锁是指锁的状态，并且是针对`Synchronized`。在Java 5通过引入锁升级的机制来实现高效`Synchronized`。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。
偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。
轻量级锁是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让其他申请的线程进入阻塞，性能降低。

## 自旋锁

在Java中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU。
典型的自旋锁实现的例子，可以参考[自旋锁的实现](http://ifeve.com/java_lock_see1/)

在自旋锁中 另有三种常见的锁形式:TicketLock ，CLHlock 和MCSlock.
https://github.com/Essviv/spring/tree/master/src/main/java/com/cmcc/syw/concurrency/lock/spinlock

```java
public class SpinLock implements Operator {
    private AtomicReference<Thread> current = new AtomicReference<>();

    public static void main(String[] args) {
        SpinLock spinLock = new SpinLock();

        final int COUNT = 200;
        for (int i = 0; i < COUNT; i++) {
            new CustomThread(spinLock).start();
        }
    }

    public void lock() {
        Thread currentThread = Thread.currentThread();
        while (!current.compareAndSet(null, currentThread)) { //获取锁失败,则开始轮询,直到成功
        }
    }

    public void unlock() {
        current.compareAndSet(Thread.currentThread(), null);
    }
}
```

```java
public class TicketLock implements Operator {
    private AtomicInteger serviceNum = new AtomicInteger();
    private AtomicInteger currentNum = new AtomicInteger();
    private ThreadLocal<Integer> ticketNum = new ThreadLocal<>();

    public static void main(String[] args) {
        TicketLock ticketLock = new TicketLock();

        final int COUNT = 200;
        for (int i = 0; i < COUNT; i++) {
            new CustomThread(ticketLock).start();
        }
    }

    public void lock() {
        //每个线程在进入这个之前,先拿到唯一的一个号
        ticketNum.set(currentNum.getAndIncrement());

        //服务号每次只会让拥有当前号码的线程进入
        while (ticketNum.get() != serviceNum.get()) {

        }
    }

    public void unlock() {
        //解锁的过程类似于叫号的过程
        serviceNum.getAndIncrement();
    }

}
```

```java
public class ClhSpinLock implements Operator {
    //前驱节点
    private ThreadLocal<Node> prev = new ThreadLocal<>();

    //当前节点
    private ThreadLocal<Node> current = new ThreadLocal<>();

    //末尾节点
    private AtomicReference<Node> tail = new AtomicReference<>();

    public static void main(String[] args) {
        ClhSpinLock clhLock = new ClhSpinLock();

        final int COUNT = 50;
        for (int i = 0; i < COUNT; i++) {
            new CustomThread(clhLock).start();
        }
    }

    public void lock() {
        //创建当前的节点
        Node currentNode = new Node();
        current.set(currentNode);

        //加入队列
        Node prevNode = tail.getAndSet(currentNode);
        prev.set(prevNode);

        //等待获取锁， 如果失败，则自旋等待
        while (prevNode != null && prevNode.isLocked()) {
        }

        //help GC
        prev.set(null);
    }

    public void unlock() {
        current.get().setLocked(false);
    }

    private class Node {
        //标识当前线程是否正在获取锁或已经获取到锁
        private volatile boolean isLocked;

        public Node() {
            this.isLocked = true;
        }

        public boolean isLocked() {
            return isLocked;
        }

        public void setLocked(boolean locked) {
            isLocked = locked;
        }
    }
}
```

```java

```

3、阻塞锁
```java
/**
 * clh锁实践代码: 当获取锁失败时，使用阻塞方式
 * <p>
 * Created by sunyiwei on 2016/12/7.
 */
public class ClhBlockLock implements Operator {
    //前驱节点
    private ThreadLocal<Node> prev = new ThreadLocal<>();

    //当前节点
    private ThreadLocal<Node> current = new ThreadLocal<>();

    //末尾节点
    private AtomicReference<Node> tail = new AtomicReference<>();

    public static void main(String[] args) {
        ClhBlockLock clhLock = new ClhBlockLock();

        final int COUNT = 50;
        for (int i = 0; i < COUNT; i++) {
            new CustomThread(clhLock).start();
        }
    }

    public void lock() {
        //创建当前的节点
        Node currentNode = new Node();
        current.set(currentNode);

        //加入队列
        Node prevNode = tail.getAndSet(currentNode);
        prev.set(prevNode);
        if (prevNode != null) {
            prevNode.setNext(currentNode);
        }

        //等待获取锁， 当获取失败时，阻塞等待
        while (prevNode != null && prevNode.isLocked()) {
            LockSupport.park(this);
        }

        //help GC
        prev.set(null);
    }

    public void unlock() {
        Node currentNode = current.get();
        currentNode.setLocked(false);

        Node nextNode = currentNode.getNext();
        if (nextNode != null) {
            LockSupport.unpark(nextNode.getThread());
        }
    }

    private class Node {
        private final Thread thread;
        //标识当前线程是否正在获取锁或已经获取到锁
        private volatile boolean isLocked;
        private Node next;

        public Node() {
            this.isLocked = true;
            this.next = null;
            this.thread = Thread.currentThread();
        }

        public boolean isLocked() {
            return isLocked;
        }

        public void setLocked(boolean locked) {
            isLocked = locked;
        }

        public Node getNext() {
            return next;
        }

        public void setNext(Node next) {
            this.next = next;
        }

        public Thread getThread() {
            return thread;
        }
    }
}
```

12、对象锁
13、线程锁
14、锁粗化
16、锁消除
17、锁膨胀
18、信号量


## CAS和Unsafe操作
上边我们介绍了volatile和JMM机制，那么从下图可以看出，Java并发的关键还有一个CAS的操作。

Java的CAS会使用现代处理器上提供的高效机器级别原子指令，这些原子指令以原子方式对内存执行读-改-写操作，这是在多处理器中实现同步的关键（从本质上来说，能够支持原子性读-改-写指令的计算机器，是顺序计算图灵机的异步等价机器，因此任何现代的多处理器都会去支持某种能对内存执行原子性读-改-写操作的原子指令）。同时，volatile变量的读/写和CAS可以实现线程之间的通信。把这些特性整合在一起，就形成了整个concurrent包得以实现的基石。如果我们仔细分析concurrent包的源代码实现，会发现一个通用化的实现模式：

1. 首先，声明共享变量为volatile；
2. 然后，使用CAS的原子条件更新来实现线程之间的同步；
3. 同时，配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的通信。

AQS，非阻塞数据结构和原子变量类（java.util.concurrent.atomic包中的类），这些concurrent包中的基础类都是使用这种模式来实现的，而concurrent包中的高层类又是依赖于这些基础类来实现的。从整体来看，concurrent包的实现示意图如下：

![img](http://dl.iteye.com/upload/attachment/0083/2584/b7b2472f-6b93-3f85-9e44-29a9ff774c8e.png) 

**Unsafe**

简单讲一下这个类。Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门，JDK中有一个类Unsafe，它提供了硬件级别的**原子操作**。

这个类尽管里面的方法都是public的，但是并没有办法使用它们，JDK API文档也没有提供任何关于这个类的方法的解释。总而言之，对于Unsafe类的使用都是受限制的，只有授信的代码才能获得该类的实例，当然JDK库里面的类是可以随意使用的。

从第一行的描述可以了解到Unsafe提供了硬件级别的操作，比如说获取某个属性在内存中的位置，比如说修改对象的字段值，即使它是私有的。不过Java本身就是为了屏蔽底层的差异，对于一般的开发而言也很少会有这样的需求。

举两个例子，比方说：

```
public native long staticFieldOffset(Field paramField);
```

这个方法可以用来获取给定的paramField的内存地址偏移量，这个值对于给定的field是唯一的且是固定不变的。再比如说：

```
public native int arrayBaseOffset(Class paramClass);
public native int arrayIndexScale(Class paramClass);
```

前一个方法是用来获取数组第一个元素的偏移地址，后一个方法是用来获取数组的转换因子即数组中元素的增量地址的。最后看三个方法：

```
public native long allocateMemory(long paramLong);
public native long reallocateMemory(long paramLong1, long paramLong2);
public native void freeMemory(long paramLong);
```

分别用来分配内存，扩充内存和释放内存的。

当然这需要有一定的C/C++基础，对内存分配有一定的了解，这也是为什么我一直认为C/C++开发者转行做Java会有优势的原因。

 

**CAS**

CAS，Compare and Swap即比较并交换，设计并发算法时常用到的一种技术，java.util.concurrent包全完建立在CAS之上，没有CAS也就没有此包，可见CAS的重要性。

当前的处理器基本都支持CAS，只不过不同的厂家的实现不一样罢了。**CAS有三个操作数：内存值V、旧的预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做并返回false**。

CAS也是通过Unsafe实现的，看下Unsafe下的三个方法：

```
public final native boolean compareAndSwapObject(Object paramObject1, long paramLong, Object paramObject2, Object paramObject3);

public final native boolean compareAndSwapInt(Object paramObject, long paramLong, int paramInt1, int paramInt2);

public final native boolean compareAndSwapLong(Object paramObject, long paramLong1, long paramLong2, long paramLong3);
```



**由CAS分析AtomicInteger原理**

java.util.concurrent.atomic包下的原子操作类都是基于CAS实现的，下面拿AtomicInteger分析一下，首先是AtomicInteger类变量的定义：

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
 try {
    valueOffset = unsafe.objectFieldOffset
        (AtomicInteger.class.getDeclaredField("value"));
  } catch (Exception ex) { throw new Error(ex); }
}

private volatile int value;
```

关于这段代码中出现的几个成员属性：

1、Unsafe是CAS的核心类，前面已经讲过了

2、valueOffset表示的是变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的原值的

3、value是用volatile修饰的，这是非常关键的

下面找一个方法getAndIncrement来研究一下AtomicInteger是如何实现的，比如我们常用的addAndGet方法：

```java
public final int addAndGet(int delta) {
    for (;;) {
        int current = get();
        int next = current + delta;
        if (compareAndSet(current, next))
            return next;
    }
}
```

```java
 public final int get() {
     return value;
 }
```

这段代码如何在不加锁的情况下通过CAS实现线程安全，我们不妨考虑一下方法的执行：

1、AtomicInteger里面的value原始值为3，即主内存中AtomicInteger的value为3，根据Java内存模型，线程1和线程2各自持有一份value的副本，值为3

2、线程1运行到第三行获取到当前的value为3，线程切换

3、线程2开始运行，获取到value为3，利用CAS对比内存中的值也为3，比较成功，修改内存，此时内存中的value改变比方说是4，线程切换

4、线程1恢复运行，利用CAS比较发现自己的value为3，内存中的value为4，得到一个重要的结论-->**此时value正在被另外一个线程修改，所以我不能去修改它**

5、线程1的compareAndSet失败，循环判断，因为value是volatile修饰的，所以它具备可见性的特性，线程2对于value的改变能被线程1看到，只要线程1发现当前获取的value是4，内存中的value也是4，说明**线程2对于value的修改已经完毕并且线程1可以尝试去修改它**

6、最后说一点，比如说此时线程3也准备修改value了，没关系，因为比较-交换是一个原子操作不可被打断，线程3修改了value，线程1进行compareAndSet的时候必然返回的false，这样线程1会继续循环去获取最新的value并进行compareAndSet，直至获取的value和内存中的value一致为止

整个过程中，利用CAS机制保证了对于value的修改的线程安全性。

 

**CAS的缺点**

CAS看起来很美，但这种操作显然无法涵盖并发下的所有场景，并且CAS从语义上来说也不是完美的，存在这样一个逻辑漏洞：如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？如果在这段期间它的值曾经被改成了B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个漏洞称为CAS操作的"ABA"问题。java.util.concurrent包为了解决这个问题，提供了一个带有标记的原子引用类"AtomicStampedReference"，它可以通过控制变量值的版本来保证CAS的正确性。不过目前来说这个类比较"鸡肋"，大部分情况下ABA问题并不会影响程序并发的正确性，如果需要解决ABA问题，使用传统的互斥同步可能回避原子类更加高效。

 CAS虽然很高效的解决原子操作，但是CAS仍然存在三大问题。ABA问题，循环时间长开销大和只能保证一个共享变量的原子操作

\1.  ABA问题。因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A-2B－3A。

从Java1.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

关于ABA问题参考文档: http://blog.hesey.net/2011/09/resolve-aba-by-atomicstampedreference.html

\2. 循环时间长开销大。自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。



\3. 只能保证一个共享变量的原子操作。当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。

来源：

https://blog.csdn.net/liu88010988/article/details/50799978

https://www.cnblogs.com/xrq730/p/4976007.html



## AQS框架源码解析

Java并发包（JUC）中提供了很多并发工具，这其中，很多我们耳熟能详的并发工具，譬如ReentrangLock、Semaphore，它们的实现都用到了一个共同的基类--**AbstractQueuedSynchronizer**,简称AQS。AQS是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。当然，我们自己也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器。

　　本章我们就一起探究下这个神奇的东东，并对其实现原理进行剖析理解

### 基本实现原理

　　AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。

```
    private volatile int state;//共享变量，使用volatile修饰保证线程可见性
```

状态信息通过procted类型的**getState**，**setState**，**compareAndSetState**进行操作

AQS支持两种同步方式：

　　**1.独占式**

　　**2.共享式**

　　这样方便使用者实现不同类型的同步组件，独占式如ReentrantLock，共享式如Semaphore，CountDownLatch，组合式的如ReentrantReadWriteLock。总之，AQS为使用提供了底层支撑，如何组装实现，使用者可以自由发挥。

同步器的设计是基于**模板方法模式**的，一般的使用方式是这样：

　　**1.使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）**

　　**2.将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。**

这其实是**模板方法模式**的一个很经典的应用。

我们来看看AQS定义的这些可重写的方法：

　　　　**protected boolean tryAcquire(int arg) : 独占式获取同步状态，试着获取，成功返回true，反之为false**

　　　　**protected boolean tryRelease(int arg) ：独占式释放同步状态，等待中的其他线程此时将有机会获取到同步状态；**

　　　　**protected int tryAcquireShared(int arg) ：共享式获取同步状态，返回值大于等于0，代表获取成功；反之获取失败；**

　　　　**protected boolean tryReleaseShared(int arg) ：共享式释放同步状态，成功为true，失败为false**

　　　　**protected boolean isHeldExclusively() ： 是否在独占模式下被线程占用。**

关于AQS的使用，我们来简单总结一下：

　　**如何使用**

　　首先，我们需要去继承AbstractQueuedSynchronizer这个类，然后我们根据我们的需求去重写相应的方法，比如要实现一个独占锁，那就去重写tryAcquire，tryRelease方法，要实现共享锁，就去重写tryAcquireShared，tryReleaseShared；最后，在我们的组件中调用AQS中的模板方法就可以了，而这些模板方法是会调用到我们之前重写的那些方法的。也就是说，我们只需要很小的工作量就可以实现自己的同步组件，重写的那些方法，仅仅是一些简单的对于共享资源state的获取和释放操作，至于像是获取资源失败，线程需要阻塞之类的操作，自然是AQS帮我们完成了。

　　**设计思想**

　　对于使用者来讲，我们无需关心获取资源失败，线程排队，线程阻塞/唤醒等一系列复杂的实现，这些都在AQS中为我们处理好了。我们只需要负责好自己的那个环节就好，也就是获取/释放共享资源state的姿势T_T。很经典的模板方法设计模式的应用，AQS为我们定义好顶级逻辑的骨架，并提取出公用的线程入队列/出队列，阻塞/唤醒等一系列复杂逻辑的实现，将部分简单的可由使用者决定的操作逻辑延迟到子类中去实现即可。

### 自定义同步器

　　**同步器代码实现**

上面大概讲了一些关于AQS如何使用的理论性的东西，接下来，我们就来看下实际如何使用，直接采用JDK官方文档中的小例子来说明问题

```java
 1 package juc;
 2 
 3 import java.util.concurrent.locks.AbstractQueuedSynchronizer;
 4 
 5 /**
 6  * Created by chengxiao on 2017/3/28.
 7  */
 8 public class Mutex implements java.io.Serializable {
 9     //静态内部类，继承AQS
10     private static class Sync extends AbstractQueuedSynchronizer {
11         //是否处于占用状态
12         protected boolean isHeldExclusively() {
13             return getState() == 1;
14         }
15         //当状态为0的时候获取锁，CAS操作成功，则state状态为1，
16         public boolean tryAcquire(int acquires) {
17             if (compareAndSetState(0, 1)) {
18                 setExclusiveOwnerThread(Thread.currentThread());
19                 return true;
20             }
21             return false;
22         }
23         //释放锁，将同步状态置为0
24         protected boolean tryRelease(int releases) {
25             if (getState() == 0) throw new IllegalMonitorStateException();
26             setExclusiveOwnerThread(null);
27             setState(0);
28             return true;
29         }
30     }
31         //同步对象完成一系列复杂的操作，我们仅需指向它即可
32         private final Sync sync = new Sync();
33         //加锁操作，代理到acquire（模板方法）上就行，acquire会调用我们重写的tryAcquire方法
34         public void lock() {
35             sync.acquire(1);
36         }
37         public boolean tryLock() {
38             return sync.tryAcquire(1);
39         }
40         //释放锁，代理到release（模板方法）上就行，release会调用我们重写的tryRelease方法。
41         public void unlock() {
42             sync.release(1);
43         }
44         public boolean isLocked() {
45             return sync.isHeldExclusively();
46         }
47 }
```



### 源码分析

 　　我们先来简单描述下AQS的基本实现，前面我们提到过，AQS维护一个共享资源state，通过内置的FIFO来完成获取资源线程的排队工作。（这个内置的同步队列称为"CLH"队列）。该队列由一个一个的Node结点组成，每个Node结点维护一个prev引用和next引用，分别指向自己的前驱和后继结点。AQS维护两个指针，分别指向队列头部head和尾部tail。

![img](https://images2015.cnblogs.com/blog/1024555/201707/1024555-20170717114859238-311670115.png)

　　其实就是个**双端双向链表**。

　　当线程获取资源失败（比如tryAcquire时试图设置state状态失败），会被构造成一个结点加入CLH队列中，同时当前线程会被阻塞在队列中（通过LockSupport.park实现，其实是等待态）。当持有同步状态的线程释放同步状态时，会唤醒后继结点，然后此结点线程继续加入到对同步状态的争夺中。

　　**Node结点**

　　Node结点是AbstractQueuedSynchronizer中的一个静态内部类，我们捡Node的几个重要属性来说一下

```
 1 static final class Node {
 2         /** waitStatus值，表示线程已被取消（等待超时或者被中断）*/
 3         static final int CANCELLED =  1;
 4         /** waitStatus值，表示后继线程需要被唤醒（unpaking）*/
 5         static final int SIGNAL    = -1;
 6         /**waitStatus值，表示结点线程等待在condition上，当被signal后，会从等待队列转移到同步到队列中 */
 7         /** waitStatus value to indicate thread is waiting on condition */
 8         static final int CONDITION = -2;
 9        /** waitStatus值，表示下一次共享式同步状态会被无条件地传播下去
10         static final int PROPAGATE = -3;
11         /** 等待状态，初始为0 */
12         volatile int waitStatus;
13         /**当前结点的前驱结点 */
14         volatile Node prev;
15         /** 当前结点的后继结点 */
16         volatile Node next;
17         /** 与当前结点关联的排队中的线程 */
18         volatile Thread thread;
19         /** ...... */
20     }  
```

### 独占式

　　**获取同步状态--acquire()**

　　来看看acquire方法，lock方法一般会直接代理到acquire上

```
1  public final void acquire(int arg) {
2         if (!tryAcquire(arg) &&
3             acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
4             selfInterrupt();
5     }
```

　　我们来简单理一下代码逻辑：

　　　　**a.首先，调用使用者重写的tryAcquire方法，若返回true，意味着获取同步状态成功，后面的逻辑不再执行；若返回false，也就是获取同步状态失败，进入b步骤；**

　　　　**b.此时，获取同步状态失败，构造独占式同步结点，通过addWatiter将此结点添加到同步队列的尾部（此时可能会有多个线程结点试图加入同步队列尾部，需要以线程安全的方  式添加）；**

　　　　**c.该结点以在队列中尝试获取同步状态，若获取不到，则阻塞结点线程，直到被前驱结点唤醒或者被中断。**

　　**addWaiter**

　　　　为获取同步状态失败的线程，构造成一个Node结点，添加到同步队列尾

```
 private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);//构造结点
        //指向尾结点tail
        Node pred = tail;
        //如果尾结点不为空，CAS快速尝试在尾部添加，若CAS设置成功，返回；否则，eng。
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }
```



　　先cas快速设置，若失败，进入enq方法　　

　　将结点添加到同步队列尾部这个操作，同时可能会有多个线程尝试添加到尾部，是非线程安全的操作。

　　以上代码可以看出，使用了compareAndSetTail这个cas操作保证安全添加尾结点。

　　**enq方法**



```
 private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { //如果队列为空，创建结点，同时被head和tail引用
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {//cas设置尾结点，不成功就一直重试
                    t.next = node;
                    return t;
                }
            }
        }
    }
```



　　enq内部是个死循环，通过CAS设置尾结点，不成功就一直重试。很经典的CAS自旋的用法，我们在之前关于原子类的源码分析中也提到过。这是一种**乐观的并发策略**。

　　最后，看下acquireQueued方法

　　**acquireQueued**



```
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {//死循环
                final Node p = node.predecessor();//找到当前结点的前驱结点
                if (p == head && tryAcquire(arg)) {//如果前驱结点是头结点，才tryAcquire，其他结点是没有机会tryAcquire的。
                    setHead(node);//获取同步状态成功，将当前结点设置为头结点。
                    p.next = null; // 方便GC
                    failed = false;
                    return interrupted;
                }
                // 如果没有获取到同步状态，通过shouldParkAfterFailedAcquire判断是否应该阻塞，parkAndCheckInterrupt用来阻塞线程
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```



　　acquireQueued内部也是一个死循环，只有前驱结点是头结点的结点，也就是老二结点，才有机会去tryAcquire；若tryAcquire成功，表示获取同步状态成功，将此结点设置为头结点；若是非老二结点，或者tryAcquire失败，则进入shouldParkAfterFailedAcquire去判断判断当前线程是否应该阻塞，若可以，调用parkAndCheckInterrupt阻塞当前线程，直到被中断或者被前驱结点唤醒。若还不能休息，继续循环。

　**shouldParkAfterFailedAcquire**

```
shouldParkAfterFailedAcquire用来判断当前结点线程是否能休息
```



```
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        //获取前驱结点的wait值 
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)//若前驱结点的状态是SIGNAL，意味着当前结点可以被安全地park
            return true;
        if (ws > 0) {
        // ws>0，只有CANCEL状态ws才大于0。若前驱结点处于CANCEL状态，也就是此结点线程已经无效，从后往前遍历，找到一个非CANCEL状态的结点，将自己设置为它的后继结点
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {  
            // 若前驱结点为其他状态，将其设置为SIGNAL状态
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }    
```

若shouldParkAfterFailedAcquire返回true，也就是当前结点的前驱结点为SIGNAL状态，则意味着当前结点可以放心休息，进入parking状态了。parkAncCheckInterrupt阻塞线程并处理中断。

```
private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);//使用LockSupport使线程进入阻塞状态
        return Thread.interrupted();// 线程是否被中断过
    }
```

　　至此，关于acquire的方法源码已经分析完毕，我们来简单总结下

　　　　**a.首先tryAcquire获取同步状态，成功则直接返回；否则，进入下一环节；**

　　　　**b.线程获取同步状态失败，就构造一个结点，加入同步队列中，这个过程要保证线程安全；**

　　　　**c.加入队列中的结点线程进入自旋状态，若是老二结点（即前驱结点为头结点），才有机会尝试去获取同步状态；否则，当其前驱结点的状态为SIGNAL，线程便可安心休息，进入阻塞状态，直到被中断或者被前驱结点唤醒。**

　　**释放同步状态--release()**

　　当前线程执行完自己的逻辑之后，需要释放同步状态，来看看release方法的逻辑



```
 public final boolean release(int arg) {
        if (tryRelease(arg)) {//调用使用者重写的tryRelease方法，若成功，唤醒其后继结点，失败则返回false
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);//唤醒后继结点
            return true;
        }
        return false;
    }
```

```
　　unparkSuccessor:唤醒后继结点　
```



```
 1 private void unparkSuccessor(Node node) {
 2         //获取wait状态
 3         int ws = node.waitStatus;
 4         if (ws < 0)
 5             compareAndSetWaitStatus(node, ws, 0);// 将等待状态waitStatus设置为初始值0
 6         Node s = node.next;//后继结点
 7         if (s == null || s.waitStatus > 0) {//若后继结点为空，或状态为CANCEL（已失效），则从后尾部往前遍历找到一个处于正常阻塞状态的结点　　　　　进行唤醒
 8             s = null;
 9             for (Node t = tail; t != null && t != node; t = t.prev)
10                 if (t.waitStatus <= 0)
11                     s = t;
12         }
13         if (s != null)
14             LockSupport.unpark(s.thread);//使用LockSupprot唤醒结点对应的线程
15     }    
```

release的同步状态相对简单，需要找到头结点的后继结点进行唤醒，若后继结点为空或处于CANCEL状态，从后向前遍历找寻一个正常的结点，唤醒其对应线程。

### 共享式

　　共享式：共享式地获取同步状态。对于独占式同步组件来讲，同一时刻只有一个线程能获取到同步状态，其他线程都得去排队等待，其待重写的尝试获取同步状态的方法tryAcquire返回值为boolean，这很容易理解；对于共享式同步组件来讲，同一时刻可以有多个线程同时获取到同步状态，这也是“共享”的意义所在。其待重写的尝试获取同步状态的方法tryAcquireShared返回值为int。

```
 protected int tryAcquireShared(int arg) {
        throw new UnsupportedOperationException();
    }
```

 

　　**1.当返回值大于0时，表示获取同步状态成功，同时还有剩余同步状态可供其他线程获取；**

　　**2.当返回值等于0时，表示获取同步状态成功，但没有可用同步状态了；**

　　**3.当返回值小于0时，表示获取同步状态失败。**

　　**获取同步状态--acquireShared**　　

```
public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)//返回值小于0，获取同步状态失败，排队去；获取同步状态成功，直接返回去干自己的事儿。
            doAcquireShared(arg);
    }
```

　　**doAcquireShared**



```
 1  private void doAcquireShared(int arg) {
 2         final Node node = addWaiter(Node.SHARED);//构造一个共享结点，添加到同步队列尾部。若队列初始为空，先添加一个无意义的傀儡结点，再将新节点添加到队列尾部。
 3         boolean failed = true;//是否获取成功
 4         try {
 5             boolean interrupted = false;//线程parking过程中是否被中断过
 6             for (;;) {//死循环
 7                 final Node p = node.predecessor();//找到前驱结点
 8                 if (p == head) {//头结点持有同步状态，只有前驱是头结点，才有机会尝试获取同步状态
 9                     int r = tryAcquireShared(arg);//尝试获取同步装填
10                     if (r >= 0) {//r>=0,获取成功
11                         setHeadAndPropagate(node, r);//获取成功就将当前结点设置为头结点，若还有可用资源，传播下去，也就是继续唤醒后继结点
12                         p.next = null; // 方便GC
13                         if (interrupted)
14                             selfInterrupt();
15                         failed = false;
16                         return;
17                     }
18                 }
19                 if (shouldParkAfterFailedAcquire(p, node) &&//是否能安心进入parking状态
20                     parkAndCheckInterrupt())//阻塞线程
21                     interrupted = true;
22             }
23         } finally {
24             if (failed)
25                 cancelAcquire(node);
26         }
27     }
```

大体逻辑与独占式的acquireQueued差距不大，只不过由于是共享式，会有多个线程同时获取到线程，也可能同时释放线程，空出很多同步状态，所以当排队中的老二获取到同步状态，如果还有可用资源，会继续传播下去。

　　**setHeadAndPropagate**



```
 private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; // Record old head for check below
        setHead(node);
        if (propagate > 0 || h == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }
```



　　**释放同步状态--releaseShared**



```
 public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();//释放同步状态
            return true;
        }
        return false;
    }
```



　　**doReleaseShared**



```
 private void doReleaseShared() {
        for (;;) {//死循环，共享模式，持有同步状态的线程可能有多个，采用循环CAS保证线程安全
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;          
                    unparkSuccessor(h);//唤醒后继结点
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                
            }
            if (h == head)              
                break;
        }
    }
```

　　代码逻辑比较容易理解，需要注意的是，共享模式，释放同步状态也是多线程的，此处采用了CAS自旋来保证。

### 总结

　　关于AQS的介绍及源码分析到此为止了。

　　AQS是JUC中很多同步组件的构建基础，简单来讲，它内部实现主要是状态变量state和一个FIFO队列来完成，同步队列的头结点是当前获取到同步状态的结点，获取同步状态state失败的线程，会被构造成一个结点（或共享式或独占式）加入到同步队列尾部（采用自旋CAS来保证此操作的线程安全），随后线程会阻塞；释放时唤醒头结点的后继结点，使其加入对同步状态的争夺中。

　　AQS为我们定义好了顶层的处理实现逻辑，我们在使用AQS构建符合我们需求的同步组件时，只需重写tryAcquire，tryAcquireShared，tryRelease，tryReleaseShared几个方法，来决定同步状态的释放和获取即可，至于背后复杂的线程排队，线程阻塞/唤醒，如何保证线程安全，都由AQS为我们完成了，这也是非常典型的模板方法的应用。AQS定义好顶级逻辑的骨架，并提取出公用的线程入队列/出队列，阻塞/唤醒等一系列复杂逻辑的实现，将部分简单的可由使用者决定的操作逻辑延迟到子类中去实现。　



**参考：**

https://www.cnblogs.com/chengxiao/archive/2017/07/24/7141160.html

https://github.com/Essviv/blogs/blob/master/source/_posts/%E5%A4%9A%E7%BA%BF%E7%A8%8B/juc/AQS%E6%A1%86%E6%9E%B6/AQS%E6%A1%86%E6%9E%B6%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md

## LockSupport与wait-notify的使用.

LockSupport，Java之初就有`Object`对象的wait和notify方法可以实现线程的阻塞和唤醒。那么它们的区别 是什么呢？

1. wait notify必须要获取sychronized锁(对象锁，类锁等)，且需要先执行wait，否则notify没有作用，也会造成wait一直没有唤醒。
2. LockSupport，park和unpark使用上没有要求(作用于线程)，直接阻塞线程，且没有顺序之分。 这个类的作用有点类似于`Semaphore`，通过许可证(`permit`)来联系使用它的线程。如果许可证可用，调用`park`方法会立即返回并在这个过程中消费这个许可，不然线程会阻塞。调用`unpark`会使许可证可用。(和`Semaphores`有些许区别,许可证不会累加，最多只有一张。相当于一个信号量(0,1),默认是0 。
3. `LockSupport`并不需要获取对象的监视器。LockSupport机制是每次`unpark`给线程1个“许可”——最多只能是1，而`park`则相反，如果当前 线程有许可，那么park方法会消耗1个并返回，否则会阻塞线程直到线程重新获得许可，在线程启动之前调用`park/unpark`方法没有任何效果。

```
// 1次unpark给线程1个许可
LockSupport.unpark(Thread.currentThread());
// 如果线程非阻塞重复调用没有任何效果
LockSupport.unpark(Thread.currentThread());
// 消耗1个许可
LockSupport.park(Thread.currentThread());
// 阻塞
LockSupport.park(Thread.currentThread());
```

4. `LockSupport.park()`也能响应中断信号，但是跟`Thread.sleep()`不同的是它不会抛出`InterruptedException` 

5. `Object.wait()`方法的Javadoc的话会发现官方也是建议下面这样的用法：

```
synchronized (obj) {
    while (<condition does not hold>)
        ……
        obj.wait();
    ……
}
```

StackOverflow上有一个问题里一个叫[xagyg的回答](http://stackoverflow.com/questions/37026/java-notify-vs-notifyall-all-over-again)解释的比较清楚，有兴趣的可以看下。 简单来说因为：

wait前会释放监视器，被唤醒后又要重新获取，这瞬间可能有其他线程刚好先获取到了监视器，从而导致状态发生了变化， 这时候用while循环来再判断一下条件（比如队列是否为空）来避免不必要或有问题的操作。 这种机制还可以用来处理伪唤醒（spurious wakeup），所谓伪唤醒就是`no reason wakeup`，对于`LockSupport.park()`来说就是除了`unpark`和`interrupt`之外的原因。



**LockSupport的实现**

学习要知其然，还要知其所以然。接下来不妨看看LockSupport的实现。

进入LockSupport的park方法，可以发现它是调用了Unsafe的park方法，这是一个本地native方法，只能通过openjdk的源码看看其本地实现了。 

![img](https://images2017.cnblogs.com/blog/1287675/201801/1287675-20180107152146081-940742591.png)

它调用了线程的Parker类型对象的park方法，如下是Parker类的定义：

![img](https://images2017.cnblogs.com/blog/1287675/201801/1287675-20180107152753721-1014137884.png)

类中定义了一个int类型的_counter变量，咱们上文中讲灵活性的那一节说，可以先执行unpark后执行park，就是通过这个变量实现，看park方法的实现代码(由于方法比较长就不整体截图了)：

![img](https://images2017.cnblogs.com/blog/1287675/201801/1287675-20180107154343534-1144444007.png)

park方法会调用Atomic::xchg方法，这个方法会原子性的将_counter赋值为0，并返回赋值前的值。如果调用park方法前，_counter大于0，则说明之前调用过unpark方法，所以park方法直接返回。

接着往下看：

![img](https://images2017.cnblogs.com/blog/1287675/201801/1287675-20180107161304096-2129308589.png)

实际上Parker类用Posix的mutex，condition来实现的阻塞唤醒。如果对mutex和condition不熟，可以简单理解为mutex就是Java里的synchronized，condition就是Object里的wait/notify操作。

park方法里调用pthread_mutex_trylock方法，就相当于Java线程进入Java的同步代码块，然后再次判断_counter是否大于零，如果大于零则将_counter设置为零。最后调用pthread_mutex_unlock解锁，相当于Java执行完退出同步代码块。如果_counter不大于零，则继续往下执行pthread_cond_wait方法，实现当前线程的阻塞。

 

最后再看看unpark方法的实现吧，这块就简单多了，直接上代码：

![img](https://images2017.cnblogs.com/blog/1287675/201801/1287675-20180107162958987-936744573.png)

图中的1和4就相当于Java的进入synchronized和退出synchronized的加锁解锁操作，代码2将_counter设置为1，同时判断先前_counter的值是否小于1，即这段代码：if(s<1)。如果不小于1，则就不会有线程被park，所以方法直接执行完毕，否则就会执行代码3，来唤醒被阻塞的线程。

 

通过阅读LockSupport的本地实现，我们不难发现这么个问题：多次调用unpark方法和调用一次unpark方法效果一样，因为都是直接将_counter赋值为1，而不是加1。简单说就是：线程A连续调用两次LockSupport.unpark(B)方法唤醒线程B，然后线程B调用两次LockSupport.park()方法， 线程B依旧会被阻塞。因为两次unpark调用效果跟一次调用一样，只能让线程B的第一次调用park方法不被阻塞，第二次调用依旧会阻塞。

 

到这里,自己实现一把“锁”用到的技术点都已经介绍完了，甚至本节还介绍了锁的具体实现，相信即使没有最后一篇介绍怎么实现“锁”，大家也能动手写个锁了。



参考：

https://www.cnblogs.com/qingquanzi/p/8228422.html

https://blog.csdn.net/secsf/article/details/78560013

https://www.cnblogs.com/fairjm/p/locksuport.html



## ReentrantLock实现原理及源码分析

　　ReentrantLock是Java并发包中提供的一个**可重入的互斥锁**。**ReentrantLock**和**synchronized**在基本用法，行为语义上都是类似的，同样都具有可重入性。只不过相比原生的Synchronized，ReentrantLock增加了一些高级的扩展功能，比如它可以实现**公平锁，**同时也可以绑定**多个Conditon**。

### 可重入性/公平锁/非公平锁

　　**可重入性**

　　　　所谓的可重入性，就是**可以支持一个线程对锁的重复获取**，原生的synchronized就具有可重入性，一个用synchronized修饰的递归方法，当线程在执行期间，它是可以反复获取到锁的，而不会出现自己把自己锁死的情况。ReentrantLock也是如此，在调用lock()方法时，已经获取到锁的线程，能够再次调用lock()方法获取锁而不被阻塞。那么有可重入锁，就有不可重入锁，我们在之前文章中自定义的一个Mutex锁就是个不可重入锁，不过使用场景极少而已。

　　**公平锁/非公平锁**

　　　　所谓公平锁,顾名思义，意指锁的获取策略相对公平，当多个线程在获取同一个锁时，必须按照锁的申请时间来依次获得锁，排排队，不能插队；非公平锁则不同，当锁被释放时，等待中的线程均有机会获得锁。synchronized是非公平锁，ReentrantLock默认也是非公平的，但是可以通过带boolean参数的构造方法指定使用公平锁，但**非公平锁的性能一般要优于公平锁。**

　　synchronized是Java原生的互斥同步锁，使用方便，对于synchronized修饰的方法或同步块，无需再显式释放锁。synchronized底层是通过monitorenter和monitorexit两个字节码指令来实现加锁解锁操作的。而ReentrantLock做为API层面的互斥锁，需要显式地去加锁解锁。　

```java
class X {
    private final ReentrantLock lock = new ReentrantLock();
    // ...
 
    public void m() {
      lock.lock();  // 加锁
      try {
        // ... 函数主题
      } finally {
        lock.unlock() //解锁
      }
    }
  }
```

### 源码分析

　　接下来我们从源码角度来看看ReentrantLock的实现原理，它是如何保证可重入性，又是如何实现公平锁的。

　　**ReentrantLock是基于AQS的，AQS是Java并发包中众多同步组件的构建基础，它通过一个int类型的状态变量state和一个FIFO队列来完成共享资源的获取，线程的排队等待等。AQS是个底层框架，采用模板方法模式，它定义了通用的较为复杂的逻辑骨架，比如线程的排队，阻塞，唤醒等，将这些复杂但实质通用的部分抽取出来，这些都是需要构建同步组件的使用者无需关心的，使用者仅需重写一些简单的指定的方法即可（其实就是对于共享变量state的一些简单的获取释放的操作）。**

　　上面简单介绍了下AQS，详细内容可参考本人的另一篇文章《**Java并发包基石-AQS详解**》，此处就不再赘述了。先来看常用的几个方法，我们从上往下推。

　　**无参构造器（默认为公平锁）**

```
public ReentrantLock() {
        sync = new NonfairSync();//默认是非公平的
    }
```

　　sync是ReentrantLock内部实现的一个同步组件，它是Reentrantlock的一个静态内部类，继承于AQS，后面我们再分析。

　　**带布尔值的构造器（是否公平）**

```
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();//fair为true，公平锁；反之，非公平锁
    }
```

　　看到了吧，此处可以指定是否采用公平锁，**FailSync和NonFailSync亦为Reentrantlock的静态内部类，都继承于Sync**。

　　再来看看几个我们常用到的方法

　　**lock()**

```
public void lock() {
        sync.lock();//代理到Sync的lock方法上
    }
```

　　Sync的lock方法是抽象的，实际的lock会代理到FairSync或是NonFairSync上（根据用户的选择来决定，公平锁还是非公平锁）

　　**lockInterruptibly()**

```
public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);//代理到sync的相应方法上，同lock方法的区别是此方法响应中断
    }
```

　　此方法响应中断，当线程在阻塞中的时候，若被中断，会抛出InterruptedException异常 

　　**tryLock()**

```
public boolean tryLock() {
        return sync.nonfairTryAcquire(1);//代理到sync的相应方法上
    }
```

　　tryLock，尝试获取锁，成功则直接返回true，不成功也不耽搁时间，立即返回false。

　　**unlock()**

```
public void unlock() {
        sync.release(1);//释放锁
    }
```

　　释放锁，调用sync的release方法，其实是AQS的release逻辑。

 　　**newCondition()**

　　　　获取一个conditon，ReentrantLock支持多个Condition

```
public Condition newCondition() {
        return sync.newCondition();
    }
```

　　其他方法就不再赘述了，若想继续了解可去API中查看。

　　**小结**

　　**其实从上面这写方法的介绍，我们都能大概梳理出ReentrantLock的处理逻辑，其内部定义了三个重要的静态内部类，Sync，NonFairSync，FairSync。Sync作为ReentrantLock中公用的同步组件，继承了AQS（要利用AQS复杂的顶层逻辑嘛，线程排队，阻塞，唤醒等等）；NonFairSync和FairSync则都继承Sync，调用Sync的公用逻辑，然后再在各自内部完成自己特定的逻辑（公平或非公平）。**

　　接下来，关于如何实现重入性，如何实现公平性，就得去看这几个静态内部类了

　　**NonFairSync（非公平可重入锁）**

```java
static final class NonfairSync extends Sync {//继承Sync
        private static final long serialVersionUID = 7316153563782823691L;
        /** 获取锁 */
        final void lock() {
            if (compareAndSetState(0, 1))//CAS设置state状态，若原值是0，将其置为1
                setExclusiveOwnerThread(Thread.currentThread());//将当前线程标记为已持有锁
            else
                acquire(1);//若设置失败，调用AQS的acquire方法，acquire又会调用我们下面重写的tryAcquire方法。这里说的调用失败有两种情况：1当前没有线程获取到资源，state为0，但是将state由0设置为1的时候，其他线程抢占资源，将state修改了，导致了CAS失败；2 state原本就不为0，也就是已经有线程获取到资源了，有可能是别的线程获取到资源，也有可能是当前线程获取的，这时线程又重复去获取，所以去tryAcquire中的nonfairTryAcquire我们应该就能看到可重入的实现逻辑了。
        }
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);//调用Sync中的方法
        }
    }   
```

　　**nonfairTryAcquire()**

　　简单总结下流程：

　　　　**1.先获取state值，若为0，意味着此时没有线程获取到资源，CAS将其设置为1，设置成功则代表获取到排他锁了；**

　　　　**2.若state大于0，肯定有线程已经抢占到资源了，此时再去判断是否就是自己抢占的，是的话，state累加，返回true，重入成功，state的值即是线程重入的次数；**

　　　　**3.其他情况，则获取锁失败。**

　　 来看看可重入公平锁的处理逻辑

　　**FairSync**

```
static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);//直接调用AQS的模板方法acquire，acquire会调用下面我们重写的这个tryAcquire
        }

        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();//获取当前线程
            int c = getState();//获取state值
            if (c == 0) {//若state为0，意味着当前没有线程获取到资源，那就可以直接获取资源了吗？NO!这不就跟之前的非公平锁的逻辑一样了嘛。看下面的逻辑
                if (!hasQueuedPredecessors() &&//判断在时间顺序上，是否有申请锁排在自己之前的线程，若没有，才能去获取，CAS设置state，并标记当前线程为持有排他锁的线程；反之，不能获取！这即是公平的处理方式。
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {//重入的处理逻辑，与上文一致，不再赘述
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

　　**可以看到，公平锁的大致逻辑与非公平锁是一致的，不同的地方在于有了!hasQueuedPredecessors()这个判断逻辑，即便state为0，也不能贸然直接去获取，要先去看有没有还在排队的线程，若没有，才能尝试去获取，做后面的处理。反之，返回false，获取失败。**

　　看看这个判断是否有排队中线程的逻辑

　　**hasQueuedPredecessors()

```
    public final boolean hasQueuedPredecessors() {
        Node t = tail; // 尾结点
        Node h = head;//头结点
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());//判断是否有排在自己之前的线程
    }
```

　需要注意的是，这个判断是否有排在自己之前的线程的逻辑稍微有些绕，我们来梳理下，由代码得知，有两种情况会返回true，我们将此逻辑分解一下（注意：返回true意味着有其他线程申请锁比自己早，需要放弃抢占）

　　1. **h !=t && (s = h.next) == null**，这个逻辑成立的一种可能是head指向头结点，tail此时还为null。考虑这种情况：当其他某个线程去获取锁失败，需构造一个结点加入同步队列中（假设此时同步队列为空），在添加的时候，需要先创建一个无意义傀儡头结点（在AQS的enq方法中，这是个自旋CAS操作），有可能在将head指向此傀儡结点完毕之后，还未将tail指向此结点。很明显，此线程时间上优于当前线程，所以，返回true，表示有等待中的线程且比自己来的还早。

　　2.**h != t && (s = h.next) != null && s.thread != Thread.currentThread()**。同步队列中已经有若干排队线程且当前线程不是队列的老二结点，此种情况会返回true。假如没有s.thread !=Thread.currentThread()这个判断的话，会怎么样呢？若当前线程已经在同步队列中是老二结点（头结点此时是个无意义的傀儡结点),此时持有锁的线程释放了资源，唤醒老二结点线程，老二结点线程重新tryAcquire（此逻辑在AQS中的acquireQueued方法中），又会调用到hasQueuedPredecessors，不加s.thread !=Thread.currentThread()这个判断的话，返回值就为true，导致tryAcquire失败。

　　最后，来看看ReentrantLock的tryRelease，定义在Sync中

```java
 protected final boolean tryRelease(int releases) {
            int c = getState() - releases;//减去1个资源
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            //若state值为0，表示当前线程已完全释放干净，返回true，上层的AQS会意识到资源已空出。若不为0，则表示线程还占有资源，只不过将此次重入的资源的释放了而已，返回false。
            if (c == 0) {
                free = true;//
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```

###  总结

　　ReentrantLock是一种可重入的，可实现公平性的互斥锁，它的设计基于AQS框架，可重入和公平性的实现逻辑都不难理解，每重入一次，state就加1，当然在释放的时候，也得一层一层释放。至于公平性，在尝试获取锁的时候多了一个判断：是否有比自己申请早的线程在同步队列中等待，若有，去等待；若没有，才允许去抢占。　　

http://www.cnblogs.com/chengxiao/p/7255941.html



## CountDownLatch

1. **含义**

   CountDownLatch可以理解为一个计数器在初始化时设置初始值，当一个线程需要等待某些操作先完成时，需要调用await()方法。这个方法让线程进入休眠状态直到等待的所有线程都执行完成。每调用一次countDown()方法内部计数器减1,直到计数器为0时唤醒。这个可以理解为特殊的CyclicBarrier。线程同步点比较特殊，为内部计数器值为0时开始。

   **方法**

   核心方法两个：countDown()和await()

   countDown():使CountDownLatch维护的内部计数器减1,每个被等待的线程完成的时候调用

   await():线程在执行到CountDownLatch的时候会将此线程置于休眠

2. CountDownLatch初始构造函数传入count，countDown()函数每次调用-1， await等待为0时返回。

3. 实现基于AQS共享锁，覆写tryAcquireShared和tryReleaseShared方法。

```java
public class CountDownLatch {
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;
 
        Sync(int count) {
            setState(count);
        }
 
        int getCount() {
            return getState();
        }
 
        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }
 
        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }
 
    private final Sync sync;
 
	//初始化一个同步器
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }
	//调用同步器的acquireSharedInterruptibly方法
	//并且是响应中断的
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
 
	//调用同步器的releaseShared方法去让state减1
    public void countDown() {
        sync.releaseShared(1);
    }
	//获取剩余的count
    public long getCount() {
        return sync.getCount();
    }
 
    public String toString() {
        return super.toString() + "[Count = " + sync.getCount() + "]";
    }
}

```



案例：

https://blog.csdn.net/ErixHao/article/details/52747955



## CyclicBarrier

1. 可以理解为循环栅栏,栅栏就是一种障碍物.假如我们将计数器设置为10,那么凑齐第一批10个线程后,计数器就会归零,然后接着凑齐下一批10个线程,这就是循环栅栏的含义.

   构造器:

```java
public CyclicBarrier(int parties, Runnable barrierAction)
```

​	parties:计数总数,也就是参与的线程总数. barrierAction 当计数器一次完成计数后,系统会执行的动作

​	**方法**

​	await()：使线程置入休眠直到最后一个线程的到来之后唤醒所有休眠的线程

2. 没有显示调用CountDown()方法
3. CountDownLatch一般只能使用一次，CyclicBarrier可以多次使用

```java
public class CyclicBarrierDemo {
    public static class Soldier implements Runnable {
        private String soldier;
        private final CyclicBarrier cyclic;

        public Soldier(CyclicBarrier cyclic, String soldier) {
            this.soldier = soldier;
            this.cyclic = cyclic;
        }

        /**
         * When an object implementing interface <code>Runnable</code> is used
         * to create a thread, starting the thread causes the object's
         * <code>run</code> method to be called in that separately executing
         * thread.
         * <p>
         * The general contract of the method <code>run</code> is that it may
         * take any action whatsoever.
         *
         * @see Thread#run()
         */
        @Override
        public void run() {
            try {
                //等待所有士兵到齐
                cyclic.await();
                doWork();
                //等待所有士兵完成工作
                cyclic.await();
            } catch (InterruptedException e) {//在等待过程中,线程被中断
                e.printStackTrace();
            } catch (BrokenBarrierException e) {//表示当前CyclicBarrier已经损坏.系统无法等到所有线程到齐了.
                e.printStackTrace();
            }
        }

        void doWork() {
            try {
                Thread.sleep(Math.abs(new Random().nextInt() % 10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(soldier + ":任务完成");
        }

    }

    public static class BarrierRun implements Runnable {
        boolean flag;
        int N;

        public BarrierRun(boolean flag, int N) {
            this.flag = flag;
            this.N = N;
        }

        /**
         * When an object implementing interface <code>Runnable</code> is used
         * to create a thread, starting the thread causes the object's
         * <code>run</code> method to be called in that separately executing
         * thread.
         * <p>
         * The general contract of the method <code>run</code> is that it may
         * take any action whatsoever.
         *
         * @see Thread#run()
         */
        @Override
        public void run() {
            if (flag) {
                System.out.println("司令:[士兵" + N + "个,任务完成!]");
            } else {
                System.out.println("司令:[士兵" + N + "个,集合完毕!]");
                flag = true;
            }
        }
    }

    public static void main(String[] args) {
        final int N = 10;
        Thread[] allSoldier = new Thread[N];
        boolean flag = false;
        CyclicBarrier cyclic = new CyclicBarrier(N, new BarrierRun(flag, N));
        //设置屏障点,主要为了执行这个方法
        System.out.println("集合队伍! ");
        for (int i = 0; i < N; i++) {
            System.out.println("士兵" + i + "报道! ");
            allSoldier[i] = new Thread(new Soldier(cyclic, "士兵" + i));
            allSoldier[i].start();
        }
    }
}
```



```java
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.BrokenBarrierException;

public class CyclicBarrierTest1 {

    private static int SIZE = 5;
    private static CyclicBarrier cb;
    public static void main(String[] args) {

        cb = new CyclicBarrier(SIZE);

        // 新建5个任务
        for(int i=0; i<SIZE; i++)
            new InnerThread().start();
    }

    static class InnerThread extends Thread{
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " wait for CyclicBarrier.");

                // 将cb的参与者数量加1
                cb.await();

                // cb的参与者数量等于5时，才继续往后执行
                System.out.println(Thread.currentThread().getName() + " continued.");
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```





比较[CountDownLatch](http://www.cnblogs.com/skywang12345/p/3533887.html)和[CyclicBarrier](http://www.cnblogs.com/skywang12345/p/3533995.html)：

 (01) CountDownLatch的作用是允许1或N个线程等待其他线程完成执行；而CyclicBarrier则是允许N个线程相互等待。

 (02) CountDownLatch的计数器无法被重置；CyclicBarrier的计数器可以被重置后使用，因此它被称为是循环的barrier。 



参考：

https://www.cnblogs.com/skywang12345/p/3533995.html

https://www.cnblogs.com/ten951/p/6212160.html

https://www.cnblogs.com/uodut/p/6830939.html#_label2



## CopyOnWriteList和CopyOnWriteSet

　Copy-On-Write简称COW，是一种用于程序设计中的优化策略。其基本思路是，从一开始大家都在共享同一个内容，当某个人想要修改这个内容的时候，才会真正把内容Copy出去形成一个新的内容然后再改，这是一种延时懒惰策略。从JDK1.5开始Java并发包里提供了两个使用CopyOnWrite机制实现的并发容器,它们是CopyOnWriteArrayList和CopyOnWriteArraySet。CopyOnWrite容器非常有用，可以在非常多的并发场景中使用到。

## 什么是CopyOnWrite容器

　　CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。



**CopyOnWriteArrayList中写操作需要大面积复制数组，所以性能肯定很差**，但是**读操作因为操作的对象和写操作不是同一个对象，读之间也不需要加锁，读和写之间的同步处理只是在写完后通过一个简单的“=”将引用指向新的数组对象上来，这个几乎不需要时间**，**这样读操作就很快很安全，适合在多线程里使用，绝对不会发生ConcurrentModificationException**，所以最后得出结论：**CopyOnWriteArrayList适合使用在读操作远远大于写操作的场景里，比如缓存。**



COWSet的底层是通过COWList实现的， 在写操作的时候，有选择性的选择addIfAbsent版本的操作.

COWList的底层实现是通过ReentrantLock来实现的，所有的写操作执行前都必须先获取到相应的RL锁，然后再进行操作.

https://www.cnblogs.com/dolphin0520/p/3938914.html



在线程对其进行些操作的时候，会拷贝一个新的数组以存放新的字段。

1. `private volatile transient Object[] array;`//保证了线程的可见性，所有操作直接操作主存。
2. 写操作加锁，读操作不加锁，相对来说读的效率更高
3. 可变操作（add()、set() 和 remove() 等等）的开销很大, 因为通常需要复制整个基础数组。
4. 
5. 写操作`Object[] newElements = Arrays.copyOf(elements, len + 1);`直接加锁，复制，新建一个数组，写回主存，get操作不加锁直接操作

```java
    /** The lock protecting all mutators */
    transient final ReentrantLock lock = new ReentrantLock();
 
    /** The array, accessed only via getArray/setArray. */
    private volatile transient Object[] array;//保证了线程的可见性

	public boolean add(E e) {
        final ReentrantLock lock = this.lock;//ReentrantLock 保证了线程的可见性和顺序性，即保证了多线程安全。
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);//在原先数组基础之上新建长度＋1的数组，并将原先数组当中的内容拷贝到新数组当中。
            newElements[len] = e;//设值
            setArray(newElements);//对新数组进行赋值
            return true;
        } finally {
            lock.unlock();
        }
    }

    public E get(int index) {
        return (E)(getArray()[index]);
    }

```

3. 迭代不会出现ConcurrentModificationException,  因为迭代器会保存一个快照`private final Object[] snapshot;`, 而不是直接操作原来的Object[] array，不可变，不会发生同步问题。
    迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等操作, 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照
```java
    public Iterator<E> iterator() {
        return new COWIterator<E>(getArray(), 0);
    }
    public ListIterator<E> listIterator() {
        return new COWIterator<E>(getArray(), 0);
    }
    public ListIterator<E> listIterator(final int index) {
        Object[] elements = getArray();
        int len = elements.length;
        if (index<0 || index>len)
            throw new IndexOutOfBoundsException("Index: "+index);
 
        return new COWIterator<E>(elements, index);
    }
 
    private static class COWIterator<E> implements ListIterator<E> {
        private final Object[] snapshot; // 保存数组的快照，是一个不可变的对象
        private int cursor;
 
        private COWIterator(Object[] elements, int initialCursor) {
            cursor = initialCursor;
            snapshot = elements;
        }
 
        public boolean hasNext() {
            return cursor < snapshot.length;
        }
 
        public boolean hasPrevious() {
            return cursor > 0;
        }
 
        @SuppressWarnings("unchecked")
        public E next() {
            if (! hasNext())
                throw new NoSuchElementException();
            return (E) snapshot[cursor++];
        }
 
        @SuppressWarnings("unchecked")
        public E previous() {
            if (! hasPrevious())
                throw new NoSuchElementException();
            return (E) snapshot[--cursor];
        }
 
        public int nextIndex() {
            return cursor;
        }
 
        public int previousIndex() {
            return cursor-1;
        }
        public void remove() {
            throw new UnsupportedOperationException();
        }
        public void set(E e) {
            throw new UnsupportedOperationException();
        }
        public void add(E e) {
            throw new UnsupportedOperationException();
        }
    }

```

参考：https://blog.csdn.net/mazhimazh/article/details/19210547



## Synchronized原理



记得刚刚开始学习Java的时候，一遇到多线程情况就是synchronized，相对于当时的我们来说synchronized是这么的神奇而又强大，那个时候我们赋予它一个名字“同步”，也成为了我们解决多线程情况的百试不爽的良药。但是，随着我们学习的进行我们知道synchronized是一个重量级锁，相对于Lock，它会显得那么笨重，以至于我们认为它不是那么的高效而慢慢摒弃它。
诚然，随着Javs SE 1.6对synchronized进行的各种优化后，synchronized并不会显得那么重了。下面跟随LZ一起来探索synchronized的实现机制、Java是如何对它进行了优化、锁优化机制、锁的存储结构和升级过程；

### 实现原理

> synchronized可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时它还可以保证共享变量的内存可见性

Java中每一个对象都可以作为锁，这是synchronized实现同步的基础：

1. 普通同步方法，锁是当前实例对象
2. 静态同步方法，锁是当前类的class对象
3. 同步方法块，锁是括号里面的对象

当一个线程访问同步代码块时，它首先是需要得到锁才能执行同步代码，当退出或者抛出异常时必须要释放锁，那么它是如何来实现这个机制的呢？我们先看一段简单的代码：

利用javap工具查看生成的class文件信息来分析Synchronize的实现
[![Synchronize-1](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/Synchronize-1_thumb-1.jpg)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/Synchronize-1-1.jpg)
从上面可以看出，**同步代码块是使用monitorenter和monitorexit指令实现的，同步方法（在这看不出来需要看JVM底层实现）依靠的是方法修饰符上的ACC_SYNCHRONIZED实现**。
**同步代码块**：monitorenter指令插入到同步代码块的开始位置，monitorexit指令插入到同步代码块的结束位置，JVM需要保证每一个monitorenter都有一个monitorexit与之相对应。任何对象都有一个monitor与之相关联，当且一个monitor被持有之后，他将处于锁定状态。线程执行到monitorenter指令时，将会尝试获取对象所对应的monitor所有权，即尝试获取对象的锁；
**同步方法**：synchronized方法则会被翻译成普通的方法调用和返回指令如:invokevirtual、areturn指令，在VM字节码层面并没有任何特别的指令来实现被synchronized修饰的方法，而是在Class文件的方法表中将该方法的access_flags字段中的synchronized标志位置1，表示该方法是同步方法并使用调用该方法的对象或该方法所属的Class在JVM的内部对象表示Klass做为锁对象。(摘自：<http://www.cnblogs.com/javaminer/p/3889023.html>)

下面我们来继续分析，但是在深入之前我们需要了解两个重要的概念：Java对象头，Monitor。

### Java对象头、monitor

Java对象头和monitor是实现synchronized的基础！下面就这两个概念来做详细介绍。

### Java对象头

synchronized用的锁是存在Java对象头里的，那么什么是Java对象头呢？Hotspot虚拟机的对象头主要包括两部分数据：Mark Word（标记字段）、Klass Pointer（类型指针）。其中Klass Point是是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，Mark Word用于存储对象自身的运行时数据，它是实现轻量级锁和偏向锁的关键，所以下面将重点阐述

Mark Word。
Mark Word用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等等。Java对象头一般占有两个机器码（在32位虚拟机中，1个机器码等于4字节，也就是32bit），但是如果对象是数组类型，则需要三个机器码，因为JVM虚拟机可以通过Java对象的元数据信息确定Java对象的大小，但是无法从数组的元数据来确认数组的大小，所以用一块来记录数组长度。下图是Java对象头的存储结构（32位虚拟机）：
[![222222_2](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/222222_2_thumb-1.jpg)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/222222_2-1.jpg)
对象头信息是与对象自身定义的数据无关的额外存储成本，但是考虑到虚拟机的空间效率，Mark Word被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据，它会根据对象的状态复用自己的存储空间，也就是说，Mark Word会随着程序的运行发生变化，变化状态如下（32位虚拟机）：
[![11111111111_2](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/11111111111_2_thumb-1.jpg)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/11111111111_2-1.jpg)

简单介绍了Java对象头，我们下面再看Monitor。

### Monitor

什么是Monitor？我们可以把它理解为一个同步工具，也可以描述为一种同步机制，它通常被描述为一个对象。
与一切皆对象一样，所有的Java对象是天生的Monitor，每一个Java对象都有成为Monitor的潜质，因为在Java的设计中 ，每一个Java对象自打娘胎里出来就带了一把看不见的锁，它叫做内部锁或者Monitor锁。
Monitor 是线程私有的数据结构，每一个线程都有一个可用monitor record列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个monitor关联（对象头的MarkWord中的LockWord指向monitor的起始地址），同时monitor中有一个Owner字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。其结构如下：
[![44444](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/44444_thumb-1.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/44444-1.png)
Owner：初始时为NULL表示当前没有任何线程拥有该monitor record，当线程成功拥有该锁后保存线程唯一标识，当锁被释放时又设置为NULL；
EntryQ:关联一个系统互斥锁（semaphore），阻塞所有试图锁住monitor record失败的线程。
RcThis:表示blocked或waiting在该monitor record上的所有线程的个数。
Nest:用来实现重入锁的计数。
HashCode:保存从对象头拷贝过来的HashCode值（可能还包含GC age）。
Candidate:用来避免不必要的阻塞或等待线程唤醒，因为每一次只有一个线程能够成功拥有锁，如果每次前一个释放锁的线程唤醒所有正在阻塞或等待的线程，会引起不必要的上下文切换（从阻塞到就绪然后因为竞争锁失败又被阻塞）从而导致性能严重下降。Candidate只有两种可能的值0表示没有需要唤醒的线程1表示要唤醒一个继任线程来竞争锁。
摘自：[Java中synchronized的实现原理与应用）](http://blog.csdn.net/u012465296/article/details/53022317)
我们知道synchronized是重量级锁，效率不怎么滴，同时这个观念也一直存在我们脑海里，不过在jdk 1.6中对synchronize的实现进行了各种优化，使得它显得不是那么重了，那么JVM采用了那些优化手段呢？

### 锁优化

jdk1.6对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。
锁主要存在四中状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。

### 自旋锁

线程的阻塞和唤醒需要CPU从用户态转为核心态，频繁的阻塞和唤醒对CPU来说是一件负担很重的工作，势必会给系统的并发性能带来很大的压力。同时我们发现在许多应用上面，对象锁的锁状态只会持续很短一段时间，为了这一段很短的时间频繁地阻塞和唤醒线程是非常不值得的。所以引入自旋锁。
何谓自旋锁？
所谓自旋锁，就是让该线程等待一段时间，不会被立即挂起，看持有锁的线程是否会很快释放锁。怎么等待呢？执行一段无意义的循环即可（自旋）。
自旋等待不能替代阻塞，先不说对处理器数量的要求（多核，貌似现在没有单核的处理器了），虽然它可以避免线程切换带来的开销，但是它占用了处理器的时间。如果持有锁的线程很快就释放了锁，那么自旋的效率就非常好，反之，自旋的线程就会白白消耗掉处理的资源，它不会做任何有意义的工作，典型的占着茅坑不拉屎，这样反而会带来性能上的浪费。所以说，自旋等待的时间（自旋的次数）必须要有一个限度，如果自旋超过了定义的时间仍然没有获取到锁，则应该被挂起。
自旋锁在JDK 1.4.2中引入，默认关闭，但是可以使用-XX:+UseSpinning开开启，在JDK1.6中默认开启。同时自旋的默认次数为10次，可以通过参数-XX:PreBlockSpin来调整；
如果通过参数-XX:preBlockSpin来调整自旋锁的自旋次数，会带来诸多不便。假如我将参数调整为10，但是系统很多线程都是等你刚刚退出的时候就释放了锁（假如你多自旋一两次就可以获取锁），你是不是很尴尬。于是JDK1.6引入自适应的自旋锁，让虚拟机会变得越来越聪明。

### 适应自旋锁

JDK 1.6引入了更加聪明的自旋锁，即自适应自旋锁。所谓自适应就意味着自旋的次数不再是固定的，它是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。它怎么做呢？线程如果自旋成功了，那么下次自旋的次数会更加多，因为虚拟机认为既然上次成功了，那么此次自旋也很有可能会再次成功，那么它就会允许自旋等待持续的次数更多。反之，如果对于某个锁，很少有自旋能够成功的，那么在以后要或者这个锁的时候自旋的次数会减少甚至省略掉自旋过程，以免浪费处理器资源。
有了自适应自旋锁，随着程序运行和性能监控信息的不断完善，虚拟机对程序锁的状况预测会越来越准确，虚拟机会变得越来越聪明。

### 锁消除

为了保证数据的完整性，我们在进行操作时需要对这部分操作进行同步控制，但是在有些情况下，JVM检测到不可能存在共享数据竞争，这是JVM会对这些同步锁进行锁消除。锁消除的依据是逃逸分析的数据支持。
如果不存在竞争，为什么还需要加锁呢？所以锁消除可以节省毫无意义的请求锁的时间。变量是否逃逸，对于虚拟机来说需要使用数据流分析来确定，但是对于我们程序员来说这还不清楚么？我们会在明明知道不存在数据竞争的代码块前加上同步吗？但是有时候程序并不是我们所想的那样？我们虽然没有显示使用锁，但是我们在使用一些JDK的内置API时，如StringBuffer、Vector、HashTable等，这个时候会存在隐形的加锁操作。比如StringBuffer的append()方法，Vector的add()方法：

在运行这段代码时，JVM可以明显检测到变量vector没有逃逸出方法vectorTest()之外，所以JVM可以大胆地将vector内部的加锁操作消除。

### 锁粗化

我们知道在使用同步锁的时候，需要让同步块的作用范围尽可能小—仅在共享数据的实际作用域中才进行同步，这样做的目的是为了使需要同步的操作数量尽可能缩小，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。
在大多数的情况下，上述观点是正确的，LZ也一直坚持着这个观点。但是如果一系列的连续加锁解锁操作，可能会导致不必要的性能损耗，所以引入锁粗话的概念。
锁粗话概念比较好理解，就是将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁。如上面实例：vector每次add的时候都需要加锁操作，JVM检测到对同一个对象（vector）连续加锁、解锁操作，会合并一个更大范围的加锁、解锁操作，即加锁解锁操作会移到for循环之外。



**偏向锁->轻量级锁->重量级锁**

偏向所锁，轻量级锁都是乐观锁，重量级锁是悲观锁。

​        一个对象刚开始实例化的时候，没有任何线程来访问它的时候。它是可偏向的，意味着，它现在认为只可能有一个线程来访问它，所以当第一个

线程来访问它的时候，它会偏向这个线程，此时，对象持有偏向锁。偏向第一个线程，这个线程在修改对象头成为偏向锁的时候使用CAS操作，并将

对象头中的ThreadID改成自己的ID，之后再次访问这个对象时，只需要对比ID，不需要再使用CAS在进行操作。

一旦有第二个线程访问这个对象，因为偏向锁不会主动释放，所以第二个线程可以看到对象时偏向状态，这时表明在这个对象上已经存在竞争了，检查原来持有该对象锁的线程是否依然存活，如果挂了，则可以将对象变为无锁状态，然后重新偏向新的线程，如果原来的线程依然存活，则马上执行那个线程的操作栈，检查该对象的使用情况，如果仍然需要持有偏向锁，则偏向锁升级为轻量级锁，（**偏向锁就是这个时候升级为轻量级锁的**）。如果不存在使用了，则可以将对象回复成无锁状态，然后重新偏向。

​        轻量级锁认为竞争存在，但是竞争的程度很轻，一般两个线程对于同一个锁的操作都会错开，或者说稍微等待一下（自旋），另一个线程就会释放锁。 但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁膨胀为重量级锁，重量级锁使除了拥有锁的线程以外的线程都阻塞，防止CPU空转。



### 轻量级锁

引入轻量级锁的主要目的是在多没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗。当关闭偏向锁功能或者多个线程竞争偏向锁导致偏向锁升级为轻量级锁，则会尝试获取轻量级锁，其步骤如下：
获取锁

1. 判断当前对象是否处于无锁状态（hashcode、0、01），若是，则JVM首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的Mark Word的拷贝（官方把这份拷贝加了一个Displaced前缀，即Displaced Mark Word）；否则执行步骤（3）；
2. JVM利用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指正，如果成功表示竞争到锁，则将锁标志位变成00（表示此对象处于轻量级锁状态），执行同步操作；如果失败则执行步骤（3）；
3. 判断当前对象的Mark Word是否指向当前线程的栈帧，如果是则表示当前线程已经持有当前对象的锁，则直接执行同步代码块；否则只能说明该锁对象已经被其他线程抢占了，这时轻量级锁需要膨胀为重量级锁，锁标志位变成10，后面等待的线程将会进入阻塞状态；

释放锁
轻量级锁的释放也是通过CAS操作来进行的，主要步骤如下：

1. 取出在获取轻量级锁保存在Displaced Mark Word中的数据；
2. 用CAS操作将取出的数据替换当前对象的Mark Word中，如果成功，则说明释放锁成功，否则执行（3）；
3. 如果CAS操作替换失败，说明有其他线程尝试获取该锁，则需要在释放锁的同时需要唤醒被挂起的线程。

对于轻量级锁，其性能提升的依据是“对于绝大部分的锁，在整个生命周期内都是不会存在竞争的”，如果打破这个依据则除了互斥的开销外，还有额外的CAS操作，因此在有多线程竞争的情况下，轻量级锁比重量级锁更慢；

------

下图是轻量级锁的获取和释放过程
[![22222222222222](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/22222222222222_thumb-1.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/22222222222222-1.png)

### 偏向锁

引入偏向锁主要目的是：为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径。上面提到了轻量级锁的加锁解锁操作是需要依赖多次CAS原子指令的。那么偏向锁是如何来减少不必要的CAS操作呢？我们可以查看Mark work的结构就明白了。只需要检查是否为偏向锁、锁标识为以及ThreadID即可，处理流程如下：
获取锁

1. 检测Mark Word是否为可偏向状态，即是否为偏向锁1，锁标识位为01；
2. 若为可偏向状态，则测试线程ID是否为当前线程ID，如果是，则执行步骤（5），否则执行步骤（3）；
3. 如果线程ID不为当前线程ID，则通过CAS操作竞争锁，竞争成功，则将Mark Word的线程ID替换为当前线程ID，否则执行线程（4）；
4. 通过CAS竞争锁失败，证明当前存在多线程竞争情况，当到达全局安全点，获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码块；
5. 执行同步代码块

释放锁
偏向锁的释放采用了一种只有竞争才会释放锁的机制，线程是不会主动去释放偏向锁，需要等待其他线程来竞争。偏向锁的撤销需要等待全局安全点（这个时间点是上没有正在执行的代码）。其步骤如下：

1. 暂停拥有偏向锁的线程，判断锁对象石是否还处于被锁定状态；
2. 撤销偏向苏，恢复到无锁状态（01）或者轻量级锁的状态；

------

下图是偏向锁的获取和释放流程
[![image2](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/image2_thumb-1.png)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/02/image2-1.png)

### 重量级锁

重量级锁通过对象内部的监视器（monitor）实现，其中monitor的本质是依赖于底层操作系统的Mutex Lock实现，操作系统实现线程之间的切换需要从用户态到内核态的切换，切换成本非常高。



参考：    

http://www.importnew.com/23511.html

 https://www.cnblogs.com/charlesblc/p/5994162.html

https://blog.csdn.net/choukekai/article/details/63688332

https://www.jianshu.com/p/e674ee68fd3f



## 线程安全集合类

copy from： http://colobu.com/2014/11/27/java-collections-overview/

### Arrays

Array是Java唯一内建的集合类型。在你预先知道所要处理数据元素个数的情况下特别有用。java.util.Arrays 包含了许多处理数据的实用方法：

- `Arrays.asList` - 可以把 Array 转换成 List。将转换结果作为其他集合类型构造函数的参数。
- `Arrays.binarySearch` - 在一个已排序的数组中或者其中一段快速查找。
- `Arrays.copyOf` - 如果你想扩大数组容量又想保留它的内容的时候可以使用这个方法。
- `Arrays.copyOfRange` - 可以复制整个数组或其中的一段。
- `Arrays.deepEquals、Arrays.deepHashCode` - Arrays.equals/hashCode的高级版本，支持嵌套的子数组。
- `Arrays.equals` - 如果你想要比较两个数组是否相等，应该调用这个方法而不是数组对象的 equals方法。 加入两个数组包含相同数量的元素， 并且对应的元素的equals是true, 这个方法才返回true。 换句话说， 只有相同(equals=true)的元素并且顺序一样的两个数组才相等。
- `Arrays.fill` - 用一个给定的值填充整个数组或其中的一段。
- `Arrays.hashCode` - 用来根据数组的内容计算其哈希值（数组对象的hashCode()不可用）。这个方法集合了Java 5的自动装箱和变长参数的特性，来实现一个简单的hashcode方法。
- `Arrays.sort` - 对整个数组或者数组的一段进行排序。也可以使用此方法用给定的比较器Comparator对对象数组进行排序。
- `Arrays.toString` - 打印数组的内容。

如果想要复制整个数组或其中一部分到另一个数组，可以调用 System.arraycopy方法。此方法从源数组中指定的位置复制指定个数的元素到目标数组的指定位置。通常这是最快的复制数组方法。（有时候用 ByteBuffer bulk复制会更快。可以参考[这篇文章](http://java-performance.info/various-methods-of-binary-serialization-in-java/)）.

最后一点，所有的集合都可以用`T[] Collection.toArray( T[] a )` 这个方法复制到数组中。通常会用这样的方式调用：

```
return coll.toArray( new T[ coll.size() ] );
```

这个方法会分配足够大的数组来储存所有的集合元素，这样 toArray 方法本身不必为返回值再分配空间了。

### 非线程安全集合

这一部分介绍的是非线程安全的集合。这些集合都在java.util包里。其中一些在Java 1.o的时候就有了（现在已经弃用），其中大多数在Java 1.4中发布。枚举集合在Java 1.5中发布，并且从这个版本之后所有的集合都支持泛型。PriorityQueue也在Java 1.5中加入。非线程安全的集合架构的最后一个发布的类是ArrayDeque ，在Java 1.6中发布。

#### List

- `ArrayList` - 最有用的List接口实现。由一个整数（数组中第一个还未被使用的元素的位置）和数组作为后台实现。像所有的List一样，ArrayList可以在必要的时候扩展它的大小。ArrayList访问元素的时间开销固定。在尾部添加元素成本低（为常数复杂度），而在头部添加元素成本很高（线性复杂度）。这是由ArrayList的实现原理——所有的元素的从角标为0开始一个接着一个排列造成的。也就是说，从要插入的元素位置往后，每个元素都要向后移动一个位置。它是一个CPU缓存友好的集合， 因为它是基于数组的。（其实也不是非常友好，因为有时数组会包含对象，这样存储的只是指向实际对象的指针）。
- `LinkedList` - Deque实现：每一个节点都保存着上一个节点和下一个节点的指针。这就意味着数据的存取和更新具有线性复杂度（这也是一个最佳化的实现，每次操作都不会遍历数组一半以上，操作成本最高的元素就是数组中间的那个）。如果想写出高效的LinkedList代码可以使用 ListIterators 。如果你想用一个Queue/Deque实现的话（你只需读取第一个和最后一个元素就行了）——考虑用ArrayDeque代替。
- `Vector` - 一个带有线程同步方法的ArrayList版本。现在直接用ArrayList代替了。

#### Queues/deques

- `ArrayDeque` - Deque（双向队列）是基于有首尾指针的数组（环形缓冲区）实现的。和LinkedList不同，这个类没有实现List接口。因此，如果没有首尾元素的话就不能取出任何元素。这个类比LinkedList要好一些，因为它产生的垃圾数量较少（在扩容的时候旧的数组会被丢弃）。
- `Stack` - 栈，一种后进先出的队列。不要在生产代码中使用，使用别的Deque来代替（ArrayDeque比较好）。
- `PriorityQueue` - 一个基于优先级的队列。使用自然顺序或者制定的比较器来排序。他的主要属性——poll/peek/remove/element会返回一个队列的最小值。不仅如此，PriorityQueue还实现了Iterable接口，队列迭代时不进行排序（或者其他顺序）。在需要排序的集合中，使用这个队列会比TreeSet等其他队列要方便。

#### Maps

- `HashMap`：最常用的Map实现。只是将一个键和值相对应，并没有其他的功能。对于复杂的hashCode method，get/put方法有固定的复杂度。
- `EnumMap`：枚举类型作为键值的Map。因为键的数量相对固定，所以在内部用一个数组储存对应值。通常来说，效率要高于HashMap。
- `HashTable`：旧HashMap的同步版本，新的代码中也使用了HashMap。
- `IdentityHashMap`：这是一个特殊的Map版本，它违背了一般Map的规则：它使用 “==” 来比较引用而不是调用Object.equals来判断相等。这个特性使得此集合在遍历图表的算法中非常实用——可以方便地在IdentityHashMap中存储处理过的节点以及相关的数据。
- `LinkedHashMap` ：HashMap和LinkedList的结合，所有元素的插入顺序存储在LinkedList中。这就是为什么迭代LinkedHashMap的条目（entry）、键和值的时候总是遵循插入的顺序。在JDK中，这是每元素消耗内存最大的集合。
- `TreeMap`：一种基于已排序且带导向信息Map的红黑树。每次插入都会按照自然顺序或者给定的比较器排序。这个Map需要实现equals方法和Comparable/Comparator。compareTo需要前后一致。这个类实现了一个NavigableMap接口：可以带有与键数量不同的入口，可以得到键的上一个或者下一个入口，可以得到另一Map某一范围的键（大致和SQL的BETWEEN运算符相同），以及其他的一些方法。
- `WeakHashMap`：这种Map通常用在数据缓存中。它将键存储在WeakReference中，就是说，如果没有强引用指向键对象的话，这些键就可以被垃圾回收线程回收。值被保存在强引用中。因此，你要确保没有引用从值指向键或者将值也保存在弱引用中m.put(key, new WeakReference(value))。

#### Sets

- `HashSet`：一个基于HashMap的Set实现。其中，所有的值为“假值”（同一个Object对象具备和HashMap同样的性能。基于这个特性，这个数据结构会消耗更多不必要的内存。
- `EnumSet`：值为枚举类型的Set。Java的每一个enum都映射成一个不同的int。这就允许使用BitSet——一个类似的集合结构，其中每一比特都映射成不同的enum。EnumSet有两种实现，RegularEnumSet——由一个单独的long存储（能够存储64个枚举值，99.9%的情况下是够用的），JumboEnumSet——由long[]存储。
- `BitSet`：一个比特Set。需要时常考虑用BitSet处理一组密集的整数Set（比如从一个预先知道的数字开始的id集合）。这个类用 long[]来存储bit。
- `LinkedHashMap`：与HashSet一样，这个类基于LinkedHashMap实现。这是唯一一个保持了插入顺序的Set。
- `TreeSet`：与HashSet类似。这个类是基于一个TreeMap实例的。这是在单线程部分唯一一个排序的Set。

#### java.util.Collections

就像有专门的java.util.Arrays来处理数组，Java中对集合也有java.util.Collections来处理。
第一组方法主要返回集合的各种数据：

- `Collections.checkedCollection / checkedList / checkedMap / checkedSet / checkedSortedMap / checkedSortedSet`：检查要添加的元素的类型并返回结果。任何尝试添加非法类型的变量都会抛出一个ClassCastException异常。这个功能可以防止在运行的时候出错。//fixme
- `Collections.emptyList / emptyMap / emptySet` ：返回一个固定的空集合，不能添加任何元素。
- `Collections.singleton / singletonList / singletonMap`：返回一个只有一个入口的 set/list/map 集合。
- `Collections.synchronizedCollection / synchronizedList / synchronizedMap / synchronizedSet / synchronizedSortedMap / synchronizedSortedSet`：获得集合的线程安全版本（多线程操作时开销低但不高效，而且不支持类似put或update这样的复合操作）
- `Collections.unmodifiableCollection / unmodifiableList / unmodifiableMap / unmodifiableSet / unmodifiableSortedMap / unmodifiableSortedSet`：返回一个不可变的集合。当一个不可变对象中包含集合的时候，可以使用此方法。

第二组方法中，其中有一些方法因为某些原因没有加入到集合中：

- `Collections.addAll`：添加一些元素或者一个数组的内容到集合中。
- `Collections.binarySearch`：和数组的Arrays.binarySearch功能相同。
- `Collections.disjoint`：检查两个集合是不是没有相同的元素。
- `Collections.fill`：用一个指定的值代替集合中的所有元素。
- `Collections.frequency`：集合中有多少元素是和给定元素相同的。
- `Collections.indexOfSubList / lastIndexOfSubList`：和String.indexOf(String) / lastIndexOf(String)方法类似——找出给定的List中第一个出现或者最后一个出现的子表。
- `Collections.max / min`：找出基于自然顺序或者比较器排序的集合中，最大的或者最小的元素。
- `Collections.replaceAll`：将集合中的某一元素替换成另一个元素。
- `Collections.reverse`：颠倒排列元素在集合中的顺序。如果你要在排序之后使用这个方法的话，在列表排序时，最好使用Collections.reverseOrder比较器。
- `Collections.rotate`：根据给定的距离旋转元素。
- `Collections.shuffle`：随机排放List集合中的节点，可以给定你自己的生成器——例如 java.util.Random / java.util.ThreadLocalRandom or java.security.SecureRandom。
- `Collections.sort`：将集合按照自然顺序或者给定的顺序排序。
- `Collections.swap`：交换集合中两个元素的位置（多数开发者都是自己实现这个操作的）。

### 并发集合

这一部分将介绍java.util.concurrent包中线程安全的集合。这些集合的一个主要特性就是保证方法的原子执行。因为并发的操作，例如add或update或者check再update，都有一次以上的调用，必须同步。因为第一步从集合中组合操作查询到的信息在开始第二步操作时可能变为无效数据。

大部分的并发集合是在Java 1.5引入的。ConcurrentSkipListMap / ConcurrentSkipListSet 和 LinkedBlockingDeque是在Java 1.6新加入的。最后加入到Java 1.7是 ConcurrentLinkedDeque 和 LinkedTransferQueue。

#### Lists

- `CopyOnWriteArrayList`：list的实现每一次更新都会产生一个新的隐含数组副本，所以这个操作成本很高。通常用在遍历操作比更新操作多的集合，比如listeners/observers集合。

#### Queues/deques

- `ArrayBlockingQueue`：基于数组实现的一个有界阻塞队，大小不能重新定义。所以当你试图向一个满的队列添加元素的时候，就会受到阻塞，直到另一个方法从队列中取出元素。
- `ConcurrentLinkedDeque / ConcurrentLinkedQueue`：基于链表实现的无界队列，添加元素不会堵塞。但是这就要求这个集合的消费者工作速度至少要和生产这一样快，不然内存就会耗尽。严重依赖于CAS(compare-and-set)操作。
- `DelayQueue`：无界的保存Delayed元素的集合。元素只有在延时已经过期的时候才能被取出。队列的第一个元素延期最小（包含负值——延时已经过期）。当你要实现一个延期任务的队列的时候使用（不要自己手动实现——使用ScheduledThreadPoolExecutor）。
- `LinkedBlockingDeque / LinkedBlockingQueue`：可选择有界或者无界基于链表的实现。在队列为空或者满的情况下使用ReentrantLock-s。
- `LinkedTransferQueue`：基于链表的无界队列。除了通常的队列操作，它还有一系列的transfer方法，可以让生产者直接给等待的消费者传递信息，这样就不用将元素存储到队列中了。这是一个基于CAS操作的无锁集合。
- `PriorityBlockingQueue`：PriorityQueue的无界的版本。
- `SynchronousQueue`：一个有界队列，其中没有任何内存容量。这就意味着任何插入操作必须等到响应的取出操作才能执行，反之亦反。如果不需要Queue接口的话，通过Exchanger类也能完成响应的功能。

#### Maps

- `ConcurrentHashMap`：get操作全并发访问，put操作可配置并发操作的哈希表。并发的级别可以通过构造函数中concurrencyLevel参数设置（默认级别16）。该参数会在Map内部划分一些分区。在put操作的时候只有只有更新的分区是锁住的。这种Map不是代替HashMap的线程安全版本——任何 get-then-put的操作都需要在外部进行同步。
- `ConcurrentSkipListMap`：基于跳跃列表（Skip List）的ConcurrentNavigableMap实现。本质上这种集合可以当做一种TreeMap的线程安全版本来使用。

#### Sets

- `ConcurrentSkipListSet`：使用 ConcurrentSkipListMap来存储的线程安全的Set。
- `CopyOnWriteArraySet`：使用CopyOnWriteArrayList来存储的线程安全的Set。

#### 相关阅读

- [Java 基本类型集合库：Trove](http://java-performance.info/primitive-types-collections-trove-library/)：Trove库概述——存储Java基本类型数据的集合库（与大多数JDK中的Objects类不同）。
- [Java常见数据类型内存占用（1）](http://java-performance.info/memory-consumption-of-java-data-types-1/)：各种类的内存占用会所名，包括enums、EnumMap、EnumSet、BitSet、ArrayList、LinkedList和ArrayDeque。
- [Java常见数据类型内存占用（2）](http://java-performance.info/memory-consumption-of-java-data-types-2/)：HashMap、HashSet、LinkedHashMap、LinkedHashSet、TreeMap、TreeSet和JDK 7 PriorityQueue内存占用，以及对应的Trove替代类说明。

#### 推荐阅读

如果想要了解更多关于Java集合的知识，推荐阅读以下书籍：

- [Cay S. Horstmann. Core Java Volume I–Fundamentals (9th Edition) (Core Series)](http://www.amazon.com/gp/product/0137081898/ref=as_li_ss_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=0137081898&linkCode=as2&tag=javaperfor07e-20) ：第13章描述了Java集合.
- [Naftalin, Wadler. Java Generics and Collections](http://www.amazon.com/gp/product/0596527756/ref=as_li_ss_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=0596527756&linkCode=as2&tag=javaperfor07e-20)：书的第2部分（第10章至第17章）专门讨论了Java集合。
- [Goetz, Peierls, Bloch, Bowbeer, Holmes, Lea. Java Concurrency in Practice](http://www.amazon.com/gp/product/0321349601/ref=as_li_ss_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=0321349601&linkCode=as2&tag=javaperfor07e-20)：最好的Java并发书籍。第5章讨论了Java 1.5引入的并发集合。

### 总结

|                 | 单线程                                                       | 并发                                                         |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Lists           | `ArrayList`——基于泛型数组`LinkedList`——不推荐使用`Vector`——已废弃（deprecated） | `CopyOnWriteArrayList`——几乎不更新，常用来遍历               |
| Queues / deques | `ArrayDeque`——基于泛型数组`Stack`——已废弃（deprecated）`PriorityQueue`——读取操作的内容已排序 | `ArrayBlockingQueue`——有界的阻塞式队列`ConcurrentLinkedDeque / ConcurrentLinkedQueue`——无界的链表队列（CAS）`DelayQueue`——元素带有延迟功能的队列`LinkedBlockingDeque / LinkedBlockingQueue`——链表阻塞队列，可设定是否带边界`LinkedTransferQueue`——可将元素`transfer`进行w/o存储`PriorityBlockingQueue`——并发`PriorityQueue``SynchronousQueue`——使用`Queue`接口进行`Exchanger` |
| Maps            | `HashMap`——通用Map`EnumMap`——键使用`enum``Hashtable`——已废弃（deprecated）`IdentityHashMap`——键使用`==`进行比较`LinkedHashMap`——保持插入顺序`TreeMap`——键已排序`WeakHashMap`——适用于缓存（cache） | `ConcurrentHashMap`——通用并发Map`ConcurrentSkipListMap`——已排序的并发Map |
| Sets            | `HashSet`——通用set`EnumSet`——`enum` Set`BitSet`——比特或密集的整数Set`LinkedHashSet`——保持插入顺序`TreeSet`——排序Set |                                                              |



## 线程创建

一般来讲线程创建有四种方式:

1. 继承Thread

2. 实现Runnable接口

3. 实现Callable接口，结合 FutureTask使用

4. 利用该线程池ExecutorService、Callable、Future来实现

   

   参考

   <https://my.oschina.net/u/566591/blog/1576410> 

   <https://cloud.tencent.com/developer/article/1038547> 





## 线程状态

![img](https://upload-images.jianshu.io/upload_images/3167361-a54a03c7b1180c12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/503) 

![img](https://upload-images.jianshu.io/upload_images/3167361-6790c4c3def60448.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)



参考： https://www.jianshu.com/p/d7c87eca472a

### **2.3.1、线程状态**

Java语言定义了5种线程状态，如下（同色块属于同一种状态）：

 ![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170718214407365-1738494708.png)

1、**新建**（New） ：创建后尚未通过start()启动的线程处于此状态。

2、**可运行**（Runable）：Runable包括了OS线程状态中的Running和Ready，即处于Runable状态的线程可能正在执行，也可能处于就绪状态等待分配CPU。

3、**等待**：此状态的线程不会被分配CPU执行时间，等待被唤醒或等待倒计时到了后自动唤醒。分为无限等待和限时等待（wait和sleep的一个区别是前者会释放对象锁后者不会）：

无限等待（Waiting）：此状态的线程不会被分配CPU执行时间，需要等待被其他线程显示唤醒。

以下方法让线程进入此状态：无时间参数的方法 obj.wait()、threadObj.join()、threadObj.join(0)、LockSupport.park() 。

限时等待（Timed Waiting）：此状态的线程也不会被分配CPU执行时间，但过一定时间后由系统自动唤醒而不用经由其他线程显示唤醒。

以下方法让线程进入此状态：有时间参数的方法 obj.wait(long timeout)、threadObj.join(long timeout)、LockSupport.parkNanos(long nanos)、LockSupport.parkUntil(long deadline)、threadObj.sleep(long timeout) 。

LockSupport的park、unpark用于挂起或恢复线程，其底层最终是调用了Unsafe类的park、unpark native方法。

4、**阻塞**（Blocking）：此状态的线程不会被分配CPU执行时间，在等待获取一个排它锁，在占有此锁的另一个线程释放该锁时此线程将结束阻塞。如程序等待进入同步区域时（如synchronized块）线程将进入此状态。

5、**结束**（tTerminated）：线程执行结束已终止，处于此状态。

### **2.3.2、线程状态转换**

详细的状态转换如下：（wait是Object的实例方法，调用时会释放持有的对象锁，其他则不会。各方法的区别可见 [join、sleep、wait、notify等的区别-MarchOn](http://www.cnblogs.com/z-sm/p/6481268.html)）

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170718215006693-574511681.png)

copy from：http://www.cnblogs.com/z-sm/p/7201822.html



## 线程优先级

Java 线程优先级使用 1 ~ 10 的整数表示：

- 最低优先级 1：`Thread.MIN_PRIORITY`
- 最高优先级 10：`Thread.MAX_PRIORITY`
- 普通优先级 5：`Thread.NORM_PRIORITY`



获取设置优先级：

​    System.out.println(Thread.currentThread().getPriority());   

​    Thread.currentThread().setPriority(Thread.MIN_PRIORITY);   



默认优先级：

**Java 默认的线程优先级是父线程的优先级，而非普通优先级Thread.NORM_PRIORITY**，因为主线程默认优先级是普通优先级`Thread.NORM_PRIORITY`，所以如果不主动设置线程优先级，则新创建的线程的优先级就是普通优先级`Thread.NORM_PRIORITY`.



优先级调度：

高优先级的线程比低优先级的线程有更高的几率得到执行，实际上这和操作系统及虚拟机版本相关，有可能即使设置了线程的优先级也不会产生任何作用。



最大优先级：

我们可以设定线程组的最大优先级，当创建属于该线程组的线程时，新线程的优先级不能超过这个最大优先级

- 系统线程组的最大优先级默认为 `Thread.MAX_PRIORITY`
- 创建线程组时最大优先级默认为父线程组（如果未指定父线程组，则其父线程组默认为当前线程所属线程组）的最大优先级
- 可以通过 `setPriority` 更改最大优先级，但无法超过父线程组的最大优先级



copy from :https://blog.csdn.net/silent_paladin/article/details/54561496



## 线程InterruptException异常

 当一个方法后面声明可能会抛出InterruptedException 异常时，说明该方法是可能会花一点时间，但是可以取消的方法。

 

抛InterruptedException的代表方法有：

\1. java.lang.Object 类的 wait 方法

\2. java.lang.Thread 类的 sleep 方法

\3. java.lang.Thread 类的 join 方法

 

-- 需要花点时间的方法

执行wait方法的线程，会进入等待区等待被notify/notify All。在等待期间，线程不会活动。

执行sleep方法的线程，会暂停执行参数内所设置的时间。

执行join方法的线程，会等待到指定的线程结束为止。

因此，上面的方法都是需要花点时间的方法。

 

-- 可以取消的方法

因为需要花时间的操作会降低程序的响应性，所以可能会取消/中途放弃执行这个方法。

这里主要是通过interrupt方法来取消。

 

\1. sleep方法与interrupt方法

interrupt方法是Thread类的实例方法，在执行的时候并不需要获取Thread实例的锁定，任何线程在任何时刻，都可以通过线程实例来调用其他线程的interrupt方法。

当在sleep中的线程被调用interrupt方法时，就会放弃暂停的状态，并抛出InterruptedException异常，这样一来，线程的控制权就交给了捕捉这个异常的catch块了。

 

\2. wait方法和interrupt方法

当线程调用wait方法后，线程在进入等待区时，会把锁定接触。当对wait中的线程调用interrupt方法时，会先重新获取锁定，再抛出InterruptedException异常，获取锁定之前，无法抛出InterruptedException异常。

 

\3. join方法和interrupt方法

当线程以join方法等待其他线程结束时，一样可以使用interrupt方法取消。因为join方法不需要获取锁定，故而与sleep一样，会马上跳到catch程序块

 

-- interrupt方法干了什么？

interrupt方法其实只是改变了中断状态而已。

而**sleep、wait和join这些方法的内部会不断的检查中断状态的值，从而自己抛出InterruptEdException。**

所以，**如果在线程进行其他处理时，调用了它的interrupt方法，线程也不会抛出InterruptedException的，所以如果要使用该方法终止线程，那么需要while(true)来检测线程中断状态**，只有当线程走到了sleep, wait, join这些方法的时候，才会抛出InterruptedException。若是没有调用sleep, wait, join这些方法，或者没有在线程里自己检查中断状态，自己抛出InterruptedException，那InterruptedException是不会抛出来的。

 

isInterrupted方法，可以用来检查中断状态

Thread.interrupted方法，可以用来检查并清除中断状态。



copyfrom： https://blog.csdn.net/derekjiang/article/details/4845757

https://www.ibm.com/developerworks/cn/java/j-jtp05236.html



线程取消或终止

方法一：设置某个“已请求取消”标志，任务定期检查该标志。

```java
    private volatile boolean cancelled;

    public void run() {
        while(!cancelled) {

        }
    }

```

方法二： 中断Interrupt

```
@overide
public void run() {
        try {
            BigInteger p = BigInteger.ONE;
            while (!Thread.currentThread().isInterrupted()) {
                queue.put(p = p.nextProbablePrime());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
void cancel() {
    Thread.currentThread().interrupt();
}
```



方法三：通过Future来实现取消 

方法四： 线程池关闭。

`shutdown`:

1. 调用之后不允许继续往线程池内继续添加线程;
2. 线程池的状态变为`SHUTDOWN`状态; 
3. 所有在调用`shutdown()`方法之前提交到`ExecutorSrvice`的任务都会执行;
4. 一旦所有线程结束执行当前任务，`ExecutorService`才会真正关闭。

`shutdownNow()`:

1. 该方法返回尚未执行的 task 的 List;
2. 线程池的状态变为`STOP`状态; 
3. 阻止所有正在等待启动的任务, 并且停止当前正在执行的任务;

 `awaitTermination`：

调用`awaitTermination`设置定时任务，代码内的意思为 2s 后检测线程池内的线程是否均执行完毕（就像老师告诉学生，“最后给你 2s 钟时间把作业写完”），若没有执行完毕，则调用`shutdownNow()`方法。

 

简单点来说，就是:
`shutdown()`调用后，不可以再 submit 新的 task，已经 submit 的将继续执行
`shutdownNow()`调用后，试图停止当前正在执行的 task，并返回尚未执行的 task 的 list

参考：https://hacpai.com/article/1488023925829



## 线程间通信 

wait/notify

join, 

threadlocal,

inheritableThreadLocal(主线程和子线程)  

**管道通信**就是使用java.io.PipedInputStream 和 java.io.PipedOutputStream进行通信

 

<https://blog.csdn.net/justloveyou_/article/details/54929949>
<https://www.cnblogs.com/hapjin/p/5492619.html>
<https://blog.csdn.net/u011514810/article/details/77131296>

 

##  Thread Pool线程池

 既然Android中线程池来自于Java，那么研究Android线程池其实也可以说是研究Java中的线程池

在Java中，线程池的概念是Executor这个接口，具体实现为ThreadPoolExecutor类，学习Java中的线程池，就可以直接学习他了

对线程池的配置，就是对ThreadPoolExecutor构造函数的参数的配置，既然这些参数这么重要，就来看看构造函数的各个参数吧。



### ThreadPoolExecutor提供了四个构造函数

```
//五个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue)

//六个参数的构造函数-1
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory)

//六个参数的构造函数-2
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          RejectedExecutionHandler handler)

//七个参数的构造函数
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

**我知道你看到这些构造函数和我一样也是吓呆了，但其实一共就7种类型，理解起来简直和理解一周有7天一样简单，而且一周有两天是周末，其实也就只有5天需要了解！相信我，毕竟扯皮，我比较擅长**

- **int corePoolSize** => 该线程池中**核心线程数最大值**

  > **核心线程：**
  >
  > 线程池新建线程的时候，如果当前线程总数小于corePoolSize，则新建的是核心线程，如果超过corePoolSize，则新建的是非核心线程
  >
  > 核心线程默认情况下会一直存活在线程池中，即使这个核心线程啥也不干(闲置状态)。
  >
  > 如果指定ThreadPoolExecutor的allowCoreThreadTimeOut这个属性为true，那么核心线程如果不干活(闲置状态)的话，超过一定时间(时长下面参数决定)，就会被销毁掉
  >
  > 很好理解吧，正常情况下你不干活我也养你，因为我总有用到你的时候，但有时候特殊情况(比如我自己都养不起了)，那你不干活我就要把你干掉了

- **int maximumPoolSize**

  > 该线程池中**线程总数最大值**
  >
  > 线程总数 = 核心线程数 + 非核心线程数。核心线程在上面解释过了，这里说下非核心线程：
  >
  > 不是核心线程的线程(别激动，把刀放下…)，其实在上面解释过了

- **long keepAliveTime**

  > 该线程池中**非核心线程闲置超时时长**
  >
  > 一个非核心线程，如果不干活(闲置状态)的时长超过这个参数所设定的时长，就会被销毁掉
  >
  > 如果设置allowCoreThreadTimeOut = true，则会作用于核心线程

- **TimeUnit unit**

  > keepAliveTime的单位，TimeUnit是一个枚举类型，其包括：
  >
  > 1. NANOSECONDS ： 1微毫秒 = 1微秒 / 1000
  > 2. MICROSECONDS ： 1微秒 = 1毫秒 / 1000
  > 3. MILLISECONDS ： 1毫秒 = 1秒 /1000
  > 4. SECONDS ： 秒
  > 5. MINUTES ： 分
  > 6. HOURS ： 小时
  > 7. DAYS ： 天

- **BlockingQueue workQueue**

  > 该线程池中的任务队列：维护着等待执行的Runnable对象
  >
  > 当所有的核心线程都在干活时，新添加的任务会被添加到这个队列中等待处理，如果队列满了，则新建非核心线程执行任务
  >
  > 常用的workQueue类型：
  >
  > 1. **SynchronousQueue：**这个队列接收到任务的时候，会直接提交给线程处理，而不保留它，如果所有线程都在工作怎么办？那就新建一个线程来处理这个任务！所以为了保证不出现<线程数达到了maximumPoolSize而不能新建线程>的错误，使用这个类型队列的时候，maximumPoolSize一般指定成Integer.MAX_VALUE，即无限大
  > 2. **LinkedBlockingQueue：**这个队列接收到任务的时候，如果当前线程数小于核心线程数，则新建线程(核心线程)处理任务；如果当前线程数等于核心线程数，则进入队列等待。由于这个队列没有最大值限制，即所有超过核心线程数的任务都将被添加到队列中，这也就导致了maximumPoolSize的设定失效，因为总线程数永远不会超过corePoolSize
  > 3. **ArrayBlockingQueue：**可以限定队列的长度，接收到任务的时候，如果没有达到corePoolSize的值，则新建线程(核心线程)执行任务，如果达到了，则入队等候，如果队列已满，则新建线程(非核心线程)执行任务，又如果总线程数到了maximumPoolSize，并且队列也满了，则发生错误
  > 4. **DelayQueue：**队列内元素必须实现Delayed接口，这就意味着你传进去的任务必须先实现Delayed接口。这个队列接收到任务时，首先先入队，只有达到了指定的延时时间，才会执行任务

- **ThreadFactory threadFactory**

  > 创建线程的方式，这是一个接口，你new他的时候需要实现他的`Thread newThread(Runnable r)`方法，一般用不上，**这是星期六，休息**
  >
  > 但我还是说一句吧(把枪放下…)
  >
  > 小伙伴应该知道AsyncTask是对线程池的封装吧？那就直接放一个AsyncTask新建线程池的threadFactory参数源码吧：
  >
  > ```
  > new ThreadFactory() {
  >     private final AtomicInteger mCount = new AtomicInteger(1);
  > 
  >     public Thread new Thread(Runnable r) {
  >         return new Thread(r,"AsyncTask #" + mCount.getAndIncrement());
  >     }
  > }
  > ```
  >
  > 这么简单？就给线程起了个名？！对啊，所以说这是星期六啊，别管他了，虽然我已经强迫你们看完了…

- **RejectedExecutionHandler handler**

  > 这玩意儿就是抛出异常专用的，比如上面提到的两个错误发生了，就会由这个handler抛出异常，你不指定他也有个默认的
  >
  > 抛异常能抛出什么花样来？所以这个星期天不管了，一边去，根本用不上

新建一个线程池的时候，一般只用5个参数的构造函数。

### 向ThreadPoolExecutor添加任务

那说了这么多，你可能有疑惑，我知道new一个ThreadPoolExecutor，大概知道各个参数是干嘛的，可是我new完了，怎么向线程池提交一个要执行的任务啊？

通过`ThreadPoolExecutor.execute(Runnable command)`方法即可向线程池内添加一个任务

### ThreadPoolExecutor的策略

上面介绍参数的时候其实已经说到了ThreadPoolExecutor执行的策略，这里给总结一下，当一个任务被添加进线程池时：

1. 线程数量未达到corePoolSize，则新建一个线程(核心线程)执行任务
2. 线程数量达到了corePools，则将任务移入队列等待
3. 队列已满，新建线程(非核心线程)执行任务
4. 队列已满，总线程数又达到了maximumPoolSize，就会由上面那位星期天(RejectedExecutionHandler)抛出异常

### 常见四种线程池

如果你不想自己写一个线程池，那么你可以从下面看看有没有符合你要求的(一般都够用了)，如果有，那么很好你直接用就行了，如果没有，那你就老老实实自己去写一个吧

Java通过Executors提供了四种线程池，这四种线程池都是直接或间接配置ThreadPoolExecutor的参数实现的，下面我都会贴出这四种线程池构造函数的源码，各位大佬们一看便知！

来，走起：

#### CachedThreadPool()

可缓存线程池：

1. 线程数无限制
2. 有空闲线程则复用空闲线程，若无空闲线程则新建线程
3. 一定程序减少频繁创建/销毁线程，减少系统开销

创建方法：

`ExecutorService cachedThreadPool = Executors.newCachedThreadPool();`

源码：

```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

通过我上面**行云流水谈笑风生天马行空滔滔不绝**的对各种参数的说明，这个源码你肯定一眼就看懂了，想都不用想(下面三种一样啦)

#### FixedThreadPool()

定长线程池：

1. 可控制线程最大并发数（同时执行的线程数）
2. 超出的线程会在队列中等待

创建方法：

```
//nThreads => 最大线程数即maximumPoolSize
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(int nThreads);

//threadFactory => 创建线程的方法，这就是我叫你别理他的那个星期六！你还看！
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(int nThreads, ThreadFactory threadFactory);
```

源码：

```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

2个参数的构造方法源码，不用我贴你也知道他把星期六放在了哪个位置！所以我就不贴了，省下篇幅给我扯皮

#### ScheduledThreadPool()

定长线程池：

1. 支持定时及周期性任务执行。

创建方法：

```
//nThreads => 最大线程数即maximumPoolSize
ExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(int corePoolSize);
```

源码：

```
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

//ScheduledThreadPoolExecutor():
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE,
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue());
}
```

#### SingleThreadExecutor()

单线程化的线程池：

1. 有且仅有一个工作线程执行任务
2. 所有任务按照指定顺序执行，即遵循队列的入队出队规则

创建方法：

`ExecutorService singleThreadPool = Executors.newSingleThreadPool();`

源码：

```
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

还有一个`Executors.newSingleThreadScheduledExecutor()`结合了3和4，就不介绍了，基本不用。



copy from : https://liuzho.github.io/2017/04/17/%E7%BA%BF%E7%A8%8B%E6%B1%A0%EF%BC%8C%E8%BF%99%E4%B8%80%E7%AF%87%E6%88%96%E8%AE%B8%E5%B0%B1%E5%A4%9F%E4%BA%86/



## 并发控制策略（Concurrency Strategies）

以对tree的并发读写为例：

1、lock-free solution

copy-on-write：确定要write的node后copy该node并对copied node做write操作，然后以原子更新方式替换原node。**最多允许一个write和多个read同时access a tree**，因为多个write同时写同一node时最后一个write操作会覆盖其他write操作。

test-and-set、compare-and-swap等原子操作，由硬件指令提供原子保证。

2、lock-based solution

coarse-grained lock：以tree为粒度进行加锁

mutex lock：排它锁，如synchronized。**write和read都加锁，最多允许一个write或一个read access a tree**。

read-write lock：读写锁，如ReentrantReadWrite lock。**write和read都加锁，最多允许一个write或多个read access a tree**。

fine-grained lock：以tree node为粒度进行加锁。可采用排它锁或读写锁。**write和read都加锁，最多允许一个write access a node，此时该node能否被read access视采用的锁而定**。

hand-over-hand lock：必须先锁住父节点才能锁住子节点，然后释放父节点锁并获取子节点的子节点的锁，以此方式从上到下直到锁住要write的节点。被锁节点的子树无法被其他write access，即使write的是a different node。

optimistic lock：先找到要write的node，然后对node的父node加锁，再对该node加锁。允许其他write access被锁节点的子树。

hybrid solution：综合copy-on-write和fine-grained-lock，**write加锁（对被加锁加点采用copy-on-write方式进行write）、read不加锁，最多允许多个write和多个read access a node。**

总结：

![点击全屏显示](https://images2017.cnblogs.com/blog/568153/201708/568153-20170805171823319-1786570161.png)

 

## Java的线程安全实现

Java中的并发正确性保障手段主要有以下几类。

1、阻塞同步（互斥同步）：悲观并发策略、重量级锁。认为共享数据一定存在访问竞争而总进行加锁。

同步和互斥的区别：同步是指多线程并发访问共享数据时，保证共享数据同一时刻只被一个（或一些，使用信号量时）线程使用 。互斥是实现同步的手段，实现方式包括临界区、互斥量、信号量等。

Java中的互斥同步手段：有[**synchronized**](http://www.cnblogs.com/z-sm/p/6502251.html)、java.util.concurrent.locks.**ReentrantLock**（重入锁）等。都是互斥锁，区别：前者是Java语言本身提供的；后者则不是，而是类库实现。后者增加了 **等待可中断、可实现公平锁、锁可绑定多个条件** 3个功能。JDK1.6后两者性能持平，优先使用synchronized。

优缺点：使用范围广。总进行加锁、线程阻塞唤醒需要用户态核心态转换、维护锁计数器等操作，特别是阻塞增加了性能消耗。**互斥同步对性能最大的影响是阻塞的实现，挂起和恢复线程的操作都需要转内内核完成，频繁切换影响OS并发性能。**

 

2、非阻塞同步：乐观并发策略、轻量级锁。先操作，检测到冲突时再补偿，硬件指令保证 操作和检测 两组合步骤的原子性。不是为了替代重量级锁，而是在没有多线程竞争时减少传统的重量级锁使用OS互斥量带来的性能消耗。

基于冲突检测：先进行操作，若没有其他线程争用共享数据则操作成功，否则进行冲突补偿措施如重试。有Test-and-Set、Fetch-and-Increment、Swqp、Compare-and-Swap、Load-Linked/Store-Conditional等。以Test-and-Set为例：

![点击全屏显示](https://images2015.cnblogs.com/blog/568153/201707/568153-20170719234528755-1624135205.png)

Java中的非阻塞同步手段：JDK1.5起提供的CAS操作——sun.misc.Unsafe类的compareAndSwapInt()、compareAndSwapLong()等方法。借助之，可以**原子更新基本数据类型**、**原子更新引用类型**、**原子更新数组元素**、**原子更新属性值**（详见[Java并发包中的原子操作类](https://blog.csdn.net/javazejian/article/details/72772470)）。CAS操作可以认为就是一种自旋锁。

优缺点：性能较高，不用挂起线程。**存在ABA问题**（可改用互斥同步解决、也可加时间戳解决）、不可重入。

 

3、非同步方案：保证线程安全并不一定要同步，不涉及共享数据的方法本身就是线程安全的，有很多，如：

**可重入代码**：不依赖存储在堆上的数据和共用的系统资源、用到的状态量都由参数传入、不调用field可重入方法等的代码。判断可重入的原则：若一个方法的返回结果是可预测的，只要输入相同数据就能返回相同结果，则是可重入的。

**线程本地存储**：把共享数据的可见范围限定在本线程内达到线程安全。如**java.lang.ThreadLocal类**，原理：每个线程都有一个Map，类型为ThreadLocalMap，Map的key为ThreadLocal变量的hash值，Value为本线程设置的该ThreadLocal变量的值。

## thread local

每个线程维护一个`ThreadLocal.ThreadLocalMap threadLocals = null;`线程私有变量来实现，当调用set方法如下：

```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}复制代码
```

set(T value) 方法中，首先获取当前线程，然后在获取到当前线程的 ThreadLocalMap，如果 ThreadLocalMap 不为 null，则将 value 保存到 ThreadLocalMap 中，并用当前 ThreadLocal 作为 key；否则创建一个 ThreadLocalMap 并给到当前线程，然后保存 value。

ThreadLocalMap 相当于一个 HashMap，是真正保存值的地方。

 

<https://juejin.im/post/5965ef1ff265da6c40737292>

## lock和lockInterruptly

lock方法会忽略中断请求，继续获取锁直到成功；而lockInterruptibly则直接抛出中断异常来立即响应中断，由上层调用者处理中断
<https://blog.csdn.net/wojiushiwo945you/article/details/42387091>

## thread.join = object.wait() = condition.await()

均会释放锁。
均需要一个锁互斥变量控制。join默认thread对象锁，object对象锁，condition归属的lock锁。
thead.sleep()不释放锁
最好使用循环判断条件包围wait操作。
wait/await操作均会被打断

## Timer的操作

TimerTask进入Timer后是需要排队执行的。
new Timer(true)那么当附属线程结束后，这个timer也会被销毁。

## readResolve方法和序列化

<https://blog.csdn.net/haydenwang8287/article/details/5964130>