# C++ 常用排序算法

本文按由浅入深、由简单到复杂的顺序，系统梳理 C++ 中常用的十种排序算法与标准库 `std::sort`，涵盖核心思想、执行过程、可编译代码、复杂度、稳定性、适用场景与优化点。各算法的示例代码均为可直接编译运行的独立源文件（语言标准 C++14 及以上，已通过 MSVC `/std:c++14 /W3` 零警告编译与运行验证）；文中少量标注为「片段」的代码块为局部函数，需嵌入对应实现中使用。

* * *

## 目录

* [0. 预备知识与评价指标](#0-预备知识与评价指标)
* [1. 冒泡排序 Bubble Sort](#1-冒泡排序-bubble-sort)
* [2. 选择排序 Selection Sort](#2-选择排序-selection-sort)
* [3. 插入排序 Insertion Sort](#3-插入排序-insertion-sort)
* [4. 希尔排序 Shell Sort](#4-希尔排序-shell-sort)
* [5. 归并排序 Merge Sort](#5-归并排序-merge-sort)
* [6. 快速排序 Quick Sort](#6-快速排序-quick-sort)
* [7. 堆排序 Heap Sort](#7-堆排序-heap-sort)
* [8. 计数排序 Counting Sort](#8-计数排序-counting-sort)
* [9. 桶排序 Bucket Sort](#9-桶排序-bucket-sort)
* [10. 基数排序 Radix Sort](#10-基数排序-radix-sort)
* [11. std::sort 的实现思路与使用要点](#11-stdsort-的实现思路与使用要点)
* [12. 原理与复杂度总结](#12-原理与复杂度总结)

* * *

## 0. 预备知识与评价指标

* **稳定性**：若相等元素在排序后的相对次序与排序前一致，则称该算法稳定。稳定性在多关键字排序（先按 A 排、再按 B 排）与结构体记录排序中至关重要。
* **原地性**：额外空间为 O(1) 时为原地排序（in-place）；需要 O(n) 级辅助数组时为非原地。
* **比较排序下界**：基于比较的排序，决策树模型给出最坏与平均复杂度的理论下界 Ω(n log n)；计数、桶、基数排序通过利用值域信息突破该下界。
* **记号约定**：n 为元素个数，k 为值域大小（max − min + 1），d 为关键字位数，r 为基数（进制），B 为桶数。
* **评价维度**：时间复杂度（最好 / 平均 / 最坏）、空间复杂度、稳定性、缓存局部性、比较与移动代价、对数据分布（有序度、重复度、均匀性）的敏感度。

### 0.1 总览对照表

| 算法  | 最好  | 平均  | 最坏  | 空间  | 稳定  | 类型  |
| --- | --- | --- | --- | --- | --- | --- |
| 冒泡排序 | O(n) | O(n²) | O(n²) | O(1) | 稳定  | 比较 · 原地 |
| 选择排序 | O(n²) | O(n²) | O(n²) | O(1) | 不稳定 | 比较 · 原地 |
| 插入排序 | O(n) | O(n²) | O(n²) | O(1) | 稳定  | 比较 · 原地 |
| 希尔排序 | O(n log n) | 依赖 gap 序列 | O(n²) ~ O(n^1.5) | O(1) | 不稳定 | 比较 · 原地 |
| 归并排序 | O(n log n) | O(n log n) | O(n log n) | O(n) | 稳定  | 比较 · 非原地 |
| 快速排序 | O(n log n) | O(n log n) | O(n²) | O(log n) | 不稳定 | 比较 · 原地 |
| 堆排序 | O(n log n) | O(n log n) | O(n log n) | O(1) | 不稳定 | 比较 · 原地 |
| 计数排序 | O(n + k) | O(n + k) | O(n + k) | O(n + k) | 稳定  | 非比较 · 非原地 |
| 桶排序 | O(n) | O(n + B) | O(n²) | O(n + B) | 取决于桶内算法 | 非比较 · 非原地 |
| 基数排序 | O(d(n + r)) | O(d(n + r)) | O(d(n + r)) | O(n + r) | 稳定  | 非比较 · 非原地 |
| `std::sort` | O(n log n) | O(n log n) | O(n log n) | O(log n) | 不稳定 | 比较 · 原地 |

* * *

## 1. 冒泡排序 Bubble Sort

### 1.1 核心思想

重复扫描待排序区间，比较相邻两元素，逆序则交换。每一轮扫描会把当前未排序区间的最大值「浮」到区间末端，从而确定一个元素的最终位置。

### 1.2 执行过程

1. 设未排序区间为 `[0, n - i)`，初始 `i = 0`。
2. 从左到右比较 `a[j]` 与 `a[j + 1]`，若 `a[j] > a[j + 1]` 则交换。
3. 一轮结束后，区间最大值就位，`i` 递增。
4. 若某轮未发生任何交换，说明序列已整体有序，可提前终止。
5. 记录本轮最后一次交换位置 `lastSwap`，其后元素已就位，下一轮可直接把右边界收缩到 `lastSwap`，避免无效比较。

### 1.3 代码实现

    #include <algorithm>
    #include <cstddef>
    #include <iostream>
    #include <vector>
    
    // 冒泡排序：提前退出 + 边界收缩优化
    void bubbleSort(std::vector<int>& a) {
        std::size_t n = a.size();
        std::size_t right = n;                       // 未排序区间右边界（开区间）
        while (right > 1) {
            std::size_t lastSwap = 0;                // 本轮最后一次交换的位置
            for (std::size_t j = 1; j < right; ++j) {
                if (a[j] < a[j - 1]) {               // 严格小于：相等不交换，保证稳定
                    std::swap(a[j], a[j - 1]);
                    lastSwap = j;
                }
            }
            if (lastSwap == 0) return;               // 本轮无交换，序列已有序
            right = lastSwap;                        // lastSwap 之后已就位，收缩右边界
        }
    }
    
    int main() {
        std::vector<int> a = {5, 2, 9, 1, 5, 6};
        bubbleSort(a);
        for (int x : a) std::cout << x << ' ';
        std::cout << '\n';
        return 0;
    }

### 1.4 复杂度与稳定性

| 指标  | 结论  |
| --- | --- |
| 时间复杂度 | 最好 O(n)（已有序，一轮即退出）；平均与最坏 O(n²) |
| 空间复杂度 | O(1)，原地 |
| 稳定性 | 稳定（仅在严格逆序时交换） |
| 比较次数 | 最坏 n(n − 1)/2；交换次数与逆序对数相等 |
| 代价特征 | 交换次数远高于插入排序（每交换只消除一个逆序对） |

### 1.5 适用场景

* 教学演示、规模极小（n ≤ 100）且对代码简洁度要求高的场合。
* 数据近乎有序时，配合提前退出可接近 O(n)。
* 链表结构下相邻交换实现简单，但仍不推荐用于性能敏感路径。

### 1.6 常见优化点

1. **提前退出标志**：`swapped == false` 时直接返回，使最好情况降为 O(n)。
2. **边界收缩**：用 `lastSwap` 替代固定减一，跳过尾部已就位的有序段。
3. **鸡尾酒排序（双向冒泡）**：交替正向、反向扫描，可同时把最小值推向头部，缓解「小值在尾部」（乌龟）问题；最坏仍为 O(n²)。
4. **本质替代**：工程中直接用插入排序替换冒泡排序，二者同为 O(n²) 稳定原地算法，但插入排序的常数因子显著更小。

* * *

## 2. 选择排序 Selection Sort

### 2.1 核心思想

每一轮在未排序区间中线性扫描出最小元素，将其与区间首元素交换，从而确定一个最终位置。与冒泡排序的区别在于：冒泡通过频繁交换逐步推进，选择排序每轮只做一次交换，比较次数固定但交换次数最少。

### 2.2 执行过程

1. 未排序区间为 `[i, n)`，初始 `i = 0`。
2. 扫描 `[i + 1, n)` 找到最小元素下标 `minIdx`。
3. 交换 `a[i]` 与 `a[minIdx]`，`i` 递增。
4. 重复至 `i == n - 1`。

### 2.3 代码实现

    #include <algorithm>
    #include <cstddef>
    #include <iostream>
    #include <vector>
    
    // 选择排序：每轮选最小的放到未排序区间首位
    void selectionSort(std::vector<int>& a) {
        std::size_t n = a.size();
        for (std::size_t i = 0; i + 1 < n; ++i) {
            std::size_t minIdx = i;
            for (std::size_t j = i + 1; j < n; ++j) {
                if (a[j] < a[minIdx]) minIdx = j;
            }
            if (minIdx != i) std::swap(a[i], a[minIdx]);
        }
    }
    
    int main() {
        std::vector<int> a = {5, 2, 9, 1, 5, 6};
        selectionSort(a);
        for (int x : a) std::cout << x << ' ';
        std::cout << '\n';
        return 0;
    }

### 2.4 复杂度与稳定性

| 指标  | 结论  |
| --- | --- |
| 时间复杂度 | 最好 / 平均 / 最坏均为 O(n²)；比较次数固定为 n(n − 1)/2 |
| 空间复杂度 | O(1)，原地 |
| 稳定性 | **不稳定**：跨元素交换会破坏相等元素的相对次序，例如 `[5a, 3, 5b]` 中 `3` 与 `5a` 交换后 `5a` 落到 `5b` 之后 |
| 交换次数 | 最多 n − 1 次，是所有 O(n²) 算法中最少的 |

### 2.5 适用场景

* 元素较大、移动（写）代价高而比较代价低的场景：交换次数 O(n) 是决定性优势。
* `n` 极小且追求实现最简。
* 需要严格限制写次数的介质（如 flash 存储上的原地排序）。

### 2.6 常见优化点

1. **双向选择排序**：每轮同时扫描最小与最大值，分别放到区间首尾，轮数减半；比较次数仍为 O(n²)，但可省去一半轮次开销。注意当最大值恰好位于 `i` 位置时的二次修正。
2. **稳定化改造**：用「区间整体右移一位」替代交换，可得到稳定版本，但移动次数上升为 O(n²)。
3. **堆化升级**：把「线性扫描找极值」替换为「堆结构取极值」，即演化为堆排序，复杂度由 O(n²) 降至 O(n log n)。

* * *

## 3. 插入排序 Insertion Sort

### 3.1 核心思想

将序列视为「已排序前缀 + 未排序后缀」，逐个把后缀首元素插入到前缀中的正确位置，插入过程中把大于该元素的项整体右移腾出空位。过程类似整理扑克牌。

### 3.2 执行过程

1. 初始前缀为 `[0, 1)`（单元素必然有序）。
2. 取 `key = a[i]`（`i` 为后缀首位置）。
3. 从 `i - 1` 向前扫描，凡是 `> key` 的元素右移一位。
4. 遇到 `<= key` 的元素或到达边界即停止，把 `key` 落入空位。
5. `i` 递增，重复至后缀耗尽。

### 3.3 代码实现

    #include <cstddef>
    #include <iostream>
    #include <vector>
    
    // 插入排序：按 [l, r) 区间排序，便于被其他算法复用
    void insertionSort(std::vector<int>& a, std::size_t l, std::size_t r) {
        for (std::size_t i = l + 1; i < r; ++i) {
            int key = a[i];
            std::size_t j = i;
            while (j > l && key < a[j - 1]) {   // 严格小于，保证稳定
                a[j] = a[j - 1];
                --j;
            }
            a[j] = key;
        }
    }
    
    void insertionSort(std::vector<int>& a) { insertionSort(a, 0, a.size()); }
    
    int main() {
        std::vector<int> a = {5, 2, 9, 1, 5, 6};
        insertionSort(a);
        for (int x : a) std::cout << x << ' ';
        std::cout << '\n';
        return 0;
    }

### 3.4 复杂度与稳定性

| 指标  | 结论  |
| --- | --- |
| 时间复杂度 | 最好 O(n)（已有序，仅 n − 1 次比较、零移动）；平均与最坏 O(n²) |
| 空间复杂度 | O(1)，原地 |
| 稳定性 | 稳定（相等元素不越过） |
| 代价特征 | 比较与移动次数均等于逆序对数；常数因子小，缓存友好（顺序访问） |
| 在线性 | 支持在线排序：新元素到来时逐个插入即可 |

### 3.5 适用场景

* 小规模数据（n ≤ 50）：实际速度快于多数 O(n log n) 算法，因此被广泛用作高级排序的「小区间收尾」子过程。
* 近乎有序的数据：逆序对少时接近 O(n)，是自适应算法的代表。
* 需要在线、增量式维护有序序列的场景。

### 3.6 常见优化点

1. **二分插入排序**：用二分查找定位插入点，比较次数由 O(n²) 降至 O(n log n)；但元素移动仍为 O(n²)，总体仍属 O(n²)，仅在比较代价高昂时才有收益。
2. **哨兵技巧**：把区间最小值预置到 `a[0]` 作哨兵，内层循环去掉 `j > l` 边界判断。
3. **作为混合子过程**：快排、归并、introsort 均在小区间（如 ≤ 16 或 ≤ 32）切换为插入排序。
4. **链表适配**：链表上插入排序只需改指针、无须移动元素，是 `std::list::sort` 一类实现的基础思路之一。

* * *

## 4. 希尔排序 Shell Sort

### 4.1 核心思想

插入排序在数据近乎有序时效率极高。希尔排序先用较大间隔 `gap` 把序列分组，对每组做插入排序，使元素大跨度地向最终位置靠拢；随后逐步缩小 `gap`，最后一轮 `gap == 1` 时即退化为一次近乎有序的插入排序。这是首个突破 O(n²) 的比较排序算法。

### 4.2 执行过程

1. 选定递减的间隔序列，如 Knuth 序列 `gap = 1, 4, 13, 40, ...`（`gap = gap * 3 + 1`）。
2. 对每个 `gap`，把下标相差 `gap` 的元素视为一组（共 `gap` 组），组内执行插入排序。
3. 采用「按位置推进」的写法可避免显式分组：对 `i` 从 `gap` 到 `n − 1`，把 `a[i]` 插入到同组的有序前缀中。
4. `gap` 递减至 1，完成最后一轮插入排序后序列有序。

### 4.3 代码实现

    #include <cstddef>
    #include <iostream>
    #include <vector>
    
    // 希尔排序：Knuth 间隔序列 1, 4, 13, 40, ...
    void shellSort(std::vector<int>& a) {
        std::size_t n = a.size();
        std::size_t gap = 1;
        while (gap < n / 3) gap = gap * 3 + 1;      // 初始化为不超过 n/3 的最大 Knuth 间隔
        for (; gap > 0; gap /= 3) {
            for (std::size_t i = gap; i < n; ++i) { // 各 gap 组交替推进
                int key = a[i];
                std::size_t j = i;
                while (j >= gap && key < a[j - gap]) {
                    a[j] = a[j - gap];
                    j -= gap;
                }
                a[j] = key;
            }
        }
    }
    
    int main() {
        std::vector<int> a = {5, 2, 9, 1, 5, 6};
        shellSort(a);
        for (int x : a) std::cout << x << ' ';
        std::cout << '\n';
        return 0;
    }

### 4.4 复杂度与稳定性

| 指标  | 结论  |
| --- | --- |
| 时间复杂度 | 取决于 gap 序列：Shell 原始序列（折半）最坏 O(n²)；Hibbard 序列（`2^k − 1`）最坏 O(n^1.5)；Sedgewick 序列约 O(n^(4/3))；Knuth 序列实践表现良好，最坏上界 O(n^1.5) |
| 空间复杂度 | O(1)，原地 |
| 稳定性 | **不稳定**：跨 `gap` 的分组插入会使相等元素相对次序改变 |
| 优势  | 中等规模（数千至数万）数据上常数小、无递归栈、代码短 |
| 劣势  | 复杂度上界不明确、性能对 gap 序列与数据分布敏感，难以做严格保证 |

### 4.5 适用场景

* 嵌入式或内存受限环境：原地、无递归、无额外缓冲。
* 中等规模数据、不愿引入快排最坏情况风险，又无法承担归并 O(n) 辅助空间的场合。
* 作为理解「递减增量」思想的过渡算法。

### 4.6 常见优化点

1. **gap 序列选择**：优先使用 Sedgewick 序列（`1, 5, 19, 41, 109, ...`）或 Ciura 经验序列（`1, 4, 10, 23, 57, 132, 301, 701`），实测优于朴素折半序列。
2. **gap 互质**：保证相邻 gap 尽量互质，可让上一轮的局部有序被下一轮充分利用。
3. **移动替代交换**：内层用「赋值右移」而非逐次 `swap`，减少三分之二的写操作。
4. **规模阈值**：n 很大时不再适用，应切换至快排或归并。

* * *

## 5. 归并排序 Merge Sort

### 5.1 核心思想

典型分治：把序列二分递归排序，再把两个已有序的子序列线性合并为一个有序序列。合并的核心是两个指针分别扫描左右段，每次取较小者写入辅助数组。

### 5.2 执行过程

1. **分解**：递归把区间 `[l, r)` 二分至长度 ≤ 1（天然有序）。
2. **合并**：为 `[l, m)` 与 `[m, r)` 分别设置指针 `i`、`j`；比较 `a[i]` 与 `a[j]`，较小者写入缓冲区并推进对应指针；相等时优先取左段，保证稳定。
3. 某段耗尽后，把另一段剩余元素直接追加。
4. 把缓冲区内容拷回原区间。
5. 若已满足 `a[m - 1] <= a[m]`，说明跨段已有序，可跳过合并。

### 5.3 代码实现

    #include <cstddef>
    #include <iostream>
    #include <vector>
    
    void mergeSort(std::vector<int>& a) {
        std::vector<int> buf(a.size());                       // 复用同一缓冲区，避免反复分配
        auto sort = [&](auto&& self, std::size_t l, std::size_t r) -> void {
            if (r - l <= 1) return;
            std::size_t m = l + (r - l) / 2;
            self(self, l, m);
            self(self, m, r);
            if (a[m - 1] <= a[m]) return;                     // 左右段已整体有序，跳过合并
            std::size_t i = l, j = m, k = l;
            while (i < m && j < r) {
                buf[k++] = (a[i] <= a[j]) ? a[i++] : a[j++];  // 相等取左，保证稳定
            }
            while (i < m) buf[k++] = a[i++];
            while (j < r) buf[k++] = a[j++];
            for (std::size_t t = l; t < r; ++t) a[t] = buf[t];
        };
        sort(sort, 0, a.size());
    }
    
    int main() {
        std::vector<int> a = {5, 2, 9, 1, 5, 6};
        mergeSort(a);
        for (int x : a) std::cout << x << ' ';
        std::cout << '\n';
        return 0;
    }

### 5.4 复杂度与稳定性

| 指标  | 结论  |
| --- | --- |
| 时间复杂度 | 最好 / 平均 / 最坏均为 O(n log n)；递归式 T(n) = 2T(n/2) + O(n) |
| 空间复杂度 | O(n) 辅助数组 + O(log n) 递归栈 |
| 稳定性 | 稳定（合并时相等取左段） |
| 比较次数 | 最坏约 n log n − n + 1，接近理论下界 |
| 访问模式 | 顺序访问，缓存友好；适合外存（磁盘）多路归并 |

### 5.5 适用场景

* 要求稳定且需要 O(n log n) 最坏保证的通用排序（`std::stable_sort` 的基础）。
* 数据量超出内存的外排序：多路归并 + 缓冲区，顺序 I/O 效率远高于快排。
* 链表排序：归并只需改指针，无需随机访问，是链表排序的首选。

### 5.6 常见优化点

1. **缓冲区复用**：全程共用一块 O(n) 缓冲（或使用双缓冲交替拷贝），避免每层递归分配。
2. **小区间切换插入排序**：子区间长度 ≤ 16 时改用插入排序，减少递归与函数调用开销。
3. **有序段短路**：`a[m - 1] <= a[m]` 时跳过合并，对部分有序数据收益明显。
4. **自然归并排序**：先线性扫描识别已存在的单调段（run），再按 run 归并，可把部分有序输入降至 O(n)。
5. **自底向上迭代实现**：从宽度 1 开始倍增合并，消除递归、便于并行化；`std::stable_sort` 即采用此类思路。
6. **原地归并**：理论上可行但常数极大，工程中通常使用 O(n) 缓冲版本。

* * *

## 6. 快速排序 Quick Sort

### 6.1 核心思想

分治 + 划分：选取一个基准值 pivot，通过一次线性扫描把区间重排为「< pivot」「== pivot」「> pivot」三段，使得 pivot 的最终位置一次确定；随后递归处理左右两段。快排不产生合并步骤，因此是原地的，且常数因子极小。

### 6.2 执行过程

1. **选主元**：取首、中、尾三元素的中位数作为 pivot（三数取中），降低遇到极端划分的概率。
2. **划分（Lomuto 方案）**：把 pivot 交换到区间末位，用指针 `i` 维护「小于 pivot」段的右边界；扫描指针 `j` 遇到 `< pivot` 的元素即与 `a[i]` 交换并推进 `i`；扫描结束后把 pivot 换到位置 `i`。
3. **递归**：对 `[l, p)` 与 `[p + 1, r)` 递归。
4. **小区间收尾**：区间长度 ≤ 16 时改用插入排序。

### 6.3 代码实现

    #include <algorithm>
    #include <cstddef>
    #include <iostream>
    #include <vector>
    
    void insertionSort(std::vector<int>& a, std::size_t l, std::size_t r) {
        for (std::size_t i = l + 1; i < r; ++i) {
            int key = a[i];
            std::size_t j = i;
            while (j > l && key < a[j - 1]) { a[j] = a[j - 1]; --j; }
            a[j] = key;
        }
    }
    
    // 三数取中 + Lomuto 划分，返回 pivot 最终下标；区间为 [l, r)
    std::size_t partition(std::vector<int>& a, std::size_t l, std::size_t r) {
        std::size_t m = l + (r - l) / 2;
        if (a[m] < a[l]) std::swap(a[m], a[l]);
        if (a[r - 1] < a[l]) std::swap(a[r - 1], a[l]);
        if (a[r - 1] < a[m]) std::swap(a[r - 1], a[m]);
        std::swap(a[m], a[r - 1]);                       // pivot 置于末位
        int pivot = a[r - 1];
        std::size_t i = l;
        for (std::size_t j = l; j < r - 1; ++j) {
            if (a[j] < pivot) std::swap(a[i++], a[j]);
        }
        std::swap(a[i], a[r - 1]);
        return i;
    }
    
    void quickSort(std::vector<int>& a) {
        const std::size_t THRESHOLD = 16;
        auto sort = [&](auto&& self, std::size_t l, std::size_t r) -> void {
            while (r - l > THRESHOLD) {                  // 尾递归消除：仅递归较小区间
                std::size_t p = partition(a, l, r);
                if (p - l < r - (p + 1)) {
                    self(self, l, p);
                    l = p + 1;
                } else {
                    self(self, p + 1, r);
                    r = p;
                }
            }
        };
        sort(sort, 0, a.size());
        insertionSort(a, 0, a.size());                   // 各小块内部有序后的整体收尾
    }
    
    int main() {
        std::vector<int> a = {5, 2, 9, 1, 5, 6};
        quickSort(a);
        for (int x : a) std::cout << x << ' ';
        std::cout << '\n';
        return 0;
    }

### 6.4 复杂度与稳定性

| 指标  | 结论  |
| --- | --- |
| 时间复杂度 | 最好 / 平均 O(n log n)；最坏 O(n²)（每次划分极度失衡，如已有序数据取端点为 pivot） |
| 空间复杂度 | O(log n) 平均递归栈，最坏 O(n)；尾递归消除后可控在 O(log n) |
| 稳定性 | 不稳定：交换操作会跨越移动相等元素 |
| 常数因子 | 在所有 O(n log n) 比较排序中最小，是实践最快的通用排序内核 |
| 缓存行为 | 划分是顺序扫描，局部性好，优于堆排序 |

### 6.5 适用场景

* 大规模、随机分布的内存数据排序（默认首选）。
* 不需要稳定性、不要求最坏情况严格保证的场合。
* 可通过 `nth_element` / `partial_sort` 派生出 Top-K、分位数等问题。

### 6.6 常见优化点

1. **主元选取**：三数取中、随机化 pivot 或「九数取中（tukey's ninther）」，把最坏情况概率降到极低。
2. **三路划分**：面对大量重复元素，把区间分为 `<`、`==`、`>` 三段，重复元素一次到位，把 O(n log n) 的退化（全等元素下 Lomuto 划分退化为 O(n²)）消除。
3. **尾递归消除 + 先递归小区间**：把栈深度限制在 O(log n)。
4. **小区间切换插入排序**：阈值取 16 ~ 32。
5. **深度限制（Introsort）**：递归深度超过 `2 * log2(n)` 时切换为堆排序，把最坏复杂度锁死在 O(n log n)，这是 `std::sort` 的做法。

三路划分核心片段（局部函数，需替换上文 `quickSort` 中的划分与递归逻辑）：

    // 三路划分：返回 < 段与 == 段的边界，区间 [l, r)
    void quickSort3Way(std::vector<int>& a, std::size_t l, std::size_t r) {
        if (r - l <= 1) return;
        int pivot = a[l + (r - l) / 2];
        std::size_t lt = l, i = l, gt = r;              // a[l,lt) < pivot, a[lt,i) == pivot, a[gt,r) > pivot
        while (i < gt) {
            if (a[i] < pivot)      std::swap(a[lt++], a[i++]);
            else if (pivot < a[i]) std::swap(a[i], --gt);
            else                   ++i;
        }
        quickSort3Way(a, l, lt);
        quickSort3Way(a, gt, r);
    }

* * *

## 7. 堆排序 Heap Sort

### 7.1 核心思想

借助二叉堆（数组表示的完全二叉树）实现选择排序的升级版：先自底向上把数组建成大顶堆（O(n)），再重复「堆顶（最大值）与堆末元素交换 → 堆规模减一 → 堆顶下沉」，最终得到升序序列。用堆结构把「找极值」从 O(n) 降至 O(log n)。

### 7.2 执行过程

1. **建堆**：从最后一个非叶节点 `n/2 − 1` 起向前，对每个节点执行下沉（sift-down），整体代价 O(n)。
2. **排序**：堆顶 `a[0]` 为当前最大值，与 `a[end − 1]` 交换，最大值就位。
3. **下沉**：堆规模减为 `end − 1`，对新堆顶执行 sift-down 恢复堆性质。
4. 重复至堆规模为 1。

### 7.3 代码实现

    #include <algorithm>
    #include <cstddef>
    #include <iostream>
    #include <vector>
    
    // 下沉：维护大顶堆性质，堆区间为 [root, end)
    void siftDown(std::vector<int>& a, std::size_t root, std::size_t end) {
        while (true) {
            std::size_t child = 2 * root + 1;
            if (child >= end) break;
            if (child + 1 < end && a[child] < a[child + 1]) ++child;  // 取较大的子节点
            if (a[root] >= a[child]) break;
            std::swap(a[root], a[child]);
            root = child;
        }
    }
    
    void heapSort(std::vector<int>& a) {
        std::size_t n = a.size();
        if (n < 2) return;
        for (std::size_t i = n / 2; i > 0; --i) siftDown(a, i - 1, n);  // Floyd 建堆 O(n)
        for (std::size_t end = n; end > 1; --end) {
            std::swap(a[0], a[end - 1]);                                // 最大值就位
            siftDown(a, 0, end - 1);
        }
    }
    
    int main() {
        std::vector<int> a = {5, 2, 9, 1, 5, 6};
        heapSort(a);
        for (int x : a) std::cout << x << ' ';
        std::cout << '\n';
        return 0;
    }

### 7.4 复杂度与稳定性

| 指标  | 结论  |
| --- | --- |
| 时间复杂度 | 建堆 O(n)（Σ 各层节点数 × 层高收敛）；排序 n − 1 次下沉，每次 O(log n)；总计最好 / 平均 / 最坏均为 O(n log n) |
| 空间复杂度 | O(1)，原地，无递归栈 |
| 稳定性 | 不稳定：堆顶与堆末的跨距离交换破坏相等元素次序 |
| 最坏保证 | 严格 O(n log n)，无退化风险，是 O(1) 空间下唯一具备该保证的比较排序 |
| 劣势  | 数组下标按 2 倍跳跃访问，缓存局部性差，常数因子大于快排 |

### 7.5 适用场景

* 需要 O(n log n) 最坏保证且不允许 O(n) 额外空间的场合。
* 内存极度受限（嵌入式、内核态）的实时系统。
* 只需 Top-K 元素：不必全排序，维护一个大小为 K 的小顶堆即可，复杂度 O(n log K)。
* 作为 Introsort 的兜底算法（`std::sort` 的退化保护）。

### 7.6 常见优化点

1. **Floyd 建堆**：自底向上而非逐个插入建堆，把建堆由 O(n log n) 降到 O(n)。
2. **挖洞法（hole）下沉**：先取出堆顶值保存为临时变量，子节点上移，最后一次性落位，把每次下沉的交换次数由 2h 次赋值降为 h + 1 次。
3. **d 叉堆**：增大分叉数可降低树高、改善局部性，但下沉时比较次数上升，通常取 4 叉有轻微收益。
4. **Bottom-up heapsort（Weak heapsort 思路）**：先沿最长路径探底再自底向上回插，比较次数降至 n log n + O(n)。
5. **避免全排序**：只需前 K 个极值时，用堆选择替代堆排序。

* * *

## 8. 计数排序 Counting Sort

### 8.1 核心思想

非比较排序。统计每个取值出现的次数，再对计数数组求前缀和得到每个元素在输出中的起始位置，最后按前缀和把元素直接放置到输出数组。要求键值为范围有限的小整数（或非负整数映射后的下标）。

### 8.2 执行过程

1. 确定值域 `[minVal, maxVal]`，令 `k = maxVal − minVal + 1`。
2. 计数：`cnt[x − minVal]++`。
3. 前缀和：`cnt[i] += cnt[i − 1]`，此时 `cnt[d]` 表示取值 `d` 的最后一个元素在输出中的下标 + 1。
4. **逆序**遍历原数组放置元素：`out[--cnt[d]] = x`，逆序是保持稳定性的关键。
5. 输出数组即为结果。

### 8.3 代码实现

    #include <cstddef>
    #include <iostream>
    #include <vector>
    
    // 计数排序：要求 a 中元素落在 [minVal, maxVal]，返回新数组（稳定）
    std::vector<int> countingSort(const std::vector<int>& a, int minVal, int maxVal) {
        int k = maxVal - minVal + 1;
        std::vector<int> cnt(k, 0);
        std::vector<int> out(a.size());
        for (int x : a) ++cnt[x - minVal];
        for (int i = 1; i < k; ++i) cnt[i] += cnt[i - 1];          // 前缀和
        for (std::size_t i = a.size(); i-- > 0;) {                 // 逆序填充，保证稳定
            int x = a[i];
            out[--cnt[x - minVal]] = x;
        }
        return out;
    }
    
    int main() {
        std::vector<int> a = {5, 2, 9, 1, 5, 6};
        std::vector<int> b = countingSort(a, 1, 9);
        for (int x : b) std::cout << x << ' ';
        std::cout << '\n';
        return 0;
    }

### 8.4 复杂度与稳定性

| 指标  | 结论  |
| --- | --- |
| 时间复杂度 | O(n + k)，最好 / 平均 / 最坏一致；当 k = O(n) 时为线性 |
| 空间复杂度 | O(n + k)（输出数组 + 计数数组） |
| 稳定性 | 稳定（前缀和 + 逆序填充） |
| 前提约束 | 键值必须能映射到紧凑整数区间；k 过大时空间与初始化代价不可接受 |
| 突破下界 | 非比较排序，不受 Ω(n log n) 约束 |

### 8.5 适用场景

* 值域远小于 n 的整数排序：成绩（0 ~ 100）、年龄、等级、枚举值。
* 作为基数排序的稳定子过程（按单个字节 / 十进制位统计）。
* 需要统计频次与排序结果同时产出的场合。

### 8.6 常见优化点

1. **值域压缩**：先扫描求 `minVal` 与 `maxVal`，避免从 0 开始开辟无用的计数空间。
2. **原地版本**：把计数结果写回原数组可省去输出数组，但会丧失稳定性。
3. **分块计数**：k 极大而 n 较小时，先做值域分块（本质是桶排序思路），避免 O(k) 初始化成为瓶颈。
4. **并行化**：计数阶段与前缀和阶段可用并行前缀和（scan）加速。

* * *

## 9. 桶排序 Bucket Sort

### 9.1 核心思想

非比较排序。假设输入在值域上近似均匀分布，把值域均分为 B 个桶，按映射函数把元素投入对应桶，对每个非空桶内部排序（通常用插入排序或 `std::sort`），最后按桶序依次拼接。其性能完全依赖于「元素在各桶中均匀分布」这一前提。

### 9.2 执行过程

1. 确定桶数 B，构造映射 `bucketIndex = (x − minVal) / ((maxVal − minVal) / B)`（浮点可简化为 `x * B`）。
2. 遍历元素，投入对应桶。
3. 对每个非空桶排序（元素少时插入排序即可）。
4. 按桶编号从小到大顺序拼接输出。

### 9.3 代码实现

    #include <algorithm>
    #include <cstddef>
    #include <iostream>
    #include <vector>
    
    // 桶排序：输入为 [0, 1) 区间内近似均匀分布的浮点数
    void bucketSort(std::vector<double>& a, std::size_t bucketCount) {
        if (bucketCount == 0) return;
        std::vector<std::vector<double>> buckets(bucketCount);
        for (double x : a) {
            std::size_t idx = static_cast<std::size_t>(x * bucketCount);
            if (idx >= bucketCount) idx = bucketCount - 1;          // 边界保护
            buckets[idx].push_back(x);
        }
        for (std::vector<double>& b : buckets) std::sort(b.begin(), b.end());
        std::size_t k = 0;
        for (const std::vector<double>& b : buckets) {
            for (double x : b) a[k++] = x;
        }
    }
    
    int main() {
        std::vector<double> a = {0.42, 0.32, 0.33, 0.52, 0.37, 0.47, 0.51};
        bucketSort(a, 5);
        for (double x : a) std::cout << x << ' ';
        std::cout << '\n';
        return 0;
    }

### 9.4 复杂度与稳定性

| 指标  | 结论  |
| --- | --- |
| 时间复杂度 | 期望 O(n + B)：均匀分布下每桶约 n/B 个元素，桶内 `std::sort` 总代价 Σ O((n/B) log(n/B)) = O(n log(n/B))；取 B = Θ(n) 即 O(n)。最坏 O(n²)（全部落进同一桶，且桶内为 O(n²) 算法） |
| 空间复杂度 | O(n + B) |
| 稳定性 | 取决于桶内算法：桶内使用稳定排序（如插入排序、归并）且桶拼接保序，则整体稳定 |
| 前提  | 输入近似均匀分布；分布严重倾斜时退化为桶内排序的复杂度 |

### 9.5 适用场景

* 均匀分布的浮点数或已知分布的大规模数据（如 [0, 1) 的概率值、归一化指标）。
* 外部排序的预处理：按值域范围划分成可装入内存的块。
* 与计数排序组合：桶内再用计数排序处理密集小值域。

### 9.6 常见优化点

1. **桶数选择**：取 B ≈ n 使期望达到线性；B 过大增加空桶开销，过小则桶内代价上升。
2. **桶内算法自适应**：元素少（≤ 32）用插入排序，多则递归桶排序或快排。
3. **动态桶**：用 `vector` 链表式桶避免预分配浪费；对未知分布可采用按样本分位数划桶（sample sort）。
4. **分布检测**：先采样估计分布，若严重倾斜则改用比较排序，避免退化。

* * *

## 10. 基数排序 Radix Sort

### 10.1 核心思想

非比较排序。把整数关键字按进制拆分为若干「位」（十进制位、字节或若干比特组），从低位到高位（LSD，最低位优先）依次对每一位执行一次**稳定**排序；全部位处理完毕后序列有序。正确性依赖于「低位已排好时，对高位做稳定排序不会打乱低位的相对次序」。

### 10.2 执行过程

1. 求最大值，确定需要处理的最大位数 `d = floor(log_r(maxVal)) + 1`。
2. 对 `exp = 1, r, r², ...`：
  * 统计当前位各数字（0 ~ r − 1）的频数 `cnt[]`；
  * 求前缀和；
  * **逆序**遍历原数组，按 `digit = (x / exp) % r` 把元素放进输出数组 `out[--cnt[digit]]`；
  * 交换输入与输出角色。
3. 完成 d 轮后得到有序序列。

### 10.3 代码实现

    #include <algorithm>
    #include <cstddef>
    #include <iostream>
    #include <vector>
    
    // LSD 基数排序：仅处理非负整数，r = 10（改用 r = 256 可直接按字节分桶，效率更高）
    void radixSort(std::vector<int>& a) {
        if (a.size() < 2) return;
        const int BASE = 10;
        int maxVal = *std::max_element(a.begin(), a.end());
        std::vector<int> out(a.size());
        for (long long exp = 1; maxVal / exp > 0; exp *= BASE) {
            int cnt[BASE] = {0};
            for (int x : a) ++cnt[(x / exp) % BASE];
            for (int i = 1; i < BASE; ++i) cnt[i] += cnt[i - 1];     // 前缀和
            for (std::size_t i = a.size(); i-- > 0;) {               // 逆序填充，保证稳定
                int x = a[i];
                int d = (x / exp) % BASE;
                out[--cnt[d]] = x;
            }
            a.swap(out);
        }
    }
    
    int main() {
        std::vector<int> a = {170, 45, 75, 90, 802, 24, 2, 66};
        radixSort(a);
        for (int x : a) std::cout << x << ' ';
        std::cout << '\n';
        return 0;
    }

### 10.4 复杂度与稳定性

| 指标  | 结论  |
| --- | --- |
| 时间复杂度 | O(d · (n + r))，其中 d 为位数、r 为基数；对 32 位整数按字节（r = 256）为 O(4n)，按十进制位为 O(10n)；最好 / 平均 / 最坏一致 |
| 空间复杂度 | O(n + r) |
| 稳定性 | 稳定（每一轮内部使用稳定计数排序） |
| 与比较排序对比 | n 大、位数固定时优于 O(n log n)；n 小时 d 的常数可能反而更差 |
| 局限  | 需要额外 O(n) 缓冲；随机写访问导致缓存局部性不佳；仅适用于整数 / 定长字符串等可按位分解的键 |

### 10.5 适用场景

* 大量定长整数、字符串（如电话号码、ID、定长编码）的排序。
* 位数远小于 log₂n 的场景（如 32 位整数对 n ≥ 10⁶）。
* 后缀数组构造中的倍增算法、并行排序中的按字节划分。

### 10.6 常见优化点

1. **基数取 2 的幂**：用 `r = 256`（一字节）或 `r = 2^11` 替代 10，用位运算替代除模，显著提速。
2. **提前终止**：若一轮中最大值对应位已为 0（即 `exp > maxVal`），可提前结束。
3. **MSD 版本**：从高位开始递归分桶，可提前结束、便于并行，但需要处理大量小桶的递归开销。
4. **负数处理**：把负数与非负数分别处理（负数取绝对值排序后逆序拼接），或整体加偏移量转为非负。
5. **混合策略**：小区间（≤ 64）切换插入排序，避免多轮扫描的固定开销。

* * *

## 11. std::sort 的实现思路与使用要点

### 11.1 核心思想

`std::sort` 是 **Introsort（内省排序）**：以快速排序为主体，用「递归深度上限」监控划分质量，一旦深度超过 `2 * floor(log2(n))` 就判定有退化风险，切换为堆排序兜底；同时所有长度小于阈值的子区间改用插入排序收尾。这一三段式设计同时取得了快排的平均性能、堆排的最坏保证与插排的小区间常数优势。

### 11.2 执行过程（以 libstdc++ 实现为例）

1. `__introsort_loop(first, last, depth_limit = 2 * log2(n))`：当区间长度 > 16 时循环划分。
2. 划分用 `__unguarded_partition`：主元由 `__move_median_to_first`（首、中、尾三数取中）选出并置于首位置，随后采用双向（Hoare 风格）扫描，省去边界判断。
3. 每递归一层 `depth_limit--`；当 `depth_limit == 0` 时，对当前区间调用 `__partial_sort`（堆排序）并返回。
4. 循环 + 仅递归较小区间，实现尾递归消除，栈深度 O(log n)。
5. 退出循环后调用 `__final_insertion_sort`：区间已被切成若干长度 ≤ 16 的块，整体做一次插入排序完成收尾。

### 11.3 代码实现（用法示例）

    #include <algorithm>
    #include <cstdlib>
    #include <functional>
    #include <iostream>
    #include <string>
    #include <vector>
    
    struct Student {
        int score;
        std::string name;
    };
    
    int main() {
        std::vector<int> a = {5, 2, 9, 1, 5, 6};
    
        std::sort(a.begin(), a.end());                              // 1. 默认升序（operator<）
        std::sort(a.begin(), a.end(), std::greater<int>());         // 2. 降序（C++14 起推荐 std::greater<>）
        std::sort(a.begin(), a.end(), [](int x, int y) {            // 3. 自定义比较器：绝对值升序
            return std::abs(x) < std::abs(y);
        });
    
        std::vector<Student> s = {{90, "Bob"}, {90, "Alice"}, {85, "Cindy"}};
        std::sort(s.begin(), s.end(), [](const Student& x, const Student& y) {
            if (x.score != y.score) return x.score < y.score;       // 主关键字：分数升序
            return x.name < y.name;                                 // 次关键字：姓名升序，构成严格弱序
        });
    
        std::stable_sort(s.begin(), s.end(),                        // 4. 稳定版本：相等元素保持原序
                         [](const Student& x, const Student& y) { return x.score < y.score; });
    
        std::vector<int> b = {7, 3, 8, 1, 9, 2};
        std::partial_sort(b.begin(), b.begin() + 3, b.end());       // 5. 前 3 小有序：O(n log k)
        std::nth_element(b.begin(), b.begin() + 2, b.end());        // 6. 第 3 小就位：平均 O(n)
    
        for (const Student& x : s) std::cout << x.score << '-' << x.name << ' ';
        std::cout << '\n';
        return 0;
    }

### 11.4 复杂度与稳定性

| 指标  | 结论  |
| --- | --- |
| 时间复杂度 | C++11 起标准保证 O(n log n) 次比较（Introsort 兜底）；C++03 仅保证平均 O(n log n) |
| 空间复杂度 | O(log n) 递归栈 |
| 稳定性 | **不稳定**；需要稳定性必须使用 `std::stable_sort`（归并，有内存时 O(n log n)、无额外内存时 O(n log² n)） |
| 迭代器要求 | 随机访问迭代器（`vector` / `deque` / 原生数组可用）；`list` / `forward_list` 须用其成员函数 `sort()`（归并实现，稳定） |
| 相关算法 | `stable_sort`、`partial_sort`（O(n log k)）、`nth_element`（平均 O(n)）、`is_sorted`、`sort_heap` |

### 11.5 适用场景

* 通用默认选择：连续内存容器（`vector`、`array`、原生数组）的排序应直接用 `std::sort`。
* 需要稳定性或排序键相同需保持输入次序：`std::stable_sort`。
* 只要前 K 个或第 K 个：`partial_sort` / `nth_element`，避免全排序开销。
* 自定义类型：提供 `operator<` 或传入比较器即可。

### 11.6 常见优化点与陷阱

1. **比较器必须是严格弱序**：`comp(a, a) == false`、不可同时 `comp(a, b)` 与 `comp(b, a)` 为真、等价关系可传递。用 `<=` 或 `!comp(y, x)` 作为比较器会导致越界访问与未定义行为。
2. **比较器开销**：比较器会被调用约 n log n 次，避免在内部做字符串拷贝、正则匹配等重操作；可预先抽取排序键。
3. **大对象排序**：`sort` 依赖移动与交换，元素体积大时应排序指针 / 索引，或把大字段拆分到结构体外。
4. **阈值行为**：`std::sort` 内部对 < 16 元素走插入排序，因此小规模数据无需手写冒泡 / 选择排序。
5. **`std::greater<>`**：C++14 起使用透明比较器 `std::greater<>` 可省去类型参数书写。
6. **不要在排序中修改元素键值**：比较器应保持无副作用，否则等价类判定失效。
7. **并行版本**：C++17 提供 `std::sort(std::execution::par, ...)`，需链接 TBB 且保证比较器线程安全。

* * *

## 12. 原理与复杂度总结

### 12.1 原理要点速览

| 算法  | 核心机制 | 关键性质 |
| --- | --- | --- |
| 冒泡排序 | 相邻比较交换，每轮把最大值浮到末尾 | 交换次数等于逆序对数；提前退出后对有序输入为 O(n) |
| 选择排序 | 每轮线性扫描最小值并与区间首位交换 | 交换次数 O(n) 最少；比较次数固定 O(n²)，无自适应性 |
| 插入排序 | 逐个把元素插入已排序前缀，大元素右移腾位 | 自适应，逆序对少时近 O(n)；顺序访问、常数最小 |
| 希尔排序 | 按递减间隔分组插入排序，最后一轮 gap = 1 | 首个突破 O(n²) 的比较排序；复杂度由 gap 序列决定 |
| 归并排序 | 二分递归 + 线性合并两个有序段 | 唯一稳定且最坏 O(n log n) 的经典算法；需 O(n) 缓冲 |
| 快速排序 | 选 pivot 划分后递归，无合并步骤 | 常数最小的 O(n log n) 内核；划分失衡时退化 O(n²) |
| 堆排序 | 建大顶堆后反复取堆顶并下沉 | 原地且最坏 O(n log n)；跳跃访问导致局部性差 |
| 计数排序 | 频数统计 + 前缀和定位 + 逆序填充 | 非比较，O(n + k)；值域必须紧凑 |
| 桶排序 | 按值域分桶，桶内排序后顺序拼接 | 期望 O(n) 依赖均匀分布；倾斜分布会退化 |
| 基数排序 | 按位（LSD）执行 d 轮稳定计数排序 | 非比较，O(d(n + r))；适用于定长整数 / 字符串 |
| `std::sort` | Introsort：快排 + 深度限制转堆排 + 小区间插排 | 最坏 O(n log n) 保证；不稳定，要求随机访问迭代器 |

### 12.2 复杂度对照表

| 算法  | 最好  | 平均  | 最坏  | 空间  | 稳定  | 原地  | 比较类 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 冒泡排序 | O(n) | O(n²) | O(n²) | O(1) | 是   | 是   | 比较  |
| 选择排序 | O(n²) | O(n²) | O(n²) | O(1) | 否   | 是   | 比较  |
| 插入排序 | O(n) | O(n²) | O(n²) | O(1) | 是   | 是   | 比较  |
| 希尔排序 | O(n log n) | 依赖 gap | O(n²) ~ O(n^1.5) | O(1) | 否   | 是   | 比较  |
| 归并排序 | O(n log n) | O(n log n) | O(n log n) | O(n) | 是   | 否   | 比较  |
| 快速排序 | O(n log n) | O(n log n) | O(n²) | O(log n) | 否   | 是   | 比较  |
| 堆排序 | O(n log n) | O(n log n) | O(n log n) | O(1) | 否   | 是   | 比较  |
| 计数排序 | O(n + k) | O(n + k) | O(n + k) | O(n + k) | 是   | 否   | 非比较 |
| 桶排序 | O(n) | O(n + B) | O(n²) | O(n + B) | 视桶内算法 | 否   | 非比较 |
| 基数排序 | O(d(n + r)) | O(d(n + r)) | O(d(n + r)) | O(n + r) | 是   | 否   | 非比较 |
| `std::sort` | O(n log n) | O(n log n) | O(n log n) | O(log n) | 否   | 是   | 比较  |
| `std::stable_sort` | O(n log n) | O(n log n) | O(n log² n) | O(n) | 是   | 否   | 比较  |

### 12.3 按维度归类

1. **稳定算法**：冒泡排序、插入排序、归并排序、计数排序、基数排序（桶排序视桶内算法而定）。
2. **不稳定算法**：选择排序、希尔排序、快速排序、堆排序、`std::sort`。
3. **原地（O(1) 额外空间）**：冒泡、选择、插入、希尔、堆排序。
4. **具备最坏 O(n log n) 保证**：归并排序、堆排序、`std::sort`（Introsort）、计数 / 基数排序（非比较意义下）。
5. **自适应（输入有序时更优）**：冒泡（O(n)）、插入（O(n)）、自然归并（O(n)）、希尔（部分）。
6. **突破比较下界**：计数、桶、基数排序，代价是对键值分布或值域有严格约束。

### 12.4 选型决策建议

| 场景  | 推荐方案 | 理由  |
| --- | --- | --- |
| 通用内存排序，无特殊约束 | `std::sort` | 最坏 O(n log n) 保证 + 最小常数因子 |
| 需要保持相等元素相对次序 | `std::stable_sort` / 归并排序 | 稳定且 O(n log n) |
| 小规模或近乎有序数据 | 插入排序 | 自适应、常数小、缓存友好 |
| 内存受限且需最坏保证 | 堆排序 | 原地 O(1)，无退化风险 |
| 只要 Top-K 或第 K 小 | 堆选择 / `nth_element` | O(n log K) 或平均 O(n)，避免全排序 |
| 小值域整数（k << n） | 计数排序 | O(n + k) 线性时间 |
| 均匀分布浮点 / 大规模定长整数 | 桶排序 / 基数排序 | 突破 Ω(n log n) 下界 |
| 数据规模超出内存 | 外部归并排序 | 顺序 I/O，多路归并可扩展 |
| 链表容器 | `list::sort` | 归并实现，稳定，无需随机访问 |

### 12.5 演进脉络

比较排序的性能演化主线是「如何减少无效比较」：冒泡与选择以 O(n²) 的相邻 / 全局扫描为起点；插入排序引入自适应性；希尔排序用递减增量做跨距预处理；归并排序以分治把复杂度压到稳定的 O(n log n)，代价是 O(n) 空间；快速排序以原地划分取得最优常数，代价是最坏退化；堆排序以 O(1) 空间取得最坏保证，代价是缓存局部性；Introsort 把三者组合，成为工业级默认解。非比较排序（计数、桶、基数）则跳出比较模型，以值域约束换取线性时间。

* * *

> 复杂度中的 `log` 均以 2 为底；`k` 为值域大小，`d` 为关键字位数，`r` 为基数，`B` 为桶数。所有代码以 C++14 及以上标准编译通过。
