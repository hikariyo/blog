---
date: 2023-09-04 14:00:00
updated: 2023-09-05 21:18:00
title: 图论 - 网络流
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 流的定义

1. 流网络 $G=(V,E)$ 是一个有向图，每条边有一个非负的容量 $c(u,v)\ge 0$，其中源点为 $s$ 汇点为 $t$。

2. 流是 $f$ 一个实值函数，它需要满足下列性质：

    1. 容量限制。对于所有的结点 $u,v\in V$ 要求 $0\le f(u,v)\le c(u,v)$。
    2. 流量守恒。对于所有的结点 $u\in\complement_V\{s,t\}$ 要求 $\sum_{v_1\in V} f(v,u)=\sum_{v_2\in V}f(u,v)$，即除源点汇点外所有流入流量和流出流量都相等。

3. 流 $f$ 的值定义如下：
    $$
    |f|=\sum_{v\in V} f(s,v)-\sum_{v\in V} f(v,s)
    $$
    意思就是所有从源点流出的流量减去流入源点的流量，初始状态后项会是 0，但是在残存网络中不一定。

4. 残存网络 $G_f$ 是流网络 $G$ 在流 $f$ 作用后构造出来的一个网络，每条边的流量 $c_f(u,v)$ 的定义如下：
    $$
    c_f(u,v)=\left\{\begin{aligned}
    &c(u,v)-f(u,v),(u,v)\in E\\
    &f(v,u),(v,u)\in E\\
    &0,\text{otherwise}
    \end{aligned}\right.
    $$
    概括一下就是，如果原图中存在 $f(u,v)\gt 0$，那么 $G_f$ 中 $c_f(v,u)=f(u,v)$，同时 $c_f(u,v)=c(u,v)-f(u,v)$。

5. 对于 $G$ 中的一个流 $f$ 和 $G_f$ 中的可行流 $f'$，有 $|f+f'|=|f|+|f'|$，这里简单说明一下，对于 $f'$ 中的某个边 $(u,v)$：

    1. 如果 $(u,v)\in E$，由于 $f'$ 也是一个可行流，那么这显然是符合流的两个性质。
    2. 如果 $(v,u)\in E$，意思就是 $f(u,v)$ 退掉了 $f'(v,u)$，由于 $f'$ 是满足流的性质的，所以对于点 $u$ 来说只是分了一部分流到了别的边，因此满足流的性质。

    所以 $f+f'$ 仍然是原流网络 $G$ 中的一个可行流。

6. 增广路径 $p$ 是在残存网络 $G_f$ 中一条从 $s\to t$ 的简单路径，要求 $\forall (u,v)\in p$ 满足 $c_f(u,v)\gt 0$

有了上面这一坨定义，下面就要给出一个结论了：如果 $G_f$ 中不存在增广路径，那么 $f$ 就是原网络的一个最大流。这个证明需要用到接下来的最小割。

## 割的定义

1. 割 $(S,T)$ 是满足 $s\in S,t\in T$ 并且 $S\cap T=\varnothing, S\cup T=V$ 的两个集合。

2. 割的容量 $c(S,T)$ 定义如下：
    $$
    c(S,T)=\sum_{v\in S}\sum _{u\in T}c(u,v)
    $$
    所有 $S \to T$ 的容量之和。其中最小割指的是割的容量的最小值。

3. 割的流量 $f(S,T)$ 定义如下：
    $$
    f(S,T)=\sum_{u\in S}\sum_{v\in T}f(u,v)-\sum_{u\in S}\sum_{v\in T}f(v,u)
    $$
    所有 $S\to T$ 的流量减去 $T\to S$ 的流量，这里有 $f(S,T)\le c(S,T)$ 显然成立。

4. 割的流量和原网络中的流的关系：$|f|=f(S,T)$
   
    要证明这个，首先要有下面三个性质。
    
    1. $f(X,Y)=-f(Y,X)$

    2. $f(X,X)=0$

    3. $f(Z,X\cup Y)=f(Z,X)+f(Z,Y),X\cap Y=\varnothing$

        这个推导一下：
        $$
        \begin{aligned}
        f(Z,X\cup Y)&=\sum_{u\in Z}\sum_{v\in X\cup Y} f(u,v)-\sum_{v\in Z}\sum_{u\in X\cup Y}f(u,v)\\
        &=\sum_{u\in Z}\sum_{v_1\in X}f(u,v_1) + \sum_{u\in Z}\sum_{v_2\in Y}f(u,v_2)-\sum_{v\in Z}\sum_{u_1\in X}f(u_1,v)-\sum_{v\in Z}\sum_{u_2\in Y}f(u_2,v)\\
        &=f(Z,X)+f(Z,Y)
        \end{aligned}
        $$
    
    4. 同样地，还有 $f(X\cup Y,Z)=f(X,Z)+f(Y,Z)$
    
    然后开始证明本小节的定理。
    $$
    \begin{aligned}
    f(S,T)&=f(S,V)-f(S,S)\\
    &=f(S,V)\\
    &=f(\{s\},V)+f(\complement_S\{s\},V)\\
    &=f(\{s\},V)\\
    &=|f|
    \end{aligned}
    $$
    其中 $f(\complement_{S}\{s\},V)=0$ 的原因是流量守恒。
    
5. 由上面的关系，可以推出 $\forall |f|,|f|=f(S,T)\le c(S,T)$，所以最大流同样是小于等于最小割的容量的。

接下来是最大流最小割定理，包含三个互为充要条件的命题。

1. 流 $f$ 是最大流。
2. 残存网络 $G_f$ 不存在增广路。
3. 存在割 $(S,T)$ 使得 $|f|=c(S,T)$

要证明这三个条件互为充要条件，我们证明 $1\to 2, 2\to 3, 3\to 1$ 即可。

+ $1\to 2$ 是很显然的，如果能找到增广路那么 $f$ 就不是最大流。

+ $3 \to 1$ 也是比较容易的，设 $|f_1|=c(S,T)$，最大流为 $f_m$，由上面的第五条性质可以知道 $|f_1|\ge |f_m|$，然后因为 $f_m$ 是最大流所以有 $|f_m|\ge |f_1|$，因此 $|f_m|=|f_1|$。由于任意一个割都满足 $c(S,T)\ge |f_m|$ 所以这里一定是最大流等于最小割。

+ $2\to 3$ 我们令残存网络 $G_f$ 中从源点 $s$ 出发的所有边中能通过 $c_f\gt 0$ 到达的点都包含于 $S$ 中，剩下的点包含在 $T$ 中，显然 $(S,T)$ 是一个割。

    下面的操作全都是对于原网络 $G$ 来说的。

    1. 所有 $S\to T$ 的流量等于容量。
        $$
        \forall x\in S,y\in T,f(x,y)=c(x,y)
        $$
        如果存在 $f(x,y)\lt c(x,y)$ 那么在 $G_f$ 中 $c_f(x,y)\gt 0$，那么 $y\in S$，矛盾。

    2. 所有 $T\to S$ 的流量都为零。
        $$
        \forall x\in S,y\in T, f(y,x)=0
        $$
        如果存在 $f(y,x)\gt 0$，那么 $c_f(x,y)=f(y,x)$，那么 $y \in S$，矛盾。

    现在开始推导 $2\to 3$ 成立。
    $$
    \begin{aligned}
    |f|&=f(S,T)\\
    &=\sum_{u\in S}\sum_{v\in T} f(u,v)-\sum_{u\in T}\sum_{v\in S} f(u,v)\\
    &=\sum_{u\in S}\sum_{v\in T} c(u,v)\\
    &=c(S,T)
    \end{aligned}
    $$

这就证明出来了定理，所以要求最大流，一直在残存网络中不断找增广路径，直到找不到为止，这个算法的实现有很多，本文记录两种。

## Edmonds-Karp 算法

> 给定一个包含 n 个点 m 条边的**有向**图，并给定每条边的容量，边的容量非负。
>
> 图中可能存在重边和自环。求从点 S 到点 T 的最大流。
>
> 题目链接：[AcWing 2171](https://www.acwing.com/problem/content/2173/)，[P3376](https://www.luogu.com.cn/problem/P3376)。

不断用 BFS 搜索出一条合适的增广路径，然后加到当前流中，直到找不到为止。EK 算法的复杂度是 $O(nm^2)$，经验值是适合于 $n=10^3,m=10^4$ 的情况。最大流的复杂度一般到不了上限。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 1010, M = 20010, INF = 1e8;
int n, m, S, T;
// f[i] 表示某个边的容量 并不是流量 这里纯粹是因为代码习惯
int h[N], e[M], ne[M], f[M], idx;
// d[v] 表示到某个点的增广路径上的最小容量
// pre[v] 表示某个点的来边 这里记录边方便修改流量
int q[N], d[N], pre[N];
bool st[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], f[idx] = c, h[a] = idx++;
}

bool bfs() {
    int hh = 0, tt = 0;
    memset(st, 0, sizeof st);
    q[0] = S, st[S] = true, d[S] = INF;
    
    while (hh <= tt) {
        int t = q[hh++];
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (!st[j] && f[i]) {
                st[j] = true;
                d[j] = min(d[t], f[i]);
                pre[j] = i;
                if (j == T) return true;
                q[++tt] = j;
            }
        }
    }
    return false;
}

