+++
date = '2026-01-28T11:10:24+08:00'
draft = false
title = '155. 最小栈'
description = '简单 · 设计 · 栈'
categories = ['Leetcode Hot 100']
tags = ['设计', '栈']
+++

## 题目

[155. 最小栈](https://leetcode.cn/problems/min-stack/description/?envType=problem-list-v2&envId=2cktkvj)
[官方题解](https://leetcode.cn/problems/min-stack/solutions/535557/zui-xiao-zhan-by-leetcode-solution/?envType=problem-list-v2&envId=2cktkvj)

## 方法一：栈

```cpp
class MinStack {
    stack<int> x_stack;
    stack<int> min_stack;

public:
    MinStack() {
        min_stack.push(INT_MAX);
    }

    void push(int val) {
        x_stack.push(val);
        min_stack.push(min(min_stack.top(), val));
    }

    void pop() {
        x_stack.pop();
        min_stack.pop();
    }

    int top() {
        return x_stack.top();
    }

    int getMin() {
        return min_stack.top();
    }
};

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack* obj = new MinStack();
 * obj->push(val);
 * obj->pop();
 * int param_3 = obj->top();
 * int param_4 = obj->getMin();
 */
```
