---
date: 2023-08-10 16:42:00
title: 动态规划 - 换根 DP
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 简介

换根 DP 是解决当根节点变化时答案也发生变化的问题的题目，通常需要两遍 DFS，一遍先钦定一个点为根处理信息，第二遍是换根 DP。

## [POI2008] STA-Station

> 给定一个 $n\le 10^6$ 个点的树，请求出一个结点，使得以这个结点为根时，所有结点的深度之和最大。
>
> 题目链接：[P3478](https://www.luogu.com.cn/problem/P3478)。

当根节点通过边 $(fa,u)$ 转移时：

1. 原先 $u$ 的子树中的任何信息不变。

2. 现在 $fa$ 也是 $u$ 的子结点，所以 $fa$ 除了 $u$ 以外的子树所有深度都加一，反映到 $sum(u)$ 上就是：
    $$
    sum(u)\leftarrow sum(fa)-[sum(u)+sz(u)]+[n-sz(u)]
    $$
     第一个中括号里是原本 $u$ 对 $fa$ 的贡献。

深度之和，大概是 $n^2$ 级别，注意开 `long long`。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1000010, M = 2000010;
typedef long long LL;
int n, h[N], sz[N], idx = 2;
struct Edge {
    int to, nxt;
} e[M];
LL mx, sum[N];
int mxp;

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void dfs(int u, int fa) {
    sz[u] = 1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa) continue;
        dfs(to, u);
        sz[u] += sz[to];
        sum[u] += sum[to] + sz[to];
    }
}

void dp(int u, int fa) {
    if (fa) sum[u] += sum[fa] - sum[u] - sz[u] + n - sz[u];
    if (sum[u] > mx) mx = sum[u], mxp = u;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa) continue;
        dp(to, u);
    }
}

int main() {
    scanf("%d", &n);
    for (int i = 1, a, b; i < n; i++) {
        scanf("%d%d", &a, &b);
        add(a, b), add(b, a);
    }
    dfs(1, 0);
    dp(1, 0);
    return !printf("%d\n", mxp);
}
```

## [ABC160F] Distributing Integers

> 给定一棵 $N$ 结点的树，求每个结点作为根时拓扑序的数量。
>
> 题目链接：[ABC160F](https://atcoder.jp/contests/abc160/tasks/abc160_f)。

首先，设 $f(u)$ 为以 $1$ 为根时 $u$ 所有子树的拓扑序数量，用 $s_i$ 表示对应子树的结点数量。

对于 $f(u)$，第一个位置一定是给 $u$，后面 $s_u-1$ 个位置按顺序分配。
$$
\begin{aligned}
f(u)&=\binom{s_u-1}{s_1} \binom{s_u-1-s_1}{s_2}\cdots \binom{s_n}{s_n} \times \prod_{v\in son_u} f(v)\\
&=\frac{(s_u-1)!}{(s_u-1-s_1)!s_1!}\times \frac{(s_u-1-s_1)!}{(s_u-1-s_1-s_2)!s_2!}\times \cdots \times 1\times \prod_{v\in son_u}f(v)\\
&=(s_u-1)! \prod_{v\in son_u} \frac{f(v)}{s_v!}\\
&=\frac{(s_u-1)!}{\prod_{v\in tree_u} s_v}
\end{aligned}
$$
当我们求解根节点的时候，设 $dp(u)$ 是以 $u$ 为根的所有结点的 $s_v$ 之积，有：
$$
f(u)=\frac{n!}{dp(u)},dp(u)=\prod_{v\in V} s_v
$$
换根时，对于新的根 $u$ 和原树中 $u$ 的父结点 $fa$，只有它们两个的权值发生了变化：
$$
u:s_u\to n\\
fa:n\to n-s_u
$$
因此，原本的式子发生的变化是：
$$
(\prod_{v\ne u,v\ne fa} s_v) s_u n\to (\prod_{v\ne u,v\ne fa} s_v) n(n-s_u)
$$
因此 $dp(u)$ 应该这样转移：
$$
dp(u)=dp(fa)\times \frac{n-s_u}{s_u}
$$
所以代码的写法就很显然了。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 200010, M = 400010, P = 1e9+7;
int h[N], e[M], ne[M], res[N], n, idx;
LL dp[N], sz[N];

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

LL qmi(LL a, LL k) {
    LL res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

void dfs(int u, int fa) {
    sz[u] = 1;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        dfs(j, u);
        sz[u] += sz[j];
    }
    dp[u] = sz[u];
	
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        dp[u] = dp[u] * dp[j] % P;
    }
}

void dfs_dp(int u, int fa) {
    if (fa) dp[u] = dp[fa] * (n-sz[u]) % P * qmi(sz[u], P-2) % P;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        dfs_dp(j, u);
    }
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d", &n);
    for (int i = 1, a, b; i < n; i++) {
        scanf("%d%d", &a, &b);
        add(a, b), add(b, a);
    }
    dfs(1, 0);
    dfs_dp(1, 0);
    LL fracn = 1;
    for (int i = 2; i <= n; i++) fracn = fracn * i % P;
    for (int i = 1; i <= n; i++) printf("%lld\n", fracn * qmi(dp[i], P-2) % P);
    return 0;
}
```

## [CSP-S2019] 树的重心

