---
date: 2026-04-03 17:39:00
updated: 2026-04-03 17:39:00
title: 计算几何 - 凸包
katex: true
tags:
- Algo
- C++
- Geometry
categories:
- XCPC
description: 本文主要讲解凸包的定义、性质，以及 Andrew 算法及其正确性。
---

## 凸组合

$n$ 元点集 $P=\{p_1,p_2,\dots, p_n\}$，对于实数 $\lambda_1,\lambda_2,\dots, \lambda_n$，满足：

1. $\lambda_i\ge 0,i=1,2,\dots,n$；
2. $\sum_{i=1}^n \lambda_i=1$；

那么称 $\sum_{i=1}^n \lambda_i p_i$ 为点集 $P$ 的一个凸组合，其中 $p_i$ 是以原点为起点，以这个点为终点的向量。

## 凸包

$n$ 元点集 $P$ 的凸包，定义为所有 $P$ 的凸组合构成的集合，形式化地：
$$
CH(P)=\left\{\sum_{i=1}^n \lambda_i p_i \Bigg| \lambda_i\ge 0,\sum_{i=1}^n \lambda_i=1 \right\}
$$
也就是说，某一点 $q$ 在凸包内部当且仅当它能被表示为 $P$ 的某个凸组合。

## 凸集

一个点集 $C$ 为凸集，当且仅当对于任意两个点 $p,q\in C$，对任意的 $\lambda\in(0,1)$，都有 $\lambda p+(1-\lambda)q\in C$。

### 凸包是一个凸集

证明 $P$ 的凸包是一个凸集：

不妨设 $p=\sum_{i=1}^n \lambda_i p_i$，$q=\sum_{i=1}^n\mu_ip_i$，那么任取一个 $\gamma\in (0,1)$，有 $\gamma p+(1-\gamma)q=\sum_{i=1}^n [\gamma \lambda_i+(1-\gamma)\mu _i]p_i$

观察系数，$\gamma\in (0,1),\lambda_i,\mu_i\ge 0$，因此每一个系数都非负；并且 $\sum_{i=1}^n[\gamma \lambda_i+(1-\gamma)\mu_i]=\gamma\sum_{i=1}^n\lambda_i +(1-\gamma)\sum_{i=1}^n \mu_i=1$。

所以 $CH(P)$ 是一个凸集。证毕。

### 凸包是“最小”的凸集

何为最小？形式化地，证明对于任意的凸集 $C$，如果 $P\subseteq C$，那么 $CH(P)\subseteq C$：

任取 $p\in CH(P)$，都存在一组 $\lambda_i$，使得 $p=\sum_{i=1}^n\lambda_ip_i$，下面我们用数学归纳法证明，形如这样的 $p\in C$。

设其中有 $k$ 个 $\lambda_i$ 不为零，因为这些 $\lambda_i$ 的和是 $1$，那么 $k\ge 1$。下设不为零的 $\lambda_i$ 分别为 $\lambda_{j_1},\lambda_{j_2},\dots,\lambda_{j_k}$。当 $k=1,2$ 时，根据定义有 $p\in C$。

若 $k=k_0$ 时，命题成立，那么 $k=k_0+1$ 时，因为 $k\ge 2$，所以显然不可能有一个 $\lambda=1$，并且：
$$
\begin{aligned}
p&=\lambda_{j_k}p_{j_k}+\sum_{i=1}^{k-1} \lambda_{j_i}p_{j_i}\\
&=\lambda_{j_k}p_{j_k}+(1-\lambda_{j_k})\sum_{i=1}^{k-1}\frac{\lambda_{j_i}}{1-\lambda_{j_k}} p_{j_i}
\end{aligned}
$$
结合 $\sum_{i=1}^k \lambda_{j_i}=1$，我们有 $\sum_{i=1}^{k-1}\cfrac{\lambda_{j_i}}{1-\lambda_{j_k}}=1$。由归纳假设，$\sum_{i=1}^{k-1}\cfrac{\lambda_{j_i}}{1-\lambda_{j_k}}p_{j_i}\in C$，把它看成一个整体后，用定义可以得知 $p\in C$。证毕。

## Andrew 算法

为了方便行文，我们引入一个记号 $\Delta(A,B,C)=(x_B-x_A)(y_C-y_A)-(x_C-x_A)(y_B-y_A)$，也就是 $AB$ 和 $AC$ 的叉积。

算法流程如下：

