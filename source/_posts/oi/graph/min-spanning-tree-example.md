---
date: 2023-12-03 17:31:00
title: 图论 - 最小生成树例题
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
description: 最小生成树例题练习，主要看如何将问题转化成最小生成树的模型，然后进行求解。
---

## [CF1245D] Shichikuji and Power Grid

> 给定 $n$ 个城市的坐标 $(x_i,y_i)$，在这个城市建设发电站的价格 $c_i$，城市 $i,j$ 连接的代价是 $w_{ij}=(k_i+k_j)(|x_i-x_j|+|y_i-y_j|)$。输出最小代价，建造发电站的城市数，和每个城市连线的条数，和每条连线。任意一种即可，输出顺序任意。
>
> 题目链接：[CF1245D](https://codeforces.com/problemset/problem/1245/D)。

可以建一个虚拟原点，它到所有结点的距离就是在这个结点上建发电站的费用，**将点权转化为边权**，这样就可以直接用最小生成树做了。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 2010, M = N*N;
typedef pair<int, int> PII;
int n, x[N], y[N], c[N], k[N], p[N];
struct Edge {
	int a, b;
	LL w;
	bool operator<(const Edge& e) const {
		return w < e.w;
	}
} edg[M];

int find(int x) {
	if (x != p[x]) p[x] = find(p[x]);
	return p[x];
}

LL getdis(int a, int b) {
	if (a > b) swap(a, b);
	if (a == 0) return c[b];
	return (LL)(k[a] + k[b]) * (abs(x[a] - x[b]) + abs(y[a] - y[b]));
}

void kruskal() {
	int m = 0;
	for (int i = 1; i <= n; i++) {
		edg[m++] = {0, i, getdis(0, i)};
	}
	for (int i = 1; i <= n; i++) {
		for (int j = i+1; j <= n; j++) {
			edg[m++] = {i, j, getdis(i, j)};
		}
	}
	sort(edg, edg+m);
	for (int i = 1; i <= n; i++) p[i] = i;
	
	vector<PII> ans;
	LL cost = 0;
	for (int i = 0; i < m; i++) {
		int a = find(edg[i].a), b = find(edg[i].b);
		if (a != b) {
			p[a] = b;
			cost += edg[i].w;
			ans.push_back({edg[i].a, edg[i].b});
		}
	}
	sort(ans.begin(), ans.end());
	int cnt = 0;
	for (const PII& p: ans) {
		if (p.first == 0) cnt++;
	}
	
	printf("%lld\n%d\n", cost, cnt);
	for (int i = 0; i < cnt; i++) {
		printf("%d ", ans[i].second);
	}
	printf("\n%d\n", ans.size() - cnt);
	for (int i = cnt; i < ans.size(); i++) {
		printf("%d %d\n", ans[i].first, ans[i].second);
	}
}

int main() {
	scanf("%d", &n);
	for (int i = 1; i <= n; i++) scanf("%d%d", &x[i], &y[i]);
	for (int i = 1; i <= n; i++) scanf("%d", &c[i]);
	for (int i = 1; i <= n; i++) scanf("%d", &k[i]);
	return kruskal(), 0;
}
```

## 最小生成树中第 k 大边

> 北极的某区域共有 n 座村庄，每座村庄的坐标用一对整数 (x,y) 表示。
>
> 为了加强联系，决定在村庄之间建立通讯网络，使每两座村庄之间都可以直接或间接通讯。
>
> 通讯工具可以是无线电收发机，也可以是卫星设备。
>
> 无线电收发机有多种不同型号，不同型号的无线电收发机有一个不同的参数 d，两座村庄之间的距离如果不超过 d，就可用该型号的无线电收发机直接通讯，d 值越大的型号价格越贵。现在要先选择某一种型号的无线电收发机，然后统一给所有庄配备，**数量不限**，但型号都是 **相同的**。
>
> 配备卫星设备的两座村庄无论相距多远都可以直接通讯，但卫星设备是 **有限的**，只能给一部分村庄配备。
>
> 现在有 k 台卫星设备，请你编一个程序，计算出应该如何分配这 k 台卫星设备，才能使所配备的无线电收发机的 d 值最小。
>
> 原题链接：[AcWing 1145](https://www.acwing.com/problem/content/1147/)。

用 kruskal 算法连 $n-k$ 条边就行，例如有 5 个点，求第 2 大边，那么连 5-2=3 条边后这三条边里的最大值就是第 k 大边。

```cpp
#include <iostream>
#include <algorithm>
#include <cmath>
using namespace std;

const int N = 510, M = N*N;

struct point {
    int x, y;
    
    double dist(point p) {
        int dx = x-p.x, dy = y-p.y;
        return sqrt(dx*dx+dy*dy);
    }
} q[N];

struct edge {
    int a, b;
    double w;
    bool operator<(const edge &e) const {
        return w < e.w;
    }
} e[M];

int p[N], n, m, k;

int find(int x) {
    if (x != p[x]) p[x] = find(p[x]);
    return p[x];
}

int main() {
    cin >> n >> k;
    for (int i = 1; i <= n; i++) {
        p[i] = i;
        cin >> q[i].x >> q[i].y;
        for (int j = 1; j < i; j++) {
            e[m++] = {i, j, q[j].dist(q[i])};
        }
    }
    sort(e, e+m);
    int cnt = 0;
    double res = 0;
    for (int i = 0; i < m && cnt < n-k; i++) {
        int a = find(e[i].a), b = find(e[i].b);
        if (a != b) {
            p[a] = b;
            cnt++;
            res = e[i].w;
        }
    }
    printf("%.2lf\n", res);
    return 0;
}
```

## 最小生成树的边权和最小完全图

> 给定一棵 N 个节点的树，要求增加若干条边，把这棵树扩充为完全图，并满足图的唯一最小生成树仍然是这棵树。
>
> 求增加的边的权值总和最小是多少。
>
> **注意：** 树中的所有边权均为整数，且新加的所有边权也必须为整数。
>
> 原题链接：[AcWing 346](https://www.acwing.com/problem/content/348/)。

最小生成树中的一条边连接左右两个连通块，当将它们合并成一个局部的完全图时，需要增加的边数为：
$$
\text{size}(l)\times \text{size}(r)-1
$$
同时这些边的边权 $w$ 需要严格大于已经存在的边，因此 $w=w_0+1$ 时这个完全图的边权之和最小。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 7010;
int p[N], sz[N], n;
struct edge {
    int a, b, w;
    bool operator<(const edge& e) const {
        return w < e.w;
    }
} e[N];

int find(int x) {
    if (x != p[x]) p[x] = find(p[x]);
    return p[x];
}

int main() {
    int T; cin >> T;
    while (T--) {
        cin >> n;
        for (int i = 1; i <= n; i++) p[i] = i, sz[i] = 1;
        
        for (int i = 0; i < n-1; i++) {
            cin >> e[i].a >> e[i].b >> e[i].w;
        }
        sort(e, e+n-1);
        int res = 0;
        for (int i = 0; i < n-1; i++) {
            int a = find(e[i].a), b = find(e[i].b);
            if (a != b) {
                p[a] = b;
                res += (sz[b]*sz[a]-1)*(e[i].w+1);
                sz[b] += sz[a];
            }
        }
        cout << res << endl;
    }
    return 0;
}
```

## [BJWC2010] 严格次小生成树

> 给 N 个点 M 条边的无向图，求严格次小生成树的边权之和，数据保证一定有解。
>
> 原题链接：[P4180](https://www.luogu.com.cn/problem/P4180)，[AcWing 1148](https://www.acwing.com/problem/content/1150/)。

方法：

1. 先求最小生成树，再枚举删掉某条边后求最小生成树。$O(m\log m+nm)$
2. 先求最小生成树，再枚举非树边，将该边加入树中同时去掉另一条边，使得最终仍然是一颗生成树。如果暴力预处理出最小生成树两点路径上边权最大值是 $O(n^2)$，总复杂度为 $O(m\log m +n^2+m)$

这里方法 2 中在顶点 $a, b$ 见的非树边 $w_0$ 一定大于等于包含 $a,b$ 的所有树边，即 $w_{0}\ge w_t$，这可以用反证法证明：假设 $\exist w_0 \lt w_t$，那么删掉 $w_t$ 连上 $w_0$ 就构造出一个比最小生成树还小的生成树，矛盾，得证。

但是 $w_0=\max w_t$ 是一个可能发生的情况。设最小生成树边权之和为 $S$，那么次小生成树边权之和应该为：
$$
S'=\min_{a,b} \{S-w_t+w_0\}>S
$$
所以这里 $w_t$ 不能只取最大值，当 $w_0 =\max w_t$ 时，就要让 $w_t$ 取一个次大值。

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long ll;
const int N = 510, M = 10010, K = 2*N;

struct edge {
    int a, b, w;
    bool f;  // 是否为树边
    bool operator<(const edge& e) const {
        return w < e.w;
    }
} edges[M];
int p[N], n, m;

// dist1: 最大值, dist2: 次大值
int h[N], w[K], e[K], ne[K], idx, dist1[N][N], dist2[N][N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

// 求从 u 出发路径上的最大值和次大值
void dfs(int u, int par, int maxd1, int maxd2, int dist1[], int dist2[]) {
    dist1[u] = maxd1;
    dist2[u] = maxd2;
    
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == par) continue;
        // 这个地方不要直接改 maxd1 maxd2 因为下一轮循环要用原始数据
        int t1 = maxd1, t2 = maxd2;
        if (w[i] > t1) t2 = t1, t1 = w[i];
        else if (t2 < w[i] && w[i] < t1) t2 = w[i];
        dfs(j, u, t1, t2, dist1, dist2);
    }
}

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) p[i] = i;
    memset(h, -1, sizeof h);
    
    for (int i = 0; i < m; i++) {
        int a, b, w; cin >> a >> b >> w;
        edges[i] = {a, b, w};
    }
    sort(edges, edges+m);
    
    // 最多 500 个结点 每条边的范围是 1~1e9 所以会爆 int
    ll S = 0;
    for (int i = 0; i < m; i++) {
        int a = edges[i].a, b = edges[i].b, w = edges[i].w;
        int pa = find(a), pb = find(b);
        if (pa != pb) {
            p[pa] = pb;
            S += w;
            edges[i].f = true;
            add(a, b, w), add(b, a, w);
        }
    }
    
    // 默认最大距离设成 -1e9 要不然不知道是真的 0 还是默认为 0
    for (int i = 1; i <= n; i++) dfs(i, -1, -1e9, -1e9, dist1[i], dist2[i]);
    
    // 枚举非树边
    ll res = 1e18;
    for (int i = 0; i < m; i++) {
        if (edges[i].f) continue;
        
        int a = edges[i].a, b = edges[i].b, w = edges[i].w;
        // S - wt + w > S
        if (w > dist1[a][b]) {
            res = min(res, S-dist1[a][b]+w);
        } else if (w > dist2[a][b]) {
            res = min(res, S-dist2[a][b]+w);
        }
    }
    cout << res << endl;
    return 0;
}
```

这里 DFS 时初始最大值设为 $-10^9$ 的意图是防止下面的这种情况：

![](/img/2023/07/acw1148.jpg)

对于这个图的次小生成树，应当是去掉 $(2, 4)$ 加上 $(1, 4)$ 构成的边权和为 $30$ 的生成树。

但如果初始化的最大值是 $0$，那么会使得 $3 \to 1 \to 2 \to 4$ 这条路径的次大值为 $0$，实际上应该是 $-\infin$，因为压根没有次大值。

当枚举到非树边 $(1, 4)$ 时，一切正常：
$$
\text{res}=S-\text{dist}_1(1, 4)+w(1, 4)=30
$$


而当枚举到非树边 $(3, 4)$ 时：
$$
\text{dist}_1(3, 4)=5\\
\text{dist}_2(3, 4)=0\\
\text{res}=S-\text{dist}_2(3,4)+w(3,4)=20
$$
这显然错了，因此初始化不能传 $0$，将 $\text{dist}_2(3, 4)$ 设成 $-10^9$ 就可以避免这个错误。

## [CF827D] Best Edge Weight

> 给定一个 $n$ 点 $m$ 边，不含重边和自环，每条边权值为 $w_i$ ，求出每个边边权最大是多少时它仍能出现在所有最小生成树上，如果无论多大都一定出现就输出 $-1$。
>
> 数据范围：$2\le n\le 2\times 10^5,n-1\le m\le 2\times 10^5,1\le w\le 10^9$
>
> 题目链接：[CF827D](https://codeforces.com/problemset/problem/827/D)。

首先先求出最小生成树。

+ 对于树边，它一定要比所有的非树边都小，才能保证一定出现在最小生成树上，所以答案是 $\min_{(i,j)\notin E_{MST}} w_{ij}-1$
+ 对于非树边，它一定要比 $a,b$ 在最小生成树的路径上的所有边都要小，所以答案是 $\min_{(i,j)\in E_{MST}} w_{ij}-1$

所以我们需要做的就是处理若干条边，给树上的路径整体取 $\min$，和求一条路径上的边权最小值，显然可以用树剖或者 LCT 做，但是这些我都不会。

可以通过倍增的思路，每次处理 $O(\log n)$ 的数据，最后再用 $O(n\log n)$ 的复杂度下放下去，复杂度也不比 LCT 大。注意最后下放的时候需要从底向上下放，这里我用了一个小 trick，就是先用 `iota` 构造一个排列，然后按照树的深度排序，这样就能做到从底向上下放数据了。

最后输出的时候按照输入边的顺序，所以在 Kruskal 需要的边结构体中存了一下边的编号，最后按照编号排个序就行了。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 200010, M = 400010, K = 19;
int n, m;
int h[N], e[M], ne[M], w[M], idx;
int p[N], fa[N][K], d[N][K], path[N][K], dep[N];

struct Edge {
    int idx, a, b, w;
    bool used;
};
vector<Edge> edge;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

void kruskal() {
    sort(edge.begin(), edge.end(), [](const Edge& a, const Edge& b){
        return a.w < b.w;
    });

    for (auto& ed: edge) {
        int a = ed.a, b = ed.b, w = ed.w;
        if (find(a) != find(b)) {
            p[find(a)] = find(b);
            add(a, b, w), add(b, a, w);
            ed.used = true;
        }
    }
}

void initdfs(int u, int father, int weight) {
    fa[u][0] = father;
    d[u][0] = weight;
    dep[u] = dep[father] + 1;
    for (int k = 1; k < K; k++) {
        int v = fa[u][k-1];
        fa[u][k] = fa[v][k-1];
        d[u][k] = max(d[u][k-1], d[v][k-1]);
    }

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == father) continue;
        initdfs(j, u, w[i]);
    }
}

void modify(int a, int b, int w) {
    if (dep[a] < dep[b]) swap(a, b);
    for (int k = K-1; k >= 0; k--) {
        if (dep[fa[a][k]] >= dep[b]) {
            path[a][k] = min(path[a][k], w);
            a = fa[a][k];
        }
    }
    if (a == b) return;
    for (int k = K-1; k >= 0; k--) {
        if (fa[a][k] != fa[b][k]) {
            path[a][k] = min(path[a][k], w);
            path[b][k] = min(path[b][k], w);
            a = fa[a][k], b = fa[b][k];
        }
    }
    path[a][0] = min(path[a][0], w);
    path[b][0] = min(path[b][0], w);
}

int getmaxd(int a, int b) {
    if (dep[a] < dep[b]) swap(a, b);

    int res = 0;
    for (int k = K-1; k >= 0; k--) {
        if (dep[fa[a][k]] >= dep[b]) {
            res = max(res, d[a][k]);
            a = fa[a][k];
        }
    }
    if (a == b) return res;

    for (int k = K-1; k >= 0; k--) {
        if (fa[a][k] != fa[b][k]) {
            res = max(res, max(d[a][k], d[b][k]));
            a = fa[a][k], b = fa[b][k];
        }
    }
    return max(res, max(d[a][0], d[b][0]));
}

void update() {
    for (auto& ed: edge) {
        if (ed.used) continue;
        modify(ed.a, ed.b, ed.w);
    }

    vector<int> nums(n);
    iota(nums.begin(), nums.end(), 1);
    sort(nums.begin(), nums.end(), [](int a, int b){
        return dep[a] > dep[b];
    });
    for (int u: nums) {
        for (int k = K-1; k; k--) {
            int v = fa[u][k-1];
            // path[u][k] -> path[u][k-1], path[v][k-1]
            path[u][k-1] = min(path[u][k-1], path[u][k]);
            path[v][k-1] = min(path[v][k-1], path[u][k]);
        }
    }
}

int main() {
    memset(h, -1, sizeof h);
    memset(path, 0x3f, sizeof path);
    scanf("%d%d", &n, &m);
    iota(p+1, p+n+1, 1);

    for (int i = 0, a, b, c; i < m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        edge.push_back({i, a, b, c});
    }

    kruskal();
    initdfs(1, 0, 0);
    update();

    sort(edge.begin(), edge.end(), [](const Edge& p, const Edge& q){
        return p.idx < q.idx;
    });
    for (auto& ed: edge) {
        if (!ed.used) printf("%d ", getmaxd(ed.a, ed.b) - 1);
        else {
            int a = ed.a, b = ed.b;
            if (dep[a] < dep[b]) swap(a, b);
            int res = path[a][0]-1;
            if (path[a][0] == 0x3f3f3f3f) res = -1;
            printf("%d ", res);
        }
    }
    return puts(""), 0;
}
```

