+++
date = '2026-03-31T16:11:15+08:00'
draft = false
title = 'C++ 算法面试中的字符串处理（STL 总结）'
categories = ['计算机基础', '面试准备']
tags = ['C++', 'STL', '字符串', '面试', '算法']
+++

## C++ 算法面试中的字符串处理（STL 总结）

在算法面试中，字符串是常见的数据类型，高效、正确地处理字符串是考察基本功的重要一环。C++ 提供了强大的 `std::string` 类，同时也需要了解 C 风格字符串。

---

### 1. C 风格字符串 vs. `std::string`

在 C++ 中，我们主要使用 `std::string`，但了解 C 风格字符串（`char*`）仍然重要。

#### 1.1 C 风格字符串 (`char*`)

- **本质**: 字符数组，以空字符 `\0` 结尾。
- **示例**: `char str[] = "hello";` 或 `char* str_ptr = "world";` (后者通常是常量字符串，不能修改)。
- **处理函数**: `<cstring>` 头文件提供了 `strlen`, `strcpy`, `strcat`, `strcmp` 等函数。
- **优点**: 性能可能稍高（在某些特定场景下），兼容 C 语言。
- **缺点**:
  - **手动内存管理**: 容易出现缓冲区溢出、内存泄漏等问题，非常危险。
  - **操作复杂**: 拼接、查找等操作需要自己管理内存，不够直观。
- **面试建议**: **除非题目明确要求或在极其特殊性能场景下，否则不推荐使用。** 主要了解其概念和风险。

#### 1.2 `std::string` (C++ 标准库)

- **本质**: C++ 标准库提供的类，内部封装了字符数组，并自动管理内存。
- **头文件**: `<string>`
- **优点**:
  - **自动内存管理**: 无需担心缓冲区溢出，内存自动分配和释放。
  - **功能丰富**: 提供大量成员函数，操作直观方便。
  - **安全**: 大大减少了 C 风格字符串的常见错误。
- **面试建议**: **始终优先使用 `std::string`。** 掌握其常用操作是面试必备。

---

### 2. `std::string` 常用操作 (STL)

以下是 `std::string` 在算法面试中最常用的一些操作：

#### 2.1 创建与初始化

```cpp
std::string s1;                   // 空字符串
std::string s2 = "hello";         // 初始化为 C 风格字符串
std::string s3(s2);               // 拷贝构造
std::string s4(s2, 1, 3);         // 从s2的索引1开始，拷贝3个字符 ("ell")
std::string s5(5, 'a');           // 初始化为 "aaaaa"
```

#### 2.2 访问字符

- **`[]` 操作符**: `char c = s[i];` O(1) 访问。不进行边界检查，越界会是未定义行为。
- **`at()` 方法**: `char c = s.at(i);` O(1) 访问。进行边界检查，越界会抛出 `std::out_of_range` 异常。
- **示例**:
  ```cpp
  std::string s = "world";
  char c1 = s[0];   // 'w'
  char c2 = s.at(4); // 'd'
  // char c3 = s.at(10); // 抛出异常
  ```

#### 2.3 长度与容量

- **`length()` / `size()`**: 返回字符串的字符数量（不包括空字符）。两者等价。
- **`empty()`**: 判断字符串是否为空。
- **`capacity()`**: 返回字符串当前能存储的最大字符数，不重新分配内存。
- **示例**:
  ```cpp
  std::string s = "hello";
  std::cout << s.length() << std::endl; // 5
  std::cout << s.size() << std::endl;   // 5
  std::cout << s.empty() << std::endl;  // false
  ```

#### 2.4 遍历

- **范围 For 循环 (C++11+)**: 最简洁推荐。
  ```cpp
  std::string s = "abc";
  for (char c : s) {
      std::cout << c << " "; // a b c
  }
  ```
- **传统 For 循环 (索引)**:
  ```cpp
  for (size_t i = 0; i < s.length(); ++i) {
      std::cout << s[i] << " ";
  }
  ```
