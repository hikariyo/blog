---
date: 2025-10-28 18:30:00
title: "[JLCPC 2025 H] Another Palindromes Problem"
katex: true
tags:
- Algo
- C++
- DS
- Hash
categories:
- XCPC
description: 线段树分别维护奇偶序列的哈希。
---

## 题目

[Codeforces Gym105922 H](https://codeforces.com/gym/105922/problem/H)

## 思路

首先转化为询问 $a_l\sim a_r$ 中，奇数位置元素构成的多重集，与偶数位置构成的多重集是否相同。

那么对于两个序列的比较，想到哈希是不难的，问题是这道题目还有区间加的操作，我们应该如何哈希。

假设我们已经有了一个针对集合的哈希函数 $F(S)$，那么对于一个多重集 $A=\{a_1,a_2,\dots,a_k\}$，下面会是一个容易想到的设计：
$$
F(A)=\sum_{i=1}^k f(a_i)
$$
也许读者会思考，为什么这里不和字符串哈希那样，把字符串当成一个 $P$ 进制数？这是因为我们设计的是集合的哈希，而集合是无序的，而利用加法的交换率设计哈希函数正好可以满足集合的无序这个特点。

那么接下来考虑更新，即思考如何完成区间加 $v$ 这一操作。
$$
F(A)\gets \sum_{i=1}^k f(a_i+v)
$$
我在 VP 时会想把 $f(x)$ 设计为一个多项式函数，即 $f(x)\equiv ax^3+bx^2+cx+d\pmod p$，其中 $p$ 是一个自己选定的质数。这确实可行，下面我来讨论一下为什么这样不是很好。

首先我们希望减少哈希碰撞，也就是两个不同的集合的哈希值一样的情况。

如果 $\sum x^2$ 与 $\sum y^2$、$\sum x^3$ 与 $\sum y^3$ 不同，$a\sum x^3+b\sum x^2$ 是有可能等于 $a\sum y^3+b\sum y^2$ 的，所以这个设计不如分别比较平方和与立方和。

而平方和与立方和给我们的操作空间就很小了，因为你选定的参数 $a,b,c,d$ 实际上是无效的，那么我们唯一有效的就是那个质数 $p$ 了。

然而，如果将 $f(x)$ 设计为指数函数，即 $f(x)\equiv a^x\pmod p$，这里的 $a,p$ 参数是我们随便选择的，碰撞的概率会更小，并且区间加也只是区间的哈希值乘 $a^v$，维护也更加便捷。

## 实现

我这里给出我的赛时代码，是比较 $\sum x^3$ 的，仅供参考。

这个题对于 $l,r$ 奇偶的讨论也很烦人，并且由于更新时并不保证 $l,r$ 长度为偶数，所以很有可能存在这个更新只更新了一个数字，而这就导致我们更新的时候需要特判一下是不是会 $l>r$，否则会 `Runtime error on test 12`。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 2e5+10;
const int P = 1e9+9;

#define ls (u<<1)
#define rs (u<<1|1)
#define Mid ((L+R) >> 1)

struct SGT {
    int s1[N<<2], s2[N<<2], s3[N<<2], tag[N<<2];
    int a[N];

    void spread(int u, int t, int L, int R) {
        int S1 = s1[u], S2 = s2[u], S3 = s3[u];
        s1[u] = (S1 + (R-L+1) * t) % P;
        s2[u] = (S2 + t*t % P * (R-L+1) % P + 2*t*S1 % P) % P;
        s3[u] = (S3 + t*t % P * t % P * (R-L+1) % P + 3*t*S2 % P + 3*t*t % P * S1 % P) % P;
        tag[u] = (tag[u] + t) % P;
    }

    void pushdown(int u, int L, int R) {
        if (tag[u] == 0) return;
        spread(ls, tag[u], L, Mid);
        spread(rs, tag[u], Mid+1, R);
        tag[u] = 0;
    }

    void pushup(int u) {
        s1[u] = (s1[ls] + s1[rs]) % P;
        s2[u] = (s2[ls] + s2[rs]) % P;
        s3[u] = (s3[ls] + s3[rs]) % P;
    }

    void modify(int u, int l, int r, int t, int L, int R) {
        if (l > r) return;
        if (l <= L && R <= r) return spread(u, t, L, R);
        pushdown(u, L, R);
        if (l <= Mid) modify(ls, l, r, t, L, Mid);
        if (Mid+1 <= r) modify(rs, l, r, t, Mid+1, R);
        pushup(u);
    }

    int query1(int u, int l, int r, int L, int R) {
        if (l > r) return 0;
        if (l <= L && R <= r) return s1[u];
        pushdown(u, L, R);
        int res = 0;
        if (l <= Mid) res = query1(ls, l, r, L, Mid);
        if (Mid+1 <= r) res = (res + query1(rs, l, r, Mid+1, R)) % P;
        return res;
    }

    int query2(int u, int l, int r, int L, int R) {
        if (l > r) return 0;
        if (l <= L && R <= r) return s2[u];
        pushdown(u, L, R);
        int res = 0;
        if (l <= Mid) res = query2(ls, l, r, L, Mid);
        if (Mid+1 <= r) res = (res + query2(rs, l, r, Mid+1, R)) % P;
        return res;
    }

    int query3(int u, int l, int r, int L, int R) {
        if (l > r) return 0;
        if (l <= L && R <= r) return s3[u];
        pushdown(u, L, R);
        int res = 0;
        if (l <= Mid) res = query3(ls, l, r, L, Mid);
        if (Mid+1 <= r) res = (res + query3(rs, l, r, Mid+1, R)) % P;
        return res;
    }

    void build(int u, int L, int R) {
        if (L == R) return s1[u] = a[L], s2[u] = a[L] * a[L], s3[u] = a[L] * a[L] % P * a[L] % P, void();
        build(ls, L, Mid);
        build(rs, Mid+1, R);
        pushup(u);
    }
} t[2];

int n, q;
signed main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    cin >> n >> q;
    for (int i = 1, a; i <= n; i++) {
        cin >> a;
        t[i % 2].a[(i + 1) / 2] = a;
    }
    if (n % 2) n++;
    t[0].build(1, 1, n/2);
    t[1].build(1, 1, n/2);

    for (int i = 1; i <= q; i++) {
        int op, l, r, v;
        cin >> op >> l >> r;

        if (op == 0) {
            cin >> v;
            if (l % 2 && r % 2) {
                t[0].modify(1, (l+1)/2, (r-1)/2, v, 1, n/2);
                t[1].modify(1, (l+1)/2, (r+1)/2, v, 1, n/2);
            }
            else if (!(l % 2) && r % 2) {
                t[0].modify(1, l/2, (r-1)/2, v, 1, n/2);
                t[1].modify(1, (l+2) / 2, (r+1)/2, v, 1, n/2);
            }
            else if (l % 2 && !(r % 2)) {
                t[0].modify(1, (l+1)/2, r/2, v, 1, n/2);
                t[1].modify(1, (l+1)/2, r/2, v, 1, n/2);
            }
            else {
                t[0].modify(1, l/2, r/2, v, 1, n/2);
                t[1].modify(1, (l+2)/2, r/2, v, 1, n/2);
            }
        }
        else {
            if (l % 2) {
                // l 奇， r 偶
                if (
                    t[0].query2(1, (l+1)/2, r/2, 1, n/2) ==
                    t[1].query2(1, (l+1)/2, r/2, 1, n/2) &&
                    t[0].query3(1, (l+1)/2, r/2, 1, n/2) ==
                    t[1].query3(1, (l+1)/2, r/2, 1, n/2)
                ) {
                    cout << "YES\n";
                }
                else cout << "NO\n";
            }
            else {
                if (
                    t[0].query2(1, l/2, (r-1)/2, 1, n/2) ==
                    t[1].query2(1, (l+2)/2, (r+1)/2, 1, n/2) &&
                    t[0].query3(1, l/2, (r-1)/2, 1, n/2) ==
                    t[1].query3(1, (l+2)/2, (r+1)/2, 1, n/2)
                ) {
                    cout << "YES\n";
                }
                else {
                    cout << "NO\n";
                }
            }
        }
    }
    return 0;
}
```



