+++
date = '2026-01-05T13:17:12+08:00'
title = '128. 最长连续序列'
description = '中等 · 哈希表优化 · O(n)解法'
categories = ['Leetcode Hot 100']
tags = ['哈希表', '并查集']
+++


## 题目

[128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/longest-consecutive-sequence/solutions/535971/zui-chang-lian-xu-xu-lie-by-leetcode-solution-3pkl/?envType=problem-list-v2&envId=2cktkvj)


## 方法一：哈希表
使用哈希表存储数组中的元素，然后对于每个元素，检查其前驱元素是否存在，以此来找到连续序列的起点，并计算序列长度。

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> num_set;
        for (auto num : nums) {
            num_set.insert(num);
        }

        int res = 0;
        for (auto num : num_set) {
            if (!num_set.count(num - 1)) {
                int cur = num;
                int len = 1;

                while (num_set.count(cur + 1)) {
                    len ++;
                    cur ++;
                }

                res = max(res, len);
            }
        }

        return res;
    }
};
```