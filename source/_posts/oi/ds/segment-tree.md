---
date: 2023-07-14 10:30:43
update: 2023-10-24 15:31:43
title: 数据结构 - 线段树
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
---

## 简介

线段树是用于维护一个区间内**具有结合律**数据的高效数据结构，单次操作复杂度为 $O(\log n)$，但常数较大。

将一个区间不断以 $mid=\lfloor \cfrac{l+r}{2}\rfloor,[l,mid],[mid+1,r]$ 为区间分为两个子树，由于除了最后一层所有层都是满的，所以用类似于存储堆的形式，即数组来存储线段树，一个结点编号为 $n$ 那么左子结点是 $2n$，右子结点是 $2n+1$。

设倒数第二层有 $2^k$ 个结点，那么上面有 $\sum_{i=0}^{k-1} 2^i = 2^k-1$ 个结点，下面一层最多有 $2^{k+1}$ 个结点，设总共有 $N$ 个元素，坏的情况下应该是倒数第二层很多，最后一层很少，因此保险起见开 $4N$ 的空间绝对够用。

## 模板 I

> 如题，已知一个数列，你需要进行下面两种操作：
>
> 1. 将某区间每一个数加上 $k$。
> 2. 求出某区间每一个数的和。
>
> **输入格式**
>
> 第一行包含两个整数 $n, m$，分别表示该数列数字的个数和操作的总个数。
>
> 第二行包含 $n$ 个用空格分隔的整数，其中第 $i$ 个数字表示数列第 $i$ 项的初始值。
>
> 接下来 $m$ 行每行包含 $3$ 或 $4$ 个整数，表示一个操作，具体如下：
>
> 1. `1 x y k`：将区间 $[x, y]$ 内每个数加上 $k$。
> 2. `2 x y`：输出区间 $[x, y]$ 内每个数的和。
>
> **输出格式**
>
> 输出包含若干行整数，即为所有操作 2 的结果。
>
> **数据范围**
>
> 对于 $100\%$ 的数据：$1 \le n, m \le {10}^5$，保证任意时刻数列中所有元素的绝对值之和 $\le {10}^{18}$。
>
> 题目链接：[P3372](https://www.luogu.com.cn/problem/P3372)。

```cpp
#include <cstdio>
using namespace std;

typedef long long LL;
const int N = 1e5+10;

struct Node {
    int l, r;
    LL sum, add;
    void spread(int d) {
        sum += (r-l+1) * (LL)d;
        add += d;
    }
} tr[N * 4];
int a[N], n, m;

void pushdown(int u) {
    if (tr[u].add) {
        tr[u<<1].spread(tr[u].add);
        tr[u<<1|1].spread(tr[u].add);
        tr[u].add = 0;
    }
}

void pushup(int u) {
    tr[u].sum = tr[u<<1].sum + tr[u<<1|1].sum;
}

void build(int u, int l, int r) {
    tr[u] = {l, r};
    if (l == r) tr[u].sum = a[l];
    else {
        int mid = l+r >> 1;
        build(u<<1, l, mid);
        build(u<<1|1, mid+1, r);
        pushup(u);
    }
}

void modify(int u, int l, int r, int d) {
    if (l <= tr[u].l && tr[u].r <= r) {
        tr[u].spread(d);
    } else {
        // 修改前先下放
        pushdown(u);
        int mid = tr[u].l + tr[u].r >> 1;
        if (l <= mid) modify(u<<1, l, r, d);
        if (mid+1 <= r) modify(u<<1|1, l, r, d);
        // 回溯时重计算
        pushup(u);
    }
}

LL query(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u].sum;
    
    pushdown(u);
    LL res = 0;
    int mid = tr[u].l + tr[u].r >> 1;
    if (l <= mid) res = query(u<<1, l, r);
    if (mid+1 <= r) res += query(u<<1|1, l, r);
    return res;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", a+i);
    build(1, 1, n);
    int t, l, r, d;
    while (m--) {
        scanf("%d%d%d", &t, &l, &r);
        if (t == 1) {
            scanf("%d", &d);
            modify(1, l, r, d);
        } else {
            printf("%lld\n", query(1, l, r));
        }
    }
    return 0;
}
```

## 模板 II

> 如题，已知一个数列，你需要进行下面三种操作：
>
> - 将某区间每一个数乘上 $x$；
> - 将某区间每一个数加上 $x$；
> - 求出某区间每一个数的和。
>
> **输入格式**
>
> 第一行包含三个整数 $n,q,m$，分别表示该数列数字的个数、操作的总个数和模数。
>
> 第二行包含 $n$ 个用空格分隔的整数，其中第 $i$ 个数字表示数列第 $i$ 项的初始值。
>
> 接下来 $q$ 行每行包含若干个整数，表示一个操作，具体如下：
>
> 操作 $1$： 格式：`1 x y k`  含义：将区间 $[x,y]$ 内每个数乘上 $k$
>
> 操作 $2$： 格式：`2 x y k`  含义：将区间 $[x,y]$ 内每个数加上 $k$
>
> 操作 $3$： 格式：`3 x y`  含义：输出区间 $[x,y]$ 内每个数的和对 $m$ 取模所得的结果
>
> **输出格式**
>
> 输出包含若干行整数，即为所有操作 $3$ 的结果。
>
> **数据范围**
>
> 对于 $100\%$ 的数据：$1 \le n \le 10^5$，$1 \le q \le 10^5$。除样例外，$m = 571373$。
>
> 题目链接：[P3373](https://www.luogu.com.cn/problem/P3373)。

### 传统做法

结点中的懒标记分别是 $\text{add}, \text{mul}$ 这里有一个技巧，可以同时传递加和乘，所有的计算都看成先乘后加，那么：
$$
a(\text{mul }x+\text{add}) + b=a\text{ mul }x+a\text{ add}+b
$$
对应的懒标记更新就是：
$$
\begin{aligned}
\text{mul}&=a \text{ mul}\\
\text{add}&=a \text{ add}+b
\end{aligned}
$$

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
#define lson (u<<1)
#define rson (u<<1|1)


int n, q, mod;
const int N = 100010;
struct Node {
	int l, r;
	LL sum;
	LL add, mul = 1;
	
	void spread(LL a, LL b) {
		mul = (a*mul) % mod;
		add = (a*add + b) % mod;
		sum = (a*sum + (r-l+1)*b) % mod;
	}
} tr[N<<2];
int a[N];

void pushdown(int u) {
	if (tr[u].add || tr[u].mul != 1) {
		tr[lson].spread(tr[u].mul, tr[u].add);
		tr[rson].spread(tr[u].mul, tr[u].add);
		tr[u].add = 0;
		tr[u].mul = 1;
	}
}

void pushup(int u) {
	tr[u].sum = (tr[lson].sum + tr[rson].sum) % mod;
}

void build(int u, int l, int r) {
	tr[u].l = l, tr[u].r = r;
	if (l == r) tr[u].sum = a[l];
	else {
		int mid = l+r >> 1;
		build(lson, l, mid);
		build(rson, mid+1, r);
		pushup(u);
	}
}

LL query(int u, int l, int r) {
	if (l <= tr[u].l && tr[u].r <= r) {
		return tr[u].sum;
	}
	pushdown(u);
	LL res = 0;
	int mid = tr[u].l + tr[u].r >> 1;
	if (l <= mid) res = query(lson, l, r);
	if (r >= mid+1) res = (res + query(rson, l, r)) % mod;
	return res;
}

void modify(int u, int l, int r, LL a, LL b) {
	if (l <= tr[u].l && tr[u].r <= r) {
		tr[u].spread(a, b);
		return;
	}
	pushdown(u);
	int mid = tr[u].l + tr[u].r >> 1;
	if (l <= mid) modify(lson, l, r, a, b);
	if (r >= mid+1) modify(rson, l, r, a, b);
	pushup(u);
}

int main() {
	scanf("%d%d%d", &n, &q, &mod);
	for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
	build(1, 1, n);
	while (q--) {
		int t, x, y, k;
		scanf("%d%d%d", &t, &x, &y);
		if (t == 3) printf("%lld\n", query(1, x, y));
		else {
			scanf("%d", &k);
			if (t == 1) modify(1, x, y, k, 0);
			else modify(1, x, y, 1, k);
		}
	}
	return 0;
}
```

### 矩阵乘法

可以观察到，对于每个区间的操作，把 $[sum,len]$ 组成向量：

1. 区间加 $k$ 会导致 $sum\leftarrow sum+k\times len$，也就是用它乘 $\begin{bmatrix}1&0\\k&1\end{bmatrix}$
2. 区间乘 $k$ 会导致 $sum \leftarrow k\times sum$，也就是用它乘 $\begin{bmatrix}k&0\\0&1\end{bmatrix}$

所以在每个结点记录一下当前需要乘的矩阵，下放后设为单位阵即可：

```c
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;
int n, q, p, a[N];

void unit(int mat[2][2]) {
    mat[0][0] = mat[1][1] = 1;
    mat[0][1] = mat[1][0] = 0;
}

void matmul(int a[2][2], int b[2][2]) {
    static int res[2][2];

    res[0][0] = (1LL * a[0][0] * b[0][0] + 1LL * a[0][1] * b[1][0]) % p;
    res[0][1] = (1LL * a[0][0] * b[0][1] + 1LL * a[0][1] * b[1][1]) % p;
    res[1][0] = (1LL * a[1][0] * b[0][0] + 1LL * a[1][1] * b[1][0]) % p;
    res[1][1] = (1LL * a[1][0] * b[0][1] + 1LL * a[1][1] * b[1][1]) % p;

    memcpy(a, res, sizeof res);
}

struct Node {
    int l, r;
    int mat[2][2];
    int sum;
} tr[N<<2];

void pushup(int u) {
    tr[u].sum = (tr[u<<1].sum + tr[u<<1|1].sum) % p;
}

void spread(int u, int mat[2][2]) {
    int a = tr[u].sum, b = tr[u].r - tr[u].l + 1;
    tr[u].sum = (1LL * a * mat[0][0] + 1LL * b * mat[1][0]) % p;
    matmul(tr[u].mat, mat);
}

void pushdown(int u) {
    spread(u<<1, tr[u].mat);
    spread(u<<1|1, tr[u].mat);
    unit(tr[u].mat);
}

void build(int u, int l, int r) {
    unit(tr[u].mat);

    tr[u].l = l, tr[u].r = r;
    if (l == r) return tr[u].sum = a[l] % p, void();
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
    pushup(u);
}

void modify(int u, int l, int r, int mat[2][2]) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u, mat);
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) modify(u<<1, l, r, mat);
    if (mid+1 <= r) modify(u<<1|1, l, r, mat);
    pushup(u);
}

