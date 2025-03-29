---
title: Day17-二叉树 part05
published: 2025-03-28
description: 二叉树，二叉搜索树，最大二叉树，合并二叉树，验证二叉搜索树
tags: [C++, 二叉树, 队列, BFS, DFS, 递归, 迭代, 二叉搜索树]
category: 算法训练营
draft: false
---

## 最大二叉树

[654.最大二叉树](https://leetcode-cn.com/problems/maximum-binary-tree/)

这里我们可以参考使用前序+中序数组来构造二叉树的思路来写，首先根节点就是数组中的最大值，然后根据最大值的下标，我们可以区分左和右区间。

假设最大值下标是 `maxIndex`，那么左区间就是 `[left, maxIndex)`，右区间就是`[maxIndex+1, right)`。

然后在区间中分别递归调用，直到区间的左边界大于等于右边界。

```cpp
TreeNode* traversal(vector<int>& nums, int left, int right) { // 传入数组的左边界和有边界
    if (left >= right) { // 如果左边界 >= 右边界， 那么就返回空节点
        return nullptr;
    }
    // 找到最大值和下标
    int maxIndex = left; // 初始化最大值的下标是数组区间的最左边的值的下标
    for (int i = left + 1; i < right; i++) { // 寻找最大值的下标，从左边界的下一个开始
        if (nums[i] > nums[maxIndex]) {
            maxIndex = i;
        }
    }
    // 前序遍历
    TreeNode* node = new TreeNode(nums[maxIndex]); // 定义新节点，最大值
    // 左区间 [left, maxindex)
    node->left = traversal(nums, left, maxIndex); // 左
    // 右区间 [maxIndex + 1, right)
    node->right = traversal(nums, maxIndex + 1, right); // 右

    return node;
}

TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
    if (nums.empty()) return nullptr;
    int left = 0, right = nums.size(); // 因为需要左开右闭区间，所以是[0, nums.size())
    return traversal(nums, left, right);
}
```

这里我觉得这种方式更好理解一些所以采用的这种写法，如果要完全按照之前的思路分成两个数组来做也是可以的，就是稍微麻烦一点。

## 合并二叉树

[617.合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)

### 递归实现

这题的思路其实不需要想的太复杂，遍历两棵树的逻辑其实和遍历一棵树是一样的，只不过需要传入两个树的根节点。

所以我们的递归函数需要输入的是两个树的根节点， 返回合并后的根节点。

然后要确定递归函数的终止条件：
1. 如果两个树都是空的，返回空节点
2. 如果树1是空的，那么合并的只有树2，也就是可以返回树2的根节点
3. 如果树2是空的，那么合并的只有树1，也就是可以返回树1的根节点

之后是确定单层的递归逻辑，当两个树都不为空时，我们就可以新建一个节点，值为两棵树的根节点之和。
然后递归调用左子树和右子树的合并函数，分别传入两棵树的左子树和右子树。

整体代码如下：

```cpp
TreeNode* mergeTrees(TreeNode* ro root1, TreeNode* root2) {
    if (ro root1 == nullptr && root2 == nullptr) { // 如果两个树都是空的，返回空
        return nullptr;
    }
    if (ro root1 == nullptr) { // 如果树1是空的，返回树2
        return root2;
    }
    if (root2 == nullptr) { // 如果树2是空的，返回树1
        return ro root1;
    }
    // 如果两个树都不为空，那么就合并
    TreeNode* node = new TreeNode(ro root1->val + root2->val); // 新建一个节点，值为两棵树的根节点之和
    node->left = mergeTrees(ro root1->left, root2->left); // 左子树合并
    node->right = mergeTrees(ro root1->right, root2->right); // 右子树合并

    return node; // 返回合并后的节点
}
```

其实这里的遍历顺序用三种之一都可以，不过用前序遍历更好理解。

### 迭代实现

对于同时操作两棵树，我们可以使用队列，将两棵树的节点同时加入队列进行比较，最后我们可以返回树1的根节点。

同样的，我们有以下条件：
1. 如果两棵树左节点都不为空，加入队列
2. 如果两棵树右节点都不为空，加入队列
3. root1的左节点 为空 root2左节点不为空，就赋值过去
4. root1的右节点 为空 root2右节点不为空，就赋值过去

最后我们只需要返回树1的根节点即可（因为修改的是root1）。

```cpp
TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
    if  root1 == NULL) return root2;
    if (root2 == NULL) return root1;
    queue<TreeNode*> que;
    que.push root1);
    que.push(root2);
    while(!que.empty()) {
        TreeNode* node1 = que.front(); que.pop();
        TreeNode* node2 = que.front(); que.pop();
        // 此时两个节点一定不为空，val相加
        node1->val += node2->val;

        // 如果两棵树左节点都不为空，加入队列
        if (node1->left != NULL && node2->left != NULL) {
            que.push(node1->left);
            que.push(node2->left);
        }
        // 如果两棵树右节点都不为空，加入队列
        if (node1->right != NULL && node2->right != NULL) {
            que.push(node1->right);
            que.push(node2->right);
        }

        //  root1的左节点 为空 t2左节点不为空，就赋值过去
        if (node1->left == NULL && node2->left != NULL) {
            node1->left = node2->left;
        }
        //  root1的右节点 为空 t2右节点不为空，就赋值过去
        if (node1->right == NULL && node2->right != NULL) {
            node1->right = node2->right;
        }
    }
    return root1;
}
```

## 二叉搜索树中的搜索

[700.二叉搜索树中的搜索](https://leetcode-cn.com/problems/search-in-a-binary-search-tree/)

### 递归实现

因为二叉搜索树的性质是左子树的值小于根节点的值，右子树的值大于根节点的值，所以我们可以根据这个性质来进行搜索。

- 如果当前节点的值等于目标值，返回当前节点
- 如果当前节点的值大于目标值，搜索左子树
- 如果当前节点的值小于目标值，搜索右子树

根据这个逻辑我们可以有以下递归的写法：

```cpp
TreeNode* searchBST(TreeNode* root, int val) {
    if (!root || root->val == val) return root; // 1. 如果当前节点为空或者当前节点的值等于目标值，返回当前节点
    // 2. 如果当前节点的值大于目标值，搜索左子树
    if (root->val > val) return searchBST(root->left, val);
    // 3. 如果当前节点的值小于目标值，搜索右子树
    if (root->val < val) return searchBST(root->right, val);
    // 都没有找到，返回空节点
    return nullptr;
}
```

### 迭代实现

迭代的写法相对就简单了，因为二叉搜索树的特殊性，也就是节点的有序性，可以不使用辅助栈或者队列就可以写出迭代法。并且对于二叉搜索树，不需要回溯的过程，因为节点的有序性就帮我们确定了搜索的方向。

从根节点开始，如果当前节点的值等于目标值，返回当前节点；如果当前节点的值大于目标值，搜索左子树；如果当前节点的值小于目标值，搜索右子树。

```cpp
TreeNode* searchBST(TreeNode* root, int val) {
    TreeNode* node = root; // 定义一个节点，指向根节点
    while (node != nullptr) { // 如果节点不为空
        if (node->val == val) { // 如果节点的值等于目标值，返回该节点
            return node;
        } else if (node->val < val) { // 如果节点的值小于目标值，去右子树查找
            node = node->right;
        } else { // 如果节点的值大于目标值，去左子树查找
            node = node->left;
        }
    }
    return nullptr; // 如果没有找到，返回空
}
```

## 验证二叉搜索树

[98.验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

在二叉搜索树中，每个节点的值都大于其左子树的所有节点的值，并且小于其右子树的所有节点的值。

因此使用中序遍历的方式来遍历二叉搜索树，得到的结果应该是一个升序的数组。

所以我们可以使用这个特性，将中序遍历的结果存储在一个数组中，然后判断这个数组是否是升序的。

```cpp
void inorder(TreeNode* root, vector<int>& nums) {
    if (root == nullptr) { // 如果节点为空，返回
        return;
    }
    inorder(root->left, nums); // 左子树
    nums.push_back(root->val); // 中序遍历，先左子树，再根节点，最后右子树
    inorder(root->right, nums); // 右子树
}
bool isValidBST(TreeNode* root) {
    vector<int> nums; // 定义一个数组，存储中序遍历的结果
    inorder(root, nums); // 中序遍历
    for (int i = 1; i < nums.size(); i++) { // 遍历数组
        if (nums[i] <= nums[i - 1]) { // 如果当前值小于等于前一个值，返回false
            return false;
        }
    }
    return true; // 如果没有返回false，说明是二叉搜索树，返回true
}
```
