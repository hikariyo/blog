---
title: 学习 - 超几何分布的期望和方差
date: 2024-04-20 18:15:00
updated: 2024-04-20 18:15:00
katex: true
categories:
- Learn
description: 由于用自己没推导过的公式心里没有底，所以写篇博客记录一下推导超几何分布的期望和方差的过程。
tags:
- Math
- Probability
---

## 简介

若随机变量 $X$ 服从超几何分布，记作 $X\sim H(N,n,M)$，则有：
$$
P(X=k)=\frac{\binom{M}{k}\binom{N-M}{n-k}}{\binom{N}{n}},k=0,1,\dots,n
$$
为了避免讨论边界情况，这里假设 $\binom{n}{m}=0,m>n$，所以上面的 $k$ 直接取遍 $0\sim n$，这样不符要求的概率为 $0$，并不会对结论造成影响。在期望和方差公式之前，首先给出一些组合数的公式及证明。

1. 组合数与某个数相乘的公式，一个常见的 trick，可以将变量乘组合数转化为常量乘组合数：

$$
  x\binom{n}{x}=x\frac{n!}{x!(n-x)!}=n\frac{(n-1)!}{(x-1)!(n-x)!}=n\binom{n-1}{x-1}
$$

2. 范德蒙德卷积：
   $$
   \sum_{i=0}^k\binom{n}{i}\binom{m}{k-i}=\binom{n+m}{k}
   $$
   这可以用二项式定理来证明：
   $$
   \begin{aligned}
   \text{[}x^k](1+x)^{n}(1+x)^{m}&=\sum_{i=0}^k\binom{n}{i}\binom{m}{k-i}\\
   [x^k](1+x)^{n+m}&=\binom{n+m}{k}
   \end{aligned}
   $$

3. 期望的线性性质：

   对于随机变量 $X,Y$ 都有 $E(X+Y)=E(X)+E(Y)$，我的理解是本质上为全概率公式：
   $$
   \begin{aligned}
   E(X+Y)&=\sum_{x}\sum_{y}(x+y)P(X=x,Y=y)\\
   &=\sum_{x}x\sum_yP(X=x,Y=y)+\sum_{y}y\sum_xP(X=x,Y=y)\\
   &=\sum_{x}xP(X=x)+\sum_y yP(Y=y)\\
   &=E(X)+E(Y)
   \end{aligned}
   $$
   注意这个不要求 $X,Y$ 独立。

用这两个公式就可以开始推导了。

## 期望

如果记 $p=\frac{M}{N}$，那么其形式上和二项分布的期望相同，即 $E(X)=np$，下面是推导过程：
$$
\begin{aligned}
E(X)&=\sum_{x=0}^{n} xP(X=x)\\
&=\sum_{x=1}^nx\frac{\binom{M}{x}\binom{N-M}{n-x}}{\binom{N}{n}}\\
&=\frac{M}{\binom{N}{n}}\sum_{x=1}^n\binom{M-1}{x-1}\binom{N-M}{n-x}\\
&=\frac{M}{\binom{N}{n}}\binom{N-1}{n-1}\\
&=M\frac{n!(N-n)!}{N!}\frac{(N-1)!}{(n-1)!(N-n)!}\\
&=n\frac{M}{N}
\end{aligned}
$$

## 方差

这个形式相较于二项分布的 $D(X)=np(1-p)$ 就比较丑了，对于 $X\sim H(N,n,M)$ 来说，有 $D(X)=n\frac{M}{N}\frac{(N-n)(N-M)}{N(N-1)}$，这也能看出来推导过程肯定不是那么简单的。
$$
\begin{aligned}
D(X)&=E(X^2)-E^2(X)\\
&=E(X(X-1))+E(X)-E^2(X)\\
&=\sum_{x=2}^{n}x(x-1)\frac{\binom{M}{x}\binom{N-M}{n-x}}{\binom{N}{n}}+n\frac{M}{N}-n^2\frac{M^2}{N^2}\\
&=\frac{M(M-1)}{\binom{N}{n}}\sum_{x=2}^{n}\binom{M-2}{x-2}\binom{N-M}{n-x}+n\frac{M}{N}-n^2\frac{M^2}{N^2}\\
&=M(M-1)\frac{(N-2)!}{(n-2)!(N-n)!}\frac{n!(N-n)!}{N!}+n\frac{M}{N}-n^2\frac{M^2}{N^2}\\
&=M(M-1)\frac{n(n-1)}{N(N-1)}+n\frac{M}{N}-n^2\frac{M^2}{N^2}\\
&=n\frac{M}{N}[\frac{(M-1)(n-1)}{N-1}+1-\frac{nM}{N}]\\
&=n\frac{M}{N}\frac{N(M-1)(n-1)+N(N-1)-nM(N-1)}{N(N-1)}\\
&=n\frac{M}{N}\frac{(N-M)(N-n)}{N(N-1)}
\end{aligned}
$$
本质上是连用两次组合数和变量乘积的性质。
