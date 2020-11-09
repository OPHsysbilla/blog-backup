---
title: Android 警醒向条例
date: 2019-01-23 00:00:00
tags:
	- Android
---
依然很简洁的条例，备忘用

*本篇是搜集关于Android的仅以提醒作用的条例，见名知意。 相关注意事项推荐阅读《阿里巴巴的安卓手册》* 

## 条例
1. onActivityResult()发生在onResume()之前
15. onFinishInflate()在加载完成xml后执行：只有当自定义View，通过在XML中创建时才会调用。不从xml布局文件中解析的话，比如通过new方式创建，则不会执行该方法；
<!--more-->
16.  foreach使用前需要判空
17. overridependingtransition()进入动画需要再startActivity之后，退出动画需要再finish后
19. view的调用过程:构造方法->onFinishInflate->onSizeChanged->onDraw
20. 如果用户finish()方法结束了Activity，则不会调用onSaveInstanceState()
21. getIntrinsicWidth()和getIntrinsicHeight()
22. canvas的好处之一是旋转画布后旋转回来，以达到图像绘制部分的旋转，不用对坐标点进行转置
23. 用transient关键字标记的成员变量不参与序列化过程
26. 滑动冲突处理：可以根据滑动距离和水平方向形成的夹角；或者根绝水平和竖直方向滑动的距离差；或者两个方向上的速度差等
27. getParent().requestDisallowInterceptTouchEvent(false)请求父亲不要阻挡儿子
28. getSuggestedMinimumWidth的逻辑：View如果没有背景，那么返回android:minWidth这个属性指定的值，这个值可以为0；如果设置了背景，则返回背景的最小宽度和minWidth中的较大值。
29. Fragment的动画setCustomAnimations()对于反射取FragmentManager中置儿子为null的是有问题的，建议用Fragment自己的onCreateAnimation()函数
30. **OOM**：尽量避免使用帧动画，使用的话应尽量避免使用过多尺寸较大的图片
31. 内存泄露：属性动画中的无限循环动画需要在Activity退出的时候及时停止，否则将导致Activity无法释放而造成内存泄露。view动画不存在这个问题；
33.  当调用到onNewIntent(intent)的时候，需要在onNewIntent() 中使用setIntent(intent)赋值给Activity的Intent.否则，后续的getIntent()都是得到老的Intent
34.  launchMode为singleTask的时候，通过Intent启到一个Activity,如果系统已经存在一个实例，系统就会将请求发送到这个实例上，但这个时候，系统就不会再调用通常情况下处理请求数据的onCreate方法，而是调用onNewIntent方法	
35. DiffUtil.DiffResult可以使RecyclerView不添加相同的条目，即部分更新
36. RelativeLayout如果cast到FrameLayout是不必要的，view就可以
37. 自定义view，一定是this(context)、this(context,attr)、super(context, attr, defaltStyle)，注意需要手动调init()或者this(context, null)与this(context,attr,0)
38. 小心使用ArrayList的subList [a, b) ，对生成对新列表的改变会改变原列表结构，可以在新列表前加final来阻止后续更改
39.  RequestLayout() , Invalidate() , layout()的区别
40. 如果需要addView的话，那么使用getLayoutParams是没用的，因为这个你自己新建的一个View，没有与父容器有任何关联。所以LayoutParams也需要你自己创建	
40. contentInsetStart消除ToolBar在自定义布局中的左部白边
40.  JSONObject中的getString()和optString()方法的区别在于optString()不报错
40. @Transient标志则不入数据库
40. overridePendingTransition转移
40. 由于Activity的TaskMode不同所以onActivityResult在Activity的生命周期中执行的先后也是不同的，更有onCreate不会执行的情况，所以onActivityResult有时候提前结束收不到result。
40. 调用startActivityForResult后onActivityResult立刻响应，而返回当前页时onActivityResult不响应主要是launchMode的问题：
	 - 第一种：当resultCode==0时，Activity的launchMode必须设置为”singleTask”或者singleInstance，否则就会出错。这是因为从Task的角度看，Android认为不同Task之间的Activity是不能传递数据的，所以不能使用NEW_TASK标识，但还是要调用forResult方法。
	 - 第二种：当resultCode！=0时，Activity的launchMode必须为standard（删除launchMode属性默认为standard模式），原因同上。
	- 只有第一层fragment会收到onActivityResult
	  - 重写onBackPressed() 自定义回退事件可以避免按Back键自动设置resultCode为RESULT_CANCELED的问题
	  - requestCode >=0就好，随便用于在onActivityResult()区别哪个子模块回传的数据，每个区分开不同的requestCode
	   - resultCode 如果B子模块可能有几种不同的结果返回，可以用setResut(int resultCode, Intent intent)予以识别区分
