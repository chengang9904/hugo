+++
date = '2026-01-08T13:04:17+08:00'
draft = false
title = '139. 单词拆分'
tags = ['动态规划', '字符串']
categories = ['Leetcode Hot 100']
description = '中等 · 字符串 · 动态规划'
+++


## 题目 
[139. 单词拆分](https://leetcode.cn/problems/word-break/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/word-break/solutions/535599/dan-ci-chai-fen-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)


### 方法1：暴搜

```cpp
class Solution {
public:
    bool dfs(string s, vector<string>& wordDict, int u) {
        if (u == s.size()) {
            return true;
        }

        for (int i = 0; i < wordDict.size(); i ++) {
            // 判断是否满足条件
            bool flag = true;
            for (int j = 0; j < wordDict[i].size(); j ++) {
                if (u + j < s.size()) {
                    if (s[u + j] != wordDict[i][j]) {
                        cout << wordDict[i] << endl;;
                        flag = false;
                        break;
                    }
                }
            }
            if (flag) {
                if (dfs(s, wordDict, u + wordDict[i].size())) return true;;
            }
        }

        return false;
    }    

    bool wordBreak(string s, vector<string>& wordDict) {
        return dfs(s, wordDict, 0);
    }
};
```

时间复杂度：O(m^n)，m为字典中单词的平均长度，n为字符串s的长度
空间复杂度：O(n)


### 方法2：动态规划

```cpp
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        unordered_set<string> wordset;
        for (auto t : wordDict) {
            wordset.insert(t);
        }

        vector<bool> dp(s.size() + 1);
        dp[0] = true;
        for (int i = 1; i <= s.size(); i ++) {
            for (int j = 0; j < i; j ++) {
                if (dp[j] && wordset.find(s.substr(j, i - j)) != wordset.end()) {
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.size()];
    }
};
```

时间复杂度：O(n^2)
空间复杂度：O(n)