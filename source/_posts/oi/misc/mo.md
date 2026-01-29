---
date: 2023-12-30 14:46:00
updated: 2024-01-12 02:22:00
title: 杂项 - 莫队
katex: true
tags:
- Algo
- C++
categories:
- OI
description: 莫队算法是利用平衡复杂度的思想来优化暴力算法，通常是通过均值不等式求出一个最优复杂度。
---

## 简介

莫队算法是利用平衡复杂度的思想来优化暴力算法，通常是通过均值不等式求出一个最优复杂度。

如果一个问题的暴力方式询问和修改复杂度差距很大，假设是求区间和的题，维护原数组是修改 $O(1)$，查询 $O(n)$。如果我们分块并且处理出每块答案，块长为 $B$，那么修改仍然是 $O(1)$，但是查询变成了 $O(\frac{n}{B}+B)$，均值不等式可以得到当 $B=\sqrt{n}$ 时有最优复杂度单次 $O(\sqrt n)$。

类似地，莫队算法也是分块，不过是对查询分块，如果一个问题知道 $[l,r]$ 区间的答案可以 $O(1)$ 转移到 $[l,r+1],[l+1,r],[l-1,r],[l,r-1]$ 的话，就可以用莫队算法，对 $l$ 分块并对 $r$ 升序排序，当块长为 $B$ 时：

1. $l$ 移动次数是每次询问都在块内移动 $mB$。
2. $r$ 移动次数是每次换块都在整个区间上移动 $\frac{n^2}{B}$

故当 $B=\frac{n}{\sqrt m}$ 时取到最小值 $O(n\sqrt m)$ 复杂度。

除了这种普通的莫队算法以外，如果带修改，可以将修改视为第三维坐标；还可以将树通过括号序压缩到一个线性表上从而支持树上操作；当添加/删除其中一者简单另一者难时，可以记录下每块的历史操作，当需要进行难的操作时直接撤销历史操作，将删除与添加互换。

注意：指针移动时需要**先添加元素，后删除元素**，最好不要反过来，因为删除的时候有可能出现因为没有添加到正在维护的数据结构导致删除无效。

## 不带修莫队

### 小 B 的询问

