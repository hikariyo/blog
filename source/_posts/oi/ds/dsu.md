---
date: 2023-06-14 21:51:43
update: 2023-11-14 23:06:00
title: 数据结构 - 并查集
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
---

## 简介

该数据结构的作用有如下两点：

1. 合并两个集合。
2. 查询两个元素是否在同一个集合中。

基本实现原理是把每个集合用一颗树表示，树根的编号是这个集合的编号，每个节点都存储它的父节点，`p[x]` 表示 `x` 的父节点，对于根节点，`p[x] = x`。

对于优化，可以在查找树根时将路径上的每一个节点的父节点都指向树根。

## 例题

> 一共有 $n$ 个数，编号是 $1$ ~ $n$，最开始每个数各自在一个集合中。现在要进行 $m$ 个操作，操作共有两种：
>
> 1. `M a b`，将编号为 $a$ 和 $b$ 的两个数所在的集合合并，如果两个数已经在同一个集合中，则忽略这个操作；
> 2. `Q a b`，询问编号为 $a$ 和 $b$ 的两个数是否在同一个集合中；

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1e5+10;
int p[N], n, m;

int root(int x) {
    if (p[x] != x) p[x] = root(p[x]);
    return p[x];
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) p[i] = i;
    char op[2]; int a, b;
    while (m--) {
        scanf("%s%d%d", op, &a, &b);
        if (*op == 'M') p[root(a)] = root(b);
        else puts(root(a) == root(b) ? "Yes" : "No");
    }
    return 0;
}
```

## [NOI2002] 银河英雄传说（带权并查集）

> 有一个划分为 N 列的星际战场，各列依次编号为 1,2,…,N。
>
> 有 N 艘战舰，也依次编号为 1,2,…,N，其中第 i 号战舰处于第 i 列。
>
> 有 T 条指令，每条指令格式为以下两种之一：
>
> 1. `M i j`，表示让第 i 号战舰所在列的全部战舰保持原有顺序，接在第 j 号战舰所在列的尾部。
> 2. `C i j`，表示询问第 i 号战舰与第 j 号战舰当前是否处于同一列中，如果在同一列中，它们之间间隔了多少艘战舰。
>
> 现在需要你编写一个程序，处理一系列的指令。
>
> **输入格式**
>
> 第一行包含整数 T，表示共有 T 条指令。
>
> 接下来 T 行，每行一个指令，指令有两种形式：`M i j` 或 `C i j`。
>
> 其中 M 和 C 为大写字母表示指令类型，i 和 j 为整数，表示指令涉及的战舰编号。
>
> **输出格式**
>
> 你的程序应当依次对输入的每一条指令进行分析和处理：
>
> 如果是 `M i j` 形式，则表示舰队排列发生了变化，你的程序要注意到这一点，但是不要输出任何信息；
>
> 如果是 `C i j` 形式，你的程序要输出一行，仅包含一个整数，表示在同一列上，第 i 号战舰与第 j 号战舰之间布置的战舰数目，如果第 i 号战舰与第 j 号战舰当前不在同一列上，则输出 −1。
>
> 题目链接：[AcWing 238](https://www.acwing.com/problem/content/240/)，[P1196](https://www.luogu.com.cn/problem/P1196)。洛谷后面有图解样例。

用前缀和的思想，设 $d_i$ 是 $i$ 到根节点的距离，那么两点间存在的点数应该是：
$$
d_{i,j}=\left \{\begin{aligned}
&0,i=j\\
&|d_i-d_j|-1,i\ne j
\end{aligned}\right .
$$
当合并两个集合 $A, B$ 时，设将 $B$ 合并到 $A$， 那么集合 $B$ 中根节点的距离就要更新成 $d_B=|A|$，然后它的子节点求距离的时候就可以找到正确的跳板求解了。

```cpp
#include <iostream>
#include <algorithm>
#include <cmath>
using namespace std;

const int N = 30010;
int p[N], d[N], s[N];

int find(int x) {
    if (p[x] != x) {
        int root = find(p[x]);
        d[x] += d[p[x]];
        p[x] = root;
    }
    return p[x];
}

