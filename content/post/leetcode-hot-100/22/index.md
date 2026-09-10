+++
date = '2026-09-10T08:10:19-07:00'
draft = false
title = '22. 括号生成'
tags = ['字符串', '回溯']
categories = ['Leetcode Hot 100']
+++


## 题目

[22. 括号生成](https://leetcode.cn/problems/generate-parentheses/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/generate-parentheses/solutions/535717/gua-hao-sheng-cheng-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

```cpp
class Solution {
public:
    void dfs(string s, int left, int u, int n, vector<string>& res) {
        if (u == n && left == 0) {
            res.push_back(s);
            return;
        } else if (u == n) {
            return;
        }

        dfs(s + "(", left + 1, u + 1, n, res);

        if (left > 0) {
            dfs(s + ")", left - 1, u + 1, n, res);
        }
    }

    vector<string> generateParenthesis(int n) {
        vector<string> res;
        dfs("", 0, 0, 2 * n, res);
        return res;
    }
};
```
