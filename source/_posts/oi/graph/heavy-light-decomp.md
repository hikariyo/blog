---
date: 2023-10-17 20:59:00
updated: 2023-10-17 20:59:00
title: 图论 - 树链剖分
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
description: 树链剖分是一种将树上路径转化为对数级别的连续区间的技术，可以把树上询问转化为区间询问，然后用线段树等数据结构维护信息。
---

## 简介

树链剖分通常是用来解决对树上路径进行修改查询的这一类问题，它可以把树上路径转化为 $O(\log n)$ 段序列的连续区间，通常使用线段树维护这些区间，所以单次询问的复杂度会是 $O(\log^2 n)$。

接下来介绍概念，对于结点 $u$ 和它的子结点 $v$：

+ 如果 $sz(v)$ 是所有子结点中最大的，称 $v$ 为重儿子，否则为轻儿子。若有多个相同任选一个即可。
+ 若 $v$ 为重儿子，那么边 $(u,v)$ 为重边，否则为轻边。

对于每个结点 $u$ 都记录一个 $top(u)$，代表 $u$ 所在重链中深度最小的那个结点，所以我们可以以 $O(1)$ 从重链上任意一个结点跳转到链顶。

接下来简单证明一下为什么可以分成 $O(\log n)$ 条连续路径：

+ 如果 $(u,v)$ 为重边，可以由上面说的那样以 $O(1)$ 的时间跳转到链顶。
+ 如果 $(u,v)$ 为轻边，即 $v$ 为轻儿子，那么一定有 $sz(v)\le \frac{1}{2}\sum_{v_i\in sons(u)} sz(v_i)$，否则 $v$ 就一定是重儿子了。那么每次经过一条轻边，点的数量都会减少一半，所以会经过 $O(\log n)$ 条轻边，将这条路径分割成了 $O(\log n)$ 条连续区间，注意一条重链上的区间是连续的。

进行树剖时，会给每个点用 $dfs$ 序重新标号，每次优先走重儿子，所以这样整条重链上的标号都是连续的，可以用合适的数据结构来维护这些区间信息，通常是使用线段树。

## [模板] 树链剖分

> 给定一棵 $n$ 个节点的树，初始时该树的根为 $1$ 号节点，每个节点有一个给定的权值。下面依次进行 $m$ 个操作，操作分为如下五种类型：
>
> - 换根：将一个指定的节点设置为树的新根。
> - 修改路径权值：给定两个节点，将这两个节点间路径上的所有节点权值（含这两个节点）增加一个给定的值。
> - 修改子树权值：给定一个节点，将以该节点为根的子树内的所有节点权值增加一个给定的值。
> - 询问路径：询问某条路径上节点的权值和。
> - 询问子树：询问某个子树内节点的权值和。
>
> **输入格式**
>
> 第一行一个整数 $n$，表示节点的个数。
>
> 第二行 $n$ 个整数表示第 $i$ 个节点的初始权值 $w_i$。
>
> 第三行 $n-1$ 个整数，表示 $i+1$ 号节点的父节点编号 $f_{i+1}\ (1 \le f_{i+1} \le n)$。
>
> 第四行一个整数 $m$，表示操作个数。
>
> 接下来 $m$ 行，每行第一个整数表示操作类型编号：$(1 \le u, v \le n)$
>
> - 若类型为 $1$，则接下来一个整数 $u$，表示新根的编号。
> - 若类型为 $2$，则接下来三个整数 $u,v,k$，分别表示路径两端的节点编号以及增加的权值。
> - 若类型为 $3$，则接下来两个整数 $u,k$，分别表示子树根节点编号以及增加的权值。
> - 若类型为 $4$，则接下来两个整数 $u,v$，表示路径两端的节点编号。
> - 若类型为 $5$，则接下来一个整数 $u$，表示子树根节点编号。
>
> 题目链接：[LOJ 139](https://loj.ac/p/139)。

我们使用两个 $dfs$ 预处理树上信息，第一个处理出每个点的深度 $dep$，子树大小 $sz$，重儿子 $son$；第二个处理 $dfs$ 序 $dfn$，重链头标号 $top$。

每次处理 $u,v$ 路径上的点时，每次挑 $top(u),top(v)$ 较深的那个跳上去，直到它们跳到同一条重链上时最后一段路径就是 $[dfn(u),dfn(v)]$ 之间的区间。

对于 $u$ 的子树，区间为 $[dfn(u),dfn(u)+sz(u)-1]$，这个在前面的树形背包中也有所涉及。

对于换根操作，路径是不关心根节点是哪里的，只有询问子树的时候才需要关心根节点是谁：

+ 若 $u=rt$，那么操作的是整棵树。
+ 若 $LCA(u,rt)=u$ 且 $u\ne rt$，那么操作整棵树后撤销对于 $rt\to u$ 路径上的倒数第二个点的子树的操作。
+ 若 $LCA(u,rt)\ne u$，那么 $u$ 的子树不变。

这与前文提到的换根 LCA 操作比较类似。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;
typedef long long LL;
int n, q, h[N], w[N], idx = 1;
struct Edge {
    int to, nxt;
} e[N];
int fa[N], son[N], sz[N], dep[N], top[N], dfn[N], dfnw[N], ts;
int rt = 1;

struct Node {
    int l, r;
    LL sum, add;
} tr[N<<2];

void pushup(int u) {
    tr[u].sum = tr[u<<1].sum + tr[u<<1|1].sum;
}

void spread(int u, LL add) {
    tr[u].add += add;
    tr[u].sum += add * (tr[u].r - tr[u].l + 1);
}

void pushdown(int u) {
    if (tr[u].add) {
        spread(u<<1, tr[u].add);
        spread(u<<1|1, tr[u].add);
        tr[u].add = 0;
    }
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) return tr[u].sum = dfnw[l], void();
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
    pushup(u);
}

LL query(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u].sum;
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    LL res = 0;
    if (l <= mid) res += query(u<<1, l, r);
    if (mid+1 <= r) res += query(u<<1|1, l, r);
    return res;
}

void update(int u, int l, int r, int k) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u, k);
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) update(u<<1, l, r, k);
    if (mid+1 <= r) update(u<<1|1, l, r, k);
    pushup(u);
}

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void dfs1(int u) {
    dep[u] = dep[fa[u]] + 1, sz[u] = 1;

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        dfs1(to);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    dfn[u] = ++ts, dfnw[ts] = w[u], top[u] = t;
    
    if (!son[u]) return;
    dfs2(son[u], t);

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == son[u]) continue;
        dfs2(to, to);
    }
}

