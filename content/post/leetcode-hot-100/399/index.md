+++
date = '2026-01-12T10:55:09+08:00'
draft = false
title = '399. 除法求值 test'
description = '中等 · 图 + 并查集'
categories = ['Leetcode Hot 100']
tags = ['图', '并查集', '深度优先搜索', '广度优先搜索']
+++

## 题目
[399. 除法求值](https://leetcode.cn/problems/evaluate-division/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/evaluate-division/solutions/535988/chu-fa-qiu-zhi-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)



> 看完题目，其实我这几个标签都想到了，图，DFS，BFS，并查集


## 方法一：图 + 深度优先搜索

```cpp
class Solution {
public:
    unordered_map<string, vector<pair<string, double>>> graph;
    
    double dfs(const string& curr, const string& target, unordered_set<string>& visited, double value) {
        if (curr == target) return value;
        visited.insert(curr);
        
        for (const auto& neighbor : graph[curr]) {
            if (visited.count(neighbor.first)) continue;
            double result = dfs(neighbor.first, target, visited, value * neighbor.second);
            if (result != -1.0) return result;
        }
        
        return -1.0;
    }
    
    vector<double> calcEquation(vector<vector<string>>& equations, vector<double>& values, vector<vector<string>>& queries) {
        for (int i = 0; i < equations.size(); ++i) {
            const string& A = equations[i][0];
            const string& B = equations[i][1];
            double k = values[i];
            graph[A].emplace_back(B, k);
            graph[B].emplace_back(A, 1.0 / k);
        }
        
        vector<double> results;
        for (const auto& query : queries) {
            const string& C = query[0];
            const string& D = query[1];
            if (!graph.count(C) || !graph.count(D)) {
                results.push_back(-1.0);
                continue;
            }
            unordered_set<string> visited;
            double result = dfs(C, D, visited, 1.0);
            results.push_back(result);
        }
        
        return results;
    }
};
``` 


## 方法二：并查集

```cpp
class Solution {
public:
    int findf(vector<int>& f, vector<double>& w, int x) {
        if (f[x] != x) {
            int father = findf(f, w, f[x]);
            w[x] = w[x] * w[f[x]];
            f[x] = father;
        }
        return f[x];
    }

    void merge(vector<int>& f, vector<double>& w, int x, int y, double val) {
        int fx = findf(f, w, x);
        int fy = findf(f, w, y);
        f[fx] = fy;
        w[fx] = val * w[y] / w[x];
    }

    vector<double> calcEquation(vector<vector<string>>& equations, vector<double>& values, vector<vector<string>>& queries) {
        int nvars = 0;
        unordered_map<string, int> variables;

        int n = equations.size();
        for (int i = 0; i < n; i++) {
            if (variables.find(equations[i][0]) == variables.end()) {
                variables[equations[i][0]] = nvars++;
            }
            if (variables.find(equations[i][1]) == variables.end()) {
                variables[equations[i][1]] = nvars++;
            }
        }
        vector<int> f(nvars);
        vector<double> w(nvars, 1.0);
        for (int i = 0; i < nvars; i++) {
            f[i] = i;
        }

        for (int i = 0; i < n; i++) {
            int va = variables[equations[i][0]], vb = variables[equations[i][1]];
            merge(f, w, va, vb, values[i]);
        }
        vector<double> ret;
        for (const auto& q: queries) {
            double result = -1.0;
            if (variables.find(q[0]) != variables.end() && variables.find(q[1]) != variables.end()) {
                int ia = variables[q[0]], ib = variables[q[1]];
                int fa = findf(f, w, ia), fb = findf(f, w, ib);
                if (fa == fb) {
                    result = w[ia] / w[ib];
                }
            }
            ret.push_back(result);
        }
        return ret;
    }
};
```

> 并查集带权重太难了。。。
