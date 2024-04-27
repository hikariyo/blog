---
date: 2023-07-29 20:24:00
title: 数学 - 欧拉函数与欧拉定理
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 欧拉函数

欧拉函数 $\varphi(N)$ 的定义如下：在 1 ~ N 中所有与 N 互质的数字的个数，计算如下：
$$
\begin{aligned}
N&=p_1^{\alpha_1}p_2^{\alpha_2}\cdots p_n^{\alpha_n}\\
\varphi(N)&=N(1-\frac{1}{p_1})(1-\frac{1}{p_2})\cdots(1-\frac{1}{p_n})
\end{aligned}
$$

证明需要用到容斥原理：先把所有 $p_i$ 的倍数排除，这时会把 $p_ip_j$ 的倍数排除两遍，把 $p_ip_jp_k$ 的倍数排除三遍；于是加入 $p_ip_j$ 的倍数，这样会把 $p_ip_jp_k$ 的倍数加入三遍；然后再排除 $p_ip_jp_k$ 的倍数... 如下：


$$
\begin{aligned}
\varphi(N)=N&-\frac{N}{p_1}-\frac{N}{p_2}-\frac{N}{p_3}\cdots\\
&+\frac{N}{p_1p_2}+\frac{N}{p_2p_3}+\frac{N}{p_1p_3}+\cdots\\
&-\frac{N}{p_1p_2p_3}-\cdots\\
\end{aligned}
$$
观察到，所有奇数个 $p$ 相乘的都为负，所有偶数个相乘的都为正，而且相乘的个数取决于有多少个$p$，因此开头给出的定义式展开符合这一点。

可以进一步说明，对于有 $m$ 个 $p$ 相乘的项共有 $\binom{n}{m}$ 个，开头给出的式子中，在 $n$ 个括号中选出 $m$ 个含 $p$ 的项相乘共有 $\binom{n}{m}$ 中选法，所以等式成立。

## 欧拉定理

若 $a$ 与 $x$ 互质，则 $a^{\varphi(x)}\equiv 1 \pmod x$：

设所有小于 $x$ 的数中所有与 $x$ 互质的数是 $x_1, x_2, \cdots, x_{\varphi(x)}$，那么 $ax_1, ax_2, \cdots, ax_{\varphi(x)}$ 也与 $x$ 互质（无论是 $a$ 还是 $x_i$ 都与 $x$ 没有相同的质因数，所以 $ax_i$ 也与 $x$ 没有相同的质因数）。

并且 $ax_1, ax_2, \cdots, ax_{\varphi(x)}$ 对 $x$ 取模两两不相同，这步用反证法证明，假设 $ax_i\equiv ax_j\pmod x$ 由于 $\gcd(a,x)=1$ 有 $x_i\equiv x_j\pmod x$ 这是矛盾的。

于是这两组数模 $x$ 之后其实是同一组数据，那么它们取模之后全部乘起来结果是相同的。
$$
\begin{aligned}
\prod_{i=1}^{\varphi(x)}[(ax_i)\bmod x]&=\prod_{i=1}^{\varphi(x)}x_i\\
a^{\varphi(x)}\prod_{i=1}^{\varphi(x)}x_i&\equiv \prod_{i=1}^{\varphi(x)}x_i \pmod x\\
\end{aligned}
$$
由于任意一个 $x_i$ 都与 $x$ 互质，所以它们的乘积也与 $x$ 互质，所以 $a^{\varphi(x)}\equiv 1\pmod x$。

特别地，如果设 $p$ 是质数，那么 $a^{\varphi(p)}\equiv a^{p-1} \equiv 1\pmod p$，欧拉定理是费马小定理的拓展。

## 拓展欧拉定理

不要求 $a,m$ 互质，且 $b\ge \varphi(m)$ 满足 $a^b\equiv a^{b\bmod  \varphi(m)+\varphi(m)}\pmod m$，这也是降幂公式。

### 引理1

如果 $\gcd(m_1,m_2)=1$ 时， $x\equiv y\pmod {m_1},x\equiv y\pmod{m_2}$ 那么一定有 $x-y\equiv 0\pmod {m_1m_2}$。

证明：$x-y$ 是 $m_1,m_2$ 的公倍数，所以 $x-y\equiv 0\pmod {m_1m_2}$

### 引理2

若 $p$ 为质数，那么 $\varphi(p^k)=p^k(1-\frac{1}{p})=p^{k-1}(p-1)$，有 $\varphi(p^k)\ge k$

