# C++ 排序算法速查卡

> **定位**：竞赛 / 备考 / 面试现场的快速查找与默写。两屏内解决检索，代码为「最小可写对版本」。
> 需要原理推导、工程优化、完整 `main` 示例时，下钻到详解版。

**配套文件**

| 文件 | 用途 | 什么时候打开 |
| :--- | :--- | :--- |
| `sorting-cheatsheet.md` | 本文件：速查 + 默写模板 | 现场检索、手撕代码前 |
| `sorting-competition.md` | 竞赛场景代码：离散化、逆序对、第 K 大、多关键字 | 写题时 |
| `sorting-interview-qa.md` | 面试高频问答 22 题 | 面试前过一遍 |
| `cpp-sorting-algorithms.md` | 详解版：原理、工程实现、优化点、完整可编译示例 | 想搞懂「为什么」时 |

---

## 目录

- [1. 速查总表](#1-速查总表)
- [2. 记忆锚点](#2-记忆锚点)
- [3. 反向索引：问题到算法](#3-反向索引问题到算法)
- [4. 默写模板](#4-默写模板)
- [5. 现场易错点清单](#5-现场易错点清单)
- [6. 写完立刻自检](#6-写完立刻自检)

---

## 1. 速查总表

| 算法 | 最好 | 平均 | 最坏 | 空间 | 稳定 | 原地 | 一句话 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 冒泡 | O(n) | O(n²) | O(n²) | O(1) | 是 | 是 | 相邻逆序就交换，每轮最大值浮到末尾 |
| 选择 | O(n²) | O(n²) | O(n²) | O(1) | 否 | 是 | 每轮选最小放首位，交换次数只有 O(n) |
| 插入 | O(n) | O(n²) | O(n²) | O(1) | 是 | 是 | 逐个插进有序前缀，近乎有序时近 O(n) |
| 希尔 | O(n log n) | 看 gap | O(n²)~O(n^1.5) | O(1) | 否 | 是 | 按递减间隔分组插入排序，最后一轮 gap=1 |
| 归并 | O(n log n) | O(n log n) | O(n log n) | O(n) | 是 | 否 | 二分递归后线性合并，稳定且有最坏保证 |
| 快排 | O(n log n) | O(n log n) | O(n²) | O(log n) | 否 | 是 | 选 pivot 划分后递归，常数最小、划分失衡则退化 |
| 堆排 | O(n log n) | O(n log n) | O(n log n) | O(1) | 否 | 是 | 建大顶堆后反复取堆顶下沉，原地且有最坏保证 |
| 计数 | O(n + k) | O(n + k) | O(n + k) | O(n + k) | 是 | 否 | 频数统计 + 前缀和定位，值域必须小 |
| 桶 | O(n) | O(n + B) | O(n²) | O(n + B) | 看桶内 | 否 | 按值域分桶后桶内排序，依赖均匀分布 |
| 基数 | O(d(n + r)) | O(d(n + r)) | O(d(n + r)) | O(n + r) | 是 | 否 | 按位做 d 轮稳定计数排序 |
| `std::sort` | O(n log n) | O(n log n) | O(n log n) | O(log n) | 否 | 是 | Introsort：快排 + 深度限制转堆排 + 小区间插排 |
| `stable_sort` | O(n log n) | O(n log n) | O(n log² n) | O(n) | 是 | 否 | 归并实现，相等元素保持原序 |

> `k` = 值域大小，`d` = 位数，`r` = 基数，`B` = 桶数。log 以 2 为底。

---

## 2. 记忆锚点

**稳定性（面试必背）**

- 稳定五家：**冒、插、归、计、基**（冒泡、插入、归并、计数、基数）
- 不稳四家：**选、希、快、堆**（选择、希尔、快排、堆排）
- 助记：「冒插归计基」都是"温柔"操作（相邻交换 / 顺序放置）；「选希快堆」都靠**跨距离交换**，跨距离交换必破坏相等元素次序。

**复杂度分档**

- O(n²) 三兄弟：冒泡、选择、插入
- O(n log n) 三兄弟：归并、快排、堆排
- 突破比较下界（线性）：计数、桶、基数

**按性质归类**

| 性质 | 算法 |
| :--- | :--- |
| 最坏也是 O(n log n) | 归并、堆排、`std::sort` |
| 原地 O(1) 空间 | 冒泡、选择、插入、希尔、堆排 |
| 输入有序时降到 O(n) | 冒泡、插入 |
| 非比较排序 | 计数、桶、基数 |
| 需要额外 O(n) 缓冲 | 归并、计数、桶、基数、`stable_sort` |

**一句话理由（防止被追问）**

- 建堆为什么 O(n)：大多数节点在底层、下沉距离短，Σ(每层节点数 × 层高) 收敛到 O(n)，不是 n 次 O(log n)。
- 快排平均为什么 O(n log n)：随机 pivot 下划分大概率不算太偏，递归树期望深度 O(log n)，每层总代价 O(n)。
- 比较排序下界为什么 Ω(n log n)：n! 种排列，决策树至少 n! 片叶子，树高 ≥ log₂(n!) = Ω(n log n)。
- 非比较排序为什么能线性：它不做"两元素比较"，决策树模型的前提不成立，代价转移到值域 k 上。

---

## 3. 反向索引：问题到算法

| 我要解决的问题 | 用它 | 复杂度 | 下钻 |
| :--- | :--- | :--- | :--- |
| 通用排序，没特殊要求 | `std::sort` | O(n log n) | [详解 §11](cpp-sorting-algorithms.md#11-stdsort-的实现思路与使用要点) |
| 相等元素要保持输入次序 | `stable_sort` / 归并 | O(n log n) | [详见](#归并排序-merge-sort) |
| 数组很小（n ≤ 50） | 插入排序 | O(n²) 但常数最小 | [详见](#插入排序-insertion-sort) |
| 数据近乎有序 | 插入排序 | 近 O(n) | [详见](#插入排序-insertion-sort) |
| 绝不能退化到 O(n²) | 归并 / 堆排 / `std::sort` | O(n log n) 最坏 | [详见](#堆排序-heap-sort) |
| 内存极紧，还要最坏保证 | 堆排序 | O(1) 额外空间 | [详见](#堆排序-heap-sort) |
| 只要前 K 个 / 第 K 个 | `nth_element` / 堆选择 | 平均 O(n) / O(n log K) | [竞赛文档](sorting-competition.md#3-第-k-大与-top-k) |
| 整数且值域小（k << n） | 计数排序 | O(n + k) | [详见](#计数排序-counting-sort) |
| 大量定长整数 / 字符串 | 基数排序 | O(d(n + r)) | [详见](#基数排序-radix-sort) |
| [0, 1) 均匀分布的浮点 | 桶排序 | 期望 O(n) | [详见](#桶排序-bucket-sort) |
| 数据装不进内存 | 外部归并排序 | 顺序 I/O | [问答 Q7](sorting-interview-qa.md) |
| 链表容器 | `list::sort()` | O(n log n) 稳定 | [问答 Q16](sorting-interview-qa.md) |
| 多关键字排序 | 单个严格弱序比较器 | — | [竞赛文档](sorting-competition.md#4-结构体多关键字排序) |
| 统计逆序对 | 归并排序过程计数 | O(n log n) | [竞赛文档](sorting-competition.md#2-逆序对统计) |
| 去重 | `sort` + `unique` | O(n log n) | [竞赛文档](sorting-competition.md#5-去重-sort--unique) |
| 值域压缩 / 离散化 | `sort` + `unique` + `lower_bound` | O(n log n) | [竞赛文档](sorting-competition.md#1-离散化) |

---

## 4. 默写模板

统一约定：`vector<int>& a`，区间一律 **[l, r) 左闭右开**，下标用 `int`（竞赛手写下 `size_t` 容易下溢）。

### 冒泡排序 Bubble Sort

```cpp
void bubbleSort(vector<int>& a) {
    int n = a.size();
    for (int i = 0; i < n - 1; ++i) {
        bool sw = false;
        for (int j = 0; j < n - 1 - i; ++j)
            if (a[j] > a[j + 1]) { swap(a[j], a[j + 1]); sw = true; }
        if (!sw) break;                      // 一轮无交换即已有序
    }
}
```

易错：`if (!sw) break;` 是最好情况 O(n) 的来源，漏写则恒为 O(n²)。

### 选择排序 Selection Sort

```cpp
void selectionSort(vector<int>& a) {
    int n = a.size();
    for (int i = 0; i < n - 1; ++i) {
        int mn = i;
        for (int j = i + 1; j < n; ++j) if (a[j] < a[mn]) mn = j;
        swap(a[i], a[mn]);
    }
}
```

易错：不稳定。经典反例 `[5a, 3, 5b]` → 第一轮 5a 与 3 交换，5a 落到 5b 后面。

### 插入排序 Insertion Sort

```cpp
void insertionSort(vector<int>& a) {
    int n = a.size();
    for (int i = 1; i < n; ++i) {
        int key = a[i], j = i - 1;
        while (j >= 0 && a[j] > key) { a[j + 1] = a[j]; --j; }
        a[j + 1] = key;
    }
}
```

易错：`while` 里必须用 `>` 而非 `>=`（否则不稳定）；`j >= 0` 的判断不能省（int 版可用哨兵优化）。

### 希尔排序 Shell Sort

```cpp
void shellSort(vector<int>& a) {
    int n = a.size();
    for (int gap = n / 2; gap > 0; gap /= 2)
        for (int i = gap; i < n; ++i) {
            int key = a[i], j = i - gap;
            while (j >= 0 && a[j] > key) { a[j + gap] = a[j]; j -= gap; }
            a[j + gap] = key;
        }
}
```

易错：最外层是 `gap` 递减，不是递归；`gap` 必须最终取到 1，否则结果不对。

### 归并排序 Merge Sort

```cpp
void mergeSort(vector<int>& a, int l, int r) {      // [l, r)
    if (r - l <= 1) return;
    int m = l + (r - l) / 2;
    mergeSort(a, l, m);
    mergeSort(a, m, r);
    vector<int> t(r - l);
    int i = l, j = m, k = 0;
    while (i < m && j < r) t[k++] = (a[i] <= a[j]) ? a[i++] : a[j++];
    while (i < m) t[k++] = a[i++];
    while (j < r) t[k++] = a[j++];
    for (int p = 0; p < k; ++p) a[l + p] = t[p];
}
```

易错：三处必须写对——
1. `mid = l + (r - l) / 2`（不用 `(l + r) / 2`，防溢出习惯）；
2. 合并时 `a[i] <= a[j]` 取左段（取右段就不稳定了）；
3. 递归边界 `[l, m)` 与 `[m, r)`，写成 `[l, m - 1]` 类闭区间会死循环。

### 快速排序 Quick Sort

```cpp
void quickSort(vector<int>& a, int l, int r) {      // [l, r)，Hoare 划分
    if (r - l <= 1) return;
    int pivot = a[l + (r - l) / 2];                 // 取 pivot 的值，不是下标
    int i = l - 1, j = r;
    while (i < j) {
        do ++i; while (a[i] < pivot);
        do --j; while (a[j] > pivot);
        if (i < j) swap(a[i], a[j]);
    }
    quickSort(a, l, i);
    quickSort(a, i, r);
}
```

易错（Hoare 版四坑，挂科率最高）：
1. **pivot 取"值"不取"下标"**——划分过程会移动 pivot 所在位置，存下标会失效；
2. 必须用 `do while`（先移动再比较），写成 `while` 会在相等元素上死循环；
3. 递归区间是 `[l, i)` 与 `[i, r)`，**不是** `[l, i)` 与 `[i + 1, r)`（后者漏掉一个元素）；
4. 边界 `i = l - 1, j = r` 配合 `[l, r)` 开区间，闭区间写法要同步改。

### 堆排序 Heap Sort

```cpp
void siftDown(vector<int>& a, int i, int n) {
    while (true) {
        int c = 2 * i + 1;
        if (c >= n) break;
        if (c + 1 < n && a[c] < a[c + 1]) ++c;
        if (a[i] >= a[c]) break;
        swap(a[i], a[c]);
        i = c;
    }
}

void heapSort(vector<int>& a) {
    int n = a.size();
    for (int i = n / 2 - 1; i >= 0; --i) siftDown(a, i, n);   // 建大顶堆 O(n)
    for (int end = n; end > 1; --end) {
        swap(a[0], a[end - 1]);                               // 堆顶最大值就位
        siftDown(a, 0, end - 1);
    }
}
```

易错：`n = 0` 时 `n / 2 - 1 = -1`，`for` 判断 `i >= 0` 用 int 是安全的（换成 `size_t` 就死循环）；建堆必须从 `n/2 - 1` 倒着来，正着建是 O(n log n)。

### 计数排序 Counting Sort

```cpp
vector<int> countingSort(const vector<int>& a, int lo, int hi) {   // 值域 [lo, hi]
    vector<int> cnt(hi - lo + 1, 0), out(a.size());
    for (int x : a) ++cnt[x - lo];
    for (int i = 1; i < (int)cnt.size(); ++i) cnt[i] += cnt[i - 1];   // 前缀和
    for (int i = (int)a.size() - 1; i >= 0; --i) out[--cnt[a[i] - lo]] = a[i];
    return out;
}
```

易错：**必须逆序填充**（`i` 从尾到头）才稳定；`a` 为空时 `(int)a.size() - 1 = -1`，循环不进入，安全。

### 桶排序 Bucket Sort

```cpp
void bucketSort(vector<double>& a, int B) {         // 输入落在 [0, 1)
    vector<vector<double>> b(B);
    for (double x : a) b[min(B - 1, (int)(x * B))].push_back(x);
    for (auto& v : b) sort(v.begin(), v.end());
    int k = 0;
    for (auto& v : b) for (double x : v) a[k++] = x;
}
```

易错：`x == 1.0` 时 `x * B == B` 越界，需 `min` 夹紧。

### 基数排序 Radix Sort

```cpp
void radixSort(vector<int>& a) {                    // 仅非负整数，BASE = 10
    int n = a.size();
    if (n < 2) return;
    int mx = *max_element(a.begin(), a.end());
    vector<int> out(n);
    for (long long e = 1; mx / e > 0; e *= 10) {
        int cnt[10] = {0};
        for (int x : a) ++cnt[x / e % 10];
        for (int i = 1; i < 10; ++i) cnt[i] += cnt[i - 1];
        for (int i = n - 1; i >= 0; --i) { int d = a[i] / e % 10; out[--cnt[d]] = a[i]; }
        a.swap(out);
    }
}
```

易错：空数组上 `max_element` 解引用是 UB，`if (n < 2) return;` 不能省；循环变量 `e` 用 `long long`（`e *= 10` 会溢出 int）；负数需另行处理。

### std::sort 用法模板

```cpp
sort(a.begin(), a.end());                                    // 升序
sort(a.begin(), a.end(), greater<int>());                    // 降序
sort(a.begin(), a.end(), [](int x, int y) { return abs(x) < abs(y); });
sort(v.begin(), v.end(), [](const Node& x, const Node& y) {  // 多关键字
    if (x.key1 != y.key1) return x.key1 < y.key1;
    return x.key2 < y.key2;
});
stable_sort(v.begin(), v.end(), cmp);                        // 稳定版
nth_element(a.begin(), a.begin() + k, a.end());              // 第 k 小就位，平均 O(n)
partial_sort(a.begin(), a.begin() + k, a.end());             // 前 k 小有序，O(n log k)
```

易错：比较器必须是**严格弱序**——`comp(a, a)` 必须为 false，用 `<=` 会越界崩溃。

---

## 5. 现场易错点清单

| # | 易错点 | 后果 | 正确写法 |
| :-- | :--- | :--- | :--- |
| 1 | `size_t` 做递减下标 | `i >= 0` 恒真，死循环 / 越界 | 下标用 `int`，或写 `for (size_t i = n; i-- > 0;)` |
| 2 | `i < n - 1` 判边界，n = 0 | `n - 1` 下溢成巨大值 | 写 `i + 1 < n` |
| 3 | 快排 pivot 存下标 | 划分中位置被移动，逻辑错 | 存 `pivot = a[mid]` 的值 |
| 4 | Hoare 用 `while` 而非 `do while` | 相等元素上死循环 | 必须先移动指针再比较 |
| 5 | 快排递归边界写成 `[l, p-1]` 与 `[p+1, r]` | 漏元素 / 越界 | Hoare 版用 `[l, i)` 与 `[i, r)` |
| 6 | 归并 `mid = (l + r) / 2` | 极端场景溢出（养成习惯用 `l + (r-l)/2`） | 同左 |
| 7 | 归并取 `a[i] < a[j]` 时先取右 | 结果不稳定 | 相等取左段：`a[i] <= a[j]` |
| 8 | 计数 / 基数顺序填充 | 结果不稳定 | 逆序遍历原数组 |
| 9 | 堆排建堆正序 `for (i = 0; i < n/2; ++i)` | 建堆退化 O(n log n)，且逻辑错 | 从 `n/2 - 1` 倒序 |
| 10 | 比较器写 `return x.k <= y.k;` | 非严格弱序 → 越界 / 崩溃 | 用 `<`，多关键字用 `!=` 分层 |
| 11 | `unique` 后忘记 `erase` | 尾部残留重复元素，size 不变 | `v.erase(unique(v.begin(), v.end()), v.end())` |
| 12 | `sort` 用于 `list` / `forward_list` | 编译失败（需随机访问迭代器） | 用成员函数 `l.sort()` |
| 13 | 逆序对计数用 `int` | 最坏 n(n-1)/2，n = 1e5 时爆 int | 用 `long long` |
| 14 | 空容器上 `max_element` 解引用 | UB | 先判 `n < 2` |
| 15 | 排序后对原下标做映射 | 原下标丢失 | 排序索引数组或结构体存下标 |

---

## 6. 写完立刻自检

手写排序后，按顺序过这 6 个用例（覆盖 90% 的现场 bug）：

| 用例 | 检查点 |
| :--- | :--- |
| `[]` | 不崩、不死循环 |
| `[1]` | 不崩 |
| `[2, 1]` | 最小逆序 |
| `[1, 2, 3]` | 已有序（触发最好情况分支） |
| `[3, 2, 1]` | 完全逆序（触发最坏情况） |
| `[5, 5, 5, 5]` | 全部相等（快排死循环 / 稳定性检查） |

**稳定性检查法**：给元素加一个"原始下标"字段，排序后检查相同键值的元素下标是否递增。

**对照法**：随机生成 1000 组数据，与 `std::sort` 结果比对，同时断言 `is_sorted`。
