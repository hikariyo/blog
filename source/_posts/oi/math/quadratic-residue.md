---
date: 2024-01-22 00:24:00
title: 数学 - 二次剩余
description: 二次剩余即模意义开根，当模数为奇素数时，用 Cipolla 算法利用扩域的思想进行求解。
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 简介

二次剩余即模意义开根，求解形如 $x^2\equiv n\pmod p$ 的方程，若方程存在非 $0$ 解，称 $n$ 是模 $p$ 的二次剩余，欧拉准则可以判断 $n$ 是否为模 $p$ 的二次剩余，本文只讨论 $p$ 为奇素数的情况。

## 欧拉准则

当 $p$ 为奇素数时，根据费马小定理：
$$
\begin{aligned}
n^{2{\frac{p-1}{2}}}&\equiv 1\pmod p\\
(n^{\frac{p-1}{2}}-1)(n^{\frac{p-1}{2}}+1)&\equiv 0\pmod p\\
\end{aligned}
$$
为方便表达引入勒让德符号：
$$
(\frac{n}{p})\equiv n^{\frac{p-1}{2}}\pmod p
$$
下面证明 $n$ 为模 $p$ 的二次剩余是 $(\frac{n}{p})\equiv 1\pmod p$ 的充要条件。

+ 充分性：存在 $x^2\equiv n\pmod p$ 那么 $n^{\frac{p-1}{2}}\equiv x^{p-1}\equiv 1\pmod p$。
+ 必要性：设 $g$ 为模 $p$ 的原根，由于 $p$ 为奇素数所以一定存在，令 $g^k\equiv n\pmod p$，那么 $g^{k\frac{p-1}{2}}\equiv 1\pmod p$，那么就有 $k\frac{p-1}{2}\equiv 0\pmod {\varphi(p)}$，也就是说 $\frac{k}{2}(p-1)$ 是 $p-1$ 的倍数，所以 $k$ 是偶数，那么 $x=g^{\frac{k}{2}}\pmod p$ 就是原方程的一个解。

关于解的数量，当 $x_1$ 是原方程的一个解时，若存在 $x_2\not\equiv x_1\pmod p$ 并且 $x_1^2\equiv x_2^2$ 可以推出：
$$
(x_1-x_2)(x_1+x_2)\equiv 0\pmod p
$$
所以只有可能是 $x_1\equiv -x_2\pmod p$，方程有两个解，当 $n=0$ 时 $x_1\equiv x_2\equiv0\pmod p$。

## Cipolla

首先找到一个 $a$ 使得 $(\frac{a^2-n}{p})\equiv -1\pmod p$，设 $g$ 为模 $p$ 的原根，且 $m\equiv g^k\pmod p$ 那么 $m^{\frac{p-1}{2}}\equiv g^{k\frac{p-1}{2}}\pmod p$，$k$ 的奇偶性决定 $m$ 是否为二次剩余，因此期望两次就可以找到满足条件的 $a$。

类比虚数的定义，令 $i^2\equiv a^2-n\pmod p$，那么有一个很好的性质：
$$
i^p\equiv i\cdot i^{p-1}\equiv i(a^2-n)^{\frac{p-1}{2}}\equiv -i\pmod p
$$
下面证明 $(a+i)^{p+1}\equiv n\pmod p$，其中 $(a+i)^p\equiv (a^p+i^p)\pmod p$ 原理是二项式定理：
$$
\begin{aligned}
(a+i)^{p+1}&\equiv(a+i)^p(a+i)\\
&\equiv(a^p+i^p)(a+i)\\
&\equiv(a-i)(a+i)\\
&\equiv a^2-i^2\\
&\equiv n\pmod p
\end{aligned}
$$
所以 $x=(a+i)^{\frac{p+1}{2}}$ 就是原方程的一个解，下面用反证法证明 $x$ 的虚部为 $0$：

