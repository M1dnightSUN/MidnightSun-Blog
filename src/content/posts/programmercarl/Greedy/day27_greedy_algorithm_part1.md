---
title: Day27-贪心算法 part01
published: 2025-04-07
description: 贪心算法，分发饼干，摆动序列，最大子数组和
tags: [C++, 贪心算法, 数组, 双指针, 排序, 动态规划]
category: 算法训练营
draft: false
---

## 贪心算法基础

贪心算法（Greedy Algorithm）是一种在每一步选择中都采取当前状态下最优或最有利的选择，从而希望通过一系列局部最优的选择，最终得到全局最优解的方法。

贪心算法的核心思想是利用局部最优解推导出全局最优解。它通常用于解决一些最优化问题，如最小生成树、最短路径、活动选择等。贪心算法的关键在于选择标准和选择过程，选择标准决定了每一步的选择，而选择过程则是根据当前状态进行决策。

它的工作方式是：

1. 从问题的某个初始状态出发；
2. 在每一步中都做出一个局部最优选择（即当前看来最好的选择）；
3. 求出每一步的的最优解；
4. 最终得到一个可行解， **但不一定是最优解**，这取决于问题本身是否适合贪心策略。

贪心算法并不总是适用，它适用于具有“贪心选择性质”和“最优子结构”的问题。

- 贪心选择性质：可以通过局部最优的选择构造出全局最优解；
- 最优子结构：一个问题的最优解包含其子问题的最优解。

## 分发饼干

