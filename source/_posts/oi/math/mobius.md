---
date: 2023-08-05 16:59:00
title: 数学 - 莫比乌斯反演
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 莫比乌斯函数

首先给出莫比乌斯函数 $\mu(n)$ 的定义：
$$
n=\prod_{i=1}^k p_i^{c_i}\\
\mu(n)=\left \{
\begin{aligned}
&0,\exist c_i\ge 2\\
&(-1)^k,\forall c_i<2\\
\end{aligned}
\right.
$$
特别地，$n=1$ 时看作 $k=0$ 因此 $\mu(1)=1$。

### 性质 1

莫比乌斯函数是积性函数，设 $n,m$ 互质，它们的质因数分解形式为 $\prod_{i=1}^k p_i^{c_i}$ 与 $\prod_{j=1}^K P_j^{C_j}$，那么：
$$
\mu(nm)=\mu\left(\prod _{i=1}^k p_i^{c_i}\times \prod_{j=1}^KP_j^{C_j}\right)
$$
由于互质，所以对于任意 $p_i,P_j$ 都是互不相同的，因此：
$$
\mu\left(\prod _{i=1}^k p_i^{c_i}\times \prod_{j=1}^KP_j^{C_j}\right)=\begin{cases}(-1)^{k+K}=\mu(n)\mu(m)\quad \forall c_i,C_j<2\\
0=\mu(n)\mu(m)\quad \exist c_iC_j\ge 2
\end{cases}
$$
综上所述，$\mu(nm)=\mu(n)\mu(m)$，这可用于线性筛预处理。

### 性质 2

下面证明 $\sum_{d\mid n}\mu(d)=[n=1]$：

当 $n=1$ 时，结果显然成立，下面证明 $n>1$ 时的情况，首先分解质因子：
$$
n=\prod_{i=1}^{k}p_i^{c_i},d=\prod_{i=1}^{k}p_i^{b_i},\forall b_i\le c_i
$$
当任意 $b_i>1$ 时，$\mu(d)=0$，所以不用考虑这种情况，因此就是从 $k$ 个质因子中选几个进行求和的问题了。
$$
\begin{aligned}
\sum_{d\mid n}\mu(d)&=\binom{k}{0}(-1)^0+\binom{k}{1}(-1)^1+\cdots+\binom{k}{k}(-1)^k\\
&=(-1+1)^k\\
&=0
\end{aligned}
$$
基本上莫比乌斯反演的题目都是可以用 $[n=1]$ 代换成莫比乌斯函数的求和，然后继续做下去的。

## 一个取整公式

下面这个公式是成立的，并且是常用的。
$$
\lfloor\frac{a}{bc}\rfloor = \lfloor\frac{\lfloor a/b\rfloor }{c}\rfloor
$$
证明：令 $a=kb+r,k=qc+p$，其中 $r,p$ 分别是带余除法的余数，则有 $r\in[0,b-1],p\in[0,c-1]$：
$$
\begin{aligned}
a&=kb+r\\
&=qbc+pb+r\\
&\le qbc+b(c-1)+b-1\\
&=qbc+bc-1
\end{aligned}
$$
根据定义，$\lfloor a/b\rfloor=k,\lfloor k/c\rfloor=q$，接下来求 $\lfloor a/(bc)\rfloor$ 的范围：
$$
\begin{aligned}
\frac{a}{bc}&=\frac{qbc+pb+r}{bc}\\
&=q+\frac{pb+r}{bc}\\
&\ge q\\
\frac{a}{bc}&\le \frac{bc(q+1)-1}{bc}\\
&= q+1-\frac{1}{bc}\\
&\lt q+1\\
\end{aligned}
$$
所以 $\lfloor a/(bc)\rfloor = q$​，[原文链接](https://math.stackexchange.com/questions/2557458/is-a-b-c-equal-to-a-bc-for-integer-division)在这里。

## YY 的 GCD

> 给定 $T=10^4$ 组询问，每组给正整数 $n,m$ 求 $1\le x\le n,1\le y\le m$ 且 $\gcd(x,y)$ 为质数的 $(x,y)$ 有多少对，其中 $1\le n,m\le 10^7$。
>
> 题目链接：[P2257](https://www.luogu.com.cn/problem/P2257)。

题目对于 $x,y$ 具有对称性，所以不妨设 $n\ge m$，首先写出要求的表达式：
$$
\begin{aligned}
\sum_{i=1}^{n}\sum_{j=1}^m[\gcd(i,j)\in \texttt{Primes}]&=\sum_{p\in \texttt{Primes}}\sum_{i=1}^n\sum_{j=1}^m[\gcd(i,j)=p]\\
&=\sum_{p}\sum_{i=1}^{\lfloor\frac{n}{p}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{p}\rfloor}[\gcd(i,j)=1]\\
\end{aligned}
$$
这里用到 $\sum_{d\mid n}\mu(d)=[n=1]$，继续化简：
$$
\begin{aligned}
\sum_{p}\sum_{i=1}^{\lfloor\frac{n}{p}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{p}\rfloor}\sum_{d\mid \gcd(i,j)}\mu(d)&=\sum_p\sum_{d=1}^{\lfloor\frac{n}{p}\rfloor}\sum_{i=1}^{\lfloor\frac{n}{p}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{p}\rfloor}[d\mid i][d\mid j]\mu(d)\\
&=\sum_p\sum_{d=1}^{\lfloor\frac{n}{p}\rfloor}\mu(d)\lfloor\frac{n}{pd}\rfloor\lfloor\frac{m}{pd}\rfloor
\end{aligned}
$$
发现这样做是无法通过本题的，目前复杂度为 $O(T\sum_p \lfloor n/p\rfloor)=O(Tn\ln\ln n)$，然后考虑怎么样对某些式子进行预处理。

要想把关于 $n,m$ 的式子提出来，只有枚举 $pd$ 的取值可以做到，令 $k=pd$，枚举 $k$，得到：
$$
\sum_{k=1}^m\lfloor\frac{n}{k}\rfloor\lfloor\frac{m}{k}\rfloor\sum_{p\mid k}\mu(\frac{k}{p})
$$
令 $f(k)=\sum_{p\mid k}\mu(\frac{k}{p})$，发现 $f(k)$ 与 $n,m$ 无关，可以预处理，具体方法是枚举素数 $p$ 的倍数，到 $n$ 为止，这样复杂度应当是 $O(\sum_p \lfloor n/p\rfloor)=O(n\ln \ln n)$，可以接受。线性筛是 $O(n)$，剩下就是一个整除分块了，这样总复杂度为 $O(n\ln\ln n+T\sqrt n)$，可以通过本题。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 10000010;
int T, n, m, primes[N], mu[N], f[N], cntp;
bitset<N> st;

void init(int n) {
    mu[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cntp++] = i, mu[i] = -1;
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i * primes[j]] = 1;
            if (i % primes[j]) mu[i * primes[j]] = -mu[i];
            else {
                mu[i * primes[j]] = 0;
                break;
            }
        }
    }

    for (int i = 0; i < cntp; i++)
        for (int j = 1; j <= n / primes[i]; j++)
            f[j * primes[i]] += mu[j];

    for (int i = 1; i <= n; i++) f[i] += f[i-1];
}

