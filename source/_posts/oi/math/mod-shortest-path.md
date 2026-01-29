---
date: 2023-10-27 16:30:00
updated: 2023-10-27 16:30:00
title: 数学 - 同余最短路
katex: true
tags:
- Algo
- C++
- Math
- Graph
categories:
- OI
description: 同余最短路是用最短路的方式求解「用若干个整数凑出其它整数」这一问题，可以看做类似于背包问题的一种 DP，但是需要用到最短路求解。
---

## 简介

同余最短路是用最短路的方式求解「用若干个整数凑出其它整数」这一问题，可以看做类似于背包问题的一种 DP，但是需要用到最短路求解。

它的思想比较神奇，设我们有的数列是 $a$，我们用 $a_1$ 当模数，$f(i)$ 表示满足 $x \bmod a_1= i$ 的最小的 $x$。假设我们已经知道整个 $f$ 数组了，那么判断一个数 $k$ 能否被凑出来，就看 $f(k\bmod a_1) \le k$，如果是的话用 $f(k\bmod a_1)$ 加上若干倍的 $a_1$ 就一定能凑出来 $k$，反之凑不出来。

这个状态转移应该这么进行：
$$
f(i)=\min_{i=1}^n \{f((j+a_i)\bmod a_1) + a_i\}
$$
满足一个最短路的形式，所以可以用最短路求解。

## [POI2003] Sums

