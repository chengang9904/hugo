+++
date = '2026-01-05T13:17:12+08:00'
draft = false
title = '128. 最长连续序列'
description = '中等 · 排序、哈希集合、区间合并'
categories = ['Leetcode Hot 100']
tags = ['哈希表', '排序', '区间合并']
+++


## 题目

[128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/longest-consecutive-sequence/solutions/535971/zui-chang-lian-xu-xu-lie-by-leetcode-solution-3pkl/?envType=problem-list-v2&envId=2cktkvj)

这道题的核心是：给你一个无序数组，要求找到最长的连续整数序列长度。

如果直接排序，思路非常直观；如果想做到线性复杂度，就要借助哈希结构，跳过重复元素，并且只从“序列起点”开始向后扩展。


## 方法一：排序

最容易想到的做法是先排序，再扫描一遍统计连续段长度。

排序之后，连续整数会聚在一起。我们只需要维护当前连续段长度 `ans`，以及答案 `res`。

需要注意两类情况：

1. 相邻元素相等，说明是重复数字，直接跳过。
2. 相邻元素差值大于 1，说明连续段断开，重新开始统计。

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        if (nums.size() == 0) return 0;
        sort(nums.begin(), nums.end());

        int res = 1, ans = 1;
        for (int i = 0; i < nums.size(); i ++) {
            if (i == 0) continue;
            if (nums[i] == nums[i - 1]) continue;

            if (nums[i] == nums[i - 1] + 1) {
                ans += 1;
                res = max(res, ans);
                continue;
            }

            ans = 1;
        }

        return res;
    }
};
```

时间复杂度：O(n log n)

空间复杂度：O(1)


## 方法二：区间合并

这个思路比排序更接近题目的本质：我们维护若干个连续区间，并在插入新数字时尝试把它接到左边、右边，或者把左右两个区间合并起来。

可以用两个映射分别表示区间边界：

- `prepost`：起点 -> 终点
- `postpre`：终点 -> 起点

再用 `visited` 去重，避免重复数字影响区间维护。

当插入数字 `x` 时，有四种情况：

1. 左边和右边都存在区间，直接合并成一个大区间。
2. 只有左边存在区间，把 `x` 接到左边区间末尾。
3. 只有右边存在区间，把 `x` 接到右边区间开头。
4. 两边都不存在，创建一个新的单元素区间。

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        if (nums.empty()) {
            return 0;
        }

        // 起点 -> 终点
        map<int, int> prepost;

        // 终点 -> 起点
        map<int, int> postpre;

        unordered_set<int> visited;

        int res = 1;

        for (int x : nums) {
            if (visited.count(x)) {
                continue;
            }
            visited.insert(x);

            bool preflag = postpre.find(x - 1) != postpre.end();
            bool postflag = prepost.find(x + 1) != prepost.end();

            if (preflag && postflag) {
                int preIdx = postpre[x - 1];
                int postIdx = prepost[x + 1];

                postpre.erase(x - 1);
                prepost.erase(x + 1);

                prepost[preIdx] = postIdx;
                postpre[postIdx] = preIdx;

                res = max(res, postIdx - preIdx + 1);
            }
            else if (preflag) {
                int preIdx = postpre[x - 1];

                postpre.erase(x - 1);

                prepost[preIdx] = x;
                postpre[x] = preIdx;

                res = max(res, x - preIdx + 1);
            }
            else if (postflag) {
                int postIdx = prepost[x + 1];

                prepost.erase(x + 1);

                prepost[x] = postIdx;
                postpre[postIdx] = x;

                res = max(res, postIdx - x + 1);
            }
            else {
                prepost[x] = x;
                postpre[x] = x;
            }
        }

        return res;
    }
};
```

这个写法的优点是区间关系非常直观，适合自己推导和理解；缺点是实现细节比较多，代码量也更大。

时间复杂度：O(n log n)

空间复杂度：O(n)


## 方法三：哈希集合 + 从起点扩展

这是官方题解的核心思路，也是面试里最推荐的写法。

先把所有数字放进哈希集合。然后遍历集合中的每个数字，只有当当前数字的前驱 `num - 1` 不存在时，才把它当作一段连续序列的起点，向后不断扩展。

这样做的关键收益是：

1. 每个连续段只会从起点扫描一次。
2. 不是起点的数字不会重复向后扩展，避免了重复计算。

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> num_set;
        for (const int& num : nums) {
            num_set.insert(num);
        }

        int longestStreak = 0;

        for (const int& num : num_set) {
            if (!num_set.count(num - 1)) {
                int currentNum = num;
                int currentStreak = 1;

                while (num_set.count(currentNum + 1)) {
                    currentNum += 1;
                    currentStreak += 1;
                }

                longestStreak = max(longestStreak, currentStreak);
            }
        }

        return longestStreak;
    }
};
```

时间复杂度：O(n)

空间复杂度：O(n)


## 小结

这题本质上是在找“连续段”的最大长度。

- 如果想要最容易写，对数组排序后扫描即可。
- 如果想把思路做得更结构化，可以用“区间合并”。
- 如果追求最优复杂度，官方的“哈希集合 + 从起点扩展”是最标准的解法。

面试或者刷题时，建议优先掌握第三种写法；前两种可以帮助理解为什么第三种能做到线性复杂度。