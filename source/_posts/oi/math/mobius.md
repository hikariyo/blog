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

## 引理

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
特别地，$n=1$ 时看作 $k=0$ 因此 $\mu(1)=1$

若函数 $S(n)$ 定义为：
$$
S(n)=\sum_{d|n}\mu(n)
$$
则其取值一定为下列情况：
$$
S(n)=\left\{\begin{aligned}
&1,n=1\\
&0,n>1
\end{aligned}\right.
$$
当 $n=1$ 时，结果显然成立，下面证明 $n>1$ 时的情况，首先分解质因子：
$$
n=\prod_{i=1}^{k}p_i^{c_i}\\
\text{Let }d=\prod_{i=1}^{k_0}p_i^{b_i}
$$
当任意 $b_i>1$ 时，$\mu(d)=0$，所以不用考虑这种情况，因此就是从 $k$ 个质因子中选几个进行求和的问题了。
$$
S(n)=C_k^0(-1)^0+C_k^1(-1)^1+\cdots+C_k^k(-1)^k
$$
由二项式定理可知：
$$
(1+x)^k=C_k^0x^0+C_k^1x^1+\cdots+C_k^kx^k
$$
所以：
$$
S(n)=(1-1)^k=0
$$
证毕。

## 定理

莫比乌斯反演是一个反解的过程，第一个版本是枚举 $n$ 的约数 $d$：
$$
F(n)=\sum_{d|n}f(d)\\
\Rightarrow f(n)=\sum_{d|n}\mu(d)F(\frac{n}{d})
$$
直接把定理的结论展开进行证明：
$$
\begin{aligned}
\text{RHS}&=\sum_{d|n}\mu(d)F(\frac{n}{d})\\
&=\sum_{d|n}\mu(d)\sum_{t|(n/d)}f(t)\\
&=\sum_{t|n}f(t)\sum_{d|(n/t)}\mu(d)\\
&=\sum_{t|n}f(t)S(\frac{n}{t})\\
&=f(n)=\text{LHS}\\
\end{aligned}
$$
交换求和顺序时，观察到 $d=1$ 时 $t$ 可以取遍 $n$ 的所有约数，那当枚举每一个 $t$ 时，与之对应的 $d$ 一定有 $t|(n/d)$，也就是 $d|(n/t)$，这个写成倍数然后代一下就能看出来。

还有另一个版本，枚举 $n$ 的倍数 $d$：
$$
F(n)=\sum_{n|d}f(d)\\
\Rightarrow f(n)=\sum_{n|d}\mu(\frac{d}{n})F(d)
$$
证明：
$$
\text{Let }d=kn,k\in \N^*\\
\begin{aligned}
\text{RHS}&=\sum_{n|d}\mu(\frac{d}{n})\sum_{d|t}f(t)\\
&=\sum_{n|t}f(t)\sum_{k|(t/n)}\mu(k)\\
&=\sum_{n|t}f(t)S(\frac{t}{n})\\
&=f(n)=\text{LHS}
\end{aligned}
$$
下面是例题。

## 一个取整公式

下面这个公式是成立的。
$$
\lfloor\frac{a}{bc}\rfloor = \lfloor\frac{\lfloor a/b\rfloor }{c}\rfloor
$$
证明：
$$
\text{Let }a=kb+r, k=qc+p\\
\text{Where }r\in[0,b),p\in[0,c)\\
\begin{aligned}
a&=kb+r\\
&=qbc+pb+r\\
&\le qbc+b(c-1)+b-1\\
&=qbc+bc-1\\
\frac{a}{bc}&\le \frac{bc(q+1)-1}{bc}\\
&= q+1-\frac{1}{bc}\\
&\lt q+1\\
\therefore \text{LHS}&=q=\text{RHS}
\end{aligned}
$$
[原文链接](https://math.stackexchange.com/questions/2557458/is-a-b-c-equal-to-a-bc-for-integer-division)在这里。

## Problem b

> 对于给出的 n 个询问，每次求有多少个数对 (x,y)，满足 a≤x≤b，c≤y≤d，且 gcd(x,y)=k。
>
> 题目链接：[AcWing 2702](https://www.acwing.com/problem/content/2704/)，[P2522](https://www.luogu.com.cn/problem/P2522)。

首先用前缀和的思想将其转化为求一个矩形内满足要求的个数。

$$
\begin{aligned}
F(n)&=\sum_{i=1}^a\sum_{j=1}^b[n|\gcd(i,j)]\\
f(n)&=\sum_{i=1}^a\sum_{j=1}^b[n=\gcd(i,j)]\\
F(n)&=\sum_{n|d}f(d)
\end{aligned}
$$
根据莫比乌斯反演，那么就有：
$$
\begin{aligned}
f(n)&=\sum_{n|d}\mu(\frac{d}{n})F(d)\\
&=\sum_{i\in\N^*} \mu(i) \lfloor\frac{a}{in}\rfloor\lfloor\frac{b}{in}\rfloor
\end{aligned}
$$
这里 $k$ 的上限就是当右侧两个整除结果出现 0 之前。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 50010;
int primes[N], mu[N], smu[N], cnt;
bool st[N];
int k;

void init(int n) {
    mu[1] = smu[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) {
            primes[cnt++] = i;
            mu[i] = -1;
        }
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i*primes[j]] = true;
            if (i % primes[j] == 0) {
                break;
            }
            mu[i*primes[j]] = -mu[i];
        }
        
        smu[i] = smu[i-1]+mu[i];
    }
    
}

