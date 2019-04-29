---
title: Handler与消息机制
date: 2019-04-28 23:06:25
tags: Handler
---



探究Handler

<!-- more -->

#### Android消息机制

- Message：硬件产生的消息-->触摸，软件产生的信息
- MessageQueue：存取Message
- Handler：向消息池发送各种消息(`Handler.sendMessage`)，并处理对应消息(`Handler.handleMessage`)
- Looper：死循环执行(`Looper.loop`)，并将消息分发给目标处理



#### ThreadLocal：

线程本地存储区TLS，用于存储线程私有的数据

通过get()，set()操作存取线程唯一的Looper



##### 一个关于Handler和Looper的典型线程例子

```java
class LooperThread extends Thread {
        public Handler mHandler;
        public void run() {
            Looper.prepare();
            mHandler = new Handler() {
                public void handleMessage(Message msg) {
                    // process incoming messages here
                }
            };
            Looper.loop();
        }
    }
```



##### Looper

执行Looper.prepare()，且在每个线程中仅能执行一次。然后创建Looper对象，此时会将Looper存入TLS中。

```java
private static void prepare(boolean quitAllowed) {
    	//一般quitAllowed为true,调用quit()时用到该参数
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```

创建Looper对象时，Looper会在构造器中初始化一个MessageQueue对象，并持有当前线程的引用。

```java
private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
	}
```

Looper.loop()后开始循环，不断的从Quene中取出Message，然后通过Message中Handler引用进行分发Message，最终将分发的Message回收到消息池进行复用。

```java
    public static void loop() {
        final Looper me = myLooper();//从TLS中获取Looper对象，
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        //省略部分代码

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
            
		//省略部分代码	
            
     	//调用Handler分发Msg，该方法中会调用HandleMessage()
        msg.target.dispatchMessage(msg);
            
        //省略部分代码 
            
        // 确保线程不被破坏，然后将Msg放入消息池复用
        final long newIdent = Binder.clearCallingIdentity();
        msg.recycleUnchecked();
            
        }
    }
```

Looper.quit()和quitSafely()调用后线程结束，停止工作。

该方法最终会调用Queue的quit()来清除消息

```java
void quit(boolean safe) {
    	//prepare()中的mQuitAllowed参数在这里用到，为false就会抛出异常
        if (!mQuitAllowed) {
            throw new IllegalStateException("Main thread not allowed to quit.");
        }

        synchronized (this) {
            if (mQuitting) {
                return;
            }
            mQuitting = true;

            if (safe) {
                //移除所有未触发的信息
                removeAllFutureMessagesLocked();
            } else {
                removeAllMessagesLocked();//Quene中所有信息都会被移除
            }

            // We can assume mPtr != 0 because mQuitting was previously false.
            nativeWake(mPtr);
        }
    }
```

safe = true时，若此时已经轮到某个信息要被处理，就不会移除该信息，只移除他之后的

safe = false时，不管有没有轮到信息，一律移除



工作机制

- Handler通过sendMessage()发送Message到MessageQueue队列；
- Looper通过loop()，不断提取出达到触发条件的Message，并将Message交给target来处理；
- 经过dispatchMessage()后，交回给Handler的handleMessage()来进行相应地处理。
- 将Message加入MessageQueue时，通过往 pipe 管道写端写入数据，会唤醒loop所在线程；如果MessageQueue中没有Message，并处于Idle状态，则会执行IdelHandler接口中的方法，往往用于做一些清理性地工作。
- quit()调用，线程结束，停止工作



利用Looper与线程关联的机制，使得在子线程的更新UI(本质还是在UI线程更新)

```java
new Thread(new Runnable() { 
    @Override 
    public void run() { 
        //工作线程
        Log.i("tag", "step 1 "+Thread.currentThread().getName()); 
        Handler handler=new Handler(getMainLooper()); //传入主线程的Looper
        handler.post(new Runnable() { 
            @Override 
            public void run() { 
                //这里是运行在UI线程的
                Log.i("tag", "step 2 "+Thread.currentThread().getName()); 
            } 
        }); 

    } 
}).start();
```





1. Handler如何与线程关联

​       Thread中维护着一个TLS(线程本地存储区)，这里会存放着线程私有的数据，如Looper。每个线程都有且仅有一个Looper，而Looper又管理着该线程中的MessageQueue，进而不断处理队列中的Message.

​       而Handler在(无参构造函数)初始化的时候会默认与当前线程的Looper进行绑定，间接与线程相关联。

2. Handler发消息是由谁管理的

   Handler中会持有Looper和MessageQueue的引用，当Handler发送消息时，会通过`MessageQueue.enqueueMessage(Message,long)`进行发送。MessageQueue通过判断那些Message需要处理，从而将Message进行排列，等待Looper进行分发

3. 消息如何回到`handleMessage(msg)`的

   `Looper.loop()`调用后，会不断的从queue中的获取Message，然后通过Message中持有的Handler引用，调用`Handler.dispatchMessage(msg)`，从而将Message分发出去，而在该方法中，会根据Message有无设置Callback，Handler有无设置CallBack，从而调用不同的handleMessage(msg)进行处理。

**消息分发的优先级：**

- Message的回调方法：`message.callback.run()`，优先级最高；

- Handler的回调方法：`Handler.mCallback.handleMessage(msg)`，优先级仅次于1；

- Handler的默认方法：`Handler.handleMessage(msg)`，优先级最低。



4. Handler是如何切换线程的

   Handler初始化的时候会指定一个线程的Looper，Looper对应着哪个线程，Handler就在哪个线程。

   而一般我们都是在主线程对Handler进行初始化的，而在工作线程进行发送消息的，那么消息就由工作线程切换到了主线程

5. Looper.loop()的死循环为什么不会卡死

   Android应用是运行在主线程的，而Looper.loop()结束运行，意味着应用也已经结束了。通过loop()不断获取 消息，再通过Handler进行处理。当loop()获取不到消息时，就会进入阻塞状态，释放掉CPU，此时不会耗费太多的资源。等到下一个消息到来主线程就会重新获取CPU，继续工作。

   从消息队列中取消息可能会阻塞，取到消息会做出相应的处理。如果某个消息处理时间过长，就可能会影响UI线程的刷新速率，造成卡顿的现象。

   进入死循环前会创建新Binder线程`(thread.attach(false) 方法函数中便会创建一个 Binder 线程)`，在这个线程中进行发送消息给主线程，从而让主线程处于工作状态。

   

6. 为什么Handler使用的是`SystemClock.uptimeMillis()`开机到现在的时间

   该方法计算的时间不包括休眠的时间，而Handler在工作线程发送消息给主线程，会受到阻塞，挂起状态的影响，当主线程处于阻塞时，发送的消息必然会唤醒主线程，这样会使主线程抢占了别的线程的资源。这样会影响用户体验。是不好的

222









笔记摘抄源自[gityuan博客](<http://gityuan.com/2015/12/26/handler-message-framework/>)

