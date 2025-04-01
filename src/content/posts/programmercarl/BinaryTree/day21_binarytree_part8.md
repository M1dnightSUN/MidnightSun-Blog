---
title: Day21-二叉树 part08
published: 2025-04-01
description: 二叉树，修建二叉搜索树，将有序数组转换为平衡二叉搜索树，把二叉搜索树转换为累加树
tags: [C++, 二叉树, 队列, BFS, DFS, 递归, 分治, 二叉搜索树]
category: 算法训练营
draft: false
---

## 修剪二叉搜索树

[669. 修剪二叉搜索树](https://leetcode-cn.com/problems/trim-a-binary-search-tree/)

本题可以使用递归来完成，利用**二叉搜索树的特性**，我们可以有效地剪枝不符合条件的子树。主要的处理逻辑分为以下几种情况：

1. **递归终止条件**  
   当当前节点为空时，说明已经遍历到空节点，直接返回 `nullptr`。

2. **当前节点的值小于 low**  
   说明当前节点及其左子树所有节点的值都小于 `low`，根据 BST 的性质，左子树上的所有节点也都不符合条件，因此可以**直接舍弃左子树和当前节点**，递归修剪右子树，并将修剪结果返回。

3. **当前节点的值大于 high**  
   说明当前节点及其右子树所有节点的值都大于 `high`，同理可以**舍弃右子树和当前节点**，递归修剪左子树，并将修剪结果返回。

4. **当前节点的值在区间 [low, high] 之内**  
   当前节点是合法的，我们保留该节点，并继续分别修剪它的左右子树。将修剪后的左子树接到当前节点的 `left` 指针，将修剪后的右子树接到当前节点的 `right` 指针。

5. **最终返回当前节点**  
   经过上述判断和修剪后，当前节点要么是合法节点，要么已经替换为左右子树中修剪后合法的部分，最终返回该节点。

:::important
不能在当前节点值不在 [low, high] 的时候直接 return nullptr，而是应该继续处理它的左子树或右子树。

例如：

- 如果当前节点值小于 `low` ，那么它的左子树所有值一定也小于 `low`，可以舍弃，但右子树可能有合法值，需要递归处理右子树。

- 同理，如果当前节点值大于 `high`，左子树可能有合法值，不能直接舍弃整个子树。


:::

例如一棵树的结构如下：
```ini
      3
     / \
    3
   / \
  0   4
   \
    2
   /
  1

L = 1, R = 3
遇到节点 0 时判断其不在范围内，直接返回 nullptr，但 0 的右子树中有 2 和 1，是合法的节点，却被错误舍弃了。
```

因此我们可以根据上面的逻辑写出以下的代码：

```cpp
TreeNode* trimBST(TreeNode* root, int low, int high) {
    // 1. 递归终止条件，如果当前节点为空，直接返回空
    if (!root) return nullptr;
    // 2. 如果当前节点的值小于 low，说明当前节点及其左子树都不符合要求，那么我们需要递归修建右子树
    if (root->val < low) {
        TreeNode* right = trimBST(root->right, low, high);
        return right;
    }
    // 3. 如果当前节点的值大于high，说明当前节点及其右子树不符合要求，那么需要递归修剪其左子树
    if (root->val > high) {
        TreeNode* left = trimBST(root->left, low, high);
        return left;
    }

    // 将向左递归的结果用root的左孩子接住
    root->left = trimBST(root->left, low, high);
    // 将向右递归的结果用root的右孩子接住
    root->right = trimBST(root->right, low, high);

    return root;
}
```

## 将有序数组转换为平衡二叉搜索树

[108. 将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)

原标题是“将有序数组转换为二叉搜索树”，但实际上要注意是将有序数组转换为**平衡二叉搜索树**。

首先平衡树的定义是：对于每个节点，左子树和右子树的高度差不超过 1。然后同时保证这棵树是一棵二叉搜索树。

在构造二叉树时，一般都可以默认使用数组的中间元素作为根节点。其实本质就是寻找数组的分割点，以分割点作为根节点，左边的元素作为左子树，右边的元素作为右子树。

核心思路：分治 + 递归

将升序数组转换为平衡 BST，本质上就是要保持结构对称：

- 每次选择数组的“中间元素”作为根节点

- 左半部分构成左子树

- 右半部分构成右子树

- 对左右子数组递归执行相同操作

所以我们可以有以下的步骤：

1. 定义递归函数 `buildBST(nums, left, right)`，表示构造从 `nums[left]` 到 `nums[right]` 的子数组对应的平衡 BST
2. 如果 `left > right`，说明区间无效，返回空指针（递归终止条件）
3. 计算中间索引：`mid = (left + right) / 2`
4. 创建根节点：`TreeNode* root = new TreeNode(nums[mid])`
5. 递归构造左子树：`root->left = buildBST(nums, left, mid - 1)`
6. 递归构造右子树：`root->right = buildBST(nums, mid + 1, right)`
7. 返回当前构建的 `root`

最终调用 `buildBST(nums, 0, nums.size() - 1)` 得到整棵树的根节点

    
```cpp
// 从 nums[left] 到 nums[right] 构造平衡 BST
TreeNode* buildBST(const vector<int>& nums, int left, int right) {
    // 递归终止条件：区间为空时返回空指针
    if (left > right) return nullptr;
    // 取中间位置作为当前子树的根节点，保持平衡性
    int mid = left + (right - left) / 2; // 防止整型溢出
    TreeNode* root = new TreeNode(nums[mid]); // 创建根节点

    root->left = buildBST(nums, left, mid - 1); // 递归构建左子树，左半部分 [left, mid - 1]
    root->right = buildBST(nums, mid + 1, right); // 递归构建右子树，右半部分 [mid + 1, right]

    return root;
}

TreeNode* sortedArrayToBST(vector<int>& nums) {
    return buildBST(nums, 0, nums.size() - 1);
}
```

## 把二叉搜索树转换为累加树

[538. 把二叉搜索树转换为累加树](https://leetcode-cn.com/problems/convert-bst-to-greater-tree/)

累加树的定义：

> 对每个节点，它的新值 = 原值 + 所有 **大于它的节点值之和**

也就是说：

- 每个节点都会“获得”所有比它大的节点值的“加成”

- 原本的 BST 被“累加”成了一个新的树结构

我们可以把二叉搜素树看作一个“有序的数组”，用中序遍历来得到这个数组，以这个数组作为例子。

假设原始 BST：

```ini
      5
     / \
    2   13

中序遍历（左→中→右）得到的序列是：2, 5, 13（升序）  
```

现在，我们希望变成累加树：

遍历顺序换成**右→中→左**（从大到小）  
初始 `sum = 0`

开始反向遍历：

1. **访问 13**
   - sum += 13 → sum = 13
   - 节点值 = 13

2. **访问 5**
   - sum += 5 → sum = 18
   - 节点值 = 18

3. **访问 2**
   - sum += 2 → sum = 20
   - 节点值 = 20

结果树：

```ini
      18
     /  \
   20    13
```

把这个树理解为一个“从右向左”不断“累加贡献值”的结构，累加树就非常好理解了。

所以我们需要实现一个类似数组的“倒序遍历”二叉搜索树，并在遍历的过程中进行累加。从树中可以看出累加的顺序是右中左，那么这个“倒序遍历”其实就是**反向中序遍历**。所以我们需要反中序遍历这个二叉树，然后顺序累加就可以了。

在数组中，我们可以用刷换那个指针来实现反向遍历，在二叉搜索树中，我们也可以使用双指针来实现累加的过程。

> 用sum来记录当前的累加值，遍历到每个节点时，先把sum加到当前节点的值上，然后再更新sum。

整体代码如下：

```cpp
int sum = 0; // 用于记录累加值
void traverse(TreeNode* root) {
    if (root == nullptr) return;
    // 先访问右子树（较大的节点）
    traverse(root->right); // 右
    // 累加当前节点值
    // 中
    sum += root->val;
    root->val = sum;
    // 再访问左子树（较小的节点）
    traverse(root->left); // 左
}

TreeNode* convertBST(TreeNode* root) {
    traverse(root);
    return root;
}
```