signed main() {
    init(N-1);
    cin >> T;
    while (T--) {
        cin >> n >> m;
        if (n < m) swap(n, m);
        int k = 1, res = 0;
        while (k <= m) {
            int nxt = min(n / (n / k), m / (m / k));
            if (nxt <= m) res += 1ll * (n / k) * (m / k) * (f[nxt] - f[k-1]);
            else res += 1ll * (n / k) * (m / k) * (f[m] - f[k-1]);
            k = nxt + 1;
        }
        cout << res << endl;
    }
    return 0;
}
```

## [HAOI2011] Problem b

> 对于给出的 $n$ 个询问，每次求有多少个数对$(x,y)$，满足 $a≤x≤b$，$c≤y≤d$，且 $\gcd(x,y)=k$​。
>
> 其中 $1\le n,k\le 5\times 10^4$
>
> 题目链接：[AcWing 2702](https://www.acwing.com/problem/content/2704/)，[P2522](https://www.luogu.com.cn/problem/P2522)。

首先用前缀和的思想，求 $(1,1)$ 到 $(n,m)$ 的数量，设 $n\le m$：

$$
\begin{aligned}
S(n,m)&=\sum_{i=1}^n\sum_{j=1}^{m}[\gcd(i,j)=k]\\
&=\sum_{i=1}^{\lfloor\frac{n}{k}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{k}\rfloor}[\gcd(i,j)=1]\\
&=\sum_{i=1}^{\lfloor\frac{n}{k}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{k}\rfloor}\sum_{d\mid \gcd(i,j)}\mu(d)\\
&=\sum_{d=1}^{\lfloor\frac{n}{k}\rfloor}\mu(d)\sum_{i=1}^{\lfloor\frac{n}{k}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{k}\rfloor}[d\mid i][d\mid j]\\
&=\sum_{d=1}^{\lfloor\frac{n}{k}\rfloor}\mu(d)\lfloor\frac{n}{kd}\rfloor\lfloor\frac{m}{kd}\rfloor
\end{aligned}
$$
所以答案为 $S(b,d)-S(b,c-1)-S(a-1,d)+S(a-1,c-1)$，对于求 $S$ 用数论分块可以做到 $O(\sqrt n)$，这样总复杂度为 $O(n\sqrt n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 50010;
int T, a, b, c, d, k, mu[N], primes[N], cntp;
bitset<N> st;

void init(int n) {
    mu[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cntp++] = i, mu[i] = -1;
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i * primes[j]] = 1;
            if (i % primes[j]) mu[i * primes[j]] = -mu[i];
            else {
                mu[i * primes[j]] = 0;
                break;
            }
        }
    }

    for (int i = 1; i <= n; i++) mu[i] += mu[i-1];
}

int S(int n, int m) {
    n /= k, m /= k;
    if (n > m) swap(n, m);
    int d = 1, res = 0;
    while (d <= n) {
        int nxt = min(n / (n / d), m / (m / d));
        if (nxt <= n) res += (mu[nxt] - mu[d-1]) * (n / d) * (m / d);
        else res += (mu[n] - mu[d-1]) * (n / d) * (m / d);
        d = nxt + 1;
    }
    return res;
}

int main() {
    init(N-1);
    cin >> T;
    while (T--) {
        cin >> a >> b >> c >> d >> k;
        cout << S(b, d) - S(b, c-1) - S(a-1, d) + S(a-1, c-1) << endl;
    }
    return 0;
}
```

