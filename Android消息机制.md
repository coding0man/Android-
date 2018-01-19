# Android消息机制

这一定是一个被写烂了的专题吧。那本媛也来凑一个热闹吧。哈哈  
这篇博客将会涉及以下内容：

* 消息机制概述
* UML图解消息机制相关类
* 从在主线程更新UI的方法带你畅游消息机制的源码，更加方便自己理解
* Handler
* Looper
* MessageQueue和Message
* 消息机制的应用

##消息机制概述
Android系统在设计的初期就已经考虑到了UI的更新问题，由于Android中的View是线程不安全的，然而程序中异步处理任务结束后更新UI元素也是必须的。这就造成了一个矛盾，最简单的解决方法肯定是给View加同步锁使其变成线程安全的。这样做不是不可以，只是会有两点坏处：  

1. 加锁会有导致效率底下
2. 由于可以在多个地方更新UI，开发就必须很小心操作，开发起来就很麻烦，一不小心就出错了。

基于以上两个缺点，这种方式被抛弃。于是机智如我谷歌爸爸。。。设置一个线程专门处理UI控件的更新，如果其他线程也需要对UI进行更新，不好意思，您把您想做的告诉那个专门处理UI线程的家伙，让它帮你做。大家各有各的任务，井水不犯河水，各司其职，效率才会高，不仅仅对于软件如此，人也是如此，我只管写我的代码，有农民伯伯帮我种吃的、有电脑公司卖给我电脑、有建筑公司给我盖房子（当然了，房子我是万万买不起的... 哼！），这些事儿好像自己也能干，但是都自己干好像就回到了远古时代，人类的进步和发展和社会分工的明确也是离不开的，而且关系很大！
好像扯的有点远了。

那么您也看出来了，消息机制其实可以很简单的用一句话概括，就是：其他线程通过给特定线程发送消息，将某项专职的工作，交给这个特定的线程去做。比如说其他线程都把处理UI显示的工作通过发送消息交给UI线程去做。

这样理解起来是不是就是so easy了呢？

##UML图解消息机制相关类

不知道上面的说法您是否可以对消息机制有了一个基本的认知呢？我曾经在想，怎么通过很简洁直观的方式去把消息机制讲明白（讲给自己，也讲给你）呢，后来我就在想，当初设计者的思路是什么样的呢？我想到了UML图，用类图来对消息机制中涉及到的几个类有一个概括的认识；通过时序图，可以很清晰的观察到整个消息机制的处理过程。  
消息主要设计到下面几个类：

* Handler：这是消息的发出的地方，也是消息处理的地方。
* Looper：这是检测消息的地方。
* MessageQueue:这是存放消息的地方，Handler把消息发到了这里，Looper从这里取出消息交给Handler进行处理
* Message：呜呜呜...他们发的是我，处理的也是我。
* Thread：我在这里专门指代的是，处理消息的线程。消息的发送是在别的线程。

话不多说，先来看一张图（UML忘的差不多了，刚补的，如果有错误，麻烦大家指出）