## [PA2014] Kuglarz

> 魔术师的桌子上有 $n$ 个杯子排成一行，编号为 $1,2,…,n$，其中某些杯子底下藏有一个小球，如果你准确地猜出是哪些杯子，你就可以获得奖品。
>
> 花费 $c_{ij}$ 元，魔术师就会告诉你杯子 $i,i+1,…,j$ 底下藏有球的总数的奇偶性。
>
> 采取最优的询问策略，你至少需要花费多少元，才能保证猜出哪些杯子底下藏着球？
>
> **输入格式**
>
> 第一行一个整数 $n$。
>
> 第 $i+1$ 行（$1\le i\le n$）有 $n+1-i$ 个整数，表示每一种询问所需的花费。
>
> 其中 $c_{ij}$（对区间 $[i,j]$ 进行询问的费用，$1\le i\le j\le n$）为第 $i+1$ 行第 $j+1-i$ 个数。
>
> **输出格式**
>
> 输出一个整数，表示最少花费。
>
> 对于 $100\%$ 的数据，$1\le n\le 2\times 10^3$，$1\le c_{ij}\le 10^9$。
>
> 题目链接：[P5994](https://www.luogu.com.cn/problem/P5994)。

询问 $l\sim r$ 的奇偶性相当于知道了 $S_{l-1},S_r$ 奇偶性是否相同，而 $S_0=0$ 已知，所以只要知道 $S_1\sim S_n$ 即可，也就是每个 $l-1\sim r$ 都连接边权为 $c_{lr}$ 的边，然后求最小生成树。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 2010;
int c[N][N], p[N];
int n;

struct Edge {
    int a, b, w;

    bool operator<(const Edge& e) const {
        return w < e.w;
    }
};

int find(int x) {
    if (x != p[x]) p[x] = find(p[x]);
    return p[x];
}

int main() {
    vector<Edge> edges;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        p[i] = i;
        for (int j = i; j <= n; j++)
            cin >> c[i][j], edges.push_back({i-1, j, c[i][j]});
    }
    
    LL res = 0;
    sort(edges.begin(), edges.end());
    for (const auto& edge: edges) {
        int a = edge.a, b = edge.b;
        if (find(a) != find(b)) {
            p[find(a)] = find(b);
            res += edge.w;
        }
    }

    return !printf("%lld\n", res);
}
```

## [CF1120D] Power Tree

> 给定一棵 $n$ 个点的有根树，$1$ 为根。定义 $u$ 为叶子当且仅当它**不是根**且度数为 $1$。
>
> 你可以选择花费 $w_i$ 的代价控制点 $i$。当一个点 $i$ 被控制时你可以选择给它的子树内的叶子的点权都加上一个自己选定的值 $v_i$ 。你需要控制若干个点，使得花费的代价尽量少，且无论怎样规定所有叶子的初始点权，都可以通过调整你选择的点的值 $v_i$ 来让所有叶子的点权变为 $0$。
>
> 你需要输出最小代价和并构造**所有**最优选取方案的**并集**。
>
> $n\le 2\times 10^5$。
>
> 题目链接：[CF1120D](https://codeforces.com/problemset/problem/1120/D)。

很容易想到控制一个节点等同于控制叶子差分序列中的 $[l,r+1]$，于是我们想要把所有点变成 $0$，而每次只能将一个点的所有值转移到另一个点上，也就是能将 $a_l$ 转移到 $a_{r+1}$ 上，于是我们对所有的 $l_i,r_i+1$ 连边，边权为 $w_i$，其中 $l_i,r_i$ 都是结点 $i$ 所能到达的叶子结点的 $dfn$ 最大值和最小值，只有叶子结点会记 $dfn$。

最后将所有数字都转移到 $dfn$ 为 $tot+1$ 的点上即可，显然叶子结点的数量最多只有 $n-1$，所以我们求 MST 的时候认为点权 $\le n$ 即可。

由于我们要取并集，所以建树后还要判断非树边是不是也有可能处于 MST 中，这里使用倍增求路径上最大值解决。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 200010, K = 20, INF = 0x3f3f3f3f;
int n;

struct DSU {
    int p[N];

    void init(int n) {
        for (int i = 1; i <= n; i++) p[i] = i;
    }

    int find(int x) {
        if (p[x] != x) p[x] = find(p[x]);
        return p[x];
    }

    void merge(int a, int b) {
        p[find(a)] = find(b);
    }
} dsu;


struct Graph2 {
    struct Edge {
        int to, nxt, w, id;
    } e[N<<1];
    int h[N], dep[N], fa[N][K], d[N][K], idx = 2;

    void add(int a, int b, int c, int id) {
        e[idx] = {b, h[a], c, id}, h[a] = idx++;
    }

    void dfs(int u, int f, int from) {
        fa[u][0] = f, d[u][0] = from, dep[u] = dep[f] + 1;
        for (int k = 1; k < K; k++) {
            fa[u][k] = fa[fa[u][k-1]][k-1];
            d[u][k] = max(d[u][k-1], d[fa[u][k-1]][k-1]);
        }
        for (int i = h[u]; i; i = e[i].nxt) {
            int to = e[i].to, w = e[i].w;
            if (to == f) continue;
            dfs(to, u, w);
        }
    }

    int maxd(int a, int b) {
        int res = 0;
        if (dep[a] < dep[b]) swap(a, b);
        for (int k = K-1; k >= 0; k--)
            if (dep[fa[a][k]] >= dep[b])
                res = max(res, d[a][k]), a = fa[a][k];
        
        if (a == b) return res;
        for (int k = K-1; k >= 0; k--)
            if (fa[a][k] != fa[b][k])
                res = max(res, max(d[a][k], d[b][k])),
                a = fa[a][k], b = fa[b][k];
        
        return max(res, max(d[a][0], d[b][0]));
    }
} G2;

struct Kruskal {
    struct Edge {
        int a, b, c, id;
        bool used;
        bool operator<(const Edge& ed) const {
            return c < ed.c;
        }
    } e[N];
    bool ans[N];
    int idx;

    void add(int a, int b, int c, int id) {
        e[++idx] = {a, b, c, id};
    }

    long long build() {
        long long res = 0;
        sort(e+1, e+1+idx);
        dsu.init(n);

        for (int i = 1; i <= idx; i++) {
            int a = e[i].a, b = e[i].b, c = e[i].c, id = e[i].id;
            if (dsu.find(a) == dsu.find(b)) continue;
            dsu.merge(a, b);
            G2.add(a, b, c, id), G2.add(b, a, c, id);
            e[i].used = true;
            res += c;
        }

        return res;
    }

    int solve() {
        int tot = 0;
        G2.dfs(1, 0, 0);
        for (int i = 1; i <= idx; i++) {
            if (e[i].used || G2.maxd(e[i].a, e[i].b) == e[i].c) {
                ans[e[i].id] = true;
                tot++;
            }
        }
        return tot;
    }
} krus;

struct Graph1 {
    int h[N], w[N], dfn[N], l[N], r[N], idx = 2, ts;
    bool leaf[N];

    struct Edge {
        int to, nxt;
    } e[N<<1];

    void add(int a, int b) {
        e[idx] = {b, h[a]}, h[a] = idx++;
    }

    void dfs(int u, int f) {
        leaf[u] = true;
        l[u] = INF, r[u] = 0;
        for (int i = h[u]; i; i = e[i].nxt) {
            int to = e[i].to;
            if (to == f) continue;
            dfs(to, u);
            leaf[u] = false;
            l[u] = min(l[u], l[to]);
            r[u] = max(r[u], r[to]);
        }

        if (leaf[u]) l[u] = r[u] = dfn[u] = ++ts;
    }

    void build() {
        for (int i = 1; i <= n; i++) {
            krus.add(l[i], r[i]+1, w[i], i);
        }
    }
} G1;

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &G1.w[i]);
    for (int i = 1, a, b; i < n; i++) scanf("%d%d", &a, &b), G1.add(a, b), G1.add(b, a);
    G1.dfs(1, 0);
    G1.build();
    long long res = krus.build();
    int tot = krus.solve();
    printf("%lld %d\n", res, tot);
    for (int i = 1; i <= n; i++)
        if (krus.ans[i])
            printf("%d ", i);
    return 0;
}
```

