---
date: 2023-06-16 18:18:18
update: 2023-08-29 21:35:00
title: 图论 - 二分图
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 简介

二分图指的是可以把点放到两个集合中，并且集合中不存在边。染色法可以判断一个图是否为二分图，匈牙利算法可以求出二分图最大匹配。匹配指的是二分图中任意两条边都不依赖于同一个顶点的一个子图，简而言之就是在这两边连线不能重复连同一个点，求这些连线的个数。

## 染色法判定二分图

> 给定一个 n 个点 m 条边的无向图，图中可能存在重边和自环。
>
> 请你判断这个图是否是二分图。

用 DFS 染色，每个节点和它的子节点颜色不能相同，染完所有节点如果没发生冲突就证明它是一个二分图。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int N = 1e5+10, M = 2*N;
int n, m;

// 看到 h 就要记得初始化... 被坑了好多次了
int h[N], e[M], ne[M], idx, color[N];

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

// 传进来的节点一定是没染色的 否则不会被传进来
// 返回染色是否成功
bool dfs(int u, int co) {
    color[u] = co;
    int nco = 3 - co;  // co == 1 ? 2 : 1
    for (int i = h[u]; i != -1; i = ne[i]) {
        int j = e[i];
        if (!color[j]) {
            if (!dfs(j, nco)) return false;
        } else if (color[j] == co) return false;  // 子节点颜色不能与父节点相同
    }
    return true;
}

int main() {
    cin.tie(0), cout.tie(0), ios::sync_with_stdio(false);
    memset(h, -1, sizeof h);
    cin >> n >> m;
    while (m--) {
        int a, b;
        cin >> a >> b;
        add(a, b), add(b, a);
    }
    
    bool flag = true;
    for (int i = 1; i <= n; i++) {
        // 只染没染过色的节点
        if (!color[i]) {
            if (!dfs(i, 1)) {
                flag = false;
                break;
            }
        }
    }
    
    if (flag) cout << "Yes";
    else cout << "No";
    
    return 0;
}
```

## 匈牙利算法

> 给定一个二分图，其左部点的个数为 $n$，右部点的个数为 $m$，边数为 $e$，求其最大匹配的边数。
>
> 左部点从 $1$ 至 $n$ 编号，右部点从 $1$ 至 $m$ 编号。
>
> 对于全部的测试点，保证：
> - $1 \leq n, m \leq 500$。
> - $1 \leq e \leq 5 \times 10^4$。
> - $1 \leq u \leq n$，$1 \leq v \leq m$。
>
> **不保证给出的图没有重边**。
>
> 题目链接：[P3386](https://www.luogu.com.cn/problem/P3386)。

事实证明时间戳优化的匈牙利速度是很优秀的，原理就是不断求增广路径。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 510;
int n1, n2, m;
vector<int> g[N];
int st[N], match[N];
int ts;

bool dfs(int u) {
    for (int to: g[u]) {
        if (st[to] == ts) continue;
        st[to] = ts;
        if (!match[to] || dfs(match[to]))
            return match[to] = u, true;
    }
    return false;
}

int main() {
    scanf("%d%d%d", &n1, &n2, &m);
    for (int i = 1, a, b; i <= m; i++) {
        scanf("%d%d", &a, &b);
        g[a].emplace_back(b);
    }

    int ans = 0;
    for (int i = 1; i <= n1; i++) {
        ++ts;
        ans += dfs(i);
    }

    printf("%d\n", ans);

    return 0;
}
```

## 最大流算法

> 题目和上面一样，同样是求二分图的最大匹配。

关于最大流算法的模板，可以参考本站关于网络流的文章，这里简述一下如何建图。

设源点，汇点分别为 $s,t$ 对于左边的每一个点，从源点出发连一条流量为 1 的边；对于右边的每一个点，连一条到达汇点流量为 1 的边。然后左边和右边的每一条边都连一条流量为 1 的边。每一个可行流都是一个合法匹配，所以这里就是求最大流，用 Dinic 算法。

