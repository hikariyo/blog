---
date: 2023-09-24 10:04:00
updated: 2023-09-24 10:04:00
title: 图论 - 基环树
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
description: 基环树是一个环上挂着若干颗树，通常是断环上一条边然后分类讨论，转为树上问题。
---

## [IOI2008] Island

> 你准备浏览一个公园，该公园由 $N$ 个岛屿组成，当地管理部门从每个岛屿 $i$ 出发向另外一个岛屿建了一座长度为 $L_i$ 的桥，不过桥是可以双向行走的。同时，每对岛屿之间都有一艘专用的往来两岛之间的渡船。相对于乘船而言，你更喜欢步行。你希望经过的桥的总长度尽可能长，但受到以下的限制：
>
> - 可以自行挑选一个岛开始游览。
> - 任何一个岛都不能游览一次以上。
> - 无论任何时间，你都可以由当前所在的岛 $S$ 去另一个从未到过的岛 $D$。从 $S$ 到 $D$ 有如下方法：
>   - 步行：仅当两个岛之间有一座桥时才有可能。对于这种情况，桥的长度会累加到你步行的总距离中。
>   - 渡船：你可以选择这种方法，仅当没有任何桥和以前使用过的渡船的组合可以由 $S$ 走到 $D$ (当检查是否可到达时，你应该考虑所有的路径，包括经过你曾游览过的那些岛)。
>
> 注意，你不必游览所有的岛，也可能无法走完所有的桥。
>
> 请你编写一个程序，给定 $N$ 座桥以及它们的长度，按照上述的规则，计算你可以走过的桥的长度之和的最大值。
>
> **输入格式**
>
> 第一行包含 $N$ 个整数，即公园内岛屿的数目。
>
> 随后的 $N$ 行每一行用来表示一个岛。第 $i$ 行由两个以单空格分隔的整数，表示由岛 $i$ 筑的桥。第一个整数表示桥另一端的岛，第二个整数表示该桥的长度 $L_i$。你可以假设对于每座桥，其端点总是位于不同的岛上。
>
> **输出格式**
>
> 仅包含一个整数，即可能的最大步行距离。
>
> 对于 $100\%$ 的数据，$2\leqslant N\leqslant 10^6,1\leqslant L_i\leqslant 10^8$。
>
> 题目链接：[P4381](https://www.luogu.com.cn/problem/P4381)。

每个点都有唯一出边，说明这是一个基环树森林。对于环的求法，和仙人掌一样用 Tarjan 求即可。题目可以看出每个岛之间是相互独立的，所以对每个岛求一个最远距离，最后加起来就是答案。

一个想法是直接用仙人掌求直径的思想求，做树形 DP 的时候只需要传入方点即可：

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

#define int long long

typedef long long LL;
const int N = 2000010, M = 4000010;
int h[N], e[M], ne[M], w[M], idx;
int fa[N], fe[N];
bool cir[N];
int dfn[N], low[N], timestamp;
LL s[N], stot[N];
int n, square;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void buildcir(int x, int y, int edge) {
    LL sum = w[edge];
    for (int k = y; k != x; k = fa[k])
        s[k] = sum, sum += w[fe[k]], cir[fe[k]] = cir[fe[k]^1] = true;
    cir[edge] = cir[edge^1] = true;
    s[x] = stot[x] = sum;
    ++square;
    for (int k = y; k != x; k = fa[k])
        stot[k] = sum, add(square, k, 0);
    add(square, x, 0);
}

void tarjan(int u, int from) {
    dfn[u] = low[u] = ++timestamp;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            fe[j] = i, fa[j] = u;
            tarjan(j, i);
            low[u] = min(low[u], low[j]);
        }
        else if (i != (from ^ 1)) low[u] = min(low[u], dfn[j]);
    }

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (dfn[j] > dfn[u] && fe[j] != i) buildcir(u, j, i);
    }
}

