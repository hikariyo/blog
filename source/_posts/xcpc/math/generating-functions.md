---
date: 2025-12-15 12:14:00
updated: 2025-12-15 12:14:00
title: 数学 - 生成函数基础
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 不包含 FFT/NTT 的基础生成函数题目。
---

## 前言

之前一直无法理解生成函数的这些概念，最近算是入了门，下面我记录下我自己理解这些式子的方式。

我们使用生成函数的最终目的是计数，所以多项式的项次数超过某个值对于我们来说就没用了，于是我们可以在$\pmod{x^n}$ 的意义下理解逆元、复合等概念。

## 常生成函数

> 一个数列 $\{a_n\}$ 对应的常生成函数为 $A(x)=\sum_{n\ge 0} a_nx^n$。

有两种物体，取 $i$ 个第一种物体的方案数为 $a_i$，取 $j$ 个第二种物体的方案数为 $b_j$，那么取 $k$ 个物体的方案数 $c_k=\sum_{i=0}^k a_ib_{k-i}$，这也称为这两个序列的**卷积**。

我们观察两个常生成函数的积，有：
$$
\left[x^k\right]A(x)B(x)=\sum_{i=0}^k a_ib_{k-i}
$$
于是 $A(x)B(x)$ 就是 $c_k$ 的常生成函数。

## 指数生成函数

> 一个数列 $\{a_n\}$ 对应的指数生成函数为 $A(x)=\sum_{n\ge 0} a_n\cfrac{x^n}{n!}$。

有两种物体，取 $i$ 个第一种物体的方案数为 $a_i$，取 $j$ 个第二种物体的方案数为 $b_j$，那么取 $k$ 个物体排成一列的方案数 $c_k=\sum_{i=0}^k\binom{k}{i}a_ib_{k-i}$。

我们观察两个指数生成函数的积，有：
$$
\begin{aligned}
\left[x^k\right]A(x)B(x)&=\sum_{i=0}^k \frac{a_i}{i!}\frac{b_{k-i}}{(k-i)!}\\
&=\sum_{i=0}^{k}\binom{k}{i}a_ib_{k-i}\frac{1}{k!}
\end{aligned}
$$
于是 $A(x)B(x)$ 就是 $c_k$ 的指数生成函数。

定义 $\exp(ax)\equiv \sum_{i=0}^{n-1}\cfrac{a^ix^i}{i!}\pmod{x^n}$，下面我们证明 $\exp(ax)\exp(bx)\equiv \exp((a+b)x)\pmod {x^n}$：
$$
\begin{aligned}
\exp(ax)\exp(bx)&\equiv \sum_{k=0}^{n-1}\left(\sum_{i=0}^k\frac{a^ib^{k-i}}{i!(k-i)!}\right) x^k\\
&\equiv \sum_{k=0}^{n-1}\left(\sum_{i=0}^k\binom{k}{i}a^ib^{k-i}\right) \frac{x^k}{k!}\\
&\equiv \sum_{k=0}^{n-1}\frac{(a+b)^kx^k}{k!}\pmod{x^n}
\end{aligned}
$$

## 逆元

> 对于 $A(x)$，满足 $A(x)B(x)\equiv 1\pmod{x^n}$ 的多项式称为 $A(x)$ 的逆元。

我们可以假设 $A(x)=\sum_{i=0}^{n-1} a_ix^i$，$B(x)=\sum_{i=0}^{n-1}b_ix^i$，然后相乘，可以得到：
$$
A(x)B(x)\equiv\sum_{k=0}^{n-1}\left(\sum_{i=0}^k a_ib_{k-i}\right) x^k\pmod{x^n}
$$
现在就可以列方程求解了：
$$
\begin{cases}
a_0b_0&=1\\
a_0b_1+a_1b_0&=0\\
\vdots\\
\sum_{i=0}^{n-1} a_ib_{n-1-i}&=0
\end{cases}
$$
所以 $b_0=a_0^{-1}$，并且递推能唯一地解出每一个 $b_i$ 的值，所以逆元是唯一的。

下面给出常见的逆元：

+ $A(x)=\sum_{i=0}^{n-1} x^i$ 和 $B(x)=1-x$；

  即 $a=\{1,1,1,\dots,1\}$ 与 $b=\{1,-1,0,\dots,0\}$，容易验证这满足条件。

