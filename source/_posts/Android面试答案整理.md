---
title: Android面试答案整理
date: 2020-11-09 18:48:47
tags:
    - 面试
categories: 面试
---

算法 、 Android基础、 梳理下自己做过的项目的细节，了解下原理

## Retrofit(动态代理)

## RN和flutter绘制原理
## Android冷启动与冷启动优化

## binder机制原理

## gradle生命周期

<!--more-->
## apk资源加载以及activitythread相关的知识，或者inputevent机制

## Android 系统启动流程吗？

## Activity的启动流程

## AMS启动流程

## 预热webview；预加载离线包；

## 系统级别除了Binder还有哪些跨进程方式？ Zygote通过Socket监听来fork新的进程，native crash发出信号kill process


## 插件化组件化　阿里Atlas 360的DroidPlugin技术（项目用到了）对比

## 动态化方案
Weex  ReactNative  flutter

## Xposed
是干嘛的？

## 音频文件会选择怎样的压缩方式？zlib？
```JS
"accept-encoding", "deflate, br"
"accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8"
```

> Android 视频压缩常见3种方案:(1)FFmpeg,(2)mp4praser,(3)自带的MediaCodec
## 媒体文件断点续传是什么formData字段？你们的录音文件是一帧一帧的流传输吗？录音文件可以停止再录吗
 
## 流式的加密和压缩

###  
使用ZipFile打开的所有ZIP文件都在内存中存在映射，所以使用ZipFile的性能优于 ZipInputStream。然而，如果同一ZIP文件的内容在程序执行期间经常改变，或是重载的话，使用ZipInputStream就成为你的首选了。
> ZipInputStream和FileInputStream流读出的时候，ZIP条目不使用高速缓冲

## AudioFlinger 和 环形FIFO
1. 环形FIFO
这样AudioTrack和AudioFlinger管理着同一个 audio_track_cblk_t，通过它实现了环形FIFO。AudioTrack向FIFO中写入音频数据，AudioFlinger从FIFO中读取音频数据，经Mixer后送给AudioHardware进行播放。
1) AudioTrack是FIFO的数据生产者；
2) AudioFlinger是FIFO的数据消费者；

2. construct the shared structure in-place.
```C++
    // 这是 C++ 的 placement new（定位创建对象）语法：new(@BUFFER) @CLASS();
    // 可以在特定内存位置上构造一个对象
    // 这里，在匿名共享内存首地址上构造了一个 audio_track_cblk_t 对象
    // 这样 AudioTrack 与 AudioFlinger 都能访问这个 audio_track_cblk_t 对象了
```

## 火焰图的原理是什么？systrace和traceView的原理是怎样的呢？他们计算函数耗时的采样率是怎样的？
> `Rabbit`计算卡顿的时候就是周期性的采集主线程堆栈。因为：
> 1. 获取主线程的堆栈可能会导致主线程卡顿
> 2. 卡顿发生时，获取到前面卡顿时的堆栈可能不准
> 每隔16.67ms在异步线程获取一下主线程的堆栈然后保存起来，在卡顿发生时，把这些周期性采集的堆栈当做卡顿时的堆栈。这种方法可以抓住大部分卡顿现场,不过也会获取一些无用的堆栈信息。

## EventBus原理是什么？
### EventBus粘性事件是什么
在订阅者订阅粘性事件时，会自动匹配最后发送的粘性事件。
> 在Android开发中，Sticky事件只指事件消费者在事件发布之后才注册的也能接收到该事件的特殊类型。Android中就有这样的实例，也就是Sticky Broadcast，即粘性广播。正常情况下如果发送者发送了某个广播，而接收者在这个广播发送后才注册自己的Receiver，这时接收者便 无法接收到刚才的广播，为此Android引入了StickyBroadcast，在广播发送结束后会保存刚刚发送的广播（Intent），这样当接收者注册完Receiver 后就可以接收到刚才已经发布的广播。这就使得我们可以预先处理一些事件，让有消费者时再把这些事件投递给消费者


### EventBus BroadCast和Handler-优缺点比较

## 有用过MVVM吗？ RecyclerView的adapter是持有的源数据源吗，放在presenter为什么不好，RecyclerView的复用过程

## 为什么不能一边offsetTopAndBottom一边动画呢？会一直触发onLayout，为啥不能同时进行
 
## 为什么要求尽量不写全局变量？
1. 要求尽量不写全局变量是因为你要确保清除全局变量的引用或者重置全局变量的值，而这很容易被遗漏。
2. 如果这个strongReference是全局变量时，就需要在不用这个对象时赋值为null，因为强引用不会被垃圾回收。

## Java的强引用、软引用、弱引用和虚引用原理
如果一个对象只具有软引用，则内存空间充足时，垃圾回收器就不会回收它；
如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。

## 点击屏幕后触发的流程讲一下，从硬件开始说。
1. 在 ViewRootImpl 里有一个 WindowInputEventReceiver 用来接受事件并进行分发。
InputChannel 发送的事件最终都是通过 WindowInputEventReceiver 进行接受。
WindowInputEventReceiver 是在 ViewRootImpl.setView 里面初始化的，setView 的调用是在 ActivityThread.handleResumeActivity -> WindowManagerGlobal.addView
2. FrameDisplayEventReceiver继承自DisplayEventReceiver接收底层的VSync信号开始处理UI过程。VSync信号由SurfaceFlinger实现并定时发送。FrameDisplayEventReceiver收到信号后，调用onVsync方法组织消息发送到主线程处理。这个消息主要内容就是run方法里面的doFrame了，这里mTimestampNanos是信号到来的时间参数。

