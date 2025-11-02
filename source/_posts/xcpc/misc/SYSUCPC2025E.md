---
date: 2025-11-02 17:53:00
title: "[SYSUCPC 2025 E] Ecosystem"
katex: true
tags:
- Algo
- C++
- Math
- DP
categories:
- XCPC
description: 关于矩阵加速 DP 的那些事。
---

## 题目

[Gym 106114 E](https://codeforces.com/gym/106114/problem/E)。

## 思路

VP 时一直在想 $dp_{i,j}$，对于前 $i$ 个动物 $t=j$ 的方案，结果这么做是根本不行的。

正确思路是，和“上台阶可以走一步或两步问方案数”这个幼儿园 DP 入门题一样，我们设 $f_j$ 表示 $t=j$ 的方案。

为什么？因为加入每一个动物都认为是一种不同的方案，所以 $f_j=\sum_{i=1}^nf_{j-a_i}$，我们要从 $t=j$ 的角度看，这是满足无后效性的。

所以，把这个式子转化一下，成为：
$$
f_j=\sum_{i=1}^{100} f_{j-i} c_i
$$
就是一个标准的矩阵乘法 $F_j=AF_{j-1}$ 了，具体地说：
$$
\begin{bmatrix}
f_{j}\\
f_{j-1}\\
\vdots\\
f_{j-99}
\end{bmatrix}=

\begin{bmatrix}
c_1&c_2&\cdots&c_{99}&c_{100}\\
1&0&\cdots&0&0\\
0&1&\cdots&0&0\\
\vdots&\vdots&&\vdots&\vdots\\
0&0&\cdots&1&0
\end{bmatrix}

\begin{bmatrix}
f_{j-1}\\
f_{j-2}\\
\vdots\\
f_{j-100}
\end{bmatrix}
$$
对于初始化，我们有：
$$
F_0=\begin{bmatrix}
f_0\\
f_{-1}\\
\vdots\\
f_{-100}
\end{bmatrix}
=
\begin{bmatrix}
1\\
0\\
\vdots\\
0
\end{bmatrix}
$$
令转移矩阵为 $A$，只需要求 $A^tF_0$ 即可。

但是如果是矩阵快速幂的话，复杂度是 $O(mn^3\log t)$，难以接受。

我们有矩阵乘法的结合律，可以在 $O(n^3\log t)$ 的复杂度预处理出 $A^{2^i}$，按照快速幂的思路，每次算一个矩阵乘向量，这样复杂度就是 $O(n^3\log t+mn^2\log t)$，可以通过。

## 实现

下面的 `calc` 函数就是模仿快速幂的求矩阵的幂乘向量的那个函数。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 100, P = 1e9+7, K = 40;

struct Vector {
    int dat[N];
};

struct Matrix {
    int dat[N][N];
    Matrix operator*(const Matrix& m) const {
        Matrix res;
        for (int i = 0; i < N; i++)
            for (int j = 0; j < N; j++)
                res.dat[i][j] = 0;

        for (int i = 0; i < N; i++)
            for (int j = 0; j < N; j++)
                for (int k = 0; k < N; k++)
                    res.dat[i][j] = (res.dat[i][j] + dat[i][k] * m.dat[k][j]) % P;
        return res;
    }

    Vector operator*(const Vector& v) const {
        Vector res;
        for (int i = 0; i < N; i++)
            res.dat[i] = 0;
        for (int i = 0; i < N; i++)
            for (int k = 0; k < N; k++)
                res.dat[i] = (res.dat[i] + dat[i][k] * v.dat[k]) % P;
        return res;
    }
};

int cnt[110];

Matrix power[K];

Vector calc(int k, Vector f) {
    int i = 0;
    Vector res = f;
    while (k) {
        if (k & 1) res = power[i] * res;
        i++;
        k >>= 1;
    }
    return res;
}

signed main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    int n, m;
    cin >> n >> m;
    for (int i = 1, a; i <= n; i++) {
        cin >> a;
        cnt[a]++;
    }

    Matrix A;
    memset(A.dat, 0, sizeof A.dat);
    for (int i = 1; i <= N; i++) A.dat[0][i-1] = cnt[i];
    for (int i = 1; i < N; i++) A.dat[i][i-1] = 1;

    power[0] = A;
    for (int i = 1; i < K; i++) power[i] = power[i-1] * power[i-1];

    Vector f;
    memset(f.dat, 0, sizeof f.dat);
    f.dat[0] = 1;

    for (int i = 1; i <= m; i++) {
        int t;
        cin >> t;
        cout << calc(t, f).dat[0] << '\n';
    }
    
    return 0;
}
```

