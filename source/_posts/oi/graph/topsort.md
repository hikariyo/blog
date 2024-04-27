---
date: 2023-09-18 07:15:00
title: 图论 - 拓扑排序
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 模板

> 给定一个 n 个点 m 条边的有向图，点的编号是 1 到 n，图中可能存在重边和自环。
>
> 请输出任意一个该有向图的拓扑序列，如果拓扑序列不存在，则输出 −1。
>
> 若一个由图中所有点构成的序列 A 满足：对于图中的每条边 (x,y)，x 在 A 中都出现在 y 之前，则称 A 是该图的一个拓扑序列。
>
> 题目链接：[AcWing 848](https://www.acwing.com/problem/content/850/)。

本题就是要按照入度由小到大排序。

```cpp
#include <cstring>
#include <iostream>
using namespace std;

const int N = 1e5+10;
int h[N], e[N], ne[N], idx, d[N];
int n, m;
int q[N];

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
    // 入度
    d[b]++;
}

bool topsort() {
    int tt = -1, hh = 0;
    for (int i = 1; i <= n; i++) {
        // 这里是 d[i] 代表第 i 节点的入度 所以从 1 ~ n 而不是从 0 开始
        if (d[i] == 0) q[++tt] = i;
    }
    while (hh <= tt) {
        int t = q[hh++];
        for (int i = h[t]; i  != -1; i = ne[i]) {
            int j = e[i];
            d[j]--;
            // 添加到队列中的是树的节点
            if (d[j] == 0) q[++tt] = j;
        }
    }
    // 如果所有节点都被遍历过，那么拓扑序列是存在的
    return tt == n-1;
}

int main() {
    memset(h, -1, sizeof h);
    cin >> n >> m;
    int a, b;
    for (int i = 0; i < m; i++) {
        cin >> a >> b;
        add(a, b);
    }
    if (topsort()) {
        for (int i = 0; i < n; i++) cout << q[i] << ' ';
    } else puts("-1");
}
```

## [CSP-S2020] 函数调用

> 某数据库应用程序提供了若干函数用以维护数据。已知这些函数的功能可分为三类：
>
> 1. 将数据中的指定元素加上一个值；
> 2. 将数据中的每一个元素乘以一个相同值；
> 3. **依次**执行若干次函数调用，保证不会出现递归（即不会直接或间接地调用本身）。
>
> **输入格式**
>
> 第一行一个正整数 $n$，表示数据的个数。  
>
> 第二行 $n$ 个整数，第 $i$ 个整数表示下标为 $i$ 的数据的初始值为 $a_i$。  
>
> 第三行一个正整数 $m$，表示数据库应用程序提供的函数个数。函数从 $1 \sim m$ 编号。  
>
> 接下来 $m$ 行中，第 $j$（$1 \le j \le m$）行的第一个整数为 $T_j$，表示 $j$ 号函数的类型：
>
> 1. 若 $T_j = 1$，接下来两个整数 $P_j, V_j$ 分别表示要执行加法的元素的下标及其增加的值；
> 2. 若 $T_j = 2$，接下来一个整数 $V_j$ 表示所有元素所乘的值；
> 3. 若 $T_j = 3$，接下来一个正整数 $C_j$ 表示 $j$ 号函数要调用的函数个数，随后 $C_j$ 个整数 $g^{(j)}_1, g^{(j)}_2, \ldots , g^{(j)}_{C_j}$ 依次表示其所调用的函数的编号。
>
> 第 $m + 4$ 行一个正整数 $Q$，表示输入的函数操作序列长度。  
>
> 第 $m + 5$ 行 $Q$ 个整数 $f_i$，第 $i$ 个整数表示第 $i$ 个执行的函数的编号。
>
> **输出格式**
>
> 一行 $n$ 个用空格隔开的整数，按照下标 $1 \sim n$ 的顺序，分别输出在执行完输入的函数序列后，数据库中每一个元素的值。**答案对** $\boldsymbol{998244353}$ **取模。**
>
> 对于所有数据：$0 \le a_i \le 10^4$，$T_j \in \{1,2,3\}$，$1 \le P_j \le n$，$0 \le V_j \le 10^4$，$1 \le g^{(j)}_k \le m$，$1 \le f_i \le m$。

首先可以把所有初始化操作看作一个 $a_i+v$ 的一号函数，然后把这些初始化与调用序列共同分配给超级节点 $0$，最后我们只要调用 $0$ 号函数就行了。

如果没有三号函数，对于下面这个操作序列：
$$
(a_1+2)(\times 3)(a_2+5)(\times 2)
$$
假设所有初始值 $a_i=0$，相当于 $a_1+2\times3\times 2,a_2+5\times 2$，也就是**后面的乘会影响前面的加**。这让我们想到倒序处理每个序列，并且记录下这一路上都乘了多少，更新 $add$ 值。特别地，三号函数的 $mul$ 是它所有子结点的 $mul$ 之积，这也符合要求。

如果你只这么做，就会获得 10pts 的成绩，原因是一个函数会在不同的地方被调用多次，而我们并没有考虑如何正确记录一个函数被调用的次数。

显然，假设一个函数的父结点（也就是调用它的函数）被调用的次数为 $p$，这个函数在这个序列中经过乘法等价调用的次数为 $q$，那么最终调用的次数就是 $pq$，然后在多个地方都这样考虑，把结果累加起来。

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 200010, M = 1200010, P = 998244353;
int topo[N];
int h[N], e[M], ne[M], cnt[N], add[N], mul[N], type[N], din[N], p[N], idx;
int val[N/2];
int n, m, q;

void addedge(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
    din[b]++;
}

void topsort() {
    int hh = 0, tt = -1;
    for (int i = 0; i <= n+m; i++) {
        if (din[i] == 0) topo[++tt] = i;
    }

    while (hh <= tt) {
        int t = topo[hh++];
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (--din[j] == 0) topo[++tt] = j;
        }
    }
}