> 小 B 有一个长为 $n$ 的整数序列 $a$，值域为 $[1,k]$。他一共有 $m$ 个询问，每个询问给定一个区间 $[l,r]$，求：
> $$
> \sum\limits_{i=1}^k c_i^2
> $$
>
>
> 其中 $c_i$ 表示数字 $i$ 在 $[l,r]$ 中的出现次数。
>
> 对于 $100\%$ 的数据，$1\le n,m,k \le 5\times 10^4$。
>
> 题目链接：[P2709](https://www.luogu.com.cn/problem/P2709)。

模板题，没什么好说的。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 50010, M = 320;
int n, m, k, now, a[N], cnt[N], ans[N];

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

struct Query {
    int l, r, id;
    bool operator<(const Query& q) const {
        // 分奇偶排序是常数优化，原理很简单，从左到右再从右到左
        if (l / M == q.l / M) return ((l / M) & 1) ? r < q.r : r > q.r;
        return l < q.l;
    }
} qry[N];

void add(int x) {
    now -= cnt[a[x]] * cnt[a[x]];
    cnt[a[x]]++;
    now += cnt[a[x]] * cnt[a[x]];
}

void sub(int x) {
    now -= cnt[a[x]] * cnt[a[x]];
    cnt[a[x]]--;
    now += cnt[a[x]] * cnt[a[x]];
}

signed main() {
    read(n), read(m), read(k);
    for (int i = 1; i <= n; i++) read(a[i]);
    for (int i = 1; i <= m; i++) read(qry[i].l), read(qry[i].r), qry[i].id = i;
    sort(qry+1, qry+1+m);

    int l = 1, r = 0;
    for (int i = 1; i <= m; i++) {
        while (l > qry[i].l) add(--l);
        while (r < qry[i].r) add(++r);
        while (l < qry[i].l) sub(l++);
        while (r > qry[i].r) sub(r--);
        ans[qry[i].id] = now;
    }

    for (int i = 1; i <= m; i++) printf("%lld\n", ans[i]);

    return 0;
}
```

### [国家集训队] 小 Z 的袜子

> 小 Z 把这 $N$ 只袜子从 $1$ 到 $N$ 编号，然后从编号 $L$ 到 $R$ 的袜子中随机选出两只来穿。尽管小 Z 并不在意两只袜子是不是完整的一双，他却很在意袜子的颜色，毕竟穿两只不同色的袜子会很尴尬。
>
> 你的任务便是告诉小 Z，他有多大的概率抽到两只颜色相同的袜子。当然，小 Z 希望这个概率尽量高，所以他可能会询问多个 $(L,R)$ 以方便自己选择。
>
> 然而数据中有 $L=R$ 的情况，请特判这种情况，输出 `0/1`。
>
> $100\%$ 的数据中，$N,M \leq 50000$，$1 \leq L < R \leq N$，$C_i \leq N$。
>
> 题目链接：[P1494](https://www.luogu.com.cn/problem/P1494)。

设 $c_i$ 为 $i$ 的出现次数，所以答案就是：
$$
\frac{\sum \binom{c_i}{2}}{\binom{r-l+1}{2}}
$$

求个 $\gcd$ 化简一下就好了。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 50010, M = 250;
int n, m, now, c[N], cnt[N], ans1[N], ans2[N];

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

struct Query {
    int l, r, id;
    bool operator<(const Query& q) const {
        if (l / M == q.l / M) return ((l / M) & 1) ? r < q.r : r > q.r;
        return l < q.l;
    }
} qry[N];

int gcd(int a, int b) {
    if (!b) return a;
    return gcd(b, a % b);
}

int C2(int x) {
    return x * (x-1) / 2;
}

void add(int x) {
    now -= C2(cnt[c[x]]++);
    now += C2(cnt[c[x]]);
}

void sub(int x) {
    now -= C2(cnt[c[x]]--);
    now += C2(cnt[c[x]]);
}

signed main() {
    read(n), read(m);
    for (int i = 1; i <= n; i++) read(c[i]);
    for (int i = 1; i <= m; i++) read(qry[i].l), read(qry[i].r), qry[i].id = i;
    sort(qry+1, qry+1+m);

    int l = 1, r = 0;
    for (int i = 1; i <= m; i++) {
        while (l > qry[i].l) add(--l);
        while (r < qry[i].r) add(++r);
        while (l < qry[i].l) sub(l++);
        while (r > qry[i].r) sub(r--);
        int a = now, b = C2(qry[i].r - qry[i].l + 1);
        if (!a || !b) ans1[qry[i].id] = 0, ans2[qry[i].id] = 1;
        else {
            int d = gcd(a, b);
            ans1[qry[i].id] = a / d, ans2[qry[i].id] = b / d;
        }
    }

    for (int i = 1; i <= m; i++) {
        printf("%lld/%lld\n", ans1[i], ans2[i]);
    }

    return 0;
}
```

### [WC2022] 秃子酋长

> 给定长度为 $n$ 的数列 $a$，$m$ 次询问 $[l,r]$ 内，排序后相邻的树在原序列中位置差的绝对值之和，保证 $a_i$ 互不相同。
>
> 数据范围：$1\le n,m\le 5\times 10^5,1\le a_i\le n$
>
> 题目链接：[P8078](https://www.luogu.com.cn/problem/P8078)。

添加或删除一个元素影响的是它在当前维护的数字中的前驱和后继，所以可以用一个平衡树来找前驱后继，为了减小常数使用压位 Trie，下面在回滚莫队中也有一道题目用了类似的方法。

具体地说，构造出 $a_i$ 逆映射 $ra_i$，添加一个元素就相当于改变了 $\text{pre}(a_i)\to \text{suc}(a_i)$ 为 $\text{pre}(a_i)\to a_i\to \text{suc}(a_i)$，判断一下前驱后继是否存在即可，删除也是类似的操作。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define B(x) ((x - 1) / M + 1)
#define lg2highbit(x) (63 - __builtin_clzll(x))
#define lg2lowbit(x) (__builtin_ctzll(x))
typedef unsigned long long ull;
const int N = 500010, M = 710, G = 4;
int n, m, a[N], ra[N], L, R;
long long ans[N], now;

template<typename T = int>
inline T read() {
    T t = 0;
    bool neg = 0;
    char ch = getchar();
    while (!isdigit(ch)) neg |= ch == '-', ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
    return neg ? -t : t;
}

struct Trie {
    ull* A[G];

    Trie() {
        int sz = 1;
        for (int i = G-1; i >= 0; i--, sz <<= 6) A[i] = new ull[sz]();
    }

    inline void ins(int x) {
        for (int i = 0; i < G; i++, x >>= 6) {
            ull mask = 1ull << (x & 63);
            if (A[i][x >> 6] & mask) return;
            A[i][x >> 6] |= mask;
        }
    }

    inline void del(int x) {
        for (int i = 0; i < G; i++, x >>= 6) {
            if (A[i][x >> 6] &= ~(1ull << (x & 63))) return;
        }
    }

    int pre(int x) {
        int i, res;
        for (i = 0; i < G; i++, x >>= 6) {
            ull s = A[i][x >> 6] & ((1ull << (x & 63)) - 1);
            if (!s) continue;
            res = x & ~63 | lg2highbit(s);
            break;
        }
        if (i >= G) return 0;
        for (; i; i--) res = res << 6 | lg2highbit(A[i-1][res]);
        return res;
    }

    int suc(int x) {
        int i, res;
        for (i = 0; i < G; i++, x >>= 6) {
            ull s = A[i][x >> 6] & (~((1ull << (x & 63)) - 1) << 1);
            if (!s) continue;
            res = x & ~63 | lg2lowbit(s);
            break;
        }
        if (i >= G) return 0;
        for (; i; i--) res = res << 6 | lg2lowbit(A[i-1][res]);
        return res;
    }

    ~Trie() {
        for (int i = 0; i < G; i++) delete[] A[i];
    }
} trie;

struct Query {
    int id, l, r;
    inline bool operator<(const Query& q) const {
        if (B(l) != B(q.l)) return l < q.l;
        return B(l) % 2 ? r > q.r : r < q.r;
    }
} qry[N];

inline void add(int p) {
    int pre = trie.pre(a[p]), suc = trie.suc(a[p]);
    if (pre && suc) now -= abs(ra[suc] - ra[pre]);
    if (pre) now += abs(p - ra[pre]);
    if (suc) now += abs(p - ra[suc]);
    trie.ins(a[p]);
}

inline void del(int p) {
    trie.del(a[p]);
    int pre = trie.pre(a[p]), suc = trie.suc(a[p]);
    if (pre && suc) now += abs(ra[suc] - ra[pre]);
    if (pre) now -= abs(p - ra[pre]);
    if (suc) now -= abs(p - ra[suc]);
}

int main() {
    n = read(), m = read();
    for (int i = 1; i <= n; i++) ra[a[i] = read()] = i;
    for (int i = 1; i <= m; i++) qry[i].id = i, qry[i].l = read(), qry[i].r = read();
    sort(qry+1, qry+1+m);

    L = 1, R = 0;
    for (int i = 1, l, r; i <= m; i++) {
        l = qry[i].l, r = qry[i].r;
        while (R < r) add(++R);
        while (L > l) add(--L);
        while (R > r) del(R--);
        while (L < l) del(L++);
        ans[qry[i].id] = now;
    }

    for (int i = 1; i <= m; i++) printf("%lld\n", ans[i]);
    return 0;
}
```

## 带修莫队

### [国家集训队] 数颜色

> 墨墨购买了一套 $N$ 支彩色画笔（其中有些颜色可能相同），摆成一排，你需要回答墨墨的提问。墨墨会向你发布如下指令：
>
> 1. $Q\ L\ R$ 代表询问你从第 $L$ 支画笔到第 $R$ 支画笔中共有几种不同颜色的画笔。
>
> 2. $R\ P\ C$ 把第 $P$ 支画笔替换为颜色 $C$。
>
> 为了满足墨墨的要求，你知道你需要干什么了吗？
>
> 对于所有数据，$n,m \leq 133333$
>
> 所有的输入数据中出现的所有整数均大于等于 $1$ 且不超过 $10^6$。
>
> 题目链接：[P1903](https://www.luogu.com.cn/problem/P1903)。

添加时间维度 $t$，设块长为 $B$，则有：

1. 移动 $l$ 是 $nB$ 次。
2. 移动 $r$ 是 $\frac{n^2}{B}$ 次。
3. 移动 $t$ 是 $\frac{n^3}{B^2}$ 次。

取 $B=n^{\frac{2}{3}}$ 有复杂度为 $O(n^{\frac{5}{3}})$，能够接受。移动时间时，判断是否会影响当前区间维护的答案，如果是则更新。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 140010, M = 2700, K = 1000010;
int n, m, k, now, a[N], cnt[K], ans[N];
int pos[N], val[N];

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

void readalpha(char& ch) {
    ch = getchar();
    while (!isalpha(ch)) ch = getchar();
}

struct Query {
    int l, r, t, id;
    bool operator<(const Query& q) const {
        if (l / M != q.l / M) return l < q.l;
        if (r / M != q.r / M) return r < q.r;
        return ((r / M) & 1) ? t < q.t : t > q.t;
    }
} qry[N];

void add(int x) {
    now += !cnt[a[x]]++;
}

void sub(int x) {
    now -= !--cnt[a[x]];
}

void upd(int l, int r, int t) {
    if (l <= pos[t] && pos[t] <= r) sub(pos[t]);
    swap(a[pos[t]], val[t]);
    if (l <= pos[t] && pos[t] <= r) add(pos[t]);
}

signed main() {
    read(n), read(m);
    for (int i = 1; i <= n; i++) read(a[i]);

    int nqrc = 0, nqry = 0;
    for (int i = 1; i <= m; i++) {
        char ch;
        readalpha(ch);
        if (ch == 'Q') ++nqry, read(qry[nqry].l), read(qry[nqry].r), qry[nqry].id = nqry, qry[nqry].t = nqrc;
        else ++nqrc, read(pos[nqrc]), read(val[nqrc]);
    }
    sort(qry+1, qry+1+nqry);

    int l = 1, r = 0, t = 0;
    for (int i = 1; i <= nqry; i++) {
        while (l > qry[i].l) add(--l);
        while (r < qry[i].r) add(++r);
        while (l < qry[i].l) sub(l++);
        while (r > qry[i].r) sub(r--);
        while (t > qry[i].t) upd(l, r, t--);
        while (t < qry[i].t) upd(l, r, ++t);
        ans[qry[i].id] = now;
    }

    for (int i = 1; i <= nqry; i++) printf("%lld\n", ans[i]);

    return 0;
}
```

### [CF940F] Machine Learning

> 给定 $n$ 个数 $a_i$，$m$ 次询问：
>
> 1. 查询对 $[l,r]$ 内每个数出现次数的 $mex$。
> 2. 单点修改。
>
> 其中 $n,m\le 10^5$。
>
> 题目链接：[CF940F](https://www.luogu.com.cn/problem/solution/CF940F)。

若一个区间的 $mex=p$，一定有 $sum=O(p^2)$，也就是说区间的 $mex$ 是区间和的根号级别，而区间出现次数之和是 $O(n)$，于是本题的 $mex$ 为 $O(\sqrt n)$，于是可以暴力。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define eb emplace_back
#define int long long
const int N = 100010, M = 2200;
int n, m, k, now, a[N], cnt[N<<1], ans[N], tot[N];
int pos[N], val[N];
vector<int> nums;

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

struct Query {
    int l, r, t, id;
    bool operator<(const Query& q) const {
        if (l / M != q.l / M) return l < q.l;
        if (r / M != q.r / M) return r < q.r;
        return ((r / M) & 1) ? t < q.t : t > q.t;
    }
} qry[N];

void add(int x) {
    --tot[cnt[a[x]]];
    ++tot[++cnt[a[x]]];
}

void sub(int x) {
    --tot[cnt[a[x]]];
    ++tot[--cnt[a[x]]];
}

void upd(int l, int r, int t) {
    if (l <= pos[t] && pos[t] <= r) sub(pos[t]);
    swap(a[pos[t]], val[t]);
    if (l <= pos[t] && pos[t] <= r) add(pos[t]);
}

signed main() {
    read(n), read(m);
    for (int i = 1; i <= n; i++) read(a[i]), nums.eb(a[i]);

    int nqrc = 0, nqry = 0;
    for (int i = 1, opt; i <= m; i++) {
        read(opt);
        if (opt == 1) ++nqry, read(qry[nqry].l), read(qry[nqry].r), qry[nqry].id = nqry, qry[nqry].t = nqrc;
        else ++nqrc, read(pos[nqrc]), read(val[nqrc]), nums.eb(val[nqrc]);
    }
    sort(qry+1, qry+1+nqry);
    sort(nums.begin(), nums.end());
    nums.erase(unique(nums.begin(), nums.end()), nums.end());
    for (int i = 1; i <= n; i++) a[i] = lower_bound(nums.begin(), nums.end(), a[i]) - nums.begin() + 1;
    for (int i = 1; i <= nqrc; i++) val[i] = lower_bound(nums.begin(), nums.end(), val[i]) - nums.begin() + 1;

    int l = 1, r = 0, t = 0;
    for (int i = 1; i <= nqry; i++) {
        while (l > qry[i].l) add(--l);
        while (r < qry[i].r) add(++r);
        while (l < qry[i].l) sub(l++);
        while (r > qry[i].r) sub(r--);
        while (t > qry[i].t) upd(l, r, t--);
        while (t < qry[i].t) upd(l, r, ++t);
        int res = 1;
        while (tot[res] >= 1) res++;
        ans[qry[i].id] = res;
    }

    for (int i = 1; i <= nqry; i++) printf("%lld\n", ans[i]);

    return 0;
}
```

## 树上莫队

### Count on a tree II

> 给定 $n$ 点的树，每个点有颜色，$m$ 询问 $a,b$ 路径上颜色的种类数量。$1\le n\le 4\times 10^4,1\le m\le 10^5$。
>
> 题目链接：[SP10707](https://www.luogu.com.cn/problem/SP10707)。

将树用括号序压到一个区间内，指每个点第一次访问和回溯时加入序列，长度为 $2n$，用 $L(u),R(u)$ 表示一个点在括号序中的第一次出现下标和第二次出现下标。

 不妨设 $L(a)\lt L(b)$，$u=LCA(a,b)$，那么分两种情况：

1. $u=a$，即 $a$ 是 $b$ 的祖先，此时 $[L(a),L(b)]$ 就是 $a\sim b$ 的路径上所有点，并且它们仅出现一次。
2. $u\ne a$，那么 $[R(a),L(b)]$ 上经过 $a\sim b$ 路径上除 $u$ 以外的所有点，并且不在路径上的点都出现了两次。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define eb emplace_back
const int N = 40010, M = 210, K = 20, Q = 100010;
int n, m, col[N], fa[N][K], dep[N], L[N], R[N], mark[N], seq[N<<1], idx;
int now, ans[Q], cnt[N];
vector<int> cols, g[N];

struct Query {
    int id, l, r, u;
    bool operator<(const Query& q) const {
        if (l / M == q.l / M) return ((l / M) & 1) ? r < q.r : r > q.r;
        return l < q.l;
    }
} qry[Q];

void update(int u) {
    mark[u] ^= 1;
    if (mark[u]) now += !cnt[col[u]]++;
    else now -= !--cnt[col[u]];
}

void dfs(int u, int f) {
    fa[u][0] = f, dep[u] = dep[f] + 1;
    seq[L[u] = ++idx] = u;
    for (int k = 1; k < K; k++) fa[u][k] = fa[fa[u][k-1]][k-1];
    for (int to: g[u]) if (to != f) dfs(to, u);
    seq[R[u] = ++idx] = u;
}

int lca(int a, int b) {
    if (dep[a] < dep[b]) swap(a, b);
    for (int k = K-1; k >= 0; k--) if (dep[fa[a][k]] >= dep[b]) a = fa[a][k];
    if (a == b) return a;
    for (int k = K-1; k >= 0; k--) if (fa[a][k] != fa[b][k]) a = fa[a][k], b = fa[b][k];
    return fa[a][0];
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &col[i]), cols.eb(col[i]);
    sort(cols.begin(), cols.end());
    for (int i = 1; i <= n; i++) col[i] = lower_bound(cols.begin(), cols.end(), col[i]) - cols.begin();
    for (int i = 1, a, b; i < n; i++) scanf("%d%d", &a, &b), g[a].eb(b), g[b].eb(a);
    dfs(1, 0);
    for (int i = 1, a, b, u; i <= m; i++) {
        scanf("%d%d", &a, &b);
        if (L[a] > L[b]) swap(a, b);
        u = lca(a, b);
        if (u == a) qry[i] = {i, L[a], L[b], 0};
        else qry[i] = {i, R[a], L[b], u};
    }
    sort(qry+1, qry+1+m);

    int l = 1, r = 0;
    for (int i = 1; i <= m; i++) {
        while (r > qry[i].r) update(seq[r--]);
        while (r < qry[i].r) update(seq[++r]);
        while (l > qry[i].l) update(seq[--l]);
        while (l < qry[i].l) update(seq[l++]);

        if (qry[i].u) update(qry[i].u);
        ans[qry[i].id] = now;
        if (qry[i].u) update(qry[i].u);
    }

    for (int i = 1; i <= m; i++) printf("%d\n", ans[i]);

    return 0;
}
```

### [WC2013] 糖果公园

> 给定 $n$ 点树，每棵树上有一种糖果，共 $m$ 种，第 $i$ 种糖果的美味指数是 $V_i$，游客第 $j$ 次尝第 $i$ 个糖果增加的权值是 $V_i\times W_j$，$q$ 次询问：
>
> 1. 修改 $x$ 点的糖果种类为 $y$。
> 2. 询问 $x,y$ 权值。
>
> 数据范围：$1\le n,m,q\le 10^5$。
>
> 题目链接：[P4074](https://www.luogu.com.cn/problem/P4074)。

路径上的权值是 $\sum_{i=1}^m \sum_{j=1}^{\text{cnt}_i}W_j\times V_i$，增量比较好维护，所以可以用树上莫队解决。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define eb emplace_back
const int N = 100010, M = 2160, K = 20;
int n, m, q, fa[N][K], dep[N], L[N], R[N], mark[N], seq[N<<1], idx;
int now, C[N], W[N], V[N], ans[N], cnt[N];
int pos[N], val[N];
vector<int> g[N];

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

struct Query {
    int id, l, r, t, u;
    bool operator<(const Query& q) const {
        if (l / M != q.l / M) return l < q.l;
        if (r / M != q.r / M) return r < q.r;
        return ((r / M) & 1) ? t < q.t : t > q.t;
    }
} qry[N];

void add(int u) {
    now += W[++cnt[C[u]]] * V[C[u]];
}

void sub(int u) {
    now -= W[cnt[C[u]]--] * V[C[u]];
}

void update(int u) {
    mark[u] ^= 1;
    if (mark[u]) add(u);
    else sub(u);
}

void updtime(int t) {
    int u = pos[t];
    if (mark[u]) sub(u);
    swap(C[u], val[t]);
    if (mark[u]) add(u);
}

void dfs(int u, int f) {
    fa[u][0] = f, dep[u] = dep[f] + 1;
    seq[L[u] = ++idx] = u;
    for (int k = 1; k < K; k++) fa[u][k] = fa[fa[u][k-1]][k-1];
    for (int to: g[u]) if (to != f) dfs(to, u);
    seq[R[u] = ++idx] = u;
}

int lca(int a, int b) {
    if (dep[a] < dep[b]) swap(a, b);
    for (int k = K-1; k >= 0; k--) if (dep[fa[a][k]] >= dep[b]) a = fa[a][k];
    if (a == b) return a;
    for (int k = K-1; k >= 0; k--) if (fa[a][k] != fa[b][k]) a = fa[a][k], b = fa[b][k];
    return fa[a][0];
}

signed main() {
    read(n), read(m), read(q);
    for (int i = 1; i <= m; i++) read(V[i]);
    for (int i = 1; i <= n; i++) read(W[i]);
    for (int i = 1, a, b; i < n; i++) read(a), read(b), g[a].eb(b), g[b].eb(a);
    for (int i = 1; i <= n; i++) read(C[i]);
    dfs(1, 0);

    int qcnt = 0, tcnt = 0;
    for (int i = 1, opt, a, b, u; i <= q; i++) {
        read(opt), read(a), read(b);
        if (opt == 0) ++tcnt, pos[tcnt] = a, val[tcnt] = b;
        else {
            ++qcnt;
            if (L[a] > L[b]) swap(a, b);
            u = lca(a, b);
            if (u == a) qry[qcnt] = {qcnt, L[a], L[b], tcnt, 0};
            else qry[qcnt] = {qcnt, R[a], L[b], tcnt, u};
        }
    }
    sort(qry+1, qry+1+qcnt);

    int l = 1, t = 0, r = 0;
    for (int i = 1; i <= qcnt; i++) {
        while (r > qry[i].r) update(seq[r--]);
        while (r < qry[i].r) update(seq[++r]);
        while (l > qry[i].l) update(seq[--l]);
        while (l < qry[i].l) update(seq[l++]);
        while (t < qry[i].t) updtime(++t);
        while (t > qry[i].t) updtime(t--);

        if (qry[i].u) update(qry[i].u);
        ans[qry[i].id] = now;
        if (qry[i].u) update(qry[i].u);
    }

    for (int i = 1; i <= qcnt; i++) printf("%lld\n", ans[i]);

    return 0;
}
```

## 回滚莫队

### Mex

> 给定 $n$ 个元素 $a_1\sim a_n$，$m$ 次询问，每次询问区间内最小没有出现过的自然数，$1\le n,m\le 2\times 10^5,0\le a_i\le 2\times 10^5$。
>
> 题目链接：[P4137](https://www.luogu.com.cn/problem/P4137)。

可以用主席树在线 $O(n\log n)$ 解决本题，但是也可以用回滚莫队 $O(n\sqrt n)$ 离线解决，这里只是练习一下莫队。

回滚莫队就是无法简单的维护增加，但是维护删除比较简单（或者相反），于是可以将 $l$ 指针每次回滚到块头，$r$ 指针换块时滚到块尾。这里有种可持久化的感觉，但是我们只需要记录下来需要回滚的操作历史，回滚时直接撤销这些操作即可。

第一个块的左边界是 $1$，所以初始化成任意一个非 $1$ 值，每次都回滚到当前块的左边界，当 $l$ 与当前左边界不同时代表换块，重新以 $O(n)$ 的代价初始化，于是复杂度是 $O(n\sqrt n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 200010, M = 450;
int n, m, now, a[N], ans[N], cnt[N];
int pos[N], raw, rawl, idx;

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

void record(int l) {
    idx = 0, raw = now, rawl = l;
}

void resume(int& l) {
    now = raw, l = rawl;
    for (int i = 1; i <= idx; i++) ++cnt[a[pos[i]]];
}

void getans(int l, int r) {
    memset(cnt, 0, sizeof cnt);
    for (int i = l; i <= r; i++) cnt[a[i]]++;
    for (now = 0; cnt[now]; now++);
}

void sub(int p) {
    pos[++idx] = p;
    if (!--cnt[a[p]]) now = min(now, a[p]);
}

struct Query {
    int id, l, r;
    bool operator<(const Query& q) const {
        if ((l-1) / M == (q.l-1) / M) return r > q.r;
        return l < q.l;
    }
} qry[N];

int main() {
    read(n), read(m);
    for (int i = 1; i <= n; i++) read(a[i]);
    for (int i = n+1; i < N; i++) a[i] = N-1;

    for (int i = 1, l, r; i <= m; i++) read(l), read(r), qry[i] = {i, l, r};
    sort(qry+1, qry+1+m);

    int l = 114514, r = 0;
    for (int i = 1; i <= m; i++) {
        int lblo = (qry[i].l - 1) / M * M + 1;
        if (l != lblo) getans(l = lblo, r = n);
        while (r > qry[i].r) sub(r--);

        record(l);
        while (l < qry[i].l) sub(l++);
        ans[qry[i].id] = now;
        resume(l);
    }

    for (int i = 1; i <= m; i++) printf("%d\n", ans[i]);
    return 0;
}
```

### [JOISC2014] 历史研究

> 给定 $n$ 个元素 $a_1\sim a_n$，规定 $a_i$ 的权值是 $\text{cnt}_{a_i}\times a_i$，$m$ 次询问 $[l,r]$ 内最大权值。$1\le n,m\le 10^5,1\le a_i\le 10^9$。
>
> 题目链接：[JOISC2014C](https://atcoder.jp/contests/joisc2014/tasks/joisc2014_c)。

这题是添加元素简单，删除比较困难，所以需要回滚莫队。

首先获取到当前根据 $l$ 分块的块边界 $L,R$ 与询问 $l,r$ 我们想要让它只添加，所以这样操作：

1. 如果 $r\le R$ 直接暴力，复杂度 $O(\sqrt n)$。
2. 如果 $r\gt R$ 由于我们在同一块内按 $r$ 升序排序，所以不断加入新的元素；对于 $l$，每次将其回滚到 $R+1$ 即可。

这样平均复杂度仍为 $O(n\sqrt n)$。调试的时候可以把块长放小，一般都是换块的时候写错了。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define eb emplace_back
const int N = 100010, M = 340;
int n, m, now, a[N], ans[N], cnt[N];
int pos[N], raw, rawl, idx;
vector<int> nums;

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

int bruteforce(int l, int r) {
    int res = 0;
    for (int i = l; i <= r; i++) res = max(res, ++cnt[a[i]] * nums[a[i]]);
    for (int i = l; i <= r; i++) cnt[a[i]] = 0;
    return res;
}

void record(int l) {
    idx = 0, raw = now, rawl = l;
}

void resume(int& l) {
    now = raw, l = rawl;
    for (int i = 1; i <= idx; i++) --cnt[a[pos[i]]];
}

void clear() {
    memset(cnt, 0, sizeof cnt);
    now = 0;
}

void add(int p) {
    pos[++idx] = p;
    now = max(now, ++cnt[a[p]] * nums[a[p]]);
}

struct Query {
    int id, l, r;
    bool operator<(const Query& q) const {
        if ((l-1) / M == (q.l-1) / M) return r < q.r;
        return l < q.l;
    }
} qry[N];

signed main() {
    read(n), read(m);
    for (int i = 1; i <= n; i++) read(a[i]), nums.eb(a[i]);
    sort(nums.begin(), nums.end());
    for (int i = 1; i <= n; i++) a[i] = lower_bound(nums.begin(), nums.end(), a[i]) - nums.begin();

    for (int i = 1, l, r; i <= m; i++) read(l), read(r), qry[i] = {i, l, r};
    sort(qry+1, qry+1+m);

    int l = 1, r = 1e18;
    for (int i = 1; i <= m; i++) {
        int rblo = (qry[i].l - 1) / M * M + M;
        if (l != rblo+1) r = rblo, l = rblo+1, clear();
        // l, r
        if (qry[i].r <= rblo) ans[qry[i].id] = bruteforce(qry[i].l, qry[i].r);
        else {
            while (r < qry[i].r) add(++r);
            record(l);
            while (l > qry[i].l) add(--l);
            ans[qry[i].id] = now;
            resume(l);
        }
    }

    for (int i = 1; i <= m; i++) printf("%lld\n", ans[i]);
    return 0;
}
```

### 最远间隔距离

> 给定 $n$ 个元素 $a_1\sim a_n$，规定在一个区间内 $a_i$ 的权值是 $r_{a_i}-l_{a_i}$，即它最后一次出现的坐标与第一次出现的坐标之差，$m$ 次询问 $[l,r]$ 内的最大权值。$1\le n,m\le 2\times 10^5,1\le a\le 2\times 10^9$。
>
> 题目链接：[P5906](https://www.luogu.com.cn/problem/P5906)。

注意回滚的时候需要检查是否是本次操作的第一次更新，如果不是第一次就忽略掉，所以倒序枚举操作历史。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define eb emplace_back
const int N = 200010, M = 450, INF = 0x3f3f3f3f;
int n, m, now, a[N], ans[N], fir[N], las[N];
int pos[N], rawfir[N], rawlas[N], rawans, rawl, idx;
vector<int> nums;

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

int bruteforce(int l, int r) {
    int res = 0;
    for (int i = l; i <= r; i++) {
        fir[a[i]] = min(fir[a[i]], i);
        res = max(res, i - fir[a[i]]);
    }
    for (int i = l; i <= r; i++) fir[a[i]] = INF;
    return res;
}

void clear() {
    now = 0;
    memset(fir, 0x3f, sizeof fir);
    memset(las, 0, sizeof las);
}

void record(int l) {
    idx = 0, rawans = now, rawl = l;
}

void resume(int& l) {
    now = rawans, l = rawl;
    for (int i = idx; i; i--) fir[pos[i]] = rawfir[i], las[pos[i]] = rawlas[i];
}

void addr(int p) {
    fir[a[p]] = min(fir[a[p]], p);
    las[a[p]] = max(las[a[p]], p);
    now = max(now, las[a[p]] - fir[a[p]]);
}

void addl(int p) {
    pos[++idx] = a[p];
    rawfir[idx] = fir[a[p]], rawlas[idx] = las[a[p]];
    fir[a[p]] = min(fir[a[p]], p);
    las[a[p]] = max(las[a[p]], p);
    now = max(now, las[a[p]] - fir[a[p]]);
}

struct Query {
    int id, l, r;
    bool operator<(const Query& q) const {
        if ((l-1) / M == (q.l-1) / M) return r < q.r;
        return l < q.l;
    }
} qry[N];

signed main() {
    read(n);
    for (int i = 1; i <= n; i++) read(a[i]), nums.eb(a[i]);
    sort(nums.begin(), nums.end());
    nums.erase(unique(nums.begin(), nums.end()), nums.end());
    for (int i = 1; i <= n; i++) a[i] = lower_bound(nums.begin(), nums.end(), a[i]) - nums.begin();
    read(m);
    for (int i = 1, l, r; i <= m; i++) read(l), read(r), qry[i] = {i, l, r};
    sort(qry+1, qry+1+m);

    int l = 1, r = N;
    for (int i = 1; i <= m; i++) {
        int rblo = (qry[i].l - 1) / M * M + M;
        if (l != rblo+1) r = rblo, l = rblo+1, clear();
        if (qry[i].r <= rblo) ans[qry[i].id] = bruteforce(qry[i].l, qry[i].r);
        else {
            while (r < qry[i].r) addr(++r);
            record(l);
            while (l > qry[i].l) addl(--l);
            ans[qry[i].id] = now;
            resume(l);
        }
    }

    for (int i = 1; i <= m; i++) printf("%d\n", ans[i]);
    return 0;
}
```

### [JSOI2009] 面试的考验
> 求区间最接近且**不相等**的两数之差的绝对值。
> 
> 对于 $100\%$ 的数据，$1\le n,q\le10^5$，$1\le a_i\le10^9$。
> 
> 题目链接：[P5926](https://www.luogu.com.cn/problem/P5926)。

处理绝对值的时候，就是查询前驱和后继，如果使用莫队，那么 $O(n\sqrt n)$ 的复杂度已经比较极限了，应该选用一个常数小的数据结构查询，可以是树状数组，也可以是压位 Trie，这里使用压位 Trie 了，搭配离散化进一步降低常数。

添加一个数字并且更新答案是容易的，这个步骤与 [P2234](https://www.luogu.com.cn/problem/P2234) 基本一样，但是删除数字是困难的，所以考虑使用回滚莫队。总复杂度 $O(n\sqrt n\log_{64}n)$，略微卡了下常，速度很快。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define lowbit(x) ((x)&(-x))
#define eb emplace_back
#define B(x) ((x - 1) / M + 1)
#define lg2highbit(x) (63 - __builtin_clzll(x))
#define lg2lowbit(x) (__builtin_ctzll(x))

typedef unsigned long long ull;
const int N = 100010, G = 4, M = 320, INF = 2e9;
vector<int> nums;
int n, q, a[N], ans[N], L, R, now;

inline void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch-'0'), ch = getchar();
}

inline void write(int t) {
    static char stk[20];
    int tp = 0;
    while (t) stk[++tp] = (t % 10) + '0', t /= 10;
    while (tp) putchar(stk[tp--]);
}

struct Query {
    int id, l, r;

    bool operator<(const Query& Q) const {
        if (B(l) == B(Q.l)) return r < Q.r;
        return l < Q.l;
    }
} qry[N];

struct Trie {
    ull* A[G];

    Trie() {
        int sz = 1;
        for (int i = G-1; i >= 0; i--, sz <<= 6) A[i] = new ull[sz]();
    }

    void clear() {
        int sz = 1;
        for (int i = G-1; i >= 0; i--, sz <<= 6) memset(A[i], 0, sz * sizeof(ull));
    }

    void ins(int x) {
        for (int i = 0; i < G; i++, x >>= 6) {
            ull mask = 1ull << (x & 63);
            if (A[i][x >> 6] & mask) return;
            A[i][x >> 6] |= mask;
        }
    }

    void del(int x) {
        for (int i = 0; i < G; i++, x >>= 6) {
            if (A[i][x >> 6] &= ~(1ull << (x & 63))) return;
        }
    }

    int pre(int x) {
        int i, res = 0;
        for (i = 0; i < G; i++, x >>= 6) {
            ull s = A[i][x >> 6] & ((1ull << (x & 63)) - 1);
            if (!s) continue;
            res = x & ~63 | lg2highbit(s);
            break;
        }
        if (i >= G) return 0;
        for (; i; i--) res = res << 6 | lg2highbit(A[i-1][res]);
        return res;
    }

    int suc(int x) {
        int i, res = 0;
        for (i = 0; i < G; i++, x >>= 6) {
            ull s = A[i][x >> 6] & (~((1ull << (x & 63)) - 1) << 1);
            if (!s) continue;
            res = x & ~63 | lg2lowbit(s);
            break;
        }
        if (i >= G) return 0;
        for (; i; i--) res = res << 6 | lg2lowbit(A[i-1][res]);
        return res;
    }

    ~Trie() {
        for (int i = 0; i < G; i++) delete[] A[i];
    }
} force, trie;

int bruteforce(int l, int r) {
    int res = INF;
    for (int i = l, pre, suc; i <= r; i++) {
        force.ins(a[i]), pre = force.pre(a[i]), suc = force.suc(a[i]);
        if (pre) res = min(res, nums[a[i]] - nums[pre]);
        if (suc) res = min(res, nums[suc] - nums[a[i]]);
    }
    for (int i = l; i <= r; i++) force.del(a[i]);
    return res;
}

int rawl, rawnow, stk[N], tp;
void record() {
    rawl = L, rawnow = now, tp = 0;
}

void add(int p) {
    stk[++tp] = a[p];
    trie.ins(a[p]);
    int pre = trie.pre(a[p]), suc = trie.suc(a[p]);
    if (pre) now = min(now, nums[a[p]] - nums[pre]);
    if (suc) now = min(now, nums[suc] - nums[a[p]]);
}

void resume() {
    L = rawl, now = rawnow;
    while (tp) trie.del(stk[tp--]);
}

int main() {
    read(n), read(q);
    nums.reserve(n+1);
    nums.eb(-1);
    for (int i = 1; i <= n; i++) read(a[i]), nums.eb(a[i]);
    sort(nums.begin(), nums.end());
    for (int i = 1; i <= n; i++) a[i] = lower_bound(nums.begin(), nums.end(), a[i]) - nums.begin();
    force.clear();

    for (int i = 1; i <= q; i++) qry[i].id = i, read(qry[i].l), read(qry[i].r);
    sort(qry+1, qry+1+q);

    for (int i = 1, l, r, rbl; i <= q; i++) {
        l = qry[i].l, r = qry[i].r, rbl = (B(l) - 1) * M + M;
        if (L != rbl+1) trie.clear(), L = rbl+1, R = rbl, now = INF;
        if (r <= rbl) ans[qry[i].id] = bruteforce(l, r);
        else {
            while (R < r) add(++R);
            record();
            while (L > l) add(--L);
            ans[qry[i].id] = now;
            resume();
        }
    }

    for (int i = 1; i <= q; i++) write(ans[i]), putchar('\n');
    return 0;
}
```

## 二次离线莫队

### 第十四分块（前体）

> 给定长度为 $n$ 的序列 $a$，$m$ 次查询 $[l,r]$ 中 $\text{popcount}(a_i\oplus a_j)=k$ 且 $i\lt j$ 的 $(i,j)$ 数量。$1\le n,m\lt 10^5,0\le a_i\lt 2^{14}$。
>
> 题目链接：[P4887](https://www.luogu.com.cn/problem/P4887)。

二次离线就是就是在莫队更新块答案的时候再次将询问离线。

对于这个题，首先处理出 $A=\{x\mid \text{popcount}(x)=k\}$，直接暴力枚举即可，根据组合意义可知 $|A|=\binom{\log_2 a+1}{k}\le \binom{14}{7}=3432$。

如果我们知道 $f_i(x)$ 表示 $1\sim i$ 中的数字对 $x$ 异或满足要求的数量的话，$f^{-1}_i(x)$ 表示 $i\sim n$ 中的数字，分析四种拓展方向：

+ 右指针向右加入新元素

   令 $g(r+1)=f_r(a_{r+1})$，可以在 $O(n|A|)$ 的复杂度求出 $g(r)$。

   对于区间 $[l,r]\to [l,r+1]\to [l,R]$ 的第一次增量是 $f_r(a_{r+1})-f_{l-1}(a_{r+1})=g(r+1)-f_{l-1}(a_{r+1})$。

   将 $f_{l-1}:[r+1,R]$ 离线，每次 $f_i\to f_{i+1}$ 的复杂度都是 $O(|A|)$，然后查询是 $O(n\sqrt n)$ 次，因此复杂度是 $O(n|A|+n\sqrt n)$。

+ 右指针向左删除旧元素

   对于区间 $[l,r]\to [l,r-1]\to[l,R]$ 的第一次增量是 $-[f_{r-1}({a_r})-f_{l-1}(a_r)]=-[g(r)-f_{l-1}(a_r)]$。

   将 $f_{l-1}:[R+1, r]$ 离线，同上。

+ 左指针向左加入新元素

   令 $g^{-1}(l-1)=f^{-1}_l(a_{l-1})$，可以 $O(n|A|)$ 的复杂度求 $g^{-1}(l)$。

   对于区间 $[l,r]\to [l-1,r]\to [L,r]$ 的第一次增量是 $f^{-1}_l(a_{l-1})-f_{r+1}^{-1}(a_{l-1})=g^{-1}(l-1)-f^{-1}_{r+1}(a_{l-1})$。

   将 $f^{-1}_{r+1}:[L,l-1]$ 离线，同上。
   
+ 左指针向右删除旧元素

   对于区间 $[l,r]\to[l+1,r]\to[L,r]$ 的第一次增量是 $-[f^{-1}_{l+1}(a_l)-f^{-1}_{r+1}(a_l)]=-[g^{-1}(l)-f^{-1}_{r+1}(a_l)]$。

   将 $f_{r+1}^{-1}:[l,L-1]$ 离线，同上。

具体地，分类型记录这些询问，每次询问都将值累加到对应的编号中。

趣事：我在写代码的时候把 $f,g$ 等数组大小也开了 $2^{14}$，调了 $3h$ 才发现，很气。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define pb push_back
const int N = 100010, M = 320;
int n, m, k;
int a[N], ans[N], f[N], g[N], g1[N];

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

struct Query {
    int id, l, r;

    bool operator<(const Query& b) const {
        if (l / M != b.l / M) return l < b.l;
        return ((l / M) & 1) ? r < b.r : r > b.r;
    }
} Q[N];

struct Query2 {
    int id, p, a, b, t;
};

bool incq(const Query2& a, const Query2& b) {
    return a.p < b.p;
}

bool decq(const Query2& a, const Query2& b) {
    return a.p > b.p;
}

vector<int> numsA;
vector<Query2> Q1, Q2;

signed main() {
    read(n), read(m), read(k);

    Q1.reserve(m), Q2.reserve(m);
    for (int i = 1; i <= n; i++) read(a[i]);
    
    for (int i = 0; i < (1<<14); i++) if (__builtin_popcount(i) == k) numsA.pb(i);
    for (int i = 1; i <= n; i++) {
        g[i] = g[i-1] + f[a[i]];
        for (int x: numsA) f[a[i] ^ x]++;
    }
    memset(f, 0, sizeof f);
    for (int i = n; i; i--) {
        g1[i] = g1[i+1] + f[a[i]];
        for (int x: numsA) f[a[i] ^ x]++;
    }
    for (int i = 1; i <= m; i++) read(Q[i].l), read(Q[i].r), Q[i].id = i;
    sort(Q+1, Q+1+m);

    int l = 1, r = 0;
    
    for (int i = 1; i <= m; i++) {
        int L = Q[i].l, R = Q[i].r, id = Q[i].id;
        if (r < R) ans[id] += g[R] - g[r], Q1.pb({id, l-1, r+1, R, -1});
        if (r > R) ans[id] -= g[r] - g[R], Q1.pb({id, l-1, R+1, r, 1});
        r = R;
        if (l > L) ans[id] += g1[L] - g1[l], Q2.pb({id, r+1, L, l-1, -1});
        if (l < L) ans[id] -= g1[l] - g1[L], Q2.pb({id, r+1, l, L-1, 1});
        l = L;
    }
    
    sort(Q1.begin(), Q1.end(), incq);
    sort(Q2.begin(), Q2.end(), decq);

    int p = 0;
    memset(f, 0, sizeof f);
    for (const auto& q: Q1) {
        while (p < q.p) {
            ++p;
            for (int x: numsA) f[a[p] ^ x]++;
        }
        for (int i = q.a; i <= q.b; i++) ans[q.id] += f[a[i]] * q.t;
    }

    memset(f, 0, sizeof f);
    p = n+1;
    for (const auto& q: Q2) {
        while (p > q.p) {
            --p;
            for (int x: numsA) f[a[p] ^ x]++;
        }
        for (int i = q.a; i <= q.b; i++) ans[q.id] += f[a[i]] * q.t;
    }
    
    for (int i = 2; i <= m; i++) ans[Q[i].id] += ans[Q[i-1].id];
    for (int i = 1; i <= m; i++) printf("%lld\n", ans[i]);
    
    return 0;
}
```

### 逆序对

区间逆序对数量，要求 $O(n\sqrt n)$，无法接受 $O(n\sqrt n\log n)$，链接：[P5047](https://www.luogu.com.cn/problem/P5047)。

当右指针向右拓展的时候，对答案的贡献应当是 $f(l,r,r+1)$，拆成前缀和就是 $f(r,r+1)-f(l-1,r+1)$，同样地，$f(r,r+1)$ 的预处理比较轻松，但是 $f(l-1,r+1)$ 就无法轻松做到这一点了。

所以，我们想要一个 $O(1)$ 查询，$O(\sqrt n)$ 更新的数据结构来维护，因为有 $O(n\sqrt n)$  次查询和 $O(n)$ 次更新，所以这个数据结构就是分块。

用 $f(l,x)$ 表示 $1\sim l$ 中大于 $x$ 的数量，用 $f^{-1}(r,x)$ 表示 $r\sim n$ 中小于 $x$ 的数量。

+ 右指针向右加入新元素 $f(l-1):[r+1,R]$
+ 右指针向左删除旧元素 $f(l-1):[R+1,r]$
+ 左指针向左加入新元素 $f^{-1}(r+1):[L,l-1]$
+ 左指针向右删除旧元素 $f^{-1}(r+1):[l,L-1]$

并且处理出 $g(r+1)=f(r,r+1),g^{-1}(l-1)=f^{-1}(l,l-1)$ 与上题相同。

初始化的时候用树状数组做到 $O(n\log n)$ 常数相较于全程用分块来说更小。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define lowbit(x) ((x)&(-x))
#define int long long
#define pb push_back
#define B(x) ((x-1) / M + 1)
const int N = 100010, M = 340;
int n, m, ans[N], a[N], g[N], g1[N];

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

struct Block {
    int val[N], tag[M];

    void init() {
        memset(val, 0, sizeof val);
        memset(tag, 0, sizeof tag);
    }

    void add(int l, int r, int x) {
        if (B(l) == B(r)) {
            for (int i = l; i <= r; i++) val[i] += x;
            return;
        }
        int L = l, R = r;
        while (B(L) == B(l)) val[L++] += x;
        while (B(R) == B(r)) val[R--] += x;
        for (int p = B(L); p <= B(R); p++) tag[p] += x;
    }

    int query(int p) {
        return val[p] + tag[B(p)];
    }
} blo;

struct BIT {
    int tr[N];

    void init() {
        memset(tr, 0, sizeof tr);
    }

    void add(int p, int x) {
        for (; p < N; p += lowbit(p)) tr[p] += x;
    }

    int query(int p) {
        int res = 0;
        for (; p; p -= lowbit(p)) res += tr[p];
        return res;
    }

    void add(int l, int r, int x) {
        add(l, x);
        add(r+1, -x);
    }
} bit;

struct Query {
    int id, l, r;
    bool operator<(const Query& q) const {
        if (l / M != q.l / M) return l < q.l;
        return ((l / M) & 1) ? r < q.r : r > q.r;
    }
} Q[N];

struct Query2 {
    int id, p, a, b, t;
};

vector<Query2> Q1, Q2;

bool incq(const Query2& a, const Query2& b) {
    return a.p < b.p;
}

bool decq(const Query2& a, const Query2& b) {
    return a.p > b.p;
}

vector<int> nums;
signed main() {
    read(n), read(m);
    nums.reserve(n);
    for (int i = 1; i <= n; i++) read(a[i]), nums.pb(a[i]);
    sort(nums.begin(), nums.end());
    for (int i = 1; i <= n; i++) a[i] = lower_bound(nums.begin(), nums.end(), a[i]) - nums.begin() + 1;
    for (int i = 1; i <= m; i++) Q[i].id = i, read(Q[i].l), read(Q[i].r);
    sort(Q+1, Q+1+m);

    // bit.init();
    for (int i = 1; i <= n; i++) {
        g[i] = g[i-1] + bit.query(a[i]);
        if (a[i] > 1) bit.add(1, a[i]-1, 1);
    }

    bit.init();
    for (int i = n; i; i--) {
        g1[i] = g1[i+1] + bit.query(a[i]);
        bit.add(a[i]+1, 1);
    }

    int l = 1, r = 0;
    
    for (int i = 1; i <= m; i++) {
        int L = Q[i].l, R = Q[i].r, id = Q[i].id;
        if (r < R) ans[id] += g[R] - g[r], Q1.pb({id, l-1, r+1, R, -1});
        if (r > R) ans[id] -= g[r] - g[R], Q1.pb({id, l-1, R+1, r, 1});
        r = R;
        if (l > L) ans[id] += g1[L] - g1[l], Q2.pb({id, r+1, L, l-1, -1});
        if (l < L) ans[id] -= g1[l] - g1[L], Q2.pb({id, r+1, l, L-1, 1});
        l = L;
    }

    sort(Q1.begin(), Q1.end(), incq);
    sort(Q2.begin(), Q2.end(), decq);

    int p = 0;
    // blo.init();
    for (const auto& q: Q1) {
        while (p < q.p) blo.add(0, a[++p]-1, 1);
        for (int i = q.a; i <= q.b; i++) ans[q.id] += blo.query(a[i]) * q.t;
    }

    p = n+1;
    blo.init();
    for (const auto& q: Q2) {
        while (p > q.p) blo.add(a[--p]+1, n+1, 1);
        for (int i = q.a; i <= q.b; i++) ans[q.id] += blo.query(a[i]) * q.t;    
    }

    for (int i = 2; i <= m; i++) ans[Q[i].id] += ans[Q[i-1].id];
    for (int i = 1; i <= m; i++) printf("%lld\n", ans[i]);

    return 0;
}
```

