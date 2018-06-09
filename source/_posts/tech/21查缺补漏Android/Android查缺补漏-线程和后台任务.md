---
title: Android查缺补漏-线程
urlname: android_review_thread
date: 2017-10-12
comments: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 21Android_review
tags:
    - Android
    - 查缺补漏
    - 线程
    - AsyncTask
    - HandlerThread
---

## Android线程的几种方式
AsyncTask/HandlerThread/Thread/IntentService
https://www.jianshu.com/p/34cffd700f75

## AsyncTask
单个线程的线程池，一个一个执行，会阻塞后边线程。可以设置 AsyncTask.executeOnExecutor()来同时执行任务。写的时候注意避免内存泄露

## HandlerThread
实质是一个Thread，不过添加了Looper和MessageQueue，这样就可以使用Handler来进行控制代码执行了。
```
// 创建一个线程，线程名字 : handlerThreadTest
        mHandlerThread = new HandlerThread("handlerThreadTest");
        mHandlerThread.start();
        
        // Handler 接收消息
        final Handler mHandler = new Handler(mHandlerThread.getLooper()) {
            @Override
            public void handleMessage(Message msg) {
                Log.e("Test", "收到 " + msg.obj.toString() + " 在 "
                        + Thread.currentThread().getName());
            }
        };
        mTextView = (TextView) findViewById(R.id.text_view);
        // 主线程发出消息
        mTextView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Message msg = new Message();
                msg.obj = "第一条信息";
                mHandler.sendMessage(msg);
                Log.e("Test", "发出 " + msg.obj.toString() + " 在 "
                        + Thread.currentThread().getName());
            }
        });
        // 子线程发出消息
        new Thread(new Runnable() {
            @Override
            public void run() {
                Message msg = new Message();
                msg.obj = "第二条信息";
                mHandler.sendMessage(msg);
                Log.e("Test", "发出 " + msg.obj.toString() + " 在 "
                        + Thread.currentThread().getName());
            }
        }).start()；  
```
但是最后不要忘记mHandlerThread.quit();否则将一直循环。另外可以在run方法里设置不同优先级android.os.Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);同样HandlerThread的构造方法也提供了设置优先级的功能，new HandlerThread("LightTaskThread", Process.THREAD_PRIORITY_BACKGROUND);AsyncTask同样设置了优先级THREAD_PRIORITY_BACKGROUND。
HandlerThread的默认优先级是Process.THREAD_PRIORITY_DEFAULT,具体值为0。线程的优先级的取值范围为-20到19。优先级高的获得的CPU资源更多，反之则越少。-20代表优先级最高，19最低。0位于中间位置，但是作为工作线程的HandlerThread没有必要设置这么高的优先级，因而需要我们降低其优先级。
THREAD_PRIORITY_DEFAULT，默认的线程优先级，值为0。
THREAD_PRIORITY_LOWEST，最低的线程级别，值为19。
THREAD_PRIORITY_BACKGROUND 后台线程建议设置这个优先级，值为10。
THREAD_PRIORITY_MORE_FAVORABLE 相对THREAD_PRIORITY_DEFAULT稍微优先，值为-1。
THREAD_PRIORITY_LESS_FAVORABLE 相对THREAD_PRIORITY_DEFAULT稍微落后一些，值为1。

## IntentService
使用了HandlerThread使得IntentService可以运行耗时任务，一般使用时结合主线程Handler或者LocalBroadCastManager来通知主界面UI的更新。

## ThreadPool线程池
线程池不允许使用 Executors 去创建，而是通过ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
说明： Executors 返回的线程池对象的弊端如下：
1） FixedThreadPool 和 SingleThreadPool:
允许的请求队列长度为 Integer.MAX_VALUE ，可能会堆积大量的请求，从而导致 OOM 。
2） CachedThreadPool 和ScheduledThreadPool :
允许的创建线程数量为 Integer.MAX_VALUE ，可能会创建大量的线程，从而导致 OOM 。

ThreadPoolExecutor(
int corePoolSize, 
int maximumPoolSize, 
long keepAliveTime, 
TimeUnit unit, 
BlockingQueue<Runnable> workQueue, 
RejectedExecutionHandler handler
);