void solve() {
    char op[2];
    int i, j;
    scanf("%s%d%d", op, &i, &j);
    int pi = find(i), pj = find(j);
    if (*op == 'M') {
        if (pi != pj) {
            p[pi] = pj;
            d[pi] = s[pj];
            s[pj] += s[pi];
        }
    } else {
        if (pi != pj) puts("-1");
        else printf("%d\n", max(abs(d[i]-d[j]) - 1, 0));
    }
}

int main() {
    for (int i = 1; i < N; i++)
        p[i] = i, s[i] = 1;
    int T; scanf("%d", &T);
    while (T--) solve();
    return 0;
}
```

## [NOI2001] 食物链

> 动物王国中有三类动物 A,B,C，这三类动物的食物链构成了有趣的环形。
>
> A 吃 B，B 吃 C，C 吃 A。
>
> 现有 N 个动物，以 1∼N 编号。
>
> 每个动物都是 A,B,C 中的一种，但是我们并不知道它到底是哪一种。
>
> 有人用两种说法对这 N 个动物所构成的食物链关系进行描述：
>
> 第一种说法是 `1 X Y`，表示 X 和 Y 是同类。
>
> 第二种说法是 `2 X Y`，表示 X 吃 Y。
>
> 此人对 N 个动物，用上述两种说法，一句接一句地说出 K 句话，这 K 句话有的是真的，有的是假的。
>
> 当一句话满足下列三条之一时，这句话就是假话，否则就是真话。
>
> 1. 当前的话与前面的某些真的话冲突，就是假话；
> 2. 当前的话中 X 或 Y 比 N 大，就是假话；
> 3. 当前的话表示 X 吃 X，就是假话。
>
> 你的任务是根据给定的 N 和 K 句话，输出假话的总数。
>
> 题目链接：[AcWing 240](https://www.acwing.com/problem/content/description/242/)，[P2024](https://www.luogu.com.cn/problem/P2024)。

结点 $u \to v$ 表示 $u$ 被 $v$ 吃，设某个点到根结点的距离为 $d$，判断 $d\bmod 3$ 的取值：

+ 余 0：当前点和根结底为同类。

+ 余 1：当前点吃根结点。

+ 余 2：当前点被根节点吃。

于是如果给的两个点 $u, v$ 已经在集合中，就可以判断出它们的关系了。

当合并两个集合使 $px \to py$ 的时候，如果 $x, y$ 是同类，那么要保证：
$$
\begin{aligned}
d_x+d_{px} &\equiv d_y(\text{mod } 3)\\
d_{px} &\equiv d_y-d_x(\text{mod } 3)\\
\end{aligned}
$$
如果 $x$ 吃 $y$ 那么要使得：
$$
\begin{aligned}
d_x+d_{px}&\equiv d_y + 1(\text{mod }3)\\
d_{px}&\equiv d_y-d_x+1
\end{aligned}
$$


```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 50010;
int n, k, p[N], d[N];
int find(int x) {
    if (x != p[x]) {
        int root = find(p[x]);
        d[x] += d[p[x]];
        p[x] = root;
    }
    return p[x];
}

