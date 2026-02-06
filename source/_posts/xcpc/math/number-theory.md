---
date: 2026-02-06 16:02:00
updated: 2026-02-06 16:02:00
title: 数学 - 初等数论选讲
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 从最小数原理和数学归纳法出发的初等数论的最基本内容。
---

## 前言

为什么要写这篇文章？因为我逐渐发现，有些推导过程比较长的题目，如果对应当熟知的那些结论不够肯定，就会容易卡住。

这其实就是一个简单的概率问题，假设你对每一个结论都只有 $0.8$ 的把握它是可信的，那么当你连续用了 $10$ 个这样的结论的时候，你对自己的可信度就已经降到了 $0.8^{10}=0.107$。

所以我们能做的就是，确信自己使用的每一个公式、每一个定理都是完全正确的，做的每一步转化都是等价转化，没有漏掉任何一个边界，这样就能尽可能少地浪费时间。

本文不包括同余理论及后面内容，因为我主要把握不清的是前面的这些内容。

## 最小数原理

在开始叙述之前，为了更好地行文，我们先来说明一个符号约定：

1. 设 $\alpha(x)$ 是关于 $x\in E$ 的一个判断，那么用记号 $(\exists x\in E)(\alpha(x))$ 表示存在 $x\in E$ 使得 $\alpha(x)$ 成立；

   类似地，用 $(\forall x\in E)(\alpha(x))$ 表示对所有的 $x\in E$ 都有 $\alpha(x)$ 成立。

   后记：好像只有一开始使用了这个记号，因为之后的命题虽然内容多但是嵌套层数不是很多。

2. 本文的自然数集 $\mathbb N$ 不包括 $0$。

> 原理 $1$：设 $T$ 是 $\mathbb N$ 的一个**非空子集**，则 $(\exists t_0\in T)(\forall t\in T)(t_0\le t)$，即 $T$ 中存在一个最小的自然数。

本文为了简化行文，将其看作一个公理，不再向前追溯是否有比它更原始的公理了。

由此原理立刻可以推出下面的定理：

> 推论 $1$：设 $S$ 是 $\mathbb Z$ 的**非空子集**，如果 $(\exists m\in \mathbb Z)(\forall s\in S)(s\le m)$，那么 $(\exists s_0\in S)(\forall s\in S)(s\le s_0)$，即若 $S$ 有上界则有最大整数。

证：令 $T=\{m-s+1:\forall s\in S\}$，那么 $T$ 非空并且是 $\mathbb N$ 的子集。

根据最小数原理，$T$ 存在最小元素 $t_0=m-s_0+1$。

下面我们用反证法证明 $(\forall s\in S)(s\le s_0)$。

如果 $(\exists s_1\in S)(s_1>s_0)$，那么 $m-s_1+1<m-s_0+1$。

也就是说，存在 $t_1=m-s_1+1\in T$，使得 $t_1< t_0$，这与 $t_0$ 是 $T$ 的最小值矛盾。

因此  $(\forall s\in S)(s\le s_0)$，原命题得证。

> 推论 $2$：设 $S$ 是 $\mathbb Z$ 的非空子集，如果 $(\exists m\in \mathbb Z)(\forall s\in S)(s\ge m)$，那么 $(\exists s_0\in S)(\forall s\in S)(s\ge s_0)$，即若 $S$ 有下界则有最小整数。

证：同上。

## 数学归纳法

有两种数学归纳法，我们可以根据最小自然数原理给出它们的证明。

> 定理 $1$：设 $P(n)$ 是关于自然数 $n$ 的一个判断，如果：
>
> + 当 $n=1$ 时，$P(n)$ 成立；
> + 由 $P(n)$ 成立可以推出 $P(n+1)$ 成立，
>
> 那么 $P(n)$ 对所有自然数 $n$ 成立。

证：若不然，设 $T$ 是 $P(n)$ 不成立的 $n$ 构成的集合。

由 $T$ 非空可知，$T$ 中有最小值 $t_0$；由第一条要求可知，$t_0\neq 1$；由第二条要求可知，$P(t_0-1)$ 不成立。