int get(int a, int b) {
    a /= k, b /= k;
    int res = 0;
    for (int i = 1, j; i <= min(a, b); i = j+1) {
        j = min(a/(a/i), b/(b/i));
        
        res += (smu[j]-smu[i-1])*(a/i)*(b/i);
    }
    return res;
}

int main() {
    int T;
    cin >> T;
    init(N-1);
    while (T--) {
        int a, b, c, d;
        cin >> a >> b >> c >> d >> k;
        printf("%d\n", get(a-1, c-1) + get(b, d) - get(b, c-1) - get(a-1, d));
    }
    return 0;
}
```

## 约数个数和

> 设 $d(x)$ 为约数个数，给定 $n,m$ 求 $\sum_{i=1}^n\sum_{j=1}^md(ij)$
>
> 题目链接：[P3327](https://www.luogu.com.cn/problem/P3327)，[AcWing 1358](https://www.acwing.com/problem/content/1360/)。

引理：$d(ij)=\sum_{x|i}\sum_{y|j}[\gcd(x,y)=1]$

证明：对于任意一个公共质因子 $p_i$，都有 $x, y$ 的取值都有 $(1, 1),(p_i,1),\cdots,(p_i^{\alpha_i},1),(1,p_i),\cdots, (1,p_i^{\beta_i})$，因此最终有 $(\alpha_i+\beta_i+1)$ 种取值，这和约数个数是等价的，如果不同时包含 $p_i$，对应次数置为 0 即可。
$$
\begin{aligned}
F(t)&=\sum_{i=1}^n\sum_{j=1}^m \sum_{x|i}\sum_{y|j}[t|\gcd(x,y)]\\
f(t)&=\sum_{i=1}^n\sum_{j=1}^m \sum_{x|i}\sum_{y|j}[t=\gcd(x,y)]\\
F(t)&=\sum_{t|d}f(d)\\
f(t)&=\sum_{t|d}\mu(\frac{d}{t})F(d)
\end{aligned}
$$
对 $F(t)$ 变形时，先考虑到 $x,y$ 分别可以取遍 $1\sim n, 1\sim m$，并且内部式子和 $i,j$ 无关，因此内部式子的值只看 $i,j$ 中有多少 $x,y$ 的倍数。
$$
\begin{aligned}
F(t)&=\sum_{x=1}^n\sum_{y=1}^m \lfloor\frac{n}{x}\rfloor\lfloor\frac{m}{y}\rfloor[t|\gcd(x,y)]\\
\end{aligned}
$$
由于 $t|x, t|y$ 所以只需要枚举 $t$ 的倍数的那个系数，设 $x'=x/t, y'=y/t,n'=n/t,m'=m/t$，这里都是向下取整，那个符号就不写了，结合文章开头提到的取整公式，则有：
$$
\begin{aligned}
F(t)&=\sum_{x'=1}^{n'}\sum_{y'=1}^{m'} \lfloor\frac{n'}{x'}\rfloor\lfloor\frac{m'}{y'}\rfloor\\
&=(\sum_{x'=1}^{n'} \lfloor\frac{n'}{x'}\rfloor)(\sum_{y'=1}^{m'}\lfloor\frac{m'}{y'}\rfloor)
\end{aligned}
$$

本质上就是常数可以提到外面去。
$$
\begin{aligned}
h(n)&=\sum_{x=1}^n\lfloor\frac{n}{x}\rfloor\\
f(t)&=\sum_{t|d}\mu(\frac{d}{t})F(d)\\
&=\sum_{t|d}\mu(\frac{d}{t})h(\lfloor\frac{n}{d}\rfloor)h(\lfloor\frac{m}{d}\rfloor)\\
&=\sum_{k=1}^{\min(n',m')}\mu(k)h(\lfloor\frac{n'}{k}\rfloor)h(\lfloor\frac{m'}{k}\rfloor)
\end{aligned}
$$
要求的是 $f(1)$，因此最终要求的就是：
$$
\sum_{i=1}^{\min(n,m)}\mu(i)h(\lfloor\frac{n}{i}\rfloor)h(\lfloor\frac{m}{i}\rfloor)
$$
可以预处理 $h(x)$ 数组。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 50010;
bool st[N];
int primes[N], cnt, mu[N], smu[N], h[N];

void init(int n) {
    mu[1] = smu[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) {
            mu[i] = -1;
            primes[cnt++] = i;
        }
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i*primes[j]] = true;
            if (i % primes[j] == 0) break;
            
            mu[i*primes[j]] = -mu[i];
        }
        
        smu[i] = smu[i-1] + mu[i];
    }
    
    for (int s = 1; s <= n; s++) {
        for (int i = 1, j; i <= s; i = j+1) {
            j = s/(s/i);
            h[s] += (j-i+1)*(s/i);
        }
    }
}

int main() {
    init(N-1);
    int T; scanf("%d", &T);
    while (T--) {
        int n, m;
        scanf("%d%d", &n, &m);
        LL res = 0;
        for (int i = 1, j; i <= n && i <= m; i = j+1) {
            j = min(n/(n/i), m/(m/i));
            res += (LL)(smu[j]-smu[i-1])*h[n/i]*h[m/i];
        }
        printf("%lld\n", res);
    }
    return 0;
}
```

