---
title: Day39-动态规划 part07
published: 2025-04-20
description: 动态规划，打家劫舍 I & II & III
tags: [C++, 动态规划, 二叉树]
category: 算法训练营
draft: false
---

## 打家劫舍 I

[198. 打家劫舍](https://leetcode.cn/problems/house-robber/)

#### 1. 确定dp数组以及下标的含义

> `dp[i]` 表示 偷前 `i` 个房子能获得的最大金额

#### 2. 确定递推公式

- 如果偷第 `i` 个房子，那么前 `i-1` 个房子不能偷，因此 `dp[i] = dp[i-2] + nums[i]`
- 如果不偷第 `i` 个房子，那么前 `i` 个房子能获得的最大金额为 `dp[i-1]`。

因此递推公式为：
> ```cpp
> dp[i] = max(dp[i-1], dp[i-2] + nums[i]);
> ```

#### 3. 初始化dp数组

- `dp[0] = nums[0]`

- `dp[1] = max(nums[0], nums[1])`

#### 4. 遍历顺序

由于 `dp[i]` 依赖于 `dp[i-1]` 和 `dp[i-2]`，因此我们需要从前往后遍历。

#### 5. 整体代码

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        if (n == 0) return 0;
        if (n == 1) return nums[0];

        vector<int> dp(n);
        dp[0] = nums[0];
        dp[1] = max(nums[0], nums[1]);

        for (int i = 2; i < n; i++) {
            dp[i] = max(dp[i - 1], dp[i - 2] + nums[i]);
        }

        return dp[n - 1];
    }
};
```

## 打家劫舍 II

#### 1. 确定dp数组以及下标的含义

和打家劫舍 I 类似，但因为房子是环形的，所以我们**不能同时偷第一个和最后一个房子**。

我们将问题拆解成两个子问题：

- 偷第 `0 ~ n-2` 间房屋（不偷最后一间）
- 偷第 `1 ~ n-1` 间房屋（不偷第一间）

每个子问题都是打家劫舍 I 的线性版本。

我们设：
- `dp[i]` 表示从第 `i` 间房屋开始偷，到当前能偷到的最大金额

#### 2. 确定递推公式

与打家劫舍 I 相同：

```cpp
dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])
```

#### 3. 初始化dp数组

对于子问题 `[start ~ end]`：

- `dp[start] = nums[start]`
- `dp[start + 1] = max(nums[start], nums[start + 1])`

#### 4. 遍历顺序

从 `start + 2` 开始到 `end`，依次计算 `dp[i]`。

#### 5. 整体代码（包含空间优化）

```cpp
class Solution {
private:
    int robRange(vector<int>& nums, int start, int end) {
        if (start == end) return nums[start];
        int pre2 = nums[start];
        int pre1 = max(nums[start], nums[start + 1]);
        for (int i = start + 2; i <= end; ++i) {
            int cur = max(pre1, pre2 + nums[i]);
            pre2 = pre1;
            pre1 = cur;
        }
        return pre1;
    }

public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        if (n == 0) return 0;
        if (n == 1) return nums[0];
        return max(robRange(nums, 0, n - 2), robRange(nums, 1, n - 1));
    }
};
```

## 打家劫舍 III

#### 1. 确定dp数组以及下标的含义

本题的结构是**二叉树**，不能用下标作为状态，因此不能直接使用数组。

我们使用**树形动态规划**，对每一个节点 `root`：

- 记录两个状态：
  - 偷当前节点：`dp[root][1]`（当前节点值 + 左右子树不偷的结果）
  - 不偷当前节点：`dp[root][0]`（左右子树偷或不偷的最大值之和）

由于树是结构体，不能直接建立数组，用函数返回 `pair<int, int>` 实现。

#### 2. 确定递推公式

设当前节点为 `node`，左右子树为 `left` 和 `right`，对应状态分别为 `(l0, l1)` 和 `(r0, r1)`：

```cpp
dp[node][1] = node->val + l0 + r0      // 偷当前节点
dp[node][0] = max(l0, l1) + max(r0, r1) // 不偷当前节点
```

#### 3. 初始化dp数组

后序遍历中，递归到空节点时返回 `{0, 0}` 表示偷与不偷都为0。

#### 4. 遍历顺序

**后序遍历**（从叶子到根），因为我们在计算当前节点状态时需要用到左右子树的状态。

#### 5. 整体代码

```cpp
class Solution {
private:
    pair<int, int> dfs(TreeNode* root) {
        if (!root) return {0, 0};

        auto left = dfs(root->left);
        auto right = dfs(root->right);

        int rob = root->val + left.second + right.second;
        int notRob = max(left.first, left.second) + max(right.first, right.second);

        return {rob, notRob};
    }

public:
    int rob(TreeNode* root) {
        auto res = dfs(root);
        return max(res.first, res.second);
    }
};
```




