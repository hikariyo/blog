---
date: 2024-06-09 08:52:00
title: 数学 - 拉格朗日插值法
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
description: 用 n+1 个点可以唯一确定一个 n 次多项式，拉格朗日插值法给出一个构造此多项式的方法。
---

## 简介

对于 $n+1$ 个点 $(x_i,y_i)$，我们希望多项式 $f(x)$ 可以使得 $f(x_i)=y_i$。若构造 $f_i(x)$ 使得：
$$
f_i(x)=\begin{cases}
1\quad x=x_i\\
0\quad x\ne x_i
\end{cases}
$$
那么就可以直接构造出 $f(x)=\sum_{i=0}^n y_if_i(x)$，那么 $f_i(x)$ 的一种构造方法是：
$$
f_i(x)=\frac{\prod_{j\ne i}(x-x_j)}{\prod_{j\ne i} (x_i-x_j)}
$$
这样当且仅当 $x=x_i$ 时不为零，且值为 $1$。

## [模板] 拉格朗日插值

> 给定 $n$ 个点 $(x_i,y_i)$ 构造多项式 $f(x)$，给定整数 $k$ 求 $f(k)\bmod 998244353$。
>
> 数据范围：$1\le n\le 2\times 10^3,1\le x_i,y_i,k\lt 998244353$。
>
> 题目链接：[P4781](https://www.luogu.com.cn/problem/P4781)。

由于最后才计算，所以我们没必要求出整个多项式每一项的系数，直接代入计算即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 2010, P = 998244353;
int n, k, x[N], y[N];

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

signed main() {
    cin >> n >> k;
    for (int i = 1; i <= n; i++) cin >> x[i] >> y[i];
    int res = 0;
    for (int i = 1; i <= n; i++) {
        int val = 1;
        for (int j = 1; j <= n; j++)
            if (i != j)
                val = val * qmi(x[i] - x[j], P-2) % P * (k - x[j]) % P;
        res = (res + y[i] * val) % P;
    }
    if (res < 0) res += P;
    cout << res << endl;
    return 0;
}
```

## [CF622F] Sum of the k-th Powers

> 给定 $n,k$ 求 $\sum_{i=1}^n i^k\bmod (10^9+7)$，其中 $1\le n\le 10^9, 1\le k \le 10^6$。
>
> 题目链接：[CF622F](https://codeforces.com/problemset/problem/622/F)。

首先令 $f(n)=\sum_{i=1}^n i^k$，可以证明这是一个 $k+1$​ 次的多项式，参考本站[数学基础公式及证明](https://blog.hikariyo.net/oi/math/math-basic)。

如果 $k$ 的范围比较小，支持 $O(k^3)$ 的做法，那么这里有一个矩阵的做法。设 $F_i=\begin{bmatrix}i^k\\\vdots\\i^0\\S_{i-1}\end{bmatrix}$，由于 $(i+1)^k=\sum_{j=0}^k\binom{k}{j}i^j$，所以构造这样一个矩阵进行转移：
$$
\begin{bmatrix}
\binom{k}{k}&\cdots&\binom{k}{0}&0\\
&\ddots&\vdots&\vdots\\
&&\binom{0}{0}&0
\\
1&&&1
\end{bmatrix}\begin{bmatrix}
i^k\\
\vdots\\
i^0\\
S_{i-1}
\end{bmatrix}=\begin{bmatrix}
(i+1)^k\\
\vdots\\
(i+1)^0\\
S_i
\end{bmatrix}
$$
遗憾的是，本题的 $k$ 达到了 $10^6$，这样是肯定不行的。

下面考虑拉格朗日插值法，并且用 $k+2$ 个点 $(1,f(1)),\dots,(k+1,f(k+1)),(k+2,f(k+2))$ 来插值（即 $x_i=i$），所以 $f(x)$ 的拉格朗日插值式为：
$$
\begin{aligned}
f(x)&=\sum_{i=1}^{k+2}f(i)\frac{\prod_{j\ne i}(x-x_j)}{\prod_{j\ne i}(x_i-x_j)}\\
&=\sum_{i=1}^{k+2}f(i)\frac{\prod_{j\ne i}(x-j)}{\prod_{j\ne i}(i-j)}\\
\end{aligned}
$$
对于分母，易知是 $i-(k+2)$ 到 $i-1$ 除了 $0$ 外的连续积，可以预处理；对于分子，可以预处理 $n-j$ 的前后缀积。不记快速幂的时间，这样就可以做到 $O(k)$ 求出 $f(n)$。由于 $f(n)$ 是多项式，所以可以先 $n\gets n\bmod P$。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int K = 1000010, P = 1e9+7;
int n, k, f[K], pn[K], sn[K], ifact[K];

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

signed main() {
    cin >> n >> k;
    int m = k+2, factm = 1;
    for (int i = 1; i <= m; i++) f[i] = (f[i-1] + qmi(i, k)) % P, factm = factm * i % P;
    ifact[m] = qmi(factm, P-2);
    for (int i = m-1; i >= 0; i--) ifact[i] = ifact[i+1] * (i+1) % P;
    pn[0] = sn[m+1] = 1;
    // 这里 n 如果很大 可能导致溢出
    n %= P;
    for (int i = 1; i <= m; i++) pn[i] = pn[i-1] * (n - i) % P;
    for (int i = m; i >= 0; i--) sn[i] = sn[i+1] * (n - i) % P;
    int res = 0;
    for (int i = 1; i <= m; i++) {
        int v1 = f[i] * pn[i-1] % P * sn[i+1] % P * ifact[m-i] % P * ifact[i-1] % P;
        if ((m - i) % 2) v1 = -v1;
        if (v1 < 0) v1 += P;
        res = (res + v1) % P;
    }
    cout << res << endl;
    return 0;
}
```