## [SDOI2014] 数表

> 对于 $n\times m$ 的数表，位置 $(i,j)$ 值是同时整除 $i,j$ 的所有自然数之和。给定 $q$ 组询问，每组询问给出 $n,m,a$ 问 $n\times m$ 的数表中所有不大于 $a$ 的数字之和。其中 $1\le n,m\le 10^5,1\le q\le 2\times 10^4$​。
>
> 题目链接：[P3312](https://www.luogu.com.cn/problem/P3312)。

对于表格中 $(i,j)$ 的值是 $V_{ij}=\sum_{d\mid \gcd(i,j)}d=\sigma\big(\gcd(i,j)\big)$，其中 $\sigma(x)$ 表示 $x$ 的约数之和。

式子对于 $i,j$ 对称，不妨设 $n\ge m$，求的是：
$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^m \sigma\Big(\gcd(i,j)\Big)&=\sum_{d=1}^m\sigma(d)\sum_{i=1}^n\sum_{j=1}^m[\gcd(i,j)=d]\\
&=\sum_{d=1}^m\sigma(d)\sum_{i=1}^n\sum_{j=1}^m[\gcd(\frac{i}{d},\frac{j}{d})=1]\\
&=\sum_{d=1}^m\sigma(d)\sum_{i=1}^{\lfloor\frac{n}{d}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{d}\rfloor}[\gcd(i,j)=1]\\
&=\sum_{d=1}^m\sigma(d)\sum_{i=1}^{\lfloor\frac{n}{d}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{d}\rfloor}\sum_{x\mid \gcd(i,j)}\mu(x)\\
&=\sum_{d=1}^m\sigma(d)\sum_{x=1}^{\lfloor\frac{m}{d}\rfloor}\sum_{i=1}^{\lfloor\frac{n}{d}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{d}\rfloor}[x\mid i][x\mid j]\mu(x)\\
&=\sum_{d=1}^m\sigma(d)\sum_{x=1}^{\lfloor\frac{m}{d}\rfloor}\lfloor\frac{n}{xd}\rfloor\lfloor\frac{m}{xd}\rfloor\mu(x)\\
&=\sum_{t=1}^m\lfloor\frac{n}{t}\rfloor\lfloor\frac{m}{t}\rfloor\sum_{d\mid t}\sigma(d)\mu(\frac{t}{d})
\end{aligned}
$$
令 $F(t)=\sum_{d\mid t}\sigma(d)\mu(\frac{t}{d})$，对于这个函数可以进行预处理，并且复杂度为 $O(\sum_{d}\lfloor n/d\rfloor)=O(n\log n)$。对于 $a$，如果 $a$ 是单调递增的，那么每次对于某些 $F(t)$ 值进行增加，所以可以离线下来排序并且用树状数组维护 $F(t)$ 的值。

这样预处理线性筛 + 约数之和的复杂度为 $O(n\log n)$，修改 $F(t)$ 值的总复杂度为 $O(n\log^2n)$，回答询问的复杂度是 $O(q\sqrt n\log n)$，并且树状数组的常数比较小，可以通过本题。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define fi first
#define se second
#define lowbit(x) ((x)&(-x))
#define int long long
typedef pair<int, int> PII;
const int N = 100010, Q = 20010, P = 1ll << 31;
int q, ans[Q], sigma[N], primes[N], mu[N], cntp;
PII s1[N];
bitset<N> st;

struct BIT {
    int tr[N];

    void add(int p, int k) {
        for (; p < N; p += lowbit(p)) {
            tr[p] = (tr[p] + k) % P;
            if (tr[p] < 0) tr[p] += P;
        }
    }

    int query(int p) {
        int res = 0;
        for (; p; p -= lowbit(p)) res = (res + tr[p]) % P;
        return res;
    }

    int query(int l, int r) {
        int res = (query(r) - query(l-1)) % P;
        if (res < 0) res += P;
        return res;
    }
} F;

struct Query {
    int id, n, m, a;
    bool operator<(const Query& qry) const {
        return a < qry.a;
    }
} qry[Q];

void init(int n) {
    mu[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cntp++] = i, mu[i] = -1;
        for (int j = 0; primes[j] <= n / i; j++) {
            st[i * primes[j]] = 1;
            if (i % primes[j]) mu[i * primes[j]] = -mu[i];
            else {
                mu[i * primes[j]] = 0;
                break;
            }
        }
    }

    for (int i = 1; i <= n; i++)
        for (int j = i; j <= n; j += i)
            sigma[j] += i;
    for (int i = 1; i <= n; i++) s1[i] = {sigma[i], i};
    sort(s1+1, s1+1+n);
}

