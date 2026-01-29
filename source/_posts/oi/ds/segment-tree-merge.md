---
date: 2023-12-04 10:30:00
updated: 2023-12-04 10:30:00
title: 数据结构 - 线段树合并
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
description: 线段树合并指建立一棵新的线段树，每个节点都是原来两棵树对应结点的合并后的结果，常与树上差分或者树形 DP 一起使用，用于维护每个点有多个信息，并且需要合并之后求最大值或者某些线段树支持的操作。
---

## 简介

线段树合并指建立一棵新的线段树，每个节点都是原来两棵树对应结点的合并后的结果，常与树上差分或者树形 DP 一起使用，用于维护每个点有多个信息，并且需要合并之后求最大值或者某些线段树支持的操作。

## 模板

> 一共有 $n$ 座房屋，并形成一个树状结构。救济粮分 $m$ 次发放，每次选择两个房屋 $(x, y)$，然后对于 $x$ 到 $y$ 的路径上（含 $x$ 和 $y$）每座房子里发放一袋 $z$ 类型的救济粮。
>
> 当所有的救济粮发放完毕后，每座房子里存放的最多的是哪种救济粮。如果某座房屋没有救济粮，则输出 $0$。
>
> 对于 $100\%$ 测试数据，保证 $1 \leq n, m \leq 10^5$，$1 \leq a,b,x,y \leq n$，$1 \leq z \leq 10^5$。
>
> 题目链接：[P4556](https://www.luogu.com.cn/problem/P4556)。

每一次对路径 $(a,b)$ 上某个值加一，令 $u=lca(a,b)$ 相当于树上差分中对 $a,b$ 加一，对 $u,fa(u)$ 减一，然后用线段树可以求出来最值，所以使用线段树合并。

由于每次操作都进行四次修改，会产生 $4n\log n$ 个结点。在合并过程中不要忘记 `pushup` 操作。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((L+R) >> 1)
const int N = 100010, K = 20;
int n, m, h[N], fa[N][K], dep[N], rt[N], ans[N], idx = 2;

struct Edge {
    int to, nxt;
} e[N<<1];

int cnt[4*N*K], id[4*N*K], ls[4*N*K], rs[4*N*K];
int segidx;

void pushup(int u) {
    if (cnt[ls[u]] >= cnt[rs[u]]) id[u] = id[ls[u]], cnt[u] = cnt[ls[u]];
    else id[u] = id[rs[u]], cnt[u] = cnt[rs[u]];
}

void modify(int& u, int x, int k, int L, int R) {
    if (!u) u = ++segidx;
    if (L == R) return cnt[u] += k, id[u] = L, void();
    if (x <= Mid) modify(ls[u], x, k, L, Mid);
    else modify(rs[u], x, k, Mid+1, R);
    pushup(u);
}

int merge(int a, int b, int L, int R) {
    if (!a || !b) return a+b;
    if (L == R) return cnt[a] += cnt[b], a;
    ls[a] = merge(ls[a], ls[b], L, Mid);
    rs[a] = merge(rs[a], rs[b], Mid+1, R);
    pushup(a);
    return a;
}

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void dfs(int u, int f) {
    fa[u][0] = f, dep[u] = dep[f] + 1;
    for (int k = 1; k < K; k++) fa[u][k] = fa[fa[u][k-1]][k-1];
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        dfs(to, u);
    }
}

int lca(int a, int b) {
    if (dep[a] < dep[b]) swap(a, b);
    for (int k = K-1; k >= 0; k--)
        if (dep[fa[a][k]] >= dep[b])
            a = fa[a][k];
    
    if (a == b) return a;
    for (int k = K-1; k >= 0; k--)
        if (fa[a][k] != fa[b][k])
            a = fa[a][k], b = fa[b][k];
    return fa[a][0];
}

void calc(int u, int f) {
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        calc(to, u);
        rt[u] = merge(rt[u], rt[to], 1, 1e5);
    }
    ans[u] = cnt[rt[u]] ? id[rt[u]] : 0;
}

