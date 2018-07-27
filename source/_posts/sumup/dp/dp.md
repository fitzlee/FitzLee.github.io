---
title: 设计模式UML和23种分类解析
urlname: design_pattern_uml_23gof_analysis
#date: 2016-06-07
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 02DesignPattern
tags:
    - 设计模式
    - 汇总
    - uml
    - 23dp
    - 分析精华
    - DesignPattern
---

# 设计模式之 UML 类图

## **前言**

为什么要学习设计模式？

**个人觉得设计模式传授的是一种思想，是一种脱离语言的编程习惯。**对于一个没有太多经验的程序员，如何写出 **简洁优雅，可复用性高，可扩展性强，高内聚低耦合** 的代码至关重要。学习别人的设计模式就是为了在没有经验的情况下写出一手不错的代码，只看不写并不能深刻体验到设计模式的巧妙之处。

 

设计模式讲的是别人千锤百炼出来的精华，如果有那么一天你再次看设计模式觉得，这些没什么特别的，那么说明你已经走上正轨，你的编程习惯已经向设计模式靠拢了。

这是我学习设计模式的开篇第一章，主要介绍下 UML 类图。在之后的文章我也会介绍设计模式的一些基本原则以及 23 种设计模式，并给出详细的代码说明。

## **UML 类图**

学习设计模式必定需要先读懂 UML 类图，下面就谈谈具体 UML 类图中的概念。

首先是关于类本身，下面我以人为例，先看 UML 图：