```
public class ThreadPoolTest {
    private static final int CORE_POOL_SIZE = 5;
    private static final int MAX_POOL_SIZE = 10;
    private static final int BLOCK_POOL_SIZE = 2;
    private static final int ALIVE_POOL_SIZE = 2;
    private static ThreadPoolExecutor executor;

    public static void main(String args[]) {
        executor = new ThreadPoolExecutor(
                CORE_POOL_SIZE,// 核心线程数 最小
                MAX_POOL_SIZE,// 最大执行线程数
                ALIVE_POOL_SIZE,// 空闲线程超时
                TimeUnit.SECONDS,// 超时时间单位
                // 当线程池达到corePoolSize时，新提交任务将被放入workQueue中，
                // 等待线程池中任务调度执行
                new ArrayBlockingQueue<Runnable>(BLOCK_POOL_SIZE),// 阻塞队列大小
                // 线程工厂，为线程池提供创建新线程的功能，它是一个接口，
                // 只有一个方法：Thread newThread(Runnable r)
                Executors.defaultThreadFactory(),
                // 线程池对拒绝任务的处理策略。一般是队列已满或者无法成功执行任务，
                // 这时ThreadPoolExecutor会调用handler的rejectedExecution
                // 方法来通知调用者
                new ThreadPoolExecutor.AbortPolicy()
        );
        executor.allowCoreThreadTimeOut(true);
        /*
         * ThreadPoolExecutor默认有四个拒绝策略：
         *
         * 1、ThreadPoolExecutor.AbortPolicy()   直接抛出异常RejectedExecutionException
         * 2、ThreadPoolExecutor.CallerRunsPolicy()    直接调用run方法并且阻塞执行
         * 3、ThreadPoolExecutor.DiscardPolicy()   直接丢弃后来的任务
         * 4、ThreadPoolExecutor.DiscardOldestPolicy()  丢弃在队列中队首的任务
         */

        for (int i = 0; i < 10; i++) {
            try {
                executor.execute(new WorkerThread("线程 --> " + i));
                LOG();
            } catch (Exception e) {
                System.out.println("AbortPolicy...");
            }
        }
        executor.shutdown();

        // 所有任务执行完毕后再次打印日志
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    System.out.println("\n\n---------执行完毕---------\n");
                    LOG();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    /**
     * 打印 Log 信息
     */
    private static void LOG() {
        System.out.println(" ==============线程池===============\n"
                + "   线程池中线程数 : " + executor.getPoolSize()
                + "   等待执行线程数 : " + executor.getQueue().size()
                + "   所有的任务数 : " + executor.getTaskCount()
                + "   执行任务的线程数 : " + executor.getActiveCount()
                + "   执行完毕的任务数 : " + executor.getCompletedTaskCount()

        );
    }

    // 模拟线程任务
    public static class WorkerThread implements Runnable {
        private String threadName;

        public WorkerThread(String threadName) {
            this.threadName = threadName;
        }

        @Override
        public synchronized void run() {

            int i = 0;
            boolean flag = true;
            try {
                while (flag) {
                    i++;
                    if (i > 2) flag = false;
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        public String getThreadName() {
            return threadName;
        }
    }

}

```

## 请解释下在单线程模型中Message,Handler,Message Queue,Looper之间的关系。
拿主线程来说，主线程启动时会调用Looper.prepare()方法，会初始化一个Looper，放入Threadlocal中，接着调用Looper.loop()不断遍历Message Queue，Handler的创建依赖与当前线程中的Looper，如果当前线程没有Looper则必须调用Looper.prepare()。Handler , sendMessage到MessageQueue，Looper不断从MessageQueue中取出消息，回调handleMessage方法。 
* Looper  Message Queue 和 Handler， HandlerThread源码
https://blog.csdn.net/lovedren/article/details/51701477
https://blog.csdn.net/charles674611395/article/details/51914659


## Android的进程间通信，Liunx操作系统的进程间通信
binder(所有android上层几乎都是binder)/socket(zygote-systemserver)
Linux机制： 管道、消息队列、共享内存、套接字、信号量、信号这些IPC机制
handler线程之间通信
## asynctask的原理 
AsyncTask是对Thread和Handler的组合包装。 
https://blog.csdn.net/iispring/article/details/50670388 
https://blog.csdn.net/epubit17/article/details/80342004

## Alarm机制
Timer TimerTask
http://coderlin.coding.me/2016/04/02/Android-%E5%88%9D%E6%AD%A5%E4%B9%8BTimmer-AlarmManager-JobSchedule/>

## JobScheduler
http://blog.csdn.net/bboyfeiyu/article/details/44809395


