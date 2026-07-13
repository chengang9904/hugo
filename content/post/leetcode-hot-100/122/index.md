+++
date = '2026-07-12T00:00:00+08:00'
draft = false
title = '122. 买卖股票的最佳时机 II'
description = '中等 · 动态规划'
categories = ['Leetcode Hot 100']
tags = ['动态规划']
+++

## 题目

[122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/solutions/535553/mai-mai-gu-piao-de-zui-jia-shi-ji-ii-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：动态规划

这题的核心是把每天的状态拆成两类：

- `0`：当前空仓，手里没有股票
- `1`：当前持股，手里有股票

`f[i][j]` 表示第 `i` 天结束后，处于状态 `j` 时的最大利润。

状态转移很直接：

- 今天空仓，要么昨天就空仓，要么昨天持股，今天把股票卖掉
- 今天持股，要么昨天就持股，要么昨天空仓，今天买入股票

对应到转移方程就是：

- `f[i][0] = max(f[i - 1][0], f[i - 1][1] + prices[i])`
- `f[i][1] = max(f[i - 1][1], f[i - 1][0] - prices[i])`

第 `0` 天要单独初始化：

- 空仓收益为 `0`
- 持股收益为 `-prices[0]`

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        if (n == 0) return 0;

        // f[i][0] 表示第 i 天结束后空仓的最大利润
        // f[i][1] 表示第 i 天结束后持股的最大利润
        vector<vector<int>> f(n, vector<int>(2));

        f[0][0] = 0;
        f[0][1] = -prices[0];

        for (int i = 1; i < n; i++) {
            f[i][0] = max(f[i - 1][0], f[i - 1][1] + prices[i]);
            f[i][1] = max(f[i - 1][1], f[i - 1][0] - prices[i]);
        }

        return f[n - 1][0];
    }
};
```

时间复杂度：O(N)，其中 N 是数组 `prices` 的长度。只需要遍历一遍数组。

空间复杂度：O(N)，需要一个二维数组记录每一天的两种状态。

如果想继续优化，也可以把二维数组压缩成两个变量，只保留上一天的状态。