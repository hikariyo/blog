---
date: 2026-10-29 12:14:00
updated: 2026-10-30 01:44:00
title: 数学 - 形式幂级数
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: FPS 计数与期望题目。
---

## 前言

形式幂级数的本质是一个无穷数列，然后我们对其进行定义了一系列操作进行**变换**。

我们一般不关心其是否收敛，但是有很多微积分中要求收敛的条件下才成立的公式在这里仍然成立，本文不关心它们为什么成立，只是拿来用。

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

## 逆元

> 对于 $A(x)$，满足 $A(x)B(x)=1$ 的多项式称为 $A(x)$ 的逆元。

我们可以假设 $A(x)=\sum_{i=0}^{n-1} a_ix^i$，$B(x)=\sum_{i=0}^{n-1}b_ix^i$，然后相乘，可以得到：
$$
A(x)B(x)=\sum_{k=0}^{n-1}\left(\sum_{i=0}^k a_ib_{k-i}\right) x^k
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

  考虑：
  $$
  \left(\frac{1}{x-1}\right)^k=\left(\sum_{i\ge 0} x^i\right)^k=\sum_{i\ge 0} c_ix^i
  $$
  考虑每一个 $c_j$ 的意义，是在 $k$ 个非负数 $i_1,i_2,\dots,i_k$ 满足 $i_1+i_2+\cdots+i_k=j$ 的解的个数，那么显然有 $c_j=\binom{j+k-1}{k-1}$。

## [CF451E] Devu and Flowers

对于每一种花，它的常生成函数是 $F_i(x)=(1-x^{f_i+1})(1-x)^{-1}$，于是答案的生成函数 $F(x)=(1-x)^{-n}\prod_{i=1}^n (1-x^{f_i+1})$。

设 $A(x)=(1-x)^{-n}$，$B(x)=\prod_{i=1}^n(1-x^{f_i+1})$，由于 $n\le 20$，所以可以暴力算出 $B(x)$，二进制枚举每一个括号给出的贡献即可，这最多会生成 $2^n$ 项，计算的复杂度是 $O(n2^n)$。

我们又知道 $A(x)$ 的每一项是 $\binom{n-1+i}{n-1}$，所以枚举 $B(x)$ 的每一项，$O(n)$ 暴力计算组合数即可，复杂度 $O(n2^n)$。

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
## [CF891E] Lust

观察操作前后的序列，我们会发现，操作前是 $a_1a_2\dots a_{x-1}a_xa_{x+1}\dots a_n$，操作后是 $a_1a_2\dots a_x\dots a_n-a_1a_2\dots a_{x-1}a_{x+1}\dots a_n$，于是 $res$ 就加它们两个之差。

我们假设 $k$ 次操作后 $a_i\gets a_i-b_i$，其中 $b_i\ge 0$，那么就有期望 $E$ 为：
$$
\begin{aligned}
E&=\frac{\sum_{b_1+\cdots+b_n=k}\left(\prod_{i=1}^n a_i-\prod_{i=1}^n (a_i-b_i)\right)\binom{k}{b_1,b_2,\dots,b_n}}{n^k}\\
&=\frac{k!}{n^k}\left(\sum_{b_1+\cdots+b_n=k}\left(\prod_{i=1}^n\frac{a_i}{b_i!}\right)-\sum_{b_1+\cdots+b_n=k}\left(\prod_{i=1}^n\frac{a_i-b_i}{b_i!}\right)\right)
\end{aligned}
$$
$\sum_{b_1+\dots+b_n=k}\prod_{i=1}^n \cfrac{*}{b_i!}$ 是一个卷积的结构，手玩一下可以推出左边是 $a_i\exp(x)$ 乘起来，右边是 $(a_i-x)\exp(x)$ 乘起来，这都是可以 $O(n^2)$计算的，最后取 $x^k$ 的系数。

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

## [P8594] 一个仇的复

只有 $1\times 2$ 的矩形可以竖着放，别的我们都认为是横着放的，包括 $1\times 1$。

既然如此，不妨枚举竖着放的 $1\times 2$ 的矩形的数量 $i$，以及它们划分出来的非空段的数量 $1\le j\le i+1$。