int query(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u].sum;
    pushdown(u);
    int res = 0, mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) res = query(u<<1, l, r);
    if (mid+1 <= r) res = (res + query(u<<1|1, l, r)) % p;
    return res;
}

int main() {
    scanf("%d%d%d", &n, &q, &p);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
    build(1, 1, n);
    
    int mat[2][2];
    for (int i = 1, op, x, y; i <= q; i++) {
        unit(mat);
        scanf("%d%d%d", &op, &x, &y);
        if (op == 3) printf("%d\n", query(1, x, y));
        else if (op == 2) scanf("%d", &mat[1][0]), modify(1, x, y, mat);
        else scanf("%d", &mat[0][0]), modify(1, x, y, mat);
    }
    return 0;
}
```

## 最大连续子段和

> 给定长度为 N 的数列 A，以及 M 条指令，每条指令可能是以下两种之一：
>
> 1. `1 x y`，查询区间 [x,y] 中的最大连续子段和，即 $\max _{x\le l \le r \le y}(\sum_{i=l}^rA[i])$。
> 2. `2 x y`，把 A[x] 改成 y。
>
> 对于每个查询指令，输出一个整数表示答案。
>
> 题目链接：[AcWing 245](https://www.acwing.com/problem/content/246/)。

每个结点需要维护四个属性：最大连续子区间和 $tmax$，最大前缀和 $lmax$，最大后缀和 $rmax$，区间和$sum$，对于一个结点 $u$ 和它的两个子结点 $l, r$ 来说，满足下面的关系：
$$
\begin{aligned}
tmax(u)&=\max\{tmax(l), tmax(r), rmax(l)+lmax(r)\}\\
lmax(u)&=\max\{lmax(l), sum(l)+lmax(r)\}\\
rmax(u)&=max\{rmax(r), rmax(l)+sum(r)\}\\
sum(u)&=sum(l)+sum(r)
\end{aligned}
$$
如果中间的两个数都是负的，有个刚写时会出现的疑问，就是会不会选了两个负数不如选一个更优。但是，如果两边所有数都是负数，那么左边的 $tmax$ 或者右边的 $tmax$ 一定就是各个子区间中最大的那个负数，所以我们这个逻辑是可以正确处理这个情况的。

在线段树中查询一段时，如果区间 $[x,y]$ 覆盖了结点 $u$ 的两个子结点，那么需要获取到这两个子结点然后仿照上面计算最大值。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 500010;

struct Node {
    int l, r;
    int tmax, lmax, rmax, sum;
    void set(int v) {
        tmax = lmax = rmax = sum = v;
    }
} tr[N<<2];
int a[N], n, m;

void pushup(Node& u, Node l, Node r) {
    u.tmax = max(l.tmax, max(r.tmax, l.rmax + r.lmax));
    u.lmax = max(l.lmax, l.sum+r.lmax);
    u.rmax = max(l.rmax+r.sum, r.rmax);
    u.sum = l.sum + r.sum;
}

void pushup(int u) {
    pushup(tr[u], tr[u<<1], tr[u<<1|1]);
}

void build(int u, int l, int r) {
    // 不管哪个结点都要初始化 l 和 r
    tr[u] = {l, r};
    if (l == r) tr[u].set(a[l]);
    else {
        int mid = l+r >> 1;
        build(u<<1, l, mid);
        build(u<<1|1, mid+1, r);
        pushup(u);
    }
}

void modify(int u, int p, int v) {
    if (tr[u].l == p && tr[u].r == p) tr[u].set(v);
    else {
        int mid = tr[u].l + tr[u].r >> 1;
        if (p <= mid) modify(u<<1, p, v);
        else modify(u<<1|1, p, v);
        pushup(u);
    }
}

Node query(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u];
    else {
        int mid = tr[u].l + tr[u].r >> 1;
        if (l > mid) return query(u<<1|1, l, r);
        else if (r < mid+1) return query(u<<1, l, r);
        else {
            Node res;
            pushup(res, query(u<<1, l, r), query(u<<1|1, l, r));
            return res;
        }
    }
}

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> a[i];
    build(1, 1, n);
    
    int t, x, y;
    while (m--) {
        cin >> t >> x >> y;
        if (t == 1) {
            if (x > y) swap(x, y);
            cout << query(1, x, y).tmax << endl;
        } else modify(1, x, y);
    }
    
    return 0;
}
```

