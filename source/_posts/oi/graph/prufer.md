---
date: 2023-09-05 16:00:00
updated: 2023-09-05 16:00:00
title: 图论 - Prufer 编码
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 简介

Prufer 编码用长度为 `n-2` 的序列表示一个无根树，构造一个 Prufer 序列的方法是：

1. 找到编号最小的叶节点，将它的父结点加入序列。
2. 删除这个节点，如果这个操作使得父结点成为叶结点，并且父结点的编号更小，那么递归处理（如果更大的话会在之后枚举到）。

也就是说，某个点在 Prufer 序列中出现的次数等于它的出度，如果我们设每条边都是父结点指向子结点。

下面证明如果有第 `n-1` 项，它一定是 `n`：

1. 如果 `n` 的父结点没有别的子结点，那么此时就只有两个点，应当是 `n` 被加入序列。
2. 如果有别的子结点，无论这个子结点是否是叶子结点，这另外一个子结点往下找一定存在编号比 `n` 小的叶结点。

将 Prufer 序列转化为无根树的方法类似于转过去的过程：

1. 找到编号最小的叶子结点，它的父结点就是 `p.head()`
2. 删掉这个结点，如果这个操作使得其父结点成为叶结点，并且父结点的编号更小，那么递归处理。

由此可以看出一个无根树和 Prufer 序列是一一对应的，所以有 $n$ 个结点的无根树的个数是 $n^{n-2}$。

## 模板

> 请实现 Prüfer 序列和无根树的相互转化。为方便你实现代码，尽管是无根树，我们在读入时仍将 $n$ 设为其根。
>
> 对于一棵无根树，设 $f_{1\dots n-1}$ 为其**父亲序列**（$f_i$ 表示 $i$ 在 $n$ 为根时的父亲），设 $p_{1 \dots n-2}$ 为其 **Prüfer 序列**。
>
> 另外，对于一个长度为 $m$ 的序列 $a_{1 \dots m}$，我们设其**权值**为 $\operatorname{xor}_{i = 1}^m i \times a_i$。
>
> 题目链接：[P6086](https://www.luogu.com.cn/problem/P6086)。

用上面提到的方法可以 $O(n)$ 的做，如果用堆的话就是 $O(n\log n)$ 了。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 5000010;
int f[N], p[N], n, m;
LL d[N];

LL score(int a[], int k) {
    LL x = 0;
    for (int i = 1; i <= k; i++) {
        x ^= (LL)i * a[i];
    }
    return x;
}

LL tree2p() {
    for (int i = 1; i <= n-1; i++) {
        scanf("%d", &f[i]);
        d[f[i]]++;
    }
    for (int i = 1, j = 1; i <= n-2; j++) {
        while (d[j]) j++;
        p[i++] = f[j];
        
        // i: p.size()
        // p[i-1]: p.back()
        while (i <= n-2 && --d[p[i-1]] == 0 && p[i-1] < j)
            p[i] = f[p[i-1]], i++;
    }
    return score(p, n-2);
}

LL p2tree() {
    for (int i = 1; i <= n-2; i++) {
        scanf("%d", &p[i]);
        d[p[i]]++;
    }
    p[n-1] = n, d[n]++;
    for (int i = 1, j = 1; i <= n-1; j++) {
        while (d[j]) j++;
        f[j] = p[i++];
        while (i <= n-1 && --d[p[i-1]] == 0 && p[i-1] < j)
            f[p[i-1]] = p[i], i++;
    }
    return score(f, n-1);
}

int main() {
    scanf("%d%d", &n, &m);
    if (m == 1) printf("%lld\n", tree2p());
    else printf("%lld\n", p2tree());
    return 0;
}
```

