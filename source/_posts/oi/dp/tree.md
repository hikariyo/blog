---
date: 2023-10-03 23:01:00
updated: 2023-10-03 23:01:00
title: 动态规划 - 树形 DP
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
description: 树形 DP 的主要思想是在树上对每个结点设计状态，并且父结点的状态可以从子结点转移。
---

## 简介

树形 DP 的主要思想是在树上对每个结点设计状态，并且父结点的状态可以从子结点转移。

## 树的重心

> 给定 $n\le 10^5$ 个点的树，求删除重心后各连通块的最大值。
>
> 重心指“删除该点后各连通块的最大值”最小的一个点。
>
> 题目链接：[AcWing 646](https://www.acwing.com/problem/content/848/)。

看起来不太像

连通块只能是当前点向下走或者向上走，当 $u$ 为当前搜到的结点时，它们分别是 $sz(to),n-sz(u)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010, M = 200010;

int h[N], sz[N], n, idx = 1;
struct Edge {
    int to, nxt;
} e[M];

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

int ans = 1e9+7;
void dfs(int u, int fa) {
    sz[u] = 1;
    
    int res = 0;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa) continue;
        dfs(to, u);
        sz[u] += sz[to];
        res = max(res, sz[to]);
    }
    res = max(res, n-sz[u]);
    ans = min(ans, res);
}

int main() {
    scanf("%d", &n);
    for (int i = 1, a, b; i < n; i++) {
        scanf("%d%d", &a, &b);
        add(a, b), add(b, a);
    }
    dfs(1, 0);
    return !printf("%d\n", ans);
}
```

## 树的直径

> 给定一棵树，树中包含 n 个结点（编号1~n）和 n−1 条无向边，每条边都有一个权值。
>
> 现在请你找到树中的一条最长路径。
>
> 换句话说，要找到一条路径，使得使得路径两端的点的距离最远。
>
> 注意：路径中可以只包含一个点。
>
> 题目链接：[AcWing 1072](https://www.acwing.com/problem/content/1074/)。

直径一定是某个顶点的子树中的所有路径的最大路径加上次大路径（可以与前者相同），这点和前面做过的次小生成树问题不同，那里的次大值必须是严格小于最大值，否则会出现问题。

这里就用 DFS 递归去找了，其返回的是从当前结点出发的最大路径，这里证明一下为什么不需要求子树的次大路径：对于根结点 $u$ 和子结点 $v_i$，用 $d_1,d_2$ 表示最大和次大值。

1. 如果 $d_1,d_2$ 连接在同一个结点 $v_i$ 上，这是不符合直径的要求的。

2. 如果 $d_1,d_2$ 分别连接在 $v_i\ne v_j$ 上，那么根据定义一定有 $d_2(v_j)\le d_1(v_j)$，从而 $d_1(v_j)$ 就是整个的次大值，而不是 $d_2(v_j)$，证毕。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 10010, M = 20010;
int h[N], e[M], w[M], ne[M], idx;
int ans;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

int dfs(int u, int fa) {
    // 初始化成零可以起到不考虑最大路径为负数的效果
    // 因为路径中可以只包含一个点
    int d1 = 0, d2 = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        int d = dfs(j, u) + w[i];
        // 如果等于最大值, 也要更新次大值, 说明有两条等大的最大路径
        if (d >= d1) d2 = d1, d1 = d;
        else if (d > d2) d2 = d;
    }
    ans = max(ans, d1 + d2);
    return d1;
}

int main() {
    int n, m;
    memset(h, -1, sizeof h);
    scanf("%d", &n);
    for (int i = 0; i < n-1; i++) {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c), add(b, a, c);
    }
    dfs(1, -1);
    printf("%d\n", ans);
    return 0;
}
```

## 树的中心

