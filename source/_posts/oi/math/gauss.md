---
date: 2023-08-02 12:11:00
update: 2024-02-03 17:53:00
title: 数学 - 高斯消元
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
description: 高斯消元用来求解 n 个 n 元线性方程组，复杂度为 O(n^3)，通常是解决一些存在环的 DP 问题。
---

## 简介

高斯消元是在 $O(n^3)$ 复杂度求解 $n$ 个 $n$ 元线性方程组的算法，通常应用在一些概率问题中，因为递推关系中存在环，所以是只能列出相关方程，然后用高斯解方程组，但是一般来说还是需要观察特殊性质来进行复杂度更优秀的 DP 操作，但是高消也是一个得部分分的工具。

简单地说，就是通过不断的联立两个方程消掉一个未知数，然后求出最后一个未知数的解，不断往回代入，求出每个未知数。

## [模板] 线性方程组

> 给定 $n$ 个线性方程组 $\sum_{j=1}^n a_{ij}x_j=b_i$，若无解输出 $-1$，无穷多解输出 $0$​，唯一解输出每个解。
>
> 数据范围：$1\le n\le 50$，$|a_{ij}|\le 100$，$|b_i|\le 300$。
>
> 题目链接：[P2455](https://www.luogu.com.cn/problem/P2455)。

为了保证精度，我们找到当前主元系数绝对值最大的那个，将它与当前行交换。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 110;
const double eps = 1e-8;
int n;
double a[N][N];

int gauss() {
    int p = 1, r = 1;
    for (; p <= n; p++) {
        int target = r;
        for (int i = r+1; i <= n; i++) if (fabs(a[i][p]) >= fabs(a[target][p])) target = i;
        if (fabs(a[target][p]) < eps) continue;
        for (int j = p; j <= n+1; j++) swap(a[r][j], a[target][j]);
        for (int j = n+1; j >= p; j--) a[r][j] /= a[r][p];
        for (int i = r+1; i <= n; i++) {
            if (fabs(a[i][p]) < eps) continue;
            for (int j = n+1; j >= p; j--) a[i][j] -= a[r][j] * a[i][p];
        }
        r++;
    }

    if (r <= n) {
        for (int i = r; i <= n; i++) if (fabs(a[i][n+1]) > eps) return -1;
        return 0;
    }

    for (int i = n; i; i--)
        for (int j = i+1; j <= n; j++)
            a[i][n+1] -= a[j][n+1] * a[i][j];

    return 1;
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) for (int j = 1; j <= n+1; j++) scanf("%lf", &a[i][j]);
    int res = gauss();
    if (res != 1) printf("%d\n", res);
    else for (int i = 1; i <= n; i++) printf("x%d=%.2lf\n", i, a[i][n+1]);
    return 0;
}
```

## [模板] 矩阵求逆

> 求 $n\times n$ 的逆矩阵，对 $10^9+7$ 取模，无解输出 `No Solution`。
>
> 数据范围：$1\le n\le 400$，$0\le a_{ij}\lt 10^9+7$。
>
> 题目链接：[P4783](https://www.luogu.com.cn/problem/P4783)。

本质上，我们对一个方程组进行的都是初等行列变换，可以用一个矩阵乘法来表示这种变换。

对于一个矩阵，如果我们对它进行初等行列变换，最终得到一个单位阵 $E$，那么这些变换矩阵的积就是原矩阵的逆矩阵，也就是同时对一个单位阵进行完全相同的变换即可得到逆矩阵。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 410, P = 1e9+7;
int n, A[N][N<<1];

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

bool gauss() {
    for (int i = 1; i <= n; i++) A[i][i+n] = 1;

    for (int r = 1; r <= n; r++) {
        int target = -1;
        for (int i = r; i <= n; i++) {
            if (A[i][r]) {
                target = i;
                break;
            }
        }
        if (target == -1) return false;
        swap(A[r], A[target]);

        int inv = qmi(A[r][r], P-2);
        for (int i = r; i <= n*2; i++) A[r][i] = A[r][i] * inv % P;

        for (int i = r+1; i <= n; i++) {
            for (int j = n*2; j >= r; j--) {
                A[i][j] = (A[i][j] - A[r][j] * A[i][r]) % P;
                if (A[i][j] < 0) A[i][j] += P;
            }
        }
    }

    for (int i = n; i; i--) {
        for (int j = i-1; j; j--) {
            for (int k = n*2; k >= i; k--) {
                A[j][k] = (A[j][k] - A[i][k] * A[j][i]) % P;
                if (A[j][k] < 0) A[j][k] += P;
            }
        }
    }
    return true;
}

signed main() {
    scanf("%lld", &n);
    for (int i = 1; i <= n; i++) for (int j = 1; j <= n; j++) scanf("%lld", &A[i][j]);
    if (!gauss()) puts("No Solution");
    else for (int i = 1; i <= n; i++) for (int j = 1; j <= n; j++) printf("%lld%c", A[i][j+n], " \n"[j == n]);
    return 0;
}
```

## [JSOI2008] 球形空间产生器

