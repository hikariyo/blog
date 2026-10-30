---
date: 2026-10-24 14:54:00
updated: 2026-10-25 17:37:00
title: 数学 - 期望
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 若干期望题目。
---

## 前言

随机变量 $X$ 的期望定义如下：
$$
\mathbb E[X]=\sum_x x\Pr(X=x)
$$
在事件 $Y=y$ 发生的情况下，条件期望的定义：
$$
\mathbb E[X\mid Y=y]=\sum_x x\Pr(X=x\mid Y=y)
$$
注意，此时 $\mathbb E[X\mid Y=y]$ 是 $y$ 的函数，不再是一个常数。

根据全概率公式，我们可以推出全期望公式：
$$
\begin{aligned}
\mathbb E[X]&=\sum_x x\Pr(X=x)\\
&=\sum_x\sum_y x\Pr(X=x\mid Y=y)\Pr(Y=y)\\
&=\sum_y \Pr(Y=y)\left(\sum_xx\Pr(X=x\mid Y=y)\right)\\
&=\sum_i \Pr(Y=y)\mathbb E[X\mid Y=y]\\
&=\mathbb E[\mathbb E[X\mid Y]]\\
\end{aligned}
$$
对若干步递推中的事件划分出合适的随机变量，可以帮助我们更方便地推导期望相关公式。

## P1654

记随机变量 $X_i$ 为 $i$ 轮后的得分，$T_i$ 表示 $i$ 轮后末尾连续段的长度。

那么就可以得到：
$$
\begin{aligned}
\mathbb E[X_i-X_{i-1}\mid T_{i-1}] &=(1-p_i)\times 0+p_i[(T_{i-1}+1)^3-T_{i-1}^3]\\
&=3p_i T_{i-1}^2+3p_i T_{i-1}+p_i
\end{aligned}
$$


紧接着，我们给出 $T_i^2$ 与 $T_i$ 的期望递推公式：
$$
\begin{aligned}
\mathbb E[T_i\mid T_{i-1}]&=(1-p_i)\times 0 + p_i(T_{i-1}+1)\\
&=p_iT_{i-1}+p_i\\

\mathbb E[T_i^2\mid T_{i-1}]&=(1-p_i)\times 0 + p_i(T_{i-1}+1)^2\\
&=p_iT_{i-1}^2+2p_iT_{i-1}+p_i\\
\end{aligned}
$$
核心的思想是定义合适的随机变量，使得能够根据描述写出条件期望的式子。

复杂度 $O(n)$。

## [CCPC 2023 Shenzhen] Rooted Tree

记执行 $i$ 次操作后的树为 $T_i$，并且定义节点深度之和 $X_i=f(T_i)$，叶子深度之和 $Y_i=g(T_i)$，并且叶子数量 $L_i=1+(M-1)i$。

考虑执行一次操作后 $X_i$ 的变化：
$$
\begin{aligned}
\mathbb E[X_i-X_{i-1}\mid T_{i-1}] &= \sum_{u\in \operatorname{leaf}(T_{i-1})}\frac{(d_u+1)M}{L_{i-1}}\\
&=\frac{MY_{i-1}}{L_{i-1}}+M
\end{aligned}
$$
同时，我们也需要递推 $Y_i$ 的变化：
$$
\begin{aligned}
\mathbb E[Y_i-Y_{i-1}\mid T_{i-1}]&=\sum_{u\in\operatorname{leaf}(T_{i-1})}\frac{(d_u+1)M-d_u}{L_{i-1}}\\
&=\frac{(M-1)Y_{i-1}}{L_{i-1}}+M
\end{aligned}
$$
整理可以得到：
$$
\frac{\mathbb E[Y_i]}{1+(M-1)i}=\frac{\mathbb E[Y_{i-1}]}{1+(M-1)(i-1)}+\frac{M}{1+M(i-1)}
$$
分别维护分子分母即可做到 $O(1)$ 递推出所有 $\mathbb E[Y_i]$，也就能推出 $\mathbb E[X_i]$。

复杂度 $O(K+\log P)$。

## [THUCAMP 2026 Day3] To Make You Cry

> 给定 $n\le 3\times 10^5$ 个节点的树，每条边 $i$ 有相互独立的 $x_i\%$ 的概率不被删除，每个点 $i$ 有权值 $w_i(1\le w_i\le 10)$，求最大权独立集的期望，对 $10^9+7$ 取模。

考虑不删边的时候怎么做。定义 $dp_{u,0}$ 表示不选 $u$，$dp_{u,1}$ 表示选 $u$，那么转移自然是：
$$
\begin{aligned}
dp_{u,0} &= \sum_{v\in\operatorname{son}(u)} \max(dp_{v,0},dp_{v,1})\\
dp_{u,1} &= w_u+\sum_{v\in\operatorname{son}(u)} dp_{v,0}
\end{aligned}
$$
如果 $(u,v)$ 这条边被删掉，那么 $dp_{u,1}$ 也可以被 $\max(dp_{v,0},dp_{v,1})$ 进行转移，于是题意目前就是 $\mathbb E[\max(dp_{1,0},dp_{1,1})]$。

一个众所周知的事实是，一般来讲 $\mathbb E[\max(X,Y)]\neq \max(\mathbb E[X],\mathbb E[Y])$，所以我们需要把 $\max$ 设计到期望里面去。

直接根据转移方程求出 $\mathbb E[dp_{u,0}]$ 与 $\mathbb E[dp_{u,1}]$ 即可，主要的问题是如何求出 $\mathbb E[\max(dp_{u,0},dp_{u,1})]$。

