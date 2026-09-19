+++
date = '2026-09-18T22:31:33-07:00'
draft = false
title = '5. 最长回文子串'
tags = ['字符串']
categories = ['Leetcode Hot 100']
+++


## 题目

[5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/longest-palindromic-substring/solutions/535727/k-ge-yi-zu-fan-zhuan-lian-biao-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)


```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        int n = s.size();
        vector<vector<bool>> f(n, vector<bool>(n, false));
        string res = s.substr(0, 1);

        for (int i = 0; i < n; i++) {
            f[i][i] = true;
            if (i < n - 1 && s[i] == s[i + 1]) {
                f[i][i + 1] = true;
                res = s.substr(i, 2);
            }
        }

        for (int d = 2; d < n; d++) {          // d = j - i
            for (int i = 0; i + d < n; i++) {
                int j = i + d;
                if (s[i] == s[j] && f[i + 1][j - 1]) {
                    f[i][j] = true;
                    res = s.substr(i, d + 1);  // d increases, so this is always the longest so far
                }
            }
        }
        return res;
    }
};
```
