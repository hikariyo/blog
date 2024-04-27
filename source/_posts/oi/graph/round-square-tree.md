---
date: 2023-09-22 15:51:00
title: 图论 - 圆方树
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 仙人掌图

圆方树是来解决仙人掌图的问题的，其中任意一条边至多只出现在一条简单回路的无向连通图称为仙人掌。

对于一个环，一定在一个边双中，但是一个边双并不一定只包含一个环，所以我们不能简单的只求出来边双算完。可以把下面的图输入 [Graph Editor](https://csacademy.com/app/graph_editor/) 观察，这整个图是边双，但是包含两个环，所以我们在 Tarjan 求的时候需要做一些处理。

```text
1
2
3
4
5
1 2
2 3
3 1
1 4
1 5
4 5
```

具体地说，把每个环中首次遍历到的点作为圆点，建立一个虚拟的方点，将方点和环上的其它点连边，边长为首次遇到的点与其的最短距离。然后求一对顶点 $(u,v)$ 的最短路时，设 $dist(u)$ 为 $u$ 到树根的路径长度，分情况讨论：

1. 如果 $t=LCA(u,v)$ 为圆点，那么它们的路径一定交汇于原图上的某个点，最短路是 $dist(u)+dist(v)-2dist(t)$
2. 如果 $t=LCA(u,v)$ 为方点，那么它们的路径一定交汇于原图上的某个环，那么最短路是 $dist(u)+dist(v)-dist(A)-dist(B)+d(A,B)$，其中 $A,B$ 为找 $LCA$ 时在 $t$ 下面的两个点。

## 模板

> 给你一个有 $n$ 个点和 $m$ 条边的仙人掌图，和 $q$ 组询问，每次询问两个点 $u,v$ 求两点之间的最短路。
>
> 数据范围：$1\le n,q\le 10000,1\le m\le 20000,1\le w\le 10^5$
>
> 题目链接：[P5236](https://www.luogu.com.cn/problem/P5236)，[AcWing 2863](https://www.acwing.com/problem/content/2866/)。

圆方树一个点最多会对应一个方点，所以点要开 $2n$，也就是两万。

因为圆方树也是树，所以边在 $2n$ 的级别，圆方树是单向从上向下指的，原图是双向的并且和圆方树边数一样，所以边数是 $6n$。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 20010, M = 60010;
int h[N], th[N], e[M], ne[M], w[M], idx;
int dfn[N], low[N], s[N], stot[N], timestamp;
int fa[N][15], depth[N], dist[N];
int fw[N], fe[N], ffa[N];
int n, m, q, square;
int A, B;

void add(int h[], int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void buildcir(int x, int y, int z) {
    int sum = z;
    add(th, x, ++square, 0);
    for (int k = y; k != x; k = ffa[k])
        s[k] = sum, sum += fw[k];
    s[x] = stot[x] = sum;
    for (int k = y; k != x; k = ffa[k])
        add(th, square, k, min(s[k], sum - s[k])), stot[k] = sum;
}

void tarjan(int u, int from) {
    dfn[u] = low[u] = ++timestamp;

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            ffa[j] = u, fw[j] = w[i], fe[j] = i;
            tarjan(j, i);
            low[u] = min(low[u], low[j]);
            // 连接圆点之间的边
            if (low[j] > dfn[u]) add(th, u, j, w[i]);
        }
        else if (i != (from ^ 1)) low[u] = min(low[u], dfn[j]);
    }

    // 构造方点
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        // 在 u 下面并且它的前一个边没连接到 u, 说明 j 是环的最后一个结点
        if (dfn[j] > dfn[u] && i != fe[j])
            buildcir(u, j, w[i]);
    }
}

void initlca(int u, int father) {
    depth[u] = depth[father] + 1;
    fa[u][0] = father;

    for (int k = 1; k <= 14; k++) fa[u][k] = fa[fa[u][k-1]][k-1];
    for (int i = th[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!depth[j]) {
            dist[j] = dist[u] + w[i];
            initlca(j, u);
        }
    }
}

int lca(int a, int b) {
    if (depth[a] < depth[b]) swap(a, b);
    for (int k = 14; ~k; k--)
        if (depth[fa[a][k]] >= depth[b])
            a = fa[a][k];
    
    if (a == b) return a;
    for (int k = 14; ~k; k--)
        if (fa[a][k] != fa[b][k])
            a = fa[a][k], b = fa[b][k];
    
    A = a, B = b;
    return fa[a][0];
}

