---
title: Andriod源码设计模式
urlname: design_pattern_android_source_dp
date: 2016-06-15
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 02DesignPattern
tags:
    - 设计模式
    - Android
    - 源码
    - DesignPattern
---

简要说明Android中设计模式，以加深对设计的理解。

1. 工厂方法模式： 接口 Iterable 中的 iterator 就是一个工厂方法， Activity 中的 onCreate 也可以看作是工厂方法

1. 抽象工厂模式： MediaPlayerFactory 是一个抽象工厂

1. 单例模式： 单例模式的应用非常广泛， Application 就是单例的， WindowManagerService 、 ActivityService 等系统级的 Service 也是单例的，具体可见 [单例模式](http://weiqianghu.github.io/2016/09/08/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F/)

1. 建造者模式 ：建造者模式在图片加载库中使用非常普遍，例如 Picasso 、 Glide 、UniversalImageLoader 中都有建造者模式的身影。在 Android 源码中 AlertDialog 中应用了建造者模式，详见 [设计模式之 Builder 模式](http://weiqianghu.github.io/2016/09/13/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B-Builder-%E6%A8%A1%E5%BC%8F/)

1. 原型模式：原型模式就是实现接口 Cloneable ，这个太多了。。。

1. 适配器模式：这个不用说了， AbsListView 和 RecyclerView 都使用了适配器模式。详见：[设计模式之适配器模式](http://weiqianghu.github.io/2016/10/10/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F/)

1. 装饰器模式：Context 、 ContextImpl 、 ContextWrapper 、ContextThemeWrapper

1. 代理模式： AIDL ，ActivityProxy（其实这也是 AIDL ）

1. 外观模式：Context 作为 Android 系统中的上帝类，封装了很多功能，这也是外观模式的应用

1. 桥接模式：Adapter 与 AdapterView 的桥接，Window 与 WindowManager 的桥接 详见：[设计模式之桥接模式](http://weiqianghu.github.io/2016/10/13/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E6%A1%A5%E6%8E%A5%E6%A8%A1%E5%BC%8F/)

1. 组合模式： View 、 ViewGroup

1. 享元模式：Message 中的 MessagePool 是用链表实现的。

1. 策略模式：动画中的 InterPolator 和 TypeEvaluator 。详见[Android 动画分析](http://weiqianghu.github.io/2016/08/23/Android-%E5%8A%A8%E7%94%BB%E5%88%86%E6%9E%90/)

1. 模板方法模式: 模板模式的应用非常广泛， Android 中 AsyncTask 的几个回调可以看作模板。

1. 观察者模式： AbsListView 和 RecyclerView 都使用了观察者模式，详见[设计模式之观察者模式](http://weiqianghu.github.io/2016/09/27/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F/)

1. 迭代子模式： 这个不用说了，在 JDK 中的集合类都是迭代子模式。

1. 责任链模式： View 中对于事件的分发处理可以看作是责任链模式。详见[利用责任链模式实现加载不同来源的数据](http://weiqianghu.github.io/2016/07/14/%E5%88%A9%E7%94%A8%E8%B4%A3%E4%BB%BB%E9%93%BE%E6%A8%A1%E5%BC%8F%E5%AE%9E%E7%8E%B0%E5%8A%A0%E8%BD%BD%E4%B8%8D%E5%90%8C%E6%9D%A5%E6%BA%90%E7%9A%84%E6%95%B0%E6%8D%AE/)

1. 命令模式： Android 事件的底层 NotifyArgs 就是一个命令对象。

1. 备忘录模式： Android 的状态保存， onSaveInstanceState 、 onRestoreInstaceState

1. 状态模式： WIFI 管理

1. 访问者模式： Java 注解 APT

1. 中介者模式： KeyGuard 功能的实现

1. 解释器模式： PackageParser 中有解释器模式的影子


**Copy from**
http://weiqianghu.github.io/2016/10/13/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B23%E7%A7%8D%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E6%80%BB%E7%BB%93/