那么 $t_0-1$ 应当也满足 $t_0-1\in T$，这与 $t_0$ 是 $T$ 中的最小值矛盾。

所以 $T$ 必然为空集，即不存在自然数 $n$ 使得 $P(n)$ 不成立。

> 定理 $2$：设 $P(n)$ 是关于自然数 $n$ 的一个判断，如果：
>
> 1. 当 $n=1$ 时，$P(n)$ 成立；
> 2. $n>1$，若“所有满足 $m<n$ 的自然数 $m$，都有 $P(m)$ 成立”可以推出 $P(n)$ 成立，
>
> 那么 $P(n)$ 对所有自然数 $n$ 成立。

证：同上。

## 整除与带余除法

> 定义 $1$：设 $a,b\in \mathbb Z$，其中 $b\neq 0$，如果 $(\exists q\in \mathbb Z)(a=bq)$，称 $b$ 整除 $a$，记作 $b\mid a$。并将 $b$ 叫做 $a$ 的因数，$a$ 叫做 $b$ 的倍数。

我们可以从这个定义出发，推出一些简单地定理。

> 定理 $1$（传递性）：若 $a,b,c\in \mathbb Z$，其中 $b,c\neq 0$，如果 $c\mid b,b\mid a$，那么 $c\mid a$。

证：存在 $q_1,q_2\in \mathbb Z$ 使得 $b=q_1c,a=q_2b$，那么 $a=q_1q_2c$，即存在 $q=q_1q_2\in\mathbb Z$ 使得 $a=qc$。

> 定理 $2$：若 $a,b,c\in \mathbb Z$，其中 $a\neq 0$，如果 $a\mid b$ 且 $a\mid c$，那么 $(\forall x,y\in \mathbb Z)(a\mid bx+cy)$。

证：存在 $q_1,q_2\in \mathbb Z$ 使得 $b=q_1a,c=q_2a$，那么 $\forall x,y\in \mathbb Z$，有 $bx+cy=(xq_1+yq_2)a$。

> 定理 $3$：若 $a,b\in \mathbb Z$，其中 $a\neq 0$，那么 $a\mid b$ 与 $a\mid |b|,|a|\mid b,|a|\mid |b|$ 等价。

证：分类讨论 $a,b$ 符号即可。

> 定理 $4$：若 $a,b\in \mathbb Z$，其中 $b>0$，那么存在唯一的 $q,r$ 使得 $a=bq+r$，其中 $q\in \mathbb Z,r\in \mathbb Z\cap [0,b)$。

证：存在性：设集合 $R=\{a-bq:\forall q\in \mathbb Z,a-bq\ge 0\}$，易知 $R$ 非空。

由最小自然数原理的推论可知，$R$ 中存在最小元素 $r_0$，设 $r_0=a-bq_0$。

若 $r_0\ge b$，那么 $r_0-b=a-b(q_0+1)\ge 0$ 且满足 $a-bq$ 的形式。

于是 $r_0-b<r_0$ 且 $r_0-b\in R$，这与 $r_0$ 的最小性矛盾。

所以存在 $q_0\in\mathbb Z,r_0\in \mathbb Z\cap [0,b)$ 使得 $a=bq_0+r_0$。

唯一性：设 $r_0,q_0$ 和 $r_1,q_1$ 都满足条件，那么 $bq_0+r_0=bq_1+r_1$。

移项，有 $r_0-r_1=b(q_1-q_0)$。

因为 $r_0-r_1\in (-b,b)\cap \mathbb Z$，其中 $b$ 的倍数只有 $0$，所以 $r_0-r_1=0$ 且 $q_1-q_0=0$。

## 最大公因数

> 定义 $1$：设 $a,b$ 是不全为 $0$ 的整数，用 $\mathcal D(a,b)$ 表示 $a,b$ 的公因数组成的集合。

我们后面可以通过证明两对整数的 $\mathcal D$ 集合相等来证明它们的公因数相等。

> 引理 $1$：设 $a,b$ 是不全为 $0$ 的整数，$\mathcal D(a,b) = \mathcal D(|a|, |b|)$。

