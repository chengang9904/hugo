+++
date = '2026-03-19T22:16:29+08:00'
draft = false
title = '394. 字符串解码'
categories = ['Leetcode Hot 100']
tags = ['栈', '递归']
+++

## 题目

[394. 字符串解码](https://leetcode.cn/problems/decode-string/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/decode-string/solutions/522553/zi-fu-chuan-jie-ma-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 单栈模拟

这道题非常好，非常考验一个人的代码功底，适合多次练习

**思路**

- 遇到数字，入栈
- 遇到字母或左括号，入栈
- 遇到右括号，出栈直到遇到左括号，得到一个子串
- 再出栈得到重复次数，构造一个新的字符串入栈

```cpp
class Solution {
public:
    string getDigits(string& s, size_t& ptr) {
        string res = "";
        while (isdigit(s[ptr])) {
            res.push_back(s[ptr++]);
        }
        return res;
    }

    string getString(vector<string>& sub) {
        string res = "";
        for (const auto& s : sub) {
            res += s;
        }
        return res;
    }

    string decodeString(string s) {
        vector<string> stk;
        size_t ptr = 0;

        while (ptr < s.size()) {
            auto cur = s[ptr];
            if (isdigit(cur)) {
                stk.push_back(getDigits(s, ptr));
            } else if (isalpha(s[ptr]) || s[ptr] == '[') {
                stk.push_back(string(1, s[ptr++]));
            } else {
                ++ptr;
                vector<string> sub;
                while (stk.back() != "[") {
                    sub.push_back(stk.back());
                    stk.pop_back();
                }
                reverse(sub.begin(), sub.end());
                stk.pop_back();
                int repTime = stoi(stk.back());
                stk.pop_back();
                string t, o = getString(sub);
                while (repTime --) t += o;
                stk.push_back(t);
            }
        }

        return getString(stk);
    }
};
```