void update_path(int u, int v, int k) {
    while (top[u] != top[v]) {
        if (dep[top[u]] < dep[top[v]]) swap(u, v);
        update(1, dfn[top[u]], dfn[u], k);
        u = fa[top[u]];
    }
    if (dfn[u] > dfn[v]) swap(u, v);
    update(1, dfn[u], dfn[v], k);
}

LL query_path(int u, int v) {
    LL res = 0;
    while (top[u] != top[v]) {
        if (dep[top[u]] < dep[top[v]]) swap(u, v);
        res += query(1, dfn[top[u]], dfn[u]);
        u = fa[top[u]];
    }
    if (dfn[u] > dfn[v]) swap(u, v);
    res += query(1, dfn[u], dfn[v]);
    return res;
}

int lca(int u, int v) {
    while (top[u] != top[v]) {
        if (dep[top[u]] < dep[top[v]]) swap(u, v);
        u = fa[top[u]];
    }
    return dep[u] < dep[v] ? u : v;
}

// find u -> p -> v, where p near v.
int find(int u, int v) {
    while (top[u] != top[v]) {
        if (fa[top[u]] == v) return top[u];
        u = fa[top[u]];
    }
    return son[v];
}

LL query_tree(int u) {
    if (lca(u, rt) != u) return query(1, dfn[u], dfn[u]+sz[u]-1);
    LL res = query(1, 1, n);
    if (rt == u) return res;

    int v = find(rt, u);
    return res - query(1, dfn[v], dfn[v]+sz[v]-1);
}

void update_tree(int u, int k) {
    if (lca(u, rt) != u) return update(1, dfn[u], dfn[u]+sz[u]-1, k);

    update(1, 1, n, k);
    if (rt == u) return;
    int v = find(rt, u);
    update(1, dfn[v], dfn[v]+sz[v]-1, -k);
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &w[i]);
    for (int i = 2; i <= n; i++) scanf("%d", &fa[i]), add(fa[i], i);
    dfs1(1);
    dfs2(1, 1);
    build(1, 1, n);

    scanf("%d", &q);
    while (q--) {
        int tp, u, v, k;
        scanf("%d", &tp);
        if (tp == 1) scanf("%d", &rt);
        else if (tp == 2) scanf("%d%d%d", &u, &v, &k), update_path(u, v, k);
        else if (tp == 3) scanf("%d%d", &u, &k), update_tree(u, k);
        else if (tp == 4) scanf("%d%d", &u, &v), printf("%lld\n", query_path(u, v));
        else scanf("%d", &u), printf("%lld\n", query_tree(u));
    }
    return 0;
}
```

## [ZJOI2008] 树的统计

> 一棵树上有 $n$ 个节点，编号分别为 $1$ 到 $n$，每个节点都有一个权值 $w$。
>
> 我们将以下面的形式来要求你对这棵树完成一些操作：
>
> I. `CHANGE u t` : 把结点 $u$ 的权值改为 $t$。
>
> II. `QMAX u v`: 询问从点 $u$ 到点 $v$ 的路径上的节点的最大权值。
>
> III. `QSUM u v`: 询问从点 $u$ 到点 $v$ 的路径上的节点的权值和。
>
> 注意：从点 $u$ 到点 $v$ 的路径上的节点包括 $u$ 和 $v$ 本身。
>
> **输入格式**
>
> 输入文件的第一行为一个整数 $n$，表示节点的个数。
>
> 接下来 $n-1$ 行，每行 $2$ 个整数 $a$ 和 $b$，表示节点 $a$ 和节点 $b$ 之间有一条边相连。
>
> 接下来一行 $n$ 个整数，第 $i$ 个整数 $w_i$ 表示节点 $i$ 的权值。
>
> 接下来 $1$ 行，为一个整数 $q$，表示操作的总数。
>
> 接下来 $q$ 行，每行一个操作，以 `CHANGE u t` 或者 `QMAX u v` 或者 `QSUM u v` 的形式给出。
>
> **输出格式**
>
> 对于每个 `QMAX` 或者 `QSUM` 的操作，每行输出一个整数表示要求输出的结果。
>
> **数据范围**
>
> 对于 $100 \%$ 的数据，保证 $1\le n \le 3\times 10^4$，$0\le q\le 2\times 10^5$。
>
> 中途操作中保证每个节点的权值 $w$ 在 $-3\times 10^4$ 到 $3\times 10^4$ 之间。
>
> 题目链接：[P2590](https://www.luogu.com.cn/problem/P2590)。

单点修改，区间查询和与最大值。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 30010, M = 60010, INF = 1e9;
int h[N], w[N], n, q, idx = 1;
struct Edge {
    int to, nxt;
} e[M];
int fa[N], sz[N], son[N], dep[N], top[N], dfn[N], dfnw[N], ts;
struct Node {
    int l, r;
    int sum, mx;
} tr[N<<2];

void pushup(int u) {
    tr[u].sum = tr[u<<1].sum + tr[u<<1|1].sum;
    tr[u].mx = max(tr[u<<1].mx, tr[u<<1|1].mx);
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) return tr[u].sum = tr[u].mx = dfnw[l], void();
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
    pushup(u);
}

int query_sum(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u].sum;
    int mid = (tr[u].l + tr[u].r) >> 1;
    int res = 0;
    if (l <= mid) res += query_sum(u<<1, l, r);
    if (mid+1 <= r) res += query_sum(u<<1|1, l, r);
    return res;
}

int query_max(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u].mx;
    int mid = (tr[u].l + tr[u].r) >> 1;
    int res = -INF;
    if (l <= mid) res = query_max(u<<1, l, r);
    if (mid+1 <= r) res = max(res, query_max(u<<1|1, l, r));
    return res;
}

void update(int u, int p, int k) {
    if (tr[u].l == p && tr[u].r == p) {
        tr[u].sum = tr[u].mx = k;
        return;
    }
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (p <= mid) update(u<<1, p, k);
    else update(u<<1|1, p, k);
    pushup(u);
}

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void dfs1(int u, int father) {
    fa[u] = father, sz[u] = 1, dep[u] = dep[father]+1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == father) continue;
        dfs1(to, u);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    dfn[u] = ++ts, dfnw[ts] = w[u], top[u] = t;

    if (!son[u]) return;
    dfs2(son[u], t);

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa[u] || to == son[u]) continue;
        dfs2(to, to);
    }
}

int query_path_sum(int x, int y) {
    int res = 0;
    while (top[x] != top[y]) {
        if (dep[top[x]] < dep[top[y]]) swap(x, y);
        res += query_sum(1, dfn[top[x]], dfn[x]);
        x = fa[top[x]];
    }
    if (dfn[x] > dfn[y]) swap(x, y);
    res += query_sum(1, dfn[x], dfn[y]);
    return res;
}

int query_path_max(int x, int y) {
    int res = -INF;
    while (top[x] != top[y]) {
        if (dep[top[x]] < dep[top[y]]) swap(x, y);
        res = max(res, query_max(1, dfn[top[x]], dfn[x]));
        x = fa[top[x]];
    }
    if (dfn[x] > dfn[y]) swap(x, y);
    res = max(res, query_max(1, dfn[x], dfn[y]));
    return res;
}

void update_vertex(int x, int k) {
    update(1, dfn[x], k);
}

int main() {
    scanf("%d", &n);
    for (int i = 1, a, b; i < n; i++)
        scanf("%d%d", &a, &b), add(a, b), add(b, a);
    for (int i = 1; i <= n; i++)
        scanf("%d", &w[i]);

    dfs1(1, 0);
    dfs2(1, 1);
    build(1, 1, n);

    scanf("%d", &q);
    char tp[10];
    int x, y;

    while (q--) {
        scanf("%s%d%d", tp, &x, &y);
        if (tp[1] == 'H') update_vertex(x, y);
        else if (tp[1] == 'M') printf("%d\n", query_path_max(x, y));
        else printf("%d\n", query_path_sum(x, y));
    }
    
    return 0;
}
```

