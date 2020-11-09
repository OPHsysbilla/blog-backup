---
title: 唔！SurfaceView了解一下？
date: 2019-01-28 16:33:41
tags:
    - SurfaceView
---

## 使用SurfaceView时需要注意
   > - All SurfaceView and SurfaceHolder.Callback methods will be called from the thread running the SurfaceView's window (typically the main thread of the application). They thus need to correctly synchronize with any state that is also touched by the drawing thread.
   > - You must ensure that the drawing thread only touches the underlying Surface while it is valid -- betweenSurfaceHolder.Callback.surfaceCreated() andSurfaceHolder.Callback.surfaceDestroyed().
## 硬件加速？
- Android里所有View控件以自定义View所使用得Canvas是硬件加速的（使用 GPU 进行加速，在 Android 上一般就是指会使用 OpenGL 进行绘制），Surface.lockCanvas获取的就是关闭硬件加速的Canvas。见[OpenGL中 Canvas 性能分析](https://mp.weixin.qq.com/s/RCix4L4E3TBcP79TMGBmtA)

## SurfaceView/TextureView的区别
- SurfaceView是一个有自己Surface的View。界面渲染可以放在单独线程而不是主线程中。它更像是一个Window，自身不能做变形和动画。
- TextureView同样也有自己的Surface。但是它只能在拥有硬件加速层层的Window中绘制，它更像是一个普通View，可以做变形和动画。
- 普通View都是共享一个Surface的，所有的绘制也都在UI线程中进行，因为UI线程还要处理其他逻辑，因此对View的更新速度和绘制帧率无法保证。这显然不适合相机实时 预览这种情况，因而SurfaceView持有一个单独的Surface，它负责管理这个Surface的格式、尺寸以及显示位置，它的Surface绘制也在单独的线程中进行，因而拥有更高 的绘制效率和帧率。