```cpp
#include <cstdio>
#include <algorithm>
#include <queue>
#include <cstring>
using namespace std;

const int N = 1010, M = 600010;
int n1, n2, m, S = 1, T = 2, INF = 1e8;
int h[N], e[M], ne[M], f[M], idx;
int d[N], cur[N], q[N];
bool st[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], f[idx] = c, h[a] = idx++;
}

bool bfs() {
    memset(d, -1, sizeof d);
    int hh = 0, tt = 0;
    q[0] = S, d[S] = 1, cur[S] = h[S];
    
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
    for (int i = cur[u]; ~i && flow < lim; i = ne[i]) {
        cur[u] = i;
        int j = e[i];
        if (f[i] && d[j] == d[u] + 1) {
            int t = find(j, min(f[i], lim - flow));
            if (!t) d[j] = -1;
            f[i] -= t, f[i^1] += t, flow += t;
        }
    }
    return flow;
}

int dinic() {
    int r = 0, flow;
    while (bfs()) {
        while (flow = find(S, INF)) r += flow;
    }
    return r;
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d%d", &n1, &n2, &m);
    while (m--) {
        int a, b;
        scanf("%d%d", &a, &b);
        a += 2;
        b += n1 + 2;
        if (!st[a]) st[a] = true, add(S, a, 1), add(a, S, 0);
        if (!st[b]) st[b] = true, add(b, T, 1), add(T, b, 0);
        add(a, b, 1), add(b, a, 0);
    }
    
    printf("%d\n", dinic());
    return 0;
}
```



## [NOIP2010] 关押罪犯

> S 城现有两座监狱，一共关押着 N 名罪犯，编号分别为 1∼N。
>
> 他们之间的关系自然也极不和谐。
>
> 很多罪犯之间甚至积怨已久，如果客观条件具备则随时可能爆发冲突。
>
> 我们用“怨气值”（一个正整数值）来表示某两名罪犯之间的仇恨程度，怨气值越大，则这两名罪犯之间的积怨越多。
>
> 如果两名怨气值为 c 的罪犯被关押在同一监狱，他们俩之间会发生摩擦，并造成影响力为 c 的冲突事件。
>
> 每年年末，警察局会将本年内监狱中的所有冲突事件按影响力从大到小排成一个列表，然后上报到 S 城 Z 市长那里。
>
> 公务繁忙的 Z 市长只会去看列表中的第一个事件的影响力，如果影响很坏，他就会考虑撤换警察局长。
>
> 在详细考察了 N 名罪犯间的矛盾关系后，警察局长觉得压力巨大。
>
> 他准备将罪犯们在两座监狱内重新分配，以求产生的冲突事件影响力都较小，从而保住自己的乌纱帽。
>
> 假设只要处于同一监狱内的某两个罪犯间有仇恨，那么他们一定会在每年的某个时候发生摩擦。
>
> 那么，应如何分配罪犯，才能使 Z 市长看到的那个冲突事件的影响力最小？这个最小值是多少？
>
> 题目链接：[AcWing 257](https://www.acwing.com/problem/content/259/)，[P1525](https://www.luogu.com.cn/problem/P1525)。

既然是求一个最大值的最小值，第一想法是二分答案判断合法性，那么判断合法性的方式就是看严格大于二分出来结果的所有边能否构成一个二分图。如果能构成二分图，两边的点就不在同一个集合中，满足要求。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int N = 20010, M = 200010;

int h[N], e[M], ne[M], w[M], idx;
int n, m;
int color[N];


void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

bool dfs(int u, int co, int p) {
    color[u] = co;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (w[i] <= p) continue;
        if (color[j] && color[j] == co) return false;
        if (!color[j] && !dfs(j, -co, p)) return false;
    }
    return true;
}

bool check(int p) {
    memset(color, 0, sizeof color);
    for (int i = 1; i <= n; i++)
        if (!color[i] && !dfs(i, 1, p)) return false;

    return true;
}


