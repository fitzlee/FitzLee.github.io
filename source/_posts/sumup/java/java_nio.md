---
title: java nio源码
urlname: java_jdk_base_nio
#date: 2016-03-06
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 00Java&JDK
tags:
    - java
    - jdk
    - nio
---

## 同步异步阻塞非阻塞

 一般来说 I/O 模型可以分为：**同步阻塞，同步非阻塞，异步阻塞，异步非阻塞 四种IO模型**

###     **同步阻塞 IO ：**

   在此种方式下，用户进程在发起一个 IO 操作以后，必须等待 IO 操作的完成，只有当真正完成了 IO 操作以后，用户进程才能运行。 JAVA传统的 IO 模型属于此种方式！

 

###     **同步非阻塞 IO:**

在此种方式下，用户进程发起一个 IO 操作以后 边可 返回做其它事情，但是用户进程需要时不时的询问 IO 操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的 CPU 资源浪费。其中目前 JAVA 的 NIO 就属于同步非阻塞 IO 。

 

###     **异步阻塞 IO ：**

   **此种方式下是指应用发起一个 IO 操作以后，不等待内核 IO 操作的完成，等内核完成 IO 操作以后会通知应用程序，这其实就是同步和异步最关键的区别，同步必须等待或者主动的去询问 IO 是否完成，**那么为什么说是阻塞的呢？因为此时是通过 select 系统调用来完成的，而 select 函数本身的实现方式是阻塞的，**而采用 select 函数有个好处就是它可以同时监听多个文件句柄，从而提高系统的并发性！**



###    **异步非阻塞 IO:**

   在此种模式下，用户进程只需要发起一个 IO 操作然后立即返回，等 IO 操作真正的完成以后，应用程序会得到 IO 操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的 IO 读写操作，因为 真正的 IO读取或者写入操作已经由 内核完成了。目前 Java 中还没有支持此种 IO 模型。



### 总结：

JAVA NIO是同步非阻塞io。同步和异步说的是消息的通知机制，阻塞非阻塞说的是线程的状态 。下面说说我的理解，
client和服务器建立了socket连接：
1、同步阻塞io：client在调用read（）方法时，stream里没有数据可读，线程停止向下执行，直至stream有数据。阻塞：体现在这个线程不能干别的了，只能在这里等着同步：是体现在消息通知机制上的，即stream有没有数据是需要我自己来判断的。

2、同步非阻塞io：调用read方法后，如果stream没有数据，方法就返回，然后这个线程就就干别的去了。非阻塞：体现在，这个线程可以去干别的，不需要一直在这等着同步：体现在消息通知机制，这个线程仍然要定时的读取stream，判断数据有没有准备好，client采用循环的方式去读取，可以看出CPU大部分被浪费了

3、异步非阻塞io：服务端调用read()方法，若stream中无数据则返回，程序继续向下执行。当stream中有数据时，操作系统会负责把数据拷贝到用户空间，然后通知这个线程，这里的消息通知机制就是异步！而不是像NIO那样，自己起一个线程去监控stream里面有没有数据！这是我的理解，不确定对不对



#### 2.1 blocking IO（阻塞）

我和女友点完餐后，不知道什么时候能做好，只好坐在餐厅里面等，直到做好，然后吃完才离开。

女友本想还和我一起逛街的，但是不知道饭能什么时候做好，只好和我一起在餐厅等，而不能去逛街，直到吃完饭才能去逛街，中间等待做饭的时间浪费掉了。这就是典型的阻塞。网络中IO阻塞如下图所示：

