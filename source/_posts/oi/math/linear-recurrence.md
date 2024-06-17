---
date: 2024-06-17 20:14:00
title: 数学 - 常系数齐次线性递推
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
description: 对于一个线性递推，利用多项式取模可以做到比矩阵优秀很多的复杂度，当然也比较难写。
---

## 多项式乘法逆

对于 $n$ 次多项式 $F(x)$，求出 $n$ 次多项式 $G(x)$ 满足 $F(x)G(x)\equiv 1\pmod {x^n}$。

考虑倍增法，若求出 $G_1(x)$ 满足：
$$
F(x)G_1(x)\equiv 1\pmod{x^{\lceil\frac{n}{2}\rceil}}
$$
移项并平方可以得到（注意 $A(x)\equiv 1\pmod {x^{\lceil\frac{n}{2}\rceil}}$ 并不能推出 $A^2(x)\equiv 1\pmod {x^n}$，只有 $A(x)\equiv 0\pmod{x^{\lceil\frac{n}{2}\rceil}}$ 才能这么做，考虑按照定义写成等号的形式理解）：
$$
\Big(F(x)G_1(x)\Big)^2+1-2F(x)G_1(x)\equiv 0\pmod {x^n}
$$
同乘 $G(x)$ 并移项可以得到：
$$
G(x)\equiv G_1(x)\Big(2-F(x)G_1(x)\Big)^2\pmod{x^n}
$$
这样做的复杂度是 $T(n)=T(n/2)+O(n\log n)=O(n\log n)$，代码写在最后。

## 多项式取模

对于 $n$ 次多项式 $F(x)$ 与 $m$ 次多项式 $G(x)$，若存在 $n-m$ 次多项式 $Q(x)$ 与 $m-1$ 次多项式 $R(x)$ 满足：
$$
F(x)=G(x)Q(x)+R(x)
$$
称 $Q(x),R(x)$ 为 $F(x)$ 除以 $G(x)$ 的商与模，这里允许 $R(x)$ 的系数中存在 $0$。

思考这个应该怎么做，如果能找到一个方法消掉某一个多项式就好做了，当然消掉只能用 $\bmod x^k$ 的方法。

这里 $F(x),G(x)Q(x)$ 都是 $n$ 次，$R(x)$ 是 $m-1$ 次，所以这里有一种方法：
$$
x^nF(\frac{1}{x})=x^mG(\frac{1}{x})x^{n-m}Q(\frac{1}{x})+x^{n-m+1}x^{m-1}R(\frac{1}{x})
$$
设 $F_R(x)$ 代表系数反过来的多项式，即 $[x^i]F(x)=[x^{n-i}]F_R(x)$，容易知道 $x^nF(\frac{1}{x})=F_R(x)$，因此：
$$
F_R(x)=G_R(x)Q_R(x)+x^{n-m+1}R_R(x)
$$
因为 $Q(x)$ 是 $n-m$ 次的，所以这里可以两边对 $x^{n-m+1}$ 取模：
$$
F_R(x)\equiv G_R(x)Q_R(x)\pmod{x^{n-m+1}}
$$
也就是：
$$
Q_R(x)\equiv\frac{F_R(x)}{G_R(x)}\pmod{x^{n-m+1}}
$$
求出 $Q(x)$ 后反代入原式即可求出 $R(x)=F(x)-G(x)Q(x)$。这样复杂度是 $O(n\log n)$。

## 常系数齐次线性递推