// 洛谷的数据是需要 long long 的，它的每条边都是 2^31 - 1
// 可能存在多条大的流量累加就爆了 int
LL EK() {
    LL r = 0;
    while (bfs()) {
        r += d[T];
        for (int i = T; i != S; i = e[pre[i] ^ 1])
            f[pre[i]] -= d[T], f[pre[i] ^ 1] += d[T];
    }
    return r;
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d%d%d", &n, &m, &S, &T);
    while (m--) {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c), add(b, a, 0);
    }
    printf("%lld\n", EK());
    return 0;
}
```

## Dinic 算法

> 给定一个包含 n 个点 m 条边的**有向**图，并给定每条边的容量，边的容量非负。
>
> 图中可能存在重边和自环。求从点 S 到点 T 的最大流。
>
> 题目链接：[AcWing 2172](https://www.acwing.com/problem/content/2174/)。

Dinic 算法和 EK 算法不同的地方在于它会一次性找多条可行流，而 EK 是一次找一条，那 Dinic 的复杂度是 $O(n^2m)$，经验上可以处理 $n=10^4, m=10^5$ 级别的数据。

这个算法也有几个优化，最大的优化是当前弧优化，相当于记录一下对于结点 $v$ 应当从哪个点开始搜，用 $cur(v)$ 表示。其它优化在代码注释中说。

``` cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 10010, M = 200010, INF = 1e8;

