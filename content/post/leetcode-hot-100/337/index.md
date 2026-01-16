+++
date = '2026-01-16T10:28:48+08:00'
draft = false
title = '337. 打家劫舍 III'
categories = ['Leetcode Hot 100']
tags = ['树', '动态规划', '深度优先搜索']
description = '中等 · 树形动态规划 + DFS'
+++


## 题目
[337. 打家劫舍 III](https://leetcode.cn/problems/house-robber-iii/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/house-robber-iii/solutions/535751/da-jia-jie-she-iii-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)



## 方法一：暴搜

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
    int robb(TreeNode* root, bool st) {
        if (root == nullptr) return 0;

        if (st) {
            return root->val + robb(root->left, false) + robb(root->right, false);
        } else {
            int left_sum = max(robb(root->left, true), robb(root->left, false));
            int right_sum = max(robb(root->right, true), robb(root->right, false));
            return left_sum + right_sum;
        }
    }

    int rob(TreeNode* root) {
        return max(robb(root, true), robb(root, false));
    }
};
```


时间复杂度：O(N^2) ，其中 N 是二叉树的节点数。对于每个节点，我们都需要递归地计算其子树的结果，最坏情况下每个节点都需要访问所有其他节点。

空间复杂度：O(H) ，其中 H 是二叉树的高度。递归调用栈的空间取决于二叉树的高度，最坏情况下二叉树退化成链表，高度为 N 。


## 方法二：树形动态规划

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
    unordered_map<TreeNode*, int> f, g;

    void dfs(TreeNode* node) {
        if (!node) 
            return;

        df(node->left);
        dfs(node->right);
        f[node] = node->val + g[node->left] + g[node->right];
        g[node] = max(f[node->left], g[node->left]) + max(f[node->right], g[node->right]);
    }

    int rob(TreeNode* root) {
        dfs(root);
        return max(f[root], g[root]);
    }
};
```

时间复杂度：O(N) ，其中 N 是二叉树的节点数。我们对每个节点访问一次。

空间复杂度：O(N) ，其中 N 是二叉树的节点数。主要为递归调用栈的空间以及哈希表的空间。


## 方法三：树形动态规划（状态压缩）

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
    pair<int, int> dfs(TreeNode* node) {
        if (!node) 
            return {0, 0};

        auto left = dfs(node->left);
        auto right = dfs(node->right);
        int rob_node = node->val + left.second + right.second;
        int not_rob_node = max(left.first, left.second) + max(right.first, right.second);
        return {rob_node, not_rob_node};
    }

    int rob(TreeNode* root) {
        auto res = dfs(root);
        return max(res.first, res.second);
    }
};
``` 

时间复杂度：O(N) ，其中 N 是二叉树的节点数。我们对每个节点访问一次。

空间复杂度：O(H) ，其中 H 是二叉树的高度。递归调用栈的空间取决于二叉树的高度，最坏情况下二叉树退化成链表，高度为 N 。


