---
title: Day24-回溯算法 part03
published: 2025-04-04
description: 回溯算法，复原IP地址，子集，子集II
tags: [C++, 回溯, 字符串]
category: 算法训练营
draft: false
---

## 复原IP地址

[93. 复原IP地址](https://leetcode-cn.com/problems/restore-ip-addresses/)

我们需要将一个字符串分割成4段合法的IP地址段，问题就变成了在字符串中寻找三个切割点，将整个字符串分为四段，每段都要符合合法条件。

其实与组合问题的思路是一样的，-字符串切割与组合问题的区别：

- 组合问题：选一个字母后，在剩余字母中继续选

- 切割问题：切掉一段后，在剩余部分继续切下一段

如果我们把这个问题转换成树形图，如图：

![IP树形图](https://file.kamacoder.com/pics/20201123203735933.png)

首先我们先定义两个全局变量：

```cpp
vector<string> path;
vector<string> ans;
```

回溯三部曲：

1. 递归参数
    因为我们在递归过程中需要知道从哪个地方进行下一层递归，所以我们需要一个 `startIndex` 参数来表示当前的起始位置。同时需要一个 `dotCount` 参数来表示当前的点的数量。

    ```cpp
    // 参数：字符串s，字符串起始位置startIndex，'.'的个数dotCount
    void backTracking(const string& s, int startIndex, int dotCount)
    ```
2. 递归终止条件

    - 当 `dotCount` 等于 4 时，说明我们已经找到了 4 段合法的 IP 地址段
    - 并且当 `startIndex` 等于字符串的长度时，说明我们已经遍历完了整个字符串

    此时可以将 `path` 中的字符串加入到 `ans` 中。 

    ```cpp
    if (dotCount == 4 && startIndex == s.size()) {
            ans.push_back(path[0] + "." + path[1] + "." + path[2] + "." + path[3]);
            return;
        }
    ```

3. 单层搜索逻辑

    - 每次分割的字符串最多只有三位，所以可以控制 `i - startIndex <= 2`，也就是每段最多三位数。
    - 重要的地方是如何切割字符串，`s.substr(startIndex, i - startIndex + 1)` 表示从 `startIndex` 开始，取 `i - startIndex + 1` 个字符，也就是取 `s[startIndex...i]`。
    - 我们将切割的字符串放入 `path` 
    - 进入下一层递归，`i + 1` 表示下一个起始位置，`dotCount + 1` 表示点的数量加 1。
    - 回溯时将 `path` 中的最后一个元素删除。
    
    ```cpp
    // 每段最多3位，i是当前段的终止位置
        for (int i = startIndex; i < s.size() && i - startIndex <= 2; ++i) {
            if (isValid(s, startIndex, i)) {
                // 将当前分割的字符串先放入path中：[startIndex, i]
                path.push_back(s.substr(startIndex, i - startIndex + 1)); // 从startIndex开始, 取i - startIndex + 1个字符，也就是取s[startIndex...i]
                backTracking(s, i + 1, dotCount + 1);
                path.pop_back(); // 回溯
            } else {
                break; // 剪枝
            }
        }
    ```

4. isValid() 函数

    - 段位以0为开头的数字不合法
    - 段位里有非正整数字符不合法
    - 段位如果大于255了不合法

    ```cpp
    bool isValid(const string& s, int start, int end) {
        if (start > end) return false;
        if (s[start] == '0' && start != end) return false; // 有前导 0
        int num = 0;
        for (int i = start; i <= end; i++) {
            if (s[i] > '9' || s[i] < '0') { // 遇到非数字字符不合法
                return false;
            }
            num = num * 10 + (s[i] - '0');
            if (num > 255) { // 如果大于255了不合法
                return false;
        }
        return true;
    }
    ```

总体的代码如下：

```cpp
class Solution {
private:
    vector<string> path;
    vector<string> ans;

    bool isValid(const string& s, int start, int end) {
        if (start > end) return false;
        if (s[start] == '0' && start != end) return false; // 有前导 0
        int num = 0;
        for (int i = start; i <= end; i++) { // 如果该字符串转换成整数的数值>255返回false
            if (!isdigit(s[i])) return false;
            num = num * 10 + (s[i] - '0');
            if (num > 255) return false;
        }
        return true;
    }
public:
    // 参数：字符串s，字符串起始位置startIndex，'.'的个数dotCount
    void backTracking(const string& s, int startIndex, int dotCount) {
        // 如果已经切了4段，检查是否用完字符串
        // 并且已经切割到最后一个字符串
        if (dotCount == 4 && startIndex == s.size()) {
            ans.push_back(path[0] + "." + path[1] + "." + path[2] + "." + path[3]);
            return;
        }
        
        // 每段最多3位，i是当前段的终止位置
        for (int i = startIndex; i < s.size() && i - startIndex <= 2; ++i) {
            if (isValid(s, startIndex, i)) {
                // 将当前分割的字符串先放入path中：[startIndex, i]
                path.push_back(s.substr(startIndex, i - startIndex + 1)); // 从startIndex开始, 取i - startIndex + 1个字符，也就是取s[startIndex...i]
                backTracking(s, i + 1, dotCount + 1);
                path.pop_back(); // 回溯
            } else {
                break; // 剪枝
            }
        }
    }

    vector<string> restoreIpAddresses(string s) {
        ans.clear();
        if (s.size() < 4 || s.size() > 12) return ans;
        backTracking(s, 0, 0);
        return ans;
    }
};
```

## 子集

[78. 子集](https://leetcode-cn.com/problems/subsets/)

如果把 子集问题、组合问题、分割问题都抽象为一棵树的话，那么组合问题和分割问题都是收集树的叶子节点，而子集问题是找树的所有节点。

因此这里的差别就体现在什么时候收集结果。以 `nums = [1,2,3]` 为例把求子集抽象为树型结构，如下：

![子集树](https://code-thinking.cdn.bcebos.com/pics/78.%E5%AD%90%E9%9B%86.png)

可以发现，我们每进行一次递归，就要把当前的 `path` 加入到结果中。

```cpp
class Solution {
private:
    vector<int> path;
    vector<vector<int>> ans;
public:
    void backTracking(vector<int>& nums, int startIndex) {
        ans.push_back(path); // 刚进入递归时，就收集结果
        if (startIndex >= nums.size()) { // 可以不写终止条件，因为满足该条件时，以下for循环不会进行
            return;
        }

        for (int i = startIndex; i < nums.size(); i++) {
            path.push_back(nums[i]);
            backTracking(nums, i + 1); // 不重复，从i + 1开始
            path.pop_back();
        }
    }
    vector<vector<int>> subsets(vector<int>& nums) {
        ans.clear();
        path.clear();
        backTracking(nums, 0);
        return ans;
    }
};
```

## 子集II

[90. 子集 II](https://leetcode-cn.com/problems/subsets-ii/)

次问题是一个综合了[子集](#子集)和[组合总和II](https://m1dnightsun.github.io/MidnightSun-Blog/posts/programmercarl/backtracking/day23_backtracking_part2/#%E7%BB%84%E5%90%88%E6%80%BB%E5%92%8Cii)的问题，无非就是在子集问题上加了一个去重的操作。

代码其实不难，主要是要注意去重的操作。

```cpp
class Solution {
private:
    vector<int> path;
    vector<vector<int>> ans;
public:
    void backTracking(vector<int>& nums, int startIndex) {
        ans.push_back(path); // 每进入一次递归就要收集结果
        if (startIndex >= nums.size());

        for (int i = startIndex; i < nums.size(); i++) {
            if (i > startIndex && nums[i] == nums[i - 1]) continue; // 去重
            path.push_back(nums[i]);
            backTracking(nums, i + 1);
            path.pop_back();
        }
    }
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        path.clear();
        ans.clear();
        sort(nums.begin(), nums.end()); // 针对去重需要排序
        backTracking(nums, 0);
        return ans;
    }
};
```

## 总结

对于组合问题和分割问题，我们都是从树的叶子节点收集结果，最后是当满足一定条件才收集结果。

而对于子集问题，我们是从树的所有节点收集结果，也就是每进行一次递归都要收集一次结果。