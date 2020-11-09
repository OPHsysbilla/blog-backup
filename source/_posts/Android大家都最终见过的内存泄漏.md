---
title: Android大家都最终见过的内存泄漏
date: 2019-08-13 15:53:04
tags:
---



#### 非UI线程使用View.post，可能会导致内存泄漏
- [在API低于24的版本上，非UI线程使用View.post，可能会导致内存泄漏](http://ivanfan.site/2018/03/18/%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F/)。因为子线程调用View.post()，如果view还未attach到window，只有UI线程的performTraversals才会去把runnable拿出来执行，子线程没有performTraversals，故这个runnable将永远不会执行。
	> 1. 文章中最后给出的结论是，要保持兼容 API 24 以下的版本，最好不要在非 UI 线程使用 View.post ，而采用 Handler 来 post runnable。
	> 2. 当然在 API 24 及以上，如果不能保证 View 一定会被 attach，那可以在引用对象销毁时，使用 View.removeCallbacks。