![img](//upload-images.jianshu.io/upload_images/5219651-18046f9283e83e81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

blocking IO（阻塞）

当用户进程调用了recvfrom这个系统调用，内核就开始了IO的第一个阶段：准备数据。对于网络IO来说，很多时候数据在一开始还没有到达（比如，还没有收到一个完整的UDP包），这个时候内核就要等待足够的数据到来。而在用户进程这边，整个进程会被阻塞。当内核一直等到数据准备好了，它就会将数据从内核中拷贝到用户内存，然后内核返回结果，用户进程才解除block的状态，重新运行起来。

所以，blocking IO的特点就是在IO执行的两个阶段都被block了。

#### 2.2 nonblocking IO（非阻塞）

我女友不甘心白白在这等，又想去逛商场，又担心饭好了。所以我们逛一会，回来询问服务员饭好了没有，来来回回好多次，饭都还没吃都快累死了啦。这就是非阻塞。需要不断的询问，是否准备好了。网络IO非阻塞如下图所示：

![img](//upload-images.jianshu.io/upload_images/5219651-6d76b69e74eb5984.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/642)

nonblocking IO（非阻塞）

从图中可以看出，当用户进程发出read操作时，如果内核中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦内核中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。

所以，用户进程其实是需要不断的主动询问内核数据好了没有。

#### 2.3 IO multiplexing（IO多路复用）

我一次性带了多个女友（女性朋友）都要在那里吃饭，然后我苦逼的在餐厅等着不断询问几个女友的饭是否有好了的，这时我的女友们就可以出去继续逛街，然后我一但发现某个女友的饭好了就打电话通知该女友来吃饭！

![img](//upload-images.jianshu.io/upload_images/5219651-b67963ba863e5258.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/663)

IO multiplexing（IO多路复用）

当用户进程调用了select，那么整个进程会被block，而同时，内核会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从内核拷贝到用户进程。

这个图和blocking IO的图其实并没有太大的不同，事实上，还更差一些。因为这里需要使用两个system call (select 和 recvfrom)，而blocking IO只调用了一个system call (recvfrom)。但是，用select的优势在于它可以同时处理多个connection。（多说一句。所以，如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。）

在IO multiplexing Model中，实际中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的process其实是一直被block的。只不过process是被select这个函数block，而不是被socket IO给block。

#### 2.4 asynchronous IO（异步）

女友不想逛街，又餐厅太吵了，回家好好休息一下。于是我们叫外卖，打个电话点餐，然后我和女友可以在家好好休息一下，饭好了送货员送到家里来。这就是典型的异步，只需要打个电话说一下，然后可以做自己的事情，饭好了就送来了。linux提供了AIO库函数实现异步，但是用的很少。目前有很多开源的异步IO库，例如libevent、libev、libuv。异步过程如下图所示：

![img](//upload-images.jianshu.io/upload_images/5219651-1521e6476caec4a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/656)

asynchronous IO（异步）

用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从内核的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，内核会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，内核会给用户进程发送一个signal，告诉它read操作完成了。

到目前为止，已经将四个IO Model都介绍完了。现在回过头来回答最初的那几个问题：blocking和non-blocking的区别在哪，synchronous IO和asynchronous IO的区别在哪。

**三、阻塞与非阻塞**

先回答最简单的这个：blocking vs non-blocking。前面的介绍中其实已经很明确的说明了这两者的区别。调用blocking IO会一直block住对应的进程直到操作完成，而non-blocking IO在内核还准备数据的情况下会立刻返回。

简单理解为需要做一件事能不能立即得到返回应答，如果不能立即获得返回，需要等待，那就阻塞了，否则就可以理解为非阻塞。详细区别如下图所示：

![img](//upload-images.jianshu.io/upload_images/5219651-ffeb97607c6b2728.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

**阻塞与非阻塞**

 

 

 

 

 



## Java NIO总结



本文译自[Jakob Jenkov的Java NIO](http://tutorials.jenkov.com/java-nio/index.html)。注意，并非逐字翻译，删除了原文中碎碎念的部分，有些地方也加入了自己的理解。

Java NIO，即Java New IO，是从Java 1.4开始引入的新的IO API，用以取代标准Java IO和网络API，当然，NIO的工作方式也与标准的IO API不同，有下面三个特点：

- Java NIO：通道（Channel）和缓冲区（Buffer）

  在标准的IO API中，使用的是字节流和字符流。在NIO中使用的是通道和缓冲区。数据总是会从通道读出到缓冲区中，或者从缓冲区写入通道中。

- Java NIO：非阻塞IO

  使用Java NIO可以实现非阻塞IO。例如，使用一个线程从通道读出数据到缓冲区中，在数据读出的同时，该线程还可以做其他事情。当数据全部读取到缓冲区中后，该线程可以继续处理它。反之，从缓冲区中将数据写入通道时也可以做相同的处理。

- Java NIO：选择器（Selector）

  选择器是Java NIO中的一个概念，指的是可以监视多个通道事件（如连接打开，数据到达等等）的一个对象。因此，单个线程可以通过使用选择器对多个通道的数据进行监视。

## Java NIO 概要

Java NIO有下面三个核心组件构成：

- 通道 Channel
- 缓冲区 Buffer
- 选择器 Selector

尽管Java NIO的类和组件有很多，但是我认为其核心API就是由通道，缓冲区和选择器组成的，而如Pipe和FileLock之类的其他组件，仅仅是为方便使用这三个核心组件而设计的工具类。因此，本章将只会关注这三个核心组件。

### 通道和缓冲区

通常，NIO的所有IO都是从一个通道开始的。通道和流(Stream)有点类似，数据可以 **从通道读出到缓冲区中，也可以从缓冲区写入到通道中**。图示说明如下：

[![Java NIO: 数据从通道读出到缓冲区，从缓冲区写入到通道](http://xintq.net/img/post/javanio-channel-buffer.png)](http://xintq.net/img/post/javanio-channel-buffer.png)Java NIO: 数据从通道读出到缓冲区，从缓冲区写入到通道

通道和缓冲区的具体类型有好几种。以下是Java NIO中通道的主要实现：

- `FileChanel`
- `DatagramChannel`
- `SocketChannel`
- `ServerSocketChannel`

如你所见，这些通道实现涵盖了UDP+TCP网络IO，以及文件IO。这些类都实现了一些有意思的接口，为简单起见，本章不会介绍这些接口。

在Java NIO中核心缓冲区的实现类如下：

- `ByteBuffer`
- `CharBuffer`
- `DoubleBuffer`
- `FloatBuffer`
- `IntBuffer`
- `LongBuffer`
- `ShortBuffer`

这些缓冲区涵盖了可以通过NIO发送的基本数据类型：

- `byte`
- `short`
- `int`
- `long`
- `float`
- `double`
- 以及`char`

除此之外，Java NIO中还有一个`MappedByteBuffer`实现，用来连接内存映射文件，本章也不做介绍。

### 选择器

选择器可以让一个线程处理多个通道。当应用程序打开多个连接（Channel），而每个连接的数据流量都很低时，就使用选择器比较方便了，比如聊天室服务器。

下图表示了如何使用一个线程通过选择器来处理三个通道：

[![Java NIO: 一个线程通过选择器来处理三个通道](http://xintq.net/img/post/javanio-selecor-3-channel.png)](http://xintq.net/img/post/javanio-selecor-3-channel.png)Java NIO: 一个线程通过选择器来处理三个通道

要使用选择器，需要先将通道注册到选择器中，之后就可以调用它的`select()`方法。该方法一直阻塞到所注册通道中某个事件就绪为止。一旦方法返回，线程就可以处理这些事件。这些事件包括有连接进来，数据接收等。

## Java NIO 通道

前文提到Java NIO 通道与流类似，但是有以下不同：

- 可以对通道进行读和写。而流只能是单向的，只能读或者只能写
- 通道可以异步读和写
- 通道中的数据总是要先读到一个缓冲区中，或者要从一个缓冲区中写入

请牢记这张图，数据可以 **从通道读出到缓冲区中，也可以从缓冲区写入到通道中**：

[![Java NIO: 数据从通道读出到缓冲区中，也可以从缓冲区写入到通道中](http://xintq.net/img/post/javanio-channel-buffer.png)](http://xintq.net/img/post/javanio-channel-buffer.png)Java NIO: 数据从通道读出到缓冲区中，也可以从缓冲区写入到通道中

### 通道的实现

Java NIO中最重要的通道实现如下：

- `FileChannel` 从文件读取数据，或将数据写入文件
- `DatagramChannel` 使用`UDP`协议通过网络来读写数据
- `SocketChannel` 使用`TCP`协议通过网络来读写数据
- `ServerSocketChannel` 像WEB服务器一样监听进来的TCP连接，对于每一个进来的连接都会创建一个`SocketChannel`

### 基本的通道举例

下面是一个使用`FileChannel`来读取数据到缓冲区的基本示例：

```
RandomAccessFile aFile = RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
while (bytesRead != -1) {
    System.out.println("\n=====Read " + bytesRead);
    buf.flip();

    while(buf.hasRemaining()){
        System.out.print((char) buf.get());
    }

    buf.clear();
    bytesRead = inChannel.read(buf);
}
aFile.close();
```

注意到`buf.flip()`调用。首先将数据读到缓冲区中，然后切换缓冲区为读模式，接着从缓冲区读数据。下一章会详细介绍这一过程。

## Java NIO 缓冲区

在与NIO的通道交互时，会用到Java NIO缓冲区。再次强调，数据可以 **从通道读出到缓冲区中，也可以从缓冲区写入到通道中**。

其实，缓冲区说白了就是一块内存区域，可以写入数据，之后再读取数据。只不过这块内存区域被包装成一个NIO的缓冲区对象，并提供了一些方法以便于对其进行操作。

### 基本的缓冲区使用方法

使用缓冲区读写数据时基本上需要以下四步：

1. 将数据写入到缓冲区中
2. 调用`buffer.flip()`方法
3. 将数据从缓冲区中读出
4. 调用`buffer.clear()`或者`buffer.compact()`方法

当往缓冲区里写入数据时，缓冲区会记录实际写入的数据量。一旦需要读出数据时，需要调用`flip()`方法将缓冲区从写模式切换为读模式。在读模式下，可以读出缓冲区中所有已写入的数据。

当所有数据读取完毕之后，需要清空缓冲区，以便准备下次写入。有两种方法：

- 调用`clear()`：清空全部缓冲区
- 或者调用`compact()`：清空缓冲区中已读数据的部分。未读数据会移动到缓冲区的开头位置，新数据会写在未读数据之后。

下面是一个简单的示例，在`write`，`flip`，`read`和`clear`部分给出了注释。

```
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

//创建一个容量为48字节的缓冲区
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf); //write：从通道中读出数据到缓冲区
while (bytesRead != -1) {
    buf.flip();  //flip: 将缓冲区切换到读模式
    while(buf.hasRemaining()){
      System.out.print((char) buf.get()); // read: 一次读取一字节
    }

    buf.clear(); //clear: 清空缓冲区，准备下次读入
    bytesRead = inChannel.read(buf);
}
aFile.close();
```

### 缓冲区三个重要属性： capacity, position和limit

之前提到，缓冲区其实就是一块内存区域，可以写入数据，之后再读取数据。只不过这块内存区域被包装成一个NIO的缓冲区对象，并提供了一些方法以便于对其进行操作。

想要了解缓冲区如何工作，就必须先理解它的三个重要属性，它们是：

- capacity 容量
- position 位置
- limit 上限

`position`和`limit`具体含义取决于缓冲区是在读模式还是写模式。`capacity`的含义固定的，表示缓冲区的固定大小，与缓冲区所在的模式无关。下图表示了在不同模式下这三个属性的含义：

[![Java NIO: 缓冲区属性](http://xintq.net/img/post/buffers-modes.png)](http://xintq.net/img/post/buffers-modes.png)Java NIO: 缓冲区属性

- 容量：capacity

  作为一块内存区域，缓冲区是有固定大小的，称之为`capacity`。也就是说只能在缓冲区中最多写入`capacity`这么多的byte，long，char类型的数据。一旦满了，就需要先清空（已读数据，或者全部清空）才能再往里面写入数据。就像往洗澡盆里放水一样。

- 位置：position

  当往缓冲期里写数据时，总是会从某个位置(`position`)开始。刚开始时，`position`是0。当写入一个byte或者long类型的数据后，`position`会指向下一个可写入的位置。显然，`postion`的最大值等于`capacity-1`。

  当从缓冲区中读取数据时，也总是会从某个位置开始。当缓冲区从写模式切换到读模式后，`postion`会被重置为0。当从缓冲区中读取数据时，`postion`会指向下一个可读取的位置。

- 上限：limit

  在写模式下，缓冲区的`limit`指的是可以往缓冲区里写入多少数据。即，写模式下`limit`和`capacity`的含义相同。

  当缓冲区切换到读模式后，`limit`指的是可以从缓冲区中读出多少数据。因此，当缓冲区切换到读模式后，`limit`会设置为写模式时`position`的值。换句话说，可以读取已写入的所有数据(`limit`被设置为已写入的字节数，该数值其实就是写模式下的`position`值)。

  上面的解释有点绕，还是拿洗澡盆举例：写模式下，`limit`就是洗澡盆的上沿高度；读模式下，`limit`就是洗澡盆中水的高度。

### 缓冲区类型

Java NIO中提供了以下几种缓冲区实现：

- `ByteBuffer`
- `MappedByteBuffer`
- `CharBuffer`
- `DoubleBuffer`
- `FloatBuffer`
- `IntBuffer`
- `LongBuffer`
- `ShortBuffer`

可以看到，缓冲区类型与不同的数据类型相对应。也就是说，缓冲区中的数据可以是以`char`， `short`，`int`，`long`， `float` 或者 `double`为单位的。

### 为缓冲区申请空间

想获取缓冲区就必须为其申请空间。每种类型的缓冲区都有`allocate()`方法。以下就是一个申请大小为 **48字节** 的`ByteBuffer`的示例：

```
ByteBuffer buf = ByteBuffer.allocate(48);
```

而下面是一个申请大小为 **1024个字符** （注意：单位不一样了哦！）的`CharBuffer`的示例：

```
CharBuffer buf = CharBuffer.allocate(1024);
```

### 像缓冲区写入数据

两种方法：

1. 将数据从通道写入到缓冲区

   ```
   int bytesRead = inChannel.read(buf); //读出到缓冲区中
   ```

2. 通过使用缓冲区的`put()`方法将数据直接写入到缓冲区

   ```
   buf.put(127);
   ```

   其实还有很多不同版本的`put()`方法，可以让你以不同的方式向缓冲区中写入数据，比如：向缓冲区中的指定位置写入数据，或者向缓冲区中写入一个byte类型的数组。具体可以参考相关的JavaDoC。

### flip()

`flip()`方法会将缓冲区从写模式切换为读模式，也就是将`limit`设置为`position`，然后重置`position`为0。

换言之，现在的`position`表示的就是当前读的位置，而`limit`则表示了在缓冲区中写入了多少字节或者字符等 - 还有多少字节或者字符可以读出。

### 从缓冲区中读取数据

也是两种方法：

1. 从缓冲区中读取数据写入到通道中

   ```
   int bytesWritten = inChannel.write(buf);
   ```

2. 使用`get()`方法，直接从缓冲区中读取数据

   ```
   byte aByte = buf.get();
   ```

   其实还有很多不同版本的`get()`方法，可以以不同的方式读取缓冲区的数据，比如：从缓冲区中的指定位置开始读出数据，或者从缓冲区中读取数据存放到byte类型的数组。具体可以参考相关的JavaDoC。

### rewind()

`rewind()`方法会将`position`重置为0，这样可以重复读取缓冲区中的所有数据。而`limit`值不变，仍然表示缓冲区中可以读出多少个元素（字节、字符等等）。

### clear() 和 compact()

缓冲区中的所有数据都读取完毕之后，就需要重置缓冲区以便再次写入。这时可以调用`clear()`方法或者`compact()`方法。

如果调用了`clear()`方法，`position`会重置为0，`limit`会设置为`capacity`，即，缓冲区会被清空。但是缓冲区中的数据不会被清除。只有这几个属性表示可以从哪里开始往缓冲区里写入数据。

如果缓冲区中有未读数据，而此时调用了`clear()`方法，那么这些未读数据就会被『遗忘』，意味着没有属性会标记哪部分数据已读，哪部分未读。

如果缓冲区中有未读数据，之后还要继续读出，但是此时要先写入一部分数据，这时就需要使用`compact()`方法，而不是`clear()`方法。

`compact()`方法会将所有未读数据复制到缓冲区开头，然后会将`position`设置为最后一个未读数据之后。而`limit`仍然会设置为`capacity`，就和`clear()`方法中的一样。现在，就可以继续往缓冲期写入而不会覆盖未读数据。

### mark() 和 reset()

通过调用`mark()`方法，可以在缓冲区中的指定位置做打个标签，随后可以通过调用`reset()`方法将缓冲区的`position`重置到打标签的位置。比如这样：

```
buffer.mark(); // 在当前位置打个标签

// 连续调用几次 buffer.get() 方法，让子弹飞一会儿。。。
// 比如：挨个儿解析每个元素

buffer.reset();  // 将position重置到打过标签的位置
```

### equals() 和 compareTo()

还可以使用`equals()` 和 `compareTo()`比较两个缓冲区。

- equals()

  两个缓冲区是相等的条件是：

  1. 有相同的数据类型（byte，char等）
  2. 在缓冲区中剩余的byte，char等个数相等
  3. 在缓冲区中所有剩余的byte，char都相同

  可以看出，`equals()`方法只比较缓冲区的一部分，而不是每一个元素。实际上，它只会比较缓冲区的剩余元素，其中剩余元素是指从`position`到`limit`之间的元素。

- compareTo()

  `compareTo()`会使用某种排序方法来比较两个缓冲区中的剩余元素（byte，char等），一个缓冲区比另外一个缓冲区小的条件是：

  1. 第一个不相等的元素小于另一个缓冲区中对应的元素
  2. 所有元素都相等，但是第一个缓冲区比另一个短（第一个缓冲区元素个数比另外一个少）

## Java NIO Scatter（分散）、Gather（聚集）

Java NIO内建了对Scatter（分散）和Gather（聚集）的支持。Scatter和Gather是从通道中读取数据，或者写入数据到通道时的概念。

从一个通道进行 **分散读取** 是指将通道中的数据读取到多个缓冲区中。也就是说，将数据从一个通道中『分散』到多个缓冲区中。**聚集写入** 到通道是指把多个缓冲区中的数据写入到一个通道中。也就是说，由多个缓冲区将数据『聚集』到一个通道中。

在需要对传输的数据进行分开处理的场合，Scatter和Gather会非常有用。比如，一份由消息头和消息体构成的数据，可能会需要将消息头和消息体分散地放在不同的缓冲区中，以便于分别处理他们。

### 分散读（Scattering Reads）

分散读就是从单个Channel中读取数据到多个Buffer中。分散读的原理如下图所示：

[![Java NIO: 分散读](http://xintq.net/img/post/javanio-scattering-reads.png)](http://xintq.net/img/post/javanio-scattering-reads.png)Java NIO: 分散读

下面的代码片段说明了如何执行分散读：

```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
```

注意到，多个缓冲区会首先组成一个数组，然后将缓冲区数组作为`channel.read()`方法的参数传入。接着`read()`方法开始按照缓冲区数组中每个缓冲区的顺序依次往里填充数据。当一个缓冲区写满之后，通道会继续向下一个缓冲区中填充。

在进行分散读时，总是会在填满一个缓冲区后再填充下一个，这就决定了这种方法不适用于处理大小不固定的消息。换言之，在消息头和消息体构成的数据中，如果消息头是固定大小的，比如128字节，那么就可以使用分散读来处理。

### 聚集写（Gathering Writes）

聚集写就是把数据从多个缓冲区中集中写到一个通道中。聚集写的原理如下图所示：

[![Java NIO: 聚集写](http://xintq.net/img/post/javanio-gathering-writes.png)](http://xintq.net/img/post/javanio-gathering-writes.png)Java NIO: 聚集写

下面的代码片段说明了如何执行聚集写：

```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//把数据写入各个缓冲区中

ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);
```

缓冲区数组会被传入到`write()`方法中，该方法会按照数组中每个缓冲区的顺序依次向通道中写入缓冲区的数据。注意，这里只会写入缓冲区中`position`和`limit`之间的数据。因此，对于一个大小为128字节的缓冲区，其中包含了58字节的数据，那么只会向通道中写入58字节。所以，与分散读相比，聚集写还可以处理大小不固定的消息。

## Java NIO 通道间的数据传递

在Java NIO中，如果有一个通道是`FileChannel`，那么数据可以从一个通道直接传递到另一个通道。在`FileChannel`中为此专门提供了`transferFrom()`方法和`transferTo()`方法。

### transferFrom()

`FileChannel.transferFrom()`方法会从源通道将数据传递到`FileChannel`。示例如下：

```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

toChannel.transferFrom(fromChannel, position, count);
```

参数`position`表示从指定位置开始往目标文件里面写入数据，`count`表示最多传输的数据个数。如果源通道中的数据少于`count`个字节，则实际传输的数据量少于请求的数据量。

此外，某些`SocketChannel`实现会只传输当前通道中已经就绪的数据，即使之后`SocketChannel`可能会到达更多的数据。因此，从`SocketChannel`向`FileChannel`中传递数据时，可能不会将请求的所有数据（`count`）都传递过去。

### transferTo()

`transferTo()`会从一个`FileChannel`向其它通道中传递数据。示例如下：

```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```

注意到这个示例与前面的示例非常相似，唯一不同的是调用了哪个`FileChannel`对象的方法，剩下的全部相同。

而对于`SocketChannel`来说，`transferTo()`方法也会有同样的问题。`SocketChannel`会一直接收来自`FileChannel`的数据，直到其目标缓冲区被填满才停止。

## Java NIO：选择器（Selector）

选择器是Java NIO中用以检查一个或者多个NIO通道的组件，决定了那个通道是可以读或者可以写。由此，单个线程可以管理多个通道，进而可以处理多个网络连接。

### 为什么使用选择器?

使用单线程处理多通道的好处是，处理通道时需要的线程更少了。事实上，可以只使用一个线程来处理所有的通道。对于操作系统来说，线程上下文切换的代价是很大的，而且每个线程也会占用一定的系统资源（比如内存）。因此，线程越少越好。

但是还需要记住一点，现在操作系统和CPU处理多任务的能力越来越强，多线程的开销也越来越小。实际上，对于多核CPU来说，没有多任务反而会浪费CPU资源。不管怎样，关于多任务还是单线程的讨论已经超出了本文的范围，这里只需要知道：你可以使用选择器来让单个线程管理多个通道就行了。

下图表示了如何使用一个线程通过选择器来处理3个通道：

[![Java NIO: 使用一个线程通过选择器来处理3个通道](http://xintq.net/img/post/javanio-selecor-3-channel.png)](http://xintq.net/img/post/javanio-selecor-3-channel.png)Java NIO: 使用一个线程通过选择器来处理3个通道

### 创建选择器

代码如下：

```
Selector selector = Selector.open();
```

### 将通道注册到选择器中

想要通过选择器来使用通道，必须先向选选择器中注册该通道。像这样：

```
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

通道只有在非阻塞模式下才能使用选择器。这就意味着`FileChannel`是不能使用选择器的，原因是`FileChannel`是无法切换到非阻塞模式下的，而基于Socket的通道是可以的。

注意`register()`方法的第二个参数。这是一个所谓的兴趣集合（interest set），意思是选择器对通道中的哪些事件感兴趣，以监听这类事件。可以监听的事件类型有四种：

1. Connect - 连接就绪
2. Accept - 建立连接
3. Read - 读就绪
4. Write - 写就绪

通道触发了一个事件，其实就是说这个事件已经就绪。因此，我们可以说一个已经成功连接到另外一个服务器的通道为『Connect』就绪状态。一个可以接受传入连接的服务器Socket通道为『Accept』就绪状态。一个有数据可以读取的通道为『Read』就绪状态。一个可以写入数据的通道为『Write』就绪状态。

这四种类型的事件用以下四个`SelectionKey`常量来表示：

1. `SelectionKey.OP_CONNECT`
2. `SelectionKey.OP_ACCEPT`
3. `SelectionKey.OP_READ`
4. `SelectionKey.OP_WRITE`

如果选择器对多个事件感兴趣，可以使用OR（或）操作将这些常量合并，像这样：

```
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

### SelectionKey

在前文中，在将通道注册到选择器时，调用了`register()`方法，返回值为`SelectionKey`对象。`SelectionKey`对象包含了几个有意思的属性：

- 兴趣集合（interest set）
- 就绪状态集合（ready set）
- 通道
- 选择器
- 附加对象 (可选)

下面详细介绍这些属性。

#### 兴趣集合（interest set）

兴趣集合（interest set）表示做 **选择** 时感兴趣的事件集合，正如前文 [将通道注册到选择器中](http://xintq.net/2017/06/12/everything-about-java-nio/#%E5%B0%86%E9%80%9A%E9%81%93%E6%B3%A8%E5%86%8C%E5%88%B0%E9%80%89%E6%8B%A9%E5%99%A8%E4%B8%AD) 的那样，可以通过`SelectionKey`来读写这个兴趣集合：

```
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;
```

显然，可以使用AND（与）操作来判断给定的集合中是否包含了某个事件。

#### 就绪状态集合（ready set）

就绪状态集合（ready set），顾名思义，就是指通道中已经就绪的操作集合。在做出『选择』之后，主要就是对就绪状态集合进行操作。关于『选择』会在随后进行介绍。可以使用下面的方法来访问该集合：

```
int readySet = selectionKey.readyOps();
```

也可以使用和兴趣集合同样的方法来检测就绪状态集合，看看当前通道有哪些事件、操作已经就绪了。此外，还可以使用下面四个方法，它们都会返回一个布尔型的值：

```
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

#### 通道 + 选择器

访问`SelectionKey`中的通道和选择器很简单，只需要：

```
Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();
```

#### 附加对象

可以在`SelectionKey`中附加一个对象，以便于识别给定通道，或者为通道附加更多信息。比如，附加与通道一起使用的缓冲区，或者附加包含更多聚集数据的对象。方法如下：

```
selectionKey.attach(theObject);
Object attachedObj = selectionKey.attachment();
```

当然，可以在注册到选择器的同时在register()方法中附加对象，像这样：

```
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

### 通过选择器『选择』通道

一旦在选择器中注册了一个或者几个通道之后，就可以调用`select()`方法。这些方法会返回你感兴趣的事件已经就绪状态的通道，比如Connect，Accept，Read 或者 Write。换言之，如果想选择可以读取数据的通道，`select()`方法会返回那些Read就绪的通道。`select()`方法有下面几种重载模式：

- `int select()` - 会被阻塞到所注册的事件就绪为止
- `int select(long timeout)` - 也会阻塞，但是可以设置阻塞超时时间（参数`timeout`）
- `int selectNow()` - 不会阻塞，会立即返回任何就绪的通道

`int`类型的返回值表示有多少个通道已经就绪，即，在最后一次调用`select()`方法后有多少通道已经就绪。假如调用了一次`select()`方法，返回了1，说明有一个通道已经就绪；此时再调用一次`select()`方法，又有一个Channel也变成就绪状态，则还是会返回1。如果不对第一次已经就绪的那个通道做任何处理的话，这时应该有两个通道是处于就绪状态的。但是在每次调用`select()`方法时，只会有一个通道变为就绪状态。

#### selectedKeys()

在调用`select()`方法后，其返回值说明有几个通道已经就绪，这时可以调用`selectedKeys()`方法，通过『已选择键集合』来访问这些通道，像这样：

```
Set<SelectionKey> selectedKeys = selector.selectedKeys();
```

当向选择器中使用`Channel.register()`方法注册通道时，会返回一个`SelectionKey`对象。该对象代表了注册到选择器中的通道。可以通过`SelectionKey`的`selectedKeySet()`方法来访问这些对象。

遍历『已选择键集合』就可以访问已经就绪的各个通道，如下所示：

```
Set<SelectionKey> selectedKeys = selector.selectedKeys();
Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // ServerSocketChannel可以接受连接
    } else if (key.isConnectable()) {
        // 已经连接到远程服务器

    } else if (key.isReadable()) {
        // 通道可以读取

    } else if (key.isWritable()) {
        // 通道可以写入
    }

    keyIterator.remove();
}
```

遍历『已选择键集合』中的每一个键值，判断每一个键值对应的状态集合，看看哪些事件已经就绪。

注意到在最后调用了`keyIterator.remove()`方法，在通道处理完成之后必须这样做的原因是选择器不会自己从已选择键集合中去掉`SelectionKey`实例。当下次通道就绪时，选择器会将它再次加入到已选择就绪集合中。

`SelectionKey.channel()`返回的通道实例需要强制转换成要处理的通道类型，比如`ServerSocketChannel`或者`SocketChannel`等。

### wakeUp()

对于一个在调用`select()`方法后被阻塞的线程，即使没有任何就绪的通道，也可以唤醒它，让它先离开`select()`方法。具体方法是，让另外一个线程来调用第一个线程所在选择器的`wakeUp()`方法。在`select()`方法中阻塞的线程会立即返回。

如果另外一个线程调用了`wakeUp()`方法，而目前没有阻塞在`select()`方法中的线程，那么下一个调用`select()`方法的线程会被立即『唤醒』。

### close()

在操作完选择器之后需要调用选择器的`close()`方法，来关闭选择器，并释放注册到该选择器上的所有的`SelectionKey`对象，而那些通道自身是不会被关闭的。

### 选择器的完整示例

这是一个完整的例子，打开选择器，向其注册通道（不包含通道的初始化），然后持续的监视选择器中的四个事件（Accept，Connect，Read，Write）的就绪状态。

```
Selector selector = Selector.open();
channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);

while(true) {
  int readyChannels = selector.select();
  if(readyChannels == 0) continue;

  Set<SelectionKey> selectedKeys = selector.selectedKeys();
  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

  while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // ServerSocketChannel可以接受连接
    } else if (key.isConnectable()) {
        // 已经连接到远程服务器
    } else if (key.isReadable()) {
        // 通道可以读取
    } else if (key.isWritable()) {
        // 通道可以写入
    }
    keyIterator.remove();
  }
}
```

## Java NIO 重要的通道实现

### FileChannel

Java NIO的`FileChannel`是连接到文件的通道，可以通过它读写文件。Java NIO的`FileChannel`可以替代标准的Java IO API来读写文件。

`FileChannel`不能设置为非阻塞模式，只能运行在阻塞模式下。

#### 打开FileChannel

在使用`FileChannel`之前必须先打开它。但是`FileChannel`是无法直接打开的，它需要通过`InputStream`，`OutputStream`，或者`RandomAccessFile`才能得到。如下：

```
RandomAccessFile aFile     = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel      inChannel = aFile.getChannel();
```

#### 从FileChannel中读取数据

通过调用`read()`方法可以完成，如下：

```
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
```

首先分配一个缓冲区，然后使用`read()`方法将数据从通道中读取到缓冲区中。方法的返回值说明有多少字节已经读出到缓冲区中，如果返回-1，说明已经到达文件末尾。

#### 向FileChannel中写入数据

通过调用`write()`方法可以完成，该方法需要传入一个缓冲区，如下：

```
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```

可以看到`write()`方法是在while循环中被调用的，因为并不知道具体会有多少字节要写入到通道中，所以只能不停地循环调用`write()`方法，直到缓冲区中没有数据为止。

#### 关闭FileChannel

当然，使用完后必须关闭`FileChannel`，像这样：

```
channel.close();
```

#### FileChannel 位置

不管是读还是写`FileChannel`，都需要从某个指定的位置开始。通过调用`FileChannel`对象的`position()`方法可以获得当前位置。还可以使用`position(long pos)`方法来设置位置，例如：

```
long pos channel.position();
channel.position(pos +123);
```

如果将位置设置到文件末尾之后，试图从该位置读取通道时，会返回-1，也就是文件结束标志。

如果将位置设置到文件末尾之后，向通道写入数据，文件会被撑大到指定的位置，并写入数据。这有可能导致『文件空洞』，也就是说物理磁盘文件中会有空隙。

#### FileChannel 大小

`long fileSize = channel.size();` 会返回当前通道所连接的文件的大小。

#### FileChannel 截断

`FileChannel.truncate()`方法用来截断文件，可以用它来将文件截断为指定的大小，比如`channel.truncate(1024);`就会将文件截成1024字节的大小。

#### FileChannel 强制写入

`FileChannel.force()`方法会将所有通道中未写入的数据全部写入到磁盘中。有时操作系统会把数据缓存到内存中以保证性能，所以不能确保写入到通道中的数据就一定会被写入到磁盘中，直到调用`force()`方法。

`force()`方法有一个布尔类型的参数，表示是否也写入文件的元数据（比如权限等），像这样：

```
channel.force(true);
```

### SocketChannel

Java NIO `SocketChannel` 是连接到 **TCP** 网络套接字的一种通道。它其实就是Java NIO中的[标准Java网络Socket](http://docs.oracle.com/javase/8/docs/api/java/net/Socket.html)。有两种创建方法：

1. 打开`SocketChannel`并连接到网络中的某个服务器
2. 在`ServerSocketChannel`中当有连接进来时，可以创建一个`SocketChannel`

#### 打开SocketChannel

```
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
```

#### 关闭SocketChannel

同样，在使用完后也需要关闭通道，像这样：

```
socketChannel.close();
```

#### 从SocketChannel中读取数据

调用`read()`方法就可以了：

```
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = socketChannel.read(buf);
```

首先分配一个缓冲区，数据会从`SocketChannel`中读取到缓冲区中。然后，调用`read()`方法读取数据，返回实际读到缓冲区中的字节数。如果返回-1，则说明已经到达数据末尾（连接被关闭）。

#### 向SocketChannel中写入数据

调用`write()`方法，并提供一个缓冲区作为参数就可以了：

```
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```

可以看到`write()`方法是在while循环中被调用的，因为并不知道具体会有多少字节要写入到通道中，所以只能不停地循环调用`write()`方法，直到缓冲区中没有数据为止。

#### 非阻塞模式

`SocketChannel`可以设置成非阻塞模式。在非阻塞模式下，可以异步地调用`connect()`， `read()` 和 `write()` 方法。

##### connect()

在非阻塞模式下，如果调用`SocketChannel`的`connect()`方法，该方法可能会在连接没有就绪时就返回。想要确定连接是否已经建立，可以调用`finishConnect()`方法，就像这样：

```
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://example.com", 80));

while(! socketChannel.finishConnect() ){
    //等待，或者干点别的...    
}
```

##### read()

非阻塞模式下的`read()`方法可能会在尚未读取任何数据时就立即返回，因此必须要注意返回值，看看到底有没有读到数据。

##### write()

非阻塞模式下的`write()`方法可能会在尚未写入任何数据时就立即返回，因此需要在循环调用`write()`方法。之前的`write()`方法也是这么用的，并没有什么不同。

#### 非阻塞模式与选择器

`SocketChannel`的非阻塞模式可以与选择器搭配，更好地进行工作。在选择器中注册若干个`SocketChannel`，可以向选择器询问哪个通道可以读，哪个通道可以写等。之后会详细介绍。

### ServerSocketChannel

Java NIO `ServerSocketChannel`是能够 **监听TCP连接** 的一种通道，就像[标准Java网络中的`ServerSocket`](http://docs.oracle.com/javase/8/docs/api/java/net/ServerSocket.html)一样，只不过它位于[`java.nio.channels`包](http://docs.oracle.com/javase/8/docs/api/java/nio/channels/ServerSocketChannel.html)包中。简单的示例如下：

```
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.socket().bind(new InetSocketAddress(9999));

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();
    // 开始处理socketChannel...
}
```

#### 打开ServerSocketChannel

像这样：

```
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
```

#### 关闭ServerSocketChannel

当然，用完之后也得关闭，像这样：

```
serverSocketChannel.close();
```

#### 监听连接

`ServerSocketChannel.accept()`方法会监听新进来的连接，该方法会返回一个带有新进入连接的`SocketChannel`。换言之，`accept()`会被阻塞到有新的连接进来。

一般我们不会满足于只监听一个连接，所以`accept()`一般会被放在一个循环中，像这样：

```
while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();
    // 开始处理socketChannel...
}
```

当然，也可以使用其他的标识来退出循环，而不是`true`的无限循环。

#### 非阻塞模式

`ServerSocketChannel`可以切换为非阻塞模式。在该模式下，`accept()`方法会立即返回，因此如果没任何连接到达的话，可能会返回空（null）。因此，这种情况下需要检查返回的`SocketChannel`是否为`null`，就像这样：

```
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();
    if(socketChannel != null){
        // 开始处理socketChannel...
    }
}
```

### Java NIO：非阻塞型服务器

即使理解了Java NIO非阻塞模式（选择器，通道，缓冲区等）是如何工作的，设计一个非阻塞型服务器仍旧很难。比起阻塞IO来说，非阻塞IO包含更多的挑战。本章会讨论非阻塞型服务器会面临的几个重要技术课题，同时会介绍一些可能的解决方案。

虽然本章描述的思想是围绕Java NIO来设计的，但是这种思想也适用于任何有类似于『选择器构架』的其他语言中。本章都是建立在Jakob Jenkov的一个简单非阻塞型服务器实现的基础上的，代码可以在[这里](https://github.com/jjenkov/java-nio-server)下载到。

#### 非阻塞型IO管道

非阻塞型IO管道是处理非阻塞IO的各个组件组成的一个链，包含了以非阻塞方式处理读和写的过程。图示如下：

[![Java NIO: 非阻塞型IO管道](http://xintq.net/img/post/javanio-non-blocking-pipeline.png)](http://xintq.net/img/post/javanio-non-blocking-pipeline.png)Java NIO: 非阻塞型IO管道

组件会使用选择器来检查某个通道何时有数据可以读取，然后主机读取数据并根据输入的数据来产生输出，之后输出会再次写入到通道中去。

非阻塞型IO管道不需要同时读数据和写数据，有些管道可能只读取数据，而有些管道可能只写入数据。

上图中只描述了单个组件。事实上一个非阻塞型IO管道可能会包含若干个组件来处理进来的数据。非阻塞型IO管道的长度取决于这个管道需要做什么。

非阻塞型IO管道可能会同时从多个通道中读取输入，比如，从多个`SocketChannel`中读取数据。

上图中的流程控制也被简化了。其实是组件主动通过选择器从通道中读取数据的，而不是像图示的那样，好像是通道主动将数据推到选择器，选择器再传递给组件，请不要误解。

#### 非阻塞型 vs. 阻塞型IO管道

非阻塞型与阻塞型IO管道的最大区别就是数据是如何从通道（Socket或者文件）中读取的。

通常，IO管道会从某个流（从Socket或者文件）中读取数据，将数据分割为一个个连贯的消息（Message）。这和词法分析中将数据流分解为一个个标记(Token)，然后用标记解析器(Tokenizer)进行解析的道理是一样的。而这里，会将数据流分解成比标记更大的消息。负责将数据流分解成消息的组件称之为 **消息读取器（Message Reader）**，简单描述如下图：

[![Java NIO: 消息读取器](http://xintq.net/img/post/javanio-message-reader.png)](http://xintq.net/img/post/javanio-message-reader.png)Java NIO: 消息读取器

阻塞型IO管道会使用类似于`InputStream`的接口，一个字节一个字节地从通道中读取数据。这种类似于`InputStream`的接口在调用是会被阻塞，直到有数据可以读出为止。这就是阻塞型消息读取器的实现原理。

使用阻塞型IO接口会大大简化消息读取器的实现。阻塞型消息读取器永远不需要考虑下面这几种情况：

- 数据流中无数据可读
- 只从数据流读取了消息的一部分
- 消息解析需要暂停和继续

类似地，阻塞型 **消息写入器（Message Writer）** （负责将消息写入到流的组件）也无需考虑下面这两种情况：

- 只写入了消息的一部分
- 消息的写入可以暂停和继续

##### 阻塞型IO管道的缺点

虽然阻塞型IO管道易于实现，不幸的是它的缺点，就是每个分解成消息的流都需要一个独立的线程来处理。必须这样做的原因是每个流的IO接口都会被阻塞，直到有数据读出为止。这就意味着单个线程不能够先尝试去读取一个流，当没有数据时再尝试去读取下一个流。只要这个线程去读取某个流，它就会被阻塞，直到有数据读出为止，该线程才能继续工作。

如果IO管道是处理很多并发连接的服务器的一部分，那么服务器就需要为每个活动的连接来创建一个线程。在只有上百个并发量的情况下，任何时候服务器可能都不会产生问题。但是，在百万级别的并发量下，这种设计可能就不能很灵活的应对了。每个线程会占用320K（32位的JVM）到1024K（64为的JVM）的内存，所以一百万个线程会占用1TB的内存！这还仅仅是在服务器处理任何传入消息之前（要知道，服务器在处理各种传入的消息时还会产生很多对象，还需要更多的内存）。

为了减少或者保持线程数，很多服务器都设计为使用一个线程池（比如100个线程）每次从到达的连接中读取消息。到达的连接会组成一个队列，线程池中的线程会依次、分批处理这个队列中的连接。如下图：

[![通过线程池处理连接队列](http://xintq.net/img/post/javanio-blocking-io-thread-pool.png)](http://xintq.net/img/post/javanio-blocking-io-thread-pool.png)通过线程池处理连接队列

但是，这种设计需要到达的各个连接经常性地发送数据。假如到达的连接长时间处于不活跃状态，最终线程池中的所有线程都可能被这些不活跃连接所阻塞。这就意味着服务器会变慢，或者没有响应。

还有一些服务器在设计时，为了缓解这一问题，会在线程池中线程的量上加入一些灵活性。比如，如果线程池中的线程已经用光，就再创建更多的线程来承载负荷。这种解决方法可以处理更多可能会让服务器失去响应的慢连接（不活跃连接）。但是记住，在服务器中实际能跑多少条线程还是有上限的，因此，如果有一百万个慢连接，服务器就可能承担不了了。

#### 基本的非阻塞型IO管道设计

非阻塞型IO管道可以使用一个线程从多个流中读取消息。这需要这些流能够切换到非阻塞模式。在非阻塞模式下，当尝试从流中读取数据时，可能会返回0，也可能返回更多的字节。如果是0，说明流中无数据可读，如果是1个字节以上，说明流中有东西可读。

为了避免检查返回值是0还是有数据可读，可以使用Java NIO的选择器。在选择器中，可以注册若干个实现了`SelectableChannel`接口的通道实例。当在选择器上调用`select()` 或者 `selectNow()`方法时，选择器会告诉你那些个通道有数据可读。如下图所示：

[![基本的非阻塞型IO管道设计](http://xintq.net/img/post/javanio-non-blocking-pipeline-design.png)](http://xintq.net/img/post/javanio-non-blocking-pipeline-design.png)基本的非阻塞型IO管道设计

#### 读取不完整消息

在从`SelectableChannel`类型的通道中读取数据时，我们并不清楚读取的数据块中到底是完整的一条消息，还是多条消息。数据块中可能只包含了消息的一部分，或者全部消息，或者好几条消息，比如1.5条或2.5条消息，如下图那样：

[![数据块中可能包含的消息数量](http://xintq.net/img/post/javanio-data-block-message-number.png)](http://xintq.net/img/post/javanio-data-block-message-number.png)数据块中可能包含的消息数量

在处理部分消息时，会遇到下面两个课题：

1. 检测数据块中是否有完整的一条消息
2. 在获取到剩余的数据之前，如何处理这些不完整的消息

消息完整性检测需要消息读取器检查数据块中的数据是否至少包含一条完整的消息。如果有一条或者多条完整的消息，那么这些消息会被送到管道中进行处理。消息完整性检测会很频繁的执行，所以必须让这一过程尽可能快速地执行。

只要数据块中存在不完整消息，不管是自身不完整，还是位于一条或者多条完整消息之后，这些消息片段必须先保存起来，直到通道中其余的部分到达为止。

消息完整性检测和不完整消息的存储都由消息读取器负责。为了避免不同通道的消息发生混乱，需要为每一个通道配备一个消息读取器。设计方案如下图：

[![每一个通道配备一个消息读取器](http://xintq.net/img/post/javanio-message-reader-per-channel.png)](http://xintq.net/img/post/javanio-message-reader-per-channel.png)每一个通道配备一个消息读取器

从选择器中获取有数据可读的通道之后，与该通道对应的消息读取器开始从通道中读取数据并分解为一条条的消息。如果分解后有完整的消息，则将这些完整消息传递到下一个组件中进行处理。

显然，消息读取器是和通信协议相关的。它需要知道所处理消息的格式。假如服务器的实现需要支持跨协议，那么就需要将消息读取器以插件的方式来实现 - 可能会以某种配置参数的形式传入到消息读取器工厂类中。

#### 保存不完整消息

现在，我们已经决定让消息读取器来保存那些不完整的消息，直到全部消息已经读取到为止，这就需要想想具体如何保存这些消息。

在设计上需要考虑以下两个方面：

1. 要尽量减少数据复制操作，因为复制的数据越多，性能越低
2. 要将完整的消息以连续的字节顺序保存，以更方便的解析这些消息

##### 每个消息读取器配备一个缓冲区

显然，这些不完整的消息需要保存在某种类型的缓冲区上。最直接的实现方法是为每个消息读取器都设置一个内部缓冲区。那么，这个缓冲区需要多大呢？它需要足够大，以至于能存放可允许的最大消息。也就是说，假如最大可允许的消息是1MB，那么每个消息读取器的内部缓冲区就需要至少1MB。

当连接数达到百万级时，让每个连接使用1MB显然是不现实的。1百万x1MB还是1TB的内存！更甚，如果最大消息大小为16MB，或者128MB呢？

##### 可变缓冲区

另外一种可能的选择是为每个消息读取器的内部缓冲区设置为大小可变的。可变缓冲区开始会很小，之后如果消息变大了，那么缓冲区也跟着变大。这样一来每个连接也就不会非得需要比如1MB的缓冲区，它们只需要按照实际需求来分配缓冲区，直到下一条消息到达为止。

可变缓冲区的实现方法有很多种，各有优缺点。接下来会详细讨论。

###### 拷贝时可变缓冲区

可变缓冲区的第一种实现方法是开始很小，比如4KB。如果消息大于4KB，就会分配更大的缓冲区，比如8KB，然后将原来4KB缓冲区中的数据复制到大缓冲区中。

这种拷贝时可变缓冲区实现的优点是，一个消息的所有数据都会存放在连续的字节数组中，进而让之后的消息解析更加容易。缺点是对消息越大，数据复制操作会越多。

为了减少数据复制次数，可以深入分析整个系统的消息大小，来确定一个可以减少数据复制次数的缓冲区大小。比如，你发现系统中大多数消息仅仅是简单的小数据量的请求和响应，一般都不会超过4KB。那么，缓冲区的初始大小就应该是4KB。

随后你可能还会发现那些大于4KB的消息常常是因为它们包含了一个文件，而经过调查发现系统中传入的文件大小不会超过128KB，那么将缓冲区的二次扩容大小设置为128KB也是合理的。

最后你发现一但消息超过了128KB之后，大小就无固定模式可寻，那么缓冲区的最终大小可能就需要设置成最大消息的大小了。

根据系统中传入消息大小而设置的以上三个缓冲区的大小值，在一定程度上可以减少数据复制的次数。4KB以下的消息永远不会复制。1百万并发连接的话，也就需要1百万x4KB=4GB的内存，现在（~2017年）的服务器都能扛得住。大小在4KB到128KB之间的消息会复制一次，而且只会往128KB的缓冲区中复制4KB的数据。而128KB到最大消息大小之间的消息会被复制两次：第一次复制4KB，第二次复制128KB，即使是最大的消息，一共也只需要复制132KB的数据。128KB以上的消息不是很多的话，这也是可以接受的。

一旦消息处理完之后，分配的内存就可以释放了。这样一来，同一个连接中下一条消息来之后就只需要最小大小的缓冲区。这就需要保证各个连接之间高效的内存共享。在同一时间，所有的连接都需要大缓冲区的情况是不太可能发生的。

Jakob Jenkov也实现了一个支持可变数组的内存缓冲区，详细可以参考[这篇文章](http://tutorials.jenkov.com/java-performance/resizable-array.html)。

###### 追加时可变缓冲区

让缓冲区可变的另外一种方法是用多个数组组成一个缓冲区。当需要更大的缓冲区时，只需要再申请一个缓冲区往里继续写数据就行。

这种缓冲区的增加方法有两种。一种是申请多个字节数组，并且将这些数组作为一个列表来保存。另外一种方法是在一个较大的、可共享的字节数组中分割一部分区域（分区），然后将这些分区作为缓冲区，保存为一个列表。这两种方法差别不大，个人认为分区策略要更好。

通过追加数组或者分区来扩大缓冲区方法的优点是在写入数据时没有数据复制发生。所有的数据都可以直接从通道复制到缓冲区或者分区中。缺点是数据没有保存在单独连续的数组中，这就使得之后的消息解析更加困难，因为解析器需要检查每个数组的结束位置以及全部数组列表中的结束位置。由于需要检查已写入数据中消息的结束位置，这一模型使用起来也不会很容易。

###### TLV编码的消息

有一些协议的消息是使用TLV（Type：类型，Length：消息长度，Value：消息内容）格式来编码的。这就意味着当消息到达之后，消息的总长度就存在消息头中，这样服务器就可以立即知道需要为全部消息分配多大内存了。

TLV编码使得内存管理更加简单。可以立即知道消息占用多大内存，缓冲区中也不会有空间浪费发生。缺点是，在所有数据到达之前就已经为这些数据预先分配好了内存。少数几个发送大块消息的慢连接也可能会耗尽所有可用内存，导致服务器无法响应。

可以通过使用内部包含多个TLV的消息格式来解决这一问题。这样一来，可以为每一个TLV来分配内存而不是整个消息，而且只需要为那些已经到达的消息来分配内存。但是，如果TLV部分很大的话，还是会像大块消息一样影响到内存管理。

另外一种变通方法是，为那些未接收的消息设定超时时间，比如10~15秒钟。如果正巧有很多条大块消息同时到达时，服务器有可能会短时间失去响应，但是这个方法可以保证服务器之后能够恢复响应。另外，在遭受到蓄意DoS（拒绝服务）攻击时，服务器上的内存还是可能会消耗殆净。

TLV编码有不同的变种。至于使用多少字节来表示类型和长度，这都取决于每种不同的TLV编码。有些TLV编码会将消息长度作为第一个字段，之后是类型，最后是消息内容（也就是LTV编码）。尽管字段的顺序不一样，但这些还是属于TLV的变种。

TLV编码格式大大简化了内存管理，这也是为什么说HTTP1.1协议很烂的原因之一。HTTP2.0中也想通过使用LTV格式编码的数据帧来解决这个问题。

##### 写入不完整的消息

在非阻塞型IO管道中数据写入也是一个课题。非阻塞模式下，当在通道上调用`write(ByteBuffer)`方法时，并不能保证到底能在`ByteBuffer`中写入多少数据。`write(ByteBuffer)`方法会返回实际写入的字节数，所以跟踪实际写入的数据量也是可能的。而真正的课题是：如何跟踪那些不完整消息的写入过程，以确保最终全部消息都能够成功写入。

为了管理通道中的不完整消息的写入过程，创建一个消息写入器。和消息读取器一样，每个通道都需要一个消息写入器。在消息写入器内部，会追踪到底写入了当前消息的多少字节。

如果消息写入器收到的消息量比它能够直接写入到通道中的多时，就需要在消息写入器中进行消息排队。消息写入器会尽可能快地向通道中写入数据。消息写入器的简单设计图如下：

[![使用消息写入器来写入不完整的消息](http://xintq.net/img/post/javanio-writing-partial-message.png)](http://xintq.net/img/post/javanio-writing-partial-message.png)使用消息写入器来写入不完整的消息

为了让消息写入器能够发送之前发送的不完整消息，需要不时地调用消息写入器，以便其能够发送更多的数据。

服务器中的连接越多，消息写入器的实例也就越多。检查一百万个消息写入器中那些有数据可以写入是一个很慢的过程。首先，一些消息写入器可能都没任何消息可写。我们不想连它们也检查。其次，并不是所有的通道都进入了写就绪的状态，我们也不想在这些未就绪的通道上浪费时间。

要检测通道是否可写，可以将其注册到选择器中。然而，并不想把所有的通道都注册到选择器中。想象一下，一百万个大部分闲置的连接，而这一百万个连接全部注册到一个选择器中。之后一旦调用`select()`方法，这些通道基本上都会进入写就绪状态（要知道大部分连接都是闲置的话，几乎是一呼百应）。这样一来，你不得不挨个检查这些通道上的消息写入器，看看有没有消息可以写。

为了避免检查消息写入器有无消息可写，同时所有的通道中是否有消息送达，可以使用以下两步式解决方法：

1. 当消息被写入到消息写入器时，消息写入器把对应的通道注册到选择器中（如果对应的通道没有被注册的话）
2. 当服务器空闲时，它可以问选择器那些通道可以写。对于每个可写入的通道，让它所对应的消息写入器向通道中写入数据。如果这个消息写入器中所有的消息都已经写入完毕，则把该通道从选择器中注销。

这小小的两步式解决方法保证了只有注册到选择器中的通道才是有消息可写入的通道。

##### 总结一下

可以看出，非阻塞服务器需要不时地检查进来的数据，看看是否有完整的消息可接收。在一条或者多条完整消息被接收之前，服务器可能要检查好多次，光检查一次是不够的。

类似地，非阻塞服务器也要不时地检查是否有数据需要写。如果有的话，还需要检查相应的连接是否已经准备好可写入。在消息排队时光检查一次是不够的，因为有可能只写入了消息的一部分。

最终，在所有的非阻塞型服务器中会有规律的执行三种『管道』：

- 检查打开的连接中是否有新数据到达的『读管道』
- 处理任何到达的完整消息的『数据处理管道』
- 检查是否可以向任何打开的连接中写入响应消息的『写管道』

这三个管道会循环执行。对他们的执行过程进行优化也是可能的。比如，在没有消息排队时可以跳过写管道。或者，没有新到的完整消息时，可以跳过消息处理管道。

服务器的总体循环时序图如下：

[![非阻塞型服务器的总体循环时序图](http://xintq.net/img/post/javanio-non-blocking-server-loop.png)](http://xintq.net/img/post/javanio-non-blocking-server-loop.png)非阻塞型服务器的总体循环时序图

感觉理解困难的话可以直接看[这里的代码](https://github.com/jjenkov/java-nio-server)。

上面代码中实现的非阻塞型服务器使用了一个双线程模型。第一个线程负责从`ServerSocketChannel`中接收连接。第二个线程来处理已经接收到的连接，比如读取消息，处理消息，以及向连接中写入响应消息。原理图如下：

[![非阻塞型服务器线程模型](http://xintq.net/img/post/javanio-non-blocking-server-thread-model.png)](http://xintq.net/img/post/javanio-non-blocking-server-thread-model.png)非阻塞型服务器线程模型

### DatagramChannel

Java NIO `DatagramChannel`是可以接受和发送 **UDP** 数据包的一种通道。因为UDP是一种无连接的传输层协议，所以不能像其他的通道那样对`DatagramChannel`读取和写入数据，而是需要发送和接收数据。

#### 打开DatagramChannel

下面的例子在UDP端口9999上打开了通道：

```
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999));
```

#### 接收数据

使用`receive()`方法，像这样：

```
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();

channel.receive(buf);
```

`receive()`方法会将接收到的数据包复制到给定的缓冲区中。如果接受到数据包比给定的缓冲区大，多出来的数据会被默默地丢掉。

#### 发送数据

使用`send()`方法，像这样：

```
String newData = "New String to write to file..."
                    + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();

int bytesSent = channel.send(buf, new InetSocketAddress("example.com", 80));
```

上面的例子中向『example.com』服务器的80端口发送一个字符串。在这个端口上没有任何监听，所以什么也不会发生。当然，我们也不知道数据包是否已经被接收，因为UDP协议并不会保证数据一定会被送达。

#### 连接到指定地址

将`DatagramChannel`绑定到某个网络地址也是可能的。因为UDP是无连接的协议，这里的绑定不是像TCP通道那样真的与服务器建立连接。而是将`DatagramChannel`锁定到这个地址，这样可以只向这个地址发送和接收数据。例如：

```
channel.connect(new InetSocketAddress("example.com", 80));
```

当连接之后，可以像使用常规的通道一样使用`read()`和`write()`方法了。然而要记住，这并不能保证数据一定会送达。例如：

```
int bytesRead = channel.read(buf);    
int bytesWritten = channel.write(buf);
```

## Java NIO Pipe

Java NIO `Pipe`是两个线程之间的单向数据连接。一个`Pipe`有一个源通道（Source Channel）和一个目的通道（Sink Channel）。向目的通道写数据，从源通道读取数据。图示如下：

[![Java NIO Pipe内部构成](http://xintq.net/img/post/javanio-pipe-internals.png)](http://xintq.net/img/post/javanio-pipe-internals.png)Java NIO Pipe内部构成

### 创建一个Pipe

```
Pipe pipe = Pipe.open();
```

### 写数据

向`Pipe`中写入数据时需要访问它的目的通道，如下：

```
Pipe.SinkChannel sinkChannel = pipe.sink();
```

之后就可以调用`SinkChannel`的`write()`方法：

```
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}
```

### 读数据

从`Pipe`中读取数据时需要访问它的源通道：

```
Pipe.SourceChannel sourceChannel = pipe.source();
```

之后就可以调用`SourceChannel`的`read()`方法：

```
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
```

返回值表示向缓冲区中读出了多少字节的数据。

## Java NIO vs. IO

在学习Java NIO和IO API是，总会有一个问题：

**什么时候该用Java NIO，什么时候该用 IO?**

本章会介绍Java NIO和IO的不同之处，用例，以及它们会对编码带来的影响。

### Java NIO和IO的主要不同之处

简单概括如下表，随后会详细展开。

| IO     | NIO        |
| ------ | ---------- |
| 面向流 | 面向缓冲区 |
| 阻塞IO | 非阻塞IO   |
| -      | 选择器     |

#### 面向流 vs. 面向缓冲区

Java IO是面向流设计的，Java NIO是面向缓冲区设计的，具体什么意思呢？

Java IO是面向流设计的，意味着可以从一个流中每次读取一个或者若干个字节。而至于如何处理读到的字节取决于你。它们不会被缓存到任何地方。甚至，无法前后移动流中的数据，想要前后移动流中的数据的话，需要先把它们缓存到一个缓冲区中。

Java NIO是面向缓冲区设计的，意味这数据先被存放到一个缓冲区中，然后再去处理它们。可以按需对缓冲区中数据来回移动。在处理上更加灵活。但是，需要检查缓冲区是否包含了所有要处理的数据。而且，还需要保证在往缓冲区中写入数据时不要覆盖里面未处理的数据。

#### 阻塞IO vs. 非阻塞IO

Java IO的各种流都是阻塞型的。也就是说，当一个线程执行了`read()`或`write()`时，这个线程会被阻塞到有数据可以读写为止。该线程在此期间不能干任何事情。

Java NIO的非阻塞模式可以使一个线程向通道请求数据，而且只会取得当前可用的部分，即使没有数据的话，线程也不会被阻塞，而是会去做其他事情。

线程在非阻塞IO调用时，空闲时间可以用在其他的通道上做IO操作。这样就保证了单个线程可以管理多个通道的输入和输出。

#### 选择器

Java NIO的选择器可以让单个线程来监视多个通道的输入。可以把多个通道注册到一个选择器中，然后使用这个线程来『选择』一个有数据可以处理的通道，或者『选择』一个可以写入的通道。选择器使得单个线程管理多通道变得更简单。

### NIO和IO对应用程序设计的影响

在IO层是选择NIO还是IO可能影响到应用程序设计的以下几个方面：

1. 对NIO和IO类的API调用
2. 数据处理
3. 数据处理线程的多少

#### API调用

显然，在使用NIO时的API调用与使用IO时是不同的，这没有什么好奇怪的。只不过不是从`InputStream`中一个字节一个字节的读取数据，而是先将数据读取到缓冲区中，然后再进行处理。

#### 数据处理

与IO相比，使用纯NIO实现时的数据处理也会有影响。

在使用IO时，会从`InputStream`或者`Reader`中一个字节一个字节的读取数据。假设一下处理以下的结构化数据流：

```
Name: Anna
Age: 25
Email: anna@mailserver.com
Phone: 1234567890
```

对上面文本的处理过程可能像下面这样：

```
InputStream input = ... ; // 从客户端套接字中获取InputStream

BufferedReader reader = new BufferedReader(new InputStreamReader(input));

String nameLine   = reader.readLine();
String ageLine    = reader.readLine();
String emailLine  = reader.readLine();
String phoneLine  = reader.readLine();
```

注意到这时的处理状态取决于程序执行到了哪一步。换言之，当第一个`reader.readLine()`返回时，可以确定地知道已经读取了一整行数据。这就是为什么说`readLine()`会一直阻塞到读完一整行数据。也可以知道这行数据包含了『Name』字段，同样地，当第二个`readLine()`返回时，可以知道这行包含了『Age』，以此类推。

可以看出，只有有新数据读取时程序才会继续执行，而且每一步都会知道读到的数据是什么。一旦执行的线程处理过了某一部分数据后，它就不会再返回（大多数情况下不会）。下图说明了这一原则：

[![Java IO: 从阻塞的流中读取数据](http://xintq.net/img/post/javanio-blocking-io.png)](http://xintq.net/img/post/javanio-blocking-io.png)Java IO: 从阻塞的流中读取数据

NIO的实现就不一样了。简单的例子如下：

```
ByteBuffer buffer = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buffer);
```

注意在第二行，将数据读到了`ByteBuffer`中，当方法返回时，也不知道到底缓冲区中读到了多少字节。这会在一定程度上是数据处理变得困难。

假设，第一次调用`read(buffer)`之后，缓冲区中只读入了半行数据，比如『Name: An』。这样的数据能处理么？并不能。还需要等到这行数据全部读到缓冲区中才能进行有效的处理。

那么要如何知道缓冲区中已经有足够的数据可以用来有效的处理呢？这个，不太可能。唯一的方法是去看缓冲区中的数据。到头来可能会去缓冲区里查看好几次才能确定所有的数据都到齐了。从程序设计角度来说，这非常低效而且可能会变得很糟糕。比如：

```
ByteBuffer buffer = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buffer);

while(! bufferFull(bytesRead) ) {
    bytesRead = inChannel.read(buffer);
}
```

`bufferFull()`方法必须追踪有多少数据已经读入到缓冲区中，然后返回`true`或者`false`，取决于缓冲区是否填满。换言之，缓冲区中的数据可以处理时，认为缓冲区已经填满。

`bufferFull()`方法会扫描整个缓冲区，但是同时还要保持缓冲区的状态。否则，下一个读入数据的位置可能不对。虽然这不太可能，但是还需要注意一下。

如果缓冲区已满，就可以处理数据了。如果未满，在某些特殊情况下可能需要处理已经到达的一部分数据。但是大多数情况下，这是不太合理的。

判断缓冲区数据是否就绪的循环设计图如下：

[![Java NIO 判断缓冲区数据是否就绪](http://xintq.net/img/post/javanio-is-buffer-data-ready.png)](http://xintq.net/img/post/javanio-is-buffer-data-ready.png)Java NIO 判断缓冲区数据是否就绪

### 小结

NIO使得用单个（或者少数几个）线程管理多个通道（网络连接或者文件）成为可能，但代价是比起处理阻塞流来，数据的解析会变得更加复杂。

如果需要同时间管理上千个连接，每个连接发送的数据量很小，比如聊天服务器，使用NIO来实现服务器可能会有优势。类似地，如果需要与其他主机维持很多打开着的连接，比如P2P网络，使用单线程来管理所有出站连接也是一个优势。这种单线程对多连接的设计思想如图所示：

[![Java NIO 单线程对多连接](http://xintq.net/img/post/javanio-a-thread-vs-conns.png)](http://xintq.net/img/post/javanio-a-thread-vs-conns.png)Java NIO 单线程对多连接

如果只有很少几个连接，它们都占用高带宽，每次发送大量数据，可能传统的IO服务器实现更适合这种场景。传统IO服务器的设计原理如下图：

[![Java IO 单线程对单连接](http://xintq.net/img/post/javanio-thread-conn-1on1.png)](http://xintq.net/img/post/javanio-thread-conn-1on1.png)Java IO 单线程对单连接

## Java NIO Path

`Path`接口是Java 6和Java7中Java NIO 2更新的一部分。在Java 7的Java NIO中加入了`Path`接口，它位于`java.nio.file`包中，所以这个接口的完整名称是[`java.nio.file.Path`](http://docs.oracle.com/javase/8/docs/api/java/nio/file/Path.html)。

`Path`接口表示文件系统中的路径，可以是文件路径或者目录路径，可以是相对路径或者绝对路径。绝对路径是从根目录开始一直到指定的文件或者目录的完整路径。相对路径是相对于某个目录开始，到指定的文件或者目录的路径。这里请不要与操作系统的PATH环境变量搞混了。`java.nio.file.Path`和PATH环境变量没有半毛钱关系。

`java.nio.file.Path`在和很多方面与[`java.io.File`](http://docs.oracle.com/javase/8/docs/api/java/io/File.html)相似，也有几个小地方不同。在大多数情况下，可以使用`Path`接口来代替`File`类。

### 创建Path实例

使用之前必须前创建`Path`实例，在这里使用`java.nio.file.Paths`类的静态方法`get()`来获取一个`Path`实例。如下所示：

```
import java.nio.file.Path;
import java.nio.file.Paths;

public class PathExample {

    public static void main(String[] args) {
        Path path = Paths.get("/tmp/data/myfile.txt");
    }
}
```

上面的代码中，可以把`Paths.get()`看成是产生`Path`实例的一个工厂。

`Paths.get()`中的参数决定了是创建一个相对路径的Path实例还是绝对路径的Path实例。上面的例子中使用的是从根目录开始的`/tmp/data/myfile.txt`绝对路径，在Windows系统中，它是这样的：`c:\\data\\myfile.txt`。

当然，也可以使用相对路径，对应的方法就变成了`Paths.get(basePath, relativePath)`。如下所示：

```
Path projects = Paths.get("d:\\data", "projects");
Path file     = Paths.get("d:\\data", "projects\\a-project\\myfile.txt");
```

第一行表示创建了一个指向`d:\data\projects`目录的实例。第二行表示创建了一个指向`d:\data\projects\a-project\myfile.txt`文件的实例。

在使用相对路径时，还可以使用两个特殊的路径标识：

- `.` 表示当前目录。

  ```
  Path currentDir = Paths.get(".");
  System.out.println(currentDir.toAbsolutePath());
  ```

  上面的代码会打印出当前代码所在目录的绝对路径。

  如果将`.`放在一个路径中间，意思就是同一个目录下，比如：

  ```
  Path currentDir = Paths.get("d:\\data\\projects\.\a-project");
  ```

  其实就是获取`d:\data\projects\a-project`路径的实例。

- `..` 表示父目录，或者上层目录。

  下面的代码会得到一个指向当前代码所在目录的上一层目录的实例。

  ```
  Path parentDir = Paths.get("..");
  ```

  如果在路径中间使用了`..`，就表示会切换当前目录的上层目录，比如：

  ```
  String path = "d:\\data\\projects\\a-project\\..\\another-project";
  Path parentDir2 = Paths.get(path);
  ```

  上面的代码会得到一个指向`d:\data\projects\another-project`的实例。

当然，可以混合使用`.`和`..`，比如：

```
Path path1 = Paths.get("d:\\data\\projects", ".\\a-project");

Path path2 = Paths.get("d:\\data\\projects\\a-project",
                       "..\\another-project");
```

#### Path.normalize()

`Path.normalize()`方法可以用来规格化路径。规格化的意思是去除路径中间的`.`和`..`，并且解析到路径所指的位置。例如：

```
String originalPath =
        "d:\\data\\projects\\a-project\\..\\another-project";

Path path1 = Paths.get(originalPath);
System.out.println("path1 = " + path1);

Path path2 = path1.normalize();
System.out.println("path2 = " + path2);
```

上面的代码直接打印`Path`实例，事实上是调用了`Path.toString()`方法。最终的输出结果为：

```
path1 = d:\data\projects\a-project\..\another-project
path2 = d:\data\projects\another-project
```

可以看到，通过规格化操作，将路径中包含的`a-project\..`部分被去除了。

## Java NIO Files

Java NIO [`Files`类](http://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html)提供了操作文件系统中文件的几种方法。本章只介绍最常用的几个方法，可以参考JavaDoC了解其他的方法。

`java.nio.file.Files`需要和`java.nio.file.Path`配合使用，所以进入本章前请务必理解`Path`。

### Files.exists()

`Files.exists()`检查文件系统中给定的Path是否已经存在。

也可以创建一个文件系统中不存在的Path实例。比如，要新建一个目录的话，会先创建一个不存在的Path实例，然后再创建这个目录。

因为Path实例指向的文件可能在文件系统不存在，所以可以使用`Files.exists()`方法来检查。例如：

```
Path path = Paths.get("data/logging.properties");

boolean pathExists =
        Files.exists(path,
            new LinkOption[]{ LinkOption.NOFOLLOW_LINKS});
```

上面的代码中，先用需要判断存在与否的路径创建一个Path实例。然后将Path实例作为第一个参数传入`Files.exists()`方法来检查。

注意到`Files.exists()`方法的第二个参数。这个参数是影响判断文件存在与否的选项组成的数组。上面的代码中使用了`LinkOption.NOFOLLOW_LINKS`选项，意思是判断文件存在与否时，不会跟踪判断符号连接（Windows下的快捷方式，macOS下的替身）所指向的文件是否存在。

### Files.createDirectory()

`Files.createDirectory()`为给定的Path实例创建一个目录。例如：

```
Path path = Paths.get("data/subdir");

try {
    Path newDir = Files.createDirectory(path);
} catch(FileAlreadyExistsException e){
    // 目录已经存在
} catch (IOException e) {
    // 其他地方出了问题
    e.printStackTrace();
}
```

以上代码第一行先以给定路径创建Path实例。接着在try-catch中调用`Files.createDirectory()`方法，并传入Path实例。若创建成功，这返回新建目录的Path实例。

如果指定的目录已经存在，就会抛出`java.nio.file.FileAlreadyExistsException`异常。如果其他地方处理问题，则会抛出`IOException`异常。比如新建目录的父目录不存在，就会抛出`IOException`异常。

#### 覆盖已存在的文件

使用`Files.copy()`方法可以强制覆盖一个已经存在的文件。比如：

```
Path sourcePath      = Paths.get("data/logging.properties");
Path destinationPath = Paths.get("data/logging-copy.properties");

try {
    Files.copy(sourcePath, destinationPath,
            StandardCopyOption.REPLACE_EXISTING);
} catch(FileAlreadyExistsException e) {
    // 目标文件已经存在
} catch (IOException e) {
    // 其他地方出了问题
    e.printStackTrace();
}
```

注意到`Files.copy()`方法的第三个参数，`StandardCopyOption.REPLACE_EXISTING`就是覆盖已经存在文件的意思。

### Files.move()

Java NIO `Files`类中包含了移动文件到另外一个位置的方法。移动文件和重命名文件实质上是一样的，除了移动文件时可以在移动位置的同时改变文件名。没错，可以使用`java.io.File`中的`renameTo()`方法，但是现在也可以使用`java.nio.file.Files`类的`Files.move()`方法做同样的事情了。比如：

```
Path sourcePath      = Paths.get("data/logging-copy.properties");
Path destinationPath = Paths.get("data/subdir/logging-moved.properties");

try {
    Files.move(sourcePath, destinationPath,
            StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) {
    //移动文件失败
    e.printStackTrace();
}
```

注意到`Files.move()`方法的第三个参数，这个参数是可选的，表示移动文件时覆盖已经存在的文件。

在移动文件失败时会抛出`IOException`异常。比如目标文件已经存在，而没有指定`StandardCopyOption.REPLACE_EXISTING`选项，或者要移动的文件不存在，等等。

### Files.delete()

`Files.delete()`方法用来删除文件或者目录。比如：

```
Path path = Paths.get("data/subdir/logging-moved.properties");

try {
    Files.delete(path);
} catch (IOException e) {
    //删除文件失败
    e.printStackTrace();
}
```

### Files.walkFileTree()

`Files.walkFileTree()`方法可以递归遍历目录树。`walkFileTree()`方法需要一个Path实例和一个`FileVisitor`作为参数。Path实例指向需要遍历的目录，而`FileVisitor`在每次遍历时都会调用。

`FileVisitor`接口的声明如下：

```
public interface FileVisitor {
    public FileVisitResult preVisitDirectory(
        Path dir, BasicFileAttributes attrs) throws IOException;

    public FileVisitResult visitFile(
        Path file, BasicFileAttributes attrs) throws IOException;

    public FileVisitResult visitFileFailed(
        Path file, IOException exc) throws IOException;

    public FileVisitResult postVisitDirectory(
        Path dir, IOException exc) throws IOException;
}
```

在传入到`walkFileTree()`方法时，必须实现该接口。`FileVisitor`接口的实现类中的各个方法会在遍历的不同阶段被调用。其实，并不需要实现`FileVisitor`接口的所有方法，直接继承`SimpleFileVisitor`类会更简单，这个类是`FileVisitor`接口的默认实现。例如：

```
Files.walkFileTree(path, new FileVisitor<Path>() {
  @Override
  public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
    System.out.println("pre visit dir:" + dir);
    return FileVisitResult.CONTINUE;
  }

  @Override
  public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
    System.out.println("visit file: " + file);
    return FileVisitResult.CONTINUE;
  }

  @Override
  public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
    System.out.println("visit file failed: " + file);
    return FileVisitResult.CONTINUE;
  }

  @Override
  public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
    System.out.println("post visit directory: " + dir);
    return FileVisitResult.CONTINUE;
  }
});
```

`FileVisitor`接口的实现类中的各个方法会在遍历的不同阶段被调用。

- `preVisitDirectory()`方法会在服务任何目录之前会被调用
- `postVisitDirectory()`方法会在访问任何目录之后被调用。
- `visitFile()`方法会在每次遍历到文件，记住，不是目录，只是文件时被调用。
- `visitFileFailed()`方法会在访问文件失败时被调用，比如访问权限不足或者其他问题。

这四个方法都应该返回一个`FileVisitResult`枚举实例。`FileVisitResult`枚举包含以下几个选项：

- `CONTINUE` - 表示遍历可以正常进行
- `TERMINATE` - 表示遍历应该结束
- `SKIP_SIBLINGS` - 表示遍历可以继续，但是不会访问同级的文件或者目录
- `SKIP_SUBTREE` - 表示遍历可以继续，但是不会继续访问当前目录。这个选项只在`preVisitDirectory()`方法返回时起作用。其他方法只会把它当做`CONTINUE`来处理。

通过这些方法的返回值，就可以确定遍历操作是否应该继续。

#### 搜索文件

下面的例子中，在`walkFileTree()`方法中使用了`SimpleFileVisitor`类，来查找名称为`README.txt`的文件。

```
Path rootPath = Paths.get("data");
String fileToFind = File.separator + "README.txt";

try {
  Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {

    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
      String fileString = file.toAbsolutePath().toString();
      //System.out.println("pathString = " + fileString);

      if(fileString.endsWith(fileToFind)){
        System.out.println("file found at path: " + file.toAbsolutePath());
        return FileVisitResult.TERMINATE;
      }
      return FileVisitResult.CONTINUE;
    }
  });
} catch(IOException e){
    e.printStackTrace();
}
```

#### 递归删除目录

`Files.walkFileTree()`方法也可以用来递归地删除一个目录下所有的子目录和文件。`Files.delete()`只能删除空目录。通过遍历所有的目录，删除每个目录中的所有文件（在`visitFile()`方法中删除），然后再删除目录本身（在`postVisitDirectory()`方法中删除），就可以删除所有的子目录和文件了。例如：

```
Path rootPath = Paths.get("data/to-delete");

try {
  Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
      System.out.println("delete file: " + file.toString());
      Files.delete(file);
      return FileVisitResult.CONTINUE;
    }

    @Override
    public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
      Files.delete(dir);
      System.out.println("delete dir: " + dir.toString());
      return FileVisitResult.CONTINUE;
    }
  });
} catch(IOException e){
  e.printStackTrace();
}
```

### Files类中的其他方法

在`java.nio.file.Files`中还包含了很多有用的方法。比如创建符号链接，确定文件大小，设置文件权限等等。详细的内容可以参考[JavaDoC](http://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html)。

## Java NIO AsynchronousFileChannel

在Java 7的Java NIO中新加入了`AsynchronousFileChannel`，它可以异步地读写文件。

### 创建`AsynchronousFileChannel`

```
Path path = Paths.get("data/test.xml");

AsynchronousFileChannel fileChannel =
    AsynchronousFileChannel.open(path, StandardOpenOption.READ);
```

上述代码中，首先创建一个指向指定文件的Path实例，然后通过调用`open()`方法，将Path实例作为第一个参数传给`AsynchronousFileChannel`。第二个参数表示了要对文件进行何种操作，`StandardOpenOption.READ`表示要打开文件进行读操作。

### 读数据

在`AsynchronousFileChannel`中有两种读文件的方法，都是通过调用`read()`方法来完成。下面详细说明。

#### 通过Future读取数据

第一种方法是调用返回`Future`对象的`read()`方法。像这样：

```
Future<Integer> operation = fileChannel.read(buffer, 0);
```

`read()`方法的第一个参数是`ByteBuffer`，从`AsynchronousFileChannel`读出的数据会转存到这个缓冲区中。第二个参数表示文件中的读取位置。

即使读操作没有完成，`read()`方法也会立即返回。通过检查返回的`Future`对象中的`isDone()`方法来判断读操作是否结束。下面是一个完整的例子：

```
AsynchronousFileChannel fileChannel =
    AsynchronousFileChannel.open(path, StandardOpenOption.READ);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;
Future<Integer> operation = fileChannel.read(buffer, position);

while(!operation.isDone());

buffer.flip();
byte[] data = new byte[buffer.limit()];
buffer.get(data);
System.out.println(new String(data));
buffer.clear();
```

首先创建一个`AsynchronousFileChannel`对象，然后又创建一个`ByteBuffer`。将`ByteBuffer`作为第一个参数传给`read()`方法，并指定从`0`位置开始读取文件。在开始读操作后，循环检查读操作是否完成，虽然这种处理方式会浪费CPU资源，但是无论哪种方法，都需要等待读操作完成。一旦读操作完成后，就打印出缓冲区中的数据。

#### 通过CompletionHandler来读取数据

`read()`方法的第二种重载方式会传入一个`CompletionHandler`接口实例作为参数，像这样：

```
fileChannel.read(buffer, position, buffer, new CompletionHandler<Integer, ByteBuffer>() {
    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        System.out.println("result = " + result);
        attachment.flip();
        byte[] data = new byte[attachment.limit()];
        attachment.get(data);
        System.out.println(new String(data));
        attachment.clear();
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {
    }
});
```

当读操作完成后，会调用`CompletionHandler`实例的`completed()`方法。`completed()`方法的第一个`Integer`参数说明了读取了多少字节的数据，第二参数`attachment`其实就是`read()`方法的第三个参数，这里读写都使用了同一个缓冲区，这里当然可以使用其他的缓冲区。

如果读操作失败了，会调用`CompletionHandler`实例的`failed()`方法。

#### 写数据

就和写数据一样，写数据时也有两种方法，都是调用`AsynchronousFileChannel`的`write()`方法。详述如下。

##### 通过Future写入数据

`AsynchronousFileChannel`可以异步地向文件中写入数据，像这样：

```
Path path = Paths.get("data/test-write.txt");
AsynchronousFileChannel fileChannel =
    AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

buffer.put("test data".getBytes());
buffer.flip();

Future<Integer> operation = fileChannel.write(buffer, position);
buffer.clear();

while(!operation.isDone());

System.out.println("Write done");
```

首先将`AsynchronousFileChannel`以写模式打开，创建一个`ByteBuffer`并写入一些数据。然后将`ByteBuffer`中的数据写入到文件中。最后，循环检查返回的`Future`对象的`isDone()`方法，看写操作是否完成。

在写入之前，可以用下面的代码确保写入的文件是一定存在的：

```
if(!Files.exists(path)){
    Files.createFile(path);
}
```

#### 通过CompletionHandler来写入数据

当然也可使用`CompletionHandler`来代替`Future`对象进行数据写入。比如：

```
Path path = Paths.get("data/test-write.txt");
if(!Files.exists(path)){
    Files.createFile(path);
}
AsynchronousFileChannel fileChannel =
    AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

buffer.put("test data".getBytes());
buffer.flip();

fileChannel.write(buffer, position, buffer, new CompletionHandler<Integer, ByteBuffer>() {

    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        System.out.println("bytes written: " + result);
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {
        System.out.println("Write failed");
        exc.printStackTrace();
    }
});
```

在写入完成后，会调用`CompletionHandler`实例的`completed()`方法，如果因为某种原因失败了的话，则会调用`failed()`方法。这里要特别注意参数`attachment`的使用方法。



## 参考：

http://xintq.net/2017/06/12/everything-about-java-nio/#%E9%98%BB%E5%A1%9Eio-vs-%E9%9D%9E%E9%98%BB%E5%A1%9Eio

https://www.zhihu.com/question/27991975

https://www.jianshu.com/p/f48c8410ac79