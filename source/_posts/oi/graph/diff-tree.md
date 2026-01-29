---
date: 2023-10-17 18:35:00
updated: 2023-10-17 18:35:00
title: 图论 - 树上差分
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
description: 树上差分可以用线性的复杂度处理树上路径的修改，最后通过树上前缀和还原出每个点的权值，在这种情况下它比树链剖分的复杂度更优。
---

## 简介

树上差分可以用线性的复杂度处理树上路径的修改，最后通过树上前缀和还原出每个点的权值，在这种情况下它比树链剖分的复杂度更优。

## [CEOI2017] One-Way Streets

> 给定一张 $n$ 个点 $m$ 条边的无向图，现在想要把这张图定向。
>
> 有 $p$ 个限制条件，每个条件形如 $(x_i,y_i)$，表示在新的有向图当中，$x_i$ 要能够沿着一些边走到 $y_i$。
>
> 现在请你求出，每条边的方向是否能够唯一确定。同时请给出这些能够唯一确定的边的方向。
>
> 数据保证有解。
>
> **输入格式**
>
> 第一行两个空格隔开的正整数 $n,m$ 。
>
> 接下来 $m$ 行，每行两个空格隔开的正整数 $a_i,b_i$，表示 $a_i,b_i$ 之间有一条边。
>
> 接下来一行一个整数 $p$，表示限制条件的个数。
>
> 接下来 $p$ 行，每行两个空格隔开的正整数 $x_i,y_i$，描述一个 $(x_i,y_i)$ 的限制条件。
>
> **输出格式**
>
> 输出一行一个长度为 $m$ 的字符串，表示每条边的答案：
>
> -    若第 $i$ 条边必须得要是 $a_i$ 指向 $b_i$ 的，那么这个字符串的第 $i$ 个字符应当为 ``R``；
>
> -    若第 $i$ 条边必须得要是 $b_i$ 指向 $a_i$ 的，那么这个字符串的第 $i$ 个字符应当为 ``L``；
>
> -    否则，若第 $i$ 条边的方向无法唯一确定，那么这个字符串的第 $i$ 个字符应当为 ``B``。
>
> 对于所有测试点，有 $1\le n,m,p\le 10^5;1\le a_i,b_i,x_i,y_i\le n$。
>
> 题目链接：[P4652](https://www.luogu.com.cn/problem/P4652)。

由于数据保证有解，不用判是否无解给这个题目难度降低了很多，就不需要用树剖那些高级的东西了。

对于 $x \to y$ 这个限制，如果 $x,y$ 处在同一个边双中，无论是顺时针还是逆时针都无所谓，所以考虑缩点，缩点之后这个图会变成一棵树。

对于树上规定 $x\to y$，显然有 $x$ 向上走到 $LCA(x,y)$，然后再向下走到 $y$，所以左半边是向上走，右半边向下走，做一个树上差分即可，给 $w(x)$ 自增，$w(y)$ 自减，之后求一个树上前缀和，对于结点 $u$ 的子结点 $v$：

1. 如果 $w(v)>0$ 代表是向上走的。
2. 如果 $w(v)<0$ 代表是向下走的。

这样就能确定出来边的方向了。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010, M = 400010;
int h[N], th[N], e[M], ne[M], idx, n, m, p;
int L[N], R[N], w[N], rel[M];
int dfn[N], low[N], stk[N], tp, ts;
int cntedcc, id[N], to[N];
bool cover[N];
char res[M];

void add(int h[], int a, int b, int r = -1) {
    e[idx] = b, ne[idx] = h[a], rel[idx] = r, h[a] = idx++;
}

void tarjan(int u, int from) {
    low[u] = dfn[u] = ++ts;
    stk[++tp] = u;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j, i);
            low[u] = min(low[u], low[j]);
        }
        else if (i != (from ^ 1)) low[u] = min(low[u], dfn[j]);
    }

    if (dfn[u] == low[u]) {
        ++cntedcc;
        int y;
        do {
            y = stk[tp--];
            id[y] = cntedcc;
        } while (y != u);
    }
}

