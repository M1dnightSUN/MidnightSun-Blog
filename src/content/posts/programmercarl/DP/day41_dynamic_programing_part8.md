---
title: Day41-动态规划 part08
published: 2025-04-21
description: 动态规划，买卖股票 I & II & III
tags: [C++, 动态规划]
category: 算法训练营
draft: false
---

## 买卖股票 I

[121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)

#### 1. 确定dp数组以及下标的含义

我们可以用一个二维的dp数组来表示状态。

> `dp[i][j]` 表示第 `i` 天持有股票的最大利润，`j` 代表持有状态，`0` 代表持有，`1` 代表不持有。
> `dp[i][0]` 表示第 `i` 天持有股票的最大利润，`dp[i][1]` 表示第 `i` 天不持有股票的最大利润。

#### 2. 确定递推公式

- 如果第 `i` 天持有股票，即 `dp[i][0]`，那么有两种情况：
    - 第 `i-1` 天也持有股票，即状态和第 `i - 1` 天一样，所持金额为 `dp[i-1][0]`。
    - 第 `i-1` 天不持有股票，说明在第 `i` 天买入了股票，所得所持现金就是买入今天的股票后所得现金，也就是 `-prices[i]`。
    
    那么 `dp[i][0]` 应当选择最大的所持金额：
    ```cpp
    dp[i][0] = max(dp[i - 1][0], -prices[i]);
    ```

- 如果第 `i` 天不持有股票，即 `dp[i][1]`，那么有两种情况：
    - 第 `i-1` 天也不持有股票，即状态和第 `i - 1` 天一样，所持金额为 `dp[i-1][1]`。
    - 第 `i-1` 天持有股票，说明在第 `i` 天卖出了股票，所得所持现金就是卖出今天的股票后所得现金，那么所持金就要加上今天卖出的股票的价格，也就是 `dp[i-1][0] + prices[i]`。

    同样的，`dp[i][1]` 应当选择最大的所持金额：
    ```cpp
    dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] + prices[i]);
    ```

#### 3. 初始化dp数组

我们可以发现 `dp[i][0]` 和 `dp[i][1]` 的都是由 `dp[i-1][0]` 和 `dp[i-1][1]` 递推而来，因此我们可以先初始化 `dp[0][0]` 和 `dp[0][1]`。

- `dp[0][0]` 表示第 `0` 天持有股票的最大利润，也就是第 `0` 天就买入了股票，所持金额为 `-prices[0]`（我们假设初始现金为0）。
- `dp[0][1]` 表示第 `0` 天不持有股票的最大利润，也就是第 `0` 天不买入股票，所持金额为 `0`。

#### 4. 遍历顺序

显然我们的遍历顺序是从前往后遍历的。

#### 5. 整体代码

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if (prices.size() <= 1) return 0;
        vector<vector<int>> dp(prices.size(), vector<int>(2, 0));
        dp[0][0] = -prices[0]; // 第 0 天持有股票
        dp[0][1] = 0; // 第 0 天不持有股票

        for (int i = 1; i < prices.size(); i++) {
            dp[i][0] = max(dp[i - 1][0], -prices[i]); // 持有股票
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] + prices[i]); // 不持有股票
        }
        return max(dp[prices.size() - 1][0], dp[prices.size() - 1][1]);
    }
};
```

## 买卖股票 II

[122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

本题和买卖股票 I 的区别在于：
- 买卖股票 I 只能进行一次交易，而买卖股票 II 可以进行多次交易。

唯一不同的地方，就是推导 `dp[i][0]` 的时候，第i天买入股票的情况。

因为一只股票可以买卖多次，所以当第i天买入股票的时候，所持有的现金可能有之前买卖过的利润。

如果第 `i` 天买入股票，所得现金就是昨天不持有股票的所得现金 **减去** 今天的股票价格 即：`dp[i - 1][1] - prices[i]`。

整体代码：

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        // 动态规划
        int n = prices.size();
        if (n < 2) return 0;
        vector<vector<int>> dp(n, vector<int>(2, 0));
        dp[0][0] = -prices[0]; // 第 0 天持有股票
        dp[0][1] = 0; // 第 0 天不持有股票

        for (int i = 1; i < n; i++) {
            // 持有股票，在前一天不持有股票的基础上买入
            // 那么今天的利润就是前一天不持有股票的利润减去今天的价格
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] - prices[i]); 
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] + prices[i]); // 不持有股票
        }
        return dp[n - 1][1]; // 返回最后一天不持有股票的最大利润
    }
};
```

## 买卖股票 III

[123. 买卖股票的最佳时机 III](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)

