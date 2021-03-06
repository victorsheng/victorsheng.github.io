---
title: geektime-jvm-26-vectorization
abbrlink: 745262539
date: 2018-09-21 09:01:52
tags:
categories:
---

# SIMD 指令

# 使用 SIMD 指令的 HotSpot Intrinsic

# 自动向量化

几点自动向量化的条件。
1. 循环变量的增量应为 1，即能够遍历整个数组。
2. 循环变量不能为 long 类型，否则 C2 无法将循环识别为计数循环。
3. 循环迭代之间最好不要有数据依赖，例如出现类似于a[i] = a[i-1]的语句。当循环展开之后，循环体内存在数据依赖，那么 C2 无法进行自动向量化。
4. 循环体内不要有分支跳转。
5. 不要手工进行循环展开。如果 C2 无法自动展开，那么它也将无法进行自动向量化。

我们可以看到，自动向量化的条件较为苛刻。而且，C2 支持的整数向量化操作并不多，据我所致只有向量加法，向量减法，按位与、或、异或，以及批量移位和批量乘法。C2 还支持向量点积的自动向量化，即两两相乘再求和，不过这需要多条 SIMD 指令才能完成，因此并不是十分高效。

# 总结与实践
今天我介绍了即时编译器中的向量化优化。
向量化优化借助的是 CPU 的 SIMD 指令，即通过单条指令控制多组数据的运算。它被称为 CPU 指令级别的并行。
HotSpot 虚拟机运用向量化优化的方式有两种。第一种是使用 HotSpot intrinsic，在调用特定方法的时候替换为使用了 SIMD 指令的高效实现。Intrinsic 属于点覆盖，只有当应用程序明确需要这些 intrinsic 的语义，才能够获得由它带来的性能提升。
第二种是依赖即时编译器进行自动向量化，在循环展开优化之后将不同迭代的运算合并为向量运算。自动向量化的触发条件较为苛刻，因此也无法覆盖大多数用例。