## 区间最大公约数

> 给定一个长度为 N 的数列 A，以及 M 条指令，每条指令可能是以下两种之一：
>
> 1. `C l r d`，表示把 A[l],A[l+1],…,A[r],都加上 d。
> 2. `Q l r`，表示询问 A[l],A[l+1],…,A[r] 的最大公约数(GCD)。
>
> 对于每个询问，输出一个整数表示答案。
>
> 题目链接：[AcWing 246](https://www.acwing.com/problem/content/247/)。

结论是，一个数列与它的差分数列最大公约数相同。
$$
\gcd(a_1,a_2,\cdots,a_n)=\gcd(a_1,a_2-a_1,\cdots,a_n-a_{n-1})
$$
考虑左边的任意一个约数 $d$，它一定也是右边的约数；右边的任意一个约数 $d$ 也一定是左边的约数：$d$ 时 $a_1, a_2-a_1$ 的约数一定也是 $(a_2-a_1)+a_1$ 的约数。

但是，对于一个数列的区间 $[L,R]$ 它的差分的 `gcd` 就不是整个 `gcd` 了，需要先求出来第一项。

用差分使得此问题转化为单点修改与单点查询。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

typedef long long ll;
const int N = 500010;

struct Node {
    int l, r;
    ll v, sum;
} tr[N<<2];

ll a[N];

ll _gcd(ll a, ll b) {
    return a ? _gcd(b%a, a) : b;
}

ll gcd(ll a, ll b) {
    return _gcd(abs(a), abs(b));
}

void pushup(Node& u, Node l, Node r) {
    u.v = gcd(l.v, r.v);
    u.sum = l.sum + r.sum;
}

void pushup(int u) {
    pushup(tr[u], tr[u<<1], tr[u<<1|1]);
}

void build(int u, int l, int r) {
    tr[u] = {l, r};
    if (l == r) {
        tr[u].v = tr[u].sum = a[l]-a[l-1];
    } else {
        int mid = l+r >> 1;
        build(u<<1, l, mid);
        build(u<<1|1, mid+1, r);
        pushup(u);
    }
}

void modify(int u, int p, ll v) {
    if (tr[u].l == p && tr[u].r == p) tr[u].v += v, tr[u].sum += v;
    else {
        int mid = tr[u].l + tr[u].r >> 1;
        if (p <= mid) modify(u<<1, p, v);
        else modify(u<<1|1, p, v);
        pushup(u);
    }
}

Node query(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u];
    else {
        int mid = tr[u].l + tr[u].r >> 1;
        if (l > mid) return query(u<<1|1, l, r);
        else if (r <= mid) return query(u<<1, l, r);
        else {
            Node res;
            pushup(res, query(u<<1, l, r), query(u<<1|1, l, r));
            return res;
        }
    }
}

int main() {
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> a[i];
    build(1, 1, n);
    char op;
    int l, r;
    ll d;
    while (m--) {
        cin >> op >> l >> r;
        if (op == 'Q') {
            ll res = query(1, 1, l).sum;
            if (l < r) res = gcd(res, query(1, l+1, r).v);
            cout << res << endl;
        } else {
            cin >> d;
            modify(1, l, d);
            if (r < n) modify(1, r+1, -d);
        }
    }
    return 0;
}
```

## 区间开根

> **输入格式**
>
> 第一行一个整数 $n$，代表数列中数的个数。
>
> 第二行 $n$ 个正整数，表示初始状态下数列中的数。
>
> 第三行一个整数 $m$，表示有 $m$ 次操作。
>
> 接下来 $m$ 行每行三个整数 `k l r`。
>
> - $k=0$ 表示给 $[l,r]$ 中的每个数开平方（下取整）。
>
> - $k=1$ 表示询问 $[l,r]$ 中各个数的和。
>
> 数据中有可能 $l>r$，所以遇到这种情况请交换 $l$ 和 $r$。
>
> **输出格式**
>
> 对于询问操作，每行输出一个回答。
>
> **数据范围**
>
> 对于 $100\%$ 的数据，$1\le n,m\le 10^5$，$1\le l,r\le n$，数列中的数大于 $0$，且不超过 $10^{12}$。
>
> 题目链接：[P4145](https://www.luogu.com.cn/problem/P4145)。

乍看起来不可做，因为知道 $\sum a_i$ 并不能 $O(1)$ 求出 $\sum\sqrt a_i$。

但是，可以知道开根多次最后的结果就是 $1$，设最多开根 $x$ 次，那么列出下列不等式：

$$
a^{\frac{1}{2^x}}<2
$$
带入 $10^{12}$ 可以解出：
$$
x>\log_2\log_2 10^{12}\approx 5.3
$$
所以最多就开 $6$ 次，这样看下来复杂度是 $O(n\log n\log \log a)$

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;

const int N = 100010;

struct Node {
    int l, r;
    LL sum, mx;
} tr[N<<2];
LL a[N];
int n, m;

void pushup(int u) {
    tr[u].sum = tr[u<<1].sum + tr[u<<1|1].sum;
    tr[u].mx = max(tr[u<<1].mx, tr[u<<1|1].mx);
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) return tr[u].sum = tr[u].mx = a[l], void();
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
    pushup(u);
}

LL query(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u].sum;

    LL res = 0;
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) res += query(u<<1, l, r);
    if (mid+1 <= r) res += query(u<<1|1, l, r);
    return res;
}

void modify(int u, int p) {
    if (tr[u].l == p && tr[u].r == p) {
        tr[u].sum = tr[u].mx = sqrt(tr[u].sum);
        return;
    }
    
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (p <= mid) modify(u<<1, p);
    else modify(u<<1|1, p);

    pushup(u);
}

void modify(int u, int l, int r) {
    if (tr[u].mx == 1) return;

    if (l <= tr[u].l && tr[u].r <= r) {
        for (int i = tr[u].l; i <= tr[u].r; i++) modify(u, i);
        return;
    }

    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) modify(u<<1, l, r);
    if (mid+1 <= r) modify(u<<1|1, l, r);
    pushup(u);
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%lld", &a[i]);
    build(1, 1, n);

    scanf("%d", &m);
    for (int i = 1, k, l, r; i <= m; i++) {
        scanf("%d%d%d", &k, &l, &r);
        if (l > r) swap(l, r);
        if (k == 1) printf("%lld\n", query(1, l, r));
        else modify(1, l, r);
    }
    return 0;
}
```

