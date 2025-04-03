---
title: Day23-回溯算法 part02
published: 2025-04-03
description: 回溯算法，组合总和，组合总和II，分割回文串
tags: [C++, 回溯, 数组, 组合, 字符串, 动态规划]
category: 算法训练营
draft: false
---

## 组合总和

[39. 组合总和](https://leetcode-cn.com/problems/combination-sum/)

这里和[组合总和III](https://m1dnightsun.github.io/MidnightSun-Blog/posts/programmercarl/backtracking/day22_backtracking_part1/#%E7%BB%84%E5%90%88%E6%80%BB%E5%92%8Ciii)
的区别在于，本题的数组中的元素是可以重复使用的，也就是说在 `for` 循环中的回溯递归过程中，选取元素的位置可以从当前的 `i` 开始，而不是 `i + 1`。

这里对于组合问题，什么时候需要用到 `start` 标识来控制元素的选择：

- 如果是用一个数组（或集合）中的元素来进行组合，那么就需要用到 `start` 来控制元素的选择。例如[77. 组合](https://leetcode-cn.com/problems/combinations/)中的 `start` 就是用来控制元素的选择的。

- 如果是多个数组（或集合）中的元素来进行组合，那么就不需要用到 `start` 来控制元素的选择。例如[17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)，各个集合之间的元素互不影响，所以不需要用到 `start` 来控制元素的选择。

所以我们可以有以下的思路：
```cpp
class Solution {
private:
    vector<int> path; // 全局变量，存储当前遍历的数字
    vector<vector<int>> ans; // 全局变量，存储所有的组合
public:
    void backTracking(vector<int>& candidates, int target, int start) {
        if (target < 0) return; // 重要：如果target小于0，说明当前路径不符合条件，直接返回
        if (target == 0) { // 如果target等于0，说明当前路径符合条件
            ans.push_back(path);
            return;
        }
        for (int i = start; i < candidates.size() && candidates[i] <= target; i++) { //一旦 candidates[i] > target，就不会再往后递归，这里的数组一定是有序的
            path.push_back(candidates[i]);
            backTracking(candidates, target - candidates[i], i); // 这里的 i 而不是 i + 1，表示可以重复使用当前元素
            path.pop_back();
        }
    }
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        // 剪枝操作一定要对数组进行排序
        sort(candidates.begin(), candidates.end());
        backTracking(candidates, target, 0);
        return ans;
    }
};
```

注意这里的剪枝操作，没有排序时，`candidates[i] > target` 不代表后面的都更大，所以不能剪枝，否则就会漏掉组合。

## 组合总和II

[40. 组合总和 II](https://leetcode-cn.com/problems/combination-sum-ii/)

这个问题其实和之前的组合问题区别不大，主要是需要对重复的元素进行去重处理。

所以这里就直接写出代码了

```cpp
class Solution {
private:
    vector<int> path;
    vector<vector<int>> ans;
public:
    void backTracking(vector<int>& candidates, int target, int start) {
        if (target < 0) return;
        if (target == 0) {
            ans.push_back(path);
            return;
        }
        for (int i = start; i < candidates.size() && candidates[i] <= target; i++) {
            // start是在该层的第一个选择，所以要先判断i>start，相当于后续子数组的“0”位置
            if (i > start && candidates[i] == candidates[i - 1]) continue; // 去重操作
            path.push_back(candidates[i]);
            backTracking(candidates, target - candidates[i], i + 1); // 从下一个元素开始，避免重复使用
            path.pop_back();
        }
    }

    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        backTracking(candidates, target, 0);
        return ans;
    }
};
```

## 分割回文串

[131. 分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/)

字符串的切割问题和之前的组合问题类似，例如对于字符串 `abcdef`：

组合问题：选取一个 `a` 之后，在 `bcdef` 中再去选取第二个，选取 `b`之后在 `cdef` 中再选取第三个.....。
切割问题：切割一个 `a` 之后，在 `bcdef` 中再去切割第二段，切割b之后在 `cdef` 中再切割第三段.....。

可以抽象为以下的树形图：

![字符串切割](https://code-thinking.cdn.bcebos.com/pics/131.%E5%88%86%E5%89%B2%E5%9B%9E%E6%96%87%E4%B8%B2.jpg)

递归用来纵向遍历，for循环用来横向遍历，切割线（就是图中的红线）切割到字符串的结尾位置，说明找到了一个切割方法。[^1]

[^1]: [代码随想录-分割回文串](https://programmercarl.com/0131.%E5%88%86%E5%89%B2%E5%9B%9E%E6%96%87%E4%B8%B2.html#%E6%80%9D%E8%B7%AF)

从树形结构的图中可以看出：切割线切到了字符串最后面，说明找到了一种切割方法，此时就是本层递归的终止条件。

```cpp
if (start == s.size()) {
    ans.push_back(path);
    return;
    }
```
对于如何分割：

- 从 `start` 开始，枚举所有的子串 `s[start..i]`，判断是否是回文串。

- 如果是回文串，就将 `s[start..i]` 加入到当前路径 `path` 中，然后递归分割剩下的子串 `s[i+1..end]`。

- 如果不是回文串，就继续枚举下一个子串 `s[start..i+1]`，直到 `i` 到达字符串的末尾。

- 递归结束后，回溯到上一个状态，移除上一次加入的子串 `s[start..i]`，继续枚举下一个子串 `s[start..i+1]`。

- 直到所有的子串都枚举完毕，返回所有的分割方案。

可以有以下的代码：

```cpp
class Solution {
private:
    vector<vector<string>> ans;  // 最终结果，包含所有分割方案
    vector<string> path;         // 当前递归路径下的分割结果

    // 判断 s[left..right] 是否是回文串
    bool isPalindrome(const string& s, int left, int right) {
        while (left < right) {
            if (s[left++] != s[right--]) return false;  // 有不相等的字符就不是回文
        }
        return true;  // 所有字符相等，说明是回文
    }

    void backtrack(const string& s, int start) {
        // 如果 start 到达字符串末尾，说明已经成功分割一组，加入结果
        if (start == s.size()) {
            ans.push_back(path);
            return;
        }

        // 枚举所有从 start 开始的子串
        for (int i = start; i < s.size(); ++i) {
            // 判断 s[start..i] 是否是回文串
            if (isPalindrome(s, start, i)) {
                // 是回文串就加入当前路径
                path.push_back(s.substr(start, i - start + 1));
                // 递归分割剩下的子串（从 i+1 开始）
                backtrack(s, i + 1);
                // 回溯，移除上一次加入的子串
                path.pop_back();
            }
        }
    }

public:
    // 主函数，调用回溯
    vector<vector<string>> partition(string s) {
        backtrack(s, 0);  // 从索引 0 开始回溯
        return ans;       // 返回所有合法的分割方案
    }
};
```

这里其实也可以使用动态规划来解决这个问题，不过当前阶段就先不使用了。