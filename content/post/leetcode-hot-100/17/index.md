+++
date = '2026-03-04T16:22:21+08:00'
draft = false
title = '17. 电话号码的字母组合'
descrption = '中等 · 字符串 · 回溯算法'
categories = ['Leetcode Hot 100']
tags = ['字符串', '回溯算法']

+++

## 题目

[17. 电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/solutions/388738/dian-hua-hao-ma-de-zi-mu-zu-he-by-leetcode-solutio/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：回溯算法

```cpp

class Solution {
public:
    vector<string> letterCombinations(string digits) {
        unordered_map<char, string> phoneMap{
            {'2', "abc"},
            {'3', "def"},
            {'4', "ghi"},
            {'5', "jkl"},
            {'6', "mno"},
            {'7', "pqrs"},
            {'8', "tuv"},
            {'9', "wxyz"}
        };

        vector<string> combinations;
        if (digits.empty()) {
            return combinations;
        }

        string combination;
        dfs(combinations, phoneMap, digits, 0, combination);
        return combinations;
    }

    void dfs(vector<string>& combinations, const unordered_map<char, string>& phoneMap, const string& digits, int index, string& combination) {
        if (index == digits.size()) {
            combinations.push_back(combination);
        } else {
            char digit = digits[index];
            const string& letters = phoneMap.at(digit);
            for (const char& letter : letters) {
                combination.push_back(letter);
                dfs(combinations, phoneMap, digits, index + 1, combination);
                combination.pop_back();
            }
        }
    }
};
```