> 给定一棵树，树中包含 n 个结点（编号1~n）和 n−1 条无向边，每条边都有一个权值。
>
> 请你在树中找到一个点，使得该点到树中其他结点的最远距离最近。
>
> 题目链接：[AcWing 1073](https://www.acwing.com/problem/content/1075/)。

一个点的最远距离分两种情况，要么是向上走，要么向下走，记录向下走的最大路径和次大路径分别是 $d_1, d_2$

当存在相邻结点 $a\leftrightarrow b \leftrightarrow c$ 时，对于 $b$：

+ 向下走的最大路径是：
    $$
    d_1(b)=\max_{\forall c_i}\{\max\{d_1(c_i),0\}+w(b,c_i)\}
    $$
    当 $d_1(c_i)$ 为负时把它舍掉，直接用 $w(b,c_i)$ 作为向下走的最大路径。

+ 向上走的最大路径是：

    $$
    up(b)=\left\{\begin{aligned}
    &\max\{d_1(a),up(a),0\}+w(a,b),b\notin d_1(a)\\
    &\max\{d_2(a),up(a),0\}+w(a,b),b\in d_1(a)
    \end{aligned}\right.
    $$

当存在两条相同的最大路径时，$d_1=d_2$，所以这种情况会被正确处理；当 $d_1,d_2,up$ 都为负时，就只保留 $w(a,b)$ 作为向上走的最大路径。

接下来分析特殊结点的性质：

+ 对于根节点，它的路径一定是向下的，因此 $up(root)=-\infin,d_1(root)\ne -\infin$；
+ 对于叶子结点，它的路径一定是向上的，因此 $d_1(v_l)=d_2(v_l)=-\infin,up(v_l)\ne -\infin$

综上，一个点到其它点的最远距离就是 $\max\{d_1(v),up(v)\}$

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;

#define max3(a, b, c) max(a, max(b, c))

const int N = 10010, M = 20010, INF = 0x3f3f3f3f;
int h[N], e[M], w[M], ne[M], idx;
int d1[N], d2[N], g1[N], up[N];
int ans = 1e9;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void dfs_d(int u, int fa) {
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        
        // 需要先知道子结点 因此是先递归再计算当前结点
        dfs_d(j, u);
        // 当子结点向下走为负数时舍掉整个子树
        int d = max(d1[j], 0) + w[i];
        if (d >= d1[u]) d2[u] = d1[u], d1[u] = d, g1[u] = j;
        else if (d > d2[u]) d2[u] = d;
    }
}

void dfs_u(int u, int fa) {
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        
        if (g1[u] == j) up[j] = max3(up[u], d2[u], 0) + w[i];
        else up[j] = max3(up[u], d1[u], 0) + w[i];
        // 需要先知道父结点 因此是先计算当前结点再递归
        dfs_u(j, u);
    }
}

