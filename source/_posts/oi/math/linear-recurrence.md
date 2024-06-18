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
G(x)\equiv G_1(x)\Big(2-F(x)G_1(x)\Big)\pmod{x^n}
$$
这样做的复杂度是 $T(n)=T(n/2)+O(n\log n)=O(n\log n)$​，代码写在最后。

注意这里倍增的过程中 $F(x)$ 与 $G_1(x)$ 次数最高次是 $x^{n-1}$ 和 $x^{\lceil\frac{n}{2}\rceil-1}$，因此 $G(x)$ 的最高次应当是 $n-1+2\lceil\frac{n}{2}\rceil -2\lt 2n$，所以要把数组留到 $2n$ 的大小才足够我们进行循环卷积。

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
求出 $Q(x)$ 后反代入原式即可求出 $R(x)=F(x)-G(x)Q(x)$。这样复杂度是 $O(n\log n)$​。

如果 $F(x)$ 真实的次数，即 $F(x)$ 不为 $0$ 的最高次数比 $G(x)$ 小，那么 $F_R(x)$ 的最低 $n-m+1$ 项一定是 $0$，此时 $Q(x)=0$，$R(x)=F(x)$，仍然符合我们对 $\bmod$ 运算的认知。当然这个可以在代码中直接特判掉来优化。

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
当然，需要 $a_0\sim a_{k-1}$​ 已知，否则也推不出来定值。多项式的题目最经典的错误是没有清空数组，只能说多注意一下吧，这种错误很难调。这样快速幂加上取模的复杂度是 $O(k\log k\log n)$，如果是矩阵乘法需要 $O(k^3\log n)$​，还是有显著的优化的。

下面是我整理的一个比较好看一点的封装好的板子。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
constexpr int N = 32010, P = 998244353;

constexpr int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

struct Poly {
    int n, lim, lgn;
    vector<int> f, rev;

    Poly() = default;

    Poly(const vector<int>& g) {
        assert(g.size());
        resize(g.size() - 1);
        for (int i = 0; i < g.size(); i++) f[i] = g[i];
    }

    Poly& resize(int n) {
        this->n = n;
        lim = 1, lgn = 0;
        while (lim <= n) lim <<= 1, lgn++;
        f.resize(lim, 0);
        rev.resize(lim, 0);
        for (int i = n+1; i < lim; i++) f[i] = 0;
        for (int i = 1; i < lim; i++) rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (lgn - 1));
        return *this;
    }

    int& operator[](int x) {
        return f[x];
    }

    int operator[](int x) const {
        return f[x];
    }

    Poly& ntt(int p) {
        int base = 3;
        if (p == -1) base = qmi(3, P - 2);

        for (int i = 0; i < lim; i++) if (i < rev[i]) swap(f[i], f[rev[i]]);
    
        for (int mid = 1; mid < lim; mid <<= 1) {
            int len = mid * 2;
            int g1 = qmi(base, (P - 1) / len);
            for (int i = 0; i < lim; i += len) {
                int g = 1;
                for (int j = 0; j < mid; j++, g = g * g1 % P) {
                    int A0 = f[i + j], A1 = g * f[i + j + mid] % P;
                    f[i + j] = (A0 + A1) % P, f[i + j + mid] = (A0 - A1 + P) % P;
                }
            }
        }

        if (p == -1) {
            int invn = qmi(lim, P-2);
            for (int i = 0; i <= n; i++) f[i] = f[i] * invn % P;
            for (int i = n+1; i < lim; i++) f[i] = 0;
        }

        return *this;
    }

    Poly& reverse() {
        ::reverse(f.begin(), f.begin() + n + 1);
        return *this;
    }

    Poly operator*(const Poly& g) const {
        Poly F = *this, G = g;
        int n = F.n + G.n;
        F.resize(n).ntt(1), G.resize(n).ntt(1);
        for (int i = 0; i < F.lim; i++) F[i] = F[i] * G[i] % P;
        F.ntt(-1);
        return F;
    }

    Poly operator-(const Poly& g) const {
        int n = max(this->n, g.n);
        Poly R;
        R.resize(n);
        for (int i = 0; i <= n; i++) R[i] = (P + f[i] - g[i]) % P;
        return R;
    }

    Poly inv(int n = -1) const {
        if (n == -1) n = this->n + 1;
        if (n == 1) return Poly({qmi(f[0], P - 2)});
        Poly G = inv((n + 1) / 2);
        Poly F = *this;
        F.resize(n * 2);
        for (int i = n; i <= n * 2; i++) F[i] = 0;
        F.ntt(1), G.resize(n * 2).ntt(1);
        for (int i = 0; i < F.lim; i++) {
            F[i] = (2ll - F[i] * G[i]) % P * G[i] % P;
            if (F[i] < 0) F[i] += P;
        }
        return F.ntt(-1).resize(n - 1);
    }

    Poly operator/(const Poly& g) const {
        if (n < g.n) return Poly({0});
        Poly G = g, Q = *this;
        int q = n - G.n;
        Q.reverse().resize(q);
        return (Q = Q * G.reverse().inv(q + 1)).resize(q).reverse();
    }

    Poly operator%(const Poly& g) const {
        if (n < g.n) return *this;
        Poly Q = *this / g;
        return (*this - g * Q).resize(g.n - 1);
    }

    Poly fqmi(int k, const Poly& p) const {
        Poly res({1}), a = *this;
        while (k) {
            if (k & 1) res = res * a % p;
            a = a * a % p;
            k >>= 1;
        }
        return res;
    }
};


int n, k, res, a[N];
Poly A, B;

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    cin >> n >> k;
    B.resize(k);
    for (int i = 0; i < k; i++) cin >> B[k-i-1], B[k-i-1] = ((-B[k-i-1]) % P + P) % P;
    B[k] = 1;
    for (int i = 0; i < k; i++) cin >> a[i], a[i] = (a[i] % P + P) % P;
    A.resize(1);
    A[1] = 1;
    A = A.fqmi(n, B);
    for (int i = 0; i < k; i++) res = (res + A[i] * a[i]) % P;
    cout << res << endl;
    return 0;
}
```

