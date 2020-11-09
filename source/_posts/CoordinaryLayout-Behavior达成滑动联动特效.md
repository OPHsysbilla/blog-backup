---
title: CoordinaryLayout-Behavior达成滑动联动特效
date: 2019-04-01 12:24:51
tags:
---
> Android常规的Touch事件传递机制是自顶向下，由外向内的，一旦确定了事件消费者View，随后的事件都将传递到该View。因为是自顶向下，父控件可以随时拦截事件，下拉刷新、拖拽排序、折叠等交互效果都可以通过这套机制完成。**Touch事件传递机制是Android开发必须掌握的基本内容。但是这套机制存在一个缺陷：子View无法通知父View处理事件。NestedScrolling就是为这个场景设计的。** ———— [Android Nested Scrolling](https://blog.kyleduo.com/2017/03/08/nested-scrolling/)