LL ans = 0;
LL dfs(int u, int fa) {
    static LL f[N], point[2*N], sc[2*N], q[N];
    if (f[u]) return f[u];
    LL d1 = 0, d2 = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        if (cir[i]) continue;
        int j = e[i];
        if (j == fa) continue;
        int d = dfs(j, u) + w[i];
        if (d > d1) d2 = d1, d1 = d;
        else if (d > d2) d2 = d;
    }
    f[u] = d1;

    if (u <= n) ans = max(ans, d1+d2);
    else {
        int k = 0;
        // s[k] 递减
        for (int i = h[u]; ~i; i = ne[i]) point[k] = e[i], sc[k] = s[e[i]] + stot[e[i]], k++;
        for (int i = 0; i < k; i++) point[i+k] = point[i], sc[i+k] = s[point[i]];
        // circle: max(d[j] + s[j] - s[i] + d[i])
        int hh = 0, tt = -1;
        for (int i = 0; i < k*2; i++) {
            while (hh <= tt && i - q[hh] >= k) hh++;
            if (hh <= tt) ans = max(ans, f[point[i]] + f[point[q[hh]]] + sc[q[hh]] - sc[i]);
            while (hh <= tt && f[point[q[tt]]] + sc[q[tt]] <= f[point[i]] + sc[i]) tt--;
            q[++tt] = i;
        }
    }

    return f[u];
}

signed main() {
    memset(h, -1, sizeof h);
    scanf("%lld", &n);
    square = n;
    for (int i = 1, j, l; i <= n; i++) {
        scanf("%lld%lld", &j, &l);
        add(i, j, l), add(j, i, l);
    }
    for (int i = 1; i <= n; i++)
        if (!dfn[i]) tarjan(i, -1);
    
    LL res = 0;
    for (int i = n+1; i <= square; i++) {
        ans = 0;
        dfs(i, 0);
        res += ans;
    }
    return !printf("%lld\n", res);
}
```

这么写有个明显的缺点，就是太占空间了，我在洛谷上过了但是在 AcWing 上 MLE 了。我们没有必要把方点建出来，直径要么走环要么不走环，分两种情况单独做 DP 也可以。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 1000010, M = 2000010;
int h[N], e[M], ne[M], w[M], idx;
int dfn[N], low[N], timestamp;
int fa[N], fe[N];
bool incircle[N];
LL d[N];
int n;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}


LL dfs(int u, int fa, LL& len) {
    if (d[u]) return d[u];
    LL d1 = 0, d2 = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa || incircle[j]) continue;
        LL dist = dfs(j, u, len) + w[i];
        if (dist > d1) d2 = d1, d1 = dist;
        else if (dist > d2) d2 = dist;
    }
    len = max(len, d1+d2);
    return d[u] = d1;
}

void circle(int x, int y, int z, LL& len) {
    static LL s[N*2], dist[N*2], q[N*2];
    int sz = 0;
    LL sum = z;

    incircle[x] = true;
    for (int k = y; k != x; k = fa[k])
        s[sz++] = sum, incircle[k] = true, sum += w[fe[k]];
    s[sz++] = sum;

    sz = 0;
    for (int k = y; k != x; k = fa[k])
        dist[sz++] = dfs(k, 0, len);
    dist[sz++] = dfs(x, 0, len);

    for (int i = 0; i < sz; i++)
        dist[i+sz] = dist[i], s[i+sz] = s[i]+sum;

    int hh = 0, tt = -1;
    for (int i = 0; i < sz*2; i++) {
        while (hh <= tt && i - q[hh] >= sz) hh++;
        // !单调队列取出的时候需要判断非空
        if(hh <= tt) len = max(len, dist[q[hh]] - s[q[hh]] + dist[i] + s[i]);
        while (hh <= tt && dist[q[tt]] - s[q[tt]] <= dist[i] - s[i]) tt--;
        q[++tt] = i;
    }
}

void tarjan(int u, int from, LL& len) {
    dfn[u] = low[u] = ++timestamp;

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            fe[j] = i, fa[j] = u;
            tarjan(j, i, len);
            low[u] = min(low[u], low[j]);
        }
        else if (i != (from ^ 1)) low[u] = min(low[u], dfn[j]);
    }

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (dfn[j] > dfn[u] && fe[j] != i)
            circle(u, j, w[i], len);
    }
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d", &n);
    for (int i = 1, j, k; i <= n; i++) {
        scanf("%d%d", &j, &k);
        add(i, j, k), add(j, i, k);
    }
    LL ans = 0;
    for (int i = 1; i <= n; i++)
        if (!dfn[i]) {
            LL len = 0;
            tarjan(i, -1, len);
            ans += len;
        }

    return !printf("%lld\n", ans);
}
```

