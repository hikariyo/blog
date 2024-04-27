---
date: 2023-07-22 15:11:11
update: 2023-09-30 20:14:00
title: 图论 - 最近公共祖先
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 倍增

> 给定一棵包含 n 个节点的有根无向树，节点编号互不相同，但不一定是 1∼n。
>
> 有 m 个询问，每个询问给出了一对节点的编号 x 和 y 询问 x 与 y 的祖孙关系。
>
> **输入格式**
>
> 输入第一行包括一个整数 表示节点个数；
>
> 接下来 n 行每行一对整数 a 和 b，表示 a 和 b 之间有一条无向边。如果 b 是 −1，那么 a 就是树的根；
>
> 第 n+2 行是一个整数 m 表示询问个数；
>
> 接下来 m 行，每行两个不同的正整数 x 和 y，表示一个询问。
>
> **输出格式**
>
> 对于每一个询问，若 x 是 y 的祖先则输出 1，若 y 是 x 的祖先则输出 2，否则输出 0。
>
> 题目链接：[AcWing 1172](https://www.acwing.com/problem/content/1174/)。

用 $f(i,j)$ 表示从结点 $i$ 开始向上走 $2^j$ 步能走到的节点，$\text{depth}(i)$ 表示深度。

设超过跟节点的 $f(i,j)=0, \text{depth}(0)=0$，这样可以减少特判；递推关系求 $f(i,j)=f[f(i,j-1),j-1]$

1. 先让两个结点 $a, b$ 跳到同一层，如果相等直接返回。
2. 让两个点同时向上跳，一直跳到最近公共祖先下一层。

每次跳的时候都将 $k$ 从大到小枚举，从二进制的角度来看一定能凑出来任意数字。

复杂度：预处理 $O(n \log n)$，查询 $O(\log n)$ 。

关于 `f` 数组的大小，这里可以举例说明：如果有 $6=110_2$ 应该让下标能达到 $\lfloor\log_26\rfloor=2$，因此应该开到 $\lfloor \log_2 n\rfloor+1$ 的大小。

```cpp
#include <iostream>
#include <queue>
#include <cstring>
using namespace std;

const int N = 40010, M = 2*N;
int depth[N], f[N][16], h[N], e[M], ne[M], idx;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u, int father) {
    depth[u] = depth[father] + 1;
    f[u][0] = father;
    for (int k = 1; k <= 15; k++)
        f[u][k] = f[f[u][k-1]][k-1];
    
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!depth[j]) dfs(j, u);
    }
}

int lca(int a, int b) {
    // 保证 a 比 b 深
    if (depth[a] < depth[b]) swap(a, b);
    for (int k = 15; k >= 0; k--) {
        if (depth[f[a][k]] >= depth[b]) a = f[a][k];
    }
    if (a == b) return a;
    for (int k = 15; k >= 0; k--) {
        if (f[a][k] != f[b][k]) a = f[a][k], b = f[b][k];
    }
    return f[a][0];
}

int main() {
    int n, m, root;
    memset(h, -1, sizeof h);
    cin >> n;
    while (n--) {
        int a, b;
        cin >> a >> b;
        if (b == -1) root = a;
        else add(a, b), add(b, a);
    }
    dfs(root, 0);
    
    cin >> m;
    while (m--) {
        int a, b;
        cin >> a >> b;
        int p = lca(a, b);
        if (p == a) puts("1");
        else if (p == b) puts("2");
        else puts("0");
    }
    return 0;
}
```

## Tarjan LCA

> 给出 n 个点的一棵树，多次询问两点之间的最短距离。
>
> - 边是**无向**的。
> - 所有节点的编号是 1,2,…,n。
>
> **输入格式**
>
> 第一行为两个整数 n 和 m。n 表示点数，m 表示询问次数；
>
> 下来 n−1 行，每行三个整数 x,y,k，表示点 x 和点 y 之间存在一条边长度为 k；
>
> 再接下来 m 行，每行两个整数 x,y，表示询问点 x 到点 y 的最短距离。
>
> 树中结点编号从 1 到 n。
>
> **输出格式**
>
> 共 m 行，对于每次询问，输出一行询问结果。
>
> 题目链接：[AcWing 1171](https://acwing.com/problem/content/1173/)。

Tarjan LCA 算法是一个离线算法，意思是它必须知道所有查询之后一次性求出所有查询结果。

它的复杂度是 $O(n+m)$，比前面提到的倍增算法 $O(n\log n+m\log n)$ 更优。

1. DFS 搜索所有结点，已经回溯过的标记为 $2$，正在搜索的标记为 $1$，未搜索的标记为 $0$。

2. 将所有回溯过的点和它的父结点用并查集合并。
3. 每当处理与当前正在搜索的点 u 相关的查询时，如果另一个点 x 已经被回溯过，那么它们的公共祖先就是 `find(x)`。
4. 距离为 $d(x)+d(u)-2d(p)$ 其中 $d(x)$ 为某结点到根节点的距离。

## 换根 LCA

> 给定一颗 $n$ 个点的无根树，给 $q$ 组询问，每组询问包含 $(u,v,w)$，求以 $w$ 为根意义下的 $LCA(u,v)$
>
> 为了方便输入，第二行的第 $i$ 个点表示 $i$ 的父结点，保证 $fa(1)=0$，即给出的是以 $1$ 为根的树。

本题是在集训时 PPT 上看到的，并没在主流 OJ 上找到，我这里就自己写了个对拍验证是否正确了。

考虑以 $1$ 为根时的 $p=LCA(u,v),a=LCA(u,w),b=LCA(v,w)$

1. 如果 $w$ 在以 $p$ 为根的子树外面，答案就是 $p$，此时 $a=b$
2. 如果 $a$ 在 $u\to p$ 的路径上，答案是 $a$，此时 $b=p$
3. 如果 $b$ 在 $v\to p$ 的路径上，答案是 $b$，此时 $a=p$

因为树上无环，不可能同时满足条件 $2,3$ 我们可以归纳出如果 $a=b$，说明答案是 $p$，否则答案是 $a,b$ 中较深的那个。

```cpp
int LCA(int u, int v, int w) {
    int p = lca(u, v), a = lca(w, u), b = lca(w, v);
    if (a == b) return p;
    if (depth[a] > depth[b]) return a;
    return b;
}
```

下面给出完整程序：

```cpp
#include <cstdio>
#include <cstring>
using namespace std;

const int N = 10010, M = 20010;
int h[N], e[M], ne[M], idx;
int fa[N][15], depth[N];
int n;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u, int f) {
    depth[u] = depth[f] + 1;
    fa[u][0] = f;
    for (int k = 1; k < 15; k++)
        fa[u][k] = fa[fa[u][k-1]][k-1];

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == f) continue;
        dfs(j, u);
    }
}

int lca(int a, int b) {
    if (depth[a] < depth[b]) swap(a, b);
    for (int k = 14; k >= 0; k--) {
        if (depth[fa[a][k]] >= depth[b]) a = fa[a][k];
    }
    if (a == b) return a;
    for (int k = 14; k >= 0; k--) {
        if (fa[a][k] != fa[b][k]) a = fa[a][k], b = fa[b][k];
    }
    return fa[a][0];
}

int LCA(int u, int v, int w) {
    int p = lca(u, v), a = lca(w, u), b = lca(w, v);
    if (a == b) return p;
    if (depth[a] > depth[b]) return a;
    return b;
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d", &n);
    for (int i = 1, j; i <= n; i++) {
        scanf("%d", &j);
        if (j) add(i, j), add(j, i);
    }
    int q, u, v, w;
    scanf("%d", &q);
    dfs(1, 0);
    while (q--) {
        scanf("%d%d%d", &u, &v, &w);
        printf("%d\n", LCA(u, v, w));
    }
    return 0;
}
```

对拍程序中只是每次读入都重新预处理一遍：

```cpp
while (q--) {
    scanf("%d%d%d", &u, &v, &w);
    dfs(w, 0);
    printf("%d\n", lca(u, v));
}
```

数据生成程序：

```cpp
#include <cstdio>
#include <cstdlib>
using namespace std;

const int N = 10010;
int fa[N];

int main() {
    srand(time(0));
    int n = rand() % 10000 + 1;
    fa[1] = 0;
    for (int i = 2; i <= n; i++) {
        int f;
        fa[i] = rand() % (i-1) + 1;
    }
    printf("%d\n", n);
    for (int i = 1; i <= n; i++) printf("%d ", fa[i]);
    printf("\n");
    int q = rand() % 10000 + 1;
    printf("%d\n", q);
    for (int i = 1, u, v, w; i <= q; i++) {
        u = rand() % n + 1, v = rand() % n + 1, w = rand() % n + 1;
        printf("%d %d %d\n", u, v, w);
    }
    return 0;
}
```

检查程序(MacOS 环境，Windows 应该要用 `fc` 之类的)：

```cpp
#include <cstdio>
#include <cstdlib>
#include <ctime>
using namespace std;

int main() {
    for (int i = 1; i <= 100; i++) {
        system("./data > lca.in");
        system("./force < lca.in > lca.ans");
        system("./std < lca.in > lca.out");
        if (system("diff lca.out lca.ans")) {
            printf("Wrong answer on #%d\n", i);
            break;
        }
        else printf("Accepted on #%d\n", i);
    }
    return 0;
}
```

可自行验证。