## [SHOI2012] 魔法树

> Harry Potter 新学了一种魔法：可以让改变树上的果子个数。满心欢喜的他找到了一个巨大的果树，来试验他的新法术。
>
> 这棵果树共有 $N$ 个节点，其中节点 $0$ 是根节点，每个节点 $u$ 的父亲记为 $fa[u]$，保证有 $fa[u] < u$ 。初始时，这棵果树上的果子都被 Dumbledore 用魔法清除掉了，所以这个果树的每个节点上都没有果子（即 $0$ 个果子）。
>
> 不幸的是，Harry 的法术学得不到位，只能对树上一段路径的节点上的果子个数统一增加一定的数量。也就是说，Harry 的魔法可以这样描述：`A u v d` 。表示将点 $u$ 和 $v$ 之间的路径上的所有节点的果子个数都加上 $d$。
>
> 接下来，为了方便检验 Harry 的魔法是否成功，你需要告诉他在释放魔法的过程中的一些有关果树的信息：`Q u`。表示当前果树中，以点 $u$ 为根的子树中，总共有多少个果子？
>
> **输入格式**
>
> 第一行一个正整数 $N (1 \leq N \leq 100000)$，表示果树的节点总数，节点以 $0,1,\dots,N - 1$ 标号，$0$ 一定代表根节点。
>
> 接下来 $N - 1$ 行，每行两个整数 $a,b (0 \leq a < b < N)$，表示 $a$ 是 $b$ 的父亲。
>
> 接下来是一个正整数 $Q(1 \leq Q \leq 100000)$，表示共有 $Q$ 次操作。
>
> 后面跟着 $Q$ 行，每行是以下两种中的一种：
>
> 1. `A u v d`，表示将 $u$ 到 $v$ 的路径上的所有节点的果子数加上 $d$。保证 $0 \leq u,v < N,0 < d < 100000$
>
> 2. `Q u`，表示询问以 $u$ 为根的子树中的总果子数，注意是包括 $u$ 本身的。
>
> **输出格式**
>
> 对于所有的 `Q` 操作，依次输出询问的答案，每行一个。答案可能会超过 $2^{32}$ ，但不会超过 $10^{15}$ 。
>
> 题目链接：[P3833](https://www.luogu.com.cn/problem/P3833)。

这道题更板了，只需要路径修改和子树查询。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;
typedef long long LL;

int h[N], n, idx = 1;
struct Edge {
    int to, nxt;
} e[N];
int dfn[N], sz[N], fa[N], top[N], dep[N], son[N], ts;

struct Node {
    int l, r;
    LL sum, add;
} tr[N<<2];

void spread(int u, int k) {
    tr[u].add += k;
    tr[u].sum += 1LL * k * (tr[u].r - tr[u].l + 1);
}

void pushdown(int u) {
    if (tr[u].add) {
        spread(u<<1, tr[u].add);
        spread(u<<1|1, tr[u].add);
        tr[u].add = 0;
    }
}

void pushup(int u) {
    tr[u].sum = tr[u<<1].sum + tr[u<<1|1].sum;
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) return;
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
}

void update(int u, int l, int r, int k) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u, k);
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) update(u<<1, l, r, k);
    if (mid+1 <= r) update(u<<1|1, l, r, k);
    pushup(u);
}

LL query(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u].sum;
    pushdown(u);

    LL res = 0;
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) res += query(u<<1, l, r);
    if (mid+1 <= r) res += query(u<<1|1, l, r);
    return res;
}

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void dfs1(int u) {
    sz[u] = 1, dep[u] = dep[fa[u]] + 1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        dfs1(to);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    dfn[u] = ++ts, top[u] = t;
    if (!son[u]) return;
    dfs2(son[u], t);
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == son[u]) continue;
        dfs2(to, to);
    }
}

LL query_tree(int u) {
    return query(1, dfn[u], dfn[u]+sz[u]-1);
}

void update_path(int u, int v, int k) {
    while (top[u] != top[v]) {
        if (dep[top[u]] < dep[top[v]]) swap(u, v);
        update(1, dfn[top[u]], dfn[u], k);
        u = fa[top[u]]; 
    }
    if (dfn[u] > dfn[v]) swap(u, v);
    update(1, dfn[u], dfn[v], k);
}

