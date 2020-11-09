---
title: Android面试答案整理
date: 2020-11-09 18:48:47
tags:
    - 面试
categories: 面试
---

OKHTTP(Socket 连接池、Http缓存、责任链)、Retrofit(动态代理)

[Okhttp3 总结研究 （面试）](https://blog.csdn.net/u012881042/article/details/79759203?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param)

说说组件化和插件化，热更新技术原理

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

### 



Android ANR的产生原因，如何定位ANR



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



