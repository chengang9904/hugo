+++
date = '2026-01-22T10:49:54+08:00'
draft = false
title = '49. 字母异位词分组'
description = '中等 · 哈希表'
categories = ['Leetcode Hot 100']
tags = ['哈希表', '字符串', '排序']
+++

## 题目

[49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/group-anagrams/solutions/535711/zi-mu-yi-wei-ci-fen-zu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：排序

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> umap;
        for (string& str : strs) {
            string key = str;
            sort(key.begin(), key.end());
            umap[key].push_back(str);
        }
        vector<vector<string>> ans;
        for (auto it = umap.begin(); it != umap.end(); ++ it) {
            ans.push_back(it->second);
        }
        return ans;
    }
};
```

时间复杂度：O(NK log K)，其中 N 是 strs 中的字符串数量，K 是字符串的最大长度。对每个字符串进行排序的时间复杂度是 O(K log K)，总共有 N 个字符串。

空间复杂度：O(NK)，需要存储所有字符串。

## 方法二：计数

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> umap;
        for (string& str : strs) {
            vector<int> count(26, 0);
            for (char c : str) {
                count[c - 'a'] ++;
            }
            string key;
            for (int i = 0; i < 26; ++ i) {
                key += '#';
                key += to_string(count[i]);
            }
            umap[key].push_back(str);
        }
        vector<vector<string>> ans;
        for (auto it = umap.begin(); it != umap.end(); ++ it) {
            ans.push_back(it->second);
        }
        return ans;
    }
};
```
