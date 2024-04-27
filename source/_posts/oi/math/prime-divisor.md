---
date: 2023-10-26 20:22:00
update: 2024-01-14 22:16:00
title: 数学 - 质数约数相关例题
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
description: 利用各种筛法可以预处理出一定范围内的质数以及相关数论函数，筛法并不局限于只能处理 [1, N] 区间内的信息，也可以是 [L,R]，然后对题目进行求解。
---

## 线性筛

> 给定一个范围 $n$，有 $q$ 个询问，每次输出 $k$ 小的素数。
>
> 数据范围 $n=10^8,1\le q\le 10^6$，保证询问的素数不大于 $n$。
>
> 题目链接：[P3383](https://www.luogu.com.cn/problem/P3383)。

简单说一下线性筛的原理。

枚举当前已经筛出来的质数，设当前筛到的数字是 $i$，第 $j$ 个质数是 $p_j$，那么有：

1. 当 $i\bmod p_j \ne 0$，由于 $p_j$ 是从小到大枚举，所以 $p_j$ 一定是 $i\times p_j$ 的最小质因子。
2. 当 $i\bmod p_j=0$，同上，但是标记完 $i\times p_j$ 后就跳出循环，因为如果继续枚举就不是最小质因子了。

这样保证了每个数字只会被筛一次，也就保证了线性的复杂度。

枚举到 $i$ 时，$[1,i]$ 间的所有质数一定已经被筛出来了，因为如果一个数不是质数，它一定含有一个比它小的质因子，就会被前面的步骤筛掉。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1e8+10;
int n, q, k;
bitset<N> st;
vector<int> primes;

void init(int n) {
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes.emplace_back(i);
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i*primes[j]] = 1;
            if (i % primes[j] == 0) break;
        }
    }
}

int main() {
    scanf("%d%d", &n, &q);
    init(n);
    while (q--) scanf("%d", &k), printf("%d\n", primes[k-1]);
    return 0;
}
```

## 素数密度

> 给定区间 $[L,R]$（$1\leq L\leq R < 2^{31}$，$R-L\leq 10^6$），请计算区间中素数的个数。
>
> **输入格式**
>
> 第一行，两个正整数 $L$ 和 $R$。
>
> **输出格式**
>
> 一行，一个整数，表示区间中素数的个数。
>
> 题目链接：[P1835](https://www.luogu.com.cn/problem/P1835)。

如果一个数 $x$ 是合数，一定存在一个小于等于 $\sqrt x$ 的质因数，所以我们把 $\sqrt {2^{31}}<50000$ 的质数全部筛出来，然后就可以判断区间 $L,R$ 内的合数了。

由于 $1$ 不会被筛掉，而 $1$ 也不是质数，所以需要特判一下。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 50010, M = 1000010;
int l, r;
vector<int> primes;
bitset<N> st;
bitset<M> pst;

void init(int n) {
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes.emplace_back(i);
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i*primes[j]] = 1;
            if (i % primes[j] == 0) break;
        }
    }
}

int main() {
    scanf("%d%d", &l, &r);
    init(N-1);

    int tot = r-l+1;
    for (int p: primes) {
        for (long long i = max((l+p-1)/p * p, 2*p); i <= r; i += p) {
            if (pst[i-l]) continue;
            pst[i-l] = 1;
            tot--;
        }
    }

    if (l == 1) tot--;
    return !printf("%d\n", tot);
}
```

## [AHOI2005] 约数研究

