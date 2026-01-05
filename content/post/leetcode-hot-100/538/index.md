+++
date = '2026-01-05T09:24:31+08:00'
title = '538. 把二叉搜索树转换为累加树'
description = '中等 · 反向中序遍历'
categories = ['Leetcode Hot 100']
tags = []
+++


## 题目

[538. 把二叉搜索树转换为累加树](https://leetcode.cn/problems/convert-bst-to-greater-tree/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/convert-bst-to-greater-tree/solutions/535761/ba-er-cha-sou-suo-shu-zhuan-huan-wei-lei-jia-shu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)


## 方法一：反向中序遍历

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
    int sum;

    TreeNode* convertBST(TreeNode* root) {
        if (root != nullptr) {
            convertBST(root->right);
            sum += root->val;
            root->val = sum;
            convertBST(root->left);
        }

        return root;
    }
};
```