> 在 $n$ 维空间的球面上给定 $n+1$ 个点的坐标，求球心坐标。数据保证唯一解。$1\le n\le 10$，$|a_{ij}|,|b_{ij}|\le 20000$。
>
> 题目链接：[P4035](https://www.luogu.com.cn/problem/P4035)。

设球心坐标 $(x_1,x_2,\cdots,x_n)$ 列出这 $n+1$ 个点的方程：
$$
\left\{
\begin{aligned}
&(x_1-a_{11})^2+\cdots+(x_n-a_{1n})^2=R^2\\
&(x_1-a_{21})^2+\cdots+(x_n-a_{2n})^2=R^2\\
&\vdots\\
&(x_1-a_{n+1,1})^2+\cdots+(x_n-a_{n+1,n})^2=R^2
\end{aligned}
\right.
$$
但这里是一个二次方程组，高斯消元能解决的是一次方程组，所以这里用每一个式子减去上一个式子，得到 $n$ 个线性方程。用 2 式减 1 式为例：
$$
\sum_{i=1}^n2(a_{2i}-a_{1i})x_i = \sum_{i=1}^n(a_{2i}^2-a_{1i}^2)\\
$$
这样就能构造出 $n$ 个线性方程组了。

```cpp
#include <cstdio>
#include <cmath>
#include <algorithm>
using namespace std;

const int N = 15;
int n;
double a[N][N];

void gauss() {
    for (int r = 0; r < n; r++) {
        int t = r;
        for (int i = r; i < n; i++)
            if (fabs(a[i][r]) > fabs(a[r][r]))
                t = i;
        
        for (int j = r; j <= n; j++)
            swap(a[r][j], a[t][j]);
        
        // 归一
        for (int j = n; j >= r; j--)
            a[r][j] /= a[r][r];
        
        // 消为 0
        for (int i = r+1; i < n; i++)
            for (int j = n; j >= r; j--)
                a[i][j] -= a[r][j] * a[i][r];
    }
    
    // 消去上三角
    for (int i = n-1; i >= 0; i--)
        for (int j = i+1; j < n; j++)
            a[i][n] -= a[i][j] * a[j][n];
}

int main() {
    scanf("%d", &n);
    for (int i = 0; i < n+1; i++)
        for (int j = 0; j < n; j++)
            scanf("%lf", &a[i][j]);
    
    // 预处理系数
    for (int i = 0; i < n; i++) {
        double b = 0;
        for (int j = 0; j < n; j++) {
            b += a[i+1][j]*a[i+1][j]-a[i][j]*a[i][j];
            a[i][j] = 2*(a[i+1][j]-a[i][j]);
        }
        a[i][n] = b;
    }
    gauss();
    for (int i = 0; i < n; i++)
        printf("%.3lf ", a[i][n]);
    return 0;
}
```

## [HNOI2013] 游走

> 给定 $n$ 点 $m$ 边无向连通图，顶点编号 $1\sim n$，边编号为 $1\sim m$。
>
> 随机游走，走到每个边获得与其编号相等的分数，求一个编号方式使得总分的期望最小，输出最小期望分值。
>
> 数据范围：$2\le n\le 500$，$1\le m\le 125000$，无重边和自环。
>
> 题目链接：[P3232](https://www.luogu.com.cn/problem/P3232)。

设 $f_u$ 为 $u$ 期望经过次数，$s_u$ 为 $u$ 连接的点数，那么边 $(u,v)$ 的期望经过次数为：
$$
E(u,v)=\frac{f_u}{s_u}+\frac{f_v}{s_v}
$$
将 $E(u,v)$ 排序，给最大的安排最小的编号，通过调整法可以证明这样是最优的，列 $f_u$ 的方程：
$$
f_u=\begin{cases}
\sum_{(u,v)\in E,v\ne n}\frac{f_v}{s_v}+1\quad u=1\\
\sum_{(u,v)\in E,v\ne n} \frac{f_v}{s_v}\quad u\ne 1\\
\end{cases}
$$
然后通过高斯消元就可以解了，本题精度要求高，因此高斯消元不能直接将本行除以主元，要最后处理。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define eb emplace_back
#define fi first
#define se second
typedef pair<int, int> PII;
const int N = 510;
const double eps = 1e-8;
vector<double> vals;
vector<int> g[N];
vector<PII> edge;
int n, m;
double a[N][N], f[N];

void gauss() {
    int p = 1, r = 1;
    for (; p <= n; p++) {
        int target = r;
        for (int i = r+1; i <= n; i++) if (fabs(a[i][p]) >= fabs(a[target][p])) target = i;
        assert(fabs(a[target][p]) > eps);
        swap(a[r], a[target]);
        // for (int j = n+1; j >= p; j--) a[r][j] /= a[r][p];
        for (int i = r+1; i <= n; i++) {
            double k = a[i][p] / a[r][p];
            for (int j = n+1; j >= p; j--) a[i][j] -= a[r][j] * k;
        }
        r++;
    }

    for (int i = n; i; i--) {
        for (int j = i+1; j <= n; j++) a[i][n+1] -= f[j] * a[i][j];
        f[i] = a[i][n+1] / a[i][i];
    }
}

double calc(const PII& p) {
    return f[p.fi] / g[p.fi].size() + f[p.se] / g[p.se].size();
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1, u, v; i <= m; i++) scanf("%d%d", &u, &v), g[u].eb(v), g[v].eb(u), edge.eb(u, v);
    for (int u = 1; u < n; u++) {
        for (int v: g[u]) if (v != n) a[u][v] = 1. / g[v].size();
        a[u][u] = -1;
    }
    a[1][n] = -1;
    --n;
    gauss();

    double ans = 0;
    for (const PII& e: edge) vals.eb(calc(e));
    sort(vals.begin(), vals.end(), greater<double>());
    for (int i = 1; i <= m; i++) ans += vals[i-1] * i;
    return !printf("%.3lf\n", ans);
}
```