int main() {
    memset(h, -1, sizeof h);
    memset(th, -1, sizeof th);

    scanf("%d%d%d", &n, &m, &q);
    for (int i = 1, a, b, c; i <= m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        add(h, a, b, c), add(h, b, a, c);
    }

    square = n;
    for (int i = 1; i <= n; i++)
        if (!dfn[i]) tarjan(i, -1), initlca(i, 0);

    for (int i = 1, a, b; i <= q; i++) {
        scanf("%d%d", &a, &b);
        int t = lca(a, b);
        if (t <= n) printf("%d\n", dist[a] + dist[b] - 2*dist[t]);
        else {
            int l = abs(s[A] - s[B]);
            l = min(l, stot[A] - l);
            printf("%d\n", dist[a] + dist[b] - dist[A] - dist[B] + l);
        }
    }
    return 0;
}
```

## 环路运输

> 在求直径之前，先用这道题看如何使用单调队列优化环形问题。
>
> 在一条环形公路旁均匀地分布着 $N$ 座仓库，编号为 $1∼N$，编号为 $i$ 的仓库与编号为 $j$ 的仓库之间的距离定义为 $dist(i,j)=min(|i−j|,N−|i−j|)$，也就是逆时针或顺时针从 $i$ 到 $j$ 中较近的一种。
>
> 每座仓库都存有货物，其中编号为 $i$ 的仓库库存量为 $A_i$。
>
> 在 $i$ 和 $j$ 两座仓库之间运送货物需要的代价为 $A_i+A_j+dist(i,j)$。
>
> 求在哪两座仓库之间运送货物需要的代价最大。
>
> 题目链接：[AcWing 291](https://www.acwing.com/problem/content/291/)。

我们使用破换成链的技巧，然后观察得到对于 $j<i$ 且处在同一半环上时花费应该是：
$$
A_j-j+A_i+i
$$
只需要使用单调队列维护$A_j-j$ 的最大值即可。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 2000010;
int A[N], q[N], n;

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &A[i]), A[i+n] = A[i];
    int hh = 0, tt = -1, res = 0;
    for (int i = 1; i <= 2*n; i++) {
        while (hh <= tt && i - q[hh] > n/2) hh++;
        res = max(res, A[i]+i+A[q[hh]]-q[hh]);
        while (hh <= tt && A[q[tt]] - q[tt] <= A[i] - i) tt--;
        q[++tt] = i;
    }
    return !printf("%d\n", res);
}
```

## [SHOI2008] 仙人掌 II

> 定义在图上两点之间的距离为这两点之间最短路径的距离。
>
> 定义一个图的直径为这张图相距最远的两个点的距离。
>
> 现在我们假定仙人图的每条边的权值都是 1，你的任务是求出给定的仙人图的直径。
>
> 输入的建图是 $m$ 条路径，每个路径的两个相邻点之间有边。
>
> 题目链接：[P4129](https://www.luogu.com.cn/problem/P4129)，[AcWing 2752](https://www.acwing.com/problem/content/description/2754/)。

首先肯定是把仙人掌图转成圆方树，然后在树上求解，类似于求树的直径。

+ 如果当前点是圆点，正常求解即可。

+ 如果当前点是方点，需要找到两个子结点 $u,v$ 使得 $dist(u)+dist(v)-dist(u,v)$ 的最小值最大。由于边权为 1，所以 $dist(u,v)=\min\{|i_u-i_v|,tot-|i_u-i_v|\}$，其中 $i_u,i_v$ 指的是沿着固定方向给环上结点做的编号。

    我们规定只枚举环的半边，这样 $dist(u,v)=i_u-i_v$，然后发现我们要求是 $dist(u)+dist(v)+i_u-i_v$，因此可以用单调队列优化。和上面说的一样。
    

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 100010, M = 300010;
int n, m, square;
int h[N], th[N], e[M], ne[M], w[M], idx;
int fe[N], ffa[N], s[N];
int dfn[N], low[N], timestamp;

void add(int h[], int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void buildcir(int x, int y, int z) {
    int sum = z;
    for (int k = y; k != x; k = ffa[k])
        s[k] = sum++;
    
    s[x] = sum;
    add(th, x, ++square, 0);
    for (int k = y; k != x; k = ffa[k])
        add(th, square, k, min(s[k], sum - s[k]));
}

void tarjan(int u, int from) {
    dfn[u] = low[u] = ++timestamp;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            ffa[j] = u, fe[j] = i;
            tarjan(j, i);
            low[u] = min(low[u], low[j]);
            if (low[j] > dfn[u]) add(th, u, j, w[i]);
        }
        else if (i != (from ^ 1)) low[u] = min(low[u], dfn[j]);
    }

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (fe[j] != i && dfn[j] > dfn[u])
            buildcir(u, j, w[i]);
    }
}