int h[N], e[M], ne[M], f[M], idx;
// cur: 当前弧
// d: 深度, 同时兼任 st 数组
int cur[N], d[N], q[N];
int n, m, S, T;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], f[idx] = c, h[a] = idx++;
}

bool bfs() {
    memset(d, -1, sizeof d);
    int hh = 0, tt = 0;
    q[0] = S, d[S] = 0, cur[S] = h[S];
    while (hh <= tt) {
        int t = q[hh++];
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (d[j] == -1 && f[i]) {
                d[j] = d[t] + 1;
                cur[j] = h[j];
                if (j == T) return true;
                q[++tt] = j;
            }
        }
    }
    return false;
}

int find(int u, int lim) {
    if (u == T) return lim;
    int flow = 0;
    // 如果 flow >= lim 那么接下来搜出来的都是 0，没用了
    for (int i = cur[u]; ~i && flow < lim; i = ne[i]) {
        // 搜到当前弧了, 前面的都不搜了, 如果还能流就等下一次 bfs 再流
        cur[u] = i;
        int j = e[i];
        if (d[j] == d[u] + 1 && f[i]) {
            int t = find(j, min(f[i], lim - flow));
            // 如果流不下去直接删掉 j 点 留给下一次 bfs 搜 这次不搜
            if (!t) d[j] = -1;
            f[i] -= t, f[i ^ 1] += t, flow += t;
        }
    }
    return flow;
}

