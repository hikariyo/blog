---
date: 2024-01-16 16:46:00
title: 杂项 - CDQ 分治
katex: true
tags:
- Algo
- C++
categories:
- OI
description: CDQ 分治是一种用来解决点对相关问题的思想，可以让一个偏序条件变成 0/1 条件，最终将满足一个条件元素的贡献到满足另一个条件的答案中，从而达到降低问题维度的目的。
---

## 简介

CDQ 分治是一种用来解决点对相关问题的思想，可以让一个偏序条件变成 $0/1$ 条件，最终将满足一个条件元素的贡献到满足另一个条件的答案中，从而达到降低问题维度的目的。

## [模板] 三维偏序

> 设 $f(i)$ 是满足 $x_j\le x_i,y_j\le y_i,z_j\le z_i$ 的点的数量，对于 $d\in [0,n)$ 输出 $f(i)=d$ 的数量。
>
> 数据范围：$1\le n\le 10^5,1\le a_i,b_i,c_i\le 2\times 10^5$。
>
> 题目链接：[P3810](https://www.luogu.com.cn/problem/P3810)。

首先在最外维按照字典序排序，这是为了保证对于任意一个位置答案只可能是在它前面的元素贡献给它的。

然后参考归并排序的思路，首先求出子区间内的答案，并且对 $y$ 排序，每当发现 $y_{p_i}>y_{p_j}$，就说明 $p_{l}\sim p_{i-1}$ 都满足 $y\le y_{p_j}$。

对于 $z$ 用树状数组记录，此时 $x$ 的偏序条件就变成了元素是否在左区间内，转化为了 $0/1$ 条件。答案只能从左区间贡献到右区间，此时处理的是一个二维偏序问题，也对应了文章开头作出的解释。

此外，在分治前需要先去重，并且能统计出来每个元素出现了多少次，因为 CDQ 分治对完全相同的元素统计会出问题。题目这种奇怪的输出方式应该是为了减小 IO 对于程序效率的影响。

时间复杂度：$T(n)=2T(n/2)+O(n\log n)=O(n\log^2n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

namespace IO { /* 省略 */ }
using IO::read;
using IO::write;

#define lowbit(x) ((x)&(-x))
const int N = 100010, M = 200010;
int n, k, ans[N];
struct Point {
    int x, y, z, s, res;
    bool operator==(const Point& p) const {
        return x == p.x && y == p.y && z == p.z;
    }

    bool operator<(const Point& p) const {
        if (x != p.x) return x < p.x;
        if (y != p.y) return y < p.y;
        return z < p.z;
    }
} p[N], tmp[N];

struct BIT {
    int tr[M];

    int query(int p) {
        int res = 0;
        for (; p; p -= lowbit(p)) res += tr[p];
        return res;
    }

    void add(int p, int k) {
        for (; p < M; p += lowbit(p)) tr[p] += k;
    }
} bit;

void CDQ(int l, int r) {
    if (l >= r) return;
    int mid = (l+r) >> 1;
    CDQ(l, mid), CDQ(mid+1, r);
    int i = l, j = mid+1, cnt = 0;
    while (i <= mid && j <= r) {
        if (p[i].y <= p[j].y) bit.add(p[i].z, p[i].s), tmp[cnt++] = p[i++];
        else p[j].res += bit.query(p[j].z), tmp[cnt++] = p[j++];
    }

    while (i <= mid) bit.add(p[i].z, p[i].s), tmp[cnt++] = p[i++];
    while (j <= r) p[j].res += bit.query(p[j].z), tmp[cnt++] = p[j++];

    for (i = l; i <= mid; i++) bit.add(p[i].z, -p[i].s);
    for (i = l; i <= r; i++) p[i] = tmp[i-l];
}

int main() {
    read(n, k);
    for (int i = 1; i <= n; i++) read(p[i].x, p[i].y, p[i].z);
    sort(p+1, p+1+n);
    int m = 0;
    for (int i = 1; i <= n; i++) {
        int ptr = i;
        while (p[ptr] == p[i]) p[i].s++, ptr++;
        p[++m] = p[i];
        i = ptr-1;
    }
    CDQ(1, m);
    for (int i = 1; i <= m; i++) ans[p[i].res + p[i].s - 1] += p[i].s;
    for (int i = 0; i < n; i++) write(ans[i]);
    return 0;
}
```

## [CQOI2017] 老C的任务

> 给定 $n$ 个点，每个点 $(x,y,p)$，再给 $m$ 个查询，每次查询 $(x_1,y_1)\sim(x_2,y_2)$ 区域内所有点的 $p$ 之和。
>
> 题目链接：[P3755](https://www.luogu.com.cn/problem/P3755)。

这里用前缀和的思想，每个查询存四个点，所以这就要存一下在统计答案的时候应该加还是减了，用一个 `sgn` 表示符号。

对于在 CDQ 分治中的 $z$ 坐标，令点 $z=0$，查询 $z=1$，这是为了在满足 $x,y$ 偏序的情况下，保证查询点会在数据点后面，这也是本题需要用 CDQ 分治的原因。当然有别的解法，因为本文是介绍 CDQ 分治的所以就写了这个。

时间复杂度：由于没有用到树状数组，$T(n)=2T(n/2)+O(n)=O(n\log n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

namespace IO { /* 省略 */ }
using IO::read;
using IO::write;

#define int long long
const int N = 500010;

struct Point {
    int x, y, z, p, sgn, id;
    int res;
    
    bool operator<(const Point& d) const {
        if (x != d.x) return x < d.x;
        if (y != d.y) return y < d.y;
        return z < d.z;
    }
} p[N], tmp[N];

int n, m, ans[N];

void CDQ(int l, int r) {
    if (l >= r) return;
    int mid = (l+r) >> 1;
    CDQ(l, mid), CDQ(mid+1, r);
    
    int i = l, j = mid+1, cnt = 0, res = 0;

    while (i <= mid && j <= r) {
        if (p[i].y <= p[j].y) res += p[i].p, tmp[cnt++] = p[i++];
        else p[j].res += res, tmp[cnt++] = p[j++];
    }
    
    while (i <= mid) tmp[cnt++] = p[i++];
    while (j <= r) p[j].res += res, tmp[cnt++] = p[j++];
    
    for (i = l; i <= r; i++) p[i] = tmp[i-l];
}

signed main() {
    read(n, m);
    for (int i = 0; i < n; i++) read(p[i].x, p[i].y, p[i].p);
    int k = n;
    for (int i = 0, x1, y1, x2, y2; i < m; i++) {
        read(x1, y1, x2, y2);
        p[k++] = {x2, y2, 1, 0, 1, i};
        p[k++] = {x1-1, y1-1, 1, 0, 1, i};
        p[k++] = {x2, y1-1, 1, 0, -1, i};
        p[k++] = {x1-1, y2, 1, 0, -1, i};
    }
    sort(p, p+k);
    CDQ(0, k-1);
    
    for (int i = 0; i < k; i++) if (p[i].z) ans[p[i].id] += p[i].res * p[i].sgn;
    for (int i = 0; i < m; i++) write(ans[i]);
    return 0;
}
```

## [CQOI2011] 动态逆序对

> 给定 $1\sim n$ 的一个排列，给定 $m$ 个要删除的元素，依次输出每个元素删除前整个序列的逆序对数。
>
> 数据范围：$1\le n\le 10^5,1\le m\le 5\times 10^4$。
>
> 题目链接：[P3157](https://www.luogu.com.cn/problem/P3157)。

定义 $x_i,y_i,z_i$ 分别是元素的值，元素在序列中的位置和删除时间，如果没删除就是 $n$，那么每次删除减少的就是在这个数前面并且比它大的没删除的数以及在这个数后面比它小的没删除的数，分别统计一下加起来就好，即：
$$
\begin{cases}
x_j\gt x_i\\
y_j\lt y_i\\
z_j\gt z_i
\end{cases}
+\begin{cases}
x_j\lt x_i\\
y_j\gt y_i\\
z_j\gt z_i
\end{cases}
$$
处理两遍就行了，复杂度 $O(n\log^2n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
namespace IO { /* 省略 */ }
using IO::read;
using IO::write;

const int N = 100010;
int n, m, a[N], b[N], d[N], val[N];

struct Point {
    int x, y, z, res;
} p[N], tmp[N];

struct BIT {
    int tr[N], tot;

    int query(int p) {
        int res = 0;
        for (; p; p -= p&-p) res += tr[p];
        return res;
    }

    void add(int p, int k) {
        tot += k;
        for (; p < N; p += p&-p) tr[p] += k;
    }
} bit;

void CDQ(int l, int r, int f) {
    if (l >= r) return;
    int mid = (l+r) >> 1;
    CDQ(l, mid, f), CDQ(mid+1, r, f);
    int i = l, j = mid+1, k = l;
    while (i <= mid && j <= r) {
        if (f*p[i].y <= f*p[j].y) bit.add(p[i].z, 1), tmp[k++] = p[i++];
        else p[j].res += bit.tot - bit.query(p[j].z), tmp[k++] = p[j++];
    }
    while (i <= mid) bit.add(p[i].z, 1), tmp[k++] = p[i++];
    while (j <= r) p[j].res += bit.tot - bit.query(p[j].z), tmp[k++] = p[j++];
    for (i = l; i <= mid; i++) bit.add(p[i].z, -1);
    for (i = l; i <= r; i++) p[i] = tmp[i];
}

signed main() {
    read(n, m);
    for (int i = 1; i <= n; i++) read(a[i]), b[a[i]] = i, p[i] = {a[i], i, n};
    for (int i = 1; i <= m; i++) read(d[i]), p[b[d[i]]].z = i;
    sort(p+1, p+1+n, [](const Point& p, const Point& q) { return p.x < q.x; });
    CDQ(1, n, -1);
    sort(p+1, p+1+n, [](const Point& p, const Point& q) { return p.x > q.x; });
    CDQ(1, n, 1);
    int cnt = 0;
    for (int i = 1; i <= n; i++) cnt += bit.tot - bit.query(a[i]), bit.add(a[i], 1);
    for (int i = 1; i <= n; i++) val[p[i].x] = p[i].res;
    for (int i = 1; i <= m; i++) write(cnt), cnt -= val[d[i]];
    return 0;
}
```

## [CF1921G] Mischievous Shooter

> 给定 $n\times m$ 的矩阵，每个点有点权 $0/1$，每个点能到达 左上/左下/右上/右下 四个方向的曼哈顿距离不超过 $k$ 的点，问能到大的点权和的最大值。原题有配图，可以去 CF 看。
>
> 数据范围：$1\le k,\sum n\times m\le 10^5$。
>
> 题目链接：[CF1921G](https://codeforces.com/contest/1921/problem/G)。

首先通过旋转矩阵可以让这几个方向统一变成左上角，然后考虑什么样的点 $(x_1,y_1)$ 可以贡献给 $(x_2,y_2)$。

由于保证 $x_1,y_1$ 在左上方，于是有 $x_1\le x_2,y_1\le y_2$，然后曼哈顿距离为 $|x_1-x_2|+|y_1-y_2|=x_2+y_2-(x_1+y_1)$。

令所有点的 $z_i=x_i+y_i$ 于是就有条件约束 $z_2-z_1\le k$，整理一下就是：
$$
\begin{cases}
x_1\le x_2\\
y_1\le y_2\\
z_2-k\le z_1\le z_2\\
\end{cases}
$$
通过 CDQ 分治可以得到满足前两维偏序性质的点，然后用树状数组求满足第三个性质的点。

复杂度同样是 $O(nm\log^2(nm))$，本题有 $O(nm)$ 做法，但是我没想到，但是 2log 也是可以过的。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010, M = 200010;

namespace IO { /* 省略 */ }
using IO::read;
using IO::write;

struct Point {
    int x, y, z, s, res;
} p[N], tmp[N];

struct BIT {
    int tr[M];

    int query(int p, int res = 0) {
        p = min(p+1, M-1);
        if (p <= 0) return 0;
        for (; p; p -= p&-p) res += tr[p];
        return res;
    }

    void modify(int p, int k) {
        if (!k) return;
        p++;
        for (; p < M; p += p&-p) tr[p] += k;
    }
} bit;

void CDQ(int l, int r, int k) {
    if (l >= r) return;
    int mid = (l+r) >> 1;
    CDQ(l, mid, k), CDQ(mid+1, r, k);

    int i = l, j = mid+1, cnt = 0;
    while (i <= mid && j <= r) {
        if (p[i].y <= p[j].y) bit.modify(p[i].z, p[i].s), tmp[cnt++] = p[i++];
        else p[j].res += bit.query(p[j].z) - bit.query(p[j].z-k-1), tmp[cnt++] = p[j++];
    }

    while (i <= mid) bit.modify(p[i].z, p[i].s), tmp[cnt++] = p[i++];
    while (j <= r) p[j].res += bit.query(p[j].z) - bit.query(p[j].z-k-1), tmp[cnt++] = p[j++];

    for (i = l; i <= mid; i++) bit.modify(p[i].z, -p[i].s);
    for (i = l; i <= r; i++) p[i] = tmp[i-l];
}

int main() {
    int T, n, m, k;
    read(T);
    while (T--) {
        read(n, m, k);
        vector<vector<char>> mat(n, vector<char>(m));

        auto rotate = [&]() -> void {
            vector<vector<char>> res(m, vector<char>(n));
            for (int i = 0; i < n; i++) for (int j = 0; j < m; j++) res[j][n-i-1] = mat[i][j];
            mat.swap(res);
            swap(n, m);
        };

        int ans = 0;
        for (int i = 0; i < n; i++) for (int j = 0; j < m; j++) read(mat[i][j]);
        for (int i = 0; i < 4; i++) {
            if (i) rotate();
            int cnt = 0;
            for (int i = 0; i < n; i++) for (int j = 0; j < m; j++) p[cnt++] = {i, j, i+j, mat[i][j] == '#', 0};
            CDQ(0, cnt-1, k);
            for (int i = 0; i < cnt; i++) ans = max(ans, p[i].res + p[i].s);
        }
        write(ans);
    }

    return 0;
}
```

## [模板] 四维 LIS

> 给定 $n$ 个点 $(a_i,b_i,c_i,d_i)$，求出 LIS。$1\le n \le 5\times 10^4$，$1\le m\le 10^9$，$m$ 是所有坐标的绝对值的最大值。
>
> 题目链接：[P3769](https://www.luogu.com.cn/problem/P3769)。

用 CDQ 分治的目的是将 $(x,y,z)$ 转化为 $(0/1,y,z)$，只能由 $0$ 向 $1$ 产生贡献。

首先将 $a$ 排序，由于 LIS 的转移式如下，其中 $P_i \le P_j$ 意思是两点满足偏序关系：
$$
dp_i=\max_{P_j\le P_i} \{dp_j+s_j \}
$$
我们无法同时求出左右区间的答案，因为右区间内部的左区间可能依赖左区间的答案，所以要先处理左区间，然后用左区间更新右区间的答案后再处理右区间。

所以我们不手动进行归并排序了，直接用 `sort`，这里注意默认的 `sort` 当比较函数认为相等时元素的相对位置是不确定的，而 `stable_sort` 会保证相等的元素相对位置不改变，一般来说还是用 `sort` 会更好。

在处理 $[l,r]$ 时，首先递归处理 $[l,mid]$，然后按照左右区间给 $a$ 分类为 $0/1$，此时问题就变成了：坐标为 $(b,c,d)$，保证 $a=0$ 给 $a=1$ 贡献答案，到这个时候成功将 $a$ 这一维度去掉了。

同上，去掉 $b$ 这一维度，给 $c$ 排序，保证满足要求的 $d$ 从 $a,b=0$ 贡献给 $a,b=1$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 50010;

namespace IO { /* 省略 */ }
using IO::read;
using IO::write;

struct Point {
    int a, b, c, d, s, ta, tb, res;
    bool operator==(const Point& p) const {
        return a == p.a && b == p.b && c == p.c && d == p.d;
    }
} p[N];

struct BIT {
    int tr[N];

    void update(int p, int k) {
        for (; p < N; p += p&-p) tr[p] = max(tr[p], k);
    }

    int query(int p) {
        int res = 0;
        for (; p; p -= p&-p) res = max(res, tr[p]);
        return res;
    }

    void clear(int p) {
        for (; p < N; p += p&-p) tr[p] = 0;
    }
} bit;

bool cmpa(const Point& p, const Point& q) {
    if (p.a != q.a) return p.a < q.a;
    if (p.b != q.b) return p.b < q.b;
    if (p.c != q.c) return p.c < q.c;
    return p.d < q.d;
}

bool cmpb(const Point& p, const Point& q) {
    if (p.b != q.b) return p.b < q.b;
    if (p.c != q.c) return p.c < q.c;
    if (p.d != q.d) return p.d < q.d;
    return p.a < q.a;
}

bool cmpc(const Point& p, const Point& q) {
    if (p.c != q.c) return p.c < q.c;
    if (p.d != q.d) return p.d < q.d;
    if (p.a != q.a) return p.a < q.a;
    return p.b < q.b;
}

void CDQ2(int l, int r) {
    if (l >= r) return;
    int mid = (l+r) >> 1;
    CDQ2(l, mid);
    for (int i = l; i <= r; i++) p[i].tb = i > mid;
    sort(p+l, p+r+1, cmpc);
    for (int i = l; i <= r; i++) {
        if (!p[i].ta && !p[i].tb) bit.update(p[i].d, p[i].res + p[i].s);
        if (p[i].ta && p[i].tb) p[i].res = max(p[i].res, bit.query(p[i].d));
    }
    for (int i = l; i <= r; i++) {
        if (!p[i].ta && !p[i].tb) bit.clear(p[i].d);
    }
    sort(p+l, p+r+1, cmpb);
    CDQ2(mid+1, r);
}

void CDQ1(int l, int r) {
    if (l >= r) return;
    int mid = (l+r) >> 1;
    CDQ1(l, mid);
    for (int i = l; i <= r; i++) p[i].ta = i > mid;
    sort(p+l, p+r+1, cmpb);
    CDQ2(l, r);
    sort(p+l, p+r+1, cmpa);
    CDQ1(mid+1, r);
}

vector<int> nums;

int main() {
    int n, ans = 0;
    read(n);
    nums.reserve(n);
    for (int i = 1; i <= n; i++) read(p[i].a, p[i].b, p[i].c, p[i].d), nums.emplace_back(p[i].d);
    sort(nums.begin(), nums.end());
    for (int i = 1; i <= n; i++) p[i].d = lower_bound(nums.begin(), nums.end(), p[i].d) - nums.begin() + 1;
    sort(p+1, p+1+n, cmpa);

    int m = 0;
    for (int i = 1; i <= n; i++) {
        int ptr = i;
        while (p[ptr] == p[i]) p[i].s++, ptr++;
        p[++m] = p[i];
        i = ptr-1;
    }
    
    CDQ1(1, m);
    for (int i = 1; i <= m; i++) ans = max(ans, p[i].res + p[i].s);
    return write(ans), 0;
}
```

内层复杂度为 $T(n)=2T(n/2)+O(n\log n)=O(n\log^2n)$，外层复杂度为 $T(n)=2T(n/2)+O(n\log^2n)=O(n\log^3n)$，可以通过本题。

也可以套三层，只不过由于常数太大无法通过全部测试点：

```cpp
void CDQ3(int l, int r) {
    if (l >= r) return;
    int mid = (l+r) >> 1;
    CDQ3(l, mid);
    for (int i = l; i <= r; i++) p[i].tc = i > mid;
    sort(p+l, p+r+1, cmpd);
    int res = 0;
    for (int i = l; i <= r; i++) {
        if (!p[i].ta && !p[i].tb && !p[i].tc) res = max(res, p[i].res + p[i].s);
        if (p[i].ta && p[i].tb && p[i].tc) p[i].res = max(p[i].res, res);
    }
    sort(p+l, p+r+1, cmpc);
    CDQ3(mid+1, r);
}

void CDQ2(int l, int r) {
    if (l >= r) return;
    int mid = (l+r) >> 1;
    CDQ2(l, mid);
    for (int i = l; i <= r; i++) p[i].tb = i > mid;
    sort(p+l, p+r+1, cmpc);
    CDQ3(l, r);
    sort(p+l, p+r+1, cmpb);
    CDQ2(mid+1, r);
}

void CDQ1(int l, int r) {
    if (l >= r) return;
    int mid = (l+r) >> 1;
    CDQ1(l, mid);
    for (int i = l; i <= r; i++) p[i].ta = i > mid;
    sort(p+l, p+r+1, cmpb);
    CDQ2(l, r);
    sort(p+l, p+r+1, cmpa);
    CDQ1(mid+1, r);
}
```