+ $A(x)=\sum_{i=0}^{n-1} k^ix^i$ 和 $B(x)=1-kx$；

  即 $a=\{k^0,k^1,k^2\dots,k^{n-1}\}$ 与 $b=\{1,-k\}$，容易验证这满足条件。

+ $A(x)=\sum_{i=0}^{n-1}\binom{k-1+i}{k-1}x^i$ 和 $B(x)=(1-x)^k$；

  这里用了前面的结论 $(1-x)^{-1}\equiv \sum_{i=0}^{n-1}x_i\pmod{x^n}$，考虑：
  $$
  (1-x)^k \times ((1-x)^{-1})^k\equiv 1\pmod{x^n}
  $$
  于是就有：
  $$
  \begin{aligned}
  ((1-x)^k)^{-1}&\equiv \left(\sum_{i=0}^{n-1} x^i\right)^k\pmod{x^n}\\
  &\equiv\sum_{j=0}^{n-1} c_j x^j\pmod{x^n}
  \end{aligned}
  $$
  考虑每一个 $c_j$ 的意义，是在 $k$ 个非负数 $i_1,i_2,\dots,i_k$ 满足 $i_1+i_2+\cdots+i_k=j$ 的解的个数，那么显然有 $c_j=\binom{j+k-1}{k-1}$。

## 卡特兰数

> $n+1$ 个数相乘 $x_0x_1\dots x_n$，求它们结合顺序的方案数。如 $n=3$ 时，$x_0(x_1(x_2x_3)),x_0((x_1x_2)x_3),(x_0x_1)(x_2x_3),((x_0x_1)x_2)x_3,(x_0(x_1x_2))x_3$。

首先，$c_0=1,c_1=1$。

对于 $c_n$，考虑最后一步运算，一定是左右两个组内都乘完之后进行的，那么我们可以枚举这两个组的分界线的位置，就得到了公式 $c_n=c_0c_{n-1}+c_1c_{n-2}+\cdots+c_{n-1}c_0$。

考虑这个序列的常生成函数 $F(x)\equiv \sum_{i=0}^{n-1}c_ix^{i}\pmod{x^n}$，我们能得到什么信息？

根据这个递推式，不难看出这是一个卷积的形式，那么我们给 $F(x)$ 平方一下，可以推出：
$$
\begin{aligned}
F^2(x)&\equiv \sum_{k=0}^{n-1}\left(\sum_{i=0}^k c_ic_{k-i}\right)x^k\\
&\equiv\sum_{k=0}^{n-1} c_{k+1}x^k\pmod{x^n}
\end{aligned}
$$
我们给两边同乘一个 $x$，有：
$$
\begin{aligned}
xF^2(x)&\equiv \sum_{k=1}^{n-1} c_k x^k\\
&\equiv F(x)-1\pmod{x^n}
\end{aligned}
$$
所以我们需要解方程 $xF^2(x)-F(x)+1\equiv 0\pmod{x^n}$。

按照传统配方的思路，可以得到 $\left(xF(x)-\cfrac{1}{2}\right)^2\equiv -x+\cfrac{1}{4}\pmod{x^{n+1}}$。

对于所有函数 $G(x)$ 满足 $G^2(x)\equiv -x+\cfrac{1}{4}\pmod{x^{n+1}}$，我们有 $xF(x)\equiv G(x)+\cfrac{1}{2}\pmod{x^{n+1}}$。

设 $G(x)$ 对应的序列为 $\{g_0,g_1,\dots,g_n\}$，我们有：
$$
\begin{cases}
g_0^2&=\cfrac{1}{4}\\
g_0g_1+g_1g_0&=-1\\
\vdots\\
\sum_{i=0}^{n}g_ig_{n-i}&=0
\end{cases}
$$
我们希望 $g_0=-\cfrac{1}{2}$，否则 $xF(x)\equiv G(x)+\cfrac{1}{2}\pmod{x^{n+1}}$ 是无解的。

于是 $g_i$ 的每一个值都可以唯一地确定下来了，并且有 $F(x)$ 的序列 $\{f_0,f_1,\dots,f_{n-1}\}$ 满足 $f_i=g_{i+1}$。

