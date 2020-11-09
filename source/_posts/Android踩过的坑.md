---
title: Android踩过的坑
date: 2019-01-25 15:52:06
tags: 
    - Android
categories: Android
---
## 从AutoScroll的View说起Scroller的原理与作用
二话不说，先看看[判断RecyclerView是否滚动到底部](https://www.cnblogs.com/jdhdevelop/p/6743501.html)中说到的computeVerticalScrollExtent()函数表示的View内容与View位置的相对滚动
![多么美丽的一张图](https://raw.githubusercontent.com/OPHsysbilla/ophsysbilla.github.io/master/images/bg/hwo_scorll_view_act_hint.png)

## 错误使用PorterDuffXferMode后效果并没有得到想要的效果！
-> 关了硬件加速还是不行555
### 为何和谷歌给的样子不同？
因为谷歌写的demo里，是两个高宽一样的bitmap在相交，如果发现有多余的位置没有被遮住，说明两个相交的位置不一样。

<!--more-->

  > 源图像在运算时，只是在源图像所在区域与对应区域的目标图像做运算。所以目标图像与源图像不相交的地方是不会参与运算的！这一点非常重要！不相交的地方不会参与运算，所以不相交的地方的图像也不会是脏数据，也不会被更新，所以不相交地方的图像也永远显示的是目标图像。 —— 《[Android高级进阶——绘图篇（五）setXfermode 设置混合模式](https://www.jianshu.com/p/78c36742d50f)》
<!--more-->
### 谁是源图像谁是目标图像
``` JAVA
    //绘制 目标图像
    canvas.drawBitmap(dstBitmap, 100, 100, paint);
    //设置 模式 为 SRC_OUT
    paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_OUT));
    //绘制源图像
    canvas.drawBitmap(srcBitmap, 100, 100, paint);
```
### 任何和空白透明像素相交的结果都是空白透明像素哦
所以SRC_OUT等需要有颜色，而不是透明的。比如SRC_OUT时需要添加颜色。

### 例子
- 用蒙版做了一个指针指示，里面有提到关于**带透明度**的混合模式与**谷歌给的Api**展示的区别：[Android 混合模式 —— PorterDuffXferMode](https://blog.csdn.net/wolinxuebin/article/details/79513353)
- 基于谷歌Api的demo修改了让你测试：[Canvas绘图之PorterDuffXfermode使用及工作原理详解](https://blog.csdn.net/iispring/article/details/50472485)
- 给了一个demo让你自己尝试不同的效果： [关于 Xfermode 正确理解姿势](https://juejin.im/entry/5a7fdc35f265da4e783297a8)
- 做了一个DST_ATOP的例子：[android遮罩Xfermode的学习](https://blog.csdn.net/z8z87878/article/details/52841320)
- 解释了SRC_OUT：[通过消除背景层与重叠部分绘制组合图形](https://blog.csdn.net/u013372185/article/details/51768147)

## 到底有没有在用心学ndk啊
1. JNIEnv*env 是一个线程对应一个env，线程间不可以共享同一个env变量。[JNI中Fatal signal 11 (SIGSEGV), code 1的错误](https://blog.csdn.net/viking_xhg/article/details/78727273)
2. 可以使用Android自带的addr2line来分析native的行信息[Android通过addr2line工具分析native crash log](https://juejin.im/post/5c3c6ec9f265da616a47df88)
3. 注意lock后再wait该锁的问题[JVM故障分析系列之五：常见的Thread Dump日志案例分析](https://www.javatang.com/archives/2017/10/26/08572060.html)

## 使用SurfaceView时需要注意
   > - All SurfaceView and SurfaceHolder.Callback methods will be called from the thread running the SurfaceView's window (typically the main thread of the application). They thus need to correctly synchronize with any state that is also touched by the drawing thread.
   > - You must ensure that the drawing thread only touches the underlying Surface while it is valid -- betweenSurfaceHolder.Callback.surfaceCreated() andSurfaceHolder.Callback.surfaceDestroyed().
### 硬件加速？
- Android里所有View控件以自定义View所使用得Canvas是硬件加速的（使用 GPU 进行加速，在 Android 上一般就是指会使用 OpenGL 进行绘制），Surface.lockCanvas获取的就是关闭硬件加速的Canvas。见[OpenGL中 Canvas 性能分析](https://mp.weixin.qq.com/s/RCix4L4E3TBcP79TMGBmtA)

### SurfaceView/TextureView的区别
- SurfaceView是一个有自己Surface的View。界面渲染可以放在单独线程而不是主线程中。它更像是一个Window，自身不能做变形和动画。
- TextureView同样也有自己的Surface。但是它只能在拥有硬件加速层层的Window中绘制，它更像是一个普通View，可以做变形和动画。
- 普通View都是共享一个Surface的，所有的绘制也都在UI线程中进行，因为UI线程还要处理其他逻辑，因此对View的更新速度和绘制帧率无法保证。这显然不适合相机实时 预览这种情况，因而SurfaceView持有一个单独的Surface，它负责管理这个Surface的格式、尺寸以及显示位置，它的Surface绘制也在单独的线程中进行，因而拥有更高 的绘制效率和帧率。

## 非UI线程使用View.post，可能会导致内存泄漏
- [在API低于24的版本上，非UI线程使用View.post，可能会导致内存泄漏](http://ivanfan.site/2018/03/18/%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F/)。因为子线程调用View.post()，如果view还未attach到window，只有UI线程的performTraversals才会去把runnable拿出来执行，子线程没有performTraversals，故这个runnable将永远不会执行。
	> 1. 文章中最后给出的结论是，要保持兼容 API 24 以下的版本，最好不要在非 UI 线程使用 View.post ，而采用 Handler 来 post runnable。
	> 2. 当然在 API 24 及以上，如果不能保证 View 一定会被 attach，那可以在引用对象销毁时，使用 View.removeCallbacks。


## Fragment的坑
1. 子fragment中写了listener，父fragment设置子fragment的listener，然后横竖屏切换时，listener会报空：
2. 在activity被异常回收之后，如果没有设置onConfigurationChange，activity会重建完整再走一遍生命周期的流程。onSaveInstanceState方法被调用的时候，已经add到FragmentManager中的fragment不被释放，旧的fragment的内存实例始终存在，且view也始终在展示并不会被destroy，因此需要在Acitivty的onCreate()和onSaveInstanceState()处添加代码：
```JAVA
    if (savedInstanceState != null) {
        savedInstanceState.remove("android:support:fragments");
    }
```

## Android Studio 的新东西
1. `find . | grep hprof-conv` 在~\Library\Android\sdk\下找出`hprof-conv`的位置，再使用命令`./hprof-conv -z infile outfile` 来转换成Eclipse的Mat工具能够识别的hprof文件，注意使用此命令时，指定的outPutFile路径需要已经被创建，否则会一直返回Usage...展示`Usage: hprof-conf [-z] infile outfile`
2. [强行刷新gradle依赖缓存，拉取远程依赖版本](https://blog.csdn.net/T_yoo_csdn/article/details/84950088)：`./gradlew build --refresh-dependencie`。
    > 只想刷新某个指定的依赖: 直接去~/.gradle/caches/modules-2目录下，rm -fr `find . -name xxx`，然后直接重编
    > 删除含有'aaa'的文件夹依赖：`find ~/.gradle/caches -type d | grep 'aaa' | xargs rm -r`

## CoordinaryLayout-Behavior达成滑动联动特效
> Android常规的Touch事件传递机制是自顶向下，由外向内的，一旦确定了事件消费者View，随后的事件都将传递到该View。因为是自顶向下，父控件可以随时拦截事件，下拉刷新、拖拽排序、折叠等交互效果都可以通过这套机制完成。**Touch事件传递机制是Android开发必须掌握的基本内容。但是这套机制存在一个缺陷：子View无法通知父View处理事件。NestedScrolling就是为这个场景设计的。** ———— [Android Nested Scrolling](https://blog.kyleduo.com/2017/03/08/nested-scrolling/)


## 在Window上做动画：Acitivity间自定义动画的可能性
这两天一直在想做一个Activity间的转场动画可以对单个view进行自定义动画的。因为ShareElement动画不支持5.0以下，所以开始思考其他的方法。
<!--more-->
读了[通过WindowManager添加view以及添加动画](https://blog.csdn.net/weixin_38695860/article/details/71410629)后发现其实shareElemet也是在做假动画，本activity一次假动画，下一个Activit一次假动画。这篇[Android WindowManager及其动画问题](https://blog.csdn.net/wangjinyu501/article/details/38847611)中说到，在WindowManager上像在Activity中使用动画效果无效，目前还没细看。

- 发现了一个特殊的效果..[图凌闪屏页及Android彩蛋探究](https://juejin.im/post/5c498230e51d450672355df1)
### 目前的结论
还是做假的动画比较靠谱...