---
title: Day43-动态规划 part11
published: 2025-04-24
description: 动态规划，
tags: [C++, 动态规划, 子序列]
category: 算法训练营
draft: false
---

## 最长公共子序列

[1143. 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)

#### 1. 确定dp数组以及下标的含义

> 定义一个二维数组 `dp[i][j]`，表示 `text1` 的前 `i` 个字符与 `text2` 的前 `j` 个字符的最长公共子序列长度。

注意：这里的“前 `i` 个字符”不包括 `text1[i]`，即 `dp` 的下标是从 1 开始表示字符串前缀长度。

#### 2. 确定递推公式

- 如果 `text1[i - 1] == text2[j - 1]`，说明两个字符串当前字符相等，那么 `dp[i][j] = dp[i - 1][j - 1] + 1`
- 否则，`dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])`

即：
```cpp
if (text1[i - 1] == text2[j - 1]) {
    dp[i][j] = dp[i - 1][j - 1] + 1;
} else {
    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
}
```

#### 3. 初始化

- 当 `i == 0` 或 `j == 0` 时，即其中一个字符串为空，最长公共子序列为 0

```cpp
vector<vector<int>> dp(text1.size() + 1, vector<int>(text2.size() + 1, 0));
```

#### 4. 遍历顺序

我们从 `i = 1` 遍历到 `text1.size()`，`j = 1` 遍历到 `text2.size()`，并根据 `text1[i - 1]` 和 `text2[j - 1]` 进行转移。

```cpp
for (int i = 1; i <= text1.size(); i++) {
    for (int j = 1; j <= text2.size(); j++) {
        ...
    }
}
```

#### 5. 整体代码

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int m = text1.size(), n = text2.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1[i - 1] == text2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[m][n];
    }
};
```

## 1035. 不相交的线

[1035. 不相交的线](https://leetcode.cn/problems/uncrossed-lines/)

#### 1. 确定dp数组以及下标的含义

> `dp[i][j]` 表示 A 的前 `i` 个元素与 B 的前 `j` 个元素之间，最多可以连接多少条不相交的线。

#### 2. 确定递推公式

本质上与最长公共子序列类似：
- 如果 `A[i-1] == B[j-1]`，则可以连接一条线：`dp[i][j] = dp[i-1][j-1] + 1`
- 否则：`dp[i][j] = max(dp[i-1][j], dp[i][j-1])`

#### 3. 初始化

- `dp[i][0] = 0`，表示 B 为空时无法连接
- `dp[0][j] = 0`，表示 A 为空时无法连接

```cpp
vector<vector<int>> dp(A.size() + 1, vector<int>(B.size() + 1, 0));
```

#### 4. 遍历顺序

从前往后遍历 A 和 B：
```cpp
for (int i = 1; i <= A.size(); i++) {
    for (int j = 1; j <= B.size(); j++) {
        ...
    }
}
```

#### 5. 整体代码

```cpp
class Solution {
public:
    int maxUncrossedLines(vector<int>& A, vector<int>& B) {
        vector<vector<int>> dp(A.size() + 1, vector<int>(B.size() + 1, 0));
        for (int i = 1; i <= A.size(); i++) {
            for (int j = 1; j <= B.size(); j++) {
                if (A[i - 1] == B[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[A.size()][B.size()];
    }
};
```

## 53. 最大子序和

[53. 最大子序和](https://leetcode.cn/problems/maximum-subarray/)

#### 1. 确定dp数组以及下标的含义

> `dp[i]` 表示以 `nums[i]` 结尾的连续子数组的最大和。

#### 2. 确定递推公式

我们有两种选择：
- 要么将 `nums[i]` 加到 `dp[i-1]` 上，表示继续累加；
- 要么舍弃前面的子数组，从 `nums[i]` 重新开始。

```cpp
dp[i] = max(dp[i - 1] + nums[i], nums[i]);
```

#### 3. 初始化

- `dp[0] = nums[0]`，表示以第一个元素结尾的最大子数组和就是它自己。

```cpp
vector<int> dp(nums.size());
dp[0] = nums[0];
```

#### 4. 遍历顺序

从前往后遍历即可。

```cpp
for (int i = 1; i < nums.size(); i++) {
    dp[i] = max(dp[i - 1] + nums[i], nums[i]);
}
```

#### 5. 整体代码

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        vector<int> dp(nums.size());
        dp[0] = nums[0];
        int result = dp[0];
        for (int i = 1; i < nums.size(); i++) {
            dp[i] = max(dp[i - 1] + nums[i], nums[i]);
            result = max(result, dp[i]);
        }
        return result;
    }
};
```

## 392. 判断子序列

[392. 判断子序列](https://leetcode.cn/problems/is-subsequence/)

#### 1. 确定dp数组以及下标的含义

这道题不一定要用动态规划，用双指针更加简单直观。

#### 2. 算法核心思路（双指针法）

- 指针 `i` 指向 `s`，`j` 指向 `t`。
- 如果 `s[i] == t[j]`，说明匹配上了，`i++`；
- 否则 `j++`，继续匹配下一个字符。
- 如果最后 `i == s.size()`，说明 `s` 是 `t` 的子序列。

#### 3. 初始化

```cpp
int i = 0, j = 0;
```

#### 4. 遍历顺序

只需要从左往右遍历两个字符串。

#### 5. 整体代码

```cpp
class Solution {
public:
    bool isSubsequence(string s, string t) {
        int i = 0, j = 0;
        while (i < s.size() && j < t.size()) {
            if (s[i] == t[j]) i++;
            j++;
        }
        return i == s.size();
    }
};
```