## [CF773F] Drivers Dissatisfaction
> 给出一张 $n$ 个点 m 条边的无向图，每条边 $(a_i,b_i)$ 有一个权值 $w_i$ 和费用 $c_i$，表示这条边 每降低 $1$ 的权值需要 $c_i$ 的花费。现在一共有 $S$ 费用可以用来降低某些边的权值（可以降到 负数），求图中的一棵权值和最小的生成树并输出方案。
>
> 题目链接：[CF773F](https://codeforces.com/problemset/problem/733/F)。

首先，我们可以证明一定存在方案使得所有费用都花到同一条边上。

假设我们花到 $i,j$ 分别 $s_i, s_j$ 的费用，那么它对答案的影响是，设原最小生成树边权和为 $k$，分情况考虑

1. 全是树边，那么新的最小生成树边权和为：
   $$
   k-\lfloor\frac{s_i}{c_i}\rfloor-\lfloor\frac{s_j}{c_j}\rfloor
   $$
   那么显然是放到 $c$ 更小的边上一定不会更劣。

2. 全是非树边，如果降了之后也不在树上显然是没用的，所以我们设降了之后就能在 MST 上，那么新的边权和为：
   $$
   k+w_i+w_j-\lfloor\frac{s_i}{c_i}\rfloor-\lfloor\frac{s_j}{c_j}\rfloor
   $$
   那么放到 $c$ 更小的边上，不仅后半部分不会更劣，而且还少了一个 $w$，所以也是成立的。

3. 一个是树边，一个是非树边，这里假设 $i,j$ 是树边和非树边，如果不是的话就交换一下，新的边权和为：
   $$
   k+w_j-\lfloor \frac{s_i}{c_i}\rfloor-\lfloor\frac{s_j}{c_j}\rfloor
   $$
   如果 $c_i$ 更小，显然只选 $c_i$ 更优；如果 $c_j$ 更小，如果答案是选 $j$ 的话也不需要选 $c_i$，所以还是成立的。

综上所述，全都放一条边上一定是有答案的。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 200010, K = 20;
int n, m, s, h[N], dep[N], fa[N][K], d[N][K], mx[N][K], idx = 2;
struct Edge {
    int to, nxt, w, id;
} e[N<<1];

void add(int a, int b, int c, int id) {
    e[idx] = {b, h[a], c, id}, h[a] = idx++;
}

struct KrusEdge {
    int a, b, c, d, id;
    bool used;
    bool operator<(const KrusEdge& ed) const {
        return c < ed.c;
    }
} edge[N];

struct DSU {
    int p[N];

    void init(int n) {
        for (int i = 1; i <= n; i++) p[i] = i;
    }

    int find(int x) {
        if (p[x] != x) p[x] = find(p[x]);
        return p[x];
    }

    void merge(int a, int b) {
        p[find(a)] = find(b);
    }
} dsu;

struct Ans {
    // delta value for ans
    int delta = 0;
    int rm = 0;
    int affect = 0;

    bool operator<(const Ans& a) const {
        return delta < a.delta;
    }
};

int kruskal() {
    sort(edge+1, edge+1+m);
    dsu.init(n);

    int sum = 0;
    for (int i = 1; i <= m; i++) {
        int a = edge[i].a, b = edge[i].b, c = edge[i].c, id = edge[i].id;
        if (dsu.find(a) == dsu.find(b)) continue;
        dsu.merge(a, b);
        edge[i].used = true;
        add(a, b, c, id), add(b, a, c, id);
        sum += edge[i].c;
    }
    return sum;
}

void dfs(int u, int f, int fromw, int fromid) {
    fa[u][0] = f, d[u][0] = fromw, mx[u][0] = fromid, dep[u] = dep[f] + 1;
    for (int k = 1; k < K; k++) {
        fa[u][k] = fa[fa[u][k-1]][k-1];
        d[u][k] = max(d[u][k-1], d[fa[u][k-1]][k-1]);
        if (d[u][k-1] > d[fa[u][k-1]][k-1]) {
            d[u][k] = d[u][k-1];
            mx[u][k] = mx[u][k-1];
        }
        else {
            d[u][k] = d[fa[u][k-1]][k-1];
            mx[u][k] = mx[fa[u][k-1]][k-1];
        }
    }

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        dfs(to, u, e[i].w, e[i].id);
    }
}

