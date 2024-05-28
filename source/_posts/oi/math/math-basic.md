---
date: 2023-10-26 17:22:00
update: 2024-05-27 15:30:00
title: 数学 - 基础公式及证明
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
description: 基础数学公式记录，主要关于复杂度分析和一些数论基础。
---

## 简介

OI 的数学中需要用到很多数学知识，虽然一般来说记一下结论就足够了，但是这里我还是挑一些印象比较深刻的结论记录一下，方便日后复习。

本文主要是关于级数公式以及数论中的一些基本公式，像欧拉定理那样的知识点记录在单独的文章中方便我专题复习使用。

## 常用级数

平方和公式还是比较常用的，下面两个式子都可以数学归纳法证明出来，就不多说了。
$$
\begin{aligned}
\sum_{x=1}^n x&=\frac{n(n+1)}{2}\\
\sum_{x=1}^n x^2&=\frac{n(n+1)(2n+1)}{6}
\end{aligned}
$$
但是，除了数学归纳法这个 Bug 一样的存在，其实还可以通过构造的方式求一下平方和公式。
$$
\begin{aligned}
2^3-1^3&=3\times 1^2+3\times 1+1\\
3^3-2^3&=3\times 2^2+3\times 2+1\\
&\cdots\\
(n+1)^3-n^3&=3n^2+3n+1
\end{aligned}
$$
累加得到：
$$
\begin{aligned}
(n+1)^3-1&=3S_n +\frac{3n(n+1)}{2}+n\\
S_n&=\frac{n(n+1)(2n+1)}{6}
\end{aligned}
$$

## 等比级数

大家都知道等比数列的求和公式：
$$
S_n=\begin{cases}
na_1,q=1\\
\frac{a_1(1-q^n)}{1-q},q\ne 1
\end{cases}
$$
当 $|q|<1$ 时，等比级数是收敛的：
$$
\lim_{n\to +\infin} S_n=\lim_{n\to +\infin}\frac{a_1(1-q^n)}{1-q}=\frac{a_1}{1-q}
$$
易证当 $|q|\ge 1$ 时，等比级数是发散的，这里就不写了。

## 调和级数

首先大家都知道调和级数和对数是一个级别的，即：
$$
\sum_{x=1}^n\frac{1}{x}=O(\ln n)
$$
这个证明起来应该是很容易的，将调和级数看作一个个小矩形的面积和，可以得到：
$$
\int_1^{n+1} \frac{1}{x}\mathrm{d}x<\sum_{x=1}^n \frac{1}{x}<\int_{1}^n\frac{1}{x}\mathrm{d}x+1
$$
化简一下：
$$
\ln(n+1)-\ln n<\sum_{x=1}^n\frac{1}{x}-\ln n<1
$$
我们知道：
$$
\lim_{n\to+\infin} \ln(n+1)-\ln n=0
$$
所以就可以粗略的证明出来：
$$
0\lt \lim_{n\to +\infin} \sum_{x=1}^n\frac{1}{x} -\ln n< 1
$$
调和级数很多时候用于计算复杂度，所以还是很重要的。

## 素数调和级数

设 $p_i$ 为第 $i$ 大的素数，那么：
$$
\sum_{i=1}^n \frac{1}{p_i}=O(\ln(\ln n))
$$
根据唯一分解定理，可以得到，当 $n\to +\infin$ 时：
$$
\prod_{i=1}^n(1+\frac{1}{p_i}+\frac{1}{p_i^2}+\cdots)=\prod_{i=1}^n\frac{1}{1-p_i^{-1}}=\sum_{i=1}^n\frac{1}{i}
$$
左边里面是个等比级数，公比为 $1/p<1$，所以它是收敛的。

两边取对数，可以得到：
$$
\sum_{i=1}^n -\ln(1-\frac{1}{p_i})=\ln(\ln n)
$$

还可以继续化简，证明 $\ln(x+1)\sim x$：

$$
\lim_{x\to 0}\frac{\ln(x+1)}{x}=\lim_{x\to 0}\ln(x+1)^{\frac{1}{x}}=1
$$

所以：
$$
\sum_{i=1}^n \frac{1}{p_i}=O(\ln(\ln n))
$$
得证。

## 约数个数

把 $n$ 质因数分解，设它有 $t$ 个质因数，那么约数个数为：
$$
\begin{aligned}
n&=\prod_{i=1}^t p_i^{k_i}\\
d(n)&=\prod_{i=1}^t(k_i+1)
\end{aligned}
$$
根据乘法原理，每一个质因数的次数都可以在 $0\sim k_i$ 中选一个，满足 $d(n)\le \sqrt{3n}$，证明如下：