int ans = 0;
int dfs(int u) {
    static int d[N], q[N], dis[N];
    if (d[u]) return d[u];

    int d1 = 0, d2 = 0;
    for (int i = th[u]; ~i; i = ne[i]) {
        int j = e[i];
        int dist = dfs(j) + w[i];
        if (dist > d1) d2 = d1, d1 = dist;
        else if (dist > d2) d2 = dist;
    }

    d[u] = d1;
    if (u <= n) ans = max(ans, d1+d2);
    else {
        int k = 0;
        // 需要用头结点确定环的大小
        dis[k++] = -0x3f3f3f3f;
        for (int i = th[u]; ~i; i = ne[i]) dis[k++] = d[e[i]];
        for (int i = 0; i < k; i++) dis[i+k] = dis[i];

        int hh = 0, tt = -1;
        for (int i = 0; i < 2*k; i++) {
            while (hh <= tt && i - q[hh] > k / 2) hh++;
            if (hh <= tt) ans = max(ans, dis[q[hh]] + dis[i] + i - q[hh]);
            while (hh <= tt && dis[q[tt]] - q[tt] <= dis[i] - i) tt--;
            q[++tt] = i;
        }
    }

    return d[u];
}

int main() {
    memset(h, -1, sizeof h);
    memset(th, -1, sizeof th);
    scanf("%d%d", &n, &m);
    square = n;
    for (int i = 1, k; i <= m; i++) {
        scanf("%d", &k);
        int last = -1, now;
        while (k--) {
            scanf("%d", &now);
            if (last != -1) add(h, last, now, 1), add(h, now, last, 1);
            last = now;
        }
    }
    tarjan(1, -1);
    dfs(1);
    return !printf("%d\n", ans);
}
```

## [SHOI2006] 仙人掌

> 支撑子图（spanning subgraph）：支撑子图也是原图的子图，这种子图可以比原来少一些边，但是不能破坏图的连通性，也不能去除原来图上的任何顶点。“支撑”的概念类似于我们熟知的“最小支撑树”。
>
> 给定一个仙人掌图，求支撑子图的数量，注意答案是一个很大很大的数（需要高精度）。
>
> 输入同上题，如果给定的图不是仙人掌图，输出 $0$，否则输出支撑子图的数量。
>
> 数据范围：$1\le n\le 20000,0\le m\le 1000$
>
> 题目链接：[P4244](https://www.luogu.com.cn/problem/P4244)。

本题涉及到如何判断一个图是否为仙人掌，这里看仙人掌的定义。

1. 是无向连通图。
2. 每条边至多出现在一个环中。

根据这两条性质，我们可以在 Tarjan 算法之后判断是否有的点没有被访问到，然后在构造环的时候判断每条边是否已经属于别的环，就可以判断出该图是否为仙人掌。

对于题目要求的那个数字，设 $\text{cnt}$ 为环上边的数量，因为每条边都可以选或者一条边也不选，所以答案是：
$$
\prod(\text{cnt}+1)
$$

```cpp
#include <cmath>
#include <vector>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 100010, M = 300010;
bool ok = true;
int h[N], th[N], e[M], ne[M], w[M], idx;
int n, m;
int ffa[N], fe[N], edgeid[M], cntcir;
int dfn[N], low[N], timestamp;
bool st[N];

vector<int> ans = {1};

vector<int> multiply(const vector<int>& a, int b) {
    vector<int> res;
    LL t = 0;
    for (int i = 0; i < a.size() || t; i++) {
        if (i < a.size()) t += (LL)a[i] * b;
        res.emplace_back(t % 10);
        t /= 10;
    }
    while (res.size() > 1 && res.back() == 0) res.pop_back();
    return res;
}

void add(int h[], int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void buildcir(int x, int y, int z) {
    cntcir++;
    int sum = z;
    int cnt = 1;
    for (int k = y; k != x; k = ffa[k]) {
        if (edgeid[fe[k]]) {
            puts("0");
            exit(0);
        }
        edgeid[fe[k]] = edgeid[fe[k]^1] = cntcir;
        cnt++;
    }

    ans = multiply(ans, cnt + 1);
}

void tarjan(int u, int from) {
    dfn[u] = low[u] = ++timestamp;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            ffa[j] = u, fe[j] = i;
            tarjan(j, i);
            low[u] = min(low[u], low[j]);
        }
        else if (i != (from ^ 1)) low[u] = min(low[u], dfn[j]);
    }

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (dfn[j] > dfn[u] && fe[j] != i)
            buildcir(u, j, w[i]);
    }
}

int main() {
    memset(h, -1, sizeof h);
    memset(th, -1, sizeof th);
    scanf("%d%d", &n, &m);
    for (int i = 1, k; i <= m; i++) {
        scanf("%d", &k);
        int last = -1, cur;
        while (k--) {
            scanf("%d", &cur);
            if (last != -1) add(h, last, cur, 1), add(h, cur, last, 1);
            last = cur;
        }
    }
    tarjan(1, -1);
    for (int i = 1; i <= n; i++) if (!dfn[i]) {
        puts("0");
        exit(0);
    }

    for (int i = ans.size()-1; i >= 0; i--) printf("%d", ans[i]);
    puts("");

    return 0;
}
```



