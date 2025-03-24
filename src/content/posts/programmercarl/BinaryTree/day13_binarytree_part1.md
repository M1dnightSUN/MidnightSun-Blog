---
title: Day13-二叉树 part01
published: 2025-03-24
description: 二叉树，二叉树的各种遍历方式
tags: [C++, 栈, 二叉树, 递归, 迭代]
category: 算法训练营
draft: false
---

## 二叉树

二叉树是一种树形结构，它的每个节点最多有两个子节点，分别是左子节点和右子节点。

二叉树的存储方式一般是链式存储：

```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};
```

如果使用数组来存储二叉树，假设父节点的下标是 `i`，那么左子节点的下标是 `2*i+1`，右子节点的下标是 `2*i+2`。

## 二叉树的递归遍历

二叉树的遍历方式主要有两种：
- 深度优先搜索（DFS）：前序遍历、中序遍历、后序遍历（递归，迭代）
- 广度优先搜索（BFS）：层序遍历（跌代）

这里我们先看深度优先搜索的递归实现，也就是三种常见的遍历方式。
这里遍历的顺序其实就是根节点的访问顺序。

关于递归的写法，我们可以确定三个要素[^1]：
1. 确定递归函数的参数和返回值：确定哪些参数是递归的过程中需要处理的，那么就在递归函数里加上这个参数， 并且还要明确每次递归的返回值是什么进而确定递归函数的返回类型。

2. 确定终止条件：写完了递归算法, 运行的时候，经常会遇到栈溢出的错误，就是没写终止条件或者终止条件写的不对，操作系统也是用一个栈的结构来保存每一层递归的信息，如果递归没有终止，操作系统的内存栈必然就会溢出。

3. 单层递归的逻辑：确定每一层递归需要处理的信息。在这里也就会重复调用自己来实现递归的过程。