#### 1. 确定dp数组以及下标的含义

和买卖股票 I, II 类似，我们可以用一个二维的dp数组来表示状态。

> `dp[i][j]` 表示第 `i` 天持有股票的最大利润，`j` 代表持有状态，`0` 代表不操作，`1` 代表第一次持有，`2` 代表第一次不持有，`3` 代表第二次持有，`4` 代表第二次不持有。

- `dp[i][0]` 表示第 `i` 天不操作的最大利润
- `dp[i][1]` 表示第 `i` 天第一次持有股票的最大利润
- `dp[i][2]` 表示第 `i` 天第一次不持有股票的最大利润
- `dp[i][3]` 表示第 `i` 天第二次持有股票的最大利润
- `dp[i][4]` 表示第 `i` 天第二次不持有股票的最大利润

#### 2. 确定递推公式

- 对于第 `i` 天不操作的最大利润，即 `dp[i][0]`，那么 `dp[i][0]` 就是前一天的最大利润 `dp[i - 1][0]`。

- 对于 `dp[i][1]` 第 `i` 天第一次持有股票的最大利润, 我们可以有两个操作：
    - 第 `i` 天买入股票，那么所持现金就是前一天不操作的最大利润减去今天的股票价格，即 `dp[i - 1][0] - prices[i]`。
    - 第 `i` 天不操作，那么所持现金就是前一天第一次持有股票的最大利润，即 `dp[i - 1][1]`。

    那么 `dp[i][1]` 应当选择最大的所持金额：
    ```cpp
    dp[i][1] = max(dp[i - 1][0] - prices[i], dp[i - 1][1]);
    ```

- 对于 `dp[i][2]` 第 `i` 天第一次不持有股票的最大利润, 类似的我们可以有两个操作：
    - 第 `i` 天卖出股票，那么所持现金就是前一天第一次持有股票的最大利润加上今天的股票价格，即 `dp[i - 1][1] + prices[i]`。
    - 第 `i` 天不操作，那么所持现金就是前一天第一次不持有股票的最大利润，即 `dp[i - 1][2]`。

    那么 `dp[i][2]` 应当选择最大的所持金额：
    ```cpp
    dp[i][2] = max(dp[i - 1][1] + prices[i], dp[i - 1][2]);
    ```

同理，我们可以推导出 `dp[i][3]` 和 `dp[i][4]` 的递推公式：
```cpp
dp[i][3] = max(dp[i - 1][2] - prices[i], dp[i - 1][3]);
dp[i][4] = max(dp[i - 1][3] + prices[i], dp[i - 1][4]);
```

#### 3. 初始化dp数组

第0天没有操作，就是0，即：`dp[0][0] = 0`;
第0天做第一次买入的操作，`dp[0][1] = -prices[0]`;
第0天做第一次卖出的操作, 可以理解为第一天就买入并卖出，`dp[0][2] = 0`;
第0天做第二次买入的操作，相当于第一天买入并卖出，再买入，`dp[0][3] = -prices[0]`;
第0天做第二次卖出的操作, 可以理解为第一天买入并卖出，之后再买入并卖出，`dp[0][4] = 0`;

#### 4. 遍历顺序
从递推公式我们可以看出，`dp[i][0]` 依赖于 `dp[i-1][0]`，因此我们需要从前往后遍历。

#### 5. 整体代码

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if (prices.size() < 2) return 0;
        int n = prices.size();
        vector<vector<int>> dp(n, vector<int>(5, 0)); // dp[i][j]表示第i天第j种状态的最大利润
        dp[0][0] = 0; // 不操作，即为0
        dp[0][1] = -prices[0]; // 第0天买入股票
        dp[0][2] = 0; // 第0天卖出股票，即为0
        dp[0][3] = -prices[0]; // 第0天第二次买入股票
        dp[0][4] = 0; // 第0天第二次卖出股票，即为0

        for (int i = 1; i < n; i++) {
            dp[i][0] = dp[i - 1][0]; // 不操作
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]); // 第i天第一次持有股票
            dp[i][2] = max(dp[i - 1][2], dp[i - 1][1] + prices[i]); // 第i天第一次不持有股票
            dp[i][3] = max(dp[i - 1][3], dp[i - 1][2] - prices[i]); // 第i天第二次持有股票
            dp[i][4] = max(dp[i - 1][4], dp[i - 1][3] + prices[i]); // 第i天第二次不持有股票
        }
        // return max(dp[n - 1][2], dp[n - 1][4]); // 返回最后一天的最大利润
        return dp[n - 1][4]; // 返回最后一天第二次卖出股票的最大利润
    }
};
```