int main() {
    int n;
    memset(up, -0x3f, sizeof up);
    memset(d1, -0x3f, sizeof d1);
    memset(d2, -0x3f, sizeof d2);
    memset(h, -1, sizeof h);
    scanf("%d", &n);
    for (int i = 0; i < n-1; i++) {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c), add(b, a, c);
    }
    dfs_d(1, -1);
    dfs_u(1, -1);
    
    int res = INF;
    for (int i = 1; i <= n; i++) {
        // printf("Node %d, up %d, down %d\n", i, up[i], d1[i]);
        res = min(res, max(d1[i], up[i]));
    }
    printf("%d\n", res);
    return 0;
}
```

## [HAOI2009] 毛毛虫

> “毛毛虫” 的大小定义为一条路径上能通过一条边到达的所有点的数量。
>
> **输入格式**
>
> 输入中第一行两个整数 $N, M$，分别表示树中结点个数和树的边数。
>
> 接下来 $M$ 行，每行两个整数 $a, b$ 表示点 $a$ 和点 $b$ 有边连接（$a, b \le N$）。你可以假定没有一对相同的 $(a, b)$ 会出现一次以上。
>
> **输出格式**
>
> 输出一行一个整数 , 表示最大的毛毛虫的大小。
>
> 对于 $100\%$ 的数据，$1\leq N \le 300000$。

设 $f_u$ 是以 $u$ 为根的最大值，这里设 $1$ 为根，$cnt_u$ 代表 $u$ 的所有子结点的数量：

1. 当 $u$ 为叶结点时，$f_u=1$

2. 当 $u$ 有唯一儿子时，$f_u=f_v+1$

3. 当 $u$ 有多个儿子时，如果 $u$ 是根节点，那么：
    $$
    f_u=f_1+f_2+cnt_u-2+1=f_1+f_2+cnt_u-1
    $$
    如果 $u$ 不是根节点，那么：
    $$
    f_u=f_1+f_2+cnt_u-2+1+1=f_1+f_2+cnt_u
    $$
    其中 $f_1,f_2$ 代表子结点中的最大和非严格次大值。

可以证明，如果存在多个儿子，“分叉” 的选择一定更优：
$$
f_1+f_2+cnt_u-1\gt f_1+cnt_u
$$
因为 $cnt_u\gt 1$ 是成立的。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 300010, M = 2*N;
int n, m, ans;
int h[N], f[N], e[M], ne[M], idx;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

int dfs(int u, int fa) {
    if (f[u]) return f[u];

    int s1 = 0, s2 = 0, cnt = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        int s = dfs(j, u);
        cnt++;
        if (s > s1) s2 = s1, s1 = s;
        else if (s > s2) s2 = s;
    }

    int res = -1;

    if (cnt >= 2) res = s1 + s2 + cnt - !fa, f[u] = s1 + cnt;
    else if (cnt == 1) res = f[u] = s1 + 1;
    else res = f[u] = 1;

    ans = max(ans, res);
    return f[u];
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d", &n, &m);
    for (int i = 0, a, b; i < m; i++) {
        scanf("%d%d", &a, &b);
        add(a, b), add(b, a);
    }
    dfs(1, 0);
    return !printf("%d\n", ans);
}
```

## [CTSC1997] 选课（树形背包）