int dinic() {
    int r = 0, flow;
    while (bfs()) while (flow = find(S, INF)) r += flow;
    return r;
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d%d%d", &n, &m, &S, &T);
    while (m--) {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c), add(b, a, 0);
    }
    printf("%d\n", dinic());
    return 0;
}
```

## 费用流

> 给定一个包含 n 个点 m 条边的**有向**图，并给定每条边的容量和费用，边的容量非负。
>
> 图中可能存在重边和自环，保证费用不会存在负环。
>
> 求从 S 到 T 的最大流，以及在流量最大时的最小费用。
>
> 题目链接：[AcWing 2176](https://www.acwing.com/problem/content/2176/)，[P3381](https://www.luogu.com.cn/problem/P3381)。

建图的时候使 $w(u,v)=-w(v,u)$ 然后在 EK 算法的基础上把 BFS 改成 SPFA 就能求出来最小费用最大流了。首先一定通过不断增广找到最大流，其次每次找到的都是一个最短路，所以最终求得的就是最小费用最大流。

```cpp
#include <queue>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 5010, M = 100010, INF = 0x3f3f3f3f;
int h[N], e[M], ne[M], w[M], f[M], idx;
// incf: incrase flow
int dist[N], pre[N], incf[N];
int n, m, S, T;
bool st[N];

void add(int a, int b, int c, int d) {
    e[idx] = b, ne[idx] = h[a], f[idx] = c, w[idx] = d, h[a] = idx++;
}

bool SPFA() {
    queue<int> q;
    memset(dist, 0x3f, sizeof dist);
    // 无需清空 st 数组, 这是 SPFA 的特性
    dist[S] = 0, incf[S] = INF, q.push(S);
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (f[i] && dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                pre[j] = i;
                incf[j] = min(incf[t], f[i]);
                if (!st[j]) st[j] = true, q.push(j);
            }
        }
    }
    return dist[T] != INF;
}

void EK(int& flow, int& cost) {
    flow = cost = 0;
    while (SPFA()) {
        flow += incf[T], cost += dist[T] * incf[T];
        for (int i = T; i != S; i = e[pre[i] ^ 1])
            f[pre[i]] -= incf[T], f[pre[i] ^ 1] += incf[T];
    }
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d%d%d", &n, &m, &S, &T);
    while (m--) {
        int a, b, c, d;
        scanf("%d%d%d%d", &a, &b, &c, &d);
        add(a, b, c, d), add(b, a, 0, -d);
    }
    int flow, cost;
    EK(flow, cost);
    printf("%d %d\n", flow, cost);
    return 0;
}
```

## 无源汇上下界可行流

> 给定一个包含 n 个点 m 条边的有向图，每条边都有一个流量下界和流量上界。
>
> 求一种可行方案使得在所有点满足流量平衡条件的前提下，所有边满足流量限制。
>
> 如果存在可行方案，则第一行输出 `YES`，接下来 m 行，每行输出一个整数，其中第 i 行的整数表示输入的第 i 条边的流量。如果不存在可行方案，直接输出一行 `NO`。
>
> 题目链接：[AcWing 2188](https://www.acwing.com/problem/content/2190/)。

题目要求就是满足流量守恒和容量限制的同时还要满足 $c_d(u,v)\le f(u,v)\le c_u(u,v)$，策略是先让每条边都流上这些流量，然后满足它流量守恒的性质，方法如下：

1. 对原图 $G=(V,E)$ 构造新图 $G'=(V',E')$，在 $E'$ 中构造 $c(u,v)=c_u(u,v)-c_l(u,v)$，此时就让这些边都流上这么多的流量了。

2. 设 $g(u)=\sum c_l(v_1,u) - \sum c_l(u,v_2)$，即入边流量减去出边流量。

    + 如果 $g(u)=0$ 就已经满足流量守恒。

    + 如果 $g(u)>0$ 说明入边流量更多，要让多的流出，需要连接 $c(S,u)=g(u)$

    + 如果 $g(u)<0$ 说明出边流量更多，要补充缺少的，需要连接 $c(u,T)=g(u)$

3. 对 $G'$ 求最大流，如果所有从源点出发的边都是满流就说明可以构造出可行方案，否则构造不出来。

对于 $G'$ 中的满流最大流，我们通过构造出不存在的虚拟边使得原本溢出或缺少的点符合流量守恒的条件，所以就能求出来一个上下界可行流。

本题要求输出边的流量，对于 $i$ 边我们知道它的反向边编号是 $2i+1$，流量是在反向边（残留网络）中的，然后每条边加上 $c_l$ 就可以。

```cpp
#include <queue>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 210, M = 21010;
int h[N], e[M], ne[M], f[M], g[N], low[M], idx;
int n, m, S, T;
int cur[N], d[N];

