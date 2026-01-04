+++
date = '2026-01-04T10:37:02+08:00'
draft = true
title = 'Leetcode Hot 100 85. 最大矩形'
categories = ['Leetcode Hot 100']
tags = ['动态规划', '矩阵']
+++


## 题目
[85. 最大矩形](https://leetcode.cn/problems/maximal-rectangle/description/?envType=problem-list-v2&envId=2cktkvj)

![alt text](image.png)

刚开始想了一个暴力的解法，但是理解错误题目意思了，是矩形不是正方形。

```
class Solution {
public:
    bool isTrue(vector<vector<char>>& matrix, int i, int j, int size) {
        for (int x = i; x < i + size; x ++) {
            for (int y = j; y < j + size; y ++) {
                if (matrix[x][y] == '0') return false;
            }
        }
        return true;
    }

    int maximalSquare(vector<vector<char>>& matrix) {
        int max_size = min(matrix.size(), matrix[0].size());
        for (int i = max_size; i > 0; i --) {
            for (int j = 0; j + i <= matrix.size(); j ++) {
                for (int k = 0; k + i <= matrix[0].size(); k ++) {
                    // 判断是否
                    if (isTrue(matrix, j, k, i)) return i * i;
                }
            }
        }
        return 0;
    }
};
```
**时间复杂度** 

O((mn)^2) ，m 和 n 分别是矩阵的行数和列数。

实际上是原题 [221. 最大正方形](https://leetcode.cn/problems/maximal-square/)

动态规划写法

$$
dp(i,j)=min(dp(i−1,j),dp(i−1,j−1),dp(i,j−1))+1
$$
```cpp
class Solution {
public:
    int maximalSquare(vector<vector<char>>& matrix) {
        if (matrix.size() == 0 || matrix[0].size() == 0) {
            return 0;
        }   
        int maxSide = 0;
        int rows = matrix.size(), columns = matrix[0].size();
        vector<vector<int>> dp(rows, vector<int>(columns));
        for (int i = 0; i < rows; i ++) {
            for (int j = 0; j < columns; j ++) {
                if (matrix[i][j] == '1') {
                    if (i == 0 || j == 0) {
                        dp[i][j] = 1;
                    } else {
                        dp[i][j] = min(dp[i - 1][j - 1], min(dp[i - 1][j], dp[i][j - 1])) + 1;
                    }
                    maxSide = max(maxSide, dp[i][j]);
                }
            } 
        }
        return maxSide * maxSide;
    }
};
```

**回到原题**

还是动态规划，提示标签里有单调栈，先放一放

![](http://cdn.cscat.cn/markdown/image-20251224145224073.png)




