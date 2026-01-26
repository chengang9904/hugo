+++
date = '2026-01-22T11:10:30+08:00'
draft = false
title = '448. 找到所有数组中消失的数字'
description = '简单 · 数组'
categories = ['Leetcode Hot 100']
tags = ['数组', '哈希表']
+++

## 题目

[448. 找到所有数组中消失的数字](https://leetcode.cn/problems/find-all-numbers-disappeared-in-an-array/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/find-all-numbers-disappeared-in-an-array/solutions/535822/zhao-dao-suo-you-shu-zu-zhong-xiao-shi-de-shu-zi-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：使用额外数组

```cpp
class Solution {
public:
    vector<int> findDisappearedNumbers(vector<int>& nums) {
        int n = nums.size();
        vector<bool> count(n + 1, false);
        for (int i = 0; i < n; i ++) {
            count[nums[i]] = true;
        }

        vector<int> res;
        for (int i = 1; i <= n; i ++) {
            if (!count[i]) {
                res.push_back(i);
            }
        }

        return res;
    }
};
```

时间复杂度：O(N)，其中 N 是数组 nums 的长度。我们需要遍历数组 nums 一次来标记出现的数字，然后再遍历一次 count 数组来找出消失的数字。

空间复杂度：O(N)，需要额外的 count 数组来存储数字的出现情况。