证明：如果固定 $k$，那么 $\varphi(p^k)$ 是对 $p$ 单调递增的，因为是两个单调递增的函数相乘。

因此，$\varphi(p^k)\ge \varphi(2^k)= 2^{k-1}\ge k$，关于最后一个不等式，我们可以构造函数 $f(x)=2^{x-1}-x$ 研究。
$$
\begin{aligned}
f(x)&=2^{x-1}-x\\
f'(x)&=2^{x-1}\ln 2-1,f'(x)\nearrow \\
f'(x_0)&=0\Rightarrow x_0=\log_2\log_2 e+1\in(1,2)\\
f(1)&=f(2)=0\\
\therefore f(x)&\ge 0,x\in \N^*
\end{aligned}
$$
证毕。

### 证明

首先，$\gcd(a,m)=1$ 时，有 $a^{\varphi(m)}\equiv 1\pmod m$，那么 $a^{b}\equiv a^{b-k\varphi(m)}\cdot a^{\varphi(m)}\pmod m$ 显然成立。主要证明 $\gcd(a,m)\ne 1$ 时的情况。

将 $m$ 分解质因数，只需要证明 $a^b\equiv a^{b\bmod \varphi(m)+\varphi(m)}\pmod {p_1^{k_1},p_2^{k_2},\cdots}$ 即可。

1. 当 $\gcd(a,p^{k})=1$ 时，由于欧拉函数是积性函数，那么 $\varphi(p^k)|\varphi(m)$：
    $$
    \begin{aligned}
    a^{b\bmod \varphi(m)+\varphi(m)}&\equiv a^{b-k\varphi(m)+\varphi(m)}\\
    &\equiv a^{b-k'\varphi(p^k)+\varphi(p^k)}\\
    &\equiv a^b \pmod {p^k}
    \end{aligned}
    $$
    最后一步直接套用上面已经讨论过的情况。

2. 当 $\gcd(a,p^k)\ne 1$ 时，有 $p|a$，设 $a=np$，由于需要保证 $b\ge \varphi(m)$，并且 $\varphi(m)\ge \varphi(p^k)\ge k$ 那么有：
    $$
    \begin{aligned}
    a^b&\equiv n^bp^b\equiv n^bp^{b-k}p^k\equiv 0\pmod {p^k}\\
    a^{b\bmod \varphi(m)+\varphi(m)}&\equiv n^{b\bmod \varphi(m)+\varphi(m)}p^{b\bmod \varphi(m)+\varphi(m)-k}p^k \equiv 0\pmod {p^k}\\
    a^b&\equiv a^{b\bmod \varphi(m)+\varphi(m)} \pmod{p^k}
    \end{aligned}
    $$

综上，$a^b\equiv a^{b\bmod  \varphi(m)+\varphi(m)}\pmod{m},b\ge \varphi(m)$，这也被称作降幂公式。

## 定义法求欧拉函数

先找质因数，然后累项求积。

```cpp
int euler(int a) {
    int res = a;
    for (int i = 2; i <= a/i; i++) {
        // 这里就是先用 if 然后在里面把 a 除干净
        if (a % i == 0) {
            // res(1-1/i) = res-res/i
            res = res - res/i;
            while (a % i == 0) a /= i;
        }
    }
    // 一定注意不要落了 a 没除干净的情况
    if (a > 1) res = res-res/a;
    return res;
}
```

## 线筛法求欧拉函数

与筛选质数时的模板类似。

```cpp
#include <bitset>
#include <iostream>
using namespace std;

const int N = 1e6+10;
int phi[N], primes[N], cnt=0;
bitset<N> st;

void eular(int n) {
    phi[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) {
            primes[cnt++] = i;
            phi[i] = i-1;
        }
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i * primes[j]] = 1;
            if (i % primes[j] == 0) {
                phi[i * primes[j]] = phi[i] * primes[j];
                break;
            } else {
                phi[i * primes[j]] = phi[i] * (primes[j] - 1);
            }
        }
    }
}
```

可以证明当要求 $i\cdot p_j$ 的时候，$\varphi(i)$ 一定已经求出来了：

1. 如果 $i$ 是质数，那么它会在进入内层 `for` 循环前被求出来。
2. 如果 $i$ 是合数，那么它会在 $i$ 循环到自己之前就被筛掉，筛掉的同时也就求出来了 $\varphi(i)$。

接下来通过公式证明 $\varphi(i)$ 的这几个递推式：

1. 如果 $i$ 是质数，那么在 $1 \sim i-1$ 中所有数都与它互质，所以有 $\varphi(i)=i-1$。
2. 如果 $i \bmod  p_j=0$，此时 $p_j$ 是 $i$ 的最小质因数，$i$ 表示为 $p_1^{\alpha_1}p_2^{\alpha_2}\cdots$ 这个式子中一定包含了 $p_j$，由欧拉函数的定义知，$\varphi(p_j\cdot i)=p_j\varphi(i)$。
3. 如果 $i\bmod p_j\ne 0$，此时 $p_j$ 不是 $i$ 的质因数，那么 $\varphi(p_j\cdot i)=\varphi(p_j)\cdot \varphi(i)=(p_j-1)\varphi(i)$。

## [模板] 降幂

> 给定 $a,m,b$ 求 $a^b \bmod m$，其中 $1\le a\le 10^9,1\le b\le10^{20000000},1\le m\le 10^8$。
>
> 题目链接：[P5091](https://www.luogu.com.cn/problem/P5091)。

对 $b$ 边读边取模即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

int a, m, b;
int phi;

int euler(int m) {
    int res = m;
    for (int i = 2; i <= m/i; i++) {
        bool flag = false;
        while (m % i == 0) {
            flag = true;
            m /= i;
        }
        if (flag) res -= res / i;
    }
    if (m > 1) res -= res / m;
    return res;
}

int read() {
    char ch = getchar();
    while (ch < '0' || ch > '9') ch = getchar();

    bool flag = false;
    int res = 0;
    while ('0' <= ch && ch <= '9') {
        res = res * 10 + (ch ^ 48);
        if (res > phi) {
            flag = true;
            res %= phi;
        }
        ch = getchar();
    }
    return res + flag * phi;
}

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = 1LL * res * a % m;
        a = 1LL * a * a % m;
        k >>= 1;
    }
    return res;
}

