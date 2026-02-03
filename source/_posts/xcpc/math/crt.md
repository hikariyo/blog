---
date: 2026-02-03 15:18:00
updated: 2026-02-03 15:18:00
title: 数学 - 中国剩余定理
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 本文主要记录 exCRT 的数学推导。
---

## [模板] exCRT

先考虑两个方程的情况，即 $x\equiv x_1\pmod{m_1},x\equiv x_2\pmod{m_2}$。

第一个式子等价于 $\exists k \in \mathbb Z,x=km_1+x_1$。

代入第二个式子，有 $km_1\equiv x_2-x_1\pmod{m_2}$。

这等价于 $\exists q\in \mathbb Z,km_1+qm_2=x_2-x_1$。

左边可以看作 $m_1$ 与 $m_2$ 的线性组合，题目保证这个方程有解，那么一定有 $(m_1,m_2)\mid x_2-x_1$。

所以这又等价于 $k\cfrac{m_1}{(m_1,m_2)}\equiv\cfrac{x_2-x_1}{(m_1,m_2)}\pmod{\cfrac{m_2}{(m_1,m_2)}}$。

由于 $\left(\cfrac{m_1}{(m_1,m_2)},\cfrac{m_2}{(m_1,m_2)}\right)=1$，那么这个方程在模 $\cfrac{m_2}{(m_1,m_2)}$ 的意义下是有唯一解的，记作 $k_0$。

所以 $\exist t\in \mathbb Z,k=k_0+t\cfrac{m_2}{(m_1,m_2)}$，带回 $x=km_1+x_1$，得到 $x=k_0m_1+x_1+\cfrac{tm_1m_2}{(m_1,m_2)}$。

上面只说明了所有解应当满足 $x\equiv k_0m_1+x_1\pmod{\cfrac{m_1m_2}{(m_1,m_2)}}$，并且找到了一个解。

但是反过来，如果 $x\equiv k_0m_1+x_1\pmod{\cfrac{m_1m_2}{(m_1,m_2)}}$，它一定是原方程的解吗？

设 $x=k_0m_1+x_1+s\cfrac{m_1m_2}{(m_1,m_2)}, s\in \mathbb Z$，那么 $x\equiv k_0m_1+x_1\equiv x_1\pmod{m_1}$，第一式成立；

接下来 $x\equiv k_0m_1+x_1\pmod{m_2}$，我们知道 $k_0\cfrac{m_1}{(m_1,m_2)}\equiv \cfrac{x_2-x_1}{(m_1,m_2)}\pmod{\cfrac{m_2}{(m_1,m_2)}}$，那么 $k_0m_1\equiv x_2-x_1\pmod{m_2}$，于是 $x\equiv x_2-x_1+x_1\equiv x_2\pmod{m_2}$。

所以我们证明了 $x$ 的所有解形如 $k_0m_1+x_1+s\cfrac{m_1m_2}{(m_1,m_2)},s\in\mathbb Z$。

那么这两个方程的全部解就是 $x\equiv k_0m_1+x_1\pmod{\cfrac{m_1m_2}{(m_1,m_2)}}$ 的全部解，我们把两个方程合并为了一个方程。

[AC Code](https://gist.github.com/hikariyo/38d6070a452e2afbf775adfa87e3c28e)。

## [CF1770C] Koxia and Number Theory

首先 $a_1\sim a_n$ 必须两两不同，然后可以给 $a$ 排个序。

然后对于 $i<j$，我们有 $(a_i+x,a_j+x)=(a_i+x,a_j-a_i)=1$。

这意味着什么？对于质数 $p$，我们有若 $p\mid (a_j-a_i)$，应当有 $p\not\mid (a_i+x)$。

于是在 $x\not\equiv -a_i\pmod p$，于是在 $\text{mod }p$ 意义下排掉了一个解。

根据中国剩余定理，对于方程组：
$$
\begin{cases}
x\equiv x_1\pmod{p_1}\\
x\equiv x_2\pmod{p_2}\\
\vdots\\
x\equiv x_k\pmod{p_k}
\end{cases}
$$
其中 $p_1\sim p_k$ 是互不相同的质数，是一定可以构造出来解的，因为合并的过程中两个模数 $m_1,m_2$ 始终满足 $(m_1,m_2)=1$。

对于每一种 $p$，我们至多能排除掉 $n$ 个数，所以对于 $p>n$ 是一定不会被全部排除掉的，所以我们只需要关心 $p\le n$ 的情况。

所以我们预处理出 $\le n$ 的质数，然后枚举这些质数，看是否有 $p\mid(a_j-a_i)$，如果有的话我们需要在 $\text{mod }p$ 的意义下排除掉 $-a_i$。

可以用 `set` 维护，那么复杂度就是 $O(\sum n^2\pi(n)\log n)$。

[AC Code](https://gist.github.com/hikariyo/ca9478f955e40363ed9a7fef639f5114)。