![消息机制类图](https://raw.githubusercontent.com/coding0man/Android-/master/attachment/MessageClassesUML.png)

##畅游源码
图在这里了，怎么看呢，整个消息机制相关的类密密麻麻支撑了一张网，咋个看嘛，，，不急不急，咱们先来思考一下咱们常用的更新UI是怎么一个操作步骤。  

1. 在主线程新建一个Handler对象，在构造方法中传入一个实现Handler.Callback接口的匿名类的对象，实现接口中的handleMessage方法
2. 在非UI线程使用Handler的sendMessage或者post方法发送一个消息
3. 然后handleMessage方法会在不久的将来马上执行，实现更新UI的操作。


那咱们就跟着这个思路来看一看这张图，先看Handler类，你会发现，Handler真的是个相当关键的核心（当然，其他部分也是不可或缺的），他几乎拥有所有其他相关对象的引用。

* Handler拥有Looper的引用，通过得到Looper对象获得Looper中保存的MessageQueue对象
* Handler拥有MessageQueue的引用，使Handler得以拥有发送消息（将Message放入MessageQueue）的能力
* Handler拥有Handler.Callback的引用，使得Handler可以方便的进行消息的处理。

####*来思考一个问题：为什么Handler在其他线程发送消息之后，就跑到了主线程的handleMessage方法中去更新UI？*
这个问题暂时先放着，等下回过头再来看。
我们现在先跟着第1，2，3步看看系统都帮我们做了什么操作呢？这就是在看源码，不要觉得很高深  
下面是鲜活的代码，为了方便您查看，我帮您摘出来了。如果有兴趣，您也可以在AS里点开看看 
 
```java

	//这是在主线程中
    Handler handler = new Handler(new Handler.Callback() {
        @Override
        public boolean handleMessage(Message msg) {

            switch (msg.what) {
                case 1:
                    Toast.makeText(mContext, "你真漂亮", Toast.LENGTH_SHORT).show();
                    break;
                case 2:
                    Toast.makeText(mContext, "你也很帅呢", Toast.LENGTH_SHORT).show();
                    break;
                default:
                    break;
            }
            return false;
        }
    });
    
	//这是我们在主线程中创建Handler时会使用的构造方法
    public Handler(Callback callback) {
        this(callback, false);//调用了下面的这个构造方法↓
    }
    
    //先不要管第二个参数。跟紧主线，别跟丢了
    public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

		  //在这里获取到Looper对象，怎么获取的，稍后再看
        mLooper = Looper.myLooper();
        //如果获取的mLooper为空，直接抛出异常，说你不能在一个没有调用Looper.prepare()方法
        //的线程里创建Handler
        //如此看来，Looper.prepare()方法重要的嘞
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        //通过mLooper对象获取MessageQueue这个消息队列（单链表）
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```
到此为止，一个Handler就创建好了，（还有一个问题是Looper.prepare方法很重要，但是我们还没有去考虑他是干嘛的，不急不急，先顺着一条线看，不然看源代码的过程会把你搞死翘翘的）先面就该进行第二步，看看Handler.sendMessage干了啥.代码段又来喽

```java
	//在一个新建的线程里使用创建好的Handler发送一个消息
    new Thread(new Runnable() {
        @Override
        public void run() {
        	//在这儿干点你想干的吧，一些耗时的计算或者网络操作啥的
            Message message = new Message();
            message.what = 1;
            handler.sendMessage(message);
        }
    }).start();
    
    //直接调用的是这个函数
    public final boolean sendMessage(Message msg){
        return sendMessageDelayed(msg, 0);
    }
    
    //转而调用了这个函数
    public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
    
    //转而又来到了这里
    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
    
    //最后的最后来到了这里
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    	//target就是Message绑定的Handler，看看类图，上面有这个细节
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        //最后的最后，调用了MessageQueue的enqueueMessage方法
        return queue.enqueueMessage(msg, uptimeMillis);
    }
    
    //再看一下MessageQueue的enqueueMessage方法，
    //我把其他一些无关的细节给删掉了，只为了更加容易阅读
    boolean enqueueMessage(Message msg, long when) {
        synchronized (this) {
            Message p = mMessages;
            if (p == null) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
            } else {
            	//下面的错误就是遍历单链表，找到链表的尾部，这个没有难度的吧？
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                    	//找到了尾部，现在的结构是这样的。
                    	//A->B->C...->pre(p)->null
                        break;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;//把next插入链表的尾部
            }
        }
        return true;
    } 
```

到此为止，第二步就结束了，成功把一个消息插入到了MessageQueue的尾部。可是你很快就会发现，第三步好像从这条路探寻不下去了。接下来就等着别人来调用Handler中的方法了，可是是谁调用的，在哪儿调用的？我们现在好像毫无头绪了？怎么办？怎么办？我们刚才不是看到一个Looper.myLooper(),和Looper.prepare()方法，说是很重要但是一直没看吗，既然现在搜寻不下去了，是不是可以回头看看了？还有一点，Looper，看起来是在循环，循环什么玩意儿呢？我们去好好看看Looper类吧。  
一共就三百来行代码，仔细看看，你会发现有一个核心方法：Looper.loop();  
同样的，我把影响阅读的非主线代码剔除了，发现Looper.loop方法就长这样：  

```java
    /**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;
        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
            try {
                msg.target.dispatchMessage(msg);
            } finally {
                
            }
        }
    }
```
清晰可见的是，Looper.loop()方法一直遍历MessqgeQueue,阻塞线程，直到获取到一个Message,然后调用了Message的一个成员变量target(其实就是Handler)的dispatchMessage(msg)方法，嗨，还真的又跟Handler扯上关系了，既然这里扯上关系了，而且还是一个分发消息的方法，可以大胆猜测就是让Handler去处理这个消息的。    
那么我们来看看这个方法：

```java
    /**
     * Handle system messages here.
     * 如果Message中callback对象不为空（这是调用handler.post(Runnable)方法发送的消息），
     * 就调用callback的run方法
     * 否则如果创建Handler的时候如果设置了Callback就调用创建时候的传入的
     * 实现Handler.Callback接口的类的对象的handleMessage方法，看这就是回调方法被调用的地方。
     * 再如果没有mCallback对象，就调用自身的handleMessage方法，为了Handler的子类复写了该方法的时候，方便调用，如，IntentService里的ServiceHandler就是继承自Handler的，并且重写了handleMessage方法。
     */
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
    
    private static void handleCallback(Message message) {
        message.callback.run();
    }
    
    //ServiceHandler继承自Service并且重写了handleMessage方法
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }
    
```

到了这里，三步走已经看完了，我想消息机制在我们心里已经又清晰了一层，但是不用急，咱们前边提的一个问题不是还没有解决吗，先把他解决掉吧，一起继续来看源码。  
咱们现在已知的是这样的，在主线程创建的Handler发送了一个消息，发送消息的代码运行在其他线程，将代码加入消息队列也是在其他线程（加了线程同步锁）。然后handleMessage发生在主线程，那么调用该方法的dispatchMessage方法也是运行在主线程的，dispatchMessage是在Looper.loop方法中调用的，也就是说loop方法也运行在主线程，那么问题就明朗了，可是loop方法是谁调用的，在哪里调用的呢？当然是系统启动的时候创建主线程之后再主线程的run方法中调用了Looper.prepare和Looper.loop方法，但是这点我还没看，留着以后再看吧。  
然后通过上面的分析，我们是不是可以自己来试着建立这样一个模型：

1. 创建一个线程A
2. 在这个线程的run方法中调用Looper.prepare和Looper.loop方法使该线程阻塞，等待消息发过来，然后处理
3. 在该线程中创建一个Hanlder，用来处理looper发送来的待处理的消息
4. 创建一些其他的线程a、b、c，做一点操作之后，通过Handler把消息传递出去，让线程A去处理。

```java
public class MyThread extends Thread {
    private static final String TAG = "MyThread";

    Handler handler = new Handler(new Handler.Callback() {
        @Override
        public boolean handleMessage(Message msg) {
            switch (msg.what) {
                case 1:
                    Log.i(TAG,"你真漂亮");
                    break;
                case 2:
                    Log.i(TAG,"你也很帅呢");
                    break;
                default:
                    break;
            }
            return false;
        }
    });


    public Handler getHandler() {
        return handler;
    }

    @Override
    public void run() {
        super.run();
        Looper.prepare();
        Looper.loop();
    }

    @Override
    public void destroy() {
        super.destroy();
        Looper.myLooper().quit();
    }
}

//
    private void testMyThread() {
        MyThread thread = new MyThread();
        thread.start();
        final Handler handler = thread.getHandler();
        
        Message message = new Message();
        message.what = 1;
        handler.sendMessage(message);

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    sleep(400);
                    Message message = new Message();
                    message.what = 2;
                    handler.sendMessage(message);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
```

试着想一想，如果把线程A看成主线程，在回调方法更新UI，那这不就是Android系统中更新UI使用的套路吗？不错，事实本就如此，工具是工具，用它来更新UI是可以的，那你当然也可以用来做一些其他的工作啊。  

在这里，通过以上的分析，不难得到下面的这个整个消息机制运行过程的时序图：
![时序图]()


ok啦，源码阅读到此为止。其他的细节，有兴趣的可以再细细研究一下。  
下面我们来对涉及到的类进行一下总结。  

##Handler

##Looper
##MessageQueue和Message
##消息机制的应用
