---
title: leetcode题目思考与记录
date: 2020-02-29 21:34:13
tags:
    - Algorithm
    - 算法
    - 面试
categories: 面试
---

# 移动0

# 二叉树的中序遍历

¥ 爬楼梯、括号生成

# 删除排序数组中的重复项

# 盛水最多的容器

# 跳表

## 约瑟夫环问题
 [见此](https://blog.csdn.net/lei396601057/article/details/109312337)

## 单调栈


1. [42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

2. [84. 柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

3. 保卫王国 - 牛客

## 无损压缩算法：deflate、Huffman Coding


##### 寻找两个有序数组的中位数
要求算法的时间复杂度为 O(log(m + n))
即找出第(m+n)/2个元素
设k=(m+n)/2，则需要在两个数组中分别寻找第k/2个元素，原因如下：

<!--more-->

题目中要求的时间复杂度为 O(log(m + n))，很容易想到的方法就是二分，现在有两个数组，要对那个数组进行二分合适？由于找的是中位数，那么这个数字的两边的元素个数是相等的，所以只需要确定一个数组中的两边元素，两一个数组的对应的补上去就可以了，为了提高效率，要选择最短的数组做二分查找 — [原文](https://blog.csdn.net/Shuffle_Ts/article/details/93142735)


##### 为什么Java中int型数据取值范围是[-2^{31}, 2^{31}-1]
[为什么Java中int型数据取值范围是[-2^{31}, 2^{31}-1]](https://blog.csdn.net/AlpinistWang/article/details/87994617?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)


##### 判断一个有符号int型整数是不是溢出：
x > INT_MAX / 10 || x == INT_MAX && x * 10 + 7 就会溢出
x < INT_MIN / 10 || x == INT_MIN. &&  x * 10 + 8 就会溢出

##### 反转数字
push一位数字x：int res = res * 10 + x
Pop一位数字：res /= 10
需要警惕是否溢出

##### Lru最近访问
Lru在Java里就是LinkedHashMap有序字典，每次get/put后把该元素挪动到双向链表尾。双向链表是因为方便删除，是所有数据的。、

### 动态规划
#### 2、3、5硬币面值（无限个数），构成总数为k的方法数
和上楼梯很像，区别在于上楼梯只是1、2两步
> 上楼梯的状态方程： `dp[0] = dp[1] = 1, dp[i] = dp[i-1] + dp[i-2] (i > 2)`。设置dp[0] = 1只是为了后续好加；
这一题就是`dp[1] = 1, dp[i] = dp[i-2] + dp[i-3] + dp[i-5]`，意义是为了总数为i的金额是由`i-2`的情况和`i-3`和`i-5`的情况达成的

#### trim树 - 搜索自动补全


#### TSP问题


#### 三重dp


#### 拓扑 + ？ 