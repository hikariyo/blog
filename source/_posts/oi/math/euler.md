---
date: 2023-07-29 20:24:00
updated: 2024-06-07 17:08:00
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

对于正整数 $n$，欧拉函数 $\varphi(n)$ 代表小于 $n$ 的数中，所有与 $n$ 互质的数的个数。一般认为 $\varphi(1) = 1$。

### 性质1

对于素数 $p$，$1\sim (p-1)$ 都与其互质，所以 $\varphi(p)=p-1$。

### 性质2

对于互质的 $m,n$ 有 $\varphi(mn)=\varphi(m)\varphi(n)$，即欧拉函数为非完全积性函数。

这个证明需要剩余系的复合，我不太会，可以参考 OI-WIKI。

### 性质3

对于素数 $p$，有 $\varphi(p^k)=p^k-p^{k-1}$。

证明：小于 $p^k$​ 的所有数字中，只有 $1p,2p,\dots,(p^{k-1}-1)p$​ 共 $p^{k-1}-1$​ 个数与 $p^k$​ 不互质，一共有 $p^{k}-1$​ 个数，故 $\varphi(p^k)=p^{k}-1-(p^{k-1}-1)=p^k-p^{k-1}$​。

### 性质4

对于任意正整数 $n$，将其质因数分解为 $n=\prod_{i=1}^k p_i^{c_i}$，有：
$$
\begin{aligned}
\varphi(n)&=\varphi\left(\prod_{i=1}^k p_i^{c_i}\right)\\
&=\prod_{i=1}^k \varphi(p_i^{c_i})\\
&=\prod_{i=1}^k p_i^{c_i}(1-\frac{1}{p_i})\\
&=n\prod_{i=1}^k (1-\frac{1}{p_i})
\end{aligned}
$$
这是也是欧拉函数的一种求法。

### 性质5

对于欧拉函数，有下面这个常见的式子：
$$
\sum_{d|n}\varphi(d)=n
$$
证明：设 $f(n)=\sum_{d|n}\varphi(d)$，可以证明 $f(n)$ 为积性函数，设 $n,m$ 互质则显然 $n,m$ 的任何一个因子 $i,j$ 也是互质的：
$$
\begin{aligned}
f(n)f(m)&=\sum_{i|n}\sum_{j|m}\varphi(i)\varphi(j)\\
&=\sum_{i|n}\sum_{j|m}\varphi(ij)\\
&=\sum_{d|nm}\varphi(d)\\
&=f(nm)
\end{aligned}
$$
对 $n$ 进行质因数分解，所以 $f(n)=f(p^{c_1})f(p^{c_2})\dots f(p^{c_k})$，又有：
$$
\begin{aligned}
f(p^c)&=\sum_{i=0}^c \varphi(p^i)\\
&=1+\sum_{i=1}^c (p^i-p^{i-1})\\
&=p^c
\end{aligned}
$$
因此 $f(n)=n$​，得证。

### 分解质因数求值

复杂度为 $O(\sqrt n\log n)$，其中 $\log n$ 是指质因数分解后的 $n$ 的所有质数的指数之和，实际肯定是远小于 $\log n$ 的。

```cpp
int phi(int n) {
    int res = n;
    // O(sqrt(n)) 分解质因数
    for (int i = 2; i <= n/i; i++) {
        // 把当前因数除干净, 这样就保证一定是质因数了, 上界是 O(log n)
        if (n % i == 0) {
            res = res - res/i;
            while (n % i == 0) n /= i;
        }
    }
    // 最后可能没有除干净，这里特判一下
    if (n > 1) res = res-res/n;
    return res;
}
```

这里你可能会对 $i^2\le n$ 这个循环条件存在疑问，因为 $n$ 随着循环是在变化的。首先 $i^2\le n$ 等价于 $i\le n/i$，然后 $i$ 是整数，所以等价于 $i\le \lfloor n/i\rfloor$。

这里设当前 $n=p_ip_j$，其中 $p_i\le p_j$，那么因为还没有除掉 $p_i$，所以 $i\le p_i$ 是成立的，因此 $i^2\le p_ip_j=n$；所以只要 $n$ 还有多于两个未除掉的质因数，就不会结束循环。

