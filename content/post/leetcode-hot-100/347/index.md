+++
date = '2026-01-13T10:21:58+08:00'
draft = false
title = '347. 前 K 个高频元素'
description = '中等 · 堆 · 哈希表'
categories = ['Leetcode Hot 100']
tags = ['数组', '哈希表', '堆']
+++

## 题目

[347. 前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/top-k-frequent-elements/solutions/535725/qian-k-ge-gao-pin-yuan-su-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)


## 哈希表 + 堆

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> umap;
        for (auto t : nums) {
            umap[t] ++;
        }

        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> heap;
        for (auto t : umap) {
            if (heap.size() == k) {
                if (t.second > heap.top().first) {
                    heap.pop();
                    heap.push({t.second, t.first});
                }
            } else {
                heap.push({t.second, t.first});
            }
        }

        vector<int> res;
        while (k --) {
            auto t = heap.top();
            res.push_back(t.second);
            heap.pop();
        }
        return res;
    }
};
```

时间复杂度：O(N log k)，其中 N 是数组的长度。遍历数组的时间复杂度是 O(N)，将元素插入堆的时间复杂度是 O(log k)，因此总时间复杂度是 O(N log k)。

空间复杂度：O(N)，主要为哈希表和堆的空间开销。

> 复习一下手写堆，插入操作up，删除操作down

数组版

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 1e5 + 10;

int m, n;
int f[N], cnt;

void up(int u) {
    while (u / 2 && f[u / 2] > f[u]) {
        swap(f[u / 2], f[u]);
        u /= 2;
    }
}

void down(int u) {
    int t = u;
    if (u * 2 <= cnt && f[u * 2] < f[t]) t = u * 2;
    if (u * 2 + 1 <= cnt && f[u * 2 + 1] < f[t]) t = u * 2 + 1;
    if (u != t) {
        swap(f[u], f[t]);
        down(t);
    }
}

int main()
{
    cin >> n >> m;
    cnt = n;
    
    for (int i = 1; i <= n; i ++) cin >> f[i];
    
    for (int i = n / 2; i; i --) down(i);
    
    while (m --) {
        cout << f[1] << ' ';
        f[1] = f[cnt --];
        down(1);
    }
    
    return 0;
}
```







## 哈希表 + 桶排序

邪修

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> umap;
        for (auto t : nums) {
            umap[t] ++;
        }

        vector<vector<int>> bucket(nums.size() + 1);
        for (auto t : umap) {
            bucket[t.second].push_back(t.first);
        }

        vector<int> res;
        for (int i = bucket.size() - 1; i >= 0 && res.size() < k; i --) {
            for (auto t : bucket[i]) {
                res.push_back(t);
                if (res.size() == k) break;
            }
        }
        return res;
    }
};
```

时间复杂度：O(N)，其中 N 是数组的长度。遍历数组的时间复杂度是 O(N)，构建桶排序的时间复杂度也是 O(N)。
空间复杂度：O(N)，主要为哈希表和桶排序的空间开销。