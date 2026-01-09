+++
date = '2026-01-09T09:24:57+08:00'
draft = false
title = '617. 合并二叉树'
description = '简单 · 递归'
categories = ['Leetcode Hot 100']
tags = ['树', '递归']
+++

## 题目

[617. 合并二叉树](https://leetcode.cn/problems/merge-two-binary-trees/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/merge-two-binary-trees/solutions/535772/he-bing-er-cha-shu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)



## 方法一：递归

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
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if (!root1 && !root2) return nullptr;
           
        if (!root1) return root2;
        if (!root2) return root1;

        root1->val = root1->val + root2->val;
        root1->left = mergeTrees(root1->left, root2->left);
        root1->right = mergeTrees(root1->right, root2->right);

        return root1;
    }
};
```


时间复杂度：O(min(M, N))，其中 M 和 N 分别是两棵二叉树的节点数。需要遍历的节点数取决于较小的那棵树。
空间复杂度：O(min(H1, H2))，其中 H1 和 H2 分别是两棵二叉树的高度。递归调用栈的空间取决于较小的那棵树的高度。


