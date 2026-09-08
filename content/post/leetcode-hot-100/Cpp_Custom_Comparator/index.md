+++
date = '2026-09-08T23:00:00+08:00'
draft = false
title = 'C++ 算法面试中的自定义比较器（sort 与 priority_queue）'
categories = ['计算机基础', '面试准备']
tags = ['C++', 'STL', 'sort', 'priority_queue', '比较器']
+++

## C++ 算法面试中的自定义比较器（sort 与 priority_queue）

写了一段代码想把 `sort` 和 `priority_queue` 自定义比较器的几种写法串一遍，结果发现自己写的版本里藏了两个坑：一个是命名冲突，一个是比较器语义理解反了。整理一下，方便以后复习。

---

### 1. sort 的比较器：四种写法，语义都一样

`sort` 的比较器回答的问题很直观：**`cmp(a, b)` 返回 true，表示 a 应该排在 b 前面**。下面四种写法效果完全一致，都是升序：

```cpp
sort(v.begin(), v.end());                    // 默认用 less<int>，等价于 a < b

sort(v.begin(), v.end(), [](const int& a, const int& b) {
    return a < b;                            // lambda，写法更灵活
});

sort(v.begin(), v.end(), greater<int>());    // 内置的“反过来”比较器，实现降序

struct AscCmp {
    bool operator()(const int& a, const int& b) const { return a < b; }
};
sort(v.begin(), v.end(), AscCmp());          // 自定义仿函数，效果同前两种升序写法
```

> 有个坑很多人会踩：比较器必须满足**严格弱序**（strict weak ordering），不能写 `a <= b` 这种"非严格"比较。原因是 `sort` 内部会用 `!cmp(a,b) && !cmp(b,a)` 来判断"相等"，如果 `cmp` 是 `<=`，那么 `cmp(a,a)` 恒为 true，直接破坏了这个假设，属于未定义行为（UB）。轻则排序结果不对，重则实际运行直接崩溃或死循环，libstdc++ 开了 `_GLIBCXX_DEBUG` 的话会直接断言失败。

### 2. priority_queue 才是重灾区：比较器语义是反的

`priority_queue` 底层就是 `push_heap`/`pop_heap` 那一套堆操作，比较器回答的不是"排序顺序"，而是**"谁的优先级更低"**：`cmp(a, b)` 返回 true，表示 a 的优先级比 b 低。堆顶永远是"优先级最高"的元素，也就是这个比较器眼里的"最大值"。

所以：

- 默认比较器是 `less<T>`（`a < b`）→ 大顶堆，`top()` 是最大值。
- 想要小顶堆，要传 `greater<T>`（`a > b`），而不是自己写一个 `a < b` 的比较器。

```cpp
priority_queue<int> pqDefault;                              // 大顶堆
priority_queue<int, vector<int>, greater<int>> pqMin;        // 小顶堆，靠 greater<int>
```

> 我原来的代码里，`struct cmp` 和后面那个 lambda 都写的是 `return a < b;`，直接传给 `priority_queue` 当比较器。这跟**完全不传比较器**的效果一模一样，还是大顶堆——如果本意是想要小顶堆，这两个比较器等于白写了。想用仿函数/lambda 实现小顶堆，比较器里要故意把 `a`、`b` 反过来写成 `a > b`：
>
> ```cpp
> struct MinHeapCmp {
>     bool operator()(const int& a, const int& b) const { return a > b; }
> };
> priority_queue<int, vector<int>, MinHeapCmp> pqMin2;
> ```

### 3. 命名冲突：struct cmp 和 auto cmp 撞车

原代码在全局定义了 `struct cmp`，又在 `main()` 里写了 `auto cmp = [](...){...};`。这两行都能编译过，但纯属侥幸：局部变量 `cmp` 会遮蔽外层的类型名 `cmp`，恰好我在声明这个变量*之前*就用完了 `cmp()`（构造仿函数），所以没出事。如果把这两段顺序换一下，或者在变量声明之后还想用 `cmp` 当类型名，编译器会直接报"cmp 不是一个类型"。

写这种对比测试代码时，不要图省事把仿函数和 lambda 都叫 `cmp`，分开命名（比如 `CmpFunctor` / `cmpLambda`），省得自己把自己绕进去。

### 4. lambda 当 priority_queue 比较器，为什么必须传实例

```cpp
auto minHeapCmp = [](const int& a, const int& b) { return a > b; };
priority_queue<int, vector<int>, decltype(minHeapCmp)> pqMin3(minHeapCmp);
```

`decltype(minHeapCmp)` 只是拿到了这个 lambda 的类型，但 C++17 及之前，无捕获 lambda 是**没有默认构造函数**的（C++20 才补上）。如果只写 `priority_queue<int, vector<int>, decltype(minHeapCmp)> pqMin3;` 不传参数，编译会直接失败。所以必须显式传实例 `pqMin3(minHeapCmp)`，让 `priority_queue` 内部拿这个实例去初始化它的比较器成员。

仿函数（`struct` + `operator()`）因为自带默认构造函数，不用管这一茬，这也是 `priority_queue` 场景里仿函数比 lambda 更常见的原因。

### 5. 重载 operator< vs 写外部比较器，怎么选

```cpp
struct Node {
    int x;
    bool operator<(const Node& other) const { return x < other.x; }
};

priority_queue<Node> pqNode;   // 不用传比较器，直接用默认的 less<Node>
```

给 `Node` 重载了 `operator<` 之后，`sort` 和 `priority_queue` 不传比较器就能直接工作。要不要这么做，看这个"大小关系"是不是类型本身**唯一、公认**的语义：

- 如果是（比如按时间排序的事件、按权值排序的边），重载 `operator<` 一劳永逸，到处都能用默认比较。
- 如果只是某次算法题里**临时**需要的排序方式（这次按 x 升序，下次可能要按 y 降序），就用 lambda 或仿函数，不要为了一次性需求去改动类型定义。

---

### 小结

- `sort` 的比较器好理解：返回 true 表示 a 排 b 前面，四种写法（默认/lambda/greater/仿函数）选哪个纯粹是场景和口味问题。
- `priority_queue` 的比较器语义是反的：返回 true 表示 a 优先级更低。默认大顶堆，想要小顶堆记得传 `greater<T>`，或者自定义比较器时故意把 `<` 写成 `>`——写成 `a < b` 只是重新实现了一遍默认行为。
- `priority_queue` 配合仿函数最省心（自带默认构造），配合 lambda 记得用 `decltype` 拿类型、构造时传实例。
- 类型自身有固定、唯一的大小关系就重载 `operator<`；只是某次排序的临时需求就用外部比较器，别为了一次性场景改类型定义。
- 别把仿函数和 lambda 起同一个名字，尤其是想把好几种写法放在一起对比复习的时候。
