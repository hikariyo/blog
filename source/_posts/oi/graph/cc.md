---
date: 2023-09-19 20:55:00
updated: 2023-09-19 20:55:00
title: 图论 - 连通分量模板
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 边的分类

这里先讨论关于边的几种分类，参考《算法导论》给出的定义。

1. 树边：如果结点 $v$ 是因算法对边 $(u,v)$ 的探索而首先被发现，则 $(u, v)$ 是一条树边。
2. 前向边：将结点 $u$ 连接到其在 DFS 树中一个后代结点 $v$ 的边 $(u,v)$，树边就是一种特殊的前向边。
3. 后向边：将结点 $u$ 链接到其在 DFS 树中一个祖先结点 $v$ 的边 $(u,v)$。
4. 横向边：或者横插边，指其他所有的边。这些边可以连接同一棵 DFS 树中的结点，只要其中一个结点不是另外一个结点的祖先，也可以连接不同 DFS 树中的两个结点。

对于无向图，不存在横向边，如果存在横向边 $(u,v)$ 那么 $u$ 会在搜索 $v$ 的时候被搜到，这时 $(v,u)$ 就是一个简单的树边，至于后面也就是后向边，而不是横向边了。

## 强连通分量

> 给定一个 n 个点 m 条边有向图，每个点有一个权值，求一条路径，使路径经过的点权值之和最大。你只需要求出这个权值和。
>
> 允许多次经过一条边或者一个点，但是，重复经过的点，权值只计算一次。
>
> 题目链接：[P3387](https://www.luogu.com.cn/problem/P3387)。

一个有向图中，强连通分量（Strongly Connected Component）内的任意两点 $u,v$ 都是可以互相到达的，也就是存在环。用 Tarjan 算法可以在线性的时间内找到有向图中的所有强连通分量。

将这些强连通分量缩为点，就得到了一个 DAG，因为没有了环的存在，就可以很方便求解一些问题了。

Tarjan 算法的流程大致如下：

1. 对于每个点 $u$，都有 $dfn(u), low(u)$ 分别表示当前点被遍历到的时间戳，和它能到达的最小时间戳，初始化时它们相同。

2. 将 $u$ 入栈。

3. 遍历 $u$ 的所有相邻结点 $v$：

    + 如果 $v$ 没被遍历过，就遍历 $v$，将 $low(u)=\min\{low(u),  low(j)\}$

    + 如果 $v$ 遍历过了并且在栈中（如果不在意味着 $v$ 属于别的强连通分量），将 $low(u)=\min \{low(u), dfn(j)\}$

        虽然我这里写了 $low(u)=\min\{low(u), low(j)\}$ 虽然也是能 AC，而且经过检验每组测试数据 $low(j)$ 对应的结点也确实在栈中，但是这和算法的逻辑不相同，因此还是写 $dfn(j)$ 较好。

4. 如果遍历完所有结点后 $dfn(u)=low(u)$ 仍然成立，就说明 $u$ 是其所在强连通分量中时间戳最小的点，弹出栈中元素直到 $u$ 为止都是其所在强连通分量中的结点。

5. 值得说明的是，如果一个点不在任何环中，它本身也是一个强连通分量。

在一个 DAG 中 DFS 序搜到的所有点，考虑下面的伪代码：

```python
def dfs(u):
    lst = []
    for j in u.next:
        dfs(j)
        lst.append(j)
```

最终在 `lst` 中的所有结点一定是按照拓扑序倒序排列的，所以用 Tarjan 搜到的所有强连通分量的编号顺序与拓扑序相反。

将图用 Tarjan 缩点后，将点权看做入边的边权，对 DAG 求一个最值即可。

在上面也说过，编号的倒序就是拓扑序的正序，然后再注意可能存在孤立的强连通分量，每个点的路径起码要和自己的权取一个最大值。

```cpp
#include <stack>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 20010, M = 200010;
int h[N], sh[N], e[M], ne[M], p[N], idx;
int dfn[N], low[N], id[N], w[N], dist[N], timestamp, cntscc;
stack<int> stk;
bool instk[N];
int n, m;

void add(int h[], int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void tarjan(int u) {
    dfn[u] = low[u] = ++timestamp;
    stk.push(u);
    instk[u] = true;

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) tarjan(j), low[u] = min(low[u], low[j]);
        else if (instk[j]) low[u] = min(low[u], dfn[j]);
    }

    if (low[u] == dfn[u]) {
        int y;
        ++cntscc;
        do {
            y = stk.top(); stk.pop();
            id[y] = cntscc;
            w[cntscc] += p[y];
            instk[y] = false;
        } while (y != u);
    }
}

int main() {
    memset(h, -1, sizeof h);
    memset(sh, -1, sizeof sh);

    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &p[i]);
    for (int i = 1, a, b; i <= m; i++) scanf("%d%d", &a, &b), add(h, a, b);

    for (int u = 1; u <= n; u++)
        if (!dfn[u]) tarjan(u);

    for (int u = 1; u <= n; u++) {
        for (int i = h[u]; ~i; i = ne[i]) {
            int j = e[i];
            if (id[u] != id[j]) add(sh, id[u], id[j]);
        }
    }
    
    int res = 0;
    for (int u = cntscc; u; u--) {
        dist[u] = max(dist[u], w[u]);
        for (int i = sh[u]; ~i; i = ne[i]) {
            int j = e[i];
            dist[j] = max(dist[j], dist[u] + w[j]);
        }
        res = max(res, dist[u]);
    }

    return !printf("%d\n", res);
}
```

## 割点

> 给出一个 $n$ 个点，$m$ 条边的无向图，求图的割点。
>
> **输入格式**
>
> 第一行输入两个正整数 $n,m$。
>
> 下面 $m$ 行每行输入两个正整数 $x,y$ 表示 $x$ 到 $y$ 有一条边。
>
> **输出格式**
>
> 第一行输出割点个数。
>
> 第二行按照节点编号从小到大输出节点，用空格隔开。
>
> 对于全部数据，$1\leq n \le 2\times 10^4$，$1\leq m \le 1 \times 10^5$。
>
> 题目链接：[P3388](https://www.luogu.com.cn/problem/P3388)。

对于边 $(x,y)$ 如果 $x$ 为割点，那么需要满足：

+ 当 $x$ 不是根节点时，$low(y)\ge dfn(x)$ 表明 $y$ 无论如何也无法搜到比 $x$ 更早的结点。
+ 当 $x$ 为根节点时，至少有两个子结点满足 $low(y_i) \ge dfn(x)$，那么 $x$ 就是割点。

```cpp
#include <set>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 20010, M = 210000;
int h[N], e[M], ne[M], idx;
int dfn[N], low[N], timestamp;
int n, m;
set<int> cut;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void tarjan(int u, int from) {
    dfn[u] = low[u] = ++timestamp;
    // 当前节点连接的点双数量
    int sons = 0;

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j, i);
            low[u] = min(low[u], low[j]);
            if (low[j] >= dfn[u]) {
                sons++;
                if (from != -1) cut.insert(u); 
            }
        }
        else low[u] = min(low[u], dfn[j]);
    }

    if (sons >= 2 && from == -1) cut.insert(u);
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d", &n, &m);

    for (int i = 1, a, b; i <= m; i++)
        scanf("%d%d", &a, &b), add(a, b), add(b, a);

    for (int i = 1; i <= n; i++)
        if (!dfn[i]) tarjan(i, -1);

    printf("%d\n", (int)cut.size());
    for (int v: cut) printf("%d ", v);
    return puts(""), 0;
}
```

## 点双连通分量

>
>对于一个 $n$ 个节点 $m$ 条无向边的图，请输出其点双连通分量的个数，并且输出每个点双连通分量。
>
>**输入格式**
>
>第一行，两个整数 $n$ 和 $m$。
>
>接下来 $m$ 行，每行两个整数 $u, v$，表示一条无向边。
>
>**输出格式**
>
>第一行一个整数 $x$ 表示点双连通分量的个数。
>
>接下来的 $x$ 行，每行第一个数 $a$ 表示该分量结点个数，然后 $a$ 个数，描述一个点双连通分量。
>
>你可以以任意顺序输出点双连通分量与点双连通分量内的结点。
>
>对于 $100\%$ 的数据，$1 \le n \le 5 \times10 ^5$，$1 \le m \le 2 \times 10^6$。
>
>题目链接：[P8435](https://www.luogu.com.cn/problem/P8435)。

### 定义

极大的，不包含割点（删掉点和它的所有邻边后整个图不连通）的连通块。注意，这里的割点对应的是整个图，而在一个子图内部它并不是割点。

### 性质

点双连通分量中的任意两点都在至少一个简单环中，等价于两点间存在两条不相交（无公共点）的路径。

必要性：无论删掉哪个点两点都可以互达，因此图中不存在割点，必要性得证。

充分性：反证，假设存在两点 $u,v$ 不同时处于一简单环中，如果两点间只有一条路径，必然存在割点；如果有大于等于两条路径，若这些路径不相交于 $u,v$ 以外的点就必定成环，所以这些路径一定交于某点，这个点也就成为了割点，就不是点双连通分量，推出矛盾，充分性得证。

注意，两个点一条边构成的无向图和孤立点也是符合定义的，它们是特例。

### 算法

1. 对于每个点 $u$ 都使 $dfn(u)=low(u)$，为其分配一个时间戳。

2. 将 $u$ 入栈。

3. 遍历 $u$ 的所有相邻结点 $v$，不包含其 DFS 树中的父结点：

    + 如果没有遍历过就 DFS 遍历，然后 $low(u)=\min\{low(u), low(v)\}$

        遍历之后判断 $u$ 点是否为割点，如果是那就弹栈弹到 $v$ 为止，再把 $u$ 加进当前点双连通分量中。

    + 否则 $low(u)=min\{low(u), dfn(v)\}$

4. 如果 $dfn(u) = low(u)$，说明 $u$ 是在它所在的双连通分量中时间戳最小的一个点，那么将栈中到它之前所有元素都加入一个双连通分量中。

```cpp
#include <vector>
#include <stack>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 500010, M = 4000010;
int h[N], e[M], ne[M], idx;
int dfn[N], low[N], timestamp, cntvdcc;
stack<int> stk;
vector<int> vdcc[N];
int n, m;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void tarjan(int u, int fa) {
    int sons = 0;
    stk.push(u);
    dfn[u] = low[u] = ++timestamp;

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j, u);
            low[u] = min(low[u], low[j]);

            // 虽然 u 可能不是割点, 但不影响它在一个点双中
            if (low[j] >= dfn[u]) {
                sons++;
                vdcc[++cntvdcc].emplace_back(u);
                int y;
                do {
                    y = stk.top(); stk.pop();
                    vdcc[cntvdcc].emplace_back(y);
                } while (y != j);
            }
        }
        else low[u] = min(low[u], dfn[j]);
    }

    // 孤立点也是点双
    if (fa == -1 && sons == 0) vdcc[++cntvdcc].emplace_back(u);
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d", &n, &m);

    for (int i = 1, a, b; i <= m; i++) scanf("%d%d", &a, &b), add(a, b), add(b, a);
    
    for (int i = 1; i <= n; i++)
        if (!dfn[i]) stack<int>().swap(stk), tarjan(i, -1);
    
    printf("%d\n", cntvdcc);
    for (int i = 1; i <= cntvdcc; i++) {
        printf("%d ", (int)vdcc[i].size());
        for (int v: vdcc[i]) printf("%d ", v);
        puts("");
    }

    return 0;
}
```

## 边双连通分量

> 对于一个 $n$ 个节点 $m$ 条无向边的图，请输出其边双连通分量的个数，并且输出每个边双连通分量。
>
> **输入格式**
>
> 第一行，两个整数 $n$ 和 $m$。
>
> 接下来 $m$ 行，每行两个整数 $u, v$，表示一条无向边。
>
> **输出格式**
>
> 第一行一个整数 $x$ 表示边双连通分量的个数。
>
> 接下来的 $x$ 行，每行第一个数 $a$ 表示该分量结点个数，然后 $a$ 个数，描述一个边双连通分量。
>
> 你可以以任意顺序输出边双连通分量与边双连通分量内的结点。
>
> 对于 $100\%$ 的数据，$1 \le n \le 5 \times10 ^5$，$1 \le m \le 2 \times 10^6$。
>
> 题目链接：[P8436](https://www.luogu.com.cn/problem/P8436)。

### 定义

极大的，不包含桥边（如果这条边删掉之后整个图不连通）的连通块。

对于一个桥边 $(x, y)$ 满足 $dfn(x)\lt low(y)$，也就是 $y$ 点能走到的最早的时间戳也晚于 $x$ 。

### 性质

边双连通分量中任意一条边都包含在至少一个简单环中。

必要性：如果该性质满足，删去任意一边都不影响环中两点之间的可达性，那么就一定是一个边双连通分量，必要性得证。

充分性：在一个边双连通分量中，反证：如果边 $(u,v)$ 不在一个简单环中，断掉该边后 $u, v$ 不可到达，充分性得证。

### 算法

1. 对于每个点 $u$ 都使 $dfn(u)=low(u)$，为其分配一个时间戳。

2. 将 $u$ 入栈。

3. 遍历 $u$ 的所有相邻结点 $v$，不包含其 DFS 树中的父结点：

    + 如果没有遍历过就 DFS 遍历，然后 $low(u)=\min\{low(u), low(v)\}$

        如果满足 $dfn(u)\lt low(v)$ 那么当前边是一条桥边，标记一下。

    + 否则 $low(u)=min\{low(u), dfn(v)\}$

4. 如果 $dfn(u) = low(u)$，说明 $u$ 是在它所在的边双连通分量中时间戳最小的一个点，那么将栈中到它之前所有元素都加入一个双连通分量中。

这里 $low(u)$ 的求法就和点双时的不一样了，点双中无所谓某个点是否能到达父结点的，而边双需要用这个判断，如果有重边的话这些边都不可能是桥边，所以传给 `tarjan` 函数用来判断的依据应该是来的边，而不是来的点。

```cpp
#include <stack>
#include <cstdio>
#include <cstring>
#include <vector>
#include <algorithm>
using namespace std;

const int N = 500010, M = 4000010;
int h[N], e[M], ne[M], idx;
int dfn[N], low[N], timestamp, cntedcc;
int n, m;
stack<int> stk;
vector<int> edcc[N];

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void tarjan(int u, int from) {
    stk.push(u);
    dfn[u] = low[u] = ++timestamp;

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];

        if (!dfn[j]) tarjan(j, i), low[u] = min(low[u], low[j]);
        else if (i != (from ^ 1)) low[u] = min(low[u], dfn[j]);
    }

    if (low[u] == dfn[u]) {
        ++cntedcc;
        int y;
        do {
            y = stk.top(); stk.pop();
            edcc[cntedcc].emplace_back(y);
        } while (y != u);
    }
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d", &n, &m);

    for (int i = 1, a, b; i <= m; i++) scanf("%d%d", &a, &b), add(a, b), add(b, a);
    for (int i = 1; i <= n; i++) if (!dfn[i]) tarjan(i, -1);

    printf("%d\n", cntedcc);
    for (int i = 1; i <= cntedcc; i++) {
        printf("%d ", (int)edcc[i].size());
        for (int v: edcc[i]) printf("%d ", v);
        puts("");
    }

    return 0;
}
```

也可以处理出所有桥边然后对每个点搜一遍，不允许走桥边，这样每次搜到的就是在同一个边双内的点。

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
using namespace std;

const int N = 500010, M = 4000010;
int h[N], e[M], ne[M], idx;
int dfn[N], low[N], timestamp;
int n, m, cntedcc;
bool isbridge[M], st[N];

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void tarjan(int u, int from) {
    dfn[u] = low[u] = ++timestamp;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];

        if (!dfn[j]) {
            tarjan(j, i);
            low[u] = min(low[u], low[j]);
            if (low[j] > dfn[u]) isbridge[i] = isbridge[i^1] = true;
        }
        else if (i != (from ^ 1)) low[u] = min(low[u], dfn[j]);
    }

    if (low[u] == dfn[u]) cntedcc++;
}

void dfs(vector<int>& out, int u) {
    st[u] = true;
    out.emplace_back(u);
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (st[j] || isbridge[i]) continue;
        dfs(out, j);
    }
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d", &n, &m);

    for (int i = 1, a, b; i <= m; i++) 
        scanf("%d%d", &a, &b), add(a, b), add(b, a);

    for (int i = 1; i <= n; i++)
        if (!dfn[i]) tarjan(i, -1);
    
    printf("%d\n", cntedcc);
    vector<int> edcc;
    for (int i = 1; i <= n; i++) {
        if (st[i]) continue;
        dfs(edcc, i);

        printf("%d ", (int)edcc.size());
        for (int v: edcc) printf("%d ", v);
        puts(""), edcc.clear();
    }

    return 0;
}
```
