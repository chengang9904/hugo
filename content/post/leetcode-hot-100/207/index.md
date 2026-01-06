+++
date = '2026-01-06T09:19:39+08:00'
draft = true
title = '207. 课程表'
categories = ['Leetcode Hot 100']
tags = ['拓扑排序', 'DFS', 'BFS']
description = '中等 · 拓扑排序 · DFS/BFS'
+++

## 题目

[207. 课程表](https://leetcode.cn/problems/course-schedule/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/course-schedule/solutions/522553/ke-cheng-biao-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 复习拓扑排序
[Acwing 基础课模板](https://www.acwing.com/activity/content/code/content/8185114/)

```cpp
#include <cstring>
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 100010;

int n, m;
int h[N], e[N], ne[N], idx;
int d[N];
int q[N];

void add(int a, int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++ ;
}

bool topsort()
{
    int hh = 0, tt = -1;

    for (int i = 1; i <= n; i ++ )
        if (!d[i])
            q[ ++ tt] = i;

    while (hh <= tt)
    {
        int t = q[hh ++ ];

        for (int i = h[t]; i != -1; i = ne[i])
        {
            int j = e[i];
            if (-- d[j] == 0)
                q[ ++ tt] = j;
        }
    }

    return tt == n - 1;
}

int main()
{
    scanf("%d%d", &n, &m);

    memset(h, -1, sizeof h);

    for (int i = 0; i < m; i ++ )
    {
        int a, b;
        scanf("%d%d", &a, &b);
        add(a, b);

        d[b] ++ ;
    }

    if (!topsort()) puts("-1");
    else
    {
        for (int i = 0; i < n; i ++ ) printf("%d ", q[i]);
        puts("");
    }

    return 0;
}
```


## 方法一：拓扑排序（Kahn算法）


```cpp
class Solution {
public:
    static const int N = 2010, M = 5010;
    int e[M], ne[M], h[N], idx;
    int d[N], q[N];

    void add(int a, int b) {
        e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
    }

    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        memset(h, -1, sizeof h);

        for (int i = 0; i < prerequisites.size(); i ++) {
            int a = prerequisites[i][0], b = prerequisites[i][1];
            add(a, b);
            d[b] ++;
        }


        int hh = 0, tt = -1;
        for (int i = 0; i < numCourses; i ++) {
            if (!d[i]) {
                q[++ tt] = i;
            }
        }

        while (hh <= tt) {
            auto t = q[hh ++];

            for (int i = h[t]; ~i; i = ne[i]) {
                auto j = e[i];
                d[j] --;
                if (!d[j]) {
                    q[++ tt] = j;
                }
            }
        }

        return numCourses - 1 == tt;
    }
};
```