39. `@IntDef` 注解自定义了一个 OrientationType 注解，用于防治用户随意设置数值。
使用 `@IntRange` 方法将行列数限制在一个较合理的范围内。
	```JAVA
	@IntDef({VERTICAL, HORIZONTAL})
	public @interface OrientationType {}            // 滚动类型
	@OrientationType
	private int mOrientation = HORIZONTAL;          // 默认水平滚动
	```
40. `View child = recycler.getViewForPosition(i);`   //如果不存在的话 RecyclerView 会自动创建
41. RecyclerView要先移除再添加，这样会移除的 View 会被先放到缓冲区中，再添加 View 时就可以直接从缓冲区中把被移除的条目直接取出来使用了，而不用重新创建，以减少开销；如果先添加，由于缓冲区中没有可以使用的 View，会进行先创建，之后再添加到界面上，最后执行移除操作会导致有大量的 View 滞留在缓冲区中，会造成严重的性能浪费
42. 对Vector、ArrayList在迭代的时候如果同时对其进行修改就会抛出 java.util.ConcurrentModificationException 异常
43. LinkedList在遍历循环中get(i)时间复杂度为O(n)，比ArrayList的O(1)性能差，使用有迭代器(Iterator)以及ForEach循环来对LinkedList进行遍历
44. 对Activity来说，所有需要传递或接收的Activity不允许设置launchmode="SingleTask"，只能设为标准模式，否则系统将在 startActivityForResult() 后直接调用 onActivityResult()
46. Builder模式下，需要Builder单独列出的原因之一是，在使用单例模式时，每次都是用的同一个instance，而参数每次都需要重新new一个；不单独写一个Builder出来（第二重）是由于每次都重新new了一个XXX实例，当然参数也每次都不一样
	```JAVA
	//第一种
	class XXX{
	    //普通单例模式
	    private static XXX instance;
	    public  static XXX getInstance(){
	        if(instance == null){
	            return new instance;
	        }
	    }
	
	    private XXX{}
	    class Builder(){
	        void set(){
	            ...
	            return Builder.this;
	        }
	    }
	}
	//第二种
	class XXX{
	    void set(){
	        ...
	        return XXX.this;
	    }
	}
	```

