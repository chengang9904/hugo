+++
date = '2026-01-05T09:05:34+08:00'
title = 'Leetcode Hot 100 338. 比特位计数'
categories = ['Leetcode Hot 100']
tags = ['']
+++

## 题目

[338. 比特位计数](https://leetcode.cn/problems/counting-bits/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/counting-bits/solutions/535906/bi-te-wei-ji-shu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：逐位计数

```cpp
class Solution {
public:
    int countB(int i) {
        int cnt = 0;
        while (i) {
            cnt += i & 1;
            i  = i >> 1;
        }
        return cnt;
    }

    vector<int> countBits(int n) {
        vector<int> res;
        for (int i = 0; i <= n; i ++) {
            res.push_back(countB(i));
        }
        return res;
    }

};
```

## 方法二：动态规划——最低有效位

$$
f[i] = f[i >> 1] + (i \& 1)
$$

```cpp
class Solution {
public:
    vector<int> countBits(int n) {
        vector<int> res;
        res.push_back(0);
        for (int i = 1; i <= n; i ++) {
            res.push_back(res[i >> 1] + (i & 1));
        }
        return res;
    }
};
```


## 方法三：动态规划——最高有效位

```cpp
class Solution {
public:
    vector<int> countBits(int n) {
        vector<int> bits(n + 1);
        int highBit = 0;
        for (int i = 1; i <= n; i++) {
            if ((i & (i - 1)) == 0) {
                highBit = i;
            }
            bits[i] = bits[i - highBit] + 1;
        }
        return bits;
    }
};
```
