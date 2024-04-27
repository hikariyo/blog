---
date: 2023-09-13 18:39:00
title: 数据结构 - 李超线段树
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
---

## [HEOI2013] Segment

> 要求在平面直角坐标系下维护两个操作：
>
> 1. 在平面上加入一条线段。记第 $i$ 条被插入的线段的标号为 $i$。
> 2. 给定一个数 $k$，询问与直线 $x = k$ 相交的线段中，交点纵坐标最大的线段的编号。
>
> **本题输入强制在线**。
>
> 输入的第一行是一个整数 $n$，代表操作的个数。
>
> 接下来 $n$ 行，每行若干个用空格隔开的整数，第 $i + 1$ 行的第一个整数为 $op$，代表第 $i$ 次操作的类型。
>
> 若 $op = 0$，则后跟一个整数 $k$，代表本次操作为查询所所有与直线 $x = (k + lastans - 1) \bmod 39989 + 1$ 相交的线段中，交点纵坐标最大的线段编号。
>
> 若 $op = 1$，则后跟四个整数 $x_0, y_0, x_1, y_1$，记 $x_i' = (x_i + lastans - 1) \bmod 39989 + 1$，$y_i' = (y_i + lastans - 1) \bmod 10^9 + 1$。本次操作为插入一条两端点分别为 $(x_0', y_0')$，$(x_1',y_1')$ 的线段。
>
> 其中 $lastans$ 为上次询问的答案，初始时，$lastans = 0$。
>
> 对于 $100\%$ 的数据，保证 $1 \leq n \leq 10^5$，$1 \leq k, x_0, x_1 \leq 39989$，$1 \leq y_0, y_1 \leq 10^9$。
>
> 题目链接：[P4097](https://www.luogu.com.cn/problem/P4097)。

李超线段树是没有 `pushdown` 的，每次查询时考虑上每一个点的标记，这也叫做标记永久化。

```cpp
#include <cstdio>
#include <cstring>
#include <cassert>
#include <algorithm>
#include <cmath>
using namespace std;

const int N = 100010, M = 40010;
const double eps = 1e-6;

struct Seg {
    double k, b;
    
    void init(int& x1, int& y1, int& x2, int& y2) {
        if (x1 > x2) swap(x1, x2), swap(y1, y2);
        if (x1 == x2) b = max(y1, y2), k = 0;
        else {
            k = (double)(y1 - y2) / (x1 - x2);
            b = y1 - k*x1;
        }
    }
} seg[N];
int n, idx;

struct Node {
    int l, r;
    int segid;
} tr[M<<2];

int compf(double x) {
    if (fabs(x) < eps) return 0;
    if (x < 0) return -1;
    return 1;
}

double apply(int id, int x) {
    return seg[id].k * x + seg[id].b;
}

struct Comp {
    int x;
    Comp(int x): x(x) {}

    bool operator()(int id1, int id2) const {
        double f1 = apply(id1, x), f2 = apply(id2, x);
        int res = compf(f1 - f2);
        if (res == -1) return true;
        else if (res == 1) return false;
        // 如果编号大就不选
        return id1 > id2;
    }
};

void modify(int u, int l, int r, int id) {
    // debug("Modifing %d which is [%d %d]\n", u, tr[u].l, tr[u].r);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= tr[u].l && tr[u].r <= r) {
        if (!tr[u].segid) return tr[u].segid = id, void();
        if (Comp(mid)(tr[u].segid, id)) swap(tr[u].segid, id);

        if (tr[u].l == tr[u].r) return;

        if (Comp(l)(tr[u].segid, id)) modify(u<<1, l, r, id);
        if (Comp(r)(tr[u].segid, id)) modify(u<<1|1, l, r, id);
        return;
    }

    if (l <= mid) modify(u<<1, l, r, id);
    if (mid+1 <= r) modify(u<<1|1, l, r, id);
}

int query(int u, int l, int r, int x) {
    int res = tr[u].segid;
    if (tr[u].l == x && tr[u].r == x) return res;

    int mid = (tr[u].l + tr[u].r) >> 1;
    if (x <= mid) return max(res, query(u<<1, l, r, x), Comp(x));
    else return max(res, query(u<<1|1, l, r, x), Comp(x));
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) return;
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
}

int main() {
    int lastans = 0, op, x, x1, x2, y1, y2;
    build(1, 1, 40000);
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &op);
        
        if (op == 0) {
            scanf("%d", &x);
            x = (x + lastans - 1) % 39989 + 1;
            printf("%d\n", lastans = query(1, 1, 40000, x));
        }
        else {
            scanf("%d%d%d%d", &x1, &y1, &x2, &y2);
            x1 = (x1 + lastans - 1) % 39989 + 1, x2 = (x2 + lastans - 1) % 39989 + 1;
            y1 = (y1 + lastans - 1) % 1000000000 + 1, y2 = (y2 + lastans - 1) % 1000000000 + 1;
            seg[++idx].init(x1, y1, x2, y2);
            modify(1, x1, x2, idx);
        }
    }
    return 0;
}
```

## [JSOI2008] Blue Mary 开公司

> 对于一个金融顾问 $i$，他设计的经营方案中，每天的收益都比前一天高，并且均增长一个相同的量 $P_i$。
>
> 由于金融顾问的工作效率不高，**所以在特定的时间，Blue Mary 只能根据他已经得到的经营方案来估算某一时间的最大收益**。由于 Blue Mary 是很没有经济头脑的，所以他在估算每天的最佳获益时完全不会考虑之前的情况，而是直接从所有金融顾问的方案中选择一个在当天获益最大的方案的当天的获益值，例如：
>
> 有如下两个金融顾问分别对前四天的收益方案做了设计：
>
> |        | 第一天 | 第二天 | 第三天 | 第四天 | $P_i$ |
> | :----: | :----: | :----: | :----: | :----: | :---: |
> | 顾问 1 |  $1$   |  $5$   |  $9$   |  $13$  |  $4$  |
> | 顾问 2 |  $2$   |  $5$   |  $8$   |  $11$  |  $3$  |
>
> 在第一天，Blue Mary 认为最大收益是 $2$（使用顾问 2 的方案），而在第三天和第四天，他认为最大收益分别是 $9$ 和 $13$（使用顾问 1 的方案）。而他认为前四天的最大收益是：$2 + 5 + 9 + 13 = 29$。
>
> 现在你作为 Blue Mary 公司的副总经理，会不时收到金融顾问的设计方案，也需要随时回答 Blue Mary 对某天的“最大收益”的询问（这里的“最大收益”是按照 Blue Mary 的计算方法）。**一开始没有收到任何方案时，你可以认为每天的最大收益值是 0**。
>
> **输入格式**
>
> 第一行 ：一个整数 $N$，表示方案和询问的总数。 
>
> 接下来 $N$ 行，每行开头一个单词 `Query` 或 `Project`。 
>
> 若单词为 `Query`，则后接一个整数 $T$，表示 Blue Mary 询问第 $T$ 天的最大收益。 
>
> 若单词为 `Project`，则后接两个实数 $S, P$，表示该种设计方案第一天的收益 $S$，以及以后每天比上一天多出的收益 $P$。
>
> **输出格式**
>
> 对于每一个 `Query`，输出一个整数，表示询问的答案，并精确到整百元（以百元为单位，例如：该天最大收益为 $210$ 或 $290$ 时，均应该输出 $2$）。没有方案时回答询问要输出 $0$。
>
> 数据范围：$1 \leq N \leq 10 ^ 5$，$1 \leq T \leq 5\times 10 ^ 4$，$0 < P < 100$，$|S| \leq 10 ^ 5$。
>
> 题目链接：[P4254](https://www.luogu.com.cn/problem/P4254)。

本题和上题不同点在于输出的是最大值而不是线段编号，插入的是直线而不是线段。所以一个好消息就是我们不用判断区间了，而且也不用给 `max` 重新写 `Comp` 了。

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 100010, M = 50010;
const double eps = 1e-6;

int n;
char op[10];

struct Seg {
    double k, b;

    void init(double k1, double b1) {
        k = k1, b = b1;
    }
} seg[N];
int idx;

int compf(double x) {
    if (fabs(x) < eps) return 0;
    if (x < 0) return -1;
    return 1;
}

double apply(int id, int x) {
    return seg[id].k * x + seg[id].b;
}

struct Node {
    int l, r, segid;
} tr[M<<2];

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) return;
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
}

