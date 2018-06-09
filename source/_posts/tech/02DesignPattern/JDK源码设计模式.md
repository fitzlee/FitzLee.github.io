---
title: JDK源码设计模式
urlname: design_pattern_JDK_source_dp
date: 2016-06-10
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 02DesignPattern
tags:
    - 设计模式
    - JDK
    - 源码
    - DesignPattern
---

## 创建型模式
**抽象工厂模式**
(通过创建的方法返回工厂本身, 可以依次创建另一个抽象/接口类型)
javax.xml.parsers.DocumentBuilderFactory#newInstance()
javax.xml.transform.TransformerFactory#newInstance()
javax.xml.xpath.XPathFactory#newInstance()

**生成器模式**
(通过创建的方法返回实例本身)
java.lang.StringBuilder#append() (非同步的)
java.lang.StringBuffer#append() (支持同步)
java.nio.ByteBuffer#put() (与之相同的有: CharBuffer, ShortBuffer, IntBuffer, LongBuffer, FloatBuffer 和 DoubleBuffer)
javax.swing.GroupLayout.Group#addComponent()
java.lang.Appendabled 的所有实现

**工厂模式**
(通过创建的方法返回一个抽象/接口类型的实现)
java.util.Calendar#getInstance()
java.util.ResourceBundle#getBundle()
java.text.NumberFormat#getInstance()
java.nio.charset.Charset#forName()
java.net.URLStreamHandlerFactory#createURLStreamHandler(String) 
(按协议返回单例对象)

**原型模式**
(通过创建的方法返回一个包含相同属性的不同实例)
java.lang.Object#clone() (类必须实现 java.lang.Cloneable)

**单例模式**
(通过创建的方法在任何时间都返回一个相同实例(通常是它本身))
java.lang.Runtime#getRuntime()
java.awt.Desktop#getDesktop()

## 结构型模式

**适配器模式**
(接收不同抽象或接口类型的一个实例, 并返回一个装饰或重写了该实例的本身/其他的抽象/接口类型的指定实现)
java.util.Arrays#asList()
java.io.InputStreamReader(InputStream) (返回一个Reader)
java.io.OutputStreamWriter(OutputStream) (返回一个Writer)
javax.xml.bind.annotation.adapters.XmlAdapter#marshal() 和 #unmarshal()

**桥接模式**
(接收不同抽象或接口类型的一个实例, 并返回一个委托或使用了该实例的本身/其他的抽象/接口类型的指定实现)
一时之间没有想到. 一个虚构的例子: new LinkedHashMap(LinkedHashSet<K>, List<V>)返回一个不可变的链式map, 并且不会克隆内部元素而是直接使用他们. java.util.Collections#newSetFromMap() 和 singletonXXX()方法是有些接近这个模式的.

**组合模式**
(通过行为式的方法接收一个相同的抽象或接口类型的实例转换为一个树状结构)
java.awt.Container#add(Component) (几乎所有Swing都是这样)
javax.faces.component.UIComponent#getChildren() (几乎所有JSF UI都是这样)

**装饰器模式**
(通过创建的方法接收一个相同的抽象或接口类型的实例, 并添加附加的行为)
java.io.InputStream, OutputStream, Reader 和 Writer的子类拥有一个持有相同类型的实例构造器.
java.util.Collections, checkedXXX(), synchronizedXXX() 和 unmodifiableXXX().
javax.servlet.http.HttpServletRequestWrapper 和 HttpServletResponseWrapper

**门面模式**
(通过行为式的方法实现, 该方法内部使用不同的独立的抽象或接口类型的实例)
javax.faces.context.FacesContext, 它内部之间使用 LifeCycle, ViewHandler, NavigationHandler 和很多无需最终能够用户关心的抽象/接口类型(可通过注入覆写).
javax.faces.context.ExternalContext, 它内部使用ServletContext, HttpSession, HttpServletRequest, HttpServletResponse等.

**享元模式**
(通过创建的方法返回一个缓存的实例, 有一些”多例”的思想)
java.lang.Integer#valueOf(int) (类似的还包括: Boolean, Byte, Character, Short 和 Long)

