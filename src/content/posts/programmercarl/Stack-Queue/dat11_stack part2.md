---
title: Day11-栈与队列 part02
published: 2025-03-22
description: 栈，队列，逆波兰表达式，滑动窗口最大值，前k个高频元素
tags: [C++, 栈, 队列, 字符串, 数组, 滑动窗口, 单调队列, 优先级队列, 哈希表]
category: 算法训练营
draft: false
---

## 逆波兰表达式

[逆波兰表达式](https://leetcode-cn.com/problems/evaluate-reverse-polish-notation/)

**逆波兰表达式**：

逆波兰表达式是一种后缀表达式，所谓后缀就是指算符写在后面。

- 平常使用的算式则是一种中缀表达式，如 `( 1 + 2 ) * ( 3 + 4 )` 。
- 该算式的逆波兰表达式写法为 `( ( 1 2 + ) ( 3 4 + ) * )` 。

逆波兰表达式主要有以下两个优点：

- 去掉括号后表达式无歧义，上式即便写成 `1 2 + 3 4 + *` 也可以依据次序计算出正确结果。
- 适合用栈操作运算：遇到数字则入栈；遇到算符则取出栈顶两个数字进行计算，并将结果压入栈中。

这里其实与二叉树的后序遍历（左右中）是差不多的。

在这里我们可以直接使用栈来解决，思路就是将数字入栈，然后遇到操作符，弹出两个数字进行运算，再将结果入栈。

图示如下[^1]：

![逆波兰表达式](https://camo.githubusercontent.com/9f7f3d3cc8df9823f36cb8566502a3c263476e49ca6b87bea9a3503d2c928eaa/68747470733a2f2f636f64652d7468696e6b696e672e63646e2e626365626f732e636f6d2f676966732f3135302e2545392538302538362545362542332541322545352538352542302545382541312541382545382542452542452545352542432538462545362542312538322545352538302542432e676966)

[^1]:[代码随想录-逆波兰表达式求值](https://programmercarl.com/0150.%E9%80%86%E6%B3%A2%E5%85%B0%E8%A1%A8%E8%BE%BE%E5%BC%8F%E6%B1%82%E5%80%BC.html#%E7%AE%97%E6%B3%95%E5%85%AC%E5%BC%80%E8%AF%BE)


```cpp
int f (int a, int b, string op) {
    if (op == "+") return a + b;
    if (op == "-") return a - b;
    if (op == "*") return a * b;
    if (op == "/") return a / b;
    return 0;
}
int evalRPN(vector<string>& tokens) {
    stack<int> stk;
    for (int i = 0; i <= tokens.size() - 1; i++){
        // 当遇到操作符的时候
        if (tokens[i] == "+" || tokens[i] == "-" || tokens[i] == "*" || tokens[i] == "/") {
            // 注意栈的弹出顺序，先弹出b再弹出a
            int b = stk.top();
            stk.pop();
            int a = stk.top();
            stk.pop();
            stk.push(f(a, b, tokens[i]));
        }
        else {
            stk.push(stoi(tokens[i])); // 这里要注意将字符串转为整数
        }
    }
    int ans = stk.top();
    stk.pop();
    return ans;
    }
```

唯一要注意的就是弹出元素的顺序，先弹出的是第二个操作数，之后才是第一个。

## 滑动窗口最大值

[239. 滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum/)

### 暴力求解

最直观的解法是使用 ` max_element()` 直接返回每次滑动窗口的最大值，然而这种方法会超时，时间复杂度为 $O(nk)$，`k` 是窗口大小，遍历数组 `n - k + 1` 次，`max_element()` 遍历k次。

```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) { // 暴力求解超时
    vector<int> ans;
    for (int i = 0; i + k <= nums.size(); i++) {
        ans.push_back(*max_element(nums.begin() + i, nums.begin() + i + k));
    }
    return ans;
}
```

### 单调队列[^2]

[^2]:[代码随想录-滑动窗口最大值](https://programmercarl.com/0239.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E6%9C%80%E5%A4%A7%E5%80%BC.html#%E5%8D%95%E8%B0%83%E9%98%9F%E5%88%97)

这里我们可以使用单调队列来解决，放进去窗口里的元素，然后随着窗口的移动，队列也一进一出，每次移动之后，队列告诉我们里面的最大值是什么。

我们需要的队列有以下操作：

```cpp
class MyQueue {
public:
    void pop(int value) {
    }
    void push(int value) {
    }
    int front() {
        return que.front();
    }
};
```

在每次移动滑动窗口的时候，调用 `pop()` 和 `push()` 方法，然后 `front()` 方法返回队列的最大值。

这里有个思路是：**没有必要维护窗口里的所有元素，只需要维护有可能成为窗口里最大值的元素就可以了，同时保证队列里的元素数值是由大到小的**。那么这个维护元素单调递减的队列就叫做单调队列，即单调递减或单调递增的队列。

以下图示展示了单调队列的过程：

![单调队列1](https://code-thinking.cdn.bcebos.com/gifs/239.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E6%9C%80%E5%A4%A7%E5%80%BC.gif)

设计单调队列的时候，`pop`，和 `push` 操作要保持如下规则：

- `pop(value)`：如果窗口移除的元素 `value` 等于单调队列的出口元素，那么队列弹出元素，否则不用任何操作
- `push(value)`：如果 `push` 的元素 `value` 大于入口元素的数值，那么就将队列入口的元素弹出，直到 `push` 元素的数值小于等于队列入口元素的数值为止

在保持以上规则的前提下，每次滑动窗口移动时，我们只需要调用 `front()` 方法就可以得到窗口的最大值。

下面是一个更清晰展示单调队列的过程：

![单调队列2](https://code-thinking.cdn.bcebos.com/gifs/239.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E6%9C%80%E5%A4%A7%E5%80%BC-2.gif)

那么使用哪种队列呢，答案是 `deque`，因为 `deque` 可以在队列的两端进行插入和删除操作。

那么根据以上规则，我们可以实现 `MyQueue` 类：
```cpp
class MyQueue {
    public:
    deque<int> que;
    // 每次pop()之前，首先要判断队列是否为空，然后比较当前要弹出的数值时候等于队列的第一个元素，如果是则弹出
    void pop(int value) {
        if (!que.empty() && value == que.front()) {
            que.pop_front();
        }
    }
    // 如果push()的数值大于入口元素的数值，那么就将队列入口的元素弹出，直到push()元素的数值小于等于队列入口元素的数值为止
    // 这样可保持队列里的元素是单调的
    void push(int value) {
        while (!que.empty() && value > que.back()) {
            que.pop_back();
        }
        que.push_back(value);
    }

    int front() {
        return que.front();
    }
};
```

然后就是遍历数组，实现求解滑动窗口的最大值：

```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) { 
    MyQueue que;
    vector<int> ans;
    for (int i = 0; i < k; i++) { // 先把前k个元素push进队列
        que.push(nums[i]);
    }
    ans.push_back(que.front()); // 记录前k个元素的最大值 
    for (int i = k; i < nums.size(); i++) { 
        que.pop(nums[i - k]); // 滑动窗口移除最前面元素
        que.push(nums[i]); // 滑动窗口前加入最后面的元素
        ans.push_back(que.front()); // 记录对应的最大值
    }
    return ans;
}
```

## 前k个高频元素

[347. 前K个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements/)

这里我们可以先用一个哈希表来统计每个元素出现的频率，然后再使用优先级队列来维护前 `k` 个高频元素。

优先级队列（priority_queue）底层通常基于堆实现，能在 `O(log n)` 的时间复杂度内插入或取出优先级最高的元素。默认情况下是大顶堆，即每次取出的都是值最大的元素。

```cpp
vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> map;
        for (int i = 0; i < nums.size(); i++) {
            map[nums[i]]++;
        }
        // 优先队列，大顶堆
        priority_queue<pair<int, int>> pq;
        for (auto it = map.begin(); it != map.end(); it++) { // 遍历map，将map中的元素push进优先队列
            pq.push(make_pair(it->second, it->first)); // 优先队列中存储的是pair<int, int>，第一个元素是频率，第二个元素是元素值
        }
        vector<int> ans;
        for (int i = 0; i < k; i++) { // 取出前k个元素
            ans.push_back(pq.top().second);
            pq.pop();
        }
        return ans;
    }
```

要注意的是，优先队列默认是大顶堆，所以我们要将频率作为第一个元素，元素值作为第二个元素，这样就可以取出前 `k` 个高频元素。

## 总结

今天的这个[单调队列](#滑动窗口最大值)还是有点难以理解，不过难的并不是 `myQueue` 类的实现，在下面的遍历逻辑里，饶了半天有点没绕明白。

最后要知道什么时候用优先级队列，什么时候用单调队列

- 优先级队列：求最大值或最小值
- 单调队列：求最大值或最小值，同时还要保持队列里的元素是单调的
