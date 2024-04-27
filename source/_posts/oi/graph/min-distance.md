---
date: 2023-06-16 00:26:52
update: 2023-07-19 16:39:02
title: 图论 - 最短路算法
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 简介

对于最短距离问题，一共有如下几种算法：

1. Dijkstra 算法是用来计算单源正权最短路算法，它的朴素版适用于稠密图，复杂度 $O(n^2)$；堆优化版适合稀疏图，复杂度 $O((n+m)\log n)$。
2. Bellman-ford 算法用来解决**（有边数限制）的含负权最短路问题**，如果有边数限制一般就只能用此算法求解，复杂度 $O(mn)$。
3. SPFA 解决 **单源含负权最短路问题**，但它不能有边数限制，复杂度一般 $O(m)$，极端 $O(mn)$。**正权图不要用 SPFA 会被卡。**
4. Floyd 解决**多源最短路**，复杂度 $O(n^3)$。

## 朴素版 Dijkstra

> 给定一个 $n\in [1, 500]$ 个点 $m \in [1, 10^5]$ 条边的有向图，图中可能存在重边和自环，所有边权均为正值。
>
> 请你求出 1 号点到 n 号点的最短距离，如果无法从 1 号点走到 n 号点，则输出 −1。

### 步骤

设结点 $v_0,\cdots,v_n$，中起点为 $s$，已经找到最短路的结点集合为 $S$，未找到最短路的结点集合为 $T$，最短路径数组为 $\text{dist}(v)$，真实的最短路为 $\delta(v)$，初始状态如下：
$$
S=\varnothing,T=V,\text{dist}(v)=\left \{ \begin{aligned}
0, v=s\\
\inf, v \ne s
\end{aligned}
\right .
$$
重复以下步骤：

1. 从集合 $T$ 中取出已知的最短路径 $v_i$ 加入 $S$ 中

2. 基于 $v_i$ 更新所有与它相邻的结点 $v_j$
    $$
    \text{dist}(v_j)=\min \left \{ \begin{aligned}
    &\text{dist}(v_j)\\
    &\text{dist}(v_i)+w(v_i,v_j)
    \end{aligned}\right .
    $$

3. 重复上述操作，直到所有结点都在集合 $S$ 中。

### 证明

参考算法导论上的过程。

1. 显然添加原点 $s$ 成立，因为 $\text{dist}(s) = \delta(s) = 0$ 是正确的。

2. 设在添加 $u$ 到集合 $S$ 之前这 $k$ 步都成立，下面证明第 $k+1$ 步成立。

3. 设 $u$ 为第一个加入集合 $S$ 后使得 $\text{dist}(u)\gt\delta(u)$ 的点。由于 $s$ 点一定是 $\delta(s)=\text{dist}(s)=0$，所以 $u\ne s$； $u$ 与 $s$ 不连通时，$\delta(u)=\text{dist}(u)=\inf$，所以 $u$ 与 $s$ 必然连通。

    于是一定存在最短路径 $s \to x \to y \to u$，其中 $y$ 是第一个不属于集合 $S$ 的点，$x$ 为 $y$ 的前驱结点，那么显然 $x \in S$ 成立。有可能 $s=x, y=u$，这不影响后面的证明。

    下面用反证法，如果最短路径是上面给出的这条，那么一定有：
    $$
    \delta(u)=\delta(y)+w(y,u) \lt \text{dist}(u)
    $$

    1. 如果 $y \in S$，也就是 $y$ 在 $u$ 之前添加到集合 $S$ 中，那么根据归纳假设有 $\delta(y)=\text{dist}(y)$，也就有：
        $$
        \text{dist}(y)+w(y,u)\lt \text{dist}(u)
        $$
        由于 $y$ 在被找到的时候，必然进行**松弛**操作，这会使得：
        $$
        \text{dist}(u)\le \text{dist}(y)+w(y, u)
        $$
        与假设矛盾。

    2. 如果 $y \notin S$，由归纳假设得到：
        $$
        \text{dist}(x)=\delta(x)
        $$
        由三角不等式得到（这对任意图都成立）：
        $$
        \delta(y)\le \delta(x)+w(x,y)\\
        $$
        因为 $x$ 加入集合 $S$ 时必然对边 $(x,y)$ 进行**松弛**操作：
        $$
        \text{dist}(y)\le \text{dist}(x)+w(x,y)
        $$
        由于 $\delta(y)$ 是最短路，有：
        $$
        \delta(y)\le \text{dist}(y)
        $$
        因此得到：
        $$
        \delta(y)=\text{dist}(y)
        $$
        因为 $u$ 先于 $y$ 被选中，且 $w(y,u)\ge 0$ 必然有：
        $$
        \text{dist}(u) \le \text{dist}(y)=\delta(y)\le\delta(y)+w(y,u)
        $$
        这一步是 Dijkstra 算法无法应用到负权图的原因，与假设矛盾。

证毕。

### 代码

代码流程如下：

1. 维护一个 `dist` 数组，代表原点到第 `i` 节点的路径长度，初始化时除了第一位设为 0，其它均为无穷大，可以用 `0x3f3f3f3f` 表示，初始化图时在图中无法到达的距离也是无穷大。
2. 在图中先找到**距离原点最近**且**未访问过**的节点，记录下来它的位置 `t`，并标记已访问这个节点。
3. 遍历整个图，将每个节点 `j` 到原点的位置更新为 `min(self,  1 -> t -> j)`，其中 `1 -> t -> j` 代表原点到 `t` 再到 `j` 的长度。
4. 重复执行 `n` 遍，其中 `n` 是图的长度。

由于这是个稠密图，所以用邻接矩阵的模式储存各个边长，也就是二维数组。

```cpp
#include <iostream>
#include <cstring>
#include <bitset>
using namespace std;

const int N = 510, INF = 0x3f3f3f3f;
int n, m;
int g[N][N], dist[N];
bitset<N> st;

int dj() {
    // 一定要先初始化!!!
    // 首节点到自身的距离为 0，其余都是无穷大
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;

    // 这里 i 没有实际含义，只是为了遍历 n 遍
    for (int i = 0; i < n; i++) {
        // 当前图中到首节点最短的节点
        int t = -1;
        for (int j = 1; j <= n; j++) if (!st[j] && (t == -1 || dist[j] < dist[t])) t = j;
        st[t] = 1;
        // 将每个位置的长度都与 1->t->j 做比较
        for (int j = 1; j <= n; j++) dist[j] = min(dist[j], dist[t] + g[t][j]);
    }
    if (dist[n] == INF) return -1;
    return dist[n];
}

int main() {
    cin.tie(0); cout.tie(0); ios::sync_with_stdio(false);
    memset(g, 0x3f, sizeof g);
    cin >> n >> m;
    int x, y, z;
    while (m--) {
        cin >> x >> y >> z;
        g[x][y] = min(g[x][y], z);
    }
    cout << dj() << '\n';
    return 0;
}
```

存在嵌套循环 `n` 次，所以复杂度是 $O(n^2)$。

## 堆优化版 Dijkstra

> 给定一个 $n\in [1, 1.5\times10^5]$ 个点 $m \in [1, 1.5\times10^5]$ 条边的有向图，图中可能存在重边和自环，所有边权均为正值。
>
> 请你求出 1 号点到 n 号点的最短距离，如果无法从 1 号点走到 n 号点，则输出 −1。

基本思想如下：

1. 还是用 `dist` 数组维护原点到 `i` 节点的距离，初始化为无穷大。
2. 从堆中取出存储**距离原点最近**且**未访问**的点 `t` 它与原点的距离为 `distance`，计算 `t` 的各个子节点到原点的距离。
3. 遍历它的子节点，使每个子节点 `j` 与原点的距离更新为 `min(self, distance + t -> j)`。
4. 直到堆中没有元素为止。

这个图是稀疏图，所以用邻接表的形式来存储，由于有权重的出现，所以要添加一个数组 `w` 储存权重。这里一定要搞清楚几个链表中到底代表什么意思，否则就非常容易写错。

```cpp
#include <iostream>
#include <bitset>
#include <cstring>
#include <queue>
using namespace std;

typedef pair<int, int> pii;

const int N = 1e6+10;
int n, m;
int h[N], e[N], ne[N], w[N], idx, dist[N];
bitset<N> st;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

int dj() {
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    // pair 的第一个元素可以不使用它，但是必须要存它，因为它是排序的依据。pair 以字典序比较大小
    priority_queue<pii, vector<pii>, greater<pii>> heap;
    heap.push({0, 1});
    while (heap.size()) {
        pii node = heap.top(); heap.pop();
        int t = node.second;
        if (st[t]) continue;
        st[t] = 1;
        // 遍历子节点 更新距离
        for (int i = h[t]; i != -1; i = ne[i]) {
            // i 相当于 Node*, e[i] 相当于解引用
            int j = e[i];
            if (dist[j] > w[i] + dist[t]) {
                dist[j] = w[i] + dist[t];
                heap.push({dist[j], j});
            }
        }
    }
    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];
}

int main() {
    memset(h, -1, sizeof h);
    cin.tie(0), cout.tie(0), ios::sync_with_stdio(false);
    cin >> n >> m;
    int a, b, c;
    while (m--) {
        cin >> a >> b >> c;
        add(a, b, c);
    }
    cout << dj() << endl;
    return 0;
}
```

这里画个图解释一下图，链表这些之间的连接方式。注意，红色的箭头只表示链表中的指针，**没有距离，这几个红箭头连起来的都是当前节点的子节点的指针**，而蓝色的箭头有距离。

![](/img/2023/06/dijkstra-1.jpg)

由上图可知这个算法遍历所有边 `m`，并且堆的操作都是对数复杂度的，最多会把 `n` 个节点全部添加进堆，所以整个复杂度是 $O((m+n)\log n)$。

## Bellman-ford

> 给定一个 $n \in [1, 500]$ 个点 $m \in [0, 10^4]$ 条边的有向图，图中可能存在重边和自环， **边权可能为负数**。
>
> 请你求出从 1 号点到 n 号点的最多经过 $k \in [1, 500]$ 条边的最短距离，如果无法从 1 号点走到 n 号点，输出 `impossible`。
>
> 注意：图中可能 **存在负权回路** 。

Bellman-ford 算法用来解决**（有边数限制）的含负权最短路问题**，如果有边数限制一般就只能用此算法求解，基本思路如下：

1. 限制多少边就重复执行多少次下列求解步骤。
2. 遍历图中所有的边（初始是无穷大），用 `dist` 数组记录每个点到原点的最短路。注意，这一步需要用到一个备份数组，因为不能在遍历的过程中用这一步产生的数据。

因此复杂度为 $O(m^2)$，可以用一个数组储存所有的边长信息。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int N = 510, M = 1e4+10;
int n, m, k;
struct Edge {
    int a, b, w;
} edges[M];
int dist[N], bak[N];

void bellmanford() {
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    for (int i = 0; i < k; i++) {
        memcpy(bak, dist, sizeof bak);
        for (int j = 0; j < m; j++) {
            int a = edges[j].a, b = edges[j].b, w = edges[j].w;
            // min(1 -> b, 1 -> a -> b)
            // 这里必须用 bak 中的数据
            // 但是 dist[b] 不能用 bak 中的数据，因为要进行更新
            dist[b] = min(dist[b], bak[a] + w);
        }
    }
    // 由于存在负权边，这里只要大于一个比较大的数就可以认为不可达
    if (dist[n] > 0x3f3f3f3f / 2) cout << "impossible";
    else cout << dist[n];
}

int main() {
    cin.tie(0), cout.tie(0), ios::sync_with_stdio(false);
    cin >> n >> m >> k;
    for (int i = 0; i < m; i++) cin >> edges[i].a >> edges[i].b >> edges[i].w;
    bellmanford();
    return 0;
}
```

## SPFA

> 给定一个 $n \in [1, 10^5]$ 个点 $m \in [0, 10^5]$ 条边的有向图，图中可能存在重边和自环， **边权可能为负数**。
>
> 请你求出从 1 号点到 n 号点的 最短距离，如果无法从 1 号点走到 n 号点，输出 `impossible`。
>
> 不存在负权回路。

SPFA 是 Bellman-ford 算法用 BFS 优化得到的，它的思路如下：

1. 用队列储存待更新的节点，初始时放入原点。
2. 遍历队列中的每个节点，计算出它的子节点与原点的距离。
3. 每当有节点的长度变短，就把这个节点放入队列中，直到队列中没有节点为止。

正确性证明：

1. 初始状态：算法开始前，只有源点 s 的 dist 值为0，其他顶点的 dist 值都是无穷大。这显然是正确的。
2. 递推关系：SPFA算法在每一次迭代中，对队列中的顶点进行松弛操作，更新其相邻顶点的 dist 值。如果某定点 v 的 dist 值在当前迭代中被更新，说明找到了一条从源点 s 经过若干边到达 v 的更短路径。
3. 终止条件：SPFA算法在每次迭代中，都会处理队列中的所有顶点，对其邻居进行松弛操作，直到队列为空。因为算法不断更新 dist 值，只要存在从源点 s 可达的顶点，它们的最短路径长度一定会被找到。当队列为时，所有顶点的最短路径长度已经确定，算法终止。

它与 Dijkstra 算法很像，但是时间复杂度一般为 $O(n)$，极端情况是 $O(mn)$，而优化过的 Dijkstra 算法可以达到 $O(m\log n)$。

```cpp
#include <iostream>
#include <cstring>
#include <bitset>
#include <queue>
using namespace std;

const int N = 1e5+10;
int n, m;
int h[N], e[N], ne[N], w[N], idx;
int dist[N];
// 某个点是否在队列中
bitset<N> st;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx, w[idx] = c, idx++;
}

void spfa() {
    queue<int> q;
    q.push(1);
    
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = 0;
        for (int i = h[t]; i != -1; i = ne[i]) {
            int j = e[i];
            // 这里不判断 st[j] 因为这个值的意思是它是否在队列中 防止重复添加
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                if (!st[j]) {
                    q.push(j);
                    st[j] = 1;
                }
            }
        }
    }
    // 与 Bellman ford 不同，因为是严格按照图 BFS 遍历，所以不可达的点不会被更新
    if (dist[n] == 0x3f3f3f3f) cout << "impossible";
    else cout << dist[n];
}