> 我们给定一个整数集合 $A$。考虑一个非负整数集合 $A'$，所有属于 $A'$ 的集合的数 $x$ 满足当且仅当能被表示成一些属于 $A$ 的元素的和（数字可重复）。
>
> 比如，当 $A = \{2,5,7\}$，属于 $A'$ 的数为：$0$（$0$ 个元素的和），$2$，$4$（$2 + 2$）和 $12$（$5 + 7$ or $7 + 5$ or $2 + 2 + 2 + 2 + 2 + 2$）；但是元素 $1$ 和 $3$ 不属于 $A'$。
>
> **输入格式**
>
> 第一行有一个整数 $n$，代表集合 $A$ 的元素个数。接下来每行一个数 $a_i$ 描述一个元素。$A = \{a_1,a_2,...,a_n\}$。
>
> 接下来一个整数 $k$，然后每行一个整数，分别代表 $b_1,b_2,...,b_k$。
>
> **输出格式**
>
> 输出 $k$ 行。如果 $b_i$ 属于 $A'$，第 $i$ 行打印 `TAK`，否则打印 `NIE`。
>
> 对于所有数据，$1 \le n \le 5 \times 10^3$，$1 \le k \le 10^4$，$1 \le a_1 < a_2 < ... < a_n \le 5 \times 10^4$，$0 \le b_i \le 10^9$。
>
> 题目链接：[P8060](https://www.luogu.com.cn/problem/P8060)。

和前文说的一样，跑这个最短路的时候不用真的把图建出来，直接枚举 $a_i$ 转移即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 50010;
int n, k, a[N], dist[N];

typedef pair<int, int> PII;

void dijkstra() {
    bitset<N> st;
    memset(dist, 0x3f, sizeof dist);
    priority_queue<PII, vector<PII>, greater<PII>> heap;
    dist[0] = 0;
    heap.push({0, 0});
    while (heap.size()) {
        int t = heap.top().second; heap.pop();
        if (st[t]) continue;
        st[t] = true;
        for (int i = 1; i <= n; i++) {
            int to = (t + a[i]) % a[1], w = a[i];
            if (dist[to] > dist[t] + w) {
                dist[to] = dist[t] + w;
                heap.push({dist[to], to});
            }
        }
    }
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
    dijkstra();
    
    scanf("%d", &k);
    for (int i = 1, b; i <= k; i++) {
        scanf("%d", &b);
        if (dist[b % a[1]] > b) puts("NIE");
        else puts("TAK");
    }
    return 0;
}
```

## [国家集训队] 墨墨的等式

> 墨墨突然对等式很感兴趣，他正在研究 $\sum_{i=1}^n a_ix_i=b$ 存在非负整数解的条件，他要求你编写一个程序，给定 $n, a_{1\dots n}, l, r$，求出有多少 $b\in[l,r]$ 可以使等式存在非负整数解。
>
> **输入格式**
>
> 第一行三个整数 $n,l,r$。
>
> 第二行 $n$ 个整数 $a_{1\dots n}$。
>
> **输出格式**
>
> 一行一个整数，表示有多少 $b\in[l,r]$ 可以使等式存在非负整数解。
>
> 对于 $100\%$ 的数据，$n \le 12$，$0 \le a_i \le 5\times 10^5$，$1 \le l \le r \le 10^{12}$。
>
> 题目链接：[P2371](https://www.luogu.com.cn/problem/P2371)。

首先把数列里的 $0$ 全部扔掉，没用。然后，把这个问题转化成求 $0\sim l-1,0\sim r$ 内有多少满足要求的数字，也就是我们只需要判断 $0\sim x$ 中有多少个数字满足要求。

枚举 $i$ 表示模 $a_1$ 后的结果是多少，然后就是判断 $0\sim x$ 中有多少个数字满足 $\text{mod }a_1 = i$，显然是只有 $dist(i)\sim x$ 中的数字是满足要求的，那么问题就转化成了求一个区间内满足 $\text{mod }a_1= i$ 的数字有多少个。

首先讨论一个更一般化的问题，在 $[L,R]$ 中 $\text{mod }p = r$ 的数字有多少个？转化成满足 $L\le kp+r\le R$ 的整数 $k$ 有多少个，接下来我们只需要找到第一个 $k$ 和最后一个 $k$ 就可以了，即：
$$
\lfloor\frac{R-r}{p}\rfloor - \lceil\frac{L-R}{p}\rceil+1
$$
回到本题，也可以套公式：
$$
\lfloor\frac{x-i}{a_1}\rfloor-\lceil\frac{dist(i)-i}{a_1}\rceil+1
$$
由于 $dist(i)$ 的实际意义是 $\text{mod }a_1=i$ 的最小数字，那么中间的式子一定整除，所以还可以进一步化简：
$$
\lfloor\frac{x-i}{a_1}\rfloor-\frac{dist(i)-i}{a_1}+1=\lfloor\frac{x-dist(i)}{a_1}\rfloor+1
$$
但是下面的代码还是用的原始公式。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
typedef pair<LL, int> PII;
const int N = 15, M = 500010;
int a[N], n;
LL dist[M];
LL l, r;

void dijkstra() {
    priority_queue<PII, vector<PII>, greater<PII>> heap;
    bitset<M> st;
    memset(dist, 0x3f, sizeof dist);
    dist[0] = 0;
    heap.push({0, 0});

    while (heap.size()) {
        int t = heap.top().second; heap.pop();
        if (st[t]) continue;
        st[t] = true;

        for (int i = 1; i <= n; i++) {
            int to = (t + a[i]) % a[1], w = a[i];
            if (dist[to] > dist[t] + w) {
                dist[to] = dist[t] + w;
                heap.push({dist[to], to});
            }
        }
    }
}

LL query(LL x) {
    LL res = 0;
    for (int i = 0; i < a[1]; i++)
        if (dist[i] <= x)
            res += (x-i)/a[1] - (dist[i]-i+a[1]-1)/a[1] + 1;
    return res;
}

int main() {
    scanf("%d%lld%lld", &n, &l, &r);

    int m = 0;
    for (int i = 1; i <= n; i++) {
        scanf("%d", &a[++m]);
        if (!a[m]) m--;
    }
    n = m;
    dijkstra();
    return !printf("%lld\n", query(r) - query(l-1));
}
```

## [ABC077D] Small Multiple

> 给定一个正整数 $K$。求 $K$ 的正整数倍的数位累加和最小是多少。$2 \le K \le {10}^5$；
>
> 题目链接：[ABC077D](https://atcoder.jp/contests/abc077/tasks/arc084_b)。
>

由于求 $K$ 的倍数，所以可以考虑求 $\text{mod }K$ 意义下的 $0$ 数位累加和最小，设 $f(x)$ 是 $x \bmod K$ 意义下 $x$ 最小数位累加和。

首先 $f(1)=1$，然后考虑如何进行转移。由于十进制下所有的整数都可以通过 $(+1)(+1)(\times 10)(+1)$ 这种方式转移，考虑 $(\times 10)$ 对数位和的影响是 $0$，$(+1)$ 对数位和的影响是 $1$，那么：
$$
f(i+1)=\min\{f(i)+1\}\\
f(10i)=\min\{f(i)\}
$$
直接写个 01BFS 或者 Dijkstra 都可以。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef pair<int, int> PII;
const int N = 100010;
int k, dist[N];
bool st[N];

int main() {
    scanf("%d", &k);
    priority_queue<PII, vector<PII>, greater<PII>> heap;
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 1;
    heap.push({1, 1});

    while (heap.size()) {
        int t = heap.top().second; heap.pop();
        if (st[t]) continue;
        st[t] = true;

        int to, w;

        to = t * 10 % k, w = 0;
        if (dist[to] > dist[t] + w) {
            dist[to] = dist[t] + w;
            heap.push({dist[to], to});
        }

        to = (t + 1) % k, w = 1;
        if (dist[to] > dist[t] + w) {
            dist[to] = dist[t] + w;
            heap.push({dist[to], to});
        }
    }

    return !printf("%d\n", dist[0]);
}
```

