---
title: 搜索 - 二分和三分
katex: true
date: 2023-06-13 12:00:00
updated: 2023-09-12 19:49:00
tags:
- Algo
- C++
- Search
categories:
- OI
---

## 二分模板

寻找左边界用第一个，寻找右边界用第二个。

```cpp
int binary1(int l, int r) {
    while (l < r) {
        int mid = l+r>>1;
        if (check(mid)) l = mid+1;
        else r = mid;
    }
    return l;
}

int binary2(int l, int r) {
    while (l < r) {
        int mid = l+r+1>>1;
        if (check(mid)) r = mid-1;
        else l = mid;
    }
    return l;
}
```

有区别的原因就是为了防止死循环：如果第二个模板用 `l+r>>1`，当 `r=l+1` 时会进入死循环。

下面是浮点二分，就简单多了。

```cpp
double binary(double l, double r) {
    // 取决于题目要求的精度
    while (r - l > 1e-6) {
        double mid = (l+r)/2;
        if (check(mid)) l = mid;
        else r = mid;
    }
    return r;
}
```



## 三次方根

> 给定一个浮点数 $n \in [-10^5, 10^5]$，求它的三次方根（保留 6 位小数）。

二分试根法。

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    double n, l = -1e5, r=1e5;
    scanf("%lf", &n);
    
    while (r - l > 1e-7) {
        double mid = (l + r) / 2;
        if (mid * mid * mid < n) l = mid;
        else r = mid;
    }
    
    printf("%.6f\n", l);
    return 0;
}
```

## 数的范围

>  给定一个按照升序排列的长度为 $n \in [1, 10^5]$ 的整数数组，以及 $q \in [1, 10^4]$ 个查询。
>
>  对于每个查询，返回一个元素 $k \in [1,10^4]$ 的起始位置和终止位置（位置从 0 开始计数）。
>
>  如果数组中不存在该元素，则返回 `-1 -1`。

也就是实现 `lower_bound` 和 `upper_bound`，比较简单，练习一下。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 100010;
int a[N], n, q, k;

int main() {
    scanf("%d%d", &n, &q);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
    while (q--) {
        scanf("%d", &k);
        
        int l = 1, r = n;
        while (l < r) {
            int mid = l+r >> 1;
            if (a[mid] >= k) r = mid;
            else l = mid+1;
        }
        
        if (a[l] != k) {
            puts("-1 -1");
            continue;
        }
        
        printf("%d ", r-1);
        l = 1, r = n;
        while (l < r) {
            int mid = l+r+1 >> 1;
            if (a[mid] <= k) l = mid;
            else r = mid-1;
        }
        printf("%d\n", r-1);
    }
    return 0;
}
```

## [CCC2021 S3] Lunch Concert

> 有 $N$ 个人，第 $i$ 个人的速度为 $W_i$ **秒每米**，听力为 $D_i$，即能听见距离他不超过 $D_i$ 米处的音乐，初始在 $P_i$ 位置。
>
> 你要在 $c$ 位置处开音乐会，这个 $c$ 由你决定且为整数。这 $N$ 个人都会靠近你直到能听到你。你要最小化每个人移动的时间之和。
>
> 题目链接：[P9025](https://www.luogu.com.cn/problem/P9025)，[AcWing 5201](https://www.acwing.com/problem/content/5204/)。

对于每个人，都可以列出来一个时间的函数，设 $x$ 为选定的位置，那么有时间 $f_i(x)$ 为：
$$
f_i(x)=\begin{cases}
W_i(P_i-D_i-x),\quad &x\lt P_i-D_i\\
0,\quad &P_i-D_i\le x\le P_i+D_i\\
W_i[x-(P_i+D_i)],\quad &x\gt P_i+D_i\\
\end{cases}
$$
明显这是一个下凸函数，如果 $f''(x) \ge 0$，那么 $\sum f_i ''(x)\ge 0$ 同样成立，所以多个下凸函数的和仍是下凸函数。虽然 $f_i'(x)$ 并不连续，但是仍然有一个上升的趋势，所以暂时可以这样理解。

对于单峰函数，我们可以用三分求解，具体做法就是取区间 $[l,r]$ 的两个三等分点 $a,b$，若 $f(a)>f(b)$ 说明要么 $a,b$ 同时在峰左边，要么 $a$ 左 $b$ 右，那么都可以舍掉 $a$ 左边的点，反之亦然。

由于我们是对整数三分，所以最好最终留一个区间，然后遍历这个小区间求最值。

初始边界的确定方法：观察所有人的数据范围都是 $0\sim 10^9$，因此如果 $l<0$ 向右移动 $l$ 一定能缩短距离所有人的距离，反之亦然，所以 $l,r=0,10^9$

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 200010;
LL p[N], w[N], d[N];
int n;

LL f(int x) {
    LL cost = 0;
    for (int i = 1; i <= n; i++) {
        if (x < p[i] - d[i]) cost += w[i]*(p[i]-d[i]-x);
        else if (x > p[i] + d[i]) cost += w[i]*(x-(p[i]+d[i]));
    }
    return cost;
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%lld%lld%lld", &p[i], &w[i], &d[i]);
    
    LL l = 0, r = 1e9;
    while (r-l > 10) {
        LL a = l + (r-l) / 3, b = r - (r-l) / 3;
        if (f(a) > f(b)) l = a;
        else r = b;
    }
    
    LL res = 1e18;
    for (int i = l; i <= r; i++) res = min(res, f(i));
    printf("%lld\n", res);
    return 0;
}
```