int main() {
    scanf("%d", &n);
    for (int i = 1, a, b; i < n; i++) {
        scanf("%d%d", &a, &b), ++a, ++b;
        add(a, b);
        fa[b] = a;
    }

    dfs1(1);
    dfs2(1, 1);
    build(1, 1, n);

    char tp[2];
    int q, u, v, d;
    scanf("%d", &q);
    while (q--) {
        scanf("%s%d", tp, &u), ++u;
        if (*tp == 'Q') printf("%lld\n", query_tree(u));
        else scanf("%d%d", &v, &d), ++v, update_path(u, v, d);
    }
    return 0;
}
```

## [NOI2015] 软件包管理器

> 你决定设计你自己的软件包管理器。不可避免地，你要解决软件包之间的依赖问题。如果软件包 $a$ 依赖软件包 $b$，那么安装软件包 $a$ 以前，必须先安装软件包 $b$。同时，如果想要卸载软件包 $b$，则必须卸载软件包 $a$。
>
> 现在你已经获得了所有的软件包之间的依赖关系。而且，由于你之前的工作，除 $0$ 号软件包以外，在你的管理器当中的软件包都会依赖一个且仅一个软件包，而 $0$ 号软件包不依赖任何一个软件包。且依赖关系不存在环（即不会存在 $m$ 个软件包 $a_1,a_2, \dots , a_m$，对于 $i<m$，$a_i$ 依赖 $a_{i+1}$，而 $a_m$ 依赖 $a_1$ 的情况）。
>
> 现在你要为你的软件包管理器写一个依赖解决程序。根据反馈，用户希望在安装和卸载某个软件包时，快速地知道这个操作实际上会改变多少个软件包的安装状态（即安装操作会安装多少个未安装的软件包，或卸载操作会卸载多少个已安装的软件包），你的任务就是实现这个部分。
>
> 注意，安装一个已安装的软件包，或卸载一个未安装的软件包，都不会改变任何软件包的安装状态，即在此情况下，改变安装状态的软件包数为 $0$。
>
> **输入格式**
>
> 第一行一个正整数 $n$，表示软件包个数，从 $0$ 开始编号。  
> 第二行有 $n-1$ 个整数，第 $i$ 个表示 $i$ 号软件包依赖的软件包编号。  
> 然后一行一个正整数 $q$，表示操作个数，格式如下：  
>
> - `install x` 表示安装 $x$ 号软件包
> - `uninstall x` 表示卸载 $x$ 号软件包
>
> 一开始所有软件包都是未安装的。  
>
> 对于每个操作，你需要输出这步操作会改变多少个软件包的安装状态，随后应用这个操作（即改变你维护的安装状态）。
>
> **输出格式**
>
> 输出 $q$ 行，每行一个整数，表示每次询问的答案。
>
> 题目链接：[P2146](https://www.luogu.com.cn/problem/P2146)。

这需要支持区间和的查询与将区间修改成一个数。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;
int h[N], fa[N], idx = 1, n, q;
int dep[N], sz[N], son[N], dfn[N], top[N], ts;

struct Edge {
    int to, nxt;
} e[N];

struct Node {
    int l, r;
    int sum, flag;
} tr[N<<2];

void add(int a, int b) {
    e[++idx] = {b, h[a]}, h[a] = idx;
}

void dfs1(int u) {
    dep[u] = dep[fa[u]] + 1, sz[u] = 1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        dfs1(to);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    dfn[u] = ++ts, top[u] = t;
    if (!son[u]) return;

    dfs2(son[u], t);

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == son[u]) continue;
        dfs2(to, to);
    }
}

void spread(int u, int val) {
    tr[u].flag = val;
    tr[u].sum = (tr[u].r - tr[u].l + 1) * val;
}

void pushdown(int u) {
    if (tr[u].flag != -1) {
        spread(u<<1, tr[u].flag);
        spread(u<<1|1, tr[u].flag);
        tr[u].flag = -1;
    }
}

void pushup(int u) {
    tr[u].sum = tr[u<<1].sum + tr[u<<1|1].sum;
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r, tr[u].flag = -1;
    if (l == r) return;
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
}

int query(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u].sum;
    pushdown(u);
    int res = 0, mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) res += query(u<<1, l, r);
    if (mid+1 <= r) res += query(u<<1|1, l, r);
    return res;
}

void change(int u, int l, int r, int v) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u, v);
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) change(u<<1, l, r, v);
    if (mid+1 <= r) change(u<<1|1, l, r, v);
    pushup(u);
}

int uninstall(int u) {
    int res = query(1, dfn[u], dfn[u]+sz[u]-1);
    change(1, dfn[u], dfn[u]+sz[u]-1, 0);
    return res;
}

int install(int u) {
    int res = dep[u];
    while (u) {
        res -= query(1, dfn[top[u]], dfn[u]);
        change(1, dfn[top[u]], dfn[u], 1);
        u = fa[top[u]];
    }
    return res;
}

int main() {
    scanf("%d", &n);
    for (int i = 2; i <= n; i++) {
        scanf("%d", &fa[i]), ++fa[i];
        add(fa[i], i);
    }
    dfs1(1);
    dfs2(1, 1);
    build(1, 1, n);

    scanf("%d", &q);

    char op[10];
    int x;
    while (q--) {
        scanf("%s%d", op, &x), ++x;
        if (*op == 'u') printf("%d\n", uninstall(x));
        else printf("%d\n", install(x));
    }
    return 0;
}
```

## [SDOI2011] 染色

> 给定一棵 $n$ 个节点的无根树，共有 $m$  个操作，操作分为两种：
>
> 1. 将节点 $a$ 到节点 $b$ 的路径上的所有点（包括 $a$ 和 $b$）都染成颜色 $c$。
> 2. 询问节点 $a$ 到节点 $b$ 的路径上的颜色段数量。
>
> 颜色段的定义是极长的连续相同颜色被认为是一段。例如 `112221` 由三段组成：`11`、`222`、`1`。
>
> **输入格式**
>
> 输入的第一行是用空格隔开的两个整数，分别代表树的节点个数 $n$ 和操作个数 $m$。
>
> 第二行有 $n$ 个用空格隔开的整数，第 $i$ 个整数 $w_i$ 代表结点 $i$ 的初始颜色。
>
> 第 $3$ 到第 $(n + 1)$ 行，每行两个用空格隔开的整数 $u, v$，代表树上存在一条连结节点 $u$ 和节点 $v$ 的边。
>
> 第 $(n + 2)$ 到第 $(n + m + 1)$ 行，每行描述一个操作，其格式为：
>
> 每行首先有一个字符 $op$，代表本次操作的类型。
>
> - 若 $op$ 为 `C`，则代表本次操作是一次染色操作，在一个空格后有三个用空格隔开的整数 $a, b, c$，代表将 $a$ 到 $b$ 的路径上所有点都染成颜色 $c$。
> - 若 $op$ 为 `Q`，则代表本次操作是一次查询操作，在一个空格后有两个用空格隔开的整数 $a, b$，表示查询 $a$ 到 $b$ 路径上的颜色段数量。
>
> **输出格式**
>
> 对于每次查询操作，输出一行一个整数代表答案。
>
> **数据规模与约定**
>
> 对于 $100\%$ 的数据，$1 \leq n, m \leq 10^5$，$1 \leq w_i, c \leq 10^9$，$1 \leq a, b, u, v \leq n$，$op$ 一定为 `C` 或 `Q`，保证给出的图是一棵树。
>
> 除原数据外，还存在一组不计分的 hack 数据。
>
> 题目链接：[P2486](https://www.luogu.com.cn/problem/P2486)。

如果我们知道两个区间的左右颜色和颜色个数，我们就可以 $O(1)$ 合并：首先将颜色个数相加，然后如果 $L.right=R.left$ 那么颜色个数减一。所以可以用线段树。

对于一条路径 $a\to b$，我们可以分为 $a\to LCA,v\to b$ 其中 $v$ 是路径 $LCA\to b$ 上的第二个结点。对于得到的这些区间，最后按照上述方法合并起来求答案即可，这里偷个懒可以直接用 `vector` 把所有区间存下来，之后统一合并。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010, M = 200010;
int a[N], n, m;
int h[N], idx = 1;
struct Edge {
    int to, nxt;
} e[M];
int fa[N], dep[N], top[N], son[N], dfn[N], dfnc[N], sz[N], ts;

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

struct Node {
    int l, r;
    int lc, rc, cnt;
    int change;
} tr[N<<2];

Node rev(Node nd) {
    swap(nd.lc, nd.rc);
    return nd;
}

void pushup(Node& u, const Node& l, const Node& r) {
    u.cnt = l.cnt + r.cnt;
    u.lc = l.lc, u.rc = r.rc;
    if (l.rc == r.lc) u.cnt--;
}

void pushup(int u) {
    pushup(tr[u], tr[u<<1], tr[u<<1|1]);
}

void spread(int u, int c) {
    tr[u].lc = tr[u].rc = c;
    tr[u].change = c;
    tr[u].cnt = 1;
}

void pushdown(int u) {
    if (tr[u].change) {
        spread(u<<1, tr[u].change);
        spread(u<<1|1, tr[u].change);
        tr[u].change = 0;
    }
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) {
        tr[u].cnt = 1;
        tr[u].lc = tr[u].rc = dfnc[l];
        return;
    }
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
    pushup(u);
}