// return id
void maxd(int a, int b, int& id, int& val) {
    val = 0, id = 0;
    if (dep[a] < dep[b]) swap(a, b);
    for (int k = K-1; k >= 0; k--)
        if (dep[fa[a][k]] >= dep[b]) {
            if (d[a][k] > val) val = d[a][k], id = mx[a][k];
            a = fa[a][k];
        }

    if (a == b) return;

    for (int k = K-1; k >= 0; k--)
        if (fa[a][k] != fa[b][k]) {
            if (d[a][k] > val) val = d[a][k], id = mx[a][k];
            if (d[b][k] > val) val = d[b][k], id = mx[b][k];
            a = fa[a][k], b = fa[b][k];
        }

    if (d[a][0] > val) val = d[a][0], id = mx[a][0];
    if (d[b][0] > val) val = d[b][0], id = mx[b][0];
}

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

signed main() {
    read(n), read(m);
    for (int i = 1; i <= m; i++) edge[i].id = i, read(edge[i].c);
    for (int i = 1; i <= m; i++) read(edge[i].d);
    for (int i = 1; i <= m; i++) read(edge[i].a), read(edge[i].b);
    read(s);

    int raw = kruskal();
    dfs(1, 0, 0, 0);

    Ans ans;
    ans.delta = 0x3f3f3f3f;
    for (int i = 1; i <= m; i++) {
        int delta = -s/edge[i].d, rm = 0;
        if (!edge[i].used) {
            int val = 0;
            maxd(edge[i].a, edge[i].b, rm, val);
            delta += edge[i].c - val;
        }
        ans = min(ans, Ans{delta, rm, edge[i].id});
    }

    printf("%lld\n", raw + ans.delta);

    for (int i = 1; i <= m; i++) {
        int id = edge[i].id, w = edge[i].c;
        if (id == ans.rm) continue;
        if (edge[i].used || id == ans.affect) {
            if (id == ans.affect) w -= s / edge[i].d;
            printf("%lld %lld\n", id, w);
        }
    }
    
    return 0;
}
```

