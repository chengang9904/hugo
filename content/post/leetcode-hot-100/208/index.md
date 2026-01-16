+++
date = '2026-01-14T12:33:11+08:00'
draft = false
title = '208. 实现 Trie (前缀树)'
description = '中等 · 设计'
categories = ['Leetcode Hot 100']
tags = ['设计', '字典树', '前缀树']
+++


## 题目

[208. 实现 Trie (前缀树)](https://leetcode.cn/problems/implement-trie-prefix-tree/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/implement-trie-prefix-tree/solutions/535766/shi-xian-trie-qian-zhui-shu-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一： 递归

```cpp
class Trie {
private:
    vector<Trie*> child;
    bool is_end;
public:
    Trie(): child(26), is_end(false) {}
    
    Trie* searchPrefix(string prefix) {
        Trie* node = this;
        for (char ch : prefix) {
            ch -= 'a';
            if (node->child[ch] == nullptr) {
                return nullptr;
            }
            node = node->child[ch];
        }
        return node;
    }

    void insert(string word) {
        Trie* node = this;
        for (char ch : word) {
            ch -= 'a';
            if (node->child[ch] == nullptr) {
                node->child[ch] = new Trie();
            }
            node = node->child[ch];
        }
        node->is_end = true;
    }
    
    bool search(string word) {
        Trie* node = searchPrefix(word);
        return node != nullptr && node->is_end;
    }
    
    bool startsWith(string prefix) {
        return searchPrefix(prefix) != nullptr;
    }
};

/**
 * Your Trie object will be instantiated and called as such:
 * Trie* obj = new Trie();
 * obj->insert(word);
 * bool param_2 = obj->search(word);
 * bool param_3 = obj->startsWith(prefix);
 */
```

