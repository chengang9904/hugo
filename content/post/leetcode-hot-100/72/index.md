+++
date = '2026-01-13T11:22:35+08:00'
draft = false
title = '72. 编辑距离'
description = '困难 · 动态规划'
categories = ['Leetcode Hot 100']
tags = ['字符串', '动态规划']
+++


## 题目

[72. 编辑距离](https://leetcode.cn/problems/edit-distance/description/?envType=problem-list-v2&envId=2cktkvj)

[官方题解](https://leetcode.cn/problems/edit-distance/solutions/535708/bian-ji-ju-chi-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

[AcWing题解](https://www.acwing.com/solution/content/46098/)


## 动态规划

```cpp
class Solution {
public:
    int f[510][510];

    int minDistance(string word1, string word2) {
        // 初始化
        int n1 = word1.size(), n2 = word2.size();
        for (int i = 0; i <= n1; i ++) {
            f[i][0] = i;
        }
        for (int i = 0; i <= n2; i ++) {
            f[0][i] = i;
        }

        for (int i = 1; i <= n1; i ++) {
            for (int j = 1; j <= n2; j ++) {
                f[i][j] = min(f[i][j - 1] + 1, f[i - 1][j] + 1);
                f[i][j] = min(f[i][j], f[i - 1][j -1] + (word1[i - 1] != word2[j - 1]));
            }
        }

        return f[n1][n2];
    }
};
```

时间复杂度：O(mn)，其中 m 和 n 分别是字符串 word1 和 word2 的长度。我们需要计算并存储一个大小为 m×n 的二维数组 f。
空间复杂度：O(mn)，即为存储二维数组 f 需要的空间。