48. 基于引用计数的垃圾回收器无法处理循环引用导致的内存泄露问题，但是其在主流的JVM中很少，几乎所有的JVM都是采用引用对象遍历的方法，垃圾回收器都会处理循环引用潜在的问题
49. **ThreadLocal、Volatile、synchronized、Atomic 的区别？**
如果只有一个`i++;`的时候，volatile和synchronized能否互换？volatile作为修饰变量的时候，变量自加中途可能发生线程调度，volatile 可以保证在一个线程的工作内存中修改了该变量的值，该变量的值立即能回显到主内存中，从而保证所有的线程看到这个变量的值是一致的。但是有个前提，因为它不具有操作的原子性，也就是它不适合在对该变量的写操作依赖于变量本身自己。就比如`i++`、`i+=1`;这种。但是可以改为`num=i+1`;如果i是一个 volatile 类型，那么num就是安全的，总之就是不能作用于自身。
**synchronized**是基于代码块的，只要包含在synchronized块中，就是线程安全的。
**AtomicInteger**，一个轻量级的synchronized。使用的并不是同步代码块，而是Lock-Free算法(一个死循环调用了底层的比较方法直到相同后才退出循环)，最终的结果就是在高并发的时候，或者说竞争激烈的时候效率比synchronized高一些。
**ThreadLocal**，线程中私有数据。主要用于线程改变内部的数据时不影响其他线程，使用时需要注意static。
再补一个，才学到的。利用clone()方法，如果是一个类的多个对象想共用对象内部的一个变量，而又不想这个变量static，可以使用浅复制方式。(查看设计模式原型模式)
50. HashTable、SynchronizedCollection、ConcurrentHashMap、Vector
51. [静态内部类单例原理](https://blog.csdn.net/mnb65482/article/details/80458571)，静态内部类单例在反序列化时会重新生成新对象因此需要抛出一个异常，反序列化是比正常创建新对象慢的
52. compare and set (CAS)，不断重新取值比较，可能会出现ABA问题（中途值被修改又还原）
53. [Message全程持有发送该消息Handler的引用 直到Handler处理完消息，Message回收释放](https://upload-images.jianshu.io/upload_images/2158254-ede2828fcff9398e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
实际上经常会出现的Diaglog泄漏就是，Dialog销毁的时候Activity已经销毁，所以会发不出那个Handler
54. git commit --amend 可以修改提交
55. onScroll和onFling的区别在于，onFline是用于滑动的最后一下
56. 不要重复 setListener，要使用 v.getId 来复用 Listener，不然会创建一堆 Listener 导致频繁 GC
57. final与static final的区别是：final在一个对象类唯一，static final在多个对象中都唯一；
一个既是static又是final的域只占据一段不能改变的存储空间，只有一份。
58. View的UI刷新，不会导致阻塞的原因是View 的绘制与 Java 代码的looper无关，而是由底层 SurfaceFlinger 自身的事件处理机制处理的
    > 因为不光是gui，同样的道理在几乎所有编程领域里都是这样的，这背后是线程同步的开销问题。显然两个线程不能同时draw，否则屏幕会花；不能同时insert map，否则内存会花；不能同时write buffer，否则文件会花。需要互斥，比如锁。结果就是同一时刻只有一个线程可以做ui。那么当两个线程互斥几率较大时，或者保证互斥的代码复杂时，选择其中一个长期持有其他发消息就是典型的解决方案。所以普遍的要求**ui只能单线程**
    
59. onChildDraw影响触摸事件的接受顺序，同样影响绘制顺序
60. 对组件 Activity 而言，viewRootImpl 的初始化在 onCreate 之后，onResume 之后，故在onCreate里开子线程更新UI不会报错
61. [Glide中into()同一View，但显示不同的图片时（url不同）是使用的同一个Bitmap引用地址，只是像素不一样，在onResourceReady()里可以打印得到的bitmap的地址是一样的](https://www.cnblogs.com/linguanh/p/8514643.html)。这是因为BitmapPool会复用Request，可能会拿到擦除过的bitmap
62. bitmap对象是通过env->NewOject(...)，所以是分配在虚拟机中的
63. RecyclerView.OnScrollListener()来监听RecyclerView的滚动距离，正常情况下是没有问题的，但是一旦RecyclerView的数据源变了的时候，比如筛选条件变了，需要清除之前的数据，然后重新添加新的数据到RecyclerView里面去，这时候记录的滚动距离就不准了；自定义RecyclerView的LinearLayoutManager来计算滚动距离，但是在滑动的时候特别消耗性能，滑动卡顿
64. AppCompatActivity获得Activity的方法[Android get hosting Activity from a view](https://stackoverflow.com/questions/8276634/android-get-hosting-activity-from-a-view/32973351#32973351)
65. [Java的正则式首先需要Pattern.compile()后必须要调用find()或者matches()才能找到](https://stackoverflow.com/questions/12911504/rationale-for-matcher-throwing-illegalstateexception-when-no-matching-method-i)
66. 在自定义控件的构造函数或者其他绘制相关地方使用系统依赖的代码，会导致可视化编辑器无法报错并提示，使用isInEditMode()可以解决在
67. 在Android Studio下的文件`app.iml`已经制定了assets文件夹的属性，所以直接新建Directory即可
68. **view.layout(l,t,r,b)在子线程中没反应，需要post()或者onAnimationStart中**
69. getGlobalVisibleRect(globalRect)在没有被遮挡（超出屏幕）时和getLocationInWindow数值相同; getDrawingRect(drawingRect)和getLocalVisibleRect(localRect)没有遮挡时测试数值相同都是相对自己的坐标
70. include设置了id，会覆盖掉layout_activity_head布局文件中根layout的id
71. 全面屏刘海屏的状态栏是很高的，在使用getLocationInWindow时需要特别注意，需要减去的状态栏高度会更高
72. onCreate期间调用setHasOptionsMenu，则该fragment在其后的onCreateOptionsMenu回调可以接受到设置标题栏的menu
73. 在selector的xml写法中，vector图标不能用`android:color/transparent`来表明空图像，因为颜色是没有高度的（instrictWidth），并且导致后续高宽不会自动更新
    > 直接在selector的xml中设置item为透明 `<item android:drawable="@android:color/transparent" android:state_checked="false" />` 的写法是错误的，会导致高宽都变化为0，并且高宽不会自动更新。应该更改vector的文件的颜色为android:fillColor="#00000000"
74. 设置Acticity为全屏会获得和adjustNothing一样的效果，不会顶起布局
75. viewstub被inflate后会消失，包括它的id，可以使用android:inflatedId="@id/vs_bottom"来规定它被inflate后叫什么，这样可以防止viewStub消失后其他引用它的布局失去了依赖
76. rxjava中的timer也可以加入自己的scheduler
77. viewstub的布局参数应当写到viewstub内，比如margin值，写在layout内是无效的 ,但是padding是可以写在layout内的；ViewStub中不能用merge
78. android中setOnClickListener会自动设置view为clickable = true，所以自定义的simpleAdapter添加setOnItemClick的时候需要自动添加view的clickable = true
79. may not a tips:  GreenDao主键自增类型必须是Long，而不是long。否则你会发现id永远为0，永远只有一个数据
80. onStop可能不会被调用（eg. 新Acitivity采用了透明主题，当前Actiivty不会回调onStop）
81. 回到原Activity  onRestart -> onStart -> onResume
82. 每个View都可以onSaveInstanceState，并且onSaveInstanceState及onRestoreInstanceState是和onStop及onStart绑定的，onSaveInstanceState在onStop前，onRestoreInstanceState在onStart后
83. onNewIntent()---->onResart()------>onStart()----->onResume(). 且当调用到onNewIntent(intent)的时候，需要在onNewIntent() 中使用setIntent(intent)赋值给Activity的Intent.否则，后续的getIntent()都是得到老的Intent。
84. singleTask的Actiivty被启用时，在其上的栈顶都会被清空（默认具有FLAG_ACTIVITY_CLEAR_TOP）
85.`fitsSystemWindows=true`要慎用，很多坑。比如WebView中输入框获取焦点弹出软键盘时出现抖动，还有哪个View设置了fitsSystemWindows=true软键盘弹出时哪个View就会被顶上去；
86. `WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS`不要用，会导致EMUI3.1的系统下面虚拟按键挡住布局
87. 事件传递
	```JAVA
	bool dispatchTouchEvent(MotionEvent ev){
	    bool consume = false;
	    if(onInterceptTouchEvent(ev)){
	        consume = onTouchEvent(ev);
	    } else {
	        consume = child.dispatchTouchEvent(ev);
	    }
	}
	```
88. FragmentStatePagerAdapter和FragmentPagerAdapter都没有从viewGroup中removeView，而是transaction中detach/ remove掉了， 因此viewPager在保存不是fragment的时候需要警惕，其他view并不会像fragment一样刷新
90. PathMeasure的forceClosed参数对绑定的Path不会产生任何影响，例如一个折线段的Path，本身是没有闭合的，forceClosed设置为True的时候，PathMeasure计算的Path是闭合的，但Path本身绘制出来是不会闭合的。 - [均摘自 gcssloop ](http://www.gcssloop.com)
forceClosed参数对PathMeasure的测量结果有影响，还是例如前面说的一个折线段的Path，本身没有闭合，forceClosed设置为True，PathMeasure的计算就会包含最后一段闭合的路径，与原来的Path不同。
91. `Math.atan2(tan[1],tan[0])`解释：tan[0] = cos = 邻边(单位圆x坐标)，tan[1] = sin = 对边(单位圆y坐标)，因此可以计算任意两点间的斜率夹角
92. pre和post，左乘进行行变换，右乘进行列变换，pre表明M左乘pre，poste表明M右乘pre，M表明矩阵
93. 由于硬件加速的问题，PathMeasure中的getSegment在讲Path添加到dst数组中时会被导致一些错误，需要通过`mDst.lineTo(0,0)`来避免这样一个Bug
94.  `measure.getMatrix(distance, matrix, 
PathMeasure.TANGENT_MATRIX_FLAG | PathMeasure.POSITION_MATRIX_FLAG);`
这个方法是用于得到路径上某一长度的位置以及该位置的正切值的矩阵
95. **透明主题的坑**：
   - 坑一：透明主题的 Activity 如果弹出键盘，并且是 adjustResize 模式，在键盘弹出的一瞬间可以看到前一个 Activity
     >  1. 原因：键盘弹起时，Activity 重新计算了高度缩短了，而键盘弹起有一个动画，在动画没有执行完毕之前，键盘所占用的空间上没有别的布局只有 Window ，而 Window 是透明的因此就看到了上一个 Activity
     >  2. 尝试的解决方案：在动画执行完毕后，将 Window 的背景设置为不透明，但是失败了，见坑二
   - 坑二：如果当前的 Window 背景带有透明度，在动态改变背景的时候会闪一下屏，如果 Window 的初始背景颜色没有透明度，动态改变背景很完美
     >  1. 原因：不详，目测是 Android 的 bug
     >  2. 解决方案：无。。。
   - 坑三：也是由解决坑二造成的，我试图通过监听键盘弹起事件来手动调节布局，这样 Activity 不需要 resize 也就没有那个 bug，但是发现监听键盘弹起的方法基本都是监听 Activity 重新布局后对比高度来判断的，因此在 adjustNothing 状态下无效，值得一提的是，跟三弟交流发现他们钻了一个空子，当 Activity 为全屏状态时 adjustResize 是不生效的，但是可以监听到键盘弹出，所以相当于是adjustNothing的效果，但这种情况没法在我这个项目使用。
96. 对window.addFlags需要在`setContentView`前
97. sqlLight数据库`not null`只针对于text为`null`的情况，但是text为`""`时是可以插入的。
98. 数据库的unique属性的列，insert()一旦失败，则批量都失败。此时可以用insert or replace，不会影响批量中其他数据的插入。
99. `setRetainInstance(true)`设置为true，表示 configuration change 的时候，fragment实例不会背重新创建，即重建时不会经过onDestroy和onCreate。
100. fragment 该方法也常常被用来处理 Activity re-creation 时候数据的保存。因此可以干很多骚操作，但即使是使用setRetainInstance还是有一定概率会被回收的，重新创建fragment后，requestCode和callback的关系就没有了。
     > fragment 该方法也常常被用来处理 Activity re-creation 时候数据的保存。因此可以干很多骚操作，但即使是使用setRetainInstance还是有一定概率会被回收的，重新创建fragment后，requestCode和callback的关系就没有了。 
     
24. `SharedPreferences`是文件的一种，读写有一定的缓存策略，在内存中会有一份`SharedPreferences`文件的缓存，因此在多进程模式下其读写就变得不可靠。当面对高并发读写访问的时候，有很大几率会**丢失数据**，故不建议在进程间通信中使用`SharedPreferences`
25. Messenger是一种轻量级的`IPC`方案，它的底层实现就是`AIDL`。Messenger是以串行的方式处理请求的，即服务端只能一个个处理，不存在并发执行的情形
101. rxjava2不接受`null`，会报错
102. `DialogFragment`的出现解决AlertDialog和PopupWindow随屏幕切换而消失的问题
103. 
	```JAVA
	Window window = getDialog().getWindow();
	        window.setLayout(ViewGroup.LayoutParams.MATCH_PARENT, 
	        ViewGroup.LayoutParams.MATCH_PARENT);
	```
	等效于
	```JAVA
	    DisplayMetrics dm = new DisplayMetrics();
	    getActivity().getWindowManager().getDefaultDisplay().getMetrics(dm);
	    if (null != window) {
	        window.setLayout(dm.widthPixels, dm.heightPixels);
	    }
	```
104. fragment的commit和commitAllowingStateLoss的区别
   - onSaveInstanceState()再onStop()前调用，而再Honeycomb前，是在onPause()前立即调用的，因为Honeycomb前对于回收来说一不可见Activity就被kill了，Honeycomb后Activity要在onStop后才能被kill掉
   - *参见[Fragment Transactions & Activity State Loss](https://www.androiddesignpatterns.com/2013/08/fragment-transaction-commit-state-loss.html)*

105. [Android 代码proguard混淆之后的错误log查看方法](https://blog.csdn.net/attheway/article/details/44220729)
   > 项目目录的progurad下有一个mapping.txt文件 这是混淆后的名字和原名字的映射关系。在ADT安装目录的sdk\tools\proguard\bin下有三个工具，其中proguardgui.bat就是用来还原的图形化工具。
   
106. 6.0后需要动态申请权限，7.0后需要通过FileProvider访问其他的权限。6.0后的动态申请权限，有可能会出现未授权时录音，[Android录音数据前几千个字节为空的问题](https://mp.weixin.qq.com/s/Xe-RbtackyzgLbh8DBqYfQ)。
107. 多进程在进程被kill掉的时候来不及正常调用onDestroy等流程，此时会通过binder通知其他进程自己死掉了，见[Binder死亡通知机制之linkToDeath](https://www.androiddesignpatterns.com/2013/08/binders-death-recipients.html)
108. Fragment的可见性：onHiddenChanged()只会在对fragment进行show/hide变换时调用，第一次进入的时候不会调用，在使用beginTransaction().hide(Fragment)会被调用，而且是在onResume之前；而在viewpager中的fragment则完全不一样，需要手动调用setUserVisible(boolean)。 
	> - [注意onHiddenChanged的特殊情况](http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2017/0422/7863.html)
	> - [Android Fragment可见性的判断与监听完全实现](https://www.smwenku.com/a/5ba1abcd2b71771a4da995ba/zh-cn/) 
109. ViewGroup默认情况不会调用onDraw()，它在init中会被设置成WILL_NOT_DRAW，这是从性能考虑，这样一来，onDraw就不会被调用了。如果我们要重要一个ViweGroup的onDraw方法，有两种方法：
	> - 在构造函数里面，给其设置一个颜色，如#00000000。
	> - 在构造函数里面，调用setWillNotDraw(false)，去掉其WILL_NOT_DRAW flag 
110.可以使用`.adjustViewBounds(true)`方法来设置宽度铺满屏幕的ImageView，但是对于[小于17的版本不能正确显示](https://inthecheesefactory.com/blog/correct-imageview-adjustviewbounds-with-adjustable-imageview/en)
111. fragment重叠的问题：只需要在onSaveInstanceState中置空fragmnet状态即可-》outState.putParcelable("android:support:fragments", null);
112. `drawable.getConstantState().newDrawable()`使用的是相同`ConstantState`下新建了一个drawable，和`getResources().getDrawable(R.drawable.xxx)`返回的对象一样。所有从本地资源读取图片资源`getResources().getDrawable(R.drawable.xxx)`的drawable都共享`ConstantState`，**对一个drawable做更改会影响其他drawable持有相同`ConstantState`的drawable**，eg. 修改alpha。可以使用`drawable.mutate()`来创建一个不共享的drawable。
	> [关于Drawable的缓存机制应该了解的知识](xhttps://www.heqiangfly.com/2017/06/15/android-knowledge-point-drawable-cache/)
113. 很悲伤的是`Rxjava`中`DisposableSubscriber`的`onStart()`是在subscribeOn指定的线程上进行的（通常是子线程），它在整个任务的开始的时候调用。而`onComplete()`、`onNext()`、`onError()`是在observeOn指定的线程上进行的（通常是UI线程）。事实上，如果不指定线程，observe都将在主线程进行
114. 注意RecycleView里都每个子项设置`focusable`=true的时候，这个时候设置点击就可能被最后一个抢掉了。
115. 当一个Activity反常销毁重建的时候，请先查看下你的“不保留活动“是不是开启了。
116. dp和px的关系，是与屏幕像素密度为160的倍数而不同：像素密度为160时1px = 1dp
117.

## 更多应该阅读的
1. CopyOnWriteArrayList
2. ThreadPoolExecutor
3. WeakHashMap
4. Activity的SingleTask模式，使得onIntent可用
5. notifyItemChanged，因为DataSetChange可能会造成闪烁
6. Producer Extends Consumer Super
7.  <? extends Fruits>   <? super Apple>
8.  基于Window.ID_ANDROID_CONTENT给定id添加子View
9.  ArrayList ，LinkedList不同步   Vector同步
10. HashSet，LinkedHashSet,HashMap 不同步 
11. HashTable 同步
12. 同步的ArrayList = Collections.synchronizedList(new ArrayList(...));
13. SpannableString
14. 数据库索引失败
15. Encapsulate Collection
16. 学习JEE规范，去看JBoss源码；学习类加载器，就去看OSGI源码

