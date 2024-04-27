---
date: 2023-12-03 17:52:00
update: 2023-12-29 23:52:00
title: 数据结构 - 可持久化
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
description: 可持久化数据结构包括可持久化数组，可持久化并查集，可持久化线段树，都是以复用现有结点的思想记录各个历史版本，从而做到在较低的空间复杂度储存每一次修改后数据结构中的数据。
---

## 可持久化数组

> 如题，你需要维护这样的一个数组，支持如下几种操作
>
>
> 1. 在某个历史版本上修改某一个位置上的值
>
> 2. 访问某个历史版本上的某一位置的值
>
>
> 此外，每进行一次操作（**对于操作2，即为生成一个完全一样的版本，不作任何改动**），就会生成一个新的版本。版本编号即为当前操作的编号（从1开始编号，版本0表示初始状态数组）
>
> 对于100%的数据：$1 \leq N, M \leq {10}^6, 1 \leq {loc}_i \leq N, 0 \leq v_i < i, -{10}^9 \leq a_i, {value}_i  \leq {10}^9$
>
> 题目链接：[P3919](https://www.luogu.com.cn/problem/P3919)。

由于线段树优秀的树形结构非常适合做可持久化，所以实际上是用的可持久化线段树。

具体地说，把每次更新都把整个路径复制下来，不改变的地方就指回原结点，所以每次都会增加 $O(\log n)$ 结点，空间复杂度是 $O(n\log n)$，至于图满大街都是，我就不贴了。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((L+R) >> 1)
const int N = 1000010, K = 22;
int val[N*K], ls[N*K], rs[N*K], a[N], rt[N], idx;
int n, m;

int query(int u, int p, int L, int R) {
    if (L == R) return val[u];
    if (p <= Mid) return query(ls[u], p, L, Mid);
    return query(rs[u], p, Mid+1, R);
}

void insert(int u, int v, int p, int k, int L, int R) {
    val[u] = val[v], ls[u] = ls[v], rs[u] = rs[v];
    if (L == R) return val[u] = k, void();
    if (p <= Mid) ls[u] = ++idx, insert(ls[u], ls[v], p, k, L, Mid);
    else rs[u] = ++idx, insert(rs[u], rs[v], p, k, Mid+1, R);
}

int build(int L, int R) {
    int u = ++idx;
    if (L == R) return val[u] = a[L], u;
    ls[u] = build(L, Mid), rs[u] = build(Mid+1, R);
    return u;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);

    rt[0] = build(1, n);
    for (int i = 1, v, opt, loc, val; i <= m; i++) {
        scanf("%d%d%d", &v, &opt, &loc);
        if (opt == 1) scanf("%d", &val), rt[i] = ++idx, insert(rt[i], rt[v], loc, val, 1, n);
        else rt[i] = rt[v], printf("%d\n", query(rt[i], loc, 1, n));
    }

    return 0;
}
```

## 可持久化并查集

> 给定 $n$ 个集合，第 $i$ 个集合内初始状态下只有一个数，为 $i$。
>
> 有 $m$ 次操作。操作分为 $3$ 种：
>
>  - `1 a b` 合并 $a,b$ 所在集合；
>
>  - `2 k` 回到第 $k$ 次操作（执行三种操作中的任意一种都记为一次操作）之后的状态；
>
>  - `3 a b` 询问 $a,b$ 是否属于同一集合，如果是则输出 $1$，否则输出 $0$。
>
> 对于 $100\%$ 的数据，$1\le n\le 10^5$，$1\le m\le 2\times 10^5$，$1 \le a, b \le n$。
>
> 题目链接：[P3402](https://www.luogu.com.cn/problem/P3402)。

首先需要证明一下按秩合并的复杂度，我喜欢用 $sz$ 合并，这样写起来省事，下面用 $h(x)$ 代表树高，$sz(x)$ 代表树的全部结点个数。

将 $A$ 合并到 $B$ 中，令 $sz(A)\le sz(B)$，接下来有两种情况：

1. $h(A)<h(B)$，合并后会使得 $h(A)\leftarrow h(A)+1$，此时依然有 $h(A)\le h(B)$，所以整体树高不变。
2. $h(A)\ge h(B)$，这会使得树高加一，但是同时有合并之后 $sz(B)\ge 2sz(A)$，也就是较小的会翻倍，而翻倍这个操作至多进行 $O(\log n)$ 次，因此树高是 $O(\log n)$ 的。

计算一下建树需要的空间：
$$
\begin{aligned}
f(n)&=1+2f(\frac{n}{2})\\
&=1+2+4f(\frac{n}{4})\\
&=1+2+4+8f(\frac{n}{8})\\
&=2^{O(\log_2 n)}-1\\
&=O(n)
\end{aligned}
$$
为了保险，我们开 $4n$ 给建树用，由于进行 $m$ 次操作，每次操作都会更改两个值，产生两个历史版本，所以加上最开始的 $4n$ 一共需要 $4n+2m\log n$ 空间。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010, M = 200010, LogN = 20;
int n, m, rt[M];
#define Mid ((L+R) >> 1)

struct Node {
    int ls, rs;
    int fa, sz;
} tr[2*M*LogN+N*4];
int idx;

void build(int &u, int L, int R) {
    u = ++idx;
    if (L == R) return tr[u].fa = L, tr[u].sz = 1, void();
    build(tr[u].ls, L, Mid), build(tr[u].rs, Mid+1, R);
}

int clone(int u) {
    tr[++idx] = tr[u];
    return idx;
}

Node query(int u, int p, int L, int R) {
    if (L == R) return tr[u];

    if (p <= Mid) return query(tr[u].ls, p, L, Mid);
    return query(tr[u].rs, p, Mid+1, R);
}

Node& update(int u, int p, int L, int R) {
    if (L == R) return tr[u];
    if (p <= Mid) return tr[u].ls = clone(tr[u].ls), update(tr[u].ls, p, L, Mid);
    return tr[u].rs = clone(tr[u].rs), update(tr[u].rs, p, Mid+1, R);
}

int find(int u, int x) {
    int fa = query(u, x, 1, n).fa;
    if (fa != x) return find(u, fa);
    return x;
}

void merge(int u, int a, int b) {
    int pa = find(u, a), pb = find(u, b);
    if (pa == pb) return;
    int A = query(u, pa, 1, n).sz, B = query(u, pb, 1, n).sz;
    if (A <= B) {
        update(u, pa, 1, n).fa = pb;
        update(u, pb, 1, n).sz = A+B;
    }
    else {
        update(u, pb, 1, n).fa = pa;
        update(u, pa, 1, n).sz = A+B;
    }
}

int main() {
    scanf("%d%d", &n, &m);
    build(rt[0], 1, n);
    for (int i = 1, opt, a, b; i <= m; i++) {
        scanf("%d", &opt);
        if (opt == 2) scanf("%d", &a), rt[i] = rt[a];
        else if (opt == 3) {
            scanf("%d%d", &a, &b);
            rt[i] = rt[i-1];
            printf("%d\n", find(rt[i], a) == find(rt[i], b));
        }
        else {
            scanf("%d%d", &a, &b);
            rt[i] = clone(rt[i-1]);
            merge(rt[i], a, b);
        }
    }
    return 0;
}
```

