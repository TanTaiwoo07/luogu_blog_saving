# 古代 C++ 与现代 C++

> **一句话结论**：社区约定以 **C++11** 为分水岭——C++11 之前称「古代 C++」（C++98/03 风格），C++11 及之后称「现代 C++」。两者是**同一门语言在不同时代的写法风格**，并非两门语言；现代写法更安全、简洁、抽象，但古代写法在存量代码中依然常见。

---

## 目录

- [一、核心结论速查](#一核心结论速查)
- [二、为什么以 C++11 为分水岭](#二为什么以-c11-为分水岭)
- [三、特性对照总表](#三特性对照总表)
- [四、古代 C++ 特征](#四古代-c-特征)
- [五、现代 C++ 特征](#五现代-c-特征)
- [六、同一功能的两种写法](#六同一功能的两种写法)
- [七、底层原理](#七底层原理)
- [八、为什么古代写法依然存在](#八为什么古代写法依然存在)
- [九、规范与延伸](#九规范与延伸)

---

## 一、核心结论速查

| 维度 | 古代 C++（C++98/03） | 现代 C++（C++11 起） |
| --- | --- | --- |
| 内存管理 | 手动 `new`/`delete`、裸指针 | 智能指针 + RAII，自动释放 |
| 对象传递 | 深拷贝 / 传引用、指针绕行 | 移动语义（`std::move`），O(1) 转移 |
| 类型书写 | 完整类型名（迭代器等冗长） | `auto` 类型推导 |
| 遍历 | 手写迭代器 `for` | 范围 `for`（`for (x : c)`） |
| 匿名函数 | 函数对象 / 独立函数 | `lambda` 表达式 |
| 空指针 | `NULL` / `0`（本质是 int） | `nullptr`（类型安全） |
| 模板能力 | 无右值引用、完美转发、变参模板 | 右值引用、完美转发、`variadic` |
| 编译期计算 | 宏 / `enum` 技巧 | `constexpr` |
| 模块系统 | `#include` 文本包含 + 宏 | `import` 模块（C++20） |
| 模板约束 | 无，报错晦涩 | `concept`/`requires`（C++20） |

---

## 二、为什么以 C++11 为分水岭

C++ 自 1985 年诞生、1998 年首个 ISO 标准（C++98）以来历经多次修订，但真正让**写法发生质变**的是 **C++11**：

- **C++98 / C++03**：带浓厚 C 风格，手动资源管理为主流。
- **C++11（2011）**：引入智能指针、移动语义、`auto`、范围 `for`、`lambda`、`nullptr`、`constexpr`、右值引用等，奠定现代风格。
- **C++14 / 17 / 20 / 23**：持续增强；`ranges`、模块、概念在 C++20 补齐函数式与编译模型短板。

> 因此社区把 **C++11 之前** 统称古代，**C++11 及之后** 统称现代。

---

## 三、特性对照总表

| 特性 | 古代写法 | 现代写法 | 引入版本 |
| --- | --- | --- | --- |
| 内存管理 | `new`/`delete` | `unique_ptr`/`shared_ptr` | C++11 |
| 所有权转移 | 拷贝 / 引用 | `std::move` + 移动构造 | C++11 |
| 类型推导 | 显式类型 | `auto` | C++11 |
| 遍历 | 迭代器 `for` | 范围 `for` | C++11 |
| 匿名函数 | `functor` / 函数 | `lambda` | C++11 |
| 空指针 | `NULL`/`0` | `nullptr` | C++11 |
| 编译期计算 | 宏 | `constexpr` | C++11 |
| 转发 | 手写重载 | 完美转发 `std::forward` | C++11 |
| 变参模板 | 不支持 | `variadic template` | C++11 |
| 惰性管线 | `for`+`if` | `std::ranges` | C++20 |
| 编译单元 | `#include` | `import` 模块 | C++20 |
| 模板约束 | 无 | `concept`/`requires` | C++20 |

---

## 四、古代 C++ 特征

### 4.1 手动内存管理
```cpp
Person* p = new Person("Alice", 30);
p->greet();
delete p;   // 遗漏即内存泄漏；异常路径更易泄漏
```
资源谁申请谁释放，全靠自觉。

### 4.2 无智能指针
`auto_ptr` 存在严重缺陷（拷贝时转移所有权，语义诡异），基本弃用；`unique_ptr`/`shared_ptr` 需等到 C++11。

### 4.3 无移动语义
对象传递一律深拷贝；传参、返回大对象时性能损耗大，只能靠传引用/指针规避。

### 4.4 无 `auto`
```cpp
std::vector<int>::iterator it = vec.begin();  // 类型必须写全
```

### 4.5 无范围 `for`
```cpp
for (std::vector<int>::iterator it = vec.begin(); it != vec.end(); ++it) {
    std::cout << *it << std::endl;
}
```

### 4.6 无 `lambda`
只能写函数对象（`functor`）或独立函数，配合 STL 算法时代码冗长。

### 4.7 `NULL` / `0` 而非 `nullptr`
`NULL` 本质为 `0`（int），在重载场景下易产生歧义。

### 4.8 无右值引用 / 完美转发 / 变参模板
模板元编程成本高，泛型代码表达力受限。

### 4.9 异常安全靠手动保证
缺乏 RAII 普及，资源泄漏常见。

### 4.10 头文件与宏主导
`#define` 大量使用，文本替换带来的调试困难与命名污染明显。

---

## 五、现代 C++ 特征

### 5.1 智能指针
```cpp
auto p = std::make_unique<Person>("Alice", 30);  // 离开作用域自动释放
```
- `unique_ptr`：独占所有权，零开销。
- `shared_ptr`：引用计数共享所有权。
- `weak_ptr`：不增加强引用，打破循环引用。

### 5.2 移动语义
```cpp
std::string s = "hello";
std::string t = std::move(s);  // 资源直接转移，不拷贝
```

### 5.3 `auto` 类型推导
```cpp
auto x  = 42;          // int
auto it = vec.begin(); // 迭代器类型自动推导
```

### 5.4 范围 `for`
```cpp
for (const auto& item : container) { /* 简洁、安全、不易越界 */ }
```

### 5.5 `lambda` 表达式
```cpp
std::sort(vec.begin(), vec.end(),
          [](int a, int b) { return a > b; });
```

### 5.6 `nullptr`
```cpp
Person* p = nullptr;  // 类型安全，不再用 NULL
```

### 5.7 `constexpr` 与编译期计算
```cpp
constexpr int square(int x) { return x * x; }
int arr[square(5)];   // 编译期得出 25
```

### 5.8 右值引用与完美转发
```cpp
template <typename T>
void wrapper(T&& arg) {
    func(std::forward<T>(arg));  // 保持实参的值类别
}
```

### 5.9 RAII 与异常安全
资源获取即初始化，析构函数自动释放，天然异常安全。

### 5.10 Ranges（C++20）
```cpp
#include <ranges>
auto result = vec
    | std::views::filter([](int x) { return x % 2 == 0; })
    | std::views::transform([](int x) { return x * x; });
```
惰性、非拥有视图，管道式写法接近函数式风格。

### 5.11 模块（C++20）
```cpp
import <iostream>;  // 替代 #include，编译更快、无宏污染
```

### 5.12 概念（C++20）
```cpp
template <std::integral T>
T add(T a, T b) { return a + b; }
```
模板参数显式约束，报错信息大幅友好。

---

## 六、同一功能的两种写法

**需求**：将 `vector` 中的偶数平方后输出。

**古代（C++98 风格）**
```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec;
    vec.push_back(1); vec.push_back(2);
    vec.push_back(3); vec.push_back(4);

    for (std::vector<int>::iterator it = vec.begin(); it != vec.end(); ++it) {
        if (*it % 2 == 0) {
            std::cout << (*it) * (*it) << std::endl;
        }
    }
    return 0;
}
```

**现代（C++20 风格）**
```cpp
#include <iostream>
#include <vector>
#include <ranges>

int main() {
    std::vector<int> vec = {1, 2, 3, 4};

    auto result = vec
        | std::views::filter([](int x) { return x % 2 == 0; })
        | std::views::transform([](int x) { return x * x; });

    for (int x : result) {
        std::cout << x << '\n';
    }
    return 0;
}
```
同样逻辑，现代写法去除迭代器样板、显式循环与手动解引用，意图更清晰。

---

## 七、底层原理

### 7.1 移动语义
- **值类别**：表达式分左值（lvalue）、右值；C++11 将右值拆为纯右值（prvalue）与将亡值（xvalue）。
- **右值引用 `T&&`**：只能绑定到右值，用于识别"可被窃取"的临时对象。
- **`std::move`**：仅做无条件右值转换（cast），本身不移动任何数据。
- **移动构造 / 移动赋值**：把源对象的内部资源指针"偷"走并将源置空，复杂度 **O(1)**，取代深拷贝 **O(n)**。
- 容器扩容、`return` 局部对象等场景自动利用移动，性能显著提升。

### 7.2 RAII 与异常安全
- **RAII（Resource Acquisition Is Initialization）**：资源生命周期绑定到栈对象生命周期。
- 栈对象无论正常退出还是因异常退出（栈展开 stack unwinding），析构函数都必然执行。
- 文件句柄、锁、堆内存等被包装为对象后无需手动释放。
- 智能指针即 RAII 在堆内存管理上的直接应用。

### 7.3 智能指针机制
- **`unique_ptr`**：独占所有权，无引用计数，零额外开销，所有权通过移动转移。
- **`shared_ptr`**：维护控制块与引用计数（强引用 `use_count`、弱引用 `weak_count`），计数归零才释放对象。
- **`weak_ptr`**：不增加强引用，专用于打破 `shared_ptr` 循环引用。
- **`make_shared` / `make_unique`**：单次分配 + 异常安全，优于裸 `new`。

### 7.4 `nullptr`
- `NULL` 是宏，通常定义为 `0`，类型为 `int`，在重载 `f(int)` 与 `f(char*)` 时产生歧义。
- `nullptr` 类型为 `std::nullptr_t`，可隐式转为任意指针类型，但**不能**转为 `int`，重载安全。

### 7.5 `auto` 与类型推导
- `auto` 采用模板实参推导规则。
- 注意 `auto` 会**退化**：顶层 `const` 与引用被剥离，需 `auto&` / `const auto&` 保留。

### 7.6 `lambda`
- 编译器为 `lambda` 生成一个匿名闭包类（functor），并重载 `operator()`。
- 捕获列表决定闭包内成员是拷贝还是引用捕获。
- 无捕获的 `lambda` 可隐式转换为函数指针。

### 7.7 `constexpr`
- `constexpr` 函数 / 变量要求在编译期可求值，将运行时开销前移到编译期。
- C++14/17 逐步放宽（允许局部变量、循环），可用于非类型模板参数、数组大小等。

### 7.8 Ranges（C++20）
- **视图（view）** 是惰性、非拥有的适配器，不拷贝元素。
- 管道 `|` 组合多个适配器，避免产生中间容器，接近函数式风格。

### 7.9 模块（C++20）
- 替代 `#include` 的文本包含模型。
- 解决三大问题：编译时间随头文件规模线性增长、宏文本污染、同一头文件被重复解析。
- `import` 导入的是已编译的模块接口单元，而非文本拷贝。

### 7.10 概念（C++20）
- 通过 `concept` / `requires` 对模板参数施加编译期约束。
- 将模板错误信息从"内部实现崩溃"前置为"约束不满足"，可读性大幅提升。
- 在表达力上取代了 `SFINAE` 的复杂写法。

---

## 八、为什么古代写法依然存在

1. **向后兼容**：C++ 长期承诺兼容旧代码，古代写法至今仍可编译，但已不推荐。
2. **教材滞后**：部分教材仍讲授 C++98 风格，初学者易先入为主。
3. **存量代码**：大量老项目与第三方库为古代风格，维护时无法回避。
4. **渐进迁移**：团队升级标准版本往往分阶段进行，混用现象普遍。

---

## 九、规范与延伸

- **C++ Core Guidelines**：由 Bjarne Stroustrup 与 Herb Sutter 主导的现代 C++ 实践准则，是采用现代写法的权威参考。
- **标准演进路径**：C++11 → 14 → 17 → 20 → 23，建议以 C++17 为最低实用基线，新项目优先采用 C++20 的 `ranges` / 模块 / 概念。
- **实践建议**：默认使用智能指针与 `auto`，优先范围 `for` 与 `lambda` + 算法；仅在确有性能瓶颈或接口约束需要时引入模板元编程与底层内存操作。
