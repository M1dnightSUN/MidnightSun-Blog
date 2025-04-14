---
title: Day34-动态规划 part02
published: 2025-04-14
description: 动态规划，不同路径，不同路径 II，整数拆分，不同的二叉搜索树
tags: [C++, 动态规划, 二叉搜索树, 数学]
category: 算法训练营
draft: false
---

## 不同路径

[62. 不同路径](https://leetcode.cn/problems/unique-paths/)

### 动态规划

1. 定义 dp 数组：`dp[i][j]` 表示从 `(0, 0)` 到 `(i, j)` 的路径数。
    - 最终我们需要求的是右下角的路径数，即 `dp[m - 1][n - 1]`。

2. 确定递推公式：
    - 我们要到达 `(i, j)`，只能从上面 `(i - 1, j)` 或者左边 `(i, j - 1)` 来。
    - 所以 `dp[i][j] = dp[i - 1][j] + dp[i][j - 1]`。

3. 确定初始条件：
    - 如果我们只有一行或者一列，那么只有一种路径。
    - 所以 `dp[0][j] = 1` 和 `dp[i][0] = 1`，因此我们可以初始化第一行和第一列为 `1`。
    - 不过也可以将dp数组直接初始化为 `1`，因为只有第一行和第一列的值是 `1`，其他的值会在后续的计算中被覆盖。

4. 确定计算顺序：
    - 从 `0` 到 `m - 1`，从 `0` 到 `n - 1`。
    - 即从左上角到右下角。

5. 返回结果：
    - `dp[m - 1][n - 1]`。

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<vector<int>> dp(m, vector<int>(n, 0));
        for (int i = 0; i < m; i++) {
            dp[i][0] = 1; // 第一列只能从上方到达
        }
        for (int j = 0; j < n; j++) {
            dp[0][j] = 1; // 第一行只能从左方到达
        }
        // 其实可以全都初始化为1 
        // vector<vector<int>> dp(m, vector<int>(n, 1)); 
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        return dp[m - 1][n - 1]; // 返回右下角的路径数
    }
};
```

### 数学方法

其实也就是组合数学的问题。

从左上角走到右下角需要走 `m - 1` 步“下”，`n - 1` 步“右”，总共 `m + n - 2` 步。
在这些步中选出 `m - 1` 步向下，其余为向右，所以总路径数是：

- `C(m + n - 2, m - 1)` 或者 `C(m + n - 2, n - 1)`。
- 其中 `C(n, k)` 表示从 `n` 个元素中选出 `k` 个元素的组合数，计算公式为：

- `C(m+n-2, m-1) = (m+n-2)! / ((m-1)! * (n-1)!)`

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        long long res = 1;
        for (int i = 1; i <= m - 1; ++i) {
            res = res * (n - 1 + i) / i;
        }
        return res;
    }
};
```

## 不同路径 II

