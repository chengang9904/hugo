+++
date = '2026-03-04T16:43:02+08:00'
draft = false
title = '75 . 颜色分类'
descrption = '中等 · 数组 · 双指针'
categories = ['Leetcode Hot 100']
tags = ['数组', '双指针']


+++

## 题目

[75. 颜色分类](https://leetcode.cn/problems/sort-colors/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/sort-colors/solutions/388743/se-yan-se-fen-lei-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：双指针

```cpp

class Solution {
public:
    void sortColors(vector<int>& nums) {
        int low = 0, mid = 0, high = nums.size() - 1;
        while (mid <= high) {
            if (nums[mid] == 0) {
                swap(nums[low], nums[mid]);
                low++;
                mid++;
            } else if (nums[mid] == 1) {
                mid++;
            } else {
                swap(nums[mid], nums[high]);
                high--;
            }
        }
    }
};
```

## 方法二：计数排序

```cpp
class Solution {
public:
    void sortColors(vector<int>& nums) {
        int count[3] = {0, 0, 0};
        for (int num : nums) {
            count[num]++;
        }

        int index = 0;
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < count[i]; j++) {
                nums[index++] = i;
            }
        }
    }
};
```
