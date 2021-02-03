---
title: Jetpack 面试答案整理
date: 2020-11-20 11:46:54
tags:
    - 面试
    - Android
categories: 面试
---
# Framgnet里的setRetainInstance干嘛用的？
在Acitivty因为`configurationChange`要销毁重建时，设置为`setRetainInstance`的Fragment的不会被重建，生命周期会受影响，即重建时不会经过onDestroy和onCreate：
<!--more-->
- `onDestroy()` 和 `onCreate()` 不会调用（因为fragment没有被重建）
- 这个`setRetainInstance`的`Framgnet`不允许在`backstack`里
- view会被重建，但是变量不会。`onCreatedView()`会被调用
- onDetach() 和 onAttach(Activity) 和 onActivityCreated(Bundle)还是依然会被调用的

> fragment 该方法也常常被用来处理 Activity re-creation 时候数据的保存。因此可以干很多骚操作，但即使是使用setRetainInstance还是有一定概率会被回收的，重新创建fragment后，requestCode和callback的关系就没有了。

## Fragment中的onCreateView和onViewCreated的区别
(1)  onViewCreated在onCreateView执行完后立即执行。
(2)  onCreateView返回的就是fragment要显示的view。

## Fragment的生命周期
[Fragment生命周期](https://juejin.cn/post/6844903752114126855)

## Fragment内存泄漏 ⭐️
`FragmentManagerImpl#dispatchCreateOptionsMenu(Menu menu, MenuInflater inflater)`中Fragment会调用创建menu，
Fragment有创建Toolbar，并把Toolbar交给Activity来创建菜单：
```Java
setHasOptionsMenu(true);
((AppCompatActivity) getActivity()).setSupportActionBar(mToolbar);
```
而在destroyView的时候一直持有Activity的toolbar没有去掉，就会有泄漏

解决方法：
1. Fragment中的菜单由自己来创建，不交给Activity
2. 菜单还是交给Activity管理，如果上一级Fragment有创建菜单那不用处理，如果没有需要在上一级Fragment清除掉引用

# ViewModelStore是如何存储的？
在`ActivityThread#performDestroyActivity`的时候通过`NonConfigurationInstances`存储到`ActivityClientRecord`中，当重建时再拿出`ActivityClientRecord`。
如果`ActivityThread`进程整个没了的话就没了..
> 重建指当系统配置发生改变时（`Configuration changes`)，系统就会销毁Activity和与之关联的Fragment然后再次重建，再次调用`performLaunchActivity`，拿到上次onDestory的时候销毁的`NonConfigurationInstances`

`onSaveInstanceState`对于大量的数据缓存有一定的局限性，大量的数据缓存则可以使用`Fragment.setRetainInstance(true)`来保存数据


一面
1：插件化。启动activity的hook方式。taskAffity。
2：okhttp支持HTTP2？http2的功能有哪些？tcp方面拥塞控制？tsl的握手和具体的非对称加密算法。非对称名称
3：handler的post(Runnable)如何实现的。callback，runnable，msg的执行优先级。阻塞是怎么实现的？为什么不会阻塞主线程？
5：求二叉树中两个节点之间的最大距离。
6：206含义，未修改资源是哪个，302含义，301含义
7：多进程通信问题。binder优势。aidl生成的java类细节。多进程遇到哪些问题？
8：动态代理传入的参数都有哪些？非接口的类能实现动态代理吗？ASM的原理
9：Application和Activity在Context的继承树上有何区别？二者使用上有何不同？
10：任意一颗二叉树，求最大节点距离

二面
1：设计一个日志系统。
2：内存泄露的分类。怎么查看内存泄露的问题
3：touch事件源码问题。
4：组件化的问题。module和app之间的区别。moduler通信是如何实现的。
5：native奔溃的日志采集，怎么处理？
6：注解实现一个提示功能：如果int的值大于了3需要提示。

三面
1：介绍下flutter的启动流程
2：介绍下flutter与weex的区别
3：组件化介绍一下
4：webview中与js通信的手段有哪些？
5：介绍下flutter_boost的原理
