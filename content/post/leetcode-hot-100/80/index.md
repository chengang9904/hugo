+++
date = '2026-09-10T07:37:12-07:00'
draft = false
title = '80. 删除排序数组中的重复项 II'
tags = ['数组', '双指针']
categories = ['Leetcode Hot 100']
+++


## 题目

[80. 删除排序数组中的重复项 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-array-ii/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/remove-duplicates-from-sorted-array-ii/solutions/535722/shan-chu-pai-xu-shu-zu-zhong-de-zhong-fu-yuan-su-ii-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

### 一开始的解法

```cpp
class Solution {
public:
    void swap2tail(vector<int>& nums, int i) {
        while (i < nums.size() - 1) {
            swap(nums[i], nums[i + 1]);
            i ++;
        }
    }

    int removeDuplicates(vector<int>& nums) {
        if (nums.size() <= 2) return nums.size();
        int n = nums.size();
        for (int i = 2; i < n; i ++) {
            while (nums[i] != nums[i - 1]) continue;
            while (nums[i] == nums[i - 1] && nums[i] == nums[i - 2]) {
                swap2tail(nums, i);
                n --;
                nums.pop_back();
            }
        }

        return n;
    }
};
```

**思路**：

像冒泡排序一样，当遇到第三次重复之后就将当前元素交换到数组末尾，并且数组长度减一，最后返回数组长度。

7/170，swap 有问题，数组会越界，整体逻辑也不对

官方题解双指针代码

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int n = nums.size();
        if (n <= 2) {
            return n;
        }
        int slow = 2, fast = 2;
        while (fast < n) {
            if (nums[slow - 2] != nums[fast]) {
                nums[slow] = nums[fast];
                ++slow;
            }
            ++fast;
        }
        return slow;
    }
};
```