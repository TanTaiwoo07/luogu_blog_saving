# C++ 排序算法资料集

面向**竞赛 / 备考 / 面试**的 C++ 排序算法资料，按使用场景拆成四份，全部为纯 Markdown（GitHub Flavored Markdown），锚点可跳转。

## 文件导航

| 文件 | 体量 | 用途 | 什么时候打开 |
| :--- | :--- | :--- | :--- |
| [`sorting-cheatsheet.md`](sorting-cheatsheet.md) | 约 400 行 | 速查卡 + 默写模板 + 易错点 | 现场检索、手撕代码前、考前过一遍 |
| [`sorting-competition.md`](sorting-competition.md) | 约 300 行 | 竞赛场景代码（离散化、逆序对、Top-K、多关键字…） | 写题时直接复制 |
| [`sorting-interview-qa.md`](sorting-interview-qa.md) | 约 220 行 | 面试高频问答 22 题 + 背诵清单 | 面试前 30 分钟 |
| [`cpp-sorting-algorithms.md`](cpp-sorting-algorithms.md) | 约 970 行 | 详解版：原理、工程实现、优化点、完整可编译示例 | 想搞懂「为什么」时下钻 |

## 三种用法

1. **竞赛现场**：开 `sorting-cheatsheet.md` 查复杂度与稳定性，开 `sorting-competition.md` 复制场景代码。
2. **面试前**：先看 `sorting-interview-qa.md` 的 22 题与背诵清单，再过 `sorting-cheatsheet.md` §4 的默写模板，最后用 §6 的 6 个用例自检。
3. **系统学习**：按 `cpp-sorting-algorithms.md` 从头读，配套速查卡做笔记。

## 一句话选型

| 场景 | 选择 |
| :--- | :--- |
| 通用排序，无特殊要求 | `std::sort` |
| 要保持相等元素原序 | `stable_sort` / 归并 |
| 小数组或近乎有序 | 插入排序 |
| 不许退化到 O(n²) | 归并 / 堆排 / `std::sort` |
| 内存极紧 + 最坏保证 | 堆排序 |
| 只要前 K 个 / 第 K 个 | `nth_element` / 小顶堆 |
| 整数值域小 | 计数排序 |
| 大量定长整数 / 字符串 | 基数排序 |
| 数据装不进内存 | 外部归并排序 |

## 版本说明

所有代码以 C++14 及以上标准为准，示例中出现的 `1'000'000'000`（数字分隔符）与结构化绑定之外的写法均在 C++14 下合法；已通过 MSVC 14.51 `/std:c++14 /W3` 编译验证。
