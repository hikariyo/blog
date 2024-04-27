---
date: 2023-12-24 06:44:00
title: 图论 - 线段树优化建图
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 简介

当涉及区间之间的连边操作时，可以考虑使用线段树优化建图，首先建立线段树，让父结点指向子结点或相反，边权为零，然后就会发现无论是最短路还是连通性，都与原图无区别。

## [CF786B] Legacy

> 在宇宙中一共有 $n$ 个星球标号为 $1 \sim n$。Rick 现在身处于标号为 $s$ 的星球(地球)但是他不知道 Morty 在哪里。  
>
> 众所周知，Rick 有一个传送枪，他用这把枪可以制造出一个从他所在的星球通往其他星球(也包括自己所在的星球)的单行道路。但是由于他还在用免费版，因此这把枪的使用是有限制的。
>
> 默认情况下他不能用这把枪开启任何传送门。在网络上有 $q$ 个售卖这些传送枪的使用方案。每一次你想要实施这个方案时你都可以购买它，但是每次购买后只能使用一次。每个方案的购买次数都是无限的。
>
> 网络上一共有三种方案可供购买：   
>
> - 开启一扇从星球 $v$ 到星球 $u$ 的传送门；  
> - 开启一扇从星球 $v$ 到标号在 $[l,r]$ 区间范围内任何一个星球的传送门。(即这扇传送门可以从一个星球出发通往多个星球)    
> - 开启一扇从标号在 $[l,r]$ 区间范围内任何一个星球到星球 $v$ 的传送门。(即这扇传送门可以从多个星球出发到达同一个星球)   
>
> Rick 并不知道 Morty 在哪儿，但是 Unity 将要通知他 Morty 的具体位置，并且他想要赶快找到通往所有星球的道路各一条并立刻出发。因此对于每一个星球(包括地球本身)他想要知道从地球到那个星球所需的最小钱数。
>
> 输入数据的第一行包括三个整数 $n$，$q$ 和 $s$ 分别表示星球的数目，可供购买的方案数目以及地球的标号。
>
> 接下来的 $q$ 行表示 $q$ 种方案。 
>
> - 输入 `1 v u w` 表示第一种方案，其中 $v,u$ 意思同上，$w$ 表示此方案价格。
> - 输入 `2 v l r w` 表示第二种方案，其中 $v,l,r$ 意思同上，$w$ 表示此方案价格。
> - 输入 `3 v l r w` 表示第三种方案，其中 $v,l,r$ 意思同上，$w$ 表示此方案价格。
>
> 对于 $100\%$ 的数据，$1\le n,q \le 10^5$，$1\le w \le 10^9$
>
> 题目链接：[CF786B](http://codeforces.com/problemset/problem/786/B)。

这个题先看一个比较朴素的写法。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define Mid ((L+R) >> 1)
typedef pair<int, int> PII;
const int N = 200010;
int n, q, s, dist[N*10];
bitset<N*10> st;
struct Edge {
    int to, w;
};
vector<Edge> g[N*10];

int idx;
struct SGT {
    int id[N], ls[N<<2], rs[N<<2], rt;

    int build(int L, int R) {
        int u = ++idx;
        if (L == R) return id[L] = u;
        ls[u] = build(L, Mid);
        rs[u] = build(Mid+1, R);
        init(u);
        return u;
    }

    void modify(int u, int l, int r, int to, int w, int L, int R) {
        if (l <= L && R <= r) return process(u, to, w);
        if (l <= Mid) modify(ls[u], l, r, to, w, L, Mid);
        if (Mid+1 <= r) modify(rs[u], l, r, to, w, Mid+1, R);
    }

    virtual void init(int u) = 0;
    virtual void process(int u, int to, int w) = 0;
};

struct SGT1: SGT {
    void init(int u) {
        g[u].push_back({ls[u], 0});
        g[u].push_back({rs[u], 0});
    }

    void process(int u, int to, int w) {
        g[to].push_back({u, w});
    }
} T1;

struct SGT2: SGT {
    void init(int u) {
        g[ls[u]].push_back({u, 0});
        g[rs[u]].push_back({u, 0});
    }

    void process(int u, int to, int w) {
        g[u].push_back({to, w});
    }
} T2;

void dijkstra(int s) {
    priority_queue<PII> heap;
    heap.push({0, s});
    fill(dist+1, dist+1+idx, 1e18);
    dist[s] = 0;
    while (!heap.empty()) {
        int t = heap.top().second; heap.pop();
        if (st[t]) continue;
        st[t] = true;
        for (const auto& e: g[t]) {
            if (dist[e.to] > dist[t] + e.w) {
                dist[e.to] = dist[t] + e.w;
                heap.push({-dist[e.to], e.to});
            }
        }
    }
}

signed main() {
    cin >> n >> q >> s;
    T1.rt = T1.build(1, n);
    T2.rt = T2.build(1, n);
    for (int i = 1; i <= n; i++) {
        g[T1.id[i]].push_back({T2.id[i], 0});
        g[T2.id[i]].push_back({T1.id[i], 0});
    }

    for (int i = 1, opt, v, u, l, r, w; i <= q; i++) {
        cin >> opt;
        if (opt == 1) {
            cin >> v >> u >> w;
            g[T1.id[v]].push_back({T1.id[u], w});
        }
        else if (opt == 2) {
            cin >> v >> l >> r >> w;
            T1.modify(T1.rt, l, r, T1.id[v], w, 1, n);
        }
        else {
            cin >> v >> l >> r >> w;
            T2.modify(T2.rt, l, r, T1.id[v], w, 1, n);
        }
    }
    dijkstra(T1.id[s]);
    for (int i = 1; i <= n; i++) {
        int d = dist[T1.id[i]];
        if (d == 1e18) cout << "-1 ";
        else cout << d << ' ';
    }
    cout << '\n';

    return 0;
}
```

## [PA11] Journeys

> 一个星球上有 $n$ 个国家和许多双向道路，国家用 $1\sim n$ 编号。
>
> 但是道路实在太多了，不能用通常的方法表示。于是我们以如下方式表示道路：$(a,b),(c,d)$ 表示，对于任意两个国家 $x,y$，如果 $a\le x\le b,c\le y\le d$，那么在 $x,y$ 之间有一条道路。
>
> 首都位于 $P$ 号国家。你想知道 $P$ 号国家到任意一个国家最少需要经过几条道路。保证 $P$ 号国家能到任意一个国家。
>
> 对于所有测试点，保证 $1\le n\le 5\times 10^5$，$1\le m\le 10^5$，$1\le a\le b\le n$，$1\le c\le d\le n$。
>
> 题目链接：[P6348](https://www.luogu.com.cn/problem/P6348)。

其实我们可以就建一个树，然后找到这个树上值最大的那个结点，第二个树的对应位置编号就与他差这个值 `offset`。

注意双向边不能复用虚点，因为 $(a,b)\to v\to (c,d)$ 如果有 $(c,d)\to v\to (a,b)$ 那么 $(a,b)$ 和 $(c,d)$ 内部就两两建边了，就不对了。此外本题也可以直接 0/1 BFS 也可以直接 Dijkstra 因为线段树有一个 $\log n$ 所以这个地方对复杂度没啥影响。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((L+R) >> 1)
const int N = 6e6+10, M = 2e7+10;

int n, m, p, dist[N], id[N], h[N], idx = 2;
int offset;
bitset<N> st;

struct Edge {
    int to, w, nxt;
} e[M];

void add(int a, int b, int c) {
    e[idx] = {b, c, h[a]}, h[a] = idx++;
}

void build(int u, int L, int R) {
    offset = max(offset, u);
    if (L == R) return id[L] = u, void();
    build(u<<1, L, Mid), build(u<<1|1, Mid+1, R);
}

void buildedges(int u, int L, int R) {
    if (L == R) return add(u, u+offset, 0), add(u+offset, u, 0);
    add(u, u<<1, 0), add(u, u<<1|1, 0);
    add((u<<1)+offset, u+offset, 0), add((u<<1|1)+offset, u+offset, 0);
    buildedges(u<<1, L, Mid), buildedges(u<<1|1, Mid+1, R);
}

void modify(int u, int p, int w, int l, int r, int L, int R) {
    if (l <= L && R <= r) {
        if (w) add(p, u, w);
        else add(u+offset, p, w);
        return;
    }
    if (l <= Mid) modify(u<<1, p, w, l, r, L, Mid);
    if (Mid+1 <= r) modify(u<<1|1, p, w, l, r, Mid+1, R);
}

void dijkstra(int S) {
    priority_queue<pair<int, int>> heap;
    heap.push({0, S});
    memset(dist, 0x3f, sizeof dist);
    dist[S] = 0;
    while (heap.size()) {
        int t = heap.top().second; heap.pop();
        if (st[t]) continue;
        st[t] = 1;
        for (int i = h[t]; i; i = e[i].nxt) {
            int to = e[i].to, w = e[i].w;
            if (dist[to] > dist[t] + w) {
                dist[to] = dist[t] + w;
                heap.push({-dist[to], to});
            }
        }
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    cin >> n >> m >> p;
    build(1, 1, n+2*m);
    buildedges(1, 1, n+2*m);
    for (int i = 1, a, b, c, d; i <= m; i++) {
        cin >> a >> b >> c >> d;
        modify(1, id[n+i*2-1], 0, a, b, 1, n+2*m);
        modify(1, id[n+i*2-1], 1, c, d, 1, n+2*m);
        modify(1, id[n+i*2], 0, c, d, 1, n+2*m);
        modify(1, id[n+i*2], 1, a, b, 1, n+2*m);
    }
    dijkstra(id[p]);
    for (int i = 1; i <= n; i++) cout << dist[id[i]] << '\n';
    return 0;
}
```

