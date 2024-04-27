---
date: 2024-01-15 05:11:00
title: 数据结构 - 左偏树
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
description: 左偏树是一种二叉树，它具有堆的性质，支持动态合并两颗左偏树，保持新树仍然满足左偏树的性质，并且合并两颗树的复杂度是 O(log(n))，同时用并查集维护堆顶，不会增加复杂度。
---

## 简介

左偏树是一种二叉树，它具有堆的性质，并且支持动态合并两颗左偏树，保持新树仍然满足左偏树的性质。并且合并两颗树的复杂度是 $O(\log n)$，同时用并查集维护堆顶，不会增加复杂度。下面记录一下左偏树的几个性质及复杂度分析。

用 $\text{dis}(u)$ 表示 $u$ 到最近的叶子结点的距离，对于叶子结点 $\text{dis}(u)=0$。对于所有的结点 $u$，都满足 $\text{dis}(\text{ls}(u))\ge \text{dis}(\text{rs}(u))$，因此 $\text{dis}(u)=\text{dis}(\text{rs}(u))+1$。

设 $\text{sz}(k)$ 为当 $k=\text{dis}(u)$ 时 $u$ 的子树大小，显然 $\text{sz}(0)=1$，那么有 $\text{dis}(\text{rs}(u))=k-1,\text{dis}(\text{ls}(u))\ge k-1$，那么就有：
$$
\text{sz}(k)\ge 2\text{sz}(k-1)+1\gt 2^k
$$
因此一直走右儿子的复杂度是 $O(\log n)$ 的，这就是左偏树保证复杂度的原理。

合并两颗左偏树的时候，与合并 FHQ-Treap 类似，首先找到应该谁当根，不妨设 $x$ 当根，$y$ 需要接着与右子树合并，因此递归合并 $\text{rs}(x),y$ 回溯时更新当前结点的 $\text{dis}$ 并且维护左偏性（即保证左儿子比右儿子深），显然这样操作一次的复杂度是 $O(\log n+\log m)$ 的，其中 $n,m$ 代表 $x,y$ 的大小。

并查集维护堆顶，并且删除一个结点之后打一个标记，这样就可以做到删除的效果了。如果需要复用结点，一定要**清空然后再重新加入**，不清空导致的问题在各大数据结构中都是让人头疼的。

## 模板

> 一开始有 $n$ 个小根堆，每个队包含一个数，支持 $m$ 次两种操作：
>
> 1. `1 x y`：将第 $x$ 个数和第 $y$ 个数所在的小根堆合并（若已经被删除或在用一个堆内，则无视此操作）。
> 2. `2 x`：输出第 $x$ 个数所在堆的最小数，并且将这个最小数删除（若有多个最小数，优先删除先输入的，若已经被删除，输出 $-1$ 并无视本操作）。
>
> 数据范围：$1\le n,m\le 10^5$ 所有数都在 `int` 的范围内。
>
> 题目链接：[P3377](https://www.luogu.com.cn/problem/P3377)。

这里用 `val[x] = -1` 标记元素是否被删除，少开一个数组。

特别地，删除时直接让这个元素和它两个儿子的父亲指向新的根就行，根据并查集的性质，之前指向这个元素的都会找到新根。

简单梳理一下，$x$ 儿子在合并的时候会把父亲指向 $x$，如果删除 $x$ 时只把 $x$ 的父亲指向改掉，那么会在并查集中造成环，所以需要更改 $x$ 和它的两个儿子的父亲指向，其它元素不用改，这样就不会出现环。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;
int n, m, p[N], dis[N], val[N], ls[N], rs[N];

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int merge(int x, int y) {
    if (x == y) return x;
    if (!x || !y) return x | y;
    if (val[x] == val[y] ? x > y : val[x] > val[y]) swap(x, y);
    rs[x] = merge(rs[x], y);
    if (dis[ls[x]] < dis[rs[x]]) swap(ls[x], rs[x]);
    dis[x] = dis[rs[x]] + 1;
    return x;
}

int main() {
    // 从头到尾也没有用到 dis 的数值，所以无需真的将 dis[0] 初始化为 -1
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &val[i]), p[i] = i;

    for (int i = 1, opt, x, y; i <= m; i++) {
        scanf("%d%d", &opt, &x);
        if (opt == 1) {
            scanf("%d", &y);
            if (val[x] == -1 || val[y] == -1) continue;
            x = find(x), y = find(y);
            p[x] = p[y] = merge(x, y);
        }
        else {
            if (val[x] == -1) {
                puts("-1");
                continue;
            }
            printf("%d\n", val[x = find(x)]);
            val[x] = -1;
            p[ls[x]] = p[rs[x]] = p[x] = merge(ls[x], rs[x]);
        }
    }
    return 0;
}
```

## Monkey King

> 曾经在一个森林中居住着 $N$ 只好斗的猴子。在最初他们我行我素，互不认识。但是猴子们不能避免争吵，且两只猴子只会在不认识对方时发生争吵，当争吵发生时，双方会邀请它们各自最强壮的朋友并发起决斗（决斗的为各自最强壮的朋友）。当然，在决斗之后两只猴子和他们各自的伙伴都认识对方了（成为朋友），虽然他们曾经有过冲突，但是他们之间绝不会再发生争吵了。
>
> 假设每只猴子有一个强壮值，强壮值将在一场决斗后减少为原先的一半（例如  $10$ 会减少到  $5$，而  $5$ 会减少到  $2$，即向下取整）。
>
> 我们也假设每只猴子都认识它自己（是自己的朋友）。即当他是他朋友中最强壮的，他自己就会去决斗。
>
> 数据范围：$N,M\leq 100000$，$s_{i}\leq 32768$，可能有多组测试数据。
>
> 题目链接：[P1456](https://www.luogu.com.cn/problem/P1456)。

减半操作就是先把这个点删掉，需要清空它的左右儿子，否则添加回去指针会乱，然后就添加回去。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;
int n, m, p[N], dis[N], val[N], ls[N], rs[N];

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int merge(int x, int y) {
    if (x == y) return x;
    if (!x || !y) return x | y;
    if (val[x] < val[y]) swap(x, y);
    rs[x] = merge(rs[x], y);
    if (dis[ls[x]] < dis[rs[x]]) swap(ls[x], rs[x]);
    dis[x] = dis[rs[x]] + 1;
    return x;
}

void half(int x) {
    val[x] >>= 1;
    p[ls[x]] = p[rs[x]] = p[x] = merge(ls[x], rs[x]);
    ls[x] = rs[x] = dis[x] = 0;  // 清空当前点
    p[x] = p[p[x]] = merge(p[x], x);
}

int query(int a, int b) {
    a = find(a), b = find(b);
    if (a == b) return -1;
    half(a), half(b);
    int res = max(val[a = find(a)], val[b = find(b)]);
    p[a] = p[b] = merge(a, b);
    return res;
}

int main() {
    while (scanf("%d", &n) == 1) {
        for (int i = 1; i <= n; i++) scanf("%d", &val[i]), p[i] = i, ls[i] = rs[i] = 0;
        scanf("%d", &m);
        for (int i = 1, x, y; i <= m; i++) {
            scanf("%d%d", &x, &y);
            printf("%d\n", query(x, y));
        }
    }
    return 0;
}
```

