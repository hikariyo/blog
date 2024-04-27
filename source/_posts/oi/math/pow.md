---
date: 2023-06-17 15:01:43
update: 2024-01-15 03:29:00
title: 数学 - 快速幂与光速幂
katex: true
tags:
- Algo
- C++
categories:
- OI
---

## 快速幂

快速幂是在 $O(\log k)$ 的复杂度下计算 $a^k$ 的结果，一般为了防止数太大结果需要对 $p$ 取模。

或者可以用二进制的方式理解，将 $k$ 拆成 $\sum_{i=0}^N b_i\cdot2^i(b_i\in\{0, 1\})$ 的形式，然后放到指数上，由指数的运算性质可以知道 $a^k=a^{2^0\cdot b_0}\cdot a^{2^1\cdot b_1}\cdots a^{2^n\cdot{b_n}}$，所以这一串的底数是 $a^1, a^2, a^4, a^8, \cdots, a^{2^n}$，于是可以写成下面的形式：

```cpp
ll fastpow(ll a, ll b, ll p) {
    ll ans = 1;
    while (b) {
        if (k & 1) ans = (ans * a) % p;
        a = (a*a) % p;
        k >>= 1;
    }
    return ans;
}
```

## 光速幂

当底数不变时，可以做到 $O(\sqrt n)$ 预处理，$O(1)$ 回答的复杂度，原理是分块，预处理出 $a^1,a^2,\dots,a^{B-1}$ 与 $a^B,a^{2B},\dots,a^{n/B}$，然后询问 $a^k$ 将其化为 $a^{k\bmod B+\lfloor k/B\rfloor}$ 即可，所以这样预处理的复杂度是 $O(B+n/B)$，当 $B=\sqrt n$ 时最优。

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

