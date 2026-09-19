+++
date = '2026-09-19T16:41:00+08:00'
draft = false
title = '算法笔试 ACM 模式的输入处理'
description = '笔试 · ACM 模式 · cin / getline / stringstream · Java / Python 对照'
categories = ['计算机基础', '面试准备']
tags = ['C++', 'Java', 'Python', 'ACM', '输入输出', '笔试']
+++

LeetCode 是核心代码模式，只写函数就行。但笔试（牛客、赛码、各家公司自己的笔试系统）和一部分面试用的是 ACM 模式：输入要自己从标准输入读，输出要自己打印。

算法写对了、结果卡在读入上，是笔试里最冤的丢分方式。这篇按输入格式把常见场景过一遍，C++ 为主，最后附 Java / Python 对照。

先给结论，C++ 记住这四条，绝大多数题都够用：

- 知道个数：直接 `cin >>`，不用管换行。
- 不知道个数，读到结束：`while (cin >> x)`。
- 一行里个数不定：`getline` 读整行，再丢给 `stringstream` 拆。
- 逗号等特殊分隔符：先把分隔符换成空格，再走上一条。

---

## 零、模板骨架

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // 读入、计算、输出

    return 0;
}
```

- `bits/stdc++.h` 一次性包含所有标准库头文件，笔试省事。GCC 能用，MSVC（Visual Studio）没有这个头文件，本地用 VS 编译会报错。
- 开头两行关掉 C / C++ 流同步、解绑 `cin` 和 `cout`，数据量大（1e5 以上）时能快很多。不用想，直接写上。
- 关了同步之后，**不要再混用** `scanf/printf` 和 `cin/cout`，输出顺序会乱。
- 换行用 `'\n'`，别用 `endl`。`endl` 每次都会刷新缓冲区，输出多的时候很慢。

---

## 一、读取指定个数的数据

最常见的格式：先告诉你有多少个，再给数据。

关键点只有一个：**`cin >>` 会自动跳过所有空白字符（空格、换行、Tab）**。所以只要知道个数，数据写在一行还是分成多行，代码都一样，完全不用管换行。

### 1. 一维数组

```text
5
3 1 4 1 5
```

```cpp
int n;
cin >> n;
vector<int> a(n);
for (int i = 0; i < n; i++) cin >> a[i];
```

### 2. 二维矩阵

```text
3 4
1 2 3 4
5 6 7 8
9 10 11 12
```

```cpp
int n, m;
cin >> n >> m;
vector<vector<int>> g(n, vector<int>(m));
for (int i = 0; i < n; i++)
    for (int j = 0; j < m; j++)
        cin >> g[i][j];
