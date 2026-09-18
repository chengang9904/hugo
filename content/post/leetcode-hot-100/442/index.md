+++
date = '2026-09-18T08:57:59-07:00'
draft = false
title = '442. 数组中重复的数据'
tags = ['数组']
categories = ['Leetcode Hot 100']
+++

## 题目

[442. 数组中重复的数据](https://leetcode.cn/problems/find-all-duplicates-in-an-array/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/find-all-duplicates-in-an-array/solutions/535727/k-ge-yi-zu-fan-zhuan-lian-biao-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)


```cpp
class Solution {
public:
    vector<int> findDuplicates(vector<int>& nums) {
        vector<int> res;
        for (int i = 0; i < nums.size(); i++) {
            int index = abs(nums[i]) - 1;
            if (nums[index] < 0) {
                res.push_back(abs(nums[i]));
            } else {
                nums[index] = -nums[index];
            }
        }
        return res;
    }
};
```
