---
title: Day15-二叉树 part03
published: 2025-03-26
description: 二叉树，平衡二叉树，二叉树的所有路径，左子叶之和，完全二叉树的节点个数
tags: [C++, 二叉树, 递归, BFS, DFS]
category: 算法训练营
draft: false
---

## 平衡二叉树

[110.平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree/)

平衡二叉树的定义是：对于树中的每个节点，其左右子树的高度差的绝对值不超过1。

既然要求的是高度，那么可以使用后序遍历的方式，

1. 明确递归函数的参数和返回值

    参数：当前传入节点。 返回值：以当前传入节点为根节点的树的高度。

    如果当前传入节点为根节点的二叉树已经不是二叉平衡树了，还返回高度的话就没有意义了。

    所以如果已经不是二叉平衡树了，可以返回-1 来标记已经不符合平衡树的规则了。

2. 明确终止条件

    递归的过程中依然是遇到空节点了为终止，返回0，表示当前节点为根节点的树高度为0

3. 明确单层递归的逻辑

    如何判断以当前传入节点为根节点的二叉树是否是平衡二叉树呢？当然是其左子树高度和其右子树高度的差值。

    分别求出其左右子树的高度，然后如果差值小于等于1，则返回当前二叉树的高度，否则返回-1，表示已经不是二叉平衡树了。

那么整体代码可以如下：

```cpp
int checkHeight(TreeNode* root) {
    if (!root) return 0;
    int left = checkHeight(root->left); // 左
    if (left == -1) return -1; // 如果左子树已经不是平衡树了，直接返回-1

    int right = checkHeight(root->right); // 右
    if (right == -1) return -1; // 如果右子树已经不是平衡树了，直接返回-1

    if (abs(left - right) > 1) return -1; // 如果当前节点的左右子树高度差大于1，说明不是平衡树，直接返回-1
    return max(left, right) + 1; // 是平衡树的情况下返回当前节点为根节点的高度
}

bool isBalanced(TreeNode* root) {
    return checkHeight(root) != -1; // 只需要比较是否等于-1即可
}
```

## 二叉树的所有路径

