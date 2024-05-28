---
date: 2023-09-21 01:33:00
title: 数学 - 快速数论变换
description: 由于 FFT 需要用到大量的浮点运算，既耗时间也会造成精度问题，所以就有了 NTT，利用原根与单位根相似的性质来做到加速求多项式积的目的。
katex: true
categories: OI
tags:
- Algo
- C++
- Math
---


## 阶与原根

若 $g,p$ 互质，使得 $g^n\equiv 1(\text{mod }p)$ 的最小正整数 $n$ 称为 $g$ 模 $p$ 的阶，记作 $\delta_p(g)$，若 $\delta_p(g)=\varphi(p)$，那么 $g$ 称作模 $p$ 的一个原根。

简而言之，如果 $g^{\varphi(p)}\equiv 1(\text{mod }p)$ 且不存在比 $\varphi(p)$ 更小的数字使其成立，那么 $g$ 就是模 $p$ 的原根。

原根有一个十分关键的性质，那就是 $g^0,g^1,g^2,\cdots,g^{\varphi(p)-1}$ 两两不同余，这个反证法证明起来很简单，为了书写方便下面省略 $(\text{mod }p)$：
$$
\exist 0\le i<j<\varphi(p),g^i\equiv g^j\\
\Rightarrow g^{j-i}\equiv 1,j-i\lt \varphi(p)
$$
这与原根的定义矛盾，所以它们就是两两不同余的。

考虑到我们 FFT 的时候进行了多次分治操作，所以我们需要找到一个素数 $p$ 使得 $\varphi(p)=p-1$ 能够被多次除 2，即能够写成 $p=q\cdot 2^k+1$ 的形式，下面是两个常用的素数。

| 原根 | 素数         | 分解                 |
| ---- | ------------ | -------------------- |
| $3$  | $998244353$  | $119\times 2^{23}+1$ |
| $3$  | $2281701377$ | $17\times 2^{27}+1$  |

然而我只稍微记了一下 $998244353$，因为这个人的出现频率比较高。

类比单位根，我们定义当 $n=2^k,k\in \N$ 时，有：
$$
g_n^k=g^{\frac{p-1}{n}k}
$$
对比单位根的定义：
$$
\omega_n^k=\exp(\frac{2\pi i}{n}k)
$$
形式上非常类似，然后性质也非常类似：
$$
\begin{aligned}
g_n^ig_n^j&\equiv g_n^{i+j}\\
(g_n^i)^j&\equiv g_n^{ij}\\
g_n^{k+n}&\equiv g_n^k\\
g_{mn}^{mk}&\equiv g_n^k\\
g_n^{k+\frac{n}{2}}&\equiv -g_n^k
\end{aligned}
$$
对最后一条，这里解释一下：由费马小定理可以知道 $g^{p-1}\equiv 1$，那么 $(g^{\frac{p-1}{2}})^2\equiv 1$，而我们上面提到过两两不同余，所以 $g^{\frac{p-1}{2}}\not\equiv g^0$，因此只能有 $g^{\frac{p-1}{2}}\equiv -1$，所以最后一个式子展开一下就能得到。

## NTT

有了这些性质，我们再来看 NTT 是如何操作的。与 FFT 一样，我们把多项式不断分割成一个奇函数，一个偶函数，这里就不详细写了：
$$
A(x)=a_0+a_1x^1+a_2x^2+\cdots+a_nx^{n}=A_1(x^2)+xA_2(x^2)
$$
有了这个，我们可以代入原根看一下了：
$$
\begin{aligned}
A(g_n^k)=A_1(g_{n/2}^k)+g_n^kA_2(g_{n/2}^k)\\
A(g_n^{k+\frac{n}{2}})=A_1(g_{n/2}^k)-g_n^kA_2(g_{n/2}^k)
\end{aligned}
$$
简直和 FFT 的单位根一模一样。

对于逆变换，我们同样再来看一下，其中 $n^{-1}$ 是 $n$ 模 $p$ 的逆元：
$$
\begin{aligned}
a_k&\equiv n^{-1}\sum_{i=0}^{n-1} A(g_n^i)(g_n^{-k})^i\\
&\equiv n^{-1}\sum_{i=0}^{n-1}[\sum_{j=0}^{n-1} a_j(g_n^i)^j(g_n^{-k})^i]\\
&\equiv n^{-1}\sum_{j=0}^{n-1}a_j[\sum_{i=0}^{n-1}(g_n^{j-k})^i]\\
\end{aligned}
$$
看一下原根的等比数列求和有什么特别的地方。
$$
\sum_{i=0}^{n-1}(g_n^k)^i\equiv\begin{cases}
\frac{1-g_{n}^{nk}}{1-g_n^k}=0&g_{n}^k \not\equiv 1\\
n&g_n^k\equiv 1
\end{cases}
$$
还是同样的套路，带回原式：
$$
a_k\equiv n^{-1} a_kn\equiv a_k
$$
这样逆变换也搞定了。相较于 FFT，NTT 的优势在于没有精度问题，而且不用进行大量的浮点数运算和三角函数运算，速度自然也上来了。但也不是绝对的，因为涉及大数取模运算势必较慢。

