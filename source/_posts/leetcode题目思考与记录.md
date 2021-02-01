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

# 爬楼梯、括号生成
<!--more-->

# 删除排序数组中的重复项

# 盛水最多的容器

# 跳表

### trim树 - 搜索自动补全


### TSP问题


### 三重dp


### 拓扑 + ？ 

### 单调栈

### 括号生成


### 大根堆
`PriorityQueue`默认是小跟堆


### 快排优化

### 归并排序 
> [剑指 Offer 51. 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)和 [315. 计算右侧小于当前元素的个数]都是归并排序的相同方法，区别在于左指针还是右指针入队时判断

[归并排序优化](https://www.cnblogs.com/noKing/p/7940531.html)
- 当递归到规模足够小时，利用插入排序（归并排序只是需要两个有序数组，小于7的时候使用插入排序可以更节省栈空间）
- 归并前判断一下是否还有必要归并（如果左边的最大的比右边的最小的还小(或者等于)，那就不用归并了，已经有序了）
- 只在排序前开辟一次空间（如文中所述， `System.copyOfRange`每次会分配空间，`System.arraycopy` 只是每次在已经开辟的空间上复制 ） 

#### 方法一 递归归并
```Java
    public int reversePairs(int[] arr) {
        int[] temp = new int[arr.length];
        return sort(arr, 0, arr.length - 1, temp);
    }

    // 变成 [left, mid] [mid + 1, right]的闭区间
    public int sort(int[] arr, int left, int right, int[] temp) {
        if (left >= right) return 0;
        // mid + 1的时候left可能会比right大1

        int mid = (right - left) / 2 + left; // 防止溢出
        int leftReverse = sort(arr, left, mid, temp);
        int rightReverse = sort(arr, mid + 1, right, temp);

        // 对已经排序后，两个有序数组来说，此处发现已经有序，不用继续
        if (arr[mid] <= arr[mid + 1]) return leftReverse + rightReverse;
        return mergeS(arr, left, mid, right, temp) + leftReverse + rightReverse;
    }

    private int mergeS(int[] arr, int l, int m, int r, int[] temp) {
        int total = r - l + 1;
        // 拷贝一遍当前排序状态的[l, r]当作辅助数组
        // 一个用于指针遍历，一个用于存放结果
        // 这里是修改了原数组存放结果，拷贝的辅助数组来遍历指针
        System.arraycopy(arr, l, temp, l, total);

        int reverseNum = 0;
        int p = l;
        int q = m + 1;
        for (int i = l; i <= r; i++) {
            if (p > m) {    // 左区间指针已经走完
                arr[i] = temp[q++];
                // 已经没有
            } else if (q > r) { // 右区间指针已经走完
                arr[i] = temp[p++];
            } else if (temp[p] <= temp[q]) { // 辅助数组大于等于时，归并排序才是稳定排序（指相同数字依然按照原数组顺序排序）
                arr[i] = temp[p++];  // 左区间更小
            } else {  // 右区间更小
                arr[i] = temp[q++];
                // 右指针前进时，左区间尚存的数都是该数都逆序数（两数组各自有序，左区间尚存的数肯定比右指针之前一个指向的数大）
                reverseNum += m - p + 1;
            }
        }
        return reverseNum;
    }
```

### 354. 俄罗斯套娃信封问题
长宽都需要大于另一个信封时才能被套娃
1. 按长升序，宽降序
> 长相同的信封不能套娃，所以长相同时需要按宽降序，忽略相同的长：使用LIS寻找不连续的上升序列的时候可以自动忽略相同的长
2. 使用LIS对宽进行LIS，寻找不连续的上升序列

### 最长连续序列
并查集，用v和v+1并组，每一个数字a的parent就是并查集里该组对应的最大数字b，`b - a + 1` 就是的距离取max

###
```Java
public String[] largestNumberList(int[] nums) {
    // Get input integers as strings.
    String[] asStrs = new String[nums.length];
    for (int i = 0; i < nums.length; i++) {
        asStrs[i] = String.valueOf(nums[i]);
    }

    // 把问题丢给string...
    Arrays.sort(asStrs, (a, b) -> (b + a).compareTo(a + b)); // 降序

    // 特别地，所有数都为0的情况下，返回0（因为前导0是要去掉的）
    if (asStrs[0].equals("0")) {
        return "0";
    }
    return asStr;
}
```

### 46. 全排列
> 总个数是卡特兰数
方法一，回溯，不重复元素下标，然后对应两个位置交换。这种不保证字典序
```Java
public void recursion(List<Integer> a, int k) {
    if (index == nums.length) {
        res.add(new ArrayList<>(a));
        return;
    }
    for (int i = 0; i < total; i++) {
        for (int j = i; j < total; j++) {
            swap(a, i, j); // 注意重要的点在这里
            recursion(a, k + 1);
            swap(a, j, i);
        }
    }
}
```
方法二，保证字典序，for循环每个位置选a里的数组，找出所有组合，然后用used去重
```Java
    public List<List<Integer>> permute(int[] nums) {
        allComposeWithNoMap(0, nums, new ArrayList<>());
        return resAll;
    } 
    
    private void allComposeNext(int index, int[] nums, List<Integer> res, Set<Integer> used) {
        if (index == nums.length) {
            resAll.add(new ArrayList<>(res));
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (used.contains(nums[i])) continue;
            res.add(nums[i]);
            used.add(nums[i]);
            allComposeNext(index + 1, nums, res, used);
            res.remove(res.size() - 1);
            used.remove(nums[i]);
        }
    }
```
### 329. 矩阵中的最长递增路径
- 从某个位置出发的最大步数是固定的。
- 由于只用记步数，所以dfs四个方向最大的步数可以记忆化记下来
- 经过自己需要++1
- memo记忆化数组不为空，表示搜索过以这个点出发

也可以用BFS+拓扑排序
设(i,j)比相邻点大，出度为0，这就是边界条件
计算全图出度，从所有出度为0的点开始BFS
每遍历一个点，就拓扑排序去掉一个出度，若出度减为0，则入队BFS
结果为BFS层序遍历的次数

### 螺旋矩阵
可以构造`visited[]`数组记录去过的地方，然后用`dir[]`数组记录要走的方向，这个会比较通用一点的解法。
简单解法是构造(l, t, r, b)方框，每走完一边就缩小范围，以下两个方法都是这样做的
#### 方法一

```Java
//    ---------
//    |       |
//    |       |
//    --------|
    private static List<Integer> calcBounds(int[][] matrix) {
        List<Integer> res = new ArrayList<>();
        int row = matrix.length;
        int col = matrix[0].length;
        int l = 0, t = 0, b = row - 1, r = col - 1;
        while (true) {
            for (int i = l; i <= r; i++) res.add(matrix[t][i]);
            
            if (++t > b) break;
            for (int i = t; i <= b; i++) res.add(matrix[i][r]);
            
            if (--r < l) break;
            for (int i = r; i >= l; i--) res.add(matrix[b][i]);
            
            if (--b < t) break;
            // 此处t已经下挪过一了
            for (int i = b; i >= t; i--) res.add(matrix[i][l]);
            if (++l > r) break;
        }
        return res;
    }
}
```

#### 方法二

```Java
//    ---------
//    |       |
//    |       |
//    |-------|
    private static List<Integer> calcBounds1(int[][] matrix) {
        List<Integer> order = new ArrayList<Integer>();
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return order;
        }
        int rows = matrix.length, columns = matrix[0].length;
        int l = 0, r = columns - 1, t = 0, b = rows - 1;

        while (l <= r && t <= b) {
            for (int column = l; column <= r; column++) {
                order.add(matrix[t][column]);
            }
            for (int row = t + 1; row <= b; row++) {
                order.add(matrix[row][r]);
            }
            if (l < r && t < b) {
                for (int column = r - 1; column > l; column--) {
                    order.add(matrix[b][column]);
                }
                for (int row = b; row > t; row--) {
                    order.add(matrix[row][l]);
                }
            }
            // 统一缩小
            l++;
            r--;
            t++;
            b--;
        }
        return order;
    }
```
### 约瑟夫环问题
 [见此](https://blog.csdn.net/lei396601057/article/details/109312337)


1. [42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

2. [84. 柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

3. 保卫王国 - 牛客

### 矩阵最长递增路径
同一个单元格对应的最长递增路径的长度是固定不变的。
所以记忆化搜索从`memo[i][j]`开始的最长递增路径

### 无损压缩算法：deflate、Huffman Coding


### 寻找两个有序数组的中位数
要求算法的时间复杂度为 O(log(m + n))
即找出第(m+n)/2个元素
设k=(m+n)/2，则需要在两个数组中分别寻找第k/2个元素，原因如下：

<!--more-->

题目中要求的时间复杂度为 O(log(m + n))，很容易想到的方法就是二分，现在有两个数组，要对那个数组进行二分合适？由于找的是中位数，那么这个数字的两边的元素个数是相等的，所以只需要确定一个数组中的两边元素，两一个数组的对应的补上去就可以了，为了提高效率，要选择最短的数组做二分查找 — [原文](https://blog.csdn.net/Shuffle_Ts/article/details/93142735)


### 为什么Java中int型数据取值范围是[-2^{31}, 2^{31}-1]
[为什么Java中int型数据取值范围是[-2^{31}, 2^{31}-1]](https://blog.csdn.net/AlpinistWang/article/details/87994617?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)


### 判断一个有符号int型整数是不是溢出：
x > INT_MAX / 10 || x == INT_MAX && x * 10 + 7 就会溢出
x < INT_MIN / 10 || x == INT_MIN. &&  x * 10 + 8 就会溢出

### 反转数字
push一位数字x：int res = res * 10 + x
Pop一位数字：res /= 10
需要警惕是否溢出

### Lru最近访问
Lru在Java里就是LinkedHashMap有序字典，每次get/put后把该元素挪动到双向链表尾。双向链表是因为方便删除，是所有数据的。
 
### 2、3、5硬币面值（无限个数），构成总数为k的方法数
和上楼梯很像，区别在于上楼梯只是1、2两步
> 上楼梯的状态方程： `dp[0] = dp[1] = 1, dp[i] = dp[i-1] + dp[i-2] (i > 2)`。设置dp[0] = 1只是为了后续好加；
这一题就是`dp[1] = 1, dp[i] = dp[i-2] + dp[i-3] + dp[i-5]`，意义是为了总数为i的金额是由`i-2`的情况和`i-3`和`i-5`的情况达成的


### 300. 最长上升子序列 LIS
#### 方法一 动态规划
设`dp[i]`保存[0, i]的最长上升子序列 （可以不连续）
中找到的一个比当前数`i`小的数`j`，如果第`j`个数字小于第`i`个数字，`i`就可以跟在`j`后面 `dp[i] = Math.max(dp[i], dp[j] + 1)`（`0 <= j < i`）
```Java
    public int lengthOfLIS(int[] nums) {
        int total = nums.length;
        int[] dp = new int[total];
        // Arrays.fill(dp, 1);
        dp[0] = 1; //注意都要初始化为1，因为自己就是一个长度为1的序列
        int max = 1;
        for (int i = 1; i < total; i++) {
            dp[i] = 1; 
            for (int j = 0; j < i; j++) {
                if (nums[i] > nums[j]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            max = Math.max(max, dp[i]);
        }
        return max;
    }
```


#### 方法二 贪心 + 二分
要使上升子序列尽可能的长，得让序列上升得尽可能慢，在上升子序列最后加上的那个数尽可能的小。
基于上面的贪心思路，维护一个数组 d[i] ，录目前最长上升子序列的长度，遍历数组看是否能二分插入其中获得最小值
最后数组的长度就是LIS
```Java
    public int lengthOfLIS(int[] nums) {
        int total = nums.length;
        int len = 0; // len = 下次要插入的地方
        int[] a = new int[total + 1]; // minElements 保存所有最小的上升数字，如果更小的数字可以插入到这个数组里，则替换掉
        for (int i = 0; i < total; i++) {
            // find where to insert in minElements 寻找在比它更小的数字后面插入
            int l = 0, r = len - 1;
            while (l <= r) {
                int mid = (r + l) >> 1;
                if (a[mid] < nums[i]) {
                    l = mid + 1; // 插入到这个数后
                } else {
                    r = mid - 1;
                }
            }
            // every element is larger than m[i], clear the array and set to m[0] 没找到比它更小的数字
            a[l] = nums[i];
            if (l == len) { // 比所有位置都大，需要在后面新加一个
                len++;
            }
        }
        return len;
    }
```

## 击鼓传花
dp  
第 i 个同学，可以收到第(i-1)个同学或者第(i+1)个同学的花。
`dp[i][j]=dp[(i-1)][(j-1+n)%n]+dp[(i-1)][(j+1+n)%n];`

[击鼓传花 - 动态规划](https://www.cnblogs.com/haimishasha/p/11296438.html)

## 字符压缩算法（字符串解码）
(括号匹配)  用栈


## 反转链表II
```Java

    public static class SolutionReverseLinkedListNodeII {
        //思路：head表示需要反转的头节点，pre表示需要反转头节点的前驱节点
        //我们需要反转n-m次，我们将head的next节点移动到需要反转链表部分的首部，需要反转链表部分剩余节点依旧保持相对顺序即可
        //比如1->2->3->4->5,m=1,n=5
        //第一次反转：1(head) 2(next) 3 4 5 反转为 2 1 3 4 5
        //第二次反转：2 1(head) 3(next) 4 5 反转为 3 2 1 4 5
        //第三次发转：3 2 1(head) 4(next) 5 反转为 4 3 2 1 5
        //第四次反转：4 3 2 1(head) 5(next) 反转为 5 4 3 2 1
        public ListNode reverseBetween(ListNode head, int m, int n) {
            ListNode dummy = new ListNode(0);
            dummy.next = head;
            ListNode prev = dummy;
            while (m > 1) {
                prev = prev.next;
                m--;
                n--;
            }
            head = prev.next;

            // 1 2 3 4 5
            while (n > 1) {
                ListNode next = head.next; //    3 （head指向的元素一直不变）
                head.next = head.next.next;   //     1-> 4
                next.next = prev.next;   //   3 -> 2 （prev.next是上一次的next，永远是反转部分的第一个，因为prev的位置不变）
                prev.next = next;  //  0 -> 2
                n--;
            }
            return dummy.next;
        }
    }
```


# 73. 矩阵置零
问题在于如何提醒是0
方法一：扫一遍有0的地方，额外开两个数组记下i和j。再遍历一次将出现的地方置0
方法二：扫一遍有0的地方标记为一个特殊值，再次遍历一次将特殊值出现的地方置0
方法三：由于可能不知道特殊值设置为什么，所以把出现0的地方对应的第一行第一列标记为0。第一行第一列需要一开始就判断有没有0。
    最后根据第一行第一列的是否有0来置0。