```

### 3. 字符网格（迷宫、地图）

```text
3 4
S..#
.#..
...E
```

每行中间没有空格，直接当字符串读，一次读一行：

```cpp
int n, m;
cin >> n >> m;
vector<string> g(n);
for (int i = 0; i < n; i++) cin >> g[i];
// 之后用 g[i][j] 访问
```

- 前提是行内没有空格。行内有空格就要用 `getline`，见第四节。

### 4. 图的边

第一行 `n m` 表示 n 个点、m 条边，接下来 m 行每行 `u v w`：

```text
4 3
1 2 5
2 3 1
3 4 2
```

```cpp
int n, m;
cin >> n >> m;
vector<vector<pair<int, int>>> adj(n + 1);   // 点编号从 1 开始，直接开 n + 1
for (int i = 0; i < m; i++) {
    int u, v, w;
    cin >> u >> v >> w;
    adj[u].push_back({v, w});
    adj[v].push_back({u, w});                // 无向图才加这一行
}
```

- 点编号从 1 开始就开 `n + 1`，别自己手动减 1 转成 0 下标，写着写着就漏了。

### 5. 多组测试数据，先给组数 T

```text
2
3
1 2 3
2
4 5
```

```cpp
int T;
cin >> T;
while (T--) {
    int n;
    cin >> n;
    vector<int> a(n);
    for (int i = 0; i < n; i++) cin >> a[i];
    // 处理这一组，输出答案
}
```

> 坑：多组数据时，每组用到的容器、计数器要么定义在循环里面，要么每组开头手动清空。定义成全局变量又忘了清，就会第一组对、后面全错。

---

## 二、不知道个数，一直读到输入结束

题目只说「输入若干个整数」或者「多组数据，处理到文件末尾」，不告诉你有多少。

### 1. 读到 EOF

```cpp
vector<int> a;
int x;
while (cin >> x) a.push_back(x);
```

`cin >> x` 读不到东西时会返回 false，循环自然结束。

### 2. 多组数据读到 EOF

```text
1 2
3 4
5 6
```

```cpp
int a, b;
while (cin >> a >> b) {
    cout << a + b << '\n';
}
```

### 3. 以特殊值结束

比如「输入 `0 0` 时结束」：

```cpp
int a, b;
while (cin >> a >> b) {
    if (a == 0 && b == 0) break;
    cout << a + b << '\n';
}
```

> 本地调试时，在控制台里手动敲输入，程序不会自己「读到结束」，会一直卡着等。要手动发 EOF：Windows 下按 `Ctrl + Z` 再回车，Linux / macOS 下按 `Ctrl + D`。
>
> 更省事的办法是把样例存成文件，重定向进去跑。注意 PowerShell 不支持 `<`：
>
> ```bash
> # Git Bash / Linux / macOS / cmd
> ./main < in.txt
>
> # PowerShell
> Get-Content in.txt | .\main.exe
> ```

---

## 三、一行数据，个数不定，空格分隔

这是最容易翻车的格式：

```text
1 2 3 4 5
10
```

第一行是数组，没给个数；第二行是 target。

这时候不能用 `while (cin >> x)`。`cin >>` 不认识换行，会把第二行的 `10` 也当成数组元素读进去。

正确做法：**先用 `getline` 把整行读成字符串，再用 `stringstream` 从这一行里拆数字**。相当于把「读到 EOF」的范围限制在一行里。

```cpp
string line;
getline(cin, line);

stringstream ss(line);
vector<int> nums;
int x;
while (ss >> x) nums.push_back(x);

int target;
cin >> target;
```

- `stringstream` 的 `>>` 和 `cin >>` 规则一样，按空白切分，多个连续空格也没关系。

### 多行，每行个数都不定

```text
1 2 3
4 5
6
```

```cpp
string line;
while (getline(cin, line)) {
    stringstream ss(line);
    vector<int> row;
    int x;
    while (ss >> x) row.push_back(x);
    if (row.empty()) continue;   // 跳过空行（包括只有空格或 \r 的行）
    // 处理这一行
}
```

- 用 `row.empty()` 判断，比用 `line.empty()` 稳：有些输入是 Windows 换行 `\r\n`，「空行」里其实还剩一个 `\r`。

---

## 四、特殊符号分隔：逗号、冒号、方括号

```text
1,2,3,4,5
```

或者直接给你 LeetCode 风格的数组：

```text
[1,2,3,4,5]
```

### 方法一：把分隔符换成空格（最推荐）

`stringstream` 只认空白，那就把逗号、方括号都换成空格，问题就回到了第三节。

```cpp
string line;
getline(cin, line);
for (char& c : line) {
    if (c == ',' || c == '[' || c == ']') c = ' ';
}

stringstream ss(line);
vector<int> nums;
int x;
while (ss >> x) nums.push_back(x);
```

再狠一点：**只要不是数字和负号，统统换成空格**。逗号、分号、方括号、`x = 1, y = 2` 这种格式，都能把数字抠出来：

```cpp
for (char& c : line) {
    if ((c < '0' || c > '9') && c != '-') c = ' ';
}
```

这招有几个前提：

- 有小数要把 `.` 也留下，读成 `double`。
- `-` 本身当分隔符用的（比如 `2024-01-05`）不行，会被当成负号。这种用方法三。
- 字母和数字混在一起的（比如 `x1 = 3`）也不行，会把 `1` 也抠出来。

### 方法二：getline 指定分隔符切分

`getline` 的第三个参数可以指定分隔符，默认是 `'\n'`。拿它在 `stringstream` 上切：

```cpp
string line;
getline(cin, line);

