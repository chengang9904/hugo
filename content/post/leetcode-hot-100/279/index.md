+++
date = '2026-01-16T10:53:29+08:00'
draft = false
title = '279. 完全平方数'
description = '中等 · 数学 + 动态规划 + 广度优先搜索'
categories = ['Leetcode Hot 100']
tags = ['数学', '动态规划', '广度优先搜索']
+++


## 题目
[279. 完全平方数](https://leetcode.cn/problems/perfect-squares/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/perfect-squares/solutions/535882/wan-quan-ping-fang-shu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)



## 方法一：动态规划 


```cpp
class Solution {
public:
    int numSquares(int n) {
        vector<int> f(n + 1, n);
        for (int i = 1; i <= n / i; i ++) {
            f[i * i] = 1;
        }

        for (int i = 1; i <= n; i ++) {
            if (f[i] == 1) continue;
            for (int j = 1; j < i; j ++) {
                f[i] = min(f[i], f[j] + f[i - j]);
            }
        }

        return f[n];
    }
};
```

时间复杂度：O(n√n)  
空间复杂度：O(n)



## 方法二：广度优先搜索

```cpp
class Solution {
public:
    int numSquares(int n) {
        vector<int> squares;
        for (int i = 1; i * i <= n; i ++) {
            squares.push_back(i * i);
        }
        queue<int> q;
        q.push(n);
        int step = 0;
        while (!q.empty()) {
            step ++;
            int size = q.size();
            for (int i = 0; i < size; i ++) {
                int remainder = q.front();
                q.pop();    
                for (int square : squares) {
                    int next = remainder - square;
                    if (next < 0) {
                        break;
                    } else if (next == 0) {
                        return step;
                    } else {
                        q.push(next);
                    }
                }
            }
        }
        return step;
    }
};
```

时间复杂度：O(n√n)

空间复杂度：O(n)


## 方法三：广搜优化

```cpp
class Solution {
public:
    int numSquares(int n) {
        vector<int> squares;
        for (int i = 1; i * i <= n; i++) {
            squares.push_back(i * i);
        }
        
        queue<int> q;
        unordered_set<int> visited; // 记录已访问状态
        q.push(n);
        visited.insert(n);
        int step = 0;
        
        while (!q.empty()) {
            step++;
            int size = q.size();
            for (int i = 0; i < size; i++) {
                int remainder = q.front();
                q.pop();
                
                for (int square : squares) {
                    int next = remainder - square;
                    if (next < 0) break;
                    if (next == 0) return step;
                    if (visited.find(next) == visited.end()) {
                        visited.insert(next);
                        q.push(next);
                    }
                }
            }
        }
        return step;
    }
};
```

时间复杂度：O(n√n)

空间复杂度：O(n)

优化了重复状态的访问，减少了不必要的计算。

## 方法四：数学方法

```cpp
class Solution {
public:
    int numSquares(int n) {
        while (n % 4 == 0) {
            n /= 4;
        }
        if (n % 8 == 7) {
            return 4;       
        }
        for (int i = 0; i * i <= n; i++) {
            int j = sqrt(n - i * i);
            if (i * i + j * j == n) {
                return (i == 0 || j == 0) ? 1 : 2;
            }
        }
        return 3;
    }
};
```

时间复杂度：O(√n)

空间复杂度：O(1)

基于数论中的四平方和定理，任何正整数都可以表示为四个整数的平方和。通过排除法，可以确定最少需要多少个完全平方数来表示给定的整数 n。