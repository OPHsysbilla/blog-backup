---
title: Linux面试答案整理
date: 2020-11-17 15:34:56
tags:
    - 面试

---
## 在Linux下进程通信
1、匿名管道(pipe)
2、命名管道(FIFO)
3、信号(signal)
4、信号量(semaphore)
5、消息队列(message queue)
6、共享内存(share memory)
7、套接字(Socket)
 
## mmu什么
[arm-linux学习-（MMU内存管理单元）](https://www.cnblogs.com/alantu2018/p/9002309.html)

## mmap和普通文件读写对比
> —— [mmap和普通文件读写对比](https://blog.csdn.net/asdfsadfasdfsa/article/details/87719494)
常规文件系统操作（调用read/fread等类函数）中，函数的调用过程：
1. 进程发起读文件请求。
2. 内核通过查找进程文件符表，定位到内核已打开文件集上的文件信息，从而找到此文件的inode。
3. inode在address_space上查找要请求的文件页是否已经缓存在页缓存中。如果存在，则直接返回这片文件页的内容。
4. 如果不存在，则通过inode定位到文件磁盘地址，将数据从磁盘复制到页缓存。之后再次发起读页面过程，进而将页缓存中的数据发给用户进程。

> 1. 总结来说，常规文件操作为了提高读写效率和保护磁盘，使用了页缓存机制。这样造成读文件时需要先将文件页从磁盘拷贝到页缓存中，由于页缓存处在内核空间，不能被用户进程直接寻址，所以还需要将页缓存中数据页再次拷贝到内存对应的用户空间中。这样，通过了两次数据拷贝过程，才能完成进程对文件内容的获取任务。写操作也是一样，待写入的buffer在内核空间不能直接访问，必须要先拷贝至内核空间对应的主存，再写回磁盘中（延迟写回），也是需要两次数据拷贝。
> 2. 而使用mmap操作文件中，创建新的虚拟内存区域和建立文件磁盘地址和虚拟内存区域映射这两步，没有任何文件拷贝操作。而之后访问数据时发现内存中并无数据而发起的缺页异常过程，可以通过已经建立好的映射关系，只使用一次数据拷贝，就从磁盘中将数据传入内存的用户空间中，供进程使用。

### mmap优点总结
1. 对文件的读取操作**跨过了页缓存**，减少了数据的拷贝次数，用内存读写取代I/O读写，提高了文件读取效率。
2. 实现了用户空间和内核空间的高效交互方式。两空间的各自修改操作可以直接反映在映射的区域内，从而被对方空间及时捕捉。
3. 提供进程间共享内存及相互通信的方式。不管是父子进程还是无亲缘关系的进程，都可以将自身用户空间映射到同一个文件或匿名映射到同一片区域。从而通过各自对映射区域的改动，达到进程间通信和进程间共享的目的。
   >  同时，如果进程A和进程B都映射了区域C，当A第一次读取C时通过缺页从磁盘复制文件页到内存中；但当B再读C的相同页面时，虽然也会产生缺页异常，但是不再需要从磁盘中复制文件过来，而可直接使用已经保存在内存中的文件数据。

4. 可用于实现高效的大规模数据传输。内存空间不足，是制约大数据操作的一个方面，解决方案往往是借助硬盘空间协助操作，补充内存的不足。但是进一步会造成大量的文件I/O操作，极大影响效率。这个问题可以通过mmap映射很好的解决。换句话说，但凡是需要用磁盘空间代替内存的时候，mmap都可以发挥其功效。

# 异步I/O
1. 见从中断讲起的好文：[epoll 的本质是什么？](https://my.oschina.net/editorial-story/blog/3052308?p=2)
<!--more-->
2. 然后可以看[彻底搞懂epoll高效运行的原理
](https://mp.weixin.qq.com/s/FRg_lSHDiZofzTZApU6z9Q)
linux的网络IO中是不存在异步IO的，linux的网络IO处理的第二阶段总是阻塞等待数据copy完成的。真正意义上的网络异步IO是Windows下的IOCP（IO完成端口）模型

## 各种I/O模型
![各种I/O模型](https://blog-10039692.file.myqcloud.com/1500017105443_4641_1500017105783.png)
> non-blocking IO仅仅要求处理的第一阶段不block即可，而asynchronous IO要求两个阶段都不能block住
## select、poll、epoll之间的区别
`select` ，`poll` ，`epoll` 都是**I/O多路复用**的机制，都是同步I/O。都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是**阻塞**的。好处在于可以以较少的代价来同时监听处理多个IO。
> **异步I/O**则无需自己负责进行读写，其实现会负责*把数据从内核拷贝到用户空间*。  
`epoll`是`Linux`所特有，而`select`则应该是`POSIX`所规定，一般操作系统均有实现
1. `select` -> 时间复杂度`O(n)`
   - 具有O(n)的无差别轮询复杂度，同时处理的流越多无差别轮询时间就越长。
   - > 仅仅知道了有I/O事件发生了，却并不知道是哪那几个流。只能无差别轮询所有流，找出能读/写的流。
   - 通过设置或者检查存放fd标志位的数据结构来处理。单个进程能监听的fd数量有限。
   - > 单个进程所能打开的最大连接数有FD_SETSIZE宏定义，默认1024
   - 需要维护一个用来存放大量fd的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大
2. `poll` -> 时间复杂度`O(n)`
`poll` 本质上和`select`没有区别。使用`pollfd`结构而不是select的`fd_set`结构
   - 消息传递需要内核拷贝。将用户传入的数组拷贝到**内核空间**，查询每个`fd`对应的设备状态数组
   - > 如果设备就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有fd后没有发现就绪设备，则挂起当前进程。直到设备就绪或者主动超时，被唤醒后它又要再次遍历fd。这个过程经历了多次无谓的遍历。
   - **没有最大连接数的限制**，原因是它是基于**链表**来存储的。缺点：
   - 1. 大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义。
   - 2. poll如果报告了fd后，没有被处理，那么下次poll时会再次报告该fd。

3. `epoll` -> 时间复杂度`O(1)`
`epoll`可以理解为**event poll**，不同于忙轮询和无差别轮询
poll实际上是**事件驱动**（每个事件关联上fd）的，哪个流发生了I/O只回调哪个流

- 通过内核和用户空间共享一块内存来实现的。利用`mmap()`文件映射内存加速与内核空间的消息传递
- 连接数上限很大，1G内存的机器上可以打开10万左右的连接。（通过`cat /proc/sys/fs/file-max`可查看）
- `epoll`每次注册新的事件到`epoll`句柄中时（在`epoll_ctl`中指定`EPOLL_CTL_ADD`），保证了每个`fd`在整个过程中只会拷贝一次。
- > 在活跃`socket`较少的情况下，使用`epoll`没有前面两者的线性下降的性能问题，但是所有`socket`都很活跃的情况下，可能会有性能问题

epoll有它的使用场景，更适合于处理高并发的场合。
如果并发量很少，比如几百个连接这个级别，select的效率会比epoll高只有连接数大到一定长度，epoll的额外消耗才能抵消select的遍历

游戏服务器一般是`one loop per thread`
多线程服务器此文 [陈硕的 Blog](https://www.cnblogs.com/solstice/archive/2010/02/12/multithreaded_server.html)

## 零拷贝
> 摘自：[浅谈 Linux下的零拷贝机制](https://www.jianshu.com/p/e76e3580e356)

1. 传统I/O
> `DMA`(`Direct Memory Access`) 直接内存访问 ：允许外设组件将I/O数据直接传送到主存储器中并且传输不需要CPU的参与，将CPU解放出来
4次用户空间与内核空间的上下文切换，以及4次数据拷贝。
其中4次数据拷贝中包括了2次DMA拷贝和2次CPU拷贝
![传统I/O](https://upload-images.jianshu.io/upload_images/4235178-40631870dd4c58db.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/426/format/webp)
   - 通过DMA引擎将内核缓冲区中的数据传递到协议引擎(第四次拷贝: socket buffer ——> protocol engine)，这次拷贝是一个独立且异步的过程（将要发送的数据放入到了一个待发送的队列中后就可以返回了）。
2. 通过`sendfile`实现的零拷贝I/O
3次数据拷贝中包括了2次DMA拷贝和1次CPU拷贝
![sendfile()零拷贝](https://upload-images.jianshu.io/upload_images/4235178-66c23adafbfbd47f.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/418/format/webp)
3. 支持 `scatter` / `gather` 的DMA达成非CPU拷贝
2次用户空间与内核空间的上下文切换，以及2次非CPU拷贝的拷贝
![scatter / gather 的DMA](https://upload-images.jianshu.io/upload_images/4235178-df9323d3ae59b8f8.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/432/format/webp)
   - 没有数据拷贝到socket缓冲区，只有相应的描述符信息会被拷贝到相应的socket缓冲区当中。该描述符包含了两方面的信息：a)kernel buffer的内存地址；b)kernel buffer的偏移量
   - sendfile零拷贝消除了所有内核空间缓冲区与用户空间缓冲区之间的数据拷贝过程，因此应用程序无法对数据进行操作
4. 通过`mmap`实现的零拷贝I/O（可以修改数据，但比`sendfile`昂贵）
4次用户空间与内核空间的上下文切换，以及3次数据拷贝。其中3次数据拷贝中包括了2次DMA拷贝和1次CPU拷贝。
![mmap的零拷贝](https://upload-images.jianshu.io/upload_images/4235178-2700bead4cf14739.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/429/format/webp)
   - 用户空间和内核空间共享这个缓冲区，不需要将数据从内核空间到用户空间的一次拷贝
   - 用户空间可以操作缓冲区

### Java 的 NIO DirectByteBuffer
> 里面的图是[It’s all about buffers: zero-copy, mmap and Java NIO](https://medium.com/@xunnan.xu/its-all-about-buffers-zero-copy-mmap-and-java-nio-50f2a1bfc05c)的翻译版
1. `HeapByteBuffer` ：调用`ByteBuffer.allocate()`时返回
   - JVM堆分配，被GC控制优化与回收。
   - 不是页对齐(`page aligned`)的。如果要和JNI底层代码通信，JVM会拷贝这部分内容到一个页对齐的缓冲区(`aligned buffer space`)
2. `DirectByteBuffer` ：调用`ByteBuffer.allocateDirect()`时返回。
   - 底层直接`malloc()`，不由JVM管理，需要自动手动释放内存防止内存泄漏
   - 页对齐(`page aligned`)的。可以直接和底层代码通信（例如OpenGL）
3. `MappedByteBuffer`：调用`FileChannel.map()`时返回
   - 是`mmap()`的包装类，用户空间虚拟内存和内核空间物理内存直接映射
   - `private` 模式是 `copy on write`， 若要修改则自己再拷贝一份到用户空间。不对其他映射相同位置的程序文件可见。
   - `READ_WRITE` 模式，`unmmp`写完后需要刷新 `TLB` 缓存和内存，修改不总是对其他相同映射对程序可见


## 文件描述符
![文件描述符](https://img-blog.csdnimg.cn/20190421151618649.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbjEzMTQx,size_16,color_FFFFFF,t_70)
- 在进程 A 中，文件描述符 1 和 20 都指向了同一个打开文件表项，标号为 23（指向了打开文件表中下标为 23 的数组元素），这可能是通过调用 dup()、dup2()、fcntl() 或者对同一个文件多次调用了 open() 函数形成的。
- 进程 A 的文件描述符 2 和进程 B 的文件描述符 2 都指向了同一个文件，这可能是在调用 fork() 后出现的（即进程 A、B 是父子进程关系），或者是不同的进程独自去调用 open() 函数打开了同一个文件，此时进程内部的描述符正好分配到与其他进程打开该文件的描述符一样。
- 进程 A 的描述符 0 和进程 B 的描述符 3 分别指向不同的打开文件表项，但这些表项均指向 i-node 表的同一个条目（标号为 1976）；换言之，它们指向了同一个文件。发生这种情况是因为每个进程各自对同一个文件发起了 open() 调用。同一个进程两次打开同一个文件，也会发生类似情况。 


## 内核空间和用户空间
32位操作系统下，寻址空间（虚拟地址空间，或叫线性地址空间）为 4G(2^32)。即一个进程的最大地址空间为 4G。
- 内核访问底层硬件设备，拥有所有权限，可以执行各种危险指令。不让普通进程使用是为了提高操作系统的稳定性及可用性。
   >  Ring3 级别时被称为运行在用户态，而运行在 Ring0 级别时被称为运行在内核态
- 高字节1G为内核空间，低字节3G为各个进程使用，称为用户空间。
- 最高 1G 的内核空间是被所有进程共享的
### 从用户态进入到内核态方式
系统调用、软中断（定时中断等）、硬件中断

实际上每个处理器在任何指定时间点上的活动概括为下列三者之一：
- 运行于用户空间，执行用户进程。
- 运行于内核空间，处于进程上下文，代表某个特定的进程执行。
- 运行于内核空间，处于中断上下文，与任何进程无关，处理某个特定的中断。

## 为什么kill -9 杀不死一号进程init
因为你无法在思考的时候关掉自己的大脑

## socket加NET_LINK