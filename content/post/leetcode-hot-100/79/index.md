+++
date = '2026-01-16T09:57:30+08:00'
draft = false
title = '79. 单词搜索'
description = '中等 · 回溯'
categories = ['Leetcode Hot 100']
tags = ['回溯', '数组', '矩阵']
+++


## 题目

[79. 单词搜索](https://leetcode.cn/problems/word-search/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/word-search/solutions/535240/dan-ci-sou-xun-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)



## 方法一：回溯

```cpp
class Solution {
public:
    int dx[4] = {0, 1, 0, -1};
    int dy[4] = {1, 0, -1, 0};

    bool dfs(vector<vector<char>>& board, const string& word, int u, int i, int j, vector<vector<bool>>& st) {
        if (u == word.size()) {
            return true;
        }
       
        if (i < 0 || i >= board.size() || j < 0 || j >= board[0].size() || st[i][j] || board[i][j] != word[u]) {
            return false;
        }

        st[i][j] = true;

        for (int k = 0; k < 4; ++k) {
            if (dfs(board, word, u + 1, i + dx[k], j + dy[k], st)) {
                return true; 
            }
        }

        st[i][j] = false;
        return false; 
    }

    bool exist(vector<vector<char>>& board, string word) {
        if (board.empty() || board[0].empty()) return false;
        int n = board.size(), m = board[0].size();
        vector<vector<bool>> st(n, vector<bool>(m, false));

        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < m; ++j) {
                if (dfs(board, word, 0, i, j, st)) {
                    return true;
                }
            }
        }

        return false;
    }
};
```

时间复杂度：O(N * 3^L)，其中 N 是矩阵中的单元格数量，L 是单词的长度。最坏情况下，我们需要从每个单元格开始进行深度优先搜索，每次搜索有最多 3 个方向可选（因为不能回到上一个单元格）。
空间复杂度：O(L)，递归调用栈的深度最大为单词的长度 L，同时还需要一个大小为 N 的布尔矩阵来记录访问状态，但主要空间开销来自递归栈。




