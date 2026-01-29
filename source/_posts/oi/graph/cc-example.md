---
date: 2023-09-19 20:57:00
updated: 2023-09-19 20:57:00
title: 图论 - 连通分量例题
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## [USACO03FALL] Popular Cows G

> 每一头牛的愿望就是变成一头最受欢迎的牛。
>
> 现在有 N 头牛，编号从 1 到 N，给你 M 对整数 (A,B)，表示牛 A 认为牛 B 受欢迎。
>
> 这种关系是具有传递性的，如果 A 认为 B 受欢迎，B 认为 C 受欢迎，那么牛 A 也认为牛 C 受欢迎。
>
> 你的任务是求出有多少头牛被除自己之外的所有牛认为是受欢迎的。
>
> 题目链接：[P2341](https://www.luogu.com.cn/problem/P2341)，[AcWing 1174](https://www.acwing.com/problem/content/1176/)。

也就是在缩点后的出度为 0 的强连通分量中是所有满足题目条件的结点，但是这里要保证出度为 0 的强连通分量是唯一的，如果不唯一那么也是无解。

```cpp
#include <iostream>
#include <algorithm>
#include <stack>
#include <cstring>
using namespace std;

const int N = 10010, M = 50010;
int h[N], e[M], ne[M], idx;
int dfn[N], sz[N], low[N], timestamp;
int id[N], dout[N], scc_cnt;
stack<int> stk;
bool instk[N];
int n, m;

void tarjan(int u) {
    stk.push(u);
    instk[u] = true;
    dfn[u] = low[u] = ++timestamp;
    
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (dfn[j] == 0){
            tarjan(j);
            low[u] = min(low[u], low[j]);
        }
        else if (instk[j]) {
            low[u] = min(low[u], dfn[j]);
        }
    }
    
    if (dfn[u] == low[u]) {
        int y;
        ++scc_cnt;
        do {
            y = stk.top(); stk.pop();
            instk[y] = false;
            id[y] = scc_cnt;
            sz[scc_cnt]++;
        } while (y != u);
    }
}

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

int main() {
    memset(h, -1, sizeof h);
    cin >> n >> m;
    for (int i = 0; i < m; i++) {
        int a, b;
        cin >> a >> b;
        add(a, b);
    }
    
    for (int i = 1; i <= n; i++)
        if (dfn[i] == 0) tarjan(i);
    
    // 对强连通分量建图 这里只统计了出度
    for (int i = 1; i <= n; i++)
        for (int j = h[i]; ~j; j = ne[j]) {
            int k = e[j];
            int a = id[i], b = id[k];
            if (a != b) dout[a]++;
        }
    
    // 按照拓扑序从后向前遍历所有强连通分量
    int zeros = 0, ans = 0;
    for (int i = 1; i <= scc_cnt; i++)
        if (dout[i] == 0) {
            if (++zeros > 1) {
                ans = 0;
                break;
            }
            ans = sz[i];
        }
    
    cout << ans << endl;
    
    return 0;
}
```

## 学校网络

> 一些学校连接在一个计算机网络上，学校之间存在软件支援协议，每个学校都有它应支援的学校名单（学校 A 支援学校 B，并不表示学校 B 一定要支援学校 A）。
>
> 当某校获得一个新软件时，无论是直接获得还是通过网络获得，该校都应立即将这个软件通过网络传送给它应支援的学校。
>
> 因此，一个新软件若想让所有学校都能使用，只需将其提供给一些学校即可。
>
> 现在请问最少需要将一个新软件直接提供给多少个学校，才能使软件能够通过网络被传送到所有学校？
>
> 最少需要添加几条新的支援关系，使得将一个新软件提供给任何一个学校，其他所有学校就都可以通过网络获得该软件？
>
> 题目链接：[AcWing 367](https://www.acwing.com/problem/content/369/)。

对于 Tarjan 缩点后的 DAG，第一问是入度为 0 的点个数，第二问是添加几条边可以让 DAG 构成强连通分量。

**定理**：在一个 DAG $G=(V,E)$ 中，设入度为 $0$ 的点集为 $P$，出度为 $0$ 的点集为 $Q$，那么最少添加 $\max\{ |P|, |Q|\}$  条边能使其成为强连通分量。

**证明**：不妨设 $|P|\le |Q|$，若 $|P|\gt |Q|$ 则在反向图 $G^T$ 中求解边数与原问题等价。现证最少添加 $|Q|$ 条边使 $G$ 成为一个强连通分量。

当 $|P|=1$ 时，这显然是成立的，需要将所有终点与起点连一条边才能使 $G$ 成为一个强连通分量。

当 $|P|>1$ 时，对起点 $p_0$ 和它不能到达的终点 $q_1$ 连一条边 $(q_1,p_0)$；经过 $|P|-1$ 次上述操作构造一个新图 $G'$ 使 $|P'|=1, |Q'|=|Q|-(|P|-1)$，那么这时转化成第一个情况，加上前面的多次操作，最终需要连的边数就是 $|Q'|+(|P|-1)=|Q|$，证毕。

```cpp
#include <iostream>
#include <cstring>
#include <stack>
using namespace std;

const int N = 110, M = 10010;
int n;
int h[N], e[M], ne[M], idx;
int dfn[N], low[N], timestamp, id[N], dout[N], din[N], scc_cnt;
stack<int> stk;
bool instk[N];

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void tarjan(int u) {
    stk.push(u);
    instk[u] = true;
    low[u] = dfn[u] = ++timestamp;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) tarjan(j), low[u] = min(low[u], low[j]);
        else if (instk[j]) low[u] = min(low[u], dfn[j]);
    }
    if (low[u] == dfn[u]) {
        int y;
        ++scc_cnt;
        do {
            y = stk.top(); stk.pop();
            instk[y] = false;
            id[y] = scc_cnt;
        } while (y != u);
    }
}

int main() {
    cin >> n;
    memset(h, -1, sizeof h);
    for (int i = 1; i <= n; i++) {
        int j;
        while (cin >> j, j) {
            add(i, j);
        }
    }
    for (int i = 1; i <= n; i++)
        if (!dfn[i]) tarjan(i);
    
    for (int i = 1; i <= n; i++)
        for (int j = h[i]; ~j; j = ne[j]) {
            int k = e[j];
            int a = id[i], b = id[k];
            if (a != b) din[b]++, dout[a]++;
        }
    
    int p = 0, q = 0;
    for (int i = 1; i <= scc_cnt; i++) {
        if (din[i] == 0) p++;
        if (dout[i] == 0) q++;
    }
    
    cout << p << endl;
    if (scc_cnt == 1) cout << 0 << endl;
    else cout << max(p, q) << endl;
    return 0;
}
```

## 冗余路径

> 为了从 F 个草场中的一个走到另一个，奶牛们有时不得不路过一些她们讨厌的可怕的树。
>
> 奶牛们已经厌倦了被迫走某一条路，所以她们想建一些新路，使每一对草场之间都会至少有两条相互分离的路径，这样她们就有多一些选择。
>
> 每对草场之间已经有至少一条路径。
>
> 给出所有 R 条双向路的描述，每条路连接了两个不同的草场，请计算最少的新建道路的数量，路径由若干道路首尾相连而成。
>
> 两条路径相互分离，是指两条路径没有一条重合的道路。
>
> 但是，两条分离的路径上可以有一些相同的草场。
>
> 对于同一对草场之间，可能已经有两条不同的道路，你也可以在它们之间再建一条道路，作为另一条不同的道路。
>
> 题目链接：[AcWing 395](https://www.acwing.com/problem/content/397/)。

对于一个边双连通分量，一定存在简单环，也就有条不同的道路，所以本题的问题就是通过边双连通分量缩点后形成的树中，最少添加几条边可以使一个树变成边双连通分量。

结论是 $\lceil \cfrac{\text{leaf}}{2}\rceil$ 其中 $\text{leaf}$ 是叶结点的数量：将所有叶结点两两连边，如果是奇数个那么将剩下的那个单独和其它任意点连一条边，就构成了边双连通分量。

值得说明的是，对于一个边的编号与其反向边总符合 (0, 1) (2, 3), (4, 5) 的规律，因此异或 1 就能取到一个边的反向边编号。

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <stack>
using namespace std;

const int N = 5010, M = 20010;
int n, m;
int h[N], e[M], ne[M], idx;
int low[N], dfn[N], id[N], d[N], timestamp, dcc_cnt;
bool bridge[M];
stack<int> stk;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void tarjan(int u, int from) {
    dfn[u] = low[u] = ++timestamp;
    stk.push(u);
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j, i);
            low[u] = min(low[u], low[j]);
            if (low[j] > dfn[u])
                bridge[i] = bridge[i^1] = true;
        } else if (i != (from ^ 1))
            low[u] = min(low[u], dfn[j]);
    }
    if (dfn[u] == low[u]) {
        ++dcc_cnt;
        int y;
        do {
            y = stk.top(); stk.pop();
            id[y] = dcc_cnt;
        } while (y != u);
    }
}