int main() {
    cin >> n >> k;
    for (int i = 1; i <= n; i++) p[i] = i;
    
    int fake = 0;
    while (k--) {
        int t, x, y;
        cin >> t >> x >> y;
        if (x > n || y > n) {
            fake++;
            continue;
        }
        
        int px = find(x), py = find(y);
        if (t == 1) {
            if (px == py) {
                if ((d[x] - d[y]) % 3 != 0) fake++;
            }
            else p[px] = py, d[px] = d[y] - d[x];
        } else {
            if (px == py) {
                if ((d[x] - d[y] - 1) % 3 != 0) fake++;
            } else p[px] = py, d[px] = d[y]-d[x]+1;
        }
    }
    cout << fake << endl;
    return 0;
}
```

## 白雪皑皑

> 现在有 $n$ 片雪花排成一列。 pty 要对雪花进行 $m$ 次染色操作，第 $i$ 次染色操作中，把第 $((i\times p+q)\bmod n)+1$ 片雪花和第 $((i\times q+p)\bmod n)+1$ 片雪花之间的雪花（包括端点）染成颜色 $i$。其中 $p,q$ 是给定的两个正整数。他想知道最后 $n$ 片雪花被染成了什么颜色。没有被染色输出 $0$。
>
> **输入格式**
>
> 输入共四行，每行一个整数，分别为 $n,m,p,q$，意义如题中所述。
>
> **输出格式**
>
> 输出共 $n$ 行，每行一个整数，第 $i$ 行表示第 $i$ 片雪花的颜色。
>
> **数据范围**
>
> 对于 $100\%$ 的数据满足：$1\leq n\leq 10^6$，$1\leq m\leq 10^7$，$1\leq m\times p+q,m\times q+p\leq 2\times 10^9$

### 线段树 85pts

看到区间修改首先想到了线段树，复杂度是 $O((m+n)\log n)\approx O(2\times 10^8)$，加上常数很难过。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 1000010;

int n, m, p, q;
struct Node {
    int l, r, val;
} tr[N<<2];

void pushdown(int u) {
    if (tr[u].val) {
        tr[u<<1].val = tr[u<<1|1].val = tr[u].val;
        tr[u].val = 0;
    }
}

void modify(int u, int l, int r, int v) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u].val = v, void();
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) modify(u<<1, l, r, v);
    if (mid+1 <= r) modify(u<<1|1, l, r, v);
}

int query(int u, int p) {
    if (tr[u].l == tr[u].r) return tr[u].val;
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (p <= mid) return query(u<<1, p);
    else return query(u<<1|1, p);
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) return;
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
}

int main() {
    scanf("%d%d%d%d", &n, &m, &p, &q);

    build(1, 1, n);
    for (int i = 1; i <= m; i++) {
        int l = (1LL*i*p+q) % n + 1, r = (1LL*i*q+p) % n + 1;
        if (l > r) swap(l, r);
        modify(1, l, r, i);
    }
    for (int i = 1; i <= n; i++) {
        printf("%d\n", query(1, i));
    }
    return 0;
}
```

### 并查集 100pts

这是一道并查集维护连通性的题目，对于每个点，我们用 $fa(i)\ge i$ 表示它不能染到的第一个点，由于染色是会被覆盖的，我们只要倒着染色就达到了跳过已经操作的元素的目的。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 1000010;

int n, m, p, q;
int fa[N], color[N];

int find(int x) {
    if (x != fa[x]) fa[x] = find(fa[x]);
    return fa[x];
}

