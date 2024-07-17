---
date: 2024-07-16 17:48:00
title: 多项式 - 初等函数
katex: true
description: 基于 FFT 可以 O(nlog(n)) 的计算多项式卷积，可以推出一系列多项式的初等函数，本文记录一些不全面初等函数及模板，包括多项式乘法、乘法逆、除法、取模。
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 简介

基于 FFT 可以做到 $O(n\log n)$ 地计算两个多项式的卷积，由此我们可以对多项式进行很多操作。本文简单介绍一下最基础的**不全面**的初等函数部分，后面我在做题时用到会对不全面的部分进行补充。

## 多项式乘法

### 点表示法

对于一个 $n-1$ 次多项式，可以用 $n$ 个不同点唯一表示出来，如：
$$
A(x)=\sum_{i=0}^{n-1}a_ix^i
$$
如果我们有 $n$ 个不同点，就相当于对于未知数 $a_0,\cdots,a_{n-1}$ 列出了 $n$ 个方程，可以用线性代数的知识（范德蒙德行列式）证明它一定是有解的。如果能快速将系数表示法转化为点表示法，那么就可以在 $O(n)$ 将两个多项式相乘。

### 单位根

单位根是 $x^n=1$ 在复数域上的所有解，记做：
$$
\omega_n^k = \exp(2\pi i\frac{k}{n})=\cos \frac{2k\pi}{n}+i\sin \frac{2k\pi}{n}
$$
根据指数函数和三角函数的性质，我们可以轻松推出下面的性质，下面 $p,q,k,m,n$ 均为整数，其中 $m,n\ne 0$，这里就不讨论其它情况了，因为后面用到的都是整数：
$$
\begin{aligned}
\omega_n^p\omega_n^q&=\omega_n^{p+q}\\
(\omega_n^p)^q&=\omega_n^{pq}\\
\omega_n^{k+\frac{n}{2}}&=-\omega_n^k\\
\omega_n^{k+n}&=\omega_n^k\\
\omega_{mn}^{mk}&=\omega_n^k
\end{aligned}
$$
这些式子直接带进指数形式就能推导出来。

### 阶与原根

若 $g,p$ 互质，使得 $g^n\equiv 1(\text{mod }p)$ 的最小正整数 $n$ 称为 $g$ 模 $p$ 的阶，记作 $\delta_p(g)$，若 $\delta_p(g)=\varphi(p)$，那么 $g$ 称作模 $p$ 的一个原根。

简而言之，如果 $\delta_p(g)=\varphi(p)$，那么 $g$ 就是模 $p$ 的原根。原根满足 $g^0,g^1,g^2,\cdots,g^{\varphi(p)-1}$ 两两不同余，如果存在的话那么 $\delta_p(g)\ne \varphi(p)$ 与其定义矛盾。考虑到我们 FFT 的时候进行了多次分治操作，所以我们需要找到一个素数 $p$ 使得 $\varphi(p)=p-1$ 能够被多次除 2，即能够写成 $p=q\cdot 2^k+1$ 的形式，类比单位根来定义：

$$
g_n^k=g^{\frac{p-1}{n}k}
$$
性质也非常类似：
$$
\begin{aligned}
g_n^ig_n^j&\equiv g_n^{i+j}\\
(g_n^i)^j&\equiv g_n^{ij}\\
g_n^{k+n}&\equiv g_n^k\\
g_n^{k+\frac{n}{2}}&\equiv -g_n^k\\
g_{mn}^{mk}&\equiv g_n^k\\
\end{aligned}
$$
对于倒数第二条：由费马小定理可以知道 $g^{p-1}\equiv 1$，那么 $(g^{\frac{p-1}{2}})^2\equiv 1$，而 $g^0\sim g^{\varphi(p)-1}$ 两两不同余，所以 $g^{\frac{p-1}{2}}\not\equiv g^0$，因此只能有 $g^{\frac{p-1}{2}}\equiv -1$。

