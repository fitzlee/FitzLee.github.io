---
title: Android性能-内存泄漏
urlname: android_app_perfermance_oom
date: 2017-04-29
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 10Android_App
tags:
    - 书籍推荐
    - Android
    - oom
    - 内存泄漏
---

泄漏原因： 
* 单例造成的内存泄漏
```
public class AppManager {
    private static AppManager instance;
    private Context context;
    private AppManager(Context context) {
        this.context = context;
    }
    public static AppManager getInstance(Context context) {
        if (instance != null) {
            instance = new AppManager(context);
        }
        return instance;
    }
}
```
* 非静态内部类(匿名类等)创建静态实例造成的内存泄漏 Handler/Thread/AsyncTas等
可以使用静态内部类+弱引用避免泄漏
```
public class MainActivity extends AppCompatActivity {

    private static TestResource mResource = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        if(mResource == null){
            mResource = new TestResource();
        }
        //...
    }
    
    class TestResource {
    //...
    }
}
```
* 资源未关闭造成的内存泄漏

对于使用了BraodcastReceiver，ContentObserver，File，Cursor，Stream，Bitmap, 属性动画或循环动画等资源，应该在Activity销毁时及时关闭或者注销，否则这些资源将不会被回收，从而造成内存泄漏。
其中Bitmap使用不当,bitmap对象使用的内存较大，当我们不再使用Bitmap对象的时候一定要执行recycler方法，这里需要指出的是当我们在代码中执行recycler方法，Bitmap并不会被立即释放掉，其只是通知虚拟机该Bitmap可以被recycler了。

* 集合容器中的内存泄露
我们通常把一些对象的引用加入到了集合容器（比如ArrayList）中，当我们不需要该对象时，并没有把它的引用从集合中清理掉，这样这个集合就会越来越大。如果这个集合是static的话，那情况就更严重了。
解决方法：在退出程序之前，将集合里的东西clear，然后置为null，再退出程序。

* 静态变量持有的应用
view持有context的



https://blog.csdn.net/north1989/article/details/51999920
https://blog.csdn.net/u013495603/article/details/50696170
https://blog.csdn.net/mxm691292118/article/details/51020023