int main() {
    scanf("%d%d%d%d", &n, &m, &p, &q);

    for (int i = 1; i <= n+1; i++) fa[i] = i;

    for (int i = m; i; i--) {
        int l = (i*p+q) % n + 1, r = (i*q+p) % n + 1;
        if (l > r) swap(l, r);
        for (int j = find(l); j <= r; j = fa[j]) {
            color[j] = i;
            fa[j] = find(j+1);
        }
    }

    for (int i = 1; i <= n; i++) {
        printf("%d\n", color[i]);
    }
    return 0;
}
```

## [CEOI1999] Parity Game

> Alice 和 Bob 在玩一个游戏：他写一个由 $0$ 和 $1$ 组成的序列。Alice 选其中的一段（比如第 $3$ 位到第 $5$ 位），问他这段里面有奇数个 $1$ 还是偶数个 $1$。Bob 回答你的问题，然后 Alice 继续问。Alice 要检查 Bob 的答案，指出在 Bob 的第几个回答一定有问题。有问题的意思就是存在一个 $01$ 序列满足这个回答前的所有回答，而且不存在序列满足这个回答前的所有回答及这个回答。
>
> **输入格式**
>
> 第 $1$ 行一个整数 $n$，是这个 $01$ 序列的长度。
>
> 第 $2$ 行一个整数 $m$，是问题和答案的个数。
>
> 第 $3$ 行开始是问题和答案，每行先有两个整数，表示你询问的段的开始位置和结束位置。然后是 Bob 的回答。`odd`表示有奇数个 $1$，`even` 表示有偶数个 $1$。
>
> **输出格式**
>
> 输出一行，一个数 $x$，表示存在一个 $01$ 序列满足第 $1$ 到第 $x$ 个回答，但是不存在序列满足第 $1$ 到第 $x+1$ 个回答。如果所有回答都没问题，你就输出所有回答的个数。
>
> 对于 $100\%$ 的数据，$1 \le  n \leq 10^9$，$m \leq 5 \times 10^3$。
>
> 题目链接：[P5937](https://www.luogu.com.cn/problem/P5937)。

首先发现序列长度远大于读入量，需要离散化。

然后我们可以通过前缀和的思想，每次读入一个 $L,R$ 相当于知道了 $S_R-S_{L-1}$ 的奇偶性，也就是知道了 $S_R$ 和 $S_{L-1}$ 的奇偶性是否相同。这个用带权并查集维护即可，两个点如果不连通就连通起来，否则判断它们到根的距离是否模二同余。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 10010, M = 5010;
vector<int> nums;
int n, m, p[N], d[N];
struct Query {
    int l, r, odd;
} qry[M];

int find(int x) {
    if (p[x] != x) {
        int rt = find(p[x]);
        d[x] += d[p[x]];
        p[x] = rt;
    }
    return p[x];
}

int findval(int x) {
    return lower_bound(nums.begin(), nums.end(), x) - nums.begin() + 1;
}

int main() {
    scanf("%d%d", &n, &m);

    char buf[5];
    for (int i = 0, l, r; i < m; i++) {
        scanf("%d%d%s", &l, &r, buf);
        nums.emplace_back(l);
        nums.emplace_back(r);
        qry[i] = {l, r, buf[0] == 'o'};
    }

    sort(nums.begin(), nums.end());
    nums.erase(unique(nums.begin(), nums.end()), nums.end());
    n = nums.size();
    for (int i = 1; i <= n; i++) p[i] = i;

    for (int i = 0; i < m; i++) {
        int l = findval(qry[i].l)-1, r = findval(qry[i].r);
        int pl = find(l), pr = find(r);
        if (pl != pr) {
            p[pl] = pr;
            d[pl] = qry[i].odd + d[r] - d[l];
        }
        else if (((d[l] - d[r]) & 1) != qry[i].odd) {
            return !printf("%d\n", i);
        }
    }
    return !printf("%d\n", m);
}
```

## [HAOI2006] 旅行

