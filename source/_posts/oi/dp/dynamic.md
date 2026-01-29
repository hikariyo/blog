---
date: 2023-12-04 11:58:00
updated: 2023-12-04 11:58:00
title: 动态规划 - 动态 DP
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
description: 动态 DP 是一种带修求 DP 值的技术，通过广义矩阵乘法维护各 DP 值之间的关系，然后修改的时候修改转移矩阵即可达到带修求 DP 值的目的，可以结合树剖将树上的某些 DP 转化成重儿子和轻儿子之间的转化，然后用树剖线段树维护广义矩阵之积。
---

## 简介

动态 DP 是一种带修求 DP 值的技术，通过广义矩阵乘法维护各 DP 值之间的关系，然后修改的时候修改转移矩阵即可达到带修求 DP 值的目的，可以结合树剖将树上的某些 DP 转化成重儿子和轻儿子之间的转化，然后用树剖线段树维护广义矩阵之积。

## 模板

> 给定一棵 $n$ 个点的树，点带点权。
>
> 有 $m$ 次操作，每次操作给定 $x,y$，表示修改点 $x$ 的权值为 $y$。
>
> 你需要在每次操作之后求出这棵树的最大权独立集的权值大小。
>
> 对于 $100\%$ 的数据，保证 $1\le n,m\le 10^5$，$1 \leq u, v , x \leq n$，$-10^2 \leq a_i, y \leq 10^2$。
>
> 题目链接：[P4719](https://www.luogu.com.cn/problem/P4719)。

设 $f(u,0/1)$ 是不选/选第 $u$ 结点的答案，那么：
$$
\begin{aligned}
f(u,0)&=\sum \max\{f(v,0),f(v,1)\}\\
f(u,1)&=w_u+\sum f(v,0)\\
\end{aligned}
$$
我们想要把它转化成一个可以写成矩阵的形式，首先进行树剖，然后用 $g(u,0/1)$ 代表 $f(u,0/1)$ 去掉重儿子后的值。

那么此时就可以写：
$$
\begin{aligned}
f(u,0)&=g(u,0)+\max\{f(\text{son},0),f(\text{son},1)\}\\
f(u,1)&=g(u,1)+f(\text{son},0)
\end{aligned}
$$
换成广义矩阵：
$$
\begin{bmatrix}
f(u,0)\\
f(u,1)
\end{bmatrix}=\begin{bmatrix}
g(u,0)&g(u,0)\\
g(u,1)&-\infin\\
\end{bmatrix}
\begin{bmatrix}
f(\text{son},0)\\
f(\text{son},1)
\end{bmatrix}
$$
每次操作都修改一下 $f,g$ 数组，方便进行下一次操作。每次查询答案就是从 $1$ 下去的重链上所有矩阵之积乘上 $\begin{bmatrix}0\\0\end{bmatrix}$，所以树剖记录一下链尾。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((L+R) >> 1)
const int N = 100010, INF = 1e8;
int n, m;
int h[N], w[N], f[N][2], g[N][2], idx = 2;
int fa[N], top[N], dfn[N], pos[N], ed[N], sz[N], son[N], ts;

struct Edge {
    int to, nxt;
} e[N<<1];

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

struct Matrix {
    int dat[2][2];

    Matrix() {
        dat[0][0] = dat[0][1] = dat[1][0] = dat[1][1] = -INF;
    }

    Matrix(initializer_list<int> lst) {
        assert(lst.size() == 4);
        int cnt = 0;
        for (int v: lst) dat[cnt/2][cnt%2] = v, cnt++;
    }

    Matrix operator*(const Matrix& mat) const {
        Matrix res;
        for (int k = 0; k < 2; k++)
            for (int i = 0; i < 2; i++)
                for (int j = 0; j < 2; j++)
                    res.dat[i][j] = max(res.dat[i][j], dat[i][k] + mat.dat[k][j]);
        return res;
    }
};

Matrix prod[N<<2];

void pushup(int u) {
    prod[u] = prod[u<<1] * prod[u<<1|1];
}

void build(int u, int L, int R) {
    if (L == R) return prod[u] = {
        g[pos[L]][0], g[pos[L]][0],
        g[pos[L]][1], -INF,
    }, void();
    build(u<<1, L, Mid), build(u<<1|1, Mid+1, R);
    pushup(u);
}

Matrix query(int u, int l, int r, int L, int R) {
    if (l <= L && R <= r) return prod[u];
    if (l > Mid) return query(u<<1|1, l, r, Mid+1, R);
    if (Mid+1 > r) return query(u<<1, l, r, L, Mid);
    return query(u<<1, l, r, L, Mid) * query(u<<1|1, l, r, Mid+1, R);
}

void modify(int u, int p, int L, int R) {
    if (L == R) return prod[u] = {
        g[pos[L]][0], g[pos[L]][0],
        g[pos[L]][1], -INF,
    }, void();
    if (p <= Mid) modify(u<<1, p, L, Mid);
    else modify(u<<1|1, p, Mid+1, R);
    pushup(u);
}

void update(int u, int k) {
    g[u][1] += k - w[u];
    w[u] = k;

    while (u) {
        int tp = top[u];
        modify(1, dfn[u], 1, n);
        Matrix mat = query(1, dfn[tp], dfn[ed[tp]], 1, n);
        int f0 = max(mat.dat[0][0], mat.dat[0][1]);
        int f1 = max(mat.dat[1][0], mat.dat[1][1]);

        int nxt = fa[top[u]];
        g[nxt][0] += max(f0, f1) - max(f[tp][0], f[tp][1]);
        g[nxt][1] += f0 - f[tp][0];
        f[tp][0] = f0, f[tp][1] = f1, u = nxt;
    }
}

void dfs1(int u, int f) {
    sz[u] = 1, fa[u] = f;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        dfs1(to, u);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    top[u] = t, dfn[u] = ++ts, pos[ts] = u;
    if (!son[u]) return ed[t] = u, void();
    dfs2(son[u], t);
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa[u] || to == son[u]) continue;
        dfs2(to, to);
    }
}

