---
date: 2023-08-05 16:06:00
updated: 2023-10-11 14:54:00
title: 数据结构 - ST 表
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
---

## 简介

ST 表是用来解决重复贡献问题的数据结构，相较于线段树，它的初始化复杂度 $O(n\log n)$，查询复杂度 $O(1)$，常数也更小，但无法修改。

重复贡献问题是指具有性质 $f(x,x)=x$ 并且具有结合律的运算，例如最大值、最小值、按位与、按位或、GCD 等。

关于预处理 $\lfloor\log_2 x\rfloor$ 的方法，可以用递推式：
$$
\lfloor \log_2 x\rfloor =\lfloor \log_2\lfloor \frac{x}{2}\rfloor\rfloor +1
$$
分类证明这个式子，设 $2^k \le x \lt 2^{k+1}$。

1. 当 $x$ 为偶数时有 $\text{RHS}=k-1+1=k=\text{LHS}$。
2. 当 $x$ 为奇数时一定有 $2^k \ne x$ 因此 $2^{k}+1\le x$，那么 $\lfloor x/2\rfloor=(x-1)/2\ge 2^{k-1}$，仍然有上面的式子成立。

所以可以用 $O(n)$ 的时间预处理出所有以二为底的对数。

## 静态 RMQ

>给定一个长度为 N 的数列，和 M 次询问，求出每一次询问的区间内数字的最大值。
>
>题目链接：[P3865](https://www.luogu.com.cn/problem/P3865)。

维护一个二维数组 $f(i,j)$ 表示从第 $i$ 位开始后 $2^j$ 个元素的最大值。

初始化时，类似于倍增 LCA 的方法：
$$
f(i,j)=\left\{\begin{aligned}
&a_i, j=0\\
&\max\{f(i,j-1), f(i+2^{j-1}, j-1)\},j>0
\end{aligned}\right .
$$
这里当 $j>0$ 时，覆盖的区间分别是 $[i, i+2^{j-1}-1],[i+2^{j-1},i+2^j-1]$，总共长度是 $2^j$，符合定义。

查询时，取 $k=\lfloor \log_2 \text{len}\rfloor$，然后返回下面的值：
$$
\max\left\{\begin{aligned}
&f(l,k)\\
&f(r-2^{k}+1,k)
\end{aligned}\right.
$$
下面这个式子的来源是方程 $r-x+1=2^k$ 然后得到 $x=r-2^k+1$，下面证明这两段一定能覆盖整个区间。
$$
\begin{aligned}
\lfloor \log_2\text{len}\rfloor &\gt \log_2\text{len}-1\\
&=\log_2\frac{\text{len}}{2}\\
\Rightarrow 2^k&\gt \frac{\text{len}}{2}\\
l+2^k-1 &\gt l+\frac{\text{len}}{2}-1\\
\Rightarrow l+2^k-1&\ge l+\frac{\text{len}}{2}=\frac{l+r+1}{2}\\
r-2^k+1 &\lt r-\frac{\text{len}}{2}+1\\
\Rightarrow r-2^k+1&\le r-\frac{\text{len}}{2}=\frac{l+r+1}{2}
\end{aligned}
$$
以 $(l+r+1)/2$ 为界，一定可以覆盖整个区间。

```cpp
#include <cstdio>
#include <cmath>
#include <algorithm>
using namespace std;

const int N = 1e5+10, M = 20;
int f[N][M], a[N], n, m;

void init() {
    for (int j = 0; j < M; j++) {
        for (int i = 1; i + (1<<j) - 1 <= n; i++) {
            if (j == 0) f[i][j] = a[i];
            // i ~ i+(1<<j-1)-1; i+(i<<j-1) ~ i + (i<<j) - 1
            else f[i][j] = max(f[i][j-1], f[i+(1<<j-1)][j-1]);
        }
    }
}

int query(int l, int r) {
    int k = log2(r-l+1);
    return max(f[l][k], f[r-(1<<k)+1][k]);
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
    init();
    while (m--) {
        int l, r;
        scanf("%d%d", &l, &r);
        printf("%d\n", query(l, r));
    }
    return 0;
}
```

## 互质子区间个数

> 给定长度为 $n$ 的正整数数列 $a$，定义 $f(l,r)$ 为：
> $$
> f(l,r)=\begin{cases}
> 1,\gcd(a_l,a_{l+1},\cdots,a_r)=1\\
> 0,\text{otherwise}
> \end{cases}
> $$
> 求出给定数列的所有子区间的权值之和，即：
> $$
> \sum_{x=1}^n\sum_{y=x}^n f(x,y)
> $$
> 数据范围：$1\le n\le 2\times10^5$，$1\le a_i\le 10^5$。

这题目是某天模拟赛的题，没在 OJ 上找到。

首先，对于区间 $\gcd$ 有一个非严格单调递减的性质，这个很容易证明：

1. 首先 $\gcd(a,b)\le \min(a,b)$，显然是成立的。
2. 此运算具有结合律，即 $\gcd(a,b,c)=\gcd(\gcd(a,b),c)$，因此区间内加入数字不会使 $\gcd$ 更大。

对于一个固定的左边界 $l$，我们要找到第一个 $r$，使得 $f(l,r)=1$，这样就是：
$$
f(l,x)=\begin{cases}
1,x\ge r\\
0,x\lt r
\end{cases}
$$
通过二分就能找到这个边界。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 200010;
int n, st[N][19], lg2[N];

int gcd(int a, int b) {
    if (!b) return a;
    return gcd(b, a % b);
}

int query(int l, int r) {
    if (r == n+1) return 1;
    int k = lg2[r-l+1];
    return gcd(st[l][k], st[r-(1<<k)+1][k]);
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &st[i][0]);
    for (int i = 2; i <= n; i++) lg2[i] = lg2[i>>1] + 1;

    for (int k = 1; k <= 18; k++)
        for (int i = 1; i + (1 << k) - 1 <= n; i++)
            st[i][k] = gcd(st[i][k-1], st[i+(1<<(k-1))][k-1]);
    
    LL res = 0;
    for (int x = 1; x <= n; x++) {
        int l = x, r = n+1;
        while (l < r) {
            int mid = (l+r) >> 1;
            if (query(x, mid) == 1) r = mid;
            else l = mid + 1;
        }
        res += n-r+1;
    }

    printf("%lld\n", res);
    return 0;
}
```

这里注意一下，可能存在 $f(l,n)=0$ 的情况，所以二分右边界给一个 $n+1$，如果用别的数据结构可以给 $a_{n+1}$ 设一个很大的质数，使得 $f(l,n+1)=1$，这样才能正确统计到答案。

## [USACO07JAN] Balanced Lineup G

> 每天,农夫 John 的 $n(1\le n\le 5\times 10^4)$ 头牛总是按同一序列排队。
>
> 有一天, John 决定让一些牛们玩一场飞盘比赛。他准备找一群在队列中位置连续的牛来进行比赛。但是为了避免水平悬殊，牛的身高不应该相差太大。John 准备了 $q(1\le q\le 1.8\times10^5)$ 个可能的牛的选择和所有牛的身高 $h_i(1\le h_i\le 10^6,1\le i\le n)$。他想知道每一组里面最高和最低的牛的身高差。
>
> **输入格式**
>
> 第一行两个数 $n,q$。
>
> 接下来 $n$ 行，每行一个数 $h_i$。
>
> 再接下来 $q$ 行，每行两个整数 $a$ 和 $b$，表示询问第 $a$ 头牛到第 $b$ 头牛里的最高和最低的牛的身高差。
>
> **输出格式**
>
> 输出共 $q$ 行，对于每一组询问，输出每一组中最高和最低的牛的身高差。
>
> 题目链接：[P2880](https://www.luogu.com.cn/problem/P2880)。

建两个 ST 表，求一下区间最大值和最小值。

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 50010;
int f[N][16], g[N][16];
int a[N], n, q;

void init() {
    for (int j = 0; j < 16; j++) {
        for (int i = 1; i+(1<<j)-1 <= n; i++) {
            if (!j) f[i][j] = g[i][j] = a[i];
            else
                f[i][j] = max(f[i][j-1], f[i+(1<<(j-1))][j-1]),
                g[i][j] = min(g[i][j-1], g[i+(1<<(j-1))][j-1]);
        }
    }
}

int query(int l, int r) {
    int k = log2(r-l+1);
    return max(f[l][k], f[r-(1<<k)+1][k]) - min(g[l][k], g[r-(1<<k)+1][k]);
}

int main() {
    scanf("%d%d", &n, &q);
    for (int i = 1; i <= n; i++)
        scanf("%d", &a[i]);
    
    memset(g, 0x3f, sizeof g);
    init();

    for (int i = 1, l, r; i <= q; i++) {
        scanf("%d%d", &l, &r);
        printf("%d\n", query(l, r));
    }

    return 0;
}
```

## [SCOI2007] 降水量

> 我们常常会说这样的话：“$X$ 年是自 $Y$ 年以来降雨量最多的”。它的含义是 $X$ 年的降雨量不超过 $Y$ 年，且对于任意 $Y < Z < X$，$Z$ 年的降雨量严格小于 $X$ 年。例如 2002、2003、2004 和 2005 年的降雨量分别为 $4920$、$5901$、$2832$ 和 $3890$，则可以说“2005 年是自 2003 年以来最多的”，但不能说“2005 年是自 2002 年以来最多的”由于有些年份的降雨量未知，有的说法是可能正确也可以不正确的。
>
> **输入格式**
>
> 输入仅一行包含一个正整数 $n$，为已知的数据。以下 $n$ 行每行两个整数 $y_i$ 和 $r_i$，为年份和降雨量，按照年份从小到大排列，即 $y_i<y_{i+1}$。下一行包含一个正整数 $m$，为询问的次数。以下 $m$ 行每行包含两个数 $Y$ 和 $X$，即询问“$X$ 年是自 $Y$ 年以来降雨量最多的。”这句话是“必真”、“必假”还是“有可能”。
>
> **输出格式**
>
> 对于每一个询问，输出 `true`、`false` 或者 `maybe`。
>
> $100 \%$ 的数据满足：$1 \le n \le 50000$，$1 \le m \le 10000$，$-10^9 \le y_i \le 10^9$，$1 \le r_i \le 10^9$，$-10^9 \le X, Y \le 10^9$。
>
> 数据保证 $Y < X$。
>
> 题目链接：[P2471](https://www.luogu.com.cn/problem/P2471)。

这题挺恶心人的，首先值域较大需离散化，其次需要判断很多条件。

1. 如果 $H_X>H_Y$ 那么是 `false`。
2. 如果 $\exists H_Z\ge H_X \text{ or } H_Y$ 那么是 `false`。
3. 如果 $Y\sim X$ 之间存在年份降水量未知返回 `maybe`。
4. 其它的是返回 `true`。

对于第二个情况我们可以用 ST 表解决。

```cpp
#include <cmath>
#include <vector>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

#define fi first
#define se second
typedef pair<int, int> PII;
const int N = 70010, M = 10010, INF = 0x3f3f3f3f;
int n, m, h[N], f[N][16];
vector<int> nums;
PII dat[N], qry[M];
int L[N], R[N], k;

void init() {
    for (int j = 1; j < 16; j++)
        for (int i = 1; i+(1<<j)-1 <= nums.size(); i++)
            f[i][j] = max(f[i][j-1], f[i+(1<<(j-1))][j-1]);
}

int query(int f[][16], int l, int r) {
    int k = log2(r-l+1);
    return max(f[l][k], f[r-(1<<k)+1][k]);
}

int find(int x) {
    return lower_bound(nums.begin(), nums.end(), x) - nums.begin() + 1;
}

int getrange(int x) {
    int p = upper_bound(L, L+k, x) - L - 1;
    if (p < 0 || p >= k) return INF;
    if (L[p] <= x && x <= R[p]) return p;
    return INF;
}

void solve(int y, int x) {
    int b = nums[y-1], a = nums[x-1];
    if (f[x][0] > h[y]) return puts("false"), void();

    if (y+1 == x) {
        // 判断是否处在同一区间中
        if (getrange(b) == getrange(a) && h[x] < INF)
            return puts("true"), void();
        return puts("maybe"), void();
    }
    
    int z = query(f, y+1, x-1);
    if (z >= h[x] || z >= h[y]) return puts("false"), void();

    if (h[x] == INF || h[y] == INF) return puts("maybe"), void();
    // 判断是否处在同一区间中
    if (getrange(b) != getrange(a)) return puts("maybe"), void();

    return puts("true"), void();
}

int main() {
    memset(h, 0x3f, sizeof h);
    scanf("%d", &n);
    for (int i = 1; i <= n; i++)
        scanf("%d%d", &dat[i].fi, &dat[i].se),
        nums.push_back(dat[i].fi);

    for (int i = 1; i <= n; i++) {
        int j = i+1;
        while (j <= n && dat[j].fi == dat[i].fi + j - i) j++;
        L[k] = dat[i].fi, R[k] = dat[j-1].fi;
        i = j-1;
        k++;
    }

    scanf("%d", &m);
    for (int i = 1; i <= m; i++)
        scanf("%d%d", &qry[i].fi, &qry[i].se),
        nums.push_back(qry[i].fi),
        nums.push_back(qry[i].se);
    
    sort(nums.begin(), nums.end());
    nums.erase(unique(nums.begin(), nums.end()), nums.end());

    for (int i = 1; i <= n; i++) {
        int idx = find(dat[i].fi);
        f[idx][0] = h[idx] = dat[i].se;
    }
    init();
    for (int i = 1; i <= m; i++) {
        int y = find(qry[i].fi), x = find(qry[i].se);
        solve(y, x);
    }
    return 0;
}
```

## [CF1878E] Iva & Pav

> 给定长度为 $n$ 的数组 $a$，定义 $f(l,r)$ 是 $[l,r]$ 的区间按位和，给定 $q$ 个查询，每个查询包含 $k,l$ 要求找到最大的 $r$ 使得 $f(l,r)\ge k$，如果不存在输出 $-1$。
>
> 数据范围：$1\le n\le 2\times 10^5, 1\le a_i \le 10^9,1\le q\le 10^5, 1\le l\le n,1\le k\le 10^9$。
>
> 题目链接：[CF1878E](https://codeforces.com/contest/1878/problem/E)。

区间按位与和或运算都是重复贡献问题，可以用 ST 表解决，本题的具体思路参考本站的拆位文章，这里给出查询和初始化的模板。

```cpp
void init() {
    for (int j = 1; j < 18; j++)
        for (int i = 1; i+(1<<j)-1 <= n; i++)
            st[i][j] = st[i][j-1] & st[i+(1<<(j-1))][j-1];
}

int query(int l, int r) {
    int k = log2(r-l+1);
    return st[l][k] & st[r-(1<<k)+1][k];
}
```

