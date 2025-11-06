---
date: 2025-11-06 15:30:00
title: "[ICPC南京 2023 M] Trapping Rain Water"
katex: true
tags:
- Algo
- C++
- DS
categories:
- XCPC
description: 线段树二分。
---

## 题目

[QOJ 7745](https://qoj.ac/problem/7745)。

## 思路

很容易想到把这个式子拆开，$\sum \min(f_i,g_i)-\sum a_i$，右边是容易维护的。

前缀最大值 $f_i\ge f_{i-1}$，后缀最大值 $g_i\le g_{i-1}$，所以 $f_i-g_i\ge f_{i-1}-g_{i-1}$。

又因为，$g_1=\max_{i=1}^n a_i\ge f_1=a_1$，因此 $f_1-g_1\le 0$，同理 $f_n-g_n\ge 0$。

因此一定存在一个 $1\le k\le n$，满足它是满足 $f_k-g_k\ge 0$ 并且 $k$ 最小的点。

那么，$i=1,2,\dots,k-1$ 时，都有 $f_i<g_i$；$i=k,k+1,\dots,n$ 时，都有 $g_i\le f_i$。

因此我们可以通过二分找到这个 $k$，并且 $\sum_{i=1}^n \min(f_i,g_i)=\sum_{i=1}^{k-1} f_i+\sum_{i=k}^{n} g_i$。

接下来的问题就是，我该如何维护 $f,g$ 两个数组？

观察每一次操作，它只会把某个 $a_x\gets a_x+v$，那么对于 $f_{x-1},f_{x-2},\dots,f_1$ 显然是没有影响的；同理，对于 $g_{x+1},g_{x+2},\dots, g_n$ 也是没有影响的。

那我们思考这样的问题，我什么时候会让 $f_x,f_{x+1},\dots,f_{n}$ 有影响？

如果 $a_x+v\le f_x$，本次修改对于 $f_x$ 显然是无效的；如果 $a_x+v>f_x$，那么本次修改仅会将 $i=x\sim (p-1)$ 的 $f_i\gets a_x+v$，其中 $p$ 是第一个满足 $a_p>a_x+v$ 的点，如果不存在这样的点，所有的后缀都会被修改为 $a_x+v$。对于 $g$ 是相似的。

于是，我们的问题就变成了，我会对 $a$ 进行单点加，并且我需要求出某个区间内第一个大于某个数 $(a_x+v)$ 的下标，并且找到这个下标后，对 $f$ 进行一个区间覆盖。这显然都是可以通过线段树完成的。

## 实现

如果你对找 $k$ 的那个步骤直接二分并且 `check` 的时候查询线段树，复杂度是 $O(n\log^2n)$，也是可以通过的。

但是实际上，可以通过线段树二分来找到那个 $k$，这样的复杂度是 $O(n\log n)$。

原理是，对于一个区间 $[l,r]$，因为 $f$ 是单调不减，$g$ 是单调不增，所以 $f_i-g_i$ 的最大值一定在 $i=r$ 时取到，并且满足 $f_r=\max f,g_r=\min g$，因此我们分别记录 $f,g$ 的区间最值，就可以做到在不记录 $f_i-g_i$ 的值的情况下也能对 $f_i-g_i$ 进行查询了。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 1e5+10;
#define ls (u<<1)
#define rs (u<<1|1)
#define Mid ((L+R) >> 1)

struct node {
    int sum, mx, mn, tag;
    void init(int val) {
        sum = mx = mn = val;
        tag = -1;
    }

    void spread(int k, int L, int R) {
        sum = k * (R-L+1);
        mx = k;
        mn = k;
        tag = k;
    }
};

node operator+(const node& l, const node& r) {
    return {l.sum + r.sum, max(l.mx, r.mx), min(l.mn, r.mn), -1};
}

struct Seg {
    int a[N];
    node dat[N<<2];

    void pushup(int u) {
        dat[u] = dat[ls] + dat[rs];
    }

    void pushdown(int u, int L, int R) {
        if (dat[u].tag == -1) return;
        dat[ls].spread(dat[u].tag, L, Mid);
        dat[rs].spread(dat[u].tag, Mid+1, R);
        dat[u].tag = -1;
    }

    void build(int u, int L, int R) {
        dat[u].init(-1);
        if (L == R) return dat[u].init(a[L]);
        build(ls, L, Mid);
        build(rs, Mid+1, R);
        pushup(u);
    }

    int querysum(int u, int l, int r, int L, int R) {
        if (l > r) return 0;
        if (l <= L && R <= r) return dat[u].sum;
        pushdown(u, L, R);
        int res = 0;
        if (l <= Mid) res += querysum(ls, l, r, L, Mid);
        if (Mid+1 <= r) res += querysum(rs, l, r, Mid+1, R);
        return res;
    }

    int query(int u, int p, int L, int R) {
        if (L == R) return dat[u].mx;
        pushdown(u, L, R);
        if (p <= Mid) return query(ls, p, L, Mid);
        else return query(rs, p, Mid+1, R);
    }

    void modify(int u, int l, int r, int k, int L, int R) {
        if (l <= L && R <= r) return dat[u].spread(k, L, R);
        pushdown(u, L, R);
        if (l <= Mid) modify(ls, l, r, k, L, Mid);
        if (Mid+1 <= r) modify(rs, l, r, k, Mid+1, R);
        pushup(u);
    }
} f, g;

int querymid(int u, int l, int r, int L, int R) {
    if (f.dat[u].mx - g.dat[u].mn < 0) return -1;
    if (L == R) return L;
    f.pushdown(u, L, R);
    g.pushdown(u, L, R);

    if (l > Mid) return querymid(rs, l, r, Mid+1, R);
    else if (Mid+1 > r) return querymid(ls, l, r, L, Mid);
    else {
        int res = querymid(ls, l, r, L, Mid);
        if (res != -1) return res;
        return querymid(rs, l, r, Mid+1, R);
    }
}


struct SegTA {
    int a[N];
    int mx[N<<2], leaf[N];
    
    void pushup(int u) {
        mx[u] = max(mx[ls], mx[rs]);
    }

    void build(int u, int L, int R) {
        if (L == R) return mx[u] = a[L], leaf[L] = u, void();
        build(ls, L, Mid);
        build(rs, Mid+1, R);
        pushup(u);
    }

    void modify(int p, int k) {
        int u = leaf[p];
        mx[u] += k;
        while (u >>= 1) pushup(u);
    }

    int findfir(int u, int l, int r, int k, int L, int R) {
        if (l > r) return -1;
        if (mx[u] < k) return -1;
        if (L == R) return L;

        if (l > Mid) return findfir(rs, l, r, k, Mid+1, R);
        else if (Mid+1 > r) return findfir(ls, l, r, k, L, Mid);
        else {
            int res = findfir(ls, l, r, k, L, Mid);
            if (res != -1) return res;
            return findfir(rs, l, r, k, Mid+1, R);
        }
    }

    int findlas(int u, int l, int r, int k, int L, int R) {
        if (l > r) return -1;
        if (mx[u] < k) return -1;
        if (L == R) return L;
        if (l > Mid) return findlas(rs, l, r, k, Mid+1, R);
        else if (Mid+1 > r) return findlas(ls, l, r, k, L, Mid);
        else {
            int res = findlas(rs, l, r, k, Mid+1, R);
            if (res != -1) return res;
            return findlas(ls, l, r, k, L, Mid);
        }
    }
} a;

void solve() {
    int n, q, suma = 0;
    cin >> n;
    for (int i = 1; i <= n; i++) cin >> a.a[i], suma += a.a[i];
    f.a[1] = a.a[1];
    for (int i = 2; i <= n; i++) f.a[i] = max(f.a[i-1], a.a[i]);
    g.a[n] = a.a[n];
    for (int i = n-1; i >= 1; i--) g.a[i] = max(g.a[i+1], a.a[i]);

    f.build(1, 1, n);
    g.build(1, 1, n);
    a.build(1, 1, n);

    cin >> q;
    while (q--) {
        int x, v;
        cin >> x >> v;
        suma += v;

        a.modify(x, v);
        int ax = a.mx[a.leaf[x]];

        if (ax > f.query(1, x, 1, n)) {
            int pos = a.findfir(1, x+1, n, ax+1, 1, n);
            if (pos == -1) pos = n+1;
            f.modify(1, x, pos-1, ax, 1, n);
        }

        if (ax > g.query(1, x, 1, n)) {
            int pos = a.findlas(1, 1, x-1, ax+1, 1, n);
            if (pos == -1) pos = 0;
            g.modify(1, pos+1, x, ax, 1, n);
        }

        int k = querymid(1, 1, n, 1, n);
        assert(k != -1);

        int ans = f.querysum(1, 1, k-1, 1, n);
        ans += g.querysum(1, k, n, 1, n);
        ans -= suma;
        cout << ans << '\n';
    }
}

signed main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    int T;
    cin >> T;
    while (T--) solve();
    return 0;
}
```