证：设 $d\mid a, d\mid b$ 由上一节的定理 $3$ 可知，$d\mid a$ 且 $d\mid b$ 等价于 $d\mid |a|$ 且 $d\mid |b|$，于是这两个集合相等。

我们今后不妨只讨论非负整数的公因数。

> 引理 $2$：设 $a,b$ 是不全为 $0$ 的非负整数，则 $\mathcal D(a,b)$ 中存在最大值。

证：由 $1\mid a$ 且 $1\mid b$ 知 $\mathcal D(a,b)$ 非空。

不妨设 $a>0$，任取 $d\in\mathcal D(a,b)$，有 $|d|\mid a$，那么存在 $q\in \mathbb Z$ 使得  $a=q|d|$。

显然 $q\neq 0$，那么 $q\ge 1$，于是 $a=q|d|\ge |d|$。

所以 $d\le |d|\le a$，即 $\mathcal D(a,b)$ 有上界，由最小自然数原理的推论可知 $\mathcal D(a,b)$ 中存在最大值。

> 定义 $2$：设 $a,b$ 是不全为 $0$ 的整数，记 $(a,b)=\max \mathcal D(a,b)$。

这个定义和顺序无关，所以根据定义就有 $(a,b)=(b,a)$，下面我们来证明一些关于最大公因数的定理。

> 定理 $1$：设 $b$ 是正整数，那么 $(0,b)=b$。

证：由引理 $2$ 知，任取 $d\in \mathcal D (0,b)$，都有 $d\le b$，那么 $b\mid 0,b\mid b$，可知 $b\in \mathcal D(0,b)$，于是 $(0,b)=b$。

> 定理 $2$：设 $a,b,q\in \mathbb Z$ 且 $b\neq 0$，那么 $(a,b)=(a+qb,b)$

证：任取 $d\in\mathcal D(a,b)$，由上一节定理 $2$ 可知 $d\in \mathcal D(a+qb,b)$，反之也成立，则 $\mathcal D(a,b)=\mathcal D(a+qb,b)$，所以 $(a,b)=(a+qb,b)$。

> 定理 $3$（裴蜀定理）：设 $a,b$ 是不全为 $0$ 的整数，存在一组 $x,y\in \mathbb Z$ 使得 $xa+yb=(a,b)$。

证：根据引理 $1$，不妨设 $a\ge b\ge 0$，那么 $a\neq 0$。

如果 $b=0$，那么由定理 $1$ 知 $(a,b)=1a+0b$，满足条件；

如果 $b\neq 0$，那么用 $b$ 除 $a$ 得到 $a=qb+r$，由定理 $2$ 知 $(a,b)=(r,b)=(b,r)$，其中 $b>r$。

令 $a_0=a,b_0=b$ 并且 $a_1=b,b_1=r$，重复上述过程，每一次 $a_i,b_i$ 中的较小值都会严格减小，一定在有限次内满足 $b_i=0$。

下面我们来用递推的方式构造出一组 $x,y$，假设已知 $(a_n,b_n)=x_na_n+y_nb_n$。

由我们的操作方式可知，$a_n=b_{n-1}, b_n=r_{n-1}$，其中 $a_{n-1}=q_{n-1}b_{n-1}+r_{n-1}$，那么：
$$
\begin{aligned}
(a_{n-1},b_{n-1})&=(a_n,b_n)\\
&=x_na_n+y_nb_n\\
&=x_nb_{n-1}+y_nr_{n-1}\\
&=x_nb_{n-1}+y_n(a_{n-1}-q_{n-1}b_{n-1})\\
&=y_na_{n-1}+(x_n-y_nq_{n-1})b_{n-1}\\
\end{aligned}
$$
所以令 $x_{n-1}=y_n,y_{n-1}=x_n-y_nq_{n-1}$ 即可，这样重复下去一定能得到第一组 $x_0,y_0$ 满足 $(a,b)=x_0a+y_0b$。

> 推论 $1$：设 $a,b$ 是不全为 $0$ 的整数，$d$ 是非零整数，则 $d\mid a$ 且 $d\mid b$ 等价于 $d\mid (a,b)$。

证：根据裴蜀定理，存在一组 $x,y$ 使得 $xa+yb=(a,b)$。

