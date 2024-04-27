---
date: 2024-01-15 02:50:00
title: 数学 - 中国剩余定理 CRT
description: 中国剩余定理是用来合并同余方程组的算法，当满足模数两两互质时有一种巧妙的构造解的方法，当不满足模数互质时同样可以通过两两合并的方式得到最终解。
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 简介

中国剩余定理是用来合并同余方程组的算法，当满足模数两两互质时有一种巧妙的构造解的方法，当不满足模数互质时同样可以通过两两合并的方式得到最终解。

## [模板] CRT

> 给定 $n$ 个方程，求 $x$ 的最小解。
> $$
> \begin{cases}
> x\equiv a_1\pmod {m_1}\\
> x\equiv a_2\pmod {m_2}\\
> \dots\\
> x\equiv a_n\pmod {m_n}
> \end{cases}
> $$
> 数据范围：$1\le n\le 10$，$0\le a_i\lt m_i\le 10^5$，$1\le \prod m_i\le 10^{18}$，满足 $\forall i\ne j,(m_i,m_j)=1$。
>
> 题目链接：[P1495](https://www.luogu.com.cn/problem/P1495)。

如果我们能构造出来一组系数 $k_i$ 使得 $\forall i\ne j,k_i\equiv 0\pmod {m_j}$ 并且 $k_i\equiv 1\pmod{m_i}$，那么一个合法解就是 $x=\sum k_ia_i$。

解集是 $x\equiv \sum k_ia_i \pmod{\prod m_i}$，这是因为 $\forall j\in [1,n],\prod m_i\equiv 0\pmod {m_j}$，并且在下面的 exCRT 中会证明实际上是在模 $\text{LCM}_{i-1}^nm_i$ 的剩余系中最多只有一个解。

令 $M_i=\frac{\prod m_j}{m_i}$，并且求出它模 $m_i$ 的逆元 $T_i$，那么 $k_i=M_iT_i$ 就是一个满足要求的构造，逆元用 exgcd 求解。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int __int128

namespace IO { /* 省略 */ }
using IO::read;
using IO::write;

const int N = 12;
int n, m[N], a[N];

pair<int, int> exgcd(int a, int b) {
    if (!b) return {1, 0};
    auto [y, x] = exgcd(b, a % b);
    return {x, y - a / b * x};
}

signed main() {
    int prod = 1, res = 0;
    read(n);
    for (int i = 1; i <= n; i++) read(m[i]);
    for (int i = 1; i <= n; i++) read(a[i]), prod *= a[i];
    for (int i = 1; i <= n; i++) {
        int Mi = prod / a[i];
        int Ti = exgcd(Mi, a[i]).first;
        res = (res + Mi * Ti * m[i]) % prod;
    }
    if (res < 0) res += prod;
    return write(res), 0;
}
```

## [模板] exCRT

> 给定 $n$ 个方程，求 $x$ 的最小解。
> $$
> \begin{cases}
> x\equiv a_1\pmod {m_1}\\
> x\equiv a_2\pmod {m_2}\\
> \dots\\
> x\equiv a_n\pmod {m_n}
> \end{cases}
> $$
> 数据范围：$1\le n\le 10$，$0\le a_i\lt m_i\le 10^{12}$，所有 $m_i$ 的最小公倍数不超过 $10^{18}$，保证一定有解。
>
> 题目链接：[P4777](https://www.luogu.com.cn/problem/P4777)。

首先，考虑如何合并两个方程的解，写成带余除法的形式，即 $x=k_1m_1+a_1$ 且 $x=k_2m_2+a_2$，移项可以得到 $k_1m_1-k_2m_2=a_2-a_1$。

这是一个标准的 exgcd 方程，求出 $k_1,k_2$ 后就找到了一个合法解 $x_0=k_1m_1+a_1$，给 $x_0$ 加上 $\text{lcm}(m_1,m_2)$ 的整数倍并不影响它是两个方程的解，下面证明模 $\text{lcm}(m_1,m_2)$ 的剩余系中只存在一个解。

假设 $\exist x, y\lt \text{lcm}(m_1,m_2),x\ne y$ 都是这两个方程的解，那么：
$$
\begin{cases}
x\equiv a_1\pmod {m_1}\\
x\equiv a_2\pmod {m_2}
\end{cases}\begin{cases}
y\equiv a_1\pmod {m_1}\\
y\equiv a_2\pmod {m_2}
\end{cases}
$$
可以推出：
$$
\begin{cases}
x-y\equiv 0\pmod {m_1}\\
x-y\equiv 0\pmod {m_2}
\end{cases}\Rightarrow x-y\equiv 0\pmod{\text{lcm}(m_1,m_2)}
$$
所以在模 $\text{lcm}(m_1,m_2)$ 的剩余系中最多只有一个解，因此我们可以合并两个方程为一个新的方程：
$$
x\equiv x_0\pmod{\text{lcm}(m_1,m_2)}
$$
两两合并得到最终结果，这个题可以用龟速乘防止溢出，也可以用 `__int128`，这里使用后者，速度更快。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int __int128

namespace IO { /* 省略 */ }
using IO::read;
using IO::write;

int exgcd(int a, int b, int& x, int& y) {
    if (!b) return x = 1, y = 0, a;
    int d = exgcd(b, a % b, y, x);
    y -= a / b * x;
    return d;
}

void merge(int a1, int a2, int m1, int m2, int& A, int& M) {
    int k1, k2, d = exgcd(m1, m2, k1, k2);
    k1 *= (a2 - a1) / d;
    M = m1 / d * m2;
    A = (k1 * m1 + a1) % M;
    if (A < 0) A += M;
}

signed main() {
    int n, a1, m1, a2, m2;
    read(n, m1, a1);
    for (int i = 2; i <= n; i++) {
        read(m2, a2);
        merge(a1, a2, m1, m2, a1, m1);
    }
    return write(a1), 0;
}
```

## [TJOI2009] 猜数字

> 给定 $n$ 个元素的数列 $a,b$ 问满足 $b_i\mid (x-a_i)$ 的最小整数解，$1\le n\le 10$，$|a_i|\le 10^9$，$1\le \prod b_i\le 10^{18}$，满足 $b_i$ 两两互质。
>
> 题目链接：[P3868](https://www.luogu.com.cn/problem/P3868)。

化简一下就是 $x\equiv a_i\pmod {b_i}$，所以直接上 CRT 即可，代码与模板题相同。
