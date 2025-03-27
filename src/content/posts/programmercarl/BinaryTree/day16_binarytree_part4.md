---
title: Day16-二叉树 part04
published: 2025-03-27
description: 二叉树，找树左下角的值，路径总和，从中序与后序遍历序列构造二叉树
tags: [C++, 二叉树, 递归, BFS, DFS]
category: 算法训练营
draft: false
---

## 找树左下角的值

[513.找树左下角的值](https://leetcode-cn.com/problems/find-bottom-left-tree-value/)

这里如果使用层序遍历那么就很轻松，只要记录最后一层的第一个节点的值即可。

```cpp
int findBottomLeftValue(TreeNode* root) {
    if (!root) return 0;
    queue<TreeNode*> que;
    que.push(root);
    int ans;
    while (!que.empty()) {
        int size = que.size();
        vector<int> vec;
        while (size--) {
            TreeNode* node = que.front();
            que.pop();
            vec.push_back(node->val);
            if (node->left) que.push(node->left);
            if (node->right) que.push(node->right);
        }
        ans = vec[0];
    }
    return ans;
}
```

当然也可以使用递归遍历的方式，我们只要找到深度最大的叶子节点，然后要找左节点的话，我们只需要使用前序遍历（或者是中序，后序都行，因为左一直在右的前面）来保证优先搜索左边节点，然后记录深度最大的叶子节点，此时就是树的最后一行最左边的值。

在递归过程中，每当深度大于当前最大深度时，我们就更新最大深度和最左边的值。所以可以定义两个全局变量。

```cpp
int maxDepth = INT_MIN; // 记录最大深度
int ans = 0;
```
在找最大深度的时候，递归的过程中依然要使用回溯。

我们可以有以下递归代码：

```cpp
int maxDepth = INT_MIN; // 全局变量记录最大深度
int result = 0; // 全局变量记录最左边的值
void traversal(TreeNode* node, int depth) { // 1. 传入根节点，和当前深度
    // 2. 递归终止条件，遇到叶子节点
    if (!node->left && !node->right) { // 找到叶子节点，比较当前深度是否大于最大深度
        if (depth > maxDepth) { // 如果当前深度大于最大深度，更新最大深度和结果
            maxDepth = depth;
            result = node->val;
        }
    }
    // 3. 单层递归逻辑
    if (node->left) { // 左
        depth++;
        traversal(node->left, depth); // -> traversal(node->left, depth + 1);
        depth--; // 回溯
    }
    if (node->right) { // 右
        depth++;
        traversal(node->right, depth); // -> traversal(node->right, depth + 1);
        depth--; // 回溯
    }
    return;
}

int findBottomLeftValue(TreeNode* root) {
    traversal(root, 0);
    return result;
}
```

要注意的是这里的遍历顺序其实只需要保证左节点在右节点前面就行，因为没有对中间节点的处理过程。

## 路径总和

[112.路径总和](https://leetcode-cn.com/problems/path-sum/)

这里和上面一样因为没有中间节点的处理逻辑，所以用什么遍历顺序都可以。

关于递归函数什么时候需要返回值，可以根据需不需要处理递归返回值来进行判断。

- 如果需要搜索整棵二叉树且不用处理递归返回值，递归函数就不要返回值。

- 如果需要搜索整棵二叉树且需要处理递归返回值，递归函数就需要返回值。

- 如果要搜索其中一条符合条件的路径，那么递归一定需要返回值，因为遇到符合条件的路径了就要及时返回。

这里进行统计的值，我们可以使用减法的方式来统计，当遍历到叶子节点是，如果当前总和为0，那么就是找到了一条符合条件的路径。

这样的做法要比使用加法更简洁，很多时候使用减法的做法会更简单。

整体的代码如下：

```cpp
 bool dfs(TreeNode* node, int sum) { // 1. 返回值：是否找到一条符合条件的路径，参数：当前节点，当前总和
    if (!node->left && !node->right) { // 2. 递归终止条件，遇到叶子节点
        if (sum == 0) return true; // 如果当前总和为0，那么就是找到了一条符合条件的路径
        else return false; // 否则返回false
    }
    // 3. 单层递归逻辑
    if (node->left) { // 左
        sum -= node->left->val; // 先减去当前节点的值
        if (dfs(node->left, sum)) return true; // -> if(dfs(node->left, sum - node->left->val))
        sum += node->left->val; // 回溯， 加上当前节点的值
    }

    if (node->right) {
        sum -= node->right->val; // 先减去当前节点的值
        if (dfs(node->right, sum)) return true; // -> if(dfs(node->right, sum - node->right->val))
        sum += node->right->val; // 回溯， 加上当前节点的值
    }
    return false;
}
bool hasPathSum(TreeNode* root, int targetSum) {
    if (!root) return false;
    return dfs(root, targetSum - root->val); // 这里要提前减去root的值
}
```
要注意的是回溯问题，先减去当前节点的值，然后递归，递归完之后再加上当前节点的值。

## 从中序与后序遍历序列构造二叉树

[106.从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

在二叉树遍历顺序中：
- 中序遍历：左 -> 中 -> 右
- 后序遍历：左 -> 右 -> 中

所以在后序遍历中，最后一个节点一定是根节点，然后在中序遍历中，根节点的左边是左子树，右边是右子树。

因此可以根据这个规则，来切割中序数组，分成左右子树

然后再根据左右子树在后序遍历中的位置，来切割后序数组，分成左右子树

当后序数组为空时，就返回空节点。

所以大体的思路可以分为以下几步：

1. 如果数组大小为零的话，说明是空节点了。

2. 如果不为空，那么取后序数组最后一个元素作为节点元素。

3. 找到后序数组最后一个元素在中序数组的位置，作为切割点

4. 切割中序数组，切成中序左数组和中序右数组 （顺序别搞反了，一定是先切中序数组）

5. 切割后序数组，切成后序左数组和后序右数组

6. 递归处理左区间和右区间

那么难点就在于如何切分中序数组和后序数组。

在切割的过程中会产生四个区间，分别是中序左区间，中序右区间，后序左区间，后序右区间。

切割点在后序数组的最后一个元素，就是用这个元素来切割中序数组的，所以必要先切割中序数组。

中序数组相对比较好切，找到切割点（后序数组的最后一个元素）在中序数组的位置，然后切割：

```cpp
// 4. 切割中序数组
// 左子树区间 [0, delimiterIndex)
vector<int> leftInorder(inorder.begin(), inorder.begin() + delimiterIndex); 
// 右子树区间 [delimiterIndex + 1, end)
vector<int> rightInorder(inorder.begin() + delimiterIndex + 1, inorder.end()); 
```

对于后序数组，首先需要舍弃最后一个元素，因为最后一个元素是根节点已经被使用了，然后再找到切割点在中序数组。后序数组没有明确的切割元素来进行左右切割，不像中序数组有明确的切割点，切割点左右分开就可以了。

此时有一个很关键的点，就是中序数组大小一定是和后序数组的大小相同的，那么我们可以根据中序数组的大小来切割后序数组。

```cpp
// postorder 舍弃末尾元素
postorder.resize(postorder.size() - 1);

// 5. 切割后序数组
// 依然左闭右开，注意这里使用了左中序数组大小作为切割点
// [0, leftInorder.size)
vector<int> leftPostorder(postorder.begin(), postorder.begin() + leftInorder.size());
// [leftInorder.size(), end)
vector<int> rightPostorder(postorder.begin() + leftInorder.size(), postorder.end());
```

此时完整的代码如下：

```cpp
TreeNode* traversal(vector<int>& inorder, vector<int>& postorder) {
    // 1. 当后序数组为空，返回空节点 
    if (postorder.empty()) return nullptr; 
    // 2. 后序遍历数组最后一个元素，就是当前的中间节点
    int rootValue = postorder[postorder.size() - 1];
    TreeNode* root = new TreeNode(rootValue);

    // 当后序数组的大小为1时，也就是叶子节点，返回这个节点
    if (postorder.size() == 1) return root;

    // 3. 找到中序遍历的切割点
    int delimiterIndex;
    for (delimiterIndex = 0; delimiterIndex < inorder.size(); delimiterIndex++) {
        if (inorder[delimiterIndex] == rootValue) break;
    }
    // 4. 切割中序数组
    // 左子树区间 [0, delimiterIndex)
    vector<int> leftInorder(inorder.begin(), inorder.begin() + delimiterIndex); 
    // 右子树区间 [delimiterIndex + 1, end)
    vector<int> rightInorder(inorder.begin() + delimiterIndex + 1, inorder.end()); 

    // postorder 舍弃末尾元素
    postorder.resize(postorder.size() - 1);

    // 5. 切割后序数组
    // 依然左闭右开，注意这里使用了左中序数组大小作为切割点
    // [0, leftInorder.size)
    vector<int> leftPostorder(postorder.begin(), postorder.begin() + leftInorder.size());
    // [leftInorder.size(), end)
    vector<int> rightPostorder(postorder.begin() + leftInorder.size(), postorder.end());

    // 6. 递归处理左区间和右区间
    root->left = traversal(leftInorder, leftPostorder);
    root->right = traversal(rightInorder, rightPostorder);

    return root;
}

TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
    if (inorder.size() == 0 || postorder.size() == 0) return NULL;
    return traversal(inorder, postorder);
}
```

对于[105.从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)也是类似的思路，只是切割点在前序数组的第一个元素。因为前序遍历是中左右，所以前序数组的第一个元素就是根节点。

## 总结

关于递归函数的写法，什么时候需要返回值，传入的参数是什么，有些还是不太清楚，需要多练习。

像这种查找路径或者是涉及到一些需要深度来判断的问题，我们需要多留意回溯的问题，有递归就一定有回溯。

从中序数组和后序数组，或者是从前序数组和中序数组都可以构造二叉树，因为可以根据中间节点来区分左右子树。
但是从前序数组和后序数组是无法构造二叉树的，因为前序和后序数组都没有明确的切割点来区分左右子树。
