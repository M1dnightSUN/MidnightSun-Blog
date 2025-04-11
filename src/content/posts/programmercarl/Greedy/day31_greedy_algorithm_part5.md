---
title: Day31-贪心算法 part05
published: 2025-04-11
description: 贪心算法，合并区间，单调递增的数字，监控二叉树
tags: [C++, 贪心算法, 二叉树, 数组, 排序]
category: 算法训练营
draft: false
---

## 合并区间

[56. 合并区间](https://leetcode.cn/problems/merge-intervals/)

这题的本质其实还是判断重叠区间的问题。

我们可以先将区间按照起始位置进行排序，然后遍历区间

- 如果当前区间的起始位置小于等于上一个区间的结束位置，则说明两个区间重叠，需要进行合并。
- 否则，说明两个区间不重叠，可以直接将当前区间加入结果中。
- 合并区间的方式是将上一个区间的结束位置更新为两个区间的结束位置的最大值。

因此整体代码不难

```cpp
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        if (intervals.size() < 2) return intervals; // 如果区间数量小于2，直接返回
        sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
            return a[0] < b[0]; // 按照区间的起始位置排序
        });
        vector<vector<int>> merged; // 存储合并后的区间
        merged.push_back(intervals[0]); // 将第一个区间加入结果
        for (int i = 1; i < intervals.size(); i++) {
            // 如果当前区间的起始位置小于等于上一个区间的结束位置，进行合并
            if (intervals[i][0] <= merged.back()[1]) {
                merged.back()[1] = max(merged.back()[1], intervals[i][1]); // 更新结束位置
            } else {
                merged.push_back(intervals[i]); // 否则，直接加入结果
            }
        }
        return merged; // 返回合并后的区间
    }
};
```

## 单调递增的数字

[738. 单调递增的数字](https://leetcode.cn/problems/monotone-increasing-digits/)

这里的思路是我们从后往前遍历数组，如果当前的数字小于前一个数字(`s[i] < s[i - 1]`)，那么我们记录下当前这个位置 `flag`，并将前一个数字减1, 重复这个过程知道遍历结束。

然后我们将 `flag` 位置之后的数字全部改为9，最后返回结果。

整体代码如下：

```cpp
class Solution {
private:
    bool check(int n) {
        string s = to_string(n); // 将数字转换为字符串
        for (int i = 1; i < s.size(); i++) { 
            // 如果当前数字小于前一个数字，返回false
            if (s[i] < s[i - 1]) {
                return false;
            }
        }
        return true;
    }
public:
    int monotoneIncreasingDigits(int n) {
        if (check(n)) return n; // 如果已经是单调递增的数字，直接返回
        string s = to_string(n);
        int flag = s.size(); // 标记位置
        for (int i = s.size() - 1; i > 0; i--) { // 从后往前遍历
            // 如果当前数字小于前一个数字，更新标记位置
            if (s[i] < s[i - 1]) {
                flag = i;
                s[i - 1]--; // 前一个数字减1
            }
        }
        // 将标记位置之后的数字全部改为9
        for (int i = flag; i < s.size(); i++) {
            s[i] = '9';
        }
        return stoi(s); // 将字符串转换为整数并返回
    }
};
```
这里需要注意的是只有从后向前遍历才能重复利用上次比较的结果。

## 监控二叉树

[968. 监控二叉树](https://leetcode.cn/problems/binary-tree-cameras/)

#TODO

