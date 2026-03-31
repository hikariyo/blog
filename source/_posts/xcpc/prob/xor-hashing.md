---
date: 2026-03-31 18:49:00
updated: 2026-03-31 18:49:00
title: 概率 - 异或哈希
katex: true
tags:
- Algo
- C++
- Prob
categories:
- XCPC
description: 为什么分类是概率？因为本文主要讨论异或哈希算法的正确性，而这个正确性就是基于概率的。
---

## 异或哈希

注意下面的运算都是 $\mathbb Z_2$ 及其组成的向量的运算，并非通常意义下的运算。

抽象地讲，异或哈希是一种映射 $f:\mathbb Z_2^N \to \mathbb Z_2^{64}$，这个映射能够保持向量加法（即异或）运算，即 $f(x+y)=f(x)+f(y),\forall x,y\in \mathbb Z_2^N$，其中 $N$ 是一个比较大的数字。

如何做到这一点？我们给 $1\sim N$ 每一个位置分配一个权值 $w_i$，保证其在向量空间 $\mathbb Z_2^{64}$ 内均匀分布且两两独立。对于任意的 $x\in \mathbb Z_2^N$，设 $x_i\in \mathbb Z_2$ 表示 $x$ 的第 $i$ 个分量的值，那么 $f:x\mapsto \sum_{i=1}^N x_iw_i$，这里的加法是向量加法（即异或）运算，乘法是数量乘法。

显然这个映射能够保持运算，那么我们的问题就是，它的碰撞概率是不是足够的小？

## 碰撞概率

在说明异或哈希碰撞率低的时候，有很多人会用生日悖论来解释。我不太认同这一观点，因为在异或哈希这个语境下，不同的向量 $x,y$ 所映到的值并非完全独立，但是生日悖论那个公式是要求每一个元素映到的值都是两两独立的。

那么应该怎么说明这个碰撞概率比较低呢？一般来说，在一个需要使用异或哈希的题目中，通常会有两种情况：

### Case 1

对于数量级为 $10^6$ 的 $K$ 个向量，需要证明它们两两碰撞的概率较低。

任取 $X,Y\in \mathbb Z_2^N,X\neq Y$，那么存在一个位置 $i$ 使得 $x_i=1,y_i=0$；如果 $f(X)=f(Y)$，那么 $f(X+Y)=0$，那么：
$$
\begin{aligned}
P(f(X)=f(Y))&=P\left(\sum_{j=1}^N (x_j+y_j)w_j=0\right)\\
&=\sum_{v\in \mathbb Z_2^{64}} P\left(w_i=v,\sum_{j\neq i}(x_j+y_j)w_j=v\right)\\
&=\sum_{v\in \mathbb Z_2^{64}} P(w_i=v)P\left(\sum_{j\neq i} (x_j+y_j)w_j=v\right)\\
&=\frac{1}{2^{64}} \sum_{v\in \mathbb Z_2^{64}} P\left(\sum_{j\neq i} (x_j+y_j)w_j=v\right)\\
&=\frac{1}{2^{64}}
\end{aligned}
$$
根据概率论中的不等式，$P(A\cup B)\le P(A)+P(B)$，我们可以对 $K^2$ 对向量发生碰撞这一事件的概率算出一个上界 $\cfrac{K^2}{2^{64}}$，带入可以算出这个上界的数量级为 $10^{-8}$。

### Case 2

对于数量级为 $10^{10}$ 的 $K$ 个向量，有一个固定向量 $A$，需要证明 $X\neq A$ 时 $f(X)=f(A)$ 的概率较低。

这里的推导过程完全和 Case 1 一样，我们最终可以算出这个事件发生的概率的一个上界是 $\cfrac{K}{2^{64}}$，数量级同样是 $10^{-8}$。

但此时我们就不能说这些向量两两碰撞的概率很低了。

## 推广

对于素数进制不进位加法 $Z_p^N$，上面的推导仍然成立。具体地说，任取 $X,Y\in\mathbb Z_p^N,X\neq Y$，那么存在一个位置 $i$ 使得 $x_i-y_i\not\equiv 0\pmod p$，于是：
$$
\begin{aligned}
P(f(X)=f(Y))&=P\left(\sum_{j=1}^N (x_j-y_j)w_j=0\right)\\
&=\sum_{v\in \mathbb Z_p^n} P\left(w_i=v(x_i-y_i)^{-1},\sum_{j\neq i}(x_j+y_j)w_j=-v\right)\\
&=\sum_{v\in \mathbb Z_p^n} P(w_i=v(x_i-y_i)^{-1})P\left(\sum_{j\neq i} (x_j+y_j)w_j=-v\right)\\
&=\frac{1}{p^n} \sum_{v\in \mathbb Z_p^n} P\left(\sum_{j\neq i} (x_j+y_j)w_j=-v\right)\\
&=\frac{1}{p^n}
\end{aligned}
$$
因此一些 $3$ 进制的问题也是可以正常使用异或哈希的思想来解决的。