void add0(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], f[idx] = c, h[a] = idx++;
}

void add(int a, int b, int c) {
    add0(a, b, c), add0(b, a, 0);
}

bool bfs() {
    queue<int> q;
    memset(d, -1, sizeof d);
    d[S] = 0, cur[S] = h[S], q.push(S);
    while (q.size()) {
        int t = q.front(); q.pop();
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (f[i] && d[j] == -1) {
                d[j] = d[t] + 1;
                cur[j] = h[j];
                if (j == T) return true;
                q.push(j);
            }
        }
    }
    return false;
}

int find(int u, int lim) {
    if (u == T) return lim;
    
    int r = 0;
    for (int i = cur[u]; ~i && r < lim; i = ne[i]) {
        cur[u] = i;
        int j = e[i];
        if (f[i] && d[j] == d[u] + 1) {
            int t = find(j, min(f[i], lim - r));
            r += t, f[i] -= t, f[i^1] += t;
            if (!t) d[j] = -1;
        }
    }
    
    return r;
}

bool dinic() {
    while (bfs()) while (find(S, 1e8));
    for (int i = h[S]; ~i; i = ne[i]) {
        if (f[i]) return false;
    }
    return true;
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d", &n, &m);
    for (int i = 0; i < m; i++) {
        int a, b, c, d;
        scanf("%d%d%d%d", &a, &b, &c, &d);
        add(a, b, d-c);
        low[i] = c, g[b] += c, g[a] -= c;
    }
    S = n+1, T = n+2;
    for (int i = 1; i <= n; i++)
        if (g[i] > 0) add(S, i, g[i]);
        else if (g[i] < 0) add(i, T, -g[i]);
    
    if (!dinic()) puts("NO");
    else {
        puts("YES");
        for (int i = 0; i < m; i++) printf("%d\n", f[i*2+1] + low[i]);
    }
    return 0;
}
```

## 有源汇上下界最大流

> 给定一个包含 n 个点 m 条边的**有向**图，每条边都有一个流量下界和流量上界。
>
> 给定源点 S 和汇点 T，求源点到汇点的最大流。
>
> 题目链接：[AcWing 2189](https://www.acwing.com/problem/content/2191/)。

连接 $c(T,S)=+\infin$，然后将此图看作无源汇图，仿照上题求一个可行流，并且需要保证新图为满流。

然后再求一遍最大流，用 $f'(T,S)+|f|$ 做最终结果，可行性：由于 $f'(T,S)$ 使这个图在预设每条边流量为 $c_l(u,v)$ 时能达到流量守恒，然后 $|f|$ 是注入 $f'(T,S)$ 后再次增广求出来的路径。

保证新图是满流后，所有反向边都满足 $f(u,S')=f(T',u)=0$，所以不会增广到新图中的 $S',T'$。

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
#include <algorithm>
using namespace std;

const int N = 250, M = 21010, INF = 1e8;
int h[N], e[M], ne[M], g[N], f[M], idx;
int n, m, S, T;
int d[N], cur[N];

void add0(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], f[idx] = c, h[a] = idx++;
}

void add(int a, int b, int c) {
    add0(a, b, c), add0(b, a, 0);
}

bool bfs() {
    queue<int> q;
    memset(d, -1, sizeof d);
    q.push(S), d[S] = 0, cur[S] = h[S];
    
    while (q.size()) {
        int t = q.front(); q.pop();
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (d[j] == -1 && f[i]) {
                d[j] = d[t] + 1;
                cur[j] = h[j];
                if (j == T) return true;
                q.push(j);
            }
        }
    }
    return false;
}

int find(int u, int lim) {
    if (u == T) return lim;
    
    int r = 0;
    for (int i = cur[u]; ~i && r < lim; i = ne[i]) {
        cur[u] = i;
        int j = e[i];
        if (f[i] && d[j] == d[u] + 1) {
            int t = find(j, min(f[i], lim - r));
            r += t, f[i] -= t, f[i^1] += t;
            if (!t) d[j] = -1;
        }
    }
    return r;
}

int dinic() {
    int r = 0, flow;
    while (bfs()) while (flow = find(S, INF)) r += flow;
    return r;
}

int main() {
    int s, t;
    memset(h, -1, sizeof h);
    scanf("%d%d%d%d", &n, &m, &s, &t);
    for (int i = 0; i < m; i++) {
        int a, b, c, d;
        scanf("%d%d%d%d", &a, &b, &c, &d);
        add(a, b, d-c);
        g[a] -= c, g[b] += c;
    }
    S = n+1, T = n+2;
    for (int i = 1; i <= n; i++)
        if (g[i] > 0) add(S, i, g[i]);
        else if (g[i] < 0) add(i, T, -g[i]);
    
    
    add(t, s, INF);
    
    dinic();
    for (int i = h[S]; ~i; i = ne[i]) {
        if (f[i]) {
            puts("No Solution");
            return 0;
        }
    }
    
    S = s, T = t;
    int r1 = f[idx-1];
    f[idx-1] = f[idx-2] = 0;
    
    int r2 = dinic();
    printf("%d\n", r1 + r2);
    return 0;
}
```