Node query(int u, int l, int r) {
    if (l > r) return rev(query(u, r, l));

    if (l <= tr[u].l && tr[u].r <= r) return tr[u];
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l > mid) return query(u<<1|1, l, r);
    if (mid+1 > r) return query(u<<1, l, r);
    Node res = {0, 0, 0, 0, 0};
    pushup(res, query(u<<1, l, r), query(u<<1|1, l, r));
    return res;
}

void update(int u, int l, int r, int c) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u, c);
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) update(u<<1, l, r, c);
    if (mid+1 <= r) update(u<<1|1, l, r, c);
    pushup(u);
}

void dfs1(int u, int father) {
    fa[u] = father, sz[u] = 1, dep[u] = dep[father]+1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == father) continue;
        dfs1(to, u);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    top[u] = t, dfn[u] = ++ts, dfnc[ts] = a[u];
    if (!son[u]) return;
    dfs2(son[u], t);

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa[u] || to == son[u]) continue;
        dfs2(to, to);
    }
}

void update_path(int a, int b, int c) {
    while (top[a] != top[b]) {
        if (dep[top[a]] < dep[top[b]]) swap(a, b);
        update(1, dfn[top[a]], dfn[a], c);
        a = fa[top[a]];
    }
    if (dfn[a] > dfn[b]) swap(a, b);
    update(1, dfn[a], dfn[b], c);
}

int lca(int a, int b) {
    while (top[a] != top[b]) {
        if (dep[top[a]] < dep[top[b]]) swap(a, b);
        a = fa[top[a]];
    }
    return dep[a] > dep[b] ? b : a;
}

int find(int a, int u) {
    while (top[a] != top[u]) {
        if (fa[top[a]] == u) return top[a];
        a = fa[top[a]];
    }
    return son[u];
}

int query_path(int a, int b) {
    // a -> LCA -> b
    int u = lca(a, b);

    vector<Node> vec;

    while (top[a] != top[u]) {
        vec.push_back(query(1, dfn[a], dfn[top[a]]));
        a = fa[top[a]];
    }
    vec.push_back(query(1, dfn[a], dfn[u]));

    if (b != u) {
        vector<Node> vec2;
        int v = find(b, u);

        while (top[b] != top[v]) {
            vec2.push_back(query(1, dfn[b], dfn[top[b]]));
            b = fa[top[b]];
        }
        
        vec2.push_back(query(1, dfn[b], dfn[v]));

        for (int i = vec2.size()-1; i >= 0; i--) {
            vec.push_back(rev(vec2[i]));
        }
    }

    int res = 0;
    for (const auto& nd: vec) res += nd.cnt;

    for (int i = 0; i < vec.size()-1; i++)
        if (vec[i].rc == vec[i+1].lc) res--;

    return res;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);

    for (int i = 1, a, b; i < n; i++) {
        scanf("%d%d", &a, &b);
        add(a, b), add(b, a);
    }

    dfs1(1, 0);
    dfs2(1, 1);
    build(1, 1, n);

    char op[2];
    for (int i = 1, a, b, c; i <= m; i++) {
        scanf("%s%d%d", op, &a, &b);
        if (*op == 'C') scanf("%d", &c), update_path(a, b, c);
        else printf("%d\n", query_path(a, b));
    }

    return 0;
}
```

## [HEOI2016/TJOI2016] 树

> 在 2016 年，佳媛姐姐刚刚学习了树，非常开心。现在他想解决这样一个问题：给定一颗有根树，根为 $1$ ，有以下两种操作：
>
> 1. 标记操作：对某个结点打上标记。（在最开始，只有结点 $1$ 有标记，其他结点均无标记，而且对于某个结点，可以打多次标记。）
>
> 2. 询问操作：询问某个结点最近的一个打了标记的祖先。（这个结点本身也算自己的祖先）
>
> 你能帮帮她吗?
>
> **输入格式**
>
> 第一行两个正整数 $N$ 和 $Q$ 分别表示节点个数和操作次数。
>
> 接下来 $N-1$ 行，每行两个正整数 $u,v \,\, (1 \leqslant u,v \leqslant n)$ 表示 $u$ 到 $v$ 有一条有向边。
>
> 接下来 $Q$ 行，形如 `oper num` ，`oper`  为 `C` 时表示这是一个标记操作, `oper` 为 `Q` 时表示这是一个询问操作。
>
> **输出格式**
>
> 输出一个正整数表示结果。
>
> **数据范围**
>
> $100\%$ 的数据，$1 \leqslant N, Q \leqslant 100000$ 。

记录一条路径上最下面的点，反映到线段树中是最右边的打过标记的点。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;

int n, q, h[N], idx = 2;
int fa[N], top[N], son[N], dfn[N], dfnid[N], dep[N], sz[N], ts;

struct Node {
    int l, r;
    int right;
} tr[N<<2];

struct Edge {
    int to, nxt;
} e[N];

void pushup(Node& u, const Node& l, const Node& r) {
    u.right = 0;
    if (r.right) u.right = r.right;
    else if (l.right) u.right = l.right;
}

void pushup(int u) {
    pushup(tr[u], tr[u<<1], tr[u<<1|1]);
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) return;
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
    pushup(u);
}

void update(int u, int p) {
    if (tr[u].l == p && tr[u].r == p) return tr[u].right = p, void();
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (p <= mid) update(u<<1, p);
    else update(u<<1|1, p);
    pushup(u);
}

Node query(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u];
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l > mid) return query(u<<1|1, l, r);
    if (mid+1 > r) return query(u<<1, l, r);
    Node res = {0, 0, 0};
    pushup(res, query(u<<1, l, r), query(u<<1|1, l, r));
    return res;
}

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void dfs1(int u) {
    sz[u] = 1, dep[u] = dep[fa[u]]+1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        dfs1(to);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    top[u] = t, dfn[u] = ++ts, dfnid[ts] = u;
    if (!son[u]) return;
    dfs2(son[u], t);
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == son[u]) continue;
        dfs2(to, to);
    }
}

int query_path(int a) {
    while (a) {
        Node nd = query(1, dfn[top[a]], dfn[a]);
        if (nd.right) return dfnid[nd.right];
        a = fa[top[a]];
    }
    return 0;
}

void update_vertex(int a) {
    update(1, dfn[a]);
}

int main() {
    scanf("%d%d", &n, &q);
    for (int i = 1, a, f; i < n; i++) {
        scanf("%d%d", &f, &a);
        add(f, a);
        fa[a] = f;
    }

    dfs1(1);
    dfs2(1, 1);
    build(1, 1, n);

    update_vertex(1);

    char tp[2];
    for (int i = 1, a; i <= q; i++) {
        scanf("%s%d", tp, &a);
        if (*tp == 'C') update_vertex(a);
        else printf("%d\n", query_path(a));
    }

    return 0;
}
```

