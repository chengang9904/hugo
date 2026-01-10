+++
date = '2026-01-10T13:26:53+08:00'
draft = false
title = '98. 验证二叉搜索树'
tags = ['树', '递归', '深度优先搜索', '二叉搜索树']
categories = ['Leetcode Hot 100']
description = '中等 · 树 · 递归 · 深度优先搜索 · 二叉搜索树'
+++


## 题目 
[98. 验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/validate-binary-search-tree/solutions/535450/yan-zheng-er-cha-sou-suo-shu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)


### 方法1: 二叉树中序遍历的递归实现

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
    vector<int> f;

    void inorder(TreeNode* root) {
        if (root == nullptr) return;
        inorder(root->left);
        f.push_back(root->val);
        inorder(root->right);
    }

    bool isValidBST(TreeNode* root) {
        inorder(root);
        for (int i = 1; i < f.size(); i ++) {
            if (f[i - 1] >= f[i]) return false;
        }

        return true;
    }
};
``` 



时间复杂度：O(n)，n为二叉树节点数

空间复杂度：O(n)，最坏情况下递归栈空间为O(n)，存储中序遍历结果也需要O(n)的空间

### 方法2: 二叉树中序遍历的迭代实现

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
    bool isValidBST(TreeNode* root) {
        stack<TreeNode*> stk;
        TreeNode* curr = root;
        long long prev = (long long)INT_MIN - 1; // 使用long long防止越界

        while (curr != nullptr || !stk.empty()) {
            while (curr != nullptr) {
                stk.push(curr);
                curr = curr->left;
            }
            curr = stk.top();
            stk.pop();

            // 检查当前节点值是否大于前一个节点值
            if (curr->val <= prev) {
                return false;
            }
            prev = curr->val;

            curr = curr->right;
        }

        return true;
    }
};
```

时间复杂度：O(n)，n为二叉树节点数

空间复杂度：O(n)，最坏情况下栈空间为O(n)


## 二叉树遍历非递归实现

### 方法：迭代实现二叉树的前序、中序、后序遍历

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
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> res;
        if (root == nullptr) return res;    
        stack<TreeNode*> stk;
        stk.push(root); 
        while (!stk.empty()) {
            TreeNode* node = stk.top();
            stk.pop();
            res.push_back(node->val);
            if (node->right) stk.push(node->right);
            if (node->left) stk.push(node->left);
        }
        return res;
    }   

    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> res;
        stack<TreeNode*> stk;
        TreeNode* curr = root;
        while (curr != nullptr || !stk.empty()) {
            while (curr != nullptr) {
                stk.push(curr);
                curr = curr->left;
            }
            curr = stk.top();
            stk.pop();
            res.push_back(curr->val);
            curr = curr->right;
        }
        return res;
    }

    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> res;
        if (root == nullptr) return res;    
        stack<TreeNode*> stk;
        stk.push(root); 
        while (!stk.empty()) {
            TreeNode* node = stk.top();
            stk.pop();
            res.push_back(node->val);
            if (node->left) stk.push(node->left);
            if (node->right) stk.push(node->right);
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```

时间复杂度：O(n)，n为二叉树节点数

空间复杂度：O(n)，最坏情况下栈空间为O(n)

## 参考资料

- [二叉树的前序、中序、后序遍历 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-preorder-traversal/description/)