一个长度为 $l$ 的非空段如果要贡献 $p$ 个矩形，方案数等价于在 $2l-2$ 个空隙中选择 $p-2$ 个，因此生成函数为：
$$
\begin{aligned}
\sum_{p\ge 2} \binom{2l-2}{p-2}x^p&=x^2\sum_{i=0}^{2l-2}\binom{2l-2}{i} x^i\\
&= x^2(1+x)^{2l-2}
\end{aligned}
$$
那么所有非空段长度之和为 $n-i$，那么选 $k-i$ 个矩形的方案数就是：
$$
\left[x^{k-i}\right] x^{2j}(1+x)^{2(n-i)-2j}\binom{n-i-1}{j-1}\binom{i+1}{j}
$$
这里的组合数来自于两个地方。

1. 设非空段的长度为 $l_1,l_2,\dots,l_j$，那么 $\sum_{k=1}^j  l_k=n-i$ 的正整数解的数量是 $\binom{n-i-1}{j-1}$
2. 设非空段的缝隙（包括第一个前面和最后一个后面）中 $1\times 2$ 矩形的数量为 $x_0,x_1,\dots, x_j$，那么 $x_0,x_j\ge 0,x_1,x_2,\dots,x_{j-1}\ge 1$，并且 $\sum_{k=0}^j x_j=i$ 的解的数量是 $\binom{i+1}{j}$。

注意特判 $i=n$ 的情况，此时只能是 $n=k$ 时有 $1$ 种方案，否则有 $0$ 种。

接下来化简式子：
$$
[n=k]+\sum_{i=0}^{\min(n-1,k)} \sum_{j=1}^{i+1}\binom{i+1}{j}\binom{2(n-i)-2j}{k-i-2j}\binom{n-i-1}{j-1}
$$
复杂度 $O(k^2)$。

## [TJOI2015] 概率论

设 $E_i$ 表示节点数为 $i$ 的二叉树的子节点期望，初始化 $E_0=0,E_1=1$。

设 $f_i$ 表示节点数为 $i$ 的二叉树数量，初始化 $f_0=f_1=1$。

枚举其左儿子的节点数量，那么有递推式：
$$
E_i=\frac{1}{f_i}\sum_{j=0}^{i-1} f_{j}f_{i-1-j}(E_{j}+E_{i-1-j})
$$
至于 $f_i$，众所周知这是卡特兰数的递推式：
$$
f_i=\sum_{j=0}^{i-1} f_jf_{i-1-j}
$$
为了尽量简化式子，我们合并 $E_if_i=h_i$，那么有：
$$
h_i=\sum_{j=0}^{i-1} f_{i-1-j} h_j+f_{j}h_{i-1-j}=2\sum_{j=0}^{i-1} f_{i-1-j}h_j
$$
令 $f$ 的生成函数是 $F(x)$，$h$ 的生成函数是 $H(x)$，能够写出 $i\ge 2$ 的递推：
$$
\begin{aligned}
\left[x^i\right] H(x)&=2\sum_{j=0}^{i-1} \left[x^{i-1-j}\right]F(x)\left[x^j\right]H(x)\\
&=2\left[x^{i-1}\right] F(x)H(x)\\
&=2\left[x^i\right] xF(x)H(x)\\
\end{aligned}
$$
根据 $h_0=0,h_1=1$ 得到 $H(x)$ 的最低次数是 $1$，$F(x)$ 的最低次数是 $0$，所以能够写出：
$$
H(x)=2xF(x)H(x)+x
$$
根据卡特兰数的生成函数 $F(x)=\cfrac{1-\sqrt{1-4x}}{2x}$，代入化简得到 $H(x)=\cfrac{x}{\sqrt{1-4x}}$。

