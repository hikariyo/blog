---
date: 2024-01-28 15:28:00
updated: 2024-01-28 15:28:00
title: 计算几何 - 向量
katex: true
description: 通过向量之间的运算可以判断出点与直线的位置关系，主要使用叉积判断点在直线的哪一侧。
tags:
- Algo
- C++
- Geometry
categories:
- OI
---

## 叉积

对于点 $A,B,C$ 其中 $y_A\ge y_B$，要判断一个点在直线的左侧还是右侧，可以通过向量 $\vec{AC},\vec{BC}$ 的叉积来判断，令 $\vec{a}=\vec{AC}\times \vec{BC}$，由右手法则可以得到，当 $C$ 在 $AB$ 右侧时，$\vec{a}$ 在 $z$ 轴的分量大于 $0$，左侧时小于 $0$，在直线上时等于 $0$。

因此，直接判断叉积在 $z$ 轴的分量与 $0$ 的关系就可以判断点 $C$ 与直线 $AB$ 的关系，下文直接用 $\vec{AC}\times\vec{BC}$ 代表其在 $z$ 轴的分量。

## Toys

> 给定 $n$ 条线段将矩形分成 $n+1$ 个区域，并给 $m$​ 个点 $(x_i,y_i)$，求每个区域中有几个点，保证点不在线段上且在矩形内部。
>
> 数据范围：$1\le n,m\le 5000$，$-10^5\le x_i,y_i\le 10^5$。
>
> 题目链接：[POJ 2318](http://poj.org/problem?id=2318)。

对每个点二分出在它左边的最后一个直线，这样做是 $O(m\log n)$​ 的。为什么不用万能头？因为 POJ 不支持。

```cpp
#include <cstdio>
using namespace std;

#define int long long
const int N = 5010;
int n, m, ans[N];

struct Vec {
    int x, y;

    int operator^(const Vec& v) {
        return x * v.y - y * v.x;
    }

    Vec operator-(const Vec& v) const {
        return {x - v.x, y - v.y};
    }
};

struct Line {
    Vec p[2];

    bool judge(Vec v) {
        return (v - p[0] ^ v - p[1]) > 0;
    }
} lines[N];

int solve(const Vec& p) {
    int l = 0, r = n;
    while (l < r) {
        int mid = (l+r+1) >> 1;
        if (lines[mid].judge(p)) l = mid;
        else r = mid - 1;
    }
    return r;
}

signed main() {
    int Y1, Y2, cnt = 0;
    while (scanf("%lld%lld%*d%lld%*d%lld", &n, &m, &Y1, &Y2), n) {
        if (cnt++) puts("");
        for (int i = 0; i <= n; i++) ans[i] = 0, lines[i].p[0].y = Y1, lines[i].p[1].y = Y2;
        for (int i = 1; i <= n; i++) scanf("%lld%lld", &lines[i].p[0].x, &lines[i].p[1].x);
        for (int i = 1, x, y; i <= m; i++) scanf("%lld%lld", &x, &y), ans[solve({x, y})]++;
        for (int i = 0; i <= n; i++) printf("%lld: %lld\n", i, ans[i]);
    }
    return 0;
}
```

## Segments

> 给 $n$ 个线段，问是否存在一条直线满足所有线段在这条直线上的投影之间存在交点，如果 $|a-b|< 10^{-8}$ 认为 $a=b$。
>
> 数据范围：$1\le n\le 100$​。
>
> 题目链接：[POJ 3304](http://poj.org/problem?id=3304)。

如果存在这样一个点，它可以是某个线段投影的端点，所以考虑枚举端点，如果存在这样一条直线，那么这条线段的端点与它在这条直线上的投影之间的连线与所有线段相交，所以枚举端点作为直线经过的一个点，此时直线只有一个参数待定了。每经过一个线段都对这个直线的参数做出了一定约束，如果取到了约束的边界，一定是它碰到了另外一个线段的某个端点。

所以做法就是，枚举 $\binom{2n}{2}$ 个点对，确定一条直线，并且检查是否能包含住所有点，检查方式就是判断线段上两个点不在直线的同侧，转化为检查符号是否相同，并且当某个点在直线上是可以接受的，那么：
$$
\text{sgn}(\vec{AC}\times\vec{BC})\cdot \text{sgn}(\vec{AD}\times\vec{BD})> 0
$$
这样复杂度是 $O(n^3)$ 的，可以通过。

```cpp
#include <cstdio>
#include <cmath>
using namespace std;

const int N = 110;
const double eps = 1e-8;
int n, T;

int sgn(double x) {
    if (fabs(x) < eps) return 0;
    return x > 0 ? 1 : -1;
}

struct Vector {
    double x, y;

    double operator*(const Vector& v) const {
        return x * v.y - y * v.x;
    }

    Vector operator-(const Vector& v) const {
        return {x - v.x, y - v.y};
    }
    
    bool operator==(const Vector& v) const {
        return !sgn(x - v.x) && !sgn(y - v.y);
    }
} pnts[N*2], seg[N][2];

bool check(Vector a, Vector b) {
    if (a == b) return false;
    for (int i = 1; i <= n; i++) {
        int A = sgn((a - seg[i][0]) * (b - seg[i][0]));
        int B = sgn((a - seg[i][1]) * (b - seg[i][1]));
        if (A * B > 0) return false;
    }
    return true;
}

int main() {
    scanf("%d", &T);
    while (T--) {
        int k = 0;
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j <= 1; j++, k++) {
                scanf("%lf%lf", &pnts[k].x, &pnts[k].y);
                seg[i][j] = pnts[k];
            }
        }
        
        for (int i = 1; i <= k; i++) {
            for (int j = i+1; j <= k; j++) {
                if (check(pnts[i], pnts[j])) {
                    puts("Yes!");
                    goto end;
                }
            }
        }

        puts("No!"); end:;
    }
    return 0;
}
```

