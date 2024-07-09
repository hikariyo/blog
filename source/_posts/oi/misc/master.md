---
date: 2024-01-16 18:26:00
title: 杂项 - 主定理及证明
katex: true
tags:
- Algo
- C++
categories:
- OI
description: 分析递归的时间复杂度时通常需要用到主定理，我之前一直当作黑盒去用它，但是越来越想了解一下这个定理是如何证明的，发现证明过程只用到了等比级数与对数的相关公式，这里记录一下。
---

## 简介

分析递归的时间复杂度时通常需要用到主定理，我之前一直当作黑盒去用它，但是越来越想了解一下这个定理是如何证明的，发现证明过程只用到了等比级数与对数的相关公式，这里记录一下。

## 主定理

对于递归式 $T(n)=aT(\cfrac{n}{b})+f(n)$ 的时间复杂度，主定理可以求解下面四种情况：

1. 当 $f(n)=O(n^{\log_ba -\epsilon})$ 其中 $\epsilon >0$ 为常数，那么 $T(n)=\Theta(n^{\log_ba})$。
2. 当 $f(n)=\Theta(n^{\log_ba})$，那么 $T(n)=\Theta(n^{\log_ba}\log n)$。
3. 当 $f(n)=\Omega(n^{\log_ba+\epsilon})$，对于常数 $0<c<1$ 和所有足够大的 $n$ 满足 $af(\cfrac{n}{b})\le cf(n)$ 那么 $T(n)=\Theta(f(n))$。
4. 当 $f(n)=\Theta(n^{\log_ba}\log^kn)$ 其中 $k>0$ 为常数，那么 $T(n)=\Theta(n^{\log_ba}\log^{k+1}n)$。

复习一下复杂度记号：$O$ 代表上界，$\Theta$ 代表同级，$\Omega$ 代表下界。由于本文是复杂度相关证明，所以就严谨一点。

## 证明

首先画出递归树，这里贴上算法导论第四版原版 109 页中的图：

![Introduction to Algorithms (Fourth Edition) P109](/img/2024/1/master.jpg)

设根结点深度为 $0$，可以看出深度为 $j$ 的层有 $a^j$ 个结点，每个结点的值是 $f(n/b)$ 那么此层之和为 $a^jf(n/b^j)$，并且最后一层有 $\Theta(a^{\log_bn})=\Theta(n^{\log_b a})$ 个结点，所以所有结点之和就是：
$$
\Theta(n^{\log_b a})+\sum_{j=0}^{\log_b n}a^jf(\frac{n}{b^j})
$$
令 $g(n)=\sum_{j=0}^{\log_bn}a^jf(n/b^j)$，下面分类讨论。

### Case I

当 $f(n)=O(n^{\log_b a-\epsilon})$，代入：
$$
\begin{aligned}
g(n)&=O\left (\sum_{j=0}^{\log_b n} a^j\Big(\frac{n}{b^j}\Big)^{\log_ba-\epsilon}\right )\\
&=O\left( \sum_{j=0}^{\log_bn}a^j\Big(\frac{n^{\log_ba-\epsilon}(b^{\epsilon})^j}{a^j}\Big) \right)\\
&=O\left(n^{\log_ba-\epsilon }\sum_{j=0}^{\log_b n}(b^{\epsilon})^j\right)\\
&=O\left( n^{\log_ba-\epsilon}\frac{n^{\epsilon}-1}{b^{\epsilon}-1}\right)\\
&=O\left( n^{\log_ba}\right)\\
\end{aligned}
$$
因此 $T(n)=\Theta(n^{\log_ba})$， 证毕。

### Case II

当 $f(n)=\Theta(n^{\log_ba})$，代入：
$$
\begin{aligned}
g(n)&=\Theta\left(\sum_{j=0}^{\log_bn}a^j\Big(\frac{n}{b^j}\Big)^{\log_ba}\right)\\
&=\Theta\left(\sum_{j=0}^{\log_bn} a^j\frac{n^{\log_ba}}{a^j}\right)\\
&=\Theta\left(n^{\log_ba}\log_bn\right)\\
&=\Theta\left(n^{\log_ba}\log n\right)
\end{aligned}
$$
因此 $T(n)=\Theta(n^{\log_ba}\log n)$，证毕。

### Case III

当 $f(n)=\Omega(n^{\log_ba+\epsilon})$，对于常数 $0<c<1$ 和所有足够大的 $n$ 满足 $af(\cfrac{n}{b})\le cf(n)$，当 $f(n)=n^{\log_ba+\epsilon}$ 时，这实际上是要求：
$$
\begin{aligned}
a(\frac{n}{b})^{\log_ba+\epsilon}&\le cn^{\log_ba+\epsilon}\\
\frac{1}{b^{\epsilon}}&\le c
\end{aligned}
$$
也就是要求 $b>1$，然后就可以用等比级数放缩：
$$
\begin{aligned}
g(n)&=\sum_{j=0}^{\log_bn}a^jf(\frac{n}{b^j})\\
&\le \sum_{j=0}^{\log_bn}c^jf(n)\\
&\le f(n)\sum_{j=0}^{\infin} c^j\\
&= f(n)(\frac{1}{1-c})\\
&=O\left(f(n)\right)
\end{aligned}
$$
根据 $g(n)$ 的定义，$g(n)=f(n)+af(n/b)+\cdots\ge f(n)$，所以 $g(n)=\Theta(f(n))$，因此 $T(n)=\Theta(f(n))$，证毕。

### Case IV

当 $f(n)=\Theta(n^{\log_ba}\log^kn)$ 其中 $k>0$ 为常数，代入：
$$
\begin{aligned}
g(n)&=\Theta\left(\sum_{j=0}^{\log_bn}a^j\Big(\frac{n}{b^j}\Big)^{\log_ba}\log^k\frac{n}{b^j}\right)\\
&=\Theta\left(n^{\log_ba}\sum_{j=0}^{\log_bn}\log^k\frac{n}{b^j}\right)\\
&=\Theta\left(n^{\log_ba}\log^{k}n\log_bn\right)\\
&=\Theta\left(n^{\log_ba}\log^{k+1}n\right)\\
\end{aligned}
$$
因此 $T(n)=\Theta(n^{\log_ba}\log^{k+1}n)$，证毕。

## 总结

我的记忆方法：对于递归式 $T(n)=aT(\cfrac{n}{b})+f(n)$，将 $f(n)$ 与 $n^{\log_ba}$ 比较，相等时为 $\Theta(n^{\log_ba}\log n)$，否则当 $f(n)$ 为多项式时，由两者较大的决定复杂度。对于 $f(n)=\Theta(n^{\log_ba}\log^kn)$ 特殊记忆。

仅当 $f(n)$ 与 $n^{\log_ba}$ 相差 $n^\epsilon$ 阶或者 $\log^{\epsilon}n$ 阶时，其中 $\epsilon$ 为常数，才能用使用主定理求解复杂度，否则需要用到 Akra-Bazzi 定理，本文就不涉及了，因为我不会。
