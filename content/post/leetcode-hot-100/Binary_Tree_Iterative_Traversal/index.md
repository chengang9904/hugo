+++
date = '2026-03-31T16:32:17+08:00'
draft = false
title = 'C++ 算法面试中的二叉树迭代遍历'
categories = ['计算机基础', '面试准备']
tags = ['C++', '算法', '二叉树', '遍历', '栈']
+++

## C++ 算法面试中的二叉树迭代遍历

在算法面试中，二叉树是常见的数据结构，其遍历算法是考察递归和栈操作的基本功。递归实现简单直观，但迭代实现更能体现对栈的理解和控制，尤其在避免递归深度过大导致栈溢出的场景下更为稳健。

---

### 1. 二叉树基础

二叉树由节点组成，每个节点最多有两个子节点：左子节点和右子节点。我们通常使用以下结构体定义二叉树节点：

```cpp
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};
```

---

### 2. 遍历方式概述

二叉树的遍历主要有三种方式，每种方式访问节点的顺序不同：

- **前序遍历 (Preorder)**: 先访问根节点，然后遍历左子树，最后遍历右子树。顺序：根 → 左 → 右
- **中序遍历 (Inorder)**: 先遍历左子树，然后访问根节点，最后遍历右子树。顺序：左 → 根 → 右
- **后序遍历 (Postorder)**: 先遍历左子树，然后遍历右子树，最后访问根节点。顺序：左 → 右 → 根

递归实现这些遍历非常简单，但这里我们重点讨论迭代实现，使用栈（Stack）来模拟递归过程。

---

### 3. 前序遍历迭代实现

#### 思路分析

前序遍历的顺序是根 → 左 → 右。在迭代中，我们使用栈来存储节点。先将根节点压入栈，然后进入循环：每次弹出栈顶节点，访问它，然后先压入右子节点，再压入左子节点（因为栈是后进先出，左子节点后压入但先弹出）。

#### 代码实现

```cpp
#include <vector>
#include <stack>

std::vector<int> preorderTraversal(TreeNode* root) {
    std::vector<int> result;
    if (!root) return result;
    std::stack<TreeNode*> stk;
    stk.push(root);
    while (!stk.empty()) {
        TreeNode* node = stk.top();
        stk.pop();
        result.push_back(node->val);
        if (node->right) stk.push(node->right);
        if (node->left) stk.push(node->left);
    }
    return result;
}
```

#### 解释

- 初始化栈，压入根节点。
- 循环中：弹出节点，记录值，先压右子节点，再压左子节点，确保左子树先被处理。

---

### 4. 中序遍历迭代实现

#### 思路分析

中序遍历的顺序是左 → 根 → 右。我们需要先遍历左子树到最左，然后访问根，再遍历右子树。使用栈模拟：沿着左子树一直压栈，直到无法继续，然后弹出访问，再转向右子树。

#### 代码实现

```cpp
#include <vector>
#include <stack>

std::vector<int> inorderTraversal(TreeNode* root) {
    std::vector<int> result;
    std::stack<TreeNode*> stk;
    TreeNode* curr = root;
    while (curr || !stk.empty()) {
        while (curr) {
            stk.push(curr);
            curr = curr->left;
        }
        curr = stk.top();
        stk.pop();
        result.push_back(curr->val);
        curr = curr->right;
    }
    return result;
}
```

#### 解释

- 使用指针 `curr` 遍历左子树，边遍历边压栈。
- 当无法继续左移时，弹出栈顶节点访问，然后转向右子树。

---

### 5. 后序遍历迭代实现

#### 思路分析

后序遍历的顺序是左 → 右 → 根，最难实现。一种方法是使用两个栈：第一个栈用于模拟前序遍历（根 → 右 → 左），第二个栈收集结果，然后弹出得到左 → 右 → 根。

#### 代码实现

```cpp
#include <vector>
#include <stack>

std::vector<int> postorderTraversal(TreeNode* root) {
    std::vector<int> result;
    if (!root) return result;
    std::stack<TreeNode*> stk1, stk2;
    stk1.push(root);
    while (!stk1.empty()) {
        TreeNode* node = stk1.top();
        stk1.pop();
        stk2.push(node);
        if (node->left) stk1.push(node->left);
        if (node->right) stk1.push(node->right);
    }
    while (!stk2.empty()) {
        result.push_back(stk2.top()->val);
        stk2.pop();
    }
    return result;
}
```

#### 解释

- `stk1` 模拟前序遍历，但压入顺序为右 → 左。
- `stk2` 收集节点，最终弹出顺序为后序。

另一种方法是使用一个栈，但需要标记访问状态，这里不展开。

---

### 6. 时间与空间复杂度分析

- **时间复杂度**: O(n)，其中 n 是节点数量，每个节点被访问一次。
- **空间复杂度**: O(h)，h 是树的高度，最坏情况下（单侧树）为 O(n)，平均情况下为 O(log n)。

---

### 7. 注意事项与面试技巧

- **递归 vs 迭代**: 递归代码简洁，但迭代避免栈溢出，面试中常要求迭代实现。
- **栈的使用**: 熟练掌握栈的压入弹出顺序，是实现的关键。
- **边界条件**: 处理空树、空节点。
- **LeetCode 相关**: 这三种遍历是 LeetCode 144、94、145 题，建议练习。

---

### 总结

二叉树迭代遍历是算法面试中的经典问题，掌握前序、中序、后序的迭代实现，能有效展示你的编程基础和对数据结构的理解。递归简单，但迭代更考验逻辑思维。

祝你面试顺利，拿到心仪的 Offer！
