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

### 引用
- [[#定义引用|定义引用]]
- [[#引用做参数|引用做参数]]

### 内联函数
- [[#定义内联函数|定义内联函数]]
- [[#注意事项|注意事项]]

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

**三种特殊写法**：

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


## 2. 引用

### 定义引用

> 引用是给已存在的变量起「别名」，操作引用就是操作原变量本身。

- 语法:`类型 &别名 = 原变量;`
- 说明:必须在定义时初始化，且绑定后不能再改绑到别的变量
- 示例:

```cpp
#include <iostream>

int main() {
    int a = 10;
    int &r = a;               // r 是 a 的别名

    r = 20;                   // 改 r 就是改 a
    std::cout << a << "\n";  // 输出 20

    std::cout << &a << "\n"; // 取 a 的地址
    std::cout << &r << "\n"; // 取 r 的地址，和上面相同
    return 0;
}
```

> 注意事项:引用和原变量是**同一个地址**（同一块内存，没有自己独立的存储空间），`&r` 和 `&a` 取到的是同一个地址。

> 说明:引用和指针的区别——引用必须定义时初始化、不能改绑、不用 `*` 解引用；指针可以为空、能重新指向、要用 `*` 取值。

### 引用做参数

> 函数形参用引用，函数里能直接修改实参，也避免复制大对象（如字符串、容器）。

```cpp
#include <iostream>

void swap(int &a, int &b) {   // 形参是引用，直接改实参
    int t = a;
    a = b;
    b = t;
}

int main() {
    int x = 1, y = 2;
    swap(x, y);               // x、y 被直接交换
    std::cout << x << " " << y << "\n";   // 输出 2 1
    return 0;
}
```

**常引用**——加 `const` 只读不能改，常用它接收参数：

```cpp
#include <iostream>
#include <string>

void print(const std::string &s) {   // 只读、不复制
    std::cout << s;
}

int main() {
    print("hello");    // 能直接传字符串字面量
    return 0;
}
```


## 3. 内联函数

### 定义内联函数

> 内联函数用 `inline` 关键字建议编译器把「函数调用」直接替换成函数体，省掉调用开销（压栈、跳转、返回），适合短小、频繁调用的函数。

- 语法:`inline 返回类型 函数名(参数) { 函数体 }`
- 示例:

```cpp
#include <iostream>

inline int square(int x) {   // 建议编译器内联
    return x * x;
}

int main() {
    std::cout << square(5) << "\n";   // 编译时可能直接替换成 5 * 5
    return 0;
}
```

### 注意事项

- `inline` 只是「建议」：编译器会自己判断，短小的函数才可能被内联；有递归、循环的大函数通常被忽略
- 通常**写在头文件里**：内联函数在每个使用它的 .cpp 里都要看得到完整定义，所以一般定义在 .h 中
- 现代编译器开优化（如 `-O2`）时会自动内联，`inline` 关键字更多是为了解决「同一个函数在多个 .cpp 里定义会重复定义」的问题
- 别滥用：只给短小、频繁调用的小函数加 `inline`
