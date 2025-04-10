---
title: Day30-贪心算法 part04
published: 2025-04-10
description: 贪心算法，引爆气球，无重叠区间，划分字母区间
tags: [C++, 贪心算法, 数组, 排序]
category: 算法训练营
draft: false
---

## 用最少数量的箭引爆气球

[452. 用最少数量的箭引爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)

为了尽量使气球重叠，我们可以将气球的右端点进行排序。
我们可以从第一个气球开始，记录当前的箭头位置，然后遍历后面的气球，如果当前气球的左边界大于箭头位置，则需要增加一支箭头，并更新箭头位置为当前气球的右边界。

如果当前气球的左边界小于等于箭头位置，则说明当前气球可以被当前箭头引爆，所以不需要增加箭头。

- 局部最优解：
    - 每次选择右端点最小的气球进行引爆

```ini
points = [[10,16],[2,8],[1,6],[7,12]]
排序后：
points = [[1,6],[2,8],[7,12],[10,16]]
    
           end        
           ↓
     1 --- 6
     
            2 --- 8       左端点小于end，不需要增加箭头
                        
                      end
                       ↓
                7 --- 12     此时的end为6，左端点大于end，需要增加箭头，并更新end为12

                    10 --- 16   此时左端点小于end，不需要增加箭头
```


```cpp
class Solution {
public:
    int findMinArrowShots(vector<vector<int>>& points) {
        if (points.empty()) return 0; // 如果没有气球，返回0
        // 按照气球的结束位置进行排序
        sort(points.begin(), points.end(), [](const vector<int>& a, const vector<int>& b) {
            return a[1] < b[1];
        });
        
        int arrows = 1; // 至少需要一支箭
        int end = points[0][1]; // 第一个气球的右边界

        for (int i = 1; i < points.size(); i++) {
            // 如果当前气球的开始位置大于上一个气球的结束位置，增加箭的数量
            if (points[i][0] > end) {
                arrows++;
                end = points[i][1]; // 更新结束位置
            }
        }

        return arrows;
    }
};
```

当然这里也可以依据左边界进行排序。

我们同样的使用一个 `end` 变量来记录当前箭头的结束位置，也就是当前箭头可以引爆的气球的右边界。

- 如果当前气球的左边界大于 `end`，说明当前气球不能被当前箭头引爆，需要增加一支箭头，并更新 `end` 为当前气球的右边界。

- 如果当前气球的左边界小于等于 `end`，说明当前气球可以被当前箭头引爆，所以不需要增加箭头。
    - 此时我们需要更新 `end` 为当前气球的右边界和 `end` 的最小值，因为我们要尽量让气球重叠。

```cpp
class Solution {
public:
    int findMinArrowShots(vector<vector<int>>& points) {
        if (points.empty()) return 0; // 如果没有气球，返回0
        // 按照气球的结束位置进行排序
        sort(points.begin(), points.end(), [](const vector<int>& a, const vector<int>& b) {
            return a[0] < b[0];
        });
        
        int arrows = 1; // 至少需要一支箭
        int end = points[0][1]; // 记录第一个气球的结束位置
        for (int i = 0; i < points.size(); i++) {
            // 如果当前气球的开始位置大于上一个气球的结束位置，说明需要新的箭
            if (points[i][0] > end) {
                arrows++; // 增加箭的数量
                end = points[i][1]; // 更新结束位置为当前气球的结束位置
            } else {
                // 如果当前气球的开始位置小于等于上一个气球的结束位置，更新结束位置为二者的最小值
                end = min(end, points[i][1]);
            }
        }
        return arrows;
    }
};
```

## 无重叠区间