- **迭代器**:
  ```cpp
  for (auto it = s.begin(); it != s.end(); ++it) {
      std::cout << *it << " ";
  }
  ```

#### 2.5 拼接 (Concatenation)

- **`+` / `+=` 运算符**: 直观方便。
- **`append()` 方法**: 更灵活，可拼接子串、重复字符等。
- **示例**:
  ```cpp
  std::string s1 = "hello";
  std::string s2 = "world";
  std::string s3 = s1 + " " + s2; // "hello world"
  s1 += " there";                 // "hello there"
  s1.append("!!!");               // "hello there!!!"
  ```

#### 2.6 子串 (Substring)

- **`substr(pos, len)`**: 返回从 `pos` 位置开始，长度为 `len` 的子串。
  - `pos` 默认是 0。
  - `len` 默认到字符串末尾。
- **示例**:
  ```cpp
  std::string s = "abcdefg";
  std::string sub1 = s.substr(2, 3); // "cde"
  std::string sub2 = s.substr(4);    // "efg" (从索引4到末尾)
  ```

#### 2.7 查找 (Search)

- **`find(str/char, pos)`**: 从 `pos` 开始查找第一个匹配的子串或字符。
  - 返回找到的第一个位置索引，未找到则返回 `std::string::npos`。
- **`rfind(str/char, pos)`**: 从 `pos` 开始反向查找。
- **`find_first_of(str/char, pos)`**: 查找给定字符串中任意字符第一次出现的位置。
- **`find_last_of(str/char, pos)`**: 查找给定字符串中任意字符最后一次出现的位置。
- **示例**:
  ```cpp
  std::string s = "hello world hello";
  size_t pos1 = s.find("world"); // 6
  size_t pos2 = s.find("xyz");   // std::string::npos
  size_t pos3 = s.rfind("hello"); // 12
  size_t pos4 = s.find_first_of("ol"); // 2 (第一个是l)
  ```

#### 2.8 替换、删除、插入

- **`replace(pos, len, str)`**: 从 `pos` 开始，替换 `len` 个字符为 `str`。
- **`erase(pos, len)`**: 从 `pos` 开始，删除 `len` 个字符。
- **`insert(pos, str)`**: 在 `pos` 处插入 `str`。
- **示例**:
  ```cpp
  std::string s = "hello world";
  s.replace(6, 5, "there"); // s becomes "hello there"
  s.erase(5, 6);            // s becomes "hello"
  s.insert(5, " wonderful"); // s becomes "hello wonderful"
  ```

#### 2.9 比较

- **`==`, `!=`, `<`, `>`, `<=`, `>=` 运算符**: 直接进行字典序比较。
- **`compare(str)`**: 返回整数，0 表示相等，负数表示当前字符串小于 `str`，正数表示大于。
- **示例**:
  ```cpp
  std::string s1 = "apple";
  std::string s2 = "banana";
  std::cout << (s1 < s2) << std::endl; // true
  std::cout << s1.compare(s2) << std::endl; // 负数
  ```

#### 2.10 转换为数字 / 从数字转换

- **字符串转数字**:
  - `std::stoi(str)`: String to int
  - `std::stol(str)`: String to long
  - `std::stoll(str)`: String to long long
  - `std::stof(str)`: String to float
  - `std::stod(str)`: String to double
- **数字转字符串**:
  - `std::to_string(num)`: 将各种数字类型转换为 `std::string`。
- **示例**:

  ```cpp
  std::string s_num = "123";
  int num = std::stoi(s_num);       // num = 123
  std::string s_float = "3.14";
  double d = std::stod(s_float);    // d = 3.14

  int val = 456;
  std::string s_from_num = std::to_string(val); // s_from_num = "456"
  ```

#### 2.11 转换为 C 风格字符串