stringstream ss(line);
string token;
vector<int> nums;
while (getline(ss, token, ',')) {
    nums.push_back(stoi(token));
}
```

比方法一麻烦，只在这两种情况下用：

- 切出来的是字符串，而且字符串里可能带空格，比如 `Tom Smith,Alice,Bob`。
- 需要保留空字段，比如 `a,,b` 要切出 3 段，中间是空串。

> 坑：
> 1. `stoi` 超出 int 范围会抛异常，数据大就用 `stoll`。
> 2. 空字段 `stoi("")` 也会抛异常，转数字之前先判空。
> 3. 行尾带 `\r` 的话，最后一个 token 也会带着 `\r`。转数字没影响，但拿去做字符串比较就对不上了。

### 方法三：格式固定时，用一个 char 把分隔符读掉

格式固定（时间 `12:30`、日期 `2024-01-05`），或者题目给了个数，最省事的是用一个 `char` 变量把分隔符「吃掉」：

```cpp
char ch;

// 12:30
int h, m;
cin >> h >> ch >> m;

// 2024-01-05
int y, mo, d;
cin >> y >> ch >> mo >> ch >> d;

// 已知 n 个数，逗号分隔：1,2,3,4,5
vector<int> a(n);
for (int i = 0; i < n; i++) {
    if (i > 0) cin >> ch;   // 第一个数前面没有逗号
    cin >> a[i];
}
```

- `cin >> ch` 同样会跳过空白，所以 `1, 2, 3` 这种逗号后面带空格的也能读。
- 用 `scanf("%d:%d", &h, &m)` 也行，但前面说了，关了同步就别和 `cin` 混用。

---

## 五、字符串：`cin >>` 和 `getline` 的区别与混用

两种读法的区别：

- `cin >> s`：读一个「单词」，碰到空白就停。
- `getline(cin, s)`：读一整行，包括中间的空格，不包括末尾的 `\n`。

混用时有个经典的坑：

```text
3
hello world
foo bar
baz
```

第一行是行数，后面每行是一个可能带空格的字符串。直接这么写会出错：

```cpp
int n;
cin >> n;
string line;
getline(cin, line);   // 读到的是空字符串！
```

> 坑：`cin >> n` 读完 `3` 就停了，`3` 后面那个换行符还留在缓冲区里。紧接着的 `getline` 一上来就碰到换行，直接结束，读到一个空行。

解决办法是在 `getline` 之前把空白跳过去：

```cpp
int n;
cin >> n;
vector<string> lines(n);
for (int i = 0; i < n; i++) {
    getline(cin >> ws, lines[i]);   // ws：跳过开头所有空白，包括残留的换行
}
```

- 为什么不用网上常见的 `cin.ignore()`？它默认只跳过一个字符。如果 `3` 后面还跟了个空格或者 `\r`，照样出错。`cin >> ws` 不管有几个都吃掉，更稳。
- `ws` 的代价是行首空格、空行也会被跳过。绝大多数题无所谓。真遇到在意行首空格的题，就在 `cin >> n` 之后单独 `getline` 一次，把这一行剩下的部分读掉扔了。

---

## 六、顺手说一下输出

- 空格分隔输出，行末不留多余空格。大部分评测会忽略行末空格，但不是全部，没必要赌：

```cpp
for (int i = 0; i < n; i++) {
    if (i > 0) cout << ' ';
    cout << a[i];
}
cout << '\n';
```

- 保留小数：

```cpp
cout << fixed << setprecision(2) << ans << '\n';   // 保留 2 位
```

- 数据范围超过 int（约 2.1e9），或者要把一堆 1e9 级别的数加起来，读入和答案都用 `long long`。

---

## 七、最通用的写法（复制即用）

把前面最常用的两个操作封装一下，笔试开场直接贴：

```cpp
#include <bits/stdc++.h>
using namespace std;

// 把一行里的整数全抠出来：兼容空格、逗号、[1,2,3] 等格式
vector<long long> parseNums(string line) {
    for (char& c : line) {
        if ((c < '0' || c > '9') && c != '-') c = ' ';
    }
    stringstream ss(line);
    vector<long long> res;
    long long x;
    while (ss >> x) res.push_back(x);
    return res;
}