int main() {
    cin >> n >> m;
    
    int l = 0, r = 0;
    memset(h, -1, sizeof h);
    for (int i = 0; i < m; i++) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c), add(b, a, c);
        r = max(r, c);
    }
    
    while (l < r) {
        int mid = l+r>>1;
        if (check(mid)) r = mid;
        else l = mid+1;
    }
    cout << l << endl;
    return 0;
}
```

## [SCOI2010] 连续攻击游戏

> lxhgww 最近迷上了一款游戏，在游戏里，他拥有很多的装备，每种装备都有 $2$ 个属性，这些属性的值用 $[1,10000]$ 之间的数表示。当他使用某种装备时，他只能使用该装备的某一个属性。并且每种装备最多只能使用一次。游戏进行到最后，lxhgww 遇到了终极 boss，这个终极 boss 很奇怪，攻击他的装备所使用的属性值必须从 $1$ 开始连续递增地攻击，才能对 boss 产生伤害。也就是说一开始的时候，lxhgww 只能使用某个属性值为 $1$ 的装备攻击 boss，然后只能使用某个属性值为 $2$ 的装备攻击 boss，然后只能使用某个属性值为 $3$ 的装备攻击 boss……以此类推。现在 lxhgww 想知道他最多能连续攻击 boss 多少次？
>
> **输入格式**
>
> 输入的第一行是一个整数 $N$，表示 lxhgww 拥有 $N$ 种装备接下来 $N$ 行，是对这 $N$ 种装备的描述，每行 $2$ 个数字，表示第 $i$ 种装备的 $2$ 个属性值。
>
> **输出格式**
>
> 输出一行，包括 $1$ 个数字，表示 lxhgww 最多能连续攻击的次数。
>
> 对于 $100\%$ 的数据，保证 $N \le 10^6$。
>
> 题目链接：[P1640](https://www.luogu.com.cn/problem/P1640)。

显然可以将装备与属性值连边，然后依次递增匹配，直到无法匹配，匈牙利算法即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int  N = 10010, M = 2000010, K = 1000010;
int h[N], e[M], ne[M], idx;
int match[K], chk[K];
int n, timestamp;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

bool find(int u) {
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (chk[j] == timestamp) continue;
        chk[j] = timestamp;
        if (!match[j] || find(match[j])) {
            match[j] = u;
            return true;
        }
    }
    return false;
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d", &n);

    for (int i = 1, a, b; i <= n; i++) {
        scanf("%d%d", &a, &b);
        add(a, i), add(b, i);
    }

    int id = 0;
    timestamp = 1;
    while (find(id+1)) id++, timestamp++;
    return !printf("%d\n", id);
}
```

## [CEOI2002] Royal guards

> 从前有一个王国，这个王国的城堡是 $m$ 行 $n$ 列的一个矩形，被分为 $m \times n$ 个方格。一些方格是墙，而另一些是空地。这个王国的国王在城堡里设了一些陷阱，每个陷阱占据一块空地。
>
> 一天，国王决定在城堡里布置守卫，他希望安排尽量多的守卫。
>
> 守卫们都是经过严格训练的，所以一旦他们发现同行或同列中有人的话，他们立即向那人射击。因此，国王希望能够合理地布置守卫，使他们互相之间不能看见，这样他们就不可能互相射击了。守卫们只能被布置在空地上，不能被布置在陷阱或墙上，且一块空地只能布置一个守卫。如果两个守卫在同一行或同一列，并且他们之间没有墙的话，他们就能互相看见。(守卫就像象棋里的车一样)
>
> 你的任务是写一个程序，根据给定的城堡，计算最多可布置多少个守卫，并设计出布置的方案。
>
> 对于全部的测试点，保证 $1 \leq m, n \leq 200$，$0 \leq a_{i, j} \leq 2$。
>
> 题目链接：[P1263](https://www.luogu.com.cn/problem/P1263)。

我们要考虑什么东西和什么东西连边，这样使得求出来的最大匹配可以对应实际问题。显然我们要求的是最大满足要求的格点数量，那么要使得一个匹配对应一个格点，可以想到把地图划分成若干小矩形，相交的两两连边，这样就好了。实现起来比较麻烦。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 210, M = 40010;
int n, m;
vector<int> g[M];
int hidx, vidx;
int hpos[M], vpos[M];
int a[N][N], id[N][N], idx;
int hori[N][N], vert[N][N];
int match[M], st[M], ts;

void getblocks() {
    // horizontal
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            if (a[i][j] == 2) continue;
            int id = ++hidx;
            hpos[id] = i;
            int k = j;
            while (k <= m && a[i][k] != 2) hori[i][k++] = id;
            j = k-1;
        }
    }

    // vertical
    for (int j = 1; j <= m; j++) {
        for (int i = 1; i <= n; i++) {
            if (a[i][j] == 2) continue;
            int id = ++vidx;
            vpos[id] = j;
            int k = i;
            while (k <= n && a[k][j] != 2) vert[k++][j] = id;
            i = k-1;
        }
    }
}

bool dfs(int u) {
    for (int to: g[u]) {
        if (st[to] == ts) continue;
        st[to] = ts;
        if (!match[to] || dfs(match[to]))
            return match[to] = u, true;
    }
    return false;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            scanf("%d", &a[i][j]);
            id[i][j] = ++idx;
        }
    }
    getblocks();

    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            if (a[i][j] == 0)
                g[hori[i][j]].emplace_back(vert[i][j]);
    
    int ans = 0;
    for (int i = 1; i <= hidx; i++) {
        ++ts;
        ans += dfs(i);
    }

    printf("%d\n", ans);
    for (int i = 1; i <= vidx; i++)
        if (match[i]) printf("%d %d\n", hpos[match[i]], vpos[i]);

    return 0;
}
```

