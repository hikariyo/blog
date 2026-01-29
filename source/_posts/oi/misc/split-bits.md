---
date: 2023-09-25 08:44:00
updated: 2023-09-25 08:44:00
title: 杂项 - 拆位
katex: true
tags:
- Algo
- C++
categories:
- OI
description: 由于位运算的各位都是相互独立的，所以在处理位运算问题的时候可以考虑把每一位单独拆开处理。特别地，对于异或操作，通过前缀和的思想，加上异或的自反性（一个数异或自己结果为零），我们可以在预处理后在常数时间内求出来一段区间的异或值。
---

## 简介

由于位运算的各位都是相互独立的，所以在处理位运算问题的时候可以考虑把每一位单独拆开处理。

特别地，对于异或操作，通过前缀和的思想，加上异或的自反性（一个数异或自己结果为零），我们可以在预处理后在常数时间内求出来一段区间的异或值。

假设 $S_i$ 是 $a_i$ 的前缀异或值，那么求区间 $[l,r]$ 的异或值就是 $S_r\oplus S_{l-1}$。

## 异或序列

> 给定 $A_1,A_2,\cdots,A_n$，求：
> $$
> \sum_{1\le i\le j\le n} A_i\oplus A_{i+1}\oplus \cdots\oplus A_j
> $$
> 也就是求所有区间的异或值之和。
>
> 对于 $100\%$ 的数据，$1\le n\le 10^5,0\le A_i\le 10^9$
>
> 题目链接：[P3917](https://www.luogu.com.cn/problem/P3917)。

首先通过前缀和的思想转化成下面的式子：
$$
\sum_{1\le i\le j\le n} S_{i-1}\oplus S_j=\sum_{0\le i\lt j\le n} S_i\oplus S_j
$$
比如对于 $S_3$，求的是 $S_3+(S_1\oplus S_3)+(S_2\oplus S_3)$，考虑看每一位对答案的贡献。

如果当前枚举到了第 $k$ 位，分两种情况。

+ $S_j$ 的第 $k$ 位是 $0$，对答案的贡献是前面 $S_i$ 中第 $k$ 位为 $1$ 的个数。
+ $S_j$ 的第 $k$ 位是 $1$，对答案的贡献是前面 $S_i$ 中第 $k$ 位为 $0$ 的个数。

注意，这里的 $S_i$ 都包括了 $S_0$，并且 $S_0=0$，所以一开始要把 $0$ 的个数初始化成 $1$。

```cpp
#include <cstdio>
using namespace std;

const int N = 100010;
int n, s[N];

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &s[i]), s[i] ^= s[i-1];
    
    long long res = 0;
    for (int k = 0; k <= 30; k++) {
        int cnt[2] = {1, 0};
        for (int i = 1; i <= n; i++) {
            int b = s[i] >> k & 1;
            res += (1LL << k) * cnt[b^1];
            cnt[b]++;
        }
    }
    
    return !printf("%lld\n", res);
}
```

## [CF1879D] Sum of XOR Functions

> 给定 $A_1,A_2,\cdots,A_n$，求：
> $$
> \sum_{1\le i\le j\le n} (A_i\oplus A_{i+1}\oplus \cdots \oplus A_{j})(j-i+1)
> $$
> 答案对 $998244353$ 取模，其中 $1\le n\le 3\times10^5,0\le A_i\le 10^9$。
>
> 题目链接：[CF1879D](https://codeforces.com/contest/1879/problem/D)。

同样，还是先求一个前缀异或值。
$$
\sum_{0\le i\lt j\le n} (S_i\oplus S_j)(j-i)
$$
还是对于 $S_3$，求的是 $(S_0\oplus S_3)(3-0)+(S_1\oplus S_3)(3-1)+(S_2\oplus S_3)(3-2)$，考虑每一位对答案的贡献。

如果当前枚举到了第 $k$ 位，分两种情况，如果一个位的结果是 $1$，也就是上面式子中的一个括号内是 $1$。

+ $S_j$ 的第 $k$ 位为 $0$，对答案的贡献是前面 $S_i$ 中第 $k$ 位为 $1$ 的个数，乘上 $j$ 后减去 $\sum i$
+ $S_j$ 的第 $k$ 位为 $1$，对答案的贡献是前面 $S_i$ 中第 $k$ 位为 $0$ 的个数，乘上 $j$ 后减去 $\sum i$

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 300010, P = 998244353;
int n, s[N];

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &s[i]), s[i] ^= s[i-1];

    LL res = 0;
    for (int k = 0; k <= 30; k++) {
        LL cnt[2] = {1, 0}, sum[2] = {0, 0};
        for (int i = 1; i <= n; i++) {
            int b = s[i] >> k & 1;
            res = (res + (1LL << k) * ((cnt[b^1] * i - sum[b^1]) % P)) % P;
            cnt[b]++, sum[b] += i;
        }
    }
    return !printf("%lld\n", res);
}
```

## [CF242E] XOR on Segment

> 给定 $n$ 个数的序列 $a_i$。执行 $m$ 次操作，有两种类型：
> 1. 求 $a_l\sim a_r$ 的和。
> 2. 把 $a_l\sim a_r$ 异或上 $x$。
>
> 对于 $100\%$ 的数据，$1\le n\le 10^5$，$1\le m\le 5\times 10^4$，$0\le a_i\le 10^6$，$1\le x\le 10^6$。
>
> 题目链接：[CF242E](https://codeforces.com/problemset/problem/242/E)。

还是拆位操作，如果我们操作的数字中只有 $0,1$ 那么区间异或导致的结果是 $s=len-s$，我们只要用 $\lceil \log_2 10^6\rceil= 20$ 个线段树维护 $0,1$ 最后合起来就行了。当然我们不会真的开 $20$ 个树，在结点里维护长度为 $20$ 的数组来维护每一位就行了。

在下传懒标记的的时候，我们需要将新的标记和当前标记异或上，但是更新当前节点的时候需要用传进来的标记，而不是运算完的当前标记：因为上一次的标记已经被运算完毕了，如果用运算完的标记就重复计算了。

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 100010;
struct Node {
    int l, r;
    int flip;
    int sum[20];
} tr[N<<2];
int a[N], n, m;

void pushup(int u) {
    for (int i = 0; i < 20; i++)
        tr[u].sum[i] = tr[u<<1].sum[i] + tr[u<<1|1].sum[i];
}

void spread(int u, int flip) {
    tr[u].flip ^= flip;
    for (int i = 0; i < 20; i++)
        if (flip >> i & 1)
            tr[u].sum[i] = tr[u].r - tr[u].l + 1 - tr[u].sum[i];
}

void pushdown(int u) {
    spread(u<<1, tr[u].flip), spread(u<<1|1, tr[u].flip);
    tr[u].flip = 0;
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) {
        for (int i = 0; i < 20; i++)
            tr[u].sum[i] = a[l] >> i & 1;
        return;
    }
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
    pushup(u);
}

void modify(int u, int l, int r, int x) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u, x);

    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) modify(u<<1, l, r, x);
    if (mid+1 <= r) modify(u<<1|1, l, r, x);
    pushup(u);
}

LL query(int u, int l, int r) {
    LL res = 0;
    if (l <= tr[u].l && tr[u].r <= r) {
        for (int i = 0; i < 20; i++) res += (LL)tr[u].sum[i] << i;
        return res;
    }
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) res += query(u<<1, l, r);
    if (mid+1 <= r) res += query(u<<1|1, l, r);
    return res;
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
    build(1, 1, n);
    scanf("%d", &m);
    for (int i = 1, t, l, r, x; i <= m; i++) {
        scanf("%d%d%d", &t, &l, &r);
        if (t == 1) printf("%lld\n", query(1, l, r));
        else scanf("%d", &x), modify(1, l, r, x);
    }
    return 0;
}
```