> Z 小镇是一个景色宜人的地方，吸引来自各地的观光客来此旅游观光。Z 小镇附近共有 $n$ 个景点（编号为 $1,2,3,\ldots,n$），这些景点被 $m$ 条道路连接着，所有道路都是双向的，两个景点之间可能有多条道路。
>
> 也许是为了保护该地的旅游资源，Z 小镇有个奇怪的规定，就是对于一条给定的公路 $r_i$，任何在该公路上行驶的车辆速度必须为 $v_i$。
>
> 速度变化太快使得游客们很不舒服，因此从一个景点前往另一个景点的时候，大家都希望选择行使过程中最大速度和最小速度的比尽可能小的路线，也就是所谓最舒适的路线。
>
> **输入格式**
>
> 第一行包含两个正整数 $n,m$。
>
> 接下来的 $m$ 行每行包含三个正整数 $x,y,v$。表示景点 $x$ 到景点 $y$ 之间有一条双向公路，车辆必须以速度 $v$ 在该公路上行驶。
>
> 最后一行包含两个正整数 $s,t$，表示想知道从景点 $s$ 到景点 $t$ 最大最小速度比最小的路径。$s$ 和 $t$ 不可能相同。
>
> **输出格**
>
> 如果景点 $s$ 到景点 $t$ 没有路径，输出` IMPOSSIBLE`。否则输出一个数，表示最小的速度比。如果需要，输出一个既约分数。
>
> **数据范围**
>
> 对于 $100\%$ 的数据，$1 \le x,y \le n \le 500$，$1 \le v < 3 \times 10^4$，$1 \le m \le 5 \times 10^3$，$x \ne y$。
>
> 题目链接：[P2502](https://www.luogu.com.cn/problem/P2502)。

看到 $m$ 比较小，所以想法是首先枚举固定一个最大值/最小值，这里选择固定最小值。

然后，想看看最大值到什么时候能使得 $s,t$ 连通。

所以，先排序，然后枚举最小边，然后不断增量地找右边界，一旦找到了就记录一下答案，然后结束循环，剪纸。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 510, M = 10010, INF = 1000;
int n, m, S, T, idx = 2;
int p[N];

int gcd(int a, int b) {
    if (!b) return a;
    return gcd(b, a % b);
}

struct Edge {
    int a, b, c;

    bool operator<(const Edge& ed) const {
        return c < ed.c;
    }
} e[M];

struct Fraction {
    int a, b;

    bool operator<(const Fraction& f) const {
        return f.b * a < f.a * b;
    }

    void minimal() {
        int d = gcd(a, b);
        a /= d, b /= d;
    }
};

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1, a, b, c; i <= m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        e[i] = {a, b, c};
    }
    scanf("%d%d", &S, &T);
    sort(e+1, e+1+m);

    Fraction ans = {INF, 1};
    for (int i = 1; i <= m; i++) {
        for (int i = 1; i <= n; i++) p[i] = i;

        for (int j = i; j <= m; j++) {
            int a = e[j].a, b = e[j].b;
            p[find(a)] = find(b);
            if (find(S) == find(T)) {
                ans = min(ans, Fraction{e[j].c, e[i].c});
                break;
            }
        }
    }

    ans.minimal();
    if (ans.a == INF) puts("IMPOSSIBLE");
    else {
        if (ans.b == 1) printf("%d\n", ans.a);
        else printf("%d/%d\n", ans.a, ans.b);
    }
    return 0;
}
```

## 染色

> 给定 $n$ 点的树和 $q$ 条路径，需要保证同一路径上的颜色相同，只能染成黑或白，求黑白点数颜色差的所有可能，$n,q\le 3\times 10^5$。

本题有两个部分，一个部分是处理出所有连通块的大小，另一部分是这些大小能够凑出的所有方案。

我在赛时对于第一部分使用了树剖，每次对一个路径上的点权用它们的 $LCA$ 取 $\max$，然而这个做法是错的，因为每次修改一条路径如果影响到了别的路径，是无法通过一个正确的复杂度去全部修改点权。

所以正解是使用并查集，每个点在并查集中的父亲代表它所在路径中最上层的 $LCA$。

对于第二部分，形式化的说，就是已知 $\sum a_i =n$ 求 $a_i$ 能凑出什么数字，显然是 01 背包问题。对于前一个条件，我们可以发现最多只存在 $O(\sqrt n)$ 个不同的 $a_i$，分两种情况考虑：

1. 当 $a_i\le\sqrt n$ 时，显然只有 $\sqrt n$ 种。
2. 当 $a_i\gt\sqrt n$时，如果超过了 $\sqrt n$ 种，那么最终的总和就会大于 $n$。

综上，只有 $O(\sqrt n)$ 个不同的数字，所以我们先转化成多重背包问题，此时复杂度硬做还是 $O(n\times \sqrt n\times \sqrt n)=O(n^2)$，但是我们可以通过二进制拆分再次转化成 01 背包问题，会从 $O(\sqrt n)$ 个物品拆出 $O(\sqrt n \log n)$ 个物品，所以此时复杂度降到了 $O(n\sqrt n\log n)$，用 `bitset` 加速可以再来一个小常数。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 300010, K = 20;
int n, q;
int fa[N][K], h[N], dep[N], p[N], sz[N], idx = 2;
int cnt[N];
bool ans[N];

struct Edge {
    int to, nxt;
} e[N<<1];

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

void merge(int a, int b) {
    int pa = find(a), pb = find(b);
    if (pa == pb) return;
    sz[pb] += sz[pa];
    p[pa] = pb;
}

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void dfs(int u, int fath) {
    fa[u][0] = fath, dep[u] = dep[fath] + 1;
    for (int k = 1; k < K; k++) fa[u][k] = fa[fa[u][k-1]][k-1];

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fath) continue;
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

signed main() {
    cin >> n >> q;
    for (int i = 1, a, b; i < n; i++) {
        cin >> a >> b;
        add(a, b), add(b, a);
    }
    dfs(1, 0);

    for (int i = 1; i <= n; i++) p[i] = i, sz[i] = 1;
    

    for (int i = 1, a, b; i <= q; i++) {
        cin >> a >> b;
        int u = lca(a, b);
        while (dep[a] > dep[u]) merge(a, fa[a][0]), a = find(a);
        while (dep[b] > dep[u]) merge(b, fa[b][0]), b = find(b);
    }

    for (int i = 1; i <= n; i++)
        if (p[i] == i) cnt[sz[i]]++;

    bitset<N> f;
    f[0] = 1;
    for (int val = 1; val <= n; val++) {
        int x = 1;
        while (cnt[val] - x >= 0) {
            f |= f << (val * x);
            cnt[val] -= x;
            x <<= 1;
        }

        if (cnt[val]) f |= f << (cnt[val] * val);
    }
    
    for (int i = 0; i <= n; i++)
        if (f[i]) ans[abs(i-(n-i))] = true;
    
    for (int i = 0; i <= n; i++)
        if (ans[i]) cout << i << ' ';

    cout << '\n';
    return 0;
}
```

