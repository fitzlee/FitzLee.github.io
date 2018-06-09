---
title: Android中间件-插件化和热补丁等
urlname: android_app_plugin_hotfix
date: 2017-04-10
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 10Android_App
tags:
    - 书籍推荐
    - Android
    - Plugin
    - hotfix
---

应用在Google市场上架之前会经过安全扫描，包含病毒的应用无法上架。目前安卓应用可以通过下载并动态加载dex/jar/elf/的方式，进行升级，以达到以下目的：问题修复，版本升级等。这都是热补丁的正常使用方式。除此之外，个别恶意应用利用热补丁技术，故意使上架检测版本和实际运行版本不一致，恶意绕过上架检测机制。

提供的动态加载接口有Dexclassloader、Desfile.loadDex、System.load、System.loadlibrary，它们底层在虚拟机的实现接口是openDexFileNative和JVM_NativeLoad，分别用于加载dex/jar和so格式的文件，应用通过调用这些接口实现动态加载。

那么热补丁插件的实现方式大概有以下几种：
##Android中间件
###插件化  
http://weishu.me/2016/01/28/understand-plugin-framework-overview/
https://github.com/wangwangheng/BestBlogReprinted_AndroidNotes/tree/master/%E6%8F%92%E4%BB%B6%E5%BC%8F%E5%BC%80%E5%8F%91/weishu%E7%B3%BB%E5%88%97
http://blog.csdn.net/ganyao939543405/article/details/76146760
https://cloud.tencent.com/developer/article/1038868
http://www.10tiao.com/html/227/201703/2650239063/1.html
https://github.com/tiann/understand-plugin-framework
http://www.10tiao.com/html/227/201703/2650239063/1.html
https://github.com/prife/VirtualAppDoc

###热修复 
http://www.androidchina.net/6213.html
https://github.com/Tencent/tinker/wiki
https://www.cnblogs.com/popfisher/p/8543973.html
深入探索Android热修复技术原理 by 阿里 pdf
AndFix原理 https://blog.csdn.net/jiangwei0910410003/article/details/53099390

###Hook框架 
http://www.snowdream.tech/2016/09/02/android-install-xposed-framework/
https://jaq.alibaba.com/community/art/show?articleid=809
##VirtualXposed 和epic
https://github.com/android-hacker/VirtualXposed
https://github.com/tiann/epic
http://weishu.me/2017/12/02/non-root-xposed/

classLoader的问题
https://blog.csdn.net/xiangzhihong8/article/details/52880327
http://weishu.me/2016/04/05/understand-plugin-framework-classloader/

插件的原理
https://github.com/wangwangheng/BestBlogReprinted_AndroidNotes/blob/master/%E6%8F%92%E4%BB%B6%E5%BC%8F%E5%BC%80%E5%8F%91/weishu%E7%B3%BB%E5%88%97/1.Android%E6%8F%92%E4%BB%B6%E5%8C%96%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90%E2%80%94%E2%80%94%E6%A6%82%E8%A6%81.md
首先要明白静态代理/动态代理/hook原理机制，静态代理在程序运行前，代理类的.class文件就已经存在了。动态代理类：在程序运行时，运用JDK本身反射机制动态创建而成，如果复杂可能要用cglib；创建出来的对象都集成的相同的接口。动态代理主要设计以下几个方法
```
Shopping women = new ShoppingImpl();
// 正常购物
System.out.println(Arrays.toString(women.doShopping(100)));
// 招代理
women = (Shopping) Proxy.newProxyInstance(Shopping.class.getClassLoader(),
                women.getClass().getInterfaces(), new ShoppingHandler(women));
System.out.println(Arrays.toString(women.doShopping(100)));

public class ShoppingHandler implements InvocationHandler {
    Object base;
    public ShoppingHandler(Object base) {
        this.base = base;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
       if ("doShopping".equals(method.getName())) {
        System.out.println(String.format("花了%s块钱", readCost));
        Object[] things = (Object[]) method.invoke(base, readCost);
       } else if {}
    }
}
```

但是hook相当于直接改变已有类的静态成员或者单例，hook常用发射的方法，来替换成员达到修改相关类的效果。如下方法：
```
public static void attachContext() throws Exception{
        // 先获取到当前的ActivityThread对象
        Class<?> activityThreadClass = Class.forName("android.app.ActivityThread");
        Method currentActivityThreadMethod = activityThreadClass.getDeclaredMethod("currentActivityThread");
        currentActivityThreadMethod.setAccessible(true);
        //currentActivityThread是一个static函数所以可以直接invoke，不需要带实例参数
        Object currentActivityThread = currentActivityThreadMethod.invoke(null);

        // 拿到原始的 mInstrumentation字段
        Field mInstrumentationField = activityThreadClass.getDeclaredField("mInstrumentation");
        mInstrumentationField.setAccessible(true);
        Instrumentation mInstrumentation = (Instrumentation) mInstrumentationField.get(currentActivityThread);

        // 创建代理对象
        Instrumentation evilInstrumentation = new EvilInstrumentation(mInstrumentation);

        // 偷梁换柱
        mInstrumentationField.set(currentActivityThread, evilInstrumentation);
}
```

