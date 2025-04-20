---
title: Day38-动态规划 part06
published: 2025-04-19
description: 动态规划，完全背包问题，零钱兑换，完全平方数，单词拆分
tags: [C++, 动态规划, 背包问题, 字符串]
category: 算法训练营
draft: false
---

## 零钱兑换

[322. 零钱兑换](https://leetcode.cn/problems/coin-change/)

这题与[零钱兑换 II](https://m1dnightsun.github.io/MidnightSun-Blog/posts/programmercarl/dp/day37_dynamic_programing_part5/#%E9%9B%B6%E9%92%B1%E5%85%91%E6%8D%A2-ii)的区别在于：本题在于求最少的硬币数量，而零钱兑换 II 是求组合数。

转换为背包问题：
- 背包容量：`amount`，即目标金额。
- 物品：`coins`，即硬币的面值。
- 物品的体积：`coins[i]`，即硬币的面值。

#### 1. 确定dp数组以及下标的含义

> `dp[i]`：表示背包容量为 `i` 时，所需的最少硬币数量。

#### 2. 确定递推公式

- 要凑满 `j - coins[i]` 的金额，最少需要 `dp[j - coins[i]]` 个硬币。
- 那么要凑满 `j` 的金额，最少需要 `dp[j - coins[i]] + 1` 个硬币，也就是 `dp[j]` 的值。
- 不放硬币 `i`：背包容量为 `j`，最少需要 `dp[j]` 个硬币。
- 我们的目标是求最少的硬币数量，因此我们需要取最小值。

因此递推公式为：
```cpp
dp[j] = min(dp[j], dp[j - coins[i]] + 1);
```

#### 3. 初始化dp数组

- 首先凑满0的金额需要0个硬币，因此 `dp[0] = 0`。
- 但因为递推公式的性质，我们要求的是最小值，因此我们需要初始化 `dp[j]` 为一个较大的数。可以用 `INT_MAX` 来表示。

#### 4. 遍历顺序

本题求的是最少的硬币数量，与排列组合问题无关，一次我们的遍历顺序可以先背包或者先物品。

````cpp
for (int i = 1; i <= amount; ++i) {
    for (int j = 0; j < coins.size(); ++j) {
        if (i >= coins[j]) {
            dp[i] = min(dp[i], dp[i - coins[j]] + 1);
        }
    }
}
````

#### 5. 整体代码

```cpp
class Solution {
private:
    void printDp(const vector<int>& dp) {
        for (auto _dp : dp) {
            if (_dp == INT_MAX) cout << "INF ";
            else cout << _dp << ' ';
        }
        cout << endl;
    }

public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, INT_MAX);
        dp[0] = 0;
        
        for (auto coin : coins) { // 先遍历物品
            // cout << "coin = " << coin << endl;
            for (auto j = coin; j <= amount; j++) { // 再遍历背包
                // 边界处理，可能出现 INT_MAX + 1 溢出
                // dp[j - coin] == INT_MAX，代表此时尚未找到能凑出 j - coin 金额的方案。
                if (dp[j - coin] != INT_MAX) { 
                    dp[j] = min(dp[j], dp[j - coin] + 1);
                }
            }
            // printDp(dp);
            // cout << "----------" << endl;
        }
        return dp[amount] == INT_MAX ? -1 : dp[amount];
    }
};
```

这里因为我们的 `dp[j]` 是从 `dp[j - coin]` 推算出来的，也就是说，我们要知道凑成金额为 `j - coin` 的最小硬币数，才能推算出凑成金额为 `j` 的最小硬币数。

- 如果 `dp[j - coin]` 是 `INT_MAX`，即说明我们还没有找到凑成金额为 `j - coin` 的方案，那么我们就不能用 `dp[j - coin] + 1` 来推算出 `dp[j]`。

## 完全平方数

[279. 完全平方数](https://leetcode.cn/problems/perfect-squares/)

我们可以把这题解释为：我们可以使用任意个数的完全平方数，去凑成一个目标值 `n`，然后求出最少的完全平方数的个数。

因此我们可以把这题转换为背包问题：
- 背包容量：`n`，即目标值。
- 物品：`1, 4, 9, 16, ...`，即完全平方数。
- 物品的体积：`i * i`，即完全平方数的值。

#### 1. 确定dp数组以及下标的含义

> `dp[j]`：表示背包容量（凑成总和）为 `j` 时，所需的最少完全平方数的个数。

#### 2. 确定递推公式

- 要凑成总和 `j`，当前遍历到的完全平方数为 `i * i`，那么我们可以选择放入这个完全平方数，或者不放入。
    - 放入 `i * i`：那么我们需要凑成 `j - i * i` 的总和，最少需要 `dp[j - i * i] + 1` 个完全平方数。
- 不放入 `i * i`：那么我们需要凑成 `j` 的总和，最少需要 `dp[j]` 个完全平方数。
- 我们的目标是求最少的完全平方数的个数，因此我们需要取最小值。

因此递推公式为：
```cpp
dp[j] = min(dp[j], dp[j - i * i] + 1);
```
可以发现这里的解法其实就和[零钱兑换](#零钱兑换)问题是一样的。

#### 3. 初始化dp数组

- 首先凑满和为 0 的完全平方数的个数为 0，因此 `dp[0] = 0`。
- 但因为递推公式的性质，我们要求的是最小值，因此我们需要初始化 `dp[j]` 为一个较大的数。可以用 `INT_MAX` 来表示。

因此初始化 `dp` 数组为：
```cpp
dp[0] = 0; // 凑成总和为0的完全平方数的个数为0
for (int j = 1; j <= n; ++j) {
    dp[j] = INT_MAX; // 凑成总和为j的完全平方数的个数为无穷大
}
```

#### 4. 遍历顺序

- 本题求的是最少的完全平方数的个数，与排列组合问题无关，一次我们的遍历顺序可以先背包或者先物品。

#### 5. 整体代码

```cpp
class Solution {
private:
    void printDp(const vector<int>& dp) {
        for (auto _dp : dp) {
            if (_dp == INT_MAX) {
                cout << "INF" << " ";
            } else {
                cout << _dp << " ";
            }
        }
        cout << endl;
        cout << "--------------" << endl;
    }

public:
    int numSquares(int n) {
        vector<int> dp(n + 1, INT_MAX);
        dp[0] = 0; // 初始化：和为0需要0个完全平方数

        for (int i = 1; i * i <= n; i++) { // 遍历物品（完全平方数）
            for (int j = i * i; j <= n; j++) { // 遍历背包容量
                if (dp[j - i * i] != INT_MAX) {
                    dp[j] = min(dp[j], dp[j - i * i] + 1);
                }
            }
            // printDp(dp);
        }
        return dp[n];
    }
};  
```

## 单词拆分

[139. 单词拆分](https://leetcode.cn/problems/word-break/)

本题是一个“字符串切割 + 判断的”问题，可以类比成**字符串长度为背包容量**，**字典中的单词为物品**，判断能否恰好组成字符串。

#### 1. 确定dp数组以及下标的含义

> `dp[i]`：表示字符串 `s[0...i-1]` 是否可以被拆分成字典中的单词。

#### 2. 确定递推公式

我们枚举每一个位置 `i`，再枚举 `j（j < i）`，看 `s[j:i]` 是否是字典中的单词，如果是且 `dp[j] == true`，那么 `dp[i] = true`

> `dp[i] = dp[j] && (s[j:i] ∈ wordDict)`

#### 3. 初始化dp数组

从递推公式中可以看出，`dp[i]` 的状态依靠 `dp[j]` 是否为 `true`，那么 `dp[0]` 就是递推的基础，`dp[0]`一定要为 `true`，否则递推下去后面都是 `false` 了。

#### 4. 遍历顺序

题目中说是拆分为一个或多个在字典中出现的单词，所以这是完全背包。还要讨论两层for循环的前后顺序。

如果求组合数就是外层for循环遍历物品，内层for遍历背包。

如果求排列数就是外层for遍历背包，内层for循环遍历物品。

而本题我们要求的其实是排列，我们可以拿以下例子：

```ini
s = "applepenapple"
wordDict = ["apple", "pen"]

那么我们在物品中选择 "apple" + "pen" + "apple"，才能组成 "applepenapple"。

如果选择 "pen" + "apple" + "apple"，虽然单词都在字典中，但是不能组成 "applepenapple"。

因此这里强调了一个单词之间顺序的问题。
```

所以我们的遍历顺序应该是：外层for循环遍历背包，内层for循环遍历物品。

```cpp
for (int i = 1; i <= s.size(); i++) { // 遍历背包容量，从1~字符串长度
    for (int j = 0; j < i; j++) { // 遍历物品（单词）
        string word = s.substr(j, i - j); // 从j到i的子串
        // 如果当前子串在字典中，并且前面部分可以拆分成单词
        // dp[j] == true 代表前面部分可以拆分成单词
        // dp[i] == true 代表当前子串可以拆分成单词
        if (set.find(word) != set.end() && dp[j]) {
            dp[i] = true;
        }
    
    }
}
```

#### 5. 整体代码

```cpp
class Solution {
private:    
    void printDp(const vector<bool>& dp) {
        for (auto _dp : dp) {
            cout << _dp << " ";
        }
        cout << endl;
        cout << "--------------" << endl;
    }

public:
    bool wordBreak(string s, vector<string>& wordDict) {
        unordered_set<string> set(wordDict.begin(), wordDict.end());

        vector<bool> dp(s.size() + 1, false);
        dp[0] = true; // 空字符串可以被拆分成字典中的单词

        for (int i = 1; i <= s.size(); i++) { // 遍历背包容量，从1~字符串长度
            for (int j = 0; j < i; j++) { // 遍历物品（单词）
                string word = s.substr(j, i - j); // 从j到i的子串
                // 如果当前子串在字典中，并且前面部分可以拆分成单词
                // dp[j] == true 代表前面部分可以拆分成单词
                // dp[i] == true 代表当前子串可以拆分成单词
                if (set.find(word) != set.end() && dp[j]) {
                    dp[i] = true;
                }
            }
            // printDp(dp);
        }
        return dp[s.size()];
    }
};
```

