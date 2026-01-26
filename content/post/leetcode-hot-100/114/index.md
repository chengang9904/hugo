+++
date = '2026-01-26T21:17:49+08:00'
draft = false
title = '114. 二叉树展开为链表'
description = '中等 · 树'
categories = ['Leetcode Hot 100']
tags = ['二叉树', '树', '链表', '递归']
+++

## 题目

[114. 二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/solutions/535822/er-cha-shu-zhan-kai-wei-lian-biao-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)


## 方法一： 递归

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
    void flatten(TreeNode* root) {
        vector<TreeNode*> nodeList;
        preOrder(root, nodeList);
        int n = nodeList.size();

        for (int i = 1; i < n; i ++) {
            TreeNode *prev = nodeList.at(i - 1), *cur = nodeList.at(i);
            prev->left = nullptr;
            prev->right = cur;
        }
    }

    void preOrder(TreeNode* root, vector<TreeNode*>& nodeList) {
        if (root != nullptr) {
            nodeList.push_back(root);
            preOrder(root->left, nodeList);
            preOrder(root->right, nodeList);
        }
    }
};
```

时间复杂度：O(n)

空间复杂度：O(n)



## 方法二： 递归（优化空间）

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
    TreeNode* prev = nullptr;
    void flatten(TreeNode* root) {
        if (root == nullptr) return;

        flatten(root->right);
        flatten(root->left);

        root->right = prev;
        root->left = nullptr;
        prev = root;
    }
};
```


时间复杂度：O(n)

空间复杂度：O(n)（递归栈）

反前序遍历
