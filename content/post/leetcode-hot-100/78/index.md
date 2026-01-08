+++
date = '2026-01-06T11:39:48+08:00'
draft = false
title = '78. 子集'
categories = ['Leetcode Hot 100']
tags = ['回溯', '位运算']
description = '中等 · 回溯 · 位运算'
+++


## 题目

[78. 子集](https://leetcode.cn/problems/subsets/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/subsets/solutions/523043/zi-ji-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)


## 方法1：回溯

刚开始写错了，写了个带长度的全排列
```cpp
class Solution {
public:
    vector<vector<int>> res;

    void dfs(vector<int> nums, int u, int length, vector<int> tmp, bool st[]) {
        if (u == length) {
            res.push_back(tmp);
        }

        for (int i = 0; i < nums.size(); i ++) {
            if (!st[i]) {
                st[i] = true;
                tmp.push_back(nums[i]);
                dfs(nums, u + 1, length, tmp, st);
                st[i] = false;
                tmp.pop_back();
            }
        }
    }

    vector<vector<int>> subsets(vector<int>& nums) {
        int n = nums.size();

        bool st[n];
        vector<int> tmp;
        for (int i = 0; i <= n; i ++) {
            memset(st, false, sizeof st);
            tmp.clear();
            dfs(nums, 0, i, tmp, st);            
        }
        return res;  
    }
};
```

正确代码
```cpp
class Solution {
public:
    vector<vector<int>> res;

    void dfs(vector<int> nums, int u, int length, vector<int> tmp) {
        if (tmp.size() == length) {
            res.push_back(tmp);
            return;
        }

        if (u == nums.size()) {
            return ;
        }

        // put
        tmp.push_back(nums[u]);
        dfs(nums, u + 1, length, tmp);
        tmp.pop_back();

        // not put
        dfs(nums, u + 1, length, tmp);
    }

    vector<vector<int>> subsets(vector<int>& nums) {
        int n = nums.size();

        bool st[n];
        vector<int> tmp;
        for (int i = 0; i <= n; i ++) {
            tmp.clear();
            dfs(nums, 0, i, tmp);
        }
        return res;  
    }
};
```

时间复杂度：O(N * 2^N)
空间复杂度：O(N)

因为我是在之前的排列代码上改的，所以其实并不需要用到length参数，可以直接在dfs里把结果加入res
```cpp
class Solution {
public:
    vector<vector<int>> res;
    vector<int> tmp;

    void dfs(vector<int> nums, int u) {
        if (u == nums.size()) {
            res.push_back(tmp);
            return;
        }

        dfs(nums, u + 1);

        tmp.push_back(nums[u]);
        dfs(nums, u + 1);
        tmp.pop_back();
    }


    vector<vector<int>> subsets(vector<int>& nums) {
        int n = nums.size();

        dfs(nums, 0);

        return res;
    }
};
```

时间复杂度：O(N * 2^N)
空间复杂度：O(N)    

## 方法2：最简洁的子集模板

```cpp
class Solution {
public:
    vector<int> tmp;
    vector<vector<int>> res;

    void dfs(vector<int> nums, int u) {
        res.push_back(tmp);

        for (int i = u; i < nums.size(); i ++) {
            tmp.push_back(nums[i]);
            dfs(nums, i + 1);
            tmp.pop_back();
        }
    }

    vector<vector<int>> subsets(vector<int>& nums) {
        dfs(nums, 0);
        return res;
    }
};
```

时间复杂度：O(N * 2^N)
空间复杂度：O(N)


## 方法3：位运算

```cpp
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        int n = nums.size();
        vector<vector<int>> res;

        for (int mask = 0; mask < (1 << n); mask ++) {
            vector<int> tmp;
            for (int i = 0; i < n; i ++) {
                if (mask & (1 << i)) {
                    tmp.push_back(nums[i]);
                }
            }
            res.push_back(tmp);
        }
        return res;
    }
};
```

时间复杂度：O(N * 2^N)
空间复杂度：O(N)

## 小结
本题有三种常见解法，回溯、位运算和迭代法（见官方题解）。