int main() {
    scanf("%d%d", &a, &m);
    phi = euler(m);
    return !printf("%d\n", qmi(a, read()));
}
```

## 上帝与集合的正确用法

> 定义 $a_0=1$，$a_n=2^{a_{n-1}}$，可以证明 $a_n\bmod p$ 在某一项后都是同一个值，求这个值。
>
> 输入 $T$ 个 $p$，对每个 $p$ 输出指定值，$T\le 10^3, p\le 10^7$。
>
> 题目链接：[P4139](https://www.luogu.com.cn/problem/P4139)。

先套一下拓展欧拉定理公式。
$$
\begin{aligned}
f(p)&=2^{2^{2^{\cdots}}\text {mod }\varphi(p)+\varphi(p)}\bmod p\\
&=2^{f(\varphi(p))+\varphi(p)}\bmod p
\end{aligned}
$$
不断取 $\varphi(p)$ 一定会到达 $1$，而 $f(1)=0$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 10000010;

int phi[N];
vector<int> primes;
bitset<N> st;

void init(int n) {
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes.emplace_back(i), phi[i] = i-1;
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i*primes[j]] = 1;
            if (i % primes[j] == 0) {
                phi[i*primes[j]] = primes[j] * phi[i];
                break;
            }
            else {
                phi[i*primes[j]] = (primes[j]-1) * phi[i];
            }
        }
    }
}

int qmi(int a, int k, int p) {
    int res = 1;
    while (k) {
        if (k & 1) res = 1LL * res * a % p;
        a = 1LL * a * a % p;
        k >>= 1;
    }
    return res;
}

int f(int p) {
    if (p == 1) return 0;
    return qmi(2, f(phi[p])+phi[p], p);
}

int main() {
    int T, p;
    init(N-1);
    scanf("%d", &T);
    while (T--) scanf("%d", &p), printf("%d\n", f(p));

    return 0;
}
```

## 可见的点