那么 $d\mid a$ 且 $d\mid b$ 能够推出 $d\mid (xa+yb)$，即 $d\mid (a,b)$；反之，由最大公因数的定义，$(a,b)\mid a$ 且 $(a,b)\mid b$，由整除的传递性可知 $d\mid (a,b)$ 则 $d\mid a$ 且 $d\mid b$。

> 推论 $2$：设 $a$ 是非零整数，$b$ 是整数，若 $(a,b)=1$，则 $a\mid bc$ 等价于 $a\mid c$。

证：若 $a\mid c$ 显然有 $a\mid bc$，反之，若 $a\mid bc$，

根据裴蜀定理，存在一组 $x,y$ 使得 $xa+yb=1$，那么 $c=c(xa+yb)=a(xc)+(bc)y$。

那么由 $a\mid a(xc)$ 和 $a\mid (bc)y$ 就可以推出 $a\mid c$。

> 定理 $4$：设 $a,b$ 是不全为 $0$ 的整数，$m$ 是正整数，则 $(ma,mb)=m(a,b)$。

证：若 $a,b$ 存在一个 $0$ 显然成立，根据引理 $1$，下面不妨设 $a,b$ 都是正整数。

1. 证 $(ma,mb)\mid m(a,b)$。

   根据公因数的定义，我们有 $(ma,mb)\mid ma$ 且 $(ma,mb)\mid mb$。

   又因为 $m\mid ma$，$m\mid mb$，根据推论 $1$，这等价于 $m\mid (ma,mb)$。

   于是根据整除的定义，有 $\cfrac{(ma,mb)}{m}\mid a$ 且 $\cfrac{(ma,mb)}{m}\mid b$。

   根据推论 $1$，这等价于 $\cfrac{(ma,mb)}{m}\mid (a,b)$。

   再根据整除的定义，有 $(ma,mb)\mid m(a,b)$。

2. 证 $m(a,b)\mid (ma,mb)$。

   根据公因数的定义，我们有 $(a,b)\mid a$ 且 $(a,b)\mid b$。

   根据整除的定义，我们有 $m(a,b)\mid ma$ 且 $m(a,b)\mid mb$。

   根据推论 $1$，有 $m(a,b)\mid (ma,mb)$。

又因为 $m(a,b)$ 和 $(ma,mb)$ 均是正数，它们相互整除，根据整除的定义容易得到它们相等。

> 推论 $3$：设 $a,b$ 是不全为 $0$ 的整数，$\left(\cfrac{a}{(a,b)},\cfrac{b}{(a,b)}\right)=1$。

证：设 $(a,b)=\left((a,b)\cfrac{a}{(a,b)},(a,b)\cfrac{b}{(a,b)}\right)=(a,b)\left(\cfrac{a}{(a,b)},\cfrac{b}{(a,b)}\right)$，又因为 $(a,b)\ge 1$，两边约掉 $(a,b)$ 即可。

> 推论 $4$：设 $a$ 是非零整数，$b$ 是整数，则 $a\mid bc$ 等价于 $\cfrac{a}{(a,b)}\mid c$。

证：由定义可以得到 $a\mid bc$ 等价于 $\cfrac{a}{(a,b)}\mid \cfrac{b}{(a,b)}c$，由推论 $2$ 知这等价于 $\cfrac{a}{(a,b)}\mid c$。

## 素数与算术基本定理

> 定义 $1$：设 $p\ge 2,p\in \mathbb N$，若 $p$ 的正因数只有 $1$ 和 $p$，称 $p$ 为素数，否则称为合数。

注意，进入这一章节，我们大部分的内容只关注正整数。

> 定理 $1$：若 $a\ge 2, a\in \mathbb{N}$，则 $a$ 除了 $1$ 以外的最小正因数是素数。

证：设这个数是 $d$，若 $d$ 是合数，那么 $d$ 存在一个更小的非 $1$ 正因数 $e$ 满足 $e\mid d$，又因为 $d\mid a$，则 $e\mid a$，其中 $1<e<d$，这与 $d$ 的最小性矛盾。

> 定理 $2$：设 $p$ 为素数，$a\in \mathbb N$，则要么 $(a,p)=1$，要么 $p\mid a$。

