---
date: 2025-10-31 13:28:00
updated: 2025-10-31 13:28:00
title: "[ICPC杭州 2024 M] Make It Divisible"
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 关于区间求 GCD 与单调栈。
---

## 题目

[QOJ 9738](https://qoj.ac/contest/1893/problem/9738)。

## 引理

关于 GCD，满足 $\gcd(a,b)=\gcd(a,|b-ka|)$。

只需证明：

1. $\gcd(a,b)=\gcd(a,b-a),a<b$：

   令 $d=\gcd(a,b)$，那么 $a=k_1d,b=k_2d$，于是 $d\mid a$ 且 $d\mid b-a$，于是 $\gcd(a,b)\mid \gcd(a,b-a)$

   同理可证 $\gcd(a,b-a)\mid \gcd(a,b)$，于是这两个数字相等。

2. $\gcd(a,b)=\gcd(a,a-b),a>b$：

   证明同上。

于是，我们可以进行若干次第一条结论得到 $\gcd(a,b)=\gcd(a,b-ka),b-ka>0$；

如果 $b-ka>0$ 且 $b-(k+k')a<0$，可以有 $\gcd(a,b-ka)=\gcd(a,k'a-(b-ka))=\gcd(a,(k+k')a-b)$。

这就证明了 $\gcd(a,b)=\gcd(a,|b-ka|)$。

## 思路

一个 trivial 的思路是，用 $O(n^2)$ 的复杂度枚举所有区间，找到符合条件的 $x$，并给它们求交集。

观察一下定义，对于一个合法的 $[l,r]$，对所有的 $l\le i\le r$，都有 $a_d\mid a_i$，那么 $a_d$ 一定是 $\min_{i=l}^r a_i$。

如果 $[l,r]$ 是一个 *divisible interval*，它以 $a_d$ 为最小值的子区间一定也是 *divisible interval*。

而题目要求所有的子区间都应该是 *divisible interval*，于是，对于最小值为 $a_d$ 的所有区间，只需要使极大的那一个满足要求就行。

又因为，所有的子区间一定以某个 $a_i$ 为最小值。所以我们反过来枚举 $a_d$，找到它控制的所有区间，并且只要使极大的那个满足就行。

找到每个 $a_d$ 控制的极大区间，可以用单调栈来实现。官方题解是笛卡尔树，这两个做法本质上是一样的。

这里有一个转化，$\forall l\le i\le r, a_d\mid a_i\Leftrightarrow \gcd(a_l,a_{l+1},\dots,a_r)=a_d$。

现在考虑 $x$ 的问题：
$$
\begin{aligned}
\gcd(a_l+x,a_{l+1}+x,\dots,a_r+x)&=\gcd(a_d+x,a_l+x,a_{l+1}+x,\dots,a_r+x)\\
&=\gcd(a_d+x,|a_l-a_d|,|a_{l+1}-a_l|,\dots,|a_r-a_{r-1}|)\\
&=\gcd(a_d+x,\gcd(|a_l-a_d|,|a_{l+1}-a_l|,\dots,|a_r-a_{r-1}|))\\
&=\gcd(a_d+x,g_d)\\
&=a_d+x
\end{aligned}
$$
我们可以通过 ST 表快速求出 $g_d$。特殊地，对于 $l=r$ 的情况，令 $g_d=0$，也就是这个式子恒成立。

如果 $g_d\neq 0$，那么一定有 $a_d+x\mid g_d$，于是我们可以枚举 $g_d$ 的因数，初步找到所有候选的 $x$。

因为我们的答案是对所有 $x$ 的集合取交集，所以可以先找到一组答案的超集，然后检查它们是不是都合法。

这样做的复杂度是 $O(n\max d(V))$，其中 $V=10^9$，$\max d(V)=1344$。

## 实现

实现上来说，如果 $n=1$，自然所有的 $x$ 都合法；如果对于最大的区间 $g_d=0$，说明每一个数都是 $a_d$。这是因为只有所有数都是 $0$，$\gcd$ 的结果才会是 $0$，观察 $g_d$ 的定义不难得到所有的数都是 $a_d$。此时也是所有的 $x$ 都合法。

还有一个需要注意的，就是枚举因数和质因数的方法并不相同，我一开始写成了枚举质因数。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 5e4+10, K=20;
int a[N], n, k, L[N],R[N];
int st[K][N], lg2[N];

void breakdown(unordered_set<int>& v, int x, int ad) {
    for (int i = 1; i*i <= x; i++) {
        if (x % i == 0) {
            if (1 <= i - ad && i - ad <= k) v.insert(i - ad);
            if (1 <= x/i - ad && x/i - ad <= k) v.insert(x/i - ad);
        }
    }
}

int query(int l, int r) {
    int k = lg2[r-l+1];
    return gcd(st[k][l], st[k][r-(1<<k)+1]);
}

void solve() {
    cin >> n >> k;
    for (int i = 2; i <= n; i++) lg2[i] = lg2[i>>1] + 1;

    int mn = 1e9;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        mn = min(mn, a[i]);
    }

    stack<int> stk;
    for (int i = 1; i <= n; i++) {
        while (stk.size() && a[stk.top()] >= a[i]) stk.pop();
        if (stk.size()) L[i] = stk.top() + 1;
        else L[i] = 1;
        stk.push(i);
    }
    stack<int>().swap(stk);
    for (int i = n; i >= 1; i--) {
        while (stk.size() && a[stk.top()] >= a[i]) stk.pop();
        if (stk.size()) R[i] = stk.top() - 1;
        else R[i] = n;
        stk.push(i);
    }
    
    for (int k = 0; k < K; k++) {
        for (int i = 1; i+(1<<k)-1 <= n; i++) {
            if (k == 0) st[k][i] = abs(a[i] - a[i-1]);
            else st[k][i] = gcd(st[k-1][i], st[k-1][i+(1<<(k-1))]);
        }
    }

    unordered_set<int> ans;
    if (n == 1) {
        cout << k << ' ' << 1ll * k * (k+1) / 2 << '\n';
        return;
    }

    int g = query(2, n);
    g = gcd(g, abs(a[1] - mn));
    if (g == 0) {
        cout << k << ' ' << 1ll * k * (k+1) / 2 << '\n';
        return;
    }
    breakdown(ans, g, mn);

    for (int i = 1; i <= n; i++) {
        if (L[i] == R[i]) continue;
        g = query(L[i]+1, R[i]);
        g = gcd(g, abs(a[L[i]] - a[i]));

        for (auto it = ans.begin(); it != ans.end(); ) {
            int x = *it;
            if (gcd(g, a[i] + x) != a[i] + x) it = ans.erase(it);
            else it++;
        }
    }
 
    int sum = 0;
    for (int x: ans) sum += x;
    cout << ans.size() << ' ' << sum << '\n';
}

signed main() {
    ios::sync_with_stdio(0);
    cin.tie(0), cout.tie(0);
    int T;
    cin >> T;
    while (T--) solve();
    return 0;
}
```

