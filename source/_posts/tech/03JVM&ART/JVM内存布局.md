---
title: JVM内存布局
urlname: jvm_art_java_memory
date: 2016-12-01
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 03JVM&ART
tags:
    - java
    - 内存
---

## JVM虚拟机内存结构，以及它们的作用
线程私有：栈区，本地方法栈，pc指针
线程共有：方法区，堆区

## JVM内存
内存分布大纲
![image.png](https://upload-images.jianshu.io/upload_images/11010834-b029a0c862019b02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
垃圾回收大纲
![image.png](https://upload-images.jianshu.io/upload_images/11010834-8b13a027fac13d84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/11010834-259b2042b7752763.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 介绍JVM中7个区域，然后把每个区域可能造成内存的溢出的情况说明
*   程序计数器：看做当前线程所执行的字节码行号指示器。是线程私有的内存，且唯一一块不报OutOfMemoryError异常的内存区域。
*   Java虚拟机栈：用于描述java方法的内存模型：每个方法被执行时都会同时创建一个栈帧用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程就对应着一个栈帧在虚拟机中从入栈到出栈的过程。如果线程请求的栈深度大于虚拟机所允许的深度就报StackOverflowError, 如果虚拟 机栈可以动态扩展，当拓展时无法申请到足够的内存会抛出OutOfMemoryError. 是线程私有的。
*   本地方法栈：与虚拟机栈相似，不同的在于它是为虚拟机使用到Native方法服务的。会抛出StackOverflowError和OutOfMemoryError。是线程私有的。
*   Java堆: 是所有线程共享的一块内存，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。如果堆上没有内存完成实例的分配就会报OutOfMemoryError.
*   方法区（永久代）：用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。当方法区无法满足内存分配需求时，会抛出OutOfMemoryError。是共享内存。
*   运行时常量池：用于存放编译器生成的各种字面量和符号引用，是方法区的一部分。无法申请内存时抛出OutOfMemoryError。
*   直接内存：不是虚拟机运行时数据的一部分，也不是java虚拟机规范中定义的区域，是计算机直接的内存空间。这部分也被频繁使用，如JAVA NIO的引入基于通道和缓存区的I/O使用native函数直接分配堆外内存。如果内存不足会报OutOfMemoryError。
