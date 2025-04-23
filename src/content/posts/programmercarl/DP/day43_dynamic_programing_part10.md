---
title: Day43-动态规划 part10
published: 2025-04-23
description: 动态规划，最长递增子序列，最长连续递增子序列，最长重复子序列
tags: [C++, 动态规划, 子序列]
category: 算法训练营
draft: false
---

## 最长递增子序列

[300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

#### 1. 确定dp数组以及下标的含义

> 我们可以定义一个一维的dp数组 `dp[i]`，表示以 `nums[i]` 结尾的最长递增子序列的长度。

#### 2. 确定递推公式

位置 `i` 的最长递增子序列的长度等于 `j` 从 `0` 到 `i - 1` 的所有位置中，`nums[j] < nums[i]` 的最大值加 `1`。

因此我们可以得到递推公式：
```cpp
if (nums[j] < nums[i]) {
    dp[i] = max(dp[i], dp[j] + 1);
}
```

#### 3. 初始化

对于每一个位置 `i`，我们都可以认为它本身就是一个递增子序列，因此我们可以将 `dp[i]` 初始化为 `1`。

```cpp
vector<int> dp(nums.size(), 1);
```

#### 4. 遍历顺序

在递推公式中，我们需要用到 `j` 从 `0` 到 `i - 1` 的所有位置，因此我们需要先遍历 `i`，再遍历 `j`，并且是从前往后遍历。

```cpp
for (int i = 0; i < nums.size(); i++) {
    for (int j = 0; j < i; j++) {
        if (nums[j] < nums[i]) {
            dp[i] = max(dp[i], dp[j] + 1);
        }
    }
}
```

#### 5. 整体代码

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> dp(nums.size(), 1); // 初始化为1
        for (int i = 0; i < nums.size(); i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) { // 如果nums[j] < nums[i]，说明可以组成递增子序列
                    dp[i] = max(dp[i], dp[j] + 1);
                }
            }
        }
        return *max_element(dp.begin(), dp.end());
    }
};
```

## 最长连续递增序列

[674. 最长连续递增序列](https://leetcode.cn/problems/longest-continuous-increasing-subsequence/)    

#### 1. 确定dp数组以及下标的含义

> 我们可以定义一个一维的dp数组 `dp[i]`，表示以 下标 `i` 结尾的最长连续递增子序列的长度。

#### 2. 确定递推公式

如果 `nums[i] > nums[i - 1]`，那么以 `nums[i]` 结尾的最长连续递增子序列的长度等于以 `nums[i - 1]` 结尾的最长连续递增子序列的长度加 `1`。

即：
```cpp
dp[i] = dp[i - 1] + 1;
```

因为是连续递增，所以我们只需要判断 `nums[i] > nums[i - 1]` 即可，不再需要判断 `j` 从 `0` 到 `i - 1` 的所有位置，也就是只用一层循环即可。

#### 3. 初始化

和之前求最长递增子序列一样，以下标 `i` 结尾的最长连续递增子序列的长度为 `1`。

```cpp
vector<int> dp(nums.size(), 1); // 初始化为1
```

#### 4. 遍历顺序

在递推公式中，我们只需要用到 `i - 1` 的位置，因此我们只需要从前往后遍历即可。

```cpp
for (int i = 1; i < nums.size(); i++) {
    if (nums[i] > nums[i - 1]) { // 如果nums[i] > nums[i - 1]，说明可以组成递增子序列
        dp[i] = dp[i - 1] + 1;
    }
}
```

#### 5. 整体代码

```cpp
class Solution {
public:
    int findLengthOfLCIS(vector<int>& nums) {
        if (nums.size() == 0) return 0; // 如果数组为空，返回0
        vector<int> dp(nums.size(), 1); // 初始化为1
        for (int i = 1; i < nums.size(); i++) {
            if (nums[i] > nums[i - 1]) { // 如果nums[i] > nums[i - 1]，说明可以组成递增子序列
                dp[i] = dp[i - 1] + 1;
            }
        }
        return *max_element(dp.begin(), dp.end()); // 返回dp的最大值
    }
};
```

## 最长重复子序列

[1143. 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)

我们可以使用一个二维的dp数组 `dp[i][j]`，表示 `AA[0...i]` 和 `B[0...j]` 的最长公共子序列的长度。

#### 1. 确定dp数组以及下标的含义

> `dp[i][j]` ：以下标 `i - 1` 为结尾的 `A`，和以下标 `j - 1` 为结尾的 `B`，最长重复子数组长度为 `dp[i][j]`。 （“以下标 `i - 1` 为结尾的 `A`” 标明一定是以 `A[i-1]` 为结尾的字符串 ）

#### 2. 确定递推公式

根据 dp数组的定义，`dp[i][j]` 由 `dp[i - 1][j - 1]` 递推而来。
- 当 `A[i - 1] == B[j - 1]` 时，`dp[i][j] = dp[i - 1][j - 1] + 1`。

#### 3. 初始化

根据 dp数组的定义，`dp[i][j]` 由 `dp[i - 1][j - 1]` 递推而来，`dp[i][0]` 和 `dp[0][j]` 实际上是没有意义的，但是在递推的过程中需要用到，因此我们可以将 `dp[i][0]` 和 `dp[0][j]` 初始化为 `0`。

```cpp
vector<vector<int>> dp(A.size() + 1, vector<int>(B.size() + 1, 0)); // 初始化为0
```

#### 4. 遍历顺序

在递推公式中，我们需要用到 `i - 1` 和 `j - 1` 的位置，因此我们需要先遍历 `i`，再遍历 `j`，并且是从前往后遍历。先遍历 `A`，再遍历 `B`。

```cpp
for (int i = 1; i <= text1.size(); i++) {
            for (int j = 1; j <= text2.size(); j++) {
                if (text1[i - 1] == text2[j - 1]) { // 如果当前字符相等，则最长公共子序列长度加1
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else { // 如果当前字符不相等，则取前一个状态的最大值
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
```

#### 5. 整体代码

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        vector<vector<int>> dp(text1.size() + 1, vector<int>(text2.size() + 1, 0)); // dp[i][j]表示text1前i个字符和text2前j个字符的最长公共子序列长度
        for (int i = 1; i <= text1.size(); i++) {
            for (int j = 1; j <= text2.size(); j++) {
                if (text1[i - 1] == text2[j - 1]) { // 如果当前字符相等，则最长公共子序列长度加1
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else { // 如果当前字符不相等，则取前一个状态的最大值
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[text1.size()][text2.size()]; // 返回最长公共子序列长度
    }
};
```