[63. 不同路径 II](https://leetcode.cn/problems/unique-paths-ii/)

1. 定义 dp 数组：这里的定义其实和[上面](#不同路径)是一样的`dp[i][j]` 表示从 `(0, 0)` 到 `(i, j)` 的路径数。
    - 最终我们需要求的是右下角的路径数，即 `dp[m - 1][n - 1]`。

2. 确定递推公式：
    - 我们要到达 `(i, j)`，只能从上面 `(i - 1, j)` 或者左边 `(i, j - 1)` 来。

    - 但是当 `(i, j)` 是障碍物时，`dp[i][j] = 0`。

    - 因此我们要有一个先决条件：`if (obstacleGrid[i][j] == 0)`

        - 所以在这个条件下 `dp[i][j] = dp[i - 1][j] + dp[i][j - 1]`。

3. 确定初始条件：
    - 如果我们只有一行或者一列，那么只有一种路径。
    - 与之前不同的是，如果第一行或者第一列有障碍物，那么包括障碍物在内之后的路径都是不能到达的。
        - 所以我们需要在初始化时判断是否有障碍物，如果有障碍物，那么后面的路径数都初始化为 `0`。

4. 确定计算顺序：
    - 从 `0` 到 `m - 1`，从 `0` 到 `n - 1`。
    - 即从左上角到右下角。

```cpp
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int m = obstacleGrid.size();
        int n = obstacleGrid[0].size();

        if (obstacleGrid[0][0] == 1 || obstacleGrid[m - 1][n - 1] == 1) return 0; // 起点或终点有障碍物
        vector<vector<int>> dp(m, vector<int>(n, 0));
        for (int i = 0; i < m && obstacleGrid[i][0] == 0; i++) {
            dp[i][0] = 1; // 第一列只能从上方到达，如果有障碍物则不初始化
        }
        for (int j = 0; j < n && obstacleGrid[0][j] == 0; j++) {
            dp[0][j] = 1; // 第一行只能从左方到达，如果有障碍物则不初始化
        }

        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                if (obstacleGrid[i][j] == 0) { // 递推前先判断当前有没有障碍物，如果没有障碍物
                    dp[i][j] = dp[i - 1][j] + dp[i][j - 1]; // 上方和左方的路径数之和
                } else {
                    dp[i][j] = 0; // 有障碍物则路径数为0
                }
            }
        }
        return dp[m - 1][n - 1]; // 返回右下角的路径数
    }
};
```

## 整数拆分

[343. 整数拆分](https://leetcode.cn/problems/integer-break/)

1. 状态定义：

```css
dp[i] 表示将正整数 i 拆分成至少两个正整数之和后，这些数的最大乘积
```

2. 状态转移方程：

对 i 来说，可以拆成 `j + (i - j)`，其中 `1 <= j < i。

那么我们有两种选择：

- 不继续拆 `i - j`，也就是说乘积为 `j * (i - j)`

- 继续拆 `i - j`，也就是说乘积为 `j * dp[i - j]`

所以状态转移为：

```cpp
dp[i] = max(dp[i], max(j * (i - j), j * dp[i - j]))
```

3. 初始化：
- `dp[0] = 0`，实际上没有意义

- `dp[1] = 0`，实际上没有意义

- `dp[2] = 1`，表示拆分成 `1 + 1`，乘积为 `1`

4. 遍历顺序：

- 外层从 `2` 到 `n`，内层从 `1` 到 `i - 1`，也可以从 `1` 到 `i / 2`，因为 `j` 和 `i - j` 是对称的。

```cpp
class Solution {
public:
    int integerBreak(int n) {
        if (n <= 2) return 1; // 1和2只能拆分成1
        vector<int> dp(n + 1, 0); // dp[i]表示整数i的最大乘积
        dp[0] = 0;
        dp[1] = 0;
        dp[2] = 1;

        for (int i = 3; i <= n; i++) {
            for (int j = 1; j <= i / 2; j++) {
                // j * (i - j)表示拆分成j和i-j的乘积
                // j * dp[i - j]表示拆分成j和i-j的乘积，i-j可以继续拆分
                // 取两者的最大值
                // 同时因为固定了i，我们还要在当前取dp[i]的最大值
                dp[i] = max(dp[i], max(j * (i - j), j * dp[i - j]));
            }
        }
        return dp[n]; // 返回整数n的最大乘积  
    }
};
```

## 不同的二叉搜索树
 
[96. 不同的二叉搜索树](https://leetcode.cn/problems/unique-binary-search-trees/)

1. 状态定义：

```css
我们定义 dp[i] 表示 i 个节点能组成的不同二叉搜索树的个数。
```

2. 状态转移方程：

假设当前总共有 `i` 个节点，我们枚举每一个数 `j` 当作根节点，那么：

- 左子树节点数为 `j - 1`

- 右子树节点数为 `i - j`

那么以 `j` 为根的 BST 数量为：

```css
dp[j - 1] * dp[i - j]
```

对所有可能的根节点求和：

```css
dp[i] = dp[0] * dp[i - 1] + dp[1] * dp[i - 2] + ... + dp[i - 1] * dp[0]
/*这其实是 卡特兰数 的定义*/
```

3. 初始化：

- `dp[0] = 1`：空树也算一种

- `dp[1] = 1`：只有一个节点，只有一种 BST

4. 遍历顺序：
节点数为 `i` 的状态是依靠 `i` 之前节点数的状态。
外层从 `2` 到 `n`，内层从 `1` 到 `i`。

```cpp
class Solution {
public:
    int numTrees(int n) {
        vector<int> dp(n + 1, 0); // dp[i]表示i个节点的二叉搜索树的个数
        dp[0] = 1; // 0个节点的二叉搜索树的个数为1（空树）
        dp[1] = 1; // 1个节点的二叉搜索树的个数为1（只有一个节点）

        for (int i = 2; i <=n; i++) {
            for (int j = 1; j <= i; j++) {
                // j作为根节点，左子树有j-1个节点，右子树有i-j个节点
                dp[i] += dp[j - 1] * dp[i - j]; // 左子树和右子树的组合
            }
        }
        return dp[n]; // 返回n个节点的二叉搜索树的个数
    }
};
```