[^1]:[代码随想录-二叉树的递归遍历](https://programmercarl.com/%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E9%80%92%E5%BD%92%E9%81%8D%E5%8E%86.html#%E6%80%9D%E8%B7%AF)

### 前序遍历

[二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

按照递归写法的思路：
1. 确定递归函数的参数和返回值：这里要存储遍历的结果，所以要传入一个 `vector<int>` 来存储结果，返回值是 `void`。

2. 确定终止条件：当节点为空时，直接返回。

3. 单层递归的逻辑：先访问根节点，再递归访问左子树，再递归访问右子树。

```cpp
void traversal(TreeNode* root, vector<int>& vec) { // 1. 确定递归函数的参数和返回值
    if (!root) return; // 2. 确定终止条件
    // 3. 单层递归的逻辑
    vec.push_back(root->val); // 中
    if (root->left) traversal(root->left, vec); // 左
    if (root->right) traversal(root->right, vec); // 右
}
```
确定好递归函数之后，遍历的函数就很好写了：
```cpp
vector<int> preorderTraversal(TreeNode* root) {
    if (!root) return {};
    vector<int> ans;
    traversal(root, ans);
    return ans;
}
```

### 中序遍历

[二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

依照之前的思路，我们只需要在前序遍历中的代码调整一下遍历顺序即可。

```cpp
void traversal(TreeNode* root, vector<int>& vec) { // 1. 确定递归函数的参数和返回值
    if (!root) return; // 2. 确定终止条件
    // 3. 单层递归的逻辑
    if (root->left) traversal(root->left, vec); // 左
    vec.push_back(root->val); // 中
    if (root->right) traversal(root->right, vec); // 右
}
```

### 后序遍历

[二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

写法同上，只需要调整遍历顺序即可。

```cpp
void traversal(TreeNode* root, vector<int>& vec) { // 1. 确定递归函数的参数和返回值
    if (!root) return; // 2. 确定终止条件
    // 3. 单层递归的逻辑
    if (root->left) traversal(root->left, vec); // 左
    if (root->right) traversal(root->right, vec); // 右
    vec.push_back(root->val); // 中
}
```

## 二叉树的迭代遍历

在二叉树的迭代遍历中，我们需要借助栈来实现。
递归的实现方式是隐式地使用了系统栈，而迭代的实现方式是显式地使用一个栈来模拟系统栈。

### 前序遍历

在前序遍历中，遍历的顺序是中左右，所以我们先将根节点入栈，然后处理根节点，**再将右子节点入栈，再将左子节点入栈**。

:::tip[前序遍历的入栈顺序]
先入栈右节点，再入栈左节点，这样在出栈的时候就是先左后右的顺序。
:::

```cpp
vector<int> preorderTraversal(TreeNode* root) {
    // 迭代法
    stack<TreeNode*> stk;
    vector<int> ans;
    if (!root) return ans;
    stk.push(root); // 先入栈根节点
        
    while (!stk.empty()) {
        // 处理中节点
        TreeNode* node = stk.top(); 
        stk.pop();
        ans.push_back(node->val);

        if (node->right) stk.push(node->right); // 右
        if (node->left) stk.push(node->left); // 左
    }
    return ans;
}
```

### 后序遍历

先序遍历是中左右，后序遍历是左右中，那么我们只需要调整一下先序遍历的代码顺序，就变成中右左的遍历顺序，然后在反转result数组，输出的结果顺序就是左右中了。

- 前序遍历->中左右
- 调整代码顺序->中右左
- 反转结果集->左右中

因此可以根据前序遍历的代码稍作调整即可。

```cpp
vector<int> postorderTraversal(TreeNode* root) {
    // 迭代法
    stack<TreeNode*> stk;
    vector<int> ans;
    if (!root) return ans;
    stk.push(root); // 先入栈根节点
        
    while (!stk.empty()) {
        // 处理中间节点
        TreeNode* node = stk.top();
        stk.pop();
        ans.push_back(node->val);

        if (node->left) stk.push(node->left); // 左
        if (node->right) stk.push(node->right); // 右
    }
    reverse(ans.begin(), ans.end()); // 反转结果集
    return ans;
}
```

### 中序遍历

在这里需要注意的是，之前的前序和后序遍历的代码在中序遍历中不能通用，因为之前的代码的处理过程是：先访问中间节点，处理的也是中间节点，**因为要访问的元素和要处理的元素都是同一个**。

而中序遍历的顺序是左中右，我们需要一步一步访问到最左下的节点，然后再处理中间节点，这造成了访问的元素和处理的元素不是同一个。

那么在使用迭代法写中序遍历，就需要借用指针的遍历来帮助访问节点，栈则用来处理节点上的元素。

所以在中序遍历中，我们需要先将根节点入栈，然后一直往左走，直到走到最左下的节点，然后处理这个节点，再处理右子节点。

```cpp
vector<int> inorderTraversal(TreeNode* root) {
    // 迭代法
    stack<TreeNode*> stk;
    vector<int> ans;
    TreeNode* node = root; // 指针遍历
    while (node || !stk.empty()) { // 当node不为空或者栈不为空时
        while (node) { // 一直往左走
            stk.push(node);
            node = node->left;
        }
        // 处理中间节点
        node = stk.top(); 
        stk.pop(); 
        ans.push_back(node->val);

        node = node->right; // 处理右子节点
    }
    return ans;
}
```

### 迭代遍历的统一写法

这里我们以中序遍历为例，来解决上述提到的访问元素和处理元素不一致的问题。

我们可以**将访问的节点放入栈中，把要处理的节点也放入栈中但是要做标记**，这样在处理的时候就可以知道这个节点是要处理的节点还是要访问的节点。

这里有两种标记的方式[^2]:
1. 就是要处理的节点放入栈之后，紧接着放入一个空指针作为标记。 这种方法可以叫做空指针标记法。

2. 加一个 `boolean` 值跟随每个节点，`false`(默认值) 表示需要为该节点和它的左右儿子安排在栈中的位次，`true` 表示该节点的位次之前已经安排过了，可以收割节点了。 这种方法可以叫做 `boolean` 标记法。

[^2]:[代码随想录-二叉树的迭代遍历](https://programmercarl.com/%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E7%BB%9F%E4%B8%80%E8%BF%AD%E4%BB%A3%E6%B3%95.html#%E6%80%9D%E8%B7%AF)

```cpp
// 空指针标记法
vector<int> inorderTraversal(TreeNode* root) {
    vector<int> result;
    stack<TreeNode*> st;
    if (root != NULL) st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top();
        if (node != NULL) {
            st.pop(); // 将该节点弹出，避免重复操作，下面再将右中左节点添加到栈中
            if (node->right) st.push(node->right);  // 添加右节点（空节点不入栈）

            st.push(node);                          // 添加中节点
            st.push(NULL); // 中节点访问过，但是还没有处理，加入空节点做为标记。

            if (node->left) st.push(node->left);    // 添加左节点（空节点不入栈）
        } else { // 只有遇到空节点的时候，才将下一个节点放进结果集
            st.pop();           // 将空节点弹出
            node = st.top();    // 重新取出栈中元素
            st.pop();
            result.push_back(node->val); // 加入到结果集
        }
    }
    return result;
}
```

```cpp
// boolean 标记法
vector<int> inorderTraversal(TreeNode* root) {
    // 迭代法
    stack<pair<TreeNode*, bool>> stk;
    vector<int> ans;
    if (!root) return ans;
    stk.push({root, false}); // 先入栈根节点
        
    while (!stk.empty()) {
        auto [node, flag] = stk.top(); // stk.top() 返回的是一个pair
        stk.pop();
        if (flag) ans.push_back(node->val); // 处理节点
        else { // 访问节点
            if (node->right) stk.push({node->right, false}); // 右
            stk.push({node, true}); // 标记
            if (node->left) stk.push({node->left, false}); // 左
        }
    }
    return ans;
}
```

## 二叉树的层序遍历

层序遍历一个二叉树。就是从左到右一层一层的去遍历二叉树。这种遍历的方式和之前的都不太一样。

需要借用一个辅助数据结构即队列来实现，队列先进先出，符合一层一层遍历的逻辑，而用栈先进后出适合模拟深度优先遍历也就是递归的逻辑。

```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    queue<TreeNode*> que;
    vector<vector<int>> ans;
    if (!root) return ans;
    que.push(root);
    while (!que.empty()) { // 当队列不为空时
        int size = que.size();  // 记录当前层的节点个数，que.size()是动态变化的
        vector<int> vec; // 存储当前层的节点值
        for (int i = 0; i < size; i++) {    // 遍历当前层的节点
            TreeNode* node = que.front(); // 取出队首元素
            que.pop();
            vec.push_back(node->val); // 存储当前层的节点值

            if (node->left) que.push(node->left); // 将左子节点入队
            if (node->right) que.push(node->right); // 将右子节点入队
        }
        ans.push_back(vec);
    }
    return ans;
}
```

## 总结

总的来说递归的写法看起来还是简单易懂，不过使用迭代的写法，就需要思考很多，希望自己能熟练掌握这种迭代写法的。

层序遍历的应用相当广泛，后序自己也会把随想录中提到的题目都做一遍。