![img](https://pic4.zhimg.com/80/v2-92bf9082359ed12dfae4476d7286674c_hd.jpg)

用 Java 代码可表示为:

```
class Student {
    private String name;
    public String getName() {
	return name;
    }
    public void takeExam(Course course) {
        course.test();
    }
}

class Course {
    private String courseName;
    public void test() {
        // take exam...
    }
}
```

- 类名叫做 Student 和 Course
- \+ 代表 public 公共，- 代表 private 私有，# 代表 protected
- 成员变量类型写在前，参数名称写在后
- 函数传递参数，参数名写在前，类型写在后
- 函数返回值写在函数签名的后面
- 两个类之间若存在关系，可使用箭头进行关联，具体关联规则在下文介绍
- 箭头上的数字代表 1 个学生可以不参加课程，也可以无限制参加各种课程
- 1 代表一个，0..* 代表 0 个到无限个

 

**在 Java 或其它面向对象设计模式中，类与类之间主要有 6 种关系，他们分别为：依赖，关联，聚合，组合，继承，实现。他们的耦合度依次增强。**

## 1. 依赖 (Dependency)

依赖关系的定义为：对于两个相对独立的对象，当一个对象负责构造另一个对象的实例，或者依赖另一个对象的服务时，这两个对象之间主要体现为依赖关系。

可以简单的理解为：类 A 使用到了类 B，而这种使用关系具有偶然性，临时性，非常弱的，但是 B 类中的变化会影响到类 A，比如某个学生要用笔写字，学生与笔的关系就是一种依赖关系，如果笔没水了，那学生就不能写字了(B 类的变化会影响类 A) 或者换另一只笔继续写字(临时性体现)。

思考下面这样的场景：

你是一名出租车司机，每天开着公司给你分配的车去载客，而每天出租车可能不同，我只是个司机，公司给我什么车我就开什么车，我使用这个车。

**具体 UML 类图表现为：**

![img](https://pic4.zhimg.com/80/v2-dce9c2eb387354bb7de9a3a47d5f3ed5_hd.jpg)

Java 代码：

```
class Driver {
    public void drive(Car car) {
	car.run();
    }
}
class Car {
    public void run(){}
} 
```

依赖关系不一定表现为形参，一共可以有三种表现形式:

```
class Driver {
    //通过形参方式发生依赖关系 
    public void drive1(Car car) {
	car.run();
    }
     //通过局部变量发生依赖关系
    public void drive2() {
	Car car = new Car();
	car.run();
    }
    //通过静态变量发生依赖关系
    public void drive3() {
	Car.run();
    }
}
```

<!-- more -->

## 2. 关联 (Association)

关联关系的定义为：对于两个相对独立的对象，当一个对象的实例与另一个对象的一些特定实例存在固定的对应关系时，这两个对象之间为关联关系。

它体现的两个类中一种强依赖关系，比如我和我的朋友，这种关系比依赖更强，不存在依赖关系中的偶然性，关系也不是临时的，一般是长期性的。

关联关系分为单向关联和双向关联：

1. 在 Java 中，单向关联表现为：类 A 当中使用了 类 B，其中类 B 是作为类 A 的成员变量。
2. 双向关联表现为: 类 A 当中使用类 B 作为成员变量，同时类 B 中也使用了类 A 作为成员变量。

 

根据可以上面的例子可以修改为以下的场景：

我是一名老司机，车是我自己的，我拥有这辆车，平时也会用着辆车去载客人。

**用 UML 类图表示为:**

![img](https://pic4.zhimg.com/80/v2-2b4dad2a2fa58923eaf6cec0460eea72_hd.jpg)

**双向关联的话箭头可以省略。**

用 Java 代码表示为：

```
class Driver {
    private Car car = new Car();
    public void drive() {
	car.run();
    }
}
class Car {
    public void run(){}
}
```

依赖和关联区别：

- 用锤子修了一下桌子，我和锤子之间就是一种依赖，我和我的同事就是一种关联。
- 依赖是一种弱关联，只要一个类用到另一个类，但是和另一个类的关系不是太明显的时候（可以说是“uses”了那个类），就可以把这种关系看成是依赖，依赖也可说是一种偶然的关系，而不是必然的关系。
- 关联是类之间的一种关系，例如老师教学生，老公和老婆这种关系是非常明显的。依赖是比较陌生，关联是我们已经认识熟悉了。

 

## 3.聚合 (Aggregation)

聚合关系是关联关系的一种，耦合度强于关联，他们的代码表现是相同的，仅仅是在语义上有所区别：关联关系的对象间是相互独立的，而聚合关系的对象之间存在着包容关系，他们之间是“整体-个体”的相互关系。

聚合关系中作为成员变量的类一般使用 set 方法赋值。

**用 UML 类图表示为：**

![img](https://pic3.zhimg.com/80/v2-ac8ead9f8e60ed1abebf847d65c1633e_hd.jpg)

Java 代码:

```
class Driver {
    private Car car = null;
    public void drive() {
	car.run();
    }
    public void setCar(Car c){
	car = c;
    }
}
class Car {
    public void run(){}
}
```

## 4. 组合 (Composition)

相比于聚合，组合是一种耦合度更强的关联关系。存在组合关系的类表示“整体-部分”的关联关系，“整体”负责“部分”的生命周期，他们之间是共生共死的；并且“部分”单独存在时没有任何意义。

对比与聚合关系，我们可以将前面的例子变为下面的场景：

1. 车是一辆私家车，是司机财产的一部分，**强调的是人财产的部分性**，则相同的代码即可表示聚合关系。
2. 车是司机必须有的财产，要想成为一个司机必须要现有财产，车要是没了，司机也不想活了。而且司机要是不干司机了，这车也就没了。

**所以，关联、聚合、组合只能配合语义，结合上下文才能够判断出来，而只给出一段代码让我们判断是关联，聚合，还是组合关系，则是无法判断的。**

**用 UML 类图表示为:**

![img](https://pic2.zhimg.com/80/v2-35f373e86c1f44ba4f3ff020ff52ff8c_hd.jpg)

```
class Driver {
    private Car car = null;
    public Driver(Car car) {
	this.car = car;
    }
    public void drive() {
	car.run();
    }
}
class Car {
    public void run(){}
}
```

再举一个恰当点的例子：

人和灵魂，身体之间是组合关系，当人的生命周期开始时，必须同时拥有灵魂和肉体，当人的生命周期结束时，灵魂肉体随之消亡；无论是灵魂还是肉体，都不能单独存在，他们必须作为人的组成部分存在。

**用 UML 类图表示为**

![img](https://pic4.zhimg.com/80/v2-86ac2db5174fcf34875a9e872ed4d27e_hd.jpg)

## 5. 继承 (Generalization)

继承表示类与类 (或者接口与接口) 之间的父子关系。在 Java 中，用关键字 extends 表示继承关系。

思考下面的场景：

人是一种高级动物，不仅可以像动物可以吃和睡觉，而且还可以学习。

**用 UML 类图表示为:**

![img](https://pic1.zhimg.com/80/v2-a5ab6c82e96b2119bf658f489886e888_hd.jpg)

Java 代码表示为:

```
class Animal {
    public void eat(){}
    public void sleep(){}
}
class People extends Animal {
    public void study(){}
}
```

## 6. 实现 (Implementation)

表示一个类实现一个或多个接口的方法。接口定义好操作的集合，由实现类去完成接口的具体操作, 在 Java 中使用 implements 表示。在 Java 中，如果实现了某个接口，那么就必须实现接口中所有的方法。

比如一个人可以吃饭和学习，那么就可以定义一个人的接口。让具体的人去实现它。

**用 UML 类图表示为：**

![img](https://pic4.zhimg.com/80/v2-f489103646695deda45ac115321d7634_hd.jpg)

 

Java 代码：

```
interface IPeople {
    public void eat();
    public void study();
}
class People implements IPeople {
    public void eat(){
    }
    public void study(){
    }
}
```







# 意图、UML类图

## 创建模式

> ### 创建型模式
>
> 与对象的创建有关。

#### 工厂方法

> 定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method 使一个类的实例化延迟到其子类。

[![简单工厂UML类图](http://jayfeng.com/images/dp_uml_factory_simple.png)](http://jayfeng.com/images/dp_uml_factory_simple.png)
[![工厂方法UML类图](http://jayfeng.com/images/dp_uml_factory_method.png)](http://jayfeng.com/images/dp_uml_factory_method.png)

#### 抽象工厂

> 提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

[![抽象工厂UML类图](http://jayfeng.com/images/dp_uml_factory_abstract.png)](http://jayfeng.com/images/dp_uml_factory_abstract.png)

#### 建造者模式

> 将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

[![建造者模式UML类图](http://jayfeng.com/images/dp_uml_builder.png)](http://jayfeng.com/images/dp_uml_builder.png)

#### 原型模式

> 用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

[![原型模式UML类图](http://jayfeng.com/images/dp_uml_prototype.png)](http://jayfeng.com/images/dp_uml_prototype.png)

#### 单例模式

> 保证一个类仅有一个实例，并提供一个访问它的全局访问点。

[![单例模式UML类图](http://jayfeng.com/images/dp_uml_singleton.png)](http://jayfeng.com/images/dp_uml_singleton.png)

> ### 结构型模式
>
> 处理类或对象的组合。

## 结构型模式

#### 适配器模式

> 将一个类的接口转换成客户希望的另外一个接口。Adapter 模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

[![类适配器UML类图](http://jayfeng.com/images/dp_uml_adapter_class.png)](http://jayfeng.com/images/dp_uml_adapter_class.png)
[![对象适配器UML类图](http://jayfeng.com/images/dp_uml_adapter_object.png)](http://jayfeng.com/images/dp_uml_adapter_object.png)

#### 桥接模式

> 将抽象部分与它的实现部分分离，使它们都可以独立地变化。

[![桥接模式UML类图](http://jayfeng.com/images/dp_uml_bridge.png)](http://jayfeng.com/images/dp_uml_bridge.png)

#### 组合模式

> 将对象组合成树形结构以表示“部分 -整体”的层次结构。 Composite使得用户对单个对象和组合对象的使用具有一致性。

[![组合模式UML类图](http://jayfeng.com/images/dp_uml_composite.png)](http://jayfeng.com/images/dp_uml_composite.png)

#### 装饰者模式

> 动态地给一个对象添加一些额外的职责。就增加功能来说, Decorator模式相比生成子类更为灵活。

[![装饰者模式UML类图](http://jayfeng.com/images/dp_uml_decorator.png)](http://jayfeng.com/images/dp_uml_decorator.png)

#### 门面模式

> 为子系统中的一组接口提供一个一致的界面，Facade模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。

[![门面模式UML类图](http://jayfeng.com/images/dp_uml_facade.png)](http://jayfeng.com/images/dp_uml_facade.png)

#### 享元模式

> 运用共享技术有效地支持大量细粒度的对象。

[![享元模式UML类图](http://jayfeng.com/images/dp_uml_flyweight.png)](http://jayfeng.com/images/dp_uml_flyweight.png)

#### 代理模式

> 为其他对象提供一种代理以控制对这个对象的访问。

[![代理模式UML类图](http://jayfeng.com/images/dp_uml_proxy.png)](http://jayfeng.com/images/dp_uml_proxy.png)

## 行为型模式

> ### 行为型模式
>
> 对类或对象怎样交互和怎样分配职责进行描述。

#### 解释器模式

> 用一个中介对象来封装一系列的对象交互。中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

[![解释器模式UML类图](http://jayfeng.com/images/dp_uml_interpreter.png)](http://jayfeng.com/images/dp_uml_interpreter.png)

#### 模板方法模式

> 定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。TemplateMethod 使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

[![模板方法模式UML类图](http://jayfeng.com/images/dp_uml_template_method.png)](http://jayfeng.com/images/dp_uml_template_method.png)

#### 职责链模式

> 使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

[![职责链模式UML类图](http://jayfeng.com/images/dp_uml_chainofresponsibility.png)](http://jayfeng.com/images/dp_uml_chainofresponsibility.png)

#### 命令模式

> 将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可撤消的操作。

[![命令模式UML类图](http://jayfeng.com/images/dp_uml_command.png)](http://jayfeng.com/images/dp_uml_command.png)

#### 迭代器模式

> 提供一种方法顺序访问一个聚合对象中各个元素, 而又不需暴露该对象的内部表示。

[![迭代器模式UML类图](http://jayfeng.com/images/dp_uml_iterator.png)](http://jayfeng.com/images/dp_uml_iterator.png)

#### 中介者模式

> 用一个中介对象来封装一系列的对象交互。中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

[![中介者模式UML类图](http://jayfeng.com/images/dp_uml_mediator.png)](http://jayfeng.com/images/dp_uml_mediator.png)

#### 备忘录模式

> 在不破坏封装性的前提下,捕获一个对象的内部状态,并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。

[![备忘录模式UML类图](http://jayfeng.com/images/dp_uml_memento.png)](http://jayfeng.com/images/dp_uml_memento.png)

#### 观察者模式

> 定义对象间的一种一对多的依赖关系 ,当一个对象的状态发生改变时 , 所有依赖于它的对象都得到通知并被自动更新。

[![观察者模式UML类图](http://jayfeng.com/images/dp_uml_observer.png)](http://jayfeng.com/images/dp_uml_observer.png)

#### 状态模式

> 允许一个对象在其内部状态改变时改变它的行为。对象看起来似乎修改了它的类。

[![状态模式UML类图](http://jayfeng.com/images/dp_uml_state.png)](http://jayfeng.com/images/dp_uml_state.png)

#### 策略模式

> 定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。本模式使得算法可独立于使用它的客户而变化。

[![策略模式UML类图](http://jayfeng.com/images/dp_uml_strategy.png)](http://jayfeng.com/images/dp_uml_strategy.png)

#### 访问者模式

> 表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作。

[![访问者模式UML类图](http://jayfeng.com/images/dp_uml_visitor.png)](http://jayfeng.com/images/dp_uml_visitor.png)

[#设计模式](http://jayfeng.com/tags/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/) [#面向对象](http://jayfeng.com/tags/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1/) [#OOP](http://jayfeng.com/tags/OOP/) [#意图](http://jayfeng.com/tags/%E6%84%8F%E5%9B%BE/) [#UML](http://jayfeng.com/tags/UML/)



# Android系统编程思想：设计模式

**关于作者**

> 郭孝星，程序员，吉他手，主要从事Android平台基础架构方面的工作，欢迎交流技术方面的问题，可以去我的[Github](https://github.com/guoxiaoxing)提issue或者发邮件至[guoxiaoxingse@163.com](mailto:guoxiaoxingse@163.com)与我交流。

**文章目录**

提到设计模式，大家并不陌生，我们之前在分析Android源码的时候也有提及，但都比较零散，不成系统。今天的这篇文章就来系统的总结一下23种 设计模式的模式定义与实现方式，让读者有一个整体上的模式。

什么是设计模式？![thinking](https://assets-cdn.github.com/images/icons/emoji/unicode/1f914.png)

> 通俗来讲，设计模式就是针对某一种特殊场景而给出的标准解决方案，它是前辈们的经验性总结，也是实现软件工程化的基础，良好的设计模式应用 可以是我们的软件变得更加健壮可维护。

设计模式按照类型划分可以分为三大类，如下所示：

- 创建型设计模式：如同它的名字那样，它是用来解耦对象的实例化过程。
- 结构型设计模式：将类和对象按照一定规则组合成一个更加强大的结构体。
- 行为型设计模式：定义类和对象的交互行为。

23种设计模式划分如下图所示：

[![img](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/program/design_pattern.png)](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/program/design_pattern.png)

注：23种设计模式很多小伙伴都烂熟于心，但是真正编程实践的时候未必会想的起来，这其实是一个潜移默化的过程，在看设计模式的时候，尽量多动手写一写，其中 手写（不借助IDE）的效果最佳，可以加深理解，理解的深了，编程的时候自然就可以想的到去应用。

## 一 创建型设计模式

> 创建型设计模式主要用来解耦对象的实例化过程，控制实例的生成。

创建型设计模式一共有六种，如下所示：

### 1.1 单例模式

模式定义

> 当系统中只需要一个实例或者一个全局访问点的时候可以使用单例模式。

- 优点：节省系统创建对象的资源，提高了系统效率，提供了统一的访问入口，可以严格控制用户对该对象的访问。
- 缺点：只有一个对象，积累的职责过重，违背了单一职责原则。构造方法为private，无法继承，扩展性较差。

单例模式的实现由很多种，如下所示：

1. 懒汉式单例
2. 双层校验锁单例
3. 容器单例
4. 静态内部类单例
5. 枚举单例

其中静态内部类单例和枚举单例都是单例模式最佳的实现，但是出于便利性的考量，双层校验锁的实现应用的更为广泛，如下所示：

```
public class DoubleCheckSingleton {
    
    // volatile关键字保证了：① instance实例对于所有线程都是可见的 ② 禁止了instance 
    // 操作指令重排序。
    private volatile static DoubleCheckSingleton instance;

    private DoubleCheckSingleton() {
    }

    public static DoubleCheckSingleton getInstance() {
        // 第一次校验，防止不必要的同步。
        if (instance == null) {
            // synchronized关键字加锁，保证每次只有一个线程执行对象初始化操作
            synchronized (DoubleCheckSingleton.class) {
                // 第二次校验，进行判空，如果为空则执行初始化
                if(instance == null){
                     instance = new DoubleCheckSingleton();
                }
            }
        }
        return instance;
    }
}
```

关于双层校验锁单例为何能实现JVM单例，它的要点在于两次判空和synchronized、volatile关键字，具体原理已经写在上方的注释里，这里 我们单独说一下volatile关键字。

说明我们说到，volatile关键字禁止了instance 操作指令重排序，我们来解释一下，我们知道instance = new DoubleCheckSingleton()这个操作 在汇编指令里大致会做三件事情：

1. 给我们知道instance分配内存。
2. 调用DoubleCheckSingleton()构造方法。
3. 将构造的对象赋值给instance。

但是在真正执行的时候，Java编译器是允许指令乱序执行的（编译优化），所以上述3步的顺序得不到保证，有可能是132，试想一下，如果线程A没有执行第2步，先执行了 第3步，而恰在此时，线程B取走了instance对象，在使用instance对象时就会有问题，双层校验锁单例失败，而volatile关键字可以禁止指令重排序从而解决这个问题。

单例模式的另一个问题就是多进程的情况下的失败问题，因为JVN里的单例是基于一个虚拟机进程的，这个时候通常的做法就是让这个单例支持跨进程调用，这个在 Android里一般用AIDL实现。

### 1.2 建造者模式

#### 模式定义

> 封装一个复杂对象的构建过程，可以按照流程来构建对象。

- 优点：它可以将一个复杂对象的构建与表示相分离，同一个构建过程，可以构成出不同的产品，简化了投建逻辑。
- 缺点：如果构建流程特别复杂，就是导致这个构建系统过于庞大，不利于控制。

建造者模式的实现，也十分简单，如下所示：

#### code：

```
public class Product {

    private String board;
    private String display;
    private String os;

    public String getBoard() {
        return board;
    }

    public String getDisplay() {
        return display;
    }

    public String getOs() {
        return os;
    }

    private Product(Builder builder) {
        // 进行构建
        this.board = builder.board;
        this.display = builder.display;
        this.os = builder.os;
    }

    public static class Builder {
        // 建造者模式还可以设置默认值
        private String board = "default value";
        private String display = "default value";
        private String os = "default value";

        public void setBoard(String board) {
            this.board = board;
        }

        public void setDisplay(String display) {
            this.display = display;
        }

        public void setOs(String os) {
            this.os = os;
        }


        public Product build() {
            return new Product(this);
        }
    }
}
```

#### Android中的应用

在Android源码中，我们最常用到的Builder模式就是AlertDialog.Builder， 使用该Builder来构建复杂的AlertDialog对象。简单示例如下 :

```
    //显示基本的AlertDialog  
    private void showDialog(Context context) {  
        AlertDialog.Builder builder = new AlertDialog.Builder(context);  
        builder.setIcon(R.drawable.icon);  
        builder.setTitle("Title");  
        builder.setMessage("Message");  
        builder.setPositiveButton("Button1",  
                new DialogInterface.OnClickListener() {  
                    public void onClick(DialogInterface dialog, int whichButton) {  
                        setTitle("点击了对话框上的Button1");  
                    }  
                });  
        builder.setNeutralButton("Button2",  
                new DialogInterface.OnClickListener() {  
                    public void onClick(DialogInterface dialog, int whichButton) {  
                        setTitle("点击了对话框上的Button2");  
                    }  
                });  
        builder.setNegativeButton("Button3",  
                new DialogInterface.OnClickListener() {  
                    public void onClick(DialogInterface dialog, int whichButton) {  
                        setTitle("点击了对话框上的Button3");  
                    }  
                });  
        builder.create().show();  // 构建AlertDialog， 并且显示
    } 
```

结果 : [![result](https://github.com/simple-android-framework/android_design_patterns_analysis/raw/master/builder/mr.simple/images/result.png)](https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/builder/mr.simple/images/result.png)

下面我们看看AlertDialog的相关源码 :

```
// AlertDialog
public class AlertDialog extends Dialog implements DialogInterface {
    // Controller, 接受Builder成员变量P中的各个参数
    private AlertController mAlert;

    // 构造函数
    protected AlertDialog(Context context, int theme) {
        this(context, theme, true);
    }

    // 4 : 构造AlertDialog
    AlertDialog(Context context, int theme, boolean createContextWrapper) {
        super(context, resolveDialogTheme(context, theme), createContextWrapper);
        mWindow.alwaysReadCloseOnTouchAttr();
        mAlert = new AlertController(getContext(), this, getWindow());
    }

    // 实际上调用的是mAlert的setTitle方法
    @Override
    public void setTitle(CharSequence title) {
        super.setTitle(title);
        mAlert.setTitle(title);
    }

    // 实际上调用的是mAlert的setCustomTitle方法
    public void setCustomTitle(View customTitleView) {
        mAlert.setCustomTitle(customTitleView);
    }
    
    public void setMessage(CharSequence message) {
        mAlert.setMessage(message);
    }

    // AlertDialog其他的代码省略
    
    // ************  Builder为AlertDialog的内部类   *******************
    public static class Builder {
        // 1 : 存储AlertDialog的各个参数, 例如title, message, icon等.
        private final AlertController.AlertParams P;
        // 属性省略
        
        /**
         * Constructor using a context for this builder and the {@link AlertDialog} it creates.
         */
        public Builder(Context context) {
            this(context, resolveDialogTheme(context, 0));
        }


        public Builder(Context context, int theme) {
            P = new AlertController.AlertParams(new ContextThemeWrapper(
                    context, resolveDialogTheme(context, theme)));
            mTheme = theme;
        }
        
        // Builder的其他代码省略 ......

        // 2 : 设置各种参数
        public Builder setTitle(CharSequence title) {
            P.mTitle = title;
            return this;
        }
        
        
        public Builder setMessage(CharSequence message) {
            P.mMessage = message;
            return this;
        }

        public Builder setIcon(int iconId) {
            P.mIconId = iconId;
            return this;
        }
        
        public Builder setPositiveButton(CharSequence text, final OnClickListener listener) {
            P.mPositiveButtonText = text;
            P.mPositiveButtonListener = listener;
            return this;
        }
        
        
        public Builder setView(View view) {
            P.mView = view;
            P.mViewSpacingSpecified = false;
            return this;
        }
        
        // 3 : 构建AlertDialog, 传递参数
        public AlertDialog create() {
            // 调用new AlertDialog构造对象， 并且将参数传递个体AlertDialog 
            final AlertDialog dialog = new AlertDialog(P.mContext, mTheme, false);
            // 5 : 将P中的参数应用的dialog中的mAlert对象中
            P.apply(dialog.mAlert);
            dialog.setCancelable(P.mCancelable);
            if (P.mCancelable) {
                dialog.setCanceledOnTouchOutside(true);
            }
            dialog.setOnCancelListener(P.mOnCancelListener);
            if (P.mOnKeyListener != null) {
                dialog.setOnKeyListener(P.mOnKeyListener);
            }
            return dialog;
        }
    }
    
}
```



#### Java应用



### 1.3 原型模式

#### 模式定义

> 当某个对象的数据结构或者构建过程特别复杂，频繁的构建势必会消耗系统性能，这个时候我们采用原型模式对原有的 对象进行克隆，构建新的对象。

- 优点：直接克隆原有实例生成新的实例，免去了复杂的构建过程，节省了系统资源。
- 缺点：

#### UML类图

[![uml](https://github.com/simple-android-framework/android_design_patterns_analysis/raw/master/prototype/mr.simple/images/prototype-uml.png)](https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/prototype/mr.simple/images/prototype-uml.png)

##### 角色介绍

- Client : 客户端用户。
- Prototype : 抽象类或者接口，声明具备clone能力。
- ConcretePrototype : 具体的原型类.

实现原型模式也很简单，主需要声明实现loneable接口，然后覆写Object的clone()方法接口。

#### code

```
public class Person implements Cloneable{

    public int age;
    public String name;

    @Override
    public Person clone() throws CloneNotSupportedException {
        return (Person) super.clone();
    }
}
```

原型模式要注意深拷贝和浅拷贝的问题，Object的clone()方法默认是钱拷贝，即对于引用对象拷贝的地址而不是值，所以要实现 深拷贝，在clone()方法里对于引用对象也有调用一下clone()方法，并且引用对象也要实现Cloneable接口和覆写clone()方法。

```
package com.dp.example.builder;


package com.dp.example.prototype;

import java.util.ArrayList;
import java.util.List;

/**
 * 文档类型, 扮演的是ConcretePrototype角色，而cloneable是代表prototype角色
 * 
 * @author mrsimple
 */
public class WordDocument implements Cloneable {
    /**
     * 文本
     */
    private String mText;
    /**
     * 图片名列表
     */
    private ArrayList<String><string> mImages = new ArrayList<String><string>();

    public WordDocument() {
        System.out.println("----------- WordDocument构造函数 -----------");
    }

    /**
     * 克隆对象
     */
    @Override
    protected WordDocument clone() {
        try {
            WordDocument doc = (WordDocument) super.clone();
            doc.mText = this.mText;
            doc.mImages = this.mImages;
            return doc;
        } catch (Exception e) {
        }

        return null;
    }

    public String getText() {
        return mText;
    }

    public void setText(String mText) {
        this.mText = mText;
    }

    public List<string> getImages() {
        return mImages;
    }

    /**
     * @param img
     */
    public void addImage(String img) {
        this.mImages.add(img);
    }

    /**
     * 打印文档内容
     */
    public void showDocument() {
        System.out.println("----------- Word Content Start -----------");
        System.out.println("Text : " + mText);
        System.out.println("Images List: ");
        for (String imgName : mImages) {
            System.out.println("image name : " + imgName);
        }
        System.out.println("----------- Word Content End -----------");
    }
}
```

通过WordDocument类模拟了word文档中的基本元素，即文字和图片。WordDocument的在该原型模式示例中扮演的角色为ConcretePrototype， 而Cloneable的角色则为Prototype。WordDocument实现了clone方法以实现对象克隆。下面我们看看Client端的使用 :

```
public class Client {
    public static void main(String[] args) {
        WordDocument originDoc = new WordDocument();
        originDoc.setText("这是一篇文档");
        originDoc.addImage("图片1");
        originDoc.addImage("图片2");
        originDoc.addImage("图片3");
        originDoc.showDocument();

        WordDocument doc2 = originDoc.clone();
        doc2.showDocument();
        
        doc2.setText("这是修改过的Doc2文本");
        doc2.showDocument();
         
        originDoc.showDocument();
    }

}
```

输出结果如下 :
[![result](https://github.com/simple-android-framework/android_design_patterns_analysis/raw/master/prototype/mr.simple/images/result.png)](https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/prototype/mr.simple/images/result.png)

可以看到，doc2是通过originDoc.clone()创建的，并且doc2第一次输出的时候和originDoc输出是一样的。即doc2是originDoc的一份拷贝，他们的内容是一样的，而doc2修改了文本内容以后并不会影响originDoc的文本内容。需要注意的是通过clone拷贝对象的时候并不会执行构造函数！

#### 浅拷贝和深拷贝

将main函数的内容修改为如下 :

```
    public static void main(String[] args) {
        WordDocument originDoc = new WordDocument();
        originDoc.setText("这是一篇文档");
        originDoc.addImage("图片1");
        originDoc.addImage("图片2");
        originDoc.addImage("图片3");
        originDoc.showDocument();

        WordDocument doc2 = originDoc.clone();
        
        doc2.showDocument();
        
        doc2.setText("这是修改过的Doc2文本");
        doc2.addImage("哈哈.jpg");
        doc2.showDocument();
        
        originDoc.showDocument();
    }
```

输出结果如下 :
[![result](https://github.com/simple-android-framework/android_design_patterns_analysis/raw/master/prototype/mr.simple/images/result-2.png)](https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/prototype/mr.simple/images/result-2.png)
细心的朋友可能发现了，在doc2添加了一张名为"哈哈.jpg"的照片，但是却也显示在originDoc中？这是怎么回事呢？ 其实学习过C++的朋友都知道，这是因为上文中WordDocument的clone方法中只是简单的进行浅拷贝，引用类型的新对象doc2的mImages只是单纯的指向了this.mImages引用，而并没有进行拷贝。doc2的mImages添加了新的图片，实际上也就是往originDoc里添加了新的图片，所以originDoc里面也有"哈哈.jpg" 。那如何解决这个问题呢？ 那就是采用深拷贝，即在拷贝对象时，对于引用型的字段也要采用拷贝的形式，而不是单纯引用的形式。示例如下 :

```
    /**
     * 克隆对象
     */
    @Override
    protected WordDocument clone() {
        try {
            WordDocument doc = (WordDocument) super.clone();
            doc.mText = this.mText;
//            doc.mImages = this.mImages;
            doc.mImages = (ArrayList<String>) this.mImages.clone();
            return doc;
        } catch (Exception e) {
        }

        return null;
    }
```

如上代码所示，将doc.mImages指向this.mImages的一份拷贝， 而不是this.mImages本身，这样在doc2添加图片时并不会影响originDoc，如图所示 :
[![result](https://github.com/simple-android-framework/android_design_patterns_analysis/raw/master/prototype/mr.simple/images/result-3.png)](https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/prototype/mr.simple/images/result-3.png)

#### Androi应用Intent

在Android源码中，我们以熟悉的Intent来分析源码中的原型模式。简单示例如下 :

```
    Uri uri = Uri.parse("smsto:0800000123");    
    Intent shareIntent = new Intent(Intent.ACTION_SENDTO, uri);    
    shareIntent.putExtra("sms_body", "The SMS text");    
    
    Intent intent = (Intent)shareIntent.clone() ;
    startActivity(intent);
```

可以看到，我们通过shareIntent.clone方法拷贝了一个对象intent, 然后执行startActivity(intent)， 随即就进入了短信页面，号码为0800000123,文本内容为The SMS text，即这些内容都与shareIntent一致。

```
    @Override
    public Object clone() {
        return new Intent(this);
    }

    /**
     * Copy constructor.
     */
    public Intent(Intent o) {
        this.mAction = o.mAction;
        this.mData = o.mData;
        this.mType = o.mType;
        this.mPackage = o.mPackage;
        this.mComponent = o.mComponent;
        this.mFlags = o.mFlags;
        if (o.mCategories != null) {
            this.mCategories = new ArraySet<String>(o.mCategories);
        }
        if (o.mExtras != null) {
            this.mExtras = new Bundle(o.mExtras);
        }
        if (o.mSourceBounds != null) {
            this.mSourceBounds = new Rect(o.mSourceBounds);
        }
        if (o.mSelector != null) {
            this.mSelector = new Intent(o.mSelector);
        }
        if (o.mClipData != null) {
            this.mClipData = new ClipData(o.mClipData);
        }
    }
```

可以看到，clone方法实际上在内部调用了new Intent(this); 这就和C++中的拷贝构造函数完全一致了，而且是深拷贝。由于该模式比较简单，就不做太多说明。



接下来我们继续看看三种工厂模式，如下所示：

- 简单工厂模式：根据传入的参数决定实例化哪个对象。
- 工厂模式：工厂模式定义了一个创建对象的接口，由子类进行对象的初始化，即工厂模式将子类的初始化推迟到了子类里。
- 抽象工厂模式：抽象工厂模式和工厂模式很相似，只是它利用接口或者抽象类定义了一个产品族，例如定义一个拨号产品族，只定义功能，不 关心实现，具体实现交由Android、iOS等操作系统自己完成。

### 1.4 简单工厂模式

#### 模式定义

> 根据传入的参数决定实例化哪个对象。

简单工厂模式是工厂模式的简化版本，无需定义抽象工厂，通常还可以利用反射来生成对象，简化操作，如下所示：

```
// 简单工厂
public class SimpleFactory {

    public static <T extends AbstractProduct> T create(Class<T> clasz) {
        AbstractProduct product = null;
        try {
            product = (AbstractProduct) Class.forName(clasz.getName()).newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return (T) product;
    }
}
```

#### Android中简单工厂模式的应用

在Android中我们了解的使用到了简单工厂方法的地方有Bitmap对象的获取、Fragment创建等。接下来我们分开看一下。

Bitmap源码分析

**首先来说我们是不能通过new方法来创建Bitmap对象的，因为Bitmap类的构造函数是私有的，只能是通过JNI实例化。**

接下来我们随便找个入口开始看，比如：

```
Bitmap bmp = BitmapFactory.decodeFile(String pathName);
12
```

我们把源码中的调用关系找出来，如下

```
public static Bitmap decodeFile(String pathName) {
    return decodeFile(pathName, null);
}

public static Bitmap decodeFile(String pathName, Options opts) {
    Bitmap bm = null;
    InputStream stream = null;
    try {
        stream = new FileInputStream(pathName);
        bm = decodeStream(stream, null, opts);
    } catch (Exception e) {
        /*  do nothing.
            If the exception happened on open, bm will be null.
        */
        Log.e("BitmapFactory", "Unable to decode stream: " + e);
    } finally {
        if (stream != null) {
            try {
                stream.close();
            } catch (IOException e) {
                // do nothing here
            }
        }
    }
    return bm;
}

public static Bitmap decodeStream(InputStream is, Rect outPadding, Options opts) {
    // we don't throw in this case, thus allowing the caller to only check
    // the cache, and not force the image to be decoded.
    if (is == null) {
        return null;
    }

    Bitmap bm = null;

    Trace.traceBegin(Trace.TRACE_TAG_GRAPHICS, "decodeBitmap");
    try {
        if (is instanceof AssetManager.AssetInputStream) {
            final long asset = ((AssetManager.AssetInputStream) is).getNativeAsset();
            bm = nativeDecodeAsset(asset, outPadding, opts);
        } else {
            bm = decodeStreamInternal(is, outPadding, opts);
        }

        if (bm == null && opts != null && opts.inBitmap != null) {
            throw new IllegalArgumentException("Problem decoding into existing bitmap");
        }

        setDensityFromOptions(bm, opts);
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_GRAPHICS);
    }

    return bm;
}

private static native Bitmap nativeDecodeStream(InputStream is, byte[] storage,
        Rect padding, Options opts);

/**
 * Set the newly decoded bitmap's density based on the Options.
 */
private static void setDensityFromOptions(Bitmap outputBitmap, Options opts) {
    if (outputBitmap == null || opts == null) return;

    final int density = opts.inDensity;
    if (density != 0) {
        outputBitmap.setDensity(density);
        final int targetDensity = opts.inTargetDensity;
        if (targetDensity == 0 || density == targetDensity || density == opts.inScreenDensity) {
            return;
        }

        byte[] np = outputBitmap.getNinePatchChunk();
        final boolean isNinePatch = np != null && NinePatch.isNinePatchChunk(np);
        if (opts.inScaled || isNinePatch) {
            outputBitmap.setDensity(targetDensity);
        }
    } else if (opts.inBitmap != null) {
        // bitmap was reused, ensure density is reset
        outputBitmap.setDensity(Bitmap.getDefaultDensity());
    }
}
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273747576777879808182838485
```

我们来分析一下调用过程，可以看到**decodeFile(String pathName)调用的是decodeFile(String pathName, Options opts)，在两个参数的decodeFile方法中又去调用了decodeStream(InputStream is, Rect outPadding, Options opts)方法，然后最终调用nativeDecodeAsset或者nativeDecodeStream来构建Bitmap对象，这两个都是native方法(Android中使用Skia库来解析图像 )。再经过setDensityFromOptions方法的一些设置解码密度之类的操作，返回我们要的Bitmap对象。**

> /** 
> \* Creates Bitmap objects from various sources, including files, streams, and byte-arrays. 
> */

看下BitmapFactory的注释我们可以看到，这个工厂支持从不同的资源创建Bitmap对象，包括files, streams, 和byte-arrays，但是调用关系都大同小异。

### 1.5 工厂模式

#### 模式定义

> 工厂模式定义了一个创建对象的接口，由子类进行对象的初始化，即工厂模式将子类的初始化推迟到了子类里。抽象工厂模式

- 优点：工厂模式符合开闭原则，当需要增加一个新产品时，只需要增加一个具体产品类和一个具体工厂类，无需修改原有的系统，外界也无需 知道具体的产品类的实现。
- 缺点：每次增加新产品的时候都会增加产品类和工厂类，势必会让系统越来越庞大。

工厂模式的实现也很简单，就是定义一个抽象类或者接口工厂，在子类工厂中决定实例化具体的类。

```
// 抽象工厂
public abstract class AbstractFactory {
    public abstract AbstractProduct create();
}

// 具体工厂
public class ConcretetFactory extends AbstractFactory{

    public static AbstractProduct create() {
        return new ConcreteProductA();
//        return new ConcreteProductB();
    }
}

// 抽象产品
public class AbstractProduct {
}

// 具体产品A
public class ConcreteProductA extends AbstractProduct {
}

// 具体产品B
public class ConcreteProductB extends AbstractProduct {
}
```



#### Android源码中的应用

工厂方法模式应用很广泛，开发中使用到的数据结构中就隐藏着对工厂方法模式的应用，例如 List、Set，List、Set 继承自 Collection 接口，而 Collection 接口继承于 Iterable 接口：

```
public interface Iterable<T> {
    Iterator<T> iterator();
}123
```

这意味着 List、Set 接口也会继承 iterator() 方法，下面以 ArrayList 为例进行分析： 
ArrayList 中 iterator() 方法的实现就是构造并返回一个迭代器对象：

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    public Iterator<E> iterator() {
        return new Itr();
    }
    // 迭代器
    private class Itr implements Iterator<E> {
        protected int limit = java.util.ArrayList.this.size;
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor < limit;
        }
        @SuppressWarnings("unchecked")
        public E next() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            int i = cursor;
            if (i >= limit)
                throw new NoSuchElementException();
            Object[] elementData = java.util.ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            try {
                java.util.ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
                limit--;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
        // 代码省略
    }
    // 代码省略
}1234567891011121314151617181920212223242526272829303132333435363738394041424344454647
```

其中的 iterator() 方法其实就相当于一个工厂方法，专为 new 对象而生，构造并返回一个具体的迭代器。



### 1.6 抽象工厂模式

模式定义

> 抽象工厂模式和工厂模式很相似，只是它利用接口或者抽象类定义了一个产品族，例如定义一个拨号产品族，只定义功能，不 关心实现，具体实现交由Android、iOS等操作系统自己完成。

- 优点：
- 缺点：

实现如下所示：

```
// 抽象产品A
public abstract class AbstractProductA {
}

// 抽象产品B
public abstract class AbstractProductB {
}

// 具体产品A1
public class ConcreteProductA1 extends AbstractProductA{
}

// 具体产品A2
public class ConcreteProductA2 extends AbstractProductA {
}

// 具体产品B1
public class ConcreteProductB1 extends AbstractProductB {
}

// 具体产品B2
public class ConcreteProductB2 extends AbstractProductB {
}

// 抽象工厂
public abstract class AbstractFactory {
    
    public abstract AbstractProductA createProductA();
    public abstract AbstractProductB createProductB();
}

// 具体工厂
public class ConcreteFactory extends AbstractFactory {
    
    @Override
    public AbstractProductA createProductA() {
        return new ConcreteProductA1();
    }

    @Override
    public AbstractProductB createProductB() {
        return new ConcreteProductB1();
    }
}
```

## 二 结构型设计模式

**通过继承和对象组合实现结构型模式，其中继承实现的是类结构型模式，对象组合实现的是对象结构型模式。注意优先使用对象组合，而不是类继承。**  

### 2.1 适配器模式

模式定义

> 适配器模式可以将一个类的接口，转换成客户端期望的另一个接口，让两个原本不兼容的接口可以无缝对接。

- 优点：
- 缺点：

```
// 目标接口
public interface TargetInterface {
    int getFive();
}

// 被适配对象
public class Adaptee {

    public int getTen() {
        return 10;
    }
}

// 适配器
public class Adapter extends Adaptee implements TargetInterface {

    @Override
    public int getFive() {
        return 5;
    }
}
```

#### 适配器的类图表示

适配器模式又分两种，1. 类适配器模式；2. 对象适配器。如图一，图二所示。

![图一](http://blog.qiji.tech/wp-content/uploads/2016/02/UML_%E7%B1%BB%E9%80%82%E9%85%8D%E5%99%A8.png)
图一 类适配器模式
![图二](http://blog.qiji.tech/wp-content/uploads/2016/02/UML_%E5%AF%B9%E8%B1%A1%E9%80%82%E9%85%8D%E5%99%A8.png)

#### Android ListView 中的 Adapter 模式

由于不同的 ListView 所要呈现的视图也是不同的，为了应对这些不同，就需要通过一个适配器来实现隔离和适配。我们可以通过一个继承自 BaseAdapter 类来实现自己的类。

```
ListView mListView = (ListView) findViewById(listView_id);
mListView.setAdapter(new MyAdapter(context, mdData));

public class MyAdapter extends BaseAdapter {
  private LayoutInflater mInflater;
  private List<String> mData;
  public MyAdapter(Context context, List<String> data) {
    this.mInflater = LayoutInflater.from(context);
    mData = data;
  }
  @Override
  public int getCount() {
    return mData.size();
  }

  @Override
  public String getItem(int positon) {
    return mData.get(positon);
  }

  @Override
  public long getItemId(int position) {
    return pos;
  }
  //解析、设置、缓存 convertView 以及相关内容
  @Override
  public View getView(int position, View convertView, ViewGroup parent) {
    ViewHolder holder = null;
    // Item View 的复用
    if (convertView == null) {
      holder = new ViewHolder();
      convertView = mInflater.inflate(R.layout.my_listview_item, null);
      holder.title = (TextView) convertView.findViewById(R.id.title);
      convertView.setTag(holder);
    } else {
      holder = (ViewHolder) convertView.getTag();
    }
    holder.title.setText(mData.get(position));
    return convertView;
  }
}

```

增加一个 Adapter 层来应对变化，将 ListView 需要的接口抽象到 Adapter 对象中，这样只要用户实现了 Adapter 的接口， ListView 就可以按照用户设定的显示效果、数量、数据来显示特定的 Item View。通过代理数据集来告知ListView 数据的个数（ getCount方法 ）以及每个数据 （ getItem方法 ），最重要的是要解决 Item View 的输出。Item View 千变万化，但终究它都是 View 类型， Adapter 统一将 Item View 输出为 View （ getView方法 ），这样就很好的应对了 Item View 的可变性。 

### 2.2 组合模式

#### 模式定义

> 将对象组成树形结构以表示整体-部分的层次结构，使得用户对单个对象和组合对象的使用具有一致性。

应用场景

- 表示对象部分-整体的层次结构。
- 从一个整体中能够独立出部分模块或者功能的场景。

#### Android具体实现代码

View.java

```
public class View ....{
 //此处省略无关代码...
}
```

ViewGroup.java

```
public abstract class ViewGroup extends View ...{

     //增加子节点
    public void addView(View child, int index) { 

    }
    //删除子节点
    public void removeView(View view) {

    }
     //查找子节点
    public View getChildAt(int index) {
      try {
            return mChildren[index];
          } catch (IndexOutOfBoundsException ex) {
            return null;
          } 
   }
}
```

### 2.3 装饰模式

#### 模式定义

> 动态的为对象增加一些额外的功能。

应用场景

- 需要透明且动态的扩展类的功能时。

#### code

```
// 抽象组件类
public abstract class AbstractComponent {
    
    protected abstract void operation();
}

// 具体组件类
public class ConcreteComponent extends AbstractComponent {
    @Override
    protected void operation() {
        
    }
}

// 抽象装饰类
public abstract class AbstractDecorator extends AbstractComponent {

    private AbstractComponent mComponent;

    public AbstractDecorator(AbstractComponent component) {
        mComponent = component;
    }

    @Override
    protected void operation() {
        mComponent.operation();
    }
}

// 具体装饰类
public class ConcreteDecorator extends AbstractDecorator {

    public ConcreteDecorator(AbstractComponent component) {
        super(component);
    }

    @Override
    protected void operation() {
        operationA();
        super.operation();
        operationB();
    }

    private void operationA() {

    }

    private void operationB() {

    }
}
```



#### Android源码中的模式实现

Android源码中的装饰模式其实我们经常接触，但并不一定有过真正的了解，Context类在Android中被称为“上帝对象”，它本质是一个抽象类，其在我们装饰模式中就相当于抽象组件，而在其内部定义了大量的抽象方法，比如我们经常会用到的startActivity方法，如下：

```
/**
 * Interface to global information about an application environment.  This is
 * an abstract class whose implementation is provided by
 * the Android system.  It
 * allows access to application-specific resources and classes, as well as
 * up-calls for application-level operations such as launching activities,
 * broadcasting and receiving intents, etc.
 */
public abstract class Context {

    // 省略代码
    
    /**
     * Same as {@link #startActivity(Intent, Bundle)} with no options
     * specified.
     *
     * @param intent The description of the activity to start.
     *
     * @throws ActivityNotFoundException &nbsp;
     *
     * @see #startActivity(Intent, Bundle)
     * @see PackageManager#resolveActivity
     */
    public abstract void startActivity(Intent intent);
    
    /**
     * Launch a new activity.  You will not receive any information about when
     * the activity exits.
     *
     * <p>Note that if this method is being called from outside of an
     * {@link android.app.Activity} Context, then the Intent must include
     * the {@link Intent#FLAG_ACTIVITY_NEW_TASK} launch flag.  This is because,
     * without being started from an existing Activity, there is no existing
     * task in which to place the new activity and thus it needs to be placed
     * in its own separate task.
     *
     * <p>This method throws {@link ActivityNotFoundException}
     * if there was no Activity found to run the given Intent.
     *
     * @param intent The description of the activity to start.
     * @param options Additional options for how the Activity should be started.
     * May be null if there are no options.  See {@link android.app.ActivityOptions}
     * for how to build the Bundle supplied here; there are no supported definitions
     * for building it manually.
     *
     * @throws ActivityNotFoundException &nbsp;
     *
     * @see #startActivity(Intent)
     * @see PackageManager#resolveActivity
     */
    public abstract void startActivity(Intent intent, @Nullable Bundle options);
    
    // 省略代码
}
```

Context真正的实现类是ContextImpl，ContextImpl继承自Context抽象类，并实现了Context中的抽象方法，具体代码如下：

```
/**
 * Common implementation of Context API, which provides the base
 * context object for Activity and other application components.
 */
class ContextImpl extends Context {
    
    // 代码省略
    
    @Override
    public void startActivity(Intent intent) {
        warnIfCallingFromSystemProcess();
        startActivity(intent, null);
    }
    
    @Override
    public void startActivity(Intent intent, Bundle options) {
        warnIfCallingFromSystemProcess();
        if ((intent.getFlags()&Intent.FLAG_ACTIVITY_NEW_TASK) == 0) {
            throw new AndroidRuntimeException(
                    "Calling startActivity() from outside of an Activity "
                    + " context requires the FLAG_ACTIVITY_NEW_TASK flag."
                    + " Is this really what you want?");
        }
        mMainThread.getInstrumentation().execStartActivity(
            getOuterContext(), mMainThread.getApplicationThread(), null,
            (Activity)null, intent, -1, options);
    }
    
    // 代码省略
}
```

这里ContextImpl就相当于组件具体实现类，那么谁来承担装饰者的身份呢？我们知道Activity从类层次上来说本质是一个Context，如果大家留意过其代码会发现Activity并非直接继承于Context，而是继承于ContextThemeWrapper，如下代码所示：

```
public class Activity extends ContextThemeWrapper
        implements LayoutInflater.Factory2,
        Window.Callback, KeyEvent.Callback,
        OnCreateContextMenuListener, ComponentCallbacks2,
        Window.OnWindowDismissedCallback {
 
    // 代码省略       
}
```

而这个ContextThemeWrapper又继承了ContextWrapper：

```
/**
 * A ContextWrapper that allows you to modify the theme from what is in the 
 * wrapped context. 
 */
public class ContextThemeWrapper extends ContextWrapper {
    // 代码省略
}
```

最终这个ContextWrapper才继承于Context，为什么类层次会这么复杂？其实这里就是一个典型的装饰模式，ContextWrapper就是我们要找的装饰者，在ContextWrapper中有一个Context的引用：

```
/**
 * Proxying implementation of Context that simply delegates all of its calls to
 * another Context.  Can be subclassed to modify behavior without changing
 * the original Context.
 */
public class ContextWrapper extends Context {
    Context mBase;

    public ContextWrapper(Context base) {
        mBase = base;
    }

    // 代码省略
}
```

到目前为止已经有点装饰模式的样子了，不过好像还缺点什么，没错，一个来自于抽象组件的方法，这里我们还是先看startActivity，在ContextWrapper中同样有一个startActivity方法：

```
/**
 * Proxying implementation of Context that simply delegates all of its calls to
 * another Context.  Can be subclassed to modify behavior without changing
 * the original Context.
 */
public class ContextWrapper extends Context {

    // 代码省略
    
    @Override
    public void startActivity(Intent intent) {
        mBase.startActivity(intent);
    }

    @Override
    public void startActivity(Intent intent, Bundle options) {
        mBase.startActivity(intent, options);
    }
    
    // 代码省略
}
```

这里可以看出ContextWrapper中的startActivity方法也仅仅是简单地调用了具体组件实现类ContextImpl中的对应方法而已，实质上ContextWrapper中所有的方法都仅仅是调用了ContextImpl中的方法，对Context具体的包装拓展则由ContextWrapper的具体子类完成，比如我们的Activity、Service和Application。



### 2.4 外观模式

#### 模式定义

> 要求一个字系统的外部与其内部的通信都通过一个统一的而对象进行。

应用场景

- 子系统在迭代的过程中可以会不断变化，甚至被替代掉，给一个统一的访问接口，避免子系统的改变影响到外部的调用者。
- 当需要构建层次结构型的系统时，为各层子系统提供访问的接口进行通信，避免直接产生依赖。



#### code

简单实现的介绍

电视遥控器是现实生活中一个比较好的外观模式的运用，遥控器可以控制电源的开源、声音的调整、频道的切换等。这个遥控器就是我们这里说的外观或者门面，而电源、声音、频道切换系统就是我们的子系统。遥控器统一对这些子模块的控制，我想你没有用过多个遥控器来分别控制电源开关、声音控制等功能。下面我们就来简单模拟一下这个系统。

实现源码

TvController.java

```
public class TvController {
    private PowerSystem mPowerSystem = new PowerSystem();
    private VoiceSystem mVoiceSystem = new VoiceSystem();
    private ChannelSystem mChannelSystem = new ChannelSystem();

    public void powerOn() {
        mPowerSystem.powerOn();
    }

    public void powerOff() {
        mPowerSystem.powerOff();
    }

    public void turnUp() {
        mVoiceSystem.turnUp();
    }

    public void turnDown() {
        mVoiceSystem.turnDown();
    }

    public void nextChannel() {
        mChannelSystem.next();
    }

    public void prevChannel() {
        mChannelSystem.prev();
    }
}
```

PowerSystem.java

```
/**
 * 电源控制系统
 */
 class PowerSystem {
    public void powerOn() {
        System.out.println("开机");
    }

    public void powerOff() {
        System.out.println("关机");
    }
}
```

VoiceSystem.java

```
/**
 * 声音控制系统
 */
class VoiceSystem {
    public void turnUp() {
        System.out.println("音量增大");
    }

    public void turnDown() {
        System.out.println("音量减小");
    }
}
```

ChannelSystem.java

```
/**
 * 频道控制系统
 */
class ChannelSystem {
    public void next() {
        System.out.println("下一频道");
    }

    public void prev() {
        System.out.println("上一频道");
    }
}
```

测试代码 :

```
public class TvController {
    private PowerSystem mPowerSystem = new PowerSystem();
    private VoiceSystem mVoiceSystem = new VoiceSystem();
    private ChannelSystem mChannelSystem = new ChannelSystem();

    public void powerOn() {
        mPowerSystem.powerOn();
    }

    public void powerOff() {
        mPowerSystem.powerOff();
    }

    public void turnUp() {
        mVoiceSystem.turnUp();
    }

    public void turnDown() {
        mVoiceSystem.turnDown();
    }

    public void nextChannel() {
        mChannelSystem.next();
    }

    public void prevChannel() {
        mChannelSystem.prev();
    }
}
```

输出结果：

```
开机
下一频道
音量增大
关机
```

上面的TvController封装了对电源、声音、频道切换的操作，为用户提供了一个统一的接口。使得用户控制电视机更加的方便、更易于使用。



#### Android应用-context封装

在开发过程中，Context是最重要的一个类型。它封装了很多重要的操作，比如startActivity()、sendBroadcast()等，几乎是开发者对应用操作的统一入口。Context是一个抽象类，它只是定义了抽象接口，真正的实现在ContextImpl类中。它就是今天我们要分析的外观类。

在应用启动时，首先会fork一个子进程，并且调用ActivityThread.main方法启动该进程。ActivityThread又会构建Application对象，然后和Activity、ContextImpl关联起来，然后再调用Activity的onCreate、onStart、onResume函数使Activity运行起来。我们看看下面的相关代码:

```
private final void handleLaunchActivity(ActivityClientRecord r, Intent customIntent) {
		// 代码省略

        // 1、创建并且加载Activity，调用其onCreate函数
        Activity a = performLaunchActivity(r, customIntent);

        if (a != null) {
            r.createdConfig = new Configuration(mConfiguration);
            Bundle oldState = r.state;
            // 2、调用Activity的onResume方法，使Activity变得可见
            handleResumeActivity(r.token, false, r.isForward);

        }
    }


     private final Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        // System.out.println("##### [" + System.currentTimeMillis() + "] ActivityThread.performLaunchActivity(" + r + ")");
		// 代码省略

        Activity activity = null;
        try {
            java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
            // 1、创建Activity
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            r.intent.setExtrasClassLoader(cl);
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(activity, e)) {
                throw new RuntimeException(
                    "Unable to instantiate activity " + component
                    + ": " + e.toString(), e);
            }
        }

        try {
            // 2、创建Application
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);

            if (activity != null) {
                // ***** 构建ContextImpl  ****** 
                ContextImpl appContext = new ContextImpl();
                appContext.init(r.packageInfo, r.token, this);
                appContext.setOuterContext(activity);
                // 获取Activity的title
                CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
                Configuration config = new Configuration(mConfiguration);
            
                 // 3、Activity与context, Application关联起来
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstance,
                        r.lastNonConfigurationChildInstances, config);
				// 代码省略

                // 4、回调Activity的onCreate方法
                mInstrumentation.callActivityOnCreate(activity, r.state);
           
                // 代码省略
            }
            r.paused = true;

            mActivities.put(r.token, r);

        } catch (SuperNotCalledException e) {
            throw e;

        } catch (Exception e) {
      
        }

        return activity;
    }


    final void handleResumeActivity(IBinder token, boolean clearHide, boolean isForward) {
   
        unscheduleGcIdler();

        // 1、最终调用Activity的onResume方法
        ActivityClientRecord r = performResumeActivity(token, clearHide);
        // 代码省略
        // 2、这里是重点，在这里使DecorView变得可见
        if (r.window == null && !a.mFinished && willBeVisible) {
                // 获取Window，即PhoneWindow类型
                r.window = r.activity.getWindow();
                // 3、获取Window的顶级视图，并且使它可见
                View decor = r.window.getDecorView();
                decor.setVisibility(View.INVISIBLE);
                // 4、获取WindowManager
                ViewManager wm = a.getWindowManager();
                // 5、构建LayoutParams参数
                WindowManager.LayoutParams l = r.window.getAttributes();
                a.mDecor = decor;
                l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
                l.softInputMode |= forwardBit;
                if (a.mVisibleFromClient) {
                    a.mWindowAdded = true;
                    // 6、将DecorView添加到WindowManager中，最终的操作是通过WindowManagerService的addView来操作
                    wm.addView(decor, l);
                }
            } else if (!willBeVisible) {
                if (localLOGV) Slog.v(
                    TAG, "Launch " + r + " mStartedActivity set");
                r.hideForNow = true;
            }
            // 代码省略
    }

 public final ActivityClientRecord performResumeActivity(IBinder token,
            boolean clearHide) {
        ActivityClientRecord r = mActivities.get(token);
       
        if (r != null && !r.activity.mFinished) {
                try {
                // 代码省略
                // 执行onResume
                r.activity.performResume();
				// 代码省略
            } catch (Exception e) {
   
            }
        }
        return r;
    }
```

Activity启动之后，Android给我们提供了操作系统服务的统一入口，也就是Activity本身。这些工作并不是Activity自己实现的，而是将操作委托给Activity父类ContextThemeWrapper的mBase对象，这个对象的实现类就是ContextImpl ( 也就是performLaunchActivity方法中构建的ContextImpl )。

```
class ContextImpl extends Context {
    private final static String TAG = "ApplicationContext";
    private final static boolean DEBUG = false;
    private final static boolean DEBUG_ICONS = false;

    private static final Object sSync = new Object();
    private static AlarmManager sAlarmManager;
    private static PowerManager sPowerManager;
    private static ConnectivityManager sConnectivityManager;
    private AudioManager mAudioManager;
    LoadedApk mPackageInfo;
    private Resources mResources;
    private PackageManager mPackageManager;
    private NotificationManager mNotificationManager = null;
    private ActivityManager mActivityManager = null;
    
	// 代码省略
    
        @Override
    public void sendBroadcast(Intent intent) {
        String resolvedType = intent.resolveTypeIfNeeded(getContentResolver());
        try {
            ActivityManagerNative.getDefault().broadcastIntent(
                mMainThread.getApplicationThread(), intent, resolvedType, null,
                Activity.RESULT_OK, null, null, null, false, false);
        } catch (RemoteException e) {
        }
    }
    
    
        @Override
    public void startActivity(Intent intent) {
        if ((intent.getFlags()&Intent.FLAG_ACTIVITY_NEW_TASK) == 0) {
            throw new AndroidRuntimeException(
                    "Calling startActivity() from outside of an Activity "
                    + " context requires the FLAG_ACTIVITY_NEW_TASK flag."
                    + " Is this really what you want?");
        }
        mMainThread.getInstrumentation().execStartActivity(
            getOuterContext(), mMainThread.getApplicationThread(), null, null, intent, -1);
    }
    
    
        @Override
    public ComponentName startService(Intent service) {
        try {
            ComponentName cn = ActivityManagerNative.getDefault().startService(
                mMainThread.getApplicationThread(), service,
                service.resolveTypeIfNeeded(getContentResolver()));
            if (cn != null && cn.getPackageName().equals("!")) {
                throw new SecurityException(
                        "Not allowed to start service " + service
                        + " without permission " + cn.getClassName());
            }
            return cn;
        } catch (RemoteException e) {
            return null;
        }
    }
    
        @Override
    public String getPackageName() {
        if (mPackageInfo != null) {
            return mPackageInfo.getPackageName();
        }
        throw new RuntimeException("Not supported in system context");
    }
}
```

可以看到，ContextImpl内部有很多xxxManager类的对象，也就是我们上文所说的各种子系统的角色。ContextImpl内部封装了一些系统级别的操作，有的子系统功能虽然没有实现，但是也提供了访问该子系统的接口，比如获取ActivityManager的getActivityManager方法。

比如我们要启动一个Activity的时候，我们调用的是startActivity方法，这个功能的内部实现实际上是Instrumentation完成的。ContextImpl封装了这个功能，使得用户根本不需要知晓Instrumentation相关的信息，直接使用startActivity即可完成相应的工作。其他的子系统功能也是类似的实现，比如启动Service和发送广播内部使用的是ActivityManagerNative等。ContextImpl的结构图如下 :
[![context](https://github.com/simple-android-framework/android_design_patterns_analysis/raw/master/facade/elsdnwn/images/contextimpl.png)](https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/facade/elsdnwn/images/contextimpl.png)

外观模式非常的简单，只是封装了子系统的操作，并且暴露接口让用户使用，避免了用户需要与多个子系统进行交互，降低了系统的耦合度、复杂度。如果没有外观模式的封装，那么用户就必须知道各个子系统的相关细节，子系统之间的交互必然造成纠缠不清的关系，影响系统的稳定性、复杂度。

### 2.5 桥接模式

模式定义

> 将抽象部分和实现部分相互分离，使它们可以独立变化。

应用场景

- 如果一个系统需要在抽象部分和实现部分增加更多的灵活性，避免两种变化的时候相互影响。
- 如果不希望使用继承而增加系统的复杂度，可以考虑使用桥接模式。
- 一个类存在两个独立变化的纬度，且这两个纬度都希望进行扩展。

#### Android 源码中的桥接模式

桥接模式在 Android 源码中应用广泛，比较典型的有 Adapter 与 AdapterView 的桥接，Window 与 WindowManager 的桥接模式。

##### Adapter 与 AdapterView 的桥接

![abs_bridge.jpg](http://weiqianghu.github.io/2016/10/13/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E6%A1%A5%E6%8E%A5%E6%A8%A1%E5%BC%8F/abs_bridge.jpg)

##### Window 与 WindowManager 的桥接

![win_bridge.jpg](http://weiqianghu.github.io/2016/10/13/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E6%A1%A5%E6%8E%A5%E6%A8%A1%E5%BC%8F/win_bridge.jpg)

通过以上两 UML 类图应该能明显的看出来桥接模式的影子。



### 2.6 享元模式

#### 模式定义

> 使用共享对象可以有效的支持大量的细粒度对象。

应用场景

- 系统中存在着大量的相似对象。
- 细粒度的对象都具有较接近的外部状态，而且内部状态与外部环境无关。
- 需要缓冲池的场景。



#### Android 源码中的享元模式

在使用 Handler 发送消息之前，我们一般都会使用如下代码调用 `mHandler.obtainMessage()` 方法获取一个 Message 对象。这其中究竟是怎么实现的呢？

```
Handler mHandler = new Handler();

public void do() {
    new Thread(new Runnable() {
        @Override
        public void run() {
            //do sth
            Message message = mHandler.obtainMessage();
            message.what = 1;
            message.obj = result;
            mHandler.sendMessage(message);
        }
    });
}
```

```
//Handler.otainMessage()方法
public final Message obtainMessage(){
    return Message.obtain(this);
}
```

可以看到 Handler.obtainMessage() 实际上调用的是 Message 的 obtain 方法，我们顺着源码看下去。

先看看 Message 类部分源码

```
// sometimes we store linked lists of these things
Message next;

private static final Object sPoolSync = new Object();//作为锁对象
private static Message sPool;//虽然名称为 sPool 但是实际上是一个指向消息队列队首的指针
private static int sPoolSize = 0;//

private static final int MAX_POOL_SIZE = 50;//「对象池」中的最大数量

public static Message obtain(Handler h) {
    Message m = obtain();//调用 obtain 方法获取 message 对象
    m.target = h;//指定 message 的目标对象
    return m;
}

//从消息对象池中取出一个 Message 对象，如果没有就创建一个
public static Message obtain() {
    synchronized (sPoolSync) {
        if (sPool != null) {
            Message m = sPool;
            sPool = m.next;
            m.next = null;
            m.flags = 0; // 清空 in-use flag
            sPoolSize--;
            return m;
        }
    }
    return new Message();//消息池中没有可复用的 Message 就创建一个新的 Message
}
```

至此，从对象池中获取对象的大致流程。无论是 Handler.obtainMessage(参数列表) 方法，还是 Message 的 obtain(参数列表) 方法，最终都会调用 Message.obtain() 方法。在 Message.obtain() 方法的实现中，会先从对象池中获取 Message 对象，如果获取不到，则创建一个新的 Message 对象，然后返回。该对象在后续的执行过程中会被回收到对象池，以便复用。

但是 Message 对象是如何被回收到「对象池」中的呢？ 从 Message 类的部分代码中我们看到 sPool 的实际类型是一个 Message 对象，而不是一个容器。另外从 obtain 方法中我们不难看到链表的踪影。难道消息池是使用链表实现的吗？

在 AS 中打开 Message 类的结构图，可以看到其中有一个 recycle 方法，我们看看里面是怎么实现的。

```
public void recycle() {
    if (isInUse()) {//判断消息是否还在使用
        if (gCheckRecycle) {//如果消息处在使用状态时被 gc 回收，就抛出异常
            throw new IllegalStateException("This message cannot be recycled because it " + "is still in use.");
        }
        return;//直接返回，取消回收操作
    }
    recycleUnchecked();//调用回收方法
}

/**
 * 回收一个可能还在使用的对象
 */
void recycleUnchecked() {
    // 只要该对象还在回收对象池中，就标记该对象为正在使用状态。
    // 清空其他状态
    flags = FLAG_IN_USE;
    what = 0;
    arg1 = 0;
    arg2 = 0;
    obj = null;
    replyTo = null;
    sendingUid = -1;
    when = 0;
    target = null;
    callback = null;
    data = null;
	//回收消息到消息池中
    synchronized (sPoolSync) {
        if (sPoolSize < MAX_POOL_SIZE) {
            next = sPool;
            sPool = this;
            sPoolSize++;
        }
    }
}
```

recycle 方法首先会判断 Message 对象是否处在使用状态。如果处在使用状态会直接返回（如果此时 GC 回收该对象会抛出异常），否则调用 recycleUnchecked 方法，具体的回收逻辑是在 recycleUnchecked 方法中实现的。首先会标记 Message 处于使用状态，然后清空对象中的其他状态。将消息存入回收池，主要是链表的操作。大致如下图所示。

[![msg](https://user-images.githubusercontent.com/16668676/29739978-6c03d780-8a7e-11e7-8aad-3da3590c2ea1.png)](https://user-images.githubusercontent.com/16668676/29739978-6c03d780-8a7e-11e7-8aad-3da3590c2ea1.png)

小结

Message 通过在内部构建一个链表来维护一个被会受到 Message 对象的对象池，当用户调用 obtain 方法时，会优先从池中获取。如果池中没有可以复用的对象就创建一个新的对象，该对象使用完之后，会被缓存到对象池中，当下次调用 obtain 方法时，他们就会被复用。

此处 Message 扮演了三个角色。既是 FlyWeight 抽象，又是 ConcreteFlyWeight 对象，同时还担任 FlyWeightFactory 角色，承担着管理对象池的职责。

想进一步了解 Android 消息机制的同学可参考[Android 消息机制解析](https://ivanljt.github.io/blog/2017/04/28/Android-%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E8%A7%A3%E6%9E%90/)。

### 2.7 代理模式

#### 模式定义

> 为其他对象提供一个代理以提供对这个对象的访问。

应用场景

- 当无法或者不想直接访问某个对象时，可以通过一个代理对象进行访问。

#### UML类图

[![url](https://github.com/simple-android-framework/android_design_patterns_analysis/raw/master/proxy/singwhatiwanna/images/proxy-uml.png)](https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/proxy/singwhatiwanna/images/proxy-uml.png)

#### 角色介绍

- 抽象对象角色：声明了目标对象和代理对象的共同接口，这样一来在任何可以使用目标对象的地方都可以使用代理对象。
- 目标对象角色：定义了代理对象所代表的目标对象。
- 代理对象角色：代理对象内部含有目标对象的引用，从而可以在任何时候操作目标对象；代理对象提供一个与目标对象相同的接口，以便可以在任何时候替代目标对象。代理对象通常在客户端调用传递给目标对象之前或之后，执行某个操作，而不是单纯地将调用传递给目标对象。

#### code：

代理模式按照代理类运行前是否存在还可以分为静态代理和动态代理，如下所示：

静态代理

```
// 被代理接口，定义要实现的功能。
public interface Subject {

    void visit();
}

// 被代理类，完成实际的功能。
public class ConcreteSubject implements Subject {

    @Override
    public void visit() {
        System.out.println("visit");
    }
}

// 静态代理类，与被代理类实现同一套接口
public class StaticProxy implements Subject {

    private Subject mSubject;

    public StaticProxy(Subject subject) {
        mSubject = subject;
    }

    @Override
    public void visit() {
        mSubject.visit();
    }
}
```

动态代理

```
// 被代理接口，定义要实现的功能。
public interface Subject {

    void visit();
}

// 被代理类，完成实际的功能。
public class ConcreteSubject implements Subject {

    @Override
    public void visit() {
        System.out.println("visit");
    }
}

// 动态代理类，实现InvocationHandler接口。
public class DynamicProxy implements InvocationHandler {

    private Subject mSubject;

    public DynamicProxy(Subject subject) {
        mSubject = subject;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        System.out.println("函数执行前自定义操作");
        // 调用被代理类的方法时会调用该方法
        method.invoke(mSubject, args);
        System.out.println("函数执行后自定义操作");
        return null;
    }
}

public class Client {

    public static void main(String[] args) {

        Subject subject = new ConcreteSubject();
        DynamicProxy proxy = new DynamicProxy(subject);

        // 动态生成代理类
        Subject proxySubject = (Subject) Proxy.newProxyInstance(DynamicProxy.class.getClassLoader()
                , subject.getClass().getInterfaces()
                , proxy);
        proxySubject.visit();

    }
}
```



#### Android代理模式-Binder

直观来说，Binder是Android中的一个类，它继承了IBinder接口。从IPC角度来说，Binder是Android中的一种跨进程通信方式，Binder还可以理解为一种虚拟的物理设备，它的设备驱动是/dev/binder，该通信方式在linux中没有；从Android Framework角度来说，Binder是ServiceManager连接各种Manager（ActivityManager、WindowManager，etc）和相应ManagerService的桥梁；从Android应用层来说，Binder是客户端和服务端进行通信的媒介，当你bindService的时候，服务端会返回一个包含了服务端业务调用的Binder对象，通过这个Binder对象，客户端就可以获取服务端提供的服务或者数据，这里的服务包括普通服务和基于AIDL的服务。

Binder一个很重要的作用是：将客户端的请求参数通过Parcel包装后传到远程服务端，远程服务端解析数据并执行对应的操作，同时客户端线程挂起，当服务端方法执行完毕后，再将返回结果写入到另外一个Parcel中并将其通过Binder传回到客户端，客户端接收到返回数据的Parcel后，Binder会解析数据包中的内容并将原始结果返回给客户端，至此，整个Binder的工作过程就完成了。由此可见，Binder更像一个数据通道，Parcel对象就在这个通道中跨进程传输，至于双方如何通信，这并不负责，只需要双方按照约定好的规范去打包和解包数据即可。

为了更好地说明Binder，这里我们先手动实现了一个Binder。为了使得逻辑更清晰，这里简化一下，我们来模拟一个银行系统，这个银行提供的功能只有一个：即查询余额，只有传递一个int的id过来，银行就会将你的余额设置为id*10，满足下大家的发财梦。

1. 先定义一个Binder接口

```
package com.ryg.design.manualbinder;

import android.os.IBinder;
import android.os.IInterface;
import android.os.RemoteException;

public interface IBank extends IInterface {

   static final String DESCRIPTOR = "com.ryg.design.manualbinder.IBank";

   static final int TRANSACTION_queryMoney = (IBinder.FIRST_CALL_TRANSACTION + 0);

   public long queryMoney(int uid) throws RemoteException;

}
```

2.创建一个Binder并实现这个上述接口

```
package com.ryg.design.manualbinder;

import android.os.Binder;
import android.os.IBinder;
import android.os.Parcel;
import android.os.RemoteException;

public class BankImpl extends Binder implements IBank {

    public BankImpl() {
        this.attachInterface(this, DESCRIPTOR);
    }

    public static IBank asInterface(IBinder obj) {
        if ((obj == null)) {
            return null;
        }
        android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
        if (((iin != null) && (iin instanceof IBank))) {
            return ((IBank) iin);
        }
        return new BankImpl.Proxy(obj);
    }

    @Override
    public IBinder asBinder() {
        return this;
    }

    @Override
    public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
            throws RemoteException {
        switch (code) {
        case INTERFACE_TRANSACTION: {
            reply.writeString(DESCRIPTOR);
            return true;
        }
        case TRANSACTION_queryMoney: {
            data.enforceInterface(DESCRIPTOR);
            int uid = data.readInt();
            long result = this.queryMoney(uid);
            reply.writeNoException();
            reply.writeLong(result);
            return true;
        }
        }
        return super.onTransact(code, data, reply, flags);
    }

    @Override
    public long queryMoney(int uid) throws RemoteException {
        return uid * 10l;
    }

    private static class Proxy implements IBank {
        private IBinder mRemote;

        Proxy(IBinder remote) {
            mRemote = remote;
        }

        @Override
        public IBinder asBinder() {
            return mRemote;
        }

        public java.lang.String getInterfaceDescriptor() {
            return DESCRIPTOR;
        }

        @Override
        public long queryMoney(int uid) throws RemoteException {
            Parcel data = Parcel.obtain();
            Parcel reply = Parcel.obtain();
            long result;
            try {
                data.writeInterfaceToken(DESCRIPTOR);
                data.writeInt(uid);
                mRemote.transact(TRANSACTION_queryMoney, data, reply, 0);
                reply.readException();
                result = reply.readLong();
            } finally {
                reply.recycle();
                data.recycle();
            }
            return result;
        }

    }

}
```

ok，到此为止，我们的Binder就完成了，这里只要创建服务端和客户端，二者就能通过我们的Binder来通信了。这里就不做这个示例了，我们的目的是分析代理模式在Binder中的使用。

我们看上述Binder的实现中，有一个叫做“Proxy”的类，它的构造方法如下：

```
  Proxy(IBinder remote) {
      mRemote = remote;
  }
```

Proxy类接收一个IBinder参数，这个参数实际上就是服务端Service中的onBind方法返回的Binder对象在客户端重新打包后的结果，因为客户端无法直接通过这个打包的Binder和服务端通信，因此客户端必须借助Proxy类来和服务端通信，这里Proxy的作用就是代理的作用，客户端所有的请求全部通过Proxy来代理，具体工作流程为：Proxy接收到客户端的请求后，会将客户端的请求参数打包到Parcel对象中，然后将Parcel对象通过它内部持有的Ibinder对象传送到服务端，服务端接收数据、执行方法后返回结果给客户端的Proxy，Proxy解析数据后返回给客户端的真正调用者。很显然，上述所分析的就是典型的代理模式。至于Binder如何传输数据，这涉及到很底层的知识，这个很难搞懂，但是数据传输的核心思想是共享内存。

## 三 行为型设计模式

![这里写图片描述](https://img-blog.csdn.net/20160308154820959) 

### 3.1 模板模式

#### 模式定义

> 定义一个操作的算法框架，而将具体实现延迟到子类中进行，使得子类在不改变整体算法框架的基础上，可以自定义算法实现。

#### 应用场景

- 多个子类有公有的方法，并且逻辑基本相同时。
- 重要复杂的算法可以把核心算法设计为模板方法，具体细节则由子类实现
- 重构代码时，把相同的代码抽取到父类中，然后通过钩子函数约束其行为。

#### Android应用-Activity/AsyncTask

* 启动一个Activity过程非常复杂，如果让开发者每次自己去调用启动Activity过程无疑是一场噩梦。好在启动Activity大部分代码时不同的，但是有很多地方需要开发者定制。也就是说，整体算法框架是相同的，但是将一些步骤延迟到子类中，比如Activity的onCreate、onStart等等。这样子类不用改变整体启动Activity过程即可重定义某些具体的操作了~ 

* Android 中 AsyncTask 的几个回调可以看作模板 

  在Android中，使用了模板方法且为我们熟知的一个典型类就是AsyncTask了，关于AsyncTask的更详细的分析请移步Android中AsyncTask的使用与源码分析，我们这里只分析在该类中使用的模板方法模式。

  在使用AsyncTask时，我们都有知道耗时的方法要放在doInBackground(Params... params)中，在doInBackground之前如果还想做一些类似初始化的操作可以写在onPreExecute方法中，当doInBackground方法执行完成后，会执行onPostExecute方法，而我们只需要构建AsyncTask对象，然后执行execute方法即可。我们可以看到，它整个执行过程其实是一个框架，具体的实现都需要子类来完成。而且它执行的算法框架是固定的，调用execute后会依次执行onPreExecute,doInBackground,onPostExecute,当然你也可以通过onProgressUpdate来更新进度。我们可以简单的理解为如下图的模式 :

  [![async-flow](https://github.com/simple-android-framework/android_design_patterns_analysis/raw/master/template-method/mr.simple/images/async-flow.png)](https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/template-method/mr.simple/images/async-flow.png)

  下面我们看源码，首先我们看执行异步任务的入口, 即execute方法 :

  ```
   public final AsyncTask<Params, Progress, Result> execute(Params... params) {
          return executeOnExecutor(sDefaultExecutor, params);
      }
  
      public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
              Params... params) {
          if (mStatus != Status.PENDING) {
              switch (mStatus) {
                  case RUNNING:
                      throw new IllegalStateException("Cannot execute task:"
                              + " the task is already running.");
                  case FINISHED:
                      throw new IllegalStateException("Cannot execute task:"
                              + " the task has already been executed "
                              + "(a task can be executed only once)");
              }
          }
  
          mStatus = Status.RUNNING;
  
          onPreExecute();
  
          mWorker.mParams = params;
          exec.execute(mFuture);
  
          return this;
      }
  ```

  可以看到execute方法(为final类型的方法)调用了executeOnExecutor方法，在该方法中会判断该任务的状态，如果不是PENDING状态则抛出异常，这也解释了为什么AsyncTask只能被执行一次，因此如果该任务已经被执行过的话那么它的状态就会变成FINISHED。继续往下看，我们看到在executeOnExecutor方法中首先执行了onPreExecute方法，并且该方法执行在UI线程。然后将params参数传递给了mWorker对象的mParams字段，然后执行了exec.execute(mFuture)方法。

  mWorker和mFuture又是什么呢？其实mWorker只是实现了Callable接口，并添加了一个参数数组字段，关于Callable和FutureTask的资料请参考[Java中的Runnable、Callable、Future、FutureTask的区别与示例](http://blog.csdn.net/bboyfeiyu/article/details/24851847)，我们挨个来分析吧，跟踪代码我们可以看到，这两个字段都是在构造函数中初始化。

  ```
     public AsyncTask() {
          mWorker = new WorkerRunnable<Params, Result>() {
              public Result call() throws Exception {
                  mTaskInvoked.set(true);
  
                  Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                  return postResult(doInBackground(mParams));
              }
          };
  
          mFuture = new FutureTask<Result>(mWorker) {
              @Override
              protected void done() {
                  try {
                      final Result result = get();
  
                      postResultIfNotInvoked(result);
                  } catch (InterruptedException e) {
                      android.util.Log.w(LOG_TAG, e);
                  } catch (ExecutionException e) {
                      throw new RuntimeException("An error occured while executing doInBackground()",
                              e.getCause());
                  } catch (CancellationException e) {
                      postResultIfNotInvoked(null);
                  } catch (Throwable t) {
                      throw new RuntimeException("An error occured while executing "
                              + "doInBackground()", t);
                  }
              }
          };
      }
  ```

  简单的说就是mFuture就包装了这个mWorker对象，会调用mWorker对象的call方法，并且将之返回给调用者。
  关于AsyncTask的更详细的分析请移步[Android中AsyncTask的使用与源码分析](http://blog.csdn.net/bboyfeiyu/article/details/8973058)，我们这里只分析模板方法模式。总之，call方法会在子线程中调用，而在call方法中又调用了doInBackground方法，因此doInBackground会执行在子线程。doInBackground会返回结果，最终通过postResult投递给UI线程。 我们再看看postResult的实现 :

  ```
      private Result postResult(Result result) {
          Message message = sHandler.obtainMessage(MESSAGE_POST_RESULT,
                  new AsyncTaskResult<Result>(this, result));
          message.sendToTarget();
          return result;
      }
  
      private static class InternalHandler extends Handler {
          @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
          @Override
          public void handleMessage(Message msg) {
              AsyncTaskResult result = (AsyncTaskResult) msg.obj;
              switch (msg.what) {
                  case MESSAGE_POST_RESULT:
                      // There is only one result
                      result.mTask.finish(result.mData[0]);
                      break;
                  case MESSAGE_POST_PROGRESS:
                      result.mTask.onProgressUpdate(result.mData);
                      break;
              }
          }
      }
  
  
      private void finish(Result result) {
          if (isCancelled()) {
              onCancelled(result);
          } else {
              onPostExecute(result);
          }
          mStatus = Status.FINISHED;
      }
  ```

  可以看到，postResult就是把一个消息( msg.what == MESSAGE_POST_RESULT)发送给sHandler，sHandler类型为InternalHandler类型，当InternalHandler接到MESSAGE_POST_RESULT类型的消息时就会调用result.mTask.finish(result.mData[0])方法。我们可以看到result为AsyncTaskResult类型，源码如下 :

  ```
      @SuppressWarnings({"RawUseOfParameterizedType"})
      private static class AsyncTaskResult<Data> {
          final AsyncTask mTask;
          final Data[] mData;
  
          AsyncTaskResult(AsyncTask task, Data... data) {
              mTask = task;
              mData = data;
          }
      }
  ```

  **可以看到mTask就是AsyncTask对象**，调用AsyncTask对象的finish方法时又调用了onPostExecute，这个时候整个执行过程就完成了。 总之，execute方法内部封装了onPreExecute, doInBackground, onPostExecute这个算法框架，用户可以根据自己的需求来在覆写这几个方法，使得用户可以很方便的使用异步任务来完成耗时操作，又可以通过onPostExecute来完成更新UI线程的工作。
  另一个比较好的模板方法示例就是Activity的声明周期函数，例如Activity从onCreate、onStart、onResume这些程式化的执行模板，这就是一个Activity的模板方法。

#### Java应用：

- `java.io.InputStream`, `java.io.OutputStream`, `java.io.Reader` 和 `java.io.Writer` 的所有非抽象方法.
- `java.util.AbstractList`, `java.util.AbstractSet` 和 `java.util.AbstractMap` 的所有非抽象方法.
- `javax.servlet.http.HttpServlet`, 所有 `doXXX()` 方法默认发送 `HTTP 405 "Method Not Allowed"` 错误到响应中. 你可以任意实现这些方法.

#### Code:

```java
package com.dp.example.templatemethod;

/**
 * 抽象的Computer
 * @author mrsimple
 *
 */
public abstract class AbstractComputer {

    protected void powerOn() {
        System.out.println("开启电源");
    }

    protected void checkHardware() {
        System.out.println("硬件检查");
    }

    protected void loadOS() {
        System.out.println("载入操作系统");
    }

    protected void login() {
        System.out.println("小白的电脑无验证，直接进入系统");
    }

    /**
     * 模板-启动电脑方法, 步骤固定为开启电源、系统检查、加载操作系统、用户登录。该方法为final， 防止算法框架被覆写.
     */
    public final void startUp() {
        System.out.println("------ 开机 START ------");
        powerOn();
        checkHardware();
        loadOS();
        login();
        System.out.println("------ 开机 END ------");
    }
}


package com.dp.example.templatemethod;

/**
 * 码农的计算机
 * 
 * @author mrsimple
 */
public class CoderComputer extends AbstractComputer {
    @Override
    protected void login() {
        System.out.println("码农只需要进行用户和密码验证就可以了");
    }
}


package com.dp.example.templatemethod;

/**
 * 军用计算机
 * 
 * @author mrsimple
 */
public class MilitaryComputer extends AbstractComputer {
    
 
    @Override
    protected void checkHardware() {
        super.checkHardware();
        System.out.println("检查硬件防火墙");
    }
    
    @Override
    protected void login() {
        System.out.println("进行指纹之别等复杂的用户验证");
    }
}


package com.dp.example.templatemethod;

public class Test {
    public static void main(String[] args) {
        AbstractComputer comp = new CoderComputer();
        comp.startUp();

        comp = new MilitaryComputer();
        comp.startUp();

    }
}

```



### 3.2 解释器模式

#### 模式定义

> 给定一个语音，定义它的文法的一种表示，并定义一个解释器。

应用场景

- 如果某个简单的语音需要解释执行并且可以将该语言中的语句表示为一个抽象语法树时可以使用解释器模式。
- 在某些特定领域不断出现的问题是，可以将该领域的问题转船为一种语法规则下的语句，然后构建解释器来解释该语句。

#### Android应用

PackageParser是对AndroidManifest.xml配置文件进行读取的，具体原理参考：[解析AndroidManifest原理](http://blog.csdn.net/zhbinary/article/details/7353739) 

这个用到的地方也不少，其一就是Android的四大组件需要在AndroidManifest.xml中定义，其实AndroidManifest.xml就定义了，等标签（语句）的属性以及其子标签，规定了具体的使用（语法），通过PackageManagerService（解释器）进行解析。PackageParser

### 3.3 策略模式

#### 模式定义

> 策略模式定义了一系列算法，并将算法封装起来可以互相替换，策略模式让算法与使用它的客户端解耦，可以独立变化。

应用场景

- 针对同一类型的问题有多种处理方式，仅仅是具体的行为有差别时。
- 需要安全的封装多种同一类型的操作时。
- 出现同一抽象类的多个字类，而又需要使用if-else来选择子类时。

#### UML类图

[![url](https://github.com/simple-android-framework/android_design_patterns_analysis/raw/master/strategy/gkerison/images/strategy-kerison-uml.png)](https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/strategy/gkerison/images/strategy-kerison-uml.png)

#### 角色介绍

- Context：用来操作策略的上下文环境。
- Strategy : 策略的抽象。
- ConcreteStrategyA、ConcreteStrategyB : 具体的策略实现。

策略模式的实现也非常简单，依赖于接口或者抽象类，如下所示：

```
// 策略接口，定义功能。
public interface IStrategy {
    void method();
}

// 策略A
public class StrategyA implements IStrategy {
    @Override
    public void method() {

    }
}

// 策略B
public class StrategyB implements IStrategy {
    @Override
    public void method() {
        
    }
}
// 设置策略->执行
public class Client{
    IStrategy strategy;
    public void setStrategy(IStrategy strategy) {
        this.strategy = strategy;
    }
    public void execute(){
        strategy.method();
    }
}
```



#### Android源码对应实现

动画里面的`插值器Interpolator`利用了策略模式, 利用`Interpolator`策略的抽象, `LinearInterpolator`,`CycleInterpolator`等插值器为具体的实现策略, 通过注入不同的插值器实现不同的动态效果.

看一下大概的类图

![img](https://github.com/suzeyu1992/repo/raw/master/project/design-pattern/%E7%9E%B0-%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B8%8EAndroid(%E7%AF%87%E4%B8%80)/imgs/UML_Animation.png) 

* 动画中的`TimeInterpolator`时间插值器, 它的作用是根据时间流逝的百分比计算出当前属性值改变的百分比, 内置的插值器有如下几种
    *  `线性插值器(LinearInterpolator)`用于匀速动画
    *  `加速减速插值器(AccelerateDecelerateInterpolator)`:起始时动画加速, 结尾时动画减速
    *  `减速插值器(DecelerateInterpolator)`: 用于随着时间的推移动画越来越慢.
* 动画中的`TypeEvalutor`类型估值器: 根据当前属性改变的百分比来计算改变后的属性值. 内置的类型估值器有如下几种
    * `整型估值器(IntEvalutor)`
    * `浮点型估值器(FloatEvalutor)`
    * `Color估值器(ArgbEvalutor)`
    
接下来就开始回忆一下从一个动画开始后, 代码究竟做了什么?

对于源码的起始点入口就是调用`View的startAnimation()`


```java
public void startAnimation(Animation animation) {
    // 1.初始化动画的开始时间
   animation.setStartTime(Animation.START_ON_FIRST_FRAME);
   // 2.对View设置动画
   setAnimation(animation);
   // 3.刷新父类缓存
   invalidateParentCaches();
   // 4.刷新View本身及子View
   invalidate(true);
}
```

这里首先设置了动画的起始时间, 然后将该动画设置到`View`中, 最后再向`ViewGroup`请求刷新视图, 随后`ViewGroup`会调用`dispatchDraw()`方法对这个`View`所在的区域进行重绘. 其实对于某一个`View`的重绘最终是调用其`ViewGroup`的`drawChild(...)`方法. 跟入一下


```java
protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
   // 简单的转发
   return child.draw(canvas, this, drawingTime);
}

boolean draw(Canvas canvas, ViewGroup parent, long drawingTime) {
    // ....
    // 查看是否需要清除动画信息
    final int flags = parent.mGroupFlags;
    // 省略无关代码
    
    // 获取设置的动画信息
    final Animation a = getAnimation();
        
    if (a != null) {
            // 绘制动画
           more = drawAnimation(parent, drawingTime, a, scalingRequired);
           //...
       } 
}
```

父类会调用子类的`draw`方法, 其中会先判断是否设置了清除动画的标记, 然后再获取该`View`动画信息, 如果设置了动画, 就会调用`View#drawAnimation()`方法. 


```java
private boolean drawAnimation(ViewGroup parent, long drawingTime,
       Animation a, boolean scalingRequired) {
   Transformation invalidationTransform;
   final int flags = parent.mGroupFlags;
   final boolean initialized = a.isInitialized();
   // 1. 判断动画是否已经初始化过
   if (!initialized) {
       a.initialize(mRight - mLeft, mBottom - mTop, parent.getWidth(), parent.getHeight());
       a.initializeInvalidateRegion(0, 0, mRight - mLeft, mBottom - mTop);
       if (mAttachInfo != null) a.setListenerHandler(mAttachInfo.mHandler);
       // 如果设置了动画的监听, 则触发对应的回调
       onAnimationStart();
   }
   // 获取Transformation对象, 存储动画的信息
   final Transformation t = parent.getChildTransformation();
   // 2. 调用Animation#getTransformation, 通过计算获取动画的相关值
   boolean more = a.getTransformation(drawingTime, t, 1f);
    

   if (more) {
        // 3. 根据具体实现, 判断当前动画类型是否需要进行调整位置大小, 然后刷新不同的区域
       if (!a.willChangeBounds()) {
           // ...
       } else {
           // 获取重绘区域
           a.getInvalidateRegion(0, 0, mRight - mLeft, mBottom - mTop, region,
                   invalidationTransform);
           parent.mPrivateFlags |= PFLAG_DRAW_ANIMATION;

            // 更新计算有效区域
           final int left = mLeft + (int) region.left;
           final int top = mTop + (int) region.top;
           
           // 进行区域更新
           parent.invalidate(left, top, left + (int) (region.width() + .5f),
                   top + (int) (region.height() + .5f));
       }
   }
   return more;
}
```

`drawAnimation`中主要操作是动画的初始化, 动画操作, 界面刷新. 动画的回调监听`onStart()`会在动画进行初始化的时候调用, 动画的具体实现是通过`Animation#getTransformation()`方法.这个方法主要获取了`缩放系数`和调用`Animation.getTransformation(long, Transformation)`来计算和应用动画效果.


```java
public boolean getTransformation(long currentTime, Transformation outTransformation) {
   //...
   float normalizedTime;
   // 1.计算当前时间的流逝百分比
   if (duration != 0) {
       normalizedTime = ((float) (currentTime - (mStartTime + startOffset))) /
               (float) duration;
   } else {
       // time is a step-change with a zero duration
       normalizedTime = currentTime < mStartTime ? 0.0f : 1.0f;
   }
   // 动画是否完成标记
   final boolean expired = normalizedTime >= 1.0f;
   mMore = !expired;

   if ((normalizedTime >= 0.0f || mFillBefore) && (normalizedTime <= 1.0f || mFillAfter)) {
        // 2.通过插值器获取动画执行百分比  , 这里获取的方法就是通过策略模式
       final float interpolatedTime = mInterpolator.getInterpolation(normalizedTime);
       // 3.应用动画效果
       applyTransformation(interpolatedTime, outTransformation);
   }

   // 4. 如果动画执行完毕, 那么触发动画完成的回调或者执行重复动画等操作
   // ...
   if (!mMore && mOneMoreTime) {
       mOneMoreTime = false;
       return true;
   }
   return mMore;
}
```

这段代码, 先计算已经流逝的的时间百分比, 然后再通过`具体的插值器`重新计算这个百分比, 也就是上面的第二步. 而具体是哪一个插值器是通过之前说的策略模式来实现的. 

第3步调用了`applyTransformation`, 这个方法在基类`Animation`中是空实现, 可以在子类查看实现如`ScaleAnimation`,`AlphaAnimation`等查看.  当这个方法内部主要通过`矩阵`来实现动画.  当这个方法执行完毕之后, View的属性也就发生了变化, 不断地重复这个过程, 动画就随之产生. 




Java应用





### 3.4 状态模式

#### 模式定义

> 允许一个对象在内部状态改变时改变它的行为。

应用场景

- 一个对象的行为取决于它的状态，并且它在运行时根据状态改变它的行为。
- 代码中包含大量与对象状态相关的判断语句。

状态模式的核心是根据不同的状态对应不同的操作，如下所示：

```
// 操作接口
public interface TVState {
    void nextChannel();

    void lastChannel();
}

// 开机状态
public class PowerOnState implements TVState {

    @Override
    public void nextChannel() {
        // next channel 
    }

    @Override
    public void lastChannel() {
        // last channel
    }
}

// 关机状态
public class PowerOffChannel implements TVState {
    @Override
    public void nextChannel() {
        // do nothing
    }

    @Override
    public void lastChannel() {
        // do nothing
    }
}
```



#### Android应用-StateMachine机制

<http://blog.csdn.net/maybe_windleave/article/details/9881991>

<http://blog.csdn.net/lilian0118/article/details/21974229>



在我们的应用开发中也可以使用源码中的`StateMachine`类，只要从源码中把`StateMachine`和`State`类拷贝到我们的工程目录就可以使用了，对于其 import 的一些类，在我们的开发环境中也全都有，不用担心。

如果你对状态机有那么一丝丝的了解，那么学习 StateMachine 类最好的资料就是它的注释了，解释的一清二楚。

简单使用阐述

Android 中的状态机是一个分层的消息处理机制，每一层都会有一到多个节点，而状态机的消息就是在这些节点之间流转处理，如下结构所示：

```
	// 状态机分层结构
          mP0
         /   \
        mP1   mS0
       /   \
      mS2   mS1
     /  \    \
    mS3  mS4  mS5  ---> initial state 初始节点
```

而节点就是`State`类，它实现了`IState`接口，除了`enter`、`exit`方法外，还有`processMessage`方法，表示用来处理节点的消息。若返回`HANDLED`则表示消息处理完成，若返回`NOT_HANDLED`则表示消息没有处理。

构造状态机

在我们使用 StateMachine 之前，要构造好所需的状态分层结构。通过`addState`方法来向状态机中添加节点，例如如下的状态结构，mS1 和 mS2 节点有公共的父节点 mP1，同时还有一个孤立的节点 mP2。

```
      // 状态分层结构设定
        mP1      mP2
       /   \
      mS2   mS1
      // 构造状态机结构代码
      addState(mP1);
          addState(mS1, mP1);
          addState(mS2, mP1);
      addState(mP2);
```

`addState`方法添加节点时，还能指定其父节点添加。

当我们构造完了状态机时，还需要指定其中一个节点为启动点，消息从启动点开始处理，通过`setInitialState`方法来指定启动点，最后通过`start`方法启动状态机。

状态机消息处理

当构造完想要的状态机结构时，就是对状态机内部消息流转的处理了。

当`start`状态机时，状态机的第一个动作就是调用节点的`enter`方法，不过，它调用的是指定的启动点的最远的父节点的`enter`方法，然后再是次一级的父节点的`enter`方法，最后才是启动点的`enter`方法，就如同上面的结构所示，先调用 mP1 点，然后才是 mS1 点的方法，此时 mS1 节点就是状态机的当前对外的点。由此可见，当启动点进入 `enter`状态时，它的父节点，直至最顶层的父节点都进入了`enter`状态了。

状态机中的每个节点都 0 个或 1 个父节点，当子节点不能处理当前消息时，它可以通过返回`NOT_HANDLED`将当前消息传递给其父节点来处理。如果一个消息从未被处理过，那么`unhandledMessage`方法将会被调用给最后一次机会来处理该消息。

除此之外，节点还可以通过`transitionTo`方法将当前节点转移至另外一个新的节点。

```
          mP0
         /   \
        mP1   mS0
       /   \
      mS2   mS1
     /  \    \
    mS3  mS4  mS5  ---> initial state
```

例如，当 mS5 处理消息，想要将当前节点转移至 mS4 时，那么它会先找到 mS5 和 mS4 最近的公有父节点 mP1。然后，除了这个最近的公有父节点 mP1 以及它的上层节点外，mS5 进入`enter`状态时启动的那些父节点都会退出调用`exit`方法。最后再由 mP1 节点下的节点调用`enter`方法直到新节点 mS4 调用了`enter`方法。

也就是说，从 mS5 到 mS4 状态的转变，先是 mS5、mS1 调用了`exit`方法，再是 mS2、mS4 调用了`enter`方法，这就是状态机中节点发生状态转移时的调用过程。

除此之外，节点还可以调用`deferMessage`和`sendMessageAtFroneOfQueue`方法。`deferMessage`方法使得消息存储在消息队列中，当状态转移到新节点时才会处理，而`sendMessageAtFrontOfQueue`方法则是将消息放置到消息队列的头部。

当状态机的所有消息都完成时，可以调用`transitionToHaltingState`方法来将状态机处理停止状态。此时，状态机将转移到`HaltingState`停止状态，并调用`halting`方法。随后收到的所有消息都只是会调用`haltedProcessMessage`方法来处理了。

若想要完全的停止状态机，则可以使用`quit`或者`quitNow`方法来处理。

以上就是状态机对于消息处理的过程，长篇的文字说明还是不如代码来的直观，这里就不贴完整的代码了，参考代码的连接如下：[GithubGist 链接地址](https://gist.github.com/glumes/e76ea009843eecb2b1e5cf9d8a38a369)

状态机实现原理分析

如果我们在初始化状态机时只是传递了一个名字，而没有传递 Looper 或者 Handler 之类的消息循环，那么状态机默认就是启用其内部的一个线程`HandlerThread`。

```java
  protected StateMachine(String name) {
        mSmThread = new HandlerThread(name); // 创建 HandlerThread 线程
        mSmThread.start();
        Looper looper = mSmThread.getLooper();
        initStateMachine(name, looper);
    }
    // 将消息循环 Looper 与 Handler 进行绑定
  private void initStateMachine(String name, Looper looper) {
        mName = name;
        mSmHandler = new SmHandler(looper, this);
    }
```

构造状态及时，在其内部开启了一个线程，并将其消息循环 Looer 传递给了 `SmHandler` 对象，而`SmHandler`对象就是状态机中最主要的用来派发消息事件和切换状态的了，它派发的消息都是在 HandlerThread 线程进行处理的。

同时，SmHandler 内部有两个数组，用来保存状态机中的链式状态关系，分别是`mStateStack`和`mTempStateStack`变量。当状态机完成启动时，就会通过上面来个变量来保存节点信息。

而状态机的消息处理，内部也是通过 SmHandler 来处理转发的。

具体的实现，建议参考这篇文章：<http://blog.csdn.net/yangwen123/article/details/10591451> 讲的实在太详细了，拜读了多遍也不敢说写的能比它更清楚。

### 3.5 观察者模式

#### 模式定义

> 定义对象间一对多的依赖关系，每当这个对象发生改变时，其他对象都能收到通知并更新自己。

应用场景

- 关联行为场景
- 事件多级触发场景
- 跨系统的消息交换场景，例如消息队列，事件总线的处理机制。

观察者模式的实现如下所示：

```
// 监听接口，通知被监听对象发生改变
public interface Listener {

    void change();
}

// 被监听者
public class Observable {

    private Listener mListener;

    // 设置监听器
    public void setListener(Listener listener) {
        mListener = listener;
    }

    public void onChange() {
        // 通知对象发生改变
        mListener.change();
    }
}

// 监听者
public class Observer {

    public void setup() {
        Observable observable = new Observable();
        observable.setListener(new Listener() {
            @Override
            public void change() {
                // TODO 监听的对象发生改变
            }
        });
    }
}
```

#### 实际应用

* android应用

  我们看看ListView的适配器，有个函数notifyDataSetChanged()函数，这个函数其实就是通知ListView的每个Item，数据源发生了变化，请各位Item重新刷新一下。 

* jdk应用
  - `java.util.Observer/java.util.Observable` (在现实世界中很少使用)
  - `java.util.EventListener` 的所有实现 (几乎所有的`Swing`都是这样)
  - `javax.servlet.http.HttpSessionBindingListener`
  - `javax.servlet.http.HttpSessionAttributeListener`
  - `javax.faces.event.PhaseListener`

#### **1 JDK源码中的观察者模式**        

​        不管是在《大话设计模式》，还是GoF的《设计模式》，还是别的关于设计模式的经典书籍，抑或是在各类技术博客，大家可以轻易地找到关于观察者模式的定义和示例。因此观察者模式的概念和基本使用不是本文的讨论重点。重点是，观察者模式的使用是如此的广泛，以至于JDK源码中已为该模式提供java.util.Observable基类和java.util.Observers接口。

![JDK中的观察者模式UML类图](https://img-blog.csdn.net/20160824023119734?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

​        JDK中的观察者模式的使用如图所示：Student类实现Observer接口，通过addObserver方法注册到Teacher的观察者列表中去；而Teacher类继承自Observable基类，当Teacher发布消息（publishMessage)时调用notifyObservers()方法，遍历观察者列表中的所有Student，调用他们的update方法；于是Teacher中的message传递到了每个Student中。

​        JDK中的Observable基类和Observer接口为我们的简单使用观察者模式提供了方便，但它有一个非常明显的缺点：为了使用观察者模式，需要让Teacher继承Observable这个基类，但是Teacher可能已经继承了Person类，而Java又是单继承，不能同时再继承Observable类。解决这个矛盾的思路有两种：一是自定义观察者模式，将add、delete、notify等方法写进Teacher类；二是使用代理模式，在Teacher中维护一个Observable类的对象，并且实现同名的方法，类似如下代码。

```
    public void addObserver(Observer observer){



        mObservable.addObserver(observer);



    }



    此处不展开讨论了，Android源码中的观察者就是采用类似第二种思路的方法。
```

#### 2 Android源码中的观察者模式

 

​      其实翻看Android源码，发现Android并没有使用JDK的Observable基类和Observer接口。Instead，Android定义了一个android.database.Observable<T>抽象类，其中T是Oberver type。Observable抽象类中只有registerObserver、unregisterObserver和unregisterAll三个抽象方法，notify的操作需要子类自己定义。与JDK中Observer接口相对应的是DataSetObserver抽象类，里面有onChanged和onValidated两个抽象方法。notify方法、DataSetObserver类、onChanged方法，是不是想到了什么东西？对，setDataSetChanged()！是不是很熟悉？

##### **2.1 ListView源码中的观察者模式**

​      在使用ListView时，数据改变后，我们会手动去调用ListView对应的adapter的setDataSetChanged()方法来通知ListView更新UI。换一种说法，ListView的UI是观察者，ListView对应的adapter中的数据是被观察者，ListView通过注册一个观察者到adapter中，以实现监听adapter的数据变化的目的。经过刨祖坟一般的Ctrl+鼠标左键之后，画出如下UML类图。图中可以看到，ListView注册到adapter中的观察者叫AdapterDataSetObserver，定义在ListView的父类AbsListview中。它又继承自AbsListView的父类AdapterView类中的同名AdapterDataSetObserver。AdapterView.AdapterDataSetObserver最终继承自DataSetObserver抽象类。

![ListView中的观察者模式UML类图](https://img-blog.csdn.net/20160824014820658?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

​      那当数据发生改变，要更新 UI 的时候怎么办呢？

Android 的做法是，在 Adapter 中添加一个『老板』对象，让 Adapter 拥有『老板』的所有权利。然后在 ListView 中，添加一个『员工』对象（继承 DataSetObserver，在重写父类方法时，更新 UI）。另外在 ListView 的 setAdapter 方法中，将『员工』和『老板』绑定起来，实现『老板』和『员工』的雇佣关系。最后开发者就可以调用 Adapter 的方法notifyDataSetChanged()，让『老板』命令『员工』干活了！

接下来我们看一下代码实现。<br />
 **BaseAdapter.Java：** 拥有DataSetObservable对象，就是说Adapter是『老板』的上司，拥有『老板』的所有权利。

```
public abstract class BaseAdapter {
    private final DataSetObservable mDataSetObservable = new DataSetObservable();
    public void registerDataSetObserver(DataSetObserver observer) {
        mDataSetObservable.registerObserver(observer);
    }
    public void unregisterDataSetObserver(DataSetObserver observer) {
        mDataSetObservable.unregisterObserver(observer);
    }
    public void notifyDataSetChanged() {
        mDataSetObservable.notifyChanged();
    }
    public void notifyDataSetInvalidated() {
        mDataSetObservable.notifyInvalidated();
    }
}
```

**ListView.Java：**setAdapter()方法中，将 AdapterDataSetObserver 注册到 Adapter 的DataSetObservable中，实现『老板』和『员工』的雇佣关系。

```
public class ListView {
    ...代码省略
    @Override
    public void setAdapter(ListAdapter adapter) {
        ...代码省略
        mDataSetObserver = new AdapterDataSetObserver();
        mAdapter.registerDataSetObserver(mDataSetObserver);
        ...代码省略
    }
    ...代码省略
}
```

AdapterDataSetObserver 继承了 DataSetObserver，重写父类函数时，进行更新界面的操作。



##### 2.2 RecyclerView中的观察者模式

 

​      由于最近使用RecyclerView代替ListView，所以把RecyclerView中的观察者模式的实现源码也挖出来了.



### 3.6 备忘录模式

#### 模式定义

> 在不破坏封装的前提下，保存一个对象的内部状态，并在该对象之外保存这个状态，以便可以将该对象恢复到保存时的状态。

应用场景

- 需要保存某个对象在某一时刻的状态。
- 外界向访问对象的状态，但是又不想直接暴露接口给外部，这时候可以将对象状态保存下来，间接的暴露给外部。

#### 实际应用

* Activity的onSaveInstanceState和onRestoreInstanceState就是用到了备忘录模式，分别用于保存和恢复。 

* #### jdk源码中应用

  - `java.util.Date` ( setter 方法就是这么做的, `Date` 内部通过长整型`long`表示)
  - `java.io.Serializable`的所有实现
  - `javax.faces.component.StateHolder`的所有实现

### 3.7 中介者模式

#### 模式定义

> 中介者模式定义了一系列对象间的交互方式，使得这些对象像话作用而又不耦合在一起。

应用场景

- 当对象间有很多的交互操作，而且一个对象的行为依赖于其他对象时，可以利用中介者模式解决紧耦合的问题。

#### Android应用

中介者模式的意图为：用一个中介对象来封装一系列的对象交互。中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。中介者模式的结构图如下：

![1532708415340](C:\Users\Li\AppData\Local\Temp\1532708415340.png)

 

​     在 ANDROID系统中keyguard的功能实现采用了中介者模式，用来中介keyguard相关的请求，包括查询keyguard的状态，影响keyguard应当显示和复位的电源管理事件，以及当keyguard显示时对窗口管理的通知事件和来自keyguard视图本身的关于keyguard是否成功unlocked的事件等。相关UML类图如下：

​      ![img](C:\Users\Li\AppData\Local\Temp\1532708445035.png)

​      其中KeyguardViewMediator作为中介者角色，与电源管理、用户管理、报警管理、声音管理、状态条管理、KeyguardViewManager、KeyguardDisplayManager、KeyguardUpdateMonitor等服务或对象交互， 读取相关状态，执行和触发keyguard事件相关的功能等 ，而KeyguardViewManager、KeyguardHostView、KeyguardUpdateMonitor类通过相关回调向KeyguardViewMediator传送Keyguard视图本身和keyguard有关状态更新方面的事件， 另外KeyguardService服务也是通过KeyguardViewMediator查询keyguard的状态 并通过IKeyguardService接口对外提供keyguard的状态信息。



### 3.8 命令模式

#### 模式定义

> 将一个请求封装成一个对象，可以将不同的请求参数化，可以对请求就行排队、日志记录以及撤销等操作。

应用场景

- 需要抽象待执行的动作，然后以参数的形式提供出来。
- 在不同的时刻，指定和排列请求。
- 需要支持撤销操作。
- 需要支持日志功能，这样当系统崩溃时，可以重做一遍。
- 需要支持事务操作。

命令模式的实现如下所示：

```
// 接收命令
public class Receiver {

    public void action() {
        // TODO 真正执行命令具体逻辑
    }
}

// 抽象命令
public interface AbstractCommand {
    
    // 执行命令
    void command();
}

// 具体命令
public class ConcreteCommand implements AbstractCommand {

    private Receiver mReceiver;

    public ConcreteCommand(Receiver receiver) {
        mReceiver = receiver;
    }

    @Override
    public void command() {
        mReceiver.action();
    }
}

// 调用者
public class Invoker {

    private AbstractCommand mCommmand;

    public Invoker(AbstractCommand command) {
        mCommmand = command;
    }

    public void invoke() {
        mCommmand.command();
    }
}
```



#### Android源码中的模式实现

Command接口中定义了一个execute方法，客户端通过Invoker调用命令操作再来调用Recriver执行命令；把对Receiver的操作请求封装在具体的命令中，使得命令发起者和命令接收者解耦。 以Android中大家常见的Runnable为例：客户端只需要new Thread(new Runnable(){}).start()就开始执行一系列相关的请求，这些请求大部分都是实现Runnable接口的匿名类。 【O_o 模式就在我们身边~】

命令接口Runnable接口定义如下：

```
package java.lang;
/**
 * Represents a command that can be executed. Often used to run code in a
 * different {@link Thread}.
 */
public interface Runnable {

    /**
     * Starts executing the active part of the class' code. This method is
     * called when a thread is started that has been created with a class which
     * implements {@code Runnable}.
     */
    public void run();
}
```

调用者Thread源码如下（省略部分代码）： Tips：命令模式在这里本来不需要继承Runnable接口，但为了方便性等，继承了Runnable接口实现了run方法，这个run是Thread自身的运行run的方法，而不是命令Runnable的run。

```
public class Thread implements Runnable {
    //省略部分无关代码...
    /* some of these are accessed directly by the VM; do not rename them */
    volatile VMThread vmThread;
    volatile ThreadGroup group;
    volatile boolean daemon;
    volatile String name;
    volatile int priority;
    volatile long stackSize;
    Runnable target;
    private static int count = 0;
    
    public synchronized void start() {
        if (hasBeenStarted) {
            throw new IllegalThreadStateException("Thread already started."); // TODO Externalize?
        }

        hasBeenStarted = true;

        VMThread.create(this, stackSize);
    }
    //省略部分代码...
}
```

上面可以看到执行start()方法的时候实际执行了VMThread.create(this, stackSize)方法；create是VMThread的本地方法，其JNI实现在 android/dalvik/vm/native/java_lang_VMThread.cpp 中的 Dalvik_java_lang_VMThread_create方法，如下：

```
static void Dalvik_java_lang_VMThread_create(const u4* args, JValue* pResult)
{
    Object* threadObj = (Object*) args[0];
    s8 stackSize = GET_ARG_LONG(args, 1);

    /* copying collector will pin threadObj for us since it was an argument */
    dvmCreateInterpThread(threadObj, (int) stackSize);
    RETURN_VOID();
}
```

而dvmCreateInterpThread的实现在Thread.app中，如下：

```
bool dvmCreateInterpThread(Object* threadObj, int reqStackSize){
    Thread* self = dvmThreadSelf();
    
    Thread* newThread = allocThread(stackSize); 
    newThread->threadObj = threadObj;
    
    Object* vmThreadObj = dvmAllocObject(gDvm.classJavaLangVMThread, ALLOC_DEFAULT);
    dvmSetFieldInt(vmThreadObj, gDvm.offJavaLangVMThread_vmData, (u4)newThread);
    dvmSetFieldObject(threadObj, gDvm.offJavaLangThread_vmThread, vmThreadObj);
    
    pthread_t threadHandle;
    int cc = pthread_create(&threadHandle, &threadAttr, interpThreadStart, newThread);

    dvmLockThreadList(self);

    assert(newThread->status == THREAD_STARTING);
    newThread->status = THREAD_VMWAIT;
    pthread_cond_broadcast(&gDvm.threadStartCond);

    dvmUnlockThreadList();
    
}

static Thread* allocThread(int interpStackSize)
{
    Thread* thread;
    thread = (Thread*) calloc(1, sizeof(Thread));
    
    thread->status = THREAD_INITIALIZING;
}
```

这里是底层代码，简单介绍下就行了： 第4行通过调用 allocThread 创建一个名为newThread的dalvik Thread并设置一些属性，第5行设置其成员变量threadObj为传入的Android Thread，这样dalvik Thread就与Android Thread对象关联起来了；第7行然后创建一个名为vmThreadObj的VMThread对象，设置其成员变量vmData为前面创建的newThread，设置 Android Thread threadObj的成员变量vmThread为这个vmThreadObj，这样Android Thread通过VMThread的成员变量vmData就和dalvik Thread关联起来了。

接下来在12行通过pthread_create创建pthread线程，并让这个线程start，这样就会进入该线程的thread entry运行，下来我们来看新线程的thread entry方法 interpThreadStart，同样只列出关键的地方：

```
//pthread entry function for threads started from interpreted code.
static void* interpThreadStart(void* arg){
    Thread* self = (Thread*) arg;
    std::string threadName(dvmGetThreadName(self));
    setThreadName(threadName.c_str());

    //Finish initializing the Thread struct.
    dvmLockThreadList(self);
    prepareThread(self);

    while (self->status != THREAD_VMWAIT)
        pthread_cond_wait(&gDvm.threadStartCond, &gDvm.threadListLock);

    dvmUnlockThreadList();

    /*
     * Add a JNI context.
     */
    self->jniEnv = dvmCreateJNIEnv(self);

    //修改状态为THREAD_RUNNING
    dvmChangeStatus(self, THREAD_RUNNING);
    
    //执行run方法
    Method* run = self->threadObj->clazz->vtable[gDvm.voffJavaLangThread_run];

    JValue unused;
    ALOGV("threadid=%d: calling run()", self->threadId);
    assert(strcmp(run->name, "run") == 0);
    dvmCallMethod(self, run, self->threadObj, &unused);
    ALOGV("threadid=%d: exiting", self->threadId);
    
    //移出线程并释放资源
    dvmDetachCurrentThread();
    return NULL;
}

//Finish initialization of a Thread struct.
static bool prepareThread(Thread* thread){
    assignThreadId(thread);
    thread->handle = pthread_self();
    thread->systemTid = dvmGetSysThreadId();
    setThreadSelf(thread);
    return true;
}

//Explore our sense of self.  Stuffs the thread pointer into TLS.
static void setThreadSelf(Thread* thread){
    int cc;
    cc = pthread_setspecific(gDvm.pthreadKeySelf, thread);
}
```

在新线程的interpThreadStart方法中，首先设置线程的名字，然后调用prepareThread设置线程id以及其它一些属性，其中调用了setThreadSelf将新dalvik Thread自身保存在TLS中，这样之后就能通过dvmThreadSelf方法从TLS中获取它。然后在29行处修改状态为THREAD_RUNNING，并在36行调用对应Android Thread的run()方法，其中调用了Runnable的run方法，运行我们自己的代码。 绕这么深才执行到我们的run方法，累不累？ v_v

```
    /**
     * Calls the <code>run()</code> method of the Runnable object the receiver
     * holds. If no Runnable is set, does nothing.
     * @see Thread#start
     */
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```

到此我们已经完成一次命令调用，至于底层run调用完毕后续执行代码，读者可以自行跟进看看~~~

### 3.9 访问者模式

#### 模式定义

> 封装一些作用于某些数据结构中的各元素的操作，它可以在不改变数据结构的前提下赋予这些元素新的操作。

应用场景

- 对象结构比较稳定，但是需要在对象结构的基础上定义新的操作。
- 需要对同一个类的不同对象执行不不同的操作，但是不希望增加操作的时候改变这些类。

#### Android源码对应实现

`APT`的注解. 简单记录一下. 首先编译器将代码抽象成一个代码元素的树, 然后在编译时对整棵树进行遍历访问, 每个元素都有一个`accept()`接收访问者的访问, 每个访问者中都有对应的`visit()`函数, 例如`visitType()`函数就是对类型元素的访问, 在每个`visit`函数中对不同的类型进行不同的处理, 这样就达到了差异处理效果, 同时将数据结构与数据操作分离, 使得每个类型的职责单一, 易于升级维护. `JDK`还特意预留了`visitUnknown()`接口应对`Java`语言后续发展可能添加的元素类型问题, 灵活的将访问者模式的缺点化解.

https://blog.csdn.net/xxxzhi/article/details/51747513

编译时注解依赖于APT(Annotation Processing Tools)实现，在编译器会自己生成相关的java类。

注解可以指定作用于哪种元素上，  比如：

PackageElement，包元素

TypeElement，类型元素

ExecutableElement，可执行元素

VariableElement，变量元素

TypeParameterElement，类型参数元素

Element基类里面有一个accept(ElementVisitor<R,p> v,P p)方法

ElementVisitor里面又根据不同的元素类型重载很很多visit方法。

比如类型元素访问者 里面是visitType(TypeElement e,P p)方法，然后里面做了这种类型元素需要的操作。



 



### 3.10 责任链模式

#### 模式定义

> 将请求的发送者和接收者进行解耦，使得多个对象都有机会处理该请求，将这些对象串成一条链，并沿着这条链子处理请求，直到有对象处理它为止。

应用场景

- 多个对象可以处理同一个请求，但是具体由哪个对象处理在运行时动态决定。
- 在请求处理者不明确的情况下向多个对象中的一个提交请求。
- 需要动态指定一组对象的处理请求。

责任链模式的实现主要在于处理器的迭代，要么使用循环迭代，要么使用链表后继，如下所示：

```
// 处理器，定位行为和下一个处理器
public abstract class Handler {
    protected Handler next;

    public abstract void handleRequest(String condition);
}

// 处理器1
public class Handler1 extends Handler {
    @Override
    public void handleRequest(String condition) {
        if (TextUtils.equals(condition, "Handler1")) {
            // process request
        } else {
            // next handler
            next.handleRequest(condition);
        }
    }
}

// 处理器2
public class Handler2 extends Handler {
    
    @Override
    public void handleRequest(String condition) {
        if (TextUtils.equals(condition, "Handler2")) {
            // process request
        } else {
            // next handler
            next.handleRequest(condition);
        }
    }
}
```



#### Android应用-事件分发

![img](https://upload-images.jianshu.io/upload_images/966283-d01a5845f7426097.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700) 

https://www.jianshu.com/p/e99b5e8bd67b

### 3.11 迭代器模式

模式定义

> 提供一个方法顺序访问一个容器内的元素，而又不暴露该对象的内部表示。

应用场景

- 遍历一个容器时。

#### UML类图

　　　 　[![iterator](https://github.com/simple-android-framework/android_design_patterns_analysis/raw/master/iterator/haoxiqiang/images/Iterator_UML_class_diagram.svg.png)](https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/iterator/haoxiqiang/images/Iterator_UML_class_diagram.svg.png)

#### 角色介绍　　

- 迭代器接口Iterator：该接口必须定义实现迭代功能的最小定义方法集比如提供hasNext()和next()方法。
- 迭代器实现类：迭代器接口Iterator的实现类。可以根据具体情况加以实现。
- 容器接口：定义基本功能以及提供类似Iterator iterator()的方法。
- 容器实现类：容器接口的实现类。必须实现Iterator iterator()方法。

#### Android应用

一个集合想要实现Iterator很是很简单的,需要注意的是每次需要重新生成一个Iterator来进行遍历.且遍历过程是单方向的,HashMap是通过一个类似HashIterator来实现的,我们为了解释简单,这里只是研究ArrayList(此处以Android L源码为例,其他版本略有不同)

```
@Override public Iterator<E> iterator() {
    return new ArrayListIterator();
}

private class ArrayListIterator implements Iterator<E> {
    /** Number of elements remaining in this iteration */
    private int remaining = size;

    /** Index of element that remove() would remove, or -1 if no such elt */
    private int removalIndex = -1;

    /** The expected modCount value */
    private int expectedModCount = modCount;

    public boolean hasNext() {
        return remaining != 0;
    }

    @SuppressWarnings("unchecked") public E next() {
        ArrayList<E> ourList = ArrayList.this;
        int rem = remaining;
        if (ourList.modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        if (rem == 0) {
            throw new NoSuchElementException();
        }
        remaining = rem - 1;
        return (E) ourList.array[removalIndex = ourList.size - rem];
    }

    public void remove() {
        Object[] a = array;
        int removalIdx = removalIndex;
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        if (removalIdx < 0) {
            throw new IllegalStateException();
        }
        System.arraycopy(a, removalIdx + 1, a, removalIdx, remaining);
        a[--size] = null;  // Prevent memory leak
        removalIndex = -1;
        expectedModCount = ++modCount;
    }
}
```

- java中的写法一般都是通过iterator()来生成Iterator,保证iterator()每次生成新的实例
- remaining初始化使用整个list的size大小,removalIndex表示remove掉的位置,modCount在集合大小发生变化的时候后都会进行一次modCount++操作,避免数据不一致,前面我写的例子这方面没有写,请务必注意这点
- hasNext方法中,因为remaining是一个size->0的变化过程,这样只需要判断非0就可以得知当前遍历的是否还有未完成的元素
- next,第一次调用的时候返回array[0]的元素,这个过程中removalIndex会被设置成当前array的index
- remove的实现是直接操作的内存中的数据,是能够直接删掉元素的,不展开了



## 三种模式关系



### 1．创建型模式

创建型模式，就是创建对象的模式，抽象了实例化的过程。它帮助一个系统独立于如何创建、组合和表示它的那些对象。关注的是对象的创建，创建型模式将创建对象的过程进行了抽象，也可以理解为将创建对象的过程进行了封装，作为客户程序仅仅需要去使用对象，而不再关系创建对象过程中的逻辑。

社会化的分工越来越细，自然在软件设计方面也是如此，因此对象的创建和对象的使用分开也就成为了必然趋势。因为对象的创建会消耗掉系统的很多资源，所以单独对对象的创建进行研究，从而能够高效地创建对象就是创建型模式要探讨的问题。这里有6个具体的创建型模式可供研究，它们分别是：

- 简单工厂模式（Simple Factory）
- 工厂方法模式（Factory Method）
- 抽象工厂模式（Abstract Factory）
- 创建者模式（Builder）
- 原型模式（Prototype）
- 单例模式（Singleton）

> 简单工厂模式不是GoF总结出来的23种设计模式之一

### 2．结构型模式

结构型模式是为解决怎样组装现有的类，设计它们的交互方式，从而达到实现一定的功能目的。结构型模式包容了对很多问题的解决。例如：扩展性（外观、组成、代理、装饰）、封装（适配器、桥接）。

在解决了对象的创建问题之后，对象的组成以及对象之间的依赖关系就成了开发人员关注的焦点，因为如何设计对象的结构、继承和依赖关系会影响到后续程序的维护性、代码的健壮性、耦合性等。对象结构的设计很容易体现出设计人员水平的高低，这里有7个具体的结构型模式可供研究，它们分别是：

- 外观模式/门面模式（Facade门面模式）
- 适配器模式（Adapter）
- 代理模式（Proxy）
- 装饰模式（Decorator）
- 桥梁模式/桥接模式（Bridge）
- 组合模式（Composite）
- 享元模式（Flyweight）

### 3．行为型模式

行为型模式涉及到算法和对象间职责的分配，行为模式描述了对象和类的模式，以及它们之间的通信模式，行为模式刻划了在程序运行时难以跟踪的复杂的控制流可分为行为类模式和行为对象模式。1. 行为类模式使用继承机制在类间分派行为。2. 行为对象模式使用对象聚合来分配行为。一些行为对象模式描述了一组对等的对象怎样相互协作以完成其中任何一个对象都无法单独完成的任务。

在对象的结构和对象的创建问题都解决了之后，就剩下对象的行为问题了，如果对象的行为设计的好，那么对象的行为就会更清晰，它们之间的协作效率就会提高，这里有11个具体的行为型模式可供研究，它们分别是：

- 模板方法模式（Template Method）
- 观察者模式（Observer）
- 状态模式（State）
- 策略模式（Strategy）
- 职责链模式（Chain of Responsibility）
- 命令模式（Command）
- 访问者模式（Visitor）
- 调停者模式（Mediator）
- 备忘录模式（Memento）
- 迭代器模式（Iterator）
- 解释器模式（Interpreter）

### 三者之间的区别和联系

> 创建型模式提供生存环境，结构型模式提供生存理由，行为型模式提供如何生存。

1. 创建型模式为其他两种模式使用提供了环境。
2. 结构型模式侧重于接口的使用，它做的一切工作都是对象或是类之间的交互，提供一个门。
3. 行为型模式顾名思义，侧重于具体行为，所以概念中才会出现职责分配和算法通信等内容。

| **的**   | **创建型模式Creational Pattern**                             | **结构型模式Structural Patterns**                            | **行为型模式Behavioral Pattern**                             |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **概念** | 创建型模式，就是创建对象的模式，抽象了实例化的过程。它帮助一个系统独立于如何创建、组合和表示它的那些对象。关注的是对象的创建，创建型模式将创建对象的过程进行了抽象，也可以理解为将创建对象的过程进行了封装，作为客户程序仅仅需要去使用对象，而不再关心创建对象过程中的逻辑 | 结构型模式是为解决怎样组装现有的类，设计他们的交互方式，从而达到实现一定的功能的目的。结构型模式包容了对很多问题的解决。例如：扩展性（外观、组成、代理、装饰）封装性（适配器，桥接） | 行为型模式涉及到算法和对象间职责的分配，行为模式描述了对象和类的模式，以及它们之间的通信模式，行为型模式刻划了在程序运行时难以跟踪的复杂的控制流可分为行为类模式和行为对象模式1.行为模式使用继承机制在类间分派行为2.行为对象模式使用对象聚合来分配行为。一些行为对象模式描述了一组对等的对象怎样相互协作以完成其中任何一个对象都无法单独完成的任务。 |
| **类**   | Factory Method                                               | Adapter(类)                                                  | Interpreter                                                  |
|          |                                                              |                                                              | Template Method                                              |
| **对象** | Abstract Factory                                             | Adapter(对象)                                                | Chain of Responsibility                                      |
|          | Builder                                                      | Bridge                                                       | Command                                                      |
|          | Prototype                                                    | Composite                                                    | Iterator                                                     |
|          | Singleton                                                    | Decorator                                                    | Mediator                                                     |
|          |                                                              | Facade                                                       | Memento                                                      |
|          |                                                              | Flyweight                                                    | Observer                                                     |
|          |                                                              | Proxy                                                        | State                                                        |
|          |                                                              |                                                              | Strategy                                                     |
|          |                                                              |                                                              | Visitor                                                      |

## 设计原则

1. 开闭原则： 对扩展开放，对修改关闭
2. 里氏转换原则： 子类继承父类，单独完全可以运行
3. 依赖倒转原则： 引用一个对象，如果这个对象有底层类型，直接引用底层类型
4. 接口隔离原则： 每一个接口应该是一种角色
5. 合成/聚合复用原则： 新的对象应使用一些已有的对象，使之成为新对象的一部分
6. 迪米特原则： 一个对象应对其他对象有尽可能少的了解



## Android 源码中设计模式

下面我们简要的介绍一下设计模式在 Android 源码中的应用
工厂方法模式： 接口 Iterable 中的 iterator 就是一个工厂方法， Activity 中的 onCreate 也可以看作是工厂方法
抽象工厂模式： MediaPlayerFactory 是一个抽象工厂
单例模式： 单例模式的应用非常广泛， Application 就是单例的， WindowManagerService 、 ActivityService 等系统级的 Service 也是单例的，具体可见 [单例模式](http://weiqianghu.github.io/2016/09/08/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F/)
建造者模式 ：建造者模式在图片加载库中使用非常普遍，例如 Picasso 、 Glide 、UniversalImageLoader 中都有建造者模式的身影。在 Android 源码中 AlertDialog 中应用了建造者模式，详见 [设计模式之 Builder 模式](http://weiqianghu.github.io/2016/09/13/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B-Builder-%E6%A8%A1%E5%BC%8F/)
原型模式：原型模式就是实现接口 Cloneable ，这个太多了。。。
适配器模式：这个不用说了， AbsListView 和 RecyclerView 都使用了适配器模式。详见：[设计模式之适配器模式](http://weiqianghu.github.io/2016/10/10/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F/)
装饰器模式：Context 、 ContextImpl 、 ContextWrapper 、ContextThemeWrapper
代理模式： AIDL ，ActivityProxy（其实这也是 AIDL ）
外观模式：Context 作为 Android 系统中的上帝类，封装了很多功能，这也是外观模式的应用
桥接模式：Adapter 与 AdapterView 的桥接，Window 与 WindowManager 的桥接 详见：[设计模式之桥接模式](http://weiqianghu.github.io/2016/10/13/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E6%A1%A5%E6%8E%A5%E6%A8%A1%E5%BC%8F/)
组合模式： View 、 ViewGroup
享元模式：Message 中的 MessagePool 是用链表实现的。
策略模式：动画中的 InterPolator 和 TypeEvaluator 。详见[Android 动画分析](http://weiqianghu.github.io/2016/08/23/Android-%E5%8A%A8%E7%94%BB%E5%88%86%E6%9E%90/)
模板方法模式: 模板模式的应用非常广泛， Android 中 AsyncTask 的几个回调可以看作模板。
观察者模式： AbsListView 和 RecyclerView 都使用了观察者模式，详见[设计模式之观察者模式](http://weiqianghu.github.io/2016/09/27/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F/)
迭代子模式： 这个不用说了，在 JDK 中的集合类都是迭代子模式。
责任链模式： View 中对于事件的分发处理可以看作是责任链模式。详见[利用责任链模式实现加载不同来源的数据](http://weiqianghu.github.io/2016/07/14/%E5%88%A9%E7%94%A8%E8%B4%A3%E4%BB%BB%E9%93%BE%E6%A8%A1%E5%BC%8F%E5%AE%9E%E7%8E%B0%E5%8A%A0%E8%BD%BD%E4%B8%8D%E5%90%8C%E6%9D%A5%E6%BA%90%E7%9A%84%E6%95%B0%E6%8D%AE/)
命令模式： Android 事件的底层 NotifyArgs 就是一个命令对象。
备忘录模式： Android 的状态保存， onSaveInstanceState 、 onRestoreInstaceState
状态模式： WIFI 管理
访问者模式： Java 注解 APT
中介者模式： KeyGuard 功能的实现
解释器模式： PackageParser 中有解释器模式的影子



## Java设计模式:JDK中的应用

### 创建型模式

抽象工厂模式(通过创建的方法返回工厂本身, 可以依次创建另一个抽象/接口类型)
javax.xml.parsers.DocumentBuilderFactory#newInstance()
javax.xml.transform.TransformerFactory#newInstance()
javax.xml.xpath.XPathFactory#newInstance()
生成器模式(通过创建的方法返回实例本身)
java.lang.StringBuilder#append() (非同步的)
java.lang.StringBuffer#append() (支持同步)
java.nio.ByteBuffer#put() (与之相同的有: CharBuffer, ShortBuffer, IntBuffer, LongBuffer, FloatBuffer 和 DoubleBuffer)
javax.swing.GroupLayout.Group#addComponent()
java.lang.Appendabled 的所有实现
工厂模式(通过创建的方法返回一个抽象/接口类型的实现)
java.util.Calendar#getInstance()
java.util.ResourceBundle#getBundle()
java.text.NumberFormat#getInstance()
java.nio.charset.Charset#forName()
java.net.URLStreamHandlerFactory#createURLStreamHandler(String) (按协议返回单例对象)
原型模式(通过创建的方法返回一个包含相同属性的不同实例)
java.lang.Object#clone() (类必须实现 java.lang.Cloneable)
单例模式(通过创建的方法在任何时间都返回一个相同实例(通常是它本身))
java.lang.Runtime#getRuntime()
java.awt.Desktop#getDesktop()

### 结构型模式

适配器模式(接收不同抽象或接口类型的一个实例, 并返回一个装饰或重写了该实例的本身/其他的抽象/接口类型的指定实现)
java.util.Arrays#asList()
java.io.InputStreamReader(InputStream) (返回一个Reader)
java.io.OutputStreamWriter(OutputStream) (返回一个Writer)
javax.xml.bind.annotation.adapters.XmlAdapter#marshal() 和 #unmarshal()
桥接模式(接收不同抽象或接口类型的一个实例, 并返回一个委托或使用了该实例的本身/其他的抽象/接口类型的指定实现)
一时之间没有想到. 一个虚构的例子: new LinkedHashMap(LinkedHashSet<K>, List<V>)返回一个不可变的链式map, 并且不会克隆内部元素而是直接使用他们. java.util.Collections#newSetFromMap() 和 singletonXXX()方法是有些接近这个模式的.
组合模式(通过行为式的方法接收一个相同的抽象或接口类型的实例转换为一个树状结构)
java.awt.Container#add(Component) (几乎所有Swing都是这样)
javax.faces.component.UIComponent#getChildren() (几乎所有JSF UI都是这样)
装饰器模式(通过创建的方法接收一个相同的抽象或接口类型的实例, 并添加附加的行为)
java.io.InputStream, OutputStream, Reader 和 Writer的子类拥有一个持有相同类型的实例构造器.
java.util.Collections, checkedXXX(), synchronizedXXX() 和 unmodifiableXXX().
javax.servlet.http.HttpServletRequestWrapper 和 HttpServletResponseWrapper
门面模式(通过行为式的方法实现, 该方法内部使用不同的独立的抽象或接口类型的实例)
javax.faces.context.FacesContext, 它内部之间使用 LifeCycle, ViewHandler, NavigationHandler 和很多无需最终能够用户关心的抽象/接口类型(可通过注入覆写).
javax.faces.context.ExternalContext, 它内部使用ServletContext, HttpSession, HttpServletRequest, HttpServletResponse等.
享元模式(通过创建的方法返回一个缓存的实例, 有一些”多例”的思想)
java.lang.Integer#valueOf(int) (类似的还包括: Boolean, Byte, Character, Short 和 Long)
代理模式(通过创建的方法返回指定的抽象或接口类型的实现, 该实现依次委托或使用了指定抽象或接口类型的不同实现)
java.lang.reflect.Proxy
java.rmi.*下所有API.

### 行为型模式

责任链模式(通过行为方法(间接地)在一个队列中调用另一个相同抽象/接口类型的相同方法)
java.util.logging.Logger#log()
javax.servlet.Filter#doFilter()
命令模式(一个抽象/接口类型内的行为方法中, 调用一个不同的象/接口类型的实现的方法, 该实现已经在其创建过程中被命令实现封装)
java.lang.Runnable的所有实现
javax.swing.Action的所有实现
解释器模式(行为方法返回一个有结构的并且与被提供的实例/类型不同的实例/类型; 注意parsing/formatting不是这个模式的组成部分, 而是用来确定该模式以及如何应用模式)
java.util.Pattern
java.text.Normalizer
java.text.Format的所有子类
javax.el.ELResolver的所有子类
迭代器模式(行为方法按顺序的自一个队列中返回不同类型的实例)
java.util.Iterator的所有实现 (除此以外, 还有 java.util.Scanner!).
java.util.Enumeration的所有实现
中介者模式(行为方法接收一个不同抽象/接口类型的实例(通常使用命令模式), 该实例委托或使用了指定实例)
java.util.Timer (所有 scheduleXXX() 方法)
java.util.concurrent.Executor#execute()
java.util.concurrent.ExecutorService (invokeXXX() 和 submit() 方法)
java.util.concurrent.ScheduledExecutorService (所有 scheduleXXX() 方法)
java.lang.reflect.Method#invoke()
备忘录模式(行为方法内部改变了整个实例的状态)
java.util.Date ( setter 方法就是这么做的, Date 内部通过长整型long表示)
java.io.Serializable的所有实现
javax.faces.component.StateHolder的所有实现
观察者或发布/订阅模式(行为方法在另一个抽象/接口类型的实例上调用方法, 并取决于自身状态)
java.util.Observer/java.util.Observable (在现实世界中很少使用)
java.util.EventListener 的所有实现 (几乎所有的Swing都是这样)
javax.servlet.http.HttpSessionBindingListener
javax.servlet.http.HttpSessionAttributeListener
javax.faces.event.PhaseListener
状态模式(行为方法通过外部可控的实例状态改变它的行为)
javax.faces.lifecycle.LifeCycle#execute() (通过FacesServlet控制, 行为取决于当前JSF生命周期的阶段(状态)
策略模式(行为方法内一个抽象/接口类型调用不同抽象/接口类型的不同实现内的方法, 该实现以方法参数型式被传进策略实现内)
java.util.Comparator#compare(), 在Collections#sort()中执行.
javax.servlet.http.HttpServlet, service() 和所有 doXXX() 方法接收 HttpServletRequest, HttpServletResponse, 并且实现者必须处理它们(而不是作为实例变量持有它们)
javax.servlet.Filter#doFilter()
模板模式(行为方法已拥有一个被抽象类型定义的”默认”行为)
java.io.InputStream, java.io.OutputStream, java.io.Reader 和 java.io.Writer 的所有非抽象方法.
java.util.AbstractList, java.util.AbstractSet 和 java.util.AbstractMap 的所有非抽象方法.
javax.servlet.http.HttpServlet, 所有 doXXX() 方法默认发送 HTTP 405 "Method Not Allowed" 错误到响应中. 你可以任意实现这些方法.
访问者模式(两个不同的抽象/接口类型中包含互相接收对方抽象/接口类型的方法; 实际调用的是另一个类型的方法和它上面的其他执行所需求的策略)
javax.lang.model.element.AnnotationValue 和 AnnotationValueVisitor
javax.lang.model.element.Element 和 ElementVisitor
javax.lang.model.type.TypeMirror 和 TypeVisitor



## 参考

UML： https://zhuanlan.zhihu.com/p/24576502

精简uml图：

http://jayfeng.com/2016/04/02/%E7%90%86%E8%A7%A3%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E5%88%86%E7%B1%BB%E3%80%81%E6%84%8F%E5%9B%BE%E3%80%81UML%E7%B1%BB%E5%9B%BE/

https://blog.csdn.net/sfdev/article/details/2845488

设计模式：

http://linbinghe.com/2017/1646c9f3.html

https://github.com/suzeyu1992/repo/tree/master/project/design-pattern

https://github.com/jeanboydev/Android-ReadTheFuckingSourceCode/tree/master/design_patterns

https://www.kancloud.cn/longxuan/my-designpattern/117488

https://github.com/jiayisheji/blog/issues/2



https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/README.md

https://www.cnblogs.com/happykoukou/p/5382649.html

https://pgzxc.github.io/2017/12/24/Android%E4%B8%AD%E7%9A%8423%E7%A7%8D%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/

https://github.com/guoxiaoxing/android-open-source-project-analysis/blob/master/doc/Android%E7%B3%BB%E7%BB%9F%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1%E7%AF%87/02Android%E7%B3%BB%E7%BB%9F%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1%E7%AF%87%EF%BC%9A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.md

https://github.com/quanke/design-pattern-java

https://github.com/simple-android-framework/android_design_patterns_analysis/blob/master/README.md

https://clarkdo.js.org/design%20patterns%20stories/2014/10/20/46/