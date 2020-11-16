---
title: Handler面试40问答案整理
date: 2020-11-16 14:21:02
tags:
	- Android

categories: Android
---


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
`MessageQueue`是基于触发时间`msg.when`时间戳倒序的优先级队列，使用同步屏障保证队列中靠后的消息会优先得到执行。一般只有Android系统能调用

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
> 主线程与工作线程之间，是共享内存地址空间的，所以是可以互相操作的，但是需要注意处理线程同步的问题。
工作线程通过主线的 Handler，向其成员变量 MessageQueue 中添加 Message

# Handler 的 IdleHandler 机制，如何理解？有什么用途
`IdleHandler` 是 `Handler` 提供的一种在消息队列空闲时，执行任务的时机 
- 执行时机是不可控的，不适合执行一些对时机要求比较高的任务。
- > 执行的时机依赖消息队列的情况，那么如果 `MessageQueue` 一直有待执行的消息时，`IdleHandler` 就一直得不到执行

在 `Looper` 事件循环的过程中，当出现空闲的时候，允许我们执行任务的一种机制