## 区间 sin

> 给出一个长度为 $n$ 的整数序列 $a_1,a_2,\ldots,a_n$，进行 $m$ 次操作，操作分为两类。
>
> 操作 $1$：给出 $l,r,v$，将 $a_l,a_{l+1},\ldots,a_r$ 分别加上 $v$。
>
> 操作 $2$：给出 $l,r$，询问 $\sum\limits_{i=l}^{r}\sin(a_i)$。
>
> **输入格式**
>
> 第一行一个整数 $n$。
>
> 接下来一行 $n$ 个整数表示 $a_1,a_2,\ldots,a_n$。
>
> 接下来一行一个整数 $m$。
>
> 接下来 $m$ 行，每行表示一个操作，操作 $1$ 表示为 `1 l r v`，操作 $2$ 表示为 `2 l r`。
>
> **输出格式**
>
> 对每个操作 $2$，输出一行，表示答案，四舍五入保留一位小数。
>
> 保证答案的绝对值大于 $0.1$，且答案的准确值的小数点后第二位不是 $4$ 或 $5$。
>
> **数据范围**
>
> 保证 $1\leq n,m,a_i,v\leq 2\times 10^5$，$1\leq l\leq r\leq n$。保证所有输入的数都是正整数。
>
> 题目链接：[P6327](https://www.luogu.com.cn/problem/P6327)。

只需要利用和角公式就可以在 $O(1)$ 的复杂度求出一个区间加上某个值后新的 $\sum \sin(a_i),\sum\cos(a_i)$ 了。
$$
\sum\sin(a_i+x)=\cos x\sum\sin a_i+\sin x\sum\cos a_i\\
\sum\cos(a_i+x)=\cos x\sum\cos a_i-\sin x\sum\sin a_i
$$
同时维护 $\sin,\cos$ 和即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 200010;

int a[N], n, m;
struct Node {
    int l, r;
    double sumsin, sumcos;
    double addsin, addcos;
} tr[N<<2];

void spread(int u, double sinx, double cosx) {
    double sumsin = tr[u].sumsin, sumcos = tr[u].sumcos;
    tr[u].sumsin = cosx * sumsin + sinx * sumcos;
    tr[u].sumcos = cosx * sumcos - sinx * sumsin;

    double addsin = tr[u].addsin, addcos = tr[u].addcos;
    tr[u].addsin = cosx * addsin + sinx * addcos;
    tr[u].addcos = cosx * addcos - sinx * addsin;
}

void pushup(int u) {
    tr[u].sumsin = tr[u<<1].sumsin + tr[u<<1|1].sumsin;
    tr[u].sumcos = tr[u<<1].sumcos + tr[u<<1|1].sumcos;
}

void pushdown(int u) {
    spread(u<<1, tr[u].addsin, tr[u].addcos);
    spread(u<<1|1, tr[u].addsin, tr[u].addcos);
    tr[u].addsin = 0, tr[u].addcos = 1;
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    tr[u].addsin = 0, tr[u].addcos = 1;
    if (l == r) {
        tr[u].sumcos = cos(a[l]);
        tr[u].sumsin = sin(a[l]);
        return;
    }
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
    pushup(u);
}

void modify(int u, int l, int r, double sinx, double cosx) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u, sinx, cosx);
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) modify(u<<1, l, r, sinx, cosx);
    if (mid+1 <= r) modify(u<<1|1, l, r, sinx, cosx);
    pushup(u);
}

double query(int u, int l, int r) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u].sumsin;
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    double res = 0;
    if (l <= mid) res += query(u<<1, l, r);
    if (mid+1 <= r) res += query(u<<1|1, l, r);
    return res;
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
    build(1, 1, n);

    scanf("%d", &m);
    for (int i = 1, opt, l, r, v; i <= m; i++) {
        scanf("%d%d%d", &opt, &l, &r);
        if (opt == 1) scanf("%d", &v), modify(1, l, r, sin(v), cos(v));
        else printf("%.1lf\n", query(1, l, r));
    }
    return 0;
}
```

## 区间第 k 小

> 如题，给定 $n$ 个整数构成的序列 $a$，将对于指定的闭区间 $[l, r]$ 查询其区间内的第 $k$ 小值。
>
> **输入格式**
>
> 第一行包含两个整数，分别表示序列的长度 $n$ 和查询的个数 $m$。
>
> 第二行包含 $n$ 个整数，第 $i$ 个整数表示序列的第 $i$ 个元素 $a_i$。
>
> 接下来 $m$ 行每行包含三个整数 $l, r, k$ , 表示查询区间 $[l, r]$ 内的第 $k$ 小值。
>
> **输出格式**
>
> 对于每次询问，输出一行一个整数表示答案。
>
> **数据范围**
>
> 对于 $100\%$ 的数据，满足 $1 \leq n,m \leq 2\times 10^5$，$|a_i| \leq 10^9$，$1 \leq l \leq r \leq n$，$1 \leq k \leq r - l + 1$。
>
> 题目链接：[P3834](https://www.luogu.com.cn/problem/P3834)。

用可持久化线段树解决这个问题，首先写一下注意事项。

1. 用 `tr[0]` 当作 `nullptr` 类似的东西去用，保证其满足 `ls = rs = cnt = 0`。
2. 不在 `Node` 中储存左右边界，直接递归传下去，这样用 `tr[0]` 一个结点可以表示任意大小的权值为零的线段树。

3. 我们建 $n$ 颗权值线段树，每次建树只是对一个点的权值加一，所以只会影响到 $\log_2 n\approx 17.6$ 个结点，所以开 $18n$ 结点够了。
4. 在查询时，就看左子树的权值够不够 $k$，如果够了就递归到左边，否则递归到右边，退出条件是 $L=R$，不是 $L=k\and R=k$，因为 $k$ 代表的是数量而非数本身。
5. 值域大需要离散化。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 200010;
int n, m, rt[N], a[N], nums[N], cntnums;

struct Node {
    int ls, rs;
    int cnt;
} tr[18 * N];
int idx;

int insert(int u, int L, int R, int k) {
    int p = ++idx;
    tr[p].cnt = tr[u].cnt + 1;
    if (L == k && R == k) return p;

    int mid = (L+R) >> 1;
    if (k <= mid) tr[p].ls = insert(tr[u].ls, L, mid, k), tr[p].rs = tr[u].rs;
    else tr[p].rs = insert(tr[u].rs, mid+1, R, k), tr[p].ls = tr[u].ls;

    return p;
}

int query(int u, int v, int L, int R, int k) {
    if (L == R) return L;
    int cnt = tr[tr[v].ls].cnt - tr[tr[u].ls].cnt;
    int mid = (L+R) >> 1;
    if (k <= cnt) return query(tr[u].ls, tr[v].ls, L, mid, k);
    else return query(tr[u].rs, tr[v].rs, mid+1, R, k-cnt);
}

int find(int x) {
    return lower_bound(nums+1, nums+1+cntnums, x) - nums;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]), nums[i] = a[i];
    sort(nums+1, nums+1+n);
    cntnums = unique(nums+1, nums+1+n) - (nums+1);

    for (int i = 1; i <= n; i++) rt[i] = insert(rt[i-1], 1, cntnums, find(a[i]));

    for (int i = 1, x, y, k; i <= m; i++) {
        scanf("%d%d%d", &x, &y, &k);
        printf("%d\n", nums[query(rt[x-1], rt[y], 1, cntnums, k)]);
    }

    return 0;
}
```

