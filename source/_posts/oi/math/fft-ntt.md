---
date: 2023-09-20 15:17:00
update: 2024-07-13 15:43:00
title: 数学 - FFT / NTT
katex: true
description: 基于 O(nlog(n)) 的系数表示法转化为点表示法的算法，结合单位根反演相关知识可以原地进行逆变换，因此可以在 O(nlog(n)) 的复杂度内计算出卷积，再结合 CRT 可以解决任意模数 NTT。
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 点表示法

对于一个 $n$ 次多项式，可以用 $n+1$ 个不同点唯一表示出来，如：
$$
A(x)=a_0+a_1x+a_2x^2+\cdots+a_nx^n
$$
如果我们有 $n+1$ 个不同点，就相当于对于未知数 $a_0,\cdots,a_n$ 列出了 $n+1$ 个方程，可以用线性代数的知识（范德蒙德行列式）证明它一定是有解的。

如果能快速将系数表示法转化为点表示法，那么就可以在 $O(n)$ 将两个多项式相乘。FFT 是一种可以将多项式从系数表示法转化为点表示法并且可逆的复杂度 $O(n\log n)$​ 的算法。 

## 单位根

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

## 阶与原根

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

## FFT

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

## 蝶形变换

根据现在的内容已经可以写出理论上 $O(n\log n)$ 的代码了，但是常数会大，假设已经知道了 $A_1(\omega_\frac{n}{2}^k),A_2(\omega_\frac{n}{2}^k)$，下面要求出 $A(\omega_n^k)$。我们希望 $A_1(\omega_{\frac{n}{2}}^k)$ 储存在 $k$ 的位置，$A_2(\omega_{\frac{n}{2}}^k)$ 储存在 $k+\frac{n}{2}$ 的位置，然后将 $k$ 位置覆盖为 $A(\omega_n^k)$。

也就是让 $A_1$ 都在左 $1\sim \frac{n}{2}-1$ 的位置，$A_2$ 在右边 $\frac{n}{2}\sim n$ 的位置。

所以每一次划分，都让下标为偶数的在左边，为奇数的在右边，并且将左右两边重新编号 $1\sim \frac{n}{2}$，相当于把原始编号右移一位，然后重复进行这个过程。

因此，最后 $n=1$ 时原始为 $i$ 位置的数字会恰好在与其二进制表示相反的地方。并且 $n=1$ 时利用的单位根是 $\omega_1^0=1$，也就是求出来的值就是系数本身。

可以构造 $r_i$ 数组表示与 $i$ 的二进制相反的数字，设 $l$ 为二进制位数，即 $\log_2n$，可以得到一个递推关系：
$$
r_i=(r_{i\gg1}\gg 1)\oplus(i_0 \ll l-1)
$$
其中 $i_0$ 表示 $i$ 的最低位，可以用 `& 1` 取出来。异或可以用或，反正最高位一定是 $0$。

## 模板

> 给定 $n$ 次多项式 $A(x)$ 和 $m$ 次多项式 $B(x)$ 的所有系数，求 $A(x)B(x)$ 的所有系数。
>
> 题目链接：[P3803](https://www.luogu.com.cn/problem/P3803)。

下面是一个 FFT 的版本：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 2100000;
const double PI = acos(-1);
int n, m, lim, bits, rev[N];

struct Complex {
    double x, y;
    Complex operator+(const Complex& t) const {
        return {x+t.x, y+t.y};
    }
    Complex operator-(const Complex& t) const {
        return {x-t.x, y-t.y};
    }
    Complex operator*(const Complex& t) const {
        return {x*t.x-y*t.y, x*t.y+y*t.x};
    }
};

void FFT(Complex F[], int sgn) {
    for (int i = 0; i < lim; i++) if (i < rev[i]) swap(F[i], F[rev[i]]);

    for (int mid = 1; mid < lim; mid <<= 1) {
        int R = mid << 1;
        // 2PI / len = PI / mid
        Complex w1 = {cos(PI / mid), sgn * sin(PI / mid)};
        for (int i = 0; i < lim; i += R) {
            // [i, i+mid) [i+mid, i+R)
            Complex w = {1, 0};
            for (int j = 0; j < mid; j++, w = w * w1) {
                Complex x = F[i+j], y = w * F[i+j+mid];
                F[i+j] = x+y, F[i+j+mid] = x-y;
            }
        }
    }

    if (sgn == -1) for (int i = 0; i < lim; i++) F[i] = F[i] * Complex{1. / lim, 0};
}

Complex A[N], B[N];

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    cin >> n >> m;

    lim = 1;
    while (lim <= n+m) lim <<= 1, bits++;
    for (int i = 0; i < lim; i++) rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (bits - 1));

    for (int i = 0; i <= n; i++) cin >> A[i].x;
    for (int i = 0; i <= m; i++) cin >> B[i].x;

    FFT(A, 1), FFT(B, 1);
    for (int i = 0; i < lim; i++) A[i] = A[i] * B[i];
    FFT(A, -1);

    for (int i = 0; i <= n+m; i++) cout << (int)round(A[i].x) << " \n"[i == n+m];
    return 0;
}
```

下面是 NTT 的版本：

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 2100000, P = 998244353;
int n, m, lim, bits, rev[N];

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

void NTT(int F[], int sgn) {
    for (int i = 0; i < lim; i++) if (i < rev[i]) swap(F[i], F[rev[i]]);

    for (int mid = 1; mid < lim; mid <<= 1) {
        int R = mid << 1, k = (P-1) / R * sgn;
        int w1 = qmi(3, (k + P-1) % (P-1));
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
        int ilim = qmi(lim, P-2);
        for (int i = 0; i < lim; i++) F[i] = F[i] * ilim % P;
    }
}

int A[N], B[N];

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    cin >> n >> m;

    lim = 1;
    while (lim <= n+m) lim <<= 1, bits++;
    for (int i = 0; i < lim; i++) rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (bits - 1));

    for (int i = 0; i <= n; i++) cin >> A[i];
    for (int i = 0; i <= m; i++) cin >> B[i];

    NTT(A, 1), NTT(B, 1);
    for (int i = 0; i < lim; i++) A[i] = A[i] * B[i] % P;
    NTT(A, -1);

    for (int i = 0; i <= n+m; i++) cout << A[i] << " \n"[i == n+m];
    return 0;
}
```

## 任意模数 NTT

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

    // p changed after calling
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

