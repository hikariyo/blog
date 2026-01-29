---
date: 2024-01-29 01:03:00
updated: 2024-01-29 01:03:00
title: 计算几何 - 凸包
katex: true
description: 凸包的定义是能包含住所有点的凸多边形的交集，通过 Andrew 算法可以在 O(nlog(n)) 排序后用栈线性维护上下凸包，从而求出所有在凸包上的点，或者用平衡树动态维护凸包。
tags:
- Algo
- C++
- Geometry
categories:
- OI
---

## 简介

凸包的定义是能包含住所有点的凸多边形的交集，通过 Andrew 算法可以在 $O(n\log n)$ 排序后用栈线性维护上下凸包，从而求出所有在凸包上的点，或者用平衡树动态维护凸包。

## [USACO5.1] Fencing the Cows

> 给定 $n$ 个点，求凸包的周长。
>
> 数据范围：$1\le n\le 10^5,-10^6\le x_i,y_i\le 10^6$。
>
> 题目链接：[P2742](https://www.luogu.com.cn/problem/P2742)。

Andrew 算法的流程是对所有点以 $x,y$ 的字典序升序排序，这就保证了在 $x$ 相同时，正着遍历是 $y$ 递增的，逆序遍历是递减的，然后就可以用一个栈来维护这些凸包。

具体来说，当栈中有 $\ge 2$​ 个元素时，设栈顶为点 $B$，栈顶下一个点为 $A$，新加入元素为 $C$，判断 $AB$ 是否能被 $AC$ 这条新边代替，如果能就弹出 $B$​，直到无法代替为止。容易看出每个点至多入栈出栈一次，所以是线性的。

注意，如果 $AB$ 与 $AC$ 共线，同样要弹出 $B$，保证在处理一段铅垂线时不会出现问题。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;
const double eps = 1e-8;
int n;

int sgn(double x) {
    if (fabs(x) < eps) return 0;
    return x > 0 ? 1 : -1;
}

struct Vec {
    double x, y;

    bool operator<(const Vec& v) const {
        if (sgn(x - v.x)) return x < v.x;
        return sgn(y - v.y) < 0;
    }

    Vec operator-(const Vec& v) const {
        return {x - v.x, y - v.y};
    }

    double operator^(const Vec& v) const {
        return x * v.y - y * v.x;
    }

    double len() const {
        return sqrt(x*x + y*y);
    }
} p[N];

bool judge(int a, int b, int c) {
    return sgn(p[a] - p[b] ^ p[c] - p[b]) >= 0;
}

double andrew() {
    static int stk[N];
    double ans = 0;
    int tp = -1;
    sort(p+1, p+1+n);
    for (int i = 1; i <= n; i++) {
        while (tp > 0 && judge(stk[tp-1], stk[tp], i)) tp--;
        stk[++tp] = i;
    }
    for (; tp; tp--) ans += (p[stk[tp]] - p[stk[tp-1]]).len();

    tp = -1;
    for (int i = n; i; i--) {
        while (tp > 0 && judge(stk[tp-1], stk[tp], i)) tp--;
        stk[++tp] = i;
    }
    for (; tp; tp--) ans += (p[stk[tp]] - p[stk[tp-1]]).len();

    return ans;
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%lf%lf", &p[i].x, &p[i].y);
    return !printf("%.2lf\n", andrew());
}
```

## [SHOI2012] 信用卡凸包

> 求 $n$ 个四角为 $\frac{1}{4}$​ 圆弧的矩形，对于每个图形，矩形在处理前的边长为 $w,h$，圆的半径为 $r$，中心坐标为 $x_i,y_i$，逆时针旋转的弧度为 $\theta_i$，求这些图形构成的凸包的周长。
>
> 数据范围：$1\le n\le 10^4$，$1\le w,h,r\le 10^6$，$-10^6\le x_i,y_i\le 10^6$，$0\le \theta_i\lt 2\pi$。
>
> 题目链接：[P3829](https://www.luogu.com.cn/problem/P3829)。

如果画出这些图形，并且画出凸包，不难得出结论即每个圆心到凸包上的边做垂线，并且按照凸包的形状连接圆心，可以看出圆心构成的凸包与真正的凸包只差了一个圆。证明也很显然，转一圈方向改变了 $2\pi$，所以应该经过圆心角为 $2\pi$ 的弧，也就是一个圆。

过 $Q_i(x_i,y_i)$ 做出的直线 $l$ 倾斜角为 $\theta_i$，构造向量 $|\vec{a}|=\frac{w}{2}-r,|\vec{b}|=\frac{h}{2}-r$，方向分别与为 $l$ 的方向向量和法向量相同，那么圆心 $P_i$ 的坐标就满足 $\vec{P_iQ_i}=\vec{a}+\vec{b},\vec{a}-\vec{b},-\vec{a}+\vec{b},-\vec{a}-\vec{b}$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;
const double eps = 1e-8, pi = acos(-1);
int n;
double w, h, r, theta[N];

int sgn(double x) {
    if (fabs(x) < eps) return 0;
    return x > 0 ? 1 : -1;
}

struct Vec {
    double x, y;

    bool operator<(const Vec& v) const {
        if (sgn(x - v.x)) return x < v.x;
        return sgn(y - v.y) < 0;
    }

    Vec& operator*=(double k) {        
        x *= k, y *= k;
        return *this;
    }

    Vec operator+(const Vec& v) const {
        return {x + v.x, y + v.y};
    }

    Vec operator-(const Vec& v) const {
        return {x - v.x, y - v.y};
    }

    double operator^(const Vec& v) const {
        return x * v.y - y * v.x;
    }

    double len() const {
        return sqrt(x*x + y*y);
    }
} p[N<<2], q[N];

bool judge(int a, int b, int c) {
    return sgn(p[a] - p[b] ^ p[c] - p[b]) >= 0;
}

void getcircles() {
    int k = 0;
    for (int i = 1; i <= n; i++) {
        Vec a = {cos(theta[i]), sin(theta[i])}, b = {-a.y, a.x};
        a *= w/2 - r, b *= h/2 - r;
        p[++k] = q[i] + a + b;
        p[++k] = q[i] + a - b;
        p[++k] = q[i] - a + b;
        p[++k] = q[i] - a - b;
    }
}

double andrew() {
    static int stk[N<<2];
    double ans = 0;
    int tp = -1;
    n <<= 2;
    sort(p+1, p+1+n);
    for (int i = 1; i <= n; i++) {
        while (tp > 0 && judge(stk[tp-1], stk[tp], i)) tp--;
        stk[++tp] = i;
    }
    for (; tp; tp--) ans += (p[stk[tp]] - p[stk[tp-1]]).len();

    tp = -1;
    for (int i = n; i; i--) {
        while (tp > 0 && judge(stk[tp-1], stk[tp], i)) tp--;
        stk[++tp] = i;
    }
    for (; tp; tp--) ans += (p[stk[tp]] - p[stk[tp-1]]).len();

    return ans;
}

int main() {
    scanf("%d%lf%lf%lf", &n, &h, &w, &r);
    for (int i = 1; i <= n; i++) scanf("%lf%lf%lf", &q[i].x, &q[i].y, &theta[i]);
    getcircles();
    return !printf("%.2lf\n", andrew() + 2 * pi * r);
}
```

## [HAOI2011] 防线修建

> 给定点 $n,x,y$ 表示 $(0,0)$，$(0,n)$，$(x,y)$ 初始三个点，再给 $m$ 个点 $(x_i,y_i)$，初始所有点都在上凸包中。
>
> 接下来 $q$ 个操作，每个操作是在当前凸包删除一个点，或者查询当前凸包的周长。
>
> 数据范围：$1\le m\le 10^5$，$1\le q\le 2\times 10^5$，$1\le n\le 10^4$。
>
> 题目链接：[P2521](https://www.luogu.com.cn/problem/P2521)。

删除是困难的，所以通过时间倒流大法可以让删除变成添加，这样我们需要操作的就是找到一个点的前驱和后继，需要用到平衡树。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 200010;
const double eps = 1e-8;
int n, m, q;
bool del[N];

int sgn(double x) {
    if (fabs(x) < eps) return 0;
    return x > 0 ? 1 : -1;
}

struct Query {
    int typ, u;
} qry[N];

struct Vec {
    double x, y;
    bool lim;

    bool operator<(const Vec& v) const {
        if (x != v.x) return x < v.x;
        return y < v.y;
    }

    Vec operator-(const Vec& v) const {
        return {x - v.x, y - v.y};
    }

    double operator^(const Vec& v) const {
        return x * v.y - y * v.x;
    }
} p[N], val[N];

double len(Vec v) {
    return sqrt(1.0 * v.x * v.x + 1.0 * v.y * v.y);
}

set<Vec> s;
double now;

void upd(Vec a, Vec b, int fac) {
    if (a.lim || b.lim) return;
    now += fac * len(a-b);
}

bool judge(Vec a, Vec b, Vec c) {
    return sgn(a - b ^ c - b) <= 0;
}

void ins(Vec v) {
    s.insert(v);
    auto it = s.find(v), pre = prev(it), nxt = next(it);
    if (judge(*pre, *it, *nxt)) return s.erase(it), void();
    upd(*pre, *nxt, -1), upd(*pre, *it, 1), upd(*it, *nxt, 1);
    while (pre != s.begin()) {
        auto ppre = prev(pre);
        if (!judge(*ppre, *pre, *it)) break;
        upd(*ppre, *pre, -1), upd(*it, *pre, -1), upd(*ppre, *it, 1);
        s.erase(pre);
        pre = ppre;
    }
    while (nxt != prev(s.end())) {
        auto nnxt = next(nxt);
        if (!judge(*it, *nxt, *nnxt)) break;
        upd(*nnxt, *nxt, -1), upd(*it, *nxt, -1), upd(*nnxt, *it, 1);
        s.erase(nxt);
        nxt = nnxt;
    }
}

signed main() {
    int x, y;
    scanf("%d%d%d%d", &n, &x, &y, &m);
    for (int i = 1; i <= m; i++) scanf("%lf%lf", &p[i].x, &p[i].y);

    scanf("%d", &q);
    for (int i = 1; i <= q; i++) {
        scanf("%d", &qry[i].typ);
        if (qry[i].typ == 1) {
            scanf("%d", &qry[i].u);
            del[qry[i].u] = true;
        }
    }

    s.insert({-1e10, -1e100, true}), s.insert({1e10, -1e100, true});
    ins({0, 0}), ins({1.0 * n, 0}), ins({1.0 * x, 1.0 * y});
    for (int i = 1; i <= m; i++) if (!del[i]) ins(p[i]);

    static double stk[N];
    int tp = 0;
    for (int i = q; i; i--) {
        if (qry[i].typ == 1) ins(p[qry[i].u]);
        else stk[++tp] = now;
    }
    while (tp) printf("%.2lf\n", stk[tp--]);
    return 0;
}
```

