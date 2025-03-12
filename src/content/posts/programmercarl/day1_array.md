---
title: Day1-数组 part01
published: 2025-03-12
description: 数组基础, 二分查找，双指针
tags: [C++, 数组, 二分查找, 移除元素, 双指针]
category: 算法训练营
draft: false
---

## 数组基础

**数组是一种存放在连续空间内的相同类型的集合**

1. 连续存储：在内存中占据连续的地址空间。
2. 固定长度：数组的长度在定义时就已经确定，不可动态调整（某些语言例如python的列表可以动态扩展）。
3. 随机访问：支持O(1)的索引访问。
4. 相同数据类型：数组中的元素类型必须相同。

静态数组不能直接插入新元素，必须创建一个更大的数据复制元素。
同理数组也无法“删除”一个元素，所谓“删除”，指的是用原来的元素进行覆盖。

## 二分查找

[704.二分查找](https://leetcode.cn/problems/binary-search/description/)

这是一个典型的二分查找问题，给定一个有序数组，查找目标值的索引。
需要注意的是，二分查找的边界条件，以及mid的计算方法。

在通常的情况下，我们可以有两种写法：
:::note[1. 左闭右闭区间]
对于第一种写法：区间的定义这就决定了二分法的代码应该如何写，因为定义target在[left, right]区间，所以有如下两点：
:::

- while (left <= right) 要使用 <= ，因为left == right是有意义的，所以使用 <=

- if (nums[middle] > target) right 要赋值为 middle - 1，因为当前这个nums[middle]一定不是target，那么接下来要查找的左区间结束下标位置就是 middle - 1

```cpp
int search(vector<int>& nums, int target) {
        int left = 0, right = nums.size() - 1;
        while (left <= right) { //因为区间是左闭右闭，因此left与right相等是有意义的，所以使用<=
            int mid = left + (right - left) / 2;
            if (nums[mid] > target) { // target在左区间， 所以[left, mid - 1]，nums[mid]已经被排除，所以mid - 1
                right = mid - 1;
            }
            else if (nums[mid] < target) { // target在右区间，所以[mid + 1, right]
                left = mid + 1;
            }
            else {
                return mid;
            }
        }
        return -1;
    }
```
    
:::note[2. 左闭右开区间]
如果说定义 target 是在一个在左闭右开的区间里，也就是[left, right) ，那么二分法的边界处理方式则截然不同。有如下两点：
:::

- while (left < right)，这里使用 < ,因为left == right在区间[left, right)是没有意义的

- if (nums[middle] > target) right 更新为 middle，因为当前nums[middle]不等于target，去左区间继续寻找，而寻找区间是左闭右开区间，所以right更新为middle，即：下一个查询区间不会去比较nums[middle]
```cpp
int search_v2(vector<int>& nums, int target) {
        int left = 0, right = nums.size(); // 因为是左闭右开，因此right = nums.size()
        while (left < right) { // 因为区间是左闭右开，因此left与right相等是没有意义的，所以使用<
            int mid = left + (right - left) / 2;
            if (nums[mid] > target) { //target在左区间，所以[left, mid)，因为左闭右开，下一次查找为[left, mid), nums[mid]并不会被再次访问
                right = mid;
            }
            else if (nums[mid] < target) { //target在有区间，所以[mid + 1, right)
                left = mid + 1;
            }
            else {
                return mid;
            }
        }
        return -1;
    }
```

- **循环条件**：根据区间的定义，循环条件有所不同：
  - **左闭右闭区间**：使用 `while (left <= right)`，因为当 `left == right` 时，区间 `[left, right]` 仍然有效。
  - **左闭右开区间**：使用 `while (left < right)`，因为当 `left == right` 时，区间 `[left, right)` 已为空。


## 对数组元素进行操作

[27.移除元素](https://leetcode-cn.com/problems/remove-element/)

:::important[数组的特性]
数组中的元素无法直接删除，只能通过覆盖的方式进行删除。
:::
最直观的想法是，遍历数组，如果遇到目标值，就将后面的元素向前移动一位，然后数组长度减一。
这里如果使用暴力解法，时间复杂度为O(n^2), 空间复杂度为O(1)。

```cpp
int removeElement(vector<int>& nums, int val) {
        int n = nums.size();
        int i = 0;
        while (i < n) {
            if (nums[i] == val) {
                for (int j = i + 1; j < n; ++j) { 
                    nums[j - 1] = nums[j]; //将后面的元素向前移动一位
                }
                --n; // 数组长度减一
            } else {
                ++i;
            }
        }
        return n;
    }
```

:::tip[双指针]
双指针法（快慢指针法）可以将时间复杂度降低到O(n)，空间复杂度为O(1)。
:::

双指针的思路实际上是用一层for循环做了暴力解法中两层for循环的操作。

- 快指针：寻找新数组的元素 ，新数组就是不含有目标元素的数组
- 慢指针：指向更新 新数组下标的位置

```cpp
// 通过将快指针指向的元素赋值给慢指针指向的元素，实现元素的覆盖
int removeElement(vector<int>& nums, int val) {
        int slow = 0;
        for (int fast = 0; fast < nums.size(); fast++) { // 定义快指针，遍历数组
            if (nums[fast] != val) { 
                // 如果快指针不等于要删除的元素，那么将快指针指向的元素赋值给慢指针指向的元素，然后慢指针向后移动一位
                // 如果快指针等于要删除的元素，那么快指针向后移动一位，慢指针不动
                nums[slow++] = nums[fast];
            }
        }
        return slow;
}
```

动画演示如下[^1]：

[^1]: [代码随想录-27.移除元素](https://programmercarl.com/0027.%E7%A7%BB%E9%99%A4%E5%85%83%E7%B4%A0.html#%E5%8F%8C%E6%8C%87%E9%92%88%E6%B3%95)

![快慢指针动画演示](https://code-thinking.cdn.bcebos.com/gifs/27.%E7%A7%BB%E9%99%A4%E5%85%83%E7%B4%A0-%E5%8F%8C%E6%8C%87%E9%92%88%E6%B3%95.gif)

## 双指针

[977.有序数组的平方](https://leetcode-cn.com/problems/squares-of-a-sorted-array/)

### 一般解法
最简单的想法是，遍历数组，将每个元素平方，然后排序。
```cpp
vector<int> sortedSquares(vector<int>& nums) {
        for (int i = 0; i < nums.size(); i++) { //时间复杂度为O(n)
            nums[i] = nums[i] * nums[i];
        }
        sort(nums.begin(), nums.end()); // 这里使用的是快排，时间复杂度为O(nlogn)
        return nums;
    }
```
### 双指针解法
:::tip[有序数组的特性]
首先数组是有序的，但因为存在负数，所以平方后的数组不一定是有序的。根据数组有序的特性，左右两端的平方值其中之一一定为最大，所以我们可以使用双指针，从两端开始遍历，将较大的平方值放入结果数组中。
:::


```cpp
vector<int> sortedSquares(vector<int>& nums) {
        int left = 0;
        int right = nums.size() - 1;
        vector<int> res(nums.size());
        int index = res.size() - 1; // 从后往前填充结果数组

        while (left <= right) { // 这里使用的是左闭右闭
            int square_left = nums[left] * nums[left];
            int square_right = nums[right] * nums[right];

            if (square_left < square_right) { //当左边的平方小于右边的平方时，将右边的平方放入结果数组中
                res[index--] = square_right;
                right--;
            } else {                          //当左边的平方大于等于右边的平方时，将左边的平方放入结果数组中
                res[index--] = square_left;
                left++;
            }
        }
        return res;
    }
```

其动画演示如下[^2]：

[^2]: [代码随想录-977.有序数组的平方](https://programmercarl.com/0977.%E6%9C%89%E5%BA%8F%E6%95%B0%E7%BB%84%E7%9A%84%E5%B9%B3%E6%96%B9.html)

![有序数组平方动画演示](https://code-thinking.cdn.bcebos.com/gifs/977.%E6%9C%89%E5%BA%8F%E6%95%B0%E7%BB%84%E7%9A%84%E5%B9%B3%E6%96%B9.gif)


## 总结

Day1总归是结束了，大概会是个好的开始吧。

其实自己之前只会写python，这次使用C++一个很大的目的还是想让自己在做题的过程中，熟悉C++的用法，多掌握一些东西倒也不是坏事。

训练营第一天的内容其实很快就做完了，大把时间还是用来想怎么写markdown，这个博客还在刚刚完成部署的阶段，后日会慢慢完善吧。

要学的东西还是太多了。