int main() {
    memset(h, -1, sizeof h);
    cin >> n >> m;
    for (int i = 0; i < m; i++) {
        int a, b;
        cin >> a >> b;
        add(a, b), add(b, a);
    }
    
    // 缩点
    for (int i = 1; i <= n; i++)
        if (!dfn[i]) tarjan(i, -1);
    
    for (int i = 0; i < idx; i++)
        if (bridge[i]) d[id[e[i]]]++;
    
    int leaf = 0;
    for (int i = 1; i <= dcc_cnt; i++)
        if (d[i] == 1) leaf++;
    cout << (leaf+1)/2 << endl;
    return 0;
}
```

## [CTU Open 2004] 电力

> 给定一个由 n 个点 m 条边构成的**无向**图，请你求出该图删除一个点之后，连通块最多有多少。
>
> 题目链接：[AcWing 1183](https://www.acwing.com/problem/content/1185/)。

每个连通块内的割点删掉后能分出的最大连通块数量为 $s$，连通块数量总共为 $cnt$，那么答案就是 $s+cnt-1$，因为 $cnt-1$ 不包含处理出来 $s$ 的那个连通块。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 10010, M = 30010;
int n, m;
int dfn[N], low[N], ts;
int h[N], e[M], ne[M], idx;
int root, ans;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void tarjan(int u) {
    bool child = false;
    int sons = 0;
    dfn[u] = low[u] = ++ts;

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j);
            low[u] = min(low[u], low[j]);
            child = true;
            if (low[j] >= dfn[u]) sons++;
        }
        else low[u] = min(low[u], dfn[j]);
    }

    if (child && u != root) sons++;
    ans = max(ans, sons);
}

int main() {
    while (scanf("%d%d", &n, &m), n || m) {
        idx = ts = ans = 0;
        memset(h, -1, sizeof h);
        memset(dfn, 0, sizeof dfn);
        for (int i = 0, a, b; i < m; i++) {
            scanf("%d%d", &a, &b);
            ++a, ++b;
            add(a, b), add(b, a);
        }

        int cntblock = 0;
        for (root = 1; root <= n; root++)
            if (!dfn[root]) tarjan(root), cntblock++;
        
        printf("%d\n", ans + cntblock - 1);
    }
    return 0;
}
```