## 

### FFT

FFT 的思想就是将一个多项式不断划分成偶函数与奇函数之和的形式，我们这里将 $n$ 化为大于它的最小 $2$ 的整次幂，多的位置就让 $a_i=0$ 即可，这样我们就不需要讨论 $n$ 的奇偶性了。
$$
\begin{aligned}
A(x)&=\sum_{i=0}^{n-1}a_ix^i\\
A_1(x)&=\sum_{i=0}^{\frac{n-2}{2}}a_{2i}x^i\\
A_2(x)&=\sum_{i=0}^{\frac{n-2}{2}}a_{2i+1}x^i\\
A(x)&=A_1(x^2)+xA_2(x^2)
\end{aligned}
$$
我们代入 $\omega_n^k(0\le k\lt \frac{n}{2})$，然后用上面说到的性质可以得到：
$$
\begin{aligned}
A(\omega_n^k)&=A_1(\omega_{\frac{n}{2}}^k)+\omega_n^kA_2(\omega_{\frac{n}{2}}^k)\\
A(\omega_n^{k+\frac{n}{2}})&=A_1(\omega_{\frac{n}{2}}^k)-\omega_n^kA_2(\omega_{\frac{n}{2}}^k)
\end{aligned}
$$
这样我们得到了 $n$ 个点，算法保证原本的多项式的次数是严格小于 $n$ 的，所以这个结果多项式也能被唯一确定出来。

还有一个问题，上面是系数表示法转换为点表示法，还需要将点表示法转化为系数表示法，这里原理是单位根反演，其实不知道也没关系，现在已经知道 $(\omega_n^k,A(\omega_n^k))$，并且 $A(x)$ 的次数是严格小于 $n$ 的，将它看作 $n-1$ 次的多项式，那么就有：
$$
\begin{aligned}
a_k&=\frac{1}{n}\sum_{i=0}^{n-1} A(\omega_n^i)(\omega_n^{-k})^i\\
&=\frac{1}{n}\sum_{i=0}^{n-1}\sum_{j=0}^{n-1}a_j(\omega_n^i)^j(\omega_n^{-k})^i\\
&=\frac{1}{n}\sum_{j=0}^{n-1}a_j\sum_{i=0}^{n-1}(\omega_n^j\omega_n^{-k})^i\\
&=\frac{1}{n}\sum_{j=0}^{n-i}a_j\sum_{i=0}^{n-1}(\omega_n^{j-k})^i\\
\end{aligned}
$$
可以看出后面是一个等比数列求和，我们应当分情况讨论。
$$
\sum_{i=0}^{n-1}(\omega_n^{p})^i=\begin{cases}
n &p=0\\
\frac{1-\omega_n^{np}}{1-\omega_n^p} &p\ne 0
\end{cases}
$$
所以只有当 $p=0$ 即 $j=k$ 时原式等于 $n$，这样就相当于把 $A(\omega_n^i)$ 当作系数，$\omega_n^{-k}$ 当作自变量再做一遍 FFT，所以一般实现 FFT 是就地修改的。接下来看时间复杂度：
$$
T(n)=2T(\frac{n}{2})+O(n)
$$
和归并排序一样，是 $O(n\log n)$​ 的复杂度。

### 蝶形变换

根据现在的内容已经可以写出理论上 $O(n\log n)$ 的代码了，但是常数会大，假设已经知道了 $A_1(\omega_\frac{n}{2}^k),A_2(\omega_\frac{n}{2}^k)$，下面要求出 $A(\omega_n^k)$。我们希望 $A_1(\omega_{\frac{n}{2}}^k)$ 储存在 $k$ 的位置，$A_2(\omega_{\frac{n}{2}}^k)$ 储存在 $k+\frac{n}{2}$ 的位置，然后将 $k$ 位置覆盖为 $A(\omega_n^k)$。

