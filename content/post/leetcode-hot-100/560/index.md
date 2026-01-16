+++
date = '2026-01-13T09:43:41+08:00'
draft = false
title = '560. 和为 K 的子数组'
description = '中等 · 哈希表'
categories = ['Leetcode Hot 100']
tags = ['数组', '哈希表', '前缀和']
+++


## 题目

[560. 和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/subarray-sum-equals-k/solutions/535743/he-wei-k-de-zi-shu-zu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)



## 枚举遍历

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int res = 0;
        for (int i = 0; i < nums.size(); i ++) {
            int sum = 0;
            for (int j = i; j < nums.size(); j ++ ) {
                sum += nums[j];
                if (sum == k) {
                    res ++;
                }
            }
        }

        return res;
    }
};
```

时间复杂度：O(n^2)

空间复杂度：O(1)

测试用例：92 / 93


## 前缀和 + 哈希表

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        unordered_map<int, int> mp;
        mp[0] = 1;
        int count = 0, pre = 0;
        for (auto& x:nums) {
            pre += x;
            if (mp.find(pre - k) != mp.end()) {
                count += mp[pre - k];
            }
            mp[pre]++;
        }
        return count;
    }
};
```


时间复杂度：O(n)

空间复杂度：O(n)