## 可持久化权值线段树

> 静态区间第 $k$ 小，$1\le n,m\le 2\times 10^5, |a_i|\le 10^9, 1\le l\le r\le n, 1\le k\le r-l+1$。
>
> 题目链接：[P3834](https://www.luogu.com.cn/problem/P3834)。

经典主席树，全称可持久化前缀权值线段树，即每次查询都用两个版本的结点相减得出每个结点在对应查询时的值，然后线段树上二分。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((L+R) >> 1)
const int N = 200010, K = 20;
int n, m, rt[N], a[N], nums[N], cntnums;
int ls[N*K], rs[N*K], cnt[N*K];
int idx;

void insert(int u, int v, int k, int L, int R) {
    cnt[u] = cnt[v] + 1, ls[u] = ls[v], rs[u] = rs[v];
    if (L == k && R == k) return;
    if (k <= Mid) ls[u] = ++idx, insert(ls[u], ls[v], k, L, Mid);
    else rs[u] = ++idx, insert(rs[u], rs[v], k, Mid+1, R);
}

int query(int u, int v, int k, int L, int R) {
    if (L == R) return L;
    int c = cnt[ls[v]] - cnt[ls[u]];
    if (k <= c) return query(ls[u], ls[v], k, L, Mid);
    else return query(rs[u], rs[v], k-c, Mid+1, R);
}

int find(int x) {
    return lower_bound(nums+1, nums+1+cntnums, x) - nums;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]), nums[i] = a[i];
    sort(nums+1, nums+1+n);
    cntnums = unique(nums+1, nums+1+n) - (nums+1);

    for (int i = 1; i <= n; i++) rt[i] = ++idx, insert(rt[i], rt[i-1], find(a[i]), 1, cntnums);

    for (int i = 1, x, y, k; i <= m; i++) {
        scanf("%d%d%d", &x, &y, &k);
        printf("%d\n", nums[query(rt[x-1], rt[y], k, 1, cntnums)]);
    }

    return 0;
}
```

## Count on a tree

> 给定一棵 $n$ 个节点的树，每个点有一个权值。有 $m$ 个询问，每次给你 $u,v,k$，你需要回答 $u \text{ xor last}$ 和 $v$ 这两个节点间第 $k$ 小的点权。  
>
> 其中 $\text{last}$ 是上一个询问的答案，定义其初始为 $0$，即第一个询问的 $u$ 是明文。
>
> 对于 $100\%$ 的数据，$1\le n,m \le 10^5$，点权在 $[1, 2 ^ {31} - 1]$ 之间。
>
> 题目链接：[P2633](https://www.luogu.com.cn/problem/P2633)。

对于每个结点，统计它到根路径上的前缀和，然后查询一个路径 $a,b$ 设 $u=lca(a,b)$，对应的每个数字的 $cnt$ 均为：
$$
cnt(a)+cnt(b)-cnt(u)-cnt(fa(u))
$$
因此这就是主席树套了一个树上的幌子：

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((L+R) >> 1)
const int N = 100010, K = 20;
int n, m, len, w[N], nums[N], h[N], rt[N], fa[N][K], dep[N], idx = 2;

struct Edge {
    int to, nxt;
} e[N<<1];

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch ^ 48), ch = getchar();
}

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

struct SGT {
    int cnt[N*K], ls[N*K], rs[N*K], idx;

    void insert(int u, int v, int p, int L, int R) {
        cnt[u] = cnt[v] + 1, ls[u] = ls[v], rs[u] = rs[v];

        if (L == R) return;
        if (p <= Mid) ls[u] = ++idx, insert(ls[u], ls[v], p, L, Mid);
        else rs[u] = ++idx, insert(rs[u], rs[v], p, Mid+1, R);
    }

    int query(int a, int b, int c, int d, int k, int L, int R) {
        if (L == R) return L;
        int tot = cnt[ls[a]] + cnt[ls[b]] - cnt[ls[c]] - cnt[ls[d]];
        if (k <= tot) return query(ls[a], ls[b], ls[c], ls[d], k, L, Mid);
        else return query(rs[a], rs[b], rs[c], rs[d], k - tot, Mid+1, R);
    }
} T;

void dfs(int u, int f) {
    rt[u] = ++T.idx;
    T.insert(rt[u], rt[f], w[u], 1, len);
    fa[u][0] = f, dep[u] = dep[f] + 1;
    for (int k = 1; k < K; k++) fa[u][k] = fa[fa[u][k-1]][k-1];
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        dfs(to, u);
    }
}

int lca(int a, int b) {
    if (dep[a] < dep[b]) swap(a, b);
    for (int k = K-1; k >= 0; k--)
        if (dep[fa[a][k]] >= dep[b])
            a = fa[a][k];
    
    if (a == b) return a;
    for (int k = K-1; k >= 0; k--)
        if (fa[a][k] != fa[b][k])
            a = fa[a][k], b = fa[b][k];
    return fa[a][0];
}

int main() {
    read(n), read(m);
    for (int i = 1; i <= n; i++) read(w[i]), nums[i] = w[i];
    sort(nums+1, nums+1+n);
    len = unique(nums+1, nums+1+n) - (nums+1);
    for (int i = 1; i <= n; i++) w[i] = lower_bound(nums+1, nums+1+len, w[i]) - nums;
    for (int i = 1, a, b; i < n; i++) {
        read(a), read(b);
        add(a, b), add(b, a);
    }
    dfs(1, 0);

    int lastans = 0;
    for (int i = 1, u, v, k; i <= m; i++) {
        read(u), read(v), read(k);
        u ^= lastans;
        int l = lca(u, v);
        lastans = nums[T.query(rt[u], rt[v], rt[l], rt[fa[l][0]], k, 1, len)];
        printf("%d\n", lastans);
    }

    return 0;
}
```