## 区间欧拉函数

> 给定数组 $a_1,a_2,\cdots,a_n$，你需要完成 $q$ 次操作。
>
> 1. `MULTIPLY l r x` 给所有 $l\le i\le r$ 的 $a_i\leftarrow a_ix$
> 2. `TOTIENT l r` 求出 $\varphi(\prod_{i=l}^r a_i)$ 对 $10^9+7$ 取模的结果。
>
> 数据范围：$1\le n\le 4\times10^5,1\le q\le2\times10^5,1\le a_i,x\le 300$。
>
> 题目链接：[CF1114F](https://codeforces.com/contest/1114/problem/F)。

回顾一下欧拉函数的定义 $\varphi(x)=x(1-p_1^{-1})(1-p_2^{-1})\cdots(1-p_k^{-1})$，所以只需要记录下来所有的质因子和乘积即可。打表可以知道 $300$ 以内的质数有 $62$ 个，可以用 `long long` 压位存储。

我一开始直接硬维护了质因数分解的结果，每个结点存两个 $62$ 大小的数组，一个存乘积的质因数分解，一个存懒标记，但是常数太大了。

首先先把质数都打表打出来，然后 $1-p_i^{-1}$ 也打表打出来，然后线段树维护即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 400010, M = 310, P = 1e9+7;

struct Node {
    int l, r;
    int mul, prod;
    LL mulp, prodp;
} tr[N<<2];

int n, q, a[N];
vector<int> primes, inv;

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = 1LL * res * a % P;
        a = 1LL * a * a % P;
        k >>= 1;
    }
    return res;
}

void init(int n) {
    bitset<M> st;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes.emplace_back(i);
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i*primes[j]] = 1;
            if (i % primes[j] == 0) break;
        }
    }
    for (int p: primes) inv.emplace_back((1-qmi(p, P-2)+P) % P);
}

LL factorize(int x) {
    LL res = 0;
    for (int i = 0; i < primes.size(); i++)
        if (x % primes[i] == 0)
            res |= 1LL << i;
    return res;
}

void pushup(int u) {
    tr[u].prod = 1LL * tr[u<<1].prod * tr[u<<1|1].prod % P;
    tr[u].prodp = tr[u<<1].prodp | tr[u<<1|1].prodp;
}

void spread(int u, int prod, LL primes) {
    tr[u].prod = 1LL * tr[u].prod * qmi(prod, tr[u].r - tr[u].l + 1) % P;
    tr[u].mul = 1LL * tr[u].mul * prod % P;
    tr[u].prodp |= primes;
    tr[u].mulp |= primes;
}

void pushdown(int u) {
    if (tr[u].mulp) {
        spread(u<<1, tr[u].mul, tr[u].mulp);
        spread(u<<1|1, tr[u].mul, tr[u].mulp);
        tr[u].mulp = 0, tr[u].mul = 1;
    }
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    tr[u].mul = 1, tr[u].mulp = 0;
    if (l == r) return tr[u].prod = a[l], tr[u].prodp = factorize(a[l]), void();
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
    pushup(u);
}

void query(int u, int l, int r, int& prod, LL& primes) {
    if (l <= tr[u].l && tr[u].r <= r) {
        prod = 1LL * prod * tr[u].prod % P;
        primes |= tr[u].prodp;
        return;
    }
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) query(u<<1, l, r, prod, primes);
    if (mid+1 <= r) query(u<<1|1, l, r, prod, primes);
}

void modify(int u, int l, int r, int prod, LL primes) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u, prod, primes);
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) modify(u<<1, l, r, prod, primes);
    if (mid+1 <= r) modify(u<<1|1, l, r, prod, primes);
    pushup(u);
}

int main() {
    init(M-1);
    scanf("%d%d", &n, &q);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
    build(1, 1, n);

    char op[10];
    int l, r, x;
    while (q--) {
        scanf("%s%d%d", op, &l, &r);
        if (*op == 'M') scanf("%d", &x), modify(1, l, r, x, factorize(x));
        else {
            int prod = 1;
            LL primes = 0;
            query(1, l, r, prod, primes);
            for (int i = 0; i < inv.size(); i++)
                if (primes >> i & 1)
                    prod = 1LL * prod * inv[i] % P;
            printf("%d\n", prod);
        }
    }
    return 0;
}
```

## 权值线段树实现平衡树

题目就是普通平衡树模板，这里主要说对于 $x$ 的前驱和后继的找法。

首先是前驱，公式是 $pre(x)=kth(rk(x)-1)$，其中 $rk(x)$ 指所有 $<x$ 的元素数量加一，所以这个公式显然是正确的。

其次是后继，公式是 $suc(x)=kth(rk(x+1))$，这里 $rk(x+1)$ 就是所有 $\le x$ 的元素数量加一，所以这个公式能够找出第一个大于 $x$ 的元素。

由于值域较大，这里使用动态开点。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 100010, LogM = 26;
int n, opt, x;

struct Node {
    int cnt;
    int ls, rs;
} tr[N*LogM];
int idx, rt;

void pushup(int u) {
    tr[u].cnt = tr[tr[u].ls].cnt + tr[tr[u].rs].cnt;
}

void insert(int& u, int x, int L, int R) {
    if (!u) u = ++idx;
    if (L == R) return ++tr[u].cnt, void();
    int Mid = (L+R) >> 1;
    if (x <= Mid) insert(tr[u].ls, x, L, Mid);
    else insert(tr[u].rs, x, Mid+1, R);
    pushup(u);
}

void del(int u, int x, int L, int R) {
    if (L == R) return --tr[u].cnt, void();
    int Mid = (L+R) >> 1;
    if (x <= Mid) del(tr[u].ls, x, L, Mid);
    else del(tr[u].rs, x, Mid+1, R);
    pushup(u);
}

int query(int u, int r, int L, int R) {
    if (!u) return 0;
    if (R <= r) return tr[u].cnt;
    int Mid = (L+R) >> 1;
    int res = query(tr[u].ls, r, L, Mid);
    if (Mid+1 <= r) res += query(tr[u].rs, r, Mid+1, R);
    return res;
}

int rk(int x, int L, int R) {
    return query(rt, x-1, L, R) + 1;
}

int kth(int u, int k, int L, int R) {
    if (L == R) return L;
    int Mid = (L+R) >> 1;
    if (k <= tr[tr[u].ls].cnt) return kth(tr[u].ls, k, L, Mid);
    else return kth(tr[u].rs, k - tr[tr[u].ls].cnt, Mid+1, R);
}

int pre(int u, int r, int L, int R) {
    return kth(u, rk(x, L, R) - 1, L, R);
}

int suc(int u, int x, int L, int R) {
    return kth(u, rk(x+1, L, R), L, R);
}

signed main() {
    cin >> n;
    while (n--) {
        cin >> opt >> x;
        if (opt == 1) insert(rt, x, -1e7, 1e7);
        else if (opt == 2) del(rt, x, -1e7, 1e7);
        else if (opt == 3) cout << rk(x, -1e7, 1e7) << '\n';
        else if (opt == 4) cout << kth(rt, x, -1e7, 1e7) << '\n';
        else if (opt == 5) cout << pre(rt, x, -1e7, 1e7) << '\n';
        else cout << suc(rt, x, -1e7, 1e7) << '\n';
    }
    return 0;
}
```