## 有源汇上下界最小流

> 给定一个包含 n 个点 m 条边的**有向**图，每条边都有一个流量下界和流量上界。
>
> 给定源点 S 和汇点 T，求源点到汇点的最小流。
>
> 题目链接：[AcWing 2190](https://www.acwing.com/problem/content/2192/)。

求完无源汇后求 $T\to S$ 的最大流 $|f|$ 然后 $f'(T,S)-|f|$ 就是最小流。

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
#include <algorithm>
using namespace std;

const int N = 50010, M = 350010, INF = 1e8;
int h[N], e[M], ne[M], g[N], f[M], idx;
int n, m, S, T;
int d[N], cur[N];

void add0(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], f[idx] = c, h[a] = idx++;
}

void add(int a, int b, int c) {
    add0(a, b, c), add0(b, a, 0);
}

bool bfs() {
    queue<int> q;
    memset(d, -1, sizeof d);
    q.push(S), d[S] = 0, cur[S] = h[S];
    
    while (q.size()) {
        int t = q.front(); q.pop();
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (d[j] == -1 && f[i]) {
                d[j] = d[t] + 1;
                cur[j] = h[j];
                if (j == T) return true;
                q.push(j);
            }
        }
    }
    return false;
}

int find(int u, int lim) {
    if (u == T) return lim;
    
    int r = 0;
    for (int i = cur[u]; ~i && r < lim; i = ne[i]) {
        cur[u] = i;
        int j = e[i];
        if (f[i] && d[j] == d[u] + 1) {
            int t = find(j, min(f[i], lim - r));
            r += t, f[i] -= t, f[i^1] += t;
            if (!t) d[j] = -1;
        }
    }
    return r;
}

