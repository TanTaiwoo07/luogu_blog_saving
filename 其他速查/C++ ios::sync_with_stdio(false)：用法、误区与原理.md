# C++ `ios::sync_with_stdio(false)`：用法、误区与原理

## 一句话结论

`std::ios::sync_with_stdio(false)` **解除 C++ 标准流（`cin`/`cout`/…）与 C 标准流（`stdin`/`stdout`/…）的同步**，让前者使用自己的缓冲区，从而显著提速；代价是**不能再把 `cin`/`cout` 与 `scanf`/`printf` 混用**。它**不是**"取消缓冲区"。

## 目录

* [一句话结论](#一句话结论)
* [1 基本用法](#1-基本用法)
  * [1.1 最小骨架](#11-最小骨架)
  * [1.2 返回值](#12-返回值)
* [2 常见搭配](#2-常见搭配)
  * [2.1 `cin.tie(nullptr)` 解除绑定](#21-cintienullptr-解除绑定)
  * [2.2 用 `'\n'` 代替 `endl`](#22-用-n-代替-endl)
  * [2.3 完整模板](#23-完整模板)
* [3 性能差距有多大](#3-性能差距有多大)
* [4 注意事项与常见误区](#4-注意事项与常见误区)
  * [4.1 它不是取消缓冲区](#41-它不是取消缓冲区)
  * [4.2 解除同步后不能与 `scanf`/`printf` 混用](#42-解除同步后不能与-scanfprintf-混用)
  * [4.3 `scanf`/`printf` 的头文件怎么写](#43-scanfprintf-的头文件怎么写)
  * [4.4 调用时机](#44-调用时机)
  * [4.5 交互式题目与需要实时输出的场景](#45-交互式题目与需要实时输出的场景)
  * [4.6 多线程](#46-多线程)
  * [4.7 其它细节](#47-其它细节)
* [5 底层原理](#5-底层原理)
  * [5.1 同步开启时的语义与实现](#51-同步开启时的语义与实现)
  * [5.2 解除同步后快在哪](#52-解除同步后快在哪)
  * [5.3 `tie` 的作用机制](#53-tie-的作用机制)
* [6 速查表](#6-速查表)

## 1 基本用法

### 1.1 最小骨架

结论：在 `main` 开头、任何读写之前调用一次即可，全程序生效。

    #include <iostream>
    
    int main() {
        std::ios::sync_with_stdio(false);   // 解除 C / C++ 标准流的同步
    
        int x;
        std::cin >> x;
        std::cout << x << '\n';
        return 0;
    }

### 1.2 返回值

结论：返回**调用前的同步状态**（默认是 `true`），一般不用关心。

    #include <iostream>
    
    int main() {
        bool before = std::ios::sync_with_stdio(false);  // before == true（默认同步）
        std::cout << std::boolalpha << before << '\n';   // true
        return 0;
    }

## 2 常见搭配

### 2.1 `cin.tie(nullptr)` 解除绑定

结论：默认 `cin` 与 `cout` 绑定，每次从 `cin` 读取前会先 **flush `cout`**；解除绑定可去掉这批强制刷新，在"读入为主"的场景收益明显。

    #include <iostream>
    
    int main() {
        std::ios::sync_with_stdio(false);
        std::cin.tie(nullptr);          // cin 不再自动 flush cout
    
        int n;
        std::cin >> n;
        std::cout << n << '\n';
        return 0;
    }

恢复绑定只需传回指针：`std::cin.tie(&std::cout);`。

### 2.2 用 `'\n'` 代替 `endl`

结论：`std::endl` 等价于"输出 `'\n'` **并立即 flush**"，等于每写一行一次系统调用；竞赛/批量输出一律用 `'\n'`。

| 写法  | 行为  | 代价  |
| --- | --- | --- |
| `std::cout << x << '\n';` | 写入缓冲区，缓冲满或程序结束时才真正写出 | 最低  |
| `std::cout << x << std::endl;` | 写 `'\n'` 后强制 flush | 每行一次 `write` 系统调用 |
| `std::cout << x << std::flush;` | 不写换行，立即 flush | 需要"立刻可见"时才用 |

    #include <iostream>
    
    int main() {
        std::ios::sync_with_stdio(false);
        std::cin.tie(nullptr);
    
        for (int i = 0; i < 3; ++i) {
            std::cout << i << '\n';           // 好：只进缓冲区
            // std::cout << i << std::endl;   // 差：每行一次 flush
        }
        return 0;
    }

### 2.3 完整模板

结论：以下三件套是标准起手式。

    #include <iostream>
    // 本方案下不要混用 scanf / printf
    
    int main() {
        std::ios::sync_with_stdio(false);
        std::cin.tie(nullptr);
    
        int n;
        std::cin >> n;
        long long sum = 0;
        for (int i = 0, x; i < n && std::cin >> x; ++i) sum += x;
        std::cout << sum << '\n';
        return 0;
    }

## 3 性能差距有多大

结论：量级上是 **2～5 倍**的差异，解除同步后通常与 `scanf`/`printf` 持平甚至更快。绝对数字取决于机器、编译器、标准库与输入形式，**需实测**。

| 方式  | 相对耗时（越小越好，仅示量级） | 说明  |
| --- | --- | --- |
| `cin`/`cout`，同步开启（默认） | 1.0（基准） | 每次读写都穿透到 C 层，等于无缓冲 |
| `cin`/`cout` + `sync_with_stdio(false)` | 约 0.2 – 0.4 | 自带独立缓冲，通常最快 |
| `scanf`/`printf` | 约 0.3 – 0.6 | 与上者大致同级，互有胜负 |
| 上述任意方式 + 每行 `endl` | 可能数倍至十倍 | `flush` 触发系统调用，抹掉缓冲收益 |

自测代码（`g++ -O2 a.cpp -o a && ./a < data.txt`，参数为任意 argv 即开启加速）：

    #include <chrono>
    #include <iostream>
    
    int main(int argc, char**) {
        bool fast = argc > 1;
        if (fast) { std::ios::sync_with_stdio(false); std::cin.tie(nullptr); }
    
        auto t0 = std::chrono::steady_clock::now();
        long long sum = 0;
        int x;
        while (std::cin >> x) sum += x;
        auto t1 = std::chrono::steady_clock::now();
    
        std::cout << "sum=" << sum << '\n';
        std::cerr << "elapsed=" 
                  << std::chrono::duration<double>(t1 - t0).count() << "s\n";
        return 0;
    }

补充判断：`10^5` 以内的数据规模通常不需要纠结；真正的瓶颈出现在 `10^6` 量级以上的读入或写出。若解除同步后仍不够，应换手写快读（`fread` 缓冲），而不是继续折腾这两个开关。

## 4 注意事项与常见误区

### 4.1 它不是取消缓冲区

结论：`sync_with_stdio(false)` 是**解除 C 与 C++ 两套流的同步关系**，使 C++ 流得以启用自己的缓冲区。它**没有**、也从未"取消缓冲区"。

反证：若 `cout` 真的变成无缓冲，"两段输出各自成块、次序颠倒"这一现象根本不会发生——恰恰因为 C++ 侧有了独立缓冲区，输出才可能被推迟到最后。

### 4.2 解除同步后不能与 `scanf`/`printf` 混用

结论：解除同步后两套流各有各的缓冲区，**交错顺序未定义（取决于实现）**。这是该函数唯一的硬性代价。

    #include <cstdio>
    #include <iostream>
    
    int main() {
        std::ios::sync_with_stdio(false);
        std::cout << "a\n";
        std::printf("b\n");
        std::cout << "c\n";
    }

可能输出：

    b
    a
    c

（该示例即 cppreference 给出的"可能输出"。）各流按自己的缓冲整块落盘，结果因实现而异；需要混用就**保持同步开启**。

实用的调试替代：用 `std::cerr`（默认 `unitbuf`，每次插入立即写出，通常直接写 fd 2），与 `cout` 是否同步无关。

### 4.3 `scanf`/`printf` 的头文件怎么写

结论：两者由 **`<cstdio>`**（C++ 风格，名字声明于 `std`，是否同时注入全局命名空间由实现决定）或 **`<stdio.h>`**（C 兼容，全局名）提供，**二者都合法，选一个并保持一致即可**。`<iostream>` 并不保证声明 `scanf`/`printf` —— 某些实现会间接包含，但那是实现细节，不可依赖。需要混用时两个头文件都要写。

    #include <cstdio>      // std::printf / std::scanf（多数实现亦可写 printf）
    #include <iostream>    // std::cin / std::cout
    
    int main() {
        std::ios::sync_with_stdio(false);
        std::cout << "C++ side\n";   // 解除同步后：不要与下面混用
        // std::printf("C side\n");  // ✗ 顺序未定义
        return 0;
    }

### 4.4 调用时机

结论：**必须在任何针对标准流的 I/O 之前调用**。原因在于实现往往在首次 I/O 时就已确定缓冲结构；若已有读写之后再改这个开关，标准明确规定其行为由实现定义（从"完全无效"到"丢弃读缓冲区"都有可能）。

    #include <iostream>
    
    int main() {
        // ✗ 错误示范：已经 I/O 过才解除同步
        int x;
        std::cin >> x;
        std::ios::sync_with_stdio(false);   // 行为由实现定义，不要这么写
    
        std::cout << x << '\n';
        return 0;
    }

### 4.5 交互式题目与需要实时输出的场景

结论：`cin.tie(nullptr)` **不适合**必须先输出、再等待对方应答的交互式评测；此时保留绑定，或手动 `flush`。否则提示语滞留在缓冲区，双方互等 → 超时。

    #include <iostream>
    
    int main() {
        std::ios::sync_with_stdio(false);
        // 不调用 cin.tie(nullptr)
    
        std::cout << "? " << 50 << std::endl;   // 关键：endl 立即 flush，对端此刻才看到提问
        int ans;
        std::cin >> ans;                        // 若提示语堵在缓冲区里，双方互等 → 超时
    
        std::cout << "! " << ans << std::endl;
        return 0;
    }

同理，写日志/打 watch 建议用 `cerr` 而非依赖即时刷新的 `cout`。

### 4.6 多线程

结论：同步开启时，这些标准流被明确保证**不产生数据竞争**（但多字符仍可能交错）；关闭同步后不要依赖这一点。需要"整段不串"的输出，用互斥锁或 C++20 的 `std::osyncstream`。

    #include <iostream>
    #include <syncstream>   // C++20
    #include <thread>
    
    void worker(int id) {
        std::osyncstream(std::cout) << "thread " << id << '\n';  // 整条语句原子输出
    }
    
    int main() {
        std::ios::sync_with_stdio(false);
        std::thread a(worker, 1), b(worker, 2);
        a.join(); b.join();
    }

### 4.7 其它细节

| 项   | 说明  |
| --- | --- |
| 作用范围 | 同时作用于 `cin`/`cout`/`cerr`/`clog` 及其宽字符版本共 8 个流，无法只对某一个生效 |
| 调用次数 | 进程级全局开关，调用一次即可；重复调用多次没有额外收益 |
| `cerr` | 默认 `unitbuf`，每次插入立即写出，`sync_with_stdio` 不改变这一点 |
| 程序异常终止 | 若 `_exit`、abort 或崩溃，`cout` 缓冲区内容不会写出 —— 解同步本身不背这个锅，缓冲式 I/O 皆如此 |
| 名称写法 | `std::ios` 是 `std::basic_ios<char>` 的别名，写 `std::ios_base::sync_with_stdio(false)` 完全等价 |

## 5 底层原理

以下是对"为什么慢 / 为什么快"的机制解释，不影响上面的用法结论。

### 5.1 同步开启时的语义与实现

标准规定，当标准流 `str` 与 C 流 `f` 同步时，以下三对操作具有**完全相同的效果**：

1. `std::fputc(f, c)` ≡ `str.rdbuf()->sputc(c)`
2. `std::fgetc(f)` ≡ `str.rdbuf()->sbumpc()`
3. `std::ungetc(c, f)` ≡ `str.rdbuf()->sputbackc(c)`

实现层面（libstdc++ / libc++ 的典型做法）：`cout` 的 `streambuf` 是一个**转发型**缓冲区，自身不持有数据，每个字符/Token 都直接交给 `stdout` 的 `FILE*`。于是两套流共享同一份 C 层缓冲，混用顺序天然正确；代价是：

* 每个字符都要多跨一层 `FILE*` 调用，无法在 C++ 侧做整块读写；
* 为满足语义必须放弃 C++ 侧缓冲，效果等同于 unbuffered；
* 额外承担一次锁/线程安全保证（见 4.6）。

### 5.2 解除同步后快在哪

关闭同步后，上等价关系不再成立，`cin`/`cout` 换用**自带缓冲区的 `filebuf`**（通常 8 KB 量级），直接与文件描述符交互，不再理会 `FILE*` 的状态。收益来自三处：

1. **系统调用次数从"每字符"降到"每缓冲块"**，这是主要来源；
2. 去掉了对 `FILE*` 一致性检查与管理的开销（含 `ungetc`/推回状态维护）；
3. 字符分类与数值转换（`num_get`）可在连续内存上批量进行。

一句话：**不是"省掉了缓冲"，而是"终于有了自己的缓冲，且不必再跟 C 层对齐"。**

### 5.3 `tie` 的作用机制

`basic_ios::tie()` 持有一个输出流指针，`cin.tie()` 默认指向 `&cout`。`basic_istream` 在构造 sentry（每次格式化/非格式化输入都会构造）时，对被绑定的流调用一次 `flush()`。

* 交互式读取：这条自动 flush 保证了"提示语先于输入可见"，是有用的；
* 批量读入：`1e6` 次读取 = `1e6` 次 `cout.flush()`，纯属浪费 —— `cin.tie(nullptr)` 就是砍掉它。

补充一点：**只有在读写交替发生时它才有用**。纯读入场景下解除绑定没有任何正确性风险——程序结束时 `cout` 仍会被 `std::ios_base::Init` 的析构函数 flush。

## 6 速查表

| 需求  | 做法  |
| --- | --- |
| 读入量大、时间卡紧 | `std::ios::sync_with_stdio(false); std::cin.tie(nullptr);` |
| 输出换行 | 用 `'\n'`，不要用 `std::endl` |
| 想用 `scanf`/`printf` | 保持同步开启（即不要调用本函数） |
| 交互式/需立即看到输出 | 保留 `cin` 与 `cout` 的绑定，或显式 `<< std::flush` |
| 多线程输出 | `std::mutex` 或 C++20 `std::osyncstream` |
| 头文件 | `cin`/`cout` → `<iostream>`；`scanf`/`printf` → `<cstdio>` 或 `<stdio.h>` |
| 还不够快 | 手写 `fread` 快读，而不是继续折腾这两个开关 |
