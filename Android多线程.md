#Android 多线程
本篇博客将涉及以下内容： 

* 多线程概述
* Thread类和Runnable接口
* 线程相关（线程生命周期、线程控制、线程同步）
* 线程通信（Handler机制）
* Android线程的创建和使用
* 线程池概念和使用

##多线程概述
众所周知，为了提供流畅的UI效果，Android系统不允许在主线程进行耗时操作，所以我们会经常使用多线程进行耗时操作，进行网络操作，数据的保存什么的。这是一个Android开发者一定会频繁接触到的知识模块，因此，为了更好的使用多线程，我们有必要把它弄清楚，弄明白。在我所阅读的很多很多书籍中，都将多线程的使用讲的非常明白，AsyncTask、HandlerThread、IntentService等等，讲的都很好，我就不在这方面多费口舌了。  
理一理思路吧：

* 为什么使用多线程？  

> Android中一个应用默认对应一个进程，一个进程有一个默认的线程（主线程/main线程/UI线程），**为了处理耗时任务，我们创建新的工作线程来处理任务，并且将处理结果告知或者不告知主线程。**

* 如何创建线程？

> 可以在任何一个线程中创建新的线程，新线程和创建线程的线程之间没有父子关系。Java中代表线程的类有且仅有Thread类，所以创建一个新线程的方法只有一种，就是创建一个Thread类的实例。
> 
> 我想肯定有人会问为什么你说只有一种，因为我觉得其他的都是变种。这些变种提供其他一些特定的功能、或者加入了一些其他的方法，都是为了方便开发者的使用。  
> 
> 所以我认为核心就是一种。
> 
> * 继承Thread类可以加入任何你想要增加的功能。
> * 实现Runnable接口也只是为Thread提供了一个运行方法
> * Callable接口只是为了拓宽Runnable,使新线程增加可以返回数据的能力。
> * AsyncTask类提供了线程池和handler机制，方便后台任务更新UI。  
> * HandlerThread继承自Thread加入了Handler机制，其他线程将要处理的任务集中到HandlerThread中进行处理。  
> * IntentService继承自Service类，加入HandlerThread机制，提供线程优先级，不容易被系统杀死。
> 
> 我要表达的是，**万变不离其宗，只有详细理解Thread，才能更好的掌握他们常说的那几种使用线程的方法**。使用嘛，看看文档就会，重要的是理解为什么是这样。

##Thread类和Runnable接口
Thread类是Runnable接口的实现类，其中run()方法判断了当前Thread对象是否含有runable实例，如果有，调用该实例的run方法。
这俩是创建多线程最最核心的内容，Runnable最简单，先说Runnable
###Runnable接口
Runnable接口定义了一个run方法，其他什么都木有。
> Runnable类只是提供了一个统一的规则/协议，给那些需要有代码需要运行的对象使用。  
> This interface is designed to provide a common protocol for objects that
  wish to execute code while they are active. For example,
  <code>Runnable</code> is implemented by class <code>Thread</code>.
  
###Thread类
Thread类是Java中线程类，使用Thread类或者其子类可以创建一个线程对象。  
Thread类中一些重要的参数（可能还有其他的重要的）：  

* **ThreadGroup group**：线程组，默认和创建该线程的线程在同一个线程组。
* **Runnable target**：有代码需要运行的对象，除了Thread的子类，其他时候需要提供，否则该线程run方法中没有可运行的代码。
* **boolean daemon** ：是否是后台线程，默认和创建该线程的线程一样。
* **int priority**：线程优先级，默认和创建该线程的线程一样。
* **String name**：线程名，默认为Thread-n,n会自己变。
* **int threadStatus**：线程当前的状态。
* **long stackSize**：线程运行需要的栈内存大小，可以不指定

创建线程时，可以指定以上参数。创建线程后调用start()方法启动线程，java虚拟机会自动在合适的时间调用线程的run方法。

```java
    /**
     * If this thread was constructed using a separate
     * <code>Runnable</code> run object, then that
     * <code>Runnable</code> object's <code>run</code> method is called;
     * otherwise, this method does nothing and returns.
     * <p>
     * Subclasses of <code>Thread</code> should override this method.
     */
     
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```
 如果创建线程的时候提供了Runable对象（target参数），则调用Runnable对象的run方法， 否则，该线程什么都不做。  