// 按指定分隔符切分成字符串，中间的空字段会保留
vector<string> split(const string& s, char delim) {
    vector<string> res;
    stringstream ss(s);
    string token;
    while (getline(ss, token, delim)) res.push_back(token);
    return res;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    string line;
    getline(cin, line);
    vector<long long> nums = parseNums(line);

    // ...

    return 0;
}
```

还有一个兜底办法：题面写得含糊、样例和描述对不上、拿不准格式的时候，先把所有行原样读进来，再慢慢解析：

```cpp
vector<string> lines;
string line;
while (getline(cin, line)) lines.push_back(line);
// lines.size() 就是总行数，lines[i] 是第 i 行
```

先跑一遍样例，把读进来的东西打印出来核对，比对着题面猜强得多。

---

## 八、Java / Python 对照

### Java

`Scanner` 最好上手，和 C++ 的写法基本一一对应：

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        // 知道个数
        int n = sc.nextInt();
        int[] a = new int[n];
        for (int i = 0; i < n; i++) a[i] = sc.nextInt();

        // 读整行：和 C++ 同一个坑，nextInt 之后要先把残留的换行吃掉
        sc.nextLine();
        String line = sc.nextLine().trim();

        String[] bySpace = line.split("\\s+");   // 空格分隔，连续多个空格也行
        String[] byComma = line.split(",");      // 逗号分隔
        // 万能版：非数字、非负号全换成空格再切，和 C++ 的 parseNums 一个思路
        String[] tokens = line.replaceAll("[^0-9-]+", " ").trim().split("\\s+");

        // 读到 EOF
        while (sc.hasNextInt()) {
            int x = sc.nextInt();
        }
    }
}
```

- 类名必须是 `Main`，牛客之类的平台不认别的名字。
- 空字符串 `split` 之后会得到一个 `[""]`，直接 `Integer.parseInt` 会报错，空行要先判断。

`Scanner` 慢，数据量到 1e5 以上可能超时，换成 `BufferedReader` + `StringTokenizer`：

```java
import java.io.*;
import java.util.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        int n = Integer.parseInt(br.readLine().trim());
        StringTokenizer st = new StringTokenizer(br.readLine());
        int[] a = new int[n];
        for (int i = 0; i < n; i++) a[i] = Integer.parseInt(st.nextToken());

        // 读到 EOF
        String line;
        while ((line = br.readLine()) != null) {
            StringTokenizer t = new StringTokenizer(line, " ,");   // 第二个参数：空格和逗号都算分隔符
            while (t.hasMoreTokens()) {
                int x = Integer.parseInt(t.nextToken());
            }
        }
    }
}
```

- 这种写法是按行读的，要求 n 个数都在同一行。换行位置不固定的题，还是用 `Scanner` 省心。
- 输出多的时候别在循环里 `System.out.println`，先拼到 `StringBuilder` 里，最后一次性打印。

### Python

Python 写输入最短：

```python
n = int(input())
a = list(map(int, input().split()))        # 一行空格分隔，个数不定也一样
b = list(map(int, input().split(',')))     # 逗号分隔
```

最通用的是一把读完所有 token，再按顺序取，完全不用管换行：

```python
import sys

data = sys.stdin.read().split()
it = iter(data)
n = int(next(it))
a = [int(next(it)) for _ in range(n)]
```

格式乱七八糟的（`[1,2,3]`、`x = 1, y = 2`），直接用正则抠数字：

```python
import re

nums = list(map(int, re.findall(r'-?\d+', input())))
```

- `input()` 慢，数据量大就在开头加 `input = sys.stdin.readline`。注意它返回的行带着 `\n`：`int()` 和 `split()` 不受影响，但读字符串要记得 `.strip()`。

---

## 小结

我的建议：

- C++ 记四个写法就够了：
  - 知道个数 → `cin >>`，不用管换行。
  - 读到结束 → `while (cin >> x)`。
  - 一行个数不定 → `getline` + `stringstream`。
  - 特殊分隔符 → 先换成空格，再 `stringstream`。只有要切字符串、要保留空字段时才用 `getline(ss, token, ',')`。
- 两个习惯永远带上：开头写 `ios::sync_with_stdio(false); cin.tie(nullptr);`；`cin >>` 后面接 `getline` 之前先 `cin >> ws`。
- Java 先用 `Scanner`，类名写 `Main`；数据量大再换 `BufferedReader`。
- Python 最省心的是 `sys.stdin.read().split()` 一把读完，再按顺序取。
- 拿不准格式，就先把所有行读进来打印看看，别对着题面猜。
