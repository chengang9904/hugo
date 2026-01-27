+++
date = '2026-01-27T11:16:46+08:00'
draft = false
title = '406. 根据身高重建队列'
description = '中等 · 贪心算法 · 排序'
categories = ['Leetcode Hot 100']
tags = ['贪心算法', '排序']
+++

## 题目

[406. 根据身高重建队列](https://leetcode.cn/problems/queue-reconstruction-by-height/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/queue-reconstruction-by-height/solutions/535564/gen-ju-shen-gao-zhong-jian-dui-lie-by-leetcod/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：贪心算法 + 排序

```cpp
class Solution {
public:
    vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
        sort(people.begin(), people.end(), [](const vector<int>& u, vector<int>& v) {
            return u[0] < v[0] || (u[0] == v[0] && u[1] > v[1]);
        });
        int n = people.size();
        vector<vector<int>> res(n);
        for (const vector<int>& person: people) {
            int spaces = person[1] + 1;
            for (int i = 0; i < n; i ++) {
                if (res[i].empty()) {
                    --spaces;
                    if (!spaces) {
                        res[i] = person;
                        break;
                    }
                }
            }
        }
        return res;
    }
};
```

时间复杂度：O(N^2)，其中 N 是数组 people 的长度。排序的时间复杂度为 O(N log N)，插入每个人的时间复杂度为 O(N)。

空间复杂度：O(N)，需要使用一个大小为 N 的数组来存储结果。