## [ZJOI2008] 骑士

> 从所有的骑士中选出一个骑士军团，使得军团内没有矛盾的两人（不存在一个骑士与他最痛恨的人一同被选入骑士军团的情况），并且，使得这支骑士军团最具有战斗力。
>
> 为了描述战斗力，我们将骑士按照 $1$ 至 $n$ 编号，给每名骑士一个战斗力的估计，一个军团的战斗力为所有骑士的战斗力总和。
>
> **输入格式**
>
> 第一行包含一个整数 $n$，描述骑士团的人数。
>
> 接下来 $n$ 行，每行两个整数，按顺序描述每一名骑士的战斗力和他最痛恨的骑士。
>
> **输出格式**
>
> 应输出一行，包含一个整数，表示你所选出的骑士军团的战斗力。
>
> 对于 $100\%$ 的测试数据，满足 $1\le n \le 10^6$，每名骑士的战斗力都是不大于 $10^6$ 的正整数。
>
> 题目链接：[P2607](https://www.luogu.com.cn/problem/P2607)。

本题可以是没有上司的舞会基环树版，我们首先找出来一个基环树中的环，假设找到了环中的 $a,b$ 相邻两点，然后我们断开边 $(a,b)$ 体现出来就是把这个边标记一下。

+ 如果不选 $a$，那么一切都和原来那道题一样，直接树形 DP 做就可以，统计答案 $f(a,0)$。
+ 如果选 $a$，那么就导致 $b$ 是一定不可以选的，但我们此时已经断掉了那条边，所以需要手动判断，将 $f(b,1)=-\infin$ 达到不选 $b$ 的效果，统计答案 $f(a,1)$。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 1000010, M = 2000010;
int h[N], e[M], ne[M], p[N], n, idx;
int fe[N];
LL f[N][2];
int dfn[N], timestamp;
int invalidedge[2];
int invalidp;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u, int fa) {
    f[u][0] = 0, f[u][1] = p[u];
    if (u == invalidp) f[u][1] = -1e18;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa || i == invalidedge[0] || i == invalidedge[1]) continue;
        dfs(j, u);
        f[u][0] += max(f[j][0], f[j][1]);
        f[u][1] += f[j][0];
    }
}

void tarjan(int u, LL& res) {
    dfn[u] = ++timestamp;

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            fe[j] = i;
            tarjan(j, res);
        }
    }

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (dfn[j] > dfn[u] && fe[j] != i) {
            invalidedge[0] = i, invalidedge[1] = i^1;
            invalidp = -1;
            dfs(u, 0);
            res = max(res, f[u][0]);
            invalidp = j;
            dfs(u, 0);
            res = max(res, f[u][1]);
        }
    }
}