> 第一行有两个整数 $N$ , $M$ 用空格隔开。( $1 \leq N \leq 300$ , $1 \leq M \leq 300$ )
>
> 接下来的 $N$ 行,第 $I+1$ 行包含两个整数 $k_i$ 和 $s_i$, $k_i$ 表示第I门课的直接先修课，$s_i$ 表示第I门课的学分。若 $k_i=0$ 表示没有直接先修课（$1 \leq {k_i} \leq N$ , $1 \leq {s_i} \leq 20$）。
>
> 输出选 $M$ 门课程的最大得分。
>
> 题目链接：[P2014](https://www.luogu.com.cn/problem/P2014)。

树形背包有一个 $O(nm^2)$ 的做法，但通过 DFS 序可以优化到 $O(nm)$，这里就不提那个更劣的解法了。

DFS 序有一个很好的性质，就是对于结点 $u$，与它相邻的不包含在它子树内的编号是 $dfn(u)+size(u)$。

我们设状态 $f(i,j)$ 是 $dfn$ 在 $i\sim n$ 内，体积为 $j$ 的选择，那么对于每个结点都分为两种情况：

1. 选子树，状态是 $f(i+1,j-v(u))+w(u)$
2. 不选子树，状态是 $f(i+\text{size}(u),j)$

如果是叶节点，这样操作的结果就是用比它 DFS 序靠后的叶结点来更新它的最大价值：
$$
f(i,j)=\max\begin{cases}
f(i+1,j-v(u))+w(u)\\
f(i+\text{size}(u),j)
\end{cases}
$$
因此我们在这个叶子结点的父结点时就能正确取到 $i\sim n$ 的最大价值。因此，我们需要倒序遍历子结点，这样能保证状态该转移的时候已经被计算出来，所以这里选择用 `vector` 存图比较方便。

注意，由于我们处理的是一个森林，所以虚拟出来一个超级源点 $0$，它的体积同样也是 $1$，因此要在程序开头给 $m$ 加一。

```cpp
#include <cstdio.
#include <vector>
#include <algorithm>
using namespace std;

const int N = 310;
int n, m;
int f[N][N], w[N], sz[N], dfn[N], cnt;
vector<int> g[N];

int dfs(int u) {
    dfn[u] = ++cnt, sz[u] = 1;
    for (int v: g[u]) sz[u] += dfs(v);
    return sz[u];
}

void dp(int u) {
    for (int i = g[u].size() - 1; i >= 0; i--) dp(g[u][i]);
    for (int j = 1; j <= m; j++)
        f[dfn[u]][j] = max(f[dfn[u]+1][j-1]+w[u], f[dfn[u]+sz[u]][j]);
}

int main() {
    scanf("%d%d", &n, &m), ++m;
    for (int i = 1, fa; i <= n; i++) {
        scanf("%d%d", &fa, &w[i]);
        g[fa].emplace_back(i);
    }
    dfs(0), dp(0);
    return !printf("%d\n", f[1][m]);
}
```

## [SDOI2006] 保安站岗（最小点覆盖）

> 给定一棵 $N(1\le N\le 1500)$ 个点的树，每条边都必须被覆盖（即每条边的两个端点至少选择一个），求满足条件的最小点权和。
>
> 题目链接：[P2458](https://www.luogu.com.cn/problem/P2458)。

考虑每个点都有三种情况：在这个点放置、在这个点的父结点放置、这个点的子结点放置。分别用 $0/1/2$ 表示这些状态。
$$
\begin{aligned}
f(u,0)&=w_u+\sum_{j\in son_u} \min\{f(j,0),f(j,1),f(j,2)\}\\
f(u,1)&=\sum_{j\in son_u} \min\{f(j,0),f(j,2)\}\\
f(u,2)&=\sum_{j\in son_u} \min\{f(j,0),f(j,2)\},\exist f(j,0)\\
\end{aligned}
$$
对于 $f(u,2)$ 我们必须选出一个 $j$ 使它的状态是在这个点放置，这样才能保证 $u$ 被观察到。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 1550, M = 3010, INF = 0x3f3f3f3f;
int h[N], w[N], e[M], ne[M], idx;
int n, f[N][3];

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u, int fa) {
    f[u][0] = w[u], f[u][1] = 0, f[u][2] = INF;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        dfs(j, u);
        f[u][0] += min(min(f[j][0], f[j][1]), f[j][2]);
        f[u][1] += min(f[j][0], f[j][2]);
    }
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        f[u][2] = min(f[u][2], f[u][1] - min(f[j][0], f[j][2]) + f[j][0]);
    }
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d", &n);
    for (int i = 1, id, m; i <= n; i++) {
        scanf("%d", &id), scanf("%d%d", &w[id], &m);
        for (int j = 0, s; j < m; j++)
            scanf("%d", &s), add(id, s), add(s, id);
    }
    dfs(1, 0);
    return !printf("%d\n", min(f[1][0], f[1][2]));
}
```

## [CSPS2019] 括号树

> **题目背景**
>
> 本题中**合法括号串**的定义如下：
>
> 1.	`()` 是合法括号串。
> 2.	如果 `A` 是合法括号串，则 `(A)` 是合法括号串。
> 3.	如果 `A`，`B` 是合法括号串，则 `AB` 是合法括号串。
>
> 本题中**子串**与**不同的子串**的定义如下：
> 1.	字符串 `S` 的子串是 `S` 中**连续**的任意个字符组成的字符串。`S` 的子串可用起始位置 $l$ 与终止位置 $r$ 来表示，记为 $S (l, r)$（$1 \leq l \leq r \leq |S |$，$|S |$ 表示 S 的长度）。
> 2.	`S` 的两个子串视作不同**当且仅当**它们在 `S` 中的位置不同，即 $l$ 不同或 $r$ 不同。
>
> **题目描述**
>
> 一个大小为 $n$ 的树包含 $n$ 个结点和 $n - 1$ 条边，每条边连接两个结点，且任意两个结点间**有且仅有**一条简单路径互相可达。
>
> 小 Q 是一个充满好奇心的小朋友，有一天他在上学的路上碰见了一个大小为 $n$ 的树，树上结点从 $1 \sim n$ 编号，$1$ 号结点为树的根。除 $1$ 号结点外，每个结点有一个父亲结点，$u$（$2 \leq u \leq n$）号结点的父亲为 $f_u$（$1 ≤ f_u < u$）号结点。
>
> 小 Q 发现这个树的每个结点上**恰有**一个括号，可能是`(` 或`)`。小 Q 定义 $s_i$ 为：将根结点到 $i$ 号结点的简单路径上的括号，按结点经过顺序依次排列组成的字符串。
>
> 显然 $s_i$ 是个括号串，但不一定是合法括号串，因此现在小 Q 想对所有的 $i$（$1\leq i\leq n$）求出，$s_i$ 中有多少个**互不相同的子串**是**合法括号串**。
>
> 这个问题难倒了小 Q，他只好向你求助。设 $s_i$ 共有 $k_i$ 个不同子串是合法括号串， 你只需要告诉小 Q 所有 $i \times k_i$ 的异或和，即：
> $$
> (1 \times k_1)\ \text{xor}\ (2 \times k_2)\ \text{xor}\ (3 \times k_3)\ \text{xor}\ \cdots\ \text{xor}\ (n \times k_n)
> $$
>
> 其中 $xor$ 是位异或运算。
>
> **输入格式**
>
> 第一行一个整数 $n$，表示树的大小。
>
> 第二行一个长为 $n$ 的由`(` 与`)` 组成的括号串，第 $i$ 个括号表示 $i$ 号结点上的括号。
>
> 第三行包含 $n − 1$ 个整数，第 $i$（$1 \leq i \lt n$）个整数表示 $i + 1$ 号结点的父亲编号 $f_{i+1}$。
>
> **输出格式**
>
> 仅一行一个整数表示答案。
>

首先考虑序列，如果一个序列 $a$ 必须以 $a_i$ 结尾的所有子括号串数量为 $t_i$，在前 $i$ 个字符中的所有数量为 $k_i$，显然需要使 `a[i]=')'`；然后 $a_j$ 为和 $a_i$ 匹配的括号串，需要 `a[j]='('`。

如果选定了 $a_i$，可以看出 $a_j\sim a_i$ 这一段是必选的，否则不是合法括号串。而 $a_1\sim a_{j-1}$ 可选可不选，所以需要把不选的情况加上，那么可以得到转移方程：
$$
t_i=t_{j-1}+1
$$
在前 $i$ 个字符中满足条件的所有子串数量累项求和即可，即对于任意 $k_i=k_{i-1}+t_i$

现在放到树上，也只是把 $j-1$ 变成了 $j$ 的父结点，因为在序列中 $-1$ 代表上一项，树中父结点就是上一项。

```cpp
#include <cstdio>
#include <stack>
#include <vector>
using namespace std;

