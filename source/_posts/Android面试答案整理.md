---
title: Android面试答案整理
date: 2020-11-09 18:48:47
tags:
    - 面试
categories: 面试
---

## Retrofit(动态代理)
> 动态代理的代理关系是在运行期确定的，Jvm帮忙生成class文件并且会删除class文件
JDK原生动态代理是Java原生支持的，不需要任何外部依赖，但是它只能基于接口进行代理；
CGLIB通过继承的方式进行代理，无论目标对象有没有实现接口都可以代理，但是无法处理final的情况

<!--more-->

## 说说组件化和插件化， 技术原理
APT在编译的开始阶段对java文件进行操作，而像AscpectJ、ASM等则是在java文件编译为字节码文件后

### 能否在加载类的时候，对字节码进行修改?
利用javaAgent和ASM字节码都可以，javaAssist还有Byte-Code都可以
![字节码增强](https://p0.meituan.net/travelcube/12e1964581f38f04488dfc6d2f84f003110966.png)
1. [Java字节码技术(二)字节码增强之ASM、JavaAssist、Agent、Instrumentation](https://blog.csdn.net/hosaos/article/details/102931887)
2. [美团 - 字节码增强技术探索](https://tech.meituan.com/2019/09/05/java-bytecode-enhancement.html)包含有ASM的基本用法和工具
    > **可否在运行时对JVM中的类进行修改并重载？**：通过Instrument充当Agent + Attach API， 依赖JVMTI的Attach API机制实现


## gradle生命周期


## 方法数越界为啥？MultiDex是怎么做的？
1. 方法数超过65536数会报错。
> 早期系统中，dexopt会把每一个类的方法id检索起来，存在一个链表结构里，而这个链表的长度是用一个short类型来保存的，2字节 = 2 ^ 16 = 2 ^ 6 * 2 ^ 10 = 64 * 1024 = 64K = 65536。新版本dexopt修复了这个问题
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

## 如何用代码自行生成接口？
之前是apt，如今是 annotationProcessor
## ASM应用场景
[Gradle+Transform+Asm自动化注入代码](https://www.jianshu.com/p/fffb81688dc5)
[字节码插桩--你也可以轻松掌握](https://www.jianshu.com/p/13d18c631549)
[【Android】函数插桩（Gradle + ASM）](https://www.jianshu.com/p/16ed4d233fd1)
1. 无埋点统计、APM、插桩
其实就是在上面的基础进行各种位置的插桩,具体例子[Android无埋点数据收集SDK关键技术](http://www.jianshu.com/p/b5ffe845fe2d)

2. 瘦包

蘑菇街的[ThinRPlugin](http://www.jianshu.com/p/b5ffe845fe2d)插件
相关原理：android中的R文件，除了styleable类型外，所有字段都是int型变量/常量，且在运行期间都不会改变。所以可以在编译时，记录R中所有字段名称及对应值，然后利用asm工具遍历所有class，将引用R字段的地方替换成对应常量，然后将R$styleable.class以外的所有R.class删除掉
BTW:类似瘦包的思路：Facebook redex(不是使用asm)




## apk资源加载以及activitythread相关的知识，或者inputevent机制

## Android 系统启动流程吗？
## 知识图谱

1. [Android初级、中级、高级、资深工程师(架构师、专家)技能图谱](https://www.jianshu.com/p/659381fcd4e5)

2. [面试官: 说一下你做过哪些性能优化?](https://juejin.im/post/6844904105438134286)
3. [面试之Android性能优化](https://www.zybuluo.com/TryLoveCatch/note/1302255)
<!--more-->

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
摘自[面试常客「Handler」的 40+ 个高频问题 Q & A 对答！](https://mp.weixin.qq.com/s?__biz=MzIxNjc0ODExMA==&mid=2247486960&idx=1&sn=9c325c52004c94f5e1a6ca80b6907962&chksm=978514d1a0f29dc77309045867f9243ed1dac77c8e3055a450553a8b84c8978cf6a4dd564939&scene=132#wechat_redirect)

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

####