---
title: kotlin简明教程
urlname: java_jdk_base_kotlin_learn
#date: 2016-03-06
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 00Java&JDK
tags:
    - java
    - jdk
    - kotlin
    - stream
---

## 第一周

**Day 1：可见性**

在 Kotlin 中一切都是默认 public 的。并且 Kotlin 还有一套丰富的可见性修饰符，例如：private, protected, internal。它们每个都以不同的方式降低了可见性。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec2a13b39894?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 2：Elvis 操作符

需要处理代码中的空值？可以使用 elvis 操作符，避免您的 “空情况” (null-erplate)。这只是替换空作为值或者返回事件情况的一个小语法。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec2f9eb04904?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 3：String 模板

格式化字符串？将$放在变量名的前面去表达字符串中的变量和表达式。使用 ${expression} 求表达式的值。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec4c1c9cd2b9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 4：When 表达式

强大的 switch！Kotlin 的 When 表达几乎可以匹配任何东西。字面值，枚举，数字范围。您甚至可以调用任意函数！

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec541988eea1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 5：循环，范围表达式与解构

for 循环在与其他两种 Kotlin 特性一起使用时可以获得超级能力：范围表达式和解构。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec5c30cf11d9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 6：属性

在 Kotlin 中，类可以具有可变和只读属性，默认情况下生成 getter 和 setter。如果需要，您也可以实现自定义的。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec674c53d9b2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 7：解构声明

Android KTX 使用解构来分配颜色的组件值。您可以在您的类中使用解构，或者扩展现有的类来添加解构。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec6d40bd751b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第一周学习小结：

本周以基本知识为主：处理空错误，简化循环和条件，属性，解构架。下一周我们将会深入探索 Kotlin 的更多功能。


<!-- more -->

## 第二周

**Day 8：简单的 bundle**

准备去通过简洁的方式去创建 bundle，不调用 putString，putInt，或它们的 20 个方法中的任何一个。一个调用让您生成一个新的 bundle，它甚至可以处理 Arrays。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec789e8747bb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 9：Parcelize

喜欢 Parcelable 的速度，但不喜欢写所有的代码？和 @Parcelize 打个招呼。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec7eaf40465d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 10：Data 类和 equality

可以创建具有一个具有处理数据的类吗？将它们标记为 "Data" 类。并默认实现生成 equals() 方法 - 相当于 hashCode()，toString() 和copy()，并检查结构是否相等。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec84ec97c844?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 11：简化 postDelay

Lambda 非常贴心，使用最后一个参数调用语法您可以取消回调，Callable 和 Runnable，例如 Android KTX 贴心的用一个小包装来处理 postDelayed。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec8b0ff8aec9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 12：默认参数

方法参数的数量是否太多？在函数中指定默认参数值。使用命名参数使代码更具可读性。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec934ff7ebf8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 13：从 Java 编程语言调用 Kotlin

在同一个项目中使用 Kotlin 和 Java？您有没有顶级功能或属性的课程？默认情况下，编译器将生成类名称 YourFileKt。通过使用 @file：JvmName 注释文件来更改它。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aec99f8891332?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 14：在没有迭代器的情况下迭代类型

迭代器用在了有趣的地方！Android KTX 将迭代器添加到 viewGroup 和 sparseArray。要定义迭代器扩展请使用 operator 关键字。 Foreach 循环将使用扩展名！

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aeca124a14688?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第二周学习小结

这周我们更深入学了 Kotlin 的特性：简洁 bundle，迭代，Data，postDelay，默认参数，序列化。下一周我们会了解更多的 Kotlin 特性并且开始探索 Android KTX。

## 第三周

**Day 15：sealed 类**

Kotlin 的 sealed 类可以让您轻松的处理错误数据，当结合 LiveData 您可以用一个 LiveData 同时代表成功和失败的路径，这比用两个不变量要好。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aecad9851213b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="704" height="293"></svg>)

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="704" height="403"></svg>)

Day 16：懒加载

懒加载是个好东西！通过使用懒加载，可以省去昂贵的属性初始化的成本直到它们真正需要。计算值然后保存并为了未来的任何时候的调用。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aecbd08571843?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 17：Lateinit

Android 中，在 onCreate 或者其它的回调初始化对象，但在 Kotlin 中不为空的对象必须初始化。那么怎么办呢？可以输入 lateinit。来承诺最终将会初始化。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aecc26b8af2b3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 18：要求 (require) 和检查 (check)

