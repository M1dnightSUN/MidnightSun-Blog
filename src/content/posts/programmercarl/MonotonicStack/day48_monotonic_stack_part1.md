---
title: Day48-单调栈 part01
published: 2025-04-28
description: 单调栈，每日温度，下一个更大元素I，下一个更大元素II
tags: [C++, 栈, 单调栈]
category: 算法训练营
draft: false
---

## 单调栈

单调栈是一种栈结构，但有特殊要求：

栈里的元素是单调递增或者单调递减排列的。

通常分为两种：

- 单调递增栈：栈内元素从栈底到栈顶是递增的。

- 单调递减栈：栈内元素从栈底到栈顶是递减的。

注意：这里的"单调"指的是栈内元素大小关系，而不是压入栈的顺序。

核心思想：
> 维护一个栈，在遍历数组时动态保持栈的单调性，一旦遇到破坏单调性的元素，就弹出栈顶，处理对应关系。

## 每日温度

[739. 每日温度](https://leetcode.cn/problems/daily-temperatures/)

用单调递减栈：

- 栈中存放下标，对应的温度是单调递减的。

- 当新温度比栈顶元素温度高时，说明找到了栈顶元素的下一个更大温度。

具体实现思路：

1. 初始化一个空栈 `stk`，和一个与 `temperatures` 长度相同的 `result` 数组，初始为全 0。

2. 从左到右遍历 `temperatures`：

    - 当前温度大于栈顶所代表的温度，说明找到了更高温度。

    - 弹出栈顶元素，更新 `result[栈顶下标] = 当前下标 - 栈顶下标`。

    - 然后继续比较，直到栈为空或栈顶温度大于当前温度。

    - 当前下标压入栈中。

3. 最后栈中剩下的元素没有更高温度，对应的 `answer[i] = 0`（初始就是 0，不需要再动）。

整体代码如下:

```cpp
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        vector<int> result(temperatures.size(), 0); // 初始化结果数组，默认值为0
        stack<int> stk; // 单调栈，存储索引

        for (int i = 0; i < temperatures.size(); i++) {
            // 当栈不为空且当前温度大于栈顶温度时，说明找到了更高的温度
            while (!stk.empty() && temperatures[i] > temperatures[stk.top()]) {
                int index = stk.top(); // 获取栈顶索引
                stk.pop(); // 弹出栈顶元素
                result[index] = i - index; // 计算天数差并存入结果数组
            }
            stk.push(i); // 将当前索引压入栈中
        }
        return result; // 返回结果数组
    }
};
```

## 下一个更大元素 I

[739. 下一个更大元素 I](https://leetcode.cn/problems/next-greater-element-i/)

思路：

预处理 nums2 中每个元素的下一个更大元素，然后查询 nums1。

用单调递减栈：
- 栈中存放 nums2 的元素。
- 遇到比栈顶大的元素时，栈顶元素的下一个更大元素就是当前元素。

具体步骤：
1. 用一个哈希表 map，记录每个元素对应的下一个更大元素。

2. 遍历 `nums2`：

    - 栈为空或者当前元素小于等于栈顶元素，入栈。

    - 如果当前元素大于栈顶元素，弹栈，并在 `map` 中记录：`map[栈顶元素] = 当前元素`。

3. 遍历 `nums1`：

    - 查找每个元素对应的下一个更大元素，如果没有则返回 -1。

整体实现：

```cpp
class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
        unordered_map<int, int> nextGreaterMap; // 存储每个元素的下一个更大元素
        stack<int> stk; // 单调栈，存储索引

        for (int i = 0; i < nums2.size(); i++) {
            // 当栈不为空且当前元素大于栈顶元素时，说明找到了下一个更大元素
            while (!stk.empty() && nums2[i] > nums2[stk.top()]) {
                nextGreaterMap[nums2[stk.top()]] = nums2[i]; // 存储下一个更大元素
                stk.pop(); // 弹出栈顶元素
            }
            stk.push(i); // 将当前索引压入栈中
        }

        vector<int> result; // 存储结果
        for (int num : nums1) {
            result.push_back(nextGreaterMap.count(num) ? nextGreaterMap[num] : -1); // 如果没有下一个更大元素，则返回-1
        }
        return result; // 返回结果数组
    }
};
```

## 下一个更大元素 II

[503. 下一个更大元素 II](https://leetcode.cn/problems/next-greater-element-ii/)

处理方式和上题差不多，只是需要循环两遍数组来模拟“环形”。

用单调递减栈：

- 遍历两遍数组（2n长度），用取模操作 `i % n` 访问元素。

具体步骤：
1. 初始化一个栈，存放下标。

2. 初始化一个数组 `answer`，全设为 -1。

3. 遍历 `2 * n` 次：

    - 当前元素大于栈顶元素代表找到了下一个更大的元素。

    - 弹出栈，更新结果。

    - 只在第一次遍历（i < n）时压栈，避免重复压入元素。

4. 最终得到所有答案。

整体代码：

```cpp
class Solution {
public:
    vector<int> nextGreaterElements(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n, -1); // 初始化结果数组，默认值为-1
        stack<int> stk; // 单调栈，存储索引

        for (int i = 0; i < 2 * n; i++) {
            int index = i % n; // 处理循环数组的索引
            // 当栈不为空且当前元素大于栈顶元素时，说明找到了下一个更大元素
            while (!stk.empty() && nums[index] > nums[stk.top()]) {
                result[stk.top()] = nums[index]; // 存储下一个更大元素
                stk.pop(); // 弹出栈顶元素
            }
            if (i < n) { // 只在第一次遍历时将索引压入栈中
                stk.push(index); // 将当前索引压入栈中
            }
        }
        return result; // 返回结果数组
    }
};
```