+++
date = '2026-01-05T10:39:13+08:00'
title = '32. 最长有效括号'
description = '困难 · 栈匹配 + 动态规划'
categories = ['Leetcode Hot 100']
tags = ['栈', '动态规划', '字符串']
+++

## 题目
[32. 最长有效括号](https://leetcode.cn/problems/longest-valid-parentheses/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/longest-valid-parentheses/solutions/535619/zui-chang-you-xiao-gua-hao-by-leetcode-solution-bjlu/?envType=problem-list-v2&envId=2cktkvj)


## 方法一：栈

理解墙的概念，先向栈中加入一个-1的墙，每次匹配到右括号 `(` 就弹栈一次，把左括号 `)` 给弹出去，然后判断栈是否为空，因为有个 `-1` 的墙，如果栈空了，说明是没有匹配的左括号，就向栈中放新的墙来替代-1，这个墙永远在栈底这个位置，如果有新的墙产生，会先把这个栈底墙弹出去，有点设计的过于巧妙了。

```cpp
class Solution {
public:
int longestValidParentheses(std::string s) {
        int res = 0;
        stack<int> stk; 
        stk.push(-1);  
        for (int i = 0; i < s.length(); ++i) {
            if (s[i] == '(') {
                stk.push(i); 
            } else { 
                stk.pop(); 
                if (stk.empty()) {
                    stk.push(i);
                } else {
                    res = max(res, i - stk.top());
                }
            }
        }
        return res;
    }
};
```

## 方法二：动态规划

有点过于复杂了

- s[i] = ')' 且 s[i-1] = '(' 时，dp[i] = dp[i-2] + 2
- s[i] = ')' 且 s[i-1] = ')' 时，若 s[i-dp[i-1]-1] = '('，则 dp[i] = dp[i-1] + dp[i-dp[i-1]-2] + 2

还要注意俩个边界条件要排除
```cpp
class Solution {
public:
    int longestValidParentheses(string s) {
        int maxans = 0, n = s.length();
        vector<int> dp(n, 0);
        for (int i = 1; i < n; i++) {
            if (s[i] == ')') {
                if (s[i - 1] == '(') {
                    dp[i] = (i >= 2 ? dp[i - 2] : 0) + 2;
                } else if (i - dp[i - 1] > 0 && s[i - dp[i - 1] - 1] == '(') {
                    dp[i] = dp[i - 1] + ((i - dp[i - 1]) >= 2 ? dp[i - dp[i - 1] - 2] : 0) + 2;
                }
                maxans = max(maxans, dp[i]);
            }
        }
        return maxans;
    }
};
```



