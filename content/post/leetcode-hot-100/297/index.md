+++
date = '2026-03-19T22:50:53+08:00'
draft = false
title = '297. 二叉树的序列化与反序列化'
categories = ['Leetcode Hot 100']
tags = ['树', '设计']

+++

## 题目

[297. 二叉树的序列化与反序列化](https://leetcode.cn/problems/serialize-and-deserialize-binary-tree/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/serialize-and-deserialize-binary-tree/solutions/522553/er-cha-shu-de-xu-lie-hua-yu-fan-xu-lie-hua-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Codec {
public:
    // Encodes a tree to a single string.
    string serialize(TreeNode* root) {
        string res;
        rserialize(root, res);
        return res;
    }

    void rserialize(TreeNode* root, string& str) {
        if (root == nullptr) {
            str += "None,";
        } else {
            str += to_string(root->val) + ",";
            rserialize(root->left, str);
            rserialize(root->right, str);
        }
    }

    TreeNode* rdeserialize(list<string>& sdata) {
        if (sdata.front() == "None") {
            sdata.erase(sdata.begin());
            return nullptr;
        }

        TreeNode* root = new TreeNode(stoi(sdata.front()));
        sdata.erase(sdata.begin());
        root->left = rdeserialize(sdata);
        root->right = rdeserialize(sdata);
        return root;
    }

    // Decodes your encoded data to tree.
    TreeNode* deserialize(string data) {
        list<string> sdata;
        string s;
        for (const auto& c: data) {
            if (c == ',') {
                sdata.push_back(s);
                s.clear();
            } else {
                s.push_back(c);
            }
        }
        if (!s.empty()) {
            sdata.push_back(s);
            s.clear();
        }
        return rdeserialize(sdata);
    }
};

// Your Codec object will be instantiated and called as such:
// Codec ser, deser;
// TreeNode* ans = deser.deserialize(ser.serialize(root));
```
