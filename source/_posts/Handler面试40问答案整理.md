---
title: Handler面试40问答案整理
date: 2020-11-16 14:21:02
tags:
	- Android
    - 面试
categories: 面试
---
摘自 [Handler高频40问] (https://mubu.com/doc/4AOg4eG-V2v)
部分可以见[Android 性能优化 - 面试经典问题](https://www.bilibili.com/video/BV1ZizbYEEcV/?spm_id_from=333.337.search-card.all.click&vd_source=728c2c15f151037482a49509ea153706)
- `epoll` 有什么优势？`Handler` 和管道(`Pipe`)的关系？`Android M` 开始，将 `Pipe` 换成了 `eventFd`是为什么？`Handler` 底层为什么使用管道，而不是 `Binder` ？`Handler` 可以 `IPC` 通信吗？
<!--more-->

# epoll为什么不阻塞
`nativePollOnce` 使用 `epoll` 监听文件描述符
`Object.wait()` 使用 `futex` Linux call
而 `Binder` 的 `oneway` 调用是阻塞的： `wait_event_interruptible` 和 `wait_up_interruptible`
# 何理解 nativePollOnce() 方法？
    1. 最终会调用到 Native 层的 pollInner() 方法。
    2. 在 pollInner() 中，处理流程：
        a. 先调用 epoll_wait()，这是阻塞方法，用户等待事件发生或超时。
        b. 对于 epoll_wait() 返回，当且仅当一下三种情况出现时，才会返回。
            POLL_ERROR，发生错误，直接跳转到 Done。
            POLL_TIMEOUT，发生超时，直接跳转到 Done。
            检测到管道有事件发生，则根据情况做响应的处理
                如果是管道有事件发生，则直接读取管道的数据。
                如果是其他事件，则处理 request，生成对应的 response 对象，push 到 response 数组。
        c. 进入 Done 标记为的代码段。
            先处理 Native 的 Message，调用 Native 的 Handler 来处理 Message。
            再处理 Response 数组，POLL_CALLBACK 类型的事件。
        d. 返回后，Java 层继续处理
# Handler MessageQueue 无消息时，为什么不出现 ANR
ANR需要先埋下炸弹，限时时间里解除才不会导致炸弹炸毁
Handler在无消息的时候是挂起的，直到下个消息来nativeWake()才接着做
> 应用未在规定的时间内处理 AMS 指定的任务才会 ANR。AMS 调用到应用端的 Binder 线程，应用再将任务封装成 Message 发送到主线程 Handler ，Looper.loop() 通过 MessageQueue.next() 拿到这个消息进行处理。如果不能及时处理这个消息呢，肯定是因为在它前面有耗时的消息处理，或者因为这个任务本身就很耗时。所以 ANR 不是因为 loop 循环，而是因为主线程中有耗时任务

1. `MessageQueue` 无消息时，会进入 `nativePollOnce()`休眠，此时无消息，处于休眠状态；
2. `MessageQueue` 有消息时，会立即通过 `nativeWake()` 唤醒去处理消息（通过往 `eventfd` 发起一个写操作，这样主线程就会收到一个可读事件进而从休眠状态被唤醒）

# 如果 Java 层 MessageQueue 中消息很少，但是响应时间却很长，是什么原因？
1. `MessageQueue` 队列中，该 `Message` 前的 `Message` 处理较为耗时；
2. `Native` 层消息过多，`Java` 层 `MessageQueue` 消息优先级最低，最后处理；
    > 先处理`Native Message`，再处理`Native Request`，最后处理`Java Message`。就是因为Native有消息才用Handler+epoll的，不然只有Java的消息wait() + notify

## 将Pipe换成了eventFd是为什么？
eventFd只用打开一个文件描述符，pipe要打开2个文件描述符，且要拷贝数据，eventFd用于通知消耗更少，完美兼容epoll逻辑，毕竟后出

# Handler 分发事件优先级，是否都可拦截？拦截的优先级如何？
可以统一拦截消息，**但无法拦截通过Runnable通过`getPostMessage(Runnable r)`生成的`Message`**。
因为它`msg.callback`不为空会优先处理`msg.callback`，不会经过统一的`Hanlder`的`mCallback`
Handler的`dispatchMessage`函数里：
```JAVA
    public void dispatchMessage(@NonNull Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                // mCallback 处理完如果返回 false，还是会继续往下走，再交给 Handler.handleMessage 处理的
                // 所以这边可以通过反射去 hook 一个 Handler ，可以监听 Handler 处理的每个消息，也可以改 msg 里面的值
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }

    private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }
```

# Handler 发送延迟消息，涉及哪些方法？原理是什么？会被开机时间影响吗
1. `xxDelayed()`和 `xxxAtTime()`，最终都会调用到 `enqueueMessage()` 入队
从链表头按最近时间戳倒序排列，搜索直到找到一个比入队消息执行时间小的消息插入其前

2. 不受开机时间影响，因其延迟消息基于`SystemClock.uptimeMillis()`，此为设备开机时间，不受时钟影响，锁屏关机的时间不会记录在内

# 主线程 Looper 何时运行？
`ActivityThread`的main方法，主动调用 Looper.prepareMainLooper() 和 Looper.loop()

# 异步Message和Message同步屏障是什么
`MessageQueue`是基于触发时间`msg.when`时间戳倒序队列（时间戳越小的，`msg.when`越小的，排在前面），使用同步屏障保证队列中靠后的消息会优先得到执行。一般只有Android系统能调用

1. 同步 Message：普通 Message；
2. 异步 Message：`msg.setAsynchronous(true)`
>  UI 更新相关的消息即为异步消息，需要优先处理
3. 同步屏障(`Sync Barrier`)：`msg.target == null` 
> 通过Hanlder发消息的时候会持有发送的Hanlder，`msg.target = this`

## 同步屏障干嘛的
同步屏障的作用是在`MessageQueue`中取下一个消息`next()`时，如果找到了`target==null`的同步屏障消息，需要循环遍历，一直往后找到第一个异步的消息。即阻碍同步消息，让异步消息优先

```JAVA
//ViewRootImpl.java
    void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            //开启同步屏障
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            //发送异步消息
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            
            
        }
    }
    // 用完后通过removeSyncBarrier()移除同步屏障
    void unscheduleTraversals() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            //移除同步屏障
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
            mChoreographer.removeCallbacks(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        }
    }

```


# nextPollTimeoutMillis什么作用
计算出距离上一个postDelay消息执行的，还要等待多久
```JAVA
    // 1.如果nextPollTimeoutMillis=-1，一直阻塞不会超时。
    // 2.如果nextPollTimeoutMillis=0，不会阻塞，立即返回。
    // 3.如果nextPollTimeoutMillis>0，最长阻塞nextPollTimeoutMillis毫秒(超时)
    //   如果期间有程序唤醒会立即返回。
    int nextPollTimeoutMillis = 0;
```

# 同一个 Message 对象能否重复 send？
1. 角度一：Java 对象层面，可被复用；原因：Message 由消息池维护，即同一个对象被回收后会被再次复用； `new Message()` & `Message.obtain()`
2. 角度二：业务层面，不能复用；原因：Message 通过 enqueueMessage() 入队时，会通过 markInUse() 标记，再次入队无法通过 isInUse() 检查，则抛出异常；
```JAVA
    // MessageQueue.java
    boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        synchronized (this) {
            if (msg.isInUse()) {
                throw new IllegalStateException(msg + " This message is already in use.");
            }

            if (mQuitting) {
                // ...
                return false;
            }
            msg.markInUse();
        }
    }
```


# Looper 如何保证线程唯一？
基于 TLS 机制，Looper 首次 `prepare()` 时，会将 Looper 存入 ThreadLocal 中，再次 `prepare() `时检查 ThreadLocal 是否已存储，如果已经有了则会抛异常，所以同一个线程只能`prepare()` 一次

# Message 消息池的结构？最大缓存多少个 Message 对象？
`Message` 类的静态属性 `sPool` 维护一个链表，最大50个
享元模式，避免重复构造 Message。
回收资源，回收时清理 msg 持有的 callback 和 target，避免内存泄露。

# Looper 的 Printer 输出的日志，有什么其他用途？依靠的原理是什么？有什么缺点？
每个`Message`执行开始和结束的时间会打印判断`Message`操作耗时，即主线程卡顿耗时，做性能监控
缺点：Printer 存在大量字符串拼接，在消息量大时，会导致性能受损。在列表快速滑动时，平均帧率降低 5 帧
> `BlockCanary`就是用此设计

# 子线程如何向主线程的 Handler 发送消息？为什么经过 Handler 就可以达到切线程的目的？
可以子线程发送消息是因为`Handler`的成员变量 `MessageQueue#Message` 维护了一个消息队列（链表），向这个消息队列插入消息，并使用同步的手段（`synchronized (this) `）就可以进行达成线程间共享消息。
向`Handler`的成员变量 `MessageQueue` 中添加消息 `Message`，下次`Looper#loop()`循环到的时候就可以检测到这个消息`Message`


> 主线程与工作线程之间，是共享内存地址空间的，所以是可以互相操作的，但是需要注意处理线程同步的问题。
- 工作线程通过主线的 `Handler`，就可以向其成员变量 MessageQueue 中添加 Message
- 主线程的`WorkHandler`只要拿到工作线程`HandlerThread`的`Looper`就可以通过向`WorkHandler`的发消息了

```JAVA
 protected void onCreate(Bundle savedInstanceState) {
     // ...
    mHandlerThread = new HandlerThread("HandlerThread/Demo");
    mHandlerThread.start();
    mHandler = new WorkHandler(mHandlerThread.getLooper());
    Message msg = mHandler.obtainMessage();
    msg.obj = Thread.currentThread().getName();
    mHandler.sendMessage(msg);
 }
```
# Handler 的 IdleHandler 机制，如何理解？有什么用途
`IdleHandler` 是 `Handler` 提供的一种在消息队列空闲时，执行任务的时机 
- 执行时机是不可控的，不适合执行一些对时机要求比较高的任务。
- > 执行的时机依赖消息队列的情况，那么如果 `MessageQueue` 一直有待执行的消息时，`IdleHandler` 就一直得不到执行

在 `Looper` 事件循环的过程中，当出现空闲的时候，允许我们执行任务的一种机制
> 出现空闲指本次`Looper`在取`MessageQueue#next()`下一条消息为空或下一条消息还没到执行时间时
```JAVA
// MessageQueue.java
    // If first time idle, then get the number of idlers to run.
    // Idle handles only run if the queue is empty or if the first message
    // in the queue (possibly a barrier) is due to be handled in the future.
    if (pendingIdleHandlerCount < 0
            && (mMessages == null || now < mMessages.when)) {
        pendingIdleHandlerCount = mIdleHandlers.size();
    }
```
`IdleHandler`的`queueIdle()` 返回值分：持续回调（true） & 一次性回调（false），false 会导致执行完后，从 mIdleHandlers 中移除
```JAVA
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
```

## IdleHandler 的 queueIdle() 返回 true，为什么不会死循环？
本地变量`pendingIdleHandlerCount` 初始化为-1，如果有机会执行的话会置为0，所以每次`MessageQueue#next()`时只经过一次，本次取消息循环里不会再重复执行。下次`Looper`再取`MessageQueue#next()`时才会执行。
```JAVA
// MessageQueue.java
    Message next() {
        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        for (;;) {
                // 在这里没有找到任何消息返回到此处
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                // ...
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

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            // 执行完IdleHandler后，重新分发最近的消息，因为IdleHandler是耗时的，过程中可能有新消息来
            nextPollTimeoutMillis = 0;
        }
    }

```

# IdleHandler 执行耗时会影响正常的消息分发吗？
会，`IdleHandler`耗时不可控，执行时机也不可控，执行完后会重置 nextPollTimeoutMillis = 0，重新分发最近消息，因为IdleHandler是耗时的，过程中可能有新消息来

# Handler 在 Activity 中使用，什么场景下会出现内存泄露？原因是什么？如何规避？
主线程生命周期长于四大组件，`msg.target` 指向 `Handler，而` `Handler` 作为内部类持有外部类 `Activity` 的引用，导致 `Activity` 泄露
1, 静态内部类 `Handler` + `Activity` 弱引用；
2. 随 `Activity` 生命周期，onDestory() 会 remove 掉所有的消息(类似`Handler#removeCallbacksAndMessages()`)；


# Handler在removeMessages时为什么两次循环
> 因为多个handler对应同一个MessageQueue对应同一个Looper，所以一个`mMessages`里有多个Handler发的消息。
> 从链表头开始删除所有 `Handler` h 发的对应消息时，中间可能会遇到条件不符合的其他Handler发送的消息，这时需要再开启一个循环从这个不符合条件的n的下一个消息nn开始搜。

其实问题就在于如何删除列表头？第二个循环已经足够删除所有不连续的 `Handler` h 发的对应消息，但无法删除链表头，因为是从链表头开始的。所有第一个循环就是干这个事的，从链表头开始删除连续的符合条件的消息
```JAVA
// MessageQueue.java
    void removeMessages(Handler h, int what, Object object) {
        if (h == null) {
            return;
        }

        synchronized (this) {
            Message p = mMessages;

            // Remove all messages at front.
            // 条件A：是这个Handler h 发的且what 和object对应的消息
            // 从mMessages链表头开始找是不是有符合条件A的消息，并摘除
            while (p != null && p.target == h && p.what == what
                   && (object == null || p.obj == object)) {
                Message n = p.next;
                mMessages = n; // 摘除p
                p.recycleUnchecked();
                p = n;
            }
            // 此处遍历完mMessages或者遇到第一个不符合条件A的消息

            // Remove all messages after front.
            while (p != null) {
                // 从这个不符合条件A的消息的下一个开始找
                Message n = p.next;
                if (n != null) {
                    // 找到符合条件A的消息
                    if (n.target == h && n.what == what
                        && (object == null || n.obj == object)) {
                            
                        Message nn = n.next;
                        n.recycleUnchecked();
                        p.next = nn; // 删掉n
                        continue; // 遍历n的下一个nn
                    }
                }
                // 没有符合A的，不删除n，直接跳过n往后遍历
                p = n;
            }
        }
    }
```
# 如何理解 HandlerThread？
- 继承 `Thread`，根据`TLS`(ThreadLocal)维护子线程的 `Looper`，在子线程`Looper#loop()`
- 持有了`Handler`，开启了`Looper.prepare()`与`Looper.loop()`循环
- 在 `IntentService` 后台任务执行时也直接持有了一个`HandlerThread`来挨个执行任务，并且`IntentService`全部执行任务完毕后会自动停止

## 普通Service是运行在哪个线程上的？
如果是Local Service，那么对应的 Service 是运行在主进程的 main 线程上的。如：onCreate，onStart 这些函数在被系统调用的时候都是在主进程的 main 线程上运行的。如果是Remote Service，那么对应的 Service 则是运行在独立进程的 main 线程上。

## IntentService有何优点？
适用场景：
1. 需要后台执行耗时操作（如网络请求、数据库批量操作）。
2. 任务之间需按顺序执行（如多个文件下载）。
注意事项
1. 构造函数要求：必须提供无参构造函数，并调用 super("threadName")。
2. 避免阻塞主线程：onHandleIntent 中的代码需在子线程完成，否则仍会导致 ANR。
3. 内存泄漏：若在 onHandleIntent 中持有 Activity 引用，需及时释放。
高频问题
1. 为什么 IntentService 能处理耗时操作？
    - 通过 HandlerThread 创建子线程，任务在子线程执行，避免主线程阻塞。请求的执行顺序是按启动顺序串行执行，队列管理由系统自动处理。
2. IntentService 的 onHandleIntent 和 Service 的 onStartCommand 有何区别？
    - onStartCommand 运行在主线程，需手动处理线程；onHandleIntent 自动在子线程执行。
3. IntentService 的好处
    - 与普通 Service、WorkManager 对比：封装了线程和队列，简化了耗时任务开发。
    - 与 WorkManager 对比：WorkManager 支持更复杂的调度策略（如网络条件、电池状态），但 IntentService 更轻量。


# 如何实现子线程等待主线程处理消息结束后，再继续执行？原理是什么？
使用 `Handler` 的 `boolean runWithScissors(final Runnable r, long timeout)`
实现 A 线程阻塞等待 B 线程处理完消息后再继续执行的功能。
## 原理
很危险可能造成死锁，是`@hide`不允许非系统调用
- 如果timeout超时了，不阻塞调用线程了直接`return false`，但是没有取消`runnable`的逻辑消息还是会排队排到执行。
- 调用线程进入阻塞(`wait()`)，不排队排到执行完成不会被唤醒，如果当前runnable里的代码持有别的锁，会造成死锁，本调用线程永远不执行了


```JAVA
// Handler
    public final boolean runWithScissors(@NonNull Runnable r, long timeout) {
        // ...
        // 如果同一个线程直接运行
        if (Looper.myLooper() == mLooper) {
            r.run();
            return true;
        }

        BlockingRunnable br = new BlockingRunnable(r);
        return br.postAndWait(this, timeout);
    }

    private static final class BlockingRunnable implements Runnable {
        private final Runnable mTask;
        private boolean mDone;

        public BlockingRunnable(Runnable task) {
            mTask = task;
        }

        @Override
        public void run() {
            try {
                // MessageQueue排到的时候执行
                mTask.run();
            } finally {
                synchronized (this) {
                    mDone = true;
                    notifyAll(); // 唤醒阻塞
                }
            }
        }

        public boolean postAndWait(Handler handler, long timeout) {
            // 入队到handler的线程
            if (!handler.post(this)) {
                return false;
            }

            // 软件实现只要没完成mDone一直while阻塞在这里
            synchronized (this) {
                if (timeout > 0) {
                    // 如果延时
                    final long expirationTime = SystemClock.uptimeMillis() + timeout;
                    while (!mDone) {
                        long delay = expirationTime - SystemClock.uptimeMillis();
                        if (delay <= 0) {
                            // 当超时退出时，这个 Runnable 依然还在目标线程的 MessageQueue 中，没有被移除掉，它最终还是会被 Handler 线程调度并执行。显然并不符合业务预期
                            return false; // timeout
                        }
                        try {
                            wait(delay);
                        } catch (InterruptedException ex) {
                        }
                    }
                } else {
                    while (!mDone) {
                        try {
                            wait();
                        } catch (InterruptedException ex) {
                        }
                    }
                }
            }
            return true;
        }
    }
```
## 为什么被标记为 hide？存在什么问题？原因是什么？
在子线程 Looper 中使用，可能导致 A 线程进入 wait 等待，而永远得不到被 notify 唤醒
- 原因： 子线程 `Looper` 允许退出，若包装的 `BlockingRunnable` 被执行前，`MessageQueue` 退出，则该 `runnable` 永远不会被执行，则会导致 A 线程一直处于 `wait` 等待，永远不会被 `notify` 唤醒
- 解决方法：要求该`Handler`的`Looper`使用`quitSafely() `不能使用`quit()`。`quit()`会清理掉所有未执行的任务。 `quitSafely()` 只会清理掉当前时间点之后(when > now)的消息。这样`runWithScissors()`发送的任务，依然会被执行。

# 性能优化
## Java线程的调度、线程的生命周期、notify和wait后会进入什么阶段

## 线程池的配置参数

## 线程对CPU资源的占用，如何能让线程跑在高和CPU上

## 一个线程占多少内存？一个线程栈大小由多少，怎么避免栈溢出

