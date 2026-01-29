---
date: 2025-10-31 16:05:00
updated: 2025-10-31 16:05:00
title: "[ICPC杭州 2024 B] Barkley III"
katex: true
tags:
- Algo
- C++
- DS
- DP
categories:
- XCPC
description: 线段树维护 DP 类似物。
---

## 题目

[QOJ 9727](https://qoj.ac/problem/9727)。

## 思路

对于查询二进制数的最大值，按照常规思路从高到底逐位考虑，显然是到某一位时，$[l,r]$ 的元素中只有一个 $0$ 的时候，把它去掉，答案最大。

如果所有的位置都是要么 $\ge 2$ 个 $0$，要么没有 $0$，那么无论去掉哪一个数字都没办法使得结果更大。

如果有多个位置只有 $1$ 个 $0$，去掉低位的答案是不优于去掉高位的答案的，因为 $2^k>2^{k-1}+2^{k-2}+\cdots+2^0$。

因此，现在我们就需要对于每一个区间，维护是否只有一个 $0$。

开一个长度 $64$ 的 `int` 数组在本题的数据范围是无法接受的，因此我们只能退而求其次，考虑对每一位的状态用几个 $0/1$ 的变量维护，这样最终就可以用几个 `ull` 来维护所有的状态了。

具体地说，我们设计三个变量 $f,g,h$ 分别表示当前位有 $0$ 个、$1$ 个、$\ge 2$ 个 $0$，显然它们是互斥的，即只能有一个人是 $1$。

设目前的节点是 $u$，左右儿子分别为 $ls,rs$，那么转移：

1. 对于 $f$，显然是 $f_{ls}$ 与 $f_{rs}$ 同时为 $1$ 时，$f_u=1$；否则 $f_u=0$。
2. 对于 $g$，当 $g_{ls}$ 与 $g_{rs}$ 只有一个为 $1$，并且 $g_x=0$ 的那个人必须 $f_x=1$。
3. 对于 $h$，只要 $h_{ls}$ 或 $h_{rs}$ 为 $1$，或者 $g_{ls}=g_{rs}=1$ 时即可。

上面是对于某一位，由于位运算每一位都是独立的，所以合起来是下面这样：

```cpp
struct node {
    int f, g, h;
    void set(int a) { h = 0; g = ~a; f = a; }
} dat[N<<2];

node operator+(const node& l, const node& r) {
    return {l.f & r.f, (l.f ^ r.f) & (l.g ^ r.g), l.h | r.h | (l.g & r.g)};
}
```

还有一个问题，区间修改按位与该怎么办？

如果看这个式子，那大概是这个题不可能做出来的，需要考虑**具体含义**。

我们先认为是对于某一位的与，如果与 $1$，显然 $f,g,h$ 均不变；如果与 $0$，那么：

1. 对于 $f$，置为 $0$；
2. 对于 $g$，如果只有一个元素，置为 $1$；否则为 $0$。
3. 对于 $h$，如果只有一个元素，置为 $0$；否则为 $1$。

所以这个线段树的修改函数应当是：

```cpp
void spread(int u, int k, int L, int R) {
    tag[u] &= k;
    dat[u].f &= k;
    if (L == R) {
        // 1 element
        assert(dat[u].h == 0);
        dat[u].g |= ~k;
    }
    else {
        // at least 2 elements
        dat[u].h |= ~k;
        dat[u].g &= k;
    }
}
```

还有一个问题，我们如何找到那个 $0$ 的位置？

我们知道，如果某一位只有一个 $0$，那么该位的 $g=1$，否则 $g=0$。

线段树区间查询的时候会查询 $O(\log n)$ 个小区间，因为合并之后 $g=1$，所以一定有且只有一个区间的 $g=1$。

当我们找到这个区间之后，它的两个子区间必然也是有且只有一个子区间的 $g=1$。

如果从高到底第 $i$ 位是首个 $g=1$ 的位，那么令 $k=2^{63-i}$ 作为一个 mask，查询会是这样的：

```cpp
int querypos(int u, int l, int r, int k, int L, int R) {
    if (l <= L && R <= r && (dat[u].g & k) == 0) return 0;
    if (L == R) return L;
    pushdown(u, L, R);
    int res = 0;
    if (l <= Mid) res += querypos(ls, l, r, k, L, Mid);
    if (Mid+1 <= r) res += querypos(rs, l, r, k, Mid+1, R);
    return res;
}
```

注意第一行代码包含了两种情况，要么是划分的 $O(\log n)$ 的那些子区间本身 $g=0$，要么是对 $g=1$ 的区间二分时 $g=0$，总之这些都应该被忽略掉。

## 实现

上面重点阐述了一些对于我比较难理解的部分的代码。

还有一个细节，$-1$ 是二进制位全是 $1$ 的数，所以将它作为一个 $\operatorname{and}$ 运算的初始元素是合适的。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int unsigned long long
#define Mid ((L+R) >> 1)
#define ls (u<<1)
#define rs (u<<1|1)
const int N = 1e6+10;

int tag[N<<2];
int a[N];

struct node {
    int f, g, h;
    void set(int a) {
        h = 0;
        g = ~a;
        f = a;
    }
} dat[N<<2];

node operator+(const node& l, const node& r) {
    return {l.f & r.f, (l.f ^ r.f) & (l.g ^ r.g), l.h | r.h | (l.g & r.g)};
}

void pushup(int u) {
    dat[u] = dat[ls] + dat[rs];
}

void spread(int u, int k, int L, int R) {
    tag[u] &= k;
    dat[u].f &= k;
    if (L == R) {
        // 1 element
        assert(dat[u].h == 0);
        dat[u].g |= ~k;
    }
    else {
        // at least 2 elements
        dat[u].h |= ~k;
        dat[u].g &= k;
    }
}

void pushdown(int u, int L, int R) {
    spread(ls, tag[u], L, Mid);
    spread(rs, tag[u], Mid+1, R);
    tag[u] = -1;
}

void build(int u, int L, int R) {
    tag[u] = -1;
    if (L == R) return dat[u].set(a[L]), void();
    build(ls, L, Mid);
    build(rs, Mid+1, R);
    pushup(u);
}

void modify(int u, int l, int r, int k, int L, int R) {
    if (l <= L && R <= r) return spread(u, k, L, R);
    pushdown(u, L, R);
    if (l <= Mid) modify(ls, l, r, k, L, Mid);
    if (Mid+1 <= r) modify(rs, l, r, k, Mid+1, R);
    pushup(u);
}

void modify(int u, int p, int k, int L, int R) {
    if (L == R) return dat[u].set(k), void();
    pushdown(u, L, R);
    if (p <= Mid) modify(ls, p, k, L, Mid);
    else modify(rs, p, k, Mid+1, R);
    pushup(u);
}

node query(int u, int l, int r, int L, int R) {
    if (l > r) return {-1ull, -1ull, -1ull};  // only when op = 3 will this case be entered
    if (l <= L && R <= r) return dat[u];
    pushdown(u, L, R);
    if (l > Mid) return query(rs, l, r, Mid+1, R);
    else if (r < Mid+1) return query(ls, l, r, L, Mid);
    else return query(ls, l, r, L, Mid) + query(rs, l, r, Mid+1, R);
}

int querypos(int u, int l, int r, int k, int L, int R) {
    if (l <= L && R <= r && (dat[u].g & k) == 0) return 0;
    if (L == R) return L;
    pushdown(u, L, R);
    int res = 0;
    if (l <= Mid) res += querypos(ls, l, r, k, L, Mid);
    if (Mid+1 <= r) res += querypos(rs, l, r, k, Mid+1, R);
    return res;
}

signed main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    int n, q;
    cin >> n >> q;
    for (int i = 1; i <= n; i++) cin >> a[i];
    build(1, 1, n);
    while (q--) {
        int op, l, r, x, s;
        cin >> op;
        if (op == 1) {
            cin >> l >> r >> x;
            modify(1, l, r, x, 1, n);
        }
        else if (op == 2) {
            cin >> s >> x;
            modify(1, s, x, 1, n);
        }
        else {
            cin >> l >> r;
            node nd = query(1, l, r, 1, n);
            int g = nd.g;
            if (g == 0) cout << nd.f << '\n';
            else {
                int k = 1ull << (63 - __builtin_clzll(nd.g));
                int pos = querypos(1, l, r, k, 1, n);
                cout << (query(1, l, pos-1, 1, n).f & query(1, pos+1, r, 1, n).f) << '\n';
            }
        }
    }
    return 0;
}
```