> 求 $\sum_{i=1}^n d(i)$，其中 $d(i)$ 表示 $i$ 的约数个数，$n\le 10^6$。
>
> 题目链接：[P1403](https://www.luogu.com.cn/problem/P1403)。

推下式子。

$$
\begin{aligned}
\sum_{i=1}^n d(i) &= \sum_{i=1}^n\sum_{j=1}^n[j|i]\\
&=\sum_{j=1}^n\sum_{i=1}^n [j|i]\\
&=\sum_{j=1}^n\lfloor\frac{n}{j}\rfloor
\end{aligned}
$$
然后写个数论分块就能在 $O(\sqrt n)$ 的复杂度做出来了。

```cpp
#include <bits/stdc++.h>
using namespace std;

int n;

int main() {
    scanf("%d", &n);

    int res = 0;
    for (int i = 1; i <= n; i++) {
        int j = n / (n / i);
        res += (j-i+1) * (n / i);
        i = j;
    }

    return !printf("%d\n", res);
}
```

## [洛谷月赛] 欧拉函数求和

> 给定 $l,r$ 求：
> $$
> \sum_{i=l}^r i-\varphi(i) \text{ mod } 666623333
> $$
> 其中 $1\le l\le r\le 10^{12}, r-l\le 10^6$。
>
> 题目链接：[P3601](https://www.luogu.com.cn/problem/P3601)。

这也是区间筛的一个例子，我们先用 $\sqrt{10^{12}}$ 以内的数枚举掉 $[l,r]$ 每一个数的质因子，对于大于 $\sqrt {10^{12}}$ 的因子特判一下即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1000010, P = 666623333;
typedef long long LL;
LL l, r;

LL phi[N], remain[N];
bitset<N> st;
vector<int> primes;

void init(int n) {
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes.emplace_back(i);
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i*primes[j]] = 1;
            if (i % primes[j] == 0) break;
        }
    }
}

int main() {
    init(N-1);
    scanf("%lld%lld", &l, &r);
    for (LL i = l; i <= r; i++) phi[i-l] = remain[i-l] = i;

    for (int p: primes) {
        for (LL i = (l+p-1)/p*p; i <= r; i += p) {
            bool flag = false;
            while (remain[i-l] % p == 0) {
                remain[i-l] /= p;
                flag = true;
            }
            if (flag) phi[i-l] -= phi[i-l] / p;
        }
    }

    LL res = 0;
    for (LL i = l; i <= r; i++) {
        if (remain[i-l] > 1) phi[i-l] -= phi[i-l] / remain[i-l];
        res = (res + i - phi[i-l]) % P;
    }
    return !printf("%lld\n", res);
}
```

## [UR #3] 次大公约数

> 设 $\text{sgcd}(x,y)$ 为 $x,y$ 的次大公约数，如果不存在则 $\text{sgcd}(x,y)=-1$，给定长度为 $n$ 的序列 $a$，输出 $\text{sgcd}(a_1,a_1),\text{sgcd}(a_1,a_2),\cdots,\text{sgcd}(a_1,a_n)$。
>
> 数据范围 $1\le n\le 10^5, 1\le a_i\le 10^{12}$
>
> 题目链接：[UOJ48](https://uoj.ac/problem/48)。

显然 $\text{sgcd}(a,b)$ 是 $\gcd(a,b)$ 除掉最小的那个质因数。

我们考虑题目反复用到了 $a_1$，而 $\gcd$ 的式子从质因数分解的角度来看应该是这样的：
$$
\gcd(\prod p_i^{x_i},\prod p_i^{y_i})=\prod p_i^{\min(x_i,y_i)}
$$
所以，我们先把 $a_1$ 分解，$\gcd$ 中的质因子一定包含于 $a_1$ 的质因子，而质因子的数量是比 $\log$ 级别还要低的，所以这题就能过了。

 ```cpp
 #include <bits/stdc++.h>
 using namespace std;
 
 typedef long long LL;
 
 const int N = 100010, M = 1000010;
 int n;
 LL a[N];
 vector<LL> factors;
 
 LL gcd(LL a, LL b) {
     if (!b) return a;
     return gcd(b, a % b);
 }
 
 LL sgcd(LL b) {
     LL g = gcd(a[1], b);
     if (g == 1) return -1;
 
     for (LL f: factors)
         if (g % f == 0)
             return g / f;
 
     // Unreachable
     return -2;
 }
 
 void factorize(LL x) {
     for (int i = 2; i <= x/i; i++) {
         bool flag = false;
         while (x % i == 0) {
             x /= i;
             flag = true;
         }
         if (flag) factors.emplace_back(i);
     }
     if (x > 1) factors.emplace_back(x);
 }
 
 int main() {
     scanf("%d", &n);
     for (int i = 1; i <= n; i++) scanf("%lld", &a[i]);
 
     factorize(a[1]);
     for (int i = 1; i <= n; i++) printf("%lld ", sgcd(a[i]));
 
     return 0;
 }
 ```

## [HAOI2008] 圆上的整点

> 给定 $r\le 2\times 10^9$，求圆 $x^2+y^2=r^2$ 上有多少个整点。
>
> 题目链接：[P2508](https://www.luogu.com.cn/problem/P2508)。

勾股方程通解，其中 $\gcd(u,v)=1$：
$$
\begin{cases}
x=\frac{1}{2}d(u^2-v^2)\\
y=duv\\
r=\frac{1}{2}d(u^2+v^2)
\end{cases}
$$
若 $\gcd(u,v)\ne 1$，我们可以乘给 $d$，所以这样做是可以不重复地找出所有符合要求的三元组 $(d,u,v)$。

首先枚举 $d$ 满足 $d\mid 2r$，然后枚举 $u$ 满足 $v^2=\frac{2r}{d}-u^2$ 则有 $u^2\lt 2r/d$，检查 $v$ 是否是完全平方数即可。

由于圆上的点是对称的，所以只需要关心第一象限内的点，即 $x>0,y>0$，然后答案乘 4 加上坐标轴上的四个点就是最终答案。

复杂度为 $O(2r \sum_{d\mid 2r} \frac{1}{\sqrt{d}})$，由于约数个数是 $O(\sqrt n)$ 的，因此复杂度上界为 $O(r)$，但是远远达不到。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
LL r;

LL gcd(LL a, LL b) {
    if (!b) return a;
    return gcd(b, a % b);
}

bool check(LL u, LL v2) {
    LL v = sqrt(v2);
    if (v * v == v2) {
        return gcd(u, v) == 1;
    }
    return false;
}

LL calc(LL d) {
    LL res = 0;
    // u != v, 因为 x != 0
    for (LL u = 1; u * u * 2 < d; u++) {
        LL v2 = d - u * u;
        res += check(u, v2);
    }
    return res;
}

int main() {
    scanf("%lld", &r);

    LL res = 0;
    for (LL d = 1; d * d <= 2 * r; d++)
        if (2 * r % d == 0)
            res += calc(2 * r / d) + (d * d  == 2 * r ? 0 : calc(d));

    return !printf("%lld\n", res * 4 + 4);
}
```

