---
title: 惹Java面试答案整理
date: 2020-11-09 20:50:10
categories: 面试
tags: 
    - Java
    - 面试
---
## Java引用类型原理剖析：WeakRefrence、SoftReference

## GC的时候怎么判断什么对象需要被回收？哪些是GC Roots可达结点

## 如何检测内存泄漏

## ReentranLock的tryLock原理是什么？ ⭐
[ReentranLock的tryLock原理是什么](https://mp.weixin.qq.com/s?__biz=MzI4MTA0OTIxMg==&mid=2247483680&idx=1&sn=e627b01321b034672d3d6ffc6f141407&source=41#wechat_redirect
)

## AbstractQueuedSynchronizer（AQS原理是什么？） ⭐
state变量+CLH双端Node队列
[AbstractQueuedSynchronizer之AQS](https://www.jianshu.com/p/d64b961eed35)
> 如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要用的是CLH队列的变体实现的，将暂时获取不到锁的线程加入到队列中，这个队列就是AQS的抽象表现。它将请求共享资源的线程封装成队列的结点(Node) ，通过CAS、自旋以及LockSuport.park()的方式，维护state变量的状态，使并发达到同步的效果。

## Java集合小抄
[Java后台面试 常见问题](https://www.jianshu.com/p/1acdfac2b4e4)
[《Java并发编程的艺术》笔记](https://www.jianshu.com/p/8d90dc5b341e)
[关于Java集合的小抄](https://cloud.tencent.com/developer/article/1342550)
[ashMap解析（主要JDK1.8，附带1.7出现的问题以及区别）](https://bbs.huaweicloud.com/blogs/173908)

synchronizedList是使用了同步代码块，vector是使用了同步方法，所以synchronizedList性能比vector要好
——[坑人无数的Java面试题之ArrayList](https://zhuanlan.zhihu.com/p/34301705)


## 从ReentrantLock的实现看AQS的原理及应用
[从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)

## 注解
- 自定义注解与元注解
- APT，编译时注解处理器（Annotion Process Tool）。
  > 获取所有该注解里的参数，可以达成生成新类的目的
- 插桩。自定义Gradle插件，利用Transform修改class文件
- 反射，运行时动态获取注解信息 getAnnotions
```C++
SPI：策略模式，可以通过配置文件动态加载。放到resources/META-INF/services下，可以用@AutoService注解代替
1、定义接口和接口实现类
2、创建resources/META-INF/services目录
3、在该目录下创建一个文件，文件名为接口名（带包全名），内容为接口实现类的带包全名
4、在代码中通过ServiceLoader动态加载并且调用实现类的内部方法。
```

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

4. `synchronized`不可中断性。前一个不释放，后一个也一直会阻塞或者等待。所以不能用 `interrupt()` 中断正在获取锁的线程
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

## 1.接口与抽象类区别？
## 2. java中的异常有哪⼏几类，分别怎么使用？
> [摘自地址](https://www.jianshu.com/p/4f84c14f1a22)
> 从根本上讲所有的异常都属于Throwable的子类，从大的方面讲分为Error（错误）和Exception（异常）。Eror是程序无法处理的异常，当发生Error时程序线程会终止运行。我们一般意义上讲的异常就是指的Exception。

## 6. ==和equals的区别
`==` 操作符比内存地址
`equals()` 函数比值，可以被继承
## 7. hashCode()⽅法的作⽤
一般用作`Hashmap`取值，重写`equals()`必重写`hashCode()`，并且保证对象为final不变更，出现存取时`hashCode()`不一致的问题。
通过对key的hashCode()进行hashing，并计算下标( n-1 & hash)，从而获得buckets的位置。如果产生碰撞，则利用key.equals()方法去链表或树中去查找对应的节点。

## 如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？
如果超过了负载因子(默认0.75)，则会重新resize一个原来长度两倍的HashMap，并且重新调用hash方法。

## HashMap可以接受null吗？
HashMap可以接受null

## 12. java中⼀一个字符占多少个字节？int，long，double占多少个字节？
`char` 2
`int` 4
`long/ double / float` 8


##   StringBuilder 和 StringBuffer 的区别是什么？String的引用到底有几个，放在内存的哪里？
1. `StringBuffer`是上了`synchonrize`锁的`StringBuilder`。
2. 在代码里写字符串相加，编译器会自动优化为每次`new`一个`StringBuilder`进行append，所以不要在循环里累加字符串，会生成很多`StringBuilder`
3. 字符串会加在`运行时常量池（Runtime Constant Pool）`里：	
   - JDK 1.6 及之前的版本中，常量池是分配在方法区中永久代(Parmanent Generation)内的，而永久代和 Java 堆是两个完全分开的区域。
   - 从JDK 1.7开始去永久代，字符串常量池已经被转移至 Java 堆中
4. 字符串的`intern()`可以返回常量池里的位置

> —— [【扯皮系列】一篇与众不同的 String、StringBuilder 和 StringBuffer 详解](https://www.cnblogs.com/cxuanBlog/p/13053615.html)


1. class文件中常量池保存的是字符串常量，类和接口名字，字段名，和其他一些在class中引用的常量。每个class都有一份。
2. 运行时常量池保存的是从class文件常量池构建的静态常量引用和符号引用。每个class都有一份。
3. 字符串常量池保存的是“字符”的实例，供运行时常量池引用。
> —— [JVM详解之:运行时常量池](https://www.cnblogs.com/flydean/p/jvm-run-time-constant-pool.html)

## 21. String s = new String("abc")创建了几个String Object？
如果"abc"已经存在于常量池，则只有1个；
否则，2个，包含插入常量池的一个；

> 常量池是JDK1.8里在堆上，此前是在方法区里

[new String()究竟创建几个对象?](https://blog.csdn.net/Sun1956/article/details/53161560?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

一个`new`出来的一个对象`new String(xx)`，在堆上，内部指向常量池里的内容

[深入理解Java：String](https://www.cnblogs.com/benwu/articles/5643502.html)

## 不关闭流(stream)会导致内存泄漏吗？文件流关闭的时机是怎样的，先打开先关闭还是按照依赖关系？
> 内存泄露：GC 回收对象采用GC Roots强引用可到达机制。当生命周期长的实例不合理地持有一个生命周期短的实例S，导致S实例无法被正常回收
1. 例如 `FileInputStream` 会被 `FinalizerReference` 这个类(GC Root)持有，在`FileInputStream` 的 `finalize()` 里会调用`close()`。
2. stream派生类使用的是装饰模式，这些包装类例如`BufferedWriter`会在`close()`里自动关闭掉源数据流。
3. `close()`往往会设计成可以多次调用的样子。有时候调用`close()`会报错是因为关闭方法中又调用了write等方法时会抛异常，例如`BufferedWriter`。这种时候只调用	`BufferedWriter`的`close()`即可。

所以需要主动调用`close()`并不是因为内存泄漏，而更多的是因为资源泄漏。
> It's not a memory leak as much as a file-handle leak.
> —— [Why is it good to close() an inputstream?](https://stackoverflow.com/questions/26541513/why-is-it-good-to-close-an-inputstream)
### 为什么要关闭流
![linux file system](https://asset.droidyue.com/image/2019_05/file-descriptor_table.jpg)
如上图从左至右有三张表
- file descriptor table 归属于单个进程
- global file table(又称open file table) 归属于系统全局
- inode table 归属于系统全局

每个进程可以开启的文件描述符个数是有限制的，如果不释放file descriptor，会导致应用后续依赖file descriptor的行为(socket连接，读写文件等)无法进行，甚至是导致进程崩溃。
所以不能总是依赖`finalize()`或者`gc()`去手动关掉流。要手动关闭流

> —— [未关闭的文件流会引起内存泄露么？](https://droidyue.com/blog/2019/06/09/will-unclosed-stream-objects-cause-memory-leaks/)


### 文件流的关闭时机
可以使用java 7后的AutoClose，

## 29. 在java中一个字符能否表示一个汉字：
char可以表示一个汉字，char默认两个字节；
Java默认UTF-16编码，一个汉字2个字节（UTF-8一个汉字3～4个字节）
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
[7.多线程下put和get操作导致的HashMap线程不安全问题](https://blog.csdn.net/u014590757/article/details/79509101)
可能产生的现象会是： 
- 1)put进行的data有可能丢失了 
- 2)一些通过remove(Object key)删除掉的元素(返回删除成功)又出来了。 
- 3)多线程检测到HashMap容量超过负载因子时会进行多次的resize，由于要rehash，所以消耗的性能也是巨大的。 

1. 发生在index相同的情况下，大家拿到的链头可能不是最新的，后一个会直接覆盖了前一个 
2. 而且当多条线程检测到容量超过负载因子时，会能发生多次resize。 

小结一下：remove与put都是一样的，由于大家拿到的不是最新链头，只要大家在Entry数组的index相同时(经过hash后的index)，就有可能出现后一个覆盖前一个的操作，即前一个的操作无效。 


> HashMap在put的时候，插入的元素超过了容量（由负载因子决定）的范围就会触发扩容操作，就是rehash，这个会重新将原数组的内容重新hash到新的扩容数组中，在多线程的环境下，存在同时其他的元素也在进行put操作，如果hash值相同，可能出现同时在同一数组下用链表表示，造成闭环，导致在get时会出现死循环，所以HashMap是线程不安全的

##  38. hashmap的计算hash值时为什么要用低 16 位与高 16 位进行异或运算
用hashCode的高16位和低16位是为了减少碰撞，因为高16位一直没用到：计算下标时会`&`一个` (length - 1)`的mask，`&`操作后只有低位用到了
> 1. 因为hashmap的hash函数是` tab[hashCode() & (length - 1)`，等价于`tab[ hascode() % length ]`，位运算比取模运算的效率要高很多。
> 2. 同时length永远为**2的幂次**，所以`(length - 1)`是一群低位为1的mask。
## 39. ConcurrentHashMap在JDK1.8的改动是什么？
- 去掉了Segment（继承ReentrantLock，只锁部分table）
  > 当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁
- 改为用synchronizied只锁住table，即每个红黑树/链表头，减少锁粒度

> 1. Segment下，每个节点都需要通过继承AQS来获得同步支持。但并不是每个节点都需要获得同步支持的，只有链表的头节点（红黑树的根节点）需要同步，这无疑带来了巨大内存浪费 
> -- 摘自[ConcurrentHashMap(JDK1.8)为什么要放弃Segment](https://my.oschina.net/liuxiaomian/blog/880088)
> 
> 2. Segment数组一旦初始化以后，是不可以扩容的


## ConcurrentHashMap是怎样计数和删除的
- 比如[谈谈ConcurrentHashMap1.7和1.8的不同实现](https://www.jianshu.com/p/e694f1e868ec)中有说到： **size()计算方式不同**
   1. JDK1.8中使用一个volatile类型的变量baseCount记录元素的个数，当插入新数据或则删除数据时更新baseCount。
   2. > 实现比1.7简单多，因为元素个数保存baseCount中，部分元素的变化个数保存在CounterCell数组中
   3. JDK 1.7的计算方式是先不加锁，如果多次计算结果相同，则说明无修改；如果多次计算结果都不同，则给每个Segment进行加锁，再计算一次元素的个数；但是**已经计算过的Segment可能会被修改**

- 写时复制的ConcurrentHashMap的[clear方法是弱一致性的](http://ifeve.com/concurrenthashmap-weakly-consistent/)，因为是不同桶结点在清理时临时加锁，所以已经被清理过的段可能会被添加新内容很正常。故现象为clear完后，里面有其他地方新加的数据。原理[HashMap? ConcurrentHashMap? 相信看完这篇没人能难住你！](https://crossoverjie.top/2018/07/23/java-senior/ConcurrentHashMap/)和[Java 8 ConcurrentHashMap 源码解读](https://swenfang.github.io/2018/06/03/Java%208%20ConcurrentHashMap%20%E6%BA%90%E7%A0%81%E8%A7%A3%E8%AF%BB/)

## HashMap和LinkedHashMap的实现原理，LRUCache的实现原理？

## 双向链表的实现的过程

## Glide LRUCache实现的过程

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

## 如果自定义一个String类，会怎么加载？自定义的String类会编译成功吗？

## kotlin const 和 val 的区别

## 泛型机制讲一下
> Java中的泛型，只在编译阶段有效。在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦出，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法。也就是说，泛型信息不会进入到运行时阶段
1. Java的泛型通过类型擦除实现，即泛型类型参数在编译后会被替换为原始类型（如Object）。例如，List<String>在字节码中会退化为List，类型信息仅在编译时保留。
2. 无法重载泛型方法（如method(List<Integer>)和method(List<Double>)会被擦除为相同签名）。
3. 无法通过反射获取运行时泛型类型（需借助TypeToken或第三方库）。
4. 泛型类无法直接在静态方法中使用其类型参数（如T），静态方法的加载时机：静态方法属于类而非实例，会在类加载时初始化。此时，泛型类的类型参数尚未确定（需实例化时指定），导致静态方法无法访问泛型参数T的具体类型。
	- 类型擦除的影响：即使静态方法试图使用T，编译后也会被替换为Object，导致类型信息丢失，无法保证类型安全。
```Java
// 静态 create 泛型方法. 
public static <T> Pair<T> create(T first, T last) {
	// 泛型方法的类型参数（如<T>）是方法自身的参数，与泛型类的类型参数无关。编译器通过调用时的参数推断类型，确保类型安全。
    return new Pair<T>(first, last);
}
```

## 泛型的常见问题，例如：泛型的通配符与边界
1. 无界通配符<?>：表示未知类型，仅支持读取（如List<?> list），不能写入（避免类型不匹配）。
2. 协变（<? extends T>）：允许读取子类型数据，如List<? extends Number>可接受List<Integer>，但不能添加元素。
3. 逆变（<? super T>）：允许写入父类型数据，如List<? super Integer>可接受List<Number>，但不能读取。
4. 边界限制：通过<T extends Number>约束类型参数，确保方法内可调用T的特定方法（如intValue）。
5. List、List、List<? extends Object>的区别
	- List<T>：类型未知，只能读取（add需类型匹配）。
	- List<Object>：可添加任何对象。
	- List<? extends Object>：与List<?>等价，但明确表示上界。
6. 为什么泛型不能有基本类型？
7. 类型擦除后需转换为Object，而基本类型无法自动装箱到Object，会导致编译错误。
8. Java的泛型不支持逆变和协变,只是能够实现逆变和协变。泛型与继承的关系若Dog extends Animal<String>，则Dog不能视为Animal<Dog>的实例，因泛型不支持协变。
```Java
	//数组支持协变。这里支持里氏替换原则：子类对象能够替换父类对象，而程序逻辑不变。
	Number[] n = new Integer[10];
	//编译不通过，泛型不支持协变。但可以使用通配符(Wildcard)模拟协变
	//List<Number> ln = new ArrayList<Integer>();//报错
	//Type mismatch: cannot convert from ArrayList<Integer> to List<Number>

	// Number的子类型都可以是泛型参数类型
	List<? extends Number> ln = new ArrayList<Integer>();
	//Integer的父类型(包括Integer)都可以是泛型参数类型
	List<? super Integer>  li = new ArrayList<Number>();
	/* 
	1.Integer是Number的子类型   √
	2.ArrayList<Integer>是List<Integer>的子类型  √
	3.Integer[] 是Number[] 的子类型   √
	4.List<Integer> 是List<Number>的子类型  ×
	5.List<Integer> 是List<? extends Integer>的子类型  ×
	6.List<Integer> 是List<? super Integer>的子类型  ×
	*/
```

```Json
```

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
> - Q：　比如 `method(String s)` `method(Object o)` 两个方法，调用`method(null)`会出现什么情况
> - A： 重载永远会匹配更精确的那一个，Object是String的超类，所以会匹配`method(String s)`。如果同时有`method(CharactSequence s)`，它们都是Object的子类，编译器不知道谁更精确了，就会编译报错。可以显示强转型到想要的重载方法里。比如`method((Object) null)`

方法调用就是指通过 .class 文件中方法的符号引用，确认方法的直接引用的过程，这个过程有可能发生在加载阶段，也有可能发生在运行阶段。

1. `重载` （不同参数、返回值）的方法在加载阶段就确定了方法的直接引用。
> 有一些方法是在加载阶段就已经确定了方法的直接引用，比如：静态方法、私有方法、实例构造器方法，这类方法的调用称为 `解析`；


2. `重写`(`Override`) 的方法需要具体到对象的实际类型，所以需要特定的 `Java` 字节码 `invokevirtual` 去确定合适的方法
## 类的生命周期是什么？加载器什么时候会被unload？
类的生命周期就是从类的加载到类实例的创建与使用，再到类对象不再被使用时可以被GC卸载回收。
由java虚拟机自带的三种类加载器加载的类在虚拟机的整个生命周期中是不会被卸载的，只有用户自定义的类加载器所加载的类才可以被卸载。

## 说说Java内存回收
年轻代：主要用来存放新创建的对象，年轻代分为eden区和两个Survivor区。大部分对象在Eden区中生成。当Eden区满时，还存活的对象会在两个Survivor区交替保存，达到一定次数的对象会晋升到老年代
## 怎么知道哪些变量内存泄漏了？类什么时候会被回收？
可达性分析算法：一个对象到GC Roots没有任何引用链相连，从GC Roots到这个对象不可达时，则证明此对象是不可用的。
### 可作为GC Roots的对象包括下面几种：
1. 虚拟机栈（栈帧中的本地变量表）中引用的对象。
2. 本地方法栈中JNI（即一般说的Native方法）引用的对象。
3. 方法区中类静态属性引用的对象。
4. 方法区中常量引用的对象。

除了常见的Java类静态属性引用的对象、常量引用的对象、栈帧中的局部变量引用的对象，我们还需要关注调用JNI代码时传递的参数、在JNI代码中创建的全局Java对象、调用synchronized和wait的对象，它们是日常开发中可能造成内存泄漏的典型例子

> 为了避免内存泄漏，我们需要谨慎持有Activity、Fragment、Context、View等复杂对象的引用，尽可能地做到及时释放。如果业务场景比较复杂，无法确定释放时机，可以通过SoftReference和WeakReference缓解问题；在C/C++代码中，我们需要尽量少地持有Java对象的引用，如果的确需要使用较长时间，优先使用JNIEnv#NewLocalRef和JNIEnv#NewWeakGlobalRef

- 使用软引用或者弱引用，把引用的释放时机延迟到GC执行时，引用持有时间过久的问题没有被彻底解决，只是最低程度地保证了引用会被释放
> - 有一个强引用非常容易被忽略：正在执行的函数中的局部变量。我们在使用软引用/弱引用时，有一条必经之路：调用get方法获取原始对象，然后创建一个局部变量引用原始对象
> - App中一旦创建了无限循环的动画且退出页面后没有停止，就会导致View泄漏。在使用属性动画时需要注意及时调用cancel

- 单例模式方便的原因是有一个静态对象始终存在于内存中，如果没有及时释放，很容易导致内存泄漏。
> 1. 让所有单例类实现一个统一的生命周期接口（可以简单一点，只有销毁方法）​。
> 2. 在销毁方法里释放单例对象持有的所有资源。
> 3. 在单例的构造函数中把静态对象注册到生命周期相关的管理器中。
> 4. 当管理器生命周期结束后，调用所有接口的销毁方法。

- 根据内存情况进行业务降级

### 对方法区的回收
永久代的垃圾收集主要回收两部分内容：废弃常量和无用的类。
判定一个常量是否是“废弃常量”比较简单，而要判定一个类是否是“无用的类”的条件则相对苛刻许多。类需要同时满足下面3个条件才能算是“无用的类”：
该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例。
加载该类的ClassLoader已经被回收。
该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。## jvm的内存模型和java的内存模型
### Java内存模型（Java Memory Model）与并发问题相关
1. 保证共享内存的正确性(可见性、有序性、原子性)
2. 缓存一致性：多线程的场景中，每个核都至少有一个L1 缓存。所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中是用到的变量的主内存副本拷贝。
3. **volatile**修改后将新值同步回主内存，在变量读取前从主内存刷新变量值
4. **synchronized**关键字保证同一时刻只允许一条线程操作，同时保证可见性、有序性、原子性，通过monitorenter和monitorexit的monitor锁来确保。
5. **final**关键字则不可修改
##### 什么是内存屏障？是为了解决什么问题？
刷新缓存或禁用指令重排序指令，解决不同线程并发读写主存同一个位置时**缓存一致性**的问题
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201104114138160.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xlaTM5NjYwMTA1Nw==,size_16,color_FFFFFF,t_70#pic_center)

#### JVM的内存模型
![jvm的内存模型](https://img-blog.csdnimg.cn/20201104110937322.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xlaTM5NjYwMTA1Nw==,size_16,color_FFFFFF,t_70#pic_center)


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

## Java的泛型是如何工作的 ? 什么是类型擦除 ?如何工作？
好处： 类型安全，提供编译期间的类型检测

1. 类型检查：在生成字节码之前提供类型检查
2. 类型擦除：所有类型参数都用他们的限定类型替换，包括类、变量和方法（类型擦除）
3. 如果类型擦除和多态性发生了冲突时，则在子类中生成桥方法解决
4. 如果调用泛型方法的返回类型被擦除，则在调用该方法时插入强制类型转换


## 如何创建一个线程
第一种：继承Thread类
第二种：实现Runnable接口
第三种：实现Callable接口


第四种：Executor框架来创建线程池
	```Java
		ExecutorService executorService = Executors.newCachedThreadPool();
		for(int i = 0; i < 5; i++){
			executorService.execute(new MyThreadFour());
		}
		executorService.shutdown();
	```

## 25. 多态：
多态，同一个方法在不同的对象上可以表现出不同的行为
编译时多态（静态链接）：方法重载。方法重载允许在同一个类中定义多个方法，方法名称相同但参数列表不同（参数数量或类型不同）。
运行时多态（动态链接）：方法重写。方法重写允许子类重新定义父类中的方法，运行时根据对象的实际类型执行相应的方法。在运行阶段根据对象的实际类型决定调用哪个方法。
> [方法重写的本质](https://www.cnblogs.com/ding-dang/p/13051143.html)
> 在Java源文件被编译到字节码文件中时，所有的变量和方法引用都作为符号引用(Symbolic Reference )保存在class文件的常量池里。比如，描述一个方法调用其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用
> ![JVM图](https://img2020.cnblogs.com/blog/1717141/202006/1717141-20200605173115350-69751287.png)

## Kotlin 中 var、val、const 关键字区别
final 可以修饰类、变量和方法。修饰类代表这个类不可被继承。修饰变量代表此变量不可被改变。修饰方法表示此方法不可被重写 (override)。
Java 中可以使用 static final 来定义常量，这个常量会存放于全局常量区，这样编译器会针对这些变量做一些优化，例如，有三个字符串常量，他们的值是一样的，那么就可以让这个三个变量指向同一块空间。我们还知道，局部变量无法声明为 static final，因为局部变量会存放在栈区，它会随着调用的结束而销毁。
Kotlin 引入一个新的关键字 const 来定义常量，但是这个常量跟 Java 的 static final 是有所区别的，如果它的值无法在编译时确定，则编译不过，因此 const 所定义的常量叫编译时常量。const 只能修饰没有自定义 getter 的 val 属性，值必须在编译时确定。


## 8. NIO是什什么？适⽤用于何种场景？
## 9. Hashmap实现原理理？如何保证HashMap线程安全？
## 10. jvm内存结构？为什什么需要GC？
## 11. NIO模型，select/epoll的区别，多路复⽤的原理？
## 13. 创建⼀一个类的实例例都有哪些⽅方法？
## 14. final/finaly/finalize区别？
## 15. Session/Cookie区别？
## 16. String/StringBuffer/StringBuilder的区别以及实现？
## 17. Servlet⽣生命周期
## 18. 如何⽤用java分配⼀一段连续的1G的内存空间?需要注意些什什么？
## 19. Java有⾃己的内存回收机制，但为什么还存在内存泄漏的问题呢？
## 20. 什什么是java序列化，如何实现java序列化（写⼀一个例例⼦子）

## 3. 常⽤用的集合类有哪些？⽐如list如何排序？
## 4. ArrayList和LinkedList内部实现大致是怎样的？他们之间的区别和优缺点？
## 5. 内存溢出是怎么回事？举个例子。

## 22. 静态对象：
## 23. final关键字：
## 24. HashMap与HashTable的区别：
## 26. 集合删除：
## 27. 参数传递与引⽤用传递：
## 28. hash冲突：
## 30. 一致性hash：
## 31. java反射机制
## 32. 幂等的处理理方式：


### 线程运行状态
### wait() 和 sleep() 的区别是什么？
wait()会释放锁，让出CPU；
wait()必须在同步块里执行，
wait()是Object的方法；
wait()只能由notify()等函数唤醒；

Synchronized是公平锁吗？不是

线程run()和start()的区别？

线程能从运行态到就绪态吗？不能，就绪态一旦退出就不返回

### dpi 
一张100dp图片，在分辨率分别为320*480和720*1280的手机上，最后实际的大小是什么？
1. 240×320的屏幕是低密度120dpi，即ldpi；
2. 320×480的屏幕是中密度160dpi，即mdpi；  通常160dpi就是基准的大小
3. 480×800的屏幕是高密度240dpi，即hdpi；  是160dpi的1.5倍数
4. 720×1280的屏幕是超高密度320dpi，即xhdpi；
5. 1080×1920的屏幕是超超高密度480dpi，即xxhdpi。