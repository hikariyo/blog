---
date: 2023-09-20 15:17:00
update: 2024-05-28 16:07:00
title: 数学 - 快速傅立叶变换
katex: true
description: 用点表示法可以 O(n) 的复杂度内求出多项式之积，FFT 可以在 O(nlog(n)) 的复杂度内在多项式的点表示法和系数表示法之间转换。
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

如果我们要算出来多项式之积，用系数表示法只能 $O(n^2)$ 算，但是点表示法允许我们在 $O(n+m)$ 的时间复杂度算出来目标多项式。

本文要说的 FFT 就是在 $O(n\log n)$ 的复杂度内将多项式在点表示法和系数表示法之间进行转换。

## 单位根

单位根是 $x^n=1$ 在复数域上的所有解，记做：
$$
\omega_n^k = \exp(\frac{2k\pi i}{n})=\cos \frac{2k\pi}{n}+i\sin \frac{2k\pi}{n}
$$
根据指数函数和三角函数的性质，我们可以轻松推出下面的性质，下面 $p,q,k,m,n$ 均为实数，其中 $m,n\ne 0$，本文就不讨论复数时是否成立了（应当也是成立的）。
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

## FFT

FFT 的思想就是将一个多项式不断划分成偶函数与奇函数之和的形式，然后分治地做下去，我们这里将 $n$ 化为大于它的最小 $2$ 的整次幂，多的位置就让 $a_i=0$ 即可，这样我们就不需要讨论 $n$ 的奇偶性了。
$$
\begin{aligned}
A(x)&=a_0+a_1x^1+a_2x^2+a_3x^3+a_4x^4+\cdots+a_{n-1}x^{n-1}\\
A_1(x)&=a_0+a_2x^1+\cdots+a_nx^{n/2}\\
A_2(x)&=a_1+a_3x^1+\cdots+a_{n-1}x^{(n-2)/2}\\
A(x)&=A_1(x^2)+xA_2(x^2)
\end{aligned}
$$
我们代入 $\omega_n^k(0\le k<n/2)$，然后用上面说到的性质可以得到：
$$
\begin{aligned}
A(\omega_n^k)&=A_1(\omega_{n/2}^k)+\omega_n^kA_2(\omega_{n/2}^k)\\
A(\omega_n^{k+n/2})&=A_1(\omega_{n/2}^k)-\omega_n^kA_2(\omega_{n/2}^k)
\end{aligned}
$$
这样我们得到了 $n$ 个点，算法保证原本的多项式的次数是严格小于 $n$ 的，所以这个结果多项式也能被唯一确定出来。

还有一个问题，我们能快速转化为点表示法，但是如何从点表示法转换回系数表示法？假设我们知道 $(\omega_n^k,A(\omega_n^k))$，并且 $A(x)$ 的次数是严格小于 $n$ 的，将它看作 $n-1$ 次的多项式，那么就有：
$$
\begin{aligned}
a_k&=\frac{1}{n}\sum_{i=0}^{n-1} A(\omega_n^i)(\omega_n^{-k})^i\\
&=\frac{1}{n}\sum_{i=0}^{n-1}\sum_{j=0}^{n-1}[a_j(\omega_n^i)^j(\omega_n^{-k})^i]\\
&=\frac{1}{n}\sum_{j=0}^{n-1}a_j[\sum_{i=0}^{n-1}(\omega_n^j\omega_n^{-k})^i]\\
&=\frac{1}{n}\sum_{j=0}^{n-i}a_j[\sum_{i=0}^{n-1}(\omega_n^{j-k})^i]\\
\end{aligned}
$$
可以看出后面是一个等比数列求和，我们应当分情况讨论。
$$
\sum_{i=0}^{n-1}(\omega_n^{p})^i=\begin{cases}
n &p=0\\
\frac{1-\omega_n^{np}}{1-\omega_n^p} &p\ne 0
\end{cases}
$$
而 $\omega_n^{np}=\omega_1^p=1$，所以下面的是 $0$，于是：
$$
\text{RHS}=\frac{1}{n} (a_k n)=a_k=\text{LHS}
$$
这样就相当于把 $A(\omega_n^i)$ 当作系数，$\omega_n^{-k}$ 当作自变量再做一遍 FFT，所以我们 FFT 是就地修改的。

接下来看时间复杂度：
$$
T(n)=2T(\frac{n}{2})+O(n)
$$
和归并排序一样，是 $O(n\log n)$ 的复杂度。

## 模板

> 给定 $n$ 次多项式 $A(x)$ 和 $m$ 次多项式 $B(x)$ 的所有系数，求 $A(x)B(x)$ 的所有系数。
>
> 题目链接：[AcWing 3122](https://www.acwing.com/problem/content/3125/)，[P3803](https://www.luogu.com.cn/problem/P3803)。

FFT 有两种写法，可以用递归，也可以用迭代，其中迭代的常数要小很多，但是递归好写，这里都给出。此外，STL 中是有复数类的，但自己写一个常数会小一些。

结果多项式是 $n+m$ 次的，所以我们需要取 $lim=2^{\lceil\log_2(n+m+1)\rceil}>n+m$ 符合我们上面的推导。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 2100000;
const double PI = acos(-1);

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
} a[N], b[N];
int n, m;

void FFT(Complex A[], int lim, int inv) {
    if (lim == 1) return;

    Complex A1[lim>>1], A2[lim>>1];
    
    for (int i = 0; i < lim; i++) {
        if (i & 1) A2[i>>1] = A[i];
        else A1[i>>1] = A[i];
    }
    
    FFT(A1, lim >> 1, inv), FFT(A2, lim >> 1, inv);
    
    Complex w1 = {cos(2*PI/lim), inv * sin(2*PI/lim)}, w = {1, 0};
    for (int k = 0; k < (lim >> 1); k++, w = w * w1) {
        Complex x = A1[k], y = w * A2[k];
        A[k] = x+y, A[k+(lim>>1)] = x-y;
    }
}

