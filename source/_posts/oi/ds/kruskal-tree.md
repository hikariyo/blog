---
date: 2023-12-22 23:03:00
updated: 2023-12-22 23:03:00
title: 数据结构 - Kruskal 重构树
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
description: 一句话说明，在 Kruskal 算法中的并查集合并两点所在连通块这一步，新建一个虚点并将左右连通块代表点经过虚点连接起来，用二叉树的结构记录并查集的合并顺序即加边顺序。
---

## 简介

Kruskal 重构树是用一颗二叉树来记录下 Kruskal 算法中合并两点所在连通块这一步骤的顺序，也就是加边顺序，新建一个虚点并将左右代表点依次与虚点连接，从而记录下来一次加边操作，最后 $LCA(a,b)$ 就记录 $(a,b)$ 间第一次加边时的信息。

## Peaks

> 给定一张 $n$ 个点、$m$ 条边的无向图，第 $i$ 个点的权值为 $a_i$，边有边权。
>
> 有 $q$ 组询问，每组询问给定三个整数 $u, x, k$，求从 $u$ 开始只经过权值 $\leq x$ 的边所能到达的权值第 $k$ 大的点的权值，如果不存在输出 $-1$。
>
> **本题强制在线。即：每次查询输入的是 $u', x', k'$，则 $u = (u' \operatorname{xor} \text{lastans}) \bmod n + 1$，$k$ 的解密方式与之相同，$x = x' \operatorname{xor} \text{lastans}$**。
>
> 对于 $100\%$ 的数据，$1 \leq n \leq 10^5$，$0 \leq m, q \leq 5 \times 10^5$，$1 \leq s, t \leq n$，$1 \leq a_i, w \leq 10^9$，$0 \leq u', x', k' < 2^{31}$。
>
> 题目链接：[P7834](https://www.luogu.com.cn/problem/P7834)。

建 Kruskal 重构树后，可以倍增找到 $x$ 能到达的最高点，查询最高点对应的 $[L,R]$ 内第 $k$ 大，自然想到主席树。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define Mid ((L+R) >> 1)
const int N = 200010, M = 500010, K = 20;
int n, m, q, fa[N][K], p[N], h[N], w[N], l[N], r[N], krt;
int rt[N], ls[N*K], rs[N*K], cnt[N*K], idx;
int ch[N][2], deg[N], ts;

void add(int a, int b) {
    deg[b]++;
    if (!ch[a][0]) ch[a][0] = b;
    else ch[a][1] = b;
}

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

struct Edge {
    int a, b, c;
    bool operator<(const Edge& e) const {
        return c < e.c;
    }
} edge[M];


int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

void insert(int& u, int v, int p, int L = 0, int R = 1e9) {
    u = ++idx;
    ls[u] = ls[v], rs[u] = rs[v], cnt[u] = cnt[v] + 1;
    if (L == R) return;
    if (p <= Mid) insert(ls[u], ls[v], p, L, Mid);
    else insert(rs[u], rs[v], p, Mid+1, R);
}

int query(int u, int v, int k, int L = 0, int R = 1e9) {
    if (L == R) return L;
    int c = cnt[rs[u]] - cnt[rs[v]];
    if (k <= c) return query(rs[u], rs[v], k, Mid+1, R);
    else return query(ls[u], ls[v], k-c, L, Mid);
}

void kruskal() {
    krt = n;
    sort(edge+1, edge+1+m);
    for (int i = 1; i <= n<<1; i++) p[i] = i;
    for (int i = 1; i <= m; i++) {
        int a = edge[i].a, b = edge[i].b, c = edge[i].c;
        int pa = find(a), pb = find(b);
        if (pa == pb) continue;
        p[pa] = p[pb] = ++krt, w[krt] = c;
        add(krt, pa), add(krt, pb);
    }
    w[0] = 1e18;
}

void dfs(int u, int f) {
    fa[u][0] = f;
    for (int k = 1; k < K; k++) fa[u][k] = fa[fa[u][k-1]][k-1];
    if (!ch[u][0]) {
        l[u] = r[u] = ++ts;
        insert(rt[ts], rt[ts-1], h[u]);
        return;
    }
    dfs(ch[u][0], u), dfs(ch[u][1], u);
    l[u] = min(l[ch[u][0]], l[ch[u][1]]);
    r[u] = max(r[ch[u][0]], r[ch[u][1]]);
}

signed main() {   
    read(n), read(m), read(q);
    for (int i = 1; i <= n; i++) read(h[i]);
    for (int i = 1, a, b, c; i <= m; i++) read(a), read(b), read(c), edge[i] = {a, b, c};
    kruskal();
    for (int i = 1; i <= krt; i++) if (!deg[i]) dfs(i, 0);

    int last = 0;
    for (int i = 1, u, x, k; i <= q; i++) {
        read(u), read(x), read(k);
        x ^= last;
        u = (u ^ last) % n + 1;
        k = (k ^ last) % n + 1;
        for (int k = K-1; k >= 0; k--) {
            if (w[fa[u][k]] <= x) u = fa[u][k];
        }
        int L = rt[l[u]-1], R = rt[r[u]];
        if (cnt[R] - cnt[L] < k) puts("-1"), last = 0;
        else printf("%lld\n", last = query(R, L, k));
    }

    return 0;
}
```

## [CF1706E] Qpwoeirut and Vertices

> 多测，给出 $n$ 个点， $m$ 条边的不带权连通无向图， $q$ 次询问至少要加完编号前多少的边，才能使得 $[l,r]$ 中的所有点两两连通。
>
> 数据范围：$\sum n\le10^5,\sum m,q\le 2\times10^5$。
>
> 题目链接：[CF1706E](https://codeforces.com/problemset/problem/1706/E)。

建立 Kruskal 重构树，由于这里边权是编号，所以直接边读边建即可，要使得 $[l,r]$ 所有点连通，就要使得 $dfn$ 最小和最大的 $[L,R]$ 连通，而恰好 $LCA(L,R)$ 就是答案。

证明：设存在 $dfn(x)<dfn(y)<dfn(z)$，那么它们的 $LCA$ 的子树一定包含 $x,y,z$ 并且 $y$ 在 $x,z$ 所包含的区间中间，所以只要包含 $x,z$ 就能包含 $y$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 200010, K = 20;
int n, m, q, ts, ch[N][2], dep[N], l[N], r[N], p[N], deg[N], dfn[N], pos[N], w[N];
int fa[K][N], st1[K][N], st2[K][N], lg2[N];

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

void dfs(int u) {
    dep[u] = dep[fa[0][u]] + 1;
    if (!ch[u][0]) return dfn[u] = l[u] = r[u] = ++ts, pos[ts] = u, void();
    dfs(ch[u][0]), dfs(ch[u][1]);
    l[u] = l[ch[u][0]], r[u] = r[ch[u][1]];
}

int lca(int a, int b) {
    if (dep[a] < dep[b]) swap(a, b);
    for (int k = K-1; k >= 0; k--) if (dep[fa[k][a]] >= dep[b]) a = fa[k][a];
    if (a == b) return a;
    for (int k = K-1; k >= 0; k--) if (fa[k][a] != fa[k][b]) a = fa[k][a], b = fa[k][b];
    return fa[0][a];
}

void solve() {
    ts = 0;
    read(n), read(m), read(q);
    for (int i = 1; i <= 2*n; i++) p[i] = i, ch[i][0] = ch[i][1] = deg[i] = w[i] = fa[0][i] = 0;
    
    // Kruskal Tree
    int krt = n;
    for (int i = 1, a, b, pa, pb; i <= m; i++) {
        read(a), read(b);
        pa = find(a), pb = find(b);
        if (pa == pb) continue;
        w[++krt] = i;
        p[pa] = p[pb] = krt;
        fa[0][pa] = fa[0][pb] = krt;
        ch[krt][0] = pa, ch[krt][1] = pb;
        ++deg[pa], ++deg[pb];
    }
    l[0] = -1, r[0] = N;

    for (int i = 1; i <= krt; i++) if (!deg[i]) dfs(i);
    for (int k = 1; k < K; k++)
        for (int i = 1; i <= krt; i++)
            fa[k][i] = fa[k-1][fa[k-1][i]];

    // Sprase Table
    for (int k = 0; k < K; k++) {
        for (int i = 1; i+(1<<k)-1 <= n; i++) {
            if (!k) st1[k][i] = st2[k][i] = dfn[i];
            else st1[k][i] = min(st1[k-1][i], st1[k-1][i+(1<<(k-1))]), st2[k][i] = max(st2[k-1][i], st2[k-1][i+(1<<(k-1))]);
        }
    }

    for (int i = 1, a, b, k, u, L, R; i <= q; i++) {
        read(a), read(b);
        k = lg2[b-a+1];
        L = min(st1[k][a], st1[k][b-(1<<k)+1]);
        R = max(st2[k][a], st2[k][b-(1<<k)+1]);
        printf("%d", w[lca(pos[L], pos[R])]);
        putchar(" \n"[i == q]);
    }
}

int main() {
    lg2[1] = 0;
    for (int i = 2; i < N; i++) lg2[i] = lg2[i>>1] + 1;

    int T;
    read(T);
    while (T--) solve();
    return 0;
}
```

## [CF1628E] Groceries in Meteor Town

> 给定 $n$ 个点的树，起初每个节点都是黑色。给 $q$ 个询问，三种类型：
>
> 1. 把编号为 $[l,r]$ 的点染成白色。
> 2. 把编号为 $[l,r]$ 的点染成黑色。
> 3. 询问从结点 $x$ 出发到达任意一个白色节点的简单路径上经过的边，最大可能的权值。不存在则输出 $-1$。
>
> 题目链接：[CF1628E](http://codeforces.com/problemset/problem/1628/E)。

由于本身就是一棵树，所以路径唯一，询问最大边权，所以升序排序，这样 $LCA(a,b)$ 表示的就是 $a,b$ 间最长边权。

求的是所有白点中和 $x$ 的 $LCA$ 深度最浅的那个，可以先求出所有白点的 $LCA$，然后 $LCA(u,x)$ 就是答案结点，因为一定存在一个白点 $v$ 使得 $LCA(v,x)=LCA(u,x)$，这个可以想两两求 $LCA$ 的过程，一定存在一个点它与别人的 $LCA$ 本来就是公共 $LCA$，所以这个做法是正确的。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((L+R) >> 1)
const int N = 600010, K = 20, INF = 0x3f3f3f3f;

int n, q, ts, dfn[N], pos[N], ch[N][2], deg[N], w[N], p[N], dep[N], fa[K][N];
int mn1[N<<1], mn2[N<<1], mx1[N<<1], mx2[N<<1], tag[N<<1];

struct Edge {
    int a, b, c;
    bool operator<(const Edge& e) const {
        return c < e.c;
    }
} edge[N];

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

void pushup(int u) {
    mn1[u] = min(mn1[u<<1], mn1[u<<1|1]);
    mx1[u] = max(mx1[u<<1], mx1[u<<1|1]);
}

void spread(int u, int k) {
    tag[u] = k;
    if (k == 1) mn1[u] = mn2[u], mx1[u] = mx2[u];
    else if (k == 2) mn1[u] = INF, mx1[u] = -INF;
}

void pushdown(int u) {
    if (tag[u]) spread(u<<1, tag[u]), spread(u<<1|1, tag[u]), tag[u] = 0;
}

void build(int u, int L, int R) {
    mn1[u] = INF, mx1[u] = -INF;
    if (L == R) return mn2[u] = mx2[u] = dfn[L], void();
    build(u<<1, L, Mid), build(u<<1|1, Mid+1, R);
    mn2[u] = min(mn2[u<<1], mn2[u<<1|1]), mx2[u] = max(mx2[u<<1], mx2[u<<1|1]);
}

void modify(int u, int l, int r, int k, int L, int R) {
    if (l <= L && R <= r) return spread(u, k);
    pushdown(u);
    if (l <= Mid) modify(u<<1, l, r, k, L, Mid);
    if (Mid+1 <= r) modify(u<<1|1, l, r, k, Mid+1, R);
    pushup(u);
}

void dfs(int u) {
    dep[u] = dep[fa[0][u]] + 1;
    if (!ch[u][0]) return dfn[u] = ++ts, pos[ts] = u, void();
    dfs(ch[u][0]), dfs(ch[u][1]);
}

int lca(int a, int b) {
    if (dep[a] < dep[b]) swap(a, b);
    for (int k = K-1; k >= 0; k--) if (dep[fa[k][a]] >= dep[b]) a = fa[k][a];
    if (a == b) return a;
    for (int k = K-1; k >= 0; k--) if (fa[k][a] != fa[k][b]) a = fa[k][a], b = fa[k][b];
    return fa[0][a];
}

int main() {
    read(n), read(q);
    for (int i = 1, a, b, c; i < n; i++) read(a), read(b), read(c), edge[i] = {a, b, c};
    sort(edge+1, edge+n);

    int krt = n;
    for (int i = 1; i <= 2*n; i++) p[i] = i;
    for (int i = 1; i < n; i++) {
        int a = edge[i].a, b = edge[i].b, c = edge[i].c;
        int pa = find(a), pb = find(b);
        if (pa == pb) continue;
        p[pa] = p[pb] = fa[0][pa] = fa[0][pb] = ++krt;
        w[krt] = c;
        ch[krt][0] = pa, ch[krt][1] = pb;
        ++deg[pa], ++deg[pb];
    }
    
    for (int i = 1; i <= krt; i++) if (!deg[i]) dfs(i);
    for (int k = 1; k < K; k++) for (int i = 1; i <= krt; i++) fa[k][i] = fa[k-1][fa[k-1][i]];
    build(1, 1, n);

    for (int i = 1, opt, x, l, r; i <= q; i++) {
        read(opt);
        if (opt != 3) read(l), read(r), modify(1, l, r, opt, 1, n);
        else {
            read(x);
            int L = mn1[1], R = mx1[1], ans = -1;
            if (L != INF && R != -INF) {
                int u = lca(pos[L], pos[R]);
                if (u != x) ans = w[lca(u, x)];
            }
            printf("%d\n", ans);
        }
    }
    
    return 0;
}
```

## [IOI2018] 狼人

> 在日本的茨城县内共有 $N$ 个城市和 $M$ 条道路。这些城市是根据人口数量的升序排列的，依次编号为 $0$ 到 $N - 1$。每条道路连接两个不同的城市，并且可以双向通行。由这些道路，你能从任意一个城市到另外任意一个城市。
>
> 你计划了 $Q$ 个行程，这些行程分别编号为 $0$ 至 $Q - 1$。第 $i(0 \leq i \leq Q - 1)$ 个行程是从城市 $S_i$ 到城市 $E_i$。
>
> 你是一个狼人。你有两种形态：**人形**和**狼形**。在每个行程开始的时候，你是人形。在每个行程结束的时候，你必须是狼形。在行程中，你必须要变身（从人形变成狼形）恰好一次，而且只能在某个城市内（包括可能是在 $S_i$ 或 $E_i$ 内）变身。
>
> 狼人的生活并不容易。当你是人形时，你必须避开人少的城市，而当你是狼形时，你必须避开人多的城市。对于每一次行程 $i(0 \leq i \leq Q - 1)$，都有两个阈值 $L_i$ 和 $R_i(0 \leq L_i \leq R_i \leq N - 1)$，用以表示哪些城市必须要避开。准确地说，当你是人形时，你必须避开城市 $0, 1, \ldots , L_i - 1$ ；而当你是狼形时，则必须避开城市 $R_i + 1, R_i + 2, \ldots , N - 1$。这就是说，在行程 $i$ 中，你必须在城市 $L_i, L_i + 1, \ldots , R_i$ 中的其中一个城市内变身。
>
> 你的任务是，对每一次行程，判定是否有可能在满足上述限制的前提下，由城市 $S_i$ 走到城市 $E_i$。你的路线可以有任意长度。
>
> - $2 \leq N \leq 200, 000$
> - $N - 1 \leq M \leq 400, 000$
> - $1 \leq Q \leq 200, 000$
>
> 题目链接：[P4899](https://www.luogu.com.cn/problem/P4899)。

首先下标全部加一，题目转化为由 $L\sim N$ 走到 $1\sim R$ 是否存在交集。

首先走 $L\sim N$ 时需要满足一条边两边都是 $\ge L$ 才能走，所以用最小值做边权，求最大 Kruskal 重构树，越往上走越小。反之亦然。

现在有了两个区间 $[l_1,r_1],[l_2,r_2]$，需要判断它们是否存在交集，那么可以转化为 [CF1093E](https://codeforces.com/problemset/problem/1093/E)，构造逆映射，这里是静态的所以直接主席树就好了。

如果搞不清楚代码逻辑这样想：每次获取到 $f_1(l_1),f_1(l_2)$ 所以主席树做的就是先用 $f_1^{-1}(f_1(i))=i$ 还原出来，然后在用 $f_2(i)$ 映射到第二个对应的 $dfn$ 上，通过查询区间和判断是否存在交集。  

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((1LL*L+R) >> 1)
const int N = 400010, K = 20;
int n, m, q;
int rt[N], ls[N*K], rs[N*K], cnt[N*K], idx;

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

struct Edge {
    int a, b, c;
} edge[N];

struct Kruskal {
    int ch[N][2], fa[N][K], deg[N], w[N], l[N], r[N], dfn[N], p[N], rev[N], ts;

    int find(int x) {
        if (p[x] != x) p[x] = find(p[x]);
        return p[x];
    }

    void dfs(int u) {
        for (int k = 1; k < K; k++) fa[u][k] = fa[fa[u][k-1]][k-1];
        if (!ch[u][0]) {
            dfn[u] = ++ts;
            l[u] = r[u] = ts;
            return;
        }
        dfs(ch[u][0]), dfs(ch[u][1]);
        l[u] = l[ch[u][0]], r[u] = r[ch[u][1]];
    }

    void build(int w0) {
        int rt = n;

        for (int i = 1; i <= n<<1; i++) p[i] = i;
        for (int i = 1; i <= m; i++) {
            int a = edge[i].a, b = edge[i].b, c = edge[i].c;
            int pa = find(a), pb = find(b);
            if (pa == pb) continue;
            p[pa] = p[pb] = ++rt;
            w[rt] = c;
            ch[rt][0] = pa, ch[rt][1] = pb, fa[pa][0] = fa[pb][0] = rt;
            ++deg[pa], ++deg[pb];
        }

        for (int i = 1; i <= rt; i++) if (!deg[i]) dfs(i);
        w[0] = w0;
        for (int i = 1; i <= n; i++) rev[dfn[i]] = i;
    }

    void queryr(int& a, int& b, int u, int v) {
        for (int k = K-1; k >= 0; k--) {
            if (w[fa[u][k]] <= v) u = fa[u][k];
        }
        a = l[u], b = r[u];
    }

    void queryl(int& a, int& b, int u, int v) {
        for (int k = K-1; k >= 0; k--) {
            if (w[fa[u][k]] >= v) u = fa[u][k];
        }
        a = l[u], b = r[u];
    }
} k2, k1;

void insert(int& u, int v, int p, int L, int R) {
    u = ++idx;
    ls[u] = ls[v], rs[u] = rs[v], cnt[u] = cnt[v] + 1;
    if (L == R) return;
    if (p <= Mid) insert(ls[u], ls[v], p, L, Mid);
    else insert(rs[u], rs[v], p, Mid+1, R);
}

int query(int u, int v, int l, int r, int L, int R) {
    if (l <= L && R <= r) return cnt[u] - cnt[v];
    int res = 0;
    if (l <= Mid) res = query(ls[u], ls[v], l, r, L, Mid);
    if (Mid+1 <= r) res += query(rs[u], rs[v], l, r, Mid+1, R);
    return res;
}

int main() {   
    read(n), read(m), read(q);
    for (int i = 1, a, b; i <= m; i++) read(a), read(b), ++a, ++b, edge[i] = {a, b};

    for (int i = 1; i <= m; i++) edge[i].c = min(edge[i].a, edge[i].b);
    sort(edge+1, edge+1+m, [](const Edge& a, const Edge& b){ return a.c > b.c; });
    k1.build(0);

    for (int i = 1; i <= m; i++) edge[i].c = max(edge[i].a, edge[i].b);
    sort(edge+1, edge+1+m, [](const Edge& a, const Edge& b){ return a.c < b.c; });
    k2.build(N);

    for (int i = 1; i <= n; i++) insert(rt[i], rt[i-1], k2.dfn[k1.rev[i]], 1, n);

    for (int i = 1, s, e, l, r; i <= q; i++) {
        read(s), read(e), read(l), read(r);
        ++s, ++e, ++l, ++r;
        int l1, r1, l2, r2;
        k1.queryl(l1, r1, s, l);
        k2.queryr(l2, r2, e, r);
        if (query(rt[r1], rt[l1-1], l2, r2, 1, n)) puts("1");
        else puts("0");
    }
    return 0;
}
```