void modify(int u, int id) {
    if (!tr[u].segid) return tr[u].segid = id, void();
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (compf(apply(tr[u].segid, mid) - apply(id, mid)) < 0) swap(id, tr[u].segid);

    if (tr[u].l == tr[u].r) return;

    if (compf(apply(tr[u].segid, tr[u].l) - apply(id, tr[u].l)) < 0) modify(u<<1, id);
    if (compf(apply(tr[u].segid, tr[u].r) - apply(id, tr[u].r)) < 0) modify(u<<1|1, id);
}

double query(int u, int x) {
    double res = apply(tr[u].segid, x);
    if (tr[u].l == x && tr[u].r == x) return res;

    int mid = (tr[u].l + tr[u].r) >> 1;
    if (x <= mid) return max(res, query(u<<1, x));
    else return max(res, query(u<<1|1, x));
}

int main() {
    int x;
    double k, b, y1;
    scanf("%d", &n);
    build(1, 1, 50000);
    for (int i = 1; i <= n; i++) {
        scanf("%s", op);
        if (*op == 'Q') {
            scanf("%d", &x);
            double res = query(1, x);
            printf("%.0lf\n", floor(res / 100));
        }
        else {
            scanf("%lf%lf", &y1, &k);
            b = y1 - k;
            seg[++idx].init(k, b);
            modify(1, idx);
        }
    }
    return 0;
}
```

## [CEOI2017] Building Bridges

> 有 $n$ 根柱子依次排列，每根柱子都有一个高度。第 $i$ 根柱子的高度为 $h_i$。
>
> 现在想要建造若干座桥，如果一座桥架在第 $i$ 根柱子和第 $j$ 根柱子之间，那么需要 $(h_i-h_j)^2$ 的代价。
>
> 在造桥前，所有用不到的柱子都会被拆除，因为他们会干扰造桥进程。第 $i$ 根柱子被拆除的代价为 $w_i$，注意 $w_i$ 不一定非负，因为可能政府希望拆除某些柱子。
>
> 现在政府想要知道，通过桥梁把第 $1$ 根柱子和第 $n$ 根柱子连接的最小代价。注意桥梁不能在端点以外的任何地方相交。
>
> 对于 $100\%$ 的数据，有 $2\le n\le 10^5;0\le h_i,\vert w_i\vert\le 10^6$。
>
> 题目链接：[P4655](https://www.luogu.com.cn/problem/P4655)。

首先本题的 DP 方程巨简单。
$$
f_i=\min_{j<i}\{ f_j+(h_i-h_j)^2+\sum_{k=j+1}^{i-1} w_k\}
$$
设数列 $\{w_i\}$ 的前缀和数列是 $\{S_i\}$，然后对于每个 $i$ 来说，把只包含 $i$ 的式子提出来，变形为：
$$
f_i=\min_{j<i}\{-2h_jh_i+f_j+h_j^2-S_j\}+h_i^2+S_{i-1}
$$

所以我们的自变量是 $h_i$，把 $\min$ 里面的式子看成直线，用李超树求最小值。

我刚开始写的时候最大值写习惯了被坑了好几次。

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

#define L(u) (tr[u].l)
#define R(u) (tr[u].r)

typedef long long LL;
const int N = 100010, M = 1000010, MaxH = 1000000;
const LL INF = 1e18;
int n;
LL s[N], f[N], h[N];

struct Seg {
    LL k, b = INF;

    void init(LL k1, LL b1) {
        k = k1, b = b1;
    }
} seg[N];

struct Node {
    int l, r;
    int segid;
} tr[M<<2];

LL apply(int id, LL x) {
    return seg[id].k * x + seg[id].b;
}

void build(int u, int l, int r) {
    tr[u].l = l, R(u) = r;
    if (l == r) return;
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
}

void modify(int u, int id) {
    int mid = (L(u) + R(u)) >> 1;
    if (!tr[u].segid) return tr[u].segid = id, void();

    // 注意这里的不等号方向, 一定要记住自己在写最大值还是最小值
    if (apply(tr[u].segid, mid) > apply(id, mid)) swap(id, tr[u].segid);
    if (L(u) == R(u)) return;

    if (apply(tr[u].segid, L(u)) > apply(id, L(u))) modify(u<<1, id);
    else if (apply(tr[u].segid, R(u)) > apply(id, R(u))) modify(u<<1|1, id);
}

LL query(int u, LL x) {
    LL res = apply(tr[u].segid, x);
    if (L(u) == R(u)) return res;

    int mid = (L(u) + R(u)) >> 1;
    if (x <= mid) return min(res, query(u<<1, x));
    else return min(res, query(u<<1|1, x));
}

int main() {
    scanf("%d", &n);
    build(1, 1, MaxH);
    for (int i = 1; i <= n; i++) scanf("%lld", &h[i]);
    for (int i = 1; i <= n; i++) scanf("%lld", &s[i]), s[i] += s[i-1];

    f[1] = 0;
    seg[1].init(-2*h[1], f[1]+h[1]*h[1]-s[1]);
    modify(1, 1);

    for (int i = 2; i <= n; i++) {
        f[i] = query(1, h[i]) + (LL)h[i]*h[i] + s[i-1];
        seg[i].init(-2*h[i], f[i]+(LL)h[i]*h[i]-s[i]);
        modify(1, i);
    }

    printf("%lld\n", f[n]);
    return 0;
}
```

