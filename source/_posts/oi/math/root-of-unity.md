---
date: 2024-07-09 18:09:00
updated: 2024-07-09 18:09:00
title: 数学 - 单位根反演
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
description: 单位根反演是 FFT 的基础，并且它也可以解决一些其它的问题。
---

## 简介

与莫反类似，单位根反演是一个可以表示 $[n\mid k]$ 的式子，由此推出的一系列应用：
$$
\frac{1}{n}\sum_{i=0}^{n-1}\omega_n^{ki}=[n\mid k]
$$
当 $n\mid k$ 时，根据单位根的定义 $\omega_n^k=\exp(2\pi i\frac{k}{n})=1$，此时原式为 $1$；当 $n\not\mid k$ 时根据等比数列求和公式得到：
$$
\frac{1}{n}\sum_{i=0}^{n-1}(\omega_n^k)^i=\frac{1}{n}\frac{1-\omega_n^{nk}}{1-\omega_n^k}
$$
同样根据 $\omega_n^{nk}=\exp(2\pi i\frac{nk}{n})=1$，而 $\omega_n^k\ne 1$，因此原式值为 $0$​。

## FFT

回顾 FFT，对于一个 $n-1$ 次的多项式 $F(x)$，当我们知道 $(\omega_n^i,F(\omega_n^i))$ 时，想根据这个求出所有 $[x^k]F(x)$，可以写成这样：
$$
\begin{aligned}
\text{[}x^k]F(x)&=\sum_{j=0}^{n-1}[j=k][x^j]F(x)\\
&=\sum_{j=0}^{n-1}[n\mid (j-k)][x^j]F(x)\\
&=\sum_{j=0}^{n-1}\left(\frac{1}{n}\sum_{i=0}^{n-1}\omega_n^{(j-k)i}\right)[x^j]F(x)\\
&=\frac{1}{n}\sum_{j=0}^{n-1}\sum_{i=0}^{n-1}[x^j]F(x)\omega_n^{(j-k)i}\\
&=\frac{1}{n}\sum_{j=0}^{n-1}\sum_{i=0}^{n-1}[x^j]F(x)\omega_n^{ij}\omega_n^{-ki}\\
&=\frac{1}{n}\sum_{i=0}^{n-1}F(\omega_n^i)\omega_{n}^{-ki}\\
&=\frac{1}{n}\sum_{i=0}^{n-1}F(\omega_n^i)(\omega_n^{-k})^i
\end{aligned}
$$

也就是 $[x^k]F(x)$ 的值是以 $F(\omega_n^i)$ 为 $[x^i]G(x)$ 求 $G(\omega_n^{-k})$，而 DFT 的过程就是在求一个多项式在 $\omega_n^k$ 处的值，因此 IDFT 可以复用 DFT 的代码，只需要把 $\omega_n^k$ 换成 $\omega_n^{-k}$ 最后再乘一下 $\frac{1}{n}$ 即可。

用模 $p$ 的原根 $g_n^k=g^{(p-1)\frac{k}{n}}$​​ 来代替单位根可以避免精度问题。

## LJJ 学二项式定理

