---
title: 你可能知道的Java知识
date: 2019-01-28 15:50:37
tags:
    - Java
---
1. clone()只拷贝第一层，[只复制第一层](https://blog.csdn.net/zhangjg_blog/article/details/18369201)
2. 类比[处理器 - 缓存 - 内存]的三级层次，[线程 - 工作内存 - 主内存]。其中线程互相不可见彼此的工作内存，并通过主内存来共享交流。
3. volatile关键字的作用是：1、防止指令重新排序；2、保证每个线程在拿到它的那一瞬间前被刷新，拿到的是主内存中的最新值。但不能说volatile修饰了变量后就实现了线程安全：例如i++，i++这个操作本身不是原子性的。线程在拿到i时是可以保证是最新值，但是在之后加一再写回去的这两步中，其他线程可能已经修改了主内存里i的值，最终导致最后写回去的值覆盖了其他线程的操作。
4. Java中锁的分类有自旋锁、可重入锁、阻塞锁等等分类，其中能够造成线程卡死的锁，只有阻塞锁。
5. 写时复制的ConcurrentHashMap的[clear方法是弱一致性的](http://ifeve.com/concurrenthashmap-weakly-consistent/)，因为是不同桶结点在清理时临时加锁，所以已经被清理过的段可能会被添加新内容很正常。故现象为clear完后，里面有其他地方新加的数据。原理[HashMap? ConcurrentHashMap? 相信看完这篇没人能难住你！](https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/)和[Java 8 ConcurrentHashMap 源码解读](https://swenfang.github.io/2018/06/03/Java%208%20ConcurrentHashMap%20%E6%BA%90%E7%A0%81%E8%A7%A3%E8%AF%BB/)