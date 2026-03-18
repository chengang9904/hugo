+++
date = '2026-03-18T21:08:39+08:00'
draft = false
title = '215. 数组中的第K个最大元素'
categories = ['Leetcode Hot 100']
tags = ['快速选择', '堆']
description = '中等 · 快速选择 · 堆'
+++

## 题目

[215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/kth-largest-element-in-an-array/solutions/522553/shu-zu-zhong-de-di-kge-zui-da-yuan-su-by-leetcode/?envType=problem-list-v2&envId=2cktkvj)

## 复习快速排序

y总模版，烂熟于心
![快速排序板子](https://www.acwing.com/activity/content/code/content/5607549/)

```cpp
class Solution {
public:
    int quick_sort(vector<int>& nums, int l, int r, int k) {
        if (l == r) return nums[l];

        int x = nums[l + r >> 1];
        int i = l - 1, j = r + 1;
        while (i < j) {
            while (nums[++i] > x);
            while (nums[--j] < x);
            if (i < j) swap(nums[i], nums[j]);
        }

        if (k <= j) return quick_sort(nums, l, j, k);
        return quick_sort(nums, j + 1, r, k);
    }

    int findKthLargest(vector<int>& nums, int k) {
        return quick_sort(nums, 0, nums.size() - 1, k - 1);
    }
};
```

## 堆排序

依然是y总的堆排模板
![堆排序板子](https://www.acwing.com/activity/content/code/content/8228029/)

```cpp
class Solution {
public:
    void down(vector<int>& h, int u, int size) {
        int t = u;
        if (u * 2 <= size && h[u * 2] > h[t]) t = 2 * u;
        if (u * 2 + 1 <= size && h[u * 2 + 1] > h[t]) t = 2 * u + 1;
        if (u != t) {
            swap(h[u], h[t]);
            down(h, t, size);
        }
    }

    int findKthLargest(vector<int>& nums, int k) {
        vector<int> h;
        h.push_back(0);

        for (int i = 0; i < nums.size(); i ++) {
            h.push_back(nums[i]);
        }

        for (int i = nums.size() / 2; i > 0; i --) down(h, i, nums.size());

        int size = nums.size();
        for (int i = 0; i < k - 1; i ++) {
            h[1] = h[size];
            down(h, 1, -- size);
        }

        return h[1];
    }
};
```