## 模板

> 给定 $n$ 次多项式 $A(x)$ 与 $m$ 次多项式 $B(x)$ 的所有系数，求 $A(x)B(x)$ 的所有系数。
>
> 题目链接：[P3803](https://www.luogu.com.cn/problem/P3803)。

我们选模数 $P=998244353,g=3,g^{-1}=332748118$，这些都是可以现场算的。

本题目的系数都是个位数，所以肯定在 $P$ 的范围以内，取模之后还是原来的数字，这个不必担心。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 2100010, P = 998244353, G = 3, GI = 332748118;
int n, m, rev[N];
LL a[N], b[N];

LL qmi(LL a, LL k) {
    LL res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

void NTT(LL A[], int lim, int base) {
    for (int i = 0; i < lim; i++) if (i < rev[i]) swap(A[i], A[rev[i]]);
    
    for (int mid = 1; mid < lim; mid <<= 1) {
        int len = mid << 1;
        LL w1 = qmi(base, (P-1) / len);
        for (int i = 0; i < lim; i += len) {
            LL w = 1;
            for (int j = 0; j < mid; j++, w = w * w1 % P) {
                LL x = A[i+j], y = w * A[i+j+mid] % P;
                A[i+j] = (x+y) % P, A[i+j+mid] = (x-y+P) % P;
            }
        }
    }
}

int main() {
    scanf("%d%d", &n, &m);
    int bits = ceil(log2(n+m+1)), lim = 1 << bits;
    for (int i = 0; i < lim; i++)
        rev[i] = (rev[i >> 1] >> 1) | ((i & 1) << (bits - 1));
    
    for (int i = 0; i <= n; i++) scanf("%lld", &a[i]);
    for (int i = 0; i <= m; i++) scanf("%lld", &b[i]);
    
    NTT(a, lim, G), NTT(b, lim, G);
    for (int i = 0; i < lim; i++) a[i] = a[i] * b[i] % P;
    NTT(a, lim, GI);
    
    LL inv = qmi(lim, P-2);
    for (int i = 0; i <= n+m; i++) printf("%lld ", a[i] * inv % P);
    return puts(""), 0;
}
```

## A*B Problem

> 给定两个正整数 $a,b$ 求 $a\times b$，其中 $1\le a,b\le 10^{1000000}$
>
> 题目链接：[P1919](https://www.luogu.com.cn/problem/P1919)，[AcWing 3123](https://www.acwing.com/problem/content/3126/)。

套模板即可，注意大数字在字符串中的低位反而是在多项式中的高位，然后对结果多项式进行一个类似高精度的处理即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 2100010, M = 1000010, P = 998244353, G = 3, GI = 332748118;
int n, m, rev[N];
LL a[N], b[N];
char p[M], q[M], res[N];

LL qmi(LL a, LL k) {
    LL res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

void NTT(LL A[], int lim, int base) {
    for (int i = 0; i < lim; i++) if (i < rev[i]) swap(A[i], A[rev[i]]);
    for (int mid = 1; mid < lim; mid <<= 1) {
        int len = mid << 1;
        LL w1 = qmi(base, (P-1) / len);
        for (int i = 0; i < lim; i += len) {
            LL w = 1;
            for (int j = 0; j < mid; j++, w = w * w1 % P) {
                LL x = A[i+j], y = w * A[i+j+mid] % P;
                A[i+j] = (x+y) % P, A[i+j+mid] = (x-y+P) % P;
            }
        }
    }
} 

int main() {
    scanf("%s%s", p, q);
    n = strlen(p) - 1, m = strlen(q) - 1;
    int bits = ceil(log2(n+m+1)), lim = 1 << bits;
    for (int i = 0; i < lim; i++) rev[i] = (rev[i>>1]>>1) | ((i&1) << (bits-1));
    for (int i = 0; i <= n; i++) a[i] = p[n-i] - '0';
    for (int i = 0; i <= m; i++) b[i] = q[m-i] - '0';
    
    NTT(a, lim, G), NTT(b, lim, G);
    for (int i = 0; i < lim; i++) a[i] = a[i] * b[i] % P;
    NTT(a, lim, GI);
    
    LL inv = qmi(lim, P-2), t = 0;
    int k = 0;
    for (; k <= n+m || t; k++) {
        if (k <= n+m) t += a[k] * inv % P;
        res[k] = t % 10 + '0';
        t /= 10;
    }
    reverse(res, res+k);
    printf("%s\n", res);
    return 0;
}
```

