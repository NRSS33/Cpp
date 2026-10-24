---
title: 语言基础
tags:
  - c++
  - 语言基础
created: 2026-10-24
---

## 目录

- [[#命名空间|命名空间]]

---

## 命名空间

**作用**：命名空间把全局作用域划分成具名区域，避免不同代码库的标识符（变量、函数、类等）重名冲突。

**用法**：
- 定义:`namespace 名称 { 声明... }`，同一命名空间可分多次定义（内容自动合并）
- 访问:`名称::成员`
- 引入:
  - `using std::cout;` — 只引入单个名字（推荐）
  - `using namespace std;` — 引入整个命名空间（别在头文件的全局作用域用）
  - `namespace fs = std::filesystem;` — 起别名
- 标准库都在 `std` 命名空间里:`std::cout`、`std::string`、`std::vector` 等

```cpp
#include <iostream>

namespace mylib {                  // 定义
    int value = 42;
}
namespace ml = mylib;              // 别名

using std::cout;                   // 只引入 cout

int main() {
    cout << ml::value << "\n";    // 用别名访问（等价 mylib::value）
    return 0;
}
```

> 附:三种特殊写法——嵌套 `namespace A::B {...}`（C++17 起）、匿名 `namespace {...}`（成员仅当前 .cpp 可见，作用类似 `static`）、内联 `inline namespace v2 {...}`（成员自动提升到外层，常用于版本管理）。