## [国家集训队] 旅游

> 给定一棵 $n$ 个节点的树，边带权，编号 $0 \sim n-1$，需要支持五种操作：
>
> - `C i w` 将输入的第 $i$ 条边权值改为 $w$；
> - `N u v` 将 $u,v$ 节点之间的边权都变为相反数；
> - `SUM u v` 询问 $u,v$ 节点之间边权和；
> - `MAX u v` 询问 $u,v$ 节点之间边权最大值；
> - `MIN u v` 询问 $u,v$ 节点之间边权最小值。
>
> 保证任意时刻所有边的权值都在 $[-1000,1000]$ 内。
>
> **输入格式**
>
> 第一行一个正整数 $n$，表示节点个数。  
>
> 接下来 $n-1$ 行，每行三个整数 $u,v,w$，表示 $u,v$ 之间有一条权值为 $w$ 的边，描述这棵树。  
>
> 然后一行一个正整数 $m$，表示操作数。
>
> 接下来 $m$ 行，每行表示一个操作。
>
> **输出格式**
>
> 对于每一个询问操作，输出一行一个整数表示答案。
>
> **数据范围**
>
> 对于 $100\%$ 的数据，$1\le n,m \le 2\times 10^5$。
>
> 题目链接：[P1505](https://www.luogu.com.cn/problem/P1505)。

我们规定这棵树是外向树，即所有边都是父结点指向子结点的，那么就可以定义每个点的点权为入边的边权。

关于 $u\to v$ 路径，我们需要修改的点权是路径上所有点，除了 $LCA$。

合并答案的时候也可以像上题那样用 `vector` 把所有区间存起来，然后统一合并。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 200010, M = 400010;
int n, m;
int h[N], w[N], idx = 2;
int fa[N], son[N], top[N], sz[N], dep[N], dfn[N], dfnw[N], ts;

struct Edge {
    int to, w, nxt;
} e[M];

struct Node {
    int l, r;
    int flip;
    int sum, max, min;
} tr[N<<2];

void pushup(Node& u, const Node& l, const Node& r) {
    u.sum = l.sum + r.sum;
    u.max = max(l.max, r.max);
    u.min = min(l.min, r.min);
}

void pushup(int u) {
    pushup(tr[u], tr[u<<1], tr[u<<1|1]);
}

void spread(int u) {
    tr[u].flip ^= 1;
    tr[u].sum *= -1, tr[u].max *= -1, tr[u].min *= -1;
    swap(tr[u].max, tr[u].min);
}

void pushdown(int u) {
    if (tr[u].flip) {
        spread(u<<1), spread(u<<1|1);
        tr[u].flip = 0;
    }
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) {
        tr[u].sum = tr[u].max = tr[u].min = dfnw[l];
        return;
    }
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
    pushup(u);
}

Node query(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u];
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l > mid) return query(u<<1|1, l, r);
    if (mid+1 > r) return query(u<<1, l, r);
    Node res = {0, 0, 0, 0, 0, 0};
    pushup(res, query(u<<1, l, r), query(u<<1|1, l, r));
    return res;
}

void update(int u, int p, int k) {
    if (tr[u].l == p && tr[u].r == p) {
        tr[u].max = tr[u].min = tr[u].sum = k;
        return;
    }
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (p <= mid) update(u<<1, p, k);
    else update(u<<1|1, p, k);
    pushup(u);
}

void flip(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u);
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) flip(u<<1, l, r);
    if (mid+1 <= r) flip(u<<1|1, l, r);
    pushup(u);
}

void add(int a, int b, int c) {
    e[idx] = {b, c, h[a]}, h[a] = idx++;
}

void dfs1(int u, int father, int weight) {
    fa[u] = father, w[u] = weight;
    sz[u] = 1, dep[u] = dep[father] + 1;

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == father) continue;
        dfs1(to, u, e[i].w);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    dfn[u] = ++ts, dfnw[ts] = w[u], top[u] = t;
    if (!son[u]) return;
    dfs2(son[u], t);

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa[u] || to == son[u]) continue;
        dfs2(to, to);
    }
}

int lca(int a, int b) {
    while (top[a] != top[b]) {
        if (dep[top[a]] < dep[top[b]]) swap(a, b);
        a = fa[top[a]];
    }
    return dep[a] > dep[b] ? b : a;
}

