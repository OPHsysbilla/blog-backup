---
title: 44个Java面试答案整理
date: 2020-11-09 18:46:55
tags:
--- 

@[toc]

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
```
基于栈的指令集系统可以很方便的做到平台无关性(x86、arm)
即使是赋值也要执行两次出栈操作
这也是为啥Java性能比C低的原因
因为操作寄存器快比操作栈快
```


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
3. 在编程时如何合理利用栈上分配降低gc压力、如何编写适合内联优化等代码（编译方向）
4. 线上经常full gc问题，排查过内存泄露问题（线上实际问题的排查经验）
5. 高并发低延迟的场景，如何调整gc参数尽量降低gc停顿时间，针对队列处理机如何尽可能提高吞吐率等（特定场景的jvm优化实践或者优化）
6. 解zgc高效的实现原理，了解Graalvm的特点（jvm最新技术）