## Mex

> 给定 $n$ 个元素的数组 $a$，$m$ 次询问 $[l,r]$ 的最小没有出现过的自然数。$1\le n,m,a_i\le 2\times10^5$。
>
> 题目链接：[P4137](https://www.luogu.com.cn/problem/P4137)。

建立主席树，维护 $a_i\to i$ 的映射，默认为 $v\to 0$，即维护每个数字最后出现的坐标。

这样查询的时候第 $r$ 个版本的线段树上查询第一个 $\lt l$ 即 $\le l-1$ 的权值，树上二分即可做到 $O(n\log n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((L+R) >> 1)
const int N = 200010, M = 20*N;

int n, m, rt[N], ls[M], rs[M], val[M], idx;

void update(int& u, int v, int p, int k, int L = 0, int R = 2e5) {
    u = ++idx, ls[u] = ls[v], rs[u] = rs[v];
    if (L == R) return val[u] = k, void();
    if (p <= Mid) update(ls[u], ls[v], p, k, L, Mid);
    else update(rs[u], rs[v], p, k, Mid+1, R);
    val[u] = min(val[ls[u]], val[rs[u]]);
}

int query(int u, int k, int L = 0, int R = 2e5) {
    if (L == R) return L;
    if (val[ls[u]] <= k) return query(ls[u], k, L, Mid);
    else return query(rs[u], k, Mid+1, R);
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1, a; i <= n; i++) {
        scanf("%d", &a);
        update(rt[i], rt[i-1], a, i);
    }
    for (int i = 1, l, r; i <= m; i++) {
        scanf("%d%d", &l, &r);
        printf("%d\n", query(rt[r], l-1));
    }
    return 0;
}
```

## [十二省联考 2019] 异或粽子

> 小粽是一个喜欢吃粽子的好孩子。今天她在家里自己做起了粽子。
>
> 小粽面前有 $n$ 种互不相同的粽子馅儿，小粽将它们摆放为了一排，并从左至右编号为 $1$ 到 $n$。第 $i$ 种馅儿具有一个非负整数的属性值 $a_i$。每种馅儿的数量都足够多，即小粽不会因为缺少原料而做不出想要的粽子。小粽准备用这些馅儿来做出 $k$ 个粽子。
>
> 小粽的做法是：选两个整数数 $l$,  $r$，满足 $1 \leqslant l \leqslant r \leqslant n$，将编号在 $[l, r]$ 范围内的所有馅儿混合做成一个粽子，所得的粽子的美味度为这些粽子的属性值的**异或和**。
>
> 小粽想品尝不同口味的粽子，因此它不希望用同样的馅儿的集合做出一个以上的粽子。
>
> 小粽希望她做出的所有粽子的美味度之和最大。请你帮她求出这个值吧！
>
> 题目链接：[P5283](https://www.luogu.com.cn/problem/P5283)。

题意：求前 $k$ 大异或和之和。

首先用前缀和转化一下题目，即求前 $k$ 大的点对 $(i,j)$ 使得 $0\le i<j\le n$ 且 $S_i\oplus S_j$ 为这个点对的权值。

这里有一个常用的 Trick，就是用堆动态维护最大值，对于点对 $P=(i,j)$ 同时记录 $L(P),R(P)$ 分别表示当前的 $i$ 的取值范围，即在 $[L(P),R(P)]$ 中所有的数中 $i$ 是使得 $S_i\oplus S_j$ 是最大的，那么取出一个值的时候就将这个区间分裂成两个，分别是 $[L(P),i-1],[i+1,R(P)]$，如果有 $i=L(P)$ 或 $i=R(P)$ 就扔掉那个区间，接下来就要解决如何在 $S_j$ 固定的情况下在一个区间内求出 $S_i\oplus S_j$ 的最大值，显然用可持久化 Trie 可以在 $O(\log a)$ 的复杂度解决，这样复杂度是 $O(n\log a+k(\log n+\log a))$，同样的 Trick 见 [P2048](https://www.luogu.com.cn/problem/P2048)。

由于 $i,j$ 是 $[0,n]$ 的范围，而我们又用的是前缀和算法，所以需要一个 $-1$ 来应对左边界为 $0$ 的情况，我们可以建一个指针指向数组的首地址的下一个地址，这样就可以用 $-1$ 作为下标了。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 500010, M = 40 * N;
int _rt[N], s[N], tr[M][2], cnt[M], id[M], idx;
int *rt = _rt+1;
int n, k;

struct Range {
    int l, r, k, raw, val;

    bool operator<(const Range& r) const {
        return val < r.val;
    }
};

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

int clone(int u) {
    idx++;
    tr[idx][0] = tr[u][0];
    tr[idx][1] = tr[u][1];
    cnt[idx] = cnt[u];
    id[idx] = id[u];
    return idx;
}

void insert(int& u, int v, int bits, int k, int ident) {
    u = clone(v), ++cnt[u];
    if (bits == -1) return id[u] = ident, void();
    int b = k >> bits & 1;
    insert(tr[u][b], tr[v][b], bits-1, k, ident);
}

int query(int u, int v, int bits, int k) {
    if (bits == -1) return id[v];
    int b = k >> bits & 1;
    if (cnt[tr[v][b^1]] - cnt[tr[u][b^1]] > 0) return query(tr[u][b^1], tr[v][b^1], bits-1, k);
    return query(tr[u][b], tr[v][b], bits-1, k);
}

signed main() {
    read(n), read(k);

    insert(rt[-1], 0, 31, 0, -1);
    insert(rt[0], rt[-1], 31, 0, 0);
    for (int i = 1, a; i <= n; i++) {
        read(a);
        insert(rt[i], rt[i-1], 31, s[i] = s[i-1] ^ a, i);
    }
    priority_queue<Range> heap;
    for (int i = 1; i <= n; i++) {
        int id = query(rt[-1], rt[i-1], 31, s[i]);
        heap.push({0, i-1, id, s[i], s[i] ^ s[id]});
    }

    int sum = 0;
    while (k--) {
        Range rg = heap.top(); heap.pop();
        sum += rg.val;
        if (rg.l < rg.k) {
            int id = query(rt[rg.l-1], rt[rg.k-1], 31, rg.raw);
            heap.push({rg.l, rg.k-1, id, rg.raw, rg.raw ^ s[id]});
        }

        if (rg.k < rg.r) {
            int id = query(rt[rg.k], rt[rg.r], 31, rg.raw);
            heap.push({rg.k+1, rg.r, id, rg.raw, rg.raw ^ s[id]});
        }
    }

    printf("%lld\n", sum);
    return 0;
}
```

## [NOI2018] 归程
> 本题的故事发生在魔力之都，在这里我们将为你介绍一些必要的设定。
>
> 魔力之都可以抽象成一个 $n$ 个节点、$m$ 条边的无向连通图（节点的编号从 $1$ 至 $n$）。我们依次用 $l,a$ 描述一条边的**长度、海拔**。
>
> 作为季风气候的代表城市，魔力之都时常有雨水相伴，因此道路积水总是不可避免的。由于整个城市的排水系统连通，因此**有积水的边一定是海拔相对最低的一些边**。我们用**水位线**来描述降雨的程度，它的意义是：所有海拔**不超过**水位线的边都是**有积水**的。
>
> Yazid 是一名来自魔力之都的 OIer，刚参加完 ION2018 的他将踏上归程，回到他温暖的家。Yazid 的家恰好在魔力之都的 $1$ 号节点。对于接下来 $Q$ 天，每一天 Yazid 都会告诉你他的出发点 $v$ ，以及当天的水位线 $p$。
>
> 每一天，Yazid 在出发点都拥有一辆车。这辆车由于一些故障不能经过有积水的边。Yazid 可以在任意节点下车，这样接下来他就可以步行经过有积水的边。但车会被留在他下车的节点并不会再被使用。
> 需要特殊说明的是，第二天车会被重置，这意味着：
> - 车会在新的出发点被准备好。
> - Yazid 不能利用之前在某处停放的车。
>
> Yazid 非常讨厌在雨天步行，因此他希望在完成回家这一目标的同时，最小化他**步行经过的边**的总长度。请你帮助 Yazid 进行计算。
>
> 本题的部分测试点将强制在线，具体细节请见【输入格式】和【子任务】。
>
> **输入格式**
>
> 单个测试点中包含多组数据。输入的第一行为一个非负整数 $T$，表示数据的组数。
>
> 接下来依次描述每组数据，对于每组数据：
>
> 第一行 $2$ 个非负整数 $n,m$，分别表示节点数、边数。
>
> 接下来 $m$ 行，每行 $4$ 个正整数 $u, v, l, a$，描述一条连接节点 $u, v$ 的、长度为 $l$、海拔为 $a$ 的边。
> 在这里，我们保证 $1 \leq u,v \leq n$。
>
> 接下来一行 $3$ 个非负数 $Q, K, S$ ，其中 $Q$ 表示总天数，$K \in {0,1}$ 是一个会在下面被用到的系数，$S$ 表示的是可能的最高水位线。
>
> 接下来 $Q$ 行依次描述每天的状况。每行 $2$ 个整数 $v_0,  p_0$ 描述一天：
> - 这一天的出发节点为 $v = (v_0 + K \times \mathrm{lastans} - 1) \bmod n + 1$。    
> - 这一天的水位线为 $p = (p_0 + K \times \mathrm{lastans}) \bmod (S + 1)$。    
>
> 其中 $\mathrm{lastans}$ 表示上一天的答案（最小步行总路程）。特别地，我们规定第 $1$ 天时 $\mathrm{lastans} = 0$。 
> 在这里，我们保证 $1 \leq v_0 \leq n$，$0 \leq p_0 \leq S$。
>
> 对于输入中的每一行，如果该行包含多个数，则用单个空格将它们隔开。
>
> **输出格式**
>
> 依次输出各组数据的答案。对于每组数据：
>
> - 输出 $Q$ 行每行一个整数，依次表示每天的最小步行总路程。
>
>**数据范围**
> 
>所有测试点均保证 $T\leq 3$，所有测试点中的所有数据均满足如下限制：
> 
>- $n\leq 2\times 10^5$，$m\leq 4\times 10^5$，$Q\leq 4\times 10^5$，$K\in\left\{0,1\right\}$，$1\leq S\leq 10^9$。
> - 对于所有边：$l\leq 10^4$，$a\leq 10^9$。
> - 任意两点之间都直接或间接通过边相连。
> 
>题目链接：[P4768](https://www.luogu.com.cn/problem/P4768)。

所以转化完成之后可以看出就是求当积水为某值的时候，所有海拔高于这个值的边是可以零花费的，于是我们先从 $1$ 开始跑 Dijkstra 处理出每个点到 $1$ 的距离，然后用可持久化并查集，将边按照海拔降序排序，对每个边依次建立可持久化并查集的版本，并且每个连通块的权值都是当前连通块中所有点的 $dist$ 最小值。

发现在初始化的时候用一个普通并查集判断连通性会更快一点，这个是我看了题解之后才想到的。

对于每个询问，找到大于询问值的最后一条边，用该边对应的并查集版本查询即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define Mid ((L+R) >> 1)
const int N = 200010, M = 400010, LogN = 20, INF = 1e18;
int n, m, dist[N], h[N], idx = 2;
bool st[N];

struct Edge {
    int to, nxt, w;
} e[M<<1];

struct DSUEdge {
    int a, b, h;

    bool operator<(const DSUEdge& ed) const {
        return h > ed.h;
    }
} edge[M];

struct PII {
    int fi, se;
    bool operator<(const PII& p) const {
        return fi > p.fi;
    }
};

struct SGT {
    struct Node {
        int ls, rs;
        int fa, sz, dis;
    } tr[M*LogN*2];

    int rt[M], idx;

    int build(int L, int R) {
        int u = ++idx;
        if (L == R) tr[u].fa = L, tr[u].sz = 1, tr[u].dis = dist[L];
        else tr[u].ls = build(L, Mid), tr[u].rs = build(Mid+1, R);
        return u;
    }

    Node& update(int& u, int p, int L, int R) {
        tr[++idx] = tr[u];
        u = idx;

        if (L == R) return tr[u];

        if (p <= Mid) return update(tr[u].ls, p, L, Mid);
        else return update(tr[u].rs, p, Mid+1, R);
    }

    Node query(int u, int p, int L, int R) {
        if (L == R) return tr[u];

        if (p <= Mid) return query(tr[u].ls, p, L, Mid);
        else return query(tr[u].rs, p, Mid+1, R);
    }
} T;

struct DSU {
    int p[N];

    int find(int x) {
        if (p[x] != x) p[x] = find(p[x]);
        return p[x];
    }

    void merge(int a, int b) {
        p[find(a)] = find(b);
    }

    void init(int n) {
        for (int i = 1; i <= n; i++) p[i] = i;
    }
} dsu;

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

void add(int a, int b, int c) {
    e[idx] = {b, h[a], c}, h[a] = idx++;
}

void dijkstra() {
    priority_queue<PII> heap;
    heap.push({0, 1});
    dist[1] = 0;

    while (!heap.empty()) {
        int t = heap.top().se; heap.pop();
        if (st[t]) continue;
        st[t] = true;
        for (int i = h[t]; i; i = e[i].nxt) {
            int to = e[i].to;
            if (dist[to] > dist[t] + e[i].w) {
                dist[to] = dist[t] + e[i].w;
                heap.push({dist[to], to});
            }
        }
    }
}

SGT::Node find(int rt, int x) {
    SGT::Node nd;
    while ((nd = T.query(rt, x, 1, n)).fa != x) {
        x = T.query(rt, x, 1, n).fa;
    }
    return nd;
}

void initdsu() {
    dijkstra();
    sort(edge+1, edge+1+m);
    T.idx = 0;

    dsu.init(n);
    T.rt[0] = T.build(1, n);
    for (int i = 1; i <= m; i++) {
        T.rt[i] = T.rt[i-1];
        
        int a = dsu.find(edge[i].a), b = dsu.find(edge[i].b);
        if (a == b) continue;
        SGT::Node& A = T.update(T.rt[i], a, 1, n), &B = T.update(T.rt[i], b, 1, n);
        if (A.sz <= B.sz) {
            A.fa = b;
            B.sz += A.sz;
            B.dis = min(A.dis, B.dis);
            dsu.merge(a, b);
        }
        else {
            B.fa = a;
            A.sz += B.sz;
            A.dis = min(A.dis, B.dis);
            dsu.merge(b, a);
        }
    }
}

void solve() {
    read(n), read(m);
    idx = 2;
    for (int i = 1; i <= n; i++) dist[i] = INF, st[i] = h[i] = 0;
    for (int i = 1, a, b, c, h; i <= m; i++) {
        read(a), read(b), read(c), read(h);
        add(a, b, c), add(b, a, c);
        edge[i] = {a, b, h};
    }
    initdsu();
    edge[0].h = INF;

    int lastans = 0, Q, K, S, v, p;
    read(Q), read(K), read(S);
    while (Q--) {
        read(v), read(p);
        v = (v + K*lastans - 1) % n + 1;
        p = (p + K*lastans) % (S+1);

        int l = 0, r = m;
        while (l < r) {
            int mid = (l+r+1) >> 1;
            if (edge[mid].h > p) l = mid;
            else r = mid-1;
        }
        SGT::Node nd = find(T.rt[r], v);
        printf("%lld\n", lastans = nd.dis);
    }
}

signed main() {
    int T;
    read(T);
    while (T--) solve();
    return 0;
}
```

## [USACO19DEC] Milk Visits G
> Farmer John 计划建造 $N$ 个农场，用 $N-1$ 条道路连接，构成一棵树（也就是说，所有农场之间都互相可以到达，并且没有环）。每个农场有一头奶牛，品种为 $1$ 到 $N$ 之间的一个整数 $T_i$。
>
> Farmer John 的 $M$ 个朋友经常前来拜访他。在朋友 $i$ 拜访之时，Farmer John 会与他的朋友沿着从农场 $A_i$ 到农场 $B_i$ 之间的唯一路径行走（可能有 $A_i = B_i$）。除此之外，他们还可以品尝他们经过的路径上任意一头奶牛的牛奶。由于 Farmer John 的朋友们大多数也是农场主，他们对牛奶有着极强的偏好。他的每个朋友都只喝某种特定品种的奶牛的牛奶。任何 Farmer John 的朋友只有在他们访问时能喝到他们偏好的牛奶才会高兴。
>
> 请求出每个朋友在拜访过后是否会高兴。
>
> 对于 $100\%$ 的数据，$1 \leq N \leq 10^5$，$1 \leq M \leq 10^5$。
>
> 题目链接：[P5838](https://www.luogu.com.cn/problem/P5838)。

本题有更优秀的离线做法，这里只是提供一个用主席树的在线做法。

考虑到维护一个区间某个数字的计数这个问题主席树轻松解决，而这里只是问是否存在，所以可以树剖主席树。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define Mid ((L+R) >> 1)
const int N = 100010, M = 20*N;
int n, q, rt[N], a[N], dfna[N], cnt[M], ls[M], rs[M], idx;
int h[N], fa[N], top[N], dep[N], dfn[N], sz[N], son[N], ts, tot = 2;
char ans[N];
struct Edge {
    int to, nxt;
} e[N<<1];

void add(int a, int b) {
    e[tot] = {b, h[a]}, h[a] = tot++;
}

void insert(int& u, int v, int p, int L, int R) {
    u = ++idx;
    cnt[u] = cnt[v] + 1, ls[u] = ls[v], rs[u] = rs[v];
    if (L == R) return;
    if (p <= Mid) insert(ls[u], ls[v], p, L, Mid);
    else insert(rs[u], rs[v], p, Mid+1, R);
}

int query(int u, int v, int p, int L, int R) {
    if (L == R) return cnt[u] - cnt[v];
    if (p <= Mid) return query(ls[u], ls[v], p, L, Mid);
    else return query(rs[u], rs[v], p, Mid+1, R);
}

void dfs1(int u, int f) {
    sz[u] = 1, fa[u] = f, dep[u] = dep[f] + 1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == f) continue;
        dfs1(to, u);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    top[u] = t, dfn[u] = ++ts, dfna[ts] = a[u];
    if (!son[u]) return;
    dfs2(son[u], t);
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == son[u] || to == fa[u]) continue;
        dfs2(to, to);
    }
}

int querypath(int a, int b, int c) {
    while (top[a] != top[b]) {
        if (dep[top[a]] < dep[top[b]]) swap(a, b);
        if (query(rt[dfn[a]], rt[dfn[top[a]]-1], c, 1, n)) return 1;
        a = fa[top[a]];
    }
    if (dfn[a] > dfn[b]) swap(a, b);
    return query(rt[dfn[b]], rt[dfn[a]-1], c, 1, n) > 0;
}

int main() {
    scanf("%d%d", &n, &q);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
    for (int i = 1, a, b; i < n; i++) scanf("%d%d", &a, &b), add(a, b), add(b, a);
    dfs1(1, 0);
    dfs2(1, 1);
    for (int i = 1; i <= n; i++) insert(rt[i], rt[i-1], dfna[i], 1, n);
    for (int i = 1, a, b, c; i <= q; i++) {
        scanf("%d%d%d", &a, &b, &c);
        ans[i] = querypath(a, b, c) ^ 48;
    }
    printf("%s\n", ans+1);
    return 0;
}
```