展开幂级数，得到通项公式：
$$
\begin{aligned}
H(x)&=x\sum_{i\ge 0} \binom{-\frac{1}{2}}{i} (-4x)^i\\
&=\sum_{i\ge0} \left(-\frac{1}{2}\right)^{\underline{i}} \frac{(-4)^i}{i!} x^{i+1}\\
&=\sum_{i\ge0} \frac{(2i-1)!!}{2^i} \frac{4^i}{i!} x^{i+1}
\end{aligned}
$$
到这里，我们需要用到化简卡特兰数通项时的一个技巧，是双阶乘与阶乘之间的关系：
$$
\frac{2n!}{(2n-1)!!}=2\cdot 4\cdots  2n=2^n n!
$$
继续化简：
$$
\begin{aligned}
H(x)&=\sum_{i\ge0} \frac{2i!}{2^i(i!2^i)} \frac{4^i}{i!} x^{i+1}\\
&=\sum_{i\ge 0} \frac{2i!}{i!i!} x^{i+1}
\end{aligned}
$$
于是我们得到 $h_n=\binom{2n-2}{n-1}$，又因为 $f_n=\frac{1}{n+1}\binom{2n}{n}$，那么得到：
$$
P=\frac{\binom{2n-2}{n-1}}{\frac{1}{n+1}\binom{2n}n{}}=\frac{n(n+1)}{2(2n-1)}
$$
复杂度 $O(1)$。

## [FJOI2016] 建筑师

设 $f_{j,i}$ 表示长度为 $i$ 的排列中，从最左边开始能看到 $j$ 个建筑的方案数。

显然我们的递推过程是类似笛卡尔树的操作，枚举一个最大值，那么最大值左边就是一个独立的子问题，右边随便放。

于是可以写出转移：
$$
\begin{aligned}
f_{j,i}&=\sum_{k=1}^i \binom{i-1}{k-1}f_{j-1,k-1} (i-k)!\\
&=\sum_{k=1}^i (i-1)!\frac{f_{j-1,k-1}}{(k-1)!}
\end{aligned}
$$
定义 $g_{j,i}=f_{j,i}/i!$，可以得到：
$$
\begin{aligned}
g_{j,i}&=\frac{1}{i}\sum_{k=1}^i g_{j-1,k-1}\\
&=\frac{1}{i}\sum_{k=0}^{i-1} g_{j-1,k}\\
\end{aligned}
$$
设 $g_j$ 的生成函数为 $G_j(x)$，那么可以得到转移：
$$
\begin{aligned}
\left[x^i\right]G_j(x)&=\frac{1}{i}[x^{i-1}]\frac{G_{j-1}(x)}{1-x}\\
&=\left[x^i\right]\int \frac{G_{j-1}(x)}{1-x}\mathrm{d} x
\end{aligned}
$$
我们看一下 $j=0,1$ 时数列是什么样：当 $j=0$ 时，只有 $f_{0,0}=1$；当 $j=1$ 时，$f_{1,i}=(i-1)!$，那么 $g_{1,i}=1/i$。

于是可以看出 $G_1(x)=-\ln(1-x)$，根据递推式不难看出 $G_j(x)=\cfrac{(-\ln(1-x))^j}{j!}$。

此时来看我们要回答的询问是什么，依然枚举最大值的位置：
$$
\begin{aligned}
\sum_{i=1}^n \binom{n-1}{i-1}f_{A-1,i-1}f_{B-1, n-i}&=\sum_{i=1}^n\binom{n-1}{i-1} (i-1)!(n-i)!g_{A-1,i-1}g_{B-1,n-i}\\
&=(n-1)!\sum_{i=1}^n g_{A-1,i-1}g_{B-1,n-i}\\
&=(n-1)![x^{n-1}]G_{A-1}(x)G_{B-1}(x)\\
&=(n-1)!\frac{(A+B-2)!}{(A-1)!(B-1)!}[x^{n-1}]G_{A+B-2}(x)\\
\end{aligned}
$$
但是我们不需要真的用 $G_j(x)$ 的公式求其通项，暴力 $O(n\max(A+B))$ 预处理即可，每次询问 $O(1)$ 回答。

## [JLOI2015] 骗我呢

每一行的差分至多有一个 $2$，其余位置都是 $1$，不妨将没有 $2$ 视作 $2$ 的位置在 $m+1$。

于是每行 $2$ 的位置 $p_i$ 需要满足 $p_i\ge p_{i-1}-1$。

于是可以很快写出一个 $O(nm)$ 的做法：设 $f_{i,j}$ 表示第 $i$ 行 $p_i=j$ 的方案数，那么转移如下：
$$
f_{i,j}=\begin{cases}
\sum_{k=1}^{j+1}f_{i-1,k}& j\le m\\
\sum_{k=1}^{m+1}f_{i-1,k}&j=m+1
\end{cases}
$$
对于第一行，有 $f_{1,1}\sim f_{1,m+1}=1$。