## [NOIP2014] 解方程

> 已知多项式方程：
>
> $$a_0+a_1x+a_2x^2+\cdots+a_nx^n=0$$ 
>
> 求这个方程在 $[1,m]$  内的整数解（$n$  和 $m$  均为正整数）。
>
> **输入格式**
>
> 输入共 $n + 2$  行。  
>
> 第一行包含 $2$  个整数 $n, m$ ，每两个整数之间用一个空格隔开。  
>
> 接下来的 $n+1$  行每行包含一个整数，依次为 $a_0,a_1,a_2\ldots a_n$ 。
>
> **输出格式**
>
> 第一行输出方程在 $[1,m]$  内的整数解的个数。  
>
> 接下来每行一个整数，按照从小到大的顺序依次输出方程在 $[1,m]$  内的一个整数解。
>
> 对于 $100\%$  的数据：$0<n\le 100,|a_i|\le 10^{10000},a_n≠0,m<10^6$ 。
>
> 题目链接：[LOJ 2503](https://loj.ac/p/2503)。

判断两个大数字是否相等，可以通过判断它们是否在模大质数下是否同余。这个判断方法只能断定两个数字是否不同，无法判断是否相同，但是在 $A\ne B$ 的情况下做到 $A\equiv B\pmod P$ 需要使得 $A-B=kP$。

本题是判断大数是否和 $0$ 相等，这种情况下还能同余的概率是 $\frac{1}{P}$，当然出题人可能刻意卡一些常见的质数，所以我们同时用两个不怎么常见的质数，$998244853,10^9+9$ 就能过。

这题放洛谷不写 `inline` 过不了，我认为这是洛谷的问题，因为 `LOJ` 开不开 `inline` 差别只有 `10ms`，洛谷直接 TLE 了。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 110, M = 10010;
const int P = 998244853, Q = 1e9+9;
int a[N], b[N], n, m;

void read(int &t, int& r) {
    t = r = 0;
    int sgn = 1;
    char ch = getchar();
    while (ch < '0' || ch > '9') {
        if (ch == '-') sgn = -1;
        ch = getchar();
    }

    while ('0' <= ch && ch <= '9') {
        t = (t * 10LL + (ch ^ 48)) % P;
        r = (r * 10LL + (ch ^ 48)) % Q;
        ch = getchar();
    }

    if (sgn == -1) t = P-t, r = Q-r;
}

bool check(int a[], int v, int p) {
    int x = 1, res = 0;
    for (int i = 0; i <= n; i++) {
        res = (res + 1LL * a[i] * x) % p;
        x = 1LL * x * v % p;
    }
    return res == 0;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 0; i <= n; i++) read(a[i], b[i]);

    vector<int> sln;
    for (int i = 1; i <= m; i++)
        if (check(a, i, P) && check(b, i, Q))
            sln.emplace_back(i);

    printf("%d\n", (int)sln.size());
    for (int s: sln) printf("%d\n", s);

    return 0;
}
```

## [CF226C] Anniversary

> 给定 $1\le l\le r\le 10^{12}$，求在 $[l,r]$ 中选 $k$ 个数字，使得 $\gcd(f_{a_1},\dots,f_{a_k})$ 最大，其中 $f$ 是斐波那契数列，答案对 $m$ 取模。
>
> 题目链接：[CF226C](https://codeforces.com/problemset/problem/226/C)。

由于 $\gcd(f_a,f_b)=f_{\gcd(a,b)}$，并且 $f_i$ 单调递增，所以就是要在 $[l,r]$ 内选 $k$ 个数使他们的 $\gcd$ 最大。

考虑枚举答案 $x$，要想让答案成立，一定有 $[l,r]$ 内 $x$ 的倍数大于 $k$，也就是：
$$
\lfloor\frac{r}{x}\rfloor-\lfloor\frac{l-1}{x}\rfloor\ge k
$$
具体地，我们直接选 $\gcd(x,2x,\cdots,kx)=x$，这是因为 $\forall k\in \N^*,\gcd(x,kx)=x\gcd(1,k)=x$。

这可以整除分块，所以复杂度为 $O(\sqrt n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
int l, r, m, k;

struct Matrix {
    int dat[2][2];

    Matrix() {
        memset(dat, 0, sizeof dat);
    }

    Matrix operator*(const Matrix& mat) const {
        Matrix res;

        for (int k = 0; k < 2; k++)
            for (int i = 0; i < 2; i++)
                for (int j = 0; j < 2; j++)
                    res.dat[i][j] = (res.dat[i][j] + dat[i][k] * mat.dat[k][j]) % m;
        
        return res;
    }
};

Matrix qmi(Matrix a, int k) {
    Matrix res;
    res.dat[0][0] = res.dat[1][1] = 1;

    while (k) {
        if (k & 1) res = res * a;
        a = a * a;
        k >>= 1;
    }

    return res;
}

signed main() {
    cin >> m >> l >> r >> k;
    --l;
    int x = 1, res = 1;
    while (x <= r) {
        if (l / x) x = min(l / (l / x), r / (r / x));
        else x = r / (r / x);

        if (r/x - l/x >= k) res = x;
        x++;
    }

    Matrix mat;
    mat.dat[0][0] = mat.dat[0][1] = mat.dat[1][0] = 1;
    cout << qmi(mat, res-1).dat[0][0] % m << '\n';
    return 0;
}
```

