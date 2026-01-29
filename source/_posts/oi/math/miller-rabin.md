---
date: 2024-01-22 16:19:00
updated: 2024-01-22 16:19:00
title: 数学 - Miller Rabin
katex: true
description: Miller Rabin 算法是一种用来快速判断一个数是否为质数的随机化算法，当数字选择恰当时可以保证在 uint64 范围内准确。
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 简介

Miller Rabin 算法是一种用来快速判断一个数是否为质数的随机化算法，当数字选择恰当时可以保证在 `uint64` 范围内准确。

## 原理

费马小定理指出，若 $p$ 为质数那么满足 $a^{p-1}\equiv 1\pmod p$，遗憾的是反过来并不成立，但是如果有 $a^{p-1}\not\equiv 1\pmod p$ 那么一定有 $p$ 不是质数，但是这个性质仍然不够强，根据 $a^{p-1}\equiv 1\pmod p$ 可以推导出来更多的条件。

若 $p$ 是非 $2$ 偶数，显然可以直接判掉，根据二次剩余的知识，若 $a^{2}\equiv 1\pmod p$，那么有 $a\equiv 1\pmod p$ 或 $a\equiv -1\pmod p$。

令 $p-1=u2^k$，从 $a^u$ 开始不断平方得到数列 $a^u,a^{2u},a^{4u},\dots,a^{2^ku}$，如果这个数列满足一开始就是 $1$，或者非最后一个数为 $-1$，然后接下来都是 $1$，那么就通过当前检验，显然这是 $p$ 为素数的必要条件。

选取 $a\in\{2, 325, 9375, 28178, 450775, 9780504, 1795265022\}$ 可以保证在 `uint64` 范围内正确，实际上直接选取前几个质数也可以做到不错的效果。

## 实现

> 输入若干行，一行一个数 $1\le x\le 10^{18}$，行数不超过 $10^5$，判断是否为质数。
>
> 题目链接：[LOJ 143](https://loj.ac/p/143)。

由于底数可能爆 `long long`，所以在乘法时需要用 `__int128` 转换。

然后因为检验的 $n$ 一定为奇数，那么 $n-1=u2^k$ 其中 $k\ge 1$，所以当 $a^u \not\equiv \pm1\pmod p$ 时，在上面提到的数列中一定是先出现 $-1$ 再出现 $1$，那么检验的目标就是判断是否在非最后一个数的位置出现了 $-1$，如果没有就不通过检验。

特别地，如果数列中 $1$ 出现在了 $-1$ 之前，也说明当前数一定不是素数，不通过检验。

复杂度 $O(\log n)$，但是常数较大。

```cpp
#include <bits/stdc++.h>
using namespace std;

namespace IO {}
using IO::read;
using IO::pc;

#define int long long
int qmi(int a, int k, int p) {
    int res = 1;
    while (k) {
        if (k & 1) res = (__int128)res * a % p;
        a = (__int128)a * a % p;
        k >>= 1;
    }
    return res;
}

bool test(int a, int u, int k, int n) {
    int v = qmi(a, u, n);
    if (v == 1 || v == 0 || v == n-1) return true;
    for (int i = 1; i <= k; i++) {
        v = (__int128)v * v % n;
        if (v == n-1 && i != k) return true;
        if (v == 1) return false;
    }
    return false;
}

bool mr(int n) {
    static int bases[] = {2, 325, 9375, 28178, 450775, 9780504, 1795265022};  
    if (n < 3 || n % 2 == 0) return n == 2;
    int u = n-1, k = 0;
    while (u % 2 == 0) u /= 2, k++;  
    for (int a: bases) if (!test(a, u, k, n)) return false;
    return true;
}

signed main() {
    for (int x; read(x); pc('\n')) pc(mr(x) ? 'Y' : 'N');
    return 0;
}
```

