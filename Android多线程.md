#Android 多线程
本篇博客将设计以下内容：

* 多线程概述
* Thread类和Runnable接口
* 线程相关（线程生命周期、线程控制、线程同步）
* 线程通信（Handler机制）
* Android线程的创建和使用
* 线程池概念和使用

##多线程概述
众所周知，为了提供流畅的UI效果，Android系统不允许在主线程进行耗时操作，所以我们会经常使用多线程进行耗时操作，进行网络操作，数据的保存什么的。这是一个Android开发者一定会频繁接触到的知识模块，因此，为了更好的使用多线程，我们有必要把它弄清楚，弄明白。在我所阅读的很多很多书籍中，都将多线程的使用讲的非常明白，AsyncTask、HandlerThread、IntentService等等，讲的都很好，我就不在这方便多费口舌了。  
理一理思路吧：

* 为什么使用多线程？  

> Android中一个应用默认对应一个进程，一个进程有一个默认的线程（主线程/main线程/UI线程），**为了处理耗时任务，我们创建新线程处理并且将处理结果告知主线程。**

* 如何创建线程？

> 可以在如何一个线程中创建新的线程，新线程和创建线程的线程之间没有父子关系。Java中代表线程的类有且仅有Thread类，所以创建一个新线程的方法只有一种，就是创建一个Thread类的实例。
> 
> 我想肯定有人会问为什么你说只有一种，因为我觉得其他的都是变种。这些变种提供其他一些特定的功能、加入了一些其他的方法，都是为了方便开发者的使用。  
> 
> 所以我说核心就是一种。
> 
> * 继承自Thread类可以加入任何你想要增加的功能。
> * 实现Runnable接口也只是为Thread提供了一个运行方法
> * Callable接口只是为了拓宽Runnable,使线程增加可以返回数据的能力。
> * AsyncTask类提供了线程池和handler机制，方便后台任务更新UI。  
> * HandlerThread继承自Thread加入了Handler机制，其他线程将要处理的任务集中到HandlerThread中进行处理。  
> * IntentService继承自Service类，加入HandlerThread机制，提供线程优先级，不容易被系统杀死。
> 
> 我的意思是说，**万变不离其宗，只有详细理解Thread，才能更好的掌握他们常说的那几种使用线程的方法**。使用嘛，看看文档就会，重要的是理解为啥，多问几个为啥。

##Thread类和Runnable接口
这俩是创建多线程最最核心的内容，Runnable最简单，先说Runnable
###Runnable接口
Runnable接口定义了一个run方法，其他什么都木有。
> Runnable类只是提供了一个统一的规则/协议，给那些需要有代码需要运行的对象使用。  
 This interface is designed to provide a common protocol for objects that
  wish to execute code while they are active. For example,
  <code>Runnable</code> is implemented by class <code>Thread</code>.
  
###Thread类
Thread类是Java中线程类，用Thread类或者其子类可以创建一个线程对象。  
Thread类中一些重要的参数（可能还有其他的重要的）：  

* **ThreadGroup group**：线程组，默认和创建该线程的线程在同一个线程组。
* **Runnable target**：有代码需要运行的对象，除了Thread的子类，其他时候需要提供，否则该线程没有可运行的代码。
* **boolean daemon** ：是否是后台线程，默认和创建该线程的线程一样。
* **int priority**：线程优先级，默认和创建该线程的线程一样。
* **String name**：线程名，默认为Thread-n,n会自己变。
* **int threadStatus**：线程当前的状态。
* **long stackSize**：线程运行需要的栈内存大小，可以不指定

创建线程时，可以指定以上参数。创建线程后调用start()方法启动线程，java虚拟机会自动在合适的事件调用线程的run方法。

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
这也就为什么很多人说，创建线程有两个方法的原因：一是继承Thread类重写run方法；二是创建一个实现了Runnable接口的类，新建该类的实例，传入Thread的构造方法中。


  
##线程相关（线程生命周期、线程控制、线程同步）
###线程生命周期
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
###线程同步
##线程通信（Handler机制）
##Android线程的创建和使用
##线程池概念和使用
 