void pushup() {
    for (int i = n+m; ~i; i--) {
        int u = topo[i];
        for (int i = h[u]; ~i; i = ne[i]) {
            int j = e[i];
            mul[u] = (LL)mul[u] * mul[j] % P;
        }
    }
}

void pushdown() {
    for (int i = 0; i <= n+m; i++) {
        LL prod = 1;
        int u = topo[i];
        for (int i = h[u]; ~i; i = ne[i]) {
            int j = e[i];
            cnt[j] = (cnt[j] + prod * cnt[u]) % P;
            prod = prod * mul[j] % P;
        }
    }
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d", &n);
    type[0] = 3;
    for (int i = 1; i <= n; i++) {
        type[i] = 1, p[i] = i;
        scanf("%d", &add[i]);
        addedge(0, i);
    }

    scanf("%d", &m);
    fill(mul, mul+n+m+1, 1);
    for (int i = 1; i <= m; i++) {
        int u = i+n;
        scanf("%d", &type[u]);
        if (type[u] == 1) scanf("%d%d", &p[u], &add[u]);
        if (type[u] == 2) scanf("%d", &mul[u]);
        if (type[u] == 3) {
            int c, f;
            scanf("%d", &c);
            while (c--) {
                scanf("%d", &f);
                addedge(u, f+n);
            }
        }
    }

    scanf("%d", &q);
    for (int i = 1, f; i <= q; i++) {
        scanf("%d", &f);
        addedge(0, f+n);
    }

    cnt[0] = 1;
    topsort();
    pushup();
    pushdown();

    for (int i = 0; i <= n+m; i++)
        if (type[i] == 1)
            val[p[i]] = (val[p[i]] + (LL)add[i] * cnt[i]) % P;

    for (int i = 1; i <= n; i++) printf("%d ", val[i]);
    puts("");

    return 0;
}
```

## [NOIP2013] 车站分级

> 一条单向的铁路线上，依次有编号为 1,2,…,n 个火车站。
>
> 每个火车站都有一个级别，最低为 1 级。
>
> 现有若干趟车次在这条线路上行驶，每一趟都满足如下要求：如果这趟车次停靠了火车站 x，则始发站、终点站之间所有级别大于等于火车站 x 的都必须停靠。（注意：起始站和终点站自然也算作事先已知需要停靠的站点） 。
>
> 现有 m 趟车次的运行情况（全部满足要求），试推算这 n 个火车站至少分为几个不同的级别。
>
> 题目保证一定有解。
>
> 题目链接：[AcWing 456](https://www.acwing.com/problem/content/458/)，[P1983](https://www.luogu.com.cn/problem/P1983)。

给出的暗示很明显了，始发站、终点站之间的所有停靠的车站等级 $l_s$ 必须严格大于没停靠的车站 $l_{e}$，这可以用反证法说明：如果 $l_e \ge l_s$ 那么它也必须停靠，构成矛盾，因此我们的结论是正确的。

所以本题差分约束的不等式组就是：
$$
\forall s,e\in[start,stop], l_s\ge l_e+1
$$
如果 $m$ 趟列车都从 $1 \to n$ 行驶，中间停靠了 $x$ 站，那么总共建的边数就是：
$$
mx(n-x)\le \frac{mn^2}{4}\le\frac{10^9}{4}
$$
这显然会爆内存的，就算不爆内存循环这么多次也要 TLE，因此这里再次用到虚拟结点 $v$，让 $l_s\ge l_v \ge l_e+1$，这样边的数量就是：
$$
m[x+(n-x)]=mn\le 10^6
$$
这就符合要求了，如果题目保证有解，就不会出现 $l_i \gt l_i$ 这种情况，也就是说这个图一定是 DAG，所以本题拓扑排序是为了求一个 DAG 的最短路。

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 2010, M = 1000010;
int n, m;
int h[N], e[M], ne[M], w[M], din[N], dist[N], idx;
bool st[N];
int q[N];

int read() {
    int x = 0;
    char ch = getchar();
    while (ch < '0' || ch > '9') ch = getchar();
    while ('0' <= ch && ch <= '9') {
        x = x*10 + (ch & 15);
        ch = getchar();
    }
    return x;
}

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
    din[b]++;
}

void topsort() {
    int hh = 0, tt = -1;
    for (int i = 1; i <= m+n; i++) {
        if (din[i] == 0) q[++tt] = i, dist[i] = 1;
    }
    
    while (hh <= tt) {
        int t = q[hh++];
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (--din[j] == 0) q[++tt] = j;
        }
    }
}

int solve() {
    for (int i = 0; i < m+n; i++) {
        int t = q[i];
        for (int j = h[t]; ~j; j = ne[j]) {
            int k = e[j];
            dist[k] = max(dist[k], dist[t] + w[j]);
        }
    }
    
    return *max_element(dist+1, dist+1+n);
}

int main() {
    n = read(), m = read();
    memset(h, -1, sizeof h);
    for (int i = 1; i <= m; i++) {
        memset(st, 0, sizeof st);
        int cnt = read();
        int start = n, end = 0;
        while (cnt--) {
            int x = read();
            st[x] = true;
            start = min(start, x), end = max(end, x);
        }
        
        int v = n+i;
        for (int j = start; j <= end; j++)
            if (st[j]) add(v, j, 0);
            else add(j, v, 1);
    }
    topsort();
    printf("%d\n", solve());
    return 0;
}
```