## [APIO2012] 派遣

> 在这个帮派里，有一名忍者被称之为 Master。除了 Master 以外，每名忍者都有且仅有一个上级。为保密，同时增强忍者们的领导力，所有与他们工作相关的指令总是由上级发送给他的直接下属，而不允许通过其他的方式发送。
>
> 现在你要招募一批忍者，并把它们派遣给顾客。你需要为每个被派遣的忍者支付一定的薪水，同时使得支付的薪水总额不超过你的预算。另外，为了发送指令，你需要选择一名忍者作为管理者，要求这个管理者可以向所有被派遣的忍者发送指令，在发送指令时，任何忍者（不管是否被派遣）都可以作为消息的传递人。管理者自己可以被派遣，也可以不被派遣。当然，如果管理者没有被排遣，你就不需要支付管理者的薪水。
>
>
> 你的目标是在预算内使顾客的满意度最大。这里定义顾客的满意度为派遣的忍者总数乘以管理者的领导力水平，其中每个忍者的领导力水平也是一定的。
>
>
> 写一个程序，给定每一个忍者 $i$ 的上级 $B_i$，薪水 $C_i$，领导力 $L_i$，以及支付给忍者们的薪水总预算 $M$，输出在预算内满足上述要求时顾客满意度的最大值。
>
> 数据范围：$1 \le N \le 10^5$，$1 \le M \le 10^9$，$0 \le B_i \lt i$，$1 \le C_i \le M$，$1 \le L_i \le 10^9$。
>
> 题目链接：[P1552](https://www.luogu.com.cn/problem/P1552)。

考虑枚举每个点作为管理者，问题就是求子树中 $\sum C_i\le M$ 时数量的最大值，贪心的思路是当 $\sum C_i\gt M$ 时从大的往小删除元素，这样做显然是不劣的，所以可以用可并堆来做。这里我一开始想成必须所有点的 $\text{LCA}$ 是枚举的点，然后无从下手，所以一定要认真读题面。

每个结点处理完了就直接把当前结点的可并堆合并到父亲上，由于题目保证 $C_i\le M$，所以当前堆一定非空，就省去了很多判断。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define eb emplace_back
const int N = 100010;
int n, m, ans, ls[N], dis[N], rs[N], val[N], sum[N], p[N], sz[N], l[N];
vector<int> g[N];

namespace IO { /* 省略 */ }
using IO::read;
using IO::write;

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int merge(int a, int b) {
    if (a == b) return a;
    if (!a || !b) return a | b;
    if (val[a] < val[b]) swap(a, b);
    rs[a] = merge(rs[a], b);
    if (dis[ls[a]] < dis[rs[a]]) swap(ls[a], rs[a]);
    dis[a] = dis[rs[a]] + 1;
    return a;
}

void dp(int u, int fa) {
    for (int to: g[u]) if (to != fa) dp(to, u);
    int f = find(u);
    while (sum[f] > m) {
        p[ls[f]] = p[rs[f]] = p[f] = merge(ls[f], rs[f]);
        sum[p[f]] = sum[f] - val[f];
        sz[p[f]] = sz[f] - 1;
        f = p[f];
    }
    int b = find(u);
    ans = max(ans, l[u] * sz[b]);
    if (fa) {
        int a = find(fa);
        p[a] = p[b] = merge(a, b);
        sz[p[a]] = sz[a] + sz[b];
        sum[p[a]] = sum[a] + sum[b];
    }
}

signed main() {
    int rt = 0;
    read(n, m);
    for (int i = 1, fa; i <= n; i++) {
        read(fa, val[i], l[i]);
        if (!fa) rt = i;
        else g[i].eb(fa), g[fa].eb(i);
        p[i] = i, sum[i] = val[i], sz[i] = 1;
    }
    dp(rt, 0);
    return write(ans), 0;
}
```