binder的hook，可以通过ServiceManager修改掉相关的其中的sCache这个变量即可，然后填相关service。
```
final String CLIPBOARD_SERVICE = "clipboard";

// 下面这一段的意思实际就是: ServiceManager.getService("clipboard");
// 只不过 ServiceManager这个类是@hide的
Class<?> serviceManager = Class.forName("android.os.ServiceManager");
Method getService = serviceManager.getDeclaredMethod("getService", String.class);
// ServiceManager里面管理的原始的Clipboard Binder对象
// 一般来说这是一个Binder代理对象
IBinder rawBinder = (IBinder) getService.invoke(null, CLIPBOARD_SERVICE);

// Hook 掉这个Binder代理对象的 queryLocalInterface 方法
// 然后在 queryLocalInterface 返回一个IInterface对象, hook掉我们感兴趣的方法即可.
IBinder hookedBinder = (IBinder) Proxy.newProxyInstance(serviceManager.getClassLoader(),
        new Class<?>[] { IBinder.class },
        new BinderProxyHookHandler(rawBinder));

// 把这个hook过的Binder代理对象放进ServiceManager的cache里面
// 以后查询的时候 会优先查询缓存里面的Binder, 这样就会使用被我们修改过的Binder了
Field cacheField = serviceManager.getDeclaredField("sCache");
cacheField.setAccessible(true);
Map<String, IBinder> cache = (Map) cacheField.get(null);
cache.put(CLIPBOARD_SERVICE, hookedBinder);
```


binder的ams hook也是通过类似的方法，替换掉了ActivityManagerNative的gDefault变量来替换。
```
Class<?> activityManagerNativeClass = Class.forName("android.app.ActivityManagerNative");

// 获取 gDefault 这个字段, 想办法替换它
Field gDefaultField = activityManagerNativeClass.getDeclaredField("gDefault");
gDefaultField.setAccessible(true);
Object gDefault = gDefaultField.get(null);

// 4.x以上的gDefault是一个 android.util.Singleton对象; 我们取出这个单例里面的字段
Class<?> singleton = Class.forName("android.util.Singleton");
Field mInstanceField = singleton.getDeclaredField("mInstance");
mInstanceField.setAccessible(true);

// ActivityManagerNative 的gDefault对象里面原始的 IActivityManager对象
Object rawIActivityManager = mInstanceField.get(gDefault);

// 创建一个这个对象的代理对象, 然后替换这个字段, 让我们的代理对象帮忙干活
Class<?> iActivityManagerInterface = Class.forName("android.app.IActivityManager");

Object proxy = Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
        new Class<?>[] { iActivityManagerInterface }, 
        		new IActivityManagerHandler(rawIActivityManager));
        		
mInstanceField.set(gDefault, proxy);
```


热修复和冷修复的原理
动态加载某一个dex或者jar包，替换有问题的类或者方法或者变量，以达到热修复的功能。热修复行不通的情况下，那么就要等待重新启动，冷启动修复对应的dex的方法或类。那么如何替换呢？
目前有两种方式，参考sophix，热修复第一种方法是底层替换原理，直接替换调ARTMethod结构体，需要适配每一个版本，比较复杂。另外一种是类加载冷启动的多dex全量替换方式，替换dexElements这个dex文件的list使得虚拟机能优先加载修复过的类，从而达到修复效果。

There are three injection points for a given method: before, after, replace.
```
Example 1: Attach a piece of code before and after all occurrences of Activity.onCreate(Bundle).

        // Target class, method with parameter types, followed by the hook callback (XC_MethodHook).
		DexposedBridge.findAndHookMethod(Activity.class, "onCreate", Bundle.class, new XC_MethodHook() {
        
            // To be invoked before Activity.onCreate().
			@Override protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
				// "thisObject" keeps the reference to the instance of target class.
				Activity instance = (Activity) param.thisObject;
        
				// The array args include all the parameters.
				Bundle bundle = (Bundle) param.args[0];
				Intent intent = new Intent();
				// XposedHelpers provide useful utility methods.
				XposedHelpers.setObjectField(param.thisObject, "mIntent", intent);
		
				// Calling setResult() will bypass the original method body use the result as method return value directly.
				if (bundle.containsKey("return"))
					param.setResult(null);
			}
					
			// To be invoked after Activity.onCreate()
			@Override protected void afterHookedMethod(MethodHookParam param) throws Throwable {
		        XposedHelpers.callMethod(param.thisObject, "sampleMethod", 2);
			}
		});
Example 2: Replace the original body of the target method.

		DexposedBridge.findAndHookMethod(Activity.class, "onCreate", Bundle.class, new XC_MethodReplacement() {
		
			@Override protected Object replaceHookedMethod(MethodHookParam param) throws Throwable {
				// Re-writing the method logic outside the original method context is a bit tricky but still viable.
				...
			}

		});

```
