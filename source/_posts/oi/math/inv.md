---
date: 2023-10-22 20:20:00
title: 数学 - 逆元
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
description: 系统整理一下逆元相关的知识点，包括快速幂求逆元、拓展欧几里得求逆元、线性递推求逆元。
---

## 简介

系统整理一下逆元相关的知识点，包括快速幂求逆元、拓展欧几里得求逆元、线性递推求逆元。

其中快速幂求逆元利用了费马小定理，它是欧拉定理的一个特殊情况。

首先逆元有几个性质：

1. 唯一性。设存在 $y_1\not\equiv y_2$ 同时为 $x$ 模 $p$ 的逆元，那么可以得到：
    $$
    \begin{aligned}
    xy_1&\equiv xy_2\pmod p\\
    x(y_1-y_2)&\equiv 0\pmod p
    \end{aligned}
    $$
    由于 $x\not\equiv 0$，所以必然有 $y_1-y_2\equiv 0$，矛盾。

2. 互质性。即只有在 $\gcd(x,p)=1$ 的情况下存在 $x$ 的逆元，如果不这样，那么把定义式用拓欧方程写出来：
    $$
    xx^{-1}+kp=1,k<0
    $$
    由于 $x^{-1},k$ 都为整数，那么：
    $$
    \gcd(x,p)|(xx^{-1}+kp)\Rightarrow \gcd(x,p)|1
    $$
    这与假设条件矛盾。

## 模板 I

> 给定 $n,p$ 求 $1\sim n$ 中所有整数在模 $p$ 意义下的乘法逆元。
>
> 这里 $a$ 模 $p$ 的乘法逆元定义为 $ax\equiv1\pmod p$ 的解。
>
> **输入格式**
>
> 一行两个正整数 $n,p$。
>
> **输出格式**
>
> 输出 $n$ 行，第 $i$ 行表示 $i$ 在模 $p$ 下的乘法逆元。
>
> **数据范围**
>
> $1 \leq n \leq 3 \times 10 ^ 6$，$n < p < 20000528$，输入保证 $p$ 为质数。
>
> 题目链接：[P3811](https://www.luogu.com.cn/problem/P3811)。

如果用快速幂求，那么复杂度是 $O(n\log p)$，时限是 $0.5s$，故意卡了这个方法。

接下来就用到线性递推了，显然 $inv(1)=1$，然后考虑将 $p$ 拆一下，令 $p=ki+r,r\in[0,i)$，那么：
$$
\begin{aligned}
ki+r&\equiv 0\\
kr^{-1}+i^{-1} &\equiv 0\\
i^{-1}&\equiv -kr^{-1}\\
&\equiv -\lfloor\frac{p}{i}\rfloor(p\bmod  i)^{-1}
\end{aligned}
$$
由于 $r<i$ 因此可以递推。

```cpp 
#include <bits/stdc++.h>
using namespace std;

const int N = 3000010;
int n, p, inv[N];

int main() {
    scanf("%d%d", &n, &p);
    inv[1] = 1;
    puts("1");
    for (int i = 2; i <= n; i++) {
        inv[i] = 1LL * -p / i * inv[p%i] % p;
        if (inv[i] < 0) inv[i] += p;
        printf("%d\n", inv[i]);
    }
    return 0;
}
```

## 模板 II

> 给定 $n$ 个正整数 $a_i$ ，求它们在模 $p$ 意义下的乘法逆元。
>
> 由于输出太多不好，所以将会给定常数 $k$，你要输出的答案为：  
> $$
> \sum\limits_{i=1}^n\frac{k^i}{a_i}
> $$
> 答案对 $p$ 取模。
>
> **输入格式**
>
> 第一行三个正整数 $n,p,k$，意义如题目描述。  
>
> 第二行 $n$ 个正整数 $a_i$，是你要求逆元的数。
>
> **输出格式**
>
> 输出一行一个整数，表示答案。
>
> **数据范围**
>
> 对于 $100\%$ 数据，$1\le n \le 5\times 10^6$，$2\le k < p \le 10^9$，$1\le a_i < p$，保证 $p$ 为质数。
>
> 题目链接：[P5431](https://www.luogu.com.cn/problem/P5431)。

如果暴力做的话复杂度是 $O(n\log p)$ 还是被卡掉了，所以要线性。

通分一下式子，发现是：
$$
\frac{1}{\prod_{i=1}^n a_i} \sum_{i=1}^n (k^i\frac{\prod_{j=1}^n a_j}{a_i})\\
=\frac{1}{\prod_{i=1}^n a_i} \sum_{i=1}^n (k^i\prod_{j=1}^{i-1} a_j\prod_{j=i+1}^n a_j)\\
$$
所以求前缀后缀积即可，开个快读卡下常就过了。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 5000010;
int n, p, k, a[N], pre[N], suf[N];

template<typename T>
void read(T& res) {
    char ch = getchar();
    T sig = 1;
    while (ch < '0' || ch > '9') {
        if (ch == '-') sig = -1;
        ch = getchar();
    }
    res = 0;
    while ('0' <= ch && ch <= '9') {
        res = res*10 + (ch&15);
        ch = getchar();
    }
    res *= sig;
}

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = 1LL * res * a % p;
        a = 1LL * a * a % p;
        k >>= 1;
    }
    return res;
}

