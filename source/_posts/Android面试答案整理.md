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
## 点击屏幕后触发的流程讲一下，从硬件开始说。

## Xposed
是干嘛的？

## 如何计算冷启动时间？冷启动的优化是什么？

## 事件分发

1. [Android事件分发机制 详解攻略，您值得拥有](https://blog.csdn.net/carson_ho/article/details/54136311)

## 如何减少卡顿？刷新原理

![img](https://user-gold-cdn.xitu.io/2020/3/27/17117aa6eb5b454e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


## 内存泄漏原理是什么，LeakCanary是怎么检测到内存分析的？
- LeakCanary将已经销毁的Activity和Fragment所对应的实例放入到弱引用中，并关联一个引用队列。通过registerActivityLifecycleCallbacks来注册对于我们销毁的Activity的监听.

- 如果实例进行了回收，那么弱引用就会放入到ReferenceQueue中
- 如果一段时间后，所监控的实例还未在ReferenceQueue中出现，那么可以证明出现了内存泄漏导致了实例没有被回收

- 使用lifeCycle来监听Activity和Fragment的生命周期，在onDestory()、onViewDestroy、onFragmentDestroy()时会调用`watch()`检查

- `watch()`会生成一个Activity的UUID到retainedKeys队列中，retainedKeys队列记录了我们执行了监控的引用对象。而queue中会保存回收的引用。所以通过二者的对比，我们就可以找到内存泄漏的引用了

- `watch()`在主线程空闲的时候 `Looper.myQueue().addIdleHandler()`一直检测，多次重试后最终会掉哟罡`ensureGone`

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

## SharedPreferences是跨进程安全的吗？工程中用什么替代？
可以mmkv，ContentProvider也更适合跨进程共享


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
4. `system_server` 接收到请求后，进行一些列准备工作后，再通过 `Binder` IPC 向App进程发送 `scheduleLaunchActivity` 请求
5. App进程binder线程（ `ApplicationThread` ）收到请求后，通过 `Handler` 向主线程发送 `LAUNCH_ACTIVITY` 消息
6. 主线程收到Message后，通过反射机制创建目标Activity，并回调Activity的`onCreate`

## 屏幕刷新的原理是什么？如何减少卡顿？
View 的 requestLayout 和 ViewRootImpl##setView 最终都会调用 ViewRootImpl 的 requestLayout 方法，然后通过 scheduleTraversals 方法向 Choreographer 提交一个绘制任务，然后再通过 DisplayEventReceiver 向底层请求 vsync 垂直同步信号，当 vsync 信号来的时候，会通过 JNI 回调回来，在通过 Handler 往消息队列 post 一个异步任务，最终是 ViewRootImpl 去执行绘制任务，最后调用 performTraversals 方法，完成绘制。
![屏幕刷新的原理](https://upload-images.jianshu.io/upload_images/24142630-409cdaf1c5111e47?imageMogr2/auto-orient/strip|imageView2/2/w/1089/format/webp)
[应用流畅度(FPS)监控](https://github.com/SusionSuc/AdvancedAndroid/blob/master/performance/rabbit/%E5%BA%94%E7%94%A8%E6%B5%81%E7%95%85%E5%BA%A6(FPS)%E7%9B%91%E6%8E%A7.md)
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

## 热修复资源替换
资源注入：资源的动态加载则相对简单。主要是参考Instant Run，通过反射调用AssetsManager的addAssets方法，将增量资源包加载到内存中来，得到新的Resources对象，然后替换掉ActivityThread等所有持有Resources的地方即可。这也是大部分热修复框架中的基本思路。

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


## Zygote Fork进程为什么是用socket，为什么不用binder？
- fork只能拷贝当前线程，不支持多线程的fork
- 不建议在多线程环境使用Fork，而Binder是多线程的。
怕父进程binder线程有锁，然后子进程的主线程一直在等其子线程(从父进程拷贝过来的子进程)的资源，但是其实父进程的子进程并没有被拷贝过来，造成死锁，所以fork不允许存在多线程。而非常巧的是Binder通讯偏偏就是多线程，所以干脆父进程（Zgote）这个时候就不使用binder线程
> 互斥锁是导致无法在多线程环境使用Fork的根本原因，大部分系统中，锁的实现基本是用户态，并非内核态（用户态更加高效）。假设在fork之前，一个线程对某个锁进行lock操作，另一个线程进行fork子进程，会导致持有锁的子线程消失了，那这个锁就会永久锁定，假如另一个线程获取该锁，就会导致死锁了。

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
	1. 基于 `mmap` 。`Client`与 `Server` 处于不同进程有着不同的虚拟地址规则，所以无法直接通信。一个页框可以映射给多个页，那么就可以将一块物理内存分别与 Client 和 Server 的虚拟内存块进行映射。映射的虚拟内存块大小将近 1M (1M-8K)，所以 IPC 通信传输的数据量也被限制为此值
		> —— 摘自[谈谈你对 binder 的理解？](https://zhuanlan.zhihu.com/p/143324649)
	2. 通过 `copy_from_user` 将数据从用户空间拷贝到内核空间，通过 `copy_to_user` 将数据从内核空间拷贝到用户空间。一般的 IPC 方式需要分别调用这两个函数，数据就拷贝了两次，而 binder 将内核空间与目标用户空间进行了 `mmap`，只需调 `copy_from_user` 拷贝一次即可。
		
2. [聊聊怎样学习Binder](https://cloud.tencent.com/developer/article/1698142)内有c++代码简单传输的过程


### 为什么 binder 驱动要运行在内核空间？可以移到用户空间吗？
> [说说你对 binder 驱动的了解？](https://zhuanlan.zhihu.com/p/152237289)

binder 驱动运行在内核空间，向上层提供 /dev/binder 设备节点及 open、mmap、ioctl 等系统调用。
| 结构体 | 说明 | 
|--|--| 
| binder_proc | 描述使用 binder 的进程，当调用 binder_open 函数时会创建 | 
| binder_thread | 描述使用 binder 的线程，当调用 binder_ioctl 函数时会创建 | 
| binder_node | 描述 binder 实体节点，对应于一个 server ，即用户态的 BpBinder 对象 | 
| binder_ref | 描述对 binder 实体节点的引用，关联到一个 binder_node | 
| binder_buffer | 描述 binder 通信过程中存储数据的Buffer | 
| binder_work | 描述一个 binder 任务 | 
| binder_transaction | 描述一次 binder 任务相关的数据信息 | 
| binder_ref_death | 描述 binder_node 即 binder server 的死亡信息 |
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

## Android ANR的产生原因，如何定位ANR
> [彻底理解安卓应用无响应机制](http://gityuan.com/2019/04/06/android-anr/)


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
Bitmap.Config ARGB_8888：由4个8位组成，即A=8，R=8，G=8，B=8，那么一个像素点占8+8+8+8=32位（4字节）
Bitmap.Config ARGB_4444：由4个4位组成，即A=4，R=4，G=4，B=4，那么一个像素点占4+4+4+4=16位 （2字节）
Bitmap.Config RGB_565：没有透明度，R=5，G=6，B=5，，那么一个像素点占5+6+5=16位（2字节）
Bitmap.Config ALPHA_8：每个像素占8位，只有透明度，没有颜色。
一个像素的位数总和越高，图像也就越逼真。
```
假设有一张480x800的图片
1. 在色彩模式为`ARGB_8888`的情况下，一个像素所占的大小为4字节，会占用 `480*800*4/1024KB=1500KB` 的内存；
2. 而在R`GB_565`的情况下，占用的内存为：`480*800*2/1024KB=750KB`

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


### 两个Activity切换生命周期是怎样的？
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