如果对每行建立生成函数，转移存在前后截断的问题，难以写出一个统一的表达式，所以我们应该按列建立生成函数。

定义 $F_j(x)=\sum_{i\ge 1} f_{i,j}x^{i-1}$，那么对 $i=2\sim m$ 有：
$$
F_i-F_{i-1}=xF_{i+1}
$$
同时，有 $F_m=F_{m+1}$，所以我们可以得到递推：
$$
F_{i-1}=F_i-xF_{i+1}
$$
设 $F_i=D_iF_m$，那么有 $D_m=D_{m+1}=1$，并且：
$$
D_{i}=D_{i+1}-xD_{i+2}
$$
求出 $D_1$ 后，代入 $F_1=x(F_1+F_2)+1$ 就有：
$$
F_m=\frac{1}{D_1-xD_1-xD_2}
$$
为了习惯我们从小到大递推的习惯，代换 $G_i=D_{m+1-i}$，我们的目标式就变成了：
$$
F_m=\frac{1}{G_m-xG_m-xG_{m-1}}=\frac{1}{G_{m+2}}
$$
那么我们的问题就变成了，已知 $G_0=G_1=1$，并且 $G_i=G_{i-1}-xG_{i-2}$，求 $[x^{n+1}]F_m$。

待定系数法解决这个递推式：
$$
G_i-\lambda G_{i-1}=(1-\lambda) \left(G_{i-1}-\frac{x}{1-\lambda}G_{i-2}\right)
$$
定义 $H_i=G_i/\lambda ^i$，则有：
$$
H_i-H_{i-1}=\frac{1-\lambda}{\lambda} (H_{i-1}-H_{i-2})
$$
并且我们一开始就可以解出 $x=\lambda(1-\lambda)$，不过我们可以先关注 $H_i$ 的式子。

由于 $H_0=1,H_1=1/\lambda$，设 $t=(1-\lambda)/\lambda$，可以推出 $H_i-H_{i-1}=t^i$，于是 $H_i=1+t+t^2+\cdots+t^i$。

所以我们可以解出 $G_i$：
$$
\begin{aligned}
G_i&=\lambda^i H_i\\
&=\left(\frac{1}{t+1}\right)^i \frac{1-t^{i+1}}{1-t}
\end{aligned}
$$
因此我们现在得到了 $F_m$ 的表达式：
$$
F_m=\frac{1}{G_{m+2}}=\frac{(1-t)(1+t)^{m+2}}{1-t^{m+3}}
$$
问题是这是关于 $t$ 的式子，我们需要拉格朗日反演求出 $[x^{n}]F_m=f_{n+1,m}$。

具体地说，由于 $t=(1+t)^2x$，那么设 $H(t)=F_m$，有：
$$
[x^{n}]H(t)=\frac{1}{n}[t^{n-1}]H'(t)(1+t)^{2n}
$$
注意，我们这里不需要真的对分式函数 $H$ 进行求导，因为我们只需要提取它的系数，所以利用 $(uv)'=u'v+uv'$ 公式可以得到：
$$
\begin{aligned}
[x^{n}]H(t)&=\frac{1}{n}[t^{n-1}]\left(H(t)(1+t)^{2n}\right)'-\frac{1}{n}[t^{n-1}]H(t)2n(1+t)^{2n-1}\\
&=[t^{n}]H(t)(1+t)^{2n}-2[t^{n}]tH(t)(1+t)^{2n-1}\\
&=[t^{n}]H(t)(1+t)^{2n-1}(1-t)\\
&=[t^{n}]\frac{(1-t)^2(1+t)^{m+2n+1}}{1-t^{m+3}}
\end{aligned}
$$
因此，答案就是：
$$
\sum_{k\ge 0}\binom{m+2n+1}{n-k(m+3)}-2\binom{m+2n+1}{n-1-k(m+3)}+\binom{m+2n+1}{n-2-k(m+3)}
$$
非法组合数取 $0$，复杂度 $O(n+m)$。