+++
date = '2026-01-14T10:58:42+08:00'
draft = false
title = '105. 从前序与中序遍历序列构造二叉树'
description = '中等 · 递归'
categories = ['Leetcode Hot 100']
tags = ['二叉树', '递归', '前序遍历', '中序遍历']
+++

## 题目

[105. 从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/solutions/535751/cong-qian-xu-yu-zhong-xu-bian-li-xu-lie-gou-zao-er-cha-shu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

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
    TreeNode* build(vector<int>& preorder, vector<int>& inorder, int l1, int r1, int l2, int r2) {
        if (l1 > r1) return nullptr;

        int len;
        for (int i = l2; i <= r2; i ++ ) {
            if (inorder[i] == preorder[l1]) {
                len = i - l2;
            }
        }

        TreeNode* root = new TreeNode(preorder[l1]);
        root->left = build(preorder, inorder, l1 + 1, l1 + len, l2, l2 + len - 1);
        root->right = build(preorder, inorder, l1 + len + 1, r1, l2 + len + 1, r2);

        return root;
    }

    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        return build(preorder, inorder, 0, preorder.size() - 1, 0, inorder.size() - 1);
    }
};
```

时间复杂度：O(n^2)

空间复杂度：O(n)


##方法二：递归 + 哈希表

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
    unordered_map<int, int> umap;

    TreeNode* build(vector<int>& preorder, vector<int>& inorder, int l1, int r1, int l2, int r2) {
        if (l1 > r1) return nullptr;

        int len = umap[preorder[l1]] - l2;

        TreeNode* root = new TreeNode(preorder[l1]);
        root->left = build(preorder, inorder, l1 + 1, l1 + len, l2, l2 + len - 1);
        root->right = build(preorder, inorder, l1 + len + 1, r1, l2 + len + 1, r2);

        return root;
    }

    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        for (int i = 0; i < inorder.size(); i ++) {
            umap[inorder[i]] = i;
        }
        return build(preorder, inorder, 0, preorder.size() - 1, 0, inorder.size() - 1);
    }
};
```

时间复杂度：O(n)

空间复杂度：O(n)