> 在一个平面直角坐标系的第一象限内，如果一个点 (x,y) 与原点 (0,0) 的连线中没有通过其他任何点，则称该点在原点处是可见的。
>
> 例如，点 (4,2) 就是不可见的，因为它与原点的连线会通过点 (2,1)。
>
> 编写一个程序，计算给定整数 N 的情况下，满足 0≤x, y≤N 的可见点 (x，y) 的数量（可见点不包括原点）。
>
> 题目链接：[AcWing 201](https://www.acwing.com/problem/content/203/)。

如果一个点 $(x_0,y_0)$ 可见，必有 $k=y_0/x_0$ 不可化简，也就是 $y_0, x_0$ 互质。

所以答案是 $2\sum_{i=1}^n \varphi(i)+1$，去掉一个直线 $y=x$，加上两个直线 $y=0,x=0$

```cpp
#include <iostream>
#include <vector>
using namespace std;

const int N = 1010, m = N-1;
int phi[N];
bool st[N];
vector<int> primes;

void init() {
    phi[1] = 1;
    for (int i = 2; i <= m; i++) {
        if (!st[i]) {
            primes.push_back(i);
            phi[i] = i-1;
        }
        for (int j = 0; primes[j] <= m/i; j++) {
            st[i*primes[j]] = true;
            if (i % primes[j] == 0) {
                phi[i*primes[j]] = phi[i] * primes[j];
                break;
            }
            phi[i*primes[j]] = phi[i] * (primes[j]-1);
        }
    }
}

int main() {
    init();
    int T; scanf("%d", &T);
    for (int i = 1; i <= T; i++) {
        int n, sum = 0;
        scanf("%d", &n);
        for (int j = 1; j <= n; j++)
            sum += phi[j];
        printf("%d %d %d\n", i, n, 2*sum+1);
    }
    return 0;
}
```

## 选择美食

> 餐厅里有 $n$ 种美食，每种美食都有一个美味值 $a_i$ 和价格 $c_i$ ，小 P 可以选择任意个美食，且每一种美食可以选择多次。
>
> 若小 P 选择的美食的编号为 $b_1\ ,\ b_2\  ...\ b_m$，则小 P 能得到的满意值则为
> $$
> \prod\limits_{i=1}^ma_{b_i}
> $$
> 现在小 P 想要使他得到的满意值恰好为 $d$ ，请你告诉他最少需要花费多少钱。
>
> 由于小 P 的口味非常独特，所以他给出的 $d$ 一定满足 $\varphi\left(d\right)\ |\ d$。
>
> 由于小 P 非常不容易满足，故 $d$ 将有可能非常大，为了方便表示，小 P 会告诉你三个数 $A , B , k$ ，那么 $d = A^k \times B$ 。
>
> $1\le n\le 10^5, 1\le a_i\le 10^9,0\le c_i\le 10^9,1\le A,B\le 10^5,0\le k\le 40,d\le 10^{201}$

首先：
$$
\varphi(d)=d(1-\frac{1}{2})(1-\frac{1}{3})\cdots=d\frac{a}{b}\\
\varphi(d)\mid d\Leftrightarrow kd\frac{a}{b}=k\\
a=kb
$$
也就是 $\varphi(d)=d/a$，要么 $\varphi(d)=d/2$ 要么 $\varphi(d)=d/3$，所以 $d$ 只能有 $2,3$ 两个约数。

将每个 $a_i$ 分解为 $2^a3^b$，设 $d=2^p3^q$，也就是需要满足：
$$
p=\sum a,q=\sum b
$$
满足这个条件的情况下保证花费最小，那么这显然是个背包问题。

在 $10^9$ 范围内，满足 $2^a3^b$ 形式的数字，如果粗略的算一下就只有 $\log_2 10^9\times \log_310^9\approx 563$，这还没容斥就已经这么少了，然后 $p,q$ 的范围分别是 $\log_2 d,\log_3 d$，用 `map` 去重，复杂度是 $O(\log n\log^2 a\log^{2}d)$，实际上远小于这个数，肯定能过。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 100010;
int f[700][450], c[N], num[N];
bool valid[N];
int n, A, B, k;

map<int, int> mp;

void extract(int& v, int fac, int& cnt) {
    while (v % fac == 0) v /= fac, cnt++;
}

signed main() {
    freopen("food.in", "r", stdin);
    freopen("food.out", "w", stdout);
    memset(f, 0x3f, sizeof f);
    f[0][0] = 0;
    cin >> n;
    for (int i = 1, v; i <= n; i++) cin >> num[i];
    for (int i = 1; i <= n; i++) cin >> c[i];
    for (int i = 1; i <= n; i++) {
        int bak = num[i], t1 = 0, t2 = 0;
        extract(bak, 2, t1);
        extract(bak, 3, t2);
        if (bak == 1) {
            if (mp.count(num[i])) mp[num[i]] = min(mp[num[i]], c[i]);
            else mp[num[i]] = c[i];
        }
    }

    int t1 = 0, t2 = 0;
    cin >> A >> B >> k;
    extract(A, 2, t1);
    extract(A, 3, t2);
    t1 *= k, t2 *= k;
    extract(B, 2, t1);
    extract(B, 3, t2);
    
    for (const auto& entry: mp) {
        int a = 0, b = 0, num = entry.first;
        extract(num, 2, a);
        extract(num, 3, b);
        for (int j = a; j <= t1; j++)
            for (int k = b; k <= t2; k++)
                f[j][k] = min(f[j][k], f[j-a][k-b] + entry.second);
    }
    
    int res = f[t1][t2];
    if (res == 0x3f3f3f3f3f3f3f3fll) res = -1;
    cout << res << '\n';
    return 0;
}
```

## 最大公约数

> 给定整数 N，求 1≤x,y≤N 且 GCD(x,y) 为素数的数对 (x,y) 有多少对。
>
> 题目链接：[AcWing 220](https://www.acwing.com/problem/content/222/)。

题目是求（中间当作一个布尔表达式）：
$$
\sum_p \sum _{i=1}^n\sum_{j=1}^n [\gcd(i,j)=p]
$$
若 $i,j$ 为 $p$ 的倍数，那么下列式子等价：
$$
\gcd(i,j)=p \Leftrightarrow \gcd(i/p,j/p)=1
$$
因此枚举 $i,j=kp$ 中的 $k$ 就可以：
$$
\sum_p \sum_{i=1}^{\lfloor n/p \rfloor}\sum_{j=1}^{\lfloor n/p \rfloor}[\gcd(i,j)=1]
$$
等价于：
$$
\sum_p[2\sum_{i=1}^{\lfloor n/p \rfloor} \varphi(i)-1]
$$
对欧拉函数简单求一个前缀和就可以。

```cpp
#include <iostream>
#include <vector>
using namespace std;

typedef long long LL;
const int N = 1e7+10;
int phi[N];
LL s[N];
bool st[N];
vector<int> primes;

void init(int n) {
    s[1] = phi[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) {
            primes.push_back(i);
            phi[i] = i-1;
        }
        
        for (int j = 0; primes[j] <= n/i; j++) {
            st[primes[j]*i] = true;
            if (i % primes[j] == 0) {
                phi[primes[j]*i] = phi[i]*primes[j];
                break;
            }
            phi[primes[j]*i] = phi[i]*(primes[j]-1);
        }
        
        s[i] = s[i-1] + phi[i];
    }
}

