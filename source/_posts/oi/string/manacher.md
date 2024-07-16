---
date: 2024-07-16 18:44:00
title: 字符串 - Manacher
katex: true
tags:
- Algo
- C++
- String
categories:
- OI
description: Manacher 可以在 O(n) 的复杂度内求解所有位置的最长回文串。
---

## Manacher

这个算法的思想是枚举回文串的中心点 $1\sim n$，记录下当前右边界最靠右的回文串位置 $mr$，以及回文串中心位置 $mid$。

为了保证回文串一定存在中心位置，即回文串的长度为奇数，那么需要在每两个字符之间加入一个辅助字符，它只需要没在原串中出现过即可。然后需要在开头和末尾加上两个不同的辅助字符，这样保证不会枚举越界。

事实上，只需要在开头加上一个辅助字符即可，此时末尾为默认的 $\texttt{NUL}$ 字符，可以保证不会越界。

用 $p_i$ 表示以 $i$ 为回文串中心的半径最大长度，当枚举到 $i$ 时，与其对称的点就是 $j=2mid-i$，存在这样的两种情况：

1. $mid<i\le mr$，当 $p_j$ 当前记录回文串内部，一定有 $p_i=p_j$；如果在外部，由于当前记录的是一个极大值，必然有 $p_i=mr-i+1$ 即 $p_i$ 在当前记录的回文串内部；如果 $p_j$ 恰好在边界处，无法得知 $i$ 是否也是恰好在边界处，因为此时 $i$ 向外扩不与当前维护的是极大值这个条件矛盾。
2. $mr<i$，此时无法从前面的递推，将 $p_i$ 赋值为 $1$​。

赋值结束后直接暴力求解 $p_i$，然后更新 $mid,mr$。因为执行循环后一定会更新 $mr$，且 $mr$ 一定是单调递增的，所以循环的执行次数是 $O(n)$ 的。因此整个算法的复杂度为 $O(n)$。下面在模板题中说明具体实现与理论推导中的微小差异。

## 模版

> 给定一个只有小写英文字母构成的字符串 $S$，求 $S$ 的最长回文串长度，其中 $1\le |S|\le 1.1\times 10^7$。
>
> 题目链接：[P3805](https://www.luogu.com.cn/problem/P3805)。

首先，暴力求解 $p_i$ 时通过判断 $s_{i+p_i}$ 与 $s_{i-p_i}$ 就可以判断 $p_i$ 是否该继续增加，这里正好当 $p_i$ 为 $k$ 时是在验证 $k+1$ 是否成立，比较巧妙。其次，$mr$ 存的实际上是 $mr+1$，即右边界的右边一个点，因为这样直接可以通过 $i+p_i$ 来更新到，不需要 $-1$；且在赋值 $p_i$ 时的第二种情况也不需要 $+1$，直接 $mr-i$​ 即可。

因为一个长度为 $k$ 的回文串经过处理后会变成 $2k+1$，所以在 $i$ 处的回文串长度为 $2p_i-1=2k+1$ 得到 $k=p_i-1$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 2.2e7 + 10;
char a[N], s[N];
int n, p[N];

void build() {
    int k = 0;
    s[++k] = '@', s[++k] = '|';
    for (int i = 1; i <= n; i++) s[++k] = a[i], s[++k] = '|';
    s[++k] = '^';
    n = k;
}

void manacher() {
    int mid = -1, mr = 0;
    for (int i = 1; i <= n; i++) {
        if (i < mr) p[i] = min(p[2*mid - i], mr - i);
        else p[i] = 1;
        
        while (s[i + p[i]] == s[i - p[i]]) p[i]++;
        if (i + p[i] > mr) mr = i + p[i], mid = i;
    }
}

int main() {
    scanf("%s", a+1);
    n = strlen(a+1);
    build();
    manacher();

    int ans = 0;
    for (int i = 1; i <= n; i++) ans = max(ans, p[i] - 1);
    printf("%d\n", ans);
    return 0;
}
```