int main() {
    cin >> n >> m;
    for (int i = 1, a, b; i < n; i++) {
        cin >> a >> b;
        add(a, b), add(b, a);
    }
    dfs(1, 0);
    
    for (int i = 1, a, b, c; i <= m; i++) {
        cin >> a >> b >> c;
        int u = lca(a, b);
        modify(rt[a], c, 1, 1, 1e5);
        modify(rt[b], c, 1, 1, 1e5);
        modify(rt[u], c, -1, 1, 1e5);
        modify(rt[fa[u][0]], c, -1, 1, 1e5);
    }
    calc(1, 0);
    
    for (int i = 1; i <= n; i++) cout << ans[i] << '\n';

    return 0;
}
```

## [NOIP2016] 天天爱跑步

> 小 C 同学认为跑步非常有趣，于是决定制作一款叫做《天天爱跑步》的游戏。《天天爱跑步》是一个养成类游戏，需要玩家每天按时上线，完成打卡任务。
>
> 这个游戏的地图可以看作一棵包含 $n$ 个结点和 $n-1$ 条边的树，每条边连接两个结点，且任意两个结点存在一条路径互相可达。树上结点编号为从 $1$ 到 $n$ 的连续正整数。
>
> 现在有 $m$ 个玩家，第 $i$ 个玩家的起点为 $s_i$，终点为 $t_i$。每天打卡任务开始时，所有玩家在第 $0$ 秒同时从自己的起点出发，以每秒跑一条边的速度，不间断地沿着最短路径向着自己的终点跑去，跑到终点后该玩家就算完成了打卡任务。（由于地图是一棵树，所以每个人的路径是唯一的）
>
> 小 C 想知道游戏的活跃度，所以在每个结点上都放置了一个观察员。在结点 $j$ 的观察员会选择在第 $w_j$ 秒观察玩家，一个玩家能被这个观察员观察到当且仅当该玩家在第 $w_j$ 秒也正好到达了结点 $j$。小 C 想知道每个观察员会观察到多少人?
>
> 注意：我们认为一个玩家到达自己的终点后该玩家就会结束游戏，他不能等待一 段时间后再被观察员观察到。 即对于把结点 $j$ 作为终点的玩家：若他在第 $w_j$ 秒前到达终点，则在结点 $j$ 的观察员不能观察到该玩家；若他正好在第 $w_j$ 秒到达终点，则在结点 $j$ 的观察员可以观察到这个玩家。
>
> 数据范围：$1\le n,m\le 3\times10^5$
>
> 题目链接：[P1600](https://www.luogu.com.cn/problem/P1600)。

对于一个路径 $(a,b)$ 令 $u=lca(a,b)$，对于路径上任意点 $p$，分两种情况：

1. 当 $p\in (a,u)$ 时，若能观察到应该满足 $w(p)=dep(a)-dep(p)$，即 $w(p)+dep(p)=dep(a)$。
2. 当 $p\in(b,son(b,u))$ 时，若能观察到应该满足 $w(p)=len-[dep(a)-dep(p)]$ 即 $w(p)-dep(p)=len-dep(a)$。

所以，维护两颗线段树，第一颗维护 $(a,u)$ 路径，第二颗维护 $(b,son(b,u))$ 路径，在每个节点上查询 $w(p)+dep(p)$ 和 $w(p)-dep(p)$ 在两棵树上的权值相加就是答案。至于负数无所谓，线段树是支持负数下标的。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((L+R) >> 1)
const int N = 300010, K = 20;

inline void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

struct SGT {
    int ls[N*K*2], rs[N*K*2], sum[N*K*2];
    int segidx, rt[N];

    int merge(int a, int b, int L, int R) {
        if (!a || !b) return a + b;
        if (L == R) return sum[a] += sum[b], a;
        ls[a] = merge(ls[a], ls[b], L, Mid);
        rs[a] = merge(rs[a], rs[b], Mid+1, R);
        return a;
    }

    void modify(int& u, int p, int k, int L, int R) {
        if (!u) u = ++segidx;
        if (L == R) return sum[u] += k, void();
        if (p <= Mid) modify(ls[u], p, k, L, Mid);
        else modify(rs[u], p, k, Mid+1, R);
    }

    int query(int u, int p, int L, int R) {
        if (!u || p < L || p > R) return 0;
        if (L == R) return sum[u];
        if (p <= Mid) return query(ls[u], p, L, Mid);
        return query(rs[u], p, Mid+1, R);
    }
} T1, T2;

int n, m, h[N], w[N], fa[N][K], dep[N], ans[N], idx = 2;
struct Edge {
    int to, nxt;
} e[N<<1];

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void dfs(int u, int f) {
    fa[u][0] = f, dep[u] = dep[f] + 1;
    for (int k = 1; k < K; k++) fa[u][k] = fa[fa[u][k-1]][k-1];
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        dfs(to, u);
    }
}

int lca(int a, int b) {
    if (dep[a] < dep[b]) swap(a, b);
    for (int k = K-1; k >= 0; k--)
        if (dep[fa[a][k]] >= dep[b])
            a = fa[a][k];
    
    if (a == b) return a;
    for (int k = K-1; k >= 0; k--)
        if (fa[a][k] != fa[b][k])
            a = fa[a][k], b = fa[b][k];
    
    return fa[a][0];
}

void calc(int u, int f) {
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        calc(to, u);
        T1.rt[u] = T1.merge(T1.rt[u], T1.rt[to], 1, n);
        T2.rt[u] = T2.merge(T2.rt[u], T2.rt[to], -n, n);
    }
    ans[u] = T1.query(T1.rt[u], w[u]+dep[u], 1, n) + T2.query(T2.rt[u], w[u]-dep[u], -n, n);
}

int main() {
    read(n), read(m);
    for (int i = 1, a, b; i < n; i++) {
        read(a), read(b);
        add(a, b), add(b, a);
    }

    dfs(1, 0);
    for (int i = 1; i <= n; i++) read(w[i]);

    for (int i = 1, a, b; i <= m; i++) {
        read(a), read(b);
        int u = lca(a, b), len = dep[a] + dep[b] - 2*dep[u];

        T1.modify(T1.rt[a], dep[a], 1, 1, n);
        T1.modify(T1.rt[fa[u][0]], dep[a], -1, 1, n);

        T2.modify(T2.rt[b], len-dep[b], 1, -n, n);
        T2.modify(T2.rt[u], len-dep[b], -1, -n, n);
    }

    calc(1, 0);
    for (int i = 1; i <= n; i++) printf("%d ", ans[i]);
    puts("");

    return 0;
}
```

