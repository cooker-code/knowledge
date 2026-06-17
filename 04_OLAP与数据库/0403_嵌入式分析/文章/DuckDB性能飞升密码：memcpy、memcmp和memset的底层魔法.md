---
title: DuckDB性能飞升密码：memcpy、memcmp和memset的底层魔法
author: yanzongshuaiDBA
date: yzshoveryzshover
url: https://mp.weixin.qq.com/s?__biz=MzU1OTgxMjA4OA==&mid=2247486458&idx=1&sn=e72823f7365519719c0af8fdc34fd490&chksm=fd7e90061c6e059af9747435b99591471fa44e463966bb7beac123841214c3e1b836fe231d64&mpshare=1&scene=24&srcid=05307wyPybPAwyhqeZSRN7yg&sharer_shareinfo=0ac0df098aa4710be3b14298d1ab7f73&sharer_shareinfo_first=0ac0df098aa4710be3b14298d1ab7f73#rd
---

DuckDB性能飞升密码：memcpy、memcmp和memset的底层魔法

memcpy、memcmp与memset这三个看似基础的C语言内存函数，却对性能有着至关重要的作用。这些直接操作内存字节的基础函数，串联起了数据拷贝、比较、初始化的全流程，每一个字节级的操作效率提升，最终都会在大规模数据处理中被放大为质变级的性能飞跃。DuckDB对这三个函数进行了改造，让其可以最大能力自动编译优化。

对于memcpy，使用FastMemcpy通过switch case进行分发对1到256字节长度字符串拷贝进行封装，让调用memcpy时其入参字符串大小值为常量。这样在编译时，当编译器看到memcmp(a, b, 8) 这种调用时，它不会生成一个真正的函数调用，而是直接生成 CPU 指令（如 mov 和 cmp）。

如下图所示，DuckDB使用模板函数进行内联，当字符串长度大于256时，调用memcpy(dest,src,size)让其自己自动SIMD优化，1-256字节大小时，通过switch case分发，使memcpy的size入参变成常量：

对于**memcmp对64字节以内**长度比较，使用MemcmpFixed模板函数，从而使得memcmp的入参变成常量，超过64字节的使用memcmp(dest,src,size)：

对于**memset，对256字节**以内的长度，通过switch case让入参变成常量：