Node query_path(int a, int b) {
    int u = lca(a, b);

    vector<Node> nodes;
    while (top[a] != top[u]) {
        nodes.push_back(query(1, dfn[top[a]], dfn[a]));
        a = fa[top[a]];
    }
    if (a != u) nodes.push_back(query(1, dfn[son[u]], dfn[a]));

    while (top[b] != top[u]) {
        nodes.push_back(query(1, dfn[top[b]], dfn[b]));
        b = fa[top[b]];
    }
    if (b != u) nodes.push_back(query(1, dfn[son[u]], dfn[b]));

    Node res = nodes[0];
    for (int i = 1; i < nodes.size(); i++) {
        Node l = res;
        pushup(res, l, nodes[i]);
    }
    return res;
}

void update_edge(int u, int k) {
    int a = e[u<<1].to, b = e[u<<1|1].to;
    if (dep[a] < dep[b]) swap(a, b);
    update(1, dfn[a], k);
}

void flip_path(int a, int b) {
    int u = lca(a, b);

    while (top[a] != top[u]) {
        flip(1, dfn[top[a]], dfn[a]);
        a = fa[top[a]];
    }
    if (a != u) flip(1, dfn[son[u]], dfn[a]);

    while (top[b] != top[u]) {
        flip(1, dfn[top[b]], dfn[b]);
        b = fa[top[b]];
    }
    if (b != u) flip(1, dfn[son[u]], dfn[b]);
}

int main() {
    scanf("%d", &n);
    for (int i = 1, a, b, c; i < n; i++) {
        scanf("%d%d%d", &a, &b, &c);
        ++a, ++b;
        add(a, b, c), add(b, a, c);
    }

    dfs1(1, 0, 0);
    dfs2(1, 1);
    build(1, 1, n);

    char op[5];
    scanf("%d", &m);
    for (int i = 1, a, b; i <= m; i++) {
        scanf("%s%d%d", op, &a, &b);
        ++a, ++b;
        if (op[0] == 'C') update_edge(a-1, b-1);
        else if (op[0] == 'N') flip_path(a, b);
        else if (op[0] == 'S') printf("%d\n", query_path(a, b).sum);
        else if (op[1] == 'A') printf("%d\n", query_path(a, b).max);
        else printf("%d\n", query_path(a, b).min);
    }

    return 0;
}
```

## [CF1023F] Mobile Phone Network

> 给定 $k$ 条未确定边权的边（不构成环），然后给 $m$ 个确定边权的边，对这些边确定合适的边权，使得所有点都连通的情况下：
> 1. 这 $k$ 条边都会被选择。
> 2. $k$ 个连接的总和最大。如果价格相同，会优先选择给定的 $k$ 条边中的边。
>
> 输出总和的最大值，如果可以达到无穷大， 输出 $-1$。
>
> 题目链接：[CF1023F](https://codeforces.com/contest/1023/problem/F)。

我们先把 $k$ 条边的权值设成正无穷，然后加入到 MST 中，然后对于给定的边跑 `kruskal`，枚举所有非树边，对它的两个端点的路径上整体取 $\min$，这个可以树剖维护。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 500010, M = 1000010, INF = 1e9+7;
int h[N], n, m, k, idx = 1;
int dep[N], w[N], son[N], fa[N], top[N], sz[N];
int dfn[N], dfnw[N], ts;
int p[N];

struct EdgeE {
    int to, nxt, w;
} e[M];

struct Edge {
    int a, b, w;
    bool used = false;
    bool operator<(const Edge& e) const {
        return w < e.w;
    }
};
vector<Edge> edges;
vector<Edge> spec;

struct Node {
    int l, r;
    int mn;
} tr[N<<2];

void add(int a, int b, int c) {
    e[idx] = {b, h[a], c}, h[a] = idx++;
}

int find(int x) {
    if (x != p[x]) p[x] = find(p[x]);
    return p[x];
}

void dfs1(int u, int father, int weight) {
    fa[u] = father, w[u] = weight, sz[u] = 1, dep[u] = dep[father]+1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == father) continue;
        dfs1(to, u, e[i].w);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    dfn[u] = ++ts, dfnw[ts] = w[u], top[u] = t;
    if (!son[u]) return;
    dfs2(son[u], t);
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == son[u] || to == fa[u]) continue;
        dfs2(to, to);
    }
}

void spread(int u, int v) {
    tr[u].mn = min(tr[u].mn, v);
}

void pushdown(int u) {
    spread(u<<1, tr[u].mn);
    spread(u<<1|1, tr[u].mn);
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    tr[u].mn = INF;
    if (l == r) return tr[u].mn = dfnw[l], void();
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
}

int query(int u, int p) {
    if (tr[u].l == p && tr[u].r == p) return tr[u].mn;
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (p <= mid) return query(u<<1, p);
    return query(u<<1|1, p);
}

void update(int u, int l, int r, int k) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u, k);
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) update(u<<1, l, r ,k);
    if (mid+1 <= r) update(u<<1|1, l, r, k);
}

int lca(int a, int b) {
    while (top[a] != top[b]) {
        if (dep[top[a]] < dep[top[b]]) swap(a, b);
        a = fa[top[a]];
    }
    return dep[a] > dep[b] ? b : a;
}

void update_path(int a, int b, int w) {
    int u = lca(a, b);
    while (top[a] != top[u]) {
        update(1, dfn[top[a]], dfn[a], w);
        a = fa[top[a]];
    }
    if (a != u) update(1, dfn[son[u]], dfn[a], w);

    while (top[b] != top[u]) {
        update(1, dfn[top[b]], dfn[b], w);
        b = fa[top[b]];
    }
    if (b != u) update(1, dfn[son[u]], dfn[b], w);
}

int main() {
    scanf("%d%d%d", &n, &k, &m);

    for (int i = 1; i <= n; i++) p[i] = i;

    int a, b, c;
    for (int i = 1; i <= k; i++) {
        scanf("%d%d", &a, &b);
        p[find(a)] = find(b);
        add(a, b, INF), add(b, a, INF);
        Edge edg;
        edg.a = a, edg.b = b, edg.w = 0;
        spec.push_back(edg);
    }

    for (int i = 1; i <= m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        Edge edg;
        edg.a = a, edg.b = b, edg.w = c;
        edges.push_back(edg);
    }
    
    // kruskal
    sort(edges.begin(), edges.end());
    for (auto& edge: edges) {
        int a = edge.a, b = edge.b, w = edge.w;
        if (find(a) != find(b)) {
            p[find(a)] = find(b);
            add(a, b, w), add(b, a, w);
            edge.used = true;
        }
    }

    dfs1(1, 0, 0);
    dfs2(1, 1);
    build(1, 1, n);

    for (const auto& edge: edges) {
        if (edge.used) continue;
        update_path(edge.a, edge.b, edge.w);
    }

    long long res = 0;
    for (const auto& edge: spec) {
        int a = edge.a, b = edge.b;
        if (dep[a] < dep[b]) swap(a, b);
        int cost = query(1, dfn[a]);
        if (cost == INF) puts("-1"), exit(0);
        res += cost;
    }

    return !printf("%lld\n", res);
}
```