## [COCI2016-2017#1] Cezar

> Mirko 想对 $n$ 个单词进行加密。加密过程是这样的：
>
> 1.  选择一个英文字母表的排列作为密钥。
> 2.  将单词中的 `a` 替换为密钥中的第一个字母，`b` 替换为密钥中的第二个字母……以此类推。
>
> 例如，以 `qwertyuiopasdfghjklzxcvbnm` 作为密钥对 `cezar` 加密后，将得到 `etmqk`。
>
> 他希望，将所有单词加密并按字典序升序排列后，最初的第 $a_i$ 个单词位于第 $i$ 位。请你判断，这能否实现。如果能，请给出任意一种可行的密钥。
>
> **输入格式**
>
> 第一行一个整数 $n$。
>
> 接下来 $n$ 行，每行一个字符串，表示待加密的单词。
>
> 最后一行 $n$ 个整数，表示 $a_i$。
>
> **输出格式**
>
> 如果 Mirko 的要求不能实现，输出 `NE`。
>
> 否则，输出 `DA`。接下来一行输出任意一种可行的密钥。
>
> **数据规模与约定**
>
> 对于 $100\%$ 的数据，$2\le n\le 100$，$1 \leq a_i \leq n$。
>
> 所有单词的长度不超过 $100$，且只包含小写字母。
>
> 题目链接：[P6286](https://www.luogu.com.cn/problem/P6286)。

首先他这个 $a$ 数组就是纯纯恶心人的，我们可以在读入的时候调整位置，直接把第 $i$ 个位置填上第 $a_i$ 的字符串。

现在问题就是是否存在一个合法解，使得字符串经过变换后是符合字典序升序排列的。

对于第 $i$ 个字符串，也就是让它的字典序小于所有 $s_j(j>i)$，其实就是对于所有 $s_i,s_j$ 第一个不相同的字符前面比后面小，只要在这两个字符连边即可。

特殊情况是如果两个字符串相等，它们必须排在一块，否则无解；如果一个字符串是另一个字符串的前缀，此时字典序的相对大小是确定的，长度更长的那个字典序更大，所以此时需要保证 $len(s_j)\ge len(s_i)$ 否则无解。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 110;
int n, din[N], q[N];
char s[N][N], *p[N];
vector<int> g[N];

bool topsort() {
    int hh = 0, tt = -1;
    for (int i = 0; i < 26; i++) if (!din[i]) q[++tt] = i;
    while (hh <= tt) {
        int t = q[hh++];
        for (int v: g[t])
            if (--din[v] == 0)
                q[++tt] = v;
    }
    return tt == 26-1;
}

int diff(const char* a, const char* b) {
    int p = strlen(a), q = strlen(b);
    for (int i = 0; i < max(p, q); i++) {
        if (a[i] != b[i]) return i;
    }
    return -1;
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%s", s[i]);
    for (int i = 1, a; i <= n; i++) scanf("%d", &a), p[i] = s[a];

    for (int i = 1; i <= n; i++) {
        int presame = i;
        for (int j = i+1; j <= n; j++) {
            int k = diff(p[i], p[j]);
            if (k == -1) {
                if (j != presame + 1) puts("NE"), exit(0);
                presame = j;
                continue;
            }
            if (!p[i][k] || !p[j][k]) {
                if (strlen(p[i]) > strlen(p[j])) puts("NE"), exit(0);
                continue;
            }
            g[p[i][k]-'a'].emplace_back(p[j][k]-'a');
            din[p[j][k]-'a']++;
        }
    }

    if (!topsort()) return puts("NE"), 0;

    static char res[30];
    for (int i = 0; i < 26; i++)
        res[q[i]] = 'a' + i;
    return !printf("DA\n%s\n", res);
}
```

## [HNOI2015] 菜肴制作

> 知名美食家小 A 被邀请至 ATM 大酒店，为其品评菜肴。ATM 酒店为小 A 准备了 $n$ 道菜肴，酒店按照为菜肴预估的质量从高到低给予 $1$ 到 $n$ 的顺序编号，预估质量最高的菜肴编号为 $1$。
>
> 由于菜肴之间口味搭配的问题，某些菜肴必须在另一些菜肴之前制作，具体的，一共有 $m$ 条形如 $i$ 号菜肴必须先于 $j$ 号菜肴制作的限制，我们将这样的限制简写为 $(i,j)$。
>
> 现在，酒店希望能求出一个最优的菜肴的制作顺序，使得小 A 能尽量先吃到质量高的菜肴：
>
> 也就是说，
>
> 1. 在满足所有限制的前提下，$1$ 号菜肴尽量优先制作。
>
> 2. 在满足所有限制，$1$ 号菜肴尽量优先制作的前提下，$2$ 号菜肴尽量优先制作。
>
> 3. 在满足所有限制，$1$ 号和 $2$ 号菜肴尽量优先的前提下，$3$ 号菜肴尽量优先制作。
>
> 4. 在满足所有限制，$1$ 号和 $2$ 号和 $3$ 号菜肴尽量优先的前提下，$4$ 号菜肴尽量优先制作。
>
> 5. 以此类推。
>
> 例 1：共 $4$ 道菜肴，两条限制 $(3,1)$、$(4,1)$，那么制作顺序是 $3,4,1,2$。
>
> 例 2：共 $5$ 道菜肴，两条限制 $(5,2)$、$(4,3)$，那么制作顺序是 $1,5,2,4,3$。
>
> 例 1 里，首先考虑 $1$，因为有限制 $(3,1)$ 和 $(4,1)$，所以只有制作完 $3$ 和 $4$ 后才能制作 $1$，而根据 3，$3$ 号又应尽量比 $4$ 号优先，所以当前可确定前三道菜的制作顺序是 $3,4,1$；接下来考虑 $2$，确定最终的制作顺序是 $3,4,1,2$。
>
> 例 $2$ 里，首先制作 $1$ 是不违背限制的；接下来考虑 $2$ 时有 $(5,2)$ 的限制，所以接下来先制作 $5$ 再制作 $2$；接下来考虑 $3$ 时有 $(4,3)$ 的限制，所以接下来先制作 $4$ 再制作 $3$，从而最终的顺序是 $1,5,2,4,3$。现在你需要求出这个最优的菜肴制作顺序。无解输出 `Impossible!`（首字母大写，其余字母小写）
>
> **输入格式**
>
> 第一行是一个正整数 $t$，表示数据组数。接下来是 $t$ 组数据。对于每组数据：第一行两个用空格分开的正整数 $n$ 和 $m$，分别表示菜肴数目和制作顺序限制的条目数。接下来 $m$ 行，每行两个正整数 $x,y$，表示 $x$ 号菜肴必须先于 $y$ 号菜肴制作的限制。
>
> **输出格式**
>
> 输出文件仅包含 $t$ 行，每行 $n$ 个整数，表示最优的菜肴制作顺序，或者 `Impossible!` 表示无解。
>
> $100\%$ 的数据满足 $n,m\le 10^5$，$1\le t\le 3$。
>
> $m$ 条限制中可能存在完全相同的限制。
>
> 题目链接：[P3243](https://www.luogu.com.cn/problem/P3243)。

在满足拓扑序合法的情况下，我们要尽量使得更小的 $i$ 满足 $i$ 排在 $i+1$ 前面。

首先，这并不等同于最小字典序，比如第三组样例：

```text
5 2
5 2
4 3
```

如果每次都是最小字典序，结果就是 `1 4 3 5 2`，而答案是 `1 5 2 4 3`。如果反过来看，越大的越要向后放，是反图的最大字典序。这个证明我也不会，摆烂。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;
int n, m, ans[N], din[N];
vector<int> g[N];

bool topsort() {
    priority_queue<int> heap;

    int cnt = 0;
    for (int i = 1; i <= n; i++) if (!din[i]) heap.push(i);
    while (heap.size()) {
        int t = heap.top(); heap.pop();
        ans[cnt++] = t;
        for (int v: g[t]) {
            if (--din[v] == 0) heap.push(v);
        }
    }
    return cnt == n;
}

void solve() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) din[i] = 0, vector<int>().swap(g[i]);
    for (int i = 0, a, b; i < m; i++) {
        scanf("%d%d", &a, &b);
        g[b].emplace_back(a);
        din[a]++;
    }
    if (!topsort()) return puts("Impossible!"), void();
    for (int i = n-1; i >= 0; i--) printf("%d ", ans[i]);
    puts("");
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) solve();
    return 0;
}
```