## [扫描线] 模板

> 求 $n$ 个四边平行于坐标轴的矩形的面积并。
>
> **输入格式**
>
> 第一行一个正整数 $n$。
>
> 接下来 $n$ 行每行四个非负整数 $x_1, y_1, x_2, y_2$，表示一个矩形的四个端点坐标为 $(x_1, y_1),(x_1, y_2),(x_2, y_2),(x_2, y_1)$。
>
> **输出格式**
>
> 一行一个正整数，表示 $n$ 个矩形的并集覆盖的总面积。
>
> **数据范围**
>
> 对于 $100\%$ 的数据，$1 \le n \le {10}^5$，$0 \le x_1 < x_2 \le {10}^9$，$0 \le y_1 < y_2 \le {10}^9$。
>
> 题目链接：[P5490](https://www.luogu.com.cn/problem/P5490)。

将矩形拆成左右两条线段，左边权值 $1$，右边权值 $-1$，然后按照 $x$ 坐标升序排序，用线段树维护非零区间的长度之和。注意线段树的每个结点都表示线段 $[L,R+1]$ 的长度，这样才可以保证不重不漏覆盖到每个区间。

由于每次修改都是对称的操作，而且每个区间只在乎有没有被覆盖到，所以不需要 `pushdown` 操作，可以分类讨论一下：

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;

const int N = 200010;
vector<int> nums;
int n;

struct Segment {
    int x, y1, y2, k;

    bool operator<(const Segment& s) const {
        return x < s.x;
    }
} seg[N];

struct Node {
    int l, r;
    int cnt, len;
} tr[N<<2];

void pushup(int u) {
    if (tr[u].cnt)
        tr[u].len = nums[tr[u].r+1] - nums[tr[u].l];
    else if (tr[u].l != tr[u].r)
        tr[u].len = tr[u<<1].len + tr[u<<1|1].len;
    else
        tr[u].len = 0;
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) return;
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
}

void modify(int u, int l, int r, int k) {
    if (l <= tr[u].l && tr[u].r <= r) return tr[u].cnt += k, pushup(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) modify(u<<1, l, r, k);
    if (mid+1 <= r) modify(u<<1|1, l, r, k);
    pushup(u);
}

int find(int x) {
    return lower_bound(nums.begin(), nums.end(), x) - nums.begin();
}

int main() {
    scanf("%d", &n);

    for (int i = 0, j = 0; i < n; i++) {
        int x1, x2, y1, y2;
        scanf("%d%d%d%d", &x1, &y1, &x2, &y2);
        seg[j++] = {x1, y1, y2, 1};
        seg[j++] = {x2, y1, y2, -1};
        nums.emplace_back(y1), nums.emplace_back(y2);
    }

    sort(nums.begin(), nums.end());
    nums.erase(unique(nums.begin(), nums.end()), nums.end());
    build(1, 0, nums.size()-2);

    sort(seg, seg+2*n);

    LL res = 0;
    for (int i = 0; i < 2*n; i++) {
        if (i) res += 1LL * (seg[i].x - seg[i-1].x) * tr[1].len;
        modify(1, find(seg[i].y1), find(seg[i].y2)-1, seg[i].k);
    }
    return !printf("%lld\n", res);
}
```

## [扫描线] 窗口的星星

> **输入格式**
>
> 本题有多组数据，第一行为 $T$，表示有 $T$ 组数据。
>
> 对于每组数据：
>
> 第一行 $3$ 个整数 $n,W,H$ 表示有 $n$ 颗星星，窗口宽为 $W$，高为 $H$。
>
> 接下来 $n$ 行，每行三个整数 $x_i,y_i,l_i$ 表示星星的坐标在 $(x_i,y_i)$，亮度为 $l_i$。
>
> **输出格式**
>
> $T$ 个整数，表示每组数据中窗口星星亮度总和的最大值，在边框上的不算在内。
>
> **数据范围**
>
> 对于 $100\%$ 的数据：$1\le T \le 10$，$1\le n \le 10^4$，$1\le W,H \le 10^6$，$0\le x_i,y_i < 2^{31}$。
>
> 题目链接：[P1502](https://www.luogu.com.cn/problem/P1502)。

我也不知道 $l_i$ 的范围是多少，这里开个 `long long` 一定没问题。

所以现在问题可以转化成每颗星星如果在 $x$ 位置被添加进来，就要在 $x+W$ 位置被删掉，所以可以用扫描线的思想。

现在我们可以假定有了两条间隔为 $W$ 的竖线，对于中间的点，可以想像成把他们压缩到一个序列中，现在要做的事情就是找到一个长度为 $H$ 的区间，使得区间中数的和最大。

直接做当然不好做，我们可以用右端点代替选择的区间，那么每个点 $y$ 都会在 $[y,y+H-1]$ 做出贡献，可以看出这样不会选中两个间距 $\Delta y\ge H$ 的点，因此对每个点都在它们对应的区间进行修改即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 20010;

struct Segment {
    int x, y1, y2, l;
    bool operator<(const Segment& s) const {
        return x < s.x;
    }
};

struct Node {
    int l, r;
    LL mx, add;
} tr[N<<2];

vector<int> nums;
vector<Segment> segs;

int find(int x) {
    return lower_bound(nums.begin(), nums.end(), x) - nums.begin();
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    tr[u].mx = tr[u].add = 0;
    if (l == r) return;
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
}

void pushup(int u) {
    tr[u].mx = max(tr[u<<1].mx, tr[u<<1|1].mx);
}

void spread(int u, int add) {
    tr[u].add += add;
    tr[u].mx += add;
}

void pushdown(int u) {
    if (tr[u].add) {
        spread(u<<1, tr[u].add);
        spread(u<<1|1, tr[u].add);
        tr[u].add = 0;
    }
}

void modify(int u, int l, int r, int k) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u, k);
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) modify(u<<1, l, r, k);
    if (mid+1 <= r) modify(u<<1|1, l, r, k);
    pushup(u);
}

int main() {
    int T, W, H, n;
    scanf("%d", &T);
    while (T--) {
        scanf("%d%d%d", &n, &W, &H);

        segs.clear();
        nums.clear();
        for (int i = 0, x, y, l; i < n; i++) {
            scanf("%d%d%d", &x, &y, &l);
            segs.push_back({x, y, y+H-1, l});
            segs.push_back({x+W, y, y+H-1, -l});
            nums.push_back(y), nums.push_back(y+H-1);
        }
        sort(segs.begin(), segs.end());
        sort(nums.begin(), nums.end());
        nums.erase(unique(nums.begin(), nums.end()), nums.end());

        build(1, 0, nums.size()-1);

        LL res = 0;
        for (int i = 0; i < segs.size(); i++) {
            int j = i;
            while (j < segs.size() && segs[j].x == segs[i].x) {
                modify(1, find(segs[j].y1), find(segs[j].y2), segs[j].l);
                j++;
            }
            res = max(res, tr[1].mx);
            i = j-1;
        }
        printf("%lld\n", res);
    }
    return 0;
}
```

