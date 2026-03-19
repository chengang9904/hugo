+++
date = '2026-03-19T20:41:47+08:00'
draft = false
title = '152. 乘积最大子数组'
categories = ['Leetcode Hot 100']
tags = ['动态规划']
description = '中等 · 动态规划'
+++

## 题目

[152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/maximum-product-subarray/solutions/522553/chen-ji-zui-da-zi-shu-zu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 暴力左右指针

```cpp
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        int res = nums[0];
        for (int i = 0; i < nums.size(); i ++) {
            int tmp = nums[i];
            res = max(res, tmp);
            for (int j = i + 1; j < nums.size(); j ++) {
                tmp *= nums[j];
                res = max(res, tmp);
            }
        }

        return res;
    }
};
```

## 动态规划

思路：动态规划，维护两个状态 `imax` 和 `imin`，分别表示以当前元素结尾的最大乘积和最小乘积。因为负数会导致乘积变小，所以当遇到负数时，需要交换 `imax` 和 `imin` 的值。然后更新 `imax` 和 `imin`，并更新结果。

```cpp
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        int res = nums[0];
        int imax = res, imin = res;

        for (int i = 1; i < nums.size(); i ++) {
            if (nums[i] < 0) swap(imax, imin);
            imax = max(nums[i], imax * nums[i]);
            imin = min(nums[i], imin * nums[i]);
            res = max(res, imax);
        }

        return res;
    }
};
```
