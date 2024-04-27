---
date: 2023-07-21 16:13:12
title: 图论 - 差分约束
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 简介

### 原理

利用**三角不等式**的可以用图论的方式求解一个不等式组，设 $\delta(s,x)$ 为 $s \to x$ 的最短路长度，那么：
$$
\delta(s,x)\le\delta(s,y)+w(y,x)
$$
对于下面的不等式组，可以构造一个图来求最短路找到一个可行解。
$$
\left\{\begin{aligned}
x_1&\le x_2+C_1\\
x_2&\le x_3+C_2\\
\cdots\\
x_i&\le x_j+C_i
\end{aligned}\right.
$$
对每个不等式 $x_i \le x_j + C_i$，变换成三角不等式 $\delta(s,i)\le \delta(s,j)+w(j,i)$，于是就可以构造出有向边 $(j,i)$ 边权为 $C_i$ 。

### 步骤

下面是差分约束的一般步骤。

1. 先将每个不等式转化为一条边。
2. 找一个超级原点 $s$ 该点一定可以遍历到所有边。
3. 从原点求一遍单源最短路。
    1. 如果存在负环，该不等式组一定无解。
    2. 如果没有负环，$\delta(s,i)$ 就是原不等式的一个可行解。

4. 如果求每个变量的最小值，应该求最长路；如果求最大值，应该求最短路。

关于最后一点，如果存在最值那么一定存在一个绝对关系 $x_0\ge C_0$ 或 $x_0\le C_0$，否则任意一组可行解都可以加上任意一个常数，最值就不存在了。

假设已知 $x_0 \le C_0$，求 $x_k$ 的最大值，那么设有两条从 $x_0 \to x_k$ 的路径：
$$
x_0\to \cdots \to x_i \to x_k\\
x_0\to \cdots \to x_j \to x_k\\
x_k\le x_i+w(i,k)\le\cdots\le x_0+\sum w_i\\
x_k\le x_j+w(j,k)\le\cdots\le x_0+\sum w_j
$$
因为要满足这些不等式组，所以要求的是所有上界的最小值，求最短路。反之亦然。

下面是例题。

## 区间

> 给定 n 个区间 $[a_i, b_i]$ 和 整数$c_i$。
>
> 构造整数集合 Z，使得 $\forall i\in [1,n]$，Z 中满足 $a_i \le x \le b_i$ 中的数不小于 $c_i$ 个。
>
> 求 $|Z|$ 的最小值。
>
> 原题链接：[AcWing 362](https://www.acwing.com/problem/content/364/)。

找差分约束的不等式组，设 $S_i$ 为 $1\sim i$ 中选中数字的个数。
$$
\left \{ \begin{aligned}
&S_0=0\\
&S_i\ge S_{i-1}\\
&S_i-S_{i-1}\le 1 \Rightarrow S_{i-1}\ge S_i-1\\
&S_b-S_{a-1}\ge c \Rightarrow S_b\ge S_{a-1}+c
\end{aligned}\right.
$$
数据范围给的 $a,b \in [0, 50000]$ 这里要用到 $a-1$ 因此给所有输入的 $a,b$ 自增一下，因此最后 $|Z|=S_{50001}$

求最小值，所以用最长路。

```cpp
#include <iostream>
#include <queue>
#include <cstring>
using namespace std;

const int N = 50010, M = 3*N;
int h[N], e[M], ne[M], w[M], dist[N], idx;
bool st[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void spfa() {
    queue<int> q;
    q.push(0);
    memset(dist, -0x3f, sizeof dist);
    dist[0] = 0;
    
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] < dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                if (!st[j]) q.push(j), st[j] = true;
            }
        }
    }
}

int main() {
    int n; cin >> n;
    memset(h, -1, sizeof h);
    for (int i = 1; i <= 50001; i++) {
        add(i-1, i, 0);
        add(i, i-1, -1);
    }
    
    while (n--) {
        int a, b, c;
        cin >> a >> b >> c;
        a++, b++;
        add(a-1, b, c);
    }
    
    spfa();
    cout << dist[50001] << endl;

    return 0;
}
```

## 雇佣收银员

> 一家超市要每天 24 小时营业，为了满足营业需求，需要雇佣一大批收银员。
>
> 已知不同时间段需要的收银员数量不同，为了能够雇佣尽可能少的人员，从而减少成本，这家超市的经理请你来帮忙出谋划策。
>
> 经理为你提供了一个各个时间段收银员最小需求数量的清单 R(0),R(1),R(2),…,R(23)。
>
> R(0) 表示午夜 00:00 到凌晨 01:00 的最小需求数量，R(1) 表示凌晨 01:00 到凌晨 02:00 的最小需求数量，以此类推。
>
> 一共有 N 个合格的申请人申请岗位，第 i 个申请人可以从 ti 时刻开始连续工作 8 小时。
>
> 收银员之间不存在替换，一定会完整地工作 8 小时，收银台的数量一定足够。
>
> 现在给定你收银员的需求清单，请你计算最少需要雇佣多少名收银员。
>
> 原题链接：[AcWing 393](https://www.acwing.com/problem/content/395/)。

列不等式，设 $x_i$ 我们雇佣从第 $i$ 时开始工作的人数，而 $cap_i$ 是人数上限，那么：
$$
\left \{ \begin{aligned}
&0\le x_i \le ca p_i\\
&x_i+x_{i-1}+\cdots+x_{i-7}\ge R(i)
\end{aligned}\right .
$$
这个式子显然不符合差分约束的形式，可以用前缀和的思想进行转化，设 $S_i$ 为前 $i(i\in [0, 23])$ 小时开始工作的人数之和。由于需要用到前缀和，所以把下标加一，那么：
$$
\left \{ \begin{aligned}
&S_0=0\\
&S_i-S_{i-1} \ge 0\\
&S_i-S_{i-1} \le cap_i\\
&S_i-S_{i-8}\ge R(i),i\ge 8\\
&S_i+S_{24}-S_{i+16}\ge R(i),0\le i \le 7
\end{aligned}\right .
$$
这里可以二分求 $S_{24}$ 的最小值，如果图中不存在正环，说明 `mid >= ans` 执行 `r=mid`；否则 `l=mid+1`，这样就能二分出最小值。

由于 $S_0=0$，因此可以构造 $S_{24}\ge S_0+S_{24},S_{24}\le S_0+S_{24}$ 保证 $S_{24}$ 为常量。

```cpp
#include <iostream>
#include <cstring>
#include <queue>
using namespace std;

const int N = 30, M = 300;
int R[30], cap[N];
int h[N], cnt[N], dist[N], e[M], ne[M], w[M], idx;
bool st[N];
int n;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

bool spfa(int S24) {

    memset(h, -1, sizeof h);
    idx = 0;
    for (int i = 1; i <= 24; i++) {
        add(i-1, i, 0);
        add(i, i-1, -cap[i]);
        if (i >= 8) add(i-8, i, R[i]);
        else add(i+16, i, R[i]-S24);
    }
    add(0, 24, S24), add(24, 0, -S24);
            
    memset(st, 0, sizeof st);
    memset(cnt, 0, sizeof cnt);
    memset(dist, -0x3f, sizeof dist);
    queue<int> q;
    q.push(0);
    dist[0] = 0;
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] < dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                cnt[j] = cnt[t] + 1;
                if (cnt[j] >= 24) return false;
                if (!st[j]) st[j] = true, q.push(j);
            }
        }
    }
    return true;
}

int main() {
    int T; cin >> T;
    while (T--) {
        memset(cap, 0, sizeof cap);
        for (int i = 1; i <= 24; i++) cin >> R[i];
        cin >> n;
        for (int i = 0; i < n; i++) {
            int t; cin >> t;
            cap[t+1]++;
        }
        
        // 枚举 S24
        int l = 1, r = n;
        while (l < r) {
            int mid = l+r>>1;
            if (spfa(mid)) r=mid;
            else l=mid+1;
        }
        if (spfa(l)) cout << l << endl;
        else cout << "No Solution" << endl;
    }
    return 0;
}
```

## [SCOI2011] 糖果

> 幼儿园里有 N 个小朋友，老师现在想要给这些小朋友们分配糖果，要求每个小朋友都要分到糖果。
>
> 但是小朋友们也有嫉妒心，总是会提出一些要求，比如小明不希望小红分到的糖果比他的多，于是在分配糖果的时候， 老师需要满足小朋友们的 K 个要求。
>
> 幼儿园的糖果总是有限的，老师想知道他至少需要准备多少个糖果，才能使得每个小朋友都能够分到糖果，并且满足小朋友们所有的要求。
>
> 输入的第一行是两个整数 N,K。
>
> 接下来 K 行，表示分配糖果时需要满足的关系，每行 3 个数字 X,A,B。
>
> - 如果 X=1．表示第 A 个小朋友分到的糖果必须和第 B 个小朋友分到的糖果一样多。
> - 如果 X=2，表示第 A 个小朋友分到的糖果必须少于第 B 个小朋友分到的糖果。
> - 如果 X=3，表示第 A 个小朋友分到的糖果必须不少于第 B 个小朋友分到的糖果。
> - 如果 X=4，表示第 A 个小朋友分到的糖果必须多于第 B 个小朋友分到的糖果。
> - 如果 X=5，表示第 A 个小朋友分到的糖果必须不多于第 B 个小朋友分到的糖果。
>
> 小朋友编号从 1 到 N。
>
> 原题链接：[AcWing 1169](https://www.acwing.com/problem/content/1171/)。

这里是求最小值，因此是最长路，设最长路长度为 $\delta(x)$ 其同样满足三角不等式 $\delta(x)\ge\delta(y)+w(y,x)$；所以对于不等式 $A\ge B+C$ 可以建 $w(B,A)=C$ 的边；所有下界的最大值才是最小值。

接下来分析条件。
$$
\left \{\begin{aligned}
&x=1,A\ge B,B\ge A\\
&x=2,B\ge A+1\\
&x=3,A\ge B\\
&x=4,A\ge B+1\\
&x=5,B\ge A\\
&A,B\ge 1
\end{aligned}\right .
$$
最后一个是隐式条件。

```cpp
#include <iostream>
#include <stack>
#include <cstring>
using namespace std;

const int N = 1e5+10, M = 2e5+10;
int n, k;
int h[N], e[M], ne[M], w[M], idx;
bool st[N];
int dist[N], cnt[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

bool spfa() {
    stack<int> q;
    for (int i = 1; i <= n; i++) {
        q.push(i);
        st[i] = true;
        dist[i] = 1;
        cnt[i] = 0;
    }
    
    int times = 0;
    while (q.size()) {
        int t = q.top(); q.pop();
        st[t] = false;
        if (++times >= 3*n) return true;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] < dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                cnt[j] = cnt[t] + 1;
                if (cnt[j] >= n) return true;
                if (!st[j]) q.push(j), st[j] = true;
            }
        }
    }
    
    return false;
}

int main() {
    scanf("%d%d", &n, &k);
    memset(h, -1, sizeof h);
    while (k--) {
        int x, a, b;
        scanf("%d%d%d", &x, &a, &b);
        if (x == 1) add(a, b, 0), add(b, a, 0);
        if (x == 2) add(a, b, 1);
        if (x == 3) add(b, a, 0);
        if (x == 4) add(b, a, 1);
        if (x == 5) add(a, b, 0);
    }
    
    if (spfa()) puts("-1");
    else {
        long long ans = 0;
        for (int i = 1; i <= n; i++) ans += dist[i];
        printf("%lld\n", ans);
    }
    return 0;
}
```

## [CF241E] Flights

> 给定 $n$ 点 $m$ 边无边权有向图，求是否存在方案，能够使得 $1\sim n$ 的所有路径的长度总和相同，并且对于每个边权都满足 $1\le w\le 2$，如果可以输出 `Yes` 并按照输入的顺序输出 $m$ 个边权，如果不可以输出 `No`。
>
> 题目链接：[CF241E](https://codeforces.com/problemset/problem/241/E)。

如果满足每个边 $(a,b)$ 的条件，可以列出来 $2m$ 个不等式：
$$
1\le d_b -d_a \le 2\\
\Rightarrow \begin{cases}
d_b\le d_a+2\\
d_a\le d_b-1
\end{cases}
$$
用 SPFA 跑出结果之后直接令边长 $(a,b)$ 为差分约束中的 $\text{dist}_b -\text{dist}_a$，这样从 $1\sim n$ 加起来一定是 $\text{dist}_n-\text{dist}_1$，能够保证所有路线的边权都相同。

但是还有一个问题没有考虑，就是有些点并不能到终点，对于这些点以及延伸出来的边，它们是否满足这个约束是无关紧要的，如果这些点之间构成环也无所谓，所以我们需要判断哪些点在 $1\sim n$ 的路径上，简单的方法就是从 $1,n$ 出发分别在正向图和反向图中进行一遍 `Flood-fill` 标记，只有两次都被标记的点处在路径上。

```cpp
#include <queue>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 1010, M = 15010;
int n, m, L[M], R[M];
int h[N], rh[N], e[M], ne[M], w[M], idx;
int dist[N], cnt[N];
bool valid[N], rvalid[N], st[N];
#define va(u) (valid[u] && rvalid[u])

void add(int h[], int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void dfs(int h[], bool st[], int u) {
    st[u] = true;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!st[j]) dfs(h, st, j);
    }
}

bool SPFA() {
    queue<int> q;
    memset(dist, 0x3f, sizeof dist);
    q.push(1), dist[1] = 0, cnt[1] = 1;

    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (!va(j)) continue;
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                cnt[j] = cnt[t] + 1;
                if (cnt[j] > n) return false;
                if (!st[j]) q.push(j), st[j] = true;
            }
        }
    }
    return true;
}

int main() {
    memset(h, -1, sizeof h);
    memset(rh, -1, sizeof rh);
    scanf("%d%d", &n, &m);
    for (int i = 0, a, b; i < m; i++) {
        scanf("%d%d", &a, &b);
        add(h, a, b, 2), add(rh, b, a, 0);
        L[i] = a, R[i] = b;
    }
    dfs(h, valid, 1), dfs(rh, rvalid, n);
    for (int i = 0; i < m; i++) add(h, R[i], L[i], -1);
    
    // 由于题目保证连通, 不用判连通性
    if (!SPFA()) return puts("No"), 0;
    puts("Yes");
    for (int i = 0; i < m; i++) {
        if (va(L[i]) && va(R[i])) printf("%d\n", dist[R[i]] - dist[L[i]]);
        else puts("1");
    }

    return 0;
}
```

## [USACO05DEC] Layout G

> 正如其他物种一样，奶牛们也喜欢在排队打饭时与它们的朋友挨在一起。FJ 有编号为 $1\dots N$ 的 $N$ 头奶牛 $(2\le N\le 1000)$。开始时，奶牛们按照编号顺序来排队。奶牛们很笨拙，因此可能有多头奶牛在同一位置上（即假设 $i$ 号奶牛位于 $P_i$，则 $P_1\le P_2\le \cdots \le P_n$）。
>
> 有些奶牛是好基友，它们希望彼此之间的距离小于等于某个数。有些奶牛是情敌，它们希望彼此之间的距离大于等于某个数。
>
> 给出 $M_L$ 对好基友的编号，以及它们希望彼此之间的距离小于等于多少；又给出 $M_D$ 对情敌的编号，以及它们希望彼此之间的距离大于等于多少 $(1\le M_L,$ $M_D\le 10^4)$。
>
> 请计算：如果满足上述所有条件，$1$ 号奶牛和 $N$ 号奶牛之间的距离最大为多少。
>
> **输入格式**
>
> 第一行：三个整数 $N, M_L, M_D$，用空格分隔。
>
> 第 $2\dots M_L+1$ 行：每行三个整数 $A, B, D$，用空格分隔，表示 $A$ 号奶牛与 $B$ 号奶牛之间的距离须 $\le D$。保证 $1\le A<B\le N,$ $1\le D\le 10^6$. 
>
> 第 $M_L+2\dots M_L+M_D+1$ 行：每行三个整数 $A, B, D$，用空格分隔，表示 $A$ 号奶牛与 $B$ 号奶牛之间的距离须 $\ge D$。保证 $1\le A<B\le N,$ $1\le D\le 10^6$. 
>
> **输出格式**
>
> 一行，一个整数。如果没有合法方案，输出 `-1`. 如果有合法方案，但 $1$ 号奶牛可以与 $N$ 号奶牛相距无穷远，输出 `-2`. 否则，输出 $1$ 号奶牛与 $N$ 号奶牛间的最大距离。
>
> 题目链接：[P4878](https://www.luogu.com.cn/problem/P4878)。

本题洛谷的 Hack 数据比较恶心，题目里是没有 $P_1\le P_2\le \cdots P_n$ 这句话的，是我自己加上的，所以这是多处来的 $n-1$ 个差分约束条件，而且给的数据也不保证 $1$ 号点无负环时别的点就没有负环，所以需要先判断负环再求最短路。

```cpp
#include <queue>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 1010, M = 21010, INF = 0x3f3f3f3f;
int n, m1, m2;
int h[N], e[M], ne[M], w[M], idx;
int cnt[N], dist[N];
bool st[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

bool SPFA_CIR() {
    queue<int> q;
    for (int i = 1; i <= n; i++) q.push(i), st[i] = true;
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                cnt[j] = cnt[t] + 1;
                if (cnt[j] > n) return true;
                if (!st[j]) st[j] = true, q.push(j);
            }
        }
    }
    return false;
}

void SPFA() {
    queue<int> q;
    memset(dist, 0x3f, sizeof dist);
    q.push(1), dist[1] = 0;
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                if (!st[j]) st[j] = true, q.push(j);
            }
        }
    }
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d%d", &n, &m1, &m2);
    int a, b, d;

    for (int i = 2; i <= n; i++) {
        add(i, i-1, 0);
    }

    while (m1--) {
        scanf("%d%d%d", &a, &b, &d);
        add(a, b, d);
    }

    while (m2--) {
        scanf("%d%d%d", &a, &b, &d);
        add(b, a, -d);
    }

    if (SPFA_CIR()) return puts("-1"), 0;
    SPFA();
    if (dist[n] == INF) puts("-2");
    else printf("%d\n", dist[n]);

    return 0;
}
```

## [POI2012] FES-Festival

> 你不知道越野赛的结果，但 Byteasar 会告诉你部分信息。现在，他要你答出：所有参赛者最多能达到多少种不同的成绩，而不违背他给的条件。（他们中的一些人可能平局，也就是同时到达终点，这种情况只算有一种成绩）。
>
> Byteasar 告诉你，所有参赛者的成绩都是整数秒。他还会为你提供了一些参赛者成绩的关系。具体是：他会给你一些数对 $(A, B)$，表示 $A$ 的成绩正好比 $B$ 快 $1$ 秒；他还会给你一些数对 $(C, D)$，表示 $C$ 的成绩不比 $D$ 慢。而你要回答的是：所有参赛者最多能达到多少种不同的成绩，而不违背他给的条件。
>
> **输入格式**
>
> 第一行有三个整数 $n, m_{1}, m_{2}$，分别表示选手人数、数对 $(A, B)$ 的数目、数对 $(C, D)$ 的数目。
>
> 接下来 $m_1$ 行，每行两个整数 $a_{i}, b_{i}$（$a_{i} \ne b_{i}$）。这表示选手 $a_{i}$ 的成绩恰比 $b_{i}$ 小 $1$ 秒。
>
> 接下来 $m_{2}$ 行，每行两个整数 $c_{i}, d_{i}$（$c_{i} \ne d_{i}$）。这表示选手 $c_{i}$ 的成绩不比 $d_{i}$ 的成绩差（也就是花的时间不会更多）。
>
> **输出格式**
>
> 如果有解，输出一行一个整数，表示所有选手最多能拿到的成绩的种数。如果无解，输出 `NIE`。
>
> 对于 $100\%$ 的数据，$2 \le n \le 600$，$1 \le m_{1} + m_{2} \le {10}^5$。
>
> 题目链接：[P3530](https://www.luogu.com.cn/problem/P3530)。

第一种情况，就是 $A=B-1$；对于第二种情况，是 $C\le D$，显然是差分约束。

对于题目要求的 “最多达到多少种不同的成绩” 这点，也就是要求我们按照要求构造出一组解，使得这组解中不同元素的个数尽可能多。在同一个强连通分量中的元素 $A,B$，它们必然有下面的不等式：
$$
B\le A+C_1\le \cdots \le B+C_1+C_2
$$
这就意味着，如果元素 $B$ 被确定下来，$A$ 是存在上下界的。而我们求最短路的时候实际上已经确定了起点元素的值为 $0$ 了。因此，我们对两两点对之间的最短路的最大值就是 $\Delta x = \max_{A,B\in V}|B-A|$，并且边权的大小只有 $0,1,-1$ 所以对答案的贡献是 $\Delta x+1$。

而不在同一个强连通分量中的点，当其中一个元素确定下来时，另外一个元素无法被确定上界或下界。

所以答案就是对每个强连通分量求一遍 $\Delta x+1$ 累加起来。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 610, M = 200010, INF = 0x3f3f3f3f;
int n, m1, m2;
int g[N][N], h[N], e[M], ne[M], w[M], cnt[N], dist[N], idx;
int dfn[N], low[N], cntscc, timestamp;
bool ins[N], st[N];
vector<int> sccs[N];
stack<int> stk;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
    g[a][b] = min(g[a][b], c);
}

bool SPFA() {
    queue<int> q;
    for (int i = 1; i <= n; i++) q.push(i), st[i] = true;

    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                cnt[j] = cnt[t] + 1;
                if (cnt[j] > n) return false;
                if (!st[j]) st[j] = true, q.push(j);
            }
        }
    }
    return true;
}

void tarjan(int u) {
    dfn[u] = low[u] = ++timestamp;
    ins[u] = true;
    stk.push(u);
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) tarjan(j), low[u] = min(low[u], low[j]);
        else if (ins[j]) low[u] = min(low[u], dfn[j]);
    }

    if (dfn[u] == low[u]) {
        ++cntscc;
        int y;
        do {
            y = stk.top(); stk.pop();
            ins[y] = false;
            sccs[cntscc].emplace_back(y);
        } while (y != u);
    }
}

int flyod(int sid) {
    for (int k: sccs[sid])
        for (int i: sccs[sid])
            for (int j: sccs[sid])
                g[i][j] = min(g[i][j], g[i][k] + g[k][j]);

    int res = 0;
    for (int i: sccs[sid])
        for (int j: sccs[sid])
            if (g[i][j] < INF / 2)
                res = max(res, g[i][j]);
    return res;
}

int main() {
    memset(g, 0x3f, sizeof g);
    memset(h, -1, sizeof h);
    scanf("%d%d%d", &n, &m1, &m2);
    int a, b;

    while (m1--) {
        scanf("%d%d", &a, &b);
        add(b, a, -1), add(a, b, 1);
    }

    while (m2--) {
        scanf("%d%d", &a, &b);
        add(b, a, 0);
    }

    if (!SPFA()) return puts("NIE"), 0;

    for (int i = 1; i <= n; i++)
        if (!dfn[i]) tarjan(i);

    int ans = 0;
    for (int i = 1; i <= cntscc; i++)
        ans += flyod(i) + 1;
    return !printf("%d\n", ans);
}
```