void dfs(int u, int fa) {
    cover[u] = true;

    for (int i = th[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        dfs(j, u);
        w[u] += w[j];
        if (w[j] > 0) to[rel[i] >> 1] = u;
        else if (w[j] < 0) to[rel[i] >> 1] = j;
    }
}

int main() {
    memset(h, -1, sizeof h);
    memset(th, -1, sizeof th);
    scanf("%d%d", &n, &m);
    for (int i = 0, a, b; i < m; i++) {
        scanf("%d%d", &a, &b);
        add(h, a, b), add(h, b, a);
        L[i] = a, R[i] = b, res[i] = 'B';
    }

    for (int i = 1; i <= n; i++)
        if (!dfn[i]) tarjan(i, -1);

    for (int u = 1; u <= n; u++) {
        for (int i = h[u]; ~i; i = ne[i]) {
            int v = e[i];
            if (id[u] != id[v]) add(th, id[u], id[v], i);
        }
    }

    scanf("%d", &p);
    for (int i = 0, a, b; i < p; i++) {
        scanf("%d%d", &a, &b);
        w[id[a]]++, w[id[b]]--;
    }

    for (int i = 1; i <= cntedcc; i++)
        if (!cover[i]) dfs(i, 0);
    
    for (int i = 0; i < m; i++) {
        if (to[i] == id[R[i]]) res[i] = 'R';
        else if (to[i] == id[L[i]]) res[i] = 'L';
    }

    return !printf("%s\n", res);
}
```

## [NOIP2015] 运输计划

> L 国有 $n$ 个星球，还有 $n-1$ 条双向航道，每条航道建立在两个星球之间，这 $n-1$ 条航道连通了 L 国的所有星球。
>
> 小 P 掌管一家物流公司， 该公司有很多个运输计划，每个运输计划形如：有一艘物流飞船需要从 $u_i$ 号星球沿最快的宇航路径飞行到 $v_i$ 号星球去。显然，飞船驶过一条航道是需要时间的，对于航道 $j$，任意飞船驶过它所花费的时间为 $t_j$，并且任意两艘飞船之间不会产生任何干扰。
>
> 为了鼓励科技创新， L 国国王同意小 P 的物流公司参与 L 国的航道建设，即允许小 P 把某一条航道改造成虫洞，飞船驶过虫洞不消耗时间。
>
> 在虫洞的建设完成前小 P 的物流公司就预接了 $m$ 个运输计划。在虫洞建设完成后，这 $m$ 个运输计划会同时开始，所有飞船一起出发。当这 $m$ 个运输计划都完成时，小 P 的物流公司的阶段性工作就完成了。
>
> 如果小 P 可以自由选择将哪一条航道改造成虫洞， 试求出小 P 的物流公司完成阶段性工作所需要的最短时间是多少？
>
> **输入格式**
>
> 第一行包括两个正整数 $n, m$，表示 L 国中星球的数量及小 P 公司预接的运输计划的数量，星球从 $1$ 到 $n$ 编号。
>
> 接下来 $n-1$ 行描述航道的建设情况，其中第 $i$ 行包含三个整数 $a_i, b_i$ 和 $t_i$，表示第 $i$ 条双向航道修建在 $a_i$ 与 $b_i$ 两个星球之间，任意飞船驶过它所花费的时间为 $t_i$。  
>
> 数据保证 
>
> 接下来 $m$ 行描述运输计划的情况，其中第 $j$ 行包含两个正整数 $u_j$ 和 $v_j$，表示第 $j$ 个运输计划是从 $u_j$ 号星球飞往 $v_j$号星球。
>
> **输出格式**
>
> 一个整数，表示小 P 的物流公司完成阶段性工作所需要的最短时间。
>
> 对于 $100\%$ 的数据，保证：$1\leq n,m\leq 300000$$1 \leq a_i,b_i \leq n$，$0 \leq t_i \leq 1000$，$1 \leq u_i,v_i \leq n$。

首先完成工作需要时间是所有路线中的边权最大值，我们要让它最小，所以最大值最小，显然是二分。

我们二分答案后考虑如何 `check` 出二分出的答案 $mid$，显然，如果存在路径 $|P_i|\gt mid$，然后我们改的边不在 $\cap P_i$ 上，那么一定会仍然存在另一条路径 $|P_j|>mid$，所以我们需要改的边就在 $\cap P_i$ 上。

对于如何求交集，我们对每个集合中的元素进行计数，如果 $cnt(x)=cnt(P)$ 说明 $x$ 在 $\cap P_i$ 上，所以这也告诉我们路径的交集可以用树上差分求。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 300010, K = 20, M = 600010;
typedef long long LL;

int n, m;
int h[N], w[N], dep[N], fa[N][K], idx = 1;
int d[N];
LL up[N];

struct Edge {
    int to, w, nxt;
} e[M];

struct Query {
    int a, b, u;
    LL len;
} qry[N];

void add(int a, int b, int c) {
    e[idx] = {b, c, h[a]}, h[a] = idx++;
}

void dfs(int u, int f, int weight) {
    dep[u] = dep[f] + 1, up[u] = up[f] + weight;
    fa[u][0] = f, w[u] = weight;

    for (int k = 1; k < K; k++)
        fa[u][k] = fa[fa[u][k-1]][k-1];
    
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        dfs(to, u, e[i].w);
    }
}

int lca(int a, int b) {
    if (dep[a] < dep[b]) swap(a, b);
    for (int k = K-1; ~k; k--)
        if (dep[fa[a][k]] >= dep[b])
            a = fa[a][k];

    if (a == b) return a;

    for (int k = K-1; ~k; k--)
        if (fa[a][k] != fa[b][k])
            a = fa[a][k], b = fa[b][k];

    return fa[a][0];
}

void calc(int u) {
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa[u][0]) continue;
        calc(to);
        d[u] += d[to];
    }
}

bool check(LL mid) {
    fill(d+1, d+1+n, 0);

    int cnt = 0;
    LL res = 0;
    for (int i = 1; i <= m; i++) {
        if (qry[i].len > mid) {
            d[qry[i].a]++, d[qry[i].b]++;
            d[qry[i].u] -= 2;
            res = max(res, qry[i].len);
            cnt++;
        }
    }
    calc(1);
    
    LL sel = 0;
    for (int i = 1; i <= n; i++) {
        if (d[i] == cnt) sel = max(sel, 1LL * w[i]);
    }

    return res - sel <= mid;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1, a, b, c; i < n; i++) {
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c), add(b, a, c);
    }
    dfs(1, 0, 0);

    LL l = 0, r = 0;
    for (int i = 1; i <= m; i++) {
        int a, b, u;
        scanf("%d%d", &a, &b);
        u = lca(a, b);
        qry[i] = {a, b, u, up[a] + up[b] - 2 * up[u]};
        r = max(r, qry[i].len);
    }

    while (l < r) {
        LL mid = (l+r) >> 1;
        if (check(mid)) r = mid;
        else l = mid+1;
    }

    return !printf("%lld\n", r);
}
```

