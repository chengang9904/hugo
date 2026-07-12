---
title: "Java 算法刷题的轻量化 VS Code 工作流"
date: 2026-07-11T22:45:00-07:00
draft: false
slug: java-algorithm-vscode-workflow
description: "一套同时适用于 LeetCode 核心代码模式、ACM 标准输入输出和面试手写测试的 Java 刷题工作流。"
categories:
  - Java
  - 算法
tags:
  - Java
  - VS Code
  - LeetCode
  - ACM
  - 算法刷题
---

写 Java 算法题时，最容易浪费时间的地方往往不是算法本身，而是项目结构、输入输出、测试代码和运行配置。

如果目标是：

- 在 VS Code 中快速编写和运行；
- 每道题都能保留；
- 支持硬编码测试；
- 支持 ACM 标准输入输出；
- 能直接复制核心代码到 LeetCode；

那么最合适的方式是：

> 每道题一个文件夹，`Solution.java` 只写核心算法，`Main.java` 负责编写测试和输入输出。

这套结构不需要 Maven、Gradle，也不需要 JUnit。

<!--more-->

## 一、目录结构

建议使用下面的结构：

```text
Algorithms/
├── .vscode/
│   └── settings.json
└── src/
    ├── p0001_twosum/
    │   ├── Solution.java
    │   └── Main.java
    ├── p0155_minstack/
    │   ├── MinStack.java
    │   └── Main.java
    └── p0695_max_area_island/
        ├── Solution.java
        └── Main.java
```

每道题单独放进一个文件夹。

包名不能以数字开头，因此推荐使用：

```text
p0001_twosum
p0695_max_area_island
```

而不是：

```text
0001_twosum
```

---

## 二、VS Code 配置

在项目根目录创建：

```text
.vscode/settings.json
```

内容如下：

```json
{
  "java.project.sourcePaths": [
    "src"
  ],
  "java.project.outputPath": "out",
  "java.debug.settings.console": "integratedTerminal"
}
```

这段配置的作用是：

- 将 `src` 作为源码目录；
- 将编译结果输出到 `out`；
- 在 VS Code 集成终端中运行程序；
- 支持标准输入。

之后直接用 VS Code 打开整个 `Algorithms` 文件夹即可。

---

## 三、核心代码：Solution.java

以 LeetCode 1 两数之和为例。

```java
package p0001_twosum;

import java.util.*;

public class Solution {

    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int need = target - nums[i];

            if (map.containsKey(need)) {
                return new int[]{map.get(need), i};
            }

            map.put(nums[i], i);
        }

        return new int[0];
    }
}
```

`Solution.java` 中只保留：

- 题目要求的方法；
- 必要的成员变量；
- 必要的辅助方法。

不要把测试逻辑写进这个文件。

提交到 LeetCode 时，只需要复制 `Solution` 类，并删除：

```java
package p0001_twosum;
```

---

## 四、测试代码：Main.java

```java
package p0001_twosum;

import java.util.*;

public class Main {

    public static void main(String[] args) {
        runTests();

        // ACM 模式时改为：
        // runFromInput();
    }

    private static void runTests() {
        Solution solution = new Solution();

        checkArray(
                "测试 1",
                new int[]{0, 1},
                solution.twoSum(new int[]{2, 7, 11, 15}, 9)
        );

        checkArray(
                "测试 2",
                new int[]{1, 2},
                solution.twoSum(new int[]{3, 2, 4}, 6)
        );

        checkArray(
                "测试 3",
                new int[]{0, 1},
                solution.twoSum(new int[]{3, 3}, 6)
        );
    }

    private static void runFromInput() {
        Scanner scanner = new Scanner(System.in);

        int n = scanner.nextInt();
        int[] nums = new int[n];

        for (int i = 0; i < n; i++) {
            nums[i] = scanner.nextInt();
        }

        int target = scanner.nextInt();

        Solution solution = new Solution();
        int[] answer = solution.twoSum(nums, target);

        System.out.println(Arrays.toString(answer));
    }

    private static void checkArray(
            String name,
            int[] expected,
            int[] actual
    ) {
        if (Arrays.equals(expected, actual)) {
            System.out.println(name + "：通过");
        } else {
            System.out.println(name + "：失败");
            System.out.println("期望：" + Arrays.toString(expected));
            System.out.println("实际：" + Arrays.toString(actual));
        }
    }
}
```

运行 `Main.java` 后，输出类似：

```text
测试 1：通过
测试 2：通过
测试 3：通过
```

---

## 五、在两种模式之间切换

### LeetCode / 面试模式

使用硬编码测试：

```java
public static void main(String[] args) {
    runTests();
}
```

适合：

- LeetCode 刷题；
- 面试现场验证；
- 调试边界条件；
- 快速构造测试用例。

