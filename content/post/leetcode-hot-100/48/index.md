+++
date = '2026-01-07T11:03:59+08:00'
draft = false
title = '48. 旋转图像'
category = ['Leetcode Hot 100']
tags = ['数组', '矩阵操作']
description = '中等 · 矩阵旋转'
+++

## 题目
[48. 旋转图像](https://leetcode.cn/problems/rotate-image/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/rotate-image/solutions/535710/xuan-zhuan-tu-xiang-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：转置 + 翻转
矩阵的转置是将矩阵的行列互换，即将元素 `matrix[i][j]` 变为 `matrix[j][i]`。翻转是将矩阵的每一行进行反转操作。
```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        // 水平翻转
        for (int i = 0; i < n / 2; ++i) {
            for (int j = 0; j < n; ++j) {
                swap(matrix[i][j], matrix[n - i - 1][j]);
            }
        }
        // 主对角线翻转
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < i; ++j) {
                swap(matrix[i][j], matrix[j][i]);
            }
        }
    }
};
```


## 方法二：四个一组交换
将矩阵分成四个一组进行交换，每次交换四个元素的位置，从外层向内层进行，直到处理完所有的层。


```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        for (int i = 0; i < n / 2; ++i) {
            for (int j = 0; j < (n + 1) / 2; ++j) {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[n - j - 1][i];
                matrix[n - j - 1][i] = matrix[n - i - 1][n - j - 1];
                matrix[n - i - 1][n - j - 1] = matrix[j][n - i - 1];
                matrix[j][n - i - 1] = temp;
            }
        }
    }
};

```
