---
date: 2023-10-31 15:54:00
title: 数据结构 - 单调栈
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
description: 单调栈是一种在线性复杂度内求解「左右第一个比某个数大」的元素的方式，与单调队列类似，都是将不可能作为答案的元素去掉来进行优化的思想。
---

## 最大全零子矩形面积

> 第一行两个整数 $N$，$M$，表示矩形土地有 $N$ 行 $M$ 列。
>
> 接下来 $N$ 行，每行 $M$ 个用空格隔开的字符 $\texttt{F}$ 或 $\texttt{R}$，描述了矩形土地。
>
> 输出一个整数，表示 $3S$ 的值，其中 $S$ 代表最大全 $\texttt{F}$ 子矩形面积。
>
> **数据范围**
>
> 对于 $100\%$ 的数据，$1 \leq N, M \leq 1000$。
>
> 题目链接：[P4147](https://www.luogu.com.cn/problem/P4147)。

首先预处理 $h(i,j)$ 表示从当前格子向上最大伸展多少。

考虑枚举一个底边，到每个点时能构成的极大的子矩形应当是 $h(i,j)$ 尽可能向左或向右延伸，所以要找到第一个小于 $h(i,j)$ 的左边和右边的下标，考虑使用单调栈。

单调栈的思想就是，如果有一个元素比原来的靠的更近，并且更优，那么原来那个元素就没用了。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1010;
int n, m, h[N][N], L[N][N], R[N][N];

int main() {
    cin >> n >> m;
    char ch;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            cin >> ch;
            if (ch == 'R') continue;
            h[i][j] = h[i-1][j] + 1;
        }
    }

    int ans = 0;
    stack<int> stk;

    for (int i = 1; i <= n; i++) {
        stack<int>().swap(stk);
        for (int j = 1; j <= m; j++) {
            while (stk.size() && h[i][stk.top()] >= h[i][j]) stk.pop();
            L[i][j] = stk.size() ? stk.top() : 0;
            stk.push(j);
        }

        stack<int>().swap(stk);
        for (int j = m; j >= 1; j--) {
            while (stk.size() && h[i][stk.top()] >= h[i][j]) stk.pop();
            R[i][j] = stk.size() ? stk.top() : m+1;
            stk.push(j);
        }
    }

    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            ans = max(ans, h[i][j] * (R[i][j] - L[i][j] - 1));

    return !printf("%d\n", 3*ans);
}
```

## 全零子矩形计数

> 给定一个 $n\times m$ 的字符矩形，求全为 `.` 的子矩形的个数。
>
> **输入格式**
>
> 第一行两个正整数 $n,m$，表示这张纸的长度和宽度。
>
> 接下来有 $n$ 行，每行 $m$ 个字符，每个字符为 `*` 或者 `.`。
>
> 字符 `*` 表示以前在这个格子上画过，字符 `.` 表示以前在这个格子上没画过。
>
> **输出格式**
>
> 仅一个整数，表示方案数。
>
> **数据范围**
>
> 对 $100\%$ 的数据，满足 $1\leq n, m\leq 1000$。
>
> 题目链接：[P1950](https://www.luogu.com.cn/problem/P1950)。

考虑枚举一个底边，然后枚举当前列 $j$，用 $h_{ij}$ 表示在 $(i,j)$ 向上能拓展的最大距离。

钦定必选当前列，并且以当前底边作为矩形的底边，这样统计出来一定不漏。

接下来考虑如何统计才能保证不重。用当前 $h_{ij}$ 向左右拓展，左边遇到比它小的就停止，右边遇到比它小或者等于的就停止，如下图：

![](/img/2023/11/p1950.jpg)

第一个矩形拓展的地方，钦定必须选择第一列，那么就不会和第二个矩形统计出来的出现重复。那半闭半开区间的目的就是防止相同的 $h_{ij}$，如果相同，那么左边的那个一定不会包含到右边那个和它等高的矩形，所以这样不会重复。

用单调栈找到左边第一个比它小的，用 $L_{ij}$ 表示；右边第一个小于等于它的，用 $R_{ij}$ 表示。

然后统计方案直接 $h_{ij}(j-L_{ij})(R_{ij}-j)$ 即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 1010;
char g[N][N];
int n, m, h[N][N], L[N][N], R[N][N];

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%s", g[i] + 1);
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++) {
            if (g[i][j] == '*') continue;
            h[i][j] = h[i-1][j] + 1;
        }
    
    long long res = 0;
    stack<int> stk;
    for (int i = 1; i <= n; i++) {
        stack<int>().swap(stk);
        for (int j = 1; j <= m; j++) {
            while (stk.size() && h[i][j] <= h[i][stk.top()]) stk.pop();
            L[i][j] = stk.size() ? stk.top() : 0;
            stk.push(j);
        }

        stack<int>().swap(stk);
        for (int j = m; j; j--) {
            while (stk.size() && h[i][j] < h[i][stk.top()]) stk.pop();
            R[i][j] = stk.size() ? stk.top() : m+1;
            stk.push(j);
        }

        for (int j = 1; j <= m; j++) {
            res += 1LL * h[i][j] * (R[i][j] - j) * (j - L[i][j]);
        }
    }
    return !printf("%lld\n", res);
}
```