## [NOI2016] 区间
> 在数轴上有 $n$ 个闭区间从 $1$ 至 $n$ 编号，第 $i$ 个闭区间为 $[l_i,r_i]$ 。
>
> 现在要从中选出 $m$ 个区间，使得这 $m$ 个区间共同包含至少一个位置。换句话说，就是使得存在一个 $x$ ，使得对于每一个被选中的区间 $[l_i,r_i]$，都有 $l_i \leq x \leq r_i$ 。
>
> 对于一个合法的选取方案，它的花费为被选中的最长区间长度减去被选中的最短区间长度。
>
> 区间 $[l_i,r_i]$ 的长度定义为 $(r_i-l_i)$ ，即等于它的右端点的值减去左端点的值。
>
> 求所有合法方案中最小的花费。如果不存在合法的方案，输出 $-1$。
>
> **输入格式**
>
> 第一行包含两个整数，分别代表 $n$ 和 $m$。
>
> 第 $2$ 到第 $(n + 1)$ 行，每行两个整数表示一个区间，第 $(i + 1)$ 行的整数 $l_i, r_i$ 分别代表第 $i$ 个区间的左右端点。
>
> **输出格式**
>
> 输出一行一个整数表示答案。
>
> **数据范围**
>
> 对于全部的测试点，保证 $1 \leq m \leq n$，$1 \leq n \leq 5 \times 10^5$，$1 \leq m \leq 2 \times 10^5$，$0 \leq l_i \leq r_i \leq 10^9$。
>
> 题目链接：[P1712](https://www.luogu.com.cn/problem/P1712)。

首先观察到区间的顺序不影响答案，所以先按照长度升序排序。

由于我们要使得极差最小，如果排序后的 $[l,r]$ 内的区间满足要求，当 $r$ 向右移动的时候，$l$ 向左一定是不比当前答案优，所以 $l,r$ 都会向右移动。

我们只需要判断加入这些区间后存在点被覆盖了 $m$ 次，并且删除 $l$ 区间后会使得覆盖 $<m$ 次，那么在 $[l,r]$ 中选中 $m$ 个互不相交的区间就一定包含了 $l,r$ 两个区间。

P.S. 上面说的 $l,r$ 都代表的是排好序的区间的索引。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 500010, M = 1000010;
int n, m;

struct Node {
    int l, r;
    int mx;
    int add;
} tr[M<<2];

void pushup(int u) {
    tr[u].mx = max(tr[u<<1].mx, tr[u<<1|1].mx);
}

void spread(int u, int add) {
    tr[u].add += add;
    tr[u].mx += add;
}

void pushdown(int u) {
    if (tr[u].add) {
        spread(u<<1, tr[u].add);
        spread(u<<1|1, tr[u].add);
        tr[u].add = 0;
    }
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) return;
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
}

void modify(int u, int l, int r, int k) {
    if (l <= tr[u].l && tr[u].r <= r) return spread(u, k);
    pushdown(u);
    int mid = (tr[u].l + tr[u].r) >> 1;
    if (l <= mid) modify(u<<1, l, r, k);
    if (mid+1 <= r) modify(u<<1|1, l, r, k);
    pushup(u);
}

struct Range {
    int l, r, len;

    bool operator<(const Range& r) const {
        return len < r.len;
    }
} rng[N];

vector<int> nums;

int find(int x) {
    return lower_bound(nums.begin(), nums.end(), x) - nums.begin();
}

int main() {
    scanf("%d%d", &n, &m);

    // indexed from 1
    nums.emplace_back(-1);
    for (int i = 0; i < n; i++) {
        scanf("%d%d", &rng[i].l, &rng[i].r);
        rng[i].len = rng[i].r - rng[i].l;
        nums.emplace_back(rng[i].l);
        nums.emplace_back(rng[i].r);
    }
    sort(rng, rng+n);
    sort(nums.begin(), nums.end());
    nums.erase(unique(nums.begin(), nums.end()), nums.end());

    for (int i = 0; i < n; i++) rng[i].l = find(rng[i].l), rng[i].r = find(rng[i].r);

    build(1, 1, nums.size()-1);
    int l = 0, r = 0;

    int ans = 0x3f3f3f3f;
    while (r < n) {
        while (r < n && tr[1].mx < m) {
            modify(1, rng[r].l, rng[r].r, 1);
            r++;
        }

        while (l < r && tr[1].mx == m) {
            modify(1, rng[l].l, rng[l].r, -1);
            if (tr[1].mx < m) ans = min(ans, rng[r-1].len-rng[l].len);
            l++;
        }
    }

    if (ans == 0x3f3f3f3f) ans = -1;
    return !printf("%d\n", ans);
}
```

## [CF1093E] Intersection of Permutations

> 给定整数 $n$ 和两个 $1,\dots,n$ 的排列 $a,b$ 与 $m$ 个操作，操作有两种：
>
>   - $1\ l_a\ r_a\ l_b\ r_b$，设 $a$ 的 $[l_a,r_a]$ 区间内的元素集合为 $S_a$，设 $b$ 的 $[l_b,r_b]$ 区间内的元素集合为 $S_b$，求 $\lvert S_a \cap S_b \rvert$。
>   - $2\ x\ y$，交换 $b$ 的第 $x$ 位与第 $y$ 位。
>
> 数据范围：$1 \le n,m \le 2 \times 10^5$。
>
> 题目链接：[CF1093E](http://codeforces.com/problemset/problem/1093/E)。

构造 $a_i$ 的逆映射 $rev_i$ 使得 $rev_{a_i}=i$，此时在 $b$ 的 $[l,r]$ 版本中添加 $rev_{b_l},\dots,rev_{b_r}$，则查询 $[l_b,r_b]$ 中查询 $[l_a,r_a]$ 的区间和即可。

需要用 BIT 套一个线段树，由于空间不够所以需要有回收内存的机制，这样就比较慢了，但是还是可以通过本题的。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define lowbit(x) ((x)&(-x))
#define Mid ((L+R) >> 1)
const int N = 200010, M = 40000010;
int n, m, a[N], b[N], rev[N];

struct VSGT {
    int idx;
    int cnt[M], ls[M], rs[M];
    vector<int> freenodes;

    void pushup(int u) {
        cnt[u] = cnt[ls[u]] + cnt[rs[u]];
    }

    int create() {
        int t;
        if (freenodes.empty()) t = ++idx;
        else t = freenodes.back(), freenodes.pop_back();
        cnt[t] = ls[t] = rs[t] = 0;
        return t;
    }

    int qry(int u, int p, int L, int R) {
        if (L == R) return cnt[u];
        if (p <= Mid) return qry(ls[u], p, L, Mid);
        else return qry(rs[u], p, Mid+1, R);
    }

    int query(int u, int l, int r, int L, int R) {
        if (!u) return 0;
        if (l <= L && R <= r) return cnt[u];
        int res = 0;
        if (l <= Mid) res += query(ls[u], l, r, L, Mid);
        if (Mid+1 <= r) res += query(rs[u], l, r, Mid+1, R);
        return res;
    }

    void modify(int& u, int p, int k, int L, int R) {
        if (!u) u = create();
        if (L == R) return cnt[u] += k, void();
        if (p <= Mid) modify(ls[u], p, k, L, Mid);
        else modify(rs[u], p, k, Mid+1, R);
        pushup(u);
        if (cnt[u] == 0) freenodes.emplace_back(u), u = 0;
    }
} T;

struct BIT {
    int rt[N];
    void modify(int p, int v, int k) {
        for (; p <= n; p += lowbit(p)) {
            T.modify(rt[p], v, k, 1, n);
        }
        T.qry(rt[0], 1, 1, n);
    }

    int query(int p, int l, int r) {
        int res = 0;
        for (; p; p -= lowbit(p)) {
            res += T.query(rt[p], l, r, 1, n);
        }
        return res;
    }
} bit;

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]), rev[a[i]] = i;
    for (int i = 1; i <= n; i++) scanf("%d", &b[i]), b[i] = rev[b[i]];

    for (int i = 1; i <= n; i++) bit.modify(i, b[i], 1);
    for (int i = 1, opt, x, y, p, q; i <= m; i++) {
        scanf("%d%d%d", &opt, &x, &y);
        if (opt == 1) {
            scanf("%d%d", &p, &q);
            printf("%d\n", bit.query(q, x, y) - bit.query(p-1, x, y));
        }
        else {
            bit.modify(x, b[x], -1);
            bit.modify(x, b[y], 1);
            bit.modify(y, b[x], 1);
            bit.modify(y, b[y], -1);
            swap(b[x], b[y]);
        }
    }

    return 0;
}
```

