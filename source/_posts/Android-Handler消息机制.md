---
title: Android Handler 消息机制
date: 2020-04-04 8:34:33
tags: [Android,Framework]
---

> 本篇基于 `Android 10.0` 总结  

Android 是基于消息机制运行的，理解消息机制的原理在日常开发中非常重要，从 App 创建开始，到点击，滑动更新页面 ，这些操作都离不开底层的消息机制，可以说消息机制是 Android 系统基石之一。<!-- more -->

# 2个前提知识
在剖析源码之前，我们要先了解2个最基本知识点，第一个就是先不考虑原理，作为 App 开发者如何使用消息机制编程，第二个就是 `linux epoll` 机制，这部分是消息机制的底层运转核心。

## 消息机制模型
在Android中只有App的主线程才能去更新UI界面，所以主线程一般又被称为UI线程，非主线程去更新UI时，系统会抛出异常错误，所以 Android 只能在主线程去更新 UI
[frameworks/base/core/java/android/view/ViewRootImpl.java](http://aospxref.com/android-10.0.0_r2/xref/frameworks/base/core/java/android/view/ViewRootImpl.java#checkThread)

```java
void checkThread() {
    if (mThread != Thread.currentThread()) {
        throw new CalledFromWrongThreadException(
                "Only the original thread that created a view hierarchy can touch its views.");
    }
}
```

为什么会设计成单线程呢？ 原因为了高效，各个UI控件若能多线程操作，线程间同步将会是个灾难，并且很可能造成资源泄露，那么主线程和子线程如何通信呢？ 消息机制就为这个设计，另外四大组件的启动过程也都离不开消息机制的驱动。

我们先来看下Android官方文档 [Looper](https://developer.android.google.cn/reference/android/os/Looper) 中关于消息机制的典型用法，其实步骤也很简单，这个例子定义了一个线程，当线程被系统调度起来执行`run()`方法后，先调用 `Looper.prepare()` 方法准备好该线程的Looper对象，然后在某个线程中初始化好一个 handler 并复写了一个处理消息的方法，最后调用 `Looper.loop()` 方法将消息队列循环起来，通过 `mHanlder` 发送和处理消息。

``` java
class LooperThread extends Thread {
  public Handler mHandler;

  public void run() {
      Looper.prepare(); // 1. 准备好 looper 对象
      mHandler = new Handler() { //2. 创建一个handler对象,并指明消息处理函数
          public void handleMessage(Message msg) {
              // process incoming messages here
          }
      };
      Looper.loop(); //3.开始该线程内部消息循环
  }
}
```

接着我们通过一个 hello world App 实际操作下，这个例子的功能是，在 hello world App 基础上，增加一个按钮，点击按钮，去模拟网络下载数据，下载完毕后，最后在界面显示 "download ok" 的提示。

MainActivity  中创建一个主线程绑定的成员变量 mHandler, 并复现 `handleMessage` 方法，内容为当有消息类型为1时的消息，就更新界面的 textview 控件显示提示

```java
public class MainActivity extends Activity {
    private Handler mHandler = new Handler(){//创建主线程绑定的handler对象
        @Override
        public void handleMessage(android.os.Message msg) {
            switch(msg.what) {
            case 1:
                mText.setText("download Ok!");
                break;
            }
        };
    };
```

在 MainActivity 的 `onCreate` 方法中设置按钮点击事件响应，当点击按钮时，开启子线程模拟下载，下载完成后，通过 `mHandler` 发送一个消息给主线程

```java
protected void onCreate(Bundle savedInstanceState) {
    ...
    mButton.setOnClickListener(new OnClickListener {
        @overide
        public void onClick() {
          new Thread(new Runable() {
            @overide
            public void run() {
                 //模拟等待 2s
                 Thread.sleep(2);
                 // 2s后发送消息给主线程通知下载完毕
                 Message msg = Message.obtain();
                 msg.what = 1;
                 mHandler.sendMessage(msg);
                }
             })
            }).start();
        ...
}
```
所以，开发者使用上就是通过  Handler, Message ,Looper 三者使用。

## linux epoll 机制简述

第二个就是 linux epoll 机制，它是 linux 中最高效的多路 IO 复用机制，也就是一个老师( epoll 文件描述符fd )  可以同时管十几个学生(需要监听的 event 事件），当某个学生有问题时，都可以举手问老师，老师可以回答问题（响应事件），它适用于大量并发少量活跃的情况下，使用 epoll 的步骤有3步：

1. 首先通过 `epoll_create()` 函数创建一个 epoll 文件描述符，参数 size 为能够监听的最大数量

   ```c++
	int epoll_create(int size)； // 创建一个epoll文件描述符，size 为能够监听的个数
   ```

2. 通过 `epoll_ctl` 方法告诉 epoll fd 需要监听哪个文件描述符以及它的什么事件

    ```c++
    int  epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
    ```

3. 调用`epoll_wait`等待监控的事件到来，当有监控的事件发生，会放到 events 数据组中， timeout 为等待的时间，若未到来，则一直阻塞

    ```c++
    int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
    ```

好了，了解了这2个背景知识，下面我们就看下消息机制模型的源码流程。

# 消息机制源码分析

## Looper 创建和循环
Looper 的本质是有一个消息队列，然后一直循环抽取消息队列里面的消息执行，当没有消息时，就休眠等待消息到来 或者做一些清理的工作。

### Looper.prepare()
任何线程在开始消息循环前都要进行准备工作，核心目的是创建一个该线程唯一的 Looper 对象（也就是消息队列），`Looper.prepare()` 方法中首先会检查该线程是否已经有了一个Looper对象，如果之前有，则会抛出异常，一个线程仅会只有一个 Looper 对象，创建好的 Looper 对象会被设置到线程私有变量中，线程私有变量是每个线程单独有的区域，由不同线程私自持有。

[/frameworks/base/core/java/android/os/Looper.java](http://aospxref.com/android-10.0.0_r2/xref/frameworks/base/core/java/android/os/Looper.java)

```java
    public static void prepare() {
        prepare(true);
    }
    
    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed)); //创建 Looper对象并设置线程私有变量中
    }

    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);// Looper 对象的构造本质是创建一个消息队列
        mThread = Thread.currentThread();
    }
```

### MessageQueue 创建
在 Looper 创建时，会创建 Java 层的消息队列 MessageQueue, 而 Java 层 MessageQueue 创建时又会调用 JNI 方法创建一个 native 层的消息队列，并持有其指针存于成员 `mPtr`中，这样方便后面 Java 层操作 native 的消息队列，我们看下 Java 层 MessageQueue 构造函数

[/frameworks/base/core/java/android/os/MessageQueue.java](http://aospxref.com/android-10.0.0_r2/xref/frameworks/base/core/java/android/os/MessageQueue.java)

```java
    package android.os;

    MessageQueue(boolean quitAllowed) {
        mQuitAllowed = quitAllowed;
        mPtr = nativeInit();//调用 android_os_MessageQueue.cpp 中native方法初始化
    }
    
    private long mPtr; // used by native code
    private native static long nativeInit();// native 方法
```

调用关系为：

`MessageQueue.java`  ->  `MessageQueue.nativeInit()` ->  `android_os_MessageQueue.cpp:android_os_MessageQueue_nativeInit()`，最后通过 JNI 层调用

这里顺便说下，Java 层如何对应有 native 层的方法呢？ 一般规则如下，MessageQueue.java 所在包名是 android.os ，MessageQueue.java 对应的 JNI 文件名为 android_os_MessageQueue.cpp，然后将包名和文件名通过下划线链接起来就是 android_os_MessageQueue.cpp，好了，我们在来看下 JNI 层做了什么事情
[/frameworks/base/core/jni/android_os_MessageQueue.cpp](http://aospxref.com/android-10.0.0_r2/xref/frameworks/base/core/jni/android_os_MessageQueue.cpp)

```java
static jlong android_os_MessageQueue_nativeInit(JNIEnv* env, jclass clazz) {
    NativeMessageQueue* nativeMessageQueue = new NativeMessageQueue();//创建一个native 层消息队列
    if (!nativeMessageQueue) {
        jniThrowRuntimeException(env, "Unable to allocate native queue");
        return 0;
    }

    nativeMessageQueue->incStrong(env);
    return reinterpret_cast<jlong>(nativeMessageQueue);//返回native层消息队列的指针
}
```

在 JNI 层也创建一个 native 的消息队列（和 Java层的消息队列没关系），然后返回一个 native层消息队列的指针存储在 `MessageQueue.mPtr`中，也就是 Java层的消息队列持有 native层的消息队列的指针，接下来我们看下 nativeMessageQueue的构造

[/frameworks/base/core/jni/android_os_MessageQueue.cpp](http://aospxref.com/android-10.0.0_r2/xref/frameworks/base/core/jni/android_os_MessageQueue.cpp)

```c++
NativeMessageQueue::NativeMessageQueue() :
        mPollEnv(NULL), mPollObj(NULL), mExceptionObj(NULL) {
    mLooper = Looper::getForThread();
    if (mLooper == NULL) {
        mLooper = new Looper(false);
        Looper::setForThread(mLooper);
    }
}
```
native 层消息队列中构造中，和 Java层一样，也会创建一个 Looper对象，将该对象存储在线程私有变量中，在 native层 Looper对象创建时，会通过  epoll 机制监听事件的发生

[/system/core/libutils/Looper.cpp](http://aospxref.com/android-10.0.0_r2/xref/system/core/libutils/Looper.cpp)

```c++
Looper::Looper(bool allowNonCallbacks)
    : mAllowNonCallbacks(allowNonCallbacks),
      mSendingMessage(false),
      mPolling(false),
      mEpollRebuildRequired(false),
      mNextRequestSeq(0),
      mResponseIndex(0),
      mNextMessageUptime(LLONG_MAX) {
    mWakeEventFd.reset(eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC));//初始化了一个 wakeEventfd 对象
    ...
    rebuildEpollLocked(); //构建 epoll 事件
}

void Looper::rebuildEpollLocked() {
    ...
    
    // Allocate the new epoll instance and register the wake pipe.
    mEpollFd.reset(epoll_create1(EPOLL_CLOEXEC)); //1. 创建一个 epoll fd
    ...

    struct epoll_event eventItem;// 构造一个监听事件描述
    memset(& eventItem, 0, sizeof(epoll_event)); // zero out unused members of data field union
    eventItem.events = EPOLLIN; //监听读入，也就是当管道中有内容时，唤醒去读取
    eventItem.data.fd = mWakeEventFd.get();
    int result = epoll_ctl(mEpollFd.get(), EPOLL_CTL_ADD, mWakeEventFd.get(), &eventItem); // 2. 将监听的wakefd 加入到 epoll fd 中，并且监听读入事件
    ...
```

好了，我们来总结下，目前已有的环境准备工作，
* 线程开始时，通过 `Looper.prepare()` 初始化一个线程唯一的Looper对象，
* 该Looper对象内部持有一个Java层的消息队列，在Java层消息队列创建时，又在 native层创建了一个 native层的消息队列和native层的Looper, 同时准备好了epoll机制环境，添加了对应需要监听的管道事件
* Java层保存了 Native层的消息队列指针，以便后续操作

好了，消息队列环境准备好了，我们就可以让 消息队列循环起来了

### Looper.Loop()
调用 `Looper.loop()`方法后，消息队列就开始循环跑起来了，

[/frameworks/base/core/java/android/os/Looper.java](http://aospxref.com/android-10.0.0_r2/xref/frameworks/base/core/java/android/os/Looper.java)

```java
public static void loop() {
    final Looper me = myLooper();//1.获取该线程的 Looper对象
    ...
    final MessageQueue queue = me.mQueue;//获取该线程的 MsgQueue
    ...//省略

    for (;;) {//开始死循环获取msg
        Message msg = queue.next(); // might block，取出下一条消息执行，没消息时阻塞等待
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return; //退出消息循
        }
        ...
        msg.target.dispatchMessage(msg);//msg.target 就是 handler，通过 handler 派发出去
        ...
        msg.recycleUnchecked();// msg 是一个消息池，用完后回收
    }
}
```
整个逻辑比较简单，首先通过 `myLooper()` 方法获取到Looper对象（存储在线程私有变量中），取到消息队列，然后就是死循环该消息队列（链表组成），不断的取消息(`queue.next()`)执行, 如果消息队列没有消息，则会等待消息到来，当有消息时，通过 Handler 进行分发出去，`msg.target` 就是一个 Handler 对象，消息发送之后，对消息进行回收。

下面在看下消息队列如何取出一条消息的，

### MessageQueue.next()
这个函数为整个消息机制的核心，核心目的就是从消息队列中取出一条信息来执行，若消息队列为空或没有消息则会阻塞，等待被唤醒，或者做些空闲的清理工作，我们逐一解释下这个函数流程：

```java
    Message next() {
        ...
        final long ptr = mPtr;
        ...
        
        //刚开始迭代时初始化参数
        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0; //首次默认不等待
        for (;;) {
            ...

            nativePollOnce(ptr, nextPollTimeoutMillis);//等消息，最大等待时间依据nextPollTimeoutMillis 的值

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {// 当消息队列不为空时
                    if (now < msg.when) {//若是延迟消息，则计算还要等待的最大timeout时间
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {//消息队列不为空，且消息执行时间符合，开始获取一条消息
                        // Got a message.
                        mBlocked = false; //将阻塞变量设为false
                        //基于链表操作，获取消息队列的首条消息
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg; //立即 return, next()就直接返回了
                    }
                } else {//如果消息队列为空，说明当前无任何消息，设置无限等待标志（-1）
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // 逻辑能走到这里，说明消息队列为空，或者首个消息队列还没到时间执行时间，那么就趁着这个完全空闲的执行，执行空闲的操作
                
                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                //若消息队列为空或者首条消息未到执行时间，获取 idlerHandler的大小
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                //若此时未有pengdingIdleHanlder的任务，说明没有空闲执行的任务，那就直接等下一条消息到来
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true; //阻塞设置为 true
                    continue; // 直接下一轮，实际是直接应用 nextPollTimeoutMillis 来做等待
                }

                //下面是有pending的空闲任务，则逐一回调执行，
                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }
            
            // pending 任务做完了，立即设置 空闲任务为0，且等待时间为0，因为可能现在消息队列可能有消息了，直接查看
            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }
```

我们总结下关键点：
* `nativePollOnce`函数为核心函数，没有消息则会阻塞等待，通过 `nextPollTimeoutMillis` 变量控制最大等待时间，核心逻辑是通过这个变量的值确定等待时间，发送延迟消息或者一直等待消息（-1）
* 首次迭代进入的时候，不会等待，立即去检查消息队列有无消息，有符合的消息则直接返回，否则设定 nextPollTimeoutMillis 值等待
* 消息列表基于单链表实现，所以取出并移除头部消息，会用到两个变量 preMsg, mMessages
* 若消息队列为空或者首个消息执行时间还没到，此时主线程完全是空闲状态，若app有注册 `idlehandler`接口，则会利用这个空闲时间做回调 ，一个实际例子可 [参考](https://blog.csdn.net/tencent_bugly/article/details/78395717)

  创建方式: 添加入  IdleHandler 方式

```java
Looper.myQueue.addIdleHandler(new MessageQueue.IdleHanlder() {
   @Override
   public boolean queueIdle() {
       //do something
       return false; //false表示执行一次后移除，ture下次msg为空继续回调
   }
})
```

下面我们继续看下核心方法 nativePollOnce

### MessageQueue.nativePollOnce

[frameworks/base/core/jni/android_os_MessageQueue.cpp](http://aospxref.com/android-10.0.0_r2/xref/frameworks/base/core/jni/android_os_MessageQueue.cpp)

JNI 层的方法，最后调用的是 native looper 的 `pollOnce(timeout)`方法
```c++
static void android_os_MessageQueue_nativePollOnce(JNIEnv* env, jobject obj,
        jlong ptr, jint timeoutMillis) {
    NativeMessageQueue* nativeMessageQueue = reinterpret_cast<NativeMessageQueue*>(ptr);
    nativeMessageQueue->pollOnce(env, obj, timeoutMillis);
}

void NativeMessageQueue::pollOnce(JNIEnv* env, jobject pollObj, int timeoutMillis) {
    mPollEnv = env;
    mPollObj = pollObj;
    mLooper->pollOnce(timeoutMillis);//调用 native 的 pollOnce 方法
    mPollObj = NULL;
    mPollEnv = NULL;
    ...
}
```

[/system/core/libutils/Looper.cpp](/system/core/libutils/Looper.cpp)

```c++
int Looper::pollOnce(int timeoutMillis, int* outFd, int* outEvents, void** outData) {
    int result = 0;
    for (;;) {
        while (mResponseIndex < mResponses.size()) {
            const Response& response = mResponses.itemAt(mResponseIndex++);
            int ident = response.request.ident;
            if (ident >= 0) {
                int fd = response.request.fd;
                int events = response.events;
                void* data = response.request.data;
#if DEBUG_POLL_AND_WAKE
                ALOGD("%p ~ pollOnce - returning signalled identifier %d: "
                        "fd=%d, events=0x%x, data=%p",
                        this, ident, fd, events, data);
#endif
                if (outFd != nullptr) *outFd = fd;
                if (outEvents != nullptr) *outEvents = events;
                if (outData != nullptr) *outData = data;
                return ident;
            }
        }

        if (result != 0) {
#if DEBUG_POLL_AND_WAKE
            ALOGD("%p ~ pollOnce - returning result %d", this, result);
#endif
            if (outFd != nullptr) *outFd = 0;
            if (outEvents != nullptr) *outEvents = 0;
            if (outData != nullptr) *outData = nullptr;
            return result;
        }

        result = pollInner(timeoutMillis);//调用的是 looper.pollInner方法
    }
}
```

Looper.poInner 方法实质就是调用 epoll wait 方法来监听事件，当之前监听的事件到来的时候，就读取里面的内容，然后返回，Java 层就可以往下去获取消息执行了

```c++
int Looper::pollInner(int timeoutMillis) {
    ...
    // Poll.
    int result = POLL_WAKE;
    ...
    struct epoll_event eventItems[EPOLL_MAX_EVENTS];
    //使用 epool_wait方式监听加入监听的事件，没有就阻塞,返回值是监听的事件到来的个数
    int eventCount = epoll_wait(mEpollFd.get(), eventItems, EPOLL_MAX_EVENTS, timeoutMillis);

    ...//错误处理到 done处

    // Handle all events.
    ...

    for (int i = 0; i < eventCount; i++) {//遍历监听到的事件数组
        int fd = eventItems[i].data.fd; // 获取 fd
        uint32_t epollEvents = eventItems[i].events;
        if (fd == mWakeEventFd.get()) { //如果 监听的事件是 当时设定的 wake fd事件
            if (epollEvents & EPOLLIN) {//监听的是读操作
                awoken(); //awoken的实质是去 去读监听事件的内容
            } else {
               ...
            }
        } else {
            ...
    }
Done: ; //处理 done的逻辑

    // 先处理 native 层的 消息
    ...
    // 在处理 respone
   ...
}

void Looper::awoken() {
    ...
    uint64_t counter;
    TEMP_FAILURE_RETRY(read(mWakeEventFd.get(), &counter, sizeof(uint64_t)));//awoke的实质是去读取管道中的内容
}
```

我们在来总结下，Java 层的消息队列建立后，开始循环，在取出下个消息时，若没有消息时，通过 epoll 机制等待被唤醒，而被唤醒后，也就是去读取下内容，将函数返回，我们在看下消息时如何发送的

## Handler 消息发送与处理
消息的发送 和 处理，都是通过 `Handler` 完成，先看下它是如何构造的

### handler 构造
Handler 的构造函数有好几个，但核心就是拿到该线程对应的 Looper对象，有了 Looper对象就有了消息队列，这里以默认构造函数为例看下流程

```java
   public Handler() {
        this(null, false);
    }

    public Handler(@Nullable Callback callback, boolean async) {
       ...
        mLooper = Looper.myLooper(); // 通过 Looper.mylooper静态方法获取当前线程的消息队列
        if (mLooper == null) {//若没有获取到，则说明之前没有初始化过，抛异常
            throw new RuntimeException(
                "Can't create handler inside thread " + Thread.currentThread()
                        + " that has not called Looper.prepare()");
        }
        //赋值其他变量
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```

### Handler.sendMessage
利用 Handler发送消息，因为消息队列是按照消息发生消息时间来排序的，所以所有发送消息的 API 最后都是调用 `sendMessageAtTime` 方法

```java
//发送消息，最终将消息入到消息队列中
    public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        ...//处理异常
        return enqueueMessage(queue, msg, uptimeMillis);
    }

    private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
            long uptimeMillis) {
        msg.target = this; // 这句非常重要，这里将 handler的赋值给了 msg.target，以便能让 handler来处理该消息
        ...
        return queue.enqueueMessage(msg, uptimeMillis); //让该消息入队列
    }
```

### MessageQueue.enqueueMessage
[/frameworks/base/core/java/android/os/MessageQueue.java](http://aospxref.com/android-10.0.0_r2/xref/frameworks/base/core/java/android/os/MessageQueue.java)

消息入消息队列过程就是一个单链表的插入过程，根据消息队列是否为空，标记 `mBlocked` 是否为 true, 为 true 则在插入后，立即唤醒主线程去获取消息，唤醒的方式是写入个值到监听的事件 fd 中。

```java
    boolean enqueueMessage(Message msg, long when) {
        ...//异常处理
        synchronized (this) {
            ...//异常处理
            msg.markInUse();
            msg.when = when;
            Message p = mMessages;//获取当前消息队列
            boolean needWake; //是否要唤醒消息队列
            if (p == null || when == 0 || when < p.when) {
                // 当消息队列为空，或者当前消息需要立即执行，该msg为首条消息，且唤醒
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked; //设置阻塞唤醒
            } else { //正常插入消息到队列的常规case
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {// 按照msg执行时间插入到对应的位置
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {// 假如需要唤醒，就立即唤醒
                nativeWake(mPtr);
            }
        }
        return true;
    }
```

[frameworks/base/core/jni/android_os_MessageQueue.cpp](http://aospxref.com/android-10.0.0_r2/xref/frameworks/base/core/jni/android_os_MessageQueue.cpp)


```c++
static void android_os_MessageQueue_nativeWake(JNIEnv* env, jclass clazz, jlong ptr) {
    NativeMessageQueue* nativeMessageQueue = reinterpret_cast<NativeMessageQueue*>(ptr);
    nativeMessageQueue->wake(); //调用 nativeMsgQueue的 wake 方法
}

void NativeMessageQueue::wake() {
    mLooper->wake();
}
```

[/system/core/libutils/Looper.cpp](http://aospxref.com/android-10.0.0_r2/xref/system/core/libutils/Looper.cpp)

```c++
void Looper::wake() {
    ...
    uint64_t inc = 1;
    // wake方法其实就是向wakefd 里面写了一个 1 进去，另一端epoll机制会监听这有内容可以读取
    ssize_t nWrite = TEMP_FAILURE_RETRY(write(mWakeEventFd.get(), &inc, sizeof(uint64_t)));
    ...
}
```

所以 handler发送消息的核心就是往消息队列按照消息执行的时间插入到队列中去，如果消息队列为空，或者插入到队首消息，则将该消息作为Head,并唤醒等待，取出消息执行。

好了，接下来我们看下消息处理的流程

### Handler.dispatchMessage

在 `Looper.loop()` 方法利用 `msg.target.dispatchMessage(msg)` 来派发处理消息，处理优先级为 `handler.post`方法 -> `handler构造callback` -> 复写`handleMessage`方法

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) { //使用Handler.post(runnbale)调用
        handleCallback(msg);
     } else {
          if (mCallback != null) {//使用Handler(callback)构造
             if (mCallback.handleMessage(msg)) {
                    return;
                }
          }
       handleMessage(msg); //回调Handler:handleMessage处理
    }
}

```
按照上面的优先级， Handler:post(runnbale)方式, 将 runnbale封装为msg，然后入队列，执行的时候首先处理。

```java
public final boolean post(Runnable r){
   return  sendMessageDelayed(getPostMessage(r), 0);
}
private static Message getPostMessage(Runnable r) {
   Message m = Message.obtain();
   m.callback = r; //将runable 赋值给msg属性
   return m;
}
```



# 总结

消息循环机制基于 Java层和Native层配合工作，Java层和Native层通过消息队列`MessageQueue` 连接，而消息的等待，唤醒则是通过 native层的 looper 执行的，handler 游走子线程和主线程之间，不断向绑定线程中的消息队列发送消息和处理消息。

1. 消息队列/Looper 对象 在 Java 层 和 Native 都对应有创建
2. Java 层消息队列无消息时阻塞，有消息来时，则会被唤醒起来取消息派发执行，底层原理就是通过 native 的 Looper 操作，利用 epoll 机制（一个线程去写值1，触发写入操作，另个线程则一直在监听该事件有值可读取了，唤醒后取出消息执行

Java层的消息机制，围绕3个类展开：

- Handler: 线程间消息的发送者和处理者，创建时会绑定到某个Looper对象
- Looper：每个线程拥有一个该对象，内部有一个消息队列，不断的循环消费消息
- MessageQueue: 一个基于链表实现的消息队列，排序为插入队列的时间

对象持有关系如下图，Handler 内部持有 Looper对象，Looper内部持有 MessageQueue对象，所以 MessageQueue 对象的创建是在 Looper初始化中进行，

![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/looper_data_struct.png)

各个类的在总体核心方法如下：
![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/looper_class_design.png)

整体的流程如下:
![](https://cloudy-liu.coding.net/p/BlogPicBed/d/BlogPicBed/git/raw/master/looper_flow.png)

## Refs
* [Android应用程序消息处理机制（Looper、Handler）分析](https://blog.csdn.net/luoshengyang/article/details/6817933)
* [Android消息机制2-Handler(Native层)](http://gityuan.com/2015/12/27/handler-message-native)

