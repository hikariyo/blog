---
date: 2024-01-22 20:37:00
update: 2024-05-27 15:25:00
title: 数学 - Pollard Rho
katex: true
description: Pollard Rho 算法利用生日悖论，可以在优秀的复杂度内找到一个合数的非 1 非自身因数，使用时需要搭配 Miller Rabin 算法首先判断一个数是否为质数。
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 简介

Pollard Rho 算法利用生日悖论，可以在优秀的复杂度内找到一个合数的非 $1$ 非自身因数，使用时需要搭配 Miller Rabin 算法首先判断一个数是否为质数。

## 生日悖论

关于生日悖论，就是说 $k$ 个人中两个人生日相同的概率实际上很高，一年有 $n$ 天，这个概率是：
$$
P=1-\prod_{i=1}^{k-1}\frac{n-i}{n}
$$
下面进行一个近似计算，用到均值不等式和 $\exp(x)>x+1(x\ne 0)$ 这个经典结论：
$$
\begin{aligned}
\prod_{i=1}^k\frac{n-i}{n}&< (\frac{1}{k}\sum_{i=1}^k\frac{n-i}{n})^{k}\\
&=(1-\frac{1+k}{2n})^k\\
&<\exp(-\frac{k^2+k}{2n})
\end{aligned}
$$
代入 $k=4\sqrt n$  时，有：
$$
\exp[-(8+\frac{2}{\sqrt n})]<\exp(-8)\approx 0.0003
$$
因此随机地从 $1\sim n$ 中选择数字，$O(\sqrt n)$ 次会选到重复的数字。所以，当一个数 $n$ 是合数，首先它的最大质因子 $p$ 是 $O(\sqrt n)$ 级别的，在模 $p$ 的意义下找到两个相同的数字的复杂度是 $O(\sqrt p)$ 即 $O(n^{\frac{1}{4}})$。

## 原理

设 $n$ 的一个因子为 $p\in(1,n)$，随机取 $c\in[1,n)$，定义序列 $x_{i+1}=(x_i^2+c)\bmod n$ 并令 $x_1=0$，可以得到一个比较随机的 $0\sim n-1$ 的序列，根据鸽巢原理 $x_i$ 是一定存在环的，并且在模 $p$ 的意义下环的长度是 $O(\sqrt p)$。

若 $p\mid x_j-x_i$，那么就有 $(x_j-x_i,n)=kp$，并且 $x_{j+1}-x_{i+1}\equiv (x_j-x_i)(x_j+x_i)\pmod n$，也就是说 $p\mid x_{j+1}-x_{i+1}$，于是只要改变 $i,j$ 的相对大小就可以判断出环上间距为 $j-i$ 的 $\gcd$ 都是 $p$ 的倍数 。

期望找到环的复杂度是 $O(\sqrt p)$，并且同时用快慢指针，也是在 $O(\sqrt p)$ 的复杂度内找到 $(i,j)$ 使得 $p\mid x_j-x_i$，加上求 $\gcd$ 的复杂度，算法的期望复杂度是 $O(n^{\frac{1}{4}}\log n)$。

## 实现

> 输入不超过 $350$ 个数字，对于每个数字 $2\le n\le 10^{18}$ 检验是否是质数，是质数就输出 `Prime`；如果不是质数，输出它最大的质因子是哪个。
>
> 题目链接：[P4718](https://www.luogu.com.cn/problem/P4718)。

先用 Miller Rabin 判掉素数，然后再用 Pollard Rho 找因数，复杂度 $O(n^{\frac{1}{4}}\log^2 n)$，第二个 $\log$ 代表质因数的数量。

```cpp
#include <bits/stdc++.h>
using namespace std;

namespace IO {}
using IO::read;
using IO::write;

#define int long long
mt19937_64 rng(chrono::steady_clock::now().time_since_epoch().count());

int gcd(int a, int b) {
    if (!b) return a;
    return gcd(b, a % b);
}

int qmi(int a, int k, int p) {
    int res = 1;
    while (k) {
        if (k & 1) res = (__int128)res * a % p;
        a = (__int128)a * a % p;
        k >>= 1;
    }
    return res;
}

int rho(int n) {
    if (n % 2 == 0) return 2;
    int c = uniform_int_distribution<int>(1, n-1)(rng);
    auto f = [=](int x){ return ((__int128)x * x + c) % n; };
    int x = 0, y = f(x);
    while (x != y) {
        int d = gcd(abs(x - y), n);
        if (d > 1) return d;
        x = f(x), y = f(f(y));
    }
    return n;
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

int solve(int n) {
    static map<int, int> mp;
    if (mp.count(n)) return mp[n];

    if (mr(n)) return mp[n] = n;
    int x = n;
    while (x == n) x = rho(n);
    return mp[n] = max(solve(x), solve(n / x));
}

signed main() {
    int n, x, t;
    read(t);
    while (t--) read(n), ((x = solve(n)) == n) ? write("Prime") : write(x);
    return 0;
}
```