> 给定 $n,s,a_0\sim a_3$ 其中 $n$ 范围 $10^{18}$，其它 $10^8$。求：
> $$
> \left(\sum_{i=0}^n\binom{n}{i}s^ia_{i\bmod 4}\right)\bmod 998244353
> $$
> 题目链接：[LOJ6485](https://loj.ac/p/6485)。

考虑枚举 $i\bmod 4=k$，而 $[i\bmod 4=k]=[4\mid (i-k)]$，把同余条件转化为整除条件然后代入单位根反演的式子：
$$
\begin{aligned}
\sum_{i=0}^n\binom{n}{i}s^ia_{i\bmod 4}&=\sum_{k=0}^3\sum_{i=0}^n\binom{n}{i}s^ia_k[i\bmod 4=k]\\
&=\sum_{k=0}^3\sum_{i=0}^n\binom{n}{i}s^ia_k[4\mid (i-k)]\\
&=\frac{1}{4}\sum_{k=0}^3\sum_{j=0}^3\sum_{i=0}^n\binom{n}{i}s^i\omega_4^{j(i-k)}a_k\\
&=\frac{1}{4}\sum_{k=0}^3\sum_{j=0}^3\left(a_k\omega_4^{-kj}\sum_{i=0}^n\binom{n}{i}s^i\omega_4^{ji}\right)\\
&=\frac{1}{4}\sum_{k=0}^3\sum_{j=0}^3a_k\omega_4^{-kj}(s\omega_4^j+1)^n\\
\end{aligned}
$$
单组询问复杂度 $O(\log n)$，有 $16$ 的常数，代码如下：

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int P = 998244353, I4 = 748683265;
int T, n, s, a[4];

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
    cin >> T;
    while (T--) {
        cin >> n >> s >> a[0] >> a[1] >> a[2] >> a[3];
        int res = 0;
        for (int k = 0; k < 4; k++) {
            int w = qmi(3, (P-1)/4), negwk = qmi(3, P-1-((P-1)/4*k));
            for (int j = 0, w1 = 1, w2 = 1; j < 4; j++, w1 = w1 * negwk % P, w2 = w2 * w % P) {
                res = (res + a[k] * w1 % P * qmi((s * w2 + 1) % P, n)) % P;
            }
        }
        cout << res * I4 % P << endl;
    }
    return 0;
}
```

## [洛谷月赛] 小猪佩奇学数学

> 给定 $n,p,k$ 满足 $1\le n,p\lt 998244353,k\in\{2^w|0\le w\le 20\}$ 问：
> $$
> \sum_{i=0}^n\binom{n}{i}p^i\lfloor\frac{i}{k}\rfloor\bmod 998244353
> $$
> 题目链接：[P5591](https://www.luogu.com.cn/problem/P5591)。

根据模的定义，可以将原式子拆成两个部分：
$$
\begin{aligned}
\sum_{i=0}^n\binom{n}{i}p^i\lfloor\frac{i}{k}\rfloor&=\frac{1}{k}\sum_{i=0}^n\binom{n}{i}ip^i-\frac{1}{k}\sum_{i=0}^n\binom{n}{i}p^i(i\bmod k)
\end{aligned}
$$
对于左半部分，根据组合数的性质有：
$$
\frac{1}{k}\sum_{i=0}^n\binom{n}{i}ip^i=\frac{np}{k}\sum_{i=1}^n\binom{n-1}{i-1}p^{i-1}=\frac{np(p+1)^{n-1}}{k}
$$
对于右半部分，同样是枚举 $i\bmod k$ 的值：
$$
\begin{aligned}
\frac{1}{k}\sum_{r=0}^{k-1}\sum_{i=0}^n\binom{n}{i}p^ir[i\bmod k=r]&=\frac{1}{k}\sum_{r=0}^{k-1}\sum_{i=0}^n\binom{n}{i}p^ir[k\mid(i-r)]\\
&=\frac{1}{k^2}\sum_{r=0}^{k-1}\sum_{i=0}^n\binom{n}{i}p^ir\sum_{j=0}^{k-1}\omega_k^{(i-r)j}\\
&=\frac{1}{k^2}\sum_{r=0}^{k-1}\sum_{j=0}^{k-1}\sum_{i=0}^n\binom{n}{i}p^ir\omega_k^{ij}\omega_{k}^{-rj}\\
&=\frac{1}{k^2}\sum_{r=0}^{k-1}\sum_{j=0}^{k-1}r\omega_{k}^{-rj}\sum_{i=0}^n\binom{n}{i}p^i\omega_k^{ij}\\
&=\frac{1}{k^2}\sum_{j=0}^{k-1}(p\omega_k^j+1)^n\sum_{r=0}^{k-1}r\omega_{k}^{-rj}\\
\end{aligned}
$$
对于后边这个等差乘等比，这里推一个结论，错位相减法。
$$
\begin{aligned}
S_n&=\sum_{i=1}^{n-1}iq^i\\
qS_n&=\sum_{i=2}^{n}(i-1)q^{i}\\
(1-q)S_n&=-(n-1)q^n+\sum_{i=1}^{n-1}q^i\\
S_n&=\frac{(1-n)q^n}{1-q}+\frac{q(1-q^{n-1})}{(1-q)^2}\\
&=\frac{q-q^n+(1-q)(1-n)q^n}{(1-q)^2}\\
&=\frac{q-nq^n+(n-1)q^{n+1}}{(1-q)^2}
\end{aligned}
$$
当然 $q=1$ 时原式等于 $\frac{n(n-1)}{2}$，这个只能特判。带入得到：
$$
\begin{aligned}
\frac{1}{k^2}\sum_{j=0}^{k-1}(p\omega_k^j+1)^n\sum_{r=0}^{k-1}r\omega_{k}^{-rj}&=\frac{1}{k^2}\sum_{j=0}^{k-1}(p\omega_k^j+1)^nS_{k,\omega_k^{-j}}
\end{aligned}
$$
懒得根据单位根性质化简 $S_k$ 了，反正能 $O(1)$ 求出来，代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int P = 998244353, I2 = 499122177;
int n, p, k;

int qmi(int a, int k = P-2) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

int S(int n, int q) {
    if (q == 1) return n * (n-1) % P * I2 % P;
    int res = (q - n * qmi(q, n) + (n-1) * qmi(q, n+1)) % P;
    res = res * qmi((1-q) * (1-q) % P) % P;
    return (res + P) % P;
}

signed main() {
    cin >> n >> p >> k;
    int r1 = n * p % P * qmi(p+1, n-1) % P * qmi(k) % P;
    int r2 = 0;
    int wk = qmi(3, (P-1) / k), wkn = qmi(wk);
    for (int j = 0, wj = 1, wjn = 1; j < k; j++, wj = wj * wk % P, wjn = wjn * wkn % P) {
        r2 = (r2 + qmi((p * wj + 1) % P, n) * S(k, wjn)) % P;
    }
    int ans = ((r1 - r2 * qmi(k * k % P)) % P + P) % P;
    cout << ans << endl;
    return 0;
}
```