int main() {
    memset(h, -1, sizeof h);

    scanf("%d", &n);
    for (int i = 1, j; i <= n; i++)
        scanf("%d%d", &p[i], &j), add(i, j), add(j, i);

    LL ans = 0;
    for (int i = 1; i <= n; i++)
        if (!dfn[i]) {
            LL res = 0;
            tarjan(i, res);
            ans += res;
        }
    
    return !printf("%lld\n", ans);
}
```

## [CF1873H] Mad City

> Marcel and Valeriu are in the mad city, which is represented by $n$ buildings with $n$ two-way roads between them.
>
> Marcel and Valeriu start at buildings $a$ and $b$ respectively. Marcel wants to catch Valeriu, in other words, be in the same building as him or meet on the same road.
>
> During each move, they choose to go to an adjacent building of their current one or stay in the same building. Because Valeriu knows Marcel so well, Valeriu can predict where Marcel will go in the next move. Valeriu can use this information to make his move. They start and end the move at the same time.
>
> It is guaranteed that any pair of buildings is connected by some path and there is at most one road between any pair of buildings.
>
> Assuming both players play optimally, answer if Valeriu has a strategy to indefinitely escape Marcel.
>
> 题目链接：[CF1873H](https://codeforces.com/contest/1873/problem/H)。

给出 $n$ 点 $n$ 条双向边，并且保证图是连通的，且无重边，那么这就是一个基环树。由于被追的人可以预测到追捕者下一步如何走，所以当他进入环时就不会被追上了。

所以这道题目就是先求出来 $b$ 到环的最小距离，然后看一下 $a$ 能不能在 $b$ 之前到达环上距离 $b$ 最近的那个点。

```cpp
#include <queue>
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 200010, M = 400010;
int n, a, b;
int h[N], e[M], ne[M], idx;
int dfn[N], fa[N], timestamp;
bool incircle[N];

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void tarjan(int u) {
    dfn[u] = ++timestamp;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            fa[j] = u;
            tarjan(j);
        }
    }

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (dfn[j] > dfn[u] && fa[j] != u) {
            incircle[u] = true;
            for (int k = j; k != u; k = fa[k]) incircle[k] = true;
        }
    }
}

void bfs_b(int& p, int& d) {
    static int dist[N];
    fill(dist, dist+1+n, -1);
    queue<int> q;
    q.push(b);
    dist[b] = 0;
    while (q.size()) {
        int t = q.front(); q.pop();
        if (incircle[t]) return p = t, d = dist[t], void();

        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] != -1) continue;
            dist[j] = dist[t] + 1;
            q.push(j);
        }
    }
}

int bfs_a(int p) {
    static int dist[N];
    fill(dist, dist+1+n, -1);
    dist[a] = 0;
    queue<int> q;
    q.push(a);
    while (q.size()) {
        int t = q.front(); q.pop();
        if (t == p) return dist[t];

        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] != -1) continue;
            dist[j] = dist[t] + 1;
            q.push(j);
        }
    }
    
    exit(-1);
}

bool solve() {
    cin >> n >> a >> b;
    fill(h, h+n+1, -1);
    fill(dfn, dfn+n+1, 0);
    fill(incircle, incircle+n+1, false);
    idx = timestamp = 0;

    for (int i = 1, u, v; i <= n; i++) {
        cin >> u >> v;
        add(u, v), add(v, u);
    }

    if (a == b) return false;

    tarjan(1);

    int p = -1, d = -1;
    bfs_b(p, d);
    if (p == -1) exit(-1);
    return bfs_a(p) > d;
}

