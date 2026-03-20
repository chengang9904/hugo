+++
date = '2026-03-20T08:12:10+08:00'
draft = false
title = '34. 在排序数组中查找元素的第一个和最后一个位置'
categories = ['Leetcode Hot 100']
tags = ['二分查找']
description = '中等 · 二分查找'
+++

## 题目

[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/solutions/522553/zai-pai-xu-shu-zu-zhong-cha-zhao-yuan-su-de-di-yi-ge-he-zui-hou-yi-ge-wei-zhi-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

直接开始默写y总的二分查找模板 ![二分查找模板](https://www.acwing.com/activity/content/code/content/8221642/)

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        vector<int> res(2);
        if (!nums.size()) return {-1, -1};

        // 先找左端点
        int l = 0, r = nums.size() - 1;
        while (l < r) {
            int mid = l + r >> 1;
            if (nums[mid] >= target) r = mid;
            else l = mid + 1;
        }
        if (nums[r] != target) res[0] = -1;
        else res[0] = r;
        // 在找右端点

        l = 0, r = nums.size() - 1;
        while (l < r) {
            int mid = l + r + 1>> 1;
            if (nums[mid] <= target) l = mid;
            else r = mid - 1;
        }
        if (nums[r] != target) res[1] = -1;
        else res[1] = r;
        return res;
    }
};
```
