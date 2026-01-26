+++
date = '2026-01-25T11:52:40+08:00'
draft = false
title = '647. 回文子串'
description = '中等 · 动态规划'
categories = ['Leetcode Hot 100']
tags = ['字符串', '动态规划']
+++

## 题目

[647. 回文子串](https://leetcode.cn/problems/palindromic-substrings/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/palindromic-substrings/solutions/535906/hui-wen-zi-chuan-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：动态规划 (自己写的)

```cpp
class Solution {
public:
    int countSubstrings(string s) {
        // 初始化
        if (s.length() < 2) {
            return 1;
        }
        int n = s.length();
        int res = 0;
        vector<vector<bool>> f(n, vector<bool>(n, false));
        for (int i = 0; i < n; i ++) {
            res ++;
            f[i][i] = true;
            if (i < n - 1 && s[i] == s[i + 1]) {
                f[i][i + 1] = true;
                res ++;
            }
        }

        for (int len = 3; len <= n; len ++) {
            for (int i = 0; i + len <= n; i ++) {
                if (f[i + 1][i + len - 2] && s[i] == s[i + len - 1]) {
                    f[i][i + len  - 1] = true;
                    res ++;
                }
            }
        }

        return res;
    }
};
```

时间复杂度：O(N^2)，其中 N 是字符串 s 的长度。需要填充一个 N x N 的二维数组。

空间复杂度：O(N^2)，需要使用一个 N x N 的二维数组来存储子问题的解。

## 方法二：动态规划 (官方题解)

```cpp
class Solution {
public:
    int countSubstrings(string s) {
        int n = s.size(), ans = 0;
        for (int i = 0; i < 2 * n - 1; ++i) {
            int l = i / 2, r = i / 2 + i % 2;
            while (l >= 0 && r < n && s[l] == s[r]) {
                --l;
                ++r;
                ++ans;
            }
        }
        return ans;
    }
};
```

时间复杂度：O(N^2)，其中 N 是字符串 s 的长度。最坏情况下，所有字符都相同，需要检查所有可能的回文子串。

空间复杂度：O(1)，只使用了常数级别的额外空间。
