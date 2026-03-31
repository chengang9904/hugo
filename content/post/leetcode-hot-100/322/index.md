+++
date = '2026-03-31T19:25:11+08:00'
draft = false
title = '322. 零钱兑换'
categories = ['算法题解', 'LeetCode 热题100']
tags = ['动态规划', '贪心算法', '回溯算法']
+++

## 题目

[322. 零钱兑换](https://leetcode.cn/problems/coin-change/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/coin-change/solutions/522557/ling-qian-dui-huan-by-leetcode/?envType=problem-list-v2&envId=2cktkvj)

## 爆搜

```cpp
class Solution {
public:
    int res = INT_MAX;

    void dfs(vector<int>& coins, int amount, int u, int cnt) {
        if (u == coins.size()) {
            if (!amount) {
                res = min(res, cnt);
                return;
            }
            return;
        }

        for (int i = 0; i * coins[u] <= amount; i ++) {
            if (cnt + i >= res) return;
            dfs(coins, amount - i * coins[u], u + 1, cnt + i);
        }
    }

    int coinChange(vector<int>& coins, int amount) {
        dfs(coins, amount, 0, 0);
        if (res == INT_MAX) return -1;
        return res;
    }
};
```

## 动态规划

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, INT_MAX);
        dp[0] = 0;
        for (int i = 1; i <= amount; i ++) {
            for (int j = 0; j < coins.size(); j ++) {
                if (i - coins[j] >= 0 && dp[i - coins[j]] != INT_MAX) {
                    dp[i] = min(dp[i], dp[i - coins[j]] + 1);
                }
            }
        }
        if (dp[amount] == INT_MAX) return -1;
        return dp[amount];
    }
};
```

## 贪心 + 回溯

```cpp
class Solution {
public:
    int res = INT_MAX;

    void dfs(vector<int>& coins, int amount, int u, int cnt) {
        if (u == coins.size()) {
            if (!amount) {
                res = min(res, cnt);
                return;
            }
            return;
        }

        for (int i = amount / coins[u]; i >= 0; i --) {
            if (cnt + i >= res) return;
            dfs(coins, amount - i * coins[u], u + 1, cnt + i);
        }
    }

    int coinChange(vector<int>& coins, int amount) {
        sort(coins.begin(), coins.end(), greater<int>());
        dfs(coins, amount, 0, 0);
        if (res == INT_MAX) return -1;
        return res;
    }
};
```