int main() {
    int T; cin >> T;
    while (T--) printf("%s\n", solve() ? "YES" : "NO");
    return 0;
}
```

## 最短距离

> 给出一个 $n$ 个点 $n$ 条边的无向连通图。
>
> 你需要支持两种操作：
>
> 1. 修改 第 $x$  条边的长度为 $y$ ；
>
> 2. 查询 点 $x$ 到点 $y$ 的最短距离。
>
> 共有 $m$ 次操作。
>
> **输入格式**
>
> 输入共 $n+m+1$ 行：
>
> 第 $1$ 行，包含两个正整数 $n,m$，表示点数即边数，操作次数。
>
> 第 $2$ 行到第 $n+1$ 行，每行包含三个正整数 $x,y,z$，表示 $x$ 与 $y$ 间有一条长度为 $z$ 的边。
>
> 第 $n+2$ 到 $n+m+1$ 行，每行包含三个正整数 $op,x,y$，表示操作种类，操作的参数（含义见【题目描述】）。
>
> **输出格式**
>
> 对于每次操作 $2$ 输出查询的结果。
>
> **数据范围**
>
> 对于 $100\%$ 的数据，保证 $z\in [0,5000],n,m\in [1,10^5]$。

如果是树上问题，路径是唯一的，换到这个问题，断边 $(a,b)$ 讨论。

1. 不经过断边，直接树剖。
2. 经过断边，$dist(x,a)+dist(a,b)+dist(b,y)$ 或者 $dist(x,b)+dist(b,a)+dist(a,y)$

单点修改，区间查询，果断树状数组。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define lowbit(x) ((x)&(-x))
const int N = 110010, M = 220010;
int h[N], n, q, idx = 2;
int fa[N], dep[N], son[N], sz[N], dfn[N], dfnw[N], w[N], top[N], ts;

struct Edge {
    int to, nxt, w;
} e[M];

int tr[N];
int rmedge;

void add(int a, int b, int c) {
    e[idx] = {b, h[a], c}, h[a] = idx++;
}

void tarjan(int u, int father) {
    static int dfn[N], fa[N], ts;
    dfn[u] = ++ts, fa[u] = father;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (!dfn[to]) tarjan(to, u);
    }

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (dfn[to] > dfn[u] && fa[to] != u) rmedge = i;
    }
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

void add(int p, int x) {
    for (; p <= n; p += lowbit(p))
        tr[p] += x;
}

void modify(int p, int x) {
    int raw = query(p, p);
    add(p, -raw+x);
}

void modify_edge(int x, int y) {
    if ((x << 1) == rmedge || (x << 1 | 1) == rmedge) {
        e[rmedge].w = e[rmedge ^ 1].w = y;
        return;
    }

    int a = e[x<<1].to, b = e[x<<1|1].to;
    if (dep[a] < dep[b]) swap(a, b);
    modify(dfn[a], y);
}

int lca(int x, int y) {
    while (top[x] != top[y]) {
        if (dep[top[x]] < dep[top[y]]) swap(x, y);
        x = fa[top[x]];
    }
    return dep[x] > dep[y] ? y : x;
}

int query_path(int x, int y) {
    int res = 0;
    int u = lca(x, y);
    while (top[x] != top[u]) {
        res += query(dfn[top[x]], dfn[x]);
        x = fa[top[x]];
    }
    if (x != u) res += query(dfn[son[u]], dfn[x]);

    while (top[y] != top[u]) {
        res += query(dfn[top[y]], dfn[y]);
        y = fa[top[y]];
    }
    if (y != u) res += query(dfn[son[u]], dfn[y]);

    return res;
}

void dfs1(int u, int father, int weight) {
    fa[u] = father, dep[u] = dep[father] + 1, sz[u] = 1, w[u] = weight;

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (i == rmedge || i == (rmedge ^ 1) || to == father) continue;
        dfs1(to, u, e[i].w);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    top[u] = t, dfn[u] = ++ts, dfnw[ts] = w[u];
    if (!son[u]) return;
    dfs2(son[u], t);

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (i == rmedge || i == (rmedge ^ 1) || to == fa[u] || to == son[u]) continue;
        dfs2(to, to);
    }
}

signed main() {
    scanf("%lld%lld", &n, &q);
    for (int i = 1, a, b, c; i <= n; i++) {
        scanf("%lld%lld%lld", &a, &b, &c);
        add(a, b, c), add(b, a, c);
    }
    tarjan(1, 0);
    dfs1(1, 0, 0), dfs2(1, 1);

    for (int i = 1; i <= n; i++) add(i, dfnw[i]);

    for (int i = 1, opt, x, y; i <= q; i++) {
        scanf("%lld%lld%lld", &opt, &x, &y);
        if (opt == 1) modify_edge(x, y);
        else {
            int res1 = query_path(x, y);
            int a = e[rmedge].to, b = e[rmedge ^ 1].to;
            int res2 = query_path(x, a) + query_path(b, y) + e[rmedge].w;
            int res3 = query_path(x, b) + query_path(a, y) + e[rmedge].w;
            printf("%lld\n", min(min(res1, res2), res3));
        }
    }

    return 0;
}
```

## [NOIP2023] 三值逻辑

