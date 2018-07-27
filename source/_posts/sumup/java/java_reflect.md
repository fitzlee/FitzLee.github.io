---
title: java reflect反射注解动态代理原理
urlname: java_jdk_base_reflect_proxy
#date: 2016-03-06
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 00Java&JDK
tags:
    - java
    - jdk
    - reflect
    - proxy
    - anotation
---

# JAVA反射

> 主要是指程序可以访问，检测和修改它本身状态或行为的一种能力，并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义。

## 反射机制是什么

> 面试有可能会问到，这句话不管你能不能理解，但是你只要记住就可以了

反射机制就是在运行状态中，**对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性**；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

用一句话总结就是反射可以实现在**运行**时可以知道**任意一个类**的**属性和方法**。

## 反射机制能做什么

反射机制主要提供了以下功能：

- 在运行时判断任意一个对象所属的类；
- 在运行时构造任意一个类的对象；
- 在运行时判断任意一个类所具有的成员变量和方法；
- 在运行时调用任意一个对象的方法；
- 生成[**动态代理**](https://www.daidingkang.cc/2017/07/18/Java%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F-%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F/)(ps:这个知识点也很重要，后续会为大家讲到)

## Java 反射机制的应用场景

- 逆向代码 ，例如反编译
- 与注解相结合的框架 例如Retrofit
- 单纯的反射机制应用框架 例如EventBus
- 动态生成类框架 例如Gson

## 反射机制的优点与缺点

为什么要用反射机制？直接创建对象不就可以了吗，这就涉及到了动态与静态的概念

- 静态编译：在编译时确定类型，绑定对象,即通过。
- 动态编译：运行时确定类型，绑定对象。动态编译最大限度发挥了java的灵活性，体现了多态的应用，有以降低类之间的藕合性。

优点

- 可以实现动态创建对象和编译，体现出很大的灵活性，特别是在J2EE的开发中它的灵活性就表现的十分明显。比如，一个大型的软件，不可能一次就把把它设计的很完美，当这个程序编译后，发布了，当发现需要更新某些功能时，我们不可能要用户把以前的卸载，再重新安装新的版本，假如这样的话，这个软件肯定是没有多少人用的。采用静态的话，需要把整个程序重新编译一次才可以实现功能的更新，而采用反射机制的话，它就可以不用卸载，只需要在运行时才动态的创建和编译，就可以实现该功能。

缺点

- 对性能有影响。使用反射基本上是一种解释操作，我们可以告诉JVM，我们希望做什么并且它满足我们的要求。这类操作总是慢于只直接执行相同的操作。

## 理解Class类和类类型

想要了解反射首先理解一下Class类，它是反射实现的基础。
类是java.lang.Class类的实例对象，而Class是所有类的类（There is a class named Class）
对于普通的对象，我们一般都会这样创建和表示：

```
Code code1 = new Code();
```

上面说了，所有的类都是Class的对象，那么如何表示呢，可不可以通过如下方式呢：

```
Class c = new Class();
```

但是我们查看Class的源码时，是这样写的：

```
private  Class(ClassLoader loader) { 
    classLoader = loader; 
}
```

可以看到构造器是私有的，只有JVM可以创建Class的对象，因此不可以像普通类一样new一个Class对象，虽然我们不能new一个Class对象，但是却可以通过已有的类得到一个Class对象，共有三种方式，如下：

```
Class c1 = Code.class; 这说明任何一个类都有一个隐含的静态成员变量class，这种方式是通过获取类的静态成员变量class得到的
Class c2 = code1.getClass(); code1是Code的一个对象，这种方式是通过一个类的对象的getClass()方法获得的 
Class c3 = Class.forName("com.trigl.reflect.Code"); 这种方法是Class类调用forName方法，通过一个类的全量限定名获得
```

这里，c1、c2、c3都是Class的对象，他们是完全一样的，而且有个学名，叫做Code的类类型（class type）。
这里就让人奇怪了，前面不是说Code是Class的对象吗，而c1、c2、c3也是Class的对象，那么Code和c1、c2、c3不就一样了吗？为什么还叫Code什么类类型？这里不要纠结于它们是否相同，只要理解类类型是干什么的就好了，顾名思义，类类型就是类的类型，也就是描述一个类是什么，都有哪些东西，所以我们可以通过类类型知道一个类的属性和方法，并且可以调用一个类的属性和方法，这就是反射的基础。

举个简单例子代码：

```
public class ReflectDemo {
    public static void main(String[] args) throws ClassNotFoundException {
        //第一种：Class c1 = Code.class;
        Class class1=ReflectDemo.class;
        System.out.println(class1.getName());

        //第二种：Class c2 = code1.getClass();
        ReflectDemo demo2= new ReflectDemo();
        Class c2 = demo2.getClass();
        System.out.println(c2.getName());

        //第三种：Class c3 = Class.forName("com.trigl.reflect.Code");
        Class class3 = Class.forName("com.tengj.reflect.ReflectDemo");
        System.out.println(class3.getName());
    }
}
```

执行结果：

```
com.tengj.reflect.ReflectDemo
com.tengj.reflect.ReflectDemo
com.tengj.reflect.ReflectDemo
```

# Java反射相关操作

> 在这里先看一下sun为我们提供了那些反射机制中的类：
> java.lang.Class;
> java.lang.reflect.Constructor; java.lang.reflect.Field;
> java.lang.reflect.Method;
> java.lang.reflect.Modifier;

前面我们知道了怎么获取Class，那么我们可以通过这个Class干什么呢？
总结如下：

- 获取成员方法Method
- 获取成员变量Field
- 获取构造函数Constructor

下面来具体介绍

1. ## 获取成员方法信息

   > **两个参数分别是方法名和方法参数类的类类型列表。**

```
public Method getDeclaredMethod(String name, Class<?>... parameterTypes) // 得到该类所有的方法，不包括父类的 
public Method getMethod(String name, Class<?>... parameterTypes) // 得到该类所有的public方法，包括父类的

//具体使用
Method[] methods = class1.getDeclaredMethods();//获取class对象的所有声明方法 
Method[] allMethods = class1.getMethods();//获取class对象的所有public方法 包括父类的方法 
Method method = class1.getMethod("info", String.class);//返回次Class对象对应类的、带指定形参列表的public方法 
Method declaredMethod = class1.getDeclaredMethod("info", String.class);//返回次Class对象对应类的、带指定形参列表的方法
```

**举个例子：**

例如类A有如下一个方法：

```
public void fun(String name,int age) {
        System.out.println("我叫"+name+",今年"+age+"岁");
    }
```

现在知道A有一个对象a，那么就可以通过：

```
Class c = Class.forName("com.tengj.reflect.Person");  //先生成class
Object o = c.newInstance();                           //newInstance可以初始化一个实例
Method method = c.getMethod("fun", String.class, int.class);//获取方法
method.invoke(o, "tengj", 10);                         //通过invoke调用该方法，参数第一个为实例对象，后面为具体参数值
```

完整代码如下：

```
public class Person {
    private String name;
    private int age;
    private String msg="hello wrold";
 public String getName() {
        return name;
  }

    public void setName(String name) {
        this.name = name;
  }

    public int getAge() {
        return age;
  }

    public void setAge(int age) {
        this.age = age;
  }

    public Person() {
    }

    private Person(String name) {
        this.name = name;
  System.out.println(name);
  }

    public void fun() {
        System.out.println("fun");
  }

    public void fun(String name,int age) {
        System.out.println("我叫"+name+",今年"+age+"岁");
  }
}

public class ReflectDemo {
    public static void main(String[] args){
        try {
            Class c = Class.forName("com.tengj.reflect.Person");
            Object o = c.newInstance();
            Method method = c.getMethod("fun", String.class, int.class);
            method.invoke(o, "tengj", 10);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

执行结果：

```
我叫tengj,今年10岁
```

怎样，是不是感觉很厉害，我们只要知道这个类的路径全称就能玩弄它于鼓掌之间。

有时候我们想获取类中所有成员方法的信息，要怎么办。可以通过以下几步来实现：

1.获取所有方法的数组：

```
Class c = Class.forName("com.tengj.reflect.Person");
Method[] methods = c.getDeclaredMethods(); // 得到该类所有的方法，不包括父类的
或者：
Method[] methods = c.getMethods();// 得到该类所有的public方法，包括父类的
```

2.然后循环这个数组就得到每个方法了：

```
for (Method method : methods)
```

完整代码如下：
person类跟上面一样，这里以及后面就不贴出来了，只贴关键代码

```
public class ReflectDemo {
    public static void main(String[] args){
        try {
            Class c = Class.forName("com.tengj.reflect.Person");
            Method[] methods = c.getDeclaredMethods();
            for(Method m:methods){
                String  methodName= m.getName();
                System.out.println(methodName);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

执行结果：

```
getName
setName
setAge
fun
fun
getAge
```

这里如果把c.getDeclaredMethods();改成c.getMethods();执行结果如下，多了很多方法，以为把Object里面的方法也打印出来了，因为Object是所有类的父类：

```
getName
setName
getAge
setAge
fun
fun
wait
wait
wait
equals
toString
hashCode
getClass
notify
notifyAll
```

1. ## 获取成员变量信息

想一想成员变量中都包括什么：**成员变量类型+成员变量名**

类的成员变量也是一个对象，它是`java.lang.reflect.Field`的一个对象，所以我们通过`java.lang.reflect.Field`里面封装的方法来获取这些信息。

单独获取某个成员变量，通过Class类的以下方法实现：

> **参数是成员变量的名字**

```
public Field getDeclaredField(String name) // 获得该类自身声明的所有变量，不包括其父类的变量
public Field getField(String name) // 获得该类自所有的public成员变量，包括其父类变量

//具体实现
Field[] allFields = class1.getDeclaredFields();//获取class对象的所有属性 
Field[] publicFields = class1.getFields();//获取class对象的public属性 
Field ageField = class1.getDeclaredField("age");//获取class指定属性 
Field desField = class1.getField("des");//获取class指定的public属性
```

**举个例子：**

例如一个类A有如下成员变量：

```
private int n;
```

如果A有一个对象a，那么就可以这样得到其成员变量：

```
Class c = a.getClass();
Field field = c.getDeclaredField("n");
```

完整代码如下：

```
public class ReflectDemo {
    public static void main(String[] args){
        try {
            Class c = Class.forName("com.tengj.reflect.Person");
            //获取成员变量
            Field field = c.getDeclaredField("msg"); //因为msg变量是private的，所以不能用getField方法
            Object o = c.newInstance();
            field.setAccessible(true);//设置是否允许访问，因为该变量是private的，所以要手动设置允许访问，如果msg是public的就不需要这行了。
            Object msg = field.get(o);
            System.out.println(msg);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

执行结果：

```
hello wrold
```

同样，如果想要获取所有成员变量的信息，可以通过以下几步

1.获取所有成员变量的数组：

```
Field[] fields = c.getDeclaredFields();
```

2.遍历变量数组，获得某个成员变量field

```
for (Field field : fields)
```

完整代码：

```
public class ReflectDemo {
    public static void main(String[] args){
        try {
            Class c = Class.forName("com.tengj.reflect.Person");
            Field[] fields = c.getDeclaredFields();
            for(Field field :fields){
                System.out.println(field.getName());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

执行结果：

```
name
age
msg
```

1. ## 获取构造函数

最后再想一想构造函数中都包括什么：`构造函数参数`
同上，类的成构造函数也是一个对象，它是`java.lang.reflect.Constructor`的一个对象，所以我们通过`java.lang.reflect.Constructor`里面封装的方法来获取这些信息。

单独获取某个构造函数,通过`Class`类的以下方法实现：

> **这个参数为构造函数参数类的类类型列表**

```
public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes) //  获得该类所有的构造器，不包括其父类的构造器
public Constructor<T> getConstructor(Class<?>... parameterTypes) // 获得该类所以public构造器，包括父类

//具体
Constructor<?>[] allConstructors = class1.getDeclaredConstructors();//获取class对象的所有声明构造函数 
Constructor<?>[] publicConstructors = class1.getConstructors();//获取class对象public构造函数 
Constructor<?> constructor = class1.getDeclaredConstructor(String.class);//获取指定声明构造函数 
Constructor publicConstructor = class1.getConstructor(String.class);//获取指定声明的public构造函数
```

**举个例子：**

例如类A有如下一个构造函数：

```
public A(String a, int b) {
    // code body
}
```

那么就可以通过：

```
Constructor constructor = a.getDeclaredConstructor(String.class, int.class);
```

来获取这个构造函数。

完整代码：

```
public class ReflectDemo {
    public static void main(String[] args){
        try {
            Class c = Class.forName("com.tengj.reflect.Person");
            //获取构造函数
            Constructor constructor = c.getDeclaredConstructor(String.class);
            constructor.setAccessible(true);//设置是否允许访问，因为该构造器是private的，所以要手动设置允许访问，如果构造器是public的就不需要这行了。
            constructor.newInstance("tengj");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

执行结果：

```
tengj
```

注意：Class的newInstance方法，只能创建只包含无参数的构造函数的类，如果某类只有带参数的构造函数，那么就要使用另外一种方式：

```
fromClass.getDeclaredConstructor(String.class).newInstance("tengj");
```

获取所有的构造函数，可以通过以下步骤实现：

1.获取该类的所有构造函数，放在一个数组中：

```
Constructor[] constructors = c.getDeclaredConstructors();
```

2.遍历构造函数数组，获得某个构造函数`constructor`:

```
for (Constructor constructor : constructors)
```

完整代码：

```
public class ReflectDemo {
    public static void main(String[] args){
            Constructor[] constructors = c.getDeclaredConstructors();
            for(Constructor constructor:constructors){
                System.out.println(constructor);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

执行结果：

```
public com.tengj.reflect.Person()
public com.tengj.reflect.Person(java.lang.String)
```

1. ## 其他方法

注解需要用到的

```
Annotation[] annotations = (Annotation[]) class1.getAnnotations();//获取class对象的所有注解 
Annotation annotation = (Annotation) class1.getAnnotation(Deprecated.class);//获取class对象指定注解 
Type genericSuperclass = class1.getGenericSuperclass();//获取class对象的直接超类的 
Type Type[] interfaceTypes = class1.getGenericInterfaces();//获取class对象的所有接口的type集合
```

获取class对象的信息

```
boolean isPrimitive = class1.isPrimitive();//判断是否是基础类型 
boolean isArray = class1.isArray();//判断是否是集合类
 boolean isAnnotation = class1.isAnnotation();//判断是否是注解类 
boolean isInterface = class1.isInterface();//判断是否是接口类 
boolean isEnum = class1.isEnum();//判断是否是枚举类 
boolean isAnonymousClass = class1.isAnonymousClass();//判断是否是匿名内部类 
boolean isAnnotationPresent = class1.isAnnotationPresent(Deprecated.class);//判断是否被某个注解类修饰 
String className = class1.getName();//获取class名字 包含包名路径 
Package aPackage = class1.getPackage();//获取class的包信息 
String simpleName = class1.getSimpleName();//获取class类名 
int modifiers = class1.getModifiers();//获取class访问权限 
Class<?>[] declaredClasses = class1.getDeclaredClasses();//内部类 
Class<?> declaringClass = class1.getDeclaringClass();//外部类

getSuperclass()：获取某类的父类  
getInterfaces()：获取某类实现的接口
```

## 通过反射了解集合泛型的本质

> 扩展的知识点，了解就可以了。后续会为大家写一篇关于泛型的文章。

首先下结论：

> **Java中集合的泛型，是防止错误输入的，只在编译阶段有效，绕过编译到了运行期就无效了。**

下面通过一个实例来验证：

```
/**
 * 集合泛型的本质
 */
public class GenericEssence {
    public static void main(String[] args) {
        List list1 = new ArrayList(); // 没有泛型 
        List<String> list2 = new ArrayList<String>(); // 有泛型


        /*
         * 1.首先观察正常添加元素方式，在编译器检查泛型，
         * 这个时候如果list2添加int类型会报错
         */
        list2.add("hello");
//      list2.add(20); // 报错！list2有泛型限制，只能添加String，添加int报错
        System.out.println("list2的长度是：" + list2.size()); // 此时list2长度为1


        /*
         * 2.然后通过反射添加元素方式，在运行期动态加载类，首先得到list1和list2
         * 的类类型相同，然后再通过方法反射绕过编译器来调用add方法，看能否插入int
         * 型的元素
         */
        Class c1 = list1.getClass();
        Class c2 = list2.getClass();
        System.out.println(c1 == c2); // 结果：true，说明类类型完全相同

        // 验证：我们可以通过方法的反射来给list2添加元素，这样可以绕过编译检查
        try {
            Method m = c2.getMethod("add", Object.class); // 通过方法反射得到add方法
            m.invoke(list2, 20); // 给list2添加一个int型的，上面显示在编译器是会报错的
            System.out.println("list2的长度是：" + list2.size()); // 结果：2，说明list2长度增加了，并没有泛型检查
        } catch (Exception e) {
            e.printStackTrace();
        }

        /*
         * 综上可以看出，在编译器的时候，泛型会限制集合内元素类型保持一致，但是编译器结束进入
         * 运行期以后，泛型就不再起作用了，即使是不同类型的元素也可以插入集合。
         */
    }
}
```

执行结果：

```
list2的长度是：1
true
list2的长度是：2
```

## 思维导图

> 有助于理解上述所讲的知识点

[![img](http://upload-images.jianshu.io/upload_images/1637925-9ce9f69f8b7c26e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://upload-images.jianshu.io/upload_images/1637925-9ce9f69f8b7c26e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

拓展阅读
[Java反射机制深入详解 - 火星十一郎 - 博客园](http://www.cnblogs.com/hxsyl/archive/2013/03/23/2977593.html)
[Java反射入门 - Trigl的博客 - CSDN博客](http://blog.csdn.net/trigl/article/details/51042403)
[Java反射机制 - ①块腹肌 - 博客园](http://www.cnblogs.com/jqyp/archive/2012/03/29/2423112.html)
[Java 反射机制浅析 - 孤旅者 - 博客园](http://www.cnblogs.com/gulvzhe/archive/2012/01/27/2330001.html)
[反射机制的理解及其用途 - 每天进步一点点！ - ITeye博客](http://uule.iteye.com/blog/1423512)
[Java动态代理与反射详解 - 浩大王 - 博客园](http://www.cnblogs.com/haodawang/p/5967185.html)

# JAVA注解

## 概念及作用

1. 概念

- 注解即元数据,就是源代码的元数据
- 注解在代码中添加信息提供了一种形式化的方法,可以在后续中更方便的 使用这些数据
- Annotation是一种应用于类、方法、参数、变量、构造器及包声明中的特殊修饰符。它是一种由JSR-175标准选择用来描述元数据的一种工具。

1. 作用

- 生成文档
- 跟踪代码依赖性，实现替代配置文件功能,减少配置。如Spring中的一些注解
- 在编译时进行格式检查，如@Override等
- 每当你创建描述符性质的类或者接口时,一旦其中包含重复性的工作，就可以考虑使用注解来简化与自动化该过程。

## 什么是java注解？

在java语法中，使用`@`符号作为开头，并在@后面紧跟注解名。被运用于类，接口，方法和字段之上，例如：

```
@Override
void myMethod() { 
......
}
```

这其中@Override就是注解。这个注解的作用也就是告诉编译器，myMethod()方法覆写了父类中的myMethod()方法。

## java中内置的注解

java中有三个内置的注解：

- **@Override:**表示当前的方法定义将覆盖超类中的方法，如果出现错误，编译器就会报错。

- **@Deprecated:**如果使用此注解，编译器会出现警告信息。

- **@SuppressWarnings:**忽略编译器的警告信息。

本文不在阐述三种内置注解的使用情节和方法，感兴趣的请看[这里](http://www.jianshu.com/p/89c07ce0c99c)

## 元注解

> 自定义注解的时候用到的，也就是自定义**注解的注解**；（这句话我自己说的，不知道对不对）

元注解的作用就是负责注解其他注解。`Java5.0`定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。

### `Java5.0`定义的4个元注解：

1. @Target
2. @Retention
3. @Documented
4. @Inherited

> java8加了两个新注解，后续我会讲到。

这些类型和它们所支持的类在java.lang.annotation包中可以找到。

### @Target

> @Target说明了Annotation所修饰的对象范围：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。

**作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）**

**取值(ElementType)有：**

| 类型           | 用途                                      |
| -------------- | ----------------------------------------- |
| CONSTRUCTOR    | 用于描述构造器                            |
| FIELD          | 用于描述域                                |
| LOCAL_VARIABLE | 用于描述局部变量                          |
| METHOD         | 用于描述方法                              |
| PACKAGE        | 用于描述包                                |
| PARAMETER      | 用于描述参数                              |
| TYPE           | 用于描述类、接口(包括注解类型) 或enum声明 |

比如说这个注解表示只能在方法中使用：

```
@Target({ElementType.METHOD})
public @interface MyCustomAnnotation {

}

//使用
public class MyClass {
   @MyCustomAnnotation
   public void myMethod()
   {
    ......
   }
}
```

### @Retention

> @Retention定义了该Annotation被保留的时间长短：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对 Annotation的“生命周期”限制。

**作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）**

**取值（RetentionPoicy）有：**

| 类型    | 用途                             | 说明                             |
| ------- | -------------------------------- | -------------------------------- |
| SOURCE  | 在源文件中有效（即源文件保留）   | 仅出现在源代码中，而被编译器丢弃 |
| CLASS   | 在class文件中有效（即class保留） | 被编译在class文件中              |
| RUNTIME | 在运行时有效（即运行时保留）     | 编译在class文件中                |

使用示例：

```
/***
 * 字段注解接口
 */
@Target(value = {ElementType.FIELD})//注解可以被添加在属性上
@Retention(value = RetentionPolicy.RUNTIME)//注解保存在JVM运行时刻,能够在运行时刻通过反射API来获取到注解的信息
public @interface Column {
    String name();//注解的name属性
}
```

### @Documented

> @Documented用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

**作用：将注解包含在javadoc中**

示例：

```
java.lang.annotation.Documented
@Documented
public @interface MyCustomAnnotation { //Annotation body}
```

### @Inherited

- 是一个标记注解

- 阐述了某个被标注的类型是被继承的

- 使用了@Inherited修饰的annotation类型被用于一个class,则这个annotation将被用于该class的子类
  @Inherited annotation类型是被标注过的class的子类所继承。类并不从实现的接口继承annotation,方法不从它所重载的方法继承annotation

- 当@Inherited annotation类型标注的annotation的Retention是RetentionPolicy.RUNTIME，则反射API增强了这种继承性。如果我们使用java.lang.reflect去查询一个@Inherited annotation类型的annotation时，反射代码检查将展开工作：检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层。

**作用：允许子类继承父类中的注解**

示例，这里的MyParentClass 使用的注解标注了@Inherited，所以子类可以继承这个注解信息：

```
java.lang.annotation.Inherited
@Inherited
public @interface MyCustomAnnotation {
}
```

```
@MyCustomAnnotation
public class MyParentClass { 
  ... 
}
```

```
public class MyChildClass extends MyParentClass { 
   ... 
}
```

## 自定义注解

### 格式

```
public @interface 注解名{
  定义体
}
```

### 注解参数的可支持数据类型:

- 所有基本数据类型(int,float,double,boolean,byte,char,long,short)
- String 类型
- Class类型
- enum类型
- Annotation类型
- 以上所有类型的数组

### 规则

- 修饰符只能是public 或默认(default)
- 参数成员只能用基本类型byte,short,int,long,float,double,boolean八种基本类型和String,Enum,Class,annotations及这些类型的数组
- 如果只有一个参数成员,最好将名称设为”value”
- 注解元素必须有确定的值,可以在注解中定义默认值,也可以使用注解时指定,非基本类型的值不可为null,常使用空字符串或0作默认值
- 在表现一个元素存在或缺失的状态时,定义一下特殊值来表示,如空字符串或负值

### 示例:

```
/**
 * test注解
 * @author ddk
 *
 */ 
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface TestAnnotation {
    /**
     * id
     * @return
     */
    public int id() default -1;
    /**
     * name
     * @return
     */
    public String name() default "";
}
```

## 注解处理器类库

> java.lang.reflect.AnnotatedElement

Java使用Annotation接口来代表程序元素前面的注解，该接口是所有Annotation类型的父接口。除此之外，Java在java.lang.reflect 包下新增了AnnotatedElement接口，该接口代表程序中可以接受注解的程序元素，该接口主要有如下几个实现类：

- 　Class：类定义
- 　Constructor：构造器定义
- 　Field：累的成员变量定义
- 　Method：类的方法定义
- 　Package：类的包定义

java.lang.reflect 包下主要包含一些实现反射功能的工具类，实际上，java.lang.reflect 包所有提供的反射API扩充了读取运行时Annotation信息的能力。当一个Annotation类型被定义为运行时的Annotation后，该注解才能是运行时可见，当class文件被装载时被保存在class文件中的Annotation才会被虚拟机读取。

AnnotatedElement 接口是所有程序元素（Class、Method和Constructor）的父接口，所以程序通过反射获取了某个类的AnnotatedElement对象之后，程序就可以调用该对象的如下四个个方法来访问Annotation信息：

- 方法1： T getAnnotation(Class annotationClass): 返回改程序元素上存在的、指定类型的注解，如果该类型注解不存在，则返回null。
- 方法2：Annotation[] getAnnotations():返回该程序元素上存在的所有注解。
- 方法3：boolean is AnnotationPresent(Class<?extends Annotation> annotationClass):判断该程序元素上是否包含指定类型的注解，存在则返回true，否则返回false.
- 方法4：Annotation[] getDeclaredAnnotations()：返回直接存在于此元素上的所有注释。与此接口中的其他方法不同，该方法将忽略继承的注释。（如果没有注释直接存在于此元素上，则返回长度为零的一个数组。）该方法的调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响。

### 注解处理器示例:

```
/***********注解声明***************/

/**
 * 水果名称注解
 * @author peida
 *
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitName {
    String value() default "";
}

/**
 * 水果颜色注解
 * @author peida
 *
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitColor {
    /**
     * 颜色枚举
     * @author peida
     *
     */
    public enum Color{ BULE,RED,GREEN};
    
    /**
     * 颜色属性
     * @return
     */
    Color fruitColor() default Color.GREEN;

}

/**
 * 水果供应者注解
 * @author peida
 *
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitProvider {
    /**
     * 供应商编号
     * @return
     */
    public int id() default -1;
    
    /**
     * 供应商名称
     * @return
     */
    public String name() default "";
    
    /**
     * 供应商地址
     * @return
     */
    public String address() default "";
}

/***********注解使用***************/

public class Apple {
    
    @FruitName("Apple")
    private String appleName;
    
    @FruitColor(fruitColor=Color.RED)
    private String appleColor;
    
    @FruitProvider(id=1,name="陕西红富士集团",address="陕西省西安市延安路89号红富士大厦")
    private String appleProvider;
    
    public void setAppleColor(String appleColor) {
        this.appleColor = appleColor;
    }
    public String getAppleColor() {
        return appleColor;
    }
    
    public void setAppleName(String appleName) {
        this.appleName = appleName;
    }
    public String getAppleName() {
        return appleName;
    }
    
    public void setAppleProvider(String appleProvider) {
        this.appleProvider = appleProvider;
    }
    public String getAppleProvider() {
        return appleProvider;
    }
    
    public void displayName(){
        System.out.println("水果的名字是：苹果");
    }
}

/***********注解处理器***************/
//其实是用的反射


public class FruitInfoUtil {
    public static void getFruitInfo(Class<?> clazz){
        
        String strFruitName=" 水果名称：";
        String strFruitColor=" 水果颜色：";
        String strFruitProvicer="供应商信息：";
        
        Field[] fields = clazz.getDeclaredFields();
        
        for(Field field :fields){
            if(field.isAnnotationPresent(FruitName.class)){
                FruitName fruitName = (FruitName) field.getAnnotation(FruitName.class);
                strFruitName=strFruitName+fruitName.value();
                System.out.println(strFruitName);
            }
            else if(field.isAnnotationPresent(FruitColor.class)){
                FruitColor fruitColor= (FruitColor) field.getAnnotation(FruitColor.class);
                strFruitColor=strFruitColor+fruitColor.fruitColor().toString();
                System.out.println(strFruitColor);
            }
            else if(field.isAnnotationPresent(FruitProvider.class)){
                FruitProvider fruitProvider= (FruitProvider) field.getAnnotation(FruitProvider.class);
                strFruitProvicer=" 供应商编号："+fruitProvider.id()+" 供应商名称："+fruitProvider.name()+" 供应商地址："+fruitProvider.address();
                System.out.println(strFruitProvicer);
            }
        }
    }
}

/***********输出结果***************/
public class FruitRun {

    /**
     * @param args
     */
    public static void main(String[] args) {
        
        FruitInfoUtil.getFruitInfo(Apple.class);
        
    }

}

====================================
 水果名称：Apple
 水果颜色：RED
 供应商编号：1 供应商名称：陕西红富士集团 供应商地址：陕西省西安市延安路89号红富士大厦
```

## Java 8 中注解新特性

- @Repeatable 元注解,表示被修饰的注解可以用在同一个声明式或者类型加上多个相同的注解（包含不同的属性值）

- @Native 元注解,本地方法

- java8 中Annotation 可以被用在任何使用 Type 的地方

```
 //初始化对象时
String myString = new @NotNull String();
//对象类型转化时
myString = (@NonNull String) str;
//使用 implements 表达式时
class MyList<T> implements @ReadOnly List<@ReadOnly T>{
...
}
//使用 throws 表达式时
public void validateValues() throws @Critical ValidationFailedException{
...
}
```

## 思维导图

[![img](http://images.cnitblog.com/blog/400827/201409/161325133151441.jpg)](http://images.cnitblog.com/blog/400827/201409/161325133151441.jpg)



# 动态代理Java Proxy和CGLIB

动态代理在Java中有着广泛的应用，比如Spring AOP，Hibernate数据查询、测试框架的后端mock、RPC，Java注解对象获取等。静态代理的代理关系在编译时就确定了，而动态代理的代理关系是在编译期确定的。静态代理实现简单，适合于代理类较少且确定的情况，而动态代理则给我们提供了更大的灵活性。今天我们来探讨Java中两种常见的动态代理方式：**JDK原生动态代理和CGLIB动态代理**。

## JDK原生动态代理

先从直观的示例说起，假设我们有一个接口`Hello`和一个简单实现`HelloImp`：

```
// 接口
interface Hello{
    String sayHello(String str);
}
// 实现
class HelloImp implements Hello{
    @Override
    public String sayHello(String str) {
        return "HelloImp: " + str;
    }
}
```

这是Java种再常见不过的场景，使用接口制定协议，然后用不同的实现来实现具体行为。假设你已经拿到上述类库，如果我们想通过日志记录对`sayHello()`的调用，使用静态代理可以这样做：

```
// 静态代理方式
class StaticProxiedHello implements Hello{
    ...
    private Hello hello = new HelloImp();
    @Override
    public String sayHello(String str) {
        logger.info("You said: " + str);
        return hello.sayHello(str);
    }
}
```

上例中静态代理类`StaticProxiedHello`作为`HelloImp`的代理，实现了相同的`Hello`接口。用Java动态代理可以这样做：

1. 首先实现一个InvocationHandler，方法调用会被转发到该类的invoke()方法。
2. 然后在需要使用Hello的时候，通过JDK动态代理获取Hello的代理对象。

```
// Java Proxy
// 1. 首先实现一个InvocationHandler，方法调用会被转发到该类的invoke()方法。
class LogInvocationHandler implements InvocationHandler{
    ...
    private Hello hello;
    public LogInvocationHandler(Hello hello) {
        this.hello = hello;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if("sayHello".equals(method.getName())) {
            logger.info("You said: " + Arrays.toString(args));
        }
        return method.invoke(hello, args);
    }
}
// 2. 然后在需要使用Hello的时候，通过JDK动态代理获取Hello的代理对象。
Hello hello = (Hello)Proxy.newProxyInstance(
    getClass().getClassLoader(), // 1. 类加载器
    new Class<?>[] {Hello.class}, // 2. 代理需要实现的接口，可以有多个
    new LogInvocationHandler(new HelloImp()));// 3. 方法调用的实际处理者
System.out.println(hello.sayHello("I love you!"));
```

运行上述代码输出结果：

```
日志信息: You said: [I love you!]
HelloImp: I love you!
```

上述代码的关键是`Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler handler)`方法，该方法会根据指定的参数动态创建代理对象。三个参数的意义如下：

1. `loader`，指定代理对象的类加载器；
2. `interfaces`，代理对象需要实现的接口，可以同时指定多个接口；
3. `handler`，方法调用的实际处理者，代理对象的方法调用都会转发到这里（*注意1）。

`newProxyInstance()`会返回一个实现了指定接口的代理对象，对该对象的所有方法调用都会转发给`InvocationHandler.invoke()`方法。理解上述代码需要对Java反射机制有一定了解。动态代理神奇的地方就是：

1. 代理对象是在程序运行时产生的，而不是编译期；
2. **对代理对象的所有接口方法调用都会转发到InvocationHandler.invoke()方法**，在`invoke()`方法里我们可以加入任何逻辑，比如修改方法参数，加入日志功能、安全检查功能等；之后我们通过某种方式执行真正的方法体，示例中通过反射调用了Hello对象的相应方法，还可以通过RPC调用远程方法。

> 注意1：对于从Object中继承的方法，JDK Proxy会把`hashCode()`、`equals()`、`toString()`这三个非接口方法转发给`InvocationHandler`，其余的Object方法则不会转发。详见[JDK Proxy官方文档](https://docs.oracle.com/javase/7/docs/api/java/lang/reflect/Proxy.html)。

如果对JDK代理后的对象类型进行深挖，可以看到如下信息：

```
# Hello代理对象的类型信息
class=class jdkproxy.$Proxy0
superClass=class java.lang.reflect.Proxy
interfaces: 
interface jdkproxy.Hello
invocationHandler=jdkproxy.LogInvocationHandler@a09ee92
```

代理对象的类型是`jdkproxy.$Proxy0`，这是个动态生成的类型，类名是形如$ProxyN的形式；父类是`java.lang.reflect.Proxy`，所有的JDK动态代理都会继承这个类；同时实现了`Hello`接口，也就是我们接口列表中指定的那些接口。

如果你还对`jdkproxy.$Proxy0`具体实现感兴趣，它大致长这个样子：

```
// JDK代理类具体实现
public final class $Proxy0 extends Proxy implements Hello
{
  ...
  public $Proxy0(InvocationHandler invocationhandler)
  {
    super(invocationhandler);
  }
  ...
  @Override
  public final String sayHello(String str){
    ...
    return super.h.invoke(this, m3, new Object[] {str});// 将方法调用转发给invocationhandler
    ...
  }
  ...
}
```

这些逻辑没什么复杂之处，但是他们是在运行时动态产生的，无需我们手动编写。更多详情，可参考BrightLoong的[Java静态代理&动态代理笔记](https://www.jianshu.com/p/e2917b0b9614)

Java动态代理为我们提供了非常灵活的代理机制，但Java动态代理是基于接口的，如果对象没有实现接口我们该如何代理呢？CGLIB登场。

## CGLIB动态代理

[CGLIB](https://github.com/cglib/cglib)(*Code Generation Library*)是一个基于[ASM](http://www.baeldung.com/java-asm)的字节码生成库，它允许我们在运行时对字节码进行修改和动态生成。CGLIB通过继承方式实现代理。

来看示例，假设我们有一个没有实现任何接口的类`HelloConcrete`：

```
public class HelloConcrete {
    public String sayHello(String str) {
        return "HelloConcrete: " + str;
    }
}
```

因为没有实现接口该类无法使用JDK代理，通过CGLIB代理实现如下：

1. 首先实现一个MethodInterceptor，方法调用会被转发到该类的intercept()方法。
2. 然后在需要使用HelloConcrete的时候，通过CGLIB动态代理获取代理对象。

```
// CGLIB动态代理
// 1. 首先实现一个MethodInterceptor，方法调用会被转发到该类的intercept()方法。
class MyMethodInterceptor implements MethodInterceptor{
  ...
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        logger.info("You said: " + Arrays.toString(args));
        return proxy.invokeSuper(obj, args);
    }
}
// 2. 然后在需要使用HelloConcrete的时候，通过CGLIB动态代理获取代理对象。
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(HelloConcrete.class);
enhancer.setCallback(new MyMethodInterceptor());

HelloConcrete hello = (HelloConcrete)enhancer.create();
System.out.println(hello.sayHello("I love you!"));
```

运行上述代码输出结果：

```
日志信息: You said: [I love you!]
HelloConcrete: I love you!
```

上述代码中，我们通过CGLIB的`Enhancer`来指定要代理的目标对象、实际处理代理逻辑的对象，最终通过调用`create()`方法得到代理对象，**对这个对象所有非final方法的调用都会转发给MethodInterceptor.intercept()方法**，在`intercept()`方法里我们可以加入任何逻辑，比如修改方法参数，加入日志功能、安全检查功能等；通过调用`MethodProxy.invokeSuper()`方法，我们将调用转发给原始对象，具体到本例，就是`HelloConcrete`的具体方法。CGLIG中[MethodInterceptor](http://cglib.sourceforge.net/apidocs/net/sf/cglib/proxy/MethodInterceptor.html)的作用跟JDK代理中的`InvocationHandler`很类似，都是方法调用的中转站。

> 注意：对于从Object中继承的方法，CGLIB代理也会进行代理，如`hashCode()`、`equals()`、`toString()`等，但是`getClass()`、`wait()`等方法不会，因为它是final方法，CGLIB无法代理。

如果对CGLIB代理之后的对象类型进行深挖，可以看到如下信息：

```
# HelloConcrete代理对象的类型信息
class=class cglib.HelloConcrete$$EnhancerByCGLIB$$e3734e52
superClass=class lh.HelloConcrete
interfaces: 
interface net.sf.cglib.proxy.Factory
invocationHandler=not java proxy class
```

我们看到使用CGLIB代理之后的对象类型是`cglib.HelloConcrete$$EnhancerByCGLIB$$e3734e52`，这是CGLIB动态生成的类型；父类是`HelloConcrete`，印证了CGLIB是通过继承实现代理；同时实现了`net.sf.cglib.proxy.Factory`接口，这个接口是CGLIB自己加入的，包含一些工具方法。

注意，既然是继承就不得不考虑final的问题。我们知道final类型不能有子类，所以CGLIB不能代理final类型，遇到这种情况会抛出类似如下异常：

```
java.lang.IllegalArgumentException: Cannot subclass final class cglib.HelloConcrete
```

同样的，final方法是不能重载的，所以也不能通过CGLIB代理，遇到这种情况不会抛异常，而是会跳过final方法只代理其他方法。

如果你还对代理类`cglib.HelloConcrete$$EnhancerByCGLIB$$e3734e52`具体实现感兴趣，它大致长这个样子：

```
// CGLIB代理类具体实现
public class HelloConcrete$$EnhancerByCGLIB$$e3734e52
  extends HelloConcrete
  implements Factory
{
  ...
  private MethodInterceptor CGLIB$CALLBACK_0; // ~~
  ...
  
  public final String sayHello(String paramString)
  {
    ...
    MethodInterceptor tmp17_14 = CGLIB$CALLBACK_0;
    if (tmp17_14 != null) {
      // 将请求转发给MethodInterceptor.intercept()方法。
      return (String)tmp17_14.intercept(this, 
              CGLIB$sayHello$0$Method, 
              new Object[] { paramString }, 
              CGLIB$sayHello$0$Proxy);
    }
    return super.sayHello(paramString);
  }
  ...
}
```

上述代码我们看到，当调用代理对象的`sayHello()`方法时，首先会尝试转发给`MethodInterceptor.intercept()`方法，如果没有`MethodInterceptor`就执行父类的`sayHello()`。这些逻辑没什么复杂之处，但是他们是在运行时动态产生的，无需我们手动编写。如何获取CGLIB代理类字节码可参考[Access the generated byte[\] array directly](https://github.com/cglib/cglib/wiki/How-To#access-the-generated-byte-array-directly)。

更多关于CGLIB的介绍可以参考Rafael Winterhalter的[cglib: The missing manual](https://dzone.com/articles/cglib-missing-manual)，一篇很深入的文章。

## 结语

本文介绍了Java两种常见动态代理机制的用法和原理，JDK原生动态代理是Java原生支持的，不需要任何外部依赖，但是它只能基于接口进行代理；CGLIB通过继承的方式进行代理，无论目标对象有没有实现接口都可以代理，但是无法处理final的情况。

动态代理是[Spring AOP](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop)(*Aspect Orient Programming, 面向切面编程*)的实现方式，了解动态代理原理，对理解Spring AOP大有帮助。

参考：http://www.cnblogs.com/CarpenterLee/p/8241042.html





# Android依赖注入 ioc 



基本思想 = 配置文件 + factory实例

https://www.cnblogs.com/Eason-S/p/5851078.html

IOC(Inversion of control):控制反转,依赖注入

概念:控制权有对象本身专享容器,由容器根据配置文件去创建实例,并创建各个实例之间的关系,则通俗的说，对象的创建再也不需要程序员来管理,而是可以有spring容器来进行创建和销毁,我们只需要关注业务逻辑.

依赖IOC容器并管理bean,有两种,一种是BeanFactory,另一种是ApplicationContext,但是APPlicationContext extends BeanFactory.

核心:Spring中,bean工厂创建的各个实例称作bean.



通过 @Inject 注解了构造函数之后，在 Activity 中的 Boss 属性声明之前也添加 @Inject 注解。像这种在属性前添加的 @Inject 注解的目的是告诉 Dagger 哪些属性需要被注入。

 

public class MainActivity extends Activity {

    @Inject Boss boss;
    
    ...

}

最后，我们在合适的位置(例如 onCreate() 函数中)调用 ObjectGraph.inject() 函数，Dagger 就会自动调用上面 (1) 中的生成方法生成依赖的实例，并注入到当前对象(MainActivity)。

 

public class MainActivity extends Activity {

    @Inject Boss boss;

 



    @Override
    
    protected void onCreate(Bundle savedInstanceState) {
    
        ObjectGraph.create(AppModule.class).inject(this);
    
    }
    
    ...

}

具体怎么注入即设置的过程后面会详细介绍，这里简单透露下，APT 会在 MainActivity 所在 package 下生成一个辅助类 MainActivity$$InjectAdapter，这个类有个 injectMembers() 函数，代码类似：

 

public void injectMembers(MainActivity paramMainActivity) {

    paramMainActivity.boss = ((Boss)boss.get());
    
    ……

}

上面我们已经通过 ObjectGraph.inject() 函数传入了 paramMainActivity，并且 boss 属性是 package 权限，所以 Dagger 只需要调用这个辅助类的 injectMembers() 函数即可完成依赖注入，这里的 boss.get() 会调用 Boss 的生成函数。

到此为止，使用 Dagger 的 @Inject 方式将一个 Boss 对象注入到 MainActivity 的流程就完成了。

```

public class Human {

    ...

    @Inject Father father;

    ...

    public Human() {

    }

}

 

public class Human {

    ...

    Father father;

    ...

    public Human() {

        father = new Father();

    }

}

```

 

**另可以参考**：https://blog.csdn.net/lmj623565791/article/details/39269193



# Spring依赖注入原理分析

2015年07月18日 23:55:58

阅读数：1817

我们知道Spring的依赖注入有四种方式，分别是get/set方法注入、构造器注入、静态工厂方法注入、实例工厂方法注入 
下面我们先分析下这几种注入方式 
**1、get/set方法注入**

```
public class SpringAction {
        //注入对象springDao
    private SpringDao springDao;
        //一定要写被注入对象的set方法
        public void setSpringDao(SpringDao springDao) {
        this.springDao = springDao;
    }
        public void ok(){
        springDao.ok();
    }
}1234567891011
```

配置文件如下：

```
<!--配置bean,配置后该类由spring管理-->
    <bean name="springAction" class="com.bless.springdemo.action.SpringAction">
        <!--(1)依赖注入,配置当前类中相应的属性-->
        <property name="springDao" ref="springDao"></property>
    </bean>
<bean name="springDao" class="com.bless.springdemo.dao.impl.SpringDaoImpl"></bean>123456
```

**2、构造器注入**

```
public class SpringAction {
    //注入对象springDao
    private SpringDao springDao;
    private User user;

    public SpringAction(SpringDao springDao,User user){
        this.springDao = springDao;
        this.user = user;
        System.out.println("构造方法调用springDao和user");
    }

        public void save(){
        springDao.save(user);
    }
}123456789101112131415
```

在XML文件中同样不用的形式，而是使用标签，ref属性同样指向其它标签的name属性：

```
<!--配置bean,配置后该类由spring管理-->
    <bean name="springAction" class="com.bless.springdemo.action.SpringAction">
        <!--(2)创建构造器注入,如果主类有带参的构造方法则需添加此配置-->
        <constructor-arg ref="springDao"></constructor-arg>
        <constructor-arg ref="user"></constructor-arg>
    </bean>
        <bean name="springDao" class="com.bless.springdemo.dao.impl.SpringDaoImpl"></bean>
        <bean name="user" class="com.bless.springdemo.vo.User"></bean>12345678
```

在XML文件中同样不用的形式，而是使用标签，ref属性同样指向其它标签的name属性： 
解决构造方法参数的不确定性，你可能会遇到构造方法传入的两参数都是同类型的，为了分清哪个该赋对应值，则需要进行一些小处理：

```
<bean name="springAction" class="com.bless.springdemo.action.SpringAction">  
        <constructor-arg index="0" ref="springDao"></constructor-arg>  
        <constructor-arg index="1" ref="user"></constructor-arg>  
</bean>  1234
```

另一种是设置参数类型：

```
<constructor-arg type="java.lang.String" ref=""/>  1
```

**3、静态工厂方法注入** 
通过调用静态工厂方法来获取自己需要的对象，为了让Spring管理所有对象，我们不能直接通过类名加方法来获取对象，那样就脱离了Spring的管理，而是通过Spring注入的形式来获取

```
package com.bless.springdemo.factory;
import com.bless.springdemo.dao.FactoryDao;
import com.bless.springdemo.dao.impl.FactoryDaoImpl;
import com.bless.springdemo.dao.impl.StaticFacotryDaoImpl;
public class DaoFactory {
    //静态工厂
    public static final FactoryDao getStaticFactoryDaoImpl(){
        return new StaticFacotryDaoImpl();
    }
}12345678910
```

同样看关键类，这里我需要注入一个FactoryDao对象，这里看起来跟第一种注入一模一样，但是看随后的xml会发现有很大差别:

```
 public class SpringAction {
        //注入对象
    private FactoryDao staticFactoryDao;

    public void staticFactoryOk(){
        staticFactoryDao.saveFactory();
    }
    //注入对象的set方法
    public void setStaticFactoryDao(FactoryDao staticFactoryDao) {
        this.staticFactoryDao = staticFactoryDao;
    }
}123456789101112
```

配置文件如下：

```
<!--配置bean,配置后该类由spring管理-->
    <bean name="springAction" class="com.bless.springdemo.action.SpringAction" >
        <!--(3)使用静态工厂的方法注入对象,对应下面的配置文件(3)-->
        <property name="staticFactoryDao" ref="staticFactoryDao"></property>
                </property>
    </bean>
    <!--(3)此处获取对象的方式是从工厂类中获取静态方法-->
    <bean name="staticFactoryDao" class="com.bless.springdemo.factory.DaoFactory" factory-method="getStaticFactoryDaoImpl"></bean>12345678
```

**4、实例工厂方法注入** 
实例工厂的意思是获取对象实例的方法不是静态的，所以你需要首先new工厂类，再调用普通的实例方法：

```
public class DaoFactory {
    //实例工厂
    public FactoryDao getFactoryDaoImpl(){
        return new FactoryDaoImpl();
    }
}123456
```

```
public class SpringAction {
    //注入对象
    private FactoryDao factoryDao;

    public void factoryOk(){
        factoryDao.saveFactory();
    }
    public void setFactoryDao(FactoryDao factoryDao) {
        this.factoryDao = factoryDao;
    }
}1234567891011
```

```
<!--配置bean,配置后该类由spring管理-->
    <bean name="springAction" class="com.bless.springdemo.action.SpringAction">
        <!--(4)使用实例工厂的方法注入对象,对应下面的配置文件(4)-->
        <property name="factoryDao" ref="factoryDao"></property>
    </bean>

    <!--(4)此处获取对象的方式是从工厂类中获取实例方法-->
    <bean name="daoFactory" class="com.bless.springdemo.factory.DaoFactory"></bean>
    <bean name="factoryDao" factory-bean="daoFactory" factory-method="getFactoryDaoImpl"></bean>123456789
```

对于第1、2种我们用的比较多，对后两种可能比较陌生。 
下面我们来分析下Spring是如何完成依赖注入的。如果我们去看Spring的源码可能涉及的类和接口相当多，不易掌握，在此我用自己的代码和方式来帮助我们Spring依赖注入的过程。 
当我们启动Spring容器的时候他会执行以下几个过程： 
**1、加载Xml配置文件(readXML(String filename))在Spring这个由ApplicationContext类完成** 
这一步会解析Xml属性，把bean的属性存放到BeanDefinition类中 
代码如下：

```
/**
     * 读取xml配置文件
     * @param filename
     */
    private void readXML(String filename) {
           SAXReader saxReader = new SAXReader();   
            Document document=null;   
            try{
             URL xmlpath = this.getClass().getClassLoader().getResource(filename);
             document = saxReader.read(xmlpath);
             Map<String,String> nsMap = new HashMap<String,String>();
             nsMap.put("ns","http://www.springframework.org/schema/beans");//加入命名空间
             XPath xsub = document.createXPath("//ns:beans/ns:bean");//创建beans/bean查询路径
             xsub.setNamespaceURIs(nsMap);//设置命名空间
             List<Element> beans = xsub.selectNodes(document);//获取文档下所有bean节点 
             for(Element element: beans){
                String id = element.attributeValue("id");//获取id属性值
                String clazz = element.attributeValue("class"); //获取class属性值        
                BeanDefinition beanDefine = new BeanDefinition(id, clazz);
                XPath propertysub =  element.createXPath("ns:property");
                propertysub.setNamespaceURIs(nsMap);//设置命名空间
                List<Element> propertys = propertysub.selectNodes(element);
                for(Element property : propertys){                  
                    String propertyName = property.attributeValue("name");
                    String propertyref = property.attributeValue("ref");
                    PropertyDefinition propertyDefinition = new PropertyDefinition(propertyName, propertyref);
                    beanDefine.getPropertys().add(propertyDefinition);
                }
                beanDefines.add(beanDefine);
             } 
            }catch(Exception e){   
                e.printStackTrace();
            }
    }12345678910111213141516171819202122232425262728293031323334
```

**2、Bean的实例化** 
在配置文件以bean的id为key，BeanDefinition为value放到Map中

```
/**
     * 完成bean的实例化
     */
    private void instanceBeans() {
        for(BeanDefinition beanDefinition : beanDefines){
            try {
                if(beanDefinition.getClassName()!=null && !"".equals(beanDefinition.getClassName().trim()))
                    sigletons.put(beanDefinition.getId(), Class.forName(beanDefinition.getClassName()).newInstance());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

    }1234567891011121314
```

**3、为Bean的输入注入值，完成依赖注入**

```
/**
     * 为bean对象的属性注入值
     */
    private void injectObject() {
        for(BeanDefinition beanDefinition : beanDefines){
            Object bean = sigletons.get(beanDefinition.getId());
            if(bean!=null){
                try {
                    PropertyDescriptor[] ps = Introspector.getBeanInfo(bean.getClass()).getPropertyDescriptors();
                    for(PropertyDefinition propertyDefinition : beanDefinition.getPropertys()){
                        for(PropertyDescriptor properdesc : ps){
                            if(propertyDefinition.getName().equals(properdesc.getName())){
                                Method setter = properdesc.getWriteMethod();//获取属性的setter方法 ,private
                                if(setter!=null){
                                    Object value = sigletons.get(propertyDefinition.getRef());
                                    setter.setAccessible(true);
                                    setter.invoke(bean, value);//把引用对象注入到属性
                                }
                                break;
                            }
                        }
                    }
                } catch (Exception e) {
                }
            }
        }
    }123456789101112131415161718192021222324252627
```

其实Spring依赖注入的过程就是这么简单，再就是各种细节了，比如懒加载、单例等的额外处理了。

 

# Android中的aop思路：

**面向切面编程（AOP，Aspect-oriented programming）**需要把程序逻辑分解成『**关注点**』（concerns，功能的内聚区域）。这意味着，在 AOP 中，我们不需要显式的修改就可以向代码中添加可执行的代码块。这种编程范式假定『横切关注点』（cross-cutting concerns，多处代码中需要的逻辑，但没有一个单独的类来实现）应该只被实现一次，且能够多次注入到需要该逻辑的地方。

**代码注入是 AOP 中的重要部分：**它在处理上述提及的横切整个应用的**『关注点』**时很有用，例如日志或者性能监控。这种方式，并不如你所想的应用甚少，相反的，每个程序员都可以有使用这种注入代码能力的场景，这样可以避免很多痛苦和无奈。

**AOP** 是一种已经存在了很多年的编程范式。我发现把它应用到 Android 开发中也很有用。经过一番调研后，我认为我们用它可以获得很多好处和有用的东西。

## 术语（迷你术语表）

在开始之前，我们先看看需要了解的词汇：

- **Cross-cutting concerns（横切关注点）:** 尽管面向对象模型中大多数类会实现单一特定的功能，但通常也会开放一些通用的附属功能给其他类。例如，我们希望在数据访问层中的类中添加日志，同时也希望当UI层中一个线程进入或者退出调用一个方法时添加日志。尽管每个类都有一个区别于其他类的主要功能，但在代码里，仍然经常需要添加一些相同的附属功能。
- **Advice（通知）:** 注入到class文件中的代码。典型的 Advice 类型有 before、after 和 around，分别表示在目标方法执行之前、执行后和完全替代目标方法执行的代码。 除了在方法中注入代码，也可能会对代码做其他修改，比如在一个class中增加字段或者接口。
- **Joint point（连接点）:** 程序中可能作为代码注入目标的特定的点，例如一个方法调用或者方法入口。
- **Pointcut（切入点）:** 告诉代码注入工具，在何处注入一段特定代码的表达式。例如，在哪些 joint points 应用一个特定的 Advice。切入点可以选择唯一一个，比如执行某一个方法，也可以有多个选择，比如，标记了一个定义成@DebguTrace 的自定义注解的所有方法。
- **Aspect（切面）:** Pointcut 和 Advice 的组合看做切面。例如，我们在应用中通过定义一个 pointcut 和给定恰当的advice，添加一个日志切面。
- **Weaving（织入）:** 注入代码（advices）到目标位置（joint points）的过程。

下面这张图简要总结了一下上述这些概念。

[![img](https://camo.githubusercontent.com/1fb49fef65877a7f80ad4622bf231f2b01804be3/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f33303638392d353538343639393866346635623463652e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/1fb49fef65877a7f80ad4622bf231f2b01804be3/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f33303638392d353538343639393866346635623463652e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

## 那么...我们何时何地应用AOP呢？

一些示例的 cross-cutting concerns 如下：

- **日志**
- **持久化**
- **性能监控**
- **数据校验**
- **缓存**
- [其他更多](http://en.wikipedia.org/wiki/Cross-cutting_concern)

取决于你所选的其中一种或其他方案 :)。

## 工具和库

有一些工具和库帮助我们使用 AOP:

- [AspectJ:](https://eclipse.org/aspectj/) 一个 JavaTM 语言的面向切面编程的无缝扩展（适用Android）。
- [Javassist for Android:](https://github.com/crimsonwoods/javassist-android) 用于字节码操作的知名 java 类库 Javassist 的 Android 平台移植版。
- [DexMaker:](https://code.google.com/p/dexmaker/) Dalvik 虚拟机上，在编译期或者运行时生成代码的 Java API。
- [ASMDEX:](http://asm.ow2.org/asmdex-index.html) 一个类似 ASM 的字节码操作库，运行在Android平台，操作Dex字节码。

## 为什么用 AspectJ？

我们下面的例子选用 AspectJ，有以下原因：

- **功能强大**
- **支持编译期和加载时代码注入**
- **易于使用**

## 示例

比方说，我们要测量一个方法的性能（执行这个方法需要多长时间）。为此我们用一个 **@DebugTrace** 的注解标记我们的这个方法，并且无需在每个注解过的方法中编写代码，就可以通过 logcat 输出结果。我们的方法是使用 AspectJ 达到这个目的。

我们看下在底层到底发生了什么：

- **我们在编译过程中增加一个新的步骤处理注解。**
- **注解的方法内会生成和注入必要的样板代码。**

在此，我必须要提到当我研究这些时，发现了[Jake Wharton’s Hugo Library](https://github.com/JakeWharton/hugo) 这个项目，支持做同样的事情。因此，我重构了我的代码，看上去和它类似。尽管，我的代码是一个更加原始和简化的版本（顺便提一下，通过看这个项目的代码，我学到了很多）。

[![img](https://camo.githubusercontent.com/511e2eb42079b442554e0c20cdf8bb39fcd68bc5/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f33303638392d373766613462613334633461666536302e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/511e2eb42079b442554e0c20cdf8bb39fcd68bc5/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f33303638392d373766613462613334633461666536302e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

### 工程结构

我们会把一个简单的示例应用拆分成两个 modules，第一个包含我们的 Android App 代码，第二个是一个 Android Library 工程，使用 AspectJ 织入代码（代码注入）。

你可能会想知道为什么我们用一个 Android Library 工程，而不是用一个纯的 Java Library：原因是为了使 AspectJ 能在 Android 上运行，我们必须在编译时做一些 hook。这只能使用 andorid-library gradle 插件完成。（先不要为此担心，后面我会给出更多细节。）

### 创建注解

首先我们创建我们的Java注解。这个注解周期声明在 class 文件上（RetentionPolicy.CLASS），可以注解构造函数和方法（ElementType.CONSTRUCTOR 和 ElementType.METHOD）。因此，我们的 DebugTrace.java 文件看上是这样的：

```
@Retention(RetentionPolicy.CLASS)
@Target({ ElementType.CONSTRUCTOR, ElementType.METHOD })
public @interface DebugTrace {}
```

### 我们的性能监控计时类

我已经创建了一个简单的计时类，包含 `start/stop` 方法。下面是 StopWatch.java 文件:

```
/**
 * Class representing a StopWatch for measuring time.
 */
public class StopWatch {
  private long startTime;
  private long endTime;
  private long elapsedTime;

  public StopWatch() {
    //empty
  }

  private void reset() {
    startTime = 0;
    endTime = 0;
    elapsedTime = 0;
  }

  public void start() {
    reset();
    startTime = System.nanoTime();
  }

  public void stop() {
    if (startTime != 0) {
      endTime = System.nanoTime();
      elapsedTime = endTime - startTime;
    } else {
      reset();
    }
  }

  public long getTotalTimeMillis() {
    return (elapsedTime != 0) ? TimeUnit.NANOSECONDS.toMillis(endTime - startTime) : 0;
  }
}
```

### DebugLog 类

我只是包装了一下 “android.util.Log”，因为我首先想到的是向 android log 中增加更多的实用功能。下面是代码：

```
/**
 * Wrapper around {@link android.util.Log}
 */
public class DebugLog {

  private DebugLog() {}

  /**
   * Send a debug log message
   *
   * @param tag Source of a log message.
   * @param message The message you would like logged.
   */
  public static void log(String tag, String message) {
    Log.d(tag, message);
  }
}
```

### Aspect 类

现在是时候创建我们的 Aspect 类（TraceAspect.java）了。Aspect 类负责管理注解的处理和代码织入。

```
/**
 * Aspect representing the cross cutting-concern: Method and Constructor Tracing.
 */
@Aspect
public class TraceAspect {

  private static final String POINTCUT_METHOD =
      "execution(@org.android10.gintonic.annotation.DebugTrace * *(..))";

  private static final String POINTCUT_CONSTRUCTOR =
      "execution(@org.android10.gintonic.annotation.DebugTrace *.new(..))";

  @Pointcut(POINTCUT_METHOD)
  public void methodAnnotatedWithDebugTrace() {}

  @Pointcut(POINTCUT_CONSTRUCTOR)
  public void constructorAnnotatedDebugTrace() {}

  @Around("methodAnnotatedWithDebugTrace() || constructorAnnotatedDebugTrace()")
  public Object weaveJoinPoint(ProceedingJoinPoint joinPoint) throws Throwable {
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    String className = methodSignature.getDeclaringType().getSimpleName();
    String methodName = methodSignature.getName();

    final StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    Object result = joinPoint.proceed();
    stopWatch.stop();

    DebugLog.log(className, buildLogMessage(methodName, stopWatch.getTotalTimeMillis()));

    return result;
  }

  /**
   * Create a log message.
   *
   * @param methodName A string with the method name.
   * @param methodDuration Duration of the method in milliseconds.
   * @return A string representing message.
   */
  private static String buildLogMessage(String methodName, long methodDuration) {
    StringBuilder message = new StringBuilder();
    message.append("Gintonic --> ");
    message.append(methodName);
    message.append(" --> ");
    message.append("[");
    message.append(methodDuration);
    message.append("ms");
    message.append("]");

    return message.toString();
  }
}
```

几个在此提到的重点：

- 我们声明了两个作为 pointcuts 的 public 方法，筛选出所有通过 `“org.android10.gintonic.annotation.DebugTrace”` 注解的方法和构造函数。
- 我们使用 `“@Around”` 注解定义了`“weaveJointPoint(ProceedingJoinPoint joinPoint)”`方法,使我们的代码注入在使用`"@DebugTrace"`注解的地方生效。
- `“Object result = joinPoint.proceed();”`这行代码是被注解的方法执行的地方。因此，在此之前，我们启动我们的计时类计时，在这之后，停止计时。
- 最后，我们构造日志信息，用 Android Log 输出。

\###使 AspectJ 运行在 Anroid 上 现在，所有代码都可以正常工作了，但是，如果我们编译我们的例子，我们并没有看到任何事情发生。原因是我们必须使用 AspectJ 的编译器（ajc，一个java编译器的扩展）对所有受 aspect 影响的类进行织入。这就是为什么，我之前提到的，我们需要在 gradle 的编译 task 中增加一些额外配置，使之能正确编译运行。

我们的 build.gradle 文件如下：

```
import com.android.build.gradle.LibraryPlugin
import org.aspectj.bridge.IMessage
import org.aspectj.bridge.MessageHandler
import org.aspectj.tools.ajc.Main

buildscript {
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:0.12.+'
    classpath 'org.aspectj:aspectjtools:1.8.1'
  }
}

apply plugin: 'android-library'

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.aspectj:aspectjrt:1.8.1'
}

android {
  compileSdkVersion 19
  buildToolsVersion '19.1.0'

  lintOptions {
    abortOnError false
  }
}

android.libraryVariants.all { variant ->
  LibraryPlugin plugin = project.plugins.getPlugin(LibraryPlugin)
  JavaCompile javaCompile = variant.javaCompile
  javaCompile.doLast {
    String[] args = ["-showWeaveInfo",
                     "-1.5",
                     "-inpath", javaCompile.destinationDir.toString(),
                     "-aspectpath", javaCompile.classpath.asPath,
                     "-d", javaCompile.destinationDir.toString(),
                     "-classpath", javaCompile.classpath.asPath,
                     "-bootclasspath", plugin.project.android.bootClasspath.join(
        File.pathSeparator)]

    MessageHandler handler = new MessageHandler(true);
    new Main().run(args, handler)

    def log = project.logger
    for (IMessage message : handler.getMessages(null, true)) {
      switch (message.getKind()) {
        case IMessage.ABORT:
        case IMessage.ERROR:
        case IMessage.FAIL:
          log.error message.message, message.thrown
          break;
        case IMessage.WARNING:
        case IMessage.INFO:
          log.info message.message, message.thrown
          break;
        case IMessage.DEBUG:
          log.debug message.message, message.thrown
          break;
      }
    }
  }
}
```

### 我们的测试方法

我们添加一个测试方法，来使用我们炫酷的 aspect 注解。我已经在主 Activity 类中增加了一个方法用来测试。看下代码：

```
  @DebugTrace
  private void testAnnotatedMethod() {
    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
```

### 运行我们的应用

我们用 gradle 命令编译部署我们的 app 到 android 设备或者模拟器上：

```
gradlew clean build installDebug
```

If we open the logcat and execute our sample, we will see a debug log with: 如果我们打开 logcat，执行我们的例子，会看到一条 debug 日志：

```
Gintonic --> testAnnotatedMethod --> [10ms]
```

**我们的第一个使用 AOP 的 Androd 应用可以工作了！** 你可以用 Dex Dump 或者任何其他的逆向工具反编译 apk 文件，看一下生成和注入的代码。

## 回顾

回顾总结如下：

- 我们已经对面向切面编程（AOP）这一范式有了初步体验。
- 代码注入是 AOP 中的重要部分。
- AspectJ 是在 Android 应用中进行代码织入的强大且易用的工具。
- 我们已经使用 AOP 能力创建了一个可以工作的示例。



参考：https://github.com/hehonghui/android-tech-frontier/blob/master/issue-22/Android%E4%B8%AD%E7%9A%84AOP%E7%BC%96%E7%A8%8B.md