void dfs3(int u) {
    g[u][0] = 0, g[u][1] = w[u];

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa[u]) continue;
        dfs3(to);
        if (to != son[u]) {
            g[u][0] += max(f[to][0], f[to][1]);
            g[u][1] += f[to][0];
        }
    }

    f[u][0] = g[u][0] + max(f[son[u]][0], f[son[u]][1]);
    f[u][1] = g[u][1] + f[son[u]][0];
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &w[i]);
    for (int i = 1, a, b; i < n; i++) scanf("%d%d", &a, &b), add(a, b), add(b, a);
    dfs1(1, 0);
    dfs2(1, 1);
    dfs3(1);
    build(1, 1, n);

    for (int i = 1, a, k; i <= m; i++) {
        scanf("%d%d", &a, &k);
        update(a, k);
        printf("%d\n", max(f[1][0], f[1][1]));
    }

    return 0;
}
```

## [NOIP2018] 保卫王国

> Z 国有 $n$ 座城市，$(n - 1)$ 条双向道路，每条双向道路连接两座城市，且任意两座城市都能通过若干条道路相互到达。  
>
> Z 国的国防部长小 Z 要在城市中驻扎军队。驻扎军队需要满足如下几个条件： 
>
> - 一座城市可以驻扎一支军队，也可以不驻扎军队。   
> - 由道路直接连接的两座城市中至少要有一座城市驻扎军队。   
> - 在城市里驻扎军队会产生花费，在编号为 $i$ 的城市中驻扎军队的花费是 $p_i$。      
>
> 小 Z 很快就规划出了一种驻扎军队的方案，使总花费最小。但是国王又给小 Z 提出了 $m$ 个要求，每个要求规定了其中两座城市是否驻扎军队。小 Z 需要针对每个要求逐一给出回答。具体而言，如果国王提出的第 $j$ 个要求能够满足上述驻扎条件（不需要考虑第 $j$ 个要求之外的其它要求），则需要给出在此要求前提下驻扎军队的最小开销。如果国王提出的第 $j$ 个要求无法满足，则需要输出 $-1$。现在请你来帮助小 Z。
>
> 输出共 $m$ 行，每行包含一个个整数，第 $j$ 行表示在满足国王第 $j$ 个要求时的最小开销， 如果无法满足国王的第 $j$ 个要求，则该行输出 $-1$。
>
> 对于 $100\%$的数据，保证 $1 \leq n,m ≤ 10^5$，$1 ≤ p_i ≤ 10^5$，$1 \leq u, v, a, b \leq n$，$a \neq b$，$x, y \in \{0, 1\}$。
>
> 题目链接：[P5024](https://www.luogu.com.cn/problem/P5024)。

这题求的是带修最小点覆盖，同样可以用动态 DP。
$$
\begin{aligned}
f(u,0)&=\sum f(v,1)\\
f(u,1)&=\sum \max\{f(v,0),f(v,1)\}
\end{aligned}
$$
同样的套路，转化一下：
$$
\begin{bmatrix}
f(u,0)\\
f(u,1)
\end{bmatrix}
=
\begin{bmatrix}
+\infin&g(u,0)\\
g(u,1)&g(u,1)
\end{bmatrix}
\begin{bmatrix}
f(\text{son},0)\\
f(\text{son},1)
\end{bmatrix}
$$
强制某个点选就是把 $g(u,0)=+\infin$，不选就是 $g(u,1)=+\infin$。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define Mid ((L+R) >> 1)
const int N = 100010, INF = 1e17;
int n, m, p[N], h[N], f[N][2], g[N][2], idx = 2;
int dfn[N], pos[N], son[N], sz[N], fa[N], top[N], ed[N], ts;
struct Edge {
    int to, nxt;
} e[N<<1];

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

struct Matrix {
    int dat[2][2];
    Matrix() { dat[0][0] = dat[0][1] = dat[1][0] = dat[1][1] = INF; }

    Matrix(initializer_list<int> lst) {
        int cnt = 0;
        for (int v: lst) dat[cnt/2][cnt%2] = v, cnt++;
    }

    Matrix operator*(const Matrix& mat) const {
        Matrix res;
        for (int k = 0; k < 2; k++)
            for (int i = 0; i < 2; i++)
                for (int j = 0; j < 2; j++)
                    res.dat[i][j] = min(res.dat[i][j], dat[i][k] + mat.dat[k][j]);
        return res;
    }
};

Matrix prod[N<<2];
void pushup(int u) {
    prod[u] = prod[u<<1] * prod[u<<1|1];
}

void build(int u, int L, int R) {
    if (L == R) return prod[u] = {
        INF, g[pos[L]][0],
        g[pos[L]][1], g[pos[L]][1],
    }, void();
    build(u<<1, L, Mid), build(u<<1|1, Mid+1, R);
    pushup(u);
}

Matrix query(int u, int l, int r, int L, int R) {
    if (l <= L && R <= r) return prod[u];
    if (l > Mid) return query(u<<1|1, l, r, Mid+1, R);
    if (Mid+1 > r) return query(u<<1, l, r, L, Mid);
    return query(u<<1, l, r, L, Mid) * query(u<<1|1, l, r, Mid+1, R);
}

void modify(int u, int p, int L, int R) {
    if (L == R) return prod[u] = {
        INF, g[pos[L]][0],
        g[pos[L]][1], g[pos[L]][1],
    }, void();
    if (p <= Mid) modify(u<<1, p, L, Mid);
    else modify(u<<1|1, p, Mid+1, R);
    pushup(u);
}

void dfs1(int u, int f) {
    fa[u] = f, sz[u] = 1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        dfs1(to, u);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    top[u] = t, dfn[u] = ++ts, pos[ts] = u;
    if (!son[u]) return ed[top[u]] = u, void();
    dfs2(son[u], t);
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa[u] || to == son[u]) continue;
        dfs2(to, to);
    }
}

void dfs3(int u) {
    g[u][0] = 0, g[u][1] = p[u];
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa[u]) continue;
        dfs3(to);
        if (to != son[u]) {
            g[u][0] += f[to][1];
            g[u][1] += min(f[to][0], f[to][1]);
        }
    }
    f[u][0] = g[u][0] + f[son[u]][1];
    f[u][1] = g[u][1] + min(f[son[u]][0], f[son[u]][1]);
}

void update(int u, int s, int val) {
    g[u][s] = val;

    while (u) {
        int tp = top[u], nxt = fa[tp];
        modify(1, dfn[u], 1, n);
        Matrix mat = query(1, dfn[tp], dfn[ed[tp]], 1, n);
        int f0 = min(mat.dat[0][0], mat.dat[0][1]);
        int f1 = min(mat.dat[1][0], mat.dat[1][1]);

        g[nxt][1] = g[nxt][1] - min(f[tp][0], f[tp][1]) + min(f0, f1);
        g[nxt][0] = g[nxt][0] - f[tp][1] + f1;
        f[tp][0] = f0, f[tp][1] = f1, u = nxt;
    }
}

int solve(int a, int x, int b, int y) {
    int raw1 = g[a][!x], raw2 = g[b][!y];
    update(a, !x, INF);
    update(b, !y, INF);

    int res = min(f[1][0], f[1][1]);

    update(a, !x, raw1);
    update(b, !y, raw2);

    return res >= INF ? -1 : res;
}

signed main() {
    scanf("%lld%lld%*s", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%lld", &p[i]);
    for (int i = 1, a, b; i < n; i++) scanf("%lld%lld", &a, &b), add(a, b), add(b, a);
    dfs1(1, 0);
    dfs2(1, 1);
    dfs3(1);
    build(1, 1, n);

    for (int i = 1, a, b, x, y; i <= m; i++) {
        scanf("%lld%lld%lld%lld", &a, &x, &b, &y);
        printf("%lld\n", solve(a, x, b, y));
    }

    return 0;
}
```