### 线性筛法求值

复杂度同线性筛，为 $O(n)$。

```cpp
int phi[N], primes[N], cntp;
bitset<N> st;

void init(int n) {
    phi[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cntp++] = i, phi[i] = i-1;
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i*primes[j]] = 1;
            if (i % primes[j]) phi[i*primes[j]] = phi[i] * phi[primes[j]];
            else {
                phi[i*primes[j]] = phi[i] * primes[j];
                break;
            }
        }
    }
}
```

可以证明当要求 $i\cdot p_j$ 的时候，$\varphi(i)$ 一定已经求出来了：

1. 如果 $i$ 是质数，那么它会在进入内层 `for` 循环前被求出来。
2. 如果 $i$ 是合数，那么它会在 $i$ 循环到自己之前就被筛掉，筛掉的同时也就求出来了 $\varphi(i)$。

接下来说明递推式：

1. 如果 $i$ 是质数，那么在 $1 \sim i-1$ 中所有数都与它互质，所以有 $\varphi(i)=i-1$。
2. 如果 $i \bmod  p_j=0$，此时 $p_j$ 是 $i$ 的最小质因数，由欧拉函数的解析式知，$\varphi(p_j\times i)=p_j\varphi(i)$。
3. 如果 $i\bmod p_j\ne 0$，此时 $p_j$ 不是 $i$ 的质因数，那么 $\varphi(p_j\cdot i)=\varphi(p_j)\varphi(i)$。

## 欧拉定理

欧拉定理：若 $a$ 与 $x$ 互质，则 $a^{\varphi(x)}\equiv 1 \pmod x$。

证明：设所有小于 $x$ 的数中所有与 $x$ 互质的数是 $x_1, x_2, \cdots, x_{\varphi(x)}$，那么 $ax_1, ax_2, \cdots, ax_{\varphi(x)}$ 也与 $x$ 互质（无论是 $a$ 还是 $x_i$ 都与 $x$ 没有相同的质因数，所以 $ax_i$ 也与 $x$ 没有相同的质因数）。

并且 $ax_1, ax_2, \cdots, ax_{\varphi(x)}$ 对 $x$ 取模两两不相同，这步用反证法证明，假设 $ax_i\equiv ax_j\pmod x$ 由于 $\gcd(a,x)=1$ 有 $x_i\equiv x_j\pmod x$ 这是矛盾的。

于是这两组数模 $x$ 之后其实是同一组数据，那么它们取模之后全部乘起来结果是相同的。
$$
\begin{aligned}
\prod_{i=1}^{\varphi(x)}(ax_i)&\equiv\prod_{i=1}^{\varphi(x)}x_i\pmod x\\
a^{\varphi(x)}\prod_{i=1}^{\varphi(x)}x_i&\equiv \prod_{i=1}^{\varphi(x)}x_i \pmod x\\
\end{aligned}
$$
由于任意一个 $x_i$ 都与 $x$ 互质，所以它们的乘积也与 $x$ 互质，所以 $a^{\varphi(x)}\equiv 1\pmod x$。

特别地，如果设 $p$ 是质数，那么 $a^{\varphi(p)}\equiv a^{p-1} \equiv 1\pmod p$，欧拉定理是费马小定理的拓展。

## 拓展欧拉定理

> 给定 $a,m,b$ 求 $a^b \bmod m$，其中 $1\le a\le 10^9,1\le b\le10^{20000000},1\le m\le 10^8$。

不要求 $a,m$ 互质，且 $b\ge \varphi(m)$ 时满足 $a^b\equiv a^{b\bmod  \varphi(m)+\varphi(m)}\pmod m$，这也是降幂公式。

### 引理1

如果 $\gcd(m_1,m_2)=1$ 时， $x\equiv y\pmod {m_1},x\equiv y\pmod{m_2}$ 那么一定有 $x-y\equiv 0\pmod {m_1m_2}$。

证明：$x-y$ 是 $m_1,m_2$ 的公倍数，所以 $x-y\equiv 0\pmod {m_1m_2}$

### 引理2

若 $p$ 为质数，那么 $\varphi(p^k)=p^k(1-\frac{1}{p})=p^{k-1}(p-1)$，有 $\varphi(p^k)\ge k$