引理：函数 $f(x)=\frac{p^x}{(x+1)^2},x\in \N,p\in \N^*$ 在 $p=2$ 时在 $[2,+\infin)$ 单增，当 $p\ge 3$ 时在 $[1,+\infin)$ 单增，证明：
$$
f'(x)=\frac{p^x\ln p\cdot (x+1)^2-p^x\cdot 2(x+1)}{(x+1)^4}=\frac{p^x(x+1)(x\ln p+\ln p-2)}{(x+1)^4}\\
$$
所以 $(x+1)\ln p-2$ 这个增函数决定 $f'(x)$ 的符号，它有且只有一个变号零点 $x=\frac{2}{\ln p}-1$，下面讨论一下 $f(x)$ 的最小值，只讨论整数情况。

+ $p=2$ 时 $x\approx 1.89$ 且 $f(x)\ge f(2)=\frac{4}{9}$。
+ $p=3$ 时 $x\approx 0.82$，并且 $f(x)\ge f(1)=\frac{3}{4}$。
+ $p\ge 4$ 时 $f(x)\ge \min\{f(1),f(0)\}=\min\{ p/4,1\}=1$

因此可以构造出：
$$
\frac{3n}{d^2(n)}=3\frac{p_1^{k_1}}{(k_1+1)^2}\frac{p_2^{k_2}}{(k_2+1)^2}\cdots\frac{p_t^{k_t}}{(k_t+1)^2}\ge 3\frac{2^{k_1}}{(k_1+1)^2}\frac{3^{k_2}}{(k_2+1)^2}\ge 3\cdot\frac{4}{9}\cdot\frac{3}{4}=1
$$
则 $d(n)\le \sqrt{3n}$，这个上界是很宽松的，看作 $O(\sqrt n)$。

## 约数之和

把 $n$ 质因数分解，设它有 $t$ 个质因数，那么约数之和：

$$
\begin{aligned}
n&=\prod_{i=1}^t p_i^{k_i}\\
\sigma(n)&=\prod_{i=1}^t (\sum_{j=0}^{k_i} p_i^j)
\end{aligned}
$$
根据唯一分解定理，这些能组合成 $n$ 的所有约数。

## 二项式定理

分析复杂度的时候可能会用到，比如状压 DP 中枚举子集的子集复杂度是 $O(3^n)$
$$
(a+b)^n=\sum_{i=0}^n \binom{n}{i} a^ib^{n-i}
$$

同时也可以证明 $(a\bmod p)^k\equiv a^k \pmod p$：
$$
(a+p)^k\equiv \sum_{i=0}^k\binom{k}{i}a^ip^{k-i}\equiv a^k\pmod p
$$
因为只有 $i=k$ 时才满足 $p^{k-i}\not\equiv 0\pmod p$。

## GCD & LCM

辗转相减法就不说了，首先根据算数基本定理，设 $a=\prod p^{\alpha},b=\prod p^{\beta},c=\prod p^{\gamma}$，则有一个更为本质的式子：
$$
\begin{aligned}
\gcd(a,b)&=\prod p^{\min(\alpha,\beta)}\\
\text{lcm}(a,b)&=\prod p^{\max(\alpha,\beta)}
\end{aligned}
$$
这个可以用来证明一些式子，比如：
$$
\begin{aligned}
\gcd(a,b)\times \text{lcm}(a,b)&=\prod p^{\min(\alpha,\beta)+\max(\alpha,\beta)}\\
&=\prod p^{\alpha+\beta}\\
&=a\times b
\end{aligned}
$$
也可以证明结合律，即：
$$
\gcd(a,\gcd(b,c))=\prod p^{\min(\alpha,\min(\beta,\gamma))}=\prod p^{\min(\alpha,\beta,\gamma)}=\gcd(a,b,c)
$$
这也可以用于求 $n$ 个数的高精 $\text{lcm}$，不过考这个也挺没意思的。

## 数论分块(整除分块)

对于正整数 $n$，当 $k=\lfloor\frac{n}{x}\rfloor$ 时，$x$ 的最大值是 $\lfloor\frac{n}{k}\rfloor$。

下面求一下这个值的来源：
$$
k=\lfloor\frac{n}{x}\rfloor=\lfloor\frac{n}{x+d}\rfloor
$$
设成余数的形式：
$$
\begin{cases}
n=kx+r_1&r_1\in[0,x)\\
n=k(x+d)+r_2&r_2\in[0,x+d)
\end{cases}
$$
可以解出：
$$
\begin{aligned}
r_2&=r_1-kd\ge 0\\
d&\le \lfloor\frac{r_1}{k}\rfloor\\
&=\lfloor \frac{n-kx}{k} \rfloor\\
&=\lfloor \frac{n}{k} \rfloor -x
\end{aligned}
$$
当 $d$ 取到最大值时，有：
$$
\begin{cases}
r_2=n-k\lfloor\frac{n}{k}\rfloor=n\bmod k\\
r_1=r_2+kd=n-kx=n-x\lfloor \frac{n}{x}\rfloor=n\bmod x
\end{cases}
$$
有 $r_1$ 满足条件，$r_2\le r_1$ 肯定也满足条件。结论得证。

