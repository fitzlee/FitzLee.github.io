---
title: java jni原理分析
urlname: java_jdk_base_jni
#date: 2016-03-06
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 00Java&JDK
tags:
    - java
    - jdk
    - jni
---

# JNI原理解析



# 第二章 开始

  本章将引导你了解如何使用Java本地接口。我们将编写一个Java应用程序调用一个C函数来答应“Hello World！”。

## 2.1 概述

  图2.1表明使用JDK或者Java SDK 2发行版编写一个调用C函数来打印“Hello World！”的Java应用程序的过程。这个过程有以下步骤组成：

![《(译文) JNI编程指南与规范 第二章 开始编程》](https://i1.wp.com/blog4jimmy.com/wp-content/uploads/2017/11/JNI2.1.png)

- 创建申明了本地方法的类（HelloWorld.java）
- 使用javac去编译这个HelloWorld源文件，得到一个HelloWorld.class的class文件。javac编译工具是JDK或者Java 2 SDK发行版中提供的。
- 使用javah -jni去生成包含本地方法函数原型的C头文件（HelloWorld.h）。javah工具也是在JDK或者Java 2 SDK发行版中带有的。
- 编写本地方法的C实现（HelloWorld.c）
- 将C实现编译成一个本地库helloWorld.dll或者libHelloWorld.so，使用主机环境中可用的C编译器和连接器
- 使用Java运行时解析器运行hello world程序。类文件（HelloWorld.class）和本机库（HelloWorld.dll或者HelloWorld.so）都在运行时加载。

本章的剩余部分将详细讲解这些步骤。

## 2.2 定义本地方法

  首先用Java编程语言编写以下程序。这个程序定义了一个包含本地方法print，类名为HelloWorld的类

```
class HelloWorld {
    private native void print();
    
    public static void main(String[] args) {
        new HelloWorld().print();
    }
    
    static {
        System.loadLibrary("HelloWorld");
    }
}
```

  HelloWorld类定义从打印本地方法的声明开始。之后是实例化HelloWorld类并调用此示例的print本地方法。类定义最后一部分是一个静态初始化器，它加载包含本地print方法的本地库。

  定义一个本地方法例如print和使用Java编程语言定义一个常规的方法存在两个不同的地方。一个本地方法声明必须包含native修饰符。native修饰符表明该方法由其他编程语言实现。此外本地方法声明以分号终结（语句终结符），因为在这个类中没有实现这个本地方法。我们会在独立的c文件中完成print方法的编写。

  在本地方法print能够被调用前，实现了print方法的本地库必须被加载。在这个例子中，我们在HelloWorld类的静态初始化块中将本地库加载进来。Java虚拟机在调用HelloWorld类的任何方法前先运行静态初始化块的代码，因此可以肯定在本地方法print被调用前，本地库就已经被加载了。

  我们定义了一个可以运行HelloWorld类的main方法。HelloWorld.main方法以与常规方法相同的方式调用print本地方法。

  System.loadLibrary使用库名字，找到与库名字相关的本地库，并将本地库加载到应用程序中。我们会在本书的后面部分讨论准确的加载过程。现在只需要记住，为了使System.loadLibrary(“HelloWorld”)能够成功，我们需要在win32系统上创建一个HelloWorld.dll文件，在Solaris系统中创建一个libHelloWorld.so文件。

## 2.3 编译HelloWorld类

  在你完成HelloWorld类的编写，将源码保存到一个名为HelloWorld.java的文件中。使用JDK或者Java SDK 2中带有的javac工具进行编译：

```
javac HelloWorld.java
```

  这条指令会在当前目录中产生一个HelloWorld.class文件。

## 2.4 创建本地方法的头文件

  接下来我们会使用javah工具来生成一个JNI类型的头文件，这个文件在后面使用C语言完成本地方法时是非常有用的。执行javah的指令如下：

```
javah -jni HelloWorld
```

  头文件的名字是类名并在其后面加上”.h”结尾。上面的指令会生成一个名字为HelloWorld.h的文件。在这里我们不会列出这个头文件的内容。这个文件的最重要的部分是Java_HelloWorld_print函数原型，它是实现了HelloWorld.print方法的C函数：

```
JNIEXPORT void JNICALL Java_HelloWorld_print
  (JNIEnv *, jobject);
```

  现在先忽略JNIEXPORT和JNICALL宏。你可能有注意到本地方法的C实现接受两个参数，尽管相应本地方法的定义（指HelloWorld.java中的定义）却没有接受任何参数。每一个本地方法实现的第一个参数是一个JNIEnv接口指针。第二个参数是引用HelloWorld对象本身，类似于C++的this指针。在本书的后面我们会讨论如何使用JNIEnv接口指针和jobject参数，但是在这个例子将忽略这两个参数。

## 2.5 编写本地方法实现

  使用javah来生成的JNI类型头文件能够帮助你使用C/C++来完成本地方法的实现。你编写的函数必须遵循生成的头文件中的函数原型。在C文件HelloWorld.c中，你可以按照下面的代码来实现HelloWorld.print方法。

```
#include <jni.h>
#include <stdio.h>
#include <HelloWorld.h>

JNIEXPORT void JNICALL Java_HelloWorld_print
  (JNIEnv *env, jobject obj)
{
    printf("Hello World!\n");
    return ;
}
```

  这个本地方法的实现是非常简单的。它使用printf函数来显示字符串“Hello World！”，然后就返回。就像前面提到的，JNIEnv指针和obj对象引用都忽略了。

  这个C程序包含三个头文件：

- jni.h：此头文件提供本地代码调用JNI函数所需的信息。 编写本机方法时，必须始终将此文件包含在C或C源文件中。
- stdio.h：上面的代码片段还包括了stdio.h因为它使用printf函数
- HelloWorld.h：通过javah工具生成的头文件。它包含Java_HelloWorld_print的函数原型。

## 2.6 编译C源码并生成一个本地库

  请记住，你在文件HelloWorld.java文件中创建一个HelloWorld类时，包含一条将本地库加载到程序中的代码：

```
System.loadLibrary("HelloWorld");
```

  现在所有需要的C源码都已经编写完成了，现在你需要编译HelloWorld.c文件并创建一个本地库。

  不同的操作系统提供不同的方式去创建本地库。在Solaris上，下述命令能够创建一个名为libHelloWorld.so的动态库。

```
cc -G -I/java/include -I/java/include/solaris HelloWorld.c -o libHelloWorld.so
```

-G编译选项表明让C编译器生成一个动态库而不是常规的Solaris可执行文件。在win32系统中，下面的指令使用Microsoft Visual C++编译器创建的动态链接库（DLL）HelloWold.dll

```
cl -Ic:\java\include -Ic:\java\include\win32 -MD -LD HelloWorld.c -FeHelloWorld.dll
```

-MD编译选项表明HelloWorld.dll和win32多线程C库链接。-LD编译选项表明C编译器产生一个DLL文件而不是常规的Win32可执行文件。当然不管在Win32还是Solaris系统，你都需要在你的电脑上设置好头文件的包含路径。

**博主注：我使用的编译环境是Ubuntu 16.04版本，JDK版本是openjdk 1.8，使用上面的指令是不行的，下面是我使用的编译指令**

```
cc -I$JAVA_HOME/include -I$JAVA_HOME/include/linux -I. -fPIC -shared HelloWorld.c -o libHelloWorld.so
```

**当然你也可以使用gcc，其中，JAVA_HOME是我配置到.bashrc中的路径：**

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

**大家可以按照各自的JAVA_HOME进行配置，这样就能够编译成功了。**

## 2.7 运行程序

  到这里，运行该程序的两个主件都已经准备好了。类文件（HelloWor.class）调用一个本地方法，本地库（HelloWorld.dll）实现这个本地方法。因为HelloWorld类包含它自己的main方法，在Solaris和Win32上，可以通过如下方法执行这个程序：

```
java HelloWorld
```

你能够看到程序输出如下信息:

```
Hello World！
```

  为了让你的程序能够正确的运行，正确设置好本地库的路径是非常重要的。本地库路径是一系列文件目录，当Java虚拟机加载本地库时会搜索这些路径。如果你没有正确设置本地库路径，你会看到如下类似的错误log：

```
java.lang.UnsatisfiedLinkError: no HelloWorld in library path 
    at java.lang.Runtime.loadLibrary(Runtime.java)
    at java.lang.System.loadLibrary(System.java) 
    at HelloWorld.main(HelloWorld.java)
```

  需要确保本地库在设置的本地库路径的其中一个目录中。如果你在Solaris系统上运行，LD_LIBRARY_PATH环境变量是用来设置本地库路径的。确保该环境变量的路径包含动态库libHelloWorld.so文件所在的目录。如果libHelloWorld.so文件在当前目录，在标准shell或者KornShell中，你可以通过如下两条指令来设置LD_LIBRARY_PATH环境变量

```
LD_LIBRARY_PATH=. 
export LD_LIBRARY_PATH
```

在C Shell中的等价指令如下：

```
setenv LD_LIBRARY_PATH .
```

  如果你是在Windows 95或者Windows NT上运行，确保HelloWorld.dll在当前目录，或者其所在的目录已经列在PATH环境变量中。

  在Java 2 SDK 1.2发行版中，你可以像下面的指令一样，通过java命令行来指定本地库的的路径：

```
java -Djava.library.path=. HelloWorld
```

-D命令行选项设置了一个Java平台属性。将java.library.path设置为“.”，“.”表明Java虚拟机会在当前路径中搜索本地库。



# 第二章 内容补充

  在原文中，构建JNI程序的方法只介绍了一个，其实构建JNI程序还有另外一种方法，这种方法我们称之为动态注册，相对的之前的方法我们称之为静态注册。我们先将方法介绍后，再看一下这两种方法有什么区别。

  从一个例子出发，例子和第二章的例子一样，也是在Java中调用一个native方法打印出“Hello World！”的字样。Java侧的代码如下：

```
class HelloWorld {
    private native void helloworld();
    
    public static void main(String[] args) {
        HelloWorld h = new HelloWorld();
        h.helloworld();
    }
    
    static {
        System.loadLibrary("HelloWorld");
    }
}
```

  代码意思都是一样的，都是在HelloWorld类的main方法中创建一个对象，然后调用native层的方法HelloWorld来打印字符串。下面部分是native层的代码：

```
#include <stdio.h>
#include <jni.h>

void JNICALL native_helloworld(JNIEnv* env, jobject ojb)
{
    printf("Hello World!\n");
    return;
}

static const JNINativeMethod gMethods[] = {
    {"helloworld", "()V", (void *)native_helloworld}
};

static jclass myClass;
static const char* const ClassName="HelloWorld";
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) 
{
        JNIEnv* env = NULL;
        jint result = -1;
        if((*vm)->GetEnv(vm, (void**)&env, JNI_VERSION_1_6) != JNI_OK)
            return -1;
            
        myClass = (*env)->FindClass(env, ClassName);
        if(myClass == NULL)
        {
            printf("Cannot get class:%s\n", ClassName); 
            return -1;
        }
        
        if((*env)->RegisterNatives(env, myClass, gMethods, sizeof(gMethods)/sizeof(gMethods[0])) < 0)
        {
            printf("Register native methods failed.\n");
            return -1;
        }
        
        printf("--------JNI_OnLoad---------\n");
        return JNI_VERSION_1_6;
}
```

- 首先要介绍的是JNI_OnLoad方法，这个方法是自动调用的，在Java侧，我们在静态代码块内通过System.loadLibrary，加载动态库的时候，相应的动态库的JNI_OnLoad方法就会自动执行。
- 在JNI_OnLoad方法中，带有的参数是JavaVM而不是我们常见的JNIENV，所以需要向获取到JNIEnv对象，然后我们通过JNIEnv的FindClass方法找到我们需要动态注册的类。
- 最后我们通过RegisterNatives方法，将gMethods数组中的Java层的函数和native层的函数动态绑定起来。

  其中JNINativeMethod是一个结构体，定义如下：

```
typedef struct {
    const char* name;
    const char* signature;
    void* fnPtr;
} JNINativeMethod;
```

  name指的是Java中定义的方法， signature指的是方法的描述符，也可以叫做签名，“()V”中“()”表示的是方法的参数为void，“V”表示的是返回值为void，具体的定义在第四章会讲到，这里可以先跳过。fnPtr指向的是native的方法名。

  最后按照第二章中介绍的方法，编译运行就可以了。

  动态注册和静态注册相比，动态注册更加灵活而且简单，不像静态注册那样，还需要用java和javah编译后才得出最后的函数名，如果中间Java文件改动，添加了native方法或者修改了native方法的函数名，又要重新做上面的步骤，麻烦。看过Android源码的朋友都应该发现了，Android中的JNI相关内容几乎都是用动态注册方法的。



## 静态注册

Java侧代码如下：

```
class Information {
    private native void name();
    private native void addr();
    
    public static void main(String[] args) {
        Information in = new Information();
        in.name();
        in.addr();
    }
    
    static {
        System.loadLibrary("Information");
    }
}
```

然后通过指令javac Information.java编译出Information.class，通过指令javah -jni Information获取到Information.h文件，这个.h文件中就包含需要完成的native方法。

创建Information_static.c文件，代码如下:

```
#include <jni.h>
#include <stdio.h>
#include "Information.h"

JNIEXPORT void JNICALL Java_Information_name
  (JNIEnv * env, jobject obj)
{
    printf("My name is Jimmy Chen.\n");
}   

JNIEXPORT void JNICALL Java_Information_addr
  (JNIEnv * env, jobject obj)
{
    printf("My address is Guangdong Province.\n");
}
```

通过指令 `cc -I$JAVA_HOME/include -I$JAVA_HOME/include/linux -I. -fPIC -shared Information_static.c -o libInformation.so` 编译出.so库，最后通过指令`java -Djava.library.path=. Information` 运行该程序就能得到想要的结果了。

## 动态注册

首先，Java侧的代码不需要变动。

native侧的话，创建一个Information_dynamic.c文件，代码如下：

```
#include <jni.h>
#include <stdio.h>

void native_name(JNIEnv * env, jobject obj)
{
    printf("My name is Jimmy Chen.\n");
}

void native_addr(JNIEnv * env, jobject obj)
{
    printf("My addr is Guangdong Provice.\n");
}

const static JNINativeMethod gMethods[] = {
    "name", "()V", (void *)native_name,
    "addr", "()V", (void *)native_addr
};

static jclass myClass;
static const char * const ClassName = "Information";
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reversed)
{
    JNIEnv *env = NULL;
    jint result = -1;
    
    if((*vm)->GetEnv(vm, (void **)&env, JNI_VERSION_1_6) != JNI_OK)
        return -1;
        
    myClass = (*env)->FindClass(env, ClassName);
    if(myClass == NULL) {
        printf("Can not find class:%s.\n", ClassName);
        return -1;
    }
    
    if((*env)->RegisterNatives(env, myClass, gMethods, sizeof(gMethods)/sizeof(gMethods[0])) < 0) {
        printf("Register native mthods error.\n");
        return -1;
    }       
    
    printf("-----JNI_OnLoad Success-----\n");
    
    return JNI_VERSION_1_6; 
}
```

最后按照静态方法的编译和执行指令，编译运行即可，记住编译的时候将C文件名改为Information_dynamic.c文件就行了。



# 第三章 基本类型、字符串和数组

  当面对Java应用程序混合本地编程语言代码时，程序员经常会问的一个问题是：Java编程语言中的数据类型是如何映射到C/C++等本地编程语言中的数据类型的。上一章中介绍的“Hello World！”示例中，我们没有任何的参数需要传输给本地方法，本地方法也没有返回任何结果给调用者。本地方法只是简单的打印一条信息然后返回。

  在实践中，许多程序都需要传送参数给本地方法，而且需要从本地方法中获取返回值。本章，我们将介绍如何在Java编程语言编写的代码和本地编程语言编写的代码之间交换数据类型。我们从原始类型，例如整型和常用的对象类型，例如字符串和数组开始讲解。我们将把任意对象的完整处理留到下一章，下一章我们将会介绍本地方法的代码如何访问字段和进行方法调用。

## 3.1 一个简单的本地方法

  让我们先从一个简单的程序开始，这个程序和上一章的HelloWorld程序没有多少不同。示例程序，Prompt.java，包含一个打印一个字符串、等待用户输入、最后将用户输入的内容返回给调用函数的本地方法。该程序的代码如下：

```
class Prompt {
    // native method that prints a prompt and reads a line
    private native String getLine(String prompt);
    
    public static void main(String args[]) {
        Prompt p = new Prompt();
        String input = p.getLine("Type a line: ");  
        System.out.println("User typed: " + input);
    }   
    
    static {
        System.loadLibrary("Prompt");   
    }
}
```

  Prompt.main调用本地方法Prompt.getLine来获取用户的输入。在静态初始化块中调用System.loadLibrary方法将名为Prompt的本地库加载到程序中。

### 3.1.1 本地方法的C原型

  Prompt.getLine方法可以使用以下C函数实现：

```
JNIEXPORT jstring JNICALL Java_Prompt_getLine
  (JNIEnv *, jobject, jstring);
```

  你可以使用javah工具来生成包含上述函数原型的头文件。JNIEXPORT和JNICALL宏（在JNI.h都在文件中定义）确保这个函数从本地库中导出而且C编译器使用该函数的正确调用约定生成代码。C函数的名称是通过连接“Java_”前缀、类名称和方法名称构成的。11.3节包含如何形成C函数名称的更准确的描述。

### 3.1.2 本地方法参数

  如2.4节所述，本地方法实现（如Java_Prompt_getLine）除了在本地方法中声明的参数外，还接受两个标准参数。第一个参数是JNIEnv接口指针，指向函数表指针的位置。每个方法表中的指针指向一个JNI函数。本地方法始终通过JNI函数之一访问Java虚拟机中的数据结构。如图3.1所示JNIEnv接口指针：

![《(译文) JNI编程指南与规范 第三章 基本类型、字符串和数组》](https://i2.wp.com/blog4jimmy.com/wp-content/uploads/2017/11/JNI3.1.png)

  第二个参数取决于本地方法是实例方法静态方法还是实例方法。实例化本地方法的第二个参数是对该方法的调用对象的应用，与C++语言中的this指针类似。静态本地方法的第二个参数是对定义该方法的类的应用。我们的例子，Java_Prompt_getLine实现为一个实例化本地方法。所以第二个参数jobject就是对对象本身的引用。

### 3.1.3 类型映射

  本地方法声明中的参数类型在本地编程语言中都有相应的类型。JNI中定义了一组对应于Java编程语言中类型的C/C++类型。在Java编程语言中存在着两种类型：基本类型，如int、float和char以及引用类型，如类、实例和数组。在Java编程语言中字符串是java.lang.String类的实例化。

  JNI以不同的方式处理基本类型和引用类型。基本类型的映射是直接的。Java编程语言中的int类型映射为C/C++的jint（定义在jni.h中，为32位有符号整型数），Java编程语言中的float类型映射为C/C++的jfloat（定义在jni.h中，为32位浮点类型数）。12.1.1节包含所有在JNI中定义的基本类型（这里简单把截图放一下，如下图)。

![《(译文) JNI编程指南与规范 第三章 基本类型、字符串和数组》](https://i2.wp.com/blog4jimmy.com/wp-content/uploads/2017/11/table12.1..png)

  JNI传递对象给本地方法作为不透明引用。不透明引用指的是引用Java虚拟机内部数据类型的C指针类型。对于程序员而言，Java虚拟机内部数据的准确布局是隐藏的。本地代码可以通过JNIEnv接口指针指向的适当的函数来操作底层对象。例如，java.lang.String对应于JNI类型jstring，jstring引用的确切位置和本地代码是不相关的。本地代码通过jni函数，例如GetStringUTFChars来获取string的内容。

  所有的JNI类型都有类型jobject（感觉应该是jobject的子类的意思），为了方便和增强类型安全性，JNI定义了一组在概念上是jobject的子类型的引用类型（A是B的子类型，那么A的每个实例对象都会是B的实例对象（博主注：大概应该是这个意思吧））。这些子类型对应于Java编程语言中经常使用的引用类型。例如：jstring表示字符串，jobjectArray表示对象数据，12.1.2节(这里也简单把截图放一下，如下图)完整的列出了JNI参考类型及其子类型的关系。

![《(译文) JNI编程指南与规范 第三章 基本类型、字符串和数组》](https://i1.wp.com/blog4jimmy.com/wp-content/uploads/2017/11/table12.2.png)

## 3.2 访问字符串

  Java_Prompt_getLine接收prompt参数为一个jstring类型。jstring类型在Java虚拟机代表着字符串，但是有不同于常规的C字符串（指向字符的指针，char *）。你不能将jstring作为常规C字符串来使用。运行下面的代码将不会得到期望的结果，而事实上很可能会导致Java虚拟机崩溃。

```
JNIEXPORT jstring JNICALL
Java_Prompt_getLine(JNIEnv *env, jobject obj, jstring prompt) {
    /* ERROR: incorrect use of jstring as a char* pointer */ 
    printf("%s", prompt); 
    ...
}
```

### 3.2.1 转换为本地字符串

  你的本地代码必须使用恰当的JNI函数将jstring对象转换为C/C++字符串。JNI同时支持将jstring转换为Unicode和UTF-8字符串。Unicode字符串使用16位值表示字符，而UTF-8字符串则使用向上兼容7位ASCII字符串的编码方法。尽管UTF-8字符串包含非ASCII字符，但是其表现类似于使用NULL终止符的C字符串。值在1到127之间的所有7位ASCII字符在UTF-8字符串中编码保持不变。一个字节中的最高位被设置了，表示多字节编码的16位Unicode值的开始。

  Java_Prompt_getLine方法调用JNI方法GetStringUTFChars来读取字符串的内容。可以通过JNIEnv接口指针使用GetStringUTFChars方法。它将通常在Java虚拟机中实现为Unicode序列的jstring引用转换为UTF-8格式的C式字符串。如果你可以确定原始的字符串只包含7位ASCII字符，你可以将转换后的字符串传给C库函数，例如printf（我们会在8.2节讨论佮处理非ASCII字符串）。

```
#include "Prompt.h"
#include <stdio.h>

JNIEXPORT jstring JNICALL Java_Prompt_getLine
  (JNIEnv *env, jobject obj, jstring prompt)
{
    char buf[128];
    const jbyte *str;
    str = (*env)->GetStringUTFChars(env, prompt, NULL);
    if(str == NULL)
        return NULL;
    printf("%s", str);
    (*env)->ReleaseStringUTFChars(env, prompt, str);
    /* We assume here that the user does not type more than
     * 127 characters */
     scanf("%s", buf);
     return (*env)->NewStringUTF(env, buf);
}
```

  不要忘记检查GetStringUTFChars的返回值，这是因为Java虚拟机的实现决定内部需要申请内存来容纳UTF-8字符串，内存的申请是有可能会失败的。如果内存申请失败，那么GetStringUTFChars将会返回NULL并且会抛出OutOfMemoryError异常。正如我们会在第六章介绍的一样，在JNI中抛出异常和在Java中抛出异常是不同的。通过JNI抛出的挂起异常不会自动更改本地C代码的控制流程。相反我们需要一个显示的return语句来跳过C函数中的剩余语句。Java_Prompt_getLine返回后，该异常会返回给Prompt.getLine的调用者Prompt.main函数中。

### 3.2.2 释放本地字符串资源

  当你的本地代码使用完通过GetStringUTFChars获取的UTF-8字符串后，需要调用ReleaseStringUTFChars，调用ReleaseStringUTFChars表明本地代码不再需要GetStringUTFChars返回的UTF-8字符串了，调用ReleaseStringUTFChars就能够释放掉UTF-8字符串占用的内存。不调用ReleaseStringUTFChars进行内存释放会导致内存泄漏，最终导致内存耗尽。

### 3.2.3 创建新字符串

  通过调用JNI函数NewStringUTF，你可以在本地代码中创建一个新的java.lang.String实例。NewStringUTF方法使用一个UTF-8格式的C式字符串作为参数并生成一个java.lang.String实例对象。新构造的java.lang.String实例和给定的UTF-8 C式字符串有相同的Unicode序列。如果虚拟机没办法申请足够的内存来构造java.lang.String实例的话，NewStringUTF会抛出一个OutOfMemoryError异常并返回NULL。在这个例子中，我们不需要检查返回值，因为本地代码会在之后立刻返回。如果NewStringUTF调用失败，OutOfMemoryError异常会在该方法的调用者Prompt.main中被抛出。如果NewStringUTF调用成功，它会返回一个指向新构造的java.lang.String对象的引用。这个新构造的实例会在Prompt.getLine中返回，并在Prompt.main中赋值给input。

### 3.2.4 其他JNI字符串方法

  除了之前介绍的GetStringUTFChars、ReleaseStringUTFChars和NewStringUTF函数外，JNI中还支持其他的字符串相关方法。GetStringChars和ReleaseStringChars获取以Unicode格式表示的字符串字符。当操作系统支持将Unicode作为本地字符串格式的时候，这些函数将会非常有用。

  UTF-8字符串常以‘\0’结尾，而Unicode字符串却不是。为了统计一个jstring引用中的Unicode字符个数时，JNI程序员可以调用GetStringLength。为了统计一个UTF-8格式的jstring对象占用多少字节时，可以对GetStringUTFChars的返回值调用ANSI C函数strlen来获得，或者直接对jstring引用调用JNI函数GetStringUTFLength来获得。GetStringUTFChars和GetStringChars方法的第三个参数需要做些而外的解释：

`const jchar *GetStringChars(JNIEnv *env, jstring str, jboolean *isCopy);`

  从GetStringChars方法返时，如果返回的字符串是原始java.lang.String实例中的字符的副本，那么isCopy指向的内存地址的值被设置为JNI_TURE。如果返回的字符串是原始java.lang.String实例中字符的的直接引用，那么isCopy指向的内存地址的值被设置为JNI_FALSE。当isCopy指向的内存地址的值被设置为JNI_FALSE时，本地代码不能修改返回的字符串的内容。如果违反该规则，将会导致原始的java.lang.String实例对象也被修改。这将打破java.lang.String不可修改规则。

  大部分情况是将NULL作为isCopy的参数传递给方法，因为你不用关注Java虚拟机返回的是java.lang.String实例的副本还是直接引用。

  通常无法预测虚拟机是否会复制给定的java.lang.String中的字符。因此程序员必须了解到诸如GetStringChars之类的函数可能需要与java.lang.String实例中的字符数成比例的时间和空间开销。在典型的Java虚拟机实现中，垃圾收集器重新定位堆中的对象。一旦将指向java.lang.String实例的直接指针传回给本地代码中，垃圾收集器将不能重新定位java.lang.String实例。换句话说，虚拟机必须固定java.lang.String实例，因为过多的固定会导致内存碎片，所以虚拟机实现可以自由的选择为每个GetStringChars调用复制字符还是固定实例。

  当你不再需要访问GetStringChars函数返回的字符串元素时，不要忘记调用ReleaseStringChars。不管GetStringChars中的isCopy设置为JNI_FALSE还是JNI_TRUE，ReleaseStringChars都是必须调用的。ReleaseStringChars会释放副本或者取消固定实例，具体取决于GetStringChars返回实例的副本还是固定实例。

### 3.2.5 在Java 2 SDK 1.2中新添加的JNI字符串函数

  为了增加虚拟机返回java.lang.String实例字符的直接指针的可能性，Java 2 SDK版本1.2引入了一组新的Get/ReleaseStringCritical函数。 在表面上，它们似乎与Get/ReleaseStringChars函数类似，如果可能的话，它们返回一个指向字符的指针; 否则，会复制一份。 但是，如何使用这些功能有很大的限制。

  你必须将这对函数里的代码视为在临界区中运行，在临界区内，本地代码不能随意（arbitrary，博主这里翻译为随意的，但是感觉有点不对，但是不知道怎么翻译好，独占？感觉跟Linux内核中断处理有点像）调用JNI函数或者其他任意的会引起当前线程阻塞以及会等待Java虚拟机中另一个线程的本地函数。例如，当前线程不能够等待另一个线程的I/O输入流。这些限制使得虚拟机可以在本地代码持有通过GetStringCritical获取的字符串元素的直接指针时禁用垃圾回收。当垃圾收集器被禁用，其他触发垃圾收集器的线程都会被挂起。在Get/ReleaseStringCritical对之间的本地代码不能调用回引起阻塞的调用以及创建新对象。否则虚拟机可能会引起死锁，考虑如下场景：

- 由另一个线程触发的垃圾回收无法进行，直到当前线程完成阻塞调用并重新启用垃圾回收。
- 同时，当前的线程无法进行，因为阻塞调用需要获得一个已经被另一个正在等待执行垃圾回收的线程持有的锁。

  重叠调用多对GetStringCritical和ReleaseStringCritical是安全的。例如：

```
jchar *s1, *s2;
s1 = (*env)->GetStringCritical(env, jstr1); if (s1 == NULL) {
... /* error handling */ }
s2 = (*env)->GetStringCritical(env, jstr2); if (s2 == NULL) {
(*env)->ReleaseStringCritical(env, jstr1, s1); ... /* error handling */
} ... /* use s1 and s2 */
(*env)->ReleaseStringCritical(env, jstr1, s1); (*env)->ReleaseStringCritical(env, jstr2, s2);
```

  Get/ReleaseStringCritical对的使用不需要以堆栈顺序严格嵌套。我们不能忘记检查其因内存不足导致返回值为NULL的情况，因为GetStringCritical仍然可以分配一个缓冲区，并且如果虚拟机内部表示不同格式的数组，则复制数组。例如，Java虚拟机可能不会连续存储数组。在这种情况下，GetStringCritical必须复制jstring实例中的所有字符，以便将本地代码的连续字符数组返回。

  为了避免死锁，你应该确保你的本地代码在调用GetStringCritical之后和在调用ReleaseStringCritical之前不应当随意调用JNI函数，在临界区中唯一允许的JNI函数是嵌套调用Get/ReleaseStringCritical和Get/ReleasePrimitiveArrayCritical。

  JNI不支持GetStringUTFCritical和ReleaseStringUTFCritical方法。这些函数几乎都需要虚拟机创建字符串的副本，这是因为虚拟机实现几乎在内部都是使用Unicode格式来存储字符串。

  另外在Java SDK 2 Release 1.2中增加的函数是GetStringRegion和GetStringUTFRegion。这些函数将字符串元素复制到一个预先分配的内存当中。Prompt.getLine方法可以使用GetStringUTFRegion进行重写：

```
JNIEXPORT jstring JNICALL
Java_Prompt_getLine(JNIEnv *env, jobject obj, jstring prompt) {
    /* assume the prompt string and user input has less than 128 characters */
    char outbuf[128], inbuf[128];
    int len = (*env)->GetStringLength(env, prompt); 
    (*env)->GetStringUTFRegion(env, prompt, 0, len, outbuf); 
    printf("%s", outbuf); 
    scanf("%s", inbuf);
    return (*env)->NewStringUTF(env, inbuf); 
}
```

  GetStringUTFRegion将字符串开始的下标和长度作为参数，这两个值都是以Unicode字符来统计。这个函数同时做边界检查，同时有必要会抛出StringIndexOutOfBoundsExecption异常。上面的代码中，我们从字符串引用本身获取到长度，因此可以确认不会出现下标越界（但是上面的代码缺少对prompt的检查，以确保其长度低于128个字符）。

### 3.2.6 JNI字符串函数总结

 &esmp;表3.1中列出所有字符串相关的JNI函数，Java 2 SDK 1.2版本增加了一些增强某些字符串操作性能的新功能。 除了提高性能之外，增加的功能不支持新的操作。

**表3.1 JNI字符串函数总结**

| JNI函数                                 | 描述                                                         | 从哪个版本开始 |
| --------------------------------------- | ------------------------------------------------------------ | -------------- |
| GetStringChars\ReleaseStringChars       | 获取或释放指向Unicode格式的字符串内容的指针。可能会返回字符串的副本。 | JDK 1.1        |
| GetStringUTFChars\ReleaseStringUTFChars | 获取或释放指向UTF-8格式的字符串内容指针。可能会返回字符串的副本 | JDK 1.1        |
| GetStringLength                         | 返回字符串中Unicode字符的数量                                | JDK 1.1        |
| GetStringUTFLength                      | 返回以UTF-8格式表示字符串所需的字节数（不包括尾数0）。       | JDK 1.1        |
| NewString                               | 创建拥有和给定的Unicode格式C式字符串相同字符序列的java.lang.String实例 | JDK 1.1        |
| NewStringUTF                            | 创建拥有和给定的UTF-8格式C式字符串相同字符序列的java.lang.String实例 | JDK 1.1        |
| GetStringCritical\ReleaseStringCritical | 获取指向Unicode格式的字符串内容的指针。 可能会返回字符串的副本。本地代码不能在Get/ReleaseStringCritical调用中间阻塞 | Java 2 SDK 1.2 |
| GetStringRegion\SetStringRegion         | 以Unicode格式将字符串的内容复制到预分配的C缓冲器到或从预分配的C缓冲区中复制。 | Java 2 JDK 1.2 |
| GetStringUTFRegion\SetStringUTFRegion   | 以UTF-8格式将字符串的内容复制到预分配的C缓冲区中或从预分配的C缓冲区中复制 | Java 2 JDK 1.2 |

### 3.2.7 选择合适的字符串函数

  图3.2中表明程序员应该如何在JDK release 1.1和Java 2 SDK release 1.2中选择合适字符串相关函数。

![《(译文) JNI编程指南与规范 第三章 基本类型、字符串和数组》](https://i0.wp.com/blog4jimmy.com/wp-content/uploads/2017/11/JNI3.2.png)

  如果你使用的是1.1或者1.1和1.2发行版的JDK，那么除了Get/ReleaseStringChars和Get/ReleaseStringUTFChars外没有其余的选择了。

  如果你是使用Java 2 JDK release 1.2及以上的版本进行编程，并且想字符串的内容复制到已经分配了C缓冲区中，使用GetStringRegion或者GetStringUTFRegion。

  对于小型固定大小的字符串，Get/SetStringRegion和Get/SetStringUTFRegion几乎总是首选函数，因为C缓冲区在C堆栈上进行分配是开销是非常小的。在字符串中复制少量字符的开销是微不足道的。

  Get/SetStringRegion和Get/SetStringUTFRegion的一个优点是它们不执行内存分配，因此不会引起意外的内存不足异常。 如果确保不能发生索引溢出，则不需要进行异常检查。Get/SetStringRegion和Get/SetStringUTFRegion的另一个优点是您可以指定起始索引和字符数。 如果本地代码仅需要访问长字符串中的字符子集，那么这些函数是合适的。

  使用GetStringCritical函数必须非常小心。你必须确保当持有一个通过GetStringCritical返回的指针时，本地代码在Java虚拟机中不会创建新对象或者会引起系统死锁的阻塞性调用。

  下面是一个实例，演示了使用GetStringCritical产生的微妙问题。下面的代码获取字符串的内容，并调用fprintf函数将字符写入到文件句柄fd中：

```
/* This is not safe! */
const char *c_str = (*env)->GetStringCritical(env, j_str, 0); 
if (c_str == NULL) {
.../* error handling */ 
}
fprintf(fd, "%s\n", c_str);
(*env)->ReleaseStringCritical(env, j_str, c_str);
```

  上述代码的问题是当当前线程禁用垃圾收集时，写入文件句柄并不总是安全的。假设，例如，另一个线程T等待从fd文件句柄读取。 让我们进一步假设操作系统缓冲的设置方式使得fprintf调用等待，直到线程T完成从fd读取所有挂起的数据。我们已经构建了可能的死锁场景：如果线程T不能分配足够的内存 作为从文件句柄读取的缓冲区，它必须请求垃圾回收。 垃圾回收请求将被阻止，直到当前线程执行ReleaseStringCritical，直到fprintf调用返回为止。 然而，fprintf调用正在等待线程T从文件句柄中完成读取。

  以下代码虽然与上述示例类似，但几乎肯定是无死锁的：

```
/* This code segment is OK. */
const char *c_str = (*env)->GetStringCritical(env, j_str, 0); 
if (c_str == NULL) { 
... /* error handling */
} 
DrawString(c_str); 
(*env)->ReleaseStringCritical(env, j_str, c_str);
```

  DrawString是一个能直接将字符串写到屏幕上的系统调用。除非屏幕显示驱动程序也是在同一虚拟机中运行的Java应用程序，否则DrawString函数将不会无限期地阻止等待垃圾收集发生。

  总而言之，您需要考虑一对Get/ReleaseStringCritical调用之间所有可能的阻塞行为。

## 3.3 访问数组

  JNI以不同的方式对待基本数据类型数组和对象数组。基本数据类型数组包含基本数据类型，例如int和boolean。对象数据包含引用类型元素，例如类实例或其他数组。例如下面使用Java编程语言编写的代码中：

```
int[] iarr;
float[] farr; 
Object[] oarr; 
int[][] arr2;
```

iarr和farr是基本数据类型数组，而oarr和arr2是对象数组。

  在本地代码中访问基本数据类型数组所需要的方法和访问字符串所需要的本地方法类似。让我们看一个基本例子，以下程序调用本地方法sumArray，它将int数组的内容相加。

```
class IntArray {
    private native int sumArray(int[] arr); 
    public static void main(String[] args) { 
        IntArray p = new IntArray(); 
        int arr[] = new int[10]; 
        for (int i = 0; i < 10; i++) { 
            arr[i] = i;
        }
        int sum = p.sumArray(arr); 
        System.out.println("sum = " + sum);
    } 
    static { 
        System.loadLibrary("IntArray"); 
    } 
}
```

### 3.3.1 在C中访问数组

  数据由jarray引用类型及其“子类型”（如jintArray）表示。正如jstring不是C式字符串一样，jarray也不是C式数据。你不能直接访问jarray引用来完成Java_IntArray_sumArray本地方法的编写。下面的C代码是非法的也不会获取到想要的结果：

```
/* This program is illegal! */ 
JNIEXPORT jint JNICALL Java_IntArray_sumArray(JNIEnv *env, jobject obj, jintArray arr) {
    int i, sum = 0;
    for (i = 0; i < 10; i++) { 
        sum += arr[i];
    } 
}
```

你应该使用恰当的JNI函数来访问基本数据类型数组中的元素，就像下面展示的正确的例子一样：

```
JNIEXPORT jint JNICALL Java_IntArray_sumArray(JNIEnv *env, jobject obj, jintArray arr) {
    jint buf[10]; 
    jint i, sum = 0;
    (*env)->GetIntArrayRegion(env, arr, 0, 10, buf); 
    for (i = 0; i < 10; i++) { 
        sum += buf[i];
    } 
    return sum; 
}
```

### 3.3.2 访问基本数据类型数组

  前面的例子中使用GetIntArrayRegion函数来复制整型数组中的所有元素到C缓冲区中。第三个参数是需要复制的元素的起始索引，第四个参数表示需要复制的元素的总数。一旦元素存储在C缓冲区中，我们就能够在本地代码中访问他们了。异常检查是不需要的，因为在这个例子中，我们知道数组的长度为10，因此不会引发索引越界问题。

  JNI支持相应的SetIntArrayRegion函数，该函数允许本机代码修改int类型的数组元素。 还支持其他原始类型的数组（如boolean、short和float类型）。

  JNI支持一系列Get/Release<Type>ArrayElements（博主注<Type> 表示的是基本类型，例如int、float等，因为博客Markdown解析不好，实在没办法弄好，各位看官就将就看了，下同）函数（包括例如Get/ReleaseIntArrayElements），允许本机代码获得对原始数组元素的直接指针。因为底层垃圾收集器不支持固定，所以虚拟机可能会返回指向基本数据类型数组的副本的指针。我们可以使用GetIntArrayElements来重写3.3.1节中的本地代码实现函数（包括例如Get/ReleaseIntArrayElements），允许本机代码获得对原始数组元素的直接指针。因为底层垃圾收集器不支持固定，所以虚拟机可能会返回指向基本数据类型数组的副本的指针。我们可以使用GetIntArrayElements来重写3.3.1节中的本地代码实现：

```
JNIEXPORT jint JNICALL Java_IntArray_sumArray(JNIEnv *env, jobject obj, jintArray arr) {
    jint *carr; 
    jint i, sum = 0;
    carr = (*env)->GetIntArrayElements(env, arr, NULL); 
    if (carr == NULL) {
        return 0; /* exception occurred */ 
    }
    for (i=0; i<10; i++) { 
        sum += carr[i];
    }
    (*env)->ReleaseIntArrayElements(env, arr, carr, 0); 
    return sum;
}
```

  GetArrayLength方法返回基本数据类型数组或对象数组中元素的个数。当第一次分配数组的时候，其长度就固定了。

  Java 2 SDK 1.2中介绍了Get/ReleasePrimitiveArrayCritical函数。当本地代码访问基本数据类型数组的时候，这些函数允许虚拟机禁用垃圾回收器。程序员注意使用这两个函数必须跟使用Get/ReleaseStringCritical函数一样小心。在Get/ReleasePrimitiveArrayCritical函数对中的本地代码不能随意调用JNI方法，不能进行可能导致死锁的阻塞操作。

### 3.3.3 访问基本数据类型数组总结

  表3.2中列出了访问基本数据类型数据的相关JNI方法，Java 2 JDK 1.2版本中增加了一些增加特定数组操作性能的函数，增加的函数没有提供新的操作，只是做了操作性能的提升而已：

**表3.2 访问基本数据类型数组总结**

| JNI函数                                                | 描述                                                         | 从哪个版本开始 |
| ------------------------------------------------------ | ------------------------------------------------------------ | -------------- |
| Get<Type>ArrayRegion\Set<Type>ArrayRegion              | 复制基本数据类型数组的内容到C缓冲区或者将C缓冲区的内容复制出来 | JDK 1.1        |
| Get<Type>ArrayElements\Release<Type>ArrayElements      | 获取一个指向基本数据类型数组内容的指针，可能会返回该数组的副本 | JDK 1.1        |
| GetArrayLength                                         | 返回数组中元素的个数                                         | JDK 1.1        |
| New<Type>Array                                         | 创建一个给定长度的数组                                       | JDK 1.1        |
| GetPrimitiveArrayCriticalReleasePrimitiveArrayCritical | 获取一个指向基本数据类型数组内容的指针，可能禁用垃圾收集器或者返回该数组的副本 | Java 2 JDK 1.2 |

### 3.3.4 选择合适的基本类型数组函数

  图3.3表明，在JDK 1.1和Java 2 JDK 1.2版本中，程序员应如何选择恰当的JNI函数来访问基本数据类型数组。

![《(译文) JNI编程指南与规范 第三章 基本类型、字符串和数组》](https://i0.wp.com/blog4jimmy.com/wp-content/uploads/2017/11/JNI3.3.png)

  如果你需要将数组内容复制到C缓冲区或者从C缓冲区中将内容复制到数组中，应当使用Get/Set<Type>ArrayRegion家族函数。这些函数会进行边界检查，并且如果有必要的话会抛出ArrayIndexOutOfBoundsException异常。第3.3.1节中的本地方法实现中使用GetIntArrayRegion方法从jarray引用中复制10个元素。

  对于小型固定大小的阵列，Get/Set<Type>ArrayRegion几乎总是首选函数，因为C缓冲区可以非常方便的从C堆栈中分配。复制少量数组元素的开销是微不足道的。

  Get/Set<Type>ArrayRegion函数允许您指定起始索引和元素数量，因此如果本地代码只需要访问大型数组中的元素的一个子集，则它们是首选函数。

  如果没有预分配的C缓冲区，则原始数组的大小不确定，并且本机代码在持有指向数组元素的指针时不发出阻塞调用，请使用Java 2 SDK版本1.2中的Get/ReleasePrimitiveArrayCritical函数。 就像Get/ReleaseStringCritical函数一样，必须非常小心地使用Get/ReleasePrimitiveArrayCritical函数，以避免死锁。

  使用Get/Release<Type>ArrayElements系列函数总是安全的。 虚拟机或者返回指向数组元素的直接指针，或者返回一个保存数组元素副本的缓冲区。

### 3.3.5 访问对象数组

  JNI提供了一对单独的函数来访问对象数组。GetObjectArrayElement返回给定索引处的元素，而SetObjectArrayElement更新给定索引处的元素。与原始数组类型的情况不同，您不能一次获取所有对象元素或复制多个对象元素。字符串和数组是引用类型，你可以使用Get/SetObjectArrayElememt访问字符串数组和数组的数组。

  下面的代码调用一个本地函数来创建一个int型二维数组，然后打印给数组的内容。

```
class ObjectArrayTest {
    private static native int[][] initInt2DArray(int size); 
    public static void main(String[] args) { 
        int[][] i2arr = initInt2DArray(3); 
        for (int i = 0; i < 3; i++) { 
            for (int j = 0; j < 3; j++) {
                System.out.print(" " + i2arr[i][j]); 
            } 
            System.out.println(); 
        } 
    } 

    static { 
        System.loadLibrary("ObjectArrayTest"); 
    } 
}
```

本地方法initInt2DArray根据给定的大小创建一个二维数组，该本地方法分配和创建二维数组的代码可能如下所示：

```
JNIEXPORT jobjectArray JNICALL
Java_ObjectArrayTest_initInt2DArray(JNIEnv *env, jclass cls, int size)
{
    jobjectArray result; 
    int i;
    jclass intArrCls = (*env)->FindClass(env, "[I"); 
    if (intArrCls == NULL) {
        return NULL; /* exception thrown */ 
    }
    result = (*env)->NewObjectArray(env, size, intArrCls, NULL);
    if (result == NULL) { 
        return NULL; /* out of memory error thrown */ 
    }
    for (i = 0; i < size; i++) { 
        jint tmp[256]; /* make sure it is large enough! */ 
        int j;
        jintArray iarr = (*env)->NewIntArray(env, size); 
        if (iarr == NULL) {
            return NULL; /* out of memory error thrown */ 
        }
        for (j = 0; j < size; j++) { 
            tmp[j] = i + j;
        }
        (*env)->SetIntArrayRegion(env, iarr, 0, size, tmp); 
        (*env)->SetObjectArrayElement(env, result, i, iarr); 
        (*env)->DeleteLocalRef(env, iarr);
    } 
    return result; 
}
```

  initInt2DArray方法首先调用JNI函数FindClass来获取一个对二维int类型数组的元素类的引用。FindClass的参数“[I”是一个对于与Java编程语言中int[]类型的JNI类描述符（12.3.2节）。如果类型查询失败，FindClass会返回NULL并抛出异常（例如由于缺少类文件或者内存不足的情况）。

  下一步，NewObjectArray函数分配一个数组，其元素类型由intArrCls类应用决定。NewObjectArray仅仅分配第一个维度，我们仍然需要填写构成第二个维度的数组元素。Java虚拟机中没有特殊的数据类型来表示多维数组。一个二维数组其实就是一个数组。

  创建第二维数组的代码是简单易懂的。NewIntArray分配独立的数组元素，SetIntArrayRegion将tmp缓冲区的内容复制到新分配的一维数组中。完成SetObjectArrayElement调用后，第i个一维数组的第j个元素的值为i+j。执行ObjectArrayTest.main方法可以获得如下输出：

```
0   1   2
1   2   3
2   3   4
```

在循环结尾调用DeleteLocalRef确保虚拟机不会因为保存JNI引用例如iarr，而导致内存耗尽。5.2.1节将解释为什么以及何时需要调用DeleteLocalRef。



# 第三章之刻意练习

## Practice 1

  在Java侧初始化两条提示语句，一个提示输入姓名，另一个提示输入住址，然后编写一个native方法，将其中的提示语句传给native方法，然后再native方法中获取输入，将输入的内容返回给Java侧，在Java侧打印native方法中输入的内容。（当然有时间的朋友可以尝试使用静态注册JNI方法和动态注册JNI方法这两种方式）

### 静态注册方式

** 1.1 Java侧代码： **

```
class Practice1 {   
    private native String getInformation(String prompt);
    
    public static void main(String[] args) {
        String NamePrompt = "Please enter your name:";
      String AddrPrompt = "Please enter your addr:";
        
        Practice1 p = new Practice1();
        String Name = p.getInformation(NamePrompt);
        String Addr = p.getInformation(AddrPrompt);
        
        System.out.println("Name: " + Name);
        System.out.println("Addr: " + Addr);
    }
    
    static {
        System.loadLibrary("Practice1");
    }
}
```

** 1.2 javah -jni产生的头文件 **

```
/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class Practice1 */

#ifndef _Included_Practice1
#define _Included_Practice1
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     Practice1
 * Method:    getInformation
 * Signature: (Ljava/lang/String;)Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_Practice1_getInformation
  (JNIEnv *, jobject, jstring);

#ifdef __cplusplus
}
#endif
#endif
```

** 1.3 静态注册方法编写 **

```
#include <jni.h>
#include <stdio.h>
#include <string.h>

#include "Practice1.h"

JNIEXPORT jstring JNICALL Java_Practice1_getInformation
  (JNIEnv * env, jobject obj, jstring prompt)
{
    // 下面这里用jbyte * 和 char *都是可以的
    char * c_prompt;
    char input[128];
    c_prompt = (*env)->GetStringUTFChars(env, prompt, NULL);
    if(c_prompt == NULL) {
        printf("GetStringUTFChars return NULL.\n");
        return NULL;
    }
    
    printf("%s", c_prompt);
    // 这里记得释放字符串占用的空间
    (*env)->ReleaseStringUTFChars(env, prompt, c_prompt);
//  scanf("%s", input);
    gets(input);
    
    // 返回创建的String对象
    return (*env)->NewStringUTF(env, input);
}
```

### 动态注册方式

  动态注册JNI方法，签名的Java文件不用改，需要改的是Native侧的实现部分，下面是修改后的c文件

```
#include <jni.h>
#include <stdio.h>

JNIEXPORT jstring JNICALL native_getInformation(JNIEnv * env, jobject obj, jstring prompt)
{
    jbyte * c_prompt;
    jbyte input[128];
    
    c_prompt = (*env)->GetStringUTFChars(env, prompt, NULL);
    if (c_prompt == NULL) {
        printf("GetStringUTFChars return NULL.\n");
        return NULL;
    }
    
    printf("%s", c_prompt);
    (*env)->ReleaseStringUTFChars(env, prompt, c_prompt);
    gets(input);
    
    return (*env)->NewStringUTF(env, input);
}

const static JNINativeMethod gMethods[] = {
    "getInformation", "(Ljava/lang/String;)Ljava/lang/String;", (void *)native_getInformation
};

static jclass myClass;
static const char * ClassName = "Practice1";
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM * vm, void * reversed)
{
    JNIEnv * env = NULL;
    jint result = -1;
    
    if((*vm)->GetEnv(vm, (void **)&env, JNI_VERSION_1_6) != JNI_OK)
        return -1;
        
    myClass = (*env)->FindClass(env, ClassName);
    if(myClass == NULL) {
        printf("FindClass return NULL.\n");
        return -1;
    }
    
    if((*env)->RegisterNatives(env, myClass, gMethods, sizeof(gMethods) / sizeof(gMethods[0])) < 0) {
        printf("RegisterNatives return error.\n");
        return -1;
    }
    
    printf("-----JNI_OnLoad Success-----\n");
    
    return JNI_VERSION_1_6;
}
```

## Practice 2

现在玩点特别的，现在我们设计一种简单的字符串加密算法，实际的算法部分我们都在native层实现，这样就可以通过编译成动态库（应该不会那么容易被破解查看里面的代码吧？），将算法部分保存起来，达到保护的作用。具体的设置想法如下：在Java侧处理字符串的输入，然后我们将字符串传给native层处理，native层的算法我们设计得简单点咯，第一个字符加1,、第二个字符加2、第三个字符加3、依次类推。字符串解密算法就是加密算法的逆过程，肯定有很多不严谨的地方，所以仅供娱乐练习JNI

** 2.1 Java侧代码 **

```
import java.util.Scanner;

class Practice2 {
    private native String string_encode(String input);
    private native String string_decode(String input);
    
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        Practice2 p = new Practice2();
        
        System.out.println("Enter the string you want to encode: ");
        String need_encode_string = sc.nextLine();
        String encode_string = p.string_encode(need_encode_string);
        System.out.println("After encoded, the string is " + encode_string);
        
        System.out.println("Enter the string you want to decode: ");
        String need_decode_string = sc.nextLine();
        String decode_string = p.string_decode(need_decode_string);
        System.out.println("After decoded, the string is " + decode_string);
    }
    
    static {
        System.loadLibrary("Practice2");
    }
}
```

** 2.2 Native侧代码 **

```
#include <jni.h>
#include <stdio.h>
#include <stdlib.h>

JNIEXPORT jstring JNICALL native_encode_string(JNIEnv * env, jobject obj, jstring input)
{
    // 直接使用JNI函数获取字符串长度
    jsize input_length = (*env)->GetStringUTFLength(env, input);
    printf("input_length = %d\n", input_length);
    // 分配C缓冲区，这里需要是char *类型，曾试过使用jchar *类型分配，到调用free的时候会出现segment fault
    char * input_buf = (char *)malloc(input_length + 1);
    if(input_buf == NULL) {
        printf("malloc buffer return error.\n");
        return NULL;
    }
    
    // 将字符串内容复制到C缓冲区中
    (*env)->GetStringUTFRegion(env, input, 0, input_length, input_buf);
    // 按照算法处理
    for(int i = 0; i < input_length; i++) {
        input_buf[i] += i;
    }
    input_buf[input_length] = '\0';
    
    // 将加密后的内容生成新的字符串
    jstring result = (*env)->NewStringUTF(env, (const char *)input_buf);
    free(input_buf);
    if(result == NULL) {
        printf("NewStringUTF return error.\n");
        return NULL;
    }
    else
        return result;
}

JNIEXPORT jstring JNICALL native_decode_string(JNIEnv * env, jobject obj, jstring input)
{
    jsize input_length = (*env)->GetStringUTFLength(env, input);
    char * input_buf = (char *)malloc(input_length + 1);
    if(input_buf == NULL) {
        printf("malloc buffer return error.\n");
        return NULL;
    }   
    
    (*env)->GetStringUTFRegion(env, input, 0, input_length, input_buf);
    // 按照算法解密
    for(int i = 0; i < input_length; i++) {
        input_buf[i] -= i;
    }
    input_buf[input_length] = '\0';
    
    jstring result = (*env)->NewStringUTF(env, (const char *)input_buf);
    free(input_buf);
    if(result == NULL) {
        printf("NewStringUTF return error.\n");
        return NULL;
    }
    else
        return result;
}

const static JNINativeMethod gMethods[] = {
    {"string_encode", "(Ljava/lang/String;)Ljava/lang/String;", native_encode_string},
    {"string_decode", "(Ljava/lang/String;)Ljava/lang/String;", native_decode_string}
};


static jclass myClass;
static const char * ClassName = "Practice2";
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM * vm, void * reversed)
{
    JNIEnv * env = NULL;
    
    if((*vm)->GetEnv(vm, (void **)&env, JNI_VERSION_1_6) != JNI_OK) {
        printf("GetEnv return error.\n");
        return -1;
    }
    
    myClass = (*env)->FindClass(env, ClassName);
    if(myClass == NULL) {
        printf("FindClass return error.\n");
        return -1;
    }
    
    if((*env)->RegisterNatives(env, myClass, gMethods, sizeof(gMethods)/sizeof(gMethods[0])) < 0) {
        printf("RegisterNatives return error.\n");
        return -1;
    }
    
    return JNI_VERSION_1_6;
}
```

  最后编译运行就可以了。

## Practice 3

下面测试下时间性能，在Java侧编写一个冒泡排序算法，在native也编写一个冒泡排序算法，比较这两个时间性能，这里不能仅仅侧native的时间性能，还应该包括调用JNI的时间。一样仅供JNI编程练习，别太较真。

** 3.1 Java侧代码 **

```
import java.util.Random;

class Practice3 {
private native void bubble_sort(int[] iarray, int arr_length);
private static final int ARRAY_SIZE = 100000;

private void BubbleSort(int[] iarray, int arr_length) {

    int temp;
    for (int n=0; n<arr_length; n++) {
        for (int m=n+1; m<arr_length; m++) {
            if(iarray[n] > iarray[m]) {
                temp = iarray[n];
                iarray[n] = iarray[m];
                iarray[m] = temp;
            }
        }
    }
}


public static void main(String args[]) {
    int max = 10000;
    int[] OriArray = new int[ARRAY_SIZE];
    int[] NativeArray = new int[ARRAY_SIZE];
    int[] JavaArray = new int[ARRAY_SIZE];
    Random random = new Random();
    Practice3 p = new Practice3();

    System.out.println("General Array:");
    for(int i = 0; i < ARRAY_SIZE; i++) {
        int s = random.nextInt(max);
        OriArray[i] = s;
        NativeArray[i] = s;
        JavaArray[i] = s;
    }

    System.out.println("Java Bubble test:");
    long JavaStart = System.currentTimeMillis();
    p.BubbleSort(JavaArray, ARRAY_SIZE);
    long JavaEnd = System.currentTimeMillis();
    System.out.println("Java bubble sort need " + ((JavaEnd - JavaStart) / 1000.0) + " seconds");

    System.out.println("Native Bubble test:");
    long NativeStart = System.currentTimeMillis();
    p.bubble_sort(NativeArray, ARRAY_SIZE);
    long NativeEnd = System.currentTimeMillis();
    System.out.println("Native bubble sort need " + ((NativeEnd - NativeStart) / 1000.0) + " seconds");

    System.out.println("Java sorted array: ");
    for (int i=0; i<30; i++) {
        System.out.print(JavaArray[i]);
        if((i + 1) % 10 == 0)
            System.out.println();
        else
            System.out.print(" ");
    }

    System.out.println("Native sorted array: ");
    for (int i=0; i<30; i++) {
        System.out.print(NativeArray[i]);
        if((i + 1) % 10 == 0)
            System.out.println();
        else
            System.out.print(" ");
    }
}

static {
    System.loadLibrary("Practice3");
}
}
```

** 3.2 Native侧代码 **

```
#include <jni.h>
#include <stdio.h>

JNIEXPORT void JNICALL native_bubble_sort(JNIEnv * env, jobject obj, jintArray array, jint size)
{
    // 在C空间中获取数组内容
    jint * iArray = (*env)->GetIntArrayElements(env, array, NULL);
    jint temp = 0;
    // 对获取到的数组内容进行排序
    for(int i = 0; i < size; i++)
        for(int j = i+1; j < size; j++) {
            if(iArray[i] > iArray[j]) {
                temp = iArray[i];
                iArray[i] = iArray[j];
                iArray[j] = temp;
            }
        }
    
    // 对排序后的数组内容写回到Java虚拟机中
    (*env)->SetIntArrayRegion(env, array, 0, size, iArray);
    
    // 释放资源
    (*env)->ReleaseIntArrayElements(env, array, iArray, 0);
    
    return;
}

const static JNINativeMethod gMethods[] = {
    {"bubble_sort", "([II)V", native_bubble_sort}
};


static jclass myClass;
static const char * ClassName = "Practice3";
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM * vm, void * reversed)
{
    JNIEnv * env = NULL;
    
    if((*vm)->GetEnv(vm, (void **)&env, JNI_VERSION_1_6) != JNI_OK) {
        printf("GetEnv return error.\n");
        return -1;
    }
    
    myClass = (*env)->FindClass(env, ClassName);
    if(myClass == NULL) {
        printf("FindClass return error.\n");
        return -1;
    }
    
    if((*env)->RegisterNatives(env, myClass, gMethods, sizeof(gMethods)/sizeof(gMethods[0])) < 0) {
        printf("RegisterNatives return error.\n");
        return -1;
    }
    
    return JNI_VERSION_1_6;
}
```

  最后发现，上面的代码，在Java侧做冒泡排序，I5-3320M的CPU，需要15.几秒，而native侧就要18.几秒，一方面Java调用native方法比Java调用Java方法要耗时，其次从虚拟机中复制数组数据到C缓冲区中也要时间。所以怎么看好像在native侧做排序都没有占到好处，不过也可能是native方法编写得不够有效率，不过目前就先这样了，毕竟才重新开始学JNI，有很多地方还不够熟悉的，后续有机会找到好方法再改进。



# 第四章 字段和方法

  现在你已经知道了JNI是如何让本地代码访问基本数据类型和引用类型，例如字符串和数组，下一步需要学习怎么样和任意对象的字段和方法进行交互。除了访问字段外，这里还包括在本地代码中调用使用Java编程语言编写的方法，这通常称为从本地代码执行回调。

  我们将首先介绍支持字段访问和方法回调的JNI函数。本章的后面部分我们会讨论通过简单但是有效的缓存技术使这些操作更加有效率。本章最后部分，我们会讨论调用本地方法和从本地方法中访问字段以及执行回调的性能特性。

## 4.1 访问字段

  Java编程语言支持两种类型的字段。类的每个实例对象都有该类实例字段的单独副本，而类的所有实例都贡献该类的静态字段。JNI提供方法使得本地代码能够获取或者设置对象中的实例字段和类中的静态字段。让我们首先看一个例子程序，看该例子是如何本地代码实现是如何访问实例字段的。

```
class InstanceFieldAccess { 
    private String s;
    private native void accessField(); 
    public static void main(String args[]) { 
        InstanceFieldAccess c = new InstanceFieldAccess(); 
        c.s = "abc"; 
        c.accessField();
        System.out.println("In Java:"); 
        System.out.println(" c.s = \"" + c.s + "\"");
    } 
    static { 
        System.loadLibrary("InstanceFieldAccess"); 
    } 
}
```

  InstanceFiledAccess类定义了一个实例字段s，main方法中创建类一个该类的对象，设置实例字段，然后调用本地方法InstanceFiledAccess.accessFiled。我们即将会看到，本地方法会打印实例字段现在得值，然后再将该实例字段的值设置为一个新的值。等到本地方法返回后，我们会再次打印这个字段的值，以演示该字段的值确实是改变了。下面是本地方法InstanceFiledAccess.accessField方法的具体实现：

```
JNIEXPORT void JNICALL
Java_InstanceFieldAccess_accessField(JNIEnv *env, jobject obj) {
    jfieldID fid; /* store the field ID */ 
    jstring jstr; 
    const char *str;

    /* Get a reference to obj’s class */ 
    jclass cls = (*env)->GetObjectClass(env, obj);
    printf("In C:\n");

    /* Look for the instance field s in cls */ 
    fid = (*env)->GetFieldID(env, cls, "s","Ljava/lang/String;"); 
    if (fid == NULL) { 
        return; /* failed to find the field */ 
    }

    /* Read the instance field s */
    jstr = (*env)->GetObjectField(env, obj, fid); 
    str = (*env)->GetStringUTFChars(env, jstr, NULL); 
    if (str == NULL) {
        return; /* out of memory */ 
    } 
    printf(" c.s = \"%s\"\n", str); 
    (*env)->ReleaseStringUTFChars(env, jstr, str);

    /* Create a new string and overwrite the instance field */ 
    jstr = (*env)->NewStringUTF(env, "123"); 
    if (jstr == NULL) {
        return; /* out of memory */ 
    }
    (*env)->SetObjectField(env, obj, fid, jstr);
}
```

搭配InstanceFieldAccess本地库执行InstanceFiledAccess可以得到如下输出：

```
In C:
    c.s = "abc" 
In Java:
    c.s = "123"
```

### 4.1.1 访问实例字段的过程

  要访问实例字段，本地方法遵循两步过程。首先，调用GetFieldID从类引用、字段名和字段描述符中取得字段ID。

```
fid = (*env)->GetFieldID(env, cls, "s", "Ljava/lang/String;");
```

这个示例代码通过在实例引用obj上调用GetObjectClass来获得类引用，obj引用将作为第二个参数传送给本地方法实现。

  你一旦取得了字段ID，你可以将对象引用和字段ID传给合适的实例字段访问函数：

```
jstr = (*env)->GetObjectField(env, obj, fid);
```

因为字符串和数组是特殊类型的对象，我们使用GetObjectField来访问字符串实例字段。除了Get/SetObjectField外，JNI还支持其他的函数例如GetIntField和SetFloatField来访问基本数据类型的实例字段。

### 4.1.2 字段描述符

  你可能注意到在上一节中，我们使用了特殊编码的C字符串”Ljava/lang/String”来代表Java编程语言中的实例字段。这些C字符串就称为JNI字段描述符。

  字符串的内容是由声明的字段决定的。例如，使用“I”代表一个int字段，“F”代表float字段，“D”代表double字段，“Z”代表boolean字段。

  引用类型的字段描述符，例如java.lang.String，以字母L开头，紧接着是JNI类描述字段并以分号作为终结符。完全限定类名中的“.”分隔符在JNI类描述符中更改为“/”，因此你为类型为java.lang.String的字段形成的字段描述符为：”Ljava/lang/String;”。

  数组类型的描述符包含“[”字符，紧接着是数组组件类型的描述符。例如，“[I”是int[]字段类型的字段描述符。12.3.3节（图片先放一下）包含字段描述符的细节以及其在Java编程语言中的对应类型。

![《(译文) JNI编程指南与规范 第四章 字段和方法》](https://i2.wp.com/blog4jimmy.com/wp-content/uploads/2017/11/12.3.3.png)

  你可以使用javap工具（随JDK或者Java 2 SDK一同发布）从类文件中生成字段描述符。通常javap会打印给定类的方法和字段类型，如果你使用-s选项（和-p选项来显示私有成员），javap只打印JNI描述符。

```
javap -s -p InstanceFieldAccess
```

上面的指令会给出包含字段s的JNI描述符信息：

```
...
s Ljava/lang/String; 
...
```

使用javap工具有助于消除手工导出JNI描述符字符串时可能发生的错误。

### 4.1.3 访问静态字段

让我们看看InstanceFieldAccess示例的一个小小的变化：

```
class StaticFielcdAccess { 
    private static int si;
    private native void accessField(); 
    public static void main(String args[]) { 
        StaticFieldAccess c = new StaticFieldAccess(); 
        StaticFieldAccess.si = 100; 
        c.accessField();
        System.out.println("In Java:"); 
        System.out.println(" StaticFieldAccess.si = " + si);
    } 
    static { 
        System.loadLibrary("StaticFieldAccess"); 
    } 
}
```

  StaticFieldAccess类包含一个静态整型资源si，StaticFieldAccess.main方法首先创建一个对象，初始化静态字段，然后调用本地方法StaticFieldAccess.accessField。正如我们即将见到的那样，本地方法先打印静态字段现在的值，然后给该静态字段设置一个新的值。为了验证这个静态字段的值是否真的改变了，在调用完该静态方法后再次打印该静态字段的值。

  下面是静态方法StaticFieldAccess.accessField的实现代码：

```
JNIEXPORT void JNICALL
Java_StaticFieldAccess_accessField(JNIEnv *env, jobject obj) {
    jfieldID fid; /* store the field ID */ 
    jint si;
    /* Get a reference to obj’s class */ 
    jclass cls = (*env)->GetObjectClass(env, obj);
    printf("In C:\n");

    /* Look for the static field si in cls */ 
    fid = (*env)->GetStaticFieldID(env, cls, "si", "I"); 
    if (fid == NULL) {
        return; /* field not found */
    }

    /* Access the static field si */
    si = (*env)->GetStaticIntField(env, cls, fid); 
    printf(" StaticFieldAccess.si = %d\n", si); 
    (*env)->SetStaticIntField(env, cls, fid, 200);
}
```

使用本地库运行程序会产生以下输出：

```
In C:
    StaticFieldAccess.si = 100 
In Java:
    StaticFieldAccess.si = 200
```

  如何访问一个静态字段和如何访问一个实例字段存在两个不同的地方：

1. 对于静态字段，你应该调用GetStaticFieldID，而相对的对于实例字段，你应该调用GetFieldID。GetStaticFieldID和GetFieldID都有相同的返回类型就fieldID。
2. 一旦取得了静态字段ID，你将类引用传送类引用给合适静态字段访问函数，而对于实例字段，你应该传送对象引用。

## 4.2 调用方法

  在Java编程语言中，存在几种类型的方法。实例方法必须通过特定类的实例来调用，而静态方法可以独立于任何实例被调用。下一节我们将讨论构造函数。

  JNI支持一组完整的函数，允许你在本地代码中进行回调操作。下面的示例程序中包含本地方法，它依次调用用Java语言实现的实例方法。

```
class InstanceMethodCall {
    private native void nativeMethod(); 
    private void callback() { 
        System.out.println("In Java");
    }
    public static void main(String args[]) { 
        InstanceMethodCall c = new InstanceMethodCall(); 
        c.nativeMethod();
    } 
    static { 
        System.loadLibrary("InstanceMethodCall"); 
    } 
}
```

下面是本地代码的实现：

```
JNIEXPORT void JNICALL
Java_InstanceMethodCall_nativeMethod(JNIEnv *env, jobject obj) {
    jclass cls = (*env)->GetObjectClass(env, obj); 
    jmethodID mid = (*env)->GetMethodID(env, cls, "callback", "()V"); 
    if (mid == NULL) { 
        return; /* method not found */
    } 
    printf("In C\n"); 
    (*env)->CallVoidMethod(env, obj, mid); 
}
```

运行上面的程序可以获得如下输出：

```
In C
In Java
```

### 4.2.1 调用实例方法

  Java_InstanceMethodCall_nativeMethod方法实现表明需要两个步骤来调用一个实例方法：

- 本地方法首先调用JNI方法GetMethodID。GetMethodID对给定的类进行方法查询。查询是基于方法的名字和方法的类型描述符的。如果这个方法不存在，GetMethodID放回NULL，在这个点上，从本地方法立刻返回并且会导致在调用InstanceMethodCall.nativeMethod的代码中抛出NoSuchMethodError异常。
- 本地方法然后调用CallVoidMethod。Ca llVoidMethod调用一个返回类型为void的实例方法。你将对象，方法ID和实际的参数（但是上面的实例中，这些都为空）传送给CallVoidMethod。

  除了CallVoidMethod方法外，JNI支持其他返回类型的方法调用函数。例如，如果你回调的方法返回一个int类型的值，然后你的本地方法可以使用CallIntMethod。类似的，你可以使用CallObjectMethod来调用返回值为对象（包含java.lang.String实例和数组）的方法。

  你可以使用CallMethod系列函数来调用接口函数。你必须从接口类型中导出方法ID。下面的代码片段，在一个java.lang.Thread实例中调用Runnable.run方法：

```
jobject thd = ...; /* a java.lang.Thread instance */ 
jmethodID mid;
jclass runnableIntf =(*env)->FindClass(env, "java/lang/Runnable"); 
if (runnableIntf == NULL) {
    ... /* error handling */
}
mid = (*env)->GetMethodID(env, runnableIntf, "run", "()V"); 
if (mid == NULL) {
    ... /* error handling */ 
}
(*env)->CallVoidMethod(env, thd, mid); 
... /* check for possible exceptions */
```

  我们已经在3.3.5节中看到过FindClass返回一个明明类的引用。这里我们也可以用它来获取一个命名接口的引用。

### 4.2.2 生成方法描述符

  JNI使用描述符字符串来表示方法类型，类似于它如何表示字段类型。方法描述符组合了参数类型和返回类型。参数类型首先出现，并被一对括号括起来，参数类型按照方法声明的中的顺序列出。多个参数之间没有分隔符，如果一个方法没有参数，则用一对空的圆括号表示。将方法的返回类型放在参数类型的右括号后面。

  举个例子，“(I)V”表明该方法有一个类型为int的参数并且返回类型为void。“()D”表明该方法不需要参数并且返回一个double值。不要让C函数原型如“int f(void)”误导你认为“(V)I”是一个有效的方法描述符，这里应该使用“()I”作为其方法描述符。方法描述符可能包含类描述符，例如如下方法：

```
native private String getLine(String);
```

其方法描述符为

```
(Ljava/lang/String;)Ljava/lang/String;
```

12.3.4节给出了如何构建JNI方法描述符的完整描述。你可以使用javap工具打印JNI方法描述符。例如执行如下指令：

```
javap -s -p InstanceMethodCall
```

你可以获取到如下输出信息：

```
...
private callback ()V
public static main ([Ljava/lang/String;)V 
private native nativeMethod ()V 
...
```

-s标志通知javap输出JNI描述符字符串，而不是他们在Java编程语言中出现类型。-p标志使javap在其输出中包含有关该类的私有成员的信息

### 4.2.3 调用静态方法

  前面的例子演示了在本地代码中如何调用一个实例方法。类似的，你可以通过下面的一些步骤从本地方法中进行静态方法回调：

- 通过GetStaticMethodID获取静态方法ID，而不是GetMethodID
- 将类、方法ID和参数传给静态方法调用函数之一：CallStaticVoidMethod，CallStaticBooleanMethod等。

  在允许你调用静态方法的函数和允许你调用实例方法的函数中有一个关键性的区别，前者使用类引用作为参数，而后者使用对象引用作为参数。例如：将类引用传递给CallStaticVoidMethod，但是将对象引用传递给CallVoidMethod。

  在Java编程语言层面，您可以使用两种可选语法来调用类Cls中的静态方法f：Cls.f或obj.f，其中obj是Cls的实例。（后者是推荐的编程风格。）在JNI中，当从本地代码发出静态方法调用时，必须始终指定类引用。

让我们看一个实例：在静态代码中使用回调调用一个静态方法。它和之前的InstanceMethodCall有一些不同：

```
class StaticMethodCall {
    private native void nativeMethod(); 
    private static void callback() { 
        System.out.println("In Java");
    }
    public static void main(String args[]) { 
        StaticMethodCall c = new StaticMethodCall(); 
        c.nativeMethod();
    } 
    static { 
        System.loadLibrary("StaticMethodCall"); 
    } 
}
```

下面是本地方法的实现：

```
JNIEXPORT void JNICALL
Java_StaticMethodCall_nativeMethod(JNIEnv *env, jobject obj) {
    jclass cls = (*env)->GetObjectClass(env, obj); 
    jmethodID mid = (*env)->GetStaticMethodID(env, cls, "callback", "()V"); 
    if (mid == NULL) { 
        return; /* method not found */
    } 
    printf("In C\n"); 
    (*env)->CallStaticVoidMethod(env, cls, mid); 
}
```

确保你将通过cls（用粗体突出显示），而不是obj传递给CallStaticVoidMethod。运行上述程序可以得到如下结果：

```
In C 
In Java
```

### 4.2.4 调用父类的实例方法

  你可以调用在父类中定义但是被实例对象所在的类覆盖的实例方法。JNI为此提供了一组CallNonvirtualMethod方法。要调用超类中定义的实例方法，请执行下面的步骤：

- 使用GetMethodID而不是GetStaticMethodID从超类引用中获取方法ID
- 将对象、超类、方法ID和参数传给非虚调用函数系列之一，例如CallNonvirtualVoidMethod、CallNonvirtualBooleanMethod等。

  你需要调用超类的实例方法的机会相对较少，该工具类似于使用Java编程语言中的以下构造来调用覆盖的超类方法（如f）：

```
super.f();
```

CallNonvirtualVoidMethod方法同样能够用来调用构造函数，如下节中介绍的一样。

## 4.3 调用构造方法

  在JNI中，可以按照类似于调用实例方法的那些步骤来来调用构造方法。要获取构造方法的方法ID，在方法描述符中将“”作为方法名并且将“V”作为返回类型。然后你可以通过传递方法ID给JNI函数（例如NewObject）来调用构造函数。以下代码实现了JNI函数NewString的等效功能，它从Unicode字符中创建一个java.lang.String对象并存储在一个C缓冲区中：

```
jstring MyNewString(JNIEnv *env, jchar *chars, jint len) {
    jclass stringClass; 
    jmethodID cid; 
    jcharArray elemArr; 
    jstring result;

    stringClass = (*env)->FindClass(env, "java/lang/String"); 
    if (stringClass == NULL) {
        return NULL; /* exception thrown */ 
    }

    /* Get the method ID for the String(char[]) constructor */ 
    cid = (*env)->GetMethodID(env, stringClass, "", "([C)V");
    if (cid == NULL) { 
        return NULL; /* exception thrown */ 
    }

    /* Create a char[] that holds the string characters */ 
    elemArr = (*env)->NewCharArray(env, len); 
    if (elemArr == NULL) {
        return NULL; /* exception thrown */ 
    } 
    (*env)->SetCharArrayRegion(env, elemArr, 0, len, chars);

    /* Construct a java.lang.String object */ 
    result = (*env)->NewObject(env, stringClass, cid, elemArr);

    /* Free local references */ 
    (*env)->DeleteLocalRef(env, elemArr); 
    (*env)->DeleteLocalRef(env, stringClass); 

    return result;
}
```

  这个例子是复杂的，值得进行仔细的分析。首先，FindClass返回java.lang.String类的引用。下一步，GetMethodID返回字符串构造函数（String(char[] chars)）的方法ID。然后我们调用NewCharArray分配一个字符数组来存放所有的字符元素。JNI函数调用由方法ID指定的构造函数。NewObject将需要构造的类的引用、构造函数的方法ID和需要传送给构造方法的参数作为参数。

  DeleteLocalRef调用允许虚拟机释放elemArr和stringClass占用的本地资源。5.2.1节会提供一个详细的描述说明什么时候和为什么需要调用DeleteLocalRef。

  字符串是对象，这个例子进一步突出了这一点。但是这个例子也引出了一个问题。鉴于我们可以使用其他的JIN函数实现等效的功能，为什么JNI还要提供NewString之类的函数呢？这是因为内置的字符串函数要比本地代码调用java.lang.String API更有效率。因为String是最使用的对象类型，所以在JNI中值得特别支持。

  也可以使用CallNonvirtualVoidMethod函数调用构造函数。在这种情况下，本地代码必须首先通过AllocObject函数创建一个为初始化的对象。上面的单个NewObject调用

```
result = (*env)->NewObject(env, stringClass, cid, elemArr);
```

可以被AllocObject后跟一个CallNonvirtualVoidMethod代替。

```
result = (*env)->AllocObject(env, stringClass); 
if (result) {
    (*env)->CallNonvirtualVoidMethod(env, result, stringClass, cid, elemArr);
    /* we need to check for possible exceptions */ 
    if ((*env)->ExceptionCheck(env)) { 
        (*env)->DeleteLocalRef(env, result); 
        result = NULL;
    }
}
```

  AllocObject创建一个为初始化的对象，并且必须小心使用，以便每个对象最多调用一个构造函数。本地代码不应该在同一个对象上多次调用构造函数。

  有时候你会发现，先创建一个为初始化的对象然后调用构造函数是非常有用的。但是在更多的时候你应该调用NewObject，并避免使用更容易产生错误的AllocObject/CallNonvirtualVoidMethod方法对。

## 4.4 缓存字段和方法ID

  获取字段和方法ID需要基于字段和方法ID的名字和描述符进行符号查找。符号查找消耗相对较多，本节我们将介绍一种能够减少这种开销的技术。这种方法是计算字段和方法ID，然后缓存它们以便后续重复使用。有两种方法来缓存字段和方法ID，具体取决于是在使用字和方法ID时执行缓存还是在静态初始化块中定义字段或者方法来执行缓存。

### 4.4.1 在使用时执行缓存

  字段或者方法ID可以在本地代码访问字段值或者执行方法回调的时候被缓存。在下面的Java_InstanceFieldAccess_accessField函数实现中，使用静态变量对方法ID进行缓存，以便在每次调用InstanceFieldAccess.accessField方法时，不需要重新计算了。

```
JNIEXPORT void JNICALL
Java_InstanceFieldAccess_accessField(JNIEnv *env, jobject obj) {
    static jfieldID fid_s = NULL; /* cached field ID for s */
    jclass cls = (*env)->GetObjectClass(env, obj); 
    jstring jstr; 
    const char *str;
    
    if (fid_s == NULL) { 
        fid_s = (*env)->GetFieldID(env, cls, "s", "Ljava/lang/String;"); 
        if (fid_s == NULL) { 
            return; /* exception already thrown */ 
        } 
    }
 
    printf("In C:\n");
    jstr = (*env)->GetObjectField(env, obj, fid_s); 
    str = (*env)->GetStringUTFChars(env, jstr, NULL); 
    if (str == NULL) {
        return; /* out of memory */ 
    } 

    printf(" c.s = \"%s\"\n", str); 
    (*env)->ReleaseStringUTFChars(env, jstr, str);
    jstr = (*env)->NewStringUTF(env, "123"); 
    if (jstr == NULL) {
        return; /* out of memory */ 
    } 

    (*env)->SetObjectField(env, obj, fid_s, jstr); 
}
```

  加粗显示的静态变量fid_s保存了为InstanceFiledAccess.s预先计算的方法ID。该静态变量初始化为NULL，当InstanceFieldAccess.accessField方法第一次被调用时，它计算该字段ID然后将其缓存到该静态变量中以方便后续使用。

  你可能注意到上面的代码中存在着明显的竞争条件。多个线程可能同时调用InstanceFieldAccess.accessField方法并且同时计算相同的字段ID。一个线程可能会覆盖另一个线程计算好的静态变量fid_s。幸运的是，虽然这种竞争条件在多线程中导致重复的工作，但是明显是无害的。同一个类的同一个字段被多个线程计算出来的字段ID必然是相同的。

  根据上面的想法，我们同样可以在MyNewString例子的开始部分缓存java.lang.String构造方法的方法ID。

```
jstring
MyNewString(JNIEnv *env, jchar *chars, jint len) {
    jclass stringClass; 
    jcharArray elemArr; 
    static jmethodID cid = NULL; 
    jstring result;

    stringClass = (*env)->FindClass(env, "java/lang/String"); 
    if (stringClass == NULL) {
        return NULL; /* exception thrown */ 
    }

    /* Note that cid is a static variable */ 
    if (cid == NULL) {
        /* Get the method ID for the String constructor */ 
        cid = (*env)->GetMethodID(env, stringClass, "", "([C)V");
        if (cid == NULL) { 
            return NULL; /* exception thrown */ 
        } 
    }

    /* Create a char[] that holds the string characters */ 
    elemArr = (*env)->NewCharArray(env, len); 
    if (elemArr == NULL) {
        return NULL; /* exception thrown */ 
    } 
    (*env)->SetCharArrayRegion(env, elemArr, 0, len, chars);
    
    /* Construct a java.lang.String object */ 
    result = (*env)->NewObject(env, stringClass, cid, elemArr);

    /* Free local references */ 
    (*env)->DeleteLocalRef(env, elemArr); 
    (*env)->DeleteLocalRef(env, stringClass); 
    return result;
}
```

  当MyNewString第一次被调用的时候，我们为java.lang.String构造器计算方法ID。加粗突出显示的静态变量cid缓存这个结果。

### 4.4.2 在类的静态初始化块中执行缓存

  当我们在使用时缓存字段或方法ID的时候，我们必须引入一个坚持来坚持字段或方法ID是否已被缓存。当ID已经被缓存时，这种方法不仅在“快速路径”上产生轻微的性能影响，而且还可能导致缓存和检查的重复工作。举个例子，如果多个本地方法全部需要访问同一个字段，然后他们就需要计算和检查相应的字段ID。在许多情况下，在程序能够有机会调用本地方法前，初始化本地方法所需要的字段和方法ID会更为方便。虚拟机会在调用该类中的任何方法前，总是执行类的静态初始化器。因此，一个计算并缓存字段和方法ID的合适位置是在该字段和方法ID的类的静态初始化块中。例如，要缓存InstanceMethodCall.callback的方法ID，我们引入了一个新的本地方法initIDs，它由InstanceMethodCall类的静态初始化器调用：

```
class InstanceMethodCall {
    private static native void initIDs(); 
    private native void nativeMethod(); 
    private void callback() { 
        System.out.println("In Java");
    }
    public static void main(String args[]) { 
        InstanceMethodCall c = new InstanceMethodCall(); 
        c.nativeMethod();
    } 
    static {
        System.loadLibrary("InstanceMethodCall"); 
        initIDs();
    } 
}
```

跟4.2节的原始代码相比，上面的程序包含二外的两行（用粗体突出显示），initIDs的实现仅仅是简单的为InstanceMethodCall.callback计算和缓存方法ID。

```
jmethodID MID_InstanceMethodCall_callback; 

JNIEXPORT void JNICALL Java_InstanceMethodCall_initIDs(JNIEnv *env, jclass cls) {
    MID_InstanceMethodCall_callback = (*env)->GetMethodID(env, cls, "callback", "()V"); 
}
```

  在InstanceMethodCall类中，在执行任何任何方法（例如nativeMethod或main）之前虚拟机先运行静态初始化块。当方法ID已经缓存到一个全局变量中，InstanceMethodCall.nativeMethod方法的本地实现就不再需要执行符号查找了。

```
JNIEXPORT void JNICALL
Java_InstanceMethodCall_nativeMethod(JNIEnv *env, jobject obj) {
    printf("In C\n"); 
    (*env)->CallVoidMethod(env, obj, MID_InstanceMethodCall_callback); 
}
```

### 4.4.3 缓存ID的两种方法之间的比较

  如果JNI程序员无法控制定义了字段和方法的类的源代码，那么在使用时缓存ID是合理的解决方案。例如在MyNewString例子当中，我们没有办法为了预先计算和缓存java.lang.String构造器的方法ID而向java.lang.String类中插入一个用户定义的initIDs本地方法。与在定义类的静态初始化块中执行缓存相比，在使用时进行缓存存在许多缺点：

- 如之前解释，在使用的时候进行缓存，在快速路径执行过程中需要进行检查，而且可能对同一个字段和方法ID进行重复的检查和初始化。
- 方法和字段ID仅在类卸载前有效，如果你是在运行时缓存字段和方法ID，则必须确保只要本地代码仍然依赖缓存ID的值时，定义类就不能被卸载或者重新加载。（下一章将介绍如何通过使用JNI创建对该类的引用来保护类不被卸载。）另一方面，如果缓存是在定义类的静态初始化块中完成的，当类被卸载并稍后重新加载时，缓存的ID将会自动重新计算。

  因此在可行的情况下，最好在其定义类的静态初始化块中缓存字段和方法ID。

## 4.5 JNI字段和方法的操作性能

  知道如何缓存字段和方法ID以提高性能后，你可能在想：使用JNI访问字段和调用方法的性能特性如何？从本地代码中执行方法回调的成本和调用本地方法的成本以及调用常规方法的成本相比如何？这个问题的答案无疑取决于底层虚拟机实现JNI的效率性了。因此不可能给出准确的性能特性，这些性能特性被保证适用于各种各样的虚拟机实现。相反我们将会分析本地方法调用和JNI字段和方法操作的固有成本，并未JNI程序员和实现者提供一般的性能指南。让我们首先开始比较Java/native调用和Java/Java调用的成本。由于以下的原因Java/native调用可能比Java/Java调用慢：

- 在Java虚机实现中，本地方法调用最有可能遵循与Java/Java调用不同的约定。因此，虚拟机必须执行额外的操作来构建参数，并在跳到本地方法入口之前设置堆栈结构。
- 虚拟机经常使用内联方法调用。内联Java/native调用比内联Java/Java调用要困难得多。

  我们估计，一个典型的虚拟机实现执行Java/native调用比执行Java/Java调用大概慢两到三倍。因为Java/Java调用只需要几个周期，所以额外的开销基本可以忽略不计，除非本地方法执行一些微不足道的操作。构建一个Java虚拟机实现，让其Java/native调用性能接近或者等于Java/Java调用是可行的。（例如，这种虚拟机可以将JNI调用规则调整为和Java/Java调用规则一样。）

  native/Java回调的性能特性在技术上类似于Java/native调用。理论上，native/Java回调的开销也可能是Java/Java调用的两到三倍内。但是在实际上，native/Java调用相对少见，虚拟机通常不会优化优化回调性能。在撰写本文时，许多虚拟机实现使得native/Java回调的开销可以比Java/Java调用高出10倍。

  使用JNI进行字段访问的开销主要是通过JNIEnv调用的成本。本地代码不是直接引用对象，而是通过C调用的返回值来引用对象。函数调用时必须的，因为它将本地代码与虚拟机实现维护的内部兑现表示隔离起来。JNI字段访问的开销是可以忽略不计的，因为函数调用只需要几个周期



# 第五章 本地和全局引用

  JNI将实例和数组类型（例如jobject、jclass、jstring和jarray）公开为不透明引用。本地代码不能直接检查不透明引用指针的内容。而是通过JNI函数来获取不透明引用所指向的数据结构。通过处理不透明引用，你不必担心依赖于特定Java虚拟机的内部对象数据结构布局。但是，在JNI中，你需要了解更多有关于不同类型的引用：

- JNI支持三中透明引用：本地引用，全局引用和弱全局引用
- 本地和全局引用拥有不同的生命周期。本地引用会被自动回收，而全局引用和弱全局引用会一直存在直到程序员将其释放
- 一个本地或全局引用保持被引用的对象不会被垃圾回收。但是一个弱全局引用允许被引用的对象呗垃圾回收。
- 并非所有的引用都可以在所有的上下文中使用。例如，在创建引用返回后的本地代码中使用本地引用是非法的。

  在本章中，我们将详细讨论这些问题。 正确管理JNI引用对于编写可靠和节省空间的代码至关重要。

## 5.1 本地和全局引用

  什么是本地和全局引用，以及他们有什么不同呢？我们会用一系列的实例来说明本地和全局引用。

### 5.1.1 本地引用

大多数JNI方法会创建本地引用。例如，JNI方法NewObject创建一个新的实例对象并返回引用该对象的本地引用。

  本地引用仅在创建它的本地方法的动态上下文中有效，并且仅在该方法的一次调用中有效。在本地方法执行期间创建的所有本地引用将在本地方法返回后被释放。

  不能在本地方法中通过静态变量来储存本地引用，并在后续调用中使用相同的引用。例如下面的代码，是4.4.4节的MyNewString方法的修改版本，在这里使用了不正确的本地引用。

```
/* This code is illegal */ jstring
MyNewString(JNIEnv *env, jchar *chars, jint len) {
    static jclass stringClass = NULL; 
    jmethodID cid;
    jcharArray elemArr; 
    jstring result;
    if (stringClass == NULL) {
        stringClass = (*env)->FindClass(env, “java/lang/String”);
        if (stringClass == NULL) { 
            return NULL;
        }
    }
    /* It is wrong to use the cached stringClass here, because it may be invalid. */ 
    cid = (*env)->GetMethodID(env, stringClass, "", "([C)V");
    ...
    elemArr = (*env)->NewCharArray(env, len); 
    ...
    result = (*env)->NewObject(env, stringClass, cid, elemArr); 
    (*env)->DeleteLocalRef(env, elemArr); 
    return result;
}
```

  这里已经消除了和我们将要讨论的没有直接关系的行。在静态变量中缓存stringClass的目的可能是想消除重复执行如下函数调用的开销：

```
FindClass(env, "java/lang/String");
```

  这不是正确的方法，因为FindClass返回一个指向java.lang.String类对象的本地引用。下面分析为什么这样会引发问题，假设C.f本地方法实现调用MyNewString：

```
JNIEXPORT jstring JNICALL Java_C_f(JNIEnv *env, jobject this) {
    char *c_str = ...; 
    ...
    return MyNewString(c_str); 
}
```

  在本地方法C.f调用返回后，虚拟机会释放所有在Java_C_f运行期间创建的本地引用。这些释放的本地引用包括对储存在stringClass变量中的类对象的本地引用。在后面，MyNewString调用将会尝试使用一个无效的本地引用，这可能导致内存损坏或者引发系统崩溃。例如，如下的代码片段使两个连续的调用到C.f并导致MyNewString遇到无效的本地引用：

```
...
... = C.f(); // The first call is perhaps OK. 
... = C.f(); // This would use an invalid local reference.
...
```

  有两种方法能够使一个本地方法无效。如前所述，在本地方法返回后，虚拟机会自动的释放所有在该本地方法执行期间创建的本地引用。另外，程序员可能想使用JNI函数例如，DeleteLocalRef来显示的管理本地引用的生命周期。

  如果虚拟机会在本地方法返回后自动释放，为什么还需要显示释放本地引用呢？本地引用防止被引用的对象被垃圾收集器回收，一直持续到本地引用无效为止。例如，在MyNewString中的DeleteLocalRef调用允许数组对象elemArr立即被垃圾收集器回收。否则，虚拟机只会在MyNewString调用的本地方法返回后（例如上面的C.f）释放elemArr对象。

  一个本地引用在其销毁之前，可能传递给多个本地方法。例如，MyNewString方法返回一个通过NewObject创建的字符串引用。然后由MyNewString的调用者决定是否释放由MyNewString返回的本地引用。在Java_C_f例子中，C.f又作为本地方法调用的结果返回MyNewString的结果。在虚拟机从JAVA_C_f函数接收到本地引用后，他将底层字符串对象给c.f的调用者然后销毁最初由JNI函数NewObject创建的本地引用，然后销毁最初由JNI函数NewObject创建的本地引用。

  地引用当然仅在创建它的线程中有效。在一个线程中创建的本地引用不能够在其他线程中使用。在本地方法中将本地引用储存在全局变量中，并期望在另一个线程中使用是一个编程错误。

### 5.1.2 全局引用

  你可以在跨多个本地方法调用中使用全局变量。全局引用可以跨多线程使用并保持有效，直到程序员释放它为止。和本地引用一样，一个全局引用能够确保被引用的对象不会被垃圾收集器回收。

  和本地引用不同的是，本地引用可以通过大多数JNI函数创建，但是全局引用只可以通过一个JNI方法（NewGlobalRef）创建。接下来的MyNewString版本显示如何使用全局引用。我们突出显示下面的代码和在上一节中错误地缓存本地引用的代码之间的区别：

```
/* This code is OK */ 
jstring MyNewString(JNIEnv *env, jchar *chars, jint len) 
{
    static jclass stringClass = NULL; 
    ...
    if (stringClass == NULL) { 
        jclass localRefCls = (*env)->FindClass(env, "java/lang/String"); 
        if (localRefCls == NULL) {
            return NULL; /* exception thrown */ 
        } 


        /* Create a global reference */
        stringClass = (*env)->NewGlobalRef(env, localRefCls);
        /* The local reference is no longer useful */ 
        (*env)->DeleteLocalRef(env, localRefCls);
        /* Is the global reference created successfully? */ 
        if (stringClass == NULL) {
            return NULL; /* out of memory exception thrown */ 
        } 

    }
    ... 
}
```

这个修改版本中将从FindClass返回的本地引用传送给NewGlobalRef，该方法创建一个指向java.lang.String对象的全局引用。我们检查在删除localRefCls后NewGlobalRef是否成功创建了stringClass，因为这两种情况下都需要删除本地引用localRefCls。

### 5.1.3 弱全局引用

  弱全局引用是在Java 2 JDK 1.2中新加入的。弱全局引用通过NewGlobalWeakRef创建，通过DeleteGlobalWeakRef释放。和全局引用一样，弱全局引用跨本地方法调用和跨线程调用依旧有效。而和全局引用不同的是，弱全局引用不能防止底层数据对象被垃圾收集器回收。

  MyNewString示例显示了如何缓存java.lang.String的全局引用。MyNewString示例可以使用弱全局引用来缓存java.lang.String类。我们是使用全局引用还是弱全局引用并不重要，因为java.lang.String是一个系统类，永远不会被垃圾收集器回收。当本机代码缓存的引用不能使底层对象不被垃圾回收时，弱全局引用变得更加有用。例如，假设一个本地方法mypks.MyCls.f对类mypks.MyCls2进行缓存。在弱全局引用中缓存类仍然允许mypkg.MyCls2被卸载。

```
JNIEXPORT void JNICALL Java_mypkg_MyCls_f(JNIEnv *env, jobject self) {
    static jclass myCls2 = NULL; 
    if (myCls2 == NULL) { 
        jclass myCls2Local = (*env)->FindClass(env, "mypkg/MyCls2"); 
        if (myCls2Local == NULL) { 
            return; /* can’t find class */
        }
        myCls2 = NewWeakGlobalRef(env, myCls2Local); 
        if (myCls2 == NULL) {
            return; /* out of memory */ 
        }
    } 
    ... /* use myCls2 */ 
}
```

我们假设MyCls和MyCls2有相同的生命周期（例如，它们可能是通过同一个类加载器加载的）。但是我们没有考虑到这样一个场景，在MyCls及其本地方法实现Java_mypks_MyCls仍在使用的情况下，MyCls2卸载并在稍后重新加载。如果这种情况发生了，我们必须检查缓存的弱引用是否仍然指向一个存活的类对象，或者指向已经没垃圾收集器回收的类对象。下一节将介绍如何对弱全局引用执行此类检查。

### 5.1.4 引用比较

  在给定的两个本地、全局、弱全局引用中，可以使用IsSameObject方法来检查它们是否引用同一个对象。例如：

```
(*env)->IsSameObject(env, obj1, obj2)
```

如果obj1和obj2引用同一个对象，那么这个方法就放回JNI_TRUE（或者1），否则返回JNI_FALSE（或者0）。

  在JNI中，NULL引用是指向Java虚拟机中的空对象，如果obj是被本地或者全局引用，则可以使用：

```
(*env)->IsSameObject(env, obj, NULL)
```

或者

```
obj == NULL
```

来确认obj是否引用一个空对象。

  弱引用对象使用的规则有一些不同。NULL弱引用引用空对象。 然而，IsSameObject对于弱全局引用具有特殊用途。您可以使用IsSameObject来确定非NULL弱全局引用是否仍然指向一个活动对象。假设wobj是一个非NULL的弱全局引用。以下调用：

```
(*env)->IsSameObject(env, wobj, NULL)
```

如果wobj引用一个已经被回收的对象，则返回JNI_TRUE，如果wobj仍然引用一个存活的对象，则返回JNI_FALSE。

## 5.2 释放引用

  除了被引用对象占用的内存外，每个JNI引用本身都会消耗一定量的内存。作为一个JNI程序员，你应该了解到你的程序在一个给定的时间内，将会使用的引用数量。特别是，你应该意识到你的程序在执行期间的某个时间点上，允许创建的本地引用数量的上限，即使这些本地引用最后会被虚拟机自动释放。暂时性的过多创建引用，可能会导致内存耗尽。

### 5.2.1 释放本地引用

在大多数情况下，在实现一个本地方法的时候，你不用过多的考虑释放本地对象。当本地方法返回到调用者处时，Java虚拟机会为你释放它们。但是JNI程序员有时候应该显示的释放本地引用以避免内存占用过多。考虑一下情况：

- 你需要在一个本地方法调用中创建大量的本地引用。这可能导致JNI内部本地引用表溢出，因此立即删除那些不再需要的本地引用将是一个好方法。例如，在以下程序段中，本地代码有遍历一个大的字符串数组的可能。每次迭代后，本地代码应该显示的释放对字符串元素的本地引用。如下所示：

```
for (i = 0; i < len; i++) {
    jstring jstr = (*env)->GetObjectArrayElement(env, arr, i); 
    ... /* process jstr */
    (*env)->DeleteLocalRef(env, jstr); 
}
```

- 你想编写一个从未知上下文中调用的函数。在第4.3节中显示的MyNewString示例说明使用DeleteLocalRef在函数中快速删除本地引用，否则每次调用MyNewString函数后都会分配两个本地引用。
- 你的本地方法不会返回。一个本地方法可能进入无限事件调度循环中，释放在循环内创建的本地引用将是非常重要的，这样它们就不会无限积累，导致内存泄漏。
- 你的本地方法访问一个大对象，因而需要创建该对象的本地引用。然后，native方法在返回给调用者前可以进行额外的计算。对大对象的本地引用将阻止对象在本地方法返回前本垃圾收集器回收，即使对象不再在本地方法的剩余部分使用。例如在以下程序片段中，由于事先已经显示的调用DeleteLocalRef了，因此在执行函数longyComputation时，垃圾收集器可能会释放lref引用的对象。

```
/* A native method implementation */ JNIEXPORT void JNICALL
Java_pkg_Cls_func(JNIEnv *env, jobject this) {
    lref = ...                       /* a large Java object */ 
    ...                            /* last use of lref */
    (*env)->DeleteLocalRef(env, lref);
    lengthyComputation();           /* may take some time */ 
    return;                        /* all local refs are freed */
}
```

### 5.2.2 在Java 2 JDK 1.2中管理本地引用

  Java 2 JDK 1.2中提供了一组而外的函数用于管理本地应用的生命周期。这些函数是EnsureLocalCapacity、NewLocalRef、PushLocalFrame和PopLocalFrame。

  JNI规范规定虚拟机能够自动确保每个本地方法能够创建至少16个本地引用。经验表明，除了与虚拟机中的对象进行复杂的交互外，对于大多数本地方法，这个数量已经足够。但是如果，需要创建而外的本地引用，那么本地方法可能会发出一个EnsureLocalCapacity调用，以确保有足够的本地引用空间。例如，上述示例的轻微变化为循环执行期间创建的所有本地参考提供足够的容量，如果有足够的内存可用：

```
/* The number of local references to be created is equal to the length of the array. */
if ((*env)->EnsureLocalCapacity(env, len)) < 0) { 
    ... /* out of memory */
} 
for (i = 0; i < len; i++) {
    jstring jstr = (*env)->GetObjectArrayElement(env, arr, i); 
    ... /* process jstr */
    /* DeleteLocalRef is no longer necessary */ 
}
```

当然和之前立即删除本地引用的版本相比，上面的版本需要消耗更多的内存。或者，Push / PopLocalFrame函数允许程序员创建本地引用的嵌套范围。 例如，我们也可以重写同样的例子，如下所示：

```
#define N_REFS ... /* the maximum number of local references used in each iteration */
for (i = 0; i < len; i++) {
    if ((*env)->PushLocalFrame(env, N_REFS) < 0) {
        ... /* out of memory */
    }
    jstr = (*env)->GetObjectArrayElement(env, arr, i); 
    ... /* process jstr */
    (*env)->PopLocalFrame(env, NULL); 
}
```

  PushLocalFrame为特定数量的本地引用创建一个新的范围。PopLocalFrame破坏超出的范围，释放该范围内的所有本地引用。使用Push/PopLocalFrame的好处是它们可以管理本地引用的生命周期，而无需担心在执行过程中可能会创建的每个本地引用。在上面的例子中，如果处理jstr的计算创建了额外的本地引用，则这些本地引用将会在PopLocalFrame返回后被释放。

  当你编写期望返回一个本地引用的实例程序时，NewLocalRef函数很有用。我们将在5.3节中演示NewLocalRef函数的用法。

  本地代码可能会创建超出默认容量16或者PushLocalFrame或者EnsureLocalCapacity调用中保留的容量的本地引用。虚拟机将会尝试分配本地引用所需的内存。然而不能保证，这些内存是可用的。如果分配内存失败，虚拟机将会退出。你应该为本地引用保留足够的内存和尽快释放本地引用以避免这种意外的虚拟机退出。

  Java 2 JDK 1.2提供了一个命令行参数-verbose:jni。当使能这个参数，虚拟机会报告超过预留容量的本地引用创建情况。

### 5.2.3 释放全局变量

  当你的本地代码不再需要访问一个全局引用时，你应该调用DeleteGlobalRef方法。如果你忘记调用这个函数，虚拟机将无法通过垃圾收集器回收相应的对象，即使这个对象再也不会在系统的其他地方中使用。

  当你的本地代码不再需要访问一个弱全局引用时，你应该调用DeleteWeakGlobalRef方法。如果你忘记调用这个函数，Java虚拟机仍然能够通过垃圾收集器收集底层对象，当时将无法回收该弱全局引用对象占用的内存。

## 5.3 引用管理规范

  我们现在已经准备好基于我们前面的几节介绍的内容，来处理在本地代码中管理JNI引用的规则。目的是消除不必要的内存使用和对象保留。

  通常来说有两种本地代码，直接实现在任意上下文中使用的本地方法和效用函数的函数。

  当编写直接实现本地方法时，你需要注意避免在循环中过多的创建本地引用以及由不返回的本地方法创建的不需要的本地引用。在本地方法返回后，留下最多16个本地引用由虚拟机删除是可以接受的。本地方法调用不能导致全局引用或者弱全局引用累积，因为全局引用和弱全局引用在本地方法返回后不会释放。编写本机实用程序函数时，必须注意不要在整个函数中的任何执行路径上泄漏任何本地引用。因为效用函数可以从意料之外的上下文重复调用，任何不必要的引用创建都可能导致内存溢出。

- 当一个返回基本类型的函数被调用时，它不会产生额外的本地、全局、弱全局引用累积副作用。
- 当一个返回引用类型的函数被调用，它不能有本地、全局、弱全局引用的额外累积，除非这个引用被当做返回值。

  为了缓存的目的，一个函数创建一些全局或弱全局引用是可以接受的，因为仅在第一次调用的时候会创建这些引用。

  如果一个函数返回一个引用，你应该使用函数规范的返回引用部分。他不应该在某些时候返回本地引用，而在其他时候返回全局引用。调用者需要知道函数的返回类型，以便正确的管理自己的JNI引用。例如，以下代码重复的调用了一个函数GetInfoString。我们需要知道GetInfoString返回的引用类型，以便能够在每次迭代后正确释放返回的JNI引用。

```
while (JNI_TRUE) {
    jstring infoString = GetInfoString(info); 
    ... /* process infoString */
    ??? /* we need to call DeleteLocalRef, DeleteGlobalRef, or DeleteWeakGlobalRef depending on the type of reference returned by GetInfoString. */
}
```

  在Java 2 JDK 1.2中，NewLocalRef函数经常用于确保函数返回一个本地引用。为了说明，让我们对MyNewString函数进行另一个（有点设计的）更改。以下版本在全局引用中缓存经常请求的字符串（例如“CommonString”）：

```
jstring MyNewString(JNIEnv *env, jchar *chars, jint len) {
    static jstring result;

    /* wstrncmp compares two Unicode strings */ 
    if (wstrncmp("CommonString", chars, len) == 0) { 
        /* refers to the global ref caching "CommonString" */ 
        static jstring cachedString = NULL; 
            if (cachedString == NULL) {
                /* create cachedString for the first time */ 
                jstring cachedStringLocal = ... ; 
                /* cache the result in a global reference */ 
                cachedString =(*env)->NewGlobalRef(env, cachedStringLocal); 
            } 
        return (*env)->NewLocalRef(env, cachedString); 
    }
    ... /* create the string as a local reference and store in result as a local reference */
    return result; 
}
```

  正常的代码路径返回作为本地引用的字符串。正如前面解释的那样，我们必须在一个全局引用中保存缓存的字符串，让其能够被多个本地方法和多个线程访问。加粗显示的行创建一个新的引用对象，其引用同一个缓存在全局引中的对象。作为其调用者契约的一部分，MyNewString经常返回一个本地引用。

  Push/PopLocalFrame方法对于管理本地引用的声明周期是非常简便的。如果你在一个本地方法的入口调用PushLocalFrame，需要在本地方法返回前调用PopLocalFrame以确保所有在本地方法执行期间创建的本地引用都会被回收。Push/PopLocalFrame函数是非常有效率的。强烈建议你使用它们。

  如果你在函数的入口调用了PushLocalFrame，记得在程序的所有退出路径上调用PopLocalFrame。例如，下面的程序有一个PushLocalFrame调用，但是却需要多个PopLocalFrame调用。

```
jobject f(JNIEnv *env, ...) 
{
    jobject result;
    if ((*env)->PushLocalFrame(env, 10) < 0) { 
        /* frame not pushed, no PopLocalFrame needed */ 
        return NULL;
    } 
    ...
    result = ...; 
    if (...) {
        /* remember to pop local frame before return */ 
        result = (*env)->PopLocalFrame(env, result); 
        return result;
    } 
    ...
    result = (*env)->PopLocalFrame(env, result); /* normal return */ return result;
}
```

  错误的放置PopLocalFrame调用回引起不确定的行为，例如导致虚拟机崩溃。

  上面的例子也表明为什么有时指定PopLocalFrame的第二个参数是有用的。result本地引用最初在由PushLocalFrame构造的新框架中创建的。PopLocalFrame将其作为第二个参数，result转换为前一贞中的新的本地引用，然后弹出最顶层的框架。



# 第六章 异常

  我们已经遇到大量在本地代码中需要检查执行JNI方法后可能产生的错误。这一章将介绍本地代码如何从这些错误状况中检测和修复。

  我们将会重点关注作为JNI函数调用的结果发生的错误，而不是在本地代码中发生的任意错误。如果一个本地方法进行了操作系统调用，则只需要按照文档说明的方式来检查系统调用中可能发生的错误。另一方面，如果本地方法想Java API方法进行回调，则必须按照本章中描述的步骤来正确的检查和修复方法执行期间可能产生的异常。

## 6.1 概述

我们通过一些列的例子来介绍JNI异常处理函数。

### 6.1.1 在本地代码中缓存和抛出异常

  下面的程序显示如何定义一个会抛出异常的本地方法。CatchThrow类定义了一个doit方法，并且表明该方法会抛出一个IllegalArgumentException：

```
class CatchThrow { 
    private native void doit() throws IllegalArgumentException;
    private void callback() throws NullPointerException { 
        throw new NullPointerException("CatchThrow.callback");
    }

    public static void main(String args[]) { 
        CatchThrow c = new CatchThrow(); 
        try {
            c.doit();
        } catch (Exception e) { 
            System.out.println("In Java:\n\t" + e);
        }
    }
 
    static { 
        System.loadLibrary("CatchThrow"); 
    } 
}
```

CatchThrow.main方法调用本地方法doit，doit的实现如下：

```
JNIEXPORT void JNICALL Java_CatchThrow_doit(JNIEnv *env, jobject obj) {
    jthrowable exc;
    jclass cls = (*env)->GetObjectClass(env, obj); 
    jmethodID mid = (*env)->GetMethodID(env, cls, "callback", "()V"); 
    if (mid == NULL) { 
        return;
    }
    (*env)->CallVoidMethod(env, obj, mid); 
    exc = (*env)->ExceptionOccurred(env); 
    if (exc) {
        /* We don't do much with the exception, except that 
        we print a debug message for it, clear it, and 
        throw a new exception. */ 
        jclass newExcCls;
        (*env)->ExceptionDescribe(env); 
        (*env)->ExceptionClear(env); 
        newExcCls = (*env)->FindClass(env,"java/lang/IllegalArgumentException"); 
        if (newExcCls == NULL) {
            /* Unable to find the exception class, give up. */ 
            return;
        } 
    (*env)->ThrowNew(env, newExcCls, "thrown from C code"); 
    } 
}
```

搭配本地库运行这个程序可以得到如下输出：

```
java.lang.NullPointerException:
    at CatchThrow.callback(CatchThrow.java) 
    at CatchThrow.doit(Native Method) 
    at CatchThrow.main(CatchThrow.java)
In Java: 
    java.lang.IllegalArgumentException: thrown from C code
```

  这个回调方法抛出一个NullPointerException异常。当CallVoidMethod将控制权返回给本地方法后，本地代码通过JNI方法ExceptionOccurred会检测到这个异常。在我们的例子当中，当异常被检测到，本地方法通过调用ExceptionDescribe会输出一个关于这个异常的描述性信息，使用ExceptionClear方法清除这个异常并且抛出一个IllegalArgumentException异常作为替代。

  通过JNI（例如通过调用ThrowNew）引起的挂起异常不会立刻破坏本地方法的执行。这和Java编程语言中异常的行为是不同的。当使用Java编程语言抛出一个异常的时候，Java虚拟机会自动将控制流程转移到最近的符合异常类型的try/catch代码块中。然后Java虚拟机会清除这个挂起的异常并执行异常处理。相比之下，在异常发生之后，JNI程序员必须显示的进行流程控制。

### 6.1.2 一个有用的辅助函数

  抛出一个异常，首先需要查找这个异常的类然后调用ThrowNew方法。为了简化这个任务，我们可以编写一个抛出一个命名异常的有用函数：

```
void JNU_ThrowByName(JNIEnv *env, const char *name, const char *msg) {
    jclass cls = (*env)->FindClass(env, name); 
    /* if cls is NULL, an exception has already been thrown */ 
    if (cls != NULL) {
        (*env)->ThrowNew(env, cls, msg); 
    }
    /* free the local ref */ 
    (*env)->DeleteLocalRef(env, cls);
}
```

  在本书中，JNU表示JNI Utilities。JNU_ThrowByName首先通过FindClass方法找到异常的类。如果FindClass调用失败（返回NULL），虚拟机必须抛出一个异常（例如NoClassDefFoundError）。在本次JNU_ThrowByName不会尝试抛出另外一个异常。如果FindClass成功返回，我们通过调用ThrowNew抛出一个命名的异常。当JNU_ThrowByName调用返回的时，它保证有一个挂起的异常，尽管这个挂起的异常不一定是由name参数指定的。在这个方法中，我们确保删除引用异常类的本地引用。如果FindClass失败并返回NULL，将NULL传递给DeleteLocalRef将会是一个空操作，这将是一个恰当的操作。

## 6.2 恰当的异常处理

  JNI程序员必须遇见所有可能的异常情况并且编写相应的代码检查和处理这些情况。适当的异常处理有时候是乏味的，但是为了提高程序的鲁棒性却是必须的。

### 6.2.1 异常检查

有两种方式检查是否有错误产生了。

（1）大多数JNI方法使用一个明显的返回值（例如NULL）来表明产生了一个错误。返回错误值也意味着在当前线程中产生了一个挂起的异常。（在返回值中编码错误情况是C中的常见用法）

下面的例子中表明GetFieldID返回NULL值来检查错误情况。例子包含两个部分：类Window定义了一些实例字段（handle，length和width）并且有一个本地方法用来缓存这些字段的字段ID。尽管这些字段确实已经在Window类中了，我们仍然需要检查GetFieldID可能返回的错误值，因为虚拟机可能无法分配足够的内容用于保存字段ID。

```
/* a class in the Java programming language */ 
public class Window { 
    long handle; 
    int length; 
    int width;
    static native void initIDs(); 
    static {
        initIDs(); 
    } 
}

/* C code that implements Window.initIDs */ 
jfieldID FID_Window_handle; 
jfieldID FID_Window_length; 
jfieldID FID_Window_width;
JNIEXPORT void JNICALL Java_Window_initIDs(JNIEnv *env, jclass classWindow) {
    FID_Window_handle =(*env)->GetFieldID(env, classWindow, "handle", "J"); 
    if (FID_Window_handle == NULL) { /* important check. */ 
        return; /* error occurred. */
    } 
    FID_Window_length =(*env)->GetFieldID(env, classWindow, "length", "I"); 
    if (FID_Window_length == NULL) { /* important check. */ 
        return; /* error occurred. */
    } 
    FID_Window_width = (*env)->GetFieldID(env, classWindow, "width", "I");   
    /* no checks necessary; we are about to return anyway */
}
```

（2）当使用一个JNI方法，它的返回值无法标记一个错误的产生的时候，本地代码就必须依赖引起异常来进行错误检查。在当前线程中，用于检查是否有挂起的异常的JNI函数是ExceptionOccurred。（ExceptionCheck在Java 2 JDK 1.2版本中加入。）例如，JNI方法CallIntMethod不能通过编码一个错误情况来作为返回值，典型的错误情况返回值例如-1和NULL都不能很好的工作，因为当他们调用这个方法时，这些都是合理的返回值。考虑有一个Fraction类，它的floor方法返回分数值的整数部分并且有其他的本地代码调用这个函数。

```
public class Fraction {
    // details such as constructors omitted 
    int over, under; 
    public int floor() {
        return Math.floor((double)over/under); 
    } 
}

/* Native code that calls Fraction.floor. Assume method ID 
MID_Fraction_floor has been initialized elsewhere. */ 
void f(JNIEnv *env, jobject fraction) {
    jint floor = (*env)->CallIntMethod(env, fraction, MID_Fraction_floor);
    /* important: check if an exception was raised */ 
    if ((*env)->ExceptionCheck(env)) { 
        return;
    } 
    ... /* use floor */ 
}
```

当JNI函数返回不同的错误代码是，本地代码仍然可能通过显示调用类检查异常，例如ExceptionCheck。不管怎么样，通过检查不同的返回值仍然是高效的。如果一个JNI方法返回其错误值，那么在当前线程后续处理中调用ExceptionCheck方法将保证返回JNI_TRUE。

### 6.2.2 异常处理

本地代码可以通过两种方式处理挂起的异常：

1. 本地代码实现能够选择立即返回，在方法调用者处进行异常处理
2. 本地代码可以通过调用ExceptionClear来清除异常然后执行它自己的异常处理函数

  在调用任何后续JNI函数之前，检查、处理和清除挂起的异常时非常重要的。在带有挂起的异常，尚未明确清理的异常时调用大多数JNI方法都有可能导致意外的结果。当在当前线程中有一个挂起的异常时，你仅可以安全的调用一小部分JNI方法，11.8.2节列出了这些JNI函数的完整列表。一般来说，当存在一个挂起的异常时，你可以调用专门的JNI函数来处理异常和释放通过JNI暴露出来的各种虚拟机资源。

  当异常发生时，经常有必要去释放各种资源。在下面的例子当中，本地方法首先通过一个GetStringChars调用获取字符串的内容。如果在后续的处理中产生错误，它会调用ReleaseStringChars：

```
JNIEXPORT void JNICALL
Java_pkg_Cls_f(JNIEnv *env, jclass cls, jstring jstr) 
{
    const jchar *cstr = (*env)->GetStringChars(env, jstr); 
    if (c_str == NULL) { 
        return;
    } 
    ...
    if (...) { /* exception occurred */ 
        (*env)->ReleaseStringChars(env, jstr, cstr); 
        return;
    } 
    ... 
    /* normal return */ 
    (*env)->ReleaseStringChars(env, jstr, cstr); 
}
```

  第一次调用ReleaseStringChars是在有挂起的线程出现的时候。本地方法实现释放字符串资源并在之后立即返回，而不需要首先清除异常。

### 6.2.3 有用的辅助函数中的异常

  程序员编写有用的辅助函数时应特别注意确保异常传播到本地调用方法中。我们特别强调一下两点：

- 优选方案，辅助函数应该提供特殊的返回值指示发生了异常。这简化了调用者检查待处理异常的任务。
- 此外，辅助函数应遵循在管理异常代码是注意管理本地应用的规则。

为了说明，让我们介绍一个基于实例方法的名字和描述符执行回调的辅助函数：

```
jvalue JNU_CallMethodByName(JNIEnv *env, jboolean *hasException, jobject obj, const char *name, const char *descriptor, ...)
{
    va_list args;
    jclass clazz;
    jmethodID mid;
    jvalue result;
    if((*env)->EnsureLocalCapacity(env, 2) == JNI_OK) {
        clazz = (*env)->GetObjectClass(env, obj);
        mid = (*env)->GetMethodID(env, clazz, name, descriptor);
        if(mid) {
            const char *p = descriptor;
            /* skip over argument types to find out the 
               return type */
            while (*p != ')') p++; 
            /* skip ')' */ 
            p++;
      va_start(args, descriptor); 
      switch (*p) { 
            case 'V':
                (*env)->CallVoidMethodV(env, obj, mid, args); 
                break;
                
            case '[': 
            case 'L':
                result.l = (*env)->CallObjectMethodV(env, obj, mid, args);
                break;
                
            case 'Z':
                result.z = (*env)->CallBooleanMethodV(env, obj, mid, args);
                break; 
                
            case 'B':
                result.b = (*env)->CallByteMethodV(env, obj, mid, args);
                break;
                
            case 'C': 
                result.c = (*env)->CallCharMethodV(env, obj, mid, args); 
                break; 
                
            case 'S': 
                result.s = (*env)->CallShortMethodV(env, obj, mid, args); 
                break; 
                
            case 'I': 
                result.i = (*env)->CallIntMethodV(env, obj, mid, args); 
                break; 
            
            case 'J':
                result.j = (*env)->CallLongMethodV(env, obj, mid, args); 
                break;
                
            case 'F': 
                result.f = (*env)->CallFloatMethodV(env, obj, mid, args); 
                break; 
                
            case 'D': 
                result.d = (*env)->CallDoubleMethodV(env, obj, mid, args); 
                break; 
                
            default: 
                (*env)->FatalError(env, "illegal descriptor"); 
            } 
            va_end(args); 
        } 
        (*env)->DeleteLocalRef(env, clazz); 
    }
    if (hasException) {
        *hasException = (*env)->ExceptionCheck(env);
    }
    return result
}
```

  除了其他参数以外，JNU_CallMethodByName还有一个指向jboolean的指针。如果在一切正常，jboolean将被设置为JNI_FALSE，如果在执行这个函数期间的任何时候发生了异常，那么jboolean将被设置为JNI_TRUE。这将给JNU_CallMethoByName的调用者一个明显的方法去检查是否有发生异常。

  JNU_CallMethodByName首先确保他能够创建两个本地引用：一个用于类引用，另一个用于方法调用返回的结果。接下来，它从对象获取到类引用，并查找到方法ID。根据返回值的类型，switch语句将调度到相应的JNI方法调用函数。回调返回后，如果hasException不为NULL，我们调用ExceptionCheck来检查挂起的异常。

  ExceptionCheck方法是在Java 2 SDK 1.2中新加进去的。它类似于ExceptionOccurred函数。不同之处在于ExceptionCheck不会返回对异常对象的引用，但是当有挂起的异常时返回JNI_TRUE，在没有挂起的异常的时候放回JNI_FALSE。当本地代码仅需要知道是否发生异常但是不需要获取对异常对象的引用时，ExceptionCheck简化了本地引用的管理。前面的代码在使用JDK 1.1中，将重写如下：

```
if (hasException) {
    jthrowable exc = (*env)->ExceptionOccurred(env); 
    *hasException = exc != NULL; 
    (*env)->DeleteLocalRef(env, exc);
}
```

  额外的DeleteLocalRef调用时必须的，以删除对异常对象的本地引用。

  使用JNU_CallMethodByName方法，我们可以重写4.2节中InstanceMethodCall.nativeMethod的实现：

```
JNIEXPORT void JNICALL
Java_InstanceMethodCall_nativeMethod(JNIEnv *env, jobject obj) {
    printf("In C\n"); 
    JNU_CallMethodByName(env, NULL, obj, "callback", "()V"); 
}
```

在JNU_CallMethodByName调用会我们不需要检查异常，因为本地代码会立刻返回。



# 第七章：调用接口

这一章用于说明在你的本地代码中如何嵌入一个Java虚拟机。Java虚拟机实现通过作为一个本地库来传输,本地应用程序可以连接此库并使用调用接口来加载Java虚拟机。的确，在JDK或Java 2 SDK版本中的标准启动器指令只不过是一个和Java虚拟机链接的简单c程序。启动器解析命令行参数、加载虚拟机、并通过调用借口运行Java程序。

## 7.1 创建Java虚拟机

为了说明调用借口，让我们先看一个加载一个Java虚拟机并调用按照如下定义的Prog.main方法的C程序

```
public class Prog {
    public static void main(String[] args) { 
        System.out.println("Hello World " + args[0]);
    }
}
```

接下来的C程序，invoke.c，加载一个Java虚拟机并调用Prog.main方法

```
#include <jni.h>
#define PATH_SEPARATOR ';' /* define it to be ':' on Solaris */ 
#define USER_CLASSPATH "." /* where Prog.class is */ 
main() {
    JNIEnv *env; 
    JavaVM *jvm; 
    jint res; 
    jclass cls; 
    jmethodID mid; 
    jstring jstr;
    jclass stringClass; 
    jobjectArray args;
#ifdef JNI_VERSION_1_2 
    JavaVMInitArgs vm_args; 
    JavaVMOption options[1]; 
    options[0].optionString = "-Djava.class.path=" USER_CLASSPATH; 
    vm_args.version = 0x00010002; 
    vm_args.options = options; 
    vm_args.nOptions = 1;
    vm_args.ignoreUnrecognized = JNI_TRUE; 
    /* Create the Java VM */
    res = JNI_CreateJavaVM(&jvm, (void**)&env, &vm_args); 
#else
    JDK1_1InitArgs vm_args; 
    char classpath[1024]; 
    vm_args.version = 0x00010001; 
    JNI_GetDefaultJavaVMInitArgs(&vm_args); 
    /* Append USER_CLASSPATH to the default system class path */ 
    sprintf(classpath, "%s%c%s", vm_args.classpath, PATH_SEPARATOR, USER_CLASSPATH); 
    vm_args.classpath = classpath; 
    /* Create the Java VM */
    res = JNI_CreateJavaVM(&jvm, &env, &vm_args); 
#endif /* JNI_VERSION_1_2 */

    if (res < 0) {
        fprintf(stderr, "Can't create Java VM\n"); 
        exit(1);
    }

    cls = (*env)->FindClass(env, "Prog"); 
    if (cls == NULL) { 
        goto destroy;
    }

    mid = (*env)->GetStaticMethodID(env, cls, "main", "([Ljava/lang/String;)V");
    if (mid == NULL) { 
        goto destroy;
    }

    jstr = (*env)->NewStringUTF(env, " from C!"); 
    if (jstr == NULL) { 
        goto destroy;
    }

    stringClass = (*env)->FindClass(env, "java/lang/String"); 
    args = (*env)->NewObjectArray(env, 1, stringClass, jstr); 
    if (args == NULL) { 
        goto destroy;
    } 

    (*env)->CallStaticVoidMethod(env, cls, mid, args); 

destroy:
    if ((*env)->ExceptionOccurred(env)) { 
        (*env)->ExceptionDescribe(env);
    } 
    (*jvm)->DestroyJavaVM(jvm);
```

  上面的代码条件性的编译特定于JDK 1.1版本的Java虚拟机实现的初始化结构体JDK1_1InitArgs。Java 2 SDK 1.2版本中仍旧可以支持JDK1_1InitArgs，尽管其引入了一种称为JavaVMInitArgs的初始化结构体。常量JAVA_VERSION_1_2在Java 2 SDK 1.2版本中定义，而在JDK 1.1版本中是没有定义的。

  当它针对的是1.1版本时，C代码通过调用JNI_GetDefaultJavaVMInitArgs来获取默认的虚拟机设置。JNI_GetDefaultJavaVMInitArgs在vm_args参数中返回诸如堆大小、栈大小、默认类路径等值。然后我们追加Prog.class所在的目录到vm_args.classpath中。

  当它是针对的是1.2版本时，C代码创建一个JavaVMInitArgs结构体。虚拟机初始化结构体保存在JavaVMOption数组当中，你可以设置和Java命令行选项对应的常规选项（如-Djava.class.path=.）和虚拟机实现特定的选项（如-Xmx64m)。设置ignoreUnrecognized字段为JNI_TRUE说明虚拟机忽略不能识别的虚拟机特定选项。

  在设置好虚拟机初始化结构体后，C程序调用JNI_CreateJavaVM来加载和初始化Java虚拟机。JNI_CreateJavaVM将填充两个返回值：

- jvm，指向新创建的Java虚拟机的接口指针
- env，指向当前线程的JNIEnv的接口指针，本地方法将通过env接口指针来调用JNI方法

  当JNI_CreateJavaVM成功返回时，当前本地线程就已经将自身引导到Java虚拟机中。在这个点上，它就像一个本地方法一样运行，因此，除了别的以外，它可以发出JNI调用来调用Prog.main方法。

  最终程序会调用DestroyJavaVM来卸载Java虚拟机。（不幸的是，你不能在JDK 1.1版本和Java 2 SDK 1.2版本中卸载Java虚拟机实现，在这些版本中，DestroyJavaVM总是返回错误代码。）运行上面的程序，将得到:

`Hello World from C!`

## 7.2 将本地应用程序与Java虚拟机相连

调用接口需要你将你的程序例如invoke.c，和一个Java虚拟机实现相连。如何和Java虚拟机相连取决于本地引用程序是仅部署在特定的Java虚拟机实现还是它被设计为与来自不同供应商的各种虚拟机实现一起工作。

### 7.2.1 与已知的Java虚拟机链接

你可能决定了你的程序仅部署在特定的Java虚拟机实现上。在这种情况下，你可以将本地应用程序和实现了虚拟机的本地库相连，例如在Solaris系统，JDK 1.1版本中，你可以使用下面的指令来编译和链接invoke.c：

```
cc -I -L -lthread -ljava invoke.c
```

  -lthread选项表明我们使用Java虚拟机以及本地线程支持（8.1.5节）。-ljava选项表明libjava.so是实现了Java虚拟机的Solaris共享库。

  在Win32系统使用Microsoft Visual C++编译器，编译和链接同一个程序的命令行为

```
cl -I -MD invoke.c -link <javai.lib dir>\javai.lib
```

  当然你需要提供与机器上的JDK安装相对应的正确的包含和库目录。-MD选项确保你的本地应用程序和Win32多线程C库链接，在JDK 1.1版本和Java 2 JDK 1.2版本的Java虚拟机实现使用的是同一个本地C库。cl命令参考Win32中附带的JDK版本1.1中的javai.lib文件，以获取有关在虚拟机中实现的调用接口函数（JNI_CreateJavaVM）的链接信息。在运行时真实使用的JDK 1.1虚拟机实现被包含在一个名为javai.dll的单独动态链接库中。相反，在链接时和运行时都是用相同的Solaris动态库（.so文件）。

  在Java 2 JDK 1.2版本中，虚拟机库的名字已更改为Solaris上的libjvm.so，以及Win32上的jvm.lib和jvm.dll。通常不同的供应商可能会以不同的方式命名虚拟机实现。

  一旦编译和链接完成，你可以从命令行运行生成的可执行文件。你可能会收到系统找不到共享库或动态链接库的错误。在Solaris系统上，如果错误信息显示系统找不到动态库libjava.so（libjvm.so在Java 2 JDK 1.2版本上），则需要将包含虚拟机库的目录添加到LD_LIBRARY_PATH变量中。在Win32系统上，错误信息可能表示为它找不到动态链接库javai.dll（或者jvm.dll在Java 2 JDK 1.2版本上），如果是这种情，请将包含该DLL的目录添加到PATH环境变量中。

### 7.2.2 与未知的Java虚拟机链接

如果应用程序旨在使用来自不同供应商的虚拟机实现，则无法将本地应用程序链接到特定的虚拟机实现库。因为JNI没有指定实现Java虚拟机的本地库的名称，所以你应该准备好使用不同的Java虚拟机实现。例如，在Win32上，虚拟机在JDK版本1.1中作为javai.dll发布，在Java 2 SDK版本1.2中作为jvm.dll发布。

  解决方案是使用运行时动态链接来加载应用程序所需的特定虚拟机库。虚拟机库的名称可以轻松的以应用程序特定的方式配置，例如以下Win32代码为虚拟机库的路径找到JNI_CreateJavaVM的函数入口点：

```
/* Win32 version */
void *JNU_FindCreateJavaVM(char *vmlibpath) {
    HINSTANCE hVM = LoadLibrary(vmlibpath); 
    if (hVM == NULL) { 
        return NULL;
    } 
    return GetProcAddress(hVM, "JNI_CreateJavaVM"); 
}
```

  LoadLibrary和GetProcAddreee是在Win32系统上，用于动态链接的API函数。尽管LoadLibrary能够接受实现了Java虚拟机的本地库的名字（例如“jvm”）或者路径（例如”C:\jdk1.2\jre\bin\classic\jvm.dll”），传送本地库绝对路劲给JNU_FindCreateJavaVM将是最好的选择。依靠LoadLibrary来搜索jvm.dll，使您的应用程序易于进行配置更改，例如PATH环境变量的添加。

  Solaris系统的版本是类似的：

```
/* Solaris version */
void *JNU_FindCreateJavaVM(char *vmlibpath) {
    void *libVM = dlopen(vmlibpath, RTLD_LAZY); 
    if (libVM == NULL) { 
        return NULL;
    } 
    return dlsym(libVM, "JNI_CreateJavaVM"); 
}
```

  dlopen和dlsym函数支持在Solaris系统上动态链接共享库。

## 7.3 附加本地线程

假设你有一个多线程应用程序，例如一个用C写的Web服务器。随着HTTP请求的到来，服务器创建一些本地线程来同时处理HTTP请求。我们希望在此服务器中嵌入一个Java虚拟机，以便多个线程可以同时在Java虚拟机中执行操作，如图7.1所示：

![《(译文) JNI编程指南与规范 第七章 调用接口》](https://i1.wp.com/blog4jimmy.com/wp-content/uploads/2017/10/JNI3.3.png)

  服务器生成的本地方法的生命周期可能比Java虚拟机更短。因此，我们需要一种方式来将本地线程附加到已经运行的Java虚拟机，在附加的本地线程中执行JNI调用，然后将本地线程从Java虚拟机中脱离不会其他已连接的线程。接下来的示例，attach.c，说明如何使用调用接口将本地线程附加到虚拟机上，改程序是使用Win32线程API编写的。可以为Solaris和其他操作系统编写类似的版本：

```
/* Note: This program only works on Win32 */ 
#include <windows.h> 
#include <jni.h>
JavaVM *jvm; /* The virtual machine instance */
 
#define PATH_SEPARATOR ';'
#define USER_CLASSPATH "." /* where Prog.class is */

void thread_fun(void *arg) {
    jint res; 
    jclass cls; 
    jmethodID mid; 
    jstring jstr;
    jclass stringClass; 
    jobjectArray args; 
    JNIEnv *env; 
    char buf[100];
    int threadNum = (int)arg;

    /* Pass NULL as the third argument */ 
#ifdef JNI_VERSION_1_2 
    res = (*jvm)->AttachCurrentThread(jvm, (void**)&env, NULL);
#else 
    res = (*jvm)->AttachCurrentThread(jvm, &env, NULL);
#endif
    if (res < 0) {
        fprintf(stderr, "Attach failed\n"); 
        return;
    }

    cls = (*env)->FindClass(env, "Prog"); 
    if (cls == NULL) { 
        goto detach;
    } 

    mid = (*env)->GetStaticMethodID(env, cls, "main", "([Ljava/lang/String;)V");
    if (mid == NULL) { 
        goto detach;
    }

    sprintf(buf, " from Thread %d", threadNum); 
    jstr = (*env)->NewStringUTF(env, buf); 
    if (jstr == NULL) { 
        goto detach;
    }

    stringClass = (*env)->FindClass(env, "java/lang/String"); 
    args = (*env)->NewObjectArray(env, 1, stringClass, jstr); 
    if (args == NULL) { 
        goto detach;
    }
    (*env)->CallStaticVoidMethod(env, cls, mid, args);
 
detach:
    if ((*env)->ExceptionOccurred(env)) { 
        (*env)->ExceptionDescribe(env);
    } 
    (*jvm)->DetachCurrentThread(jvm); 
} 

main() {
    JNIEnv *env; 
    int i;
    jint res;

#ifdef JNI_VERSION_1_2 
    JavaVMInitArgs vm_args; 
    JavaVMOption options[1]; 
    options[0].optionString = "-Djava.class.path=" USER_CLASSPATH; 
    vm_args.version = 0x00010002; 
    vm_args.options = options; 
    vm_args.nOptions = 1;
    vm_args.ignoreUnrecognized = TRUE; /* Create the Java VM */
    res = JNI_CreateJavaVM(&jvm, (void**)&env, &vm_args); 
#else
    JDK1_1InitArgs vm_args; 
    char classpath[1024]; 
    vm_args.version = 0x00010001; 
    JNI_GetDefaultJavaVMInitArgs(&vm_args); 
    /* Append USER_CLASSPATH to the default system class path */ 
    sprintf(classpath, "%s%c%s", vm_args.classpath, PATH_SEPARATOR, USER_CLASSPATH); 
    vm_args.classpath = classpath; 
    /* Create the Java VM */
    res = JNI_CreateJavaVM(&jvm, &env, &vm_args); 
#endif /* JNI_VERSION_1_2 */

    if (res < 0) {
        fprintf(stderr, "Can't create Java VM\n"); 
        exit(1);
    } 
    for (i = 0; i < 5; i++)
        /* We pass the thread number to every thread */ 
        _beginthread(thread_fun, 0, (void *)i); 
        Sleep(1000); /* wait for threads to start */ 
        (*jvm)->DestroyJavaVM(jvm);
}
```

  attach.c是invoke.c的变体，本地代码不是在主线程中调用Prog.main，而是启动五个线程。一旦它启动线程，它等待他们启动然后调用DestroyJavaVM。每个产生的线程都将自己链接到Java虚拟机，调用Prog.main方法，最后在其终止运行前将其从Java虚拟机中分离出来。DestroyJavaVM将在所有五个线程终止后返回。现在我们先忽略DestroyJavaVM的返回值，因为这个方法在JDK 1.1版本和Java 2 JDK 1.2版本上并没有完全实现。

  JNI_AttachCurrentThread将NULL作为其第三个参数，Java 2 JDK 1.2版本中引入JNI_ThreadAttachArgs结构体，它允许你指定其他参数，例如你要附加的线程组。JNI_ThreadAttachArgs结构体的细节将在13.2节中作为JNI_AttachCurrentThread规范的一部分进行描述。

  当程序执行DetachCurrentThread函数时，它将释放属于当前线程的所有本地引用。运行程序会产生以下输出：

```
Hello World from thread 1 
Hello World from thread 0 
Hello World from thread 4 
Hello World from thread 2 
Hello World from thread 3
```

  输出的确切顺序可能会根据线程调度中的随机因素而变化。



## JNI实现原理

JNI系列：JavaVM和JNIEnv等原理
http://blog4jimmy.com/2017/11/242.html
http://blog4jimmy.com/category/the_java_native_interface_programmer_guide_and_specificationjni
http://blog.guorongfei.com/2017/01/24/android-jni-tips-md/
https://www.cnblogs.com/fnlingnzb-learner/p/7366025.html
https://www.zybuluo.com/cxm-2016/note/566619
https://blog.csdn.net/omnispace/article/details/73320940
反射：
http://blog4jimmy.com/2017/11/224.html

http://androidxref.com/8.0.0_r4/xref/libnativehelper/include/nativehelper/jni.h

插件机制：
https://github.com/tiann/epic/tree/master/library/src/main/cpp
https://blog.csdn.net/omnispace/article/details/73320940

 JNI的实现可涉及两个关键类:JNIEnv和JavaVM。
JavaVM：这个代表java的虚拟机。所有的工作都是从获取虚拟机的接口开始的。
            第一种方式，在加载动态链接库的时候，JVM会调用JNI_OnLoad(JavaVM* jvm, void* reserved)（如果定义了该函数）。第一个参数会传入JavaVM指针。
            第二种方式，在native code中调用JNI_CreateJavaVM(&jvm, (void**)&env, &vm_args)可以得到JavaVM指针。
            两种情况下，都可以用全局变量，比如JavaVM* g_jvm来保存获得的指针以便在任意上下文中使用。
            Android系统是利用第二种方式Invocation interface来创建JVM的。
     
JNIEnv：JNI Interface Pointer, 是提供JNI Native函数的基础环境，线程相关，不同线程的JNIEnv相互独立。
            JNIEnv只在当前线程中有效。本地方法不 能将JNIEnv从一个线程传递到另一个线程中。相同的 Java 线程中对本地方法多次调用时，传递给该本地方法的JNIEnv是相同的。但是，一个本地方法可被不同的 Java 线程所调用，因此可以接受不同的 JNIEnv。

         JavaVM则可以在进程中的各线程间共享。理论上一个进程可以有多个JavaVM,但Android只允许一个（JavaVm and JIEnv）。需要强调的是JNIEnv是跟线程相关的。sdk文档中强调了do not cache JNIEnv*，要用的时候在不同线程中再通过JavaVM *jvm的方法来获取与当前线程相关的JNIEnv*。两者都可以理解为函数表(Function Pointer Table), 前者是使用Java程序创建的运行环境(从属于一个JVM)提供JNI Native函数。

注意点：
http://www.10tiao.com/html/330/201711/2653579453/1.html

pthread_key:
https://zhuanlan.zhihu.com/p/33411235

http://blog.csdn.net/zsl_oo7/article/details/71081291
http://wiki.jikexueyuan.com/project/jni-ndk-developer-guide/function.html

```
## 调用Java层的方法

1.  通过 `jclass clazz = env->FindClass("含有路径的类名");` 找到类
2.  通过 `jmethodID mid = env->GetMethodID(clazz,"方法名","方法签名信息");`找到Java层方法的ID 

    *   注意 jmethodID 是一个专门记录 Java 层方法的类型
    *   类似的还有一个 jfieldID
3.  通过 `env->CallxxxMethod(jobj,mid,param1,param2...);` 调用 Java 层的方法 

    *   CallxxxMethod 中的 xxx 是 Java 方法的返回值类型，比如 CallVoidMethod，CallIntMethod
    *   第一个参数是指调用哪个对象的方法，就是 Java 中`.`前面的那个对象
    *   第二个参数 Java 中的 MethodID
    *   后面的参数就是 Java 方法的参数了，其类型都要是 java 中能处理的类型，比如 jstring，jint，jobject

## get和set Java层的field

1.  通过 `jclass clazz = env->FindClass("含有路径的类名");` 找到类
2.  通过 `jfieldID fid = env->GetFieldID(clazz,"成员名","成员类型标示");`找到Java层成员变量的ID
3.  通过 `GetxxxField(env,obj,fid);` / `SetxxxField(env,obj,fid,value);` 来get/set相应的成员变量

```