- **`c_str()`**: 返回一个指向内部 C 风格字符串的 `const char*` 指针。**注意**: 这个指针的生命周期与 `std::string` 对象绑定，当 `std::string` 对象被修改或销毁时，指针会失效。
- **`data()` (C++11+)**: 类似于 `c_str()`，但对于非 `const` 字符串，返回的指针可用于修改（C++11 `data()` 返回 `const char*`, C++17 `data()` 返回 `char*`）。
- **示例**:
  ```cpp
  std::string s = "test";
  const char* c_str_ptr = s.c_str();
  std::cout << c_str_ptr << std::endl; // "test"
  ```

---

### 3. STL 其它相关工具

#### 3.1 `std::string_view` (C++17)

- **作用**: 提供一个轻量级的、非拥有（non-owning）的字符串视图。它不复制字符串数据，只存储一个指向字符序列的指针和长度。
- **优点**:
  - **性能提升**: 避免了不必要的字符串拷贝，特别适合作为函数参数传递大型字符串片段。
  - **灵活性**: 可以指向 `std::string`、C 风格字符串或任何连续字符序列。
- **缺点**: 不拥有数据，其指向的原始字符串必须在 `string_view` 有效期内保持有效。
- **面试建议**: 如果面试官问到 C++17 新特性或字符串性能优化，可以提及 `string_view`。

#### 3.2 `std::stringstream`

- **头文件**: `<sstream>`
- **作用**: 允许你像使用 `std::cin` 和 `std::cout` 那样，方便地对字符串进行格式化输入输出。
- **场景**:
  - 将不同类型的数据组合成一个字符串。
  - 从字符串中按格式解析不同类型的数据。
- **示例**:

  ```cpp
  #include <sstream>
  // 构建字符串
  std::stringstream ss_build;
  ss_build << "Value: " << 123 << " " << 3.14;
  std::string result = ss_build.str(); // "Value: 123 3.14"

  // 解析字符串
  std::string data = "Name: Alice Age: 30";
  std::stringstream ss_parse(data);
  std::string tag1, name, tag2;
  int age;
  ss_parse >> tag1 >> name >> tag2 >> age;
  // tag1="Name:", name="Alice", tag2="Age:", age=30
  ```

- **面试建议**: 解决字符串与数字、复杂格式化字符串之间的转换问题时非常有用。

#### 3.3 `std::vector<char>`

- 有时，当你需要一个可变大小的字符缓冲区，并进行频繁的字符添加/修改时，`std::vector<char>` 可能比 `std::string` 在某些情况下更灵活或性能更好（尤其是在你知道最终大小，可以 `reserve` 的时候）。
- 可以将 `std::vector<char>` 转换为 `std::string`：`std::string s(vec.begin(), vec.end());`

---

### 4. 性能与注意事项

- **传参**: 在函数参数中，如果不需要修改字符串，请始终使用 `const std::string&` (`const` 引用) 来避免不必要的拷贝，提高性能。
- **频繁修改**: `std::string` 的底层数据结构是动态数组。频繁的修改操作（如在字符串中间插入、删除）可能导致内存的重新分配和数据拷贝，影响性能。
  - 如果需要大量拼接，考虑预留容量 (`s.reserve(new_capacity)`) 或使用 `stringstream`。
  - 在特定场景下，如果需要原地修改大量字符，`std::vector<char>` 可能更合适。
- **C 风格字符串的陷阱**:
  - **缓冲区溢出**: `strcpy` 不检查目标缓冲区大小，非常容易溢出。
  - **内存管理**: 确保 `new char[]` 和 `delete[]` 配对使用，避免内存泄漏。
  - **空字符 `\0`**: C 风格字符串必须以 `\0` 结尾，否则函数无法判断其长度。

---

### 总结

在 C++ 算法面试中，`std::string` 是处理字符串的首选工具。熟练掌握其创建、访问、遍历、拼接、子串、查找、转换等常用操作是基础。同时，了解 `std::string_view`（C++17）、`std::stringstream` 等高级工具，并在适当的场景下使用它们，能体现你对 C++ 语言和 STL 的深入理解。

祝你面试顺利，拿到心仪的 Offer！
