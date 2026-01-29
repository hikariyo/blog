---
date: 2023-08-26 14:51:00
updated: 2023-12-29 02:13:00
title: 数据结构 - 树状数组
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
---

## 简介

树状数组可以看做简化版的线段树，它进行单点修改和区间查询的常数是比线段树更优的。

对于原数组 $a[n]$，树状数组 $c[n]$ 是一个等长的数组，并且对于任意 $c[i]$ 表示以 $a[i]$ 结尾且长度为 $\text{lowbit}(i)$ 的区间和。

下面用 $f(x)$ 代表 $\text{lowbit}(x)$，$L(x),R(x)$ 代表 $x$ 覆盖的左闭右开区间，证明其正确性。

若 $A_0=k2^{i+1}+2^i$，则 $f(A_0)=2^i,L(A_0)=A_0-f(A_0)=k2^{i+1},R(A_0)=A_0$。

则 $A_1=A_0+f(A_0)=(k+1)2^{i+1}$，那么 $L(A_1)=A_1-f(A_1)=(k+1)2^{i+1}-f(A_1)$。

由于 $f(A_1)\ge 2^{i+1}$，所以 $L(A_1)\le k2^{i+1}=L(A_0)$，且 $R(A_1)=A_1=(k+1)2^{i+1}\gt A_0=R(A_0)$，所以 $A_1$ 一定覆盖 $A_0$。

所以一个点的父节点是 $x+f(x)$，它左边与它相邻且不是它子结点的点是 $L(x)=x-f(x)$。

## 模板题

> 已知一个数列，你需要进行下面两种操作：
>
> - 将某一个数加上 $x$
>
> - 求出某区间每一个数的和
>
> 题目链接：[P3374](https://www.luogu.com.cn/problem/P3374)。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 500010;
int tr[N], n, m;

#define lowbit(x) ((x)&(-x))

void add(int p, int v) {
    for (; p < N; p += lowbit(p))
        tr[p] += v;
}

int query(int p) {
    int res = 0;
    for (; p; p -= lowbit(p))
        res += tr[p];
    return res;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        int v; scanf("%d", &v);
        add(i, v);
    }
    
    while (m--) {
        int op, x, y;
        scanf("%d%d%d", &op, &x, &y);
        if (op == 1) add(x, y);
        else printf("%d\n", query(y) - query(x-1));
    }
    return 0;
}
```

## 逆序对

> 定义：$i<j$ 且 $a_i>a_j$ 就称为一个逆序对，统计逆序对数目。
>
> 题目链接：[P1908](https://www.luogu.com.cn/problem/P1908)。

本题可以用归并排序那样的分治算法，并且它更好，但是这里我们用树状数组来解决这个问题。

首先注意到值域比较大，所以需要离散化。当枚举到 $a_i$ 时，我们需要知道前面有多少个数大于 $a_i$，如果我们用树状数组来统计每个数字出现的次数，也就是求一下 $[a_i+1,n]$ 的区间和，其中 $n$ 是离散化后的最大值。

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 500010;
int n, a[N];
LL tr[N];
vector<int> nums;

#define lowbit(x) ((x)&(-x))

void add(int p, int v) {
    for (; p < N; p += lowbit(p))
        tr[p] += v;
}

LL query(int p) {
    LL res = 0;
    for (; p; p -= lowbit(p))
        res += tr[p];
    return res;
}

LL query(int l, int r) {
    return query(r) - query(l-1);
}

int find(int x) {
    return lower_bound(nums.begin(), nums.end(), x) - nums.begin() + 1;
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &a[i]);
        nums.push_back(a[i]);
    }
    sort(nums.begin(), nums.end());
    nums.erase(unique(nums.begin(), nums.end()), nums.end());
    LL res = 0;
    for (int i = 1; i <= n; i++) {
        int t = find(a[i]);
        res += query(t+1, nums.size());
        add(t, 1);
    }
    printf("%lld\n", res);
    return 0;
}
```

## The Battle of Chibi (LIS)

