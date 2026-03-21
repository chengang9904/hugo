+++
date = '2026-03-21T15:55:48+08:00'
draft = false
title = '76. 最小覆盖子串'
categories = ['Leetcode Hot 100']
tags = ['滑动窗口']
description = '困难 · 滑动窗口'
+++

## 题目

[76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/minimum-window-substring/solutions/522557/zui-xiao-fu-gai-zi-chuan-by-leetcode/?envType=problem-list-v2&envId=2cktkvj)

## 滑动窗口

抄了一遍答案

```cpp
class Solution {
public:
    unordered_map <char, int> ori, cnt;

    bool check() {
        for (const auto &p: ori) {
            if (cnt[p.first] < p.second) {
                return false;
            }
        }

        return true;
    }

    string minWindow(string s, string t) {
        for (const auto &c: t) {
            ++ ori[c];
        }

        int l = 0, r = -1;
        int len = INT_MAX, ansL = -1, ansR = -1;

        while (r < int(s.size())) {
            if (ori.find(s[++r]) != ori.end()) {
                ++ cnt[s[r]];
            }

            while (check() && l <= r) {
                if (r - l + 1 < len) {
                    len = r - l + 1;
                    ansL = l;
                }
                if (ori.find(s[l]) != ori.end()) {
                    --cnt[s[l]];
                }
                ++l;
            }
        }

        return ansL == -1 ? string() : s.substr(ansL, len);
    }
};
```
