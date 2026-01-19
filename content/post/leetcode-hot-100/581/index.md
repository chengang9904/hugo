+++
date = '2026-01-19T09:52:22+08:00'
draft = false
title = '581. 最短无序连续子数组'
description = '中等 · 数组'
categories = ['Leetcode Hot 100']
tags = ['数组', '排序']
+++

## 题目

[581. 最短无序连续子数组](https://leetcode.cn/problems/shortest-unsorted-continuous-subarray/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/shortest-unsorted-continuous-subarray/solutions/535706/zui-duan-wu-xu-lian-xu-zi-shu-zu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：排序

```cpp

class Solution {
public:
    int findUnsortedSubarray(vector<int>& nums) {
        vector<int> nums_sort = vector<int>(nums);
        sort(nums_sort.begin(), nums_sort.end());
        int l = 0, r = nums.size() - 1;
        while (l < nums.size() && nums[l] == nums_sort[l]) {
            l ++;
        }

        while (r >= 0 && nums[r] == nums_sort[r]) {
            r --;
        }

        return r >= l ? r - l + 1 : 0;
    }
};
```

时间复杂度：O(n log n)，其中 n 是数组的长度。排序需要 O(n log n) 的时间。

空间复杂度：O(n)，需要额外的数组来存储排序后的结果。

## 方法二：栈

```cpp
class Solution {
public:
    int findUnsortedSubarray(vector<int>& nums) {
        int n = nums.size();
        stack<int> stk;
        int l = n, r = -1;

        for (int i = 0; i < n; ++i) {
            while (!stk.empty() && nums[stk.top()] > nums[i]) {
                l = min(l, stk.top());
                stk.pop();
            }
            stk.push(i);
        }

        stk = stack<int>();

        for (int i = n - 1; i >= 0; --i) {
            while (!stk.empty() && nums[stk.top()] < nums[i]) {
                r = max(r, stk.top());
                stk.pop();
            }
            stk.push(i);
        }

        return r >= l ? r - l + 1 : 0;
    }
};
```

时间复杂度：O(n)，其中 n 是数组的长度。每个元素最多被压入和弹出栈一次。

空间复杂度：O(n)，栈在最坏情况下可能存储所有元素的索引。