证明：如果固定 $k$，那么 $\varphi(p^k)$ 是对 $p$ 单调递增的，因为是两个单调递增的恒正的函数相乘。

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

### 定理

首先，$\gcd(a,m)=1$ 时，有 $a^{\varphi(m)}\equiv 1\pmod m$，那么 $a^{b}\equiv a^{b\bmod\varphi(m)+\varphi(m)}\pmod m$ 显然成立。主要证明 $\gcd(a,m)\ne 1$ 时的情况。

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

综上，$a^b\equiv a^{b\bmod  \varphi(m)+\varphi(m)}\pmod{m},b\ge \varphi(m)$​，这也被称作降幂公式。

### 代码

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

## 幂塔

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

## [SDOI2008] 仪仗队

> 给定正整数 $1\le n\le40000$，求出 $(0,0)$ 到 $(x,y)$ 的所有点中直接相连不经过其它点的所有点的数量。
>
> 题目链接：[P2158](https://www.luogu.com.cn/problem/P2158)。

不经过其它点，除了 $(0,1),(1,0)$ 外，其它点都是 $(x,y)$ 且满足 $1\le x,y\le n-1$，所以先 $n\gets n-1$，然后考虑何为“不经过其它点”。

若 $(0,0)$ 到 $(x_1,y_1)$ 经过 $(x_0,y_0)$，其中 $x_0<x_1,y_0<y_1$。那么这个连线，$Ox$ 轴，$x=x_1,x=x_2$ 这四条直线构成的两个三角形存在相似关系，也就是有 $y_1/x_1=y_0/x_0$，也就是说 $\gcd(x_1,y_1) \ne 1$。

当 $\gcd(x_1,y_1)=1$ 时，则该点满足条件（这是原命题“若 $(x_1,y_1)$ 不满足条件，则 $\gcd(x_1,y_1)\ne 1$”的逆否命题）。

所以答案就会是（别忘了 $(0,1),(1,0)$ 这两个点）：
$$
2+\sum_{i=1}^n\sum_{j=1}^n[\gcd(i,j)=1]
$$
原式具有对称性，继续化简：
$$
2+2\sum_{i=1}^n\sum_{j\le i}[\gcd(i,j)=1]-\sum_{i=1}^n[\gcd(i,i)=1]=1+2\sum_{i=1}^n\varphi(i)\\
$$
所以做一个线性筛就行了；其实暴力 $O(n^2\log n)$ 在这个范围同样可以通过。注意特判 $n=1$，上面的情况都是基于 $n\ge 2$ 时的。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 40010;
int primes[N], phi[N], cntp;
bitset<N> st;

void init(int n) {
    phi[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cntp++] = i, phi[i] = i - 1;
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i * primes[j]] = 1;
            if (i % primes[j]) phi[i * primes[j]] = phi[i] * phi[primes[j]];
            else {
                phi[i * primes[j]] = phi[i] * primes[j];
                break;
            }
        }
    }
}

