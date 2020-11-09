---
title: 在Window上做动画：Acitivity间自定义动画的可能性
date: 2019-01-25 15:52:06
tags: 
    - Android,
    - WindowManager
    - 动画
categories: Android动画
---


## 前言
这两天一直在想做一个Activity间的转场动画可以对单个view进行自定义动画的。因为ShareElement动画不支持5.0以下，所以开始思考其他的方法。
<!--more-->
读了[通过WindowManager添加view以及添加动画](https://blog.csdn.net/weixin_38695860/article/details/71410629)后发现其实shareElemet也是在做假动画，本activity一次假动画，下一个Activit一次假动画。这篇[Android WindowManager及其动画问题](https://blog.csdn.net/wangjinyu501/article/details/38847611)中说到，在WindowManager上像在Activity中使用动画效果无效，目前还没细看。

- 发现了一个特殊的效果..[图凌闪屏页及Android彩蛋探究](https://juejin.im/post/5c498230e51d450672355df1)
## 目前的结论
还是做假的动画比较靠谱...