int dinic() {
    int r = 0, flow;
    while (bfs()) while (flow = find(S, INF)) r += flow;
    return r;
}

int main() {
    int s, t;
    memset(h, -1, sizeof h);
    scanf("%d%d%d%d", &n, &m, &s, &t);
    for (int i = 0; i < m; i++) {
        int a, b, c, d;
        scanf("%d%d%d%d", &a, &b, &c, &d);
        add(a, b, d-c);
        g[a] -= c, g[b] += c;
    }
    S = n+1, T = n+2;
    for (int i = 1; i <= n; i++)
        if (g[i] > 0) add(S, i, g[i]);
        else if (g[i] < 0) add(i, T, -g[i]);
    
    
    add(t, s, INF);
    
    dinic();
    for (int i = h[S]; ~i; i = ne[i]) {
        if (f[i]) {
            puts("No Solution");
            return 0;
        }
    }
    
    S = t, T = s;
    int r1 = f[idx-1];
    f[idx-1] = f[idx-2] = 0;
    
    int r2 = dinic();
    printf("%d\n", r1 - r2);
    return 0;
}
```

## 多源汇最大流

> 给定一个包含 n 个点 m 条边的**有向**图，并给定每条边的容量，边的容量非负。
>
> 其中有 Sc 个源点，Tc 个汇点。
>
> 题目链接：[AcWing 2234](https://www.acwing.com/problem/content/2236/)。

连接 $c(S,S')=c(T',T)=+\infin$ 即可。

 ```cpp
 #include <cstdio>
 #include <queue>
 #include <cstring>
 #include <algorithm>
 using namespace std;
 
 const int N = 20010, M = 250010, INF = 1e8;
 int h[N], e[M], ne[M], f[M], idx;
 int cur[N], d[N];
 int n, m, S, T;
 
 void add0(int a, int b, int c) {
     e[idx] = b, ne[idx] = h[a], f[idx] = c, h[a] = idx++;
 }
 
 void add(int a, int b, int c) {
     add0(a, b, c), add0(b, a, 0);
 }
 
 bool bfs() {
     queue<int> q;
     memset(d, -1, sizeof d);
     d[S] = 0, q.push(S), cur[S] = h[S];
     
     while (q.size()) {
         int t = q.front(); q.pop();
         for (int i = h[t]; ~i; i = ne[i]) {
             int j = e[i];
             if (d[j] == -1 && f[i]) {
                 d[j] = d[t] + 1;
                 cur[j] = h[j]; 
                 if (j == T) return true;
                 q.push(j);
             }
         }
     }
     
     return false;
 }
 
 int find(int u, int lim) {
     if (u == T) return lim;
     
     int r = 0;
     for (int i = cur[u]; ~i && r < lim; i = ne[i]) {
         int j = e[i];
         cur[u] = i;
         if (f[i] && d[j] == d[u] + 1) {
             int t = find(j, min(f[i], lim - r));
             r += t, f[i] -= t, f[i^1] += t;
             if (!t) d[j] = -1;
         }
     }
     
     return r;
 }
 
 int dinic() {
     int r = 0, flow;
     while (bfs()) while (flow = find(S, INF)) r += flow;
     return r;
 }
 
 int main() {
     memset(h, -1, sizeof h);
     int nS, nT;
     scanf("%d%d%d%d", &n, &m, &nS, &nT);
     S = n+nS+nT+1, T = S+1;
     while (nS--) {
         int iS;
         scanf("%d", &iS);
         add(S, iS, INF);
     }
     while (nT--) {
         int iT;
         scanf("%d", &iT);
         add(iT, T, INF);
     }
     while (m--) {
         int a, b, c;
         scanf("%d%d%d", &a, &b, &c);
         add(a, b, c);
     }
     printf("%d\n", dinic());
     return 0;
 }
 ```