> 小 L 今天学习了 Kleene 三值逻辑。
>
> 在三值逻辑中，一个变量的值可能为：真（$\mathit{True}$，简写作 $\mathit{T}$）、假（$\mathit{False}$，简写作 $\mathit{F}$）或未确定（$\mathit{Unknown}$，简写作 $\mathit{U}$）。
>
> 在三值逻辑上也可以定义逻辑运算。由于小 L 学习进度很慢，只掌握了逻辑非运算 $\lnot$，其运算法则为：
> $$\lnot \mathit{T} = \mathit{F}, \lnot \mathit{F} = \mathit{T}, \lnot\mathit{U} = \mathit{U}.$$
>
> 现在小 L 有 $n$ 个三值逻辑变量 $x_1,\cdots, x_n$。小 L 想进行一些有趣的尝试，于是他写下了 $m$ 条语句。语句有以下三种类型，其中 $\leftarrow$ 表示赋值：
>
> 1. $x_i \leftarrow v$，其中 $v$ 为 $\mathit{T}, \mathit{F}, \mathit{U}$ 的一种；
> 2. $x_i \leftarrow x_j$；
> 3. $x_i \leftarrow \lnot x_j$。
>
> 一开始，小 L 会给这些变量赋初值，然后按顺序运行这 $m$ 条语句。
>
> 小 L 希望执行了所有语句后，所有变量的最终值与初值都相等。在此前提下，小 L 希望初值中 $\mathit{Unknown}$ 的变量尽可能少。
>
> 在本题中，你需要帮助小 L 找到 $\mathit{Unknown}$ 变量个数最少的赋初值方案，使得执行了所有语句后所有变量的最终值和初始值相等。小 L 保证，至少对于本题的所有测试用例，这样的赋初值方案都必然是存在的。
>
> $1 \le t \le 6$，$1 \le n,m \le 10 ^ 5$；
>
> 题目链接：[P9869](https://www.luogu.com.cn/problem/P9869)。

可以看出最终每个变量要么是定值，要么是依赖别人，设为定值时构成自环，那么整个图是基环森林，赋值矛盾的条件是出现 $x=\neg x$，如果设 $x=y$ 时边权为 $0$，$x=\neg y$ 时为 $1$，那么出现奇环时矛盾。

如果你用基环树找环，这个题恶心的地方就是存在重边，所以还是用染色法判断。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define UNKNOWN 0
#define TRUE 1
#define FALSE -1
#define UNINITIALIZED 998244353

const int N = 100010;
int c, t, n, m, val[N], dep[N];
int h[N], col[N], idx = 2;
struct Edge {
    int to, nxt, w;
} e[N<<1];

void add(int a, int b, int c) {
    e[idx] = {b, h[a], c}, h[a] = idx++;
}

int dfs(int u, int co, bool& unk) {
    int sz = 1;
    col[u] = co;
    if (val[u] == UNKNOWN) unk = true;

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to, w = e[i].w;
        int nco = w ? -co : co;
        if (col[to] && col[to] != nco) unk = true;
        else if (!col[to]) sz += dfs(to, nco, unk);
    }
    return sz;
}

void solve() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) dep[i] = i, col[i] = h[i] = 0, val[i] = UNINITIALIZED;
    idx = 2;

    char v[5];
    int i, j;
    while (m--) {
        scanf("%s", v);
        if (*v == '+') {
            scanf("%d%d", &i, &j);
            if (val[j] != UNINITIALIZED) val[i] = val[j], dep[i] = i;
            else dep[i] = dep[j], val[i] = UNINITIALIZED;
        }
        else if (*v == '-') {
            scanf("%d%d", &i, &j);
            if (val[j] != UNINITIALIZED) val[i] = -val[j], dep[i] = i;
            else dep[i] = -dep[j], val[i] = UNINITIALIZED;
        }
        else if (*v == 'T') {
            scanf("%d", &i);
            val[i] = TRUE, dep[i] = i;
        }
        else if (*v == 'F') {
            scanf("%d", &i);
            val[i] = FALSE, dep[i] = i;
        }
        else {
            scanf("%d", &i);
            val[i] = UNKNOWN, dep[i] = i;
        }
    }

    for (int i = 1; i <= n; i++) {
        if (dep[i] < 0) add(-dep[i], i, 1), add(i, -dep[i], 1);
        else add(dep[i], i, 0), add(i, dep[i], 0);
    }

    int ans = 0;
    for (int i = 1; i <= n; i++) {
        if (col[i]) continue;
        bool unk = false;
        int sz = dfs(i, 1, unk);
        if (unk) ans += sz;
    }
    printf("%d\n", ans);
}

