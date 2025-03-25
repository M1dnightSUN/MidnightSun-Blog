---
title: Day14-二叉树 part02
published: 2025-03-25
description: 二叉树，翻转二叉树，对称二叉树，二叉树的最大深度，二叉树的最小深度
tags: [C++, 二叉树, 递归, 迭代, BFS, DFS]
category: 算法训练营
draft: false
---

## 翻转二叉树

[226.翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

对于二叉树的翻转，其实用前序遍历和后序遍历都能实现，但是用中序遍历的话会使一些节点的左右子树翻转两次。

如果使用中序遍历的话：

```
      4                   4
    /   \  数次遍历后    /   \
   2     7    --->     7     2  --> 此时再遍历右子树会使以2位根节点的子树再次翻转
  / \   / \           / \   / \
 1   3 6   9         9   6 3   1
```
所以实现其实很简单，在遍历的时候只要交换左右子树然后递归遍历左右子树即可。

```cpp
TreeNode* invertTree(TreeNode* root) {
    if (!root) return root;
    if (!root->left && !root->right) return root; // 递归终止条件，是叶子节点直接返回
    // swap(root->left, root->right);
    // 中
    TreeNode* node = root->right;
    root->right = root->left;
    root->left = node;

    if (root->left) invertTree(root->left); // 左
    if (root->right) invertTree(root->right); // 右
    return root;
}
```

也可以使用迭代的同意写法方式来实现

```cpp
TreeNode* invertTree(TreeNode* root) { // 前序遍历
    stack<TreeNode*> stk;
    if (root) stk.push(root);
    while (!stk.empty()) {
        TreeNode* node = stk.top();
        if (node) {
            stk.pop();
            
            if (node->right) stk.push(node->right); // 右
            if (node->left) stk.push(node->left); // 左
            // 中
            stk.push(node);
            stk.push(nullptr);
        }
        else {
            stk.pop();
            node = stk.top();
            stk.pop();
            swap(node->left, node->right);
        }
    }
    return root;
}
```

## 对称二叉树

[101.对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

要判断一个二叉树是否为对称二叉树，需要判断左右子树节点是否对称：

1. 如果左子树节点和右子树节点都为空，那么说明左右对称，返回 `true`。
2. 如果左右子树有其中一个为空，则说明不对称，返回 `false`。
3. 如果左右子树节点都不为空，那么需要判断左右子树的值是否相等，且左子树的左节点和右子树的右节点对称，左子树的右节点和右子树的左节点对称。

### 递归写法

那么在递归函数中需要传入两个节点（t1, t2分别表示左右子树）来判断，根据上面的条件可以写出：

```cpp
// 1. 左右子树节点都为空
if (!t1 && !t2) return true;
// 2. 左右子树有其中一个为空
if (!t1 || !t2) return false; // 如果上面条件满足那么就会直接返回 true 不会进行这个判断。
// 3. 左右子树节点都不为空
if (t1->val != t2->val) return false;
```

所以可以有以下的递归函数：

```cpp
bool isMirro(TreeNode* t1, TreeNode* t2) {
    if (!t1 && !t2) return true;
    if (!t1 || !t2) return false;

    return ((t1->val == t2->val) 
            && isMirro(t1->left, t2->right) // 左子树的左节点和右子树的右节点对称
            && isMirro(t1->right, t2->left)); // 左子树的右节点和右子树的左节点对称
}

bool isSymmetric(TreeNode* root) { 
    if (!root) return false;
    return isMirro(root->left, root->right); // 传入左右子树节点
}
```

### 迭代写法

在这里我们可以使用队列来实现迭代的写法，队列中存放的是左右子树的节点，每次取出两个节点进行比较，然后将左子树的左节点和右子树的右节点入队，左子树的右节点和右子树的左节点入队。

使用队列的图示如下[^1]：

![使用队列判断对称二叉树](https://code-thinking.cdn.bcebos.com/gifs/101.%E5%AF%B9%E7%A7%B0%E4%BA%8C%E5%8F%89%E6%A0%91.gif)

[^1]:[代码随想录-对称二叉树](https://programmercarl.com/0101.%E5%AF%B9%E7%A7%B0%E4%BA%8C%E5%8F%89%E6%A0%91.html#%E6%80%9D%E8%B7%AF)

迭代写法的代码如下：

```cpp
bool isSymmetric(TreeNode* root) { // 迭代写法
    if (!root) return true;
    queue<TreeNode*> que;
    que.push(root->left);
    que.push(root->right);

    while (!que.empty()) {
        TreeNode* t1 = que.front(); // 取出两个节点
        que.pop();
        TreeNode* t2 = que.front();
        que.pop();

        if (!t1 && !t2) continue; // 1. 左右子树节点都为空
        if (!t1 || !t2 || t1->val != t2->val) return false; // 2. 左右子树有其中一个为空或者值不相等

        que.push(t1->left); // 将左子树的左节点和右子树的右节点入队
        que.push(t2->right);
        que.push(t1->right);// 将左子树的右节点和右子树的左节点入队
        que.push(t2->left);
    }
    return true;
}
```

## 二叉树的最大深度

[104.二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

如果求的是二叉树的深度，那么可以用前序遍历，如果求的是二叉树的高度，那么可以用后序遍历。

- 二叉树节点的深度：指从根节点到该节点的最长简单路径边的条数或者节点数（取决于深度从0开始还是从1开始）
- 二叉树节点的高度：指从该节点到叶子节点的最长简单路径边的条数或者节点数（取决于高度从0开始还是从1开始）

而二叉树的最大深度就是二叉树的高度。

所以这题可以很轻松的解决：

```cpp
int maxDepth(TreeNode* root) {
        if (!root) return 0; // 如果是空节点，返回高度0

        int left = maxDepth(root->left); // 左，递归求左子树的高度
        int right = maxDepth(root->right); // 右，递归求右子树的高度
        return max(left, right) + 1; // 中，返回左右子树的最大高度加上根节点的高度
    }
```

当然还可以使用层序遍历来解决，遍历的层数就是二叉树的高度，也就是最大深度。

```cpp
int maxDepth(TreeNode* root) {
    int depth = 0;
    if (!root) return depth;
    queue<TreeNode*> que;
    que.push(root);
    while (!que.empty()) { // 层序遍历
        depth++; // 每遍历一层，深度加1
        int size = que.size();
        while (size--) {
            TreeNode* node = que.front();
            que.pop();

            if (node->left) que.push(node->left);
            if (node->right) que.push(node->right);
        }
    }
    return depth;
}
```

## 二叉树的最小深度

[111.二叉树的最小深度](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/)

这里和[二叉树的最大深度](#二叉树的最大深度)不同的是，最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

- 如果一个节点是空节点，那么它的最小深度为0。
- 如果一个节点只有左子树，右子树为空，那么它的最小深度为左子树的最小深度+1。
- 如果一个节点只有右子树，左子树为空，那么它的最小深度为右子树的最小深度+1。

因此这里是与最大深度的本质区别，如果按照最大深度的写法，那么可能会返回空子树的深度。

同样是后序遍历：

```cpp
int minDepth(TreeNode* root) {
    if (!root) return 0;

    int left = minDepth(root->left); // 左
    int right = minDepth(root->right); // 右

    if (!root->left && root->right) { // 如果左子树为空，右子树不为空
        return right + 1; // 返回右子树的最小深度+1
    }
    if (root->left && !root->right) { // 如果右子树为空，左子树不为空
        return left + 1; // 返回左子树的最小深度+1
    }

    return min(left, right) + 1; // 返回左右子树的最小深度+1
}
```

## 总结

- 二叉树的翻转，可以使用前序遍历和后序遍历，也可以使用迭代的方式实现。
- 对称二叉树，要同时判断左右子树的相应左右节点是否对称。
- 对于于求二叉树的高度，用前序最合适
- 对于求二叉树的深度，用后序最合适
- 对于二叉树的最小深度，需要注意左右子树为空的情况。