signed main() {
    init(N-1);
    cin >> q;
    for (int i = 1, n, m, a; i <= q; i++) {
        cin >> n >> m >> a;
        if (n < m) swap(n, m);
        qry[i] = {i, n, m, a};
    }
    sort(qry+1, qry+1+q);
    for (int i = 1, j = 1; i <= q; i++) {
        while (j < N && s1[j].fi <= qry[i].a) {
            int d = s1[j].se;
            for (int t = d; t <= N-1; t += d) F.add(t, sigma[d] * mu[t / d]);
            j++;
        }

        int res = 0, t = 1, n = qry[i].n, m = qry[i].m;
        while (t <= m) {
            int nxt = min(n / (n / t), m / (m / t));
            if (nxt <= m) res = (res + 1ll * (m / t) * (n / t) * F.query(t, nxt)) % P;
            else res = (res + 1ll * (m / t) * (n / t) * F.query(t, m)) % P;
            t = nxt + 1;
        }
        ans[qry[i].id] = res;
    }

    for (int i = 1; i <= q; i++) cout << ans[i] << endl;
    return 0;
}
```

## [SDOI2017] 数字表格

> 给定 $T$ 组询问，每组询问给出 $n,m$ 问：
> $$
> \prod_{i=1}^n\prod_{j=1}^m f\Big(\gcd(i,j)\Big) \bmod (10^9+7)
> $$
> 其中 $f(n)$ 表示斐波那契数列的第 $n$ 项，且 $1\le T\le 10^3,1\le n,m\le 10^6$。
>
> 题目链接：[P3704](https://www.luogu.com.cn/problem/P3704)。

其实这几个题都比较类似，不妨设 $n\ge m$。
$$
\begin{aligned}
\prod_{i=1}^n\prod_{j=1}^m f\Big(\gcd(i,j)\Big)&=\prod_{d=1}^m\prod_{i=1}^n\prod_{j=1}^mf(d)[\gcd(i,j)=d]\\
&=\prod_{d=1}^m\Big(f(d)\Big)^{\sum_{i=1}^{\lfloor n/d\rfloor}\sum_{j=1}^{\lfloor m/d\rfloor}[\gcd(i,j)=1]}\\
&=\prod_{d=1}^m\Big(f(d)\Big)^{\sum_{x=1}^{\lfloor m/d\rfloor}\lfloor\frac{n}{xd}\rfloor\lfloor\frac{m}{xd}\rfloor\mu(x)}\\
&=\prod_{t=1}^m\prod_{d\mid t}\Big(f(d)\Big)^{\lfloor\frac{n}{t}\rfloor\lfloor\frac{m}{t}\rfloor\mu(\frac{t}{d})}\\
&=\prod_{t=1}^m\left(\prod_{d\mid t}\Big(f(d)\Big)^{\mu(\frac{t}{d})}\right)^{\lfloor\frac{m}{t}\rfloor\lfloor\frac{n}{t}\rfloor}\\
&=\prod_{t=1}^m\Big(F(t)\Big)^{\lfloor\frac{n}{t}\rfloor\lfloor\frac{m}{t}\rfloor}
\end{aligned}
$$
其中 $F(t)$ 可以 $O(n\log n)$ 预处理，整除分块预处理 $F(t)$ 的前缀积即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 1000010, P = 1e9+7;
int n, m, T;
int F[N], f[N], mu[N], primes[N], cntp;
bitset<N> st;

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

void init(int n) {
    mu[1] = 1, f[1] = 1;
    for (int i = 2; i <= n; i++) {
        f[i] = (f[i-1] + f[i-2]) % P;
        if (!st[i]) primes[cntp++] = i, mu[i] = -1;
        for (int j = 0; primes[j] <= n / i; j++) {
            st[i * primes[j]] = 1;
            if (i % primes[j]) mu[i * primes[j]] = -mu[i];
            else {
                mu[i * primes[j]] = 0;
                break;
            }
        }
    }

    for (int i = 0; i <= n; i++) F[i] = 1;
    for (int i = 1; i <= n; i++) {
        for (int j = i; j <= n; j += i) {
            if (mu[j / i] == -1) F[j] = F[j] * qmi(f[i], P-2) % P;
            else if (mu[j / i] == 1) F[j] = F[j] * f[i] % P;
        }
    }
    for (int i = 1; i <= n; i++) F[i] = F[i-1] * F[i] % P;
}

signed main() {
    cin >> T;
    init(N-1);
    while (T--) {
        cin >> n >> m;
        if (n < m) swap(n, m);
        int t = 1, res = 1;
        while (t <= m) {
            int nxt = min(n / (n / t), m / (m / t));
            if (nxt <= m) res = res * qmi(F[nxt] * qmi(F[t-1], P-2) % P, (n / t) * (m / t) % (P-1)) % P;
            else res = res * qmi(F[m] * qmi(F[t-1], P-2) % P, (n / t) * (m / t) % (P-1)) % P;
            t = nxt + 1;
        }
        cout << res << endl;
    }
    return 0;
} 
```

