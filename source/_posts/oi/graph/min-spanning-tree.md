---
date: 2023-06-16 17:01:22
updated: 2023-06-23 23:45:24
title: 图论 - 最小生成树
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 简介

最小生成树问题是指用图中所有节点构造一个边权重之和最小的树，可以用 Prim 算法和 Kruskal 算法解决稠密图和稀疏图的最小生成树问题。

## Prim

> 给定一个 n 个点 m 条边的无向图，图中可能存在重边和自环，边权可能为负数。
>
> 求最小生成树的树边权重之和，如果最小生成树不存在则输出 `impossible`。

Prim 算法适用于稠密图，与 Dijkstra 算法思想一致，基本思路如下：

1. 从当前未访问的点中找到距离已建立集合的最短的点 `t`。
2. 如果 `t` 不存在，那么直接返回 `impossible`，如果 `t` 存在，那就把它加入集合（标记 `st[t] = 1` 即可）
3. 循环 `n` 次。

```cpp
#include <iostream>
#include <bitset>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 510;
int n, m;
int g[N][N], dist[N];
bitset<N> st;

void prim() {
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    int res = 0;
    // 这里要循环 n 次而不是 m 次
    // 才能保证循环到每个节点
    for (int i = 0; i < n; i++) {
        // 找出当前未访问过的距离已经建成的集合距离最短的点
        int t = -1;
        for (int j = 1; j <= n; j++)
            if (!st[j] && (t == -1 || dist[j] < dist[t]))
                t = j;
        st[t] = 1;
        // 当前节点与已构建的集合之间没有边
        if (dist[t] == 0x3f3f3f3f) {
            cout << "impossible";
            return;
        }
        // t 是距集合最近的点
        res += dist[t];
        for (int j = 1; j <= n; j++)
            // 这里是 [t][j] 还是 [j][t] 无所谓 它们一定一样
            dist[j] = min(dist[j], g[t][j]);
    }
    // 注意这里输出的是 res 不是 dist[n], dist[n] 表示的是第n个节点到集合的距离
    cout << res << '\n';
}

int main() {
    cin >> n >> m;
    memset(g, 0x3f, sizeof g);
    while (m--) {
        int a, b, c;
        cin >> a >> b >> c;
        g[a][b] = g[b][a] = min(g[a][b], c);
    }
    prim();
    return 0;
}
```

易错点：

1. 没有把所有边初始成 `0x3f3f3f3f`。
2. 输入边时只设置了 `a -> b` 而没设置 `b -> a`。
3. 没有取重边的最小值。
4. 比较距离时是 `min(dist[j], g[t][j])`，而不是 `dist[t] + g[t][j]`，因为它比较的是 `j` 这个元素到集合的距离，而 `t` 是刚选出来的对已建集合距离最小的点。

## Kruskal

> 给定一个 n 个点 m 条边的无向图，图中可能存在重边和自环，边权可能为负数。
>
> 求最小生成树的树边权重之和，如果最小生成树不存在则输出 `impossible`。

Kruskal 算法适用于稀疏图，基本思路如下：

1. 先把所有边按照权重排序，这里可以写一个结构体然后重载 `<` 运算符，这一步是 $O(m\log m)$。
2. 遍历排完序的边，看两条边是否属于同一个集合，如果不属于就把它们合并，此时它们之间的权重一定是最小的，因为排过序，所以不用考虑重边问题。遍历的复杂度是 $O(m)$，合并集合的操作用并查集做，复杂度是 $O(1)$。

综上，这个算法的复杂度是 $O(m \log m)$ 因此适合稀疏图。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 2e5+10;
int m, n;
// 并查集
int p[N];

struct Edge {
    int a, b, w;
    // 依据权重排序
    bool operator<(const Edge& e) const {
        return w < e.w;
    }
} edges[N];

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int main() {
    cin.tie(0), cout.tie(0), ios::sync_with_stdio(false);
    cin >> n >> m;
    for (int i = 1; i <= n; i++)
        p[i] = i;
    for (int i = 0; i < m; i++)
        cin >> edges[i].a >> edges[i].b >> edges[i].w;

    // 排序数组 传入头指针和尾指针后一位
    // 注意分清 m 和 n
    sort(edges, edges+m);
    
    int cnt = 0, res = 0;
    for (int i = 0; i < m; i++) {
        int a = edges[i].a, b = edges[i].b, w = edges[i].w;
        int pa = find(a), pb = find(b);
        if (pa != pb) {
            // 把 b 加到 a 里面(反过来也可以 无所谓)
            // 因为当前已经是有序的了 所以如果之前 b 已经在 a 中那么现在找到的这条边
            // 就不是最短边 因此算法可以保证找到最小边 不需要去重
            p[pb] = pa;
            cnt++;
            res += w;
        }
    }
    
    // n 个节点应该连 n-1 条边
    if (cnt < n-1) cout << "impossible";
    else cout << res;
}
```

易错点：

1. 排序时搞错 `m` 和 `n`，`edges` 这个数组的长度是 `m`。
2. 并查集初始化让每个元素是它的根节点。
3. 漏 `cnt` 变量。