int main() {
    cin.tie(0), cout.tie(0), ios::sync_with_stdio(false);
    // 不要忘记初始化！
    memset(h, -1, sizeof h);
    cin >> n >> m;
    for (int i = 0; i < m; i++) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c);
    }
    spfa();
    return 0;
}
```



## DAG 中的单源最短路

对于 DAG（Directed Acyclic Graph，有向无环图）无论有无负权边，可以直接用拓扑序求出最小距离。

证明其正确性很容易：

1. 显然对原点 $s$ 成立，对不可达的边成立。
2. 设前 $k$ 步成立，接下来证明第 $k+1$ 步处理的节点 $u$ 成立。
3. 因为是按照拓扑序遍历整个图，因此由归纳假设得到 $u$ 的所有前驱节点一定是正确的，由于 $u$ 的所有前驱节点都对 $u$ 进行过**松弛**操作，因此 $u$ 的最短路也是正确的。

下面是代码实现。

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <queue>
using namespace std;

const int N = 510, M = 10010;
int h[N], e[M], ne[M], w[M], din[N], dist[N], idx;
int n, m, S, T;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
    din[b]++;
}

void topsort() {
    queue<int> q;
    memset(dist, 0x3f, sizeof dist);
    dist[S] = 0;
    
    for (int i = 1; i <= n; i++) {
        if (din[i] == 0) q.push(i);
    }
    
    while (q.size()) {
        int t = q.front(); q.pop();
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            dist[j] = min(dist[j], dist[t] + w[i]);
            if (--din[j] == 0) q.push(j);
        }
    }
}

int main() {
    cin >> n >> m >> S >> T;
    memset(h, -1, sizeof h);
    while (m--) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c);
    }
    topsort();
    cout << dist[T] << endl;
    return 0;
}
```

