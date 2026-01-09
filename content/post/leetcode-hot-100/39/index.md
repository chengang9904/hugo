+++
date = '2026-01-09T09:54:04+08:00'
draft = false
title = '39. 组合总和'
categories = ['Leetcode Hot 100']
tags = ['回溯', '组合']
description = '中等 · 回溯'
+++

## 题目

[39. 组合总和](https://leetcode.cn/problems/combination-sum/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/combination-sum/solutions/535708/zu-he-zong-he-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)



## 方法一：回溯

```cpp
class Solution {
public:
    vector<vector<int>> res;

    void dfs(vector<int>& candidates, int target, int u, vector<int> tmp) {
        if (target == 0) {
            res.push_back(tmp);
            return;
        }

        if (target < 0) return;

        if (u == candidates.size()) return;

        // fang
        tmp.push_back(candidates[u]);
        dfs(candidates, target - candidates[u], u, tmp);
        tmp.pop_back();

        dfs(candidates, target, u + 1, tmp);
    }

    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        dfs(candidates, target, 0, vector<int>());
        return res;
    }
};
```


时间复杂度：O(N^(T/M + 1))，其中 N 是候选数组的长度，T 是目标值，M 是候选数组中的最小值。最坏情况下，递归树的高度为 T/M，每一层有 N 个分支。
空间复杂度：O(T/M)，递归调用栈的空间取决于递归树的高度，即 T/M。


## 优化

vector<int> tmp 改为类成员变量，避免每次递归时传递和复制 tmp，从而节省空间和时间开销。

```cpp
class Solution {
public:
    vector<vector<int>> res;
    vector<int> tmp;
    void dfs(vector<int>& candidates, int target, int u) {
        if (target == 0) {
            res.push_back(tmp);
            return;
        }

        if (u == candidates.size()) return;

        dfs(candidates, target, u + 1);

        // fang
        if (target - candidates[u] >= 0) {
            tmp.push_back(candidates[u]);
            dfs(candidates, target - candidates[u], u);
            tmp.pop_back();
        }
    }

    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        dfs(candidates, target, 0);
        return res;
    }
};
```