根据广义二项式定理，有：
$$
\left(-x+\frac{1}{4}\right)^{\frac{1}{2}}\equiv \sum_{i=0}^n \frac{(\frac{1}{2})^{\underline{i}}}{i!} (-x)^i(\frac{1}{4})^{\frac{1}{2}-i}\pmod{x^{n+1}}
$$
本文不讨论为什么可以把高数的东西拿过来用，只证明这个式子是对的。
$$
\begin{aligned}
-x+\frac{1}{4}&\equiv \sum_{k=0}^n\left(\sum_{i=0}^k \binom{\frac{1}{2}}{i}\binom{\frac{1}{2}}{k-i}\right) (-1)^k(\frac{1}{4})^{1-k}x^k\\
&\equiv\sum_{k=0}^n\binom{1}{k} (-1)^k(\frac{1}{4})^{1-k}x^k\\
&\equiv \frac{1}{4}-x\pmod{x^{n+1}}
\end{aligned}
$$
这里用到了范德蒙德卷积，它对于上指标是实数的情况仍然是成立的。

于是我们就得到了 $g_i$ 的解：
$$
\begin{aligned}
g_i&=-\frac{(\frac{1}{2})^{\underline{i}}}{i!}(-1)^i(\frac{1}{4})^{\frac{1}{2}-i}\\
&=-\frac{1}{2} \frac{(\frac{1}{2})^{\underline{i}}}{i!} (-4)^i
\end{aligned}
$$
当 $i\ge 2$ 时，我们可以看到 $(\frac{1}{2})^{\underline{i}}=\frac{1}{2}(-\frac{1}{2})\cdots(\frac{1}{2}-i+1)$ 有 $i-1$ 个负项，加上式子最前面的符号，可以把 $(-4)^i$ 的负号全部抵消掉，并且有：
$$
\begin{aligned}
g_i&=\frac{(\frac{1}{2})^{\overline{i-1}}4^{i-1}}{i!}\\
&=\frac{(1\times 3\times \cdots \times (2i-3)) 2^{i-1}}{i!}\\
&=\frac{(1\times 3\times \cdots \times (2i-3))(1\times 2\times\cdots\times (i-1)) 2^{i-1}}{i!(i-1)!}\\
&=\frac{(1\times 3\times \cdots \times (2i-3))(2\times 4\times\cdots\times (2i-2))}{i!(i-1)!}\\
&=\frac{1}{i}\frac{(2i-2)!}{(i-1)!(i-1)!}\\
&=\frac{1}{i}\binom{2i-2}{i-1}
\end{aligned}
$$
于是 $f_i=g_{i+1}=\frac{1}{i+1}\binom{2i}{i}$，可以验证 $i=0$ 时是成立的。

## 说明

虽然上面我们都是在模 $x^n$ 的意义下讨论的，实际上做题的时候可以认为 $n$ 足够大，把生成函数写成 $F(x)=\sum_{i\ge 0} f_i x^i$ 的形式即可。

## [CF451E] Devu and Flowers

对于每一种花，它的常生成函数是 $F_i(x)=(1-x^{f_i+1})(1-x)^{-1}$，于是答案的生成函数 $F(x)=(1-x)^{-n}\prod_{i=1}^n (1-x^{f_i+1})$。

设 $A(x)=(1-x)^{-n}$，$B(x)=\prod_{i=1}^n(1-x^{f_i+1})$，由于 $n\le 20$，所以可以暴力算出 $B(x)$，二进制枚举每一个括号给出的贡献即可，这最多会生成 $2^n$ 项，计算的复杂度是 $O(n2^n)$。

我们又知道 $A(x)$ 的每一项是 $\binom{n-1+i}{n-1}$，所以枚举 $B(x)$ 的每一项，$O(n)$ 暴力计算组合数即可，复杂度 $O(n2^n)$。

