---
title: Day22-回溯算法 part01
published: 2025-04-02
description: 回溯算法，组合问题，组合总和III，电话号码字母组合
tags: [C++, 回溯, 数组, 组合, 排列, 哈希表]
category: 算法训练营
draft: false
---

## 回溯算法

回溯算法就是在尝试一条路径的过程中不断深入，如果发现不能达到目标，就回退一步，换另一种选择继续尝试，直到找到解或者穷尽所有解。

它本质上是一个 **递归过程 + 状态恢复**。

回溯算法与 `DFS` 的关系：回溯算法是深度优先搜索（DFS）的一种特例。但回溯算法更强调“撤销操作”，是“DFS + 剪枝 + 撤销”的过程。

回溯法，一般可以解决如下几种问题：

1. 组合问题：N个数里面按一定规则找出k个数的集合
2. 切割问题：一个字符串按一定规则有几种切割方式
3. 子集问题：一个N个数的集合里有多少符合条件的子集
4. 排列问题：N个数按一定规则全排列，有几种排列方式
5. 棋盘问题：N皇后，解数独等等

回溯算法通用模板(C++伪代码)：

```cpp
void backtrack(参数) {
    if (满足结束条件) {
        保存结果;
        return;
    }

    for (每一个可选选择) {
        if (不合法) continue;

        做出选择;
        backtrack(进入下一层); // 递归
        撤销选择; // 回溯
    }
}
```

## 组合

