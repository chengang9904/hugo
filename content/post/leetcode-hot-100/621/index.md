+++
date = '2026-03-20T08:16:45+08:00'
draft = false
title = '621. 任务调度器'
categories = ['Leetcode Hot 100']
tags = ['贪心', '优先队列']
description = '中等 · 贪心 · 优先队列'
title = '621'
+++

## 题目

[621. 任务调度器](https://leetcode.cn/problems/task-scheduler/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/task-scheduler/solutions/522553/ren-wu-diao-du-qi-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 模拟

```cpp
class Solution {
public:
    int leastInterval(vector<char>& tasks, int n) {
        unordered_map<char, int> freq;
        for (const char& c : tasks) {
            ++ freq[c];
        }

        int m = freq.size();
        vector<int> nextValid, rest;
        for (auto [_, v]: freq) {
            nextValid.push_back(1);
            rest.push_back(v);
        }

        int time = 0;
        for (int i = 0; i < tasks.size(); ++ i) {
            ++ time;
            int minNextValid = INT_MAX;
            for (int j = 0; j < m; j ++) {
                if (rest[j]) {
                    minNextValid = min(minNextValid, nextValid[j]);
                }
            }
            time = max(time, minNextValid);
            int best = -1;
            for (int j = 0; j < m; j ++) {
                if (rest[j]  && nextValid[j] <= time) {
                    if (best == -1 | rest[j] > rest[best]) {
                        best = j;
                    }
                }
            }
            nextValid[best] = time + n + 1;
            -- rest[best];
        }

        return time;
    }
};
```

思维有一点点复杂，锻炼代码能力