## [SCOI2015] 情报

> 奈特公司是一个巨大的情报公司，它有着庞大的情报网络。情报网络中共有 $n$ 名情报员。每名情报员可能有若干名 (可能没有) 下线，除 $1$ 名大头目外其余 $n-1$ 名情报员有且仅有 $1$ 名上线。奈特公司纪律森严，每名情报员只能与自己的上、下线联系，同时，情报网络中任意两名情报员一定能够通过情报网络传递情报。
>
> 奈特公司每天会派发以下两种任务中的一个任务：
>
> 1. 搜集情报：指派 $T$ 号情报员搜集情报；
> 2. 传递情报：将一条情报从 $X$ 号情报员传递给 $Y$ 号情报员。
>
> 情报员最初处于潜伏阶段，他们是相对安全的，我们认为此时所有情报员的危险值为 $0$；一旦某个情报员开始搜集情报，他的危险值就会持续增加，每天增加 $1$ 点危险值 (开始搜集情报的当天危险值仍为 $0$，第 $2$ 天危险值为 $1$，第 $3$ 天危险值为 $2$，以此类推)。传递情报并不会使情报员的危险值增加。
>
> 为了保证传递情报的过程相对安全，每条情报都有一个风险控制值 $C$。奈特公司认为，参与传递这条情报的所有情报员中，危险值大于 $C$ 的情报员将对该条情报构成威胁。现在，奈特公司希望知道，对于每个传递情报任务，参与传递的情报员有多少个，其中对该条情报构成威胁的情报员有多少个。
>
> **输入格式**
>
> 第一行包含一个正整数 $n$，表示情报员个数。
> 笫二行包含 $n$ 个非负整数，其中第 $i$ 个整数 $P_i$ 表示 $i$ 号情报员上线的编号。特别地，若 $P_i=0$，表示 $i$ 号情报员是大头目。
> 第三行包含一个正整数 $q$，表示奈特公司将派发 $q$ 个任务 (每天一个)。
>
> 随后 $q$ 行，依次描述 $q$ 个任务。每行首先有一个正整数 $k$。
>
> - 若 $k=1$，表示任务是传递情报，随后有三个正整数 $X_i$、$Y_i$、$C_i$，依次表示传递情报的起点、终点和风险控制值。
> - 若 $k=2$，表示任务是搜集情报，随后有 $1$ 个正整数 $T_i$，表示搜集情报的情报员编号。
>
> **输出格式**
>
>
> 对于每个传递情报任务输出一行，包含两个整数，分别是参与传递情报的情报员个数和对该条情报构成威胁的情报员个数。
>
> 数据范围：$n\leqslant 2\times 10^5,Q\leqslant 2\times 10^5,0<P_i,C_i\leqslant N,1\leqslant T_i,X_i,Y_i\leqslant n$。
>
> 题目链接：[P4216](https://www.luogu.com.cn/problem/P4216)。

操作抽象一下，就是这样：

1. 查询 $X,Y$ 路径上满足 $T-T_u>C$ 的点的数量。
2. 给结点打上当前时间戳。

注意到第二个操作的时间戳是严格单调递增的，我们把第一个操作移项一下：$T-C>T_u$，那么我们可以离线下来按照 $T-C$ 排序，每次把满足要求的 $u$ 的点权设为 $1$，然后求一个 $X\sim Y$ 路径上的点权和。

对于结点个数，$dep(X)+dep(Y)-2dep(LCA)+1$。

由于只有单点加和区间查询，这里用树状数组比较好。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 200010;
int n, q, rt, tr[N];
int fa[N], h[N], sz[N], son[N], top[N], dfn[N], dep[N], idx = 1, ts;

#define lowbit(x) ((x)&(-x))

void modify(int p, int x) {
    for (; p <= n; p += lowbit(p))
        tr[p] += x;
}

int query(int p) {
    int res = 0;
    for (; p; p -= lowbit(p))
        res += tr[p];
    return res;
}

int query(int l, int r) {
    return query(r) - query(l-1);
}

struct Edge {
    int to, nxt;
} e[N];

struct Query1 {
    int x, y, c, t;

    bool operator<(const Query1& q) const {
        return t-c < q.t-q.c;
    }
};

struct Query2 {
    int u, t;
};

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void dfs1(int u) {
    sz[u] = 1, dep[u] = dep[fa[u]] + 1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        dfs1(to);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    top[u] = t, dfn[u] = ++ts;
    if (!son[u]) return;
    dfs2(son[u], t);

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == son[u]) continue;
        dfs2(to, to);
    }
}

int lca(int a, int b, int& sum) {
    while (top[a] != top[b]) {
        if (dep[top[a]] < dep[top[b]]) swap(a, b);
        sum += query(dfn[top[a]], dfn[a]);
        a = fa[top[a]];
    }
    if (dfn[a] > dfn[b]) swap(a, b);
    sum += query(dfn[a], dfn[b]);
    return a;
}

vector<Query1> qry1;
queue<Query2> qry2;
int ans1[N], ans2[N];

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &fa[i]);
        if (fa[i]) add(fa[i], i);
        else rt = i;
    }
    scanf("%d", &q);
    for (int i = 1, tp, x, y, c; i <= q; i++) {
        scanf("%d", &tp);
        if (tp == 1) scanf("%d%d%d", &x, &y, &c), qry1.push_back({x, y, c, i});
        else scanf("%d", &x), qry2.push({x, i});
    }
    sort(qry1.begin(), qry1.end());
    dfs1(rt), dfs2(rt, rt);

    for (const Query1& qry: qry1) {
        while (qry2.size()) {
            int t = qry2.front().t, u = qry2.front().u;
            if (qry.t - qry.c <= t) break;
            modify(dfn[u], 1);
            qry2.pop();
        }

        int a = qry.x, b = qry.y;
        int u = lca(a, b, ans2[qry.t]);
        ans1[qry.t] = dep[a] + dep[b] - 2*dep[u] + 1;
    }

    for (int i = 1; i <= q; i++)
        if (ans1[i]) printf("%d %d\n", ans1[i], ans2[i]);

    return 0;
}
```