## [JSOI2009] 等差数列

> 给定长度为 $n$ 的数列，$q$ 次操作，将 $[l,r]$ 插入一个等差数列或者询问 $[l,r]$ 最少可以划分为几个等差数列，$1\le n,q\le 10^5$。
>
> 题目链接：[P4243](https://www.luogu.com.cn/problem/P4243)。

首先转化为差分，区间加线段树可以维护，询问就是询问形如 $f(A_1B_1B_1A_2B_2A_3B_3B_3)=3$ 这种形式，并且第一项有可能等于后面若干项，看起来是难以合并的。

转换一下，如果我们强制删掉第一项，也就是维护 $f(B_1B_1A_2B_2A_3B_3B_3)=3$，查询的时候就要查 $[l+1,r]$ 的答案了，特判掉 $l=r$ 的情况即可。

所以合并 $[l_1,r_1],[l_2,r_2]$ 的时候枚举 $r_1,l_2$ 哪一个作为后面的开头，用 $f(0/1,0/1)$ 代表区间是 开/闭 区间。

对于初始化，$f(0,0)=0,f(0,1)=f(1,0)=f(1,1)=1$，可以将半开半闭区间看作一个数轴上的一个点，所以这样对应的答案实际意义是用一个开头查询长度为 $0$ 的后缀，答案是 $1$；全开区间就没有什么实际意义了，是无效的，所以是 $0$ 个方案。

转移的时候就是：
$$
f(1,1)=\min\begin{cases}
f_1(1,1)+f_2(1,1)-[a_{r_1}=a_{l_2}]\\
f_1(1,0)+f_2(1,1)\\
f_1(1,1)+f_2(0,1)
\end{cases}
$$
下面两种情况分别对应将 $a_{r_1},a_{l_2}$ 当作右区间的开头，第一种情况如果 $a_{r_1}\ne a_{l_2}$ 是不合法的，因为 $[l_2,r_2]$ 包含了 $[l_2+1,r_2]$，但是它一定不会比第三种更优，所以 $\min$ 就可以过滤掉这种非法转移了。

```cpp
#include <bits/stdc++.h>
using namespace std;

namespace IO { /* 省略 */ }
using IO::read;
using IO::write;

#define Mid ((L+R) >> 1)
const int N = 100010;

struct Data {
    int l, r;
    int f[2][2];
} dat[N<<2];

Data operator+(const Data& l, const Data& r) {
    Data d = {l.l, r.r};
    int k = l.r == r.l;
    d.f[1][1] = min(l.f[1][1] + r.f[1][1] - k, min(l.f[1][0] + r.f[1][1], l.f[1][1] + r.f[0][1]));
    d.f[1][0] = min(l.f[1][1] + r.f[1][0] - k, min(l.f[1][0] + r.f[1][0], l.f[1][1] + r.f[0][0]));
    d.f[0][1] = min(l.f[0][1] + r.f[1][1] - k, min(l.f[0][0] + r.f[1][1], l.f[0][1] + r.f[0][1]));
    d.f[0][0] = min(l.f[0][1] + r.f[1][0] - k, min(l.f[0][0] + r.f[1][0], l.f[0][1] + r.f[0][0]));
    return d;
}

int n, q, add[N<<2], a[N];

void pushup(int u) {
    dat[u] = dat[u<<1] + dat[u<<1|1];
}

void spread(int u, int k) {
    add[u] += k, dat[u].l += k, dat[u].r += k;
}

void pushdown(int u) {
    spread(u<<1, add[u]), spread(u<<1|1, add[u]);
    add[u] = 0;
}

void build(int u, int L, int R) {
    if (L == R) {
        dat[u] = {a[L], a[L]};
        dat[u].f[1][1] = dat[u].f[1][0] = dat[u].f[0][1] = 1;
        return;
    }
    build(u<<1, L, Mid), build(u<<1|1, Mid+1, R);
    pushup(u);
}

Data query(int u, int l, int r, int L, int R) {
    if (l <= L && R <= r) return dat[u];
    pushdown(u);
    if (l > Mid) return query(u<<1|1, l, r, Mid+1, R);
    if (Mid+1 > r) return query(u<<1, l, r, L, Mid);
    return query(u<<1, l, r, L, Mid) + query(u<<1|1, l, r, Mid+1, R);
}

void modify(int u, int l, int r, int k, int L, int R) {
    if (l <= L && R <= r) return spread(u, k);
    pushdown(u);
    if (l <= Mid) modify(u<<1, l, r, k, L, Mid);
    if (Mid+1 <= r) modify(u<<1|1, l, r, k, Mid+1, R);
    pushup(u);
}

int main() {
    read(n);
    for (int i = 1; i <= n; i++) read(a[i]);
    for (int i = n; i; i--) a[i] -= a[i-1];
    build(1, 1, n);
    read(q);
    for (int i = 1, l, r, a, b; i <= q; i++) {
        char op;
        read(op);
        if (op == 'A') {
            read(l, r, a, b);
            modify(1, l, l, a, 1, n);
            if (l < r) modify(1, l+1, r, b, 1, n);
            if (r < n) modify(1, r+1, r+1, -a - (r-l) * b, 1, n);
        }
        else {
            read(l, r);
            if (l == r) write(1);
            else write(query(1, l+1, r, 1, n).f[1][1]);
        }
    }
    return 0;
}
```

