---
date: 2024-01-21 02:50:00
title: 数学 - BSGS
description: BSGS 算法即大步小步算法是利用分块与哈希表，在根号的复杂度求解离散对数问题的算法。
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
---

## [模板] BSGS

> 给定方程 $a^{x}\equiv b\pmod p$，保证 $p$ 是质数，求最小的非负整数解 $x$ 或报告无解。
>
> 题目链接：[P3846](https://www.luogu.com.cn/problem/P3846)。

由欧拉定理可知：
$$
a^x\equiv a^{x\bmod \varphi(p)}\pmod p
$$
因此如果原方程有解，那么一定存在 $[0,\varphi(p))$ 范围内的解。

考虑分块，令块长为 $\sqrt p$ 并且令 $x=i\sqrt p-j$ 其中 $i,j\in[1,\sqrt p]$ 使得 $x\in[0,p-1]$ 那么有：
$$
\begin{aligned}
a^{i\sqrt p-j}&\equiv b\pmod p\\
a^{i\sqrt p}&\equiv a^jb\pmod p
\end{aligned}
$$
所以在 $O(\sqrt p)$ 的时间把 $a^jb,a^{i\sqrt p}$ 都打表出来就可以了，代码在下方 exBSGS 一并给出。

事实上，如果用哈希表，那么时间复杂度理论上确实是 $O(\sqrt p)$，但是默认的 `unordered_map` 是可以被构造的数据[卡到平方](https://codeforces.com/blog/entry/62393)的，用 `map` 可以保证复杂度为 $O(\sqrt p\log p)$。

## [模板] exBSGS

> 给定方程 $a^x\equiv b\pmod p$ 不保证 $p$ 是质数，求最小的非负整数解或报告无解。
>
> 题目链接：[P4195](https://www.luogu.com.cn/problem/P4195)。

普通 BSGS 算法的适用条件是 $(a,p)= 1$，那么当 $(a,p)\ne 1$ 时，可以对原方程进行变形：
$$
a\cdot a^{x-1}-kp=b
$$
由裴蜀定理可知当 $(a,p)\not\mid b$ 时此方程无解，直接报告无解，否则：
$$
\frac{a}{(a,p)} a^{x-1}-k\frac{p}{(a,p)}=\frac{b}{(a,p)}
$$
写成同余方程的形式：
$$
\frac{a}{(a,p)}a^{x-1}\equiv \frac{b}{(a,p)}\pmod{\frac{p}{(a,p)}}
$$
由于 $(\frac{a}{(a,p)},\frac{p}{(a,p)})=1$ 那么前者在模后者意义下的逆元一定是存在的，那么：
$$
a^{x-1}\equiv \frac{b}{(a,p)}[\frac{a}{(a,p)}]^{-1}\pmod {\frac{p}{(a,p)}}
$$
递归求解，显然当 $x-1$ 最小时 $x$ 最小，所以最终求到的还是最小解。

当 $p=1$ 或者 $b=1$ 时可以快速知道 $x=0$，其它情况就不要判了，容易出错。

```cpp
#include <bits/stdc++.h>
using namespace std;

namespace IO {}
using IO::read;
using IO::write;

#define int long long

int gcd(int a, int b) {
    if (!b) return a;
    return gcd(b, a % b);
}

int exgcd(int a, int b, int& x, int& y) {
    if (!b) return x = 1, y = 0, a;
    int d = exgcd(b, a % b, y, x);
    return y -= a / b * x, d;
}

int inv(int a, int p) {
    int res, k, mo, d = exgcd(a, p, res, k);
    mo = p / d;
    res %= mo;
    if (res < 0) res += mo;
    return res;
}

map<int, int> mp;

int BSGS(int a, int b, int p) {
    mp.clear();
    int m = sqrt(p) + 1, aj = a;
    for (int j = 1; j <= m; j++) {
        mp[b * aj % p] = j;
        if (j != m) aj = aj * a % p;
    }
    int ai = aj;
    for (int i = 1; i <= m; i++, ai = ai * aj % p)
        if (mp.count(ai)) return i * m - mp[ai];
    return -1;
}

int exBSGS(int a, int b, int p) {
    a %= p, b %= p;
    if (p == 1 || b == 1) return 0;

    int d = gcd(a, p);
    if (d == 1) return BSGS(a, b, p);
    if (b % d) return -1;
    int res = exBSGS(a, b / d * inv(a / d, p / d), p / d);
    if (res == -1) return -1;
    return res + 1;
}

signed main() {
    int a, p, b;
    while (read(a, p, b), a || p || b) {
        int x = exBSGS(a, b, p);
        if (x == -1) write("No Solution");
        else write(x);
    }
    return 0;
}
```

## 斐波那契数列

> 给定数列满足 $f_1=f_2=1,f_n=f_{n-1}+f_{n-2}(n\ge 2)$，求 $f_n\bmod p$，其中 $n\le 10^{30000000},p\lt 2^{31}$。
>
> 题目链接：[P4000](https://www.luogu.com.cn/problem/P4000)。

首先不加证明地给出结论，斐波那契数列在模 $p$ 意义下循环节的长度 $\pi(p)\le 6p$，根据递推式：
$$
F_n=\begin{bmatrix}F_n\\F_{n-1}\end{bmatrix},A=\begin{bmatrix} 1&1\\1&0\end{bmatrix},F_n=A^nF_0
$$
为了满足斐波那契数列的性质，可以推出 $f_0=0$，$f_{-1}=1$，这两个值组成 $F_0$ 向量。

设循环节为 $x$ 那么就有 $f_n\equiv f_{n\bmod x}\pmod p$，这可以很好的将 $n$ 取模。

于是我们要求解的方程就是：
$$
\begin{aligned}
F_{0}A^{x}&\equiv F_{0}\pmod p\\
A^{x}&\equiv E\pmod p
\end{aligned}
$$
其中 $E$ 为单位阵，这里的同余代表元素对应同余，在这里请不要纠结是否真的有有这样的表达方式，由于 $x$ 是一个 $6p$ 范围内的数字，所以可以 BSGS 求解。但是，上面的 BSGS 检验了 $x=0$ 是否为方程的解，让 $i,j\in [1,m]$；而显然这里要求 $x>0$ 才有意义，所以写法略微发生改变，需要让 $i\in[1,m],j\in[0,m-1]$ 然后判断 $im-j\in[1,m^2]$ 就求出非 $0$ 解了。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
typedef unsigned long long ull;
int p;
char s[30000010];

struct Matrix {
    int dat[2][2];

    Matrix() { memset(dat, 0, sizeof dat); }

    ull hash() {
        ull res = 0;
        for (int i = 0; i < 2; i++)
            for (int j = 0; j < 2; j++)
                res = res * p + dat[i][j];
        return res;
    }

    Matrix operator*(const Matrix& mat) {
        Matrix res;
        for (int i = 0; i < 2; i++)
            for (int k = 0; k < 2; k++)
                for (int j = 0; j < 2; j++)
                    res.dat[i][j] = (res.dat[i][j] + dat[i][k] * mat.dat[k][j]) % p;
        return res;
    }
} A, E;

map<ull, int> mp;

Matrix qmi(Matrix a, int k) {
    Matrix res = E;
    while (k) {
        if (k & 1) res = res * a;
        a = a * a;
        k >>= 1;
    }
    return res;
}

int BSGS() {
    int m = sqrt(6*p) + 1;
    Matrix Aj = E;
    for (int j = 0; j < m; j++, Aj = Aj * A) mp[Aj.hash()] = j;
    Matrix Ai = Aj;
    for (int i = 1; i <= m; i++, Ai = Ai * Aj)
        if (mp.count(Ai.hash())) return i * m - mp[Ai.hash()];
    assert(false);
}

signed main() {
    A.dat[0][0] = A.dat[0][1] = A.dat[1][0] = 1;
    E.dat[0][0] = E.dat[1][1] = 1;
    scanf("%s%lld", s, &p);
    int x = BSGS(), n = 0;
    for (int i = 0; s[i]; i++) n = (n * 10 + s[i] - '0') % x;
    return !printf("%lld\n", qmi(A, n).dat[0][1]);
}
```

## [SDOI2013] 随机数生成器

> 共 $T$ 组测试数据，给定 $p,a,b,x_1,t$ 其中数列 $x_i$ 满足递推式：$x_{i+1}\equiv ax_i+b\pmod p$，问 $i$ 最小为多少使得 $x_i=t$ 或报告无解。
>
> 数据范围：$1\le T\le 50$，$2\le p\le 10^9$，$0\le a,b,x_1,t\lt p$，$p$ 为质数。
>
> 题目链接：[P3306](https://www.luogu.com.cn/problem/P3306)。

首先用矩阵写出来递推式：
$$
\begin{bmatrix}
x_{i+1}\\
1
\end{bmatrix}
=\begin{bmatrix}
a&b\\
0&1
\end{bmatrix}
\begin{bmatrix}
x_i\\1
\end{bmatrix}
$$
令 $A$ 为转移矩阵，和上面的题一样，信仰矩阵的循环节是关于 $p$ 线性的，先写一个辅助程序观察循环节的数量级：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100000010;
int primes[N/10], cnt;
bitset<N> st;
mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());

int rd(int l, int r) {
    return uniform_int_distribution<int>(l, r)(rng);
}

void init(int n) {
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[++cnt] = i;
        for (int j = 1; i <= n/primes[j]; j++) {
            st[i*primes[j]] = 1;
            if (i % primes[j] == 0) break;
        }
    }
}

int calc(int a, int b, int x1, int p) {
    static int st[N], ts;
    st[x1] = ++ts;
    int ans = 0;
    while (1) {
        int nxt = (1ll * a * x1 + b) % p;
        if (st[nxt] == ts) return ans;
        st[x1 = nxt] = ts, ans++;
    }
    assert(false);
}

int main() {
    int n = N-1;
    init(n);
    for (int i = 1; i <= 10; i++) {
        int k = rd(1, cnt);
        int p = primes[k];
        int a = rd(0, p-1), b = rd(0, p-1), x1 = rd(0, p-1);
        printf("%d: %d,\n", p, calc(a, b, x1, p));
    }
    return 0;
}
```

发现循环节大概率不会超过 $p$，和我们的感觉是相同的，所以为了保险起见直接将值域定为 $1\sim 4p$，然后如果在这个区域内都没有解就认为无解，那么接下来考虑方程 $X=A^nX_0$，按照 BSGS 那样分块。

首先根据递推式可以求出 $A$ 的逆矩阵：
$$
A^{-1}=\begin{bmatrix}
\frac{1}{a}&-\frac{b}{a}\\
0&1
\end{bmatrix}
$$
令 $m=2\sqrt p$ 为块长，若 $n$ 有解一定可以拆成 $n=im-j$，其中 $i\in[1,m],j\in[0,m-1]$：
$$
(A^{-1})^{im}X=(A^{-1})^jX_0
$$
对 $A^{-1}X_0,A^{-2}X_0,\dots,A^{-j}X_0$ 打表，然后枚举 $i$ 判断 $A^{-im}X$ 是否存在即可，第一次找到的就是最小值，如果找不到视为无解。

由于上面的基础是 $a$ 存在模 $p$ 逆元，即 $a\ne 0$，当 $a=0$ 时有 $x_i=b$ 直接根据 $t$ 特判即可。

 ```cpp
 #include <bits/stdc++.h>
 using namespace std;
 
 namespace IO {}
 using IO::read;
 using IO::write;
 
 #define int long long
 map<int, int> mp;
 int T, a, b, x1, t, p;
 
 struct Matrix {
     int dat[2][2];
 
     Matrix() { memset(dat, 0, sizeof dat); }
 
     Matrix operator*(const Matrix& mat) const {
         Matrix res;
         res.dat[0][0] = (dat[0][0] * mat.dat[0][0] + dat[0][1] * mat.dat[1][0]) % p;
         res.dat[0][1] = (dat[0][0] * mat.dat[0][1] + dat[0][1] * mat.dat[1][1]) % p;
         res.dat[1][0] = (dat[1][0] * mat.dat[0][0] + dat[1][1] * mat.dat[1][0]) % p;
         res.dat[1][1] = (dat[1][0] * mat.dat[0][1] + dat[1][1] * mat.dat[1][1]) % p;
         return res;
     }
 
     int mul(int x) {
         return (dat[0][0] * x + dat[0][1]) % p;
     }
 } E;
 
 int qmi(int a, int k, int p) {
     int res = 1;
     while (k) {
         if (k & 1) res = res * a % p;
         a = a * a % p;
         k >>= 1;
     }
     return res;
 }
 
 int BSGS() {
     mp.clear();
     int m = sqrt(p) * 2, inva = qmi(a, p-2, p);
     Matrix A;
     A.dat[0][0] = inva, A.dat[0][1] = -b * inva % p + p, A.dat[1][1] = 1;
     int x0 = A.mul(x1);
     Matrix Aj = E;
     for (int j = 0; j < m; j++, Aj = Aj * A) mp[Aj.mul(x0)] = j;
     Matrix Ai = Aj;
     for (int i = 1; i <= m; i++, Ai = Ai * Aj) {
         int x = Ai.mul(t);
         if (mp.count(x)) return i * m - mp[x];
     }
     return -1;
 }
 
 int solve() {
     if (a) return BSGS();
     if (t == x1) return 1;
     if (t == b) return 2;
     return -1;
 }
 
 signed main() {
     E.dat[0][0] = E.dat[1][1] = 1;
     read(T);
     while (T--) {
         read(p, a, b, x1, t);
         write(solve());
     }
     return 0;
 }
 ```

