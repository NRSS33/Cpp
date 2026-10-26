---
title: 语言基础
tags:
  - c++
  - 语言基础
created: 2026-10-24
---

## 目录

### 命名空间
- [[#定义与使用|定义与使用]]
- [[#声明与实现分离|声明与实现分离]]
- [[#特殊写法|特殊写法]]

---

## 1. 命名空间

### 定义与使用

> 命名空间把全局作用域划分成具名区域，避免不同代码库的标识符（变量、函数、类等）重名冲突。

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

### 声明与实现分离

> 声明放头文件、实现放源文件。头文件和源文件里的 `namespace mylib {...}` 是同一个命名空间（可分多次定义、自动合并），所以能分开写。

```cpp
// mylib.h —— 只声明
#ifndef MYLIB_H            // 防止头文件被重复包含
#define MYLIB_H

namespace mylib {
    int add(int a, int b);   // 声明，不写函数体
    void hello();
}

#endif
```

```cpp
// mylib.cpp —— 实现
#include "mylib.h"

namespace mylib {            // 重新打开同一个命名空间
    int add(int a, int b) {
        return a + b;
    }

    void hello() {
        // ...
    }
}
```

```cpp
// main.cpp —— 调用
#include "mylib.h"

int main() {
    int x = mylib::add(1, 2);   // 用 :: 调用
    mylib::hello();
    return 0;
}
```

### 特殊写法

**嵌套命名空间**——一层套一层：

```cpp
namespace outer {               // 外层命名空间
    namespace inner {           // 内层命名空间
        int n = 1;
    }
}

int main() {
    int x = outer::inner::n;    // 访问要写完整路径
    return 0;
}
```

**匿名命名空间**——没有名字，成员仅当前 .cpp 可见，作用类似 `static`：

```cpp
namespace {                            // 没有名字
    int secret = 7;
    void helper() { /* 仅本文件可见 */ }
}

int main() {
    helper();              // 直接访问
    int x = ::secret;      // 也可以加 :: 限定
    return 0;
}
```

**内联命名空间**——成员自动"提升"到外层，常用于版本管理：

```cpp
namespace lib {
    inline namespace v2 { int f() { return 2; } }   // 当前版本
    namespace v1       { int f() { return 1; } }    // 旧版本
}

int main() {
    int a = lib::f();        // 直接访问，实际是 v2::f()
    int b = lib::v1::f();    // 旧版本要写 lib::v1::
    return 0;
}
```
