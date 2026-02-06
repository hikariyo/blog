---
date: 2026-02-02 23:52:00
updated: 2026-02-06 15:30:00
title: 构造 - 专题训练 II
katex: true
tags:
- Algo
- C++
- Constructive
categories:
- XCPC
description: 本专题记录广义的构造题目，只要带有构造性步骤的都算，每篇文章至多 10 题。
---

## [CF1463B] Find The Array

考虑构造出一种情况 $b_i,b_{i+1}$ 之间的约束条件永真，一个自然的想法是令 $b_i$ 为 $2$ 的幂。

既然是 $2$ 的幂次，那就让每一个绝对值里的值尽可能小，所以这里找 $k$ 使得 $2^k \le a_i<2^{k+1}$。

此时 $2\sum |a_i-b_i|=\sum(2a_i-2^{k+1})<\sum a_i=S$。

题解给出了另一种方法，既然想让这个约束条件永真，只要相邻两个元素有一个 $1$ 也能满足这一点，自然另一个就填对应的 $a_i$。

然后，$2\sum_{i=\texttt{odd}}(a_i-1)+2\sum_{i=\texttt{even}} (a_i-1)=2S-2n$，所以一定有一个值 $\le S-n<S$。

[AC Code](https://gist.github.com/hikariyo/184ed2cf1de1c8b6aa7507b53ac9856c)。

## [CF1366D] Two Divisors

首先，$(d_1,d_2)\mid d_1+d_2$，所以 $(d_1,d_2)=1$ 是 $(d_1+d_2,a_i)=1$ 成立的必要条件。

进一步地，从算术基本定理的角度考虑，把 $a_i$ 分解后变成 $p_1^{k_1}p_2^{k_2}\dots p_m^{k_m}$。

显然 $m=1$ 时，任意一个非 $1$ 的因子 $d$ 都满足 $p_1\mid d$，不可能找到 $d_1,d_2$ 满足 $d_1\perp d_2$。

当 $m\ge 2$ 时，我们需要找到 $d_1,d_2$ 满足 $d_1\perp d_2$ 并且 $\forall 1\le j\le m, p_j\not\mid d_1+d_2$。

当 $p_j\mid d_1$ 时，$p_j\mid d_1+d_2$ 成立当且仅当 $p_j\mid d_2$，而又因为 $d_1\perp d_2$ 得知这是不可能的。

因此这就给出了一个构造方法，让每一个 $p_j$ 都是 $d_1$ 或 $d_2$ 的因子即可。

[AC Code](https://gist.github.com/hikariyo/0f0750cc82d468770c65bebd28a2ee68)。

