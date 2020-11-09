---
title: Fragment的坑
date: 2019-03-18 16:42:02
tags: 
    - Fragment
---
1. 子fragment中写了listener，父fragment设置子fragment的listener，然后横竖屏切换时，listener会报空：
2. 在activity被异常回收之后，如果没有设置onConfigurationChange，activity会重建完整再走一遍生命周期的流程。onSaveInstanceState方法被调用的时候，已经add到FragmentManager中的fragment不被释放，旧的fragment的内存实例始终存在，且view也始终在展示并不会被destroy，因此需要在Acitivty的onCreate()和onSaveInstanceState()处添加代码：
```JAVA
    if (savedInstanceState != null) {
        savedInstanceState.remove("android:support:fragments");
    }
```