这里没有例题，我编了一个测试数据。

```text
5 7 2 4
1 2 3
1 3 5
2 3 2
3 5 -3
2 5 1
3 4 8
5 4 4
```

![](/img/2023/07/dag-min-dist.png)

如上图，它应该输出最短路长度 `3`，路径是 `2 -> 3 -> 5 -> 4`，这样可以用 $O(n+m)$ 的复杂度求最短路。

## Floyd 算法

> 给定一个 n 个点 m 条边的有向图，图中可能存在重边和自环，边权可能为负数。
>
> 再给定 k 个询问，每个询问包含两个整数 x 和 y，表示查询从点 x 到点 y 的最短距离，如果路径不存在，则输出 `impossible`。

Floyd 算法解决求多源最短路径的问题，原理是 DP。

+ 状态表示 `g(k, i, j)`：

    + 含义：从 i 到 j 经过前 k 个点的最短路。
    + 存储：最小值。

+ 状态转移方程：分为不包含 `k` 点 和 包含 `k` 点 两种情况。
    $$
    g(k,i,j)=\min \left\{\begin{aligned}
    &g(k-1,i,j)\\
    &g(k-1,i,k)+g(k-1,k,j)
    \end{aligned}\right .
    $$

+ 初始化：每个点到自己的距离为 0 其余正无穷。
    $$
    g(k,i,j)=\left \{ \begin{aligned}
    &0,i=j\\
    &\inf,i\ne j
    \end{aligned}\right .
    $$

+ 优化维度：观察下面两个式子。
    $$
    g(k,i,k)=\min\{g(k-1,i,k),g(k-1,i,k)+g(k-1,k,k)\}\\
    g(k,k,j)=\min\{g(k-1,k,j),g(k-1,k,k)+g(k-1,k,j)\}
    $$
    由初始化可知，$g(k,x,x)\equiv 0$ 因此得到下列恒等式。
    $$
    g(k,i,k)=g(k-1,i,k)\\
    g(k,k,j)=g(k-1,k,j)
    $$
    所以可以直接去掉第一维。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int N = 210, INF = 1e9;
int g[N][N];
int n,m,k;

void floyd() {
    for (int k = 1; k <= n; k++) {
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                g[i][j] = min(g[i][j], g[i][k] + g[k][j]);
            }
        }
    }
}


int main() {
    cin.tie(0), cout.tie(0), ios::sync_with_stdio(false);
    cin >> n >> m >> k;
    // 对于自环初始化为0，其它都是正无穷
    for (int i = 1; i <= n; i++) for (int j = 1; j <= n; j++) if(i==j) g[i][j]=0;else g[i][j]=INF;
    while (m--) {
        int a, b, c;
        cin >> a >> b >> c;
        // 如果有重边取最短的
        g[a][b] = min(g[a][b], c);
    }
    floyd();
    while (k--) {
        int x, y;
        cin >> x >> y;
        if (g[x][y] > INF / 2) cout << "impossible"  << endl;
        else cout << g[x][y] << endl;
    }
    return 0;
}
```