> 给定 $n,k,f_1\sim f_k,a_0\sim a_{k-1}$，若：
> $$
> a_i= \sum_{j=1}^kf_ja_{i-j}
> $$
> 求 $a_n\bmod 998244353$ 的值，其中 $n=10^9,k=32000$。
>
> 题目链接：[P4723](https://www.luogu.com.cn/problem/P4723)。 

考虑一个朴素的斐波那契数列的展开过程：
$$
\begin{aligned}
a_5&=a_4+a_3\\
&=2a_3+a_2\\
&=3a_2+2a_1\\
&=5a_1+3a_0
\end{aligned}
$$
如果从一个多项式的角度来看：
$$
0x^0+0x^1+0x^2+0x^3+0x^4+\textcolor{red}{1}x^5\\
0x^0+0x^1+0x^2+\textcolor{red}1x^3+\textcolor{red}1x^4+0x^5\\
0x^0+0x^1+\textcolor{red}1x^2+\textcolor{red}2x^3+0x^4+0x^5\\
0x^0+\textcolor{red}2x^1+\textcolor{red}3x^2+0x^3+0x^4+0x^5\\
\textcolor{red}3x^0+\textcolor{red}5x^1+0x^2+0x^3+0x^4+0x^5\\
$$
做的事情就是不断地减去 $\lambda x^\mu(x^2-x-1)$，这就是一个取模的过程。因此 $a_n$ 的取值就 $x^n\bmod F(x)$ 有关，其中 $F(x)$ 是数列的特征多项式。具体地：
$$
F(x)=x^k+\sum_{j=1}^k-f_jx^{k-j}
$$
则 $a_n$ 满足的式子是：
$$
a_n=\sum_{i=0}^{k-1} [x^i](x^n\bmod F(x))a_i
$$
当然，需要 $a_0\sim a_{k-1}$​ 已知，否则也推不出来定值。多项式的题目最经典的错误是没有清空数组，只能说多注意一下吧，这种错误很难调。有时间我再整理一个更好看一点的板子，现在先用数组凑合一下。

这样快速幂加上取模的复杂度是 $O(k\log k\log n)$，如果是矩阵乘法需要 $O(k^3\log n)$，还是有显著的优化的。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
constexpr int N = 650010, P = 998244353;
int rev[N];

constexpr int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

void NTT(int A[], int lim, int base) {
    for (int i = 0; i < lim; i++) if (i < rev[i]) swap(A[i], A[rev[i]]);
    
    for (int mid = 1; mid < lim; mid <<= 1) {
        int len = mid * 2;
        int g1 = qmi(base, (P-1) / len);
        for (int i = 0; i < lim; i += len) {
            int g = 1;
            for (int j = 0; j < mid; j++, g = g * g1 % P) {
                int A0 = A[i+j], A1 = g * A[i+j+mid] % P;
                A[i+j] = (A0 + A1) % P, A[i+j+mid] = (A0 - A1 + P) % P;
            }
        }
    }
}

void mul(int F[], const int G[], int n, int m, int k) {
    static int A[N];
    int lim = 1, lgn = 0;
    for (int i = 0; i <= m; i++) A[i] = G[i];
    while (lim <= (n+m)) lim <<= 1, lgn++;
    for (int i = m+1; i < lim; i++) A[i] = F[i] = 0;
    for (int i = 1; i < lim; i++) rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (lgn - 1)); 
    NTT(F, lim, 3), NTT(A, lim, 3);
    for (int i = 0; i < lim; i++) F[i] = F[i] * A[i] % P;
    NTT(F, lim, qmi(3, P-2));
    int invn = qmi(lim, P-2);
    for (int i = 0; i <= k; i++) F[i] = F[i] * invn % P;
    for (int i = k+1; i < lim; i++) F[i] = 0;
}

// Clear numbers exceed n
void inv(const int F[], int G[], int n) {
    static int A[N];
    if (n == 1) return G[0] = qmi(F[0], P-2), void();
    inv(F, G, (n+1) >> 1);
    int lim = 1, lgn = 0;
    while (lim <= (n << 1)) lgn++, lim <<= 1;
    for (int i = 1; i < lim; i++) rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (lgn - 1)); 
    for (int i = ((n+1) >> 1)+1; i < lim; i++) G[i] = 0;

    for (int i = 0; i <= n; i++) A[i] = F[i];
    for (int i = n+1; i < lim; i++) A[i] = 0;

    NTT(G, lim, 3), NTT(A, lim, 3);
    for (int i = 0; i < lim; i++) {
        G[i] = ((2LL - A[i] * G[i]) % P) * G[i] % P;
        if (G[i] < 0) G[i] += P;
    }
    NTT(G, lim, qmi(3, P-2));
    int invn = qmi(lim, P-2);
    for (int i = 0; i <= n; i++) G[i] = G[i] * invn % P;
    for (int i = n+1; i < lim; i++) G[i] = 0;
}

void div(const int FR[], const int GR[], int Q[], int R[], int n, int m) {
    static int A[N], IGR[N];
    inv(GR, IGR, n-m);
    for (int i = 0; i <= n-m; i++) Q[i] = FR[i];
    mul(Q, IGR, n-m, n-m, n-m);
    reverse(Q, Q+n-m+1);
    for (int i = 0; i < m; i++) A[i] = GR[m-i];
    mul(A, Q, m-1, m-1, m-1);
    for (int i = 0; i < m; i++) R[i] = (FR[n-i] - A[i] + P) % P;
}

int res, n, k, f[N], a[N], A[N];

void fqmi(int A[], int n, const int PR[]) {
    static int R[N], Q[N], A2[N], R2[N];
    R[0] = 1;
    while (n) {
        if (n & 1) {
            // R = R * A % P
            mul(R, A, k-1, k-1, 2*k-2);
            reverse(R, R+2*k-2+1);
            div(R, PR, Q, R2, 2*k-2, k);
            for (int i = 0; i < k; i++) R[i] = R2[i];
            for (int i = k; i <= 2*k-2; i++) R[i] = 0;
        }
        // A = A * A % P
        for (int i = 0; i < k; i++) A2[i] = A[i];
        mul(A2, A, k-1, k-1, 2*k-2);
        reverse(A2, A2+2*k-2+1);
        div(A2, PR, Q, A, 2*k-2, k);
        for (int i = k; i <= 2*k-2; i++) A[i] = 0;
        n >>= 1;
    }
    for (int i = 0; i < k; i++) A[i] = R[i];
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    cin >> n >> k;
    for (int i = 1; i <= k; i++) {
        cin >> f[i];
        f[i] = ((-f[i]) % P + P) % P;
    }
    f[0] = 1;
    for (int i = 0; i < k; i++) {
        cin >> a[i];
        a[i] = (a[i] % P + P) % P;
    }
    A[1] = 1;
    fqmi(A, n, f);
    for (int i = 0; i < k; i++) res = (res + A[i] * a[i]) % P;
    cout << res << endl;
    return 0;
}
```