[257.二叉树的所有路径](https://leetcode-cn.com/problems/binary-tree-paths/)

这里要求的是从根节点到叶子节点的所有路径，所以很自然可以想到使用前序遍历。因为前序遍历是中左右，符合从根节点到叶子节点的路径。

并且在这里涉及到回溯，每当找完一条路径（也就是访问到叶子节点的时候），需要回退到上一个节点，继续遍历其他路径。

1. 明确递归函数的参数和返回值

    参数：当前传入节点，一个用来存储路径的 `path`，一个用来存储所有路径的结果 `result`。
    ```cpp
    void traversal(TreeNode* node, vector<int>& path, vector<string>& result)
    // 中，因为最后一个节点也要加入到path中
    path.push_back(node->val);
    ```

2. 明确终止条件

    递归的过程中遇到叶子节点的时候，那么需要把 `path`中存储的节点记录下来保存到 `result`中，然后返回。在这里可以不需要判断当前节点是否为空。

    ```cpp
    if (!node->left && !node->right) { // 终止条件：遇到叶子节点
            string sPath;
            for (int i = 0; i < path.size() - 1; i++) { // 遍历到倒数第二个
                sPath += to_string(path[i]);
                sPath += "->";
            }
            sPath += to_string(path[path.size() - 1]); // 最后一个节点不需要加->
            result.push_back(sPath);
            return;
        }
    ```

3. 明确单层递归的逻辑

    然后是递归和回溯的过程，上面说过没有判断是否为空，那么在这里递归的时候，如果为空就不进行下一层递归了。
    
    需要注意的是回溯和递归是一一对应的，因此在每次递归之后，需要回溯到上一个节点。

    所以递归前要加上判断语句，下面要递归的节点是否为空，整体如下

    ```cpp
    if (node->left) {
        traversal(node->left, path, result);
        path.pop_back(); // 回溯
    }
    if (node->right) {
        traversal(node->right, path, result);
        path.pop_back(); //回溯
    }
    ```

最后的代码如下：

```cpp
vector<string> binaryTreePaths(TreeNode* root) {
    vector<string> ans;
    vector<int> path;
    if (!root) return ans;
    traversal(root, path, ans);
    return ans;
}
```

## 左子叶之和

[404.左叶子之和](https://leetcode-cn.com/problems/sum-of-left-leaves/)

这里提到的是求左子叶之和，我们可以使用后序遍历的方式，将左右子树的左子叶之和返回给上一层。

那么如何判断一个节点是左子叶呢？

:::tip
如果一个节点的左节点不为空，且该左节点的左右节点都为空，那么这个节点就是左子叶。
:::

1. 明确递归函数的参数和返回值

    参数：当前传入节点。 返回值：当前节点的左子叶之和。

    ```cpp
    int traversal (TreeNode* node)
    ```

2. 明确终止条件

    如果遍历到空节点，那么左叶子值一定是0。
    当前遍历的节点是叶子节点，那其左叶子也必定是0。
    ```cpp
    if (!node) return 0;
    if (!node->left && !node->right) return 0;
    ```

3. 明确单层递归的逻辑

    根据后序遍历，先往左递归，因为最后的结果一定会出现在往左递归的过程中，所以结果要在这一过程中保存

    ```cpp
    int left = traversal(node->left);
    if (node->left && !node->left->left && !node->left->right) {
        left = node->left->val;
    }
    int right = traversal(node->right);
    
    return left + right;
    ```

整体代码如下：

```cpp
int traversal (TreeNode* node) {
    if (!node) return 0;
    if (!node->left && !node->right) return 0;

    int left = traversal(node->left); // 左
    if (node->left && !node->left->left && !node->left->right) {
        left = node->left->val; 
    }
    int right = traversal(node->right); // 右
    
    return left + right; // 中
}
int sumOfLeftLeaves(TreeNode* root) {
    return traversal(root);
}
```

## 完全二叉树的节点个数

[222.完全二叉树的节点个数](https://leetcode-cn.com/problems/count-complete-tree-nodes/)

这里可以简单地使用后序遍历的方式，分别求出左右子树的节点个数，然后加上根节点，就是整个完全二叉树的节点个数。

```cpp
int traversal(TreeNode* node) {
    if (!node) return 0;

    int left = traversal(node->left); // 左
    int right = traversal(node->right); // 右

    int res = left + right + 1; // 中
    return res;
}
int countNodes(TreeNode* root) {
    if (!root) return 0;
    return traversal(root);
}
```

在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，并且最下面一层的节点都集中在该层最左边的若干位置。若最底层为第 `h` 层，则该层包含 `1~ 2^(h-1)`  个节点。

完全二叉树只有两种情况
- 情况一：就是满二叉树，可以直接用 `2^树深度 - 1` 来计算，注意这里根节点深度为1。
- 情况二：最后一层叶子节点没有满。分别递归左孩子，和右孩子，递归到某一深度一定会有左孩子或者右孩子为满二叉树，然后依然可以按照情况1来计算。

这里的关键在于如何去判断一个左子树或者右子树是不是满二叉树。

在完全二叉树中，如果递归向左遍历的深度等于递归向右遍历的深度，那说明就是满二叉树。如图所示：

![完全二叉树1](https://file.kamacoder.com/pics/20220829163709.png)

在完全二叉树中，如果递归向左遍历的深度不等于递归向右遍历的深度，则说明不是满二叉树

![完全二叉树2](https://file.kamacoder.com/pics/20220829163709.png)

判断其子树是不是满二叉树，如果是则利用公式计算这个子树（满二叉树）的节点数量，如果不是则继续递归，那么 在递归三部曲中，第二部：终止条件的写法应该是这样的：

```cpp
if (root == nullptr) return 0; 
// 开始根据左深度和右深度是否相同来判断该子树是不是满二叉树
TreeNode* left = root->left;
TreeNode* right = root->right;
int leftDepth = 0, rightDepth = 0; // 这里初始为0是有目的的，为了下面求指数方便
while (left) {  // 求左子树深度
    left = left->left;
    leftDepth++;
}
while (right) { // 求右子树深度
    right = right->right;
    rightDepth++;
}
if (leftDepth == rightDepth) {
    return (2 << leftDepth) - 1; // 注意(2<<1) 相当于2^2，返回满足满二叉树的子树节点数量
}
```

第三部，单层递归的逻辑：（可以看出使用后序遍历）

```cpp
int leftTreeNum = countNodes(root->left);       // 左
int rightTreeNum = countNodes(root->right);     // 右
int result = leftTreeNum + rightTreeNum + 1;    // 中
return result;
```

最后的代码如下：

```cpp
int countNodes(TreeNode* root) {
    if (root == nullptr) return 0;
    TreeNode* left = root->left;
    TreeNode* right = root->right;
    int leftDepth = 0, rightDepth = 0; // 这里初始为0是有目的的，为了下面求指数方便
    while (left) {  // 求左子树深度
        left = left->left;
        leftDepth++;
    }
    while (right) { // 求右子树深度
        right = right->right;
        rightDepth++;
    }
    if (leftDepth == rightDepth) {
        return (2 << leftDepth) - 1; // 注意(2<<1) 相当于2^2，所以leftDepth初始为0
    }
    return countNodes(root->left) + countNodes(root->right) + 1;
}
```

