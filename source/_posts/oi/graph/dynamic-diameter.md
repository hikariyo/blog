---
date: 2024-05-28 16:54:00
title: 图论 - 动态修改非负边权树的直径问题
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
description: 用线段树维护欧拉序可以做到 O(nlog(n)) 复杂度动态带修改非负边权的求出一个树的直径。若边权存在负数，应当使用动态 DP 等方法解决。
---

## [CF1192B] Dynamic Diameter

> 给 $n$ 个点的无向树，$q$ 次操作，每次修改一条边权（不超过 $w-1$），修改后输出直径大小，强制在线。
>
> 数据范围：$2\le n\le 10^5,1\le q\le 10^5,1\le w\le 2\times 10^{13}$​。
>
> 题目链接：[CF1192B](https://codeforces.com/contest/1192/problem/B)。

首先介绍欧拉序：第一次访问以及每次访问完一个子树时，将当前结点加入序列，所以大小是 $1+\sum \text{deg}_i =2n-1$​，根节点额外加一。

设根为 $1$，这个树是外向树，将边权化为点权，其中 $1$ 的点权是 $0$。设 $d_i$ 表示节点 $i$ 到 $1$ 路径上的点权之和，也就是到 $1$ 的所有边权之和。

然后考虑对于 $a,b$ 两点，如何求出 $\text{lca}(a,b)$。设 $f_a$ 为 $a$ 在欧拉序中第一次出现的编号，不妨设 $f_a<f_b$，那么 $b$ 不可能是 $a$ 的祖先，于是分两种情况：

1. $a$ 为 $b$ 的祖先，一定有 $f_a\sim f_b$ 为 $a\sim b$ 路径上所有的点，那么 $f_a$ 就是 $f_a\sim f_b$ 中 $d$ 最小的那个。
2. $a$ 与 $b$ 在 $\text{lca}(a,b)$ 不同的子树中，那么 $\text{lca}(a,b)$ 一定是 $f_a\sim f_b$ 中 $d$ 最小的那个。 

因此就可以通过下式计算 $a,b$ 的路径长度：
$$
\text{dist}(a,b)=d_a+d_b-2\min_{f_a\le f_c\le f_b} d_c
$$
因为树的直径就是 $\max_{a,b}\text{dist}(a,b)$，所以就有：
$$
\text{diam}=\max_{f_a\le f_c\le f_b}\{d_a+d_b-2d_c\}
$$
当改变结点 $u$ 的权值使其增加 $\Delta$ 时，会将 $u$ 子树中所有 $d_i\gets d_i+\Delta$。

设 $g_u$ 为 $u$ 在欧拉序中出现的最后位置，所以将 $f_u\sim g_u$ 区间加 $\Delta$ 即可。

下面考虑如何合并两个线段树的结点，下面直接用 $u$ 代替 $f_u$​。

设线段树中 $\text{mn}$ 为 $d_u$ 的最小值，$\text{mx}$ 为 $d_u$ 的最大值，$\text{au}$ 为 $d_a-2d_u$ 的最大值（保证 $a\le u$），$\text{ua}$ 为 $d_a-2d_u$ 的最大值（保证 $u\le a$），$\text{diam}$ 为当前的直径。

设 $a,b,u$ 三点中 $u=\text{lca}(a,b)$，那么一定有 $a\le u\le b$，有下面三种情况：

1. 若 $a,u,b$ 均在左或者右，有 $\text{diam}\gets \max\{\text{diam}_l,\text{diam}_r\}$。
2. 若 $a,u$ 在左，$b$​ 在右，有 $\text{diam}\gets \text{au}_l+\text{mx}_r$。
3. 若 $a$ 在左，$u,b$ 在右，有 $\text{diam}\gets \text{mx}_l + \text{ua}_r$。

对于 $\text{au}$ 和 $\text{ua}$ 的更新也类似，由于 $a=u$ 是合法的，所以初始设成 $-d$ 即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define ls (u<<1)
#define rs (u<<1|1)
#define Mid ((L+R) >> 1)
const int N = 100010;

struct Edge {
    int to, nxt, w;
} e[N<<1];
int n, q, W, f[N], g[N], d[N], h[N], E[N], p[N], seq[N<<1], idx = 2, sidx;
int mn[N<<3], mx[N<<3], au[N<<3], ua[N<<3], diam[N<<3], tag[N<<3];

void add(int a, int b, int c) {
    e[idx] = {b, h[a], c}, h[a] = idx++;
}

void dfs(int u) {
    seq[f[u] = g[u] = ++sidx] = u;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (f[to]) continue;
        p[i >> 1] = to;
        d[to] = d[u] + e[i].w;
        dfs(to);
        seq[g[u] = ++sidx] = u;
    }
}