## ELAPSED_REALTIME_WAKEUP和RTC_WAKEUP的区别
AlarmManager.ELAPSED_REALTIME_WAKEUP type is used to trigger the alarm since boot time:
alarmManager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP, 600000, pendingIntent);
will actually make the alarm go off 10 min after the device boots.
There is a timer that starts running when the device boots up to measure the uptime of the device and this is the type that triggers your alarm according to the uptime of the device.
Whereas, AlarmManager.RTC_WAKEUP will trigger the alarm according to the time of the clock. For example if you do:
long thirtySecondsFromNow = System.currentTimeMillis() + 30 * 1000;
alarmManager.set(AlarmManager.RTC_WAKEUP, thirtySecondsFromNow , pendingIntent);
this, on the other hand, will trigger the alarm 30 seconds from now.
AlarmManager.ELAPSED_REALTIME_WAKEUP type is rarely used compared to AlarmManager.RTC_WAKEUP

来自 <http://stackoverflow.com/questions/5938213/android-alarmmanager-rtc-wakeup-vs-elapsed-realtime-wakeup> 

## handler和timer的延迟对比
Handler vs Timer
在我们Android开发过程中，经常需要执行一些短周期的定时任务，这时候有两个选择Timer或者Handler。然而个人认为：Handler在多个方面比Timer更为优秀，更推荐使用。
一.易用性
1. 可重复执行
	• Handler可以重复执行某个任务。
	• Timer若在某个任务执行/取消之后，再次执行则会抛出一个IllegalStateException异常。为了避免这个异常，需要重新创建一个Timer对象。
2. 周期可调整
若想要执行一个越来越快的定时任务，Handler可以做到，而Timer则消耗较大。
	• Handler
private Handler handler = new Handler();
int mDelayTime = 1000;
private Runnable runnable = new Runnable() {
   public void run() {
      update();
      if （mDelayTime > 0） {
         handler.postDelayed(this,mDelayTime); 
         mDelayTime -= 100;
      }
   }
};
handler.postDelayed(runnable,1000);
如以上例子，就可以实现对周期的动态调整。
	• Timer的scheduleAtFixedRate(TimerTask task, long delay, long period)只能执行固定周期的任务，所以不可以动态地调整周期。若想要动态调整，则需要在执行玩一个定时器任务后，再启动一个新的任务时设置新的时间。
3. UI界面更新
	• Handler：在创建的时候可以指定所在的线程，一般在Activity中构建的，即主线程上，所以可以在回调方法中很方便的更新界面。
	• Timer：异步回调，所以必须借助Handler去更新界面，不方便。
既然都得用Handler去更新界面了，为何不如把定时的功能也交给Handler去做呢？
二.内存占比
Timer比Handler更占内存。
接下来的Demo例子通过两种方法循环地打印日志，然后通过MAT插件来查看这两个类所需要调用的对象所产生的占比。


## Async转sync的方法 异步->同步
You could do this with CountdownLatch, which might be the lightest synchronization primitive in java.util.concurrent:
private boolean findPrinter(final Context ctx) {
    final CountdownLatch latch = new CountdownLatch(1);
    final boolean[] result = {false};
...
BluetoothDiscoverer.findPrinters(ctx, new DiscoveryHandler() {
...
public void discoveryFinished() {
            result[0] = true;
            latch.countDown();
        }
public void discoveryError(String arg0) {
            result[0] = false;
            latch.countDown();
        }
...
    }
// before final return
    // wait for 10 seconds for the response
    latch.await(10, TimeUnit.SECONDS);
//return the result, it will return false when there is timeout
    return result[0];
}
来自 <https://stackoverflow.com/questions/20659961/java-synchronous-callback> 

## Handler集中形式原理
https://blog.csdn.net/reakingf/article/details/52054598
**Looper**
其中threadlocal保证一个线程只有一个Looper
一个Looper一个MessageQueue
不断循环获取msg，从target中取得相应得handler来进行dispatchmsg
```
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}

public static void loop() {
    for (;;) {
            Message msg = queue.next(); // might block
            msg.target.dispatchMessage(msg);
    }
}
```

**处理顺序Handler.dispatchMessage**
``` 
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```
**msg**
- sendEmptyMessageDelayed(what, time)->sendMessageAtTime 
msg.what = what

- postAtTime(Runnable r, long uptimeMillis)
sendMessageDelayed(getPostMessage(r), 0);
getPostMessage()
m.callback = r

- enqueue msg
msg.target = 记录不同handler
```
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```
**hanlder**
Handler(Handler.Callback callback)