## 如何计算冷启动时间？冷启动的优化是什么？
![启动app过程](https://img-blog.csdn.net/20180823215319329?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FpYW41MjBhbw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## View的事件分发
1. [Android事件分发机制 详解攻略，您值得拥有](https://blog.csdn.net/carson_ho/article/details/54136311)

## cancel事件什么时候触发
 1. View 收到 ACTION_DOWN 事件以后，上一个事件还没有结束（可能因为 APP 的切换、ANR 等导致系统扔掉了后续的事件），这个时候会先执行一次 ACTION_CANCEL
 2. 子 View 之前拦截了事件，但是后面父 View 重新拦截了事件，这个时候会给子 View 发送 ACTION_CANCEL 事件
 > ——[事件分发三连问：事件是如何从屏幕点击最终到达 Activity 的？CANCEL 事件什么时候会触发？如何解决滑动冲突？](https://blog.csdn.net/Androiddddd/article/details/108804170)
3. 针对多指POINTER_ID的情况是用一个链表保存的。[ViewGroup事件分发总结-TouchTarget](https://blog.csdn.net/dehang0/article/details/104317611)
 
## View的绘制流程
1. 每个Activity 的根布局都是 DecorView， DecorView 是继承自 Fragment，DecorView 的 mParent 是 ViewRootImpl 

2. View 发起刷新的操作时，会层层通知到 ViewRootImpl 的 scheduleTraversals() 里去，然后这个方法会将遍历绘制 View 树的操作 performTraversals() 封装到 Runnable 里，传给 Choreographer，以当前的时间戳放进一个 mCallbackQueue 队列里，然后调用了 native 层的vsync信号
   > onVsync() 是底层会回调的，可以理解成每隔 16.6ms 一个帧信号来的时候，底层就会回调这个方法，当然前提是我们得先注册
3.  invalidate()，requestLayout()，等之类刷新界面的操作时，并不是马上就会执行这些刷新的操作，而是通过 ViewRootImpl 的 scheduleTraversals() 先向底层注册监听下一个屏幕刷新信号事件，然后等下一个屏幕刷新信号来的时候，才会去通过 performTraversals() 遍历绘制 View 树来执行这些刷新操作
   > - 16.6ms内会过滤一帧内重复的刷新请求
   > - 添加同步屏障的消息拦截同步消息的执行
      > 1. 主线程的 Looper 会一直循环调用 MessageQueue 的 next() 来取出队头的 Message 执行， 当 next() 方法在取 Message 时发现队头是一个同步屏障的消息时，就会去遍历整个队列，只寻找设置了异步标志的消息
	  > 2. 有找到异步消息，那么就取出这个异步消息来执行
	  > 3. 没有找到异步消息就让 next() 方法陷入阻塞状态，主线程此时就是处于空闲状态的，也就是没在干任何事。直到这个同步屏障消息被移除出队列，否则主线程就一直不会去处理同步屏幕后面的同步消息。

### 为什么会丢帧	
- 一是遍历绘制 View 树计算屏幕数据的时间超过了 16.6ms
- 二是主线程一直在处理其他耗时的消息，导致遍历绘制 View 树的工作迟迟不能开始，从而超过了 16.6 ms 底层切换下一帧画面的时机

## 计算一个view的嵌套层级
递归往上调用`getParent()`
## 有了解音频底下audioPlayer的原理吗

## 上传媒体文件断点续传是什么formData字段？你们的录音文件是一帧一帧的流传输吗？录音文件可以停止再录吗
 
## View的事件分发
1. [Android事件分发机制 详解攻略，您值得拥有](https://blog.csdn.net/carson_ho/article/details/54136311)

## Activity启动后View何时开始绘制（onCreate中还是onResume之后？）⭐
[Activity启动后View何时开始绘制（onCreate中还是onResume之后？）](https://www.jianshu.com/p/c5d200dde486)
- 由于onCreate会先于handleResumeActivity执行，我们在onCreate中调用了setContentView，也只是生成DecorView并给这个DecorView的内容设置了布局而已，此时还并没有把这个DecorView添加到Window中，同样，WMS中也还没有这个Window，所以此时并不能做任何事情（绘制、接收点击事件等），虽然调用了requestLayout和invalidate，并不会真正触发布局和重绘（因为还没有与ViewRootImpl进行绑定）
- Activity与Window产生联系，是在调用activity#attach方法中，生成了一个PhoneWindow，并把这个activity对象自身，设置给了Window的Callback回调（Activity实现了Window的Callback接口）
- Window与WindowManagerService产生联系，是在handleResumeActivity中，先执行了onResume方法，通过调用WindowManagerImpl#addView方法将这个Activity对应的DecorView添加到了这个Window中，addView方法是一个IPC操作，将这个Window也添加到了WindowManagerService中；
- ViewRootImpl与Window产生联系，是在WindowManagerImpl#addView方法中，这个过程中会new一个ViewRootImpl与DecorView相对应，保存在WindowManagerGloable中；
- 总的来说，就是setContentView生成了DecorView及其视图，在onResume之后才把这个视图添加进了Window和WMS中，具备了交互能力；

## 【异常处理】Activity和进程发生异常时的自动恢复逻辑
[【异常处理】Activity和进程发生异常时的自动恢复逻辑](https://blog.csdn.net/u013718730/article/details/102021431)

## Activity渲染完成第一帧时机
[ Activity渲染完成第一帧时机](https://blog.csdn.net/brycegao321/article/details/101147431)


 ## Android匿名内存
 同时多进程间通过mmap共享文件数据的时候，仅需要一块物理内存就够了。
​ Android中使用mmap，可以通过RandomAccessFile与MappedByteBuffer来配合。
通过randomAccessFile.getChannel().map获取到MappedByteBuffer。然后调用ByteBuffer的put方法添加数据。
## SharedPreferences是跨进程安全的吗？工程中用什么替代？
可以mmkv，ContentProvider也更适合跨进程共
> SharedPreferences 的 N 宗罪：
> 1. 跨进程不安全
> 2. 加载缓慢：异步加载，但是异步加载线程没有设置优先级，如果这时候主线程读取数据需要等待加载线程执行完毕 (也就是主线程等待低优先级线程锁的问题)
> 3. 全量写入：无论是 commit 还是 apply，即使改动一个条目，也会把全部内容写到文件
> 4. 卡顿：异步落盘机制在应用崩溃时会导致数据丢失

## ContentProvider是线程安全的吗？SharedPreferences是线程安全的吗？
- SharedPreferences是线程安全的，不是进程安全
- ContentProvider不是线程安全的，是进程安全的

## 如何减少卡顿？刷新原理
![img](https://user-gold-cdn.xitu.io/2020/3/27/17117aa6eb5b454e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


## 内存泄漏原理是什么，LeakCanary是怎么检测到内存分析的？
- LeakCanary将已经销毁的Activity和Fragment所对应的实例放入到弱引用中，并关联一个引用队列。通过registerActivityLifecycleCallbacks来注册对于我们销毁的Activity的监听.

- 如果实例进行了回收，那么弱引用就会放入到ReferenceQueue中
- 如果一段时间后，所监控的实例还未在ReferenceQueue中出现，那么可以证明出现了内存泄漏导致了实例没有被回收

- 使用lifeCycle来监听Activity和Fragment的生命周期，在onDestory()、onViewDestroy、onFragmentDestroy()时会调用`watch()`检查

- `watch()`会生成一个Activity的UUID到retainedKeys队列中，retainedKeys队列记录了我们执行了监控的引用对象。而queue中会保存回收的引用。所以通过二者的对比，我们就可以找到内存泄漏的引用了

- `watch()`在主线程空闲的时候 `Looper.myQueue().addIdleHandler()`一直检测，多次重试后最终会调用`ensureGone`

- 如果监控对象没有回收，执行一次gc()，检测移除已经回收的监控对象，如果还是没有回收，证明发生了内存泄漏。此时dump出hprof文件到另一个前台服务IntentService，启动时会调用onHandleIntent方法，该方法在父类中实现了。实现中会调用onHandleIntentInForeground()方法
	> 对于执行垃圾回收需要使用Runtime.getRuntime().gc()， 

- 调用接口，将结果回调给listenerClassName所对应的类（这里是DisplayLeakService类）来进行处理

## Serializable 和 Parceable 原理
- Serializable 的原理是通过 `ObjectInputStream` 和 `ObjectOutputStream` 来实现的
通过反射递归序列化内部对象
  > 可以重写writeObject 和 readObject 方法，实现序列化的加密功能，或者版本兼容
	```Java
	// 序列化
	E/test:SerializableTestData writeReplace
	E/test:SerializableTestData writeObject

	// 反序列化
	E/test:SerializableTestData readObject
	E/test:SerializableTestData readResolve
	```
  > 默认会忽略 static 变量以及被声明为 transient 的字段

## ContentProvider 的原理是什么？为啥工程选用ContentProvider而不是AIDL来IPC
- 利用了 Android 的 Binder 和匿名共享内存机制。
- 通过 Binder 传递 CursorWindow 对象内部的匿名共享内存的文件描述符。这样在跨进程传输中，结果数据并不需要跨进程传输，而是在不同进程中通过传输的匿名共享内存文件描述符来操作同一块匿名内存，这样来实现不同进程访问相同数据的目的

使用上需要注意：
- 对于 `client` 端就是通过 `context.getContentResolver() `来获取到一个 `ContentResolver` 对象，然后调用对象的 `query`， `delete` ，`update` 等方法
- 通过方法中的 `Uri` 参数来匹配的到相应的 `contentprovider` 的
- `ContentProvider` 会先于 `Application` 的 `onCreate` 生成， 它的`onCreate` 是在主线程执行，`query` 等四个函数是在子线程，`call` 函数是自定义的需要自己check
- 没安装 ContentProvider就先安装
> - 当 `provider` 进程不存在时，先创建进程并 `publish` 相关的 `provider`
> - 进程已经存在，但是`provider` 不存在的话，会用 oneway 的binder请求创建并进入 `wait()` 状态，安装完provider信息,则 `notifyAll()` 处于等待状态的进程/线程
- 如果 ContentProvider 是 exported，当支持执行 SQL 语句时就需要注意 SQL 注入的问题。另外如果我们传入的参数是一个文件路径，然后返回文件的内容，这个时候也要校验合法性，不然整个应用的私有数据都有可能被别人拿到


### 为什么使用ContentProvider存在调用进程被杀死的情况？ 
在4.1之前，应用程序访问了ContentProvider，但是这个ContentProvider意外挂了，这个时候应用程序也将被连带杀死；
4.1之后。ContentProvider 会维持一个引用计数：
- `Stable` provider：若使用过程中，provider要是挂了，你的进程也必挂。
- `Unstable` provider：若使用过程中，provider要是挂了，你的进程不会挂。但你会收到一个DeadObjectException的异常，可进行容错处理。

### 插件中没有在manifest中注册的ContentProvider如何跑起来
> 类似于某些Router的做法。hook了startActivity，把ActivityThread的Instrument给反射改变为，一个动态代理了StartActivity的EvilInstrument再装回去。
- 定义一个占坑的ContentProvider（运行在一个独立的进程）
- hook掉插件Activity的Context,并返回自定义的PluginContentResolver
- PluginContentResolver在获取ContentProvider时，先把个占坑的ContentProvider唤醒。即让它在ActivityManagerService中跑起来
- 返回给插件一个IContentProvider的动态代理。
> ——[VirtualApk插件化关于ContentProvider的处理](http://susion.work/2019/02/28/android%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/framework/ContentProvider%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B%E5%88%86%E6%9E%90/)


## App启动流程
1.首先是点击App图标，此时是运行在 `Launcher` 进程,通过 `ActivityManagerServiceBinder` IPC 的形式向 `system_server` 进程发起`startActivity` 的请求
2. `system_server` 进程接收到请求后，通过 `Process#start` 方法向 `zygote` 进程发送创建进程的请求
`zygote` 进程 `fork` 出新的子进程，即 `App` 进程
3. 然后进入 `ActivityThread#main` 方法中，这时运行在App进程中，通过 `ActivityManagerServiceBinder` IPC的形式向 `system_server` 进程发起 `attachApplication` 请求
	> 获得AMS(ActivityManagerService)实例调用 `mgr.attachApplication(mAppThread);`
4. `system_server` 接收到请求后，进行一些列准备工作后，再通过 `Binder` IPC 向App进程binder线程( `ApplicationThread` )发送 `scheduleLaunchActivity` 请求
5. App进程binder线程（ `ApplicationThread` ）收到请求后，通过 `Handler` 向主线程发送 `LAUNCH_ACTIVITY` 消息
6. 主线程收到Message后，通过反射机制创建目标Activity，并回调Activity的`onCreate`

## 屏幕刷新的原理是什么？如何减少卡顿？
View 的 requestLayout 和 ViewRootImpl##setView 最终都会调用 ViewRootImpl 的 requestLayout 方法，然后通过 scheduleTraversals 方法向 Choreographer 提交一个绘制任务，然后再通过 DisplayEventReceiver 向底层请求 vsync 垂直同步信号，当 vsync 信号来的时候，会通过 JNI 回调回来，在通过 Handler 往消息队列 post 一个异步任务，最终是 ViewRootImpl 去执行绘制任务，最后调用 performTraversals 方法，完成绘制。
![屏幕刷新的原理](https://upload-images.jianshu.io/upload_images/24142630-409cdaf1c5111e47?imageMogr2/auto-orient/strip|imageView2/2/w/1089/format/webp)
[应用流畅度(FPS)监控](https://github.com/SusionSuc/AdvancedAndroid/blob/master/performance/rabbit/%E5%BA%94%E7%94%A8%E6%B5%81%E7%95%85%E5%BA%A6(FPS)%E7%9B%91%E6%8E%A7.md)
[面试官又来了：你的app卡顿过吗？](https://juejin.cn/post/6844903949560971277)
### 卡顿原因：
现在的App每秒中最多能绘制60帧，1000ms/60帧=16.67ms/帧，也就是说对图像绘制的要求是平均每帧的绘制时间为16.67ms
- 主线程有其它耗时操作，导致doFrame 没有机会在 vsync 信号发出之后 16 毫秒内调用；
- 当前doFrame方法耗时，绘制太久，下一个 vsync 信号来的时候这一帧还没画完，造成掉帧。
事实上比起每一帧的耗时稍长来说，跳帧（掉帧）才会更易被用户察觉。1s30帧可能没影响，但是这掉的一帧时间太长是会被看出来的


### 监控卡顿
1. 基于主线程 `Looper` 的 Printer 分发消息的时间差值，监控每次 dispatchMessage 的执行耗时。
	> 开源框架 `BlockCanary` 也是给 `Looper` 设置 `Printer` 来监控每个msg的操作时间的。当时耗时超过阈值，会开启一个独立线程查询此段时间内的堆栈、cpu、logcat、内存信息格式化到文件。
	> 发送一个延时runable，如果指定的时间消息分发没有完成，说明应用发生了卡顿，这之后开始执行mRunable，在mRunable进行相关信息采集及提示APP发生卡顿
2. 基于 `Choreographer` 回调函数 `postFrameCallback#doFrame(long)` 监控相邻两次 Vsync 事件通知的时间差。
    > 同时通过另外一条线程循环记录主线程堆栈信息，并在每次 Vsync 事件 doFrame 通知回来时，循环注册该监听对象，间接统计两次 Vsync 事件的时间间隔，当超出阈值时，取出记录的堆栈进行分析上报
    > 开源框架 `rabbit-client` \ `ArgusAPM` \ `LogMonitor` 都是基于此来达成的 
3. 通过反射向Choreographer指定的队列最前端中插入了一个Runnable，在Choreographer.doFrame()时会依次运行这里插入的Runnable时依次调用。
    > `Matrix-TraceCanary` 微信的卡顿检测方案就是如此。[微信APM Matrix解析](https://blog.yorek.xyz/android/3rd-library/matrix/)。
	
	> 采用的ASM插桩的方式，支持fps和堆栈获取的定位，但是需要自己根据asm插桩的方法id来自己分析堆栈，定位精确度高，性能消耗小，比较可惜的是目前没有界面展示，对代码有一定的侵入性。如果线上使用可以考虑。
    
	- `Matrix-TraceCanary` 老版本是用Choreographer，现在是依赖Looper.getMainLooper().setMessageLogging()，理由如下[LoopMonitor内存问题](https://github.com/Tencent/matrix/issues/203)
	> * Choreographer 只是负责ui绘制部分的监听，ui卡顿不一定是绘制卡顿引起的，有可能是主线程耗时操作引起的ui卡顿，比如两次绘制之间有10个msg，Choreographer只能监听到这10个msg造成了卡顿，但是没办法定位到具体是哪个msg导致的卡顿，如果用looper来监听，就可以准确监听到是哪个msg导致的卡顿
	> * 静态页面可能导致帧率较低，因此帧率低可能并不真的是卡顿，所以用这个统计出来的不是很准
	> * 其次是不同设备帧率标准可能不一样，现在很多设备有120hz、90hz、60hz，模拟器有的是30hz


前两种可以较方便的捕捉到卡顿的堆栈，但其最大的不足在于：
1. 无法获取到各个函数的执行耗时。
2. 通过其他线程循环获取主线程的堆栈，如果稍微处理不及时，很容易导致获取的堆栈有所偏移，不够准确。
3. 基于消息队列的卡顿监控并不准确，正在运行的函数有可能并不是真正耗时的函数
4. 某个线程不断地获取主线程堆栈的代价是巨大的，它要暂停主线程的运行
    —— [Matrix Android TraceCanary Wiki](https://github.com/Tencent/matrix/wiki/Matrix-Android-TraceCanary)
### 监控卡顿的第二种方法方案： Rabbit 
在Choreographer.doFrame()回调中通过Handler向HandlerThread发送一个消息。
在HandlerThread中采集主线程的堆栈并保存在map中(避免采集重复的堆栈信息)
以固定的堆栈采集周期向HandlerThread继续发送堆栈采集的消息
当Choreographer.doFrame()的frameCostNs超过了卡顿阈值时，就把在frameCostNs这个时间内采集到的堆栈作为卡顿现场

## 如何计算函数耗时？
1. 在应用启动时，默认打开 Trace 功能（Debug.startMethodTracing），应用内所有函数在执行前后将会经过该函数（`dalvik` 上 `dvmMethodTraceAdd` 函数 或 `art` 上 `Trace::LogMethodTraceEvent` 函数）， 通过hack手段代理该函数，在每个执行方法前后进行打点记录。
   > 最大的好处是能统计到包括系统函数在内的所有函数出入口，对代码或字节码不用做任何修改，所以对apk包的大小没有影响，但由于方式比较hack，在兼容性和安全性上存在一定的风险。 

2. 修改字节码的方式，在编译期修改所有 class 文件中的函数字节码，对所有函数前后进行打点插桩。
   > 利用 Java 字节码修改工具（如 BCEL、ASM、Javassis等），在编译期间收集所有生成的 class 文件，扫描文件内的方法指令进行统一的打点插桩，同样也可以高效的记录函数执行过程中的信息
   > 无法统计系统内执行的函数，但往往造成卡顿的函数并不是系统内执行的函数
   > 无需 hook 任何函数，所以在兼容性方面会比第一个方案更可靠。

## apk资源加是如何加载的？热修复的资源修复的原理是什么？
[android 应用的启动过程分析](https://www.jianshu.com/p/a1f40b39b3de)
App启动流程中 `attachApplication()` 方法，最终通过binder，跨进程调用到 `ApplicationThread#bindApplication()` 然后调用`ActivityThread#handleBindApplication` 方法，我们从这个方法开始看

Apk的资源是通过AssetManager.addAssetPath方法来完成加载，那么我们就可以通过反射构建自己的AssetManager对象，然后把调用addAssetPath加载自己的资源，然后把自己构建的AssetManager通过反射设置给mAssets变量，这样下次加载资源就是用的我们AssetManager，也就是用的我们更换后的资源

### AndResGuard如何达到资源优化的？
[从R文件索引看资源优化](https://linjiang.tech/2020/01/20/r-resources-arsc/)
[浅谈Android中的R文件作用以及将R资源inline减少包大小](https://yuweiguocn.github.io/android-r-inline/)
[关于-r-的一切](https://medium.com/@morefreefg/%E5%85%B3%E4%BA%8E-r-%E7%9A%84%E4%B8%80%E5%88%87-355f5049bc2c
[每日一问 这么多R.java 有卵用呀？](https://wanandroid.com/wenda/show/9974)
- Android在构建过程中会根据资源生成R文件，里面包含了资源索引，使用该索引可以在最终生成的`resources.arsc`资源映射表中找到对应资源
- 在Library工程中引用的R资源索引不是final的。同时一个资源在两个module中重复定义了，R.txt文件中会存在多份
- java编译器在编译时会将final常量进行inline内联操作，也就是App工程中的java源码编译后的class中使用的R资源索引全部会替换为常量值
- 最后使用的都是`resources.arsc`，所以R文件可以精简
  1. 可以在编译完成之后将 module 里面对于 R 的引用换成「内联」的，这样就可以少了一次内存寻址，也可以删掉被内联后的 R.class，减少了包体积又做了性能优化。
  2. 可以在编译的时候通过固定 id 来减少增删改资源带来的大量 id 变动，导致 R.java 被“连根拔起”，带来下游依赖它的 java/kotlin 文件重新编译。 
  > stable-ids 在「热修复」、「插件化」中有很大的用处

## 说说组件化和插件化、热更新（热补丁）、热修复技术原理
热修复与插件化都利用classloader实现加载新功能。热修复比插件化复杂，插件化只是增加新的功能或资源文件，所以不涉及抢先加载旧类的使命。热修复为了修复bug，要将新的同名类替旧的同名bug类，要抢在加载bug类之前加载新的类。

## Blockcanary有什么缺点
原理是替换logger的实现，塞了一个自己的，但你能塞，别人也能塞，别人在你blockcanary初始化后面塞进来就导致blockcanary失效了，同时对于dispatch中的耗时操作无法监控到，因为这个没有走handler是native调用的

## 组件化与插件化
以下来自[Android 浅谈模块化、组件化、插件化、热修复的简单理解](https://blog.csdn.net/csdn_aiyang/article/details/103735995?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-3&spm=1001.2101.3001.4242)
组件化：
- 分成不同library，降低耦合度。
- 解决公共依赖关系，同一层组件不能互相引用，也不能基础引用业务的这种反向引用。
- 需要处理Activity的跳转问题。有路由和反射两种方式。
- 资源命名问题。需要让各个组件中使用统一前缀
插件化：
> 实现原理
> 1. 通过dexclassloader加载。
> 2. 代理模式添加生命周期。
> 3. hook思想跳过清单验证。
- 宿主和插件分开编译
- 动态更新插件
- 按需下载插件
- 缓解65535方法数限制
热修复与插件化都利用classloader实现加载新功能，热修复比插件化复杂。
插件化只是增加新的功能或资源文件，所以不涉及抢先加载旧类的使命。热修复为了修复bug，要将新的同名类替旧的同名bug类，要抢在加载bug类之前加载新的类。
## 热修复
MultiDex将本地加载的dex文件插入到DexPathList中的dexElements中，后续程序运行时就会自动到该变量中去查找新的类，这就是该篇讲的热修复的原理。

## 混淆proguard
proguard主要的目的是混淆代码，保护应用源代码。次要的功能还有移除无用类等，优化字节码，缩小包体积。

压缩(Shrink)：检测并移除代码中无用的类、字段、方法和特性(Attribute)
优化(Optimize)：字节码进行优化，移除无用的指令。
混淆(Obfuscate)：使用a、b、c、d这样简短而无意义的名称，对垒、字段和方法进行重命名。
预检测(Preveirfy)：在Java平台对处理后的代码进行预检测，确保加载class文件是可执行的。


## 热修复资源替换
资源注入：资源的动态加载则相对简单。主要是参考Instant Run，通过反射调用AssetsManager的addAssets方法，将增量资源包加载到内存中来，得到新的Resources对象，然后替换掉ActivityThread等所有持有Resources的地方即可。这也是大部分热修复框架中的基本思路。

## 如何识别重复资源？
使用AndResGuard可以更加缩减包体积。
识别重复资源很简单，只要计算一下md5就行了。
并且我们在resources.arsc中可以拿到所有的资源，那么我们就可以对resources.arsc中的所有资源进行处理，根据md5进行去重，把使用了相同资源的资源id都指向同一个资源，把多余的资源删除掉，再回写入resources.arsc就好了

## 65536的问题
一个DEX文件中的method个数采用使用原生类型short来索引文件的方法，也就是2个字节共计最多表达65536个method。所以当method数过多的时候，就必须使用multidex

### Dex分包：MultiDex
[Android Dex分包之旅](http://yydcdut.com/2016/03/20/split-dex/index.html)含有 `65536` \  `LinearAlloc` 等问题的解决方式
[Multidex分包Class依赖处理原理](https://yangxiaobinhaoshuai.github.io/2018/04/22/Multidex%E5%88%86%E5%8C%85Class%E4%BE%9D%E8%B5%96%E5%A4%84%E7%90%86%E5%8E%9F%E7%90%86-md/)`manifest_keep.txt`脚本源码
DEX分包是为了解决65536方法限制，系统在应用打包APK阶段，会将有调用关系的类打包在同一个Dex文件中，并且同一个dex中的类会被打上`CLASS_ISPREVERIFIED`的标志。因为加载后的类不能卸载，必须通过重启后虚拟机进行加载才能实现修复，所以此方案不支持即时生效。

### QQ空间超级补丁
是把BUG方法修复以后放到一个patch.dex，拿到当前应用BaseDexClassloader后，通过反射获取到DexPathList属性对象pathList、再反射调用pathList的dexElements方法把patch.dex转化为`Element[]`，两个`Element[]`进行合并，最后把patch.dex插入到dexElements数组的最前面，让虚拟机去加载修复完后的方法，就可以达到修复目的。

### 微信Tinker
采用的是DEX差量包，整体替换DEX的方案。主要的原理是与QQ空间超级补丁技术基本相同，但不将patch.dex增加到elements数组中。差量的方式拿到patch.dex，开启新进程的服务TinkerPatchService，将patch.dex与应用中的classes.dex合并，得到一个新的fix_classess.dex。通过反射操作得到PathClassLoader的DexPatchList，再反射调用patchlist的makeDexElements()方法，把fix_classess.dex直接替换到Element[]数组中去，达到修复的目的。从而提高了兼容性和稳定性。
### CLASS_ISPREVERIFIED问题
如果引用者和被引用者的类(直接引用关系)在同一个Dex时,那么在虚拟机启动时，被引用类就会被打上CLASS_ISPREVERIFIED标志。这样被引用的类就不能进行热修复操作了。怎么办？

原因是：那么我们就要阻止被引用类打上CLASS_ISPREVERIFIED标志，两个有调用关系的类不在同一个Dex文件中，那么就会抛“unexpected DEX problem”异常报错。解决办法，就是单独放一个AnitLazyLoad类在另外DEX中，在每一个类的构造方法中引用其他DEX中的唯一AnitLazyLoad类，避免类被打上CLASS_ISPREVERIFIED标志。
### MultiDex如何将指定class打包到主Dex中
配置一个文件，仅该文件下的内容会被打入主Dex中
### 模块化、组件化、插件化通讯方式不同之处
- 模块化相互引入，抽取了公共的common模块，其他模块自然要引入这个module。
- 组件化主流是隐式和路由。隐式使解耦和灵活大大降低，因此路由是主流。
- 插件化本身是不同进程
### 如何加快MultiDex的速度
#### PreMultiDex方案
- 安装一个新的apk的时候，先在Worker线程里做好MultiDex的解压和Optimize工作，安装apk并启动后，直接使用之前Optimize产生的odex文件，这样就可以避免第一次启动时候的Optimize工作。
- 缺点：第一次安装的apk没有作用，而且事先需要使用内置的apk更新功能把新版本的apk文件下载下来后，才能做PreMultiDex工作。
> 安装dex的时候，核心是创建DexFile对象并使用其Native方法对dex文件进行opt处理，同时生产一个与dex文件(.zip)同名的已经opt过的dex文件(.dex)。
> - 如果安装dex的时候，这个opt过的dex文件已经存在，则跳过这个过程，这会节省许多耗时。
> - 所以下载Apk完成的时候，预先解压dex文件，并预先触发安装dex文件以生产opt过的dex文件。
#### 异步MultiDex方案
启动App的时候，先显示一个简单的Splash闪屏界面，然后启动Worker线程执行MultiDex#install(Context)工作，就可以避免UI线程阻塞。不过要确保启动以及启动MultiDex#install(Context)所需要的类都在主dex里面(手动分包)，而且需要处理好进程同步问题。

## art和darvlik区别
[JAVA虚拟机与Android虚拟机的区别](https://www.jianshu.com/p/8edac8e09b3e)内有PC指针变换过程

1. Delvik环境下，每次运行都需要通过及时编译器（JIT）将字节码转为机器码。
> - 安装过程比较快，但程序启动每次都会重复的JIT（Just-In-Time）过程
> - JIT会在运行时分析应用程序的代码，识别哪些方法可以归类为热方法，这些方法会被JIT编译器编译成对应的汇编代码，然后存储到代码缓存中，以后调用这些方法时就不用解释执行了，可以直接使用代码缓存中已编译好的汇编代码。这能显著提升应用程序的执行效率

`.odex`是对dex的优化，deodex在系统第一次开机时会提取所有apk内的dex文件，odex优化将dex提前提取出，加快了开机的速度和程序运行的速度
> `.dex`字节码，是针对Android设备优化后的DVM所使用的运行时编译字节码

2. ART（Android Runtime）通过预编译（`AOT, Ahead-Of-Time`）在第一次安装时将字节码转为机器码。生成`OAT`文件，仍以.odex保存，但是与Dalvik下不同，这个文件是可执行文件。
> dex、odex 均可通过dex2oat生成oat文件，以实现兼容性
> 以空间换时间，可能会增加10%-20%的存储空间和更长时间的应用安装
> 在APK安装的时候就会做预先编译动作，编译好的文件是OAT文件
> `OAT`文件文件本质上是一个`ELF`文件，这里与`dex`(`Odex`)文件最大的区别是OAT文件不再是字节码文件，而是一个可执行文件，可以更底层的与硬件接触
，运行时也省去了预编译和转译的时间

### JVM 和 Darvlik VM 的区别
1. JVM基于栈，DVM基于寄存器
2. JVM一个类一个class文件，每个class冗余变量多，IO查找类时慢。Dalvik去除冗余并整合class，生成的是整个工程的一个dex文件。（如果分包的话，会有多个dex文件）
> - class文件中包含多个不同的方法签名，如果A类文件引用B类文件中的方法，方法签名也会被复制到A类文件中（在虚拟机加载类的连接阶段将会使用该签名链接到B类的对应方法），多个不同的类会同时包含相同的方法签名

> - dx工具对JAVA类文件重新排列，将所有JAVA类文件中的常量池分解，消除其中的冗余信息，重新组合形成一个常量池，所有的类文件共享同一个常量池

3. JAVA虚拟机运行的是JAVA字节码，Dalvik虚拟机运行的是Dalvik字节码


## 点击屏幕后触发的流程讲一下，从硬件开始说。

## view的绘制流程，为什么会多次调用onMeasure
- onResume(Activity)
- onPostResume(Activity)
- onAttachedToWindow(View)
- onMeasure(View)
- onMeasure(View)
- onLayout(View)
- onSizeChanged(View)
- onMeasure(View)
- onLayout(View)
- onDraw(View)

## RecyclerView缓存有了解吗 ⭐
在RecyclerView中，并不是每次绘制表项，都会重新创建ViewHolder对象，也不是每次都会重新绑定ViewHolder数据。
- RecyclerView 通过Recycler获得下一个待绘制表项。Recycler有4个层次用于缓存ViewHolder对象，优先级从高到底依次为
  1. ArrayList<ViewHolder> mAttachedScrap
    > - 存放的是dettach掉的视图
    > - 从mAttachedScrap 中复用的ViewHolder不需要重新创建也不需要重新绑定数据
	> 用于布局过程中屏幕可见表项的回收和复用。
	> - 先 detach 并 缓存表项到 scrap 结构中，然后紧接着又在填充表项时从中取出。因为 RecyclerView 要做表项动画，为了确定动画的种类和起终点，需要比对动画前和动画后，就得布局两次，分别是预布局和后布局（布局即是往列表中填充表项），

  2. ArrayList<ViewHolder>  mChangedScrap
    > - 存放的是dettach掉的视图
    > - 和上边的mAttachedScrap是一样的，唯一不同的从名字也可以看出来，它存放的是发生了变化的ViewHolder，要重新走Adapter的绑定方法的
  3. ArrayList<ViewHolder> mCachedViews
    > - 复用的ViewHolder，只能复用于指定位置(position)的表项。
	> - 只有“列表回滚”这一种场景（刚滚出屏幕的表项再次进入屏幕），才有可能命中该缓存。该缓存存放在默认大小为 2 的ArrayList中
  4. ViewCacheExtension mViewCacheExtension
    > 自定义缓存
  5. RecycledViewPool mRecyclerPool
     > - 复用的ViewHolder需要重新绑定数据。
	 > - 对ViewHolder按viewType分类存储（通过SparseArray），每个viewType相同的包含默认大小为5的ArrayList来村ViewHolder。
  如果四层缓存都未命中，则重新创建并绑定ViewHolder对象。
     > - 总是先回收到mCachedViews，当它放不下的时候，按照先进先出原则将最先进入的ViewHolder存入回收池
  6.  ArrayList<ViewHolder> mHiddenViews
     > - 是个缓存被隐藏的ViewHolder的ArrayList

## 如何判断当前线程是主线程？为啥looper可以拿到线程，ThreadLocal是用来干嘛的？ThreadLocal原理是什么？
1. `Looper.getMainLooper().getThread()`得到主线程， 与当前线程做比较
2. `Looper`和一个线程通过`ThreadLocal`一一绑定 
3. `ThreadLocal`为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。例如数据库连接，Session会话管理可以重用线程的`ThreadLocal`   
4. 每个线程(`Thread`)会持有一个`ThreadLocalMap`, 线程(`Thread`) 和 `ThreadLocalMap` 是 1对1，`ThreadLocalMap`的key是 `ThreadLocal`本身，为了防止内存泄漏，所以持有的是 `threadLocal` 的弱引用
   > 内存泄漏：`ThreadLocal#set( Entry(key, value) ) `，而`Entry extends WeakReference<ThreadLocal<?>>`的，所以当`ThreadLocal=null`时，GC会把`ThreadLocal`回收，但是`Thread`不死，`ThreadLocalMap`就会一直存在 ，`GC`把`ThreadLocal`回收后，`ThreadLocalMap`还存在一条无用的信息(null，value还在)，这样就造成了内存泄漏，所以在`ThreadLocal`使用完成后，请调用`remove`方法
5. ThreadLocal本身并不存储值，它只是作为一个key来让线程从ThreadLocalMap获取value
6. 一个线程可以有多个TreadLocal来存放不同类型的对象的，但是他们都将放到你当前线程的ThreadLocalMap里

### ThreadLocal是干嘛的，作用是什么 
<!--more-->
- 每个线程Thread维护了ThreadLocalMap这么一个Map变量，保证线程隔离且有唯一的ThreadLocal
- 创建ThreadLocalMap的key是ThreadLocal对象本身（保证同一个线程总是取到同一个ThreadLocal），value则是要存储的对象
- 会造成内存泄漏

## 点击一个按钮后一直按住，同时挪开屏幕
[事件分发三连问：事件是如何从屏幕点击最终到达 Activity 的？CANCEL 事件什么时候会触发？如何解决滑动冲突？](https://blog.csdn.net/Androiddddd/article/details/108804170)

- 只要clickable和longClickable有一个为真，那么onTouchEvent就返回true。哪怕一个View是disable状态的 
  > ——[Android事件分发机制面试题](https://www.cnblogs.com/androidxufeng/p/12795620.html)
```JAVA
// 经典的责任链模式
bool dispatchTouchEvent(MotionEvent ev){
	bool consume = false;
	if(onInterceptTouchEvent(ev)){
		consume = onTouchEvent(ev);
	} else {
		consume = child.dispatchTouchEvent(ev);
	}
}
```

## ActivityThread是UI线程吗，它并没有继承自Thread？ActivityThread 中的 H 用来干嘛的
> 当Zygote启动时，会分裂出system_server并进行不断地ipc轮询,system_server会创建AMS等服务。当你在桌面点击一个app图标时，并且这个app在内存中是无实例的。ams会通知system_server,由system_server通知Zygote去fork出子进程并执行ActivityThread的main方法。main方法的调用是在子进程的主线程中。
ActivityThread就是主线程
内部类 H 继承 Handler 用来接受Application和Activity的各种生命事件


## Zygote Fork进程为什么是用socket，为什么不用binder？
- fork只能拷贝当前线程，不支持多线程的fork
- 不建议在多线程环境使用Fork，而Binder是多线程的。
怕父进程binder线程有锁，然后子进程的主线程一直在等其子线程(从父进程拷贝过来的子进程)的资源，但是其实父进程的子进程并没有被拷贝过来，造成死锁，所以fork不允许存在多线程。而非常巧的是Binder通讯偏偏就是多线程，所以干脆父进程（Zgote）这个时候就不使用binder线程
> 互斥锁是导致无法在多线程环境使用Fork的根本原因，大部分系统中，锁的实现基本是用户态，并非内核态（用户态更加高效）。假设在fork之前，一个线程对某个锁进行lock操作，另一个线程进行fork子进程，会导致持有锁的子线程消失了，那这个锁就会永久锁定，假如另一个线程获取该锁，就会导致死锁了。

## 如何传输一张大的Bitmap？ashmem匿名内存 
用户通过mmap得到的offset地址当
被用户首次访问时会因为缺页而导致异常函数shmem_fault被执行,这样用户访问的offset对应的物理内存页将被动态申请,
ashmem的表现就像Heap堆一样,所以使用ashmem分配的内存,不像malloc,calloc之类,物理内存立即分配,
ashmem只有当用户访问offset地址对应的物理页时,相应的内存物理页才被缺页异常处理,进而为offset虚拟地址申请和映射实体
物理内存页,带来的直接好处就是,应用程序可以申请很大的内存,而应用程序最终实际占用的物理内存数量,会根据

## SharedPreferences 缺点？用mmkv代替其优点是什么
[SharedPreferences 是线程安全的吗？它的 commit 和 apply 方法有什么区别？](https://github.com/moosphan/android-daily-interview/issues/15)
1. IO瓶颈。读写都要从内存写入到文件中。xml文件形式存储在本地
  >	1、Editor的commit方法，每次执行时同步写入磁盘。
  >	2、Editor的apply方法，每次执行时在单线程池中加入写入磁盘Task，异步写入。
  > apply没有返回值。使用apply方法可以极大的提高性能。多个写入操作可以合并为一个commit/apply，将多个写入操作合并后也能提高IO性能。
2. 读写操作的锁均是针对SP实例对象的。将数据拆分到不同的sp文件中，降低单文件访问频率，多文件均摊访问，以减少锁耗时。
  > 在get操作时，会锁定SharedPreferences对象，互斥其他操作，而当put，commit时，则会锁定Editor对象，使用写入锁进行互斥
3. sp并不支持跨进程，因为它不能保证更新本地数据后被另一个进程所知道。

> - SharedPreferences是线程安全的，它的内部实现使用了大量synchronized关键字；SharedPreferences不是进程安全的
> 第一次调用getSharedPreferences会加载磁盘 xml 文件（这个加载过程是异步的，通过new Thread来执行，所以并不会在构造SharedPreferences的时候阻塞线程，但是会阻塞getXxx/putXxx/remove/clear等调用）
> 如果第一次调用getSharedPreferences时还没从磁盘加载完毕就马上调用getXxx/putXxx，那么getXxx/putXxx操作会阻塞，直到从磁盘加载数据完成后才返回
> 所有的getXxx都是从内存中取的数据，数据来源于SharedPreferences.mMap
> apply同步回写（commitToMemory()）内存SharedPreferences.mMap，然后把异步回写磁盘的任务放到一个单线程的线程池队列中等待调度。apply不需要等待写入磁盘完成，而是马上返回
> commit同步回写（commitToMemory()）内存SharedPreferences.mMap，然后如果mDiskWritesInFlight（此时需要将数据写入磁盘，但还未处理或未处理完成的次数）的值等于1，那么直接在调用commit的线程执行回写磁盘的操作，否则把异步回写磁盘的任务放到一个单线程的线程池队列中等待调度。commit会阻塞调用线程，知道写入磁盘完成才返回
MODE_MULTI_PROCESS是在每次getSharedPreferences时检查磁盘上配置文件上次修改时间和文件大小，一旦所有修改则会重新从磁盘加载文件，所以并不能保证多进程数据的实时同步

### mmkv优点
1. 通过 `mmap` 内存映射文件，提供一段可供随时写入的内存块。由操作系统负责将内存回写到文件（缺页中断回调），不必担心 crash 导致数据丢失
2. 多进程访问，
3. `Ashmem` 匿名共享内存，发现它在进程退出后就会消失，不会落地到文件上
4. 有数据加密和CRC校验
2. 使用protobuf压缩
3. 增量写入。直接 append 到内存末尾，最新的数据在最后。
4. 文件重整。以内存 pagesize 为单位申请空间，在空间用尽之前都是 append 模式；当 append 到文件末尾时，进行文件重整、key 排重

### mmkv多进程如何锁的
1. 通过校验文件,在读取数据时,来做校验,就实现了多个进程的数据同步
2. 使用flock文件锁。但是文件锁不支持递归，所以需要自己维护计数和锁升级降级[MMKV for Android 多进程设计与实现](https://github.com/Tencent/MMKV/wiki/android_ipc)

## 如何进程传输一张大图或者复用大图？Ashmem匿名共享内存，MemoryFile用过吗？SharedMemory呢？
1. Java层Android也提供了一个名为MemoryFile的类提供方便使用匿名共享内存
2. 实际上匿名共享内存创建出来也是一个文件，不过因为是在tmpfs临时文件系统才叫做匿名的
3. 共享了一个文件描述符。对映射的区域进行读写，换句话说就是对共享内存这段地址区域直接进行读写，没有经过write，read的系统调用
4. Ashmem这个区域的内存并不属于Java Heap,也不属于Native Heap。可以自动释放物理内存 `pin` & `unpin`
> Fresco5.0以下使用Ashmem匿名共享内存来复用Bitmap
Ashmem匿名共享内存使用的步骤可以分为4步：[Android 重学系列 Ashmem匿名共享内存](https://www.jianshu.com/p/6a8513fdb792)
1. open /dev/ashmem驱动连通ashmem驱动。
2. ioctl 发送ASHMEM_SET_NAME命令为该ashmem创建名字。
3. ioctl 发送ASHMEM_SET_SIZE命令为ashmem设置大小
4. mmap 做内存映射。
5. 对该文件描述符进行读写即可。 
628012

##  多进程的好处
1. Android系统对每个应用进程的内存占用是有限制的，而且占用内存越大的进程，通常被系统杀死的可能性越大。让一个组件运行在单独的进程中，可以减少主进程所占用的内存，降低被系统杀死的概率.
2）如果子进程因为某种原因崩溃了，不会直接导致主程序的崩溃，可以降低我们程序的崩溃率。
3）即使主进程退出了，我们的子进程仍然可以继续工作，假设子进程是推送服务，在主进程退出的情况下，仍然能够保证用户可以收到推送消息
 

## 说说组件化和插件化， 技术原理
APT在编译的开始阶段对java文件进行操作，而像AscpectJ、ASM等则是在java文件编译为字节码文件后

### 能否在加载类的时候，对字节码进行修改?
利用javaAgent和ASM字节码都可以，javaAssist还有Byte-Code都可以
![字节码增强](https://p0.meituan.net/travelcube/12e1964581f38f04488dfc6d2f84f003110966.png)
1. [Java字节码技术(二)字节码增强之ASM、JavaAssist、Agent、Instrumentation](https://blog.csdn.net/hosaos/article/details/102931887)
2. [美团 - 字节码增强技术探索](https://tech.meituan.com/2019/09/05/java-bytecode-enhancement.html)包含有ASM的基本用法和工具
    > **可否在运行时对JVM中的类进行修改并重载？**：通过Instrument充当Agent + Attach API， 依赖JVMTI的Attach API机制实现

## 方法数越界为啥？MultiDex是怎么做的？
1. 方法数超过65536数会报错。
> 早期系统中，dexopt会把每一个类的方法id检索起来，存在一个链表结构里，而这个链表的长度是用一个short类型来保存的，2字节 = 2 ^ 16 = 2 ^ 6 * 2 ^ 10= 64 * 1024 = 64K = 65536。新版本dexopt修复了这个问题
2. 方法数并没有超过65536，编译也完成了，但是在android2.3以前的系统安装的时候，会异常中止安装
> `dexopt`的执行过程是在第一次加载dex文件的时候执行的。这个过程产生了一个`ODEX`文件，全称`Optimised Dex`。这个`ODEX`文件是在安装过程中从apk里提取出的可运行文件，是优化dex产生的，再把apk包里的dex文件删除，这样就做到了预先提取。如果没有ODEX文件，那么系统会从apk包中提取dex然后再运行。所以优化后可以加快软件的启动速度，预先提取，减少对RAM的占用。
`dexopt`采用一个固定大小的缓冲区（`LinearAlloc`）来存储应用中所有方法的信息，那么之所以会出现在老版本停止安装，是因为老版本的缓冲区的大小是5M

### MultiDex解决方案
- 将APK文件中除主dex文件之外的dex文件追加到PathClassLoader（也就是BaseClassLoader）中DexPathListde Element[]数组中。这样在加载一个类的时候就会遍历所有的dex文件，保证了打包的类都能够正常加载。
```Java
/** 
打包时，把一个应用分成多个dex，例：classes.dex、classes2.dex、classes3.dex…，加载的时候把这些dex都追加到DexPathList对应的数组中，这样就解决了方法数的限制。
Andorid 5.0之后，ART虚拟机天然支持MultiDex。
Andorid 5.0之前，系统只加载一个主dex，其它的dex采用MultiDex手段来加载。
*/
``` 
> 使用反射修改DexPathList。
> 1. 通过一定的方式把dex文件抽取出来；
> 2. 把这些dex文件追加到DexPathList的Element[]数组的后面；
> 3. 这个过程要尽可能的早，所以一般是在Application的attachBaseContext()方法中。
> 由于需要IO加载额外的dex文件，应用的启动速度会降低，当其他dex文件较大的时候，甚至会出现ANR现象

### 如何用代码自行生成接口？
之前是apt，如今是 annotationProcessor

### ASM应用场景
[Gradle+Transform+Asm自动化注入代码](https://www.jianshu.com/p/fffb81688dc5)
[字节码插桩--你也可以轻松掌握](https://www.jianshu.com/p/13d18c631549)
[【Android】函数插桩（Gradle + ASM）](https://www.jianshu.com/p/16ed4d233fd1)
1. 无埋点统计、APM、插桩
其实就是在上面的基础进行各种位置的插桩,具体例子[Android无埋点数据收集SDK关键技术](http://www.jianshu.com/p/b5ffe845fe2d)

2. 瘦包

蘑菇街的[ThinRPlugin](http://www.jianshu.com/p/b5ffe845fe2d)插件
相关原理：android中的R文件，除了styleable类型外，所有字段都是int型变量/常量，且在运行期间都不会改变。所以可以在编译时，记录R中所有字段名称及对应值，然后利用asm工具遍历所有class，将引用R字段的地方替换成对应常量，然后将R$styleable.class以外的所有R.class删除掉
BTW:类似瘦包的思路：Facebook redex(不是使用asm)
## 知识图谱
1. [Android初级、中级、高级、资深工程师(架构师、专家)技能图谱](https://www.jianshu.com/p/659381fcd4e5)

2. [面试官: 说一下你做过哪些性能优化?](https://juejin.im/post/6844904105438134286)
3. [面试之Android性能优化](https://www.zybuluo.com/TryLoveCatch/note/1302255)

## Retrofit(动态代理)
> 动态代理的代理关系是在运行期确定的，Jvm帮忙生成class文件并且会删除class文件
JDK原生动态代理是Java原生支持的，不需要任何外部依赖，但是它只能基于接口进行代理；
CGLIB通过继承的方式进行代理，无论目标对象有没有实现接口都可以代理，但是无法处理final的情况


## Binder怎么学

1. [为什么Binder的通信只进行了一次拷贝](https://mubu.com/doc/explore/21079) ：
	1. 基于 `mmap` 。`Client`与 `Server` 处于不同进程有着不同的虚拟地址规则，所以无法直接通信。一个页框可以映射给多个页，那么就可以将一块物理内存分别与 Client 和 Server 的虚拟内存块进行映射。映射的虚拟内存块大小将近 1M (1M-8K = 1016K)，所以 IPC 通信传输的数据量也被限制为此值
		> —— 摘自[谈谈你对 binder 的理解？](https://zhuanlan.zhihu.com/p/143324649)
	2. 通过 `copy_from_user` 将数据从用户空间拷贝到内核空间，通过 `copy_to_user` 将数据从内核空间拷贝到用户空间。一般的 IPC 方式需要分别调用这两个函数，数据就拷贝了两次，而 binder 将内核空间与目标用户空间进行了 `mmap`，只需调 `copy_from_user` 拷贝一次即可。
		
2. [聊聊怎样学习Binder](https://cloud.tencent.com/developer/article/1698142)内有c++代码简单传输的过程

### binder每次传输大小有限制吗？
- 限制了大小为1M-2页(1页=4k) = 1024K - 8 K = 1016k
- 因此要传输图像数据这个大小根本不够。
- 加上Binder内部有对每一个Binder内核缓冲区有自己的调度算法，没办法满足以最快的速度传输到SF进程中。
> 所以Android选择使用共享内存的方式传递数据，也就是Ashmem匿名内存。 
```C
#define BINDER_VM_SIZE ((1 * 1024 * 1024) - sysconf(_SC_PAGE_SIZE) * 2) 
ProcessState::ProcessState(const char *driver)
    : mDriverName(String8(driver))
    , mDriverFD(open_driver(driver))
    , mVMStart(MAP_FAILED)
    , mThreadCountLock(PTHREAD_MUTEX_INITIALIZER)
    , mThreadCountDecrement(PTHREAD_COND_INITIALIZER)
    , mExecutingThreadsCount(0)
    , mMaxThreads(DEFAULT_MAX_BINDER_THREADS)
    , mStarvationStartTimeMs(0)
    , mManagesContexts(false)
    , mBinderContextCheckFunc(NULL)
    , mBinderContextUserData(NULL)
    , mThreadPoolStarted(false)
    , mThreadPoolSeq(1)
{
    if (mDriverFD >= 0) {
        // mmap the binder, providing a chunk of virtual address space to receive transactions.
        mVMStart = mmap(0, BINDER_VM_SIZE, PROT_READ, MAP_PRIVATE | MAP_NORESERVE, mDriverFD, 0);
...
    }
}
```

### 为什么 binder 驱动要运行在内核空间？可以移到用户空间吗？
> [说说你对 binder 驱动的了解？](https://zhuanlan.zhihu.com/p/152237289)

binder 驱动运行在内核空间，向上层提供 /dev/binder 设备节点及 open、mmap、ioctl 等系统调用。
| 结构体             | 说明                                                               |
| ------------------ | ------------------------------------------------------------------ |
| binder_proc        | 描述使用 binder 的进程，当调用 binder_open 函数时会创建            |
| binder_thread      | 描述使用 binder 的线程，当调用 binder_ioctl 函数时会创建           |
| binder_node        | 描述 binder 实体节点，对应于一个 server ，即用户态的 BpBinder 对象 |
| binder_ref         | 描述对 binder 实体节点的引用，关联到一个 binder_node               |
| binder_buffer      | 描述 binder 通信过程中存储数据的Buffer                             |
| binder_work        | 描述一个 binder 任务                                               |
| binder_transaction | 描述一次 binder 任务相关的数据信息                                 |
| binder_ref_death   | 描述 binder_node 即 binder server 的死亡信息                       |
——————————————————
### oneway 是什么？如何异步等待？
> [看你简历上写熟悉 AIDL，说一说 oneway 吧](https://zhuanlan.zhihu.com/p/143082762)

oneway 修饰的 AIDL 接口方法，是单向调用，不需要等待另一个进程的返回结果，所以方法的返回类型也只允许是 void。
即应用进程只向 binder 驱动发送一次数据就结束返回，不再等待回复数据。binder 驱动会将这个请求串行排列在队列里一个个执行。

1. 同步调用
![同步调用](https://pic3.zhimg.com/80/v2-7b2c17503a7abea3e2cb6befaa7f603a_1440w.jpg)

2. oneway的发送
![oneway的发送](https://pic2.zhimg.com/80/v2-899eafe99d5c70e5cffe9edde3a9cc7d_1440w.jpg)
	> 由外部发送给 binder 驱动的都是 BC_ 开头，由 binder 驱动发往外部的都是 BR_开头。

###  Intent 传值为啥会有大小限制？那跨进程传递大图怎么办？
应用进程在启动 `Binder` 机制时会映射一块 1M 大小的内存，一个进程内所有正在进行的 `Binder` 事务共享这 1M 的缓冲区 。当使用 Intent 进行 IPC 时申请的缓存超过 1M ，其他事务占用的内存时，就会申请失败抛 `TransactionTooLargeException` 异常了。
1. 解决方法：通过 `putBinder` 的方式传 `Bitmap` 就不会抛异常.
2. 原因： `Intent` 启动组件时，系统禁掉了文件描述符 fd 机制 (`mAllowFds = false`), bitmap 无法利用共享内存，只能拷贝到 `Binder` 映射的缓冲区，导致缓冲区超限, 触发异常
3. 通过 putBinder 的方式，避免了 Intent 禁用描述符的影响，bitmap 写 parcel 时的 allowFds 默认是 true , 可以利用共享内存，所以能高效传输图片。
```JAVA
Bundle b = new Bundle();
b.putBinder(“binder”, new IRemoteCaller.Stub() {    
	@Override    
	public Bitmap getBitmap() {           
		return mBitmap;    
	}
});

// or

b.putBinder("bitmap", new ImageBinder(mBitmap));
```
底部是`ashmem`传递的。
> bundle 在IPC传递过程中是值的深拷贝，而在Fragment的setArgument之类的地方是引用传递
> Bitmap，本身就已经实现了 Parcelable 是可以支持序列化

### Intent传输限制怎么办？
1. 减少传输数据量
2. 非 ipc 通过内存共享，使用静态变量或者使用EventBus等类似的通信工具
3. 是 ipc 可以用 file 或者 socket 文件共享（比如通过MemoryFile开辟一片共享内存，然后传递FileDescriptor，接收端用这个fd读）
4. 匿名内存：flying-pigeon 是一个IPC 跨进程通信组件，底层是匿名内存+Binder ， 突破1MB大小限制，无需写AIDL文件，让实现跨进程通信就像写一个接口一样简单
5. 持久化
### 理解AIDL中的in，out，inout吗

1. 一文看得懂的源码：[聊聊怎样学习Binder](https://cloud.tencent.com/developer/article/1698142)

2. 一文超翔实的源码：[Android Bander设计与实现 - 设计篇](https://blog.csdn.net/universus/article/details/6211589)

## 崩溃率日均多少？发生崩溃crash怎么办？

## 如何获得crash时的堆栈
崩溃主要分为java层crash，c/c++层crash，anr导致的crash。

1. java层crash：可以通过手机`logcat`日志分析，也可以继承`UncaughtExceptionHandler`，并通过`Thread.setDefaultUncaughtExceptionHandler`来处理系统的异常，基本能收集到所有java层的crash的问题。
2. c/c++层的crash：通过logcat日志进行分析，也可以通过`Breakpad`的来收集dump文件。可能收集不了任何堆栈，这要通过别的手段进行分析。
3. anr的crash：通过logcat日志分析，anr的日志也可以通过/data/anr/traces.txt进行分析（高版本会没有权限读取这个文件）

### 获得logcat和Jave堆栈的方法
1. 获取logcat
logcat日志流程是这样的，应用层 --> liblog.so --> logd，底层使用ring buffer来存储数据。
获取的方式有以下三种：
   1. 通过logcat命令获取。
      > 优点：非常简单，兼容性好。

      > 缺点：整个链路比较长，可控性差，失败率高，特别是堆破坏或者堆内存不足时，基本会失败。
   2. hook liblog.so实现。通过hook liblog.so 中__android_log_buf_write 方法，将内容重定向到自己的buffer中。
      > 优点：简单，兼容性相对还好。

      > 缺点：要一直打开。
   3. 自定义获取代码。通过移植底层获取logcat的实现，通过socket直接跟logd交互。
      > 优点：比较灵活，预先分配好资源，成功率也比较高。

      > 缺点：实现非常复杂
二. 获取Java 堆栈
   native崩溃时，通过unwind只能拿到Native堆栈。我们希望可以拿到当时各个线程的Java堆栈
   1. Thread.getAllStackTraces()。
       > 优点：简单，兼容性好。

       > 缺点：
       >   1. 成功率不高，依靠系统接口在极端情况也会失败。
       >   2. 7.0之后这个接口是没有主线程堆栈。
       >   3. 使用Java层的接口需要暂停线程
   2. hook libart.so。
   
      [通过 hook ThreadList获取更完整的 Java 堆栈](#hook-ThreadList-to-get-full-stack)，获得跟ANR一样的堆栈。为了稳定性，我们会在fork子进程执行。
      
      获取Java堆栈的方法还可以用在卡顿时，因为使用fork进程，所以可以做到完全不卡主进程。
      > 优点：信息很全，基本跟ANR的日志一样，有native线程状态，锁信息等等。

      > 缺点：黑科技的兼容性问题，失败时可以用Thread.getAllStackTraces()兜底
      

### GC吞吐量是什么？怎么决定GC的
注意阻塞式 GC 的次数和耗时，因为它会暂停应用线程，可能导致应用发生卡顿

### 
评论里提到的hook gc来避免gc引起的memory churn的技术，或者常见的引起内存泄漏几种情况的解决、数据结构优化(arraymap等)，更换序列化方案，view复用，object pool等优化方法，也没有具体的去讲解内存相关的操作系统概念和Android虚拟机heap space的结构和allocator的执行原理


### tgkill 是什么？ 

### 性能测试工具APM == 腾讯GT 

### 为啥32位只有4GB虚拟内存，可分配的只有3GB？

### Android 微信高性能日志存储库Xlog的使用

### c++重定向原理？printf为什么可以输入变长的参数？

### 线程数太多为啥会引起oom

### hook liblog.so获取logcat是通过plt hook来操作么？可以用爱奇艺的那个xhook用到生产环境上么
 

### 通过 hook ThreadList获取更完整的 Java 堆栈
<a name="hook-ThreadList-to-get-full-stack"></a>
用的是偏移的方法，但是我们这里是通过fork进程来做的，就算崩溃也不会影响主进程。

用ThreadList::ForEach遍历art线程，首先要拿到ThreadList指针，我查了符号表，GetThreadList并未导出，如何获取呢？如果用thread_list_在Runtime类中的偏移获取，会有兼容性问题，文中所说的黑科技，是否有更好的办法？

前一阵在研究ANR监控，发现可以通过拦截SIGQUIT信号监听ANR，调用Runtime::DumpForSigQuit获取trace文件的信息，再通过tgkill发送SIGQUIT给Signal Catcher线程。线上观察了一段时间，基本没有兼容性问题，除了Android 7及以下谷歌本身的bug偶尔会造成崩溃: b/36445592 Don't use pthread_getschedparam since pthread may have exited.

这个方法在Android 5会有问题，需要手动暂停线程并做锁状态检查，要用到几个未导出符号(其中ThreadList的resumeAll/suspendAll通过ScopedSuspendAll解决了)

### WatchDog干嘛的
看门狗 WatchDog 的作用是监控重要服务的运行状态，当重要服务停止时，发生 Timeout 异常崩溃，WatchDog 负责将应用重启。而当关闭 WatchDog（执行stop（）方法）后，当重要服务停止时，也不会发生 Timeout 异常，是一种通过非正常手段防止异常发生的方法。
### NDK为什么能编译通过忘记写返回值的函数
貌似目前NDK中对于错误的检查还不够完善，比如对于函数的返回值，写了一个函数本身要写返回值，后面忘记写了发现IDE没有提示，并且能编译通过，但是在运行的时候会出现崩溃，而且崩溃的信息非常少，貌似就记得最后一行signal 6 还是 signal -1，当时的我看到这个是非常蒙逼的，大量的回退代码，添加日志，勉强找到，要是早点知道signal ，拿着对应的code对比，可能没那么痛苦

### 获取JNI/C++崩溃堆栈，通过DumpReferenceTables 统计JNI的引用表
找到了DumpReferenceTables工具的出处：
在dalvik.system.VMDebug类中，是一个native方法，亦是static方法；在JNI中可以这么调用
jclass vm_class = env->FindClass("dalvik/system/VMDebug");
jmethodID dump_mid = env->GetStaticMethodID( vm_class, "dumpReferenceTables", "()V" );
env->CallStaticVoidMethod( vm_class, dump_mid );

### Android Bitmap 内存分配的变化
- 在 Android 3.0 之前，Bitmap 对象放在 Java 堆，而像素数据是放在 Native 内存中。如果不手动调用 recycle，Bitmap Native 内存的回收完全依赖 finalize 函数回调，熟悉 Java 的同学应该知道，这个时机不太可控。
- Android 3.0～Android 7.0 将 Bitmap 对象和像素数据统一放到 Java 堆中，这样就算我们不调用 recycle，Bitmap 内存也会随着对象一起被回收。不过 Bitmap 是内存消耗的大户，把它的内存放到 Java 堆中似乎不是那么美妙。即使是最新的华为 Mate 20，最大的 Java 堆限制也才到 512MB，可能我的物理内存还有 5GB，但是应用还是会因为 Java 堆内存不足导致 OOM。Bitmap 放到 Java 堆的另外一个问题会引起大量的 GC，对系统内存也没有完全利用起来。
- 有没有一种实现，可以将 Bitmap 内存放到 Native 中，也可以做到和对象一起快速释放，同时 GC 的时候也能考虑这些内存防止被滥用？NativeAllocationRegistry 可以一次满足你这三个要求，Android 8.0 正是使用这个辅助回收 Native 内存的机制，来实现像素数据放到 Native 内存中。Android 8.0 还新增了硬件位图 Hardware Bitmap，它可以减少图片内存并提升绘制效率。
- Android 9.0 ？ ⭐
- 
## Android ANR的产生原因，如何定位ANR
> [彻底理解安卓应用无响应机制](http://gityuan.com/2019/04/06/android-anr/)
需要注意：**监控消息队列的运行时间**无法准确地判断是否真正出现了 ANR 异常，也无法得到完整的 ANR 日志。这个方案更应该放到卡顿的性能范畴。

## onCreate 方法里写死循环会 ANR 吗
ANR 的四种场景：
1. Service TimeOut: service 未在规定时间执行完成： 前台服务 20s，后台 200s
2. BroadCastQueue TimeOut: 未在规定时间内未处理完广播：前台广播 10s 内, 后台 60s 内
3. ContentProvider TimeOut: publish 在 10s 内没有完成
4. Input Dispatching timeout: 5s 内未响应键盘输入、触摸屏幕等事件
Activity 的生命周期回调的阻塞并不在触发 ANR 的场景里面，所以并不会直接触发 ANR。只不过死循环阻塞了主线程，如果系统再有上述的四种事件发生，就无法在相应的时间内处理从而触发 ANR

## Handler 40问
见[Handler面试40问答案整理](https://ophsysbilla.github.io/2020-11-16-Handler%E9%9D%A2%E8%AF%9540%E9%97%AE%E7%AD%94%E6%A1%88%E6%95%B4%E7%90%86.html)

## 图片内存泄漏如何处理
内存泄漏是指没有使用的对象资源与GC-Root保持可达路径，导致系统无法进行回收。
1. 使用软引用引用Bitmap，Bitmap在确定不调用后需要recycle()，然后设置为null
2. 分辨率大的图片需要等比例缩小

## 图片占用内存大小如何计算
```Java
Bitmap.Config ARGB_8888(4B)：由4个8位组成，即A=8，R=8，G=8，B=8，那么一个像素点占8+8+8+8=32位（4字节）
Bitmap.Config ARGB_4444(2B)：由4个4位组成，即A=4，R=4，G=4，B=4，那么一个像素点占4+4+4+4=16位 （2字节）
Bitmap.Config RGB_565(2B)：没有透明度，R=5，G=6，B=5，，那么一个像素点占5+6+5=16位（2字节）
Bitmap.Config ALPHA_8(1B)：每个像素占8位，只有透明度，没有颜色。
Bitmap.Config RGBA_F16(8B)：
Bitmap.Config HARDWARE：不可变不能被复制，仅存在于graphic memory里，只是为了draw
一个像素的位数总和越高，图像也就越逼真。
```
在色彩模式为`ARGB_8888`的情况下，假设有一张 1080 x 452 的图片，`Bitmap.decodeResource()`源代码中里时：
新高度 = 图片宽度 *（ 设备dpi / 资源目录对应dpi ）
新高度 = 图片高度 *（ 设备dpi / 资源目录对应dpi ）


|文件(fold)|密度值(density)|
|:-----|:-----|
|ldpi|120|
|mdpi|160|
|hdpi|240|
|xhdpi|320|
|xxhdpi|480|

由于是`drawable`文件夹下的图片，默认为160dpi，假设设备dpi为480
所以占用的Bitmap内存是：
`1080 * ( 480dpi / 160dpi ) * 452 * ( 480dpi / 160dpi ) * 4 = 4393440B = 4290KB  = 4.18MB` （两次除以1024） 

如果是网络的图片，就不会进行dpi转换，直接是分辨率 * 像素大小
`1080 * 452  * 4 = 1952640 = 1906KB  = 1.86MB` 

## OkHttp相关
### 断点续传
1. 客户端请求 `Range: bytes=200-1000` 见[Range](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Range)
2. 服务器返回 `206 Partial Content` 标示Content-Range: bytes 200-1000/67589 
3. 同时服务器响应的header中有`Accept-Ranges: bytes`
4. 使用RandomAccessFile允许自由定义文件记录指针，在同一个文件的不同位置进行**并发读写**
5. 客户端头的 `If-Range` 需要和 `Range` 配合起来使用，否则会被服务端忽略
![](https://cdn.ancii.com/article/image/v1/Qj/zb/RO/ORzjbQVFJasX5aR2DaTg2_Zx5KSH75KdTtVIpx5BAibeJW3s2CVwUVt3uFqg5I-4TSjNCnhoAJhLsjcM99fslnxQJbDmwU5z8sIvZN3h5MQ.jpg)

> 以下摘自 [图解：HTTP 范围请求，助力断点续传、多线程下载的核心原理](https://juejin.im/post/6844903642034765837)：
> 1. 范围请求需要 HTTP/1.1 及之上支持
> 3. 客户端通过在请求头中添加 `Range` 这个请求头，来指定请求的内容实体的字节范围。
> 4. 服务端通过 `Content-Range` 来标识当前返回的内容实体范围，并使用 `Content-Length` 来标识当前返回的本次压缩后要读多少字节。
> 5. 客户端可以通过 `If-Range` 来区分资源文件是否变动，它的值来自 `ETag` 或者 `Last-Modifled`。如果资源文件有改动，服务器会返回code 200重新走下载流程。

#### 为什么多线程下载可以提速
见这篇文章：[撸了个多线程断点续传下载器，我从中学习到了这些知识](https://zhuanlan.zhihu.com/p/165048187)
个人理解，当发生了丢包，检测到拥塞时，TCP慢启动会减半线性增长发送窗口。而多线程一起下载，其他线程可能正在正常下载，只影响发生丢包的线程

![拥塞窗口](https://pic3.zhimg.com/80/v2-d6c924f27796e5cc5aad061a70399812_1440w.jpg)

### 请求队列的使用的线程池
Dispatcher类里有**两个异步队列，一个同步队列**，`readyAsyncCalls`是没排上队的请求。
> 没排上队是因为在进队时，目前正在执行的请求大于64或者同Host的请求大于5了，参考线程池的思想。
`runningSyncCalls`和`runningAsyncCalls`都是正在执行（包括已经取消但没结束）的请求，区别是一个同步一个异步。
结束后。
在每个请求停止后会去promoteCall()，遍历`readyAsyncCalls`能够执行执行，放进`runningAsyncCalls`里。和线程池里的`Worker`一个套路。
### okhttp责任链，递归调用拦截器
根据网络的分层思想，把每一步要做的操作顺序排列为一个list。
每个拦截器(`Intercept`)都要手动调用`chain.proceed()`去调用list里下一个index的拦截器(`Intercept`)来达到调用下一层的效果，并递归返回response。
> 除了发起真正的网络请求的`CallServerInterceptor`，它不用调用下一层，只用递归返回response

#### 应用拦截器interceptors和网络拦截器networkInterceptors的区别在哪
区别在应用拦截器是在整个操作前，网络拦截器是在真正请求前
由于有多次重试等拦截器的存在，一般应用拦截器只会执行一次，网络拦截器可能会走很多次
```JAVA
	Response getResponseWithInterceptorChain() throws IOException {
		// Build a full stack of interceptors.interceptors
		List<Interceptor> interceptors = new ArrayList<>();
		interceptors.addAll(client.interceptors());//应用拦截器
		//重定向、重试
		interceptors.add(retryAndFollowUpInterceptor);
		//用户应用层和网络从桥梁。主要包含：
		//1. 将用户的request，转变为网络层的request，比如添加各种请求头，UA ,Cookie , Content-Type等。
		//2. 将网络层的response转变为用户层的response，比如解压缩，除去各种请求头等。
		interceptors.add(new BridgeInterceptor(client.cookieJar()));
		//缓存
		interceptors.add(new CacheInterceptor(client.internalCache()));
		//负责与服务器之间建立连接。
		interceptors.add(new ConnectInterceptor(client));
		if (!forWebSocket) {
		  interceptors.addAll(client.networkInterceptors());//网络拦截器
		}
		//负责请求服务器
		interceptors.add(new CallServerInterceptor(forWebSocket));
		
		//第一个chain
		Interceptor.Chain chain = new RealInterceptorChain(
		    interceptors, null, null, null, 0, originalRequest);
		//通过链式请求的得到response
		return chain.proceed(originalRequest);
    }
```

### OKHTTP Socket连接池
复用Http连接减少请求延迟。
避免网络连接的时延，以及避免TCP调谐带来的带宽过小的问题。
okhttp记录了每个socket流使用情况，同时设定了每个socket能同时使用多少流。

#### socket复用有何标准
- 共享socket,HTTP/2支持所有连接到同一个主机的请求共享socket

对每个socket来说：
1. host完全一样，可以直接复用return true
2. 没有达到socket连接池复用数量的总上陷，且这个socket流还没有关闭
2. okhttp只在2.0的时候会复用
4. 有相同IP地址的（说明两个对比的连接都已经DNS得到过IP），且没有使用代理，有了代理就不知道最初的IP是多少了 
5. 复用socket连接意味着跳过了https握手，所以要验证证书，如果证书可以覆盖的话，IP不一样也可以

> 一个socket如果被复用，只有http2.0允许同一个socket在同一个时候写入多个流数据。http1.x协议下当前socket没有其他流正在读写时可以复用，http2.0对流数量没有限制

> - http1.x
> 在http 1.x协议下，所有的请求的都是顺序的，即使使用了管道技术（可以同时按顺序连续发送请求，但消息的返回还是按照请求发送的顺序返回）也是如此，因此一个socket在任何时刻只能有一个流在写入，这意味着正在写入数据的socket无法被另一个请求复用

> - http2.0
> http2.0协议使用了多路复用技术，允许同一个socket在同一个时候写入多个流数据，每个流有id，会进行组装，因此，这个时候正在写入数据的socket是可以被复用的。

#### 一个socket何时会被关闭？socket池何时被清理
[okhttp通过一个单独的线程来清理socket池](https://juejin.im/entry/6844903750000181256)。每次put一个新连接的时候都会判断当前是否有在清理socket池。
这个线程`while(true) `循环找出空闲时间最长的sockect连接，默认可以保存5个空闲线程，并且5分钟内依旧空闲的话，将被关闭回收。
> 规定时间t = sockect默认设定的生存时间
1. 如果哪个空闲连接超过了规定时间t，关掉这个socket，立刻返回继续找下一个
2. 如果每个空闲连接都没有超过规定时间t的，就返回空闲时间最长的sockect连接，得到`diff = 规定时间 - 空闲时间`，开始`wait(diff)`，`wait()`完后继续下一次清理
3. 如果没有空闲连接， 最少`wait()`等待一个规定时间t
4. 没有任何连接，返回-1退出`while(true)`，等待下次put个新连接的时候的时候重新开启清理socket池的线程
> 只要连接池中有连接，基本清理线程就一直存在，直到所有连接被释放该线程才会停止

清理的线程池okHttp使用的是：
```JAVA
  private static final Executor executor = new ThreadPoolExecutor(0 /* corePoolSize */,
      Integer.MAX_VALUE /* maximumPoolSize */, 60L /* keepAliveTime */, TimeUnit.SECONDS,
      new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp ConnectionPool", true));
```
和newCachedThreadPool一样，可以动态添加到无限大的，用的SynchronousQueue当等待队列所以新来的线程没有地方去就会阻塞住，直到开了一个空线程来执行。
[Okhttp3 总结研究 （面试）](https://blog.csdn.net/u012881042/article/details/79759203?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param)

### OkHttp的HTTP缓存
主要是根据`Header(Date\Expires\Last-Modified\ETag\Age)`来缓存响应数据减少重复的网络请求

## 

## 点击桌面图标进入我们软件应用时发生了什么？
通过翻阅 Application 启动的源码，当我们点击桌面图标进入我们软件应用的时候，会由 AMS 通过 Socket 给 Zygote 发送一个 fork 子进程的消息，当 Zygote fork 子进程完成之后会通过反射启动 ActivityThread##main 函数，最后又由 AMS 通过 aidl 告诉 ActivityThread##H 来反射启动创建Application 实例，并且依次执行 `attachBaseContext` 、`onCreate` 生命周期，由此可见我们不能在这 2 个生命周期里做主线程耗时操作。

## OOM是什么，怎么导致的？

- java.lang.OutOfMemoryError: Java heap space ------>java堆内存溢出，此种情况最常见，一般由于内存泄露或者堆的大小设置不当引起。对于内存泄露，需要通过内存监控软件查找程序中的泄露代码，而堆大小可以通过虚拟机参数-Xms,-Xmx等修改。
- java.lang.OutOfMemoryError: PermGen space ------>java永久代溢出，即方法区溢出了，一般出现于大量Class或者jsp页面，或者采用cglib等反射机制的情况，因为上述情况会产生大量的Class信息存储于方法区。此种情况可以通过更改方法区的大小来解决，使用类似-XX:PermSize=64m -XX:MaxPermSize=256m的形式修改。另外，过多的常量尤其是字符串也会导致方法区溢出。
- java.lang.StackOverflowError ------> 不会抛OOM error，但也是比较常见的Java内存溢出。JAVA虚拟机栈溢出，一般是由于程序中存在死循环或者深度递归调用造成的，栈大小设置太小也会出现此种溢出。可以通过虚拟机参数-Xss来设置栈的大小。

## Glide是如何监听周期的？
#### 监听生命周期

用了一个没有UI的Fragment（名为RequestManagerFragment）入到传入的非Application的Context的FragmentManager中

监听传入的Context的AcitivityLifeCycle（由RequestManager实现的事实上）

#### 监听网络状态

依然是用上面的fragment的AcitivityLifeCycle回调达到的

在onStart和onStop的时候分别注册、反注册网络状态的监听，来达到在变得有网的情况下重启请求的目的

#### 监听内存状态

在Glide构造的时候会调用registerComponentCallbacks进行全局注册

> registerComponentCallbacks是ApplicationContext提供具有内存过低等时机的回调（onTrimMemory、onLowMemory、onConfigurationChanged）

在`onTrimMemory`时，会依次执行：

```JAVA
memoryCache.trimMemory(level);
bitmapPool.trimMemory(level);
arrayPool.trimMemory(level);
```

### Glide 加载 Gif 的原理
就是将 Gif 解码成多张图片进行无限轮播，每帧切换都是一次图片加载请求，再加载到新的一帧数据之后会对旧的一帧数据进行清除，然后再继续下一帧数据的加载请求，以此类推，使用 Handler 发送消息实现循环播放

## Parcelable和Serializable区别
Parcelable的性能比Serializable好，在内存开销方面较小，所以在内存间数据传输时推荐使用Parcelable，如activity间传输数据，而Serializable可将数据持久化方便保存，所以在需要保存或网络传输数据时选择Serializable，因为android不同版本Parcelable可能不同，所以不推荐使用Parcelable进行数据持久化。

Serializable序列化不保存静态变量，可以使用Transient关键字对部分字段不进行序列化，也可以覆盖writeObject、readObject方法以实现序列化过程自定义。

## 两个Activity切换生命周期是怎样的？
 [两个Activity切换生命周期是怎样的？](https://blog.csdn.net/lei396601057/article/details/109272301)
假设 从 A Activity 跳转到 B Activity
- lanchMode不同：
1. B Activity 的 `launchMode` 为 `standard` ｜｜ B Activity 没有可复用的实例时：A.onPause -> B.onCrete -> B.onStart -> B.onResume -> A.onStop
2.  B Activity 的 `launchMode` 为 `singleTop` && B Activity 已经在栈顶时：（一些特殊情况如通知栏点击、连点）此时只有 B 页面自己有生命周期变化: B.onPause -> B.onNewIntent -> B.onResume
3. 当 B Activity 的 `launchMode` 为 `singleInstance` \ `singleTask` && 对应的 B Activity 有可复用的实例时: A.onPause -> B.onNewIntent -> B.onRestart -> B.onStart -> B.onResume -> A.onStop -> ( 如果 A 被移出栈的话还有一个 A.onDestory)

> 摘自[5 道刁钻的 Activity 生命周期面试题](https://zhuanlan.zhihu.com/p/150787285)
- 生命周期回调都是 AMS 通过 Binder 通知应用进程调用的；而弹出 Dialog、Toast、PopupWindow 本质上都直接是通过 WindowManager.addView() 显示的（没有经过 AMS），所以不会对生命周期有任何影响。
- 如果是启动一个 `Theme` 为 `Dialog` 或 `Transparent` 的 Activity , 则生命周期为： A.onPause -> B.onCrete -> B.onStart -> B.onResume。
	- 注意这边没有前一个 Activity 不会回调 onStop，因为只有在 Activity 切到后台不可见才会回调 onStop；
	- 弹出 Dialog 主题的 Activity 时前一个页面还是可见的，只是失去了焦点而已所以仅有 onPause 回调。
	- > 这是因为透明的弹窗前一个 Acitivity 被认为其实还可见

### launchMode的适用场景
1. singleTop 适合接收通知启动的内容显示页面。例如，某个新闻客户端的新闻内容页面，如果收到10个新闻推送，每次都打开一个新闻内容页面是很烦人的。

2. singleTask 适合作为程序入口点。例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。之前打开过的页面，打开之前的页面就ok，不再新建。

3. singleInstance 适合需要与程序分离开的页面。例如闹铃提醒，将闹铃提醒与闹铃设置分离。singleInstance不要用于中间页面，如果用于中间页面，跳转会有问题，比如：A -> B (singleInstance) -> C，完全退出后，在此启动，首先打开的是B。

#### Activity 在 onResume 之后才显示的原因是什么
- `onCreate` 方法里调用 `setContentView` 。里面是直接调用 `window` 的 `setContentView`，创建一个 `DecorView` 用来包住我们创建的布局。
- 加载好了布局，生成一个 `ViewTree` ，`WindowManager#addView` 方法最终将 `DecorView` 添加到 `WMS`。
- 在 `onResume` 回调之后，会创建一个 `ViewRootImpl` ，有了它之后应用端就可以和 `WMS` 进行双向调用了。同时也是通过 `ViewRootImpl` 从 `WMS` 申请 `Surface` 来绘制 `ViewTree` 

#### onActivityResult 在哪两个生命周期之间回调
> You will receive this call immediately before onResume() when your activity is re-starting
`onActivityResult` 回调先于该 Activity 的所有生命周期回调，从 B Activity 返回 A Activity 的生命周期调用为： B.onPause -> A.onActivityResult -> A.onRestart -> A.onStart -> A.onResume

## lateinit var和by lazy
lateinit var只是让编译期忽略对属性未初始化的检查，后续在哪里以及何时初始化还需要开发者自己决定。
by lazy 用了双重检测初始化 SynchronizedLazyImpl， 保证变量在被取到的时候初始化一次


## AsyncTask为什么坑？工程里用什么代替AsyncTask那？
- 如果AsyncTask被声明为Activity的非静态的内部类，那么AsyncTask会保留一个对创建了AsyncTask的Activity的引用。
- 屏幕旋转或Activity在后台被系统杀掉等情况会导致Activity的重新创建，之前运行的AsyncTask会持有一个之前Activity的引用，这个引用已经无效。
- doInBackground方法中有一个不可中断的操作，那么cancel掉是不会让AsyncTask停止的。cancel调用了thread的interrupt操作
- 在Android 1.6之前的版本，AsyncTask是串行的，在1.6至2.3的版本，改成了并行的。在2.3之后的版本又做了修改，可以支持并行和串行，当想要串行执行时，直接执行execute()方法，如果需要并行执行，则要执行executeOnExecutor(Executor)。
  > 串行是通过一个ArrayDeque来完成的，这导致必须要任务必须要串行等待排队，如果中间谁阻塞时间过长会拖慢整个队列，而且不知道任务什么时候会执行
  > 即使用executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR)通过线程池并行，这个线程池也是有最大并发数限制的，超过就会报错
- AsyncTask的并行线程池是一个静态变量，所有 AsyncTask 共享一个线程池

## IM进程为什么单开一个进程？
IM单开进程是因为压到后台时，进程越大越容易被杀掉、进程优先级越低越容易被杀掉

## IM协议包头都包含什么？
1. 包长度
2. 包序列号 SEQ
3. 操作类型 COMMAND （比如心跳、文本消息、重置消息）
4. 返回码
5. 为了兼容加入的协议版本号
6. 了负载均衡加入的模块id
……

> ———[移动IM开发指南1：如何进行技术选型](http://yunxin.163.com/blog/im9-0608/)
## 老生常谈，你知道有什么后台保活方式？
保活可以减少 Application 创建跟初始化的时间，让冷启动变成温启动。
都是想要提高进程优先级`OOM_ADJ`，且在进程被杀死后拉活。
> ——[进程保活](https://blog.yorek.xyz/android/paid/zsxq/week16-keep-app-alive/)

- 监听息屏和解锁的广播，在息屏时启动一个只有一个像素的透明Activity，此时应用就位于前台了，优先级为0。在解锁时将此Activity销毁。
  > 如果保活的Activity位于另外一个进程中，需要注意广播只能在代码中动态注册。
- 利用JobScheduler机制。搭配前台Service技术提高进程优先级
- 第一种主要有系统闹钟，各种事件的 BroadcastReceiver，任务被移除的回调通知等。带通知的前台Service。
  > 从8.0开始，很多广播只能在代码中动态注册，无法静态注册。也就是说，App被杀死后，无法接收到系统的广播了
- 第二种已知的就是在 4.4 及以前版本上，使用 native 进程，并将该进程从 davilk 父进程中脱离，挂接到 init 进程上，以此避开系统的查杀。然后在这个 native 进程中，定时唤起应用。为了让这个 native 进程更轻巧，可以使用 exec 的方式启动一个可执行文件，以除掉直接 fork 带入的 Zygote 进程环境。另外，这种方式也被用在监听自己应用被卸载时弹出调查窗口。

- 第三种方式互相调用指定的 Service，或发指定得广播即可，互相唤醒。

> —— [Android 即时通讯开发小结（一）](http://yunxin.163.com/blog/im5-0608/)


## 长连接和心跳为什么要这样设计？
- 对于客户端而言，使用 TCP 长连接来实现业务的最大驱动力在于：在当前连接可用的情况下，每一次请求都只是简单的数据发送和接受，免去了 DNS 解析，连接建立等时间，大大加快了请求的速度，同时也有利于接受服务器的实时消息
- 接保持的前提必然是检测连接的可用性（对方还活着），并在连接不可用时主动放弃当前连接并建立新的连接。

### 可以使用TCP自带的Keep Alive来做长连接吗
不用。
- Keep Alive时间太长，且固定在系统参数里。
- keepalive只能检测连接是否存活，不能检测连接是否可用。比如服务器因为负载过高导致无法响应请求但是连接仍然存在，此时keepalive无法判断连接是否可用。
- 如果TCP连接中的另一方因为停电突然断网，我们并不知道连接断开，此时发送数据失败会进行重传，由于重传包的优先级要高于keepalive的数据包，因此keepalive的数据包无法发送出去。只有在长时间的重传失败之后我们才能判断此连接断开了。


## 群发消息如何做保障避免消息风暴扩散？
群消息还是非常有意思的，可达性、实时性、离线消息、消息风暴扩散等等等等，做个总结：

1）不管是群在线消息，还是群离线消息，应用层的ACK是可达性的保障；
2）群消息只存一份，不用为每个用户存储离线群msg_id，只需存储一个最近ack的群消息id/time；
3）为了减少消息风暴，可以批量ACK；
4）如果收到重复消息，需要msg_id去重，让用户无感知；
5）离线消息过多，可以分页拉取（按需拉取）优化。
> ——[群消息这么复杂，怎么能做到不丢不重？](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651959643&idx=1&sn=844afa6a31770fa587474ecd73c3b3b3&chksm=bd2d04878a5a8d91c5c93ad8e85254185c63eb419457efbedcba54a9b8a9053da1e8980a694a&scene=21#wechat_redirect)

## 如何保证不丢消息
1）im系统是通过超时、重传、确认、去重的机制来保证消息的可靠投递，不丢不重
2）一个“你好”的发送，包含上半场msg:R/A/N与下半场ack:R/A/N的6个报文

3）im系统难以做到系统层面的不丢不重，只能做到业务层面的不丢不重
> ——[微信为什么不丢消息？](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651959606&idx=1&sn=f9561231dd33bcd0550b8d0d59d6b876&chksm=bd2d04ea8a5a8dfce90c870279a7f74b7aedd802c2d699dd919d7e40ebe30699381517c2d54b&scene=21#wechat_redirect)

## 如何加快dns耗费的时间？
DNS 解析速度无比龟速，一次 DNS 解析的时间甚至能赶上一次 TCP 连接的时间，几秒，十几秒，甚至三，四十秒的请求时间都很常见。另一方面，由于运营商的不作为和作为，移动网络下 DNS 也呈现了解析准确度低和经常被劫持的状态。

针对这些情况可以考虑使用 HTTP DNS，github 也有很多开源的实现，比如HttpDNSLib。而最激进的方法自然是像前面优化 lbs 一样，在个个环节中避免 DNS 解析：

lbs 返回服务器 ip 而非域名
本地缓存 lbs ip 地址备用，请求 lbs 时优先使用 ip 而非域名
> ——[移动IM开发指南3：如何优化登录模块](https://zhuanlan.zhihu.com/p/38926987)

## 如何防作弊考勤签到？
- LBS + wifi/基站位置
- GPS 方式的位置信息来自卫星，精度很高，但是 GPS 方式仅在户外有效，其首次获取位置时间较长并且非常耗费电量。

## Android动画原理
`Android`中动画分为3种：
`Tween Animation`（补间动画）：通过对场景里的对象不断做图像变换产生的动画效果。只支持RotateAnimation旋转、 AlphaAnimation透明度、 ScaleAnimation缩放、 TranslateAnimation移动。
  > 只是改变了view的绘制，view的点击区域及位置没有真正变化

`Frame Animation`（帧动画）：顺序播放事先做好的图像，是一种画面转换动画。类似多个图片的连续播放
  > - 要在代码中调用Imageview的setBackgroundResource方法，如果直接在XML布局文件中设置其src属性当触发动画时会FC。
  > - 在动画start()之前要先stop()，不然在第一次动画之后会停在最后一帧，这样动画就只会触发一次。
  > - 不要在onCreate中调用start，因为AnimationDrawable还没有完全跟Window相关联，如果想要界面显示时就开始动画的话，可以在onWindowFoucsChanged()中调用start()。

`Property Animation`：属性动画，通过动态地改变对象的属性从而达到动画效果，属性动画为API 11新特性。会真正改变view的属性
  > `ValueAnimator`封装了一个`Interpolator`定义了属性值在开始值与结束值之间的插值方法。 表示fraction
  > 还封装了一个`TypeAnimator`，根据开始、结束值与`TimeIniterpolator`计算得到的`fraction`计算出属性值。常用的是`FloatEvaluator`，生成float类型的变化。表示value
  > 子类 `ObjectAnimator` 设置自定义属性时，必须有对应的一个setter函数：set<PropertyName>（驼峰命名法）和相应属性的getter方法：get<PropertyName>
  > 使用Node来达成二叉树保存动画依赖


## View的事件分发流程
[View的事件分发流程](https://juejin.cn/post/6844903902257627144)
1. 如果你在500毫秒内抬起手指，那么你就只能执行点击事件，不能执行长按事件；
2. 如果你在500毫秒后抬起，并且你设置了onLongClickListener并在onLongClick()方法中返回了false 或者 你没有设置
3. onLongClickListener回调，那么你执行完长按事件后还可以执行点击事件
4. 如果你设置了onLongClickListener回调并在onLongClick()方法中返回了true，那么你就不能执行点击事件
5. OnLongClickListener的onLongClick()方法的优先级高于onClickListener的onClick()方法
6. 手指在抬起前，不小心移动了一下，就会触发ACTION_CANCEL或ACTION_MOVE，这个时候它就会根据条件(手指是否移出View的范围)通过调用 removeLongPressCallback()或 removeTapCallback()方法移除CheckForLongPress或CheckForTap任务

1. 如果同时设置了OnTouchListener、OnLongClickListener和OnClickListener回调，根据优先级事件的传递顺序是：`onTouch() -> onLongClick() -> onClick()`
> 其中除了`onClick()`都有boolean返回值，返回值能决定下一个方法是否被调用，`onClick()`优先级最低，连返回值都没有

> `onTouch()` 在 `dispatchTouchEvent() `中

> 检查长按的在`View::onTouchEvent()`的A`CTION_DOWN`中，如果View是可以点击的，`View::onTouchEvent()`最终一定返回true，表示消费了此事件。父容器就不会继续找了

2. 如果大家希望自己的View增加它的touch范围，可以尝试使用TouchDelegate
3. enable = false 状态下的View是直接消费点击事件
4. 

```Java 
    // View.java   
	// View的dispatchTouchEvent()方法开始进入View的事件分发流程
    public boolean dispatchTouchEvent(MotionEvent event) {
        boolean result = false;
		ListenerInfo li = mListenerInfo;
		if (li != null && li.mOnTouchListener != null
				&& (mViewFlags & ENABLED_MASK) == ENABLED
				&& li.mOnTouchListener.onTouch(this, event)) { // 先询问listener.onTouch
			result = true;
		}
		if (!result && onTouchEvent(event)) { // 询问本view的onTouchEvent。如果控件可点击(clickabale或longClickabale，只要有一个为true)，onTouchEvent()返回true。
			result = true;
		}
        return result;
    }
```

```Java


```
## singleTask

`singleTask`: 该Activity全局唯一。相同的`taskAffnity`属性在一个task栈里，多个相同的`taskAffnity`的栈只会展示一个`Activity`在最近打开的应用里。如果栈里存在有旧Activity，则不会调用OnCreate，而是OnNewIntent，且会清除到栈顶的所有 Activity。 

`singleInstance`: 一个task里只有这一个Activity。再在`singleInstance` 里打开新Activity，会将task 栈拿过来覆盖在上面 

`singleTop` 如果是当前顶部的第一个，则复用
> 当前singleTop模式的act正处于栈顶时，跳转该act会调用onNewintent方法且不会重新创建该act实例，只会重新调用该实例，生命周期为：
> - onPause->onNewIntent->onResume

## 在Activity的oncreate方法中调用finish都执行哪些生命周期 
onCreate()->onDestory()

## fragment生命周期
![fragment生命周期](https://img-blog.csdnimg.cn/20190102215232426.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l6X2NmbQ==,size_16,color_FFFFFF,t_70)

## kotlin协程原理和suspend原理
suspend原理：编译器帮忙实现了状态机，根据不同状态调用不同回调

## LruCache与DiskLruCache缓存详解
1. LruCache就是通过设置`LinkedHanhMap`的链表顺序为Lru来实现Lru缓存的
2. 经常被操作的都放在链表的尾端，表头对象肯定就是不常用的
3. 线程安全的

## Databinding 理解
-  `ActivityMainBindlmpl.java` 是通过APT技术自己生成出来的。
  > `APT`(`Annotation Processing Tool`)即注解处理：在编译期，通过注解生成.java文件
- 双向绑定，没有用反射
  > `mRebindRunnable` 最终会执行到 `executeBindings()` 方法。把从JavaBean属性中读出来的值设置到View上，实现了Model到View更新；为View控件设置监听(例如色图setTextWatcher)，以改变对应JavaBean的属性的值，实现了View到Model的更新。
- 在ViewDataBinding的静态代码块，会有一个全局的监听`OnAttachStateChangeListener`，用来通知数据更新：每当View改变的时候，这个监听中的回调方法就会被执行，通过一个Handler去post `mRebindRunnable` ，从而更新Model
- 需要额外消耗内存的地方
  > - 定义了一个**额外的数组**来记录这些控件
  > - 每个Activity都会有一个Runable，10个Activity就会创建10个Runable
- runnable执行方法异同
```Java
	if (USE_CHOREOGRAPHER) { // > sdkVersion16
		mChoreographer.postFrameCallback(mFrameCallback);
	} else {
		mUIThreadHandler.post(mRebindRunnable);
	}
```
## onResume中Handler.post(Runnable)为什么获取不到宽高？

## bitmap优化
主要在 
1. 尺寸压缩
2. 颜色质量
3. inBitmap
  > 使用inBitmap，在4.4之前，只能重用相同大小的bitmap的内存区域，而4.4之后你可以重用任何bitmap的内存区域，只要这块内存比将要分配内存的bitmap大就可以（必须有相同的解码格式，例如大家都是8888的）这里最好的方法就是使用LRUCache来缓存bitmap，后面来了新的bitmap，可以从cache中按照api版本找到最适合重用的bitmap，来重用它的内存区域。 
4. cache）