## [POI2008] BLO-Blockade

> B 城有 $n(\le 10^5)$ 个城镇，$m(\le 5\times 10^5)$ 条双向道路。
>
> 每条道路连结两个不同的城镇，没有重复的道路，所有城镇连通。
>
> 把城镇看作节点，把道路看作边，容易发现，整个城市构成了一个无向图。
>
> 请你对于每个节点 $i$ 求出，把与节点 $i$ 关联的所有边去掉以后(不去掉节点 $i$ 本身)， 无向图有多少个有序点 $(x,y)$，满足 $x$ 和 $y$ 不连通。
>
> 题目链接：[P3469](https://www.luogu.com.cn/problem/P3469)。

对于每个点，分两种情况：

1. 不是割点，与剩下 $n-1$ 个点能连上，这样答案是 $2(n-1)$
2. 是割点，设周围连通块的大小为 $sz_i$，那么答案是 $[\sum sz_i(n-sz_i-1)]+2(n-1)$，每个连通块和其它连通块的连线（不包含割点）再加上割点的连线。

从缩点后的 DFS 树上看，我们把 $low(j)\ge dfn(u)$ 的子结点 $j$ 当作连通块，然后把 $low(u)<dfn(u)$ 的划到上面的连通块中，由于无向图的 DFS 树上没有横叉边的存在，这样划分是可以正确统计到结果的，如下图：

![](/img/2023/10/p3469.jpg)

如图，对于以 $2$ 为根的子树来说，$(3,4,5),(6)$ 就是两个下面的连通块，$(7,8),(1)$ 实际上都是上面的连通块。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 100010, M = 1000010;
int n, m, h[N], idx = 2;
int dfn[N], low[N], ts;
struct Edge {
    int to, ne;
} e[M];
int root, sz[N];
LL ans[N];

void add(int h[], int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void tarjan(int u, int fa) {
    dfn[u] = low[u] = ++ts;
    bool cut = false;
    int sons = 0;

    sz[u] = 1;
    // u 下面的连通块
    LL sum = 0;
    for (int i = h[u]; i; i = e[i].ne) {
        int j = e[i].to;
        if (!dfn[j]) {
            tarjan(j, u);
            low[u] = min(low[u], low[j]);
            sz[u] += sz[j];

            if (low[j] >= dfn[u]) {
                sum += sz[j];
                ans[u] += 1LL * sz[j] * (n - sz[j] - 1);

                sons++;
                if (u != root || sons >= 2) cut = true;
            }
        }
        else low[u] = min(low[u], dfn[j]);
    }

    if (root == u && sons == 0) cut = true;

    if (!cut) return ans[u] = (n-1) * 2, void();
    ans[u] += (n - sum - 1) * sum + (n-1) * 2;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 0, a, b; i < m; i++) {
        scanf("%d%d", &a, &b);
        add(h, a, b), add(h, b, a);
    }

    // 数据保证图连通
    root = 1;
    tarjan(1, 0);
    
    for (int u = 1; u <= n; u++) printf("%lld\n", ans[u]);
    return 0;
}
```