typedef long long LL;
const int N = 500010;

int n, fa[N];
LL t[N], k[N];
char s[N];
stack<int> stk;
vector<int> g[N];

void dfs(int u) {
    k[u] = k[fa[u]];
    if (s[u] == '(') {
        stk.push(u);
        for (int j: g[u]) dfs(j);
        return stk.pop();
    }
    
    // s[u] == ')'
    if (stk.size()) {
        int p = stk.top(); stk.pop();
        t[u] = t[fa[p]] + 1;
        k[u] += t[u];
        for (int j: g[u]) dfs(j);
        return stk.push(p);
    }
    
    for (int j: g[u]) dfs(j);
}

int main() {
    scanf("%d%s", &n, s+1);
    for (int i = 2; i <= n; i++)
        scanf("%d", &fa[i]), g[fa[i]].emplace_back(i);
    dfs(1);
    LL res = 0;
    for (int i = 1; i <= n; i++)
        res ^= i*k[i];
    return !printf("%lld\n", res);
}
```

## [USACO12FEB] Nearby Cows G

> 给定一棵 $n$ 个点的树，点带权，对于每个节点求出距离它不超过 $k$ 的所有节点权值和 $m$。
>
> 题目链接：[P3047](https://www.luogu.com.cn/problem/P3047)。

思路是预处理出 $f(u,k)$ 表示结点 $u$ 向下走 $k$ 层的所有点权之和，很容易得出递推式：
$$
f(u,k)=w_u+\sum_{j\in son_u} f(j,k-1)
$$
查询的时候，我们就一级一级的祖先查，写一个循环即可。注意这里要用到一个容斥，设 $pre$ 是上一次跳转的结点，那么当前加上的权重应当是：
$$
f(u,i)-f(pre,i-1)
$$
这样避免重复计算。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 100010, M = 200010, K = 25;
int h[N], w[N], e[M], ne[M], idx;
int fa[N], f[N][K];
int n, k;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u, int father) {
    fa[u] = father;
    for (int i = 0; i <= k; i++) f[u][i] = w[u];
    

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == father) continue;
        dfs(j, u);
        for (int p = 1; p <= k; p++) f[u][p] += f[j][p-1];
    }
}

int get(int u) {
    int res = 0, i = k, pre = 0;
    while (i >= 0 && u) {
        res += f[u][i];
        if (pre != 0) res -= f[pre][i-1];
        pre = u, u = fa[u], i--;
    }
    return res;
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d", &n, &k);
    for (int i = 1, u, v; i < n; i++)
        scanf("%d%d", &u, &v), add(u, v), add(v, u);
    for (int i = 1; i <= n; i++)
        scanf("%d", &w[i]);
    dfs(1, 0);
    for (int i = 1; i <= n; i++) printf("%d\n", get(i));
    return 0;
}
```

