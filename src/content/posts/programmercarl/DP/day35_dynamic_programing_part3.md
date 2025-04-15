---
title: Day35-动态规划 part03
published: 2025-04-15
description: 动态规划，0-1背包问题，分割等和子集
tags: [C++, 动态规划]
category: 算法训练营
draft: false
---

## 0-1背包问题

0 - 1 背包问题定义：

> 有 `N` 个物品，每个物品有一个 体积（或重量）`w[i]` 和 价值 `v[i]`。现在给你一个容量为 `C` 的背包（不能超过容量），请你从这 `N` 个物品中选出若干个，使得在不超过背包容量的前提下，总价值最大。

“01”表示每个物品只能选择一次（要么选，要么不选）

### 二维 dp 解法

#### 状态定义：

定义二维数组 `dp[i][j]`，表示**使用第 `0` 到第 `i` 件物品**，在背包容量为 `j` 的限制下所能达到的最大价值。

#### 状态转移方程：

对于第 `i` 件物品（体积为 `weight[i]`，价值为 `value[i]`）：

- **不选**：`dp[i][j] = dp[i - 1][j]`

    如果我们**不选择第 `i` 件物品**，那么当前背包容量为 `j` 时的最大价值就是：

    **从前 `0` 到 `i-1` 这些物品中选取**，在容量限制为 `j` 的条件下所能达到的最大价值。

    换句话说，我们**跳过第 `i` 件物品，不把它放入背包中**。

是否还需要我把整段上下文整合输出？如果你要用于文档用途，我可以重新排好完整段落。

- **选**：`dp[i][j] = dp[i - 1][j - weight[i]] + value[i]`（前提是 `j >= weight[i]`）

    如果我们选择了第 `i` 件物品：

    - 会消耗 `weight[i]` 的容量，背包剩余空间变成 `j - weight[i]`
    - 从前 `i-1` 件物品中在剩余容量内选取的最大价值为 `dp[i - 1][j - weight[i]]`
    - 加上当前物品的价值 `value[i]`，就是选当前物品的总价值

    > 注意：只有当 `j >= weight[i]` 时，背包才放得下这件物品

因此状态转移方程为：
```cpp
dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
```

#### 初始化：

- `dp[0][j]` 表示只使用第 0 件物品，在容量为 `j` 时的最大价值：
  - 若 `j < weight[0]`，则 `dp[0][j] = 0`
  - 若 `j >= weight[0]`，则 `dp[0][j] = value[0]`

可以统一初始化方式如下：

```cpp
for (int j = 0; j <= N; ++j) {
    dp[0][j] = (j >= weight[0]) ? value[0] : 0;
}
```

#### 最终答案：

- `dp[M - 1][N]`，表示使用前 `M` 件物品（编号 `0`~`M-1`），在容量为 `N` 时的最大价值。

#### 示例（模板）：

```cpp
vector<vector<int>> dp(M, vector<int>(N + 1, 0));

// 初始化第0件物品
for (int j = 0; j <= N; ++j) {
    if (j >= weight[0]) dp[0][j] = value[0];
}

for (int i = 1; i < M; ++i) {
    for (int j = 0; j <= N; ++j) {
        if (j < weight[i])
            dp[i][j] = dp[i - 1][j];
        else
            dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
    }
}
```

### 一维 dp 解法

#### 状态定义：

定义一维数组 `dp[j]`，表示在背包容量为 `j` 时，可获得的最大价值（考虑过的所有物品来自 `weight[0..i]`）。

#### 状态转移方程：

对于第 `i` 件物品（体积 `weight[i]`，价值 `value[i]`）：

- 如果当前容量 `j >= weight[i]`，可以选择该物品，状态转移为：

```cpp
dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
```

- 如果 `j < weight[i]`，不能选该物品，`dp[j]` 保持不变。

> **注意**：为防止一个物品被重复使用，必须**倒序枚举容量 `j`**。

#### 初始化：

- `dp[0] = 0`：表示容量为 0 时，最大价值只能是 0
- 其余 `dp[j]` 也初始化为 `0`（未放入任何物品）

#### 遍历顺序：

在一维 dp 中，为防止每个物品被选多次，遍历顺序必须如下：

```cpp
for (int i = 0; i < M; ++i) {                  // 遍历每一个物品
    for (int j = N; j >= weight[i]; --j) {     // 容量从大到小倒序遍历
        dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
    }
}
```

- 外层循环表示依次考虑物品 `0 ~ M-1`
- 内层倒序循环容量，从 `N` 到 `weight[i]`，以确保当前物品不会在同一轮中被重复使用

#### 最终答案：

- `dp[N]` 表示背包容量为 `N` 时所能取得的最大价值

#### 示例（模板）：

```cpp
vector<int> dp(N + 1, 0);

for (int i = 0; i < M; ++i) {
    for (int j = N; j >= weight[i]; --j) {
        dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
    }
}
```

## 分割等和子集

[416. 分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)

我们想把数组分成两个子集，使得两个子集的和相等。

等价于：在数组中选出一些元素，使它们的总和为整个数组和的一半。

记数组总和为 `sum`，只要能选出若干个数使它们的和为 `sum / 2`，那么剩下的就是另外一半。

- 数组元素都是正整数
- 每个数只能选一次（**0-1 背包**）
- 不求最大价值，而是判断是否能达到某个“目标和” → **判断型背包**


#### 举例：

```cpp
nums = [1, 5, 11, 5]
sum = 22 → target = 11

我们要判断：能否从数组中选出若干个数，使它们的和恰好为 11？
```

这正是一个 **0-1 背包问题**：
- 每个数是一个“物品”，
- 数值是物品的“体积”，
- 背包容量为 `target = sum / 2`，
- 每个物品只能选择一次。

### 动态规划解法（一维数组）

#### 状态定义：

定义 `dp[j]` 表示：**是否能选出若干个数，使它们的和恰好为 `j`**。

- `dp[j] = true`：能选出和为 `j` 的子集
- `dp[j] = false`：不能选出

#### 状态转移方程：

遍历数组中每个数 `num`，从容量 `target` 倒序遍历：

```cpp
for (int j = target; j >= num; --j) {
    dp[j] = dp[j] || dp[j - num];
}
```

含义：
- 不选当前数字：`dp[j]` 保持原样
- 选当前数字：如果 `dp[j - num]` 是 true，则 `dp[j]` 也为 true

#### 初始化：

- `dp[0] = true`：容量为 0 的背包一定可以被“什么都不选”填满
- 其余 `dp[j] = false`

#### 遍历顺序：

- 外层遍历每个物品 `i`
- 内层倒序遍历容量 `j`，从 `target` 到 `num`

倒序是为了确保每个物品最多只能使用一次（0-1背包的特性）

#### 最终结果：

- 判断 `dp[target]` 是否为 `true`，表示能否恰好选出和为 `sum / 2` 的子集

#### 整体代码如下：

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum = accumulate(nums.begin(), nums.end(), 0);
        if (sum % 2 != 0) return false; // 总和为奇数，肯定不能平分

        int target = sum / 2;
        vector<bool> dp(target + 1, false);
        dp[0] = true; // 初始条件

        for (int num : nums) {
            for (int j = target; j >= num; --j) {
                dp[j] = dp[j] || dp[j - num];
            }
        }

        return dp[target];
    }
};
```

#### 时间复杂度：

- 时间复杂度：`O(N * target)`，其中 `N` 为数组长度，`target = sum / 2`
- 空间复杂度：`O(target)`，使用一维 `dp` 数组优化空间
