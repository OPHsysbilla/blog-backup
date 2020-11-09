---
title: 错误的PorterDuffXferMode混合模式
date: 2019-01-25 15:38:29
tags:
    - Android,
    - PorterDuffXferMode
    - 动画
---
# 使用PorterDuffXferMode后效果并没有得到想要的效果！
-> 关了硬件加速还是不行555
## 为何和谷歌给的样子不同？
因为谷歌写的demo里，是两个高宽一样的bitmap在相交，如果发现有多余的位置没有被遮住，说明两个相交的位置不一样。
  > 源图像在运算时，只是在源图像所在区域与对应区域的目标图像做运算。所以目标图像与源图像不相交的地方是不会参与运算的！这一点非常重要！不相交的地方不会参与运算，所以不相交的地方的图像也不会是脏数据，也不会被更新，所以不相交地方的图像也永远显示的是目标图像。 —— 《[Android高级进阶——绘图篇（五）setXfermode 设置混合模式](https://www.jianshu.com/p/78c36742d50f)》
<!--more-->
## 谁是源图像谁是目标图像
``` JAVA
    //绘制 目标图像
    canvas.drawBitmap(dstBitmap, 100, 100, paint);
    //设置 模式 为 SRC_OUT
    paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_OUT));
    //绘制源图像
    canvas.drawBitmap(srcBitmap, 100, 100, paint);
```
## 任何和空白透明像素相交的结果都是空白透明像素哦
所以SRC_OUT等需要有颜色，而不是透明的。比如SRC_OUT时需要添加颜色。

## 例子
- 用蒙版做了一个指针指示，里面有提到关于**带透明度**的混合模式与**谷歌给的Api**展示的区别：[Android 混合模式 —— PorterDuffXferMode](https://blog.csdn.net/wolinxuebin/article/details/79513353)
- 基于谷歌Api的demo修改了让你测试：[Canvas绘图之PorterDuffXfermode使用及工作原理详解](https://blog.csdn.net/iispring/article/details/50472485)
- 给了一个demo让你自己尝试不同的效果： [关于 Xfermode 正确理解姿势](https://juejin.im/entry/5a7fdc35f265da4e783297a8)
- 做了一个DST_ATOP的例子：[android遮罩Xfermode的学习](https://blog.csdn.net/z8z87878/article/details/52841320)
- 解释了SRC_OUT：[通过消除背景层与重叠部分绘制组合图形](https://blog.csdn.net/u013372185/article/details/51768147)