[AC Code](https://gist.github.com/hikariyo/685bcfa6d16848155897fd438fb14770)。

## [CEOI2004] Sweets

这和上一个题目几乎一样，只不过对于答案的生成函数 $F(x)$，需要求 $\sum_{i=a}^b[x^i]F(x)$。

考虑 $G(x)=\sum_{i\ge 0} g_i x^i$ 其中 $g_i=\sum_{j=0}^i f_j$。

令 $h=\{1,1,1,\dots\}$ 就 $G(x)=F(x)H(x)$，所以我们给 $F(x)$ 再乘一个 $(1-x)^{-1}$ 即可。

这里有一个别的问题，我们怎么求 $\binom{n+i}{n}\bmod 2004$？

写一下它的式子：
$$
\binom{n+i}{n}=\frac{(n+i)^{\underline{n}}}{n!}
$$
由于组合数一定是整数，所以 $n!\mid (n+1)^{\underline{n}}$，先算 $r=(n+i)^{\underline{n}}\bmod (2004n!)$，我们有：
$$
(n+i)^{\underline{n}}=q(2004n!)+r
$$
其中 $n!\mid r$，于是：
$$
\frac{(n+i)^{\underline{n}}}{n!}=q(2004!)+\frac{r}{n!}
$$
[AC Code](https://gist.github.com/hikariyo/aa42683ed1013a740dbe264e02b4c0e0)。

## [POJ3734] Blocks

对于 $a=\{1,0,1,0,\dots\}$ 的指数生成函数，为 $F_1(x)=1+\cfrac{x^2}{2!}+\cfrac{x^4}{4!}+\cdots=\cfrac{\exp(x)+\exp(-x)}{2}$；对于奇偶都行的为 $F_2(x)=\exp(x)$，所以答案的指数生成函数就是：
$$
\begin{aligned}
F(x)&=\left(F_1(x)F_2(x)\right)^2\\
&=\left(\frac{\exp(x)+\exp(-x)}{2} \exp(x)\right)^2\\
&=\frac{\exp(4x)+2\exp(2x)+1}{4}
\end{aligned}
$$
由于我们要求 $n![x^n]F(x)$，其中 $n\ge 1$，所以可以不用管那个常数项。

继续计算：
$$
\begin{aligned}
n![x^n]F(x)&=\frac{n!}{4}\left(\frac{4^n}{n!}+2\frac{2^n}{n!}\right)\\
&=\frac{4^{n}+2^{n+1}}{4}
\end{aligned}
$$
[AC Code](https://gist.github.com/hikariyo/ada5be7f2e2b294f6ad651ac7793fa3d)。

## [CF891E] Lust

观察操作前后的序列，我们会发现，操作前是 $a_1a_2\dots a_{x-1}a_xa_{x+1}\dots a_n$，操作后是 $a_1a_2\dots a_x\dots a_n-a_1a_2\dots a_{x-1}a_{x+1}\dots a_n$，于是 $res$ 就加它们两个之差。

我们假设 $k$ 次操作后 $a_i\gets a_i-b_i$，其中 $b_i\ge 0$，那么就有期望 $E$ 为：
$$
\begin{aligned}
E&=\frac{\sum_{b_1+\cdots+b_n=k}\left(\prod_{i=1}^n a_i-\prod_{i=1}^n (a_i-b_i)\right)\binom{k}{b_1,b_2,\dots,b_n}}{n^k}\\
&=\frac{k!}{n^k}\left(\sum_{b_1+\cdots+b_n=k}\left(\prod_{i=1}^n\frac{a_i}{b_i!}\right)-\sum_{b_1+\cdots+b_n=k}\left(\prod_{i=1}^n\frac{a_i-b_i}{b_i!}\right)\right)
\end{aligned}
$$
不难看出 $\sum_{b_1+\dots+b_n=k}\prod_{i=1}^n \cfrac{*}{b_i!}$ 是一个卷积的结构，手玩一下可以推出左边是 $a_i\exp(x)$ 乘起来，右边是 $(a_i-x)\exp(x)$ 乘起来，这都是可以 $O(n^2)$计算的，最后取 $x^k$ 的系数。

形式化地，我们写作：
$$
\begin{aligned}
E&=\left[x^k\right]\frac{k!}{n^k}\left(\prod_{i=1}^n a_i\exp(x)+\prod_{i=1}^n(a_i-x)\exp(x)\right)\\
&=\left[x^k\right]\frac{k!}{n^k}\exp(nx)\left(\prod_{i=1}^n a_i+\prod_{i=1}^n(a_i-x)\right)\\
&=\left[x^k\right]\frac{k!}{n^k}\exp(nx)F(x)\\
&=\frac{k!}{n^k}\sum_{i=1}^n[x^i]F(x)[x^{k-i}]\exp(nx)\\
&=\frac{k!}{n^k}\sum_{i=1}^n[x^i]F(x)\frac{n^{k-i}}{(k-i)!}\\
&=\frac{1}{n^k}\sum_{i=1}^n[x^i]F(x) n^{k-i}\frac{k!}{(k-i)!}\\
\end{aligned}
$$
我们可以 $O(n)$ 算出 $\cfrac{k!}{(k-i)!}$，由于 $F(x)$ 的次数最高就是 $n$，于是我们的 $i$ 最大到 $n$ 即可，这样的复杂度是 $O(n^2)$ 的。

[AC Code](https://gist.github.com/hikariyo/60c3d391072cf528248cf76b5b40b9a2)。