int main() {
    scanf("%d%d", &n, &m);
    int lim = 1 << (int)ceil(log2(n+m+1));
    for (int i = 0; i <= n; i++) scanf("%lf", &a[i].x);
    for (int i = 0; i <= m; i++) scanf("%lf", &b[i].x);
    
    FFT(a, lim, 1), FFT(b, lim, 1);
    for (int i = 0; i < lim; i++) a[i] = a[i] * b[i];
    FFT(a, lim, -1);
    
    for (int i = 0; i <= n+m; i++) printf("%d ", (int)round(a[i].x / lim));
    return 0;
}
```

如果用迭代写，注意到最底层的地方，都满足如果是奇数就左边，偶数就右边，所以二进制最低位是 0 交换后最高位就为 1，反复这样操作下去就会发现每个位置和它要交换的位置满足二进制表示x相反：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 2100000;
const double PI = acos(-1);

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
} a[N], b[N];
int n, m;
int rev[N];

void FFT(Complex A[], int lim, int inv) {
    for (int i = 0; i < lim; i++)
        // 如果不加判断就会交换两次 相当于没换
        if (i < rev[i]) swap(A[i], A[rev[i]]);
    
    for (int mid = 1; mid < lim; mid <<= 1) {
        // len = 2mid, 所以 2*PI / len = PI / mid
        Complex w1 = {cos(PI/mid), inv * sin(PI/mid)};
        
        int R = mid << 1;
        // 每个区间都分割成 [i, i+mid), [i+mid, i+R) 两个子区间
        for (int i = 0; i < lim; i += R) {
            Complex w = {1, 0};
            // 枚举区间中的数, i+j 是左边的, i+j+mid 是右边的
            for (int j = 0; j < mid; j++, w = w*w1) {
                // A1[k]: A[i+j], A2[k]: A[i+j+mid]
                // A[k]: A[i+j]
                Complex x = A[i+j], y = w * A[i+j+mid];
                A[i+j] = x+y, A[i+j+mid] = x-y;
            }
        }
    }
}

int main() {
    scanf("%d%d", &n, &m);
    int bits = ceil(log2(n+m+1)), lim = 1 << bits;
    for (int i = 0; i < lim; i++) rev[i] = (rev[i>>1] >> 1) | ((i & 1) << (bits-1));
    
    for (int i = 0; i <= n; i++) scanf("%lf", &a[i].x);
    for (int i = 0; i <= m; i++) scanf("%lf", &b[i].x);
    
    FFT(a, lim, 1), FFT(b, lim, 1);
    for (int i = 0; i < lim; i++) a[i] = a[i] * b[i];
    FFT(a, lim, -1);
    
    for (int i = 0; i <= n+m; i++) printf("%d ", (int)round(a[i].x / lim));
    return 0;
}
```

## A*B Problem

> 给定两个正整数 $a,b$ 求 $a\times b$，其中 $1\le a,b\le 10^{1000000}$
>
> 题目链接：[P1919](https://www.luogu.com.cn/problem/P1919)，[AcWing 3123](https://www.acwing.com/problem/content/3126/)。

我们把大数字看作多项式，第 $i$ 位置上的数字看作 $a_i$，然后求出来 $A(10)B(10)$ 就行了。

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 2100010, M = 1000010;
const double PI = acos(-1);

struct Complex {
    double x, y;
    
    Complex operator+(const Complex& t) const {
        return {x + t.x, y + t.y};
    }
    
    Complex operator-(const Complex& t) const {
        return {x - t.x, y - t.y};
    }
    
    Complex operator*(const Complex& t) const {
        return {x*t.x - y*t.y, x*t.y + y*t.x};
    }
} a[N], b[N];

int n, m, rev[N];
char p[M], q[M], res[N];

void FFT(Complex A[], int lim, int inv) {
    for (int i = 0; i < lim; i++) if (i < rev[i]) swap(A[i], A[rev[i]]);
    
    for (int mid = 1; mid < lim; mid <<= 1) {
        int len = mid << 1;
        Complex w1 = {cos(PI / mid), inv * sin(PI / mid)};
        for (int i = 0; i < lim; i += len) {
            Complex w = {1, 0};
            for (int j = 0; j < mid; j++, w = w * w1) {
                Complex x = A[i+j], y = w * A[i+j+mid];
                A[i+j] = x+y, A[i+j+mid] = x-y;
            }
        }
    }
}

int main() {
    scanf("%s%s", p, q);
    n = strlen(p) - 1, m = strlen(q) - 1;
    for (int i = 0; i <= n; i++) a[i].x = p[n-i] - '0';
    for (int i = 0; i <= m; i++) b[i].x = q[m-i] - '0';
    
    int bits = ceil(log2(n+m+1)), lim = 1 << bits;
    for (int i = 0; i < lim; i++) rev[i] = (rev[i>>1]>>1) | ((i & 1) << (bits - 1));
    
    FFT(a, lim, 1), FFT(b, lim, 1);
    for (int i = 0; i < lim; i++) a[i] = a[i] * b[i];
    FFT(a, lim, -1);
    
    int k = 0, t = 0;
    for (int i = 0; i <= n+m || t; i++) {
        if (i <= n+m) t += round(a[i].x / lim);
        res[k++] = t % 10 + '0';
        t /= 10;
    }
    reverse(res, res+k);
    printf("%s\n", res);
    return 0;
}
```