您方法的参数是有效的吗？用 require 在使用前可以检查它们，如果它们是无效的将会抛出 IllegalArgumentException。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aecc7fef37a65?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

您的封闭类的状态是否正确？可以使用 check 来验证。如果检查的值为 false，它将抛出 IllegalStateException。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aecd62b91ab89?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 19：内联 (InLine)

等不及要使用 lambdas 来生成一个新的接口？kotlin 可以使您制定一个 inline 的方法 -- 这意味着调用将替换方法体，用很非常简单的方法来生成 lambda 的接口。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aecdd7599cec4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 20：运算符重载

用操作符重载快更快速写 Kotlin。像 Path，Range或 SpannableStrings 这样的对象允许像加法或减法这样的操作。通过 Kotlin，您可以实现自己的操作符。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aece5ba25ab09?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 21：顶级方法和参数

类的实用方法？将它们添加到源文件的顶层。在 Java 中，它们被编译为该类的静态方法。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aecebfc5bfb31?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aecf2ebb0c0d4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第三周学习小结：

本周主要讨论一些基本的 Kotlin 特性，如运算符重载，内联，运算符重载，懒加载，以及非常强大的 inLine,并展示了使用 Android KTX 处理内容值，捆绑包和回调时如何编写更简洁的代码。

## 第四周

**Day 22：简单的内容值**

将 ContentValues 的强大功能与 Kotlin 的简洁性相结合。使用 Android KTX 只传递一个 Pair <StringKey，Value> 创建 ContentValues。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aecff265f43fd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 23：DSLs

特定于域的语言可以通过使用类型安全的构建器来完成。它们为简化 API 做出贡献；您也可以自己借助扩展 lambdas 和类型安全构建器等功能构建它们。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed0883f3c373?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed10bedae793?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed18bf09c5e6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 24：具体化

具体化的概念例子：Android KTX 中的 Context.systemService() 使用泛化来通过泛型传递 “真实” 类型。没有通过 getSystemService。

**Android KTX：Context.systemService()**

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed268c361dba?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 25：Delegates

通过 by 用您的工作委托给另一个类。通过类继承，并将属性访问器逻辑与委托者属性重用。

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed2d098e2c45?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 26：延期方法

没有更多的 Util 类。通过使用扩展功能扩展类的功能。把您要扩展的类的名字放在您添加的方法的名字前面。

**扩展功能的一些特性：**

- 不是成员函数
- 不要以任何方式修改原始类
- 通过静态类型信息解决编译时间
- 会被编译为静态函数
- 不要多态性

例如：String.toUri()

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="708" height="156"></svg>)

Day 27：Drawable.toBitmap() 轻松转换

如果您曾经将 Drawable 转换为 Bitmap，那么您知道需要多少样？Android KTX 具有一系列功能，可以使您的代码在使用图形包中的类时更加简洁。

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="700" height="148"></svg>)

Day 28：Sequences, lazy 和 generators

序列是从未存在的列表。序列是迭代器的表亲，一次只能懒散地产生一个值。这在使用 map 和 fifter 时非常重要 - 它们将创建序列，而不是为每一步都复制列表！

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed5a61127dc9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed618aa7e0ae?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 29：更简单的 Spans

功能强大但很难使用 - 这就是 Spans API 感觉的文本样式。 Android KTX 为一些最常见的 span 添加了扩展功能，并使 API 更易于使用。Android KTX: 可跨越字符串的构建器

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed685c007a9a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 30：updatePadding 扩展

通过默认参数扩展现有的 API 通常会让每个人都高兴。 Android KTX 允许您使用默认参数在视图的一侧设置填充。一行代码可以节省很多代码！Android KTX: View.updatePadding

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed6f10af5c1b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Day 31：范围外 run，let，with，apply

让我们运行一些标准的 Kotlin 函数！简短而强大，run，let，with 和 appy 都有一个接收器 (this)，可能有一个参数 (it) 并可能有一个返回值。差异如下：

**run** 

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed77bdef7959?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

let

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed7e2328ffe6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

with

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed845cba0212?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

apply

![img](https://user-gold-cdn.xitu.io/2018/5/30/163aed897db47360?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第四周学习小结：

本周我们涵盖了更多语言特性，如 interop,refied 和 sequence，并且在 Android KTX，展示了它帮助您编写简洁易读的代码的一些方法。我们也讨论了高级特性：领域特定语言 (DSL)。

 

参考：

https://juejin.im/post/5b0e1a6ef265da092a2b773f#heading-1

 

 

 