int main() {
    scanf("%d%d", &c, &t);
    while (t--) solve();
    return 0;
}
```

如果你非要用基环树判，也可以。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define UNKNOWN 0
#define TRUE 1
#define FALSE -1
#define UNINITIALIZED 998244353

const int N = 100010;
int c, t, n, m, val[N], fe[N], dep[N];
bool mark[N<<1], ins[N], inc[N], st[N];
int h[N], idx = 2;
stack<int> stk;
struct Edge {
    int to, nxt, w;
} e[N<<1];

void add(int a, int b, int c) {
    e[idx] = {b, h[a], c}, h[a] = idx++;
}

int dfs(int u, int f, bool& unk) {
    int sz = 1;
    st[u] = true;
    if (val[u] == UNKNOWN) unk = true;

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to, w = e[i].w;
        if (to == f || inc[to]) continue;
        sz += dfs(to, u, unk);
    }
    return sz;
}

bool findcir(int u, bool& unk, vector<int>& cyc) {
    ins[u] = true;
    stk.push(u);
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (mark[i]) continue;

        fe[to] = i;
        mark[i] = mark[i^1] = true;
        if (ins[to]) {
            int y, sum = 0;
            do {
                y = stk.top();
                stk.pop();
                cyc.emplace_back(y);
                inc[y] = true;
                sum += e[fe[y]].w;
            } while (y != to);
            unk = sum & 1;
            return true;
        }

        if (findcir(to, unk, cyc)) return true;
        mark[i] = mark[i^1] = false;
    }

    ins[u] = false;
    stk.pop();
    return false;
}

void solve() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) dep[i] = i, st[i] = inc[i] = ins[i] = mark[i] = mark[i+n] = h[i] = 0, val[i] = UNINITIALIZED;
    idx = 2;

    char v[5];
    int i, j;
    while (m--) {
        scanf("%s", v);
        if (*v == '+') {
            scanf("%d%d", &i, &j);
            if (val[j] != UNINITIALIZED) val[i] = val[j], dep[i] = i;
            else dep[i] = dep[j], val[i] = UNINITIALIZED;
        }
        else if (*v == '-') {
            scanf("%d%d", &i, &j);
            if (val[j] != UNINITIALIZED) val[i] = -val[j], dep[i] = i;
            else dep[i] = -dep[j], val[i] = UNINITIALIZED;
        }
        else if (*v == 'T') {
            scanf("%d", &i);
            val[i] = TRUE, dep[i] = i;
        }
        else if (*v == 'F') {
            scanf("%d", &i);
            val[i] = FALSE, dep[i] = i;
        }
        else {
            scanf("%d", &i);
            val[i] = UNKNOWN, dep[i] = i;
        }
    }

    for (int i = 1; i <= n; i++) {
        if (dep[i] < 0) add(-dep[i], i, 1), add(i, -dep[i], 1);
        else add(dep[i], i, 0), add(i, dep[i], 0);
    }
    int ans = 0;
    bool unk = 0, unktot = 0;
    vector<int> cyc;
    for (int i = 1; i <= n; i++) {
        if (st[i]) continue;
        cyc.clear();
        unktot = 0;
        if (!findcir(i, unktot, cyc)) cyc.emplace_back(i);

        for (int c: cyc) {
            unk = 0;
            int sz = dfs(c, 0, unk);
            if (unk || unktot) ans += sz;
        }
    }

    printf("%d\n", ans);
}

int main() {
    scanf("%d%d", &c, &t);
    while (t--) solve();
    return 0;
}
```

