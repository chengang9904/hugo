+++
date = '2026-09-14T06:40:11-07:00'
draft = false
title = '15. 三数之和'
tags = ['数组', '双指针']
categories = ['Leetcode Hot 100']
+++

## 题目

[15. 三数之和](https://leetcode.cn/problems/3sum/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/3sum/solutions/535726/san-shu-zhi-he-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)



```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> res;
        sort(nums.begin(), nums.end());

        int n = nums.size();
        for (int i = 0; i < n - 2; i ++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;

            int j = i + 1, k = n - 1;
            while (j < k) {
                if (nums[i] + nums[j] + nums[k] < 0) {
                    j ++;
                } else if (nums[i] + nums[j] + nums[k] > 0) {
                    k --;
                } else {
                    res.push_back({nums[i], nums[j], nums[k]});
                    while (j < k && nums[j + 1] == nums[j]) j ++;
                    while (j < k && nums[k - 1] == nums[k]) k --;
                    j ++, k --;
                }
            }
        }

        return res;
    }
};
```