> 简单题意：给 $T$ 组数据，长度为 $n$ 的数列 $a$ 中，找出长度为 $m$ 的严格上升子序列的个数，答案对 `1e9+7` 取模。
>
> 题目链接：[UVA12983](https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=862&page=show_problem&problem=4866)，[UVA12983(Luogu)](https://www.luogu.com.cn/problem/UVA12983)。

这是一道 DP 题，但是可以用树状数组来加速。

+ 状态表示 $f(i,j)$：长度为 $i$ 以 $a_j$ 结尾的最长上升子序列的个数。

+ 状态转移：
    $$
    f(i,j)=\sum_{a_k<a_j, k<j} f(i-1,k)
    $$
    只要我们在循环到 $j$ 时把之前的所有 $f(i-1,k)$ 中 $a_k<a_j$ 的值累加起来即可，这可以用树状数组优化，树状数组的索引是 $a_i$ 的值，值是每个 $f(i-1,k)$ 每次求的都是 $[0,a_i-1]$ 间所有满足要求的值之和。

    由于牵扯到值域的问题，这里就需要用到离散化。

+ 初始化：$f(1,j)=1$

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <vector>
using namespace std;

const int N = 1010, mod = 1e9+7;
vector<int> nums;
int tr[N], n, m;
int a[N], f[N][N];
#define lowbit(x) ((x)&(-x))

inline void add(int p, int v) {
    for (; p < N; p += lowbit(p)) {
        tr[p] = (tr[p] + v) % mod;
    }
}

inline int query(int p) {
    int res = 0;
    for (; p; p -= lowbit(p))
        res = (res + tr[p]) % mod;
    return res;
}

int find(int x) {
    return lower_bound(nums.begin(), nums.end(), x) - nums.begin() + 1;
}

int solve() {
    // clear f
    for (int j = 1; j <= n; j++) f[1][j] = 1;
    for (int i = 2; i <= m; i++) {
        memset(tr, 0, sizeof tr);
        for (int j = 1; j <= n; j++) {
            f[i][j] = query(a[j]-1);
            add(a[j], f[i-1][j]);
        }
    }
    int res = 0;
    for (int j = 1; j <= n; j++) res = (res + f[m][j]) % mod;
    return res;
}

int main() {
    int T; scanf("%d", &T);
    for (int C = 1; C <= T; C++) {
        nums.clear();
        scanf("%d%d", &n, &m);
        for (int i = 1; i <= n; i++) {
            scanf("%d", &a[i]);
            nums.push_back(a[i]);
        }
        sort(nums.begin(), nums.end());
        nums.erase(unique(nums.begin(), nums.end()), nums.end());
        for (int i = 1; i <= n; i++) {
            a[i] = find(a[i]);
        }
        printf("Case #%d: %d\n", C, solve());
    }
    return 0;
}
```

## [THUPC2024 初赛] 二进制

> 小 F 给出了一个这里有一个长为 $n\le 10^6$ 的二进制串 $s$，下标从 $1$ 到 $n$，且 $\forall i\in[1,n],s_i\in \{0,1\}$，他想要删除若干二进制子串。
>
> 具体的，小 F 做出了 $n$ 次尝试。
>
> 在第 $i\in[1,n]$ 次尝试中，他会先写出正整数 $i$ 的二进制串表示 $t$（无前导零，左侧为高位，例如 $10$ 可以写为 $1010$）。
>
> 随后找到这个二进制表示 $t$ 在 $s$ 中从左到右 **第一次** 出现的位置，并删除这个串。
>
> 注意，删除后左右部分的串会拼接在一起 **形成一个新的串**，请注意新串下标的改变。
>
> 若当前 $t$ 不在 $s$ 中存在，则小 F 对串 $s$ 不作出改变。
>
> 你需要回答每一次尝试中，$t$ 在 $s$ 中第一次出现的位置的左端点以及 $t$ 在 $s$ 中出现了多少次。
>
> 定义两次出现不同当且仅当出现的位置的左端点不同。
>
> 请注意输入输出效率。
>
> 题目链接：[LOJ 6906](https://loj.ac/p/6906)。

首先，对于一个数字 $n$，它的二进制位数是 $\lfloor \log_2 n\rfloor + 1$，为了方便阅读，下文用 $\log n$ 代替。

我们每次删除的都是 $1\sim n$ 的二进制串，因此它的长度应该小于等于 $\log n$，所以我们对于原串 $s$ 的每位 $i$ 都向后 $\log n$ 个字符都扫描一遍，假设获得到的数字是 $v$，那么就添加一个 $v\to i$ 的映射关系。

删除的时候就在这个位置的前 $\log n$，后 $\log v$ 都更新一下。这样更新的原因是后 $\log v$ 个元素是被删除掉的，所以需要删掉对应的映射关系；前 $\log n$ 个向后关系有所改变，所以需要重新更新。

考虑如何处理坐标的变化，肯定是不能在映射里面修改的，这样复杂度太大了，所以用记录偏移量的方式，每次删除相当于在一个位置的后面的真实坐标都要减去 $\log v$，所以这里开一个树状数组就可以了。

然后考虑这个映射关系如何维护，由于 $v$ 于 $n$ 数量级相同，所以可以直接开数组，然后这个数组内需要一个支持动态 `add,delete,min,size` 的数据结构，所以用可删堆来实现，相较于 `std::set` 常数要小一些。

删除之后再次访问的时候需要跳过删除掉的地方，这个操作可以用链表实现。由于更新的时候需要向前走，所以是双链表。

复杂度分析：删除的时候每次更新 $O(\log n)$ 个元素，每次更新的复杂度是 $O(\log^2 n)$，所以删除的复杂度是 $O(\log^3 n)$。

但是一共只会执行 $O(\frac{n}{\log n})$ 次删除，所以最终的复杂度是 $O(n\log^2 n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define lowbit(x) ((x)&(-x))
const int N = 1200010;

struct RemovableHeap {
    priority_queue<int, vector<int>, greater<int>> v, d;
    void remove(int x) {
        d.push(x);
    }

    int top() {
        while (d.size() && v.top() == d.top()) v.pop(), d.pop();
        return v.top();
    }

    void push(int x) {
        v.push(x);
    }

    int size() {
        return v.size() - d.size();
    }
} heap[N];

int n, a[N], lg2[N], nxt[N], pre[N], lg2n;

struct BIT {
    int tr[N];

    int query(int p) {
        int res = 0;
        for (; p; p -= lowbit(p)) res += tr[p];
        return res;
    }

    void add(int p, int k) {
        for (; p < N; p += lowbit(p)) tr[p] += k;
    }
} bit;

void update(int s, bool add = true) {
    if (a[s] == 0) return;

    for (int i = s, now = 0, cnt = 0; i && cnt < lg2n; i = nxt[i], cnt++) {
        now = now << 1 | a[i];
        if (add) heap[now].push(s);
        else heap[now].remove(s);
    }
}

void remove(int s, int bits) {
    int ne = s, raws = s;
    bit.add(s, -bits);
    for (int i = 0; i < bits; i++) {
        update(ne, false);
        ne = nxt[ne];
    }

    pre[ne] = pre[s];
    for (int i = 0; s && i < lg2n; i++) {
        s = pre[s];
        update(s, false);
    }
    s = raws;
    if (pre[s]) nxt[pre[s]] = ne;
    for (int i = 0; s && i < lg2n; i++) {
        s = pre[s];
        update(s);
    }
}

int main() {
    static char s[N];
    scanf("%d%s", &n, s+1);
    for (int i = 1; i <= n; i++) a[i] = s[i] ^ 48;
    // lg2[i] 表示 floor(log2(i)) + 1
    lg2[1] = 0;
    for (int i = 2; i <= n; i++) lg2[i] = lg2[i>>1] + 1;

    for (int i = 1; i <= n; i++) lg2[i]++, nxt[i] = i+1, pre[i] = i-1;
    nxt[n] = 0, lg2n = lg2[n];

    for (int i = 1; i <= n; i++) update(i);
    for (int i = 1; i <= n; i++) {
        if (heap[i].size() == 0) puts("-1 0");
        else {
            int p = heap[i].top(), sz = heap[i].size();
            printf("%d %d\n", p + bit.query(p), sz);
            remove(p, lg2[i]);
        }
    }

    return 0;
}
```

## [CF1915F] Greetings

> 多测，给定 $n$ 个区间 $[l_i,r_i]$ 求出每个区间包含的区间个数之和，$\sum n\le 2\times 10^5,-10^9\le l_i\le r_i\le10^9$。
>
> 题目链接：[CF1915F](https://codeforces.com/problemset/problem/1915/F)。

首先排序用时间维度代替 $r_i$ 维度，然后在所有 $r_j\le r_i$ 的区间中查找 $l_j\ge l_i$ 的数量，先考虑如何将当前区间贡献出去，可以在 $(-\infin,l_i]$ 加一，这样查询 $l$ 的时候就会在当 $l\le l_i$ 的时候加上当前区间的贡献，这样是正确的。

将坐标离散化成 $[1,2n]$ 中的数字。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define lowbit(x) ((x)&(-x))
#define int long long
#define eb emplace_back
const int N = 200010;
int n;

struct BIT {
    int tr[N*2];

    void init(int n) {
        for (int i = 1; i <= n; i++) tr[i] = 0;
    }

    void add(int p, int k) {
        for (; p < N*2; p += lowbit(p)) tr[p] += k;
    }

    int query(int p) {
        int res = 0;
        for (; p; p -= lowbit(p)) res += tr[p];
        return res;
    }
} bit;

struct Object {
    int l, r;
    bool operator<(const Object& obj) const {
        if (r == obj.r) return l < obj.l;
        return r < obj.r;
    }
} obj[N];

int solve() {
    cin >> n;
    vector<int> nums;
    nums.reserve(n*2);
    for (int i = 1; i <= n; i++) cin >> obj[i].l >> obj[i].r, nums.eb(obj[i].l), nums.eb(obj[i].r);
    sort(nums.begin(), nums.end());
    sort(obj+1, obj+1+n);

    int ans = 0;
    bit.init(n*2);
    for (int i = 1; i <= n; i++) {
        int l = lower_bound(nums.begin(), nums.end(), obj[i].l) - nums.begin() + 1;
        ans += bit.query(l);
        bit.add(l+1, -1);
        bit.add(1, 1);
    }

    return ans;
}

signed main() {
    int t;
    cin >> t;
    while (t--) cout << solve() << '\n';
    return 0;
}
```