1. 将点集 $P$ 按照 $x$ 为第一关键字、$y$ 为第二关键字的字典序升序排序；
2. 维护一个栈 $S$，顺序遍历 $P$ 中节点，假设当前遍历到 $p_i$：
   + 若 $|S|\le 1$，直接将 $p_i$ 入栈；
   + 否则，设 $A$ 为栈顶下一个元素，$B$ 为栈顶元素，如果 $\Delta (A,B,p_i)>0$，那么 $p_i$ 入栈，否则弹出栈顶元素，重复上一步。
3. 遍历完成后得到下凸壳，反方向遍历一遍得到上凸壳。按照算法的执行逻辑，不难看出 $p_1,p_n$ 一定是下凸壳的起点和终点、上凸壳的终点和起点。

于是我们得到了一个逆时针的、首尾相接的边的多边形。

设 $C_1,C_2,\dots,C_m$ 为这个多边形的顶点。为了简化行文，我们令 $C_{m+1}=C_1$。

下证所有多边形内部的点构成的集合 $C$，是原点集 $P$ 的凸包 $CH(P)$。

形式化地，集合 $C$ 被定义为：
$$
C=\left\{Q\in \mathbb R^2\mid \forall i\in[1,m],\Delta(C_i,C_{i+1},Q)\ge 0\right\}
$$
要证明 $C=CH(P)$，我们只能证明这两个集合互相包含。 

### P 是 C 的子集

我们将通过数学归纳法证明这一结论，我们将要证明：任取 $P$ 中的一个点 $p$，对于任意 $i\in [1,m-1]$，都有 $\Delta(C_i,C_{i+1},p)\ge 0$。

为了避免讨论 $x$ 相同的情况，我们给所有的点做一个线性变换，让每一个点都左乘变换矩阵 $T=\begin{bmatrix}1&\varepsilon\\0&1\end{bmatrix}$，显然 $\det(T)=1$，通过简单的代数展开可以得到，$\Delta(A,B,C)\ge 0$ 当且仅当 $\Delta(TA,TB,TC)\ge 0$。由于点集是有限的，我们总能取到一个 $\varepsilon$，使得变换后 $x$ 互不相同，并且变换之后仍然保持字典序升序。

一旦允许了这样的事情之后，这个证明就变得及其简单了：当我们处理下凸壳，进行弹出栈的操作时，每一段横坐标对应的新的斜率总会比原来的斜率更小，从而这个边界的每一个点对应的 $y$ 坐标都在减小。通过归纳假设，我们可以断定原先的点 $y$ 坐标必然仍然 $\ge $ 边界的 $y$ 坐标。

通过代数变形可以得知这和 $\Delta\ge 0$ 是等价的，又因为对原图的任意三个点 $\Delta(A,B,C)\ge 0$ 当且仅当 $\Delta(TA,TB,TC)\ge 0$，所以我们证明了对于下凸壳上的所有点， $\Delta(C_i,C_{i+1},p)\ge 0$ 恒成立。

对于上凸壳也是同理，因此 $P\subseteq C$。

### CH(P) 是 C 的子集

我们可以先给出一个结论，对任意凸组合，都有 $\Delta(A,B,\sum \lambda p)=\sum \lambda\Delta(A,B,p)$。

这个证明比较麻烦，但是只是中学数学的公因式消消乐，本文不做过多阐述。

有了上面这个公式，就可以直接得到：任取 $p,q\in C$，任取 $\lambda\in[0,1]$，对于所有的 $C_iC_{i+1}$，都有 $\Delta(C_i,C_{i+1},\lambda p+(1-\lambda)q)=\lambda \Delta(C_i,C_{i+1},p)+(1-\lambda)\Delta(C_i,C_{i+1},q)\ge 0$，因此 $C$ 是一个凸集。

我们之前证明了，如果 $C$ 是凸集，并且 $P\subseteq C$，就有 $CH(P)\subseteq C$。

### C 是 CH(P) 的子集

这个命题的意思就是要我们证明，$C$ 中任意一个点都可以表示成 $P$ 中点的凸组合。

同样是给每一个点都左乘变换矩阵 $T$，保证 $x$ 互不相同之后来研究。

对于 $C$ 中任意一个点 $Q(x,y)$，总可以在下凸包找到 $x_i\le x\le x_{i+1}$，那么过 $Q$ 引一条铅垂线，与下面这个边的交点是可以被边的顶点用凸组合表示的，对于上凸包同理，那么这两个交点进一步可以用凸组合表示出 $Q$。我承认这里没有那么严谨，民科轻喷。

因此 $C\subseteq CH(P)$。

