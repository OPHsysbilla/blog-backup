---
title: 到底有没有在用心学ndk啊
date: 2019-08-07 16:06:10
tags:
---

1. JNIEnv*env 是一个线程对应一个env，线程间不可以共享同一个env变量。[JNI中Fatal signal 11 (SIGSEGV), code 1的错误](https://blog.csdn.net/viking_xhg/article/details/78727273)
2. 可以使用Android自带的addr2line来分析native的行信息[Android通过addr2line工具分析native crash log](https://juejin.im/post/5c3c6ec9f265da616a47df88)
3. 注意lock后再wait该锁的问题[JVM故障分析系列之五：常见的Thread Dump日志案例分析](https://www.javatang.com/archives/2017/10/26/08572060.html)