void pushup(int u) {
    diam[u] = max(max(diam[ls], diam[rs]), max(au[ls] + mx[rs], mx[ls] + ua[rs]));
    au[u] = max(max(au[ls], au[rs]), mx[ls] - 2 * mn[rs]);
    ua[u] = max(max(ua[ls], ua[rs]), mx[rs] - 2 * mn[ls]);
    mx[u] = max(mx[ls], mx[rs]);
    mn[u] = min(mn[ls], mn[rs]);
}

void spread(int u, int k) {
    mx[u] += k, mn[u] += k, tag[u] += k, au[u] -= k, ua[u] -= k;
}

void pushdown(int u) {
    if (!tag[u]) return;
    spread(ls, tag[u]);
    spread(rs, tag[u]);
    tag[u] = 0;
}

void build(int u, int L, int R) {
    if (L == R) {
        mn[u] = mx[u] = d[seq[L]];
        au[u] = ua[u] = -d[seq[L]];
        diam[u] = 0;
        return;
    }
    build(ls, L, Mid), build(rs, Mid+1, R);
    pushup(u);
}

void modify(int u, int l, int r, int k, int L, int R) {
    if (l <= L && R <= r) return spread(u, k);
    pushdown(u);
    if (l <= Mid) modify(ls, l, r, k, L, Mid);
    if (Mid+1 <= r) modify(rs, l, r, k, Mid+1, R);
    pushup(u);
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    cin >> n >> q >> W;
    for (int i = 1, a, b, c; i < n; i++) {
        cin >> a >> b >> c;
        E[i] = c;
        add(a, b, c), add(b, a, c);
    }
    dfs(1);
    build(1, 1, sidx);
    for (int i = 1, las = 0, dd, ee; i <= q; i++) {
        cin >> dd >> ee;
        dd = (dd + las) % (n-1) + 1;
        ee = (ee + las) % W;
        modify(1, f[p[dd]], g[p[dd]], ee - E[dd], 1, sidx);
        E[dd] = ee;
        cout << (las = diam[1]) << '\n';
    }
    return 0;
}
```

## [SDCPC2024 L] 路径的交

> 给定 $n$ 结点 $(n-1)$ 边 $(u_i,v_i,w_i)$ 的树。处理 $q$ 次询问 $a_i,b_i,k_i$：首先将第 $a_i$ 条边权临时改为 $b_i$，随后选出 $k_i$ 个端点互不相同的路径，最大化它们的交的总权值。处理完一个询问后，需要将边权恢复原状。
>
> 数据范围：$1\le n,q\le 5\times 10^5,1\le w_i\le 10^9,1\le k_i\le \lfloor\frac{n}{2}\rfloor$。

当 $k$ 固定时，一条边可以对答案作出贡献，当且仅当它两边两个连通块的大小都 $\ge k$​。

若两条不同的边符合条件且不相连，显然它们的祖先边一定也符合条件，所以所有满足条件的边构成一个连通块，即一个子树，答案即这个子树的直径。

当 $k$ 增加时，边是减小的，所以将询问离线下来排序，问题就变成了不断删除某些边，并临时改变某些边的权值，求树的直径。

对于删除，将边权置为 $0$ 即可。注意临时改动的边可能已经被删掉，此时应不做任何改动。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define Mid ((L+R) >> 1)
#define ls (u<<1)
#define rs (u<<1|1)
const int N = 500010;

struct Edge {
    int to, nxt, w;
} e[N<<1];

struct Query {
    int id, a, b, k;
    bool operator<(const Query& q) const {
        return k < q.k;
    }
} qry[N];

struct Point {
    int w, v;
    bool operator<(const Point& p) const {
        return w < p.w;
    }
} points[N];

int n, q, h[N], w[N], d[N], f[N], g[N], p[N], sz[N], ans[N], seq[N<<1], idx = 2, sidx;
int mn[N<<3], mx[N<<3], au[N<<3], ua[N<<3], diam[N<<3], tag[N<<3];

void add(int a, int b, int c) {
    e[idx] = {b, h[a], c}, h[a] = idx++;
}

void dfs(int u) {
    sz[u] = 1;
    seq[f[u] = g[u] = ++sidx] = u;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (f[to]) continue;
        p[i >> 1] = to;
        d[to] = d[u] + e[i].w;
        w[to] = e[i].w;
        dfs(to);
        sz[u] += sz[to];
        seq[g[u] = ++sidx] = u;
    }
    points[u] = {min(sz[u], n - sz[u]), u};
}

void pushup(int u) {
    diam[u] = max(max(diam[ls], diam[rs]), max(au[ls] + mx[rs], mx[ls] + ua[rs]));
    au[u] = max(max(au[ls], au[rs]), mx[ls] - 2 * mn[rs]);
    ua[u] = max(max(ua[ls], ua[rs]), mx[rs] - 2 * mn[ls]);
    mx[u] = max(mx[ls], mx[rs]);
    mn[u] = min(mn[ls], mn[rs]);
}

void spread(int u, int k) {
    mx[u] += k, mn[u] += k, tag[u] += k, au[u] -= k, ua[u] -= k;
}

void pushdown(int u) {
    if (!tag[u]) return;
    spread(ls, tag[u]);
    spread(rs, tag[u]);
    tag[u] = 0;
}

void build(int u, int L, int R) {
    if (L == R) {
        mn[u] = mx[u] = d[seq[L]];
        au[u] = ua[u] = -d[seq[L]];
        diam[u] = 0;
        return;
    }
    build(ls, L, Mid), build(rs, Mid+1, R);
    pushup(u);
}

void modify(int u, int l, int r, int k, int L, int R) {
    if (l <= L && R <= r) return spread(u, k);
    pushdown(u);
    if (l <= Mid) modify(ls, l, r, k, L, Mid);
    if (Mid+1 <= r) modify(rs, l, r, k, Mid+1, R);
    pushup(u);
}

void change(int u, int k) {
    if (w[u] == 0) return;
    modify(1, f[u], g[u], k - w[u], 1, sidx);
    w[u] = k;
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    cin >> n >> q;

    for (int i = 1, a, b, c; i < n; i++) {
        cin >> a >> b >> c;
        add(a, b, c), add(b, a, c);
    }

    for (int i = 1, a, b, k; i <= q; i++) {
        cin >> a >> b >> k;
        qry[i] = {i, a, b, k};
    }
    dfs(1);
    build(1, 1, sidx);

    sort(qry+1, qry+1+q);
    sort(points+1, points+1+n);
    for (int i = 1, j = 1, a, b, k; i <= q; i++) {
        a = qry[i].a, b = qry[i].b, k = qry[i].k;
        while (points[j].w < k && j <= n) change(points[j++].v, 0);
        int old = w[p[a]];
        change(p[a], b);
        ans[qry[i].id] = diam[1];
        change(p[a], old);
    }

    for (int i = 1; i <= q; i++) cout << ans[i] << '\n';
    return 0;
}
```

