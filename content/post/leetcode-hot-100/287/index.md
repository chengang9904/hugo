+++
date = '2026-01-12T10:16:10+08:00'
draft = false
title = '287. 寻找重复数'
description = '中等 · 快慢指针'
categories = ['Leetcode Hot 100']
tags = []
+++

## 题目

[287. 寻找重复数](https://leetcode.cn/problems/find-the-duplicate-number/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/find-the-duplicate-number/solutions/535710/xun-zhao-zhong-fu-shu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)


## 方法一：自己写的

```cpp
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        for (int i = 0; i < nums.size(); i ++) {
            if (nums[i] == i + 1) continue;
            int t = nums[i];
            nums[i] = 0;
            while (nums[t - 1] != 0) {
                cout << nums[t - 1] << endl;
                if (nums[t - 1] == t) {
                    return t;
                }
                int p = t - 1;
                t = nums[t - 1];
                nums[p] = p + 1; 
            }
            nums[t - 1] = t;
        }
        return 0;
    }
};
```

时间复杂度：O(n)
空间复杂度：O(1)


## 方法二：快慢指针

```cpp
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        int slow = nums[0];
        int fast = nums[0];

        // 找到相遇点
        do {
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while (slow != fast);

        // 找到入口点
        slow = nums[0];
        while (slow != fast) {
            slow = nums[slow];
            fast = nums[fast];
        }

        return slow;
    }
};
```

时间复杂度：O(n)
空间复杂度：O(1)