int main() {
    int n, res = 1;
    cin >> n;
    if (n == 1) {
        cout << 0 << endl;
        return 0;
    }
    n--;
    init(n);
    for (int i = 1; i <= n; i++) res += 2 * phi[i];
    cout << res << endl;
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

本题主要是 $\varphi(d)\mid d$ 引出的性质。首先：
$$
\varphi(d)=d(1-\frac{1}{2})(1-\frac{1}{3})\cdots=d\frac{a}{b}\\
\varphi(d)\mid d\Leftrightarrow kd\frac{a}{b}=d\\
b=ka
$$
也就是要求 $\varphi(d)=d/k$，要么 $\varphi(d)=d/2$ 要么 $\varphi(d)=d/3$，所以 $d$ 只能有 $2,3$ 两个约数。

将每个 $a_i$ 分解为 $2^a3^b$，设 $d=2^p3^q$，也就是需要满足 $p=\sum a,q=\sum b$ 的情况下保证花费最小，那么这显然是个背包问题。

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

## [蓝桥杯2018国B] 矩阵求和

> 给定正整数 $n$，求：
> $$
> \sum_{i=1}^n\sum_{j=1}^n \Big(\gcd(i,j)\Big)^2
> $$
> 其中 $1\le n\le 10^7$，答案对 $10^9+7$​ 取模。
>
> 题目链接：[P8670](https://www.luogu.com.cn/problem/P8670)。

由于 $\gcd$ 具有交换律，所以原式具有对称性，所以可以引入限制条件，并减掉重复的项，原式化为：
$$
2\sum_{i=1}^n\sum_{j\le i}\Big(\gcd(i,j)\Big)^2-\sum_{i=1}^ni^2
$$
设 $S(n)=\sum_{i=1}^n i^2$，众所周知 $S(n)=\frac{n(n+1)(2n+1)}{6}$，设 $d=\gcd(i,j)$ 必然有 $d\mid i,d\mid j$，所以 $\gcd(i/d,j/d)=1$，由于 $j\le i$ 所以 $j/d\le i/d$，原式可化为：
$$
2\sum_{i=1}^n\sum_{d\mid i}d^2\varphi(\frac{i}{d})-S(n)=2\sum_{i=1}^n\sum_{d\mid i}(\frac{i}{d})^2\varphi(d)-S(n)
$$
注意 $d,i/d$ 同样具有对称性，然后考虑 $\varphi(d)$ 对谁造成了贡献，考虑只有当 $i=d,2d,\dots,\lfloor \frac{n}{d}\rfloor d$ 时才会造成贡献，所以 $\varphi(d)$ 的贡献就是 $S(\lfloor\frac{n}{d}\rfloor)$。原式等价于：
$$
2\sum_{d=1}^n\varphi(d)S(\lfloor\frac{n}{d}\rfloor)-S(n)
$$
线性筛 $O(n)$ 足够通过本题，如果要求更高可以整除分块 + 高级筛法，这里就不拓展了，因为我也不会。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 10000010, P = 1e9+7;
int phi[N], primes[N], cntp;
bitset<N> st;

void init(int n) {
    phi[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cntp++] = i, phi[i] = i-1;
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i*primes[j]] = 1;
            if (i % primes[j]) phi[i*primes[j]] = phi[i] * phi[primes[j]];
            else {
                phi[i*primes[j]] = phi[i] * primes[j];
                break;
            }
        }
    }
}

int S(int n) {
    return 1ll * n * (n+1) % P * (2 * n + 1) % P * 166666668ll % P;
}

int Solve(int n) {
    int res = 0;
    init(n);
    for (int i = 1; i <= n; i++) res = (res + 1ll * S(n / i) * phi[i]) % P;
    res = (((2ll * res - S(n)) % P) + P) % P;
    return res;
}