设存在 $c,d$ 使得 $(c+di)^2\equiv n\pmod p$ 且 $d\ne 0$，那么有：
$$
c^2+2cdi+d^2(a^2-n)\equiv n\pmod p\\
$$
即然 $d\ne 0$ 一定是 $c=0$ 否则无法满足等式，此时：
$$
i^2\equiv nd^{-2}\pmod p
$$
由于 $d^{-2}$ 一定是二次剩余，并且 $n$ 也是二次剩余，所以 $i^2$ 也是二次剩余，与假设矛盾，所以 $x$ 的虚部一定为 $0$。

因此算法的时间复杂度主要在求幂上，用快速幂做到 $O(\log p)$，题目是 [P5491](https://www.luogu.com.cn/problem/P5491)。

```cpp
#include <bits/stdc++.h>
using namespace std;

namespace IO {}
using IO::read;
using IO::write;

#define int long long
int T, n, p, i2;

struct Complex {
    int real, imag;
    Complex operator*(const Complex& c) const {
        // 这里三个大数相乘注意取模
        return {(real * c.real + imag * c.imag % p * i2) % p, (real * c.imag + imag * c.real) % p};
    }
};

mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % p;
        a = a * a % p;
        k >>= 1;
    }
    return res;
}

Complex qmi(Complex a, int k) {
    Complex res = {1, 0};
    while (k) {
        if (k & 1) res = res * a;
        a = a * a;
        k >>= 1;
    }
    return res;
}

int cipolla() {
    if (!n) return 0;
    if (qmi(n, (p-1) / 2) != 1) return -1;

    int a;
    do {
        a = uniform_int_distribution<>(1, p-1)(rng);
        i2 = (a * a - n) % p;
        if (i2 < 0) i2 += p;
    } while (qmi(i2, (p-1) / 2) == 1);

    return qmi(Complex{a, 1}, (p+1) / 2).real;
}

signed main() {
    read(T);
    while (T--) {
        read(n, p);
        int x = cipolla(), y = p-x;
        if (x == 0) write(0);
        else if (x == -1) write("Hola!");
        else x > y && (swap(x, y), 0), write(x, ' '), write(y);
    }
    return 0;
}
```

## 扩域

Cipolla 算法用到了扩域的思想，那么这个思想其实可以应用到诸多递推数列上，如果存在通项公式，就能通过扩域的方法求解。

众所周知，斐波那契的通项公式是：
$$
f_n=\frac{1}{\sqrt 5}\left(\Big(\frac{1+\sqrt 5}{2}\Big)^n-\Big(\frac{1-\sqrt 5}{2}\Big)^n\right)
$$
如果模数 $p=10^9+7$，那么 $5$ 是非二次剩余，但是可以这样定义：
$$
i^2\equiv 5\pmod p
$$
此时可以改写通项公式了：
$$
f_n\equiv \frac{i}{5}\left(\Big(\frac{1+i}{2}\Big)^n-\Big(\frac{1-i}{2}\Big)^n\right)\pmod p
$$
实现了 [P1962](https://www.luogu.com.cn/problem/P1962)：

```cpp
#include <bits/stdc++.h>
using namespace std;

namespace IO {}
using IO::read;
using IO::write;

#define int long long
const int P = 1000000007, I5 = 400000003, I2 = 500000004;
int n;

struct Complex {
    int real, imag;

    Complex operator*(const Complex& c) const {
        return {(real * c.real + imag * c.imag * 5) % P, (real * c.imag + imag * c.real) % P};
    }

    Complex operator-(const Complex& c) const {
        int a = (real - c.real) % P, b = (imag - c.imag) % P;
        if (a < 0) a += P;
        if (b < 0) b += P;
        return {a, b};
    }
};

Complex qmi(Complex a, int k) {
    Complex res = {1, 0};
    while (k) {
        if (k & 1) res = res * a;
        a = a * a;
        k >>= 1;
    }
    return res;
}

signed main() {
    read(n);
    Complex a = {0, I5}, b = qmi({I2, I2}, n) - qmi({I2, P-I2}, n);
    return write((a * b).real), 0;
}
```

