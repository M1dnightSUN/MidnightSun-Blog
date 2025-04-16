---
title: Day36-动态规划 part04
published: 2025-04-16
description: 动态规划
tags: [C++, 动态规划]
category: 算法训练营
draft: false
---

## 最后一块石头的重量II

[1049. 最后一块石头的重量 II](https://leetcode.cn/problems/last-stone-weight-ii/)

题中要求的是两两石头相撞之后的重量差的最小值。

这也就意味着，我们应该尽可能让石头分成重量相等的两堆，那么相撞之后剩下的石头就是最小的重量。

因此这也和[分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)是相似的，也就是一个0 - 1背包问题。

唯一的区别是：分割等和子集是求是否可以分割成两堆相等的重量，如果凑不成就返回false，否则返回true。
而这个题是求最小的重量差，因此可以两堆石头的重量不相等。

转换成背包问题：
- 物品就是石头，物品的重量为 `stone[i]`，物品的价值为 `stone[i]`。
- 物品的数量为 `n`，背包的容量为 `sum(stone) / 2`。

#### 1. 确定dp数组以及下标的含义

`dp][j]`：表示容量为 `j` 的背包，最大可以装入的石头重量为 `dp[j]`。

> “最多可以装的价值为 `dp[j]`” 等同于 “最多可以背的重量为 `dp[j]`” 

#### 2. 确定递推公式

0 - 1背包问题的递推公式为：`dp[j] = max(dp[j], dp[j - weight[i]] + value[i])`。

对于本题来说，则转变成了：`dp[j] = max(dp[j], dp[j - stone[i]] + stone[i])`。

#### 3. 初始化dp数组

- 首先，`dp[0] = 0`，表示背包容量为0时，最大可以装入的石头重量为0。
- dp数组定义为多大：
    - 题中给出的石头数量范围是 `1 <= stone.length <= 30`，重量范围是 `1 <= stone[i] <= 100`。
    - 那么在极端情况下，石头的数量为30，重量为100，那么总重量为3000。
    - 我们要求的是背包容量为在总重量的一半，因此我们可以将背包的容量定义为 `1501`的长度，也就是 `vector<int> dp(1501, 0)`。

#### 4. 确定遍历顺序

在使用一维dp数组时，外层应该遍历的是物品，内层循环应该是倒序遍历背包容量。

```cpp
for (int i = 0; i < n; ++i) {                  // 遍历每一个物品
    for (int j = sum / 2; j >= stone[i]; --j) { // 容量从大到小倒序遍历
        dp[j] = max(dp[j], dp[j - stone[i]] + stone[i]);
    }
}
```

#### 5. 推导dp数组样例

假设我们的输入：`stones = [2,7,4,1,8,1]`, `sum = 23`，那么我们可以将背包的容量定义为 `sum / 2 = 11`。

```css
i = 0: 用stone[0], 遍历背包: dp[0...11]: [0, 0, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2]
i = 1: 用stone[1], 遍历背包: dp[0...11]: [0, 0, 2, 2, 2, 2, 2, 7, 7, 9, 9, 9]
i = 2: 用stone[2], 遍历背包: dp[0...11]: [0, 0, 2, 2, 2, 2, 2, 7, 7, 9, 9, 11]
i = 3: 用stone[3], 遍历背包: dp[0...11]: [0, 0, 2, 2, 2, 2, 2, 7, 7, 9, 9, 11]
i = 4: 用stone[4], 遍历背包: dp[0...11]: [0, 0, 2, 2, 2, 2, 2, 7, 7, 9, 9, 11]
i = 5: 用stone[5], 遍历背包: dp[0...11]: [0, 0, 2, 2, 2, 2, 2, 7, 7, 9, 9, 11]

我们得到最终结果：dp[11] = 11，说明我们可以将石头分成两堆，重量分别为11和12，那么此时的差值为1，也就是最小的差值。
```

#### 6. 返回结果

最后 `dp[target]` 里表示的就是容量为 `target` 的背包，最大可以装入的石头重量。
那么分成两堆石头的话，剩下的石头重量就是 `sum - dp[target]`。

**在计算 `target` 的时候，`target = sum / 2` 因为是向下取整，所以 `sum - dp[target]` 一定是大于等于`dp[target]` 的。**

那么相撞之后剩下的石头最小重量就是：`(sum - dp[target]) - dp[target]`。

#### 7. 代码实现

```cpp
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        vector<int> dp(1501, 0); // dp[i]表示容量为i的背包能装的最大重量
        int sum = accumulate(stones.begin(), stones.end(), 0); // 计算所有石头的总重量
        int target = sum / 2; // 背包的最大容量为总重量的一半

        for (int i = 0; i < stones.size(); i++) { // 遍历每个石头
            for (int j = target; j >= stones[i]; j--) { // 从后往前遍历每个容量
                // 如果当前容量j大于等于当前石头的重量stones[i]，则可以选择放入这个石头
                // 更新dp[j]为放入当前石头后的最大重量
                dp[j] = max(dp[j], dp[j - stones[i]] + stones[i]);
            }
        }
        // 分成两堆石头，一堆是dp[target]，另一堆是sum - dp[target]
        // 返回两堆石头的重量差的绝对值，即为最后剩下的石头的重量
        return abs(dp[target] - (sum - dp[target]));
    }
};
```

#### 8. 复杂度分析 & 总结
- 时间复杂度：O(n * target)，其中n为石头的数量，target为背包的容量。
- 空间复杂度：O(target)，使用了一维dp数组来存储每个容量的最大重量。

本题其实和分割等和子集几乎是一样的，只是最后对 `dp[target]` 的处理方式不同。

割等和子集相当于是求背包是否正好装满，而本题是求背包最多能装多少。


## 目标和

[494. 目标和](https://leetcode.cn/problems/target-sum/)

### 回溯暴力解法

这里其实和[组合总和](https://leetcode.cn/problems/combination-sum/)的思路是一样的，只不过我们可以在每次选择数字的时候选择加号或者减号。

```cpp
class Solution {
private:
    int dfs(vector<int>& nums, int i, int target) {
        if (i == nums.size()) { // 到达数组末尾
            // 如果目标值为0，说明找到了一个符合条件的组合，返回1；否则返回0
            return target == 0 ? 1 : 0;
        }
        // 递归调用，分别考虑加上当前数字和减去当前数字的两种情况
        return dfs(nums, i + 1, target - nums[i]) + dfs(nums, i + 1, target + nums[i]);
    }

public:
    int findTargetSumWays(vector<int>& nums, int target) {
        // 通过递归的方式计算所有可能的组合
        return dfs(nums, 0, target);
    }
};
```

但是使用回溯暴力求解法的时间复杂度是O(2^n)，其中n为数组的长度。
在力扣上提交的数据为：
- 141/141 cases passed (948 ms)
- Your runtime beats 11.67 % of cpp submissions
- Your memory usage beats 97.37 % of cpp submissions (11.3 MB)

### 动态规划

本题可以通过数学推导转换为 0-1 背包问题：

我们将数组划分为两个子集:

- 一个子集 `P` 的数添加 `+`

- 另一个子集 `N` 的数添加 `-`

目标是使得最终表达式等于 `target`。

```cpp
sum(P) - sum(N) = target // P表示加号的数，N表示减号的数
sum(P) + sum(N) = sum // sum表示所有数的和
```

两式相加可以得到：

```cpp
2 * sum(P) = target + sum
            ↓
sum(P) = (target + sum) / 2
```
所以问题转化为：从数组中选一些数字，使它们的和等于 `(target + sum) / 2`，每个数字只能用一次，一共有多少种选法。

#### 1. 前置判断（剪枝）

首先我们要判断：

```cpp
int sum = accumulate(nums.begin(), nums.end(), 0);
if ((target + sum) % 2 != 0 || abs(target) > sum) return 0;
```

- `(target + sum)` 必须为偶数，否则 P 不为整数，不能划分

- 如果 `target > sum`，显然无法达到目标

#### 2. 确定dp数组以及下标的含义

`dp[j]`：表示和为 `j` 的子集个数（从 nums 中选，元素不可重复）

#### 3. 确定递推公式

对于当前数字 `num`，我们可以选择放入背包或者不放入背包：

- 不选当前数字：`dp[j]` 保持原样

```cpp
dp[j] = dp[j]
```

- 选当前数 `num`:
    - 那么必须从一个“原本和为 `j - num` 的子集”中，加上当前 `num`，才能变成和为 j。
    - 所以我们有：

    ```cpp
    dp[j] = dp[j] + dp[j - num]
    ```
    - 表示：我们把所有“原本和为 `j - num` 的子集”，都加上 `num`，得到新的和为 `j` 的子集。

最终状态转移方程为：

```cpp
for (int j = target; j >= num; --j) {
    dp[j] += dp[j - num];
}
```

- `dp[j - num]` 表示之前已有多少种方法能组成 `j - num`

- 把当前 `num` 加进去就能组成 `j`

#### 4. 初始化dp数组

- `dp[0] = 1`：表示和为0的子集只有一种，就是不选任何数字
- 其余 `dp[j] = 0`

#### 5. 确定遍历顺序

- 外层遍历每个物品 `i`
- 内层倒序遍历容量 `j`，从 `target` 到 `num`

#### 6. 代码实现

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int sum = accumulate(nums.begin(), nums.end(), 0); // 计算所有数字的总和
        if (sum < target || (sum - target) % 2 != 0) return 0; // 如果目标不可能达到，返回0
        int s = (sum - target) / 2; // 将问题转化为背包问题，求子集和为s的组合数

        vector<int> dp(s + 1, 0); // dp[i]表示和为i的组合数
        dp[0] = 1; // 和为0的组合数为1（即不选任何数字）

        for (int num : nums) { // 遍历每个数字
            for (int j = s; j >= num; j--) { // 从后往前遍历每个容量
                dp[j] += dp[j - num]; // 更新dp[j]为当前数字num的组合数
            }
        }
        return dp[s]; // 返回和为s的组合数
    }
};
```

## 一和零

[474. 一和零](https://leetcode.cn/problems/ones-and-zeroes/)

#### 1. 状态定义

我们使用二维数组 `dp[i][j]` 表示状态：

- `dp[i][j]` 表示在限定 `i` 个 `0` 和 `j` 个 `1` 的容量下，最多可以选择多少个字符串。
- 初始时所有 `dp[i][j] = 0`，表示一个字符串都不选。

#### 2. 状态初始化

- 初始化条件为 `dp[0][0] = 0`，含义是：不使用任何 `0` 或 `1`，最多能选 0 个字符串；
- 其余 `dp[i][j]` 也初始化为 0，表示在该容量下尚未选择任何字符串。

#### 3. 状态转移方程

对于每个字符串 `s ∈ strs`，我们统计它的 `0` 和 `1` 数量分别为 `zeros` 和 `ones`。然后进行如下状态更新：

```cpp
for (int i = m; i >= zeros; --i) {
    for (int j = n; j >= ones; --j) {
        dp[i][j] = max(dp[i][j], dp[i - zeros][j - ones] + 1);
    }
}
```

解释如下：

- **不选当前字符串 `s`**：`dp[i][j]` 保持不变；
- **选当前字符串 `s`**：
  - 要消耗 `zeros` 个 0 和 `ones` 个 1；
  - 那么剩余容量为 `i - zeros` 和 `j - ones`；
  - 对应的最大选择数为 `dp[i - zeros][j - ones]`，加上当前字符串，变为 `dp[i - zeros][j - ones] + 1`；
- 两种方案取最大值，表示当前容量下最多可选字符串数。

#### 4. 遍历顺序

必须从大到小（倒序）枚举 `i` 和 `j`，这是因为每个字符串只能使用一次（符合 0-1 背包特性）。如果从小到大遍历，会导致同一个字符串被多次选入。

#### 5. 最终结果

状态转移结束后，最终结果为 `dp[m][n]`，表示在不超过 `m` 个 0 和 `n` 个 1 的条件下，最多可以选择多少个字符串。

#### 6. 代码实现

```cpp
class Solution {
public:
    int findMaxForm(vector<string>& strs, int m, int n) {
        // 初始化二维 dp 数组，dp[i][j] 表示在 i 个 0 和 j 个 1 的容量限制下，最多可选字符串数量
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        // 遍历每个字符串，相当于遍历每个“物品”
        for (const string& s : strs) {
            // 统计当前字符串中的 0 和 1 的个数
            int zeros = 0, ones = 0;
            for (char c : s) {
                if (c == '0') zeros++;
                else if (c == '1') ones++;
            }
            // 进行二维 0-1 背包状态更新，必须倒序遍历（保证每个字符串最多只被使用一次）
            for (int i = m; i >= zeros; --i) {
                for (int j = n; j >= ones; --j) {
                    // 状态转移：选 or 不选当前字符串
                    dp[i][j] = max(dp[i][j], dp[i - zeros][j - ones] + 1);
                }
            }
        }
        // 返回最大可选字符串数量
        return dp[m][n];
    }
};
```

#### 状态表变化过程（示例）

以如下数据为例：

- `strs = ["10", "0", "1"]`
- `m = 1`（最多 1 个 0），`n = 1`（最多 1 个 1）

##### 初始化状态表（dp）：

| i \ j | 0 | 1 |
|-----|---|---|
| 0   | 0 | 0 |
| 1   | 0 | 0 |

---

##### 处理字符串 `"10"`：

- `zeros = 1`，`ones = 1`
- 从 `i=1` 到 `zeros=1`，`j=1` 到 `ones=1`：

```cpp
dp[1][1] = max(dp[1][1], dp[0][0] + 1) = max(0, 1) = 1
```

更新后的表：

| i \ j | 0 | 1 |
|-----|---|---|
| 0   | 0 | 0 |
| 1   | 0 | 1 |

##### 处理字符串 `"0"`：

- `zeros = 1`，`ones = 0`

```cpp
dp[1][1] = max(1, dp[0][1] + 1) = max(1, 1) = 1
dp[1][0] = max(0, dp[0][0] + 1) = max(0, 1) = 1
```

更新后的表：

| i \ j | 0 | 1 |
|-----|---|---|
| 0   | 0 | 0 |
| 1   | 1 | 1 |

##### 处理字符串 `"1"`：

- `zeros = 0`，`ones = 1`

```cpp
dp[1][1] = max(1, dp[1][0] + 1) = max(1, 2) = 2
dp[0][1] = max(0, dp[0][0] + 1) = max(0, 1) = 1
```

最终状态表：

| i \ j | 0 | 1 |
|-----|---|---|
| 0   | 0 | 1 |
| 1   | 1 | 2 |

##### 最终结果：

`dp[1][1] = 2`，最多可以选 2 个字符串，比如：`"0"` 和 `"1"`，或 `"10"` 和 `"1"`。
