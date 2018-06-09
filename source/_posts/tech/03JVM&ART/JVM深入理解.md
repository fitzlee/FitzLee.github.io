---
title: JVM深入理解
urlname: jvm_art_jvm_deep_learn
date: 2017-01-06
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 03JVM&ART
tags:
    - jvm
    - 深入理解
---

Java虚拟机概览
https://www.cnblogs.com/2014asm/p/7999049.html

## 内存分布概要
![image.png](https://upload-images.jianshu.io/upload_images/11010834-b029a0c862019b02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 垃圾回收概要
![image.png](https://upload-images.jianshu.io/upload_images/11010834-8b13a027fac13d84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/11010834-259b2042b7752763.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## JVM类字节码结构
![image.png](https://upload-images.jianshu.io/upload_images/11010834-69c39eb46c77c1ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 类加载机制
![image.png](https://upload-images.jianshu.io/upload_images/11010834-29050b0ee7c4fa2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 类执行引擎
- 多态
从虚拟机角度看Java多态->（重写override）的实现原理
https://www.cnblogs.com/2014asm/p/8633660.html
- 动态委派和静态委派
