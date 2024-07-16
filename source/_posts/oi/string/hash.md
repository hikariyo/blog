---
date: 2023-06-15 15:10:05
update: 2024-06-14 22:55:00
title: String - Hash
katex: true
tags:
- Algo
- C++
- String
categories:
- OI
---

## 简介

Hash 的思想就是设计函数 $f(A)$ 将某个数据类型映射为整数，由于相同的元素映射后得到的值仍然相同，所以当 $f(x)=f(y)$ 时认为 $x=y$ 即 Hash 的思想。

为了防止哈希碰撞即 $f(x)=f(y)$ 时 $x\ne y$，可以多设计几个函数 $f_1,f_2,\dots$​ 尽可能避免哈希碰撞的发生。可以看出这是一个用概率来换复杂度的算法。

注意，任何数据结构都可以用哈希来判断是否相等，只不过字符串哈希用的比较多一点，所以本文归类为字符串。

## [模板] 字符串哈希

> 给定长度为 $n$ 的字符串 $s$，再给 $m$ 个询问，询问 $s_{l_1\sim r_1}$ 与 $s_{l_2\sim r_2}$ 是否相同。其中 $1\le n,m\le 10^5$。
>
> 题目链接：[AcWing 841](https://www.acwing.com/problem/content/843/)。

一种常见的字符串哈希的方法是将字符串看作一个 $K=131$ 进制的数字，然后对某个大质数 $P$​​ 取模，因为其因子少可以减少碰撞概率。

以数组 $h_i$ 代表字符串 $1\sim i$  换做 $K$ 进制的数字 $\bmod P$ 的结果，即 $h_i=\sum_{j=1}^i s_j\times K^{i-j}$，有递推式 $h_i=h_{i-1}\times K+s_i$ 且 $h_0=0$。

对于 $i\sim j$ 中的值，计算 $h_j -h_i\times K^{j-i+1}$ 就可以得到结果了。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 100010, K = 131;
int n, m;
string s;

template<int P>
struct StringHash {
    int h[N], pow[N];

    int qmul(int a, int k) {
        int res = 0;
        while (k) {
            if (k & 1) res = (res + a) % P;
            a = (a + a) % P;
            k >>= 1;
        }
        return res;
    }

    void init(const string& s) {
        pow[0] = 1;
        for (int i = 1; i <= s.size(); i++) {
            pow[i] = qmul(pow[i-1], K);
            h[i] = (qmul(h[i-1], K) + s[i-1]) % P;
        }
    }

    int get(int l, int r) {
        int res = (h[r] - qmul(h[l-1], pow[r-l+1])) % P;
        if (res < 0) res += P;
        return res;
    }
};

StringHash<1'000'000'000'000'000'003ll> h1;
StringHash<1'000'000'000'000'000'009ll> h2;

signed main() {
    cin >> n >> m >> s;
    h1.init(s), h2.init(s);
    for (int i = 1, l1, r1, l2, r2; i <= m; i++) {
        cin >> l1 >> r1 >> l2 >> r2;
        if (h1.get(l1, r1) == h1.get(l2, r2) && h2.get(l1, r1) == h2.get(l2, r2)) cout << "Yes\n";
        else cout << "No\n";
    }
    return 0;
}
```