也就是让 $A_1$ 都在左 $1\sim \frac{n}{2}-1$ 的位置，$A_2$ 在右边 $\frac{n}{2}\sim n$ 的位置。

所以每一次划分，都让下标为偶数的在左边，为奇数的在右边，并且将左右两边重新编号 $1\sim \frac{n}{2}$，相当于把原始编号右移一位，然后重复进行这个过程。

因此，最后 $n=1$ 时原始为 $i$ 位置的数字会恰好在与其二进制表示相反的地方。并且 $n=1$ 时利用的单位根是 $\omega_1^0=1$，也就是求出来的值就是系数本身。

可以构造 $r_i$ 数组表示与 $i$ 的二进制相反的数字，设 $l$ 为二进制位数，即 $\log_2n$，可以得到一个递推关系：
$$
r_i=(r_{i\gg1}\gg 1)\oplus(i_0 \ll l-1)
$$
其中 $i_0$ 表示 $i$ 的最低位，可以用 `& 1` 取出来。异或可以用或，反正最高位一定是 $0$​。

对于 NTT 的模板见文末的我封装好的一个多项式模板。

### 任意模数 NTT

> 给定两个多项式 $F(x),G(x)$ 求出 $F(x)G(x)$ 系数对 $p$ 取模，不保证 $p=a\cdot 2^k+1$​。
>
> 数据范围：$1\le n,m\le 10^5$，$0\le a_i,b_i\le 10^9$，$2\le p\le 10^9+9$。
>
> 题目链接：[P4245](https://www.luogu.com.cn/problem/P4245)。

本质上是利用 CRT 求出结果的精确值后再对给定的模数取模。先找到三个可以 NTT 的大质数 $P_1,P_2,P_3$ 然后分别做 NTT，因为系数最大是 $10^9\times 10^9\times 10^5=10^{23}<2^{127}$，可以用 `i128` 表达出最终结果，然后对 $p$ 取模。

对一个数字 $x$，如果有：
$$
\begin{cases}
x\equiv x_1\pmod{P_1}\\
x\equiv x_2\pmod{P_2}\\
x\equiv x_3\pmod{P_3}
\end{cases}
$$
对于前两个式子，有：
$$
\begin{aligned}
x_1+k_1P_1&=x_2+k'P_2\\
x_1+k_1P_1&\equiv x_2\pmod{P_2}\\
k_1&\equiv \frac{x_2-x_1}{P_1}\pmod{P_2}\\
x&\equiv x_1+k_1P_1\pmod{P_1P_2}
\end{aligned}
$$
再继续联立：
$$
\begin{aligned}
x_1+k_1P_1+k_2P_1P_2&=x_3+k''P_3\\
x_1+k_1P_1+k_2P_1P_2&\equiv x_3\pmod{P_3}\\
k_2&\equiv\frac{x_3-x_1-k_1P_1}{P_1P_2}\pmod{P_3}\\
x&\equiv x_1+k_1P_1+k_2P_1P_2\pmod{P_1P_2P_3}
\end{aligned}
$$
由于 $x<P_1P_2P_3$，所以最后一个同余号可以改成等号，代码如下：

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define ll __int128
const int N = 210000, P1 = 998244353, P2 = 469762049, P3 = 1004535809;
int n, m, p, lim, bits, rev[N];

template<int P>
int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

template<int P, int G>
struct Poly {
    int F[N];

    int& operator[](int i) {
        return F[i];
    }

    void NTT(int sgn) {
        for (int i = 0; i < lim; i++) if (i < rev[i]) swap(F[i], F[rev[i]]);

        for (int mid = 1; mid < lim; mid <<= 1) {
            int R = mid << 1, k = (P-1) / R * sgn;
            int w1 = qmi<P>(G, (k + P-1) % (P-1));
            for (int i = 0; i < lim; i += R) {
                // [i, i+mid) [i+mid, i+R)
                int w = 1;
                for (int j = 0; j < mid; j++, w = w * w1 % P) {
                    int x = F[i+j], y = w * F[i+j+mid] % P;
                    F[i+j] = (x+y) % P, F[i+j+mid] = (x-y+P) % P;
                }
            }
        }

        if (sgn == -1) {
            int ilim = qmi<P>(lim, P-2);
            for (int i = 0; i < lim; i++) F[i] = F[i] * ilim % P;
        }
    }

    // H changed after calling
    void operator*=(Poly<P, G>& H) {
        NTT(1), H.NTT(1);
        for (int i = 0; i < lim; i++) F[i] = F[i] * H[i] % P;
        NTT(-1);
    }
};

Poly<P1, 3> A1, B1;
Poly<P2, 3> A2, B2;
Poly<P3, 3> A3, B3;

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    cin >> n >> m >> p;
    lim = 1;
    while (lim <= n+m) lim <<= 1, bits++;
    for (int i = 0; i < lim; i++) rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (bits - 1));
    
    for (int i = 0; i <= n; i++) cin >> A1[i], A2[i] = A3[i] = A1[i];
    for (int i = 0; i <= m; i++) cin >> B1[i], B2[i] = B3[i] = B1[i];
    A1 *= B1, A2 *= B2, A3 *= B3;

    for (int i = 0; i <= n+m; i++) {
        int k1 = ((A2[i] - A1[i]) * qmi<P2>(P1, P2-2) % P2 + P2) % P2;
        ll x = A1[i] + k1 * P1;
        int k2 = ((A3[i] - x % P3) * qmi<P3>(P1 * P2 % P3, P3 - 2) % P3 + P3) % P3;
        x = x + (ll)k2 * P1 * P2;
        cout << (int)(x % p) << " \n"[i == n+m];
    }

    return 0;
}
```

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

## 多项式除法 & 取模

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

## 多项式对数函数

对于 $n$ 次多项式 $F(x)$，定义 $\ln F(x)$ 为 $\int\frac{F'(x)}{F(x)}\mathrm{d}x+C$，其中 $C$ 一般是 $0$，具体情况具体分析。

那么也就是需要实现一个多项式的求导和积分，我们知道，对于 $F'(x)$ 和 $F(x)$ 间的系数关系：
$$
[x^i]F'(x)=[x^{i+1}]F(x)\times (i+1)
$$
同样，知道 $F(x)$ 与 $\int F(x)\mathrm{d}x$ 间的系数关系：
$$
[x^i]\int F(x)\mathrm{d}x=[x^{i-1}]F(x)\times \frac{1}{i}
$$
所以就是套用一下这些即可。



```cpp
struct Poly {
    // n: deg(F(x))
    // lim: 2^(ceil(log2(n)))
    int n, lim;
    vector<int> f, rev;

    Poly() = default;

    Poly(const vector<int>& g) {
        assert(g.size());
        resize(g.size() - 1);
        for (int i = 0; i < g.size(); i++) f[i] = g[i];
    }

    Poly& resize(int n) {
        this->n = n;
        lim = 1;
        int lgn = 0;
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
        return F.ntt(-1);
    }

    Poly operator-(const Poly& g) const {
        int n = max(this->n, g.n);
        Poly R;
        R.resize(n);
        for (int i = 0; i <= n; i++) R[i] = (P + f[i] - g[i]) % P;
        return R;
    }

    // 1/F(x) (mod x^n)
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

    Poly diff() const {
        Poly df = *this;
        for (int i = 0; i < n; i++) df[i] = df[i+1] * (i+1) % P;
        return df;
    }

    Poly intergal(int C) const {
        Poly F = *this;
        F.resize(n+1);
        for (int i = n+1; i; i--) F[i] = F[i-1] * qmi(i, P-2) % P;
        F[0] = C;
        return F;
    }

    Poly ln(int C) const {
        return (diff() * inv()).intergal(C);
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
```