### ACM 模式

改为：

```java
public static void main(String[] args) {
    runFromInput();
}
```

适合：

- 洛谷；
- 牛客；
- Codeforces；
- 需要标准输入输出的笔试平台。

核心算法仍然放在 `Solution.java` 中，不需要修改。

---

## 六、常用测试方法

### 比较整数

```java
private static void check(
        String name,
        int expected,
        int actual
) {
    if (expected == actual) {
        System.out.println(name + "：通过");
    } else {
        System.out.println(name + "：失败");
        System.out.println("期望：" + expected);
        System.out.println("实际：" + actual);
    }
}
```

### 比较字符串

```java
private static void check(
        String name,
        String expected,
        String actual
) {
    if (Objects.equals(expected, actual)) {
        System.out.println(name + "：通过");
    } else {
        System.out.println(name + "：失败");
        System.out.println("期望：" + expected);
        System.out.println("实际：" + actual);
    }
}
```

### 比较一维数组

```java
private static void checkArray(
        String name,
        int[] expected,
        int[] actual
) {
    if (Arrays.equals(expected, actual)) {
        System.out.println(name + "：通过");
    } else {
        System.out.println(name + "：失败");
        System.out.println("期望：" + Arrays.toString(expected));
        System.out.println("实际：" + Arrays.toString(actual));
    }
}
```

### 比较二维数组

```java
private static void checkArray(
        String name,
        int[][] expected,
        int[][] actual
) {
    if (Arrays.deepEquals(expected, actual)) {
        System.out.println(name + "：通过");
    } else {
        System.out.println(name + "：失败");
        System.out.println("期望：" + Arrays.deepToString(expected));
        System.out.println("实际：" + Arrays.deepToString(actual));
    }
}
```

### 比较 List

```java
private static <T> void checkList(
        String name,
        List<T> expected,
        List<T> actual
) {
    if (Objects.equals(expected, actual)) {
        System.out.println(name + "：通过");
    } else {
        System.out.println(name + "：失败");
        System.out.println("期望：" + expected);
        System.out.println("实际：" + actual);
    }
}
```

---

## 七、注意会修改输入的算法

有些算法会直接修改原数组，例如岛屿问题中的 DFS：

```java
grid[row][col] = 0;
```

因此，同一个测试数组不能重复使用：

```java
solution.maxAreaOfIsland(grid);
solution.maxAreaOfIsland(grid);
```

第二次运行时，原数组已经被修改。

可以写一个二维数组复制方法：

```java
private static int[][] copyGrid(int[][] grid) {
    int[][] copy = new int[grid.length][];

    for (int i = 0; i < grid.length; i++) {
        copy[i] = grid[i].clone();
    }

    return copy;
}
```

使用时：

```java
check(
        "测试 1",
        5,
        solution.maxAreaOfIsland(copyGrid(grid))
);
```

---

## 八、面试时的单文件版本

正式面试通常不需要拆成两个文件，可以临时写成一个文件：

```java
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Solution solution = new Solution();

        int[] result = solution.twoSum(
                new int[]{2, 7, 11, 15},
                9
        );

        System.out.println(Arrays.toString(result));
    }
}

class Solution {

    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int need = target - nums[i];

            if (map.containsKey(need)) {
                return new int[]{map.get(need), i};
            }

            map.put(nums[i], i);
        }

        return new int[0];
    }
}
```

推荐顺序：

1. 先完成核心方法；
2. 写一个普通测试；
3. 补充边界测试；
4. 运行验证；
5. 再分析复杂度。

---

## 九、为什么不建议一开始使用 JUnit

JUnit 很适合正式项目，但对刷题来说通常过重。

它会额外引入：

- Maven 或 Gradle；
- 测试目录；
- 测试依赖；
- 注解；
- 更多项目配置。

刷算法题时，手写 `check` 方法已经足够：

```java
check("测试 1", 5, actual);
```

这种方式没有依赖，复制方便，也更接近面试环境。

---

## 十、推荐工作流

每写一道新题：

```text
1. 在 src 下创建题目文件夹
2. 新建 Solution.java
3. 新建 Main.java
4. 在 Solution.java 中写核心算法
5. 在 Main.java 中写硬编码测试
6. 运行 Main.java
7. 通过后复制 Solution 到 LeetCode
```

平时刷题：

```text
Main.java 编写测试
        ↓
Solution.java 编写算法
        ↓
本地运行
        ↓
复制 Solution 到 LeetCode
```

ACM 模式：

```text
Main.java 读取标准输入
        ↓
调用 Solution
        ↓
输出答案
```

这套工作流的核心是：

> 核心算法和测试代码分离，但不引入复杂工程配置。

它既适合日常刷题，也适合面试和笔试环境。
