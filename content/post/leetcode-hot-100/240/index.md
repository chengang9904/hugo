+++
date = '2026-01-04T14:24:40+08:00'
title = 'Leetcode Hot 100 240. 搜索二维矩阵 II'
categories = ['Leetcode Hot 100']
tags = ['二分', '矩阵']
+++

## 题目
[240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/search-a-2d-matrix-ii/solutions/535821/sou-suo-er-wei-ju-zhen-ii-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：从右上角开始搜索

```cpp
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        if (matrix.size() == 0 || matrix[0].size() == 0) return false;

        int rows = matrix.size(), columns = matrix[0].size();
        // 从右上角(0, columns)->(rows, 0)
        // 如果小于就target就下移，大于左移
        int x = 0, y = columns - 1;
        while (x < rows && y >= 0) {
            if (matrix[x][y] < target) {
                x ++;
            } else if (matrix[x][y] > target) {
                y --;
            } else {
                return true;
            }
        }
        return false;
    }
};
```

**时间复杂度**

O(m + n)，m 和 n 分别是矩阵的行数和列数。

## 方法二：爆搜

```cpp
class Solution {
public:
    bool dfs(vector<vector<int>>& matrix, int target, int x, int y) {
        if (x >= matrix.size() || y >= matrix[0].size())  {
            return false;
        }
        
        if (matrix[x][y] == target) {
            return true;
        } else if (matrix[x][y] < target) {
            return dfs(matrix, target, x + 1, y) || dfs(matrix, target, x, y +1);
        } else {
            return false;
        }
    }

    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        return dfs(matrix, target, 0, 0);
    }
};
```

**时间复杂度**

O(2^(m+n))，m 和 n 分别是矩阵的行数和列数。

117 / 130 个通过的测试用例