## 加边

> 给定 $n$ 个点 $m$ 条的带权无向图，给定序列 $a$，表示花费 $a_i+a_j$ 可以将边 $(i,j)$ 的权修改为正无穷，$q$ 个询问，每个询问代表将所有 $\ge x$ 的边连起来后，最小花多少代价可以将整个图连通。

不难发现如果保证 $x$ 从大到小排列，那么每过一个询问都可以加入新的边，并且维护这些连通块，所以想到离线询问加上并查集维护。

我们要最小化连通块之间的 $\sum a_i+a_j$，如果固定 $a_i$，那么就变成了 $(cnt-1)a_i + (\sum a_j) - a_i$，我们维护所有连通块的最小值 $a_i$，然后维护所有连通块最小值之和 $S=\sum a_j$，每个询问的答案就是 $(cnt-2)a_i + S$。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 300010;
int p[N], n, m, q, a[N], ans[N];
int mn[N], cnt, sum;

struct Edge {
	int a, b, c;
	bool operator<(const Edge& ed) const {
		return c > ed.c;
	}
} edge[N];

struct Query {
	int id, x;
	bool operator<(const Query& q) const {
		return x > q.x;
	}
} qry[N];

int find(int x) {
	if (x != p[x]) p[x] = find(p[x]);
	return p[x];
}

signed main() {
	freopen("add.in", "r", stdin);
	freopen("add.out", "w", stdout);
	cin >> n >> m >> q;
	for (int i = 1; i <= n; i++) cin >> a[i];
	
	int MIN = 0x3f3f3f3f;
	for (int i = 1; i <= n; i++) p[i] = i, mn[i] = a[i], sum += a[i], MIN = min(MIN, a[i]);
	cnt = n;

	for (int i = 1; i <= m; i++) {
		cin >> edge[i].a >> edge[i].b >> edge[i].c;
	}
	
	for (int i = 1, x; i <= q; i++) {
		cin >> x;
		qry[i].id = i, qry[i].x = x;
	}
	
	sort(edge+1, edge+1+m);
	int tp = 1;
	sort(qry+1, qry+1+q);
	for (int i = 1; i <= q; i++) {
		while (tp <= m && edge[tp].c >= qry[i].x) {
			int a = edge[tp].a, b = edge[tp].b;
			int pa = find(a), pb = find(b);
			if (pa != pb) {
				sum -= max(mn[pb], mn[pa]);
				p[pa] = pb;
				mn[pb] = min(mn[pb], mn[pa]);
				cnt--;
			}
			tp++;
		}
		
		ans[qry[i].id] = MIN * (cnt-2) + sum;
	}
	
	for (int i = 1; i <= q; i++) cout << ans[i] << '\n';
	
	return 0;
}
```