[455. 分发饼干](https://leetcode.cn/problems/assign-cookies/)

这里我们可以对两个数组进行排序，然后在每一次试行中，将当前最小的饼干分配给当前最小的孩子，如果当前最小的饼干不能满足当前最小的孩子，那么就将下一个饼干分配给当前最小的孩子。

所以这里的代码其实很简单：

```cpp
class Solution {
public:
    int findContentChildren(vector<int>& g, vector<int>& s) {
        sort(g.begin(), g.end());
        sort(s.begin(), s.end());
        int count = 0;
        int i = 0, j = 0;
        while (i < g.size() && j < s.size()) {
            if (g[i] <= s[j]) { // 当前饼干值满足当前胃口值
                count++;
                i++;
                j++;
            }
            else { // 当前饼干值不满足当前胃口值，小孩不动，饼干前移
                j++;
            }
        }
        return count;
    }
};
```

或者是，尽量使用大的饼干去满足胃口大的小孩，其思路是一样的。

## 摆动序列

[376. 摆动序列](https://leetcode.cn/problems/wiggle-subsequence/)

## 贪心算法

局部最优：删除单调坡度上的节点（不包括单调坡度两端的节点），那么这个坡度就可以有两个局部峰值。

整体最优：整个序列有最多的局部峰值，从而达到最长摆动序列。

在计算是否有峰值的时候，计算 `prediff(nums[i] - nums[i-1])` 和 `curdiff(nums[i+1] - nums[i])`，如果 `prediff < 0 && curdiff > 0` 或者 `prediff > 0 && curdiff < 0` 此时就有波动就需要统计。

那么这里有三种情况需要考虑：
1. 上下坡中有平坡

![](https://file.kamacoder.com/pics/20230106170449.png)
![](https://file.kamacoder.com/pics/20230106172613.png)

在图中，当 `i` 指向第一个 2 的时候，`prediff > 0 && curdiff = 0` ，当 `i` 指向最后一个 2 的时候 `prediff = 0 && curdiff < 0`。

如果我们采用，删左面三个 2 的规则，那么 当 `prediff = 0 && curdiff < 0` 也要记录一个峰值，因为他是把之前相同的元素都删掉留下的峰值。

所以我们记录峰值的条件应该是： `(preDiff <= 0 && curDiff > 0) || (preDiff >= 0 && curDiff < 0)`

2. 数组首位两端

![](https://file.kamacoder.com/pics/20201124174357612.png)

因为我们在计算 `prediff(nums[i] - nums[i-1])` 和 `curdiff(nums[i+1] - nums[i])` 的时候，至少需要三个数字才能计算，而数组只有两个数字。

这里我们可以写死，就是如果只有两个元素，且元素不同，那么结果为 2。

因此我们可以规定，最前面还有一个数字，他的值等于 `nums[0]`，这样它就有坡度了即 `preDiff = 0`。
针对以上情形，result 初始为 1（默认最右面有一个峰值），此时 `curDiff > 0 && preDiff <= 0`，那么 `result++`（计算了左面的峰值），最后得到的 `result` 就是 2（峰值个数为 2 即摆动序列长度为 2）

3. 单调坡中有平坡

![](https://file.kamacoder.com/pics/20230108171505.png)

图中，我们可以看出，如果用情况1在三个地方记录峰值，但其实结果因为是 2，因为单调中的平坡不能算峰值（即摆动）。

我们只需要在这个坡度摆动变化的时候，更新 `prediff` 就行，这样 `prediff` 在 单调区间有平坡的时候就不会发生变化.

所以整体代码如下：

```cpp
class Solution {
public:
    int wiggleMaxLength(vector<int>& nums) {
        if (nums.size() <= 1) return nums.size();
        int curDiff = 0; // 当前一对差值
        int preDiff = 0; // 前一对差值，默认最前面还有一个数等于nums[0]
        int result = 1;  // 记录峰值个数，序列默认序列最右边有一个峰值
        for (int i = 0; i < nums.size() - 1; i++) { // 到倒数第二个
            curDiff = nums[i + 1] - nums[i];
            // 出现峰值
            if ((preDiff <= 0 && curDiff > 0) || (preDiff >= 0 && curDiff < 0)) {
                result++;
                preDiff = curDiff; // 注意这里，只在摆动变化的时候更新prediff
            }
        }
        return result;
    }
};
```

### 贪心 + 动态规划

我们维护两个变量：

- `up[i]`：以第 `i` 个元素结尾，最后一个波动是上升时的最长摆动序列长度
- `down[i]`：以第 `i` 个元素结尾，最后一个波动是下降时的最长摆动序列长度
- `up[i]` 和 `down[i]` 的初始值都为1，因为单个元素本身就是一个摆动序列。

但我们可以把它进一步简化成两个变量 `up` 和 `down`，因为我们只关心上一个状态（不需要保存整个数组）。

具体思路
- 如果 `nums[i] > nums[i-1]`，说明当前是一个上升，那么我们希望它接在一个下降后面，才能构成「摆动」。
    - 所以：`up = down + 1`
- 如果 `nums[i] < nums[i-1]`，说明当前是一个下降，希望它接在一个上升后面：
    - 所以：`down = up + 1`
- 如果两者相等，那就忽略它（不会更新任何一个）。

我们以题中给出的例子 `nums = [1, 7, 4, 9, 2, 5]` 为例，输入：`nums = [1, 7, 4, 9, 2, 5]`，`i` 从 1 开始：

| i | nums[i] | nums[i]-nums[i-1] | up  | down |
|---|---------|-------------------|-----|------|
| 1 | 7       | 6  ↑              | 2   | 1    |
| 2 | 4       | -3 ↓              | 2   | 3    |
| 3 | 9       | 5  ↑              | 4   | 3    |
| 4 | 2       | -7 ↓              | 4   | 5    |
| 5 | 5       | 3  ↑              | 6   | 5    |

整体代码如下：

```cpp
int wiggleMaxLength(vector<int>& nums) {
    if (nums.size() < 2) return nums.size(); // 只有一个元素，返回1
    int up = 1, down = 1; // up和down的初始值都为1，因为单个元素本身就是一个摆动序列
    for (int i = 1; i < nums.size(); ++i) { // 从第二个元素开始遍历
        if (nums[i] > nums[i - 1]) up = down + 1; // 当前是一个上升，那么我们希望它接在一个下降后面，才能构成「摆动」
        else if (nums[i] < nums[i - 1]) down = up + 1; // 当前是一个下降，希望它接在一个上升后面
    }
    // 整个过程中并不知道最终的波动是“升”还是“降”
    // 所以我们需要记录两种情况的最大长度
    // 最终返回 max(up, down)，才是整个序列中最长的摆动子序列长度。
    return max(up, down);
}
```

## 最大子数组和

[53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/)

局部最优：当前“连续和”为负数的时候立刻放弃，从下一个元素重新计算“连续和”，因为负数加上下一个元素 “连续和”只会越来越小。

因此我们可以遍历 `nums` 数组，计算当前的连续和，如果当前的连续和小于 0，那么就将当前的连续和置为 0, 并且从下一个元素重新计算连续和。

同时我们还需要维护一个变量 `maxSum`，用来记录当前的最大连续和，因此不会出现类似错过某一个最大连续和的情况。

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int maxSum = INT_MIN; // 记录当前的最大连续和
        int curSum = 0; // 记录当前的连续和
        for (int i = 0; i < nums.size(); i++) {
            curSum += nums[i]; // 当前的连续和
            maxSum = max(maxSum, curSum); // 更新最大连续和
            if (curSum < 0) { // 如果当前的连续和小于0，那么就放弃当前的连续和，从下一个元素重新计算
                curSum = 0;
            }
        }
        return maxSum;
    }
};
```