+++
date = '2026-03-23T10:03:19+08:00'
draft = false
title = '23. 合并K个升序链表'
categories = ['Leetcode Hot 100']
tags = ['分治', '堆']
description = '困难 · 分治 · 堆'
+++

## 题目

[23. 合并K个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/description/?envType=problem-list-v2&envId=2cktkvj)  
[官方题解](https://leetcode.cn/problems/merge-k-sorted-lists/solutions/522559/he-bing-kge-sheng-xu-lian-biao-by-leetcode/?envType=problem-list-v2&envId=2cktkvj)

## 普通的归并写法

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* addToTail(ListNode* tail, ListNode* addNode) {
        addNode->next = nullptr;
        tail->next = addNode;
        return tail->next;
    }

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        ListNode* dummyHead = new ListNode(-1);
        ListNode* tail = dummyHead;
        lists.erase(remove(lists.begin(), lists.end(), nullptr), lists.end());
        while (lists.size()) {
            if (lists.empty()) break;
            ListNode* minNode = lists[0];
            int minList = 0;
            for (int i = 0; i < lists.size(); i ++) {
                if (lists[i]->val < minNode->val) {
                    minNode = lists[i];
                    minList = i;
                }
            }
            lists[minList] = lists[minList]->next;
            if (lists[minList] == nullptr) lists.erase(lists.begin() + minList);
            tail = addToTail(tail, minNode);
        }
        return dummyHead->next;
    }
};
```

## 堆优化

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        auto cmp = [](ListNode* a, ListNode* b) {
            return a->val > b->val;
        };
        priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);
        for (ListNode* list : lists) {
            if (list) {
                pq.push(list);
            }
        }
        ListNode* dummyHead = new ListNode(-1);
        ListNode* tail = dummyHead;
        while (!pq.empty()) {
            ListNode* node = pq.top();
            pq.pop();
            tail->next = node;
            tail = node;
            if (node->next) {
                pq.push(node->next);
            }
        }
        return dummyHead->next;
    }
};
```