## [USACO17DEC] Barn Painting G（计数 DP）

>  给定一颗 $N$ 个节点组成的树，你要给每个点涂上三种颜色之一，其中有 $K$ 个节点已染色，要求任意两相邻节点颜色不同，求合法染色方案数。答案对 $10^9+7$ 取模。
>
> 题目链接：[P4084](https://www.luogu.com.cn/problem/P4084)。

可以设 $f(u,c)$ 表示当前点涂 $c$ 颜色的方案数，如果输入已经给定了 $u$ 结点的颜色，就把其它颜色设为 $0$，否则都设为 $1$。

容易得到转移方程：
$$
f(u,1)=\prod_{j\in son_u} [f(j,2)+f(j,3)]\\
f(u,2)=\prod_{j\in son_u} [f(j,1)+f(j,3)]\\
f(u,3)=\prod_{j\in son_u} [f(j,1)+f(j,2)]
$$
这里一开始用 $0,1,2$ 当颜色坑了我半天，因为我直接用 $if(color(u))$ 判断，而 $0$ 又是一个颜色，所以就疯狂 WA，还是不省这点空间了。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;

const int N = 100010, M = 200010, P = 1e9+7;
int h[N], color[N], e[M], ne[M], idx;
int n, k;
LL f[N][4];

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u, int fa) {
    if (color[u]) f[u][color[u]] = 1;
    else f[u][1] = f[u][2] = f[u][3] = 1;

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        dfs(j, u);
        f[u][1] = f[u][1] * (f[j][2] + f[j][3]) % P;
        f[u][2] = f[u][2] * (f[j][1] + f[j][3]) % P;
        f[u][3] = f[u][3] * (f[j][1] + f[j][2]) % P;
    }
}

int main() {
    memset(h, -1, sizeof h);

    scanf("%d%d", &n, &k);
    for (int i = 1, a, b; i < n; i++) {
        scanf("%d%d", &a, &b);
        add(a, b), add(b, a);
    }
    
    for (int i = 0, a, b; i < k; i++) {
        scanf("%d%d", &a, &b);
        color[a] = b;
    }

    dfs(1, 0);
    return !printf("%lld\n", (f[1][1] + f[1][2] + f[1][3]) % P);
}
```

