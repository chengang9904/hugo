+++
date = '2026-09-08T04:54:41-07:00'
draft = true
title = '306. 累加数'
description = '中等 · 回溯'
categories = ['Leetcode Hot 100']
tags = ['回溯', '字符串']
+++

## 题目

[306. 累加数](https://leetcode.cn/problems/additive-number/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/additive-number/solutions/522554/lei-jia-shu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：回溯

一开始想过爆搜或者 dp，但爆搜复杂度太高，dp 也不好设计状态。这道题的递归性很强，本质是回溯：先枚举前两个数的切分点，之后每一步都用前两个数之和去匹配下一段子串，一旦匹配失败就回退，换一种切法。

回溯过程中要注意两点：

- 不能有前导零，除非这个数本身就是 `0`。
- 每一步都用前两个数之和去匹配下一段子串，匹配不上就直接返回 `false`，交给上一层去尝试别的切法。

```cpp
class Solution {
public:
    int getNum(string& num, int i, int j) {
        string t = num.substr(i, j - i);
        return stoi(t);
    }

    bool isAddNum(string& num, int i, int j, int k) {
        // 算第一个数和第二个数
        int a, b;
        a = getNum(num, i, j);
        b = getNum(num, j, k);
        int c = a + b;
        // 相加后，依次与第三个数比对，如果对就往下找，不对，直接返回 false
        int len = to_string(c).size();
        if (k + len > num.size()) return false;
        string c_s = num.substr(k, len);
        if (stoi(c_s) == c) return isAddNum(num, j, k, k + len);
        return false;
    }

    bool isAdditiveNumber(string num) {
        int n = num.size();
        // 枚举第二个数字
        for (int i = 1; i <= n - 2; i++) {
            // 判断第一个数字是否有前导零
            if (num[0] == '0' && i > 1) return false;

            // j：第三个数字
            for (int j = i + 1; j <= n - 1; j++) {
                // 判断第二个数字是否有前导零
                if (num[i] == '0' && j - i > 1) break;

                if (isAddNum(num, 0, i, j))
                    return true;
            }
        }

        return false;
    }
};
```

> 这里有几个错误：
> 1. stoi 可能会溢出
> 2. 漏加终止条件

这里为了简单，直接用 `stoi` 计算两数之和，没有处理大数溢出——题目里数字最长可以到 35 位，超出 `int` 甚至 `long long` 的表示范围时会出问题。实际提交前可以换成字符串加法来保证正确性。

时间复杂度：O(n³)，枚举前两个数的切分点各需要 O(n)，每次验证链条最长也是 O(n)。

空间复杂度：O(n)，递归深度和子串拷贝都跟数字长度成正比。

有个问题就是，`stoi` 可能会溢出，题目里数字最长可以到 35 位，超出 `int` 甚至 `long long` 的表示范围时会出问题。实际提交前可以换成字符串加法来保证正确性。

官方题解给出的代码是用字符串加法来处理大数的，下面是官方题解的代码，valid 用 while 循环：

```cpp
class Solution {
public:
    bool isAdditiveNumber(string num) {
        int n = num.size();
        for (int secondStart = 1; secondStart < n - 1; ++secondStart) {
            if (num[0] == '0' && secondStart != 1) {
                break;
            }
            for (int secondEnd = secondStart; secondEnd < n - 1; ++secondEnd) {
                if (num[secondStart] == '0' && secondStart != secondEnd) {
                    break;
                }
                if (valid(secondStart, secondEnd, num)) {
                    return true;
                }
            }
        }
        return false;
    }

    bool valid(int secondStart, int secondEnd, string num) {
        int n = num.size();
        int firstStart = 0, firstEnd = secondStart - 1;
        while (secondEnd <= n - 1) {
            string third = stringAdd(num, firstStart, firstEnd, secondStart, secondEnd);
            int thirdStart = secondEnd + 1;
            int thirdEnd = secondEnd + third.size();
            if (thirdEnd >= n || !(num.substr(thirdStart, thirdEnd - thirdStart + 1) == third)) {
                break;
            }
            if (thirdEnd == n - 1) {
                return true;
            }
            firstStart = secondStart;
            firstEnd = secondEnd;
            secondStart = thirdStart;
            secondEnd = thirdEnd;
        }
        return false;
    }

    string stringAdd(string s, int firstStart, int firstEnd, int secondStart, int secondEnd) {
        string third;
        int carry = 0, cur = 0;
        while (firstEnd >= firstStart || secondEnd >= secondStart || carry != 0) {
            cur = carry;
            if (firstEnd >= firstStart) {
                cur += s[firstEnd] - '0';
                --firstEnd;
            }
            if (secondEnd >= secondStart) {
                cur += s[secondEnd] - '0';
                --secondEnd;
            }
            carry = cur / 10;
            cur %= 10;
            third.push_back(cur + '0');
        }
        reverse(third.begin(), third.end());
        return third;
    }
};
```