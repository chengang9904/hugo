+++
date = '2026-09-14T06:04:02-07:00'
draft = false
title = '146. LRU 缓存机制'
tags = ['哈希表', '链表']
categories = ['Leetcode Hot 100']
+++

## 题目

[146. LRU 缓存机制](https://leetcode.cn/problems/lru-cache/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/lru-cache/solutions/535732/lru-huan-cun-ji-zhi-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)



```cpp
struct DlinkNode {
    int key, val;
    DlinkNode* prev;
    DlinkNode* next;
    DlinkNode(): key(0), val(0), prev(nullptr), next(nullptr) {}
    DlinkNode(int _key, int _val): key(_key), val(_val), prev(nullptr), next(nullptr) {}
};


class LRUCache {
private:
int capacity;
int size;
unordered_map<int, DlinkNode*> cache;
DlinkNode* head, *tail;

public:
    LRUCache(int _capacity): capacity(_capacity), size(0) {
        head = new DlinkNode();
        tail = new DlinkNode();
        head->next = tail;
        tail->prev = head;
    }

    void remove(DlinkNode* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    void addToHead(DlinkNode* node) {
        node->next = head->next;
        node->prev = head;
        head->next = node;
        node->next->prev = node;        
    }

    DlinkNode* removeTail() {
        DlinkNode* removed = tail->prev;
        remove(removed);
        return removed;
    }
    
    int get(int key) {
        if (!cache.count(key)) {
            return -1;
        } else {
            DlinkNode* node = cache[key];
            remove(node);
            addToHead(node);
            return node->val;
        }
    }
    
    void put(int key, int value) {
        if (cache.count(key)) {
            DlinkNode* node = cache[key];
            remove(node);
            addToHead(node);
            node->val = value;
        } else {
            DlinkNode* newNode = new DlinkNode(key, value);
            cache[key] = newNode;
            addToHead(newNode);
            size ++;
            if (size > capacity) {
                DlinkNode* removed = removeTail();
                cache.erase(removed->key);
                delete removed;
                size --;
            }
        }
    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
 ```