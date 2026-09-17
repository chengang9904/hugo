+++
date = '2026-09-14T06:42:12-07:00'
draft = false
title = '25. K 个一组翻转链表'
tags = ['链表']
categories = ['Leetcode Hot 100']
+++

## 题目

[25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/reverse-nodes-in-k-group/solutions/535727/k-ge-yi-zu-fan-zhuan-lian-biao-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

思路

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
    ListNode* reverse_in(ListNode* head) {
        if (!head) return nullptr;
        ListNode* prev = nullptr;
        ListNode* cur = head;
        while (cur) {
            ListNode* t = cur->next;
            cur->next = prev;
            prev = cur;
            cur = t;
        }
        return prev;
    }

    ListNode* reverseKGroup(ListNode* head, int k) {
        ListNode* dummyHead = new ListNode();
        dummyHead->next = head;
        ListNode *last_tail, *in_head, *in_tail, *next_head, *cur;
        cur = dummyHead;
        while (cur != nullptr) {
            last_tail = cur;
            in_head = cur->next;
            for (int i = 0; i < k; i ++) {
                if (cur) cur = cur->next;
            }
            if (cur != nullptr) {
                in_tail = cur;
                next_head = cur->next;
                in_tail->next = nullptr;   // 断开链表
                reverse_in(in_head);
                last_tail->next = in_tail;
                in_head->next = next_head;
                cur = in_head;
            }
        }

        return dummyHead->next;
    }
};
``` 

一定要记得断开链表，因为在翻转链表时，如果不将尾节点的 next 指针置为 nullptr，翻转后的链表会形成环，导致后续操作出错。

## 官方题解

```
class Solution {
public:
    // 翻转一个子链表，并且返回新的头与尾
    pair<ListNode*, ListNode*> myReverse(ListNode* head, ListNode* tail) {
        ListNode* prev = tail->next;
        ListNode* p = head;
        while (prev != tail) {
            ListNode* nex = p->next;
            p->next = prev;
            prev = p;
            p = nex;
        }
        return {tail, head};
    }

    ListNode* reverseKGroup(ListNode* head, int k) {
        ListNode* hair = new ListNode(0);
        hair->next = head;
        ListNode* pre = hair;

        while (head) {
            ListNode* tail = pre;
            // 查看剩余部分长度是否大于等于 k
            for (int i = 0; i < k; ++i) {
                tail = tail->next;
                if (!tail) {
                    return hair->next;
                }
            }
            ListNode* nex = tail->next;
            // 这里是 C++17 的写法，也可以写成
            // pair<ListNode*, ListNode*> result = myReverse(head, tail);
            // head = result.first;
            // tail = result.second;
            tie(head, tail) = myReverse(head, tail);
            // 把子链表重新接回原链表
            pre->next = head;
            tail->next = nex;
            pre = tail;
            head = tail->next;
        }

        return hair->next;
    }
};

```