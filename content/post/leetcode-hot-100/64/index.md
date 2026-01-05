+++
date = '2026-01-05T12:39:13+08:00'
title = '64. 最小路径和'
description = '中等 · 动态规划 · 最短路径'
categories = ['Leetcode Hot 100']
tags = ['数组', '动态规划']
+++

## 题目

[64. 最小路径和](https://leetcode.cn/problems/minimum-path-sum/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/minimum-path-sum/solutions/535954/zui-xiao-lu-jing-he-by-leetcode-solution-3pkl/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：动态规划

使用二维数组 `f[i][j]` 表示从起点到位置 `(i, j)` 的最小路径和。状态转移方程为：

- `f[0][0] = grid[0][0]`
- `f[i][j] = min(f[i-1][j], f[i][j-1]) + grid[i][j]`（当 i > 0 且 j > 0 时）
- 边界情况：第一行和第一列只能从左或上累加。

```cpp
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();
        vector<vector<int>> f(m, vector<int>(n));
        for (int i = 0; i < m; i ++) 
            for(int j = 0; j < n; j ++) {
                if (i == 0 && j == 0) {
                    f[0][0] = grid[0][0];
                } else if (i == 0) {
                    f[i][j] = f[i][j - 1] + grid[i][j];
                } else if (j == 0) {
                    f[i][j] = f[i - 1][j] + grid[i][j];
                } else {
                    f[i][j] = min(f[i - 1][j], f[i][j - 1]) + grid[i][j];
                }
            }
        
        return f[m - 1][n - 1];
    }
};
```

时间复杂度：O(m _ n)，空间复杂度：O(m _ n)。

## 方法二：动态规划（空间优化）

由于每次计算只依赖上一行和当前行，可以用一维数组 `f[j]` 表示当前行的最小路径和。

- 初始化第一行时，`f[j]` 累加左边值。
- 后续行：`f[j]` 先保存上一行的值，然后更新为 `min(f[j], f[j-1]) + grid[i][j]`。

```cpp
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();
        vector<int> f(n);
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (i == 0 && j == 0) {
                    f[0] = grid[0][0];
                } else if (i == 0) {
                    f[j] = f[j - 1] + grid[i][j];
                } else if (j == 0) {
                    f[j] = f[j] + grid[i][j];  // f[j] 为上一行的 f[j]
                } else {
                    f[j] = min(f[j], f[j - 1]) + grid[i][j];  // f[j] 为上一行的 f[j]
                }
            }
        }
        return f[n - 1];
    }
};
```

时间复杂度：O(m \* n)，空间复杂度：O(n)。