## [PKUWC2018] Minimax

> 小 $C$ 有一棵 $n$ 个结点的有根树，根是 $1$ 号结点，且每个结点最多有两个子结点。
>
> 定义结点 $x$ 的权值为：
>
> 1.若 $x$ 没有子结点，那么它的权值会在输入里给出，**保证这类点中每个结点的权值互不相同**。
>
> 2.若 $x$ 有子结点，那么它的权值有 $p_x$ 的概率是它的子结点的权值的最大值，有 $1-p_x$ 的概率是它的子结点的权值的最小值。
>
> 现在小 $C$ 想知道，假设 $1$ 号结点的权值有 $m$ 种可能性，**权值第 $i$ 小**的可能性的权值是 $V_i$，它的概率为 $D_i(D_i>0)$，求：
>
> $$
> \sum_{i=1}^{m}i\cdot V_i\cdot D_i^2\bmod  998244353
> $$
> 对于所有数据，满足 $0 < p_i \cdot 10000 < 10000$，所以易证明所有叶子的权值都有概率被根取到。
>
> 题目链接：[P5298](https://www.luogu.com.cn/problem/P5298)。

也就是求在根处所有取值的概率，由于题目给出了这是二叉树，所以考虑转移方程。
$$
\begin{aligned}
dp(v)&=dp_L(v)[p\sum_{i=1}^{v-1} dp_R(i)+(1-p)\sum_{i=v+1}^n dp_R(i)]+dp_R(v)[p\sum_{i=1}^{v-1}dp_L(i)+(1-p)\sum_{i=v+1}^ndp_L(i)]\\
&=dp_L(v)[p\times pre_R(v-1)+(1-p)\times suf_R(v+1)]+dp_R(v)[p\times pre_L(v-1)+(1-p)\times suf_L(v+1)]
\end{aligned}
$$
由于每个结点的值都是唯一的，所以 $dp_L(v),dp_R(v)$ 必然有一个是 $0$。

在合并的过程中，最终一定有左边或者右边某个节点为空，那么假设左边的线段树非空，右边的线段树空，对于结点 $u$ 来说，那么此时的 $pre_R(v-1),suf_R(v+1)$ 值就已经确定了，就是递归的一路走下来所有获得到的 $sum$ 值，所以此时给左边线段树的 $u$ 结点每个数都乘上面推出来的式子，所以是需要懒标记的。

由于一个结点可能只有一个儿子，我们要判掉这种情况，直接让线段树的 $rt$ 继承一下即可，否则会得到错误的结果。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define Mid ((L+R) >> 1)
const int N = 300010, LogN = 20, INV = 796898467, P = 998244353;
int n, m, fa[N], ch[N][2], w[N], nums[N], D[N];

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

int rt[N], ls[N*LogN], rs[N*LogN], sum[N*LogN], mul[N*LogN], idx;

int find(int x) {
    return lower_bound(nums+1, nums+1+m, x) - nums;
}

int create() {
    int u = ++idx;
    sum[u] = 0, mul[u] = 1;
    return u;
}

void pushup(int u) {
    sum[u] = (sum[ls[u]] + sum[rs[u]]) % P;
}

void update(int u, int k) {
    mul[u] = mul[u] * k % P;
    sum[u] = sum[u] * k % P;
}

void pushdown(int u) {
    if (mul[u] == 1) return;
    if (ls[u]) update(ls[u], mul[u]);
    if (rs[u]) update(rs[u], mul[u]);
    mul[u] = 1;
}

void insert(int& u, int p, int L, int R) {
    if (!u) u = create();
    if (L == R) return sum[u] = 1, void();
    if (p <= Mid) insert(ls[u], p, L, Mid);
    else insert(rs[u], p, Mid+1, R);
    pushup(u);
}

int merge(int x, int y, int psx, int rsx, int psy, int rsy, int p, int L, int R) {
    if (!x && !y) return 0;
    if (x && !y) return update(x, (p*psy + (1-p+P)*rsy) % P), x;
    if (!x && y) return update(y, (p*psx + (1-p+P)*rsx) % P), y;
    pushdown(x), pushdown(y);

    int srsx = sum[rs[x]], srsy = sum[rs[y]], slsx = sum[ls[x]], slsy = sum[ls[y]];
    ls[x] = merge(ls[x], ls[y], psx, (rsx + srsx) % P, psy, (rsy + srsy) % P, p, L, Mid);
    rs[x] = merge(rs[x], rs[y], (psx + slsx) % P, rsx, (psy + slsy) % P, rsy, p, Mid+1, R);
    pushup(x);
    return x;
}

void getans(int u, int L, int R) {
    if (L == R) return D[L] = sum[u], void();
    pushdown(u);
    getans(ls[u], L, Mid);
    getans(rs[u], Mid+1, R);
}

void dfs(int u) {
    if (!u) return;
    if (!ch[u][0]) return insert(rt[u], find(w[u]), 1, m);
    dfs(ch[u][0]), dfs(ch[u][1]);
    if (!ch[u][1]) rt[u] = rt[ch[u][0]];
    else rt[u] = merge(rt[ch[u][0]], rt[ch[u][1]], 0, 0, 0, 0, w[u], 1, m);
}

signed main() {
    read(n);
    for (int i = 1; i <= n; i++) {
        read(fa[i]);
        if (!fa[i]) continue;
        if (!ch[fa[i]][0]) ch[fa[i]][0] = i;
        else ch[fa[i]][1] = i;
    }

    for (int i = 1; i <= n; i++) {
        read(w[i]);
        if (!ch[i][0]) nums[++m] = w[i];
        else w[i] = w[i] * INV % P;
    }

    sort(nums+1, nums+1+m);
    dfs(1);
    getans(rt[1], 1, m);

    int res = 0;
    for (int i = 1; i <= m; i++) {
        res = (res + nums[i] * D[i] % P * D[i] % P * i) % P;
    }
    printf("%lld\n", res);

    return 0;
}
```

## [CF1009F] Dominant Indices

> 给定一棵以 $1$ 为根，$n\le 10^6$ 个节点的树。设 $d(u,x)$ 为 $u$ 子树中到 $u$ 距离为 $x$ 的节点数。  
>
> 对于每个点，求一个最小的 $k$，使得 $d(u,k)$ 最大。
> 
> 题目链接：[CF1009F](https://codeforces.com/problemset/problem/1009/F)。
> 

所以这题的 DP 很显然，并且每个节点的信息都是可以继承的，所以考虑使用线段树合并。

$$
dp(u,k) = \sum_v dp(v,k-1)
$$
因为和深度相关，不妨改造一下 $dp$ 式子。

$$
dp(u,d) = \sum_v dp(v, d)\\
dp(u,dep(u)) \leftarrow dp(u, dep(u)) + 1
$$

对每个点求的答案就是，当 $dp(u,d)$ 最大时，$d-dep(u)$ 的最小值，分别维护一个最大值和最大值下标即可，线段树轻松做到。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((L+R) >> 1)
const int N = 1000010, K = 24;
int rt[N], mx[N*K], mxp[N*K], ls[N*K], rs[N*K], dep[N];
int h[N], ans[N], n, segidx, idx = 2;
struct Edge {
    int to, nxt;
} e[N<<1];

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void pushup(int u) {
    if (mx[ls[u]] >= mx[rs[u]]) mx[u] = mx[ls[u]], mxp[u] = mxp[ls[u]];
    else mx[u] = mx[rs[u]], mxp[u] = mxp[rs[u]];
}

int merge(int u, int v, int L, int R) {
    if (!u || !v) return u+v;
    if (L == R) return mx[u] += mx[v], u;
    ls[u] = merge(ls[u], ls[v], L, Mid);
    rs[u] = merge(rs[u], rs[v], Mid+1, R);
    pushup(u);
    return u;
}

void inc(int& u, int p, int L, int R) {
    if (!u) u = ++segidx;
    if (L == R) return ++mx[u], mxp[u] = p, void();
    if (p <= Mid) inc(ls[u], p, L, Mid);
    else inc(rs[u], p, Mid+1, R);
    pushup(u);
}

void dfs(int u, int f) {
    dep[u] = dep[f] + 1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        dfs(to, u);
    }
}

void dp(int u, int f) {
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        dp(to, u);
        rt[u] = merge(rt[u], rt[to], 1, n);
    }
    inc(rt[u], dep[u], 1, n);
    ans[u] = mxp[rt[u]] - dep[u];
}

int main() {
    scanf("%d", &n);
    for (int i = 1, a, b; i < n; i++) scanf("%d%d", &a, &b), add(a, b), add(b, a);
    dfs(1, 0);
    dp(1, 0);
    for (int i = 1; i <= n; i++) printf("%d\n", ans[i]);
    return 0;
}
```

