+++
date = '2026-09-08T06:18:24-07:00'
draft = false
title = '862'
+++

## 题目

[862. 和至少为 K 的最短子数组](https://leetcode.cn/problems/shortest-subarray-with-sum-at-least-k/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/shortest-subarray-with-sum-at-least-k/solutions/535906/hui-wen-zi-chuan-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

看完题目，首先想到是前缀和，感觉双重循环也比较好写，数据范围是长度是 10^5，时间复杂度 O(N^2) 会超时，先写了再说，之前前缀和都是用数组写的，这次使用 vector 写一下

```cpp
class Solution {
public:
    int shortestSubarray(vector<int>& nums, int k) {
        // 前缀和 + 贪心试试
        // 先准备前缀和数组
        vector<int> a;
        a.push_back(0);
        for (int i = 0; i < nums.size(); i ++) {
            a.push_back(nums[i] + a[i]);
        }
        // 然后遍历
        for (int i = 1; i <= nums.size(); i ++) {
            for (int j = 0; j + i <= nums.size(); j ++) {
                int t = a[j + i] - a[j];
                if (t >= k) return i;
            } 
        }
        
        return -1;
    }
};
```

73/95 不出意外

改进，双端的单调队列

```cpp
class Solution {
public:
    int shortestSubarray(vector<int>& nums, int k) {
        int n = nums.size();
        vector<long> a(n + 1);
        for (int i = 0; i < n; i ++) {
            a[i + 1] = a[i] + nums[i];
        }

        int res = n + 1;
        deque<int> q;
        for (int i = 0; i <= n; i ++) {
            // 维护单调队列，枚举当前第 i 个前缀和
            long cur = a[i];

            // 如果当前前缀和与队首前缀和的差值大于等于 k，则更新结果并弹出队首元素
            while (!q.empty() && cur - a[q.front()] >= k) {
                res = min(res, i - q.front());
                q.pop_front();
            }

            // 维护单调队列的单调性，确保队列中的前缀和是递增的
            while (!q.empty() && a[q.back()] >= cur) {
                q.pop_back();
            }
            q.push_back(i);
        }
        return res < n + 1 ? res : -1;
    }
};
```

