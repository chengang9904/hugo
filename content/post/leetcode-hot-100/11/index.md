+++
date = '2026-01-15T11:12:34+08:00'
draft = false
title = '11. 盛最多水的容器'
description = '中等 · 双指针'
categories = ['Leetcode Hot 100']
tags = ['双指针', '贪心']
+++


## 题目

[11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/container-with-most-water/solutions/535301/sheng-zui-duo-shui-de-rong-qi-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)


## 方法一：暴力

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int res = 0;
        for (int i = 0; i < height.size() - 1; i ++) {
            for (int j = i + 1; j < height.size(); j ++) {
                res = max(res, (j - i) * (min(height[i], height[j])));
            }
        }

        return res;
    }
};
```

时间复杂度：O(n^2)

空间复杂度：O(1)

## 方法二：双指针

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int left = 0, right = height.size() - 1;
        int res = 0;
        while (left < right) {
            res = max(res, (right - left) * (min(height[right], height[left])));
            if (height[left] <= height[right]) {
                left ++;
            } else {
                right --;
            }
        }  
        return res; 
    }
};
``` 

时间复杂度：O(n)

空间复杂度：O(1)


