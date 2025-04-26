---
title: Day43-动态规划 part12
published: 2025-04-25
description: 动态规划，不同的子序列，两个字符串的删除操作，编辑距离
tags: [C++, 动态规划]
category: 算法训练营
draft: false
---

## 115. 不同的子序列

[115. 不同的子序列](https://leetcode.cn/problems/distinct-subsequences/)

#### 1. 确定dp数组以及下标的含义

> 定义一个二维数组 `dp[i][j]`，表示 `s` 的前 `i` 个字符中，组成 `t` 的前 `j` 个字符的子序列个数。

#### 2. 确定递推公式

- 如果 `s[i - 1] == t[j - 1]`，那么可以有两种选择：
  - 要用 `s[i-1]` 来匹配 `t[j-1]`：即 `dp[i-1][j-1]`
  - 不用 `s[i-1]`，直接继承前面的状态：即 `dp[i-1][j]`
- 因此递推公式为：
```cpp
if (s[i - 1] == t[j - 1]) {
    dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
} else {
    dp[i][j] = dp[i - 1][j];
}
```

#### 3. 初始化

- `dp[0][0] = 1`，空字符串可以匹配空字符串。
- `dp[i][0] = 1`，任何字符串都可以通过删除全部字符得到空字符串。
- `dp[0][j] = 0`（`j > 0`），空字符串无法组成非空字符串。

```cpp
vector<vector<long long>> dp(s.size() + 1, vector<long long>(t.size() + 1, 0));
dp[0][0] = 1;
for (int i = 1; i <= s.size(); i++) dp[i][0] = 1;
```

#### 4. 遍历顺序

从 `i = 1` 遍历到 `s.size()`，`j = 1` 遍历到 `t.size()`，先遍历 `s`，再遍历 `t`。

```cpp
for (int i = 1; i <= s.size(); i++) {
    for (int j = 1; j <= t.size(); j++) {
        ...
    }
}
```

#### 5. 整体代码

```cpp
class Solution {
public:
    int numDistinct(string s, string t) {
        vector<vector<long long>> dp(s.size() + 1, vector<long long>(t.size() + 1, 0));
        for (int i = 0; i <= s.size(); i++) dp[i][0] = 1;
        
        for (int i = 1; i <= s.size(); i++) {
            for (int j = 1; j <= t.size(); j++) {
                if (s[i - 1] == t[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
                } else {
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        
        return dp[s.size()][t.size()];
    }
};
```

## 583. 两个字符串的删除操作

[583. 两个字符串的删除操作](https://leetcode.cn/problems/delete-operation-for-two-strings/)

#### 1. 确定dp数组以及下标的含义

> 定义 `dp[i][j]` 表示将 `word1` 的前 `i` 个字符和 `word2` 的前 `j` 个字符变得相同所需要删除的最少步数。

#### 2. 确定递推公式

- 如果 `word1[i-1] == word2[j-1]`，那么不需要删除字符，继承前面的状态：
  ```cpp
  dp[i][j] = dp[i-1][j-1];
  ```
- 如果 `word1[i-1] != word2[j-1]`，要么删 `word1[i-1]`，要么删 `word2[j-1]`，取最小值再加一：
  ```cpp
  dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + 1;
  ```

#### 3. 初始化

- `dp[i][0] = i`，将 `word1` 删除成空字符串
- `dp[0][j] = j`，将 `word2` 删除成空字符串

```cpp
vector<vector<int>> dp(word1.size() + 1, vector<int>(word2.size() + 1, 0));
for (int i = 0; i <= word1.size(); i++) dp[i][0] = i;
for (int j = 0; j <= word2.size(); j++) dp[0][j] = j;
```

#### 4. 遍历顺序

从 `i = 1` 遍历到 `word1.size()`，`j = 1` 遍历到 `word2.size()`，从前向后遍历。

```cpp
for (int i = 1; i <= word1.size(); i++) {
    for (int j = 1; j <= word2.size(); j++) {
        ...
    }
}
```

#### 5. 整体代码

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        vector<vector<int>> dp(word1.size() + 1, vector<int>(word2.size() + 1, 0));
        for (int i = 0; i <= word1.size(); i++) dp[i][0] = i;
        for (int j = 0; j <= word2.size(); j++) dp[0][j] = j;

        for (int i = 1; i <= word1.size(); i++) {
            for (int j = 1; j <= word2.size(); j++) {
                if (word1[i - 1] == word2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + 1;
                }
            }
        }

        return dp[word1.size()][word2.size()];
    }
};
```

---

## 72. 编辑距离

[72. 编辑距离](https://leetcode.cn/problems/edit-distance/)

#### 1. 确定dp数组以及下标的含义

> 定义 `dp[i][j]` 表示将 `word1` 的前 `i` 个字符转换成 `word2` 的前 `j` 个字符所使用的最少编辑操作次数。

编辑操作包括：插入、删除、替换。

#### 2. 确定递推公式

- 如果 `word1[i-1] == word2[j-1]`，则不需要任何操作：
  ```cpp
  dp[i][j] = dp[i-1][j-1];
  ```
- 如果 `word1[i-1] != word2[j-1]`，则考虑三种情况：
  - 插入：`dp[i][j-1] + 1`
  - 删除：`dp[i-1][j] + 1`
  - 替换：`dp[i-1][j-1] + 1`

取三者最小值：
```cpp
dp[i][j] = min({dp[i-1][j] + 1, dp[i][j-1] + 1, dp[i-1][j-1] + 1});
```

#### 3. 初始化

- `dp[i][0] = i`，把 `word1` 变成空字符串
- `dp[0][j] = j`，把空字符串变成 `word2`

```cpp
vector<vector<int>> dp(word1.size() + 1, vector<int>(word2.size() + 1, 0));
for (int i = 0; i <= word1.size(); i++) dp[i][0] = i;
for (int j = 0; j <= word2.size(); j++) dp[0][j] = j;
```

#### 4. 遍历顺序

从 `i = 1` 遍历到 `word1.size()`，`j = 1` 遍历到 `word2.size()`，先遍历 `word1` 再遍历 `word2`。

```cpp
for (int i = 1; i <= word1.size(); i++) {
    for (int j = 1; j <= word2.size(); j++) {
        ...
    }
}
```

#### 5. 整体代码

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        vector<vector<int>> dp(word1.size() + 1, vector<int>(word2.size() + 1, 0));
        for (int i = 0; i <= word1.size(); i++) dp[i][0] = i;
        for (int j = 0; j <= word2.size(); j++) dp[0][j] = j;

        for (int i = 1; i <= word1.size(); i++) {
            for (int j = 1; j <= word2.size(); j++) {
                if (word1[i - 1] == word2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = min({dp[i - 1][j] + 1, dp[i][j - 1] + 1, dp[i - 1][j - 1] + 1});
                }
            }
        }

        return dp[word1.size()][word2.size()];
    }
};
```