## [NOI2014] 起床困难综合症

> 一句话题意：给定整数 $n,m$ 和 $n$ 个位运算，求在 $[0,m]$ 中哪个数字能在经过这些运算后结果最大。
>
> 题目链接：[LOJ 2244](https://loj.ac/p/2244)，[AcWing 998](https://www.acwing.com/problem/content/1000/)。

从高位向低位枚举，接下来是贪心：

+ 如果当前位填了 1 导致原数字大于 $m$ 就不能填。

+ 如果当前位填 1 获得的结果小于等于填 0 获得的结果，就填 0。

+ 否则填 1，因为你如果当前位不填 1 下面填再多也无法超过这个数字，这是一个简单的等比级数问题。
    $$
    \sum_{i=0}^{n-1}2^i = 2^n-1 < 2^n
    $$

由于 $m$ 最大是 $10^9$，所以就有 $\lceil\log_2 10^9\rceil=30$ 个二进制位，在 $2^0\sim 2^{29}$ 之间。

```cpp
#include <cstdio>
using namespace std;

const int N = 1e5+10;

struct Op {
    char type[5];
    int num;
} op[N];
int n, m;

int calc(int k, int val) {
    for (int i = 0; i < n; i++) {
        int x = op[i].num >> k & 1;
        switch (op[i].type[0]) {
            case 'O': val |= x; break;
            case 'A': val &= x; break;
            default: val ^= x; break;
        }
    }
    return val;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 0; i < n; i++)
        scanf("%s%d", op[i].type, &op[i].num);
    
    int val = 0, res = 0;
    for (int i = 29; i >= 0; i--) {
        int res0 = calc(i, 0), res1 = calc(i, 1);
        // 注意位运算的优先级比逻辑运算低
        if ((val ^ (1 << i)) <= m && res1 > res0) {
            val ^= 1 << i;
            res ^= res1 << i;
        } else {
            res ^= res0 << i;
        }
    }
    printf("%d\n", res);
    return 0;
}
```

## [CF1878E] Iva & Pav

> 给定长度为 $n$ 的数组 $a$，定义 $f(l,r)$ 是 $[l,r]$ 的区间按位和，给定 $q$ 个查询，每个查询包含 $k,l$ 要求找到最大的 $r$ 使得 $f(l,r)\ge k$，如果不存在输出 $-1$。
>
> 数据范围：$1\le n\le 2\times 10^5, 1\le a_i \le 10^9,1\le q\le 10^5, 1\le l\le n,1\le k\le 10^9$。
>
> 题目链接：[CF1878E](https://codeforces.com/contest/1878/problem/E)。

与运算是非严格单调递减的，即 $\forall a,a\&b\le a$，这与或相反。有了这个性质，我们就可以二分找答案了，下面的问题变成了如何快速求出一段区间的与运算和。

一个简单的方法是用线段树，赛时我也确实是这么做的，下面贴出代码：

```cpp
#include <algorithm>
#include <iostream>
#include <cstdio>
#include <cstring>
using namespace std;

const int N = 200010;
int a[N], n;
struct Node {
	int l, r, v;
} tr[N<<2];

void pushup(int u) {
	tr[u].v = tr[u<<1].v & tr[u<<1|1].v;
}

void build(int u, int l, int r) {
	tr[u].l = l, tr[u].r = r;
	if (l == r) return tr[u].v = a[l], void();
	int mid = l+r >> 1;
	build(u<<1, l, mid), build(u<<1|1, mid+1, r);
	pushup(u);
}

int query(int u, int l, int r) {
	if (l <= tr[u].l && tr[u].r <= r) return tr[u].v;
	
	int mid = tr[u].l + tr[u].r >> 1;
	if (r < mid+1) return query(u<<1, l, r);
	if (l > mid) return query(u<<1|1, l, r);
	
	return query(u<<1, l, r) & query(u<<1|1, l, r);
}

void solve() {
	cin >> n;
	for (int i = 1; i <= n; i++) cin >> a[i];
	build(1, 1, n);
	int q, start, k;
	cin >> q;
	while (q--) {
		cin >> start >> k;
		int l = start, r = n;
		while (l < r) {
			int mid = l+r+1 >> 1;
			if (query(1, start, mid) >= k) l = mid;
			else r = mid-1;
		}
		if (r == start && a[start] < k) cout << -1 << ' ';
		else cout << r << ' ';
	}
	cout << '\n';
}

int main() {
	int T; cin >> T;
	while (T--) solve();
	return 0;
}
```

既然出现在这篇文章中，那么肯定是有拆位的做法，具体想法就是对于二进制位 $i$，当且仅当 $S_{i,r}-S_{i,l-1}=r-l+1$ 时是有效的，其余情况都是 $0$，所以可以这么写：

```cpp
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <iostream>
using namespace std;

const int N = 200010;
int a[N], s[30][N], n;

int query(int l, int r) {
    int res = 0;
    for (int i = 0; i < 30; i++)
        if (s[i][r] - s[i][l-1] == r-l+1)
            res += 1 << i;
    return res;
}

void solve() {
    cin >> n;
    for (int i = 1; i <= n; i++) cin >> a[i];
    for (int i = 0; i < 30; i++)
        for (int j = 1; j <= n; j++)
            s[i][j] = s[i][j-1] + (a[j] >> i & 1);

    int q, start, k;
    cin >> q;
    while (q--) {
        cin >> start >> k;
        int l = start, r = n;
        while (l < r) {
            int mid = l+r+1 >> 1;
            if (query(start, mid) >= k) l = mid;
            else r = mid-1;
        }
        if (r == start && a[start] < k) cout << -1 << ' ';
        else cout << r << ' ';
    }
    cout << '\n';
}

int main() {
    int T; cin >> T;
    while (T--) solve();
    return 0;
}
```

此外，本题还可以用 ST 表做，放到本站介绍 ST 表的文章中了。
