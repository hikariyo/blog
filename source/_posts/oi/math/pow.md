---
date: 2023-06-17 15:01:43
title: 数学 - 快速幂与光速幂
katex: true
tags:
- Algo
- C++
categories:
- OI
---

## 快速幂

快速幂是在 $O(\log k)$ 的复杂度下计算 $a^k\bmod p$ 的结果。

可以用二进制的方式理解，令 $k=\sum b_i2^i$，那么 $a^k=a^{\sum b_i2^i}=\prod (a^{2^i})^{b_i}$，所以在每一次循环中处理出 $a\gets a^2$ 即可，相当于处理出 $a_i=a_{i-1}^2=(a^{2^{i-1}})^2=a^{2^i}$，类似于一个滚动数组优化。

```cpp
int qmi(int a, int k, int p) {
    int res = 1;
    while (k) {
        if (k & 1) res = 1ll * res * a % p;
        a = 1ll * a * a % p;
        k >>= 1;
    }
    return res;
}
```

## 光速幂

当底数不变时，预处理出 $a^1\sim a^{B-1}$ 与 $a^B\sim a^{B\lfloor n/B\rfloor}$，然后询问 $a^k$ 将其化为 $a^{k\bmod B+\lfloor k/B\rfloor B}=a^{k\bmod B}\times (a^B)^{\lfloor k/B\rfloor}$ 即可，所以这样预处理的复杂度是 $O(B+n/B)$，当 $B=\sqrt n$ 时最优为 $O(\sqrt n)$ 的复杂度。

为了方便书写，这里让 $a^i$ 与 $a^{Bi}$ 最高都处理到 $B-1$，这样表达的最大幂次是 $B(B-1)+B-1=B^2-1$，也就若幂次最高为 $n$ 只要让 $B^2-1\ge n$ 即可。

```cpp
int el[B], blo[B];

void init(int x) {
    el[0] = blo[0] = 1;

    for (int i = 1; i < B; i++)
        el[i] = el[i - 1] * x % P;

    int b = el[B - 1] * x % P;

    for (int i = 1; i < B; i++)
        blo[i] = blo[i - 1] * b % P;
}

int get(int k) {
    return blo[k / B] * el[k % B] % P;
}
```