int main() {
    read(n), read(p), read(k);

    pre[0] = 1, suf[n+1] = 1;
    for (int i = 1; i <= n; i++) read(a[i]);
    for (int i = 1; i <= n; i++) pre[i] = 1LL * pre[i-1] * a[i] % p;
    for (int i = n; i >= 1; i--) suf[i] = 1LL * suf[i+1] * a[i] % p;

    int prod = k, res = 0;
    for (int i = 1; i <= n; i++) {
        res = (res + 1LL * prod * pre[i-1] % p * suf[i+1]) % p;
        prod = 1LL * prod * k % p;
    }
    return !printf("%lld\n", 1LL * res * qmi(pre[n], p-2) % p);
}
```

## [NOIP2012] 同余方程

> 求关于 $x$ 的同余方程 $ax \equiv 1 \pmod {b}$ 的最小正整数解。
>
> **输入格式**
>
> 一行，包含两个整数 $a,b$，用一个空格隔开。
>
> **输出格式**
>
> 一个整数 $x_0$，即最小正整数解。输入数据保证一定有解。
>
> **数据范围**
>
> 对于 $100\%$ 的数据，$2 ≤a, b≤ 2\times 10^9$。
>
> 题目链接：[P1082](https://www.luogu.com.cn/problem/P1082)。

不保证 $b$ 为质数，那么没法用费马小定理，这里用拓展欧几里得算法求解。

```cpp
#include <bits/stdc++.h>
using namespace std;

void exgcd(int a, int b, int& x, int &y) {
    if (b == 0) {
        x = 1, y = 0;
        return;
    }
    exgcd(b, a % b, y, x);
    y -= a/b * x;
}

int main() {
    int a, b, x, y;
    scanf("%d%d", &a, &b);
    exgcd(a, b, x, y);
    x %= b;
    if (x < 0) x += b;
    return !printf("%d\n", x);
}
```

## 数字变换

> 给定两个自然数 $A,B$，每次可以选择如下操作之一：
>
> - 将 $A$ 变成 $A+11\bmod 998244353$。
>
> - 将 $A$ 变成 $A\times 21\bmod 998244353$。
>
> - 将 $A$ 变成 $A^{31}\bmod 998244353$。
>
> 问至少需要多少次操作才能将 $A$ 变成 $B$。可以证明一定有解。

首先考虑到这些操作得到的数并没有什么规律，看作较为随机的操作，所以如果单纯爆搜是 $O(p)$ 的复杂度。那么结合生日悖论，我们设现在已经取了 $k$ 个数字，并没有取到与之前相同的数字的概率是：
$$
\prod_{i=0}^{k-1} \frac{p-i}{p}
$$
计算出，当 $k=4\sqrt p$ 时，没有取到相同数字的概率已经降到了 $0.03\%$，所以我们只需要 $O(\sqrt p)$ 次查找就一定能找到相同的数字，所以考虑双向搜索。

前两个的逆元都好说，第三个的逆就需要单独摘出来看看了。

首先 $998244353$ 是质数，由欧拉定理可以得到：
$$
a^{b}\equiv a^{b\bmod \varphi(p)}\pmod p
$$
设知道 $A^{31}=a$ 那么 $a^k =A$ 就有：
$$
\begin{aligned}
a^{31k}&\equiv a\pmod p\\
31k&\equiv 1\pmod {\varphi(p)}
\end{aligned}
$$
这个根不一定是唯一的，但是我们一定能通过这种方式构造出一个合法解，用 `map` 存储状态，复杂度 $O(\sqrt p\log p)$ 可过。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int P = 998244353;

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

int bfs(queue<int>& q, map<int, int>& d, map<int, int>& d2, bool inv) {
    int t = q.front(); q.pop();
    int dt = d[t];
    if (d2.count(t)) return d2[t] + dt;

    int to;
    if (inv) to = (t - 11 + P) % P;
    else to = (t + 11) % P;
    if (!d.count(to)) d[to] = dt + 1, q.push(to);

    if (inv) to = (t * 617960790) % P;
    else to = (t * 21) % P;
    if (!d.count(to)) d[to] = dt + 1, q.push(to);

    if (inv) to = qmi(t, 225410015);
    else to = qmi(t, 31);
    if (!d.count(to)) d[to] = dt + 1, q.push(to);
    
    return -1;
}

int solve(int A, int B) {
    map<int, int> d1, d2;
    queue<int> q1, q2;
    d1[A] = 0, d2[B] = 0;
    q1.push(A), q2.push(B);

    int res = -1, inv = 1;
    while (res == -1) {
        if (inv) res = bfs(q2, d2, d1, true);
        else res = bfs(q1, d1, d2, false);
        inv ^= 1;
    }
    return res;
}

signed main() {
    // 31^{-1} \equiv 225410015 (mod 998244352)
    // 21^{-1} \equiv 617960790 (mod 998244353)
    int A, B;
    cin >> A >> B;
    cout << solve(A, B) << '\n';

    return 0;
}
```