int main() {
    int n;
    cin >> n;
    cout << Solve(n) << endl;
    return 0;
}
```

## [XJTUPC2024] 筛法

> 给定正整数 $n$，求：
> $$
> \sum_{i=1}^n\sum_{j=1}^n \lfloor\frac{n}{\max(i,j)}\rfloor[\gcd(i,j)=1]
> $$
> 其中  $1\le n\le 10^9$​。
>
> 题目链接：[P10532](https://www.luogu.com.cn/problem/P10532)。

首先 $i,j$ 是对称的，所以：
$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^n\lfloor\frac{n}{\max(i,j)}\rfloor[\gcd(i,j)=1]&=2\sum_{i=1}^n\sum_{j=1}^i\lfloor\frac{n}{i}\rfloor[\gcd(i,j)=1]-\sum_{i=1}^n \lfloor\frac{n}{i}\rfloor[i=1]\\
&=2\sum_{i=1}^n\lfloor\frac{n}{i}\rfloor\varphi(i)-n\\
\end{aligned}
$$
观察：$\lfloor n/i\rfloor$ 是 $i$ 的所有倍数中 $\le n$ 的个数，所以可以枚举 $d$ 为其所有合法倍数的因数，使 $\varphi(d)$ 贡献到答案，故原式等价于：
$$
\begin{aligned}
2\sum_{i=1}^n\sum_{d\mid i} \varphi(d)-n&=2\sum_{i=1}^n i-n\\
&=n(n+1)-n\\
&=n^2
\end{aligned}
$$
所以代码就不写了。

## [SDOI2012] Longge 的问题

> 给定正整数 $n$，求：
> $$
> \sum_{i=1}^n \gcd(i,n)
> $$
> 其中 $1\le n\lt 2^{32}$​。
>
> 题目链接：[P2303](https://www.luogu.com.cn/problem/P2303)。

还是枚举 $d=\gcd(i,n)$，则有 $d\mid n$，且 $d$ 出现的次数为满足 $\gcd(i/d,n/d)=1$ 的 $i$ 的个数，就是 $\varphi(\frac{n}{d})$，所以要求的值是：
$$
\sum_{d\mid n} d\varphi(\frac{n}{d})=\sum_{d\mid n}\frac{n}{d}\varphi(d)
$$
对 $n$ 质因数分解，以质因数分解的形式枚举所有 $d\mid n$，这个是 $O(\sqrt n)$ 的，然后对每个 $d$ 用解析式求 $\varphi(d)$，这个大概是 $O(\log n)$ 的，所以可以通过本题。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int K = 1 << 20;
// factor, exponent, dfs exponent
int n, fac[K], ep[K], dep[K], k, ans;

void dfs(int u, int now) {
    if (u > k) {
        int phi = now;
        for (int i = 1; i <= k; i++) if (dep[i]) phi -= phi / fac[i];
        ans += phi * n / now;
        return;
    }
    for (dep[u] = 0; dep[u] <= ep[u]; dep[u]++, now = now * fac[u]) dfs(u+1, now);
}

signed main() {
    cin >> n;
    int n_copy = n;
    for (int i = 2; i <= n_copy / i; i++) {
        if (n_copy % i == 0) {
            int cnt = 0;
            while (n_copy % i == 0) n_copy /= i, cnt++;
            ++k, fac[k] = i, ep[k] = cnt;
        }
    }
    if (n_copy > 1) ++k, fac[k] = n_copy, ep[k] = 1;
    dfs(1, 1);
    cout << ans << endl;
    return 0;
}
```

## [湖北省队互测] GCD

> 给定正整数 $n$，求 $1\le x,y\le n$ 且 $\gcd(x,y)$ 为素数的 $(x,y)$ 有多少对，其中 $\le n\le 10^7$。
>
> 题目链接：[BZOJ2818](https://darkbzoj.cc/problem/2818)。

枚举不超过 $n$ 的素数，然后为了书写方便令 $N=\lfloor n/p\rfloor$：
$$
\begin{aligned}
\sum_p\sum_{i=1}^n\sum_{j=1}^n[\gcd(i,j)=p]&=\sum_{p}\sum_{i=1}^n\sum_{j=1}^n[p\mid i][p\mid j][\gcd(\frac{i}{p},\frac{j}{p})=1]\\
&=\sum_p\sum_{i=1}^{\lfloor\frac{n}{p}\rfloor}\sum_{j=1}^{\lfloor\frac{n}{p}\rfloor}[\gcd(i,j)=1]\\
&=\sum_p\left(2\sum_{i=1}^N\sum_{j=1}^i[\gcd(i,j)=1]-\sum_{i=1}^N[\gcd(i,i)=1]\right)\\
&=\sum_p\left(2\sum_{i=1}^N\varphi(i)-1\right)\\
\end{aligned}
$$
线性筛 $O(n)$ 预处理 $\varphi(i)$ 的前缀和与不超过 $n$ 的素数，所以总复杂度为 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 10000010;
int n, primes[N], cntp;
long long ans, phi[N];
bitset<N> st;

void init(int n) {
    phi[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cntp++] = i, phi[i] = i-1;
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i * primes[j]] = 1;
            if (i % primes[j]) phi[i * primes[j]] = phi[i] * phi[primes[j]];
            else {
                phi[i * primes[j]] = phi[i] * primes[j];
                break;
            }
        }
    }
    for (int i = 1; i <= n; i++) phi[i] += phi[i-1];
}

int main() {
    cin >> n;
    init(n);
    for (int i = 0; i < cntp; i++) ans += 2ll * phi[n / primes[i]] - 1;
    cout << ans << endl;
    return 0;
}
```