证：由最大公因数的定义，$(a,p)\mid p$。

由素数的定义， $(a,p)$ 只能为 $1$ 或者 $p$。

若 $(a,p)=p$，根据公因数的定义，可以推出 $p\mid a$；若 $p\mid a$，又因为 $p\mid p$，所以 $p\in \mathcal D(a,p)$。又因为 $(a,p)$ 只能取 $1$ 或 $p$，所以 $(a,p)=p$。

若 $(a,p)=1$，则 $p\not \mid a$（若不然，可以推出 $(a,p)=p$）；若 $p\not \mid a$，则 $(a,p)=1$（若不然，可以推出 $p\mid a$）。

> 推论 $1$：设 $p$ 为素数，$a\in\mathbb N$，若 $p\mid ab$ 则 $p\mid a$ 或 $p\mid b$。

证：我们只需要证明这两个命题不同时为假即可。

若 $p\not \mid a$，则 $(a,p)=1$，由上一节的推论 $2$ 可知 $p\mid ab$ 可以推出 $p\mid b$。

> 定理 $3$（算术基本定理）：设 $n\ge 2,n\in \mathbb N$，则 $n$ 存在唯一素因子分解。

证：存在性，这里使用第二种数学归纳法。

当 $n=2$ 时，$n=2^1$ 是一个素因子分解式。

当 $n>2$ 时，由定理 $1$ 可知 $n$ 存在一个最小素因数 $p$。如果 $n=p$，那么 $p$ 就是一个素因子分解；如果 $n\neq p$，那么若 $\cfrac{n}{p}<n$ 存在素因数分解 $p_1p_2\dots p_k$，可以推出 $n$ 能被分解成 $p\times p_1p_2\dots p_k$。

唯一性，这里是一个构造性的证明方法。

当 $n=2$ 时，$n=2^1$ 就是唯一的素因子分解。

当 $n>2$ 时，设 $n=p_1p_2\dots p_k=q_1q_2\dots q_m$，其中所有的 $p_i,q_j$ 都是素数。

那么 $n$ 由最小素因数 $p_i$，由 $p_i\mid q_1q_2\dots q_m$ 结合推论 $1$ 知存在一个 $q_j$ 使得 $p_i\mid q_j$。

又因为 $q_j$ 是素数，且 $p_i\neq 1$，那么 $p_i=q_j$。

两边除掉 $p_i$ 和 $q_j$，得到一个 $n'=n/p_i$，它也有这两种素因数分解式，按照上述流程一直处理下去可以得到每一个 $p_i$ 都有对应的 $q_j$，并且 $k=m$（若不然，可推出某些素数乘积等于 $1$）。

> 定义 $2$：设 $a\ge 2,a\in\mathbb N$，则根据算术基本定理有 $a=p_1^{k_1}p_2^{k_2}\dots p_{m}^{k_m}$，定义 $V_p(a)$：若存在 $p_i=p$，则 $V_p(a)=k_i$；否则 $V_p(a)=0$。特别地，对所有的 $p$，都有 $V_p(1)=0$。

简单地说 $V_p(a)$ 就是 $a$ 分解之后 $p$ 对应的那个指数，当它为 $0$ 时 $p^0=1$ 也是恰当的。

> 推论 $2$：设 $a,b\in \mathbb N$，则 $(a,b)=\prod_p p^{\min\{V_p(a),V_p(b)\}}$。

证：令 $g=\prod_p p^{\min\{V_p(a),V_p(b)\}}$，由整除的定义易知 $g\in \mathcal D(a,b)$。

取正因子 $d\in\mathcal D(a,b)$，则 $\forall p,V_p(\cfrac{a}{d})=V_p(a)-V_p(d)\ge 0$，同理 $\forall p,V_p(d)\le V_p(b)$，那么 $\forall p,V_p(d)\le \min\{ V_p(a),V_p(b)\}$。

因此 $\forall p,V_p(d)\le V_p(g)$，那么根据整除的定义易知 $d\mid g$，于是 $d\le g$，则 $g$ 是 $\mathcal D(a,b)$ 中的最大值，即 $g=(a,b)$。

