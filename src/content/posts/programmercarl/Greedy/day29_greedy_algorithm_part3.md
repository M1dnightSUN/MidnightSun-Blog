---
title: Day29-贪心算法 part03
published: 2025-04-09
description: 贪心算法，加油站，分发糖果，柠檬水找零，根据身高重建队列
tags: [C++, 贪心算法]
category: 算法训练营
draft: false
---

## 加油站

[134. 加油站](https://leetcode.cn/problems/gas-station/)

我们可以设定一个净增的油量，也就是每次到达加油站获取的油量，减去下一次所需要的油量。

也就是说 `total = gas[i] - cost[i]`，然后对 `total` 做一个累加，如果 `total` 小于 0，则说明无论从哪里开始都无法到达终点。

反之，如果 `total` 大于 0，则我们先不管是从哪里出发的，一定存在一个点 `start`，使得从 `start` 出发可以到达终点。

我们可以定义一个变量 `total`，用来计算从第 `0` 个加油站出发到达终点的净油量。
同时定义一个变量 `cur`，用来表示当前的净油量。

如果 `cur` 小于 0，则说明我们已经无法到达下一个加油站了，所以我们需要将 `start` 更新为 `i + 1`，并将 `cur` 重置为 `0`。 

整体代码如下：
```cpp
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int total = 0; // 总的净油量
        int cur = 0;   // 当前段的净油量
        int start = 0; // 起始加油站索引

        for (int i = 0; i < gas.size(); i++) {
            total += gas[i] - cost[i]; // 计算总净油量
            cur += gas[i] - cost[i];   // 计算当前段净油量

            if (cur < 0) { // 当前段净油量不足以到达下一站
                start = i + 1; // 更新起始加油站为下一站
                cur = 0;       // 重置当前段净油量
            }
        }
        // 如果总净油量为正，则说明可以绕行一圈，否则返回 -1
        return total >= 0 ? start : -1;
    }
};
```

```ini
gas  =  [1, 2, 3, 4, 5]
cost = [3, 4, 5, 1, 2]
net = gas[i] - cost[i] = [-2, -2, -2, 3, 3]

站点:     0     1     2     3     4
         ↓     ↓     ↓     ↓     ↓
净油量: -2   -2   -2   +3   +3

cur:   <---当前段累计油量差

```
核心在于：
- 我们只关心有没有一个起点 start，从这个起点出发，走一圈能不能回到起点。

- `cur` 每次失败就重置成 0，表示当前这段肯定不能作为起点，那我们就跳过。

- 最后如果 `total >= 0`，说明全局油量足够绕一圈，这时候 start 一定就是那个唯一可行的起点。

## 分发糖果

[135. 分发糖果](https://leetcode.cn/problems/candy/)

题目要求分发糖果，规则如下：
1. 每个孩子至少分配一个糖果。
2. 相邻的孩子中，评分更高的孩子必须获得更多的糖果。

我们可以初始化一个数组将每人分得1个糖果，然后分两次遍历数组：

1. **从左到右遍历**：
   - 如果当前孩子的评分比左边的孩子高，则当前孩子的糖果数应该比左边孩子多 1。
   - 这样可以保证从左到右的规则满足。

2. **从右到左遍历**：
   - 如果当前孩子的评分比右边的孩子高，则当前孩子的糖果数应该比右边孩子多 1。
   - **同时需要取当前糖果数和右边孩子糖果数 +1 的最大值**，以确保不覆盖从左到右遍历的结果。

3. **计算总糖果数**：
   - 遍历糖果数组，将所有孩子的糖果数累加即可。

整体代码如下：
```cpp
class Solution {
public:
    int candy(vector<int>& ratings) {
        int n = ratings.size();
        vector<int> candies(n, 1); // 每个孩子至少分配一个糖果

        // 从左到右遍历，确保右边评分更高的孩子糖果更多
        for (int i = 1; i < n; i++) {
            if (ratings[i] > ratings[i - 1]) { // 当前孩子评分高于左边孩子
                candies[i] = candies[i - 1] + 1; // 当前孩子糖果数比左边孩子多 1
            }
        }
        // 从右到左遍历，确保左边评分更高的孩子糖果更多
        for (int i = n - 2; i >= 0; i--) { // 从倒数第二个孩子开始遍历
            if (ratings[i] > ratings[i + 1]) { // 当前孩子评分高于右边孩子
                // 如果 ratings[i] > ratings[i + 1]，此时candy[i]（第i个小孩的糖果数量）就有两个选择
                // 一个是candy[i + 1] + 1（从右边这个加1得到的糖果数量）
                // 一个是candyVec[i]（之前比较右孩子大于左孩子得到的糖果数量）
                // 那么我们取局部最优，也就是取两者的最大值
                candies[i] = max(candies[i], candies[i + 1] + 1);
            }
        }
        // 计算糖果总数
        return accumulate(candies.begin(), candies.end(), 0);
    }
};
```
因此这题其实利用了两次贪心策略，第一次是从左到右，每次保证右边的孩子比左边的孩子多 1，第二次是从右到左，每次保证左边的孩子比右边的孩子多 1，同时还要考虑在左到右过程中可能已经被加过的糖果，因此我们取两者的最大值。

## 柠檬水找零

[860. 柠檬水找零](https://leetcode.cn/problems/lemonade-change/)

这里我们根据题意，对于客户支付的情况一共有三种：

1. 支付5：
    - 不需要找零，并收下5美元

2. 支付10：
    - 只能找零5美元，并收下10美元
    - 如果没有5美元的零钱，则无法找零，返回 `false`

3. 支付20：
    - 需要找零15美元
        - 如果有10美元和5美元的零钱，则找零10美元和5美元
        - 如果没有10美元的零钱，则需要找零3个5美元
        - 如果都不满足，则无法找零，返回 `false`
    - 如果没有5美元的零钱，则无法找零，返回 `false`

我们的局部最优情况在于，当支付20美元时，优先找10美元和5美元的零钱，这样可以保证后续支付5美元和10美元时有足够的零钱可找，因为5美元能够用来找零的情况更多。因此遇到账单20，优先消耗美元10，完成本次找零。

所以我们可以维护两个变量 `five` 和 `ten`，分别表示5美元和10美元的零钱数量。

```cpp
class Solution {
public:
    bool lemonadeChange(vector<int>& bills) {
        int five = 0, ten = 0; // 记录5美元和10美元的数量

        for (int bill : bills) {
            if (bill == 5) {
                five++; // 收到5美元，不需要找零
            } 
            else if (bill == 10) {
                if (five > 0) { // 找零1张5美元，并收下10美元
                    five--;
                    ten++;
                } 
                else {
                    return false; // 无法找零
                }
            } 
            else if (bill == 20) {
                if (ten > 0 && five > 0) { // 优先找零1张10美元和1张5美元
                    ten--;
                    five--;
                } 
                else if (five >= 3) { // 否则找零3张5美元
                    five -= 3;
                } 
                else {
                    return false; // 无法找零
                }
            }
        }
        return true; // 所有顾客都成功找零
    }
};
```

## 根据身高重建队列

[406. 根据身高重建队列](https://leetcode.cn/problems/queue-reconstruction-by-height/)

这题的题意其实是，需要重新构造一个队列，使得每个人的前面有 `k` 个人的身高大于等于他。

```cpp
result = [ ?, ?, ?, ?, ?, ? ]
```
使得对于每一个人在这个 result 队列中:

```ini
他前面有正好 k 个“身高 ≥ 他身高”的人
```

我们可以先将所有人按照身高从高到低排序，如果身高相同，则按照 `k` 从小到大排序。

```cpp
class Solution {
public:
    vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
        // 按身高降序排序，如果身高相同则按 k 值升序排序
        sort(people.begin(), people.end(), [](const vector<int>& a, const vector<int>& b) {
            return a[0] > b[0] || (a[0] == b[0] && a[1] < b[1]);
        });

        vector<vector<int>> result;
        // 按照 k 值插入到结果队列中
        for (const auto& person : people) {
            result.insert(result.begin() + person[1], person);
        }

        return result;
    }
};
```