考虑到这里我们需要以某种方式比较 $dp_{u,0}$ 和 $dp_{u,1}$，所以给它做个差：
$$
dp_{u,1}-dp_{u,0}=w_u-\sum_{v\in\operatorname{son}(u)} \max(0,dp_{v,1}-dp_{v,0})
$$
显然可以看出 $dp_{u,1}-dp_{u,0}\le w_u\le 10$，并且小于 $0$ 的值我们是不关心的，所以记 $f_{u,j}$ 表示 $\max(0,dp_{u,1}-dp_{u,0})=j$ 的概率。

初始 $f_{u,w_u}=1$，枚举每个 $(u,v)$，更新：
$$
\begin{aligned}
\forall k, f_{u,\max(0,j-k)} &\gets f_{v,k} \times f_{u,j} \times \frac{x_{(u,v)}}{100}\\
f_{u,j}&\gets f_{u,j}\times \left(1-\frac{x_{(u,v)}}{100}\right)
\end{aligned}
$$
即枚举不删和删两种情况，如果删边则 $\max(0,dp_{u,1}-dp_{u,0})$ 不变化，如果不删则需要枚举子节点 $v$ 的 $\max(0, dp_{v,1}-dp_{v,0})$ 的值。

那么就有：
$$
\begin{aligned}
\mathbb E[\max(dp_{u,0},dp_{u,1})]&=\mathbb E[\max(0,dp_{u,1}-dp_{u,0})+dp_{u,0}]\\
&=\mathbb E[\max(0,dp_{u,1}-dp_{u,0})]+\mathbb E[dp_{u,0}]\\
&=\sum_{j=0}^{10} jf_{u,j}+\mathbb E[dp_{u,0}]
\end{aligned}
$$
复杂度 $O(n(\max w)^2)$。

## [ZJCPC2026] PalindromemordnilaP

首先，每一种操作相当于在 $n!$ 个排列中等概率选择一种，并且当 $s$ 成为回文串时停止操作，并且其权值是停止之前的操作次数。

对一个 $(i,n-i+1)$ 的 Pair 进行分类讨论：

1. 若 $n$ 为奇数，那么中心位置改不改都可以。
2. 若 $s_i=s_{n-i+1}$，那么 $s_i=c$ 时，这两个位置改不改都可以；$s_i\neq c$ 时，这两个位置要么都改、要么都不改。
3. 若 $s_i\neq s_{n-i+1}$，那么当两个位置都不为 $c$ 时，这两个位置都必须改；当其中一个位置是 $c$ 时，另一个位置必须改，这个位置改不改都可以。

我们设改不改都可以的位置数量为 $p$，必须改的位置数量为 $a$，两个位置要么都改、要么都不改的 Pair 数量为 $b$。

这样一来，一个排列的权值就是 $a$ 类位置出现的最后一个下标，并且要保证此时 $b$ 类 Pair 要么都改、要么都不改。

由于一个排列删掉改不改都可以的 $p$ 个位置后不会影响停止操作的相对位置，所以我们不妨先把这些位置删掉，最后再加回来。

现在的问题是，对于这 $(a+2b)!$ 个排列中，统计权值为 $a,a+2,\dots,a+2b$ 的排列的数量。

设 $f_i$ 表示**给定** $i$ 个 Pair，长度为 $a+2i$ 且在最后一个位置终止的排列的数量，注意这里是 $a$ 个必改位置和给定的 $2i$ 个位置。

于是我们对这 $(a+2i)!$ 个排列的终止位置进行划分：
$$
\begin{aligned}
(a+2i)!&=\sum_{j=0}^i \binom{i}{j} f_j(2i-2j)!\\
\frac{(a+2i)!}{i!}&=\sum_{j=0}^i \frac{f_j}{j!}\frac{(2i-2j)!}{(i-j)!}
\end{aligned}
$$
右侧是一个卷积的形式，所以我们定义 $f_i/i!$ 的生成函数为 $F$，$2i!/i!$ 的生成函数为 $G$，$(a+2i)!/i!$ 的生成函数为 $H$，就有：
$$
F=\frac{H}{G}
$$
于是只需要多项式求逆即可求出所有 $f_i$。

但是我们仍然需要最后把 $p$ 个位置插回原排列，这就用到了我们之前提到的条件期望。

具体地说，设 $K$ 是删掉 $p$ 个位置后排列的权值，$X$ 是加回来后排列的权值，那么可以列出表达式：
$$
\mathbb E[X\mid K=k]=k+\mathbb E\left[\sum_{i=0}^{k-1} x_i\right]
$$
其中 $x_i$ 表示在 $a+2b$ 个位置中，第 $i$ 个位置后面插入的 $p$ 类位置的数量。

由对称性，我们得到 $\mathbb E[x_0]=\mathbb E[x_1]=\cdots=\mathbb E[x_{a+2b}]$，又因为 $\sum_{i=0}^{a+2b} x_i=p$，因此就有：
$$
\begin{aligned}
\mathbb E[X\mid K=k]&=k+\mathbb E\left[\sum_{i=0}^{k-1} x_i\right]\\
&=k+k\mathbb E[x_0]\\
&=k\left(1+\frac{p}{a+2b+1}\right)
\end{aligned}
$$
最后给出 $\mathbb E[K]$ 的表达式：
$$
\mathbb E[K]=\sum_{j=0}^{b}\frac{1}{(a+2b)!}\binom{b}{j}f_j(2b-2j)!(a+2j)
$$
复杂度 $O(n\log n)$。