## [国家集训队] Crash 的数字表格

> 给定正整数 $n,m$ 求：
> $$
> \left(\sum_{i=1}^n\sum_{j=1}^m\text{lcm}(i,j)\right)\bmod 20101009
> $$
> 其中 $1\le n,m\le 10^7$。
>
> 题目链接：[P1829](https://www.luogu.com.cn/problem/P1829)。

首先 $\text{lcm}(i,j)=ij/\gcd(i,j)$，假设 $n\ge m$，设 $S(n)=\sum_{i=1}^n i$，有：
$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^m \text{lcm}(i,j)&=\sum_{i=1}^n\sum_{j=1}^m \frac{ij}{\gcd(i,j)}\\
&=\sum_{d=1}^m\frac{1}{d}\sum_{i=1}^n\sum_{j=1}^mij[\gcd(i,j)=d]\\
&=\sum_{d=1}^m\frac{1}{d}\sum_{i=1}^{\lfloor\frac{n}{d}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{d}\rfloor}ijd^2[\gcd(i,j)=1]\\
&=\sum_{d=1}^m d\sum_{i=1}^{\lfloor\frac{n}{d}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{d}\rfloor}ij\sum_{x\mid \gcd(i,j)}\mu(x)\\
&=\sum_{d=1}^md\sum_{x=1}^m\sum_{i=1}^{\lfloor\frac{n}{d}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{d}\rfloor} ij\mu(x)[x\mid i][x\mid j]\\
&=\sum_{d=1}^md\sum_{x=1}^mx^2\mu(x)\sum_{i=1}^{\lfloor\frac{n}{xd}\rfloor}\sum_{j=1}^{\lfloor\frac{m}{xd}\rfloor}ij\\
&=\sum_{d=1}^md\sum_{x=1}^mx^2\mu(x)S(\lfloor\frac{m}{xd}\rfloor)S(\lfloor\frac{n}{xd}\rfloor)\\
&=\sum_{t=1}^mtS(\lfloor\frac{n}{t}\rfloor)S(\lfloor\frac{m}{t}\rfloor)\sum_{d\mid t}\frac{t}{d}\mu(\frac{t}{d})\\
&=\sum_{t=1}^mtS(\lfloor\frac{n}{t}\rfloor)S(\lfloor\frac{m}{t}\rfloor)\sum_{d\mid t}d\mu(d)\\
\end{aligned}
$$
实测小常数的 $O(n\log n)$ 是可以通过的，但是针对这个数据范围需要尽量写一个线性的。

设 $f(n)=\sum_{d\mid n}d\mu(d)$，显然 $f(1)=1$，看能否在线性筛中递推：

1. 若 $p$ 为质数，有 $f(p)=1\mu(1)+p\mu(p)=1-p$ 成立。

2. 若 $p$ 小于 $n$ 的最小质因子，那么对于任意 $d\mid n$，有 $pd\mid pn$ 且 $pd\not\mid n$​，因此：
   $$
   \begin{aligned}
   f(pn)&=\sum_{d\mid n}d\mu(d)+\sum_{d\mid n} pd\mu(pd)\\
   &=\sum_{d\mid n}d\mu(d)-p\sum_{d\mid n} d\mu(d)\\
   &=(1-p)f(n)
   \end{aligned}
   $$

3. 若 $p$ 为 $n$ 的最小质因子，这并不会造成更多的贡献，因为只有当 $d$ 的质因数次数不超过 $1$ 时才会对 $f(n)$ 造成贡献，而 $p$ 本来就存在于 $n$，所以 $f(pn)=f(n)$。

所以就可以开始写了，甚至不需要数论分块，复杂度 $O(n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 10000010, P = 20101009;
int n, m, primes[N], f[N], cntp;
bitset<N> st;

void init(int n) {
    f[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cntp++] = i, f[i] = 1-i;
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i * primes[j]] = 1;
            if (i % primes[j]) f[i * primes[j]] = (1 - primes[j]) * f[i];
            else {
                f[i * primes[j]] = f[i];
                break;
            }
        }
    }
}

