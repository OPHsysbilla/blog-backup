---
title: Android面试答案整理
date: 2020-11-09 18:48:47
tags:
    - 面试
categories: 面试
---

## Retrofit(动态代理)

## 说说组件化和插件化，热更新技术原理

## 知识图谱

1. [Android初级、中级、高级、资深工程师(架构师、专家)技能图谱](https://www.jianshu.com/p/659381fcd4e5)

2. [面试官: 说一下你做过哪些性能优化?](https://juejin.im/post/6844904105438134286)

<!--more-->

## Binder怎么学

1. [为什么Binder的通信只进行了一次拷贝](https://mubu.com/doc/explore/21079)
2. [聊聊怎样学习Binder](https://cloud.tencent.com/developer/article/1698142)内有c++代码简单传输的过程

### 理解AIDL中的in，out，inout吗

1. 一文看得懂的源码：[聊聊怎样学习Binder](https://cloud.tencent.com/developer/article/1698142)

2. 一文超翔实的源码：[Android Bander设计与实现 - 设计篇](https://blog.csdn.net/universus/article/details/6211589)

## Android ANR的产生原因，如何定位ANR


## OkHttp相关
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



## 事件分发

1. [Android事件分发机制 详解攻略，您值得拥有](https://blog.csdn.net/carson_ho/article/details/54136311)

## 如何减少卡顿？刷新原理

![img](https://user-gold-cdn.xitu.io/2020/3/27/17117aa6eb5b454e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



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

```
memoryCache.trimMemory(level);
bitmapPool.trimMemory(level);
arrayPool.trimMemory(level);
```


### [两个Activity切换生命周期是怎样的？](https://blog.csdn.net/lei396601057/article/details/109272301)