[435. 无重叠区间](https://leetcode.cn/problems/non-overlapping-intervals/)

这里的整体思路其实和[无重叠区间](#无重叠区间)的思路是一样的，我们同样可以根据结束位置进行排序。
我们可以使用一个 `end` 变量来记录当前区间的结束位置，也就是当前区间的右边界。

- 如果当前区间的开始位置大于等于 `end`，则说明无重叠，此时我们要更新 `end` 为当前区间的右边界。

- 如果当前区间的开始位置小于 `end`，则说明有重叠，此时我们要更新 `end` 为当前区间的右边界和 `end` 的最小值，并使 `count++`。

```ini
intervals = [[1,2],[2,3],[3,4],[1,3]]
排序后：
intervals = [[1,2],[1,3],[2,3],[3,4]]
    
         end = 2        
          ↓
    1 ---  2

    1 ----------- 3       左边界小于end，count++，相当于删除了[1,3]，end更新为2（2和3的最小值）
                        
         end = 2
          ↓          
          2 ----- 3     此时的end为2，左边界等于end，说明没有重叠（指[1, 2]），并更新end为右边界3

                  3 --- 4   此时左边界大于end，说明没有重叠，更新end为4
```

```cpp
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        if (intervals.size() < 2) return 0; // 如果区间数量小于2，返回0

        sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b){return a[1] < b[1];});

        int count = 0;
        int end = intervals[0][1]; // 记录第一个区间的结束位置

        for (int i = 1; i < intervals.size(); i++) {
            // 如果当前区间的开始位置大于等于上一个区间的结束位置，说明没有重叠
            // 更新结束位置为当前区间的结束位置
            if (intervals[i][0] >= end) {
                end = intervals[i][1]; // 更新结束位置为当前区间的结束位置
            } else {
                count++; // 如果当前区间与上一个区间重叠，增加计数
                end = min(end, intervals[i][1]); // 更新结束位置为二者的最小值
            }
        }
        return count;
    }
};
```

## 划分字母区间

[763. 划分字母区间](https://leetcode.cn/problems/partition-labels/)

本题的题意可以分解为，给定任意一个字符 `ch`，该区间需要包含所有的 `ch` 字符。

我们可以使用一个 `map` 来记录每一个字符的最后出现位置。（当然也可以使用一个数组来记录 `int LastIndex[26] = -1;`）

使用一个 `start` 变量来记录当前区间的开始位置，`end` 变量来记录当前区间的结束位置。

在每次遍历中：
- 更新 `end` 为当前字符的最后出现位置和 `end` 的最大值。
- 如果当前字符的下标等于 `end`，说明当前区间已经划分完成，此时我们可以将 `start` 更新为 `i + 1`，并将 `end` 重置为 `0`。

```cpp
class Solution {
public:
    vector<int> partitionLabels(string s) {
        unordered_map<char, int> lastIndex; // 记录每个字符最后出现的位置
        for (int i = 0; i < s.size(); i++) {
            lastIndex[s[i]] = i; // 更新字符的最后出现位置
        }

        vector<int> result; // 存储划分的区间长度
        int start = 0; // 当前区间的起始位置
        int end = 0; // 当前区间的结束位置

        for (int i = 0; i < s.size(); i++) {
            end = max(end, lastIndex[s[i]]); // 更新当前区间的结束位置
            if (i == end) { // 如果当前索引等于结束位置，说明可以划分一个区间
                result.push_back(end - start + 1); // 计算区间长度并加入结果
                start = i + 1; // 更新起始位置为下一个字符
            }
        }
        return result; 
    }
};
```

## 总结

这三题的思路其实都是一个重叠区间的问题，[引爆气球](#用最少数量的箭引爆气球) 和 [无重叠区间](#无重叠区间) 是典型的区间选择问题，而[划分字母区间](#划分字母区间)是利用区间合并来进行划分的问题。都通过“优先选择最有利的区间”实现了贪心最优化。

| 题号 | 本质问题       | 贪心点               | 排序依据 |
|------|----------------|----------------------|------------------|
| 435  | 最少移除重叠区间 | 每次保留最早结束的区间 | 按 end 升序       |
| 452  | 最少箭数射爆气球 | 每次一箭射最右的可覆盖区间 | 按 end 升序       |
| 763  | 最多不重叠分区   | 当前分区的最大右边界到达时划分 | 字符最远出现位置 |