> 对于 $n$ 个点的树中每条边，求断掉后分裂出的两个子树的重心编号之和，$n\le 3\times 10^5$。
>
> 题目链接：[P5666](https://www.luogu.com.cn/problem/P5666)。

### 重心性质 #1

首先不看这个题，证明一个充要条件：点 $u$ 是重心 $\Leftrightarrow$ 删除 $u$ 后构成的连通块大小的最大值 $\le n/2$

为了方便理解，我们这里换根为 $u$，所以删除 $u$ 后的所有连通块就是以 $u$ 的子结点为根的子树。

1. 充分性：假设 $\exists v\in son(u),sz_1(v)>n/2$，那么以 $v$ 为根时，$sz_2(u)\le n/2$，此时 $u$ 一定不是重心。

2. 必要性：假设 $u$ 不是重心，满足 $\forall v\in son(u),sz_1(v)\le n/2$，那么换到 $v$ 为根时，$sz_2(u)=n-sz_1(v)\ge n/2$。如果 $sz_1(v)=sz_2(u)=n/2$，那么 $u,v$ 都是重心，否则 $u$ 是重心，因为换到相邻的点不会更优。

    这里 $sz_1$ 指换根前的大小，$sz_2$ 指换根后的大小。

### 重心性质 #2

一棵树如果存在多个重心，设 $u$ 是其中一个重心，设存在边 $(u,v)$ 那么有：

1. $sz(v)\le n/2$，这是重心的性质。
2. $n-sz(v)\ge n/2$，这是不等式的性质。

重心不可能在 $v$ 的子树中，因为向下走 $n-sz$ 会更大，所以重心只可能是 $v$，也就是两个重心之间必然有边，而树上不存在环，所以重心最多只有两个。

当有两个重心时，还需要满足 $n-sz(v)\le n/2$，于是 $sz(v)=n-sz(v)=n/2$，还需要 $n$ 为偶数。

### 重心性质 #3

以 $u$ 为根时，重心一定在 $u$ 所在重链上。如果 $u$ 为重心显然成立，如果不为重心有 $sz(son(v))>n/2$，继续跳，最后一个满足 $n-sz(u)\le n/2$ 的一定是重心，如果有两个那么就是它的父结点。

证明：如果不是重心或者第二重心在下面，那么一定存在重儿子满足 $sz(v)\le n/2$，那么换到 $v$ 就会有 $n-sz(v)\le n/2$，这样就和 $u$ 是最后一个满足条件的矛盾。

所以我们可以用倍增求出这最后一个点。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 300010, M = 600010, K = 19;
int n;
int h[N], sz[N], p[N], f[N][K];
long long res = 0;

int idx = 2;
struct Edge {
    int to, nxt;
} e[M];

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void calc_res(int u) {
    int n = sz[u];
    for (int k = K-1; k >= 0; k--)
        if (f[u][k] && n - sz[f[u][k]] <= n / 2)
            u = f[u][k];

    res += u;
    if (n % 2 == 0 && sz[u] == n/2) res += p[u];
}

void calc_f(int u, int s) {
    f[u][0] = s;
    for (int k = 1; k < K; k++) f[u][k] = f[f[u][k-1]][k-1];
}

void dfs(int u, int fa) {
    sz[u] = 1, p[u] = fa;
    int s = 0;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa) continue;
        dfs(to, u);
        sz[u] += sz[to];
        if (sz[s] < sz[to]) s = to;
    }
    calc_f(u, s);
}

void dp(int u, int fa) {
    int su = f[u][0], szu = sz[u];

    // dp 的过程中找重儿子和次重儿子比较好
    int s1 = 0, s2 = 0;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (sz[to] > sz[s1]) s2 = s1, s1 = to;
        else if (sz[to] > sz[s2]) s2 = to;
    }
    
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa) continue;
        calc_res(to);

        if (to == s1) calc_f(u, s2);
        else calc_f(u, s1);

        p[u] = to;
        sz[u] = n - sz[to];

        calc_res(u);

        dp(to, u);
    }

    p[u] = fa, sz[u] = szu;
    calc_f(u, su);
}

void solve() {
    scanf("%d", &n);
    idx = 2;
    for (int i = 1; i <= n; i++) h[i] = 0;

    for (int i = 1, a, b; i < n; i++) {
        scanf("%d%d", &a, &b);
        add(a, b), add(b, a);
    }

    res = 0;
    dfs(1, 0);
    dp(1, 0);

    printf("%lld\n", res);
}

int main() {
    int T;
    scanf("%d", &T);

    while (T--) solve();

    return 0;
}
```

## [CF685B] Kay and Snowflake

> 给定 $n$ 个点的树和 $q$ 次询问，每次询问一个点 $u$，求以 $u$ 为根时重心的标号，若有多个输出任意一个。
>
> 题目链接：[CF685B](https://codeforces.com/contest/685/problem/B)。

我不太能理解线性做法，这里先用倍增写一个 $O((n+q)\log n)$ 的在线方法。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 300010, K = 20;
int h[N], sz[N], f[N][K], n, q, idx = 2;
struct Edge {
    int to, nxt;
} e[N];

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void dfs(int u) {
    sz[u] = 1;

    int s = 0;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        dfs(to);
        sz[u] += sz[to];

        if (sz[s] < sz[to]) s = to;
    }

    f[u][0] = s;
    for (int k = 1; k < K; k++) f[u][k] = f[f[u][k-1]][k-1];
}

int query(int u) {
    int tot = sz[u];
    for (int k = K-1; k >= 0; k--)
        if (tot - sz[f[u][k]] <= tot / 2)
            u = f[u][k];
    return u;
}

int main() {
    scanf("%d%d", &n, &q);
    for (int i = 2, fa; i <= n; i++) {
        scanf("%d", &fa);
        add(fa, i);
    }

    dfs(1);
    for (int i = 1, u; i <= q; i++) {
        scanf("%d", &u);
        printf("%d\n", query(u));
    }

    return 0;
}
```



