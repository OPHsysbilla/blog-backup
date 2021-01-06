---
title: 惹Java面试答案整理
date: 2020-11-09 20:50:10
categories: 面试
tags: 
    - Java
    - 面试
---


## ThreadLocal是干嘛的，作用是什么 
<!--more-->

## 注解
- 自定义注解与元注解
- APT，编译时注解处理器（Annotion Process Tool）。
  > 获取所有该注解里的参数，可以达成生成新类的目的
- 插桩。自定义Gradle插件，利用Transform修改class文件
- 反射，运行时动态获取注解信息 getAnnotions

> Retrofit使用到了注解。
> 1. Retrofit使用动态代理模式实现定义的网络请求接口。
> 2. 在重写invoke方法的时候构建了一个ServiceMethod对象，再使用参数的注解分析得到网络请求方式httpMethod，拼接成一个省略域名的URL
```JAVA
	// 定义网络请求的API接口形如：
	interface GithubApiService { 
		@GET("users/{name}/repos")
		Call<ResponseBody> searchRepoInfo(@Path("name") String name);
	}
```

## Java IO
 Java中字符是采用Unicode标准，一个字符是16位

## 如何检测是否被Hook动态注入反射修改值？
可以保留该对象的hashCode，每次调用的时候比对该对象的值

### Java 双亲委派机制的破坏—SPI机制
SPI(Service Provider Interface 服务提供者接口)机制在**运行时**才来加载具体的接口实现类，按照SPI规范指定接口实现即可加载自定义的接口实现类，实现策略模式和热拔插效果。这是一种“面向接口编程＋策略模式＋配置文件”组合实现的动态加载机制
> 在运行时修改资源文件可以达到加载不同实现类的功能。但是这并不是SPI的主要用法，SPI是服务提供接口，主要是给第三方用的，例如：只提供InterfaceSpi接口，不提供实现。第三方需要使用该接口时自己提供实现，然后配置在配置文件中即可。

## synchronized的底层原理是什么？

