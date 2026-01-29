---
date: 2023-11-16 18:21:00
updated: 2023-11-16 18:21:00
title: 数学 - 康托展开
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
description: 康托展开是对排列的完美哈希，其值为一个排列在字典序中的排行，通过树状数组进行优化后可以在 O(n log n) 的复杂度对一个排列康托展开。
---

## 康托展开

> 给定一个排列，求其按照字典序的排名，对 $998244353$ 取模，$N\le 1000000$。
>
> 题目链接：[P5367](https://www.luogu.com.cn/problem/P5367)。

对于一个排列 $p$，当枚举到 $p_i$ 时，显然只要满足 $\forall j<i,x\ne p_j$ 并且 $x\lt p_i$ 就可以作为它前驱序列的开头，然后后面做一个全排列，即 $\text{cnt}(x)\times (n-i)!$ 这里 $\text{cnt}(x)$ 可以用树状数组求。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define lowbit(x) ((x)&(-x))
const int N = 1000010, P = 998244353;
int n, p[N], tr[N], fact[N];

void add(int p, int k) {
    for (; p < N; p += lowbit(p))
        tr[p] += k;
}

int query(int p) {
    int res = 0;
    for (; p; p -= lowbit(p))
        res += tr[p];
    return res;
}

signed main() {
    cin >> n;
    fact[0] = 1;
    for (int i = 1; i <= n; i++) {
        cin >> p[i];
        add(i, 1);
        fact[i] = fact[i-1] * i % P;
    }

    int res = 0;
    for (int i = 1; i <= n; i++) {
        res = (res + query(p[i]-1) * fact[n-i]) % P;
        add(p[i], -1);
    }
    cout << res+1 << '\n';
    return 0;
}
```

## 交换次数

> 给定排列，求将第一个排列按照字典序的顺序转化到这个排列需要经过多少次，并且求出每次经过的步数（相邻两项交换一次算一步）。

### 打表

赛时遇到找规律，直接打表。赛后我竟然半天没写出来打表的程序，然后原始文件还被我覆盖掉了，太悲伤了。

首先我们将一个排列变成另一个排列，需要的最少步数，一定是先确定好第一项，然后确定第二项，依次往下，这样就能保证是最小的。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int n = 5;

int count(vector<int>& src, const vector<int>& target) {
    auto change = [&](int x) -> int {
        int s = 0, t = 0;
        for (int i = 0; i < n; i++) {
            if (src[i] == x) s = i;
            if (target[i] == x) t = i;
        }
        
        int cnt = 0;
        if (s > t) for (int i = s; i > t; i--) swap(src[i], src[i-1]), cnt++;
        else for (int i = s; i < t; i++) swap(src[i], src[i+1]), cnt++;
        return cnt;
    };

    int res = 0;
    for (int i = 0; i < n; i++) {
        res += change(target[i]);
    }
    return res;
}

int main() {
    freopen("table.out", "w", stdout);
    vector<int> p(n), src;
    for (int i = 0; i < n; i++) p[i] = i+1;
    bool first = true;
    do {
        if (!first) cout << count(src, p) << '\n';
        src = p;
        first = false;
    } while (next_permutation(p.begin(), p.end()));
    return 0;
}
```

通过更改 $n$ 的值造成的影响可以列出来，这里循环节指的是包含最大数字的循环节：

| $n$  | 最大数字 | 循环节长度 | 总长   |
| ---- | -------- | ---------- | ------ |
| $2$  | $1$      | $1$        | $1$    |
| $3$  | $2$      | $2$        | $5$    |
| $4$  | $4$      | $6$        | $23$   |
| $5$  | $7$      | $24$       | $119$  |
| $6$  | $11$     | $120$      | $719$  |
| $7$  | $16$     | $720$      | $5039$ |

可以人为的把总长加一，然后明显观察到这是阶乘的关系，然后最大数字的差分是 $1,2,3,4,5$ 递增的关系，因此我们进行求和，可以认为对于长度为 $n$ 的排列，一开始总长为 $n!$ 的数列中全是 $1$：

+ 隔 $2!$ 数字加 $1$
+ 隔 $3!$ 数字加 $2$
+ 隔 $4!$ 数字加 $3$
+ 隔 $5!$ 数字加 $4$

因此：
$$
\begin{aligned}
f(n)&=n!+\frac{n!}{2!}\times 1+\frac{n!}{3!}\times 2+\cdots\\
&=n!+n!\sum_{i=2}^{n} \frac{i-1}{i!}
\end{aligned}
$$
我们最后把最大的那个数减掉，是 $1+\frac{n(n-1)}{2}$。

现在我们可以计算出长度为 $n$ 的排列从 $1\sim n$ 到 $n\sim 1$ 需要的操作次数了，考虑按照康托展开那样计算。

考虑到我们上面是按照 $n$ 个循环节来计算的，所以我们这里有长度为 $n-i+1$ 的 $\text{cnt}(x)$ 个循环节，猜想直接累加 $\text{cnt}(x)\times f(n-i+1)/(n-i+1)$，发现答案正确。

### 推公式

打表的方法看上去比较扯淡，但是考场上做出来大于一切，下面开始正经推导。

我们将排列 $1\sim n$ 变成 $n\sim 1$，计作 $f(n)$，那么：

1. 把 $1\sim n$ 变成 $1,n,\dots,2$ 需要 $f(n-1)$
2. 把 $2$ 移到头上变成 $2,1,n,\dots,3$ 需要 $n-1$
3. 由于 $2$ 开头的字典序最小的是 $2,1,3,\dots,n$ 翻转 $n-2$ 个数字需要 $\sum_{i=1}^{n-3} i=(n-2)(n-3)/2=\binom{n-2}2$。

所以换一个头需要的操作步数是 $n-1+(n-2)(n-3)/2$，我们需要 $n$ 次翻转和 $n-1$ 次换头，所以公式就是：
$$
f(n)=nf(n-1)+(n-1)(n-1+\binom{n-2}2)
$$
显然 $f(1)=0$，递推即可。但是类似数位 DP 的感觉去思考，需要最后再来一次换头的操作，但是这里我们调用的是 $f(n-i)$ 但是换头加的是 $n-i+\binom{n-i-1}2$，因为是把后面整个 $n-i$ 都换个头。

用组合数的好处是纯粹是好写，不是必须用组合数才能表示。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define lowbit(x) ((x)&(-x))
const int N = 1000010, P = 1e9+7;
int n, p[N], tr[N], fact[N], f[N];

void add(int p, int k) {
    for (; p < N; p += lowbit(p))
        tr[p] += k;
}

int query(int p) {
    int res = 0;
    for (; p; p -= lowbit(p))
        res += tr[p];
    return res;
}

// C(x, 2)
int C2(int x) {
    if (x <= 1) return 0;
    return x*(x-1)/2 % P;
}

signed main() {
    freopen("perm.in", "r", stdin);
    freopen("perm.out", "w", stdout);
    cin >> n;
    fact[0] = 1;
    for (int i = 1; i <= n; i++) {
        cin >> p[i];
        add(i, 1);
        fact[i] = fact[i-1] * i % P;
    }
    f[1] = 0;
    for (int i = 2; i <= n; i++) f[i] = (i*f[i-1] + (i-1) * (i-1 + C2(i-2))) % P;

    int id = 0, step = 0;
    for (int i = 1; i <= n; i++) {
        id = (id + query(p[i]-1) * fact[n-i]) % P;
        step = (step + query(p[i]-1) * (f[n-i] + n-i + C2(n-i-1))) % P;
        add(p[i], -1);
    }
    cout << id << ' ' << step << '\n';
    return 0;
}
```