## [NOI2007] 货币兑换

> 小 Y 最近在一家金券交易所工作。该金券交易所只发行交易两种金券：A 纪念券（以下简称 A 券）和 B 纪念券（以下简称 B 券）。每个持有金券的顾客都有一个自己的帐户。金券的数目可以是一个实数。
>
> 每天随着市场的起伏波动，两种金券都有自己当时的价值，即每一单位金券当天可以兑换的人民币数目。我们记录第 $K$ 天中 A 券和 B 券的价值分别为 $A_K$ 和 $B_K$（元/单位金券）。
>
> 为了方便顾客，金券交易所提供了一种非常方便的交易方式：比例交易法。
>
> 比例交易法分为两个方面：
>
> a)  卖出金券：顾客提供一个 $[0, 100]$ 内的实数 $OP$ 作为卖出比例，其意义为：将 $OP\%$ 的 A 券和 $OP\%$ 的 B 券以当时的价值兑换为人民币；
>
> b)  买入金券：顾客支付 $IP$ 元人民币，交易所将会兑换给用户总价值为 $IP$ 的金券，并且，满足提供给顾客的 A 券和 B 券的比例在第 $K$ 天恰好为 $\mathrm{Rate}_ K$；
>
> 例如，假定接下来 $3$ 天内的 $A_K,B_K,\mathrm{Rate}_ K$ 的变化分别为：
>
> | 时间   | $A_K$ | $B_K$ | $\mathrm{Rate}_ K$ |
> | ------ | ----- | ----- | ------------------ |
> | 第一天 | $1$   | $1$   | $1$                |
> | 第二天 | $1$   | $2$   | $2$                |
> | 第三天 | $2$   | $2$   | $3$                |
>
> 假定在第一天时，用户手中有 $100$ 元人民币但是没有任何金券。
>
> 用户可以执行以下的操作：
>
> | 时间   | 用户操作      | 人民币(元) | A 券的数量 | B 券的数量 |
> | ------ | ------------- | ---------- | ---------- | ---------- |
> | 开户   | 无            | $100$      | $0$        | $0$        |
> | 第一天 | 买入 $100$ 元 | $0$        | $50$       | $50$       |
> | 第二天 | 卖出 $50\%$   | $75$       | $25$       | $25$       |
> | 第二天 | 买入 $60$ 元  | $15$       | $55$       | $40$       |
> | 第三天 | 卖出 $100\%$  | $205$      | $0$        | $0$        |
>
> 注意到，同一天内可以进行多次操作。
>
> 小 Y 是一个很有经济头脑的员工，通过较长时间的运作和行情测算，他已经知道了未来 $N$ 天内的 A 券和 B 券的价值以及 $\mathrm{Rate}$。他还希望能够计算出来，如果开始时拥有 $S$ 元钱，那么 $N$ 天后最多能够获得多少元钱。
>
> **输入格式**
>
> 第一行两个正整数 $N,S$，分别表示小 Y 能预知的天数以及初始时拥有的钱数。
>
> 接下来 $N$ 行，第 $K$ 行三个实数 $A_K,B_K,\mathrm{Rate} _ K$ ，意义如题目中所述。
>
> **输出格式**
>
> 只有一个实数 $\mathrm{MaxProfit}$，表示第 $N$ 天的操作结束时能够获得的最大的金钱数目。答案保留 $3$ 位小数。
>
> 对于 $100\%$ 的测试数据，满足：
>
> $0 < A_K \leq 10$，$0 < B_K\le 10$，$0 < \mathrm{Rate}_K \le 100$，$\mathrm{MaxProfit}  \leq 10^9$。
>
> 必然存在一种最优的买卖方案满足：
>
> 每次买进操作使用完所有的人民币，每次卖出操作卖出所有的金券。
>
> 题目链接：[P4027](https://www.luogu.com.cn/problem/P4027)。

最后一句话特别重要，~~要没有反正我做不出来，有了照样做不出来~~。

首先列方程，计算出如果有 $f_j$ 元在第 $j$ 天分别买进多少券，设 $A,B$ 买入 $a_j,b_j$ 单位，$R_j$ 表示 $\text{Rate}_j$，首先有 $a=bR_j$，其次有 $aA_j+bB_j=x$，所以我们可以解出来：
$$
a_j=\frac{R_jf_j}{R_jA_j+B_j},b_j=\frac{f_j}{R_jA_j+B_j}
$$
然后由每次操作要么全部满入，要么全部卖出，那么就可以列出：
$$
f_i=\max_{j<i}\{a_jA_i+b_jB_i\}=B_i\cdot \max_{j<i}\{a_j\frac{A_i}{B_i} +b_j\}
$$
将 $A_i/B_i$ 看作自变量，$a_j$ 看作斜率，所以又可以用李超树做了。这里我们可以把自变量离散化一下，一个是防止精度问题，另一个是既然是值域线段树如果是实数到底开多少点如果不离散化也无从得知。

当然，也可以选择什么都不买，即 $f_i=\max\{f_i,f_{i-1}\}$ 这里直接用滚动数组优化了，也就是压根不开 $f$ 这个数组，直接用单个变量。这里写一个动态开点的李超树练习一下。

```cpp
#include <cmath>
#include <cstdio>
#include <vector>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 100010;
const double eps = 1e-7;
int n, m;
double k[N], t[N], A[N], B[N], C[N], R[N];
vector<double> nums;

struct Node {
    int lson, rson;
    int id;
} tr[N];
int root = 1, idx = 1;

bool compf(double a, double b) {
    double x = a - b;
    if (fabs(x) < eps) return false;
    return x < 0;
}

double apply(int id, int x) {
    return nums[x-1] * k[id] + t[id];
}

int find(double x) {
    // 我这里一开始没加自定义比较 WA 了好多次
    return lower_bound(nums.begin(), nums.end(), x, compf) - nums.begin() + 1;
}

double query(int& u, int L, int R, int x) {
    int mid = (L + R) >> 1;
    double res = apply(tr[u].id, x);

    if (x <= mid && tr[u].lson) return max(res, query(tr[u].lson, L, mid, x));
    else if (mid+1 <= x && tr[u].rson) return max(res, query(tr[u].rson, mid+1, R, x));
    else return res;
}

void modify(int& u, int L, int R, int id) {
    if (!u) u = ++idx;
    if (!tr[u].id) return tr[u].id = id, void();

    int mid = (L + R) >> 1;
    if (compf(apply(tr[u].id, mid), apply(id, mid))) swap(id, tr[u].id);
    if (L == R) return;

    if (compf(apply(tr[u].id, L), apply(id, L))) modify(tr[u].lson, L, mid, id);
    if (compf(apply(tr[u].id, R), apply(id, R))) modify(tr[u].rson, mid+1, R, id);
}

int main() {
    double f;
    scanf("%d%lf", &n, &f);

    for (int i = 1; i <= n; i++) {
        scanf("%lf%lf%lf", &A[i], &B[i], &R[i]);
        C[i] = A[i] / B[i];
        nums.emplace_back(C[i]);
    }

    sort(nums.begin(), nums.end(), compf);
    nums.erase(unique(nums.begin(), nums.end(), [](double a, double b){ return fabs(a-b) < eps; }), nums.end());
    m = nums.size();

    for (int i = 1; i <= n; i++) {
        f = max(f, B[i] * query(root, 1, m, find(C[i])));
        double a = R[i]*f/(R[i]*A[i]+B[i]), b = f/(R[i]*A[i]+B[i]);
        k[i] = a, t[i] = b;
        modify(root, 1, m, i);
    }

    printf("%.3lf\n", f);
    return 0;
}
```

## [NOI2014] 购票

> 全国的城市构成了一棵以 SZ 市为根的有根树，每个城市与它的父亲用道路连接。为了方便起见，我们将全国的 $n$ 个城市用 $1\sim n$ 的整数编号。其中 SZ 市的编号为 $1$。对于除 SZ 市之外的任意一个城市 $v$，我们给出了它在这棵树上的父亲城市 $f_v$  以及到父亲城市道路的长度 $s_v$。
>
> 从城市 $v$ 前往 SZ 市的方法为：选择城市 $v$ 的一个祖先 $a$，支付购票的费用，乘坐交通工具到达 $a$。再选择城市 $a$ 的一个祖先 $b$，支付费用并到达 $b$。以此类推，直至到达 SZ 市。
>
> 对于任意一个城市 $v$，我们会给出一个交通工具的距离限制 $l_v$。对于城市 $v$ 的祖先 A，只有当它们之间所有道路的总长度不超过 $l_v$  时，从城市 $v$ 才可以通过一次购票到达城市 A，否则不能通过一次购票到达。  
>
> 对于每个城市 $v$，我们还会给出两个非负整数 $p_v,q_v$  作为票价参数。若城市 $v$ 到城市 A 所有道路的总长度为 $d$，那么从城市 $v$ 到城市 A 购买的票价为 $dp_v+q_v$。
>
> 求所有结点到根结点的最小花费。
>
> **输入格式**
>
> 第一行包含两个非负整数 $n,t$，分别表示城市的个数和数据类型（其意义将在「提示与说明」中提到）。
>
> 接下来 $2 \sim n$ 行，每行描述一个除 SZ 之外的城市。其中第 $v$ 行包含五个非负整数 $f_v,s_v,p_v,q_v,l_v$，分别表示城市 $v$ 的父亲城市，它到父亲城市道路的长度，票价的两个参数和距离限制。
>
> 请注意：输入不包含编号为 1 的 SZ 市，第 $2\sim n$ 行分别描述的是城市 $2$ 到城市 $n$。
>
> **输出格式**
>
> 输出包含 $n-1$ 行，每行包含一个整数。
>
> 其中第 $v$ 行表示从城市 $v+1$ 出发，到达 SZ 市最少的购票费用。
>
> 同样请注意：输出不包含编号为 1 的 SZ 市。
>
> 对于所有数据，$n\leq 2 \times 10^5, 0 \leq p_v \leq 10^6,\ 0 \leq q_v \leq 10^{12},\ 1\leq f_v<v,\ 0<s_v\leq lv \leq 2 \times 10^{11}$，且任意城市到 SZ 市的总路程长度不超过 $2 \times 10^{11}$。
>
> 题目链接：[P2305](https://www.luogu.com.cn/problem/P2305)。

容易观察出是一个 DP，推一下转移方程，这里我不知道条件写在 $\min$ 上面是不是符合规范，但是全写在下边肯定不好看：
$$
f_i=\min_{j\in path(1,i)}^{d_i-d_j\le l_i}\{f_j+p_i(d_i-d_j)+q_i\}
$$
拆一下，可以看出是斜率优化：
$$
f_i=\min\{-d_jp_i+f_j\}+p_id_i+q_i
$$
我们把问题抽象一下，相当于问根到当前结点的路径上后若干个点中若干直线在某点的最小值。

因为李超树维护的并不具备可减性，所以可持久化李超树不能很好的解决这个问题，但是对于在具体点处求值是很好合并的（说人话就是取最小值是很容易的）所以我们用树套树可以解决这一问题。

关于本题，我们希望找到特定结点区间内插入的直线，所以用树剖维护结点区间，然后树剖线段树上每个结点对应的李超树都是在这个线段树结点内维护的树上结点插入的所有直线，那么分析一下复杂度，树剖可以 $O(\log n)$ 条区间，每个区间线段树都要进行 $O(\log n)$ 次修改，每次对李超树的修改都要造成 $O(\log p)$ 的复杂度，一共是 $n$ 个结点，所以最终的复杂度是 $O(n\log^2n\log p)$，给了 $3s$ 时限，加上树剖优秀的常数，是可以 A 掉本题的，[提交记录](https://www.luogu.com.cn/record/135109130)。

但是这里可以把树剖的 $O(\log n)$ 去掉，对于当前点到祖先的区间这一问题，我们使用出栈序的技术，可以把复杂度继续降。

出栈序是每个点在回溯前记录的编号，可以把下面这个图输入 [Graph Editor](https://csacademy.com/app/graph_editor/)：

```text
1
2
3
4
5
6
6 5
5 1
5 2
5 3
5 4
```

将 $6$ 作为根，我们假设遍历到 $2$，并且我们要寻问它到根这个区间，那么会询问 $[2,6]$ 这个区间，而询问时右边的子树中都没有被遍历到，因此不会对数据造成影响，类似于可持久化维护值域数据结构，后面的版本并不会对当前版本数据结构进行影响。

再扯一下李超树的空间复杂度，李超树单次加入直线最多只会新增 $1$ 个结点，所以我们进行了 $n$ 次线段树上修改，每次线段树都会分成 $O(\log n)$ 区间，因此空间复杂度也是 $O(n\log n)$，常数大概是 $2$，这个地方我不太清楚，我们这里直接开 $4n\log n$ 一定可以。实际上直接开 $20n$ 可以通过。

基于此事实，可以写出下面的代码，复杂度 $O(n\log n\log p)$，比较稳：

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 2e5+10, LogN = 18, M = 1e6, INF = 1e18;
int h[N], fa[N], f[N], p[N], q[N], l[N], id[N], d[N], pre[N], stk[N], tp, n, idx = 2;

struct Edge {
    int to, nxt, w;
} e[N];

void add(int a, int b, int c) {
    e[idx] = {b, h[a], c}, h[a] = idx++;
}

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t * 10 + (ch ^ 48), ch = getchar();
}

namespace LICHAO {
    struct Segment {
        int k, b;
    } seg[N];

    struct Node {
        int segid;
        int ls, rs;
    } tr[4*N*LogN];

    int idx, segidx;

    int newseg(int k, int b) {
        seg[++segidx] = {k, b};
        return segidx;
    }

    int apply(int id, int x) {
        return seg[id].k * x + seg[id].b;
    }

    int query(int u, int x, int L, int R) {
        int res = INF;
        if (!u) return res;
        if (tr[u].segid) res = apply(tr[u].segid, x);
        
        int mid = (L+R) >> 1;
        if (x <= mid) return min(res, query(tr[u].ls, x, L, mid));
        else return min(res, query(tr[u].rs, x, mid+1, R));
    }

    void update(int& u, int id, int L, int R) {
        if (!u) u = ++idx;
        if (!tr[u].segid) return tr[u].segid = id, void();

        int mid = (L+R) >> 1;
        if (apply(id, mid) < apply(tr[u].segid, mid)) swap(id, tr[u].segid);
        if (L == R) return;

        if (apply(id, L) < apply(tr[u].segid, L)) update(tr[u].ls, id, L, mid);
        if (apply(id, R) < apply(tr[u].segid, R)) update(tr[u].rs, id, mid+1, R);
    }
}

namespace SEG {
    struct Node {
        int rt;
    } tr[N<<2];

    int query(int u, int x, int l, int r, int L, int R) {
        if (l <= L && R <= r) return LICHAO::query(tr[u].rt, x, 0, M);
        int res = INF, mid = (L+R) >> 1;
        if (l <= mid) res = query(u<<1, x, l, r, L, mid);
        if (mid+1 <= r) res = min(res, query(u<<1|1, x, l, r, mid+1, R));
        return res;
    }

    void update(int u, int id, int x, int L, int R) {
        LICHAO::update(tr[u].rt, id, 0, M);
        if (L == x && R == x) return;
        int mid = (L+R) >> 1;
        if (x <= mid) update(u<<1, id, x, L, mid);
        else update(u<<1|1, id, x, mid+1, R);
    }
}

void init(int u, int from) {
    static int ft;
    d[u] = d[fa[u]] + from;
    for (int i = h[u]; i; i = e[i].nxt) init(e[i].to, e[i].w);
    id[u] = ++ft;
}

void dp(int u) {
    ++tp;
    pre[tp] = d[u];
    stk[tp] = u;

    if (u != 1) f[u] = SEG::query(1, p[u], id[u], id[stk[lower_bound(pre+1, pre+tp+1, d[u] - l[u]) - pre]], 1, n) + p[u]*d[u] + q[u];
    SEG::update(1, LICHAO::newseg(-d[u], f[u]), id[u], 1, n);
    for (int i = h[u]; i; i = e[i].nxt) dp(e[i].to);
    --tp;
}

signed main() {
    int t;
    read(n), read(t);
    for (int i = 2, w; i <= n; i++) {
        read(fa[i]), read(w), read(p[i]), read(q[i]), read(l[i]);
        add(fa[i], i, w);
    }

    init(1, 0);
    dp(1);
    for (int i = 2; i <= n; i++) printf("%lld\n", f[i]);
    return 0;
}
```

