+++
date = '2026-01-28T10:15:36+08:00'
draft = false
title = '494. 目标和'
description = '中等 · 动态规划 · 回溯算法'
categories = ['Leetcode Hot 100']
tags = ['动态规划', '回溯算法']
+++

## 题目

[494. 目标和](https://leetcode.cn/problems/target-sum/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/target-sum/solutions/535553/mu-biao-he-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：动态规划

```cpp
class Solution {
public:

    int findTargetSumWays(vector<int>& nums, int target) {
        int n = nums.size();
        vector<vector<int>> f(n + 1, vector<int>(2001, 0));
        f[0][1000] = 1;
        for (int i = 1; i <= n; i ++) {
            for (int j = 0; j < 2001; j ++) {
                if (j - nums[i - 1] > 0 && j - nums[i - 1] < 2001)
                    f[i][j] += f[i - 1][j - nums[i - 1]];
                if (j + nums[i - 1] > 0 && j + nums[i - 1] < 2001)
                    f[i][j] += f[i - 1][j + nums[i - 1]];
            }
        }
        return f[n][1000 + target];
    }
};
```

时间复杂度：O(N \* M)，其中 N 是数组 nums 的长度，M 是可能的和的范围（在本题中为 2001）。

空间复杂度：O(N _ M)，需要使用一个大小为 N _ M 的二维数组来存储状态。

## 方法二：回溯算法

```cpp
class Solution {
public:
    int cnt;

    void dfs(vector<int>& nums, int target, int u) {
        int n = nums.size();
        if (u == n) {
            if (target == 0) {
                cnt ++;
            }
            return;
        }

        dfs(nums, target - nums[u], u + 1);
        dfs(nums, target + nums[u], u + 1);
    }

    int findTargetSumWays(vector<int>& nums, int target) {
        dfs(nums, target, 0);
        return cnt;
    }
};
```

时间复杂度：O(2^N)，其中 N 是数组 nums 的长度。每个元素有两种选择（加或减），因此总的选择数为 2^N。

空间复杂度：O(N)，递归调用栈的深度为 N。

## 方法三：回溯算法 + 记忆化搜索

```cpp
class Solution {
public:
    int dfs(vector<int>& nums, int target, int u, vector<vector<int>>& memo) {
        int n = nums.size();
        if (u == n) {
            return target == 0 ? 1 : 0;
        }
        if (memo[u][target + 1000] != -1) {
            return memo[u][target + 1000];
        }

        int add = dfs(nums, target - nums[u], u + 1, memo);
        int subtract = dfs(nums, target + nums[u], u + 1, memo);
        memo[u][target + 1000] = add + subtract;
        return memo[u][target + 1000];
    }

    int findTargetSumWays(vector<int>& nums, int target) {
        int n = nums.size();
        vector<vector<int>> memo(n, vector<int>(2001, -1));
        return dfs(nums, target, 0, memo);
    }
};
```

## 动态规划官方题解

```cpp
class Solution {
    public int findTargetSumWays(int[] nums, int target) {
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }
        int diff = sum - target;
        if (diff < 0 || diff % 2 != 0) {
            return 0;
        }
        int neg = diff / 2;
        int[] dp = new int[neg + 1];
        dp[0] = 1;
        for (int num : nums) {
            for (int j = neg; j >= num; j--) {
                dp[j] += dp[j - num];
            }
        }
        return dp[neg];
    }
}
```

思路

按照题意，其实就是准备两个背包，一个背包package_a存放标记为正的元素，另一个背包package_b存放标记为负的元素。package_a - package_b = target。

设nums的元素和为sum, 可以列出方程：

```
package_a - package_b = target;
package_a + package_b = sum;
```

则 package_a = (target + sum)/2。
所以根据题意给的target和sum，我们可以求出package_a的值。

那这道题就可以转化为：给定一个大小为package_a的背包，有多少种组合方式能把背包装满？ 妥妥的0-1背包。
