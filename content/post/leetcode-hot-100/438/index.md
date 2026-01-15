+++
date = '2026-01-15T11:07:10+08:00'
draft = false
title = '438. 找到字符串中所有字母异位词'
description = '中等 · 滑动窗口'
categories = ['Leetcode Hot 100']
tags = ['滑动窗口', '字符串', '哈希表']
+++



## 题目

[438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/find-all-anagrams-in-a-string/solutions/535532/zhao-dao-zi-fu-chuan-zhong-suo-you-zi-mu-yi-wei-ci-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)


## 方法一：滑动窗口

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        vector<int> countP(26, 0), countS(26, 0);
        vector<int> result;
        int lenP = p.length(), lenS = s.length();

        for (char c : p) {
            countP[c - 'a']++;
        }

        for (int i = 0; i < lenS; i++) {
            countS[s[i] - 'a']++;

            if (i >= lenP) {
                countS[s[i - lenP] - 'a']--;
            }

            if (countS == countP) {
                result.push_back(i - lenP + 1);
            }
        }

        return result;
    }
};
```

## 方法二：滑动窗口优化

```cpp
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        vector<int> count(26, 0);
        vector<int> result;
        int lenP = p.length(), lenS = s.length();
        int left = 0, right = 0, diff = lenP;

        for (char c : p) {
            count[c - 'a']++;
        }

        while (right < lenS) {
            if (count[s[right] - 'a'] > 0) {
                diff--;
            }
            count[s[right] - 'a']--;
            right++;

            if (right - left == lenP) {
                if (diff == 0) {
                    result.push_back(left);
                }
                if (count[s[left] - 'a'] >= 0) {
                    diff++;
                }
                count[s[left] - 'a']++;
                left++;
            }
        }

        return result;
    }
};
```