## [BZOJ3328] PYXFIB

> 给定 $n,k,p$ 其中 $n\le 10^{18},k\le 2\times 10^4,p\le 10^9$ 且 $p$ 为质数，保证 $p\bmod k=1$，求：
> $$
> \sum_{i=0}^{\lfloor\frac{n}{k}\rfloor}\binom{n}{ik}f_{ik}
> $$
> 其中 $f_0=f_1=1$，对于 $n\ge 2$ 有 $f_n=f_{n-1}+f_{n-2}$。
>
> 题目链接：[P10664](https://www.luogu.com.cn/problem/P10664)。

首先转化一下，写成单位根反演可以做的式子：
$$
\begin{aligned}
\sum_{i=0}^n[k\mid i]\binom{n}{i}f_i&=\sum_{i=0}^n\left(\frac{1}{k}\sum_{j=0}^{k-1}\omega_{k}^{ij}\right)\binom{n}{i}f_i\\
&=\frac{1}{k}\sum_{i=0}^n\sum_{j=0}^{k-1}\binom{n}{i}f_i\omega_{k}^{ij}
\end{aligned}
$$
对于斐波那契和组合数乘一块的情况，在 [JLCPC2024 C](https://codeforces.com/gym/105170/problem/C) 中也有出现，这里是我写的[题解](https://zhuanlan.zhihu.com/p/698751799)。总之就是由于斐波那契的转移矩阵 $A$ 和单位阵 $E$ 具有交换律，因此可以对矩阵用二项式定理。接着推：
$$
\begin{aligned}
\frac{1}{k}\sum_{j=0}^{k-1}\sum_{i=0}^n\binom{n}{i}A^iF_0\omega_{k}^{ij}&=\frac{1}{k}\sum_{j=0}^{k-1}\sum_{i=0}^n\binom{n}{i}(A\omega_k^j)^iE^{n-i}F_0\\
&=\frac{1}{k}\sum_{j=0}^{k-1}(A\omega_k^j+E)^nF_0
\end{aligned}
$$
这样就可以做到 $O(k\log n)$ 解决了。对于单位根，由于题目保证 $p\bmod k=1$，因此 $\omega_k^j\equiv g^{\frac{j}{k}(p-1)}\pmod p$，需要找出来一个原根。这里用原根是为了保证当 $k\not\equiv 0\pmod{\varphi(p)}$ 时有 $g^k\not\equiv 1\pmod p$，否则单位根反演的式子就不成立了。

由于最小原根大概是 $O(p^{\frac{1}{4}})$ 的，原根的充要条件是 $\delta_p(g)=\varphi(p)$。对于任意 $g$，由于 $\delta_p(g)$ 保证 $a^0,a^1,\dots,a^{\delta_p(g)-1}$ 互不相同，所以一定有 $\delta_p(g)\mid \varphi(p)$，直接用 $\varphi(p)$ 的所有因子 $d$ 验证是否有 $g^d\equiv 1\pmod p$ 即可。如果存在 $d\mid \varphi(p)$ 使得 $g^d\equiv 1\pmod p$，那么 $d$ 的倍数一定也满足条件，所以直接枚举 $\varphi(p)/p_i$ 即可，其中 $p_i$ 为 $\varphi(p)$ 的质因子。 

所以质因数分解 $\varphi(p)$，这里的复杂度大概是 $O(\sqrt p)$。然后枚举原根，验证的复杂度不计。找到原根后，复杂度为 $O(k\log n)$。单次复杂度为 $O(\sqrt p+k\log n)$，可以通过。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define eb emplace_back
#define int long long
int T, n, k, p, g;
vector<int> fac;

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % p;
        a = a * a % p;
        k >>= 1;
    }
    return res;
}