这也就为什么很多人说，创建线程有两个方法的原因：一是继承Thread类重写run方法；二是新建一个实现了Runnable接口的类，创建该类的实例，传入Thread的构造方法中。


  
##线程相关（线程生命周期、线程控制、线程同步）
###线程生命周期
Java虚拟机中线程的生命周期和操作系统中线程生命周期，没有映射关系，操作系统中线程状态的切换可以去看再熟悉一下操作系统相关知识，这里贴一个图先看看：  

![操作系统中线程状态](https://github.com/coding0man/Android-/blob/master/attachment/thread_state.png?raw=true)  

线程从创建到结束会经历一系列生命周期：

* NEW
* RUNNABLE
* BLOCKED
* WAITING
* TIMED_WAITING
* TERMINATED

```java
    /**
     * A thread can be in only one state at a given point in time.
     * These states are virtual machine states which do not reflect
     * any operating system thread states.
     * 线程的状态是相对于Java虚拟机的，和系统中线程的状态没有对应关系
     */
    public enum State {
        /**
         * 还没有调用start方法的线程
         */
        NEW,

        /**
         * 可以运行但是不一定在运行，可能在等待其他的一些资源，如CPU
         */
        RUNNABLE,

        /**
         * BLOCK 和 WAITING  结合起来更加好理解一点
         * 一旦一个线程调用了Object.wait、Thread.join或者Locksupport.park方法,
         * 线程进入WAITING状态，
         * 进入WAITING状态的线程需要等待其他线程的唤醒，唤醒成功后并不一定马上可以执行(RUNNABLE)
         * 线程进入BLOCK状态，等待执行机会的到来。
         * 
         */
        BLOCKED,
        WAITING,

        /**
         * 调用下列方法后进入TIMED_WAITING状态
         * Thread.sleep
         * Object.wait(long)
         * Thread.join(long)
         * LockSupport.parkNanos
         * LockSupport.parkUntil
         * 
         */
        TIMED_WAITING,

        /**
         * 线程执行结束。
         */
        TERMINATED;
    }
```


###线程控制
线程的执行通常是由系统调用的，开发者并不能准确指定线程什么时候运行，但是还是可以通过一些设定来对线程进行适当的控制。 
 
1. **setPriority(int range 1-10)**：改变线程的优先级，使其获得更多的执行机会。

	> 提供三个优先级：
	> 
	> ```java
	> /**
   >  * The minimum priority that a thread can have.
   >  */
   >  public final static int MIN_PRIORITY = 1;
   > /**
   >  * The default priority that is assigned to a thread.
   >  */
   > public final static int NORM_PRIORITY = 5;
   > /**
   >  * The maximum priority that a thread can have.
   >  */
   > public final static int MAX_PRIORITY = 10;
	> ```  
	> 默认优先级和创建该线程的线程的优先级相同，范围为1-10，建议使用1-5-10 这三个
	> 
	> 

2. **join**:在A线程的run方法中调用B.join,在A线程阻塞，直到B线程执行完成。
3. **sleep**:使线程进入睡眠状态，在睡觉的时候，不会被唤醒，睡醒之后进入RUNNABLE状态，等待执行
4. **yeild**:暂停当前线程的执行，让CPU重新进行调度。给同等或者更高优先级的线程让步，如果没有这样的线程，该线程会被重新执行。
5. **setDaemon()**:设置为后台线程的线程们不能单独存在，当所有非后台线程执行结束，后台线程会自己结束。
6. **wait**:在线程同步中使线程等待，直到被唤醒。
6. **park，unpark**：将线程暂时挂起，直到某个时间或者直到被唤醒。

###线程同步

跟上面说的一样，由于线程的执行是不可控的，所以如果不加以控制，总是可能会发生错误的。相信开发过程中肯定遇到需要一个在Adapter中更新显示的数据，同时网络罗数据返回时修改Adapter中绑定的数据，这个时候如果使用ArrayList一不小心就会发生错误。所以我们需要了解线程同步，在需要的时候使用线程同步，避免错误的发生。锁机制还是比较复杂的，这里这是介绍一下大概的内容。  
线程同步总是会有性能问题，只在必要的时候使用线程同步。  
线程同步主要涉及以下内容：  

#### synchronized  
使用synchronized关键字对方法或者代码块增加同步锁，在执行结束（正常结束、return、发生异常和错误）或者在同步块内调用wait(sleep,yeild不会释放)方法之后，同步锁自动释放。

```java
	//下面示例代码是来自Thread类中用到的synchrozined
	//启动线程
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        // Android-changed: throw if 'started' is true
        if (threadStatus != 0 || started)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        started = false;
        try {
            nativeCreate(this, stackSize, daemon);
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
    
    //设置线程优先级
    public final void setPriority(int newPriority) {
        ThreadGroup g;
        checkAccess();
        if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
            throw new IllegalArgumentException();
        }
        if((g = getThreadGroup()) != null) {
            if (newPriority > g.getMaxPriority()) {
                newPriority = g.getMaxPriority();
            }
            synchronized(this) {
                this.priority = newPriority;
                if (isAlive()) {
                    nativeSetPriority(newPriority);
                }
            }
        }
    }
```

#### Lock
显式地新建一个Lock对象，在适当的位置进行lock，释放锁时直接调用unLock即可释放锁资源。Lock比synchronized好用的一点是对于同一个资源，可以显式地控制锁之间的互斥关系，增强程序执行效率。比如常见的读写锁，readLock和writeLock,readLock和readLock之间不互斥，两个进程都可以随时读取，这在一定程度上提高了效率。

```java
//我们直接来看使用示例：
//对读操作添加readLock，
//当其他的线程需要写数据的时候，需要等待当前线程锁的释放，
//当其他线程需要读数据时可以直接读，不受其他readLock的影响
//
//对写操作添加writeLock，
//禁止其他线程对数据进行读写。直到当前线程释放。
class RWDictionary {
    private final Map<String, Data> m = new TreeMap<>();
    private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    private final Lock r = rwl.readLock();
    private final Lock w = rwl.writeLock();

    public Data get(String key) {
        r.lock();
        try {
            return m.get(key);
        } finally {
            r.unlock();
        }
    }

    public List<String> allKeys() {
        r.lock();
        try {
            return new ArrayList<>(m.keySet());
        } finally {
            r.unlock();
        }
    }

    public Data put(String key, Data value) {
        w.lock();
        try {
            return m.put(key, value);
        } finally {
            w.unlock();
        }
    }

    public void clear() {
        w.lock();
        try {
            m.clear();
        } finally {
            w.unlock();
        }
    }
}
```


##线程通信（Handler机制）
Android中线程之间的通信主要就是消息机制(Handler机制)。  
消息机制广泛应用在Android中的UI更新，但是其实他能做的可远远不止UI的更新，对于多任务的集中式处理也是消息机制的一大应用场景，同时对于AysncTask、HandlerThread、IntentService等等而言，内部也都使用了消息机制。  

消息机制的基本原理是:  
> 使用一个MessageQueue来作为连接两个线程的中介    
> （线程资源放在栈内存区、MessageQueue放在堆内存中，同一进程的不同线程可以访问同一块堆内存），
> 两个线程都可以对MessageQueue进行操作，从而实现线程的通信。  

具体的实现逻辑是：

> 1. 在线程A中创建Looper并且开始消息的循环，
> 2. 利用A中创建好的Looper创建一个Handler（Handler的构造函数中默认使用当前线程的Looper）
> 3. 新重建一个线程B，让线程B持有Handler的引用，
> 4. 在线程B中处理任务，通过Handler发送消息对象Message将处理结果传递出去(通过handler.post(Runnable)或者handler.sendMessage(Message)等方法，发送的消息最终都会被包装成一个Message对象，如post(Runnable)方法会首先创建一个Message，然后将Runnable作为Message中的Runnable callback实例变量)
> 5. Message最终会放入MessageQueue中，并且被线程A的Looper获取到
> 6. Looper获取到Message之后，根据Message中保存的Handler的引用，调用Handler的dispatchMessage()方法，如果Message中包含callback,则执行callback的run()方法,否则调用handleMessage()方法处理消息。
> 7. 流程结束之后调用looper.quit()关闭Looper。

关于消息机制的详细内容，可以回头翻一下之前在郭霖公众号里我的一片投稿，也可以点击文章最后的查看原文通过链接访问：
[Android消息机制](http://blog.csdn.net/code__man/article/details/79108191)的文章链接为：http://blog.csdn.net/code__man/article/details/79108191



##Android线程的创建和使用
Android中线程的跟Java一样，使用Thread类代表线程。创建Thread类有以下几种方法：

* **继承Thread类，重写run方法,创建子类实例**，调用Thread.start()方法。

```java
//step1:定义自己的Thread，也可以直接用局部类实现

package com.nullponinter.study.ThreadsTest;

import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.util.Log;

import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * Created by Francis on 15/01/2018.
 */

public class MyThread extends Thread {
    private static final String TAG = "MyThread";

    @Override
    public void run() {
        super.run();
        //你可以在这里
        //写这个需要处理的逻辑
    }

    @Override
    public void destroy() {
        super.destroy();
    }
}

//Step2: 直接使用
MyThread myThread = new MyThread();
myThread.start();
```

* **在Thread类的构造方法中传入一个实现Runnable接口的类的实例**，作为Thread.target变量，在run方法中调用。

	这里又可以分为两种情况：
	
	* 直接实现Runnable接口，重写run方法，传入构造方法
	* 使用FutureTask作为Runnable对象（在FutureTask的构造方法中传入Callable对象），将FutureTask作为Thread.target变量。
	* 你也可以根据自己的需要，去随便对Runnable做点什么
	
```java
    /**
     * 测试futureTask
     */
    public static void testThread() {
    
    
    	//使用Runable
        new Thread(new Runnable() {
            @Override
            public void run() {
				//do something
			
            }
        }).start();

		//使用FutureTask
        final FutureTask<Long> futureTask = new FutureTask(new Callable<Long>() {
            @Override
            public Long call() throws Exception {
                Long sum = 0L;
                for (int i = 0; i < 100000000; i++) {
                    sum += i;
                }
                return sum;
            }
        });


        new Thread(futureTask).start();
        try {
            Log.e("====sum=", "llllll");
            Long sum = futureTask.get();
            Log.e("====sum=", sum + "");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        
    }
```

* **使用AsyncTask类**，在doInBackground方法中做耗时操作，调用更新进度方法，返回操作结果等、在onProgressUpdate方法中更新进度、在onPostExecute方法中操作返回结果，更新UI。

```java
    /**
     * 测试asyncTask的方法
     */
    private void testAsyncTask() {

        AsyncTask asyncTask = new AsyncTask<String,Integer,String>(){

            @Override
            protected String doInBackground(String... strings) {
                return null;
            }

            @Override
            protected void onPreExecute() {
                super.onPreExecute();
            }

            @Override
            protected void onPostExecute(String s) {
                super.onPostExecute(s);
            }

            @Override
            protected void onProgressUpdate(Integer... values) {
                super.onProgressUpdate(values);
            }
        };
        
        asyncTask.execute("");
    }
```

* **使用HandlerThread**,将特定的任务集中到一个线程来做，其他线程将任务发送到HandlerThread。

```java
    /**
     * 测试HandlerThread
     */
    private void testHandlerThread() {

        //使用的Handler构造方法没有提供Looper，默认使用当前线程的Looper，
        // 所以回调方法中的代码也在当前线程执行
        final Handler currentThreadHandler = new Handler(new Handler.Callback() {
            @Override
            public boolean handleMessage(Message msg) {
                StringBuilder sb = new StringBuilder("currentThreadHandler")
                        .append("\nmessage what=").append(msg.what)
                        .append("\nthreadName=").append(Thread.currentThread().getName());
                Log.e("===", sb.toString());
                return false;
            }
        });

        HandlerThread handlerThread = new HandlerThread("MyTestThread");
        handlerThread.start();
        //使用的Handler构造方法提供了handlerThread的Looper，
        // 所以回调方法中的代码在handlerThread中执行，
        //该线程是工作线程，不能用于更新UI
        final Handler handlerThreadHandler = new Handler(handlerThread.getLooper(), new Handler.Callback() {
            @Override
            public boolean handleMessage(Message msg) {
                StringBuilder sb = new StringBuilder("handlerThreadHandler")
                        .append("\nmessage what=").append(msg.what)
                        .append("\nthreadName=").append(Thread.currentThread().getName());
                Log.e("===", sb.toString());
                return false;
            }
        });

        final Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                int random = Math.abs(new Random().nextInt());
                int sum = 0;
                for (int i = 0; i < random; i++) {
                    sum = sum + i;
                }

                if (sum % 2==0) {
                    handlerThreadHandler.sendEmptyMessage(sum % 2);
                }else {
                    currentThreadHandler.sendEmptyMessage(sum % 2);
                }
            }
        });

        thread.start();
        thread.start();
        thread.start();
        thread.start();
        thread.start();
    }
    //输出的结果有两种情况
    
    //handlerThreadHandler
    //message what=0
    //threadName=MyTestThread
    //或
    //currentThreadHandler
    //message what=1
    //threadName=main
```

* **继承IntentService**，重写onHandleIntent方法处理Intent，其他线程直接startService(Intent)就可以了。

```java

//Step1 实现自己的IntentService类
package com.nullponinter.study.ThreadsTest;

import android.app.IntentService;
import android.content.Intent;
import android.support.annotation.Nullable;
import android.util.Log;

/**
 * Created by Francis on 15/01/2018.
 */

public class MyIntentService extends IntentService {
    public MyIntentService() {
        super("MyIntentService");
    }
    /**
     * Creates an IntentService.  Invoked by your subclass's constructor.
     *
     * @param name Used to name the worker thread, important only for debugging.
     */
    public MyIntentService(String name) {
        super(name);
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.e("===","IntentService.onCreate() runs on "+Thread.currentThread().getName());

        Log.e("===","IntentService被创建");

    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.e("===","IntentService被销毁");

    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {

        Log.e("===","intent service onHandleIntent run on "+Thread.currentThread().getName());
        int randomCount = (int) Math.random()%100;

        int sum = 0;
        for (int i = 0; i < randomCount; i++) {
            sum += i;
        }
        Log.e("===","sum==="+sum);

    }
}

//Step2 使用IntentService
private void testIntentService() {
     for (int i = 0; i < 3; i++) {
        Intent intent = new Intent(this, MyIntentService.class);
        startService(intent);
    }
}

```

这几种方式各有各的好处，可以根据自己的需要选择不同的实现。具体的使用方法，网上的同类教程太多了，没有写的必要了。
##线程池概念和使用
众所周知的是线程的创建、销毁都是需要系统帮我们做很多事情的。频繁的创建、销毁线程对系统消耗是比较很大的，为了避免系统资源的浪费，对于生命周期不长，需要频繁创建销毁的线程，我们都建议使用线程池来进行线程的管理。  
在前面我们提到过：
> Runnable类提供了一个统一的规则/协议，给那些需要有代码需要运行的对象使用。  
 This interface is designed to provide a common protocol for objects that
  wish to execute code while they are active. For example,
  <code>Runnable</code> is implemented by class <code>Thread</code>.

既然Runnable代表一个可以执行的内容，那么事实上，我们可以创建一个线程，多个Runnable对象使用同一个Thread对象进行Runnable中run方法体的执行。线程池的原理正基于此。(Callable也是一样的)  
###相关概念 
涉及到的相关类简介：

```java
//继承关系介绍:
* public interface Executor
* public interface ExecutorService extends Executor
* public abstract class AbstractExecutorService implements ExecutorService
* public class ThreadPoolExecutor extends AbstractExecutorService
* public class Executors
```

* **Executor**:线程池的基础接口，只有一个<code>void execute(Runnable command)方法</code>，方法是具体实现由子类去完成。
* **ExecutorService**:Executor的子接口，提供了几个submit方法，拓展execute方法，提供了shutdown方法关闭池，提供几个invoke方法执行Callable列表。
* **AbstractExecutorService**：实现ExecutorService接口的抽象类，开发者可以创建该类的子类创建自己的线程池实现。
* **ThreadPoolExecutor**:继承自AbstractExecutorService，提供了很多可供配置的参数，为开发提供便利，用来创建真正的线程池的类。
* **Executors**:工厂类，将ThreadPoolExecutor设一个默认值，允许开发者提供极少的参数即可创建出可满足大多数情况的线程池的实现。

###ThreadPoolExecutor中的参数

```java
    /**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     *        池中核心线程的数量
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     *        最大线程数量
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     *        非核心线程的存活时间，
     *        如果非核心线程在等待这么久之后还没有任务执行，就要死掉了
     * @param unit the time unit for the {@code keepAliveTime} argument
     * 		 存活时间的单位
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     *        任务的等待队列，定义了需要等待执行的线程应该放在哪里，
     *        以及重新取出来执行的规则是什么
     * @param threadFactory the factory to use when the executor
     *        creates a new thread
     *        创建新线程的线程工厂
     * @param handler the handler to use when execution is blocked
     *        because the thread bounds and queue capacities are reached
     *        饱和策略
     *        当新任务到达但是池中数量已达到最大值，
     *        并且任务队列也已经满了的情况下，
     *        任务拒绝的处理
     *        默认的处理方式是直接抛出异常
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue}
     *         or {@code threadFactory} or {@code handler} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) 
```

###几种常用的线程池
Executors工厂类一共提供了很多种（大概12种）new开头的方法供使用，大致可以分成下面几类：

* **newCachedThreadPool**:

> 该池核心线程数为0，最大线程数没有上限，队列中不保存任务，线程的存活周期是60s  
> 所以，该池适合于处理生存期很短的异步任务  
> **SynchronousQueue:**一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态
> 
```java
    /**
     * Creates a thread pool that creates new threads as needed, but
     * will reuse previously constructed threads when they are
     * available.  These pools will typically improve the performance
     * of programs that execute many short-lived asynchronous tasks.
     * Calls to {@code execute} will reuse previously constructed
     * threads if available. If no existing thread is available, a new
     * thread will be created and added to the pool. Threads that have
     * not been used for sixty seconds are terminated and removed from
     * the cache. Thus, a pool that remains idle for long enough will
     * not consume any resources. Note that pools with similar
     * properties but different details (for example, timeout parameters)
     * may be created using {@link ThreadPoolExecutor} constructors.
     *
     * @return the newly created thread pool
     */
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

* **newFixedThreadPool**:

> 核心线程数等于最大线程数，任务数超过线程数时放入阻塞队列中等待执行  
> 合适于数量相对稳定，时间较长的任务    
>
```java
     /**
     * Creates a thread pool that reuses a fixed number of threads
     * operating off a shared unbounded queue.  At any point, at most
     * {@code nThreads} threads will be active processing tasks.
     * If additional tasks are submitted when all threads are active,
     * they will wait in the queue until a thread is available.
     * If any thread terminates due to a failure during execution
     * prior to shutdown, a new one will take its place if needed to
     * execute subsequent tasks.  The threads in the pool will exist
     * until it is explicitly {@link ExecutorService#shutdown shutdown}.
     *
     * @param nThreads the number of threads in the pool
     * @return the newly created thread pool
     * @throws IllegalArgumentException if {@code nThreads <= 0}
     */
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

* **newScheduledThreadPool**:

>
> 核心线程数自定义，最大线程数没有上限，10s的保活时间  
> DelayedWorkQueue：
>
```java
    /**
     * Creates a thread pool that can schedule commands to run after a
     * given delay, or to execute periodically.
     * @param corePoolSize the number of threads to keep in the pool,
     * even if they are idle
     * @return a newly created scheduled thread pool
     * @throws IllegalArgumentException if {@code corePoolSize < 0}
     */
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
    }
```

* **newSingleThreadExecutor**:

> 单例线程，任意时间池中只能有一个线程
>
```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```






 