int S(int n) {
    return 1ll * (1 + n) * n / 2 % P;
}

int main() {
    cin >> n >> m;
    if (n < m) swap(n, m);
    init(n);
    int res = 0;
    for (int t = 1; t <= m; t++) res = (res + 1ll * t * S(n / t) % P * S(m / t) % P * f[t]) % P;
    if (res < 0) res += P;
    cout << res << endl;
    return 0;
}
```

## [湖北省队互测2014] 一个人的数论

> 给定 $n=\prod_{i=1}^{\omega}p_i^{a_i}$，设 $f_d(n)$ 是小于 $n$ 且与 $n$ 互质的正整数的 $d$ 次方之和。输出 $f_d(n)\bmod (10^9+7)$​。
>
> 数据范围：$0\le d\le 100,1\le \omega\le 10^3,2\le p_i,a_i\le 10^9$。
>
> 题目链接：[P6271](https://www.luogu.com.cn/problem/P6271)。

对于 $S(n)=\sum_{i=1}^n i^d$ 是一个 $d+1$ 次方的多项式，且常数项为 $0$。设系数为 $s_1\sim s_{d+1}$，这个可以高斯消元在 $O(d^3)$ 求解。

接下来化简式子：
$$
\begin{aligned}
f_d(n)&=\sum_{i=1}^ni^d[\gcd(i,n)=1]\\
&=\sum_{i=1}^ni^d\sum_{x\mid \gcd(i,n)}\mu(x)\\
&=\sum_{x\mid n}\sum_{i=1}^n[x\mid i]i^d\mu(x)\\
&=\sum_{x\mid n}x^d\mu(x)\sum_{i=1}^{\frac{n}{x}}i^d\\
&=\sum_{x\mid n}x^d\mu(x)S(\frac{n}{x})\\
\end{aligned}
$$
上面说过 $S(n)$ 是一个 $d+1$ 次的多项式，所以把它写成多项式的形式：
$$
\begin{aligned}
f_d(n)&=\sum_{i=1}^{d+1}\sum_{x\mid n} s_i\left(\frac{n}{x}\right)^ix^d\mu(x)\\
&=\sum_{i=1}^{d+1}\sum_{x\mid n} s_in^ix^{d-i}\mu(x)\\
&=\sum_{i=1}^{d+1}s_in^i\sum_{x\mid n}x^{d-i}\mu(x)
\end{aligned}
$$
设 $F_i(n)=\sum_{x\mid n} x^{d-i}\mu(x)$，看是否能快速求出 $F_i(n)$。先证明一下它是积性函数，设 $m,n$ 互质，则：
$$
\begin{aligned}
F_i(n)F_i(m)&=\left(\sum_{x\mid n}x^{d-i}\mu(x)\right)\left(\sum_{y\mid m}y^{d-i}\mu(y)\right)\\
&=\sum_{x\mid n}\sum_{y\mid m}(xy)^{d-i}\mu(x)\mu(y)\\
&=\sum_{(xy)\mid (mn)} (xy)^{d-i}\mu(xy)\\
&=F_i(nm)
\end{aligned}
$$
且对于素数 $p$，有：
$$
\begin{aligned}
F_i(p^c)&=\sum_{j=0}^{c}(p^j)^{d-i}\mu(p^j)\\
&=1-p^{d-i}
\end{aligned}
$$
不计快速幂，可以在 $O(\omega)$ 的复杂度求出 $F_i(n)$，所以最终的式子是：
$$
f_d(n)=\sum_{i=1}^{d+1}s_in^iF_i(n)
$$
这样复杂度是 $O(d^3+d\omega)$，可以接受。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int D = 110, W = 1010, P = 1e9+7;
int d, w, s[D], p[W], a[W], A[D][D];

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

void gauss(int n) {
    for (int p = 1; p <= n; p++) {
        int target = p;
        for (int i = p+1; i <= n; i++) if (A[i][p]) target = i;
        for (int j = p; j <= n+1; j++) swap(A[p][j], A[target][j]);
        int inv = qmi(A[p][p], P-2);
        for (int j = p; j <= n+1; j++) A[p][j] = A[p][j] * inv % P;
        for (int i = p+1; i <= n; i++)
            for (int j = n+1; j >= p; j--) {
                A[i][j] = (A[i][j] - A[p][j] * A[i][p]) % P;
                if (A[i][j] < 0) A[i][j] += P;
            }
    }

    for (int i = n; i; i--) {
        for (int j = i+1; j <= n; j++) {
            A[i][n+1] = (A[i][n+1] - A[j][n+1] * A[i][j]) % P;
            if (A[i][n+1] < 0) A[i][n+1] += P;
        }
    }
    
    for (int i = 1; i <= n; i++) s[i] = A[i][n+1];
}

int F(int i) {
    int res = 1, power = d-i;
    if (power < 0) power += P-1;
    for (int j = 1; j <= w; j++) {
        res = res * (P + 1 - qmi(p[j], power)) % P;
    }
    return res;
}

signed main() {
    cin >> d >> w;
    for (int i = 1; i <= w; i++) cin >> p[i] >> a[i];
    for (int i = 1; i <= d+1; i++) {
        // 这里是保证 A[i][x] 为 pow(i, x), 实际上 A[i][0] 是不会参与高斯消元的过程的
        for (int j = 0, a = 1; j <= d+1; j++, a = a * i % P) A[i][j] = a;
        A[i][d+2] = (A[i-1][d+2] + A[i][d]) % P;
    }
    gauss(d+1);

    int res = 0, n = 1;
    for (int i = 1; i <= w; i++) n = n * qmi(p[i], a[i]) % P;
    for (int i = 1, ni = n; i <= d+1; i++, ni = ni * n % P) {
        res = (res + s[i] * F(i) % P * ni) % P;
    }
    cout << res << endl;
    return 0;
}
```