struct Matrix {
    int dat[2][2];
    Matrix() { memset(dat, 0, sizeof dat); }

    Matrix operator*(const Matrix& mat) const {
        Matrix res;
        for (int i = 0; i < 2; i++)
            for (int k = 0; k < 2; k++)
                for (int j = 0; j < 2; j++)
                    res.dat[i][j] = (res.dat[i][j] + dat[i][k] * mat.dat[k][j]) % p;
        return res;
    }

    Matrix qmi(int k) {
        Matrix res, a = *this;
        res.dat[0][0] = res.dat[1][1] = 1;
        while (k) {
            if (k & 1) res = res * a;
            a = a * a;
            k >>= 1;
        }
        return res;
    }
};

signed main() {
    cin >> T;
    while (T--) {
        fac.clear();
        cin >> n >> k >> p;
        int phi = p-1;
        for (int i = 2; i*i <= phi; i++) {
            if (phi % i) continue;
            while (phi % i == 0) phi /= i;
            fac.eb(i);
        }
        if (phi != 1) fac.eb(phi);

        for (g = 2; g < p; g++) {
            for (int f: fac) if (qmi(g, (p-1) / f) == 1) goto nxt;
            break;
            nxt:;
        }

        int res = 0, w1 = qmi(g, (p-1) / k);
        for (int w = 1, j = 0; j < k; j++, w = w * w1 % p) {
            Matrix f;
            f.dat[0][0] = f.dat[0][1] = f.dat[1][0] = w;
            f.dat[0][0]++, f.dat[1][1]++;
            // f = A*w+E
            f = f.qmi(n);
            res = (res + f.dat[0][0]) % p;
        }
        cout << res * qmi(k, p-2) % p << endl;
    }
    return 0;
}
```



