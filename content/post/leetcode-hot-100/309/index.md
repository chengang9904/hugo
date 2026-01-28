+++
date = '2026-01-28T12:45:22+08:00'
draft = false
title = '309. 最佳买卖股票时机含冷冻期'
description = '中等 · 动态规划'
categories = ['Leetcode Hot 100']
tags = ['动态规划']
+++

## 题目

[309. 最佳买卖股票时机含冷冻期](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/solutions/535549/zui-jia-mai-mai-gu-piao-shi-ji-han-leng-dong-q-2/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：动态规划

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        if (n == 0) return 0;

        // dp[i][0]: 持有
        // dp[i][1]: 冷冻期（今天刚卖）
        // dp[i][2]: 非冷冻期空仓
        vector<vector<int>> dp(n, vector<int>(3));

        dp[0][0] = -prices[0]; // 第0天买入，收益为负
        dp[0][1] = 0;
        dp[0][2] = 0;

        for (int i = 1; i < n; i++) {
            // 今天持有：昨天就持有，或者昨天是非冷冻期今天买入
            dp[i][0] = max(dp[i-1][0], dp[i-1][2] - prices[i]);
            // 今天是冷冻期：今天卖出
            dp[i][1] = dp[i-1][0] + prices[i];
            // 今天是非冷冻期空仓：昨天是冷冻期，或者昨天就是非冷冻期空仓
            dp[i][2] = max(dp[i-1][1], dp[i-1][2]);
        }

        // 最后一天手里有股票肯定不赚钱，取不持有股票的两种情况最大值
        return max(dp[n-1][1], dp[n-1][2]);
    }
};
```

时间复杂度：O(N)，其中 N 是数组 prices 的长度。我们只需遍历一次数组。

空间复杂度：O(N)，需要使用一个大小为 N 的二维数组来存储状态。
