# C++ 竞赛排序场景代码

> **定位**：竞赛 / 机试中「排序」的真实用法。全部为可直接编译运行的完整程序，现场复制改数据即可用。
> 手写算法模板见 [`sorting-cheatsheet.md`](sorting-cheatsheet.md)，原理与工程实现见 [`cpp-sorting-algorithms.md`](cpp-sorting-algorithms.md)。

## 目录

- [1. 离散化](#1-离散化)
- [2. 逆序对统计](#2-逆序对统计)
- [3. 第 K 大与 Top-K](#3-第-k-大与-top-k)
- [4. 结构体多关键字排序](#4-结构体多关键字排序)
- [5. 去重 sort + unique](#5-去重-sort--unique)
- [6. 频次统计](#6-频次统计)
- [7. 排序后的二分与双指针](#7-排序后的二分与双指针)
- [8. 竞赛易错点](#8-竞赛易错点)

---

## 1. 离散化

**用途**：值域很大（1e9）但元素个数不多（1e5），把值映射到紧凑下标，供树状数组 / 线段使用。
**复杂度**：O(n log n)。

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> a = {100, -3, 2, 2, 1'000'000'000, -3};

    vector<int> xs = a;                                           // 1) 复制
    sort(xs.begin(), xs.end());                                   // 2) 排序
    xs.erase(unique(xs.begin(), xs.end()), xs.end());             // 3) 去重

    for (int& x : a) {
        int id0 = lower_bound(xs.begin(), xs.end(), x) - xs.begin();      // 0-based
        int id1 = lower_bound(xs.begin(), xs.end(), x) - xs.begin() + 1;  // 1-based
        cout << x << " -> " << id0 << " / " << id1 << '\n';
    }
    return 0;
}
```

要点：
- 三步固定套路：**排序 → `unique` → `lower_bound`**，映射后相对大小关系保持不变。
- 需要"第几小"时用 0-based 下标；喂给树状数组时用 1-based（树状数组不接受 0 下标）。
- 只需要判大小、不需要还原原值时可省去原数组，直接在离散后的下标上做。

---

## 2. 逆序对统计

**用途**：统计 `i < j 且 a[i] > a[j]` 的对数。归并排序合并过程中的自然副产品。
**复杂度**：O(n log n)；计数必须用 `long long`（n = 1e5 时最多约 5e9，爆 int）。

```cpp
#include <iostream>
#include <vector>
using namespace std;

long long mergeCount(vector<int>& a, int l, int r) {              // [l, r)
    if (r - l <= 1) return 0;
    int m = l + (r - l) / 2;
    long long res = mergeCount(a, l, m) + mergeCount(a, m, r);
    vector<int> t(r - l);
    int i = l, j = m, k = 0;
    while (i < m && j < r) {
        if (a[i] <= a[j]) {
            t[k++] = a[i++];
        } else {
            res += m - i;                                         // 左段剩余全部与 a[j] 构成逆序对
            t[k++] = a[j++];
        }
    }
    while (i < m) t[k++] = a[i++];
    while (j < r) t[k++] = a[j++];
    for (int p = 0; p < k; ++p) a[l + p] = t[p];
    return res;
}

int main() {
    vector<int> a = {3, 1, 2, 5, 4};
    cout << mergeCount(a, 0, a.size()) << '\n';                   // 3：(3,1) (3,2) (5,4)
    return 0;
}
```

要点：
- `res += m - i` 是核心：右段取走 `a[j]` 时，左段 `[i, m)` 全部比它大。
- 会**破坏原数组**（就地排序），需要保留原数组时先拷贝一份。
- 值域可控时也可用树状数组做，同为 O(n log n)，代码更短，配合离散化使用。

---

## 3. 第 K 大与 Top-K

**用途**：不排序整个数组，只取极值。
**复杂度**：`nth_element` 平均 O(n)；堆维护 Top-K 为 O(n log K)。

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

// 堆维护 Top-K 大：O(n log K)，K 远小于 n 时最优
vector<int> topKLargest(const vector<int>& a, int k) {
    priority_queue<int, vector<int>, greater<int>> pq;            // 小顶堆：堆顶是 K 个中最小的
    for (int x : a) {
        if ((int)pq.size() < k) pq.push(x);
        else if (x > pq.top()) { pq.pop(); pq.push(x); }
    }
    vector<int> res;
    while (!pq.empty()) { res.push_back(pq.top()); pq.pop(); }
    reverse(res.begin(), res.end());                              // 从大到小
    return res;
}

int main() {
    vector<int> a = {5, 2, 9, 1, 5, 6, 8};

    vector<int> b = a;
    nth_element(b.begin(), b.begin() + 3, b.end());               // 第 4 小就位，左边都 <= 它
    cout << "4th smallest: " << b[3] << '\n';

    vector<int> c = a;
    nth_element(c.begin(), c.begin() + 2, c.end(), greater<int>()); // 第 3 大
    cout << "3rd largest: " << c[2] << '\n';

    for (int x : topKLargest(a, 3)) cout << x << ' ';             // 9 8 6
    cout << '\n';
    return 0;
}
```

要点：
- `nth_element(first, nth, last)`：**`nth` 位置的元素就位**，且它左边的都不大于它，左右两侧内部无序。
- 取第 K 大用 `greater<int>()`，或换算成第 `n - K` 小（0-based：`begin() + (n - K)`）。
- 数据量极大（装不进内存）时只能用堆，用 `nth_element` 需要全部数据在内存里。
- 手写快速选择（`quickSelect`）期望 O(n)，但 `nth_element` 在标准库里已经是 introselect，更稳。

---

## 4. 结构体多关键字排序

**用途**：按多个字段排序，竞赛最高频的用法之一。
**复杂度**：O(n log n)。

```cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <tuple>
#include <vector>
using namespace std;

struct Node {
    int score;
    int time;
    int id;
};

int main() {
    vector<Node> v = {{90, 30, 1}, {90, 20, 2}, {85, 10, 3}, {90, 20, 4}};

    // 写法一：分层比较（推荐，方向可任意组合）
    sort(v.begin(), v.end(), [](const Node& x, const Node& y) {
        if (x.score != y.score) return x.score > y.score;         // 分数降序
        if (x.time  != y.time ) return x.time  < y.time;          // 用时升序
        return x.id < y.id;                                       // id 兜底，保证全序
    });

    // 写法二：同向排序时用 tie（字典序，需要 <tuple>）
    vector<Node> w = {{90, 30, 1}, {90, 20, 2}, {85, 10, 3}};
    sort(w.begin(), w.end(), [](const Node& x, const Node& y) {
        return tie(x.score, x.time) < tie(y.score, y.time);
    });

    for (const Node& n : v) cout << n.score << ' ' << n.time << ' ' << n.id << '\n';
    return 0;
}
```

要点：
- 比较器必须是**严格弱序**：`comp(x, x) == false`。写 `<=` 会导致越界访问或崩溃。
- 最后一个字段（如 `id`）用来兜底，可保证"完全有序"，否则等价元素的顺序不确定（不稳定排序）。
- 需要"先按 A 排、再按 B 排、保留 A 的相对次序"时，必须用 `stable_sort` 分两次排。

---

## 5. 去重 sort + unique

**复杂度**：O(n log n)（排序主导）。

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> a = {5, 2, 9, 2, 5, 6, 5};

    sort(a.begin(), a.end());                                     // unique 只对相邻重复有效
    a.erase(unique(a.begin(), a.end()), a.end());                 // 必须 erase，size 才真正变小

    cout << "unique count: " << a.size() << '\n';
    for (int x : a) cout << x << ' ';                             // 2 5 6 9
    cout << '\n';

    // 判断是否存在重复元素
    vector<int> b = {1, 2, 2, 3};
    sort(b.begin(), b.end());
    bool hasDup = adjacent_find(b.begin(), b.end()) != b.end();
    cout << (hasDup ? "has dup" : "no dup") << '\n';
    return 0;
}
```

要点：
- `unique` 只是**把重复元素挪到尾部并返回新逻辑末尾**，容器 `size` 不变，必须配 `erase`。
- `unique` 前必须先 `sort`，否则只去相邻重复。
- 只要"有没有重复"用 `adjacent_find`，比 `unique` 语义更直接。

---

## 6. 频次统计

计数排序在竞赛中最常用的形态：不是排序本身，而是 O(n) 统计每个值出现多少次。

```cpp
#include <algorithm>
#include <iostream>
#include <unordered_map>
#include <vector>
using namespace std;

int main() {
    vector<int> a = {5, 2, 9, 2, 5, 6, 5, 2};

    // 情形一：值域小（如 0 ~ 100），直接开数组，O(n + k)
    const int MAXV = 100;
    vector<int> cnt(MAXV + 1, 0);
    for (int x : a) ++cnt[x];
    cout << "cnt[5] = " << cnt[5] << '\n';

    // 情形二：值域大（如 1e9），排序后分组统计，O(n log n)，无哈希开销
    vector<int> b = a;
    sort(b.begin(), b.end());
    for (int i = 0; i < (int)b.size(); ) {
        int j = i;
        while (j < (int)b.size() && b[j] == b[i]) ++j;
        cout << b[i] << " x" << (j - i) << '\n';
        i = j;
    }

    // 情形三：值域大且只需计数，用哈希，平均 O(n)
    unordered_map<int, int> freq;
    for (int x : a) ++freq[x];
    cout << "freq[2] = " << freq[2] << '\n';
    return 0;
}
```

要点：
- 值域可控时数组计数最快；值域大时优先"排序 + 分组"（比哈希更稳定，无最坏退化）。
- `unordered_map` 有被构造卡哈希的风险（CF 经典 hack），正式比赛慎用，或加自定义哈希。

---

## 7. 排序后的二分与双指针

排序的真正价值在于它解锁了 O(log n) 查找与 O(n) 双指针。

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> a = {5, 2, 9, 2, 6, 1};
    sort(a.begin(), a.end());                                     // 1 2 2 5 6 9

    // 二分：第一个 >= x / 第一个 > x / 值的出现次数
    auto lb = lower_bound(a.begin(), a.end(), 2);                 // 指向第一个 2
    auto ub = upper_bound(a.begin(), a.end(), 2);                 // 指向第一个 > 2
    cout << "count(2) = " << (ub - lb) << '\n';                   // 2

    // 双指针：有序数组中找两数之和 == target
    int target = 8;
    int l = 0, r = (int)a.size() - 1;
    while (l < r) {
        int s = a[l] + a[r];
        if (s == target) { cout << a[l] << " + " << a[r] << " = " << target << '\n'; break; }
        if (s < target) ++l; else --r;
    }
    return 0;
}
```

要点：
- `lower_bound` / `upper_bound` 要求区间**已有序**，否则结果未定义。
- `upper_bound(x) - lower_bound(x)` 即 x 的出现次数；`lower_bound` 减 `begin()` 即排名。
- 双指针依赖有序性做"和太大就右移右指针"的单调判断，无序数组不可用。

---

## 8. 竞赛易错点

| # | 易错点 | 后果 | 规避 |
| :-- | :--- | :--- | :--- |
| 1 | 逆序对计数用 `int` | n = 1e5 时约 5e9，溢出 | `long long` |
| 2 | `unique` 后不 `erase` | size 不变，尾部脏数据 | 固定写 `v.erase(unique(...), v.end())` |
| 3 | 不 `sort` 直接 `unique` | 只去相邻重复 | 顺序固定：先 sort |
| 4 | 比较器写 `<=` | 非严格弱序，RE | 用 `<`，多字段用 `!=` 分层 |
| 5 | 比较器漏兜底字段 | 等价类顺序不定，结果不可复现 | 加 `id` 作最后一层 |
| 6 | `lower_bound` 用在无序数组 | 结果未定义 | 确认已排序 |
| 7 | 树状数组用 0 下标 | 死循环（lowbit(0) = 0） | 离散化后 +1 转 1-based |
| 8 | `nth_element` 的 k 是 0-based | 取第 K 大算错 | 第 K 大 → `begin() + (K - 1)` 配 `greater` |
| 9 | 排序大对象（结构体很大） | 移动开销大，TLE | 排序索引数组 / 指针数组 |
| 10 | 递归版归并层数过深 | 栈溢出（n 极大时） | 用迭代版或全局临时数组 |