**代理模式**
(通过创建的方法返回指定的抽象或接口类型的实现, 该实现依次委托或使用了指定抽象或接口类型的不同实现)
java.lang.reflect.Proxy
java.rmi.*下所有API.

##行为型模式

**责任链模式**
(通过行为方法(间接地)在一个队列中调用另一个相同抽象/接口类型的相同方法)
java.util.logging.Logger#log()
javax.servlet.Filter#doFilter()

**命令模式**
(一个抽象/接口类型内的行为方法中, 调用一个不同的象/接口类型的实现的方法, 该实现已经在其创建过程中被命令实现封装)
java.lang.Runnable的所有实现
javax.swing.Action的所有实现

**解释器模式**
(行为方法返回一个有结构的并且与被提供的实例/类型不同的实例/类型; 注意parsing/formatting不是这个模式的组成部分, 而是用来确定该模式以及如何应用模式)
java.util.Pattern
java.text.Normalizer
java.text.Format的所有子类
javax.el.ELResolver的所有子类

**迭代器模式**
(行为方法按顺序的自一个队列中返回不同类型的实例)
java.util.Iterator的所有实现 (除此以外, 还有 java.util.Scanner!).
java.util.Enumeration的所有实现

**中介者模式**
(行为方法接收一个不同抽象/接口类型的实例(通常使用命令模式), 该实例委托或使用了指定实例)
java.util.Timer (所有 scheduleXXX() 方法)
java.util.concurrent.Executor#execute()
java.util.concurrent.ExecutorService (invokeXXX() 和 submit() 方法)
java.util.concurrent.ScheduledExecutorService (所有 scheduleXXX() 方法)
java.lang.reflect.Method#invoke()

**备忘录模式**
(行为方法内部改变了整个实例的状态)
java.util.Date ( setter 方法就是这么做的, Date 内部通过长整型long表示)
java.io.Serializable的所有实现
javax.faces.component.StateHolder的所有实现
观察者或发布/订阅模式(行为方法在另一个抽象/接口类型的实例上调用方法, 并取决于自身状态)
java.util.Observer/java.util.Observable (在现实世界中很少使用)
java.util.EventListener 的所有实现 (几乎所有的Swing都是这样)
javax.servlet.http.HttpSessionBindingListener
javax.servlet.http.HttpSessionAttributeListener
javax.faces.event.PhaseListener

**状态模式**
(行为方法通过外部可控的实例状态改变它的行为)
javax.faces.lifecycle.LifeCycle#execute() (通过FacesServlet控制, 行为取决于当前JSF生命周期的阶段(状态)

**策略模式**
(行为方法内一个抽象/接口类型调用不同抽象/接口类型的不同实现内的方法, 该实现以方法参数型式被传进策略实现内)
java.util.Comparator#compare(), 在Collections#sort()中执行.
javax.servlet.http.HttpServlet, service() 和所有 doXXX() 方法接收 HttpServletRequest, HttpServletResponse, 并且实现者必须处理它们(而不是作为实例变量持有它们)
javax.servlet.Filter#doFilter()

**模板模式**
(行为方法已拥有一个被抽象类型定义的”默认”行为)
java.io.InputStream, java.io.OutputStream, java.io.Reader 和 java.io.Writer 的所有非抽象方法.
java.util.AbstractList, java.util.AbstractSet 和 java.util.AbstractMap 的所有非抽象方法.
javax.servlet.http.HttpServlet, 所有 doXXX() 方法默认发送 HTTP 405 "Method Not Allowed" 错误到响应中. 你可以任意实现这些方法.

**访问者模式**
(两个不同的抽象/接口类型中包含互相接收对方抽象/接口类型的方法; 实际调用的是另一个类型的方法和它上面的其他执行所需求的策略)
javax.lang.model.element.AnnotationValue 和 AnnotationValueVisitor
javax.lang.model.element.Element 和 ElementVisitor
javax.lang.model.type.TypeMirror 和 TypeVisitor
