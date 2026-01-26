+++
date = '2026-01-08T20:27:02+08:00'
draft = false
title = '437. 路径总和 III'
categories = ['Leetcode Hot 100']
tags = ['树', '深度优先搜索', '回溯']
description = '中等 · 树 · 深度优先搜索 · 回溯'
+++

## 题目

[437. 路径总和 III](https://leetcode.cn/problems/path-sum-iii/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/path-sum-iii/solutions/535606/lu-jing-zong-he-iii-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

### 方法1：自己写的

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int res;
    void dfs(TreeNode* root, bool flag, int sum, int targetSum) {
        if (!root) {
            return;
        }

        if (flag) {
            if (sum + root->val == targetSum) res ++;
            dfs(root->left, true, sum + root->val, targetSum);
            dfs(root->right, true, sum + root->val, targetSum);
        } else {
            if (root->val == targetSum) res ++;
            dfs(root->left, true, root->val, targetSum);
            dfs(root->left, false, 0, targetSum);
            dfs(root->right, true, root->val, targetSum);
            dfs(root->right, false, 0, targetSum);
        }
    }

    int pathSum(TreeNode* root, int targetSum) {
        dfs(root, false, 0, targetSum);
        return res;
    }
};
```

时间复杂度：O(n^2)，n为节点数

空间复杂度：O(h)，h为树的高度

127/130 test cases passed.

### 方法2：官方题解爆搜-比我的优雅

```cpp
class Solution {
public:
    int rootSum(TreeNode* root, long long targetSum) {
        if (!root) {
            return 0;
        }

        int ret = 0;
        if (root->val == targetSum) {
            ret++;
        }

        ret += rootSum(root->left, targetSum - root->val);
        ret += rootSum(root->right, targetSum - root->val);
        return ret;
    }

    int pathSum(TreeNode* root, int targetSum) {
        if (!root) {
            return 0;
        }

        int ret = rootSum(root, targetSum);
        ret += pathSum(root->left, targetSum);
        ret += pathSum(root->right, targetSum);
        return ret;
    }
};
```

时间复杂度：O(n^2)，n为节点数

空间复杂度：O(h)，h为树的高度

### 方法3：前缀和 + 哈希表

```cpp
class Solution {
public:
    unordered_map<long long, int> prefix;

    int dfs(TreeNode *root, long long curr, int targetSum) {
        if (!root) {
            return 0;
        }

        int ret = 0;
        curr += root->val;
        if (prefix.count(curr - targetSum)) {
            ret = prefix[curr - targetSum];
        }

        prefix[curr]++;
        ret += dfs(root->left, curr, targetSum);
        ret += dfs(root->right, curr, targetSum);
        prefix[curr]--;

        return ret;
    }

    int pathSum(TreeNode* root, int targetSum) {
        prefix[0] = 1;
        return dfs(root, 0, targetSum);
    }
};
```

时间复杂度：O(n)，n为节点数

空间复杂度：O(h)，h为树的高度