int main() {
    int n;
    cin >> n;
    init(n);
    LL ans = 0;
    for (int p : primes) {
        ans += 2*s[n/p]-1;
    }
    cout << ans << endl;
    return 0;
}
```

## [SDOI2012] 最大公约数求和

> 给定一个整数 $N$，求出 $\sum_{i=1}^n\gcd(i,n)$
>
> 题目链接：[P2303](https://www.luogu.com.cn/problem/P2303)，[AcWing 221](https://www.acwing.com/problem/content/223/)。

首先一定有 $\gcd(i,n)|n$，因此枚举 $n$ 的所有约数，看它在整个过程中出现了几次。

对于其中一个约数 $d$，当 $\gcd(i,n)=d$ 时，有 $\gcd(i/d,n/d)=1$，其中 $i/d\in[1,n/d]$，因此它出现的个数就是 $\varphi(n/d)$，这样就有：
$$
\begin{aligned}
\sum_{i=1}^n\gcd(i,n)&=\sum_{d|n}d\varphi(\frac{n}{d})\\
&=\sum_{d|n}\frac{n}{d}\varphi(d)\\
&=n\sum_{d|n}\prod_{i=1}^{k_d}(1-\frac{1}{p_i})
\end{aligned}
$$

这里有一个常用的技巧，就是如果是枚举所有约数的某个性质的和，可以借鉴约数之和公式。
$$
(1+p_1^1+p_1^2+\cdots+p_1^{c_1})(1+p_2^1+p_2^2+\cdots+p_2^{c_2})\cdots
$$
映射到下面：
$$
[1+(1-\frac{1}{p_1})+(1-\frac{1}{p_1})+\cdots+(1-\frac{1}{p_1})]\cdots\\
=\prod_{i=1}^k [1+c_i(1-\frac{1}{p_i})]
$$
所以原式等于：
$$
n\prod_{i=1}^k[1+c_i(1-\frac{1}{p_i})]
$$

```cpp
#include <cstdio>
using namespace std;

typedef long long LL;

int main() {
    LL n;
    scanf("%lld", &n);
    LL res = n;
    
    for (int i = 2; i <= n/i; i++) {
        if (n % i == 0) {
            int c = 0, p = i;
            while (n % i == 0) n /= i, c++;
            // 先除再乘防溢出
            res /= p;
            res = res * (p + c*(p-1));
        }
    }
    if (n > 1) {
        res /= n;
        res = res * (n + n - 1);
    }
    printf("%lld\n", res);
    return 0;
}
```