## [CF803F] Coprime Subsequences

> 给定长度为 $n$ 的序列 $a_1,\dots,a_n$，问多少个子序列 $T$ 满足 $\gcd(T)=1$，答案对 $10^9+7$ 取模。其中 $1\le n,a_i\le 10^5$。
>
> 题目链接：[CF803F](https://codeforces.com/problemset/problem/803/F)。

还是化简式子：
$$
\begin{aligned}
\sum_{T\subset S}[\gcd(T)=1]&=\sum_{T\subset S}\sum_{d\mid \gcd(T)} \mu(d)\\
&=\sum_{d=1}^{\max(S)}\sum_{d\mid \gcd(T)} \mu(d)\\
&=\sum_{d=1}^{\max(S)}\mu(d)\left(2^{\sum_{i=1}^n [d\mid a_i]}-1\right)
\end{aligned}
$$
所以枚举 $d$ 再看它的倍数个数就行了，这样开一个桶来储存序列，复杂度是 $O(n\log n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010, P = 1e9+7;
int n, mx, cnt[N], primes[N], mu[N], cntp;
bitset<N> st;

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = 1ll * res * a % P;
        a = 1ll * a * a % P;
        k >>= 1;
    }
    return res;
}

void init(int n) {
    mu[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cntp++] = i, mu[i] = -1;
        for (int j = 0; primes[j] <= n / i; j++) {
            st[i * primes[j]] = 1;
            if (i % primes[j]) mu[i * primes[j]] = -mu[i];
            else {
                mu[i * primes[j]] = 0;
                break;
            }
        }
    }
}

