+++
date = '2026-03-19T22:20:02+08:00'
draft = false
title = '312. 戳气球'
categories = ['Leetcode Hot 100']
tags = ['动态规划']
+++

## 题目

[312. 戳气球](https://leetcode.cn/problems/burst-balloons/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/burst-balloons/solutions/522553/chu-qi-qiu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

太南了，直接上代码，直接背答案算了，只要记住状态方程

```cpp
class Solution {
public:
    int maxCoins(vector<int>& nums) {
        int n = nums.size();
        vector<vector<int>> f(n + 2, vector<int>(n + 2));
        vector<int> val(n + 2);
        val[0] = 1, val[n + 1] = 1;
        for (int i = 1; i <= n; i ++) {
            val[i] = nums[i - 1];
        }
        for (int i = n - 1; i >= 0; i --) {
            for (int j = i + 2; j <= n + 1; j ++) {
                for (int k = i + 1; k < j; k ++) {
                    int sum = val[i] * val[k] * val[j];
                    sum += f[i][k] + f[k][j];
                    f[i][j] = max(f[i][j], sum);
                }
            }
        }

        return f[0][n + 1];
    }
};
```