[77. 组合](https://leetcode-cn.com/problems/combinations/)

这里的关键点：
- 数字之间的顺序不重要，只要组合内的数字集合相同就认为是相同的结果。

- 要求输出所有可能的组合，这要求遍历所有可能的选择。

如果要使用 `for` 循环来实现，可能会导致当 `k` 较大时，循环的次数会非常多，导致时间复杂度过高。
因此我们可以使用回溯算法来实现。

我们可以有以下思路：

**1. 参数**

- 路径（path）：当前已经选择的数字集合，代表当前的决策过程。

- 候选列表：在数字 1 到 n 中，当前还可以选择的数字。

- 起点（start）：为了避免重复组合，我们规定每次递归选择数字时只能从当前数字之后的数字中进行选择。

    - 例如，第一次选择从 1 开始；如果选了 1，那么接下来只能从 2 开始，这样保证了组合的顺序和唯一性。

**2. 终止条件**

- 当路径的长度等于 k 时，说明当前构建的组合已经完整，此时将当前的路径记录下来，然后返回上层，进行其他分支的探索。

    - 终止条件：if (path.size() == k)。

**3. 递归过程**

- 循环选择：从当前起点 start 到 n 进行遍历，每一次循环：

    - 选择数字 i：将 i 加入当前路径。

    - 递归调用：更新起点为 i+1，因为后续的选择只考虑比 i 大的数字。

    - 撤销选择：递归返回后，将 i 从路径中移除，以便探索其他可能的数字。

    这个过程对应于在一棵决策树中深度优先地遍历所有可能的路径。

根据[回溯算法的通用模板](#回溯算法)，我们可以实现如下代码：

```cpp
vector<int> path; // 全局变量存储组合
vector<vector<int>> ans; // 全局变量存储所有的组合

void backTracking(int n, int k, int start, vector<int>& path) {
    if (path.size() == k) { // 如果path长度等于k，那么存储结果并返回
        ans.push_back(path);
        return;
    }
    // 单层搜索逻辑
    for (int i = start; i <= n; i++) { // i == n 是有意义的
        path.push_back(i); // 将当前数字 i 添加到组合中
        backTracking(n, k, i + 1, path); // 递归：将起点更新为 i+1，表示后续只选择比 i 大的数字
        path.pop_back(); // 回溯：将 i 移除，恢复状态，以便尝试其他选择
    }
    return;
}
vector<vector<int>> combine(int n, int k) {
    backTracking(n, k, 1, path); // 根据题意这里是从1开始
    return ans;
}
```

:::tip[剪枝操作]
在 `for` 循环中，我们可以进行剪枝操作，避免不必要的递归调用。
例如，如果当前路径的长度加上剩余的数字数量小于 k，那么就没有必要继续递归下去，因为不可能满足条件。
```cpp
if (path.size() + (n - i + 1) < k) { // 当剩余数字数量小于 k 时，剪枝
    break; // 剪枝操作
}

path_size(): 当前路径的长度
(n - i + 1): 剩余数字的数量
```
:::

我们也可以将这个问题抽象成一个树的遍历问题[^1]：
- 每个节点代表一个数字的选择。
- 每个节点的子节点代表在当前选择的基础上，继续选择下一个数字。
- 叶子节点代表一个完整的组合。

如图所示：

![抽象成树的问题](https://file.kamacoder.com/pics/20201123195223940.png)

[^1]: [代码随想录-组合](https://programmercarl.com/0077.%E7%BB%84%E5%90%88.html#%E6%80%9D%E8%B7%AF)

每次从集合中选取元素，可选择的范围随着选择的进行而收缩，调整可选择的范围。

图中可以发现 `n` 相当于树的宽度，`k` 相当于树的深度。

## 组合总和III

[216. 组合总和 III](https://leetcode-cn.com/problems/combination-sum-iii/)

这里和[组合问题](#组合)是相似的回溯算法，只是添加了一些限制条件，我们可以用相同的思路来写以下的代码：

```cpp
vector<int> path; // 全局变量存储组合
vector<vector<int>> ans; // 全局变量存储所有的组合

void backTracking(int n, int k, int start, vector<int>& path) {
    if (n == 0 && path.size() == k) { // 如果相加之和被减到0, 且path的大小等于k，说明满足条件
        ans.push_back(path);
        return;
    }
    // 单层搜索逻辑
    // 剪枝操作 i + (k - path.size()) + 1 <= 9，和之前的剪枝是一个原理
    for (int i = start; i <= 9; i++) {
        path.push_back(i); // 将当前数字 i 添加到组合中
        backTracking(n - i, k, i + 1, path); // 隐藏的回溯逻辑，n的值没有被改变
        path.pop_back(); // 回溯：将 i 移除，恢复状态，以便尝试其他选择
    }
}

vector<vector<int>> combinationSum3(int k, int n) {
    backTracking(n, k, 1, path); // 根据题意这里是从1开始
    return ans;
}
```

:::tip[剪枝操作]
在 `for` 循环中，进行剪枝避免不必要的递归调用。
例如，如果当前路径的长度加上剩余的数字数量小于 `k`，就没有必要继续递归下去。
```cpp
if (path.size() + (n - i + 1) < k) { // 当剩余数字数量小于 k 时，剪枝
    break; // 剪枝操作
}
path_size(): 当前路径的长度
(n - i + 1): 剩余数字的数量
```
:::

总体来说和之前的组合问题是一个类型。

## 电话号码字母组合

[17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

这是一个排列问题，不同于组合（无序），本题要求输出所有有顺序的字母排列组合，每一位数字都有自己的字母选择。

```ini
"23" -> 两位数字
→ 第1位可选 ['a','b','c']
→ 第2位可选 ['d','e','f']
目标是从每一位中选出一个字母，组成一个字符串，最终生成所有可能结果。
```

我们可以把问题想成一个多叉树：

- 每一层表示一个数字；

- 每一层的分支是当前数字可选的字母；

- 我们从第一层一路向下选择，直到路径长度和输入数字长度相等，得到一个完整组合。

```ini
我们以“23”为例
                  " "
        /          |          \
        a          b           c                 ← 第一个数字 2 的字母
      / | \      / | \      /  |  \
     d  e  f    d  e  f    d   e   f             ← 第二个数字 3 的字母

从根节点开始，每层做一个选择，最终到达叶子节点，就完成一个组合。
```

根据前面的经验我们可以写出以下的代码：

```cpp
class Solution {
private:
    string path;
    vector<string> ans;
    // 数字到字母的映射
    vector<string> phoneMap = {
        "",     // 0
        "",     // 1
        "abc",  // 2
        "def",  // 3
        "ghi",  // 4
        "jkl",  // 5
        "mno",  // 6
        "pqrs", // 7
        "tuv",  // 8
        "wxyz"  // 9
    };
public:
    // 回溯函数：用于从 digits[index] 开始，尝试所有可能的组合
    void backTracking(const string &digits, int index) { // index是传入字符串的下标
        // 终止条件：当 index 等于 digits 的长度，说明所有数字都处理完
        if (index == digits.size()) {
            // 到达末尾，把当前路径保存
            if (!path.empty()) ans.push_back(path);
            return;
        }
        // 当前要处理的数字字符，ASCII 值处理方式
        int digit = digits[index] - '0';
        // 遍历该数字对应的所有字母
        for (char c : phoneMap[digit]) {
            path.push_back(c);
            backTracking(digits, index + 1); // 递归下一层
            path.pop_back();
        }
        return;
    }

    vector<string> letterCombinations(string digits) {
        backTracking(digits, 0); // 从第0位开始处理
        return ans;
    }
};
```