int main() {
    cin >> n;
    for (int i = 1, a; i <= n; i++) cin >> a, cnt[a]++, mx = max(mx, a);
    init(mx);
    int res = 0;
    for (int d = 1; d <= mx; d++) {
        int now = 0;
        for (int nd = d; nd <= mx; nd += d) now += cnt[nd];
        res = (res + 1ll * mu[d] * (qmi(2, now) - 1)) % P;
        if (res < 0) res += P;
    }
    cout << res << endl;
    return 0;
}
```

进一步地，可以对这个题进行拓展，求满足 $\gcd(T)\times \operatorname{len}(T)=k$ 的字序列个数，其中 $1\le n,a_i,k\le 5\times 10^5$。

思路是首先要枚举最大公约数 $t$，然后化简式子：
$$
\sum_{t\mid k}\sum_{T\subset S,\operatorname{len}(T)=k/t}[\gcd(T)=t]
$$
对于每一个 $t$，找到所有 $x\in S$ 使得 $t\mid x$，这一步可以通过枚举倍数的方式做到 $O(n\log n)$，接下来假设 $S'$ 为满足上述条件的 $x$ 且 $x\gets \frac{x}{t}$，那么：
$$
\begin{aligned}
\sum_{t\mid k}\sum_{T\subset S',|T|=k/t}[\gcd(T)=1]&=\sum_{t\mid k}\sum_{T\subset S,|T|=k/t}\sum_{d\mid \gcd(T)}\mu(d)\\
&=\sum_{t\mid k}\sum_{d=1}^{\max(S')}\mu(d)\binom{\sum_{i=1}^{n'}[d\mid a'_i]}{k/t}\\
&=\sum_{t\mid k}\sum_{d=1}^{\lfloor \max(S)/t\rfloor}\mu(d)\binom{\sum_{i=1}^n[dt\mid a_i]}{k/t}
\end{aligned}
$$
设 $f(x)=\sum_{i=1}^n [x\mid a_i]$，这是可以 $O(n\log n)$ 预处理的；对于最终要计算的式子，这最坏是 $O(\sum_{t=1}^n \frac{n}{t})=O(n\log n)$ 的。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 600010, P = 1e9+7;
int n, mx, k, cnt[N], primes[N], mu[N], fact[N], infact[N], f[N], cntp;
bitset<N> st;

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = 1ll * res * a % P;
        a = 1ll * a * a % P;
        k >>= 1;
    }
    return res;
}

void init(int n) {
    mu[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cntp++] = i, mu[i] = -1;
        for (int j = 0; primes[j] <= n / i; j++) {
            st[i * primes[j]] = 1;
            if (i % primes[j]) mu[i * primes[j]] = -mu[i];
            else {
                mu[i * primes[j]] = 0;
                break;
            }
        }
    }
}

int C(int n, int m) {
    if (n < m) return 0;
    return 1ll * fact[n] * infact[m] % P * infact[n-m] % P;
}

signed main() {
    cin >> n >> k;
    fact[0] = infact[0] = 1;
    for (int i = 1; i <= n; i++) fact[i] = 1ll * fact[i-1] * i % P;
    infact[n] = qmi(fact[n], P-2);
    for (int i = n-1; i; i--) infact[i] = 1ll * (i+1) * infact[i+1] % P;

    for (int i = 1, a; i <= n; i++) cin >> a, cnt[a]++, mx = max(mx, a);
    init(mx);
    for (int i = 1; i <= mx; i++)
        for (int j = i; j <= mx; j += i)
            f[i] += cnt[j];
    
    int res = 0;
    for (int t = 1; t <= k; t++) {
        if (k % t) continue;
        for (int d = 1; d <= mx / t; d++) {
            res = (res + 1ll * mu[d] * C(f[d * t], k / t)) % P;
        }
    }
    if (res < 0) res += P;
    cout << res << endl;
    return 0;
}
```