## 美丽的序列

> 给定序列 $a_1,a_2,\dots,a_n$，定义区间 $[l,r]$ 的 “美丽度” 是 $\min\{a_l,\dots,a_r\}\times (r-l+1)$ 求所有子区间的美丽度的最大值。
>
> 题目链接：[P2659](https://www.luogu.com.cn/problem/P2659)。

这个函数当区间长度增长的时候，区间最小值是单调不升的，所以如果我们固定最小值为 $a_i$，那么它能达到的最大区间长度就是这个最小值对应的最大美丽度。单调栈即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 2000010;
stack<int> stk;
int n, a[N], L[N], R[N];

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
    for (int i = 1; i <= n; i++) {
        while (stk.size() && a[stk.top()] >= a[i]) stk.pop();
        L[i] = stk.size() ? stk.top() : 0;
        stk.push(i);
    }

    stack<int>().swap(stk);
    for (int i = n; i >= 1; i--) {
        while (stk.size() && a[stk.top()] >= a[i]) stk.pop();
        R[i] = stk.size() ? stk.top() : n+1;
        stk.push(i);
    }

    long long ans = 0;
    for (int i = 1; i <= n; i++) ans = max(ans, 1LL * (R[i] - L[i] - 1) * a[i]);
    return !printf("%lld\n", ans);
}
```

## [eJOI2020] Fountain

> 有一个喷泉由 $N$ 个圆盘组成，从上到下以此编号为 $1 \sim N$，第 $i$ 个喷泉的直径为 $D_i$，容量为 $C_i$，当一个圆盘里的水大于了这个圆盘的容量，那么水就会溢出往下流，直到流入半径大于这个圆盘的圆盘里。如果下面没有满足要求的圆盘，水就会流到喷泉下的水池里。
>
> 现在给定 $Q$ 组询问，每一组询问这么描述：
>
> - 向第 $R_i$ 个圆盘里倒入 $V_i$ 的水，求水最后会流到哪一个圆盘停止。
>
> 如果最终流入了水池里，那么输出 $0$。
>
> **注意，每个询问互不影响。**
>
> **输入格式**
>
> 第一行两个整数 $N,Q$ 代表圆盘数和询问数。
>
> 接下来 $N$ 行每行两个整数 $D_i,C_i$ 代表一个圆盘。
>
> 接下来 $Q$ 行每行两个整数 $R_i,V_i$ 代表一个询问。
>
> **输入格式**
>
> $Q$ 行每行一个整数代表询问的答案。
>
> 对于 $100\%$ 的数据：$2 \le N \le 10^5$，$1 \le Q \le 2 \times 10^5$，$1 \le C_i \le 1000$，$1 \le D_i,V_i \le 10^9$，$1\le R_i \le N$。
>
> 题目链接：[P7167](https://www.luogu.com.cn/problem/P7167)。

考虑到一个圆盘最近的要么只有一个，要么是地面，所以这就是一棵树，让地面成为树根就行。

所以先用单调栈建树，然后每次询问就是不断查询连续路径和首次超过给定值的那个结点。

考虑倍增优化，用 $sum(u,i)$ 表示向上走区间 $(u,u+2^i]$ 内的和（这里加法意思是向上走），这里左开右闭的原因是连续端点相交的区间求和必须是半开半闭比较好维护。

然后我们从大到小枚举幂指数，最终如果还剩多余的水就再给下一层（跳一次父亲），否则就不跳。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 100010, INF = 0x3f3f3f3f;
int n, q;
int d[N], c[N], pre[N][17];
LL sum[N][17];

int query(int k, int tot) {
    tot -= c[k];

    for (int i = 16; i >= 0; i--) {
        if (tot >= sum[k][i]) {
            tot -= sum[k][i];
            k = pre[k][i];
        }
    }

    if (tot > 0) k = pre[k][0];
    return k;
}

int main() {
    scanf("%d%d", &n, &q);
    d[0] = c[0] = INF;
    for (int i = n; i >= 1; i--) scanf("%d%d", &d[i], &c[i]);
    stack<int> stk;

    for (int k = 0; k <= 16; k++) sum[0][k] = INF;

    for (int i = 1; i <= n; i++) {
        while (stk.size() && d[stk.top()] <= d[i]) stk.pop();        
        if (stk.size()) pre[i][0] = stk.top();
        stk.push(i);

        sum[i][0] = c[pre[i][0]];
        for (int k = 1; k <= 16; k++) {
            pre[i][k] = pre[pre[i][k-1]][k-1];
            sum[i][k] = sum[i][k-1] + sum[pre[i][k-1]][k-1];
        }
    }

    for (int i = 1, k, v; i <= q; i++) {
        scanf("%d%d", &k, &v);
        k = n-k+1;
        k = query(k, v);
        if (!k) puts("0");
        else printf("%d\n", n-k+1);
    }

    return 0;
}
```