> [Java synchronized原理总结](https://zhuanlan.zhihu.com/p/29866981)
内存可见性：
- 对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值
- 对一个变量执行unlock操作之前，必须先把此变量同步回主内存中（执行store和write操作）

锁的内存语义：
- 当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中
- 当线程获取锁时，JMM会把该线程对应的本地内存置为无效。从而使得被监视器保护的临界区代码必须从主内存中读取共享变量

1. `synchronized`：原子性、可见性、有序性。
   > `volatile`不具备原子性
   > 可见性指一个线程的变量更新被其他线程所看到。需要插入一条类似`load addl $0x0, (%esp)`指令，变量更新值后需要将线程内存缓存刷回主内存。见内存屏障
   > 有序性通过插入内存屏障指令防止重排序，保证某些指令一定在另一些指令前完成。`happen-before`

2. `synchronized`和`ReentrantLock`都是可重入锁，都是悲观锁，都是排他锁。
   > 可重入锁：同一个线程拥有了锁仍然还可以重复申请锁。
   > 乐观锁：Java中是无锁编程的CAS算法。更新资源时先判断资源是否已经被修改，如果被修改则报错or重试。适合读操作多的场景。
   > 悲观锁：在自己用同步资源的时候一定有别人会修改它，我必须锁住同步资源。适合写操作多的场景。
   乐观锁是通过`cmpxchg`指令，去比较寄存器中的 A 和 内存中的值 V
   - 如果相等，就把要写入的新值 B 存入内存中。
   - 如果不相等，就将内存值 V 赋值给寄存器中的值 A。然后while循环进行重试，直到设置成功为止
   共享锁：读锁的共享锁可保证并发读非常高效，而读写、写读、写写的过程互斥，因为读锁和写锁是分离的。
   > `ReentrantReadWriteLock`里用一个 `int` `32bit` 高 `16bit` 表示读，低 `16bit` 表示写。除开重入的情况（同一个线程重复拿写锁，写锁+1）外，必须等写锁为0表示写锁释放后，才能拿到锁，要保证其他读线程就感知到当前写线程的操作

3. 同步块是由`monitorenter`指令进入，然后`monitorexit`释放锁
   > 在执行monitorenter之前需要尝试获取锁。第二个`monitorexit`是由编译器自动生成的，在发生异常`athrow`时处理异常然后释放掉锁

4. 不可中断性。前一个不释放，后一个也一直会阻塞或者等待。所以不能用 `interrupt()` 中断正在获取锁的线程
  
   > 中断操作`Thread.interrupt()`只是给线程的一个建议，最终怎么执行看线程本身的状态
   > - 若线程被中断前，该线程处于阻塞状态(调用了 `wait` , `sleep` , `join` 方法)，那么该线程将会立即从阻塞状态中退出，并抛出一个`InterruptedException`异常，同时，该线程的中断状态被设为false, 除此之外，不会发生任何事。
   > - 若线程被中断前，如果该线程处于非阻塞状态(未调用过wait,sleep,join方法)，那么该线程的中断状态将被设为true, 除此之外，不会发生任何事。`interrupt()`并不属于被`Thread.interrupt()`中断的阻塞番位，所以一个正在执行 `synchronized` 方法的线程被`interrupt()`并不会被中断抛出异常。
   > `Lock`的有额外方法是可以测试中断，代码里在获取锁前测试中断状态

5. `synchronized`非公平锁。`ReentrantLock`可以设置公平锁。
	- 非公平锁指会先尝试插队，插队失败再排队，可以减少唤起线程的开销，可能会饿死 线程。
	- 公平锁缺点是吞吐率低，除第一个线程外都会阻塞，唤醒线程开销大。
	- > 公平锁在获取同步状态时，会先判断当前线程是不是第一个线程
6. 不能知道该线程有没有拿到锁，`Lock`可以知道当前线程是否获取到锁
7. 重量级锁 > 轻量级锁 > 偏向锁 > 无锁，升级不可逆转。除非第一次JIT编译时锁消除，JVM自动消除。来自[不可不说的Java“锁”事](https://tech.meituan.com/2018/11/15/java-lock.html)
8. - 修饰实例方法，对当前实例对象this加锁
   - 修饰静态方法，对当前类的Class对象加锁
   - 修饰代码块，指定一个加锁的对象，给对象加锁
9. `synchronized`锁存储在对象头的`Mark Word`里。存储对象的HashCode，分代年龄和锁标志位信息。
   
   JVM堆内存中实例对象：
   ![JVM堆内存中实例对象](https://pic4.zhimg.com/80/v2-85cce26769b1d9584e4af2048880307b_1440w.jpg)
    > 多出来的1行记录的是数组长度

    > 如64位机器是`Mark Word`8个字节，如果对象指针`Klass`4个字节，**对齐填充**4个字节，共16字节

   `Mark Word`里存储的数据会随着锁标志位的变化而变化，以32位的JDK为例：
   ![Mark Word存储的数据](https://pic1.zhimg.com/80/v2-5b27b769965544fb297a12c7f162e588_1440w.jpg)

10. `Mark Word`里有
	- owner，指向当前获得锁的线程
	- EntryList，等待获取锁的线程
	- WaitList，调用wait()后释放锁的线程，需要notify来使得线程能进EntryList

11. 偏向锁比较`Thread ID`，同一线程执行同步资源时自动获取。简单地测试一下对象头的`Mark Word`里是否存储着指向当前线程的偏向锁
   - 只需要在置换`ThreadID`的时候依赖一次CAS原子指令
   - 大多数情况下锁不存在多线程竞争且总是由同一线程获得
   - 获取偏向锁失败，表示至少有其他线程曾竞争过偏向锁。
   - 偏向锁不会主动释放，等到竞争出现才释放偏向锁：当到达不执行字节码的安全点时，才会暂停拥有偏向锁的线程A。
   	 > 如果拥有偏向锁的线程A活着，A撤销偏向锁，升级为轻量锁，正在竞争的其他线程会进入自旋等待轻量锁。
	 > 如果线程A不活跃，对象锁变为无锁状态，重新偏向

12. 轻量级锁：在线程近乎交替执行同步块时提高性能。CAS和有限自旋，避免线程阻塞和唤醒。
   - 在当前线程A的栈帧建立一份`Lock Record`，复制一份线程A要获取的锁对象`Mark Word`到这个空间里。
   - 自旋锁CAS尝试改变要获取的锁对象`Mark Word`指向`Lock Record`，且`Lock Record`的`owner`指向线程A。
   - 如果复制替换失败了，判断`Mark Word`是否指向当前栈帧，有说明已经获得轻量锁。如果没有，需要自适应自旋等
   - 若自旋失败，膨胀为重量级锁。释放轻量锁的同时，唤醒被挂起的线程角逐重量级锁。

13. 重量级锁，没有获取锁的线程阻塞等待。`Mark Word`中存储的是指向互斥锁的指针，等待锁的线程都会进入阻塞状态
	> 需要系统调用，涉及用户态和内核态的转换，通过`Monitor`来实现线程同步，`Monitor`是依赖于底层的操作系统的 `Mutex Lock`（互斥锁）来实现的。
	> `Monitor`是线程私有的数据结构，每个线程都有可用`monitor record`列表，同时还有一个全局的可用列表。
	> 每一个被锁住的对象都会和一个`monitor`关联
	> `monitor`的`Owner`字段存放拥有该锁的线程的Thread ID 

15. `notify`/`notifyAll`方法调用后，并不会马上释放监视器锁，而是在相应的`synchronized(){}`/`synchronized`方法执行结束后才自动释放锁

## 什么是自旋锁
自旋锁：自旋锁不会阻塞线程，只是while来让线程「等一会儿」再检查资源是否可用。
   > - 多数情况下，线程持有锁的时间都不会太长，直接挂起线程得不偿失。自旋锁是做几个空循环等待锁，通过有限的自旋次数避免因内核态切换，会自适应判断死循环直到超出阈值。
   > - 死循环是为了防止线程被挂起。异常事件和设备的中断也会发生内核态和用户态的切换。实际上用户态内核态切换耗时主要是在保存和恢复TSS任务状态段(`task state segment`)。
   > - 自适应自旋锁的自旋时间由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定
## 如何停止一个线程？只使用volatile可以吗？volatile内存屏障是什么？
1. 线程处于争取锁的状态时，使用`Thread.interrupt()`是无法中断线程的。如尝试获取`synchronized`锁的线程无法中断停止。
2. 线程处于阻塞状态时，使用`Thread.interrupt()`方式中断该线程，抛出一个`InterruptedException`。中断状态将会被复位(由中断状态改为非中断状态)。
    > 来自[JavaDoc: How do I stop a thread](https://docs.oracle.com/javase/1.5.0/docs/guide/misc/threadPrimitiveDeprecation.html)：如果一个线程在休眠中被interrupt了，那么它的中断标记位会重置为false，并抛出一个interruptedException的异常。所以有两种最佳的处理方式：
	> 1. 方法里try-catch然后再进行一次interrupt，将中断标记位设置为true，这样调用的方法仍然能捕捉到中断信号。
	> 2. 发现中断标记位置为true后直接方法签名上直接抛出去，这样外层一层一层往出抛，最后run()里处理这个异常。
3. 非阻塞状态的线程需要我们手动进行中断检测并结束程序。使用interrupt()更改子线程的标志位，并且在子线程的while循环里判断isInterrupted()状态 。
	```JAVA 
		public void run(){
			while(true){
				//判断当前线程是否被中断
				if (this.isInterrupted()){ 
					break;
				}
			} 
		} 
		// other place ..
		tread.interrupt();
	```
4. 非阻塞状态的线程也可以用`interrupted()`
	```JAVA
	public void run(){
		try {
		//判断当前线程是否已中断,注意interrupted方法是静态的,执行后会对中断状态进行复位
		while (!Thread.interrupted()) {
			TimeUnit.SECONDS.sleep(2);
		}
		} catch (InterruptedException e) {

		}
	}
	```

### 不可以单独用volatile修饰的bool标志位退出线程循环
> 来自[JavaDoc: How do I stop a thread](https://docs.oracle.com/javase/1.5.0/docs/guide/misc/threadPrimitiveDeprecation.html)。同时`volitale`需要和`synchronized`同用。
单独使用一个`volatile`修饰的bool标志位退出循环还是会有问题，当while循环里被阻塞的时候（比如`BlockingQueue的put函数（使用ReenterLock）`），此时线程已经阻塞住是无法走到while判断bool标志位的地方的。

### volatile内存屏障是什么
volatile赋值后会多执行一个`load addl $0x0, (%esp)`，相当于插入一个内存屏障：指令重排序时不能把后面的指令重排序到内存屏障之前的位置
> 读的时候从主内存而非缓存读，写的时候有任何修改要同步更新主内存
```JAVA
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
           Main Memory
│                               │
   ┌───────┐┌───────┐┌───────┐
│  │ var A ││ var B ││ var C │  │
   └───────┘└───────┘└───────┘
│     │ ▲               │ ▲     │
 ─ ─ ─│─│─ ─ ─ ─ ─ ─ ─ ─│─│─ ─ ─
      │ │               │ │
┌ ─ ─ ┼ ┼ ─ ─ ┐   ┌ ─ ─ ┼ ┼ ─ ─ ┐
      ▼ │               ▼ │
│  ┌───────┐  │   │  ┌───────┐  │
   │ var A │         │ var C │
│  └───────┘  │   │  └───────┘  │
   Thread 1          Thread 2
└ ─ ─ ─ ─ ─ ─ ┘   └ ─ ─ ─ ─ ─ ─ ┘
```

## 线程池里的参数是什么意思
以下内容参考[Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
1. 首先检测线程池运行状态，如果不是RUNNING，则直接拒绝，线程池要保证在RUNNING的状态下执行任务。
2. 如果workerCount < corePoolSize，则创建并启动一个线程来执行新提交的任务。
3. 如果workerCount >= corePoolSize，且线程池内的阻塞队列未满，则将任务添加到该阻塞队列中。
4. 如果workerCount >= corePoolSize && workerCount < maximumPoolSize，且线程池内的阻塞队列已满，则创建并启动一个线程来执行新提交的任务。
5. 如果workerCount >= maximumPoolSize，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默认的处理方式是直接抛异常。
<!--more-->

![图4 任务调度流程](https://p0.meituan.net/travelcube/31bad766983e212431077ca8da92762050214.png)

> 例如：线程池总大小为128（maximumPoolSize），还有一个缓冲队列（sWorkQueue，缓冲队列可以放10个任务），当尝试去添加第139个任务时程序崩溃。当线程池中的数量大于corePoolSize，缓冲队列已满，并且线程池中的数量小于maximumPoolSize，将会创建新的线程来处理被添加的任务

## 什么是守护进程？守护进程结束的时候一定会调用final吗
守护进程需要在开启前设置
JVM退出时，不必关心守护线程是否已结束，所以final可能不会被调用
所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出，但不是立马退出
> 注意：守护线程不能持有任何需要关闭的资源，例如打开文件等，因为虚拟机退出时，守护线程没有任何机会来关闭文件，这会导致数据丢失
## 你可能知道的Java知识？
1. clone()只拷贝第一层，[只复制第一层](https://blog.csdn.net/zhangjg_blog/article/details/18369201)
2. 类比[处理器 - 缓存 - 内存]的三级层次，[线程 - 工作内存 - 主内存]。其中线程互相不可见彼此的工作内存，并通过主内存来共享交流。
3. volatile关键字的作用是：1、防止指令重新排序；2、保证每个线程在拿到它的那一瞬间前被刷新，拿到的是主内存中的最新值。但不能说volatile修饰了变量后就实现了线程安全：例如i++，i++这个操作本身不是原子性的。线程在拿到i时是可以保证是最新值，但是在之后加一再写回去的这两步中，其他线程可能已经修改了主内存里i的值，最终导致最后写回去的值覆盖了其他线程的操作。
4. Java中锁的分类有自旋锁、可重入锁、阻塞锁等等分类，其中能够造成线程卡死的锁，只有阻塞锁。
5. 写时复制的ConcurrentHashMap的[clear方法是弱一致性的](http://ifeve.com/concurrenthashmap-weakly-consistent/)，因为是不同桶结点在清理时临时加锁，所以已经被清理过的段可能会被添加新内容很正常。故现象为clear完后，里面有其他地方新加的数据。原理[HashMap? ConcurrentHashMap? 相信看完这篇没人能难住你！](https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/)和[Java 8 ConcurrentHashMap 源码解读](https://swenfang.github.io/2018/06/03/Java%208%20ConcurrentHashMap%20%E6%BA%90%E7%A0%81%E8%A7%A3%E8%AF%BB/)

## 1.接口与抽象类区别？
## 2. java中的异常有哪⼏几类，分别怎么使用？
> [摘自地址](https://www.jianshu.com/p/4f84c14f1a22)
> 从根本上讲所有的异常都属于Throwable的子类，从大的方面讲分为Error（错误）和Exception（异常）。Eror是程序无法处理的异常，当发生Error时程序线程会终止运行。我们一般意义上讲的异常就是指的Exception。

## 3. 常⽤用的集合类有哪些？⽐如list如何排序？
## 4. ArrayList和LinkedList内部实现大致是怎样的？他们之间的区别和优缺点？
## 5. 内存溢出是怎么回事？举个例子。
## 6. ==和equals的区别
`==` 操作符比内存地址
`equals()` 函数比值，可以被继承
## 7. hashCode()⽅法的作⽤
一般用作`Hashmap`取值，重写`equals()`必重写`hashCode()`，并且保证对象为final不变更，出现存取时`hashCode()`不一致的问题
## 8. NIO是什什么？适⽤用于何种场景？
## 9. Hashmap实现原理理？如何保证HashMap线程安全？
## 10. jvm内存结构？为什什么需要GC？
## 11. NIO模型，select/epoll的区别，多路复⽤的原理？
## 12. java中⼀一个字符占多少个字节？int，long，double占多少个字节？
`char` 2
`int` 4
`long/ double / float` 8

## 13. 创建⼀一个类的实例例都有哪些⽅方法？
## 14. final/finaly/finalize区别？
## 15. Session/Cookie区别？
## 16. String/StringBuffer/StringBuilder的区别以及实现？
## 17. Servlet⽣生命周期
## 18. 如何⽤用java分配⼀一段连续的1G的内存空间?需要注意些什什么？
## 19. Java有⾃己的内存回收机制，但为什么还存在内存泄漏的问题呢？
## 20. 什什么是java序列化，如何实现java序列化（写⼀一个例例⼦子）
## 21. String s = new String("abc")创建了几个String Object？
如果"abc"已经存在于常量池，则只有1个；
否则，2个，包含插入常量池的一个；

> 常量池是JDK1.8里在堆上，此前是在方法区里

[new String()究竟创建几个对象?](https://blog.csdn.net/Sun1956/article/details/53161560?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

一个`new`出来的一个对象`new String(xx)`，在堆上，内部指向常量池里的内容

[深入理解Java：String](https://www.cnblogs.com/benwu/articles/5643502.html)
## 22. 静态对象：
## 23. final关键字：
## 24. HashMap与HashTable的区别：
## 25. 多态：
## 26. 集合删除：
## 27. 参数传递与引⽤用传递：
## 28. hash冲突：
## 29. 在java中一个字符能否表示一个汉字：
char可以表示一个汉字，char默认两个字节；
Java默认UTF-16编码，一个汉字2个字节（UTF-8一个汉字3～4个字节）
## 30. 一致性hash：
## 31. java反射机制
## 32. 幂等的处理理方式：
## 33. hashmap在jdk1.8中的改动?
1. 冲突时大于8个元素时由链表改为了红黑树
2. `put`时是先插入，再判断是否要扩容
3. `resize()`时是尾插法，JDK1.7头插法多线程会导致环形链表Infinite Loop
4. 扩容后新位置的计算方式变为 ` tab[hashCode() & (length - 1)`，只用计算多出来的那一位二进制位；因为扩容每次都是2倍大小，ta数组长度保证是2的幂次。JDK1.7的扩容位置计算是重新hash一遍
> 图文并茂的改动详情见：
> - [美团面试题：Hashmap的结构，1.7和1.8有哪些区别，史上最深入的分析](
https://blog.csdn.net/qq_36520235/article/details/82417949?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)
> - [史上最详细的 JDK 1.8 HashMap 源码解析](https://blog.csdn.net/v123411739/article/details/78996181)
## 34. java 8 流式使⽤用：
## 35. java域的概念：
> 域（Field），Java里指代的是变量，如类的静态变量和成员变量。初始化的先后顺序
## 36. jdk1.8中ConcurrentHashMap size⼤于8时会转化成红⿊黑树，请问有什么作⽤，如果通过remove操作，size⼩于8了了，会发⽣什么？
> 1. 红黑树可以把链表的时间复杂度O(N)操作转化为O(logN)，查找效率更高；
> 2. HashMap删除元素时，红黑树树中元素小于UNTREEIFY_THRESHOLD = 6时，**会进行树转为链表**。
> `static final int UNTREEIFY_THRESHOLD = 6;`

## 37. Hashmap为什么是线程不安全的？在多线程情况下扩容会出现 CPU 接近 100%的情况吗？是因为出现了死循环吗？
HashMap在put的时候，插入的元素超过了容量（由负载因子决定）的范围就会触发扩容操作，就是rehash，这个会重新将原数组的内容重新hash到新的扩容数组中，在多线程的环境下，存在同时其他的元素也在进行put操作，如果hash值相同，可能出现同时在同一数组下用链表表示，造成闭环，导致在get时会出现死循环，所以HashMap是线程不安全的

##  38. hashmap的计算hash值时为什么要用低 16 位与高 16 位进行异或运算
用hashCode的高16位和低16位是为了减少碰撞，因为高16位一直没用到：计算下标时会`&`一个` (length - 1)`的mask，`&`操作后只有低位用到了
> 1. 因为hashmap的hash函数是` tab[hashCode() & (length - 1)`，等价于`tab[ hascode() % length ]`，位运算比取模运算的效率要高很多。
> 2. 同时length永远为**2的幂次**，所以`(length - 1)`是一群低位为1的mask。
## 39. ConcurrentHashMap在JDK1.8的改动是什么？
- 去掉了Segment（继承ReentrantLock，只锁部分table）
- 改为用synchronizied只锁住table，即每个红黑树/链表头，减少锁粒度

> 1. Segment下，每个节点都需要通过继承AQS来获得同步支持。但并不是每个节点都需要获得同步支持的，只有链表的头节点（红黑树的根节点）需要同步，这无疑带来了巨大内存浪费 
> -- 摘自[ConcurrentHashMap(JDK1.8)为什么要放弃Segment](https://my.oschina.net/liuxiaomian/blog/880088)
> 
> 2. Segment数组一旦初始化以后，是不可以扩容的

- 比如[谈谈ConcurrentHashMap1.7和1.8的不同实现](https://www.jianshu.com/p/e694f1e868ec)中有说到： **size()计算方式不同**
   1. JDK1.8中使用一个volatile类型的变量baseCount记录元素的个数，当插入新数据或则删除数据时更新baseCount。
   2. > 实现比1.7简单多，因为元素个数保存baseCount中，部分元素的变化个数保存在CounterCell数组中
   3. JDK 1.7的计算方式是先不加锁，如果多次计算结果相同，则说明无修改；如果多次计算结果都不同，则给每个Segment进行加锁，再计算一次元素的个数；但是**已经计算过的Segment可能会被修改**

## 40. synchronizied是可重入锁吗？和ReentrantLock区别是什么？
> ReentrantLock的锁实现是用aqs,会占用额外空间.
synchronizied是底层的jvm的线程竞争.

## 41. String为什么避免在循环里用+拼接
因为每次执行“+”操作时jvm都要`new`一个`StringBuffer`对象来处理，最后用`[StringBuffer].toString()`得到最终的值

## 42. JVM类加载机制特点
	1. 全盘负责，
	2. 双亲委派机制，先让父类加载器试图加载该类
	3. 缓存机制，一次加载后类加载存在缓存里

-  要判断两个类是否“相同”，就算包路径完全一致，但是加载他们的ClassLoader不一样，那么这两个类也会被认为是两个不同的类
## 可以加载类的时候，对字节码进行修改吗？
[Java探针-Java Agent技术-阿里面试题](https://www.cnblogs.com/aspirant/p/8796974.html)
## JVM类加载过程，什么是双亲委派机制？
> [吊打面试官-类加载器](https://zhuanlan.zhihu.com/p/138823011)
![类加载过程](https://img-blog.csdnimg.cn/20201103165654872.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xlaTM5NjYwMTA1Nw==,size_16,color_FFFFFF,t_70#pic_center)
```JAVA
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException{
        Class<?> c = findLoadedClass(name);  //查找是否加载过此类
        if (c == null) {
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);  //调用父类ClassLoader加载
                } else {
                    c = findBootstrapClassOrNull(name); //父类为null，表示为BootstrapClassLoader
                }
            } catch (ClassNotFoundException e) {
            }
            if (c == null) {  //父类查找为null，调用自己的查找
                c = findClass(name);
            }
        }
        return c;
}
```

1. 加载(`ClassLoader的loadClass()方法`)
	- 类的全限定名(如`cn.edu.hdu.test.HelloWorld.class`)读取二进制字节流
	- 将其转换为运行时数据结构(存储到**方法区**中)
	- 内存中生成一个java.lang.Class对象来作为这个类的入口
2. 连接(`Linking`)
	- 验证
		- 文件格式（是否以魔术0xCAFEBABE开头）、
		- 元数据（这个类是否有父类）、
		- 字节码、
		- 符号引用验证（确保解析动作能进行）
	- 准备
	为类中的所有**静态变量**分配内存空间，初始默认值 （`static int a = 123;`准备阶段时默认为0，而`String b = "xx";` 为null ）
		> 用final修饰的静态字段在准备阶段就被初始化为一个编译时常量表达式
	- 解析
	将常量池中所有的符号引用转为直接引用（得到类或者字段、方法在**内存中的指针或者偏移量**，以便直接调用该方法）
		> 这个阶段是唯一可以推迟在初始化之后再执行的，Class.loadClass()执行完获得的对象是连接后的对象。
		
3. 初始化
	- 没有静态块编译器可以不生产`<clinit>()函数`
	- 接口与类不同的是，执行接口的`<clinit>()`方法不需要先执行父接口的`<clinit>`()方法
	- `<clinit>`静态变量初始化顺序由语义顺序决定
	- 多线程调用不同类的`<clinit>`时JVM会保证线程安全，也因此会阻塞；可以**利用静态内部类实现线程安全的单例模式**
	
父类静态 - 父类构造函数 - 子类静态 - 子类构造函数
5. 使用
6. 卸载
	- 该类所有的实例都已经被GC
    - 加载该类的ClassLoader已经被GC。
	- 该类的java.lang.Class 对象没有在任何地方被引用，包括反射

#### 何时触发初始化

 1. 为一个类型创建一个新的对象实例时（比如new、反射、序列化） 
 2. 调静态方法
 3. 赋值或读取类或接口的静态字段
 	> 用final修饰的静态字段除外，常量会在编译阶段存入调用类的常量池中

3. 调用JavaAPI中的反射方法时（比如调用java.lang.Class中的方法，或者java.lang.reflect包中其他类的方法）
4. 初始化一个类的派生类时（超类必须提前完成初始化操作，接口例外）
5. JVM启动包含main方法的启动类时。

#### 重载和重写
方法调用就是指通过 .class 文件中方法的符号引用，确认方法的直接引用的过程，这个过程有可能发生在加载阶段，也有可能发生在运行阶段。

1. `重载` （不同参数、返回值）的方法在加载阶段就确定了方法的直接引用。
> 有一些方法是在加载阶段就已经确定了方法的直接引用，比如：静态方法、私有方法、实例构造器方法，这类方法的调用称为 `解析`；


2. `重写`(`Override`) 的方法需要具体到对象的实际类型，所以需要特定的 `Java` 字节码 `invokevirtual` 去确定合适的方法
## 类的生命周期是什么？加载器什么时候会被unload？
类的生命周期就是从类的加载到类实例的创建与使用，再到类对象不再被使用时可以被GC卸载回收。
由java虚拟机自带的三种类加载器加载的类在虚拟机的整个生命周期中是不会被卸载的，只有用户自定义的类加载器所加载的类才可以被卸载。
## 说说Java内存回收
年轻代：主要用来存放新创建的对象，年轻代分为eden区和两个Survivor区。大部分对象在Eden区中生成。当Eden区满时，还存活的对象会在两个Survivor区交替保存，达到一定次数的对象会晋升到老年代

## jvm的内存模型和java的内存模型
### Java内存模型（Java Memory Model）与并发问题相关
1. 保证共享内存的正确性(可见性、有序性、原子性)
2. 缓存一致性：多线程的场景中，每个核都至少有一个L1 缓存。所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中是用到的变量的主内存副本拷贝。
3. **volatile**修改后将新值同步回主内存，在变量读取前从主内存刷新变量值
4. **synchronized**关键字保证同一时刻只允许一条线程操作，同时保证可见性、有序性、原子性，通过monitorenter和monitorexit的monitor锁来确保。
5. **final**关键字则不可修改
##### 什么是内存屏障？是为了解决什么问题？
刷新缓存或禁用指令重排序指令，解决不同线程并发读写主存同一个位置时**缓存一致性**的问题
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201104114138160.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xlaTM5NjYwMTA1Nw==,size_16,color_FFFFFF,t_70#pic_center)

#### JVM的内存模型![jvm的内存模型](https://img-blog.csdnimg.cn/20201104110937322.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xlaTM5NjYwMTA1Nw==,size_16,color_FFFFFF,t_70#pic_center)

#### [栈帧、操作数栈和局部变量表](https://juejin.im/post/6844903941805703181)分别都是什么作用呢？
![详解栈帧](https://img-blog.csdnimg.cn/20201104111007699.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xlaTM5NjYwMTA1Nw==,size_16,color_FFFFFF,t_70#pic_center)

- 基于栈的指令集系统可以很方便的做到平台无关性(x86、arm)，即使是赋值也要执行两次出栈操作。这也是为啥Java性能比C低的原因，因为操作寄存器快比操作栈快

> JVM的指令集是基于栈而不是寄存器，基于栈可以具备很好的跨平台性（因为寄存器指令集往往和硬件挂钩），但缺点在于，要完成同样的操作，基于栈的实现需要更多指令才能完成（因为栈只是一个FILO结构，需要频繁压栈出栈）。另外，由于栈是在内存实现的，而寄存器是在CPU的高速缓存区，相较而言，基于栈的速度要慢很多，这也是为了跨平台性而做出的牺牲。

- 在编译程序代码的时候，栈帧中需要多大的局部变量表，多深的操作数栈都已经完全确定了，并且写入到方法表的Code属性中，因此一个栈帧需要多大的内存，不会受到程序运行期变量数据的影响

1. 局部变量表
	> 编译为Class文件时，方法的Code属性中的max_locals中确定了该方法所需分配的局部变量表的最大容量。
	> 每个表项容量(Variable Slot)为32位，64位的数据则占用2个表项。

2. 操作数栈(Operand Stack)，一个后入先出栈(LIFO)
	> 栈的最大深度在编译的时候写入到方法的Code属性的max_stacks数据项中

3. 动态连接
	推迟到运行期，将方法的符号引用转化为内存地址的偏移量直接引用
	> 静态解析：类加载阶段或者第一次使用时就直接转化为直接引用，这类转化称为
	
4. 方法返回地址
	- 正常退出（没有抛出任何异常）
		把当前栈帧出栈，把返回值压入调用者的操作数栈中。恢复上下文。
		> 恢复上下文 ：包括恢复上层方法的局部变量表和操作数栈，调整PC计数器的值以指向方法调用指令后的下一条指令
	- 异常退出
		方法异常退出时，返回地址是通过异常处理器表确定的
		> 方法正常退出时，调用者的PC计数值可以作为返回地址，栈帧中可能保存此计数值。
	
#### 内存的可见性与java内存模型对原子性、可见性、有序性的保证机制；

## Jvm调优
1. 常用的gc算法的特点、执行过程，和适用场景。例如g1适合对最大延迟有要求的场合，zgc适用于64为系统的大内存服务中；
2. [常用的jvm参数，明白对不同参数的调整会有怎样的影响](https://zhuanlan.zhihu.com/p/38348646)，适用什么样的场景。例如垃圾回收的并发数、偏向锁设置等；
[深入理解JVM(2)——GC算法与内存分配策略](https://crowhawk.github.io/2017/08/10/jvm_2/)
3. 在编程时如何合理利用栈上分配降低gc压力、如何编写适合内联优化等代码（编译方向）
4. 线上经常full gc问题，排查过内存泄露问题（线上实际问题的排查经验）
5. 高并发低延迟的场景，如何调整gc参数尽量降低gc停顿时间，针对队列处理机如何尽可能提高吞吐率等（特定场景的jvm优化实践或者优化）
6. 解zgc高效的实现原理，了解Graalvm的特点（jvm最新技术）

