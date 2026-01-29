---
date: 2025-12-15 12:14:00
updated: 2025-12-15 12:14:00
title: 数学 - 拉格朗日插值
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 拉格朗日插值在算法竞赛中的应用。
---

## 定义

给定 $n$ 个点 $(x_i,y_i)$ 满足 $x_i\neq x_j(i\neq j)$，它们能唯一确定一个 $n-1$ 次的多项式 $f(x)$：
$$
f(x)=\sum_{i=1}^{n}y_i\prod_{j\neq i}\frac{x-x_j}{x_i-x_j}
$$
容易验证，带入 $f(x_k)$，当 $i\neq k$ 时，后面的那个乘积总会有 $0$；当 $i=k$ 时，后面的乘积为 $1$，并且这个多项式是 $n-1$ 次的。

## [LGP5667] 拉格朗日插值2

这个题目相较于模板题更为常见，给定的 $x_i=i$。

设 $h_i=f(m+i)$，数据范围保证了 $m>n$，要求的是 $h_0\sim h_n$。

列式子：
$$
\begin{aligned}
h_k&=\sum_{i=0}^n y_i\prod_{j\neq i}\frac{m+k-j}{i-j}\\
&=(m+k)^{\underline{n+1}}\sum_{i=0}^n \frac{y_i}{i!(n-i)!(m+k-i)(-1)^{n-i}}\\
&=(m+k)^{\underline{n+1}}\sum_{i=0}^n \frac{f_i}{m+k-i}\\
\end{aligned}
$$
其中 $f_i=\cfrac{y_i}{i!(n-i)!(-1)^{n-i}}$，这是可以 $O(n)$ 预处理出来的。

观察这个形式，我们希望右边是卷积 $\sum_{i=0}^k f_ig_{k-i}$ 的样子，但是这个求和是到 $n$，所以我们需要进行一步转化。

具体地说，令 $g_{n+k-i}=\cfrac{1}{m+k-i}$，则 $g_i=\cfrac{1}{m-n+i}$，就有：
$$
h_k=(m+k)^{\underline{n+1}}\sum_{i=0}^{n+k} f_i g_{n+k-i}
$$
当 $i>n$ 时，$f_i=0$，所以这和原式是等价的。

于是，我们只要预处理出 $f,g$ 数组，做卷积 $c=f*g$，就有 $h_k=(m+k)^{\underline{n+1}}c_{n+k}$，前面的下降幂维护一个窗口即可。

[AC Code](https://gist.github.com/hikariyo/7af630a928f4cc6bbb4734d5529b3ca8)。

## [CF622F] The Sum of the k-th Powers

首先 $S(x)=\sum_{i=1}^x i^k$ 是一个 $k+1$ 次的多项式，于是我们可以根据 $(j,\sum_{i=0}^j i^k)$ 令 $j=0\sim k+1$ 即可插值出这样一个多项式 $S(x)$。

我们只需要求 $S(n)$ 这个点处的值，当 $n\le k+1$ 时我们直接把预处理的信息输出即可，当 $n>k+1$ 时我们带入公式：
$$
\begin{aligned}
S_n&=\sum_{i=0}^{k+1}S_i\prod_{i\neq j}\frac{n-j}{i-j}\\
&=\sum_{i=0}^{k+1} S_i\frac{n^{\underline{k+2}}}{i!(k+1-i)!(-1)^{k+1-i}(n-i)}\\
&=n^{\underline{k+2}}\sum_{i=0}^{k+1} \frac{S_i}{i!(k+1-i)!(-1)^{k+1-i}(n-i)}
\end{aligned}
$$
直接 $O(k\log P)$ 做即可，其中 $P$ 是模数。

[AC Code](https://gist.github.com/hikariyo/d96620b96eef0bbf9470427eb30da28d)。

## [ICPC2019 Nanchang Inventional] Polynomial

首先我们要清楚，对于 $S(x)=\sum_{i=0}^x f(i)$ 是一个 $n+1$ 次的多项式，所以我们可以通过 $(j,S(j))$ 其中 $j=0\sim n+1$ 来唯一地确定这个多项式。

因此我们可以通过一次拉格朗日插值求出 $f(n+1)$，然后对 $f$ 求前缀和即可得到 $S_0\sim S_{n+1}$。

对于每次询问，求 $S_R-S_{L-1}$ 即可，复杂度为 $O(Tmn\log P)$，其中 $P$ 是模数。

[AC Code](https://gist.github.com/hikariyo/c86a83dd765788df886dbbffdd8acac9)。

## [ICPC2021 Brazil Subregional] Assigning Prizes

设 $dp(i,j)$ 表示 $x_i=j$ 时，$i\sim n$ 的方案数。

转移显然有 $dp(i,j)=\sum_{k=p_{i+1}}^j dp(i+1, k)=S(i+1, j)-S(i+1, p_{i+1}-1)$，并且当 $S(n,j)=j-p_n+1(j\ge p_n)$。

注意，这里 $p_i\gets \max_{j=i}^n p_j$，我们希望当 $j<p_i$ 时， $dp(i,j)=0$。

观察这个关系，我们可以看出，$S(n,x)$ 是一条直线，而 $S(n-1,x)$ 是连续的 $S(n,y)$ 的和，所以它是一个二次的曲线，以此类推，可以推出 $S(i,x)$ 能确定一个 $n-i+1$ 次的多项式。

然而，如果你直接这样做，是无论如何也无法维护好 $dp$ 和 $S$ 的关系的，因为前后位置的下标对应都不一样。

因此，由于 $(p_i,S(i,p_i))\sim (R,S(i,R))$ 这些点在一个 $n-i+1$ 次的多项式上，我们令 $(0,S(i,0))\sim (p_i-1,S(i,p_i-1))$ 也在这个多项式上。

拓展完定义后我们需要保证当题目数据给出 $p_1>R$ 时，程序输出 $0$，这里需要**特判**。

那么对于 $S(i, x)$ 这个多项式，我们可以先根据文章一开头给出的递推式计算出 $dp(i, x)$。由于 $S(i+1,x)$ 是一个 $n-i$ 次的多项式，那么连续的和就是一个 $n-i+1$ 次的多项式。

对于 $\prod_{j=0}^{i-1} (x-j)\prod_{j=i+1}^{deg}(x-j)$ 我们需要预处理前缀后缀积，让拉格朗日插值的复杂度单次为 $O(n)$。上面题目并没有卡 $\log P$ 的做法，这个题目卡了，所以最终的复杂度为 $O(n^2)$。

[AC Code](https://gist.github.com/hikariyo/7f4596917fc7c0dfbde4f47acb5ad00a)。

