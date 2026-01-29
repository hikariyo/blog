---
date: 2025-10-28 19:23:00
updated: 2025-10-28 19:23:00
title: "[CCPC重庆 2024 A] 乘积，欧拉函数，求和"
katex: true
tags:
- Algo
- C++
- Math
- DP
categories:
- XCPC
description: 状压 DP 求欧拉函数的值。
---

## 题目

[QOJ 9619](https://qoj.ac/contest/1840/problem/9619)。

## 思路

如果质因数的种类足够小，设 $\pi(V)$ 表示小于 $V$ 的素数个数，那么显然有一个 $O(n2^{\pi(V)})$ 的 状压 dp 思路：

设质数数列 $p_1,p_2,\dots,p_{\pi(V)}$，它们构成的某个集合为 $S$，那么 $S$ 可以用一个二进制数表示。

设 $f(i,S)$ 表示 $a_1\sim a_i$，$\prod a$ 的质因数集合为 $S$ 的答案，那么对于所有的 $S$ 有：
$$
f(i,S\cup s_i) \gets f(i,S\cup s_i) + a_i f(i-1,S)\prod_{p\in T}(1-p^{-1})
$$
这里的 $T$ 集合表示的就是在 $s_i$ 中存在而在 $S$ 中不存在的质数，形式化地，$T=s_i\cap \complement_U S$，其中 $U$ 是全集。

我们可以令 $P(T)=\prod_{p\in T} (1-p^{-1})$ 来降低复杂度。

不幸地是，质因数的种类并非足够小，本题 $V=3000$，这么做的复杂度不可接受。

但是有一个常见且简单的结论，那就是对于一个 $\le V$ 的数，它的 $\ge \sqrt V$ 质因数至多一个。

经过打表发现 $\pi(\sqrt V)=16$，于是我们对每个数字 $a_i$ 处理出 $s_i$ 与 $lp_i$，分别表示前 $16$ 个质因数的情况以及 $\ge \sqrt{V}$ 的那一个质因数。

然后将 $a_i$ 按照 $lp_i$ 升序排序，将 $lp_i$ 相同的放在一起，现在我们用两个数组 $f(i,S),g(i,S)$ 分别表示不含当前 $lp$ 与含当前 $lp$ 且前 $16$ 个质因数情况为 $S$ 的答案，那么转移会有：
$$
g(i,S\cup s_i) \gets g(i,S\cup s_i) + a_i f(i-1,S)P(s_i\cap \complement_U S)(1-lp^{-1})\\
g(i,S\cup s_i) \gets g(i,S\cup s_i) + a_i g(i-1,S)P(s_i\cap \complement_U S)\\
$$
处理完一段相同的 $lp$ 后，将 $g$ 数组清空，并且令 $f(i,S)\gets f(i,S)+g(i,S)$，因为后面的不可能含有 $lp$ 了。

## 实现

实现上来说，我们没有必要开二维数组，因为并集对应的是或运算，它一定是 $S\operatorname{or}s_i\ge S$，所以我们从大到小枚举 $S$ 就行了。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int P = 998244353, N = 3010, K = 16;
int n;
int f[1<<K], g[1<<K], phifac[1<<K];
int primes[K] = {
    2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53,
};

struct Number {
    int a;

    int state;
    // large prime factor
    int lp;

    bool operator<(const Number& a) const {
        return lp < a.lp;
    }
} num[N];

int qpow(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

void add(int& a, int b) {
    a = (a + b) % P;
}

signed main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);

    cin >> n;
    for (int i = 1, a; i <= n; i++) {
        cin >> a;
        num[i].a = a;
        for (int p = 0; p < K; p++) {
            while (a % primes[p] == 0) {
                num[i].state |= 1 << p;
                a /= primes[p];
            }
        }
        // 如果不存在大于 sqrt(V) 的质数，那么 lp 显然是 1
        num[i].lp = a;
    }
    sort(num+1, num+1+n);

    f[0] = 1;
    for (int S = 0; S < 1<<K; S++) {
        // 这里是上文的 P(T)
        phifac[S] = 1;
        for (int k = 0; k < K; k++) {
            if (S >> k & 1) {
                phifac[S] = phifac[S] * (primes[k] - 1) % P * qpow(primes[k], P-2) % P;
            }
        }
    }

    for (int i = 1; i <= n; i++) {
        int j = i;

        memset(g, 0, sizeof g);
        while (j <= n && num[j].lp == num[i].lp) {
            int ffac = (num[j].lp - 1) * qpow(num[j].lp, P-2) % P;
            for (int S = (1<<K)-1; S >= 0; S--) {
                if (num[i].lp == 1) {
                    int fac = f[S] * num[j].a % P * phifac[num[j].state & ~S] % P;
                    add(f[S | num[j].state], fac);
                }
                else {
                    int facf = f[S] * num[j].a % P * ffac % P * phifac[num[j].state & ~S] % P;
                    int facg = g[S] * num[j].a % P * phifac[num[j].state & ~S] % P;
                    add(g[S | num[j].state], facf + facg);
                }
            }
            j++;
        }

        i = j-1;
        for (int S = 0; S < 1 << K; S++) add(f[S], g[S]);
    }
    int ans = 0;
    for (int S = 0; S < 1 << K; S++) add(ans, f[S]);
    cout << ans << '\n';
    return 0;
}
```

