---
title: Day46-动态规划 part13
published: 2025-04-25
description: 动态规划，回文子串，最长回文子序列 
tags: [C++, 动态规划]
category: 算法训练营
draft: false
---

## 647. 回文子串

[647. 回文子串](https://leetcode.cn/problems/palindromic-substrings/)

#### 1. 确定dp数组以及下标的含义

> 定义一个二维数组 `dp[i][j]`，表示字符串 `s` 中从下标 `i` 到下标 `j` 这一子串是否是回文子串。

如果是回文子串，则 `dp[i][j] = true`，否则为 `false`。

#### 2. 确定递推公式

- 如果 `s[i] == s[j]`，并且：
  - `j - i <= 1`，说明子串长度是 1 或 2，直接就是回文子串；
  - 或者 `dp[i+1][j-1] == true`，说明去掉两端字符后，中间子串是回文，那么整个也是回文。
- 因此递推公式为：
```cpp
if (s[i] == s[j]) {
    if (j - i <= 1) dp[i][j] = true;
    else dp[i][j] = dp[i + 1][j - 1];
}
```

#### 3. 初始化

- 初始时，`dp[i][i] = true`，单个字符本身就是回文子串。

```cpp
vector<vector<bool>> dp(s.size(), vector<bool>(s.size(), false));
for (int i = 0; i < s.size(); i++) {
    dp[i][i] = true;
}
```

#### 4. 遍历顺序

因为 `dp[i][j]` 依赖于 `dp[i+1][j-1]`，所以 `i` 要从大到小遍历，`j` 要从小到大遍历。

```cpp
for (int i = s.size() - 1; i >= 0; i--) {
    for (int j = i + 1; j < s.size(); j++) {
        ...
    }
}
```

#### 5. 整体代码

```cpp
class Solution {
public:
    int countSubstrings(string s) {
        int n = s.size();
        vector<vector<bool>> dp(n, vector<bool>(n, false));
        int result = 0;

        for (int i = n - 1; i >= 0; i--) {
            for (int j = i; j < n; j++) {
                if (s[i] == s[j]) {
                    if (j - i <= 1) dp[i][j] = true;
                    else dp[i][j] = dp[i + 1][j - 1];
                }
                if (dp[i][j]) result++;
            }
        }
        return result;
    }
};
```

## 516. 最长回文子序列

[516. 最长回文子序列](https://leetcode.cn/problems/longest-palindromic-subsequence/)

#### 1. 确定dp数组以及下标的含义

> 定义一个二维数组 `dp[i][j]`，表示字符串 `s` 中从下标 `i` 到下标 `j` 的子串内，**最长回文子序列的长度**。

#### 2. 确定递推公式

- 如果 `s[i] == s[j]`，那么两端可以构成回文，长度加 2：
```cpp
dp[i][j] = dp[i+1][j-1] + 2;
```
- 如果 `s[i] != s[j]`，则取删除左边或者右边元素后能得到的最长回文子序列长度的最大值：
```cpp
dp[i][j] = max(dp[i+1][j], dp[i][j-1]);
```

#### 3. 初始化

- `dp[i][i] = 1`，每个单独的字符，最长回文子序列长度就是 1。

```cpp
vector<vector<int>> dp(s.size(), vector<int>(s.size(), 0));
for (int i = 0; i < s.size(); i++) {
    dp[i][i] = 1;
}
```

#### 4. 遍历顺序

因为 `dp[i][j]` 依赖于 `dp[i+1][j-1]`、`dp[i+1][j]` 和 `dp[i][j-1]`，所以 `i` 要从大到小遍历，`j` 从小到大遍历。

```cpp
for (int i = s.size() - 1; i >= 0; i--) {
    for (int j = i + 1; j < s.size(); j++) {
        ...
    }
}
```

#### 5. 整体代码

```cpp
class Solution {
public:
    int longestPalindromeSubseq(string s) {
        int n = s.size();
        vector<vector<int>> dp(n, vector<int>(n, 0));
        for (int i = 0; i < n; i++) {
            dp[i][i] = 1;
        }

        for (int i = n - 1; i >= 0; i--) {
            for (int j = i + 1; j < n; j++) {
                if (s[i] == s[j]) {
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                } else {
                    dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[0][n - 1];
    }
};
```
