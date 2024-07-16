---
date: 2024-06-17 20:14:00
title: 多项式 - 常系数齐次线性递推
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
description: 对于一个线性递推，利用多项式取模可以做到比矩阵优秀很多的复杂度，当然也比较难写。
---

## 常系数齐次线性递推

> 给定 $n,k,f_1\sim f_k,a_0\sim a_{k-1}$，若：
> $$
> a_i= \sum_{j=1}^kf_ja_{i-j}
> $$
> 求 $a_n\bmod 998244353$ 的值，其中 $n=10^9,k=32000$。
>
> 题目链接：[P4723](https://www.luogu.com.cn/problem/P4723)。 

考虑一个朴素的斐波那契数列的展开过程：
$$
\begin{aligned}
a_5&=a_4+a_3\\
&=2a_3+a_2\\
&=3a_2+2a_1\\
&=5a_1+3a_0
\end{aligned}
$$
如果从一个多项式的角度来看：
$$
0x^0+0x^1+0x^2+0x^3+0x^4+\textcolor{red}{1}x^5\\
0x^0+0x^1+0x^2+\textcolor{red}1x^3+\textcolor{red}1x^4+0x^5\\
0x^0+0x^1+\textcolor{red}1x^2+\textcolor{red}2x^3+0x^4+0x^5\\
0x^0+\textcolor{red}2x^1+\textcolor{red}3x^2+0x^3+0x^4+0x^5\\
\textcolor{red}3x^0+\textcolor{red}5x^1+0x^2+0x^3+0x^4+0x^5\\
$$
做的事情就是不断地减去 $\lambda x^\mu(x^2-x-1)$，这就是一个取模的过程。因此 $a_n$ 的取值就 $x^n\bmod F(x)$ 有关，其中 $F(x)$ 是数列的特征多项式。具体地：
$$
F(x)=x^k+\sum_{j=1}^k-f_jx^{k-j}
$$
则 $a_n$ 满足的式子是：
$$
a_n=\sum_{i=0}^{k-1} [x^i](x^n\bmod F(x))a_i
$$
当然，需要 $a_0\sim a_{k-1}$​ 已知，否则也推不出来定值。多项式的题目最经典的错误是没有清空数组，只能说多注意一下吧，这种错误很难调。这样快速幂加上取模的复杂度是 $O(k\log k\log n)$，如果是矩阵乘法需要 $O(k^3\log n)$​，还是有显著的优化的。

```cpp
#include <bits/stdc++.h>
using namespace std;

/* Template Poly */

int n, k, res, a[N];
Poly A, B;

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    cin >> n >> k;
    B.resize(k);
    for (int i = 0; i < k; i++) cin >> B[k-i-1], B[k-i-1] = ((-B[k-i-1]) % P + P) % P;
    B[k] = 1;
    for (int i = 0; i < k; i++) cin >> a[i], a[i] = (a[i] % P + P) % P;
    A.resize(1);
    A[1] = 1;
    A = A.fqmi(n, B);
    for (int i = 0; i < k; i++) res = (res + A[i] * a[i]) % P;
    cout << res << endl;
    return 0;
}
```

