---
date: 2026-05-06 15:53:00
updated: 2026-05-06 15:53:00
title: 数据结构 - 并查集
katex: true
tags:
- Algo
- C++
- DS
categories:
- XCPC
description: 记录一些并查集相关算法的正确性证明，不包含复杂度分析。
---

## 符号约定

下文固定一个正整数 $n$，定义全集 $U=\{1,2,\dots,n\}$。

并查集用一个父亲函数 $p:U\to U$ 表示，初始化 $\forall x\in U,p(x)=x$。

称一个父亲函数 $p$ 是合法的，当且仅当对任意 $x\in U$，不断应用 $p$：$x,\ p(x),\ p(p(x)),\dots$ 最终一定会到达某个点 $r$，并且这个点满足 $r=p(r)$，这个 $r$ 就叫做 $x$ 在父亲函数 $p$ 下的根，记为 $\operatorname{root}_p(x)$。

一个集合划分 $\mathcal P$  指 $U$ 的若干个非空子集，它们两两不交，且并集为 $U$。

对合法的 $p$，划分 $\mathcal P_p$ 定义为 $\{\{x\in U\mid \operatorname{root}_p(x)=r\}\mid r\in U,r=p(r)\}$。

## 路径压缩

求 $\operatorname{root}_p(x)$ 的函数 $\operatorname{find}(x)$ 的操作过程如下：

1. 若 $p(x)=x$，返回 $x$；
2. 若 $p(x)\neq x$，先递归求出 $r=\operatorname{find}(p(x))$，再令 $p(x)\gets r$，并返回 $r$。

假设 $x$ 到根路径为 $v_1,v_2,\dots,v_k$，其中 $v_k$ 是根，那么本次操作就是 $\forall i,\ 1\le i<k,\ p(v_i)\gets v_k$。

下证设进行操作前父亲函数 $p$ 是合法的，那么进行一次操作之后父亲函数 $p'$ 也是合法的，并且 $\forall y\in U,\ \operatorname{root}_{p'}(y)=\operatorname{root}_p(y)$。

**证明**：

任取一个 $y\in U$：

+ 如果从 $y$ 出发不断应用 $p$ 的过程中没有经过 $v_1,v_2,\dots,v_{k-1}$，那么 $y$ 在 $p$ 下走过的路径与在 $p'$ 下相同，因此仍到达同一个根；
+ 否则设从 $y$ 出发在 $p$ 下第一次经过的被修改点是 $v_i$，则在 $p'$ 下到达 $v_i$ 后下一步直接到达 $v_k$，此时 $\operatorname{root}_{p'}(y)=\operatorname{root}_p(y)=v_k$，并且 $p(v_k)=p'(v_k)=v_k$。

## 集合合并

合并 $x,y$ 所在集合的函数 $\operatorname{merge}(x,y)$ 的操作过程如下：

1. 求 $r_x=\operatorname{find}(x)$ 且 $r_y=\operatorname{find}(y)$；
2. 若 $r_x=r_y$，不做更改，否则令 $p(r_x)$ 修改为 $r_y$。

设执行了两次 $\operatorname{find}$ 之后，修改 $p(r_x)$ 为 $r_y$ 前的父亲函数是 $p$，修改后的父亲函数是 $p'$，那么

+ 满足 $\operatorname{root}_p(u)=r_x$ 的 $u$ 满足 $\operatorname{root}_{p'}(u)=r_y$；

+ 满足 $\operatorname{root}_p(u)=r_y$ 的 $u$ 满足 $\operatorname{root}_{p'}(u)=r_y$；

+ 满足 $\operatorname{root}_p(u)\notin\{r_x,r_y\}$ 的 $u$ 满足 $\operatorname{root}_{p'}(u)=\operatorname{root}_p(u)$。

若 $r_x=r_y$，则 $p'=p$，显然合法；否则 $r_x,r_y$ 是两个不同的根，令 $r_x$ 指向 $r_y$ 后所有点都能找到一个根，所以 $p'$ 仍然是合法的。

## DSU-NEXT

下面介绍一个常用的 trick，用来处理对一个固定的 $n$，每次处理 $l\sim r$ 中的所有点，但是每一个点只需要被处理一次，如将初始全 $0$ 的序列区间修改为 $1$。

定义带哨兵的全集 $I=\{1,2,\dots,n+1\}$，其中 $n+1$ 永远不会被删除，维护一个存活集合 $S\subseteq I$，初始为 $S=I$。

支持对这个集合 $S$ 进行如下两种操作：

1. 对 $x\in S,x\neq n+1$，令 $S\gets S\setminus \{x\}$
2. 在 $S$ 中查询 $\ge i$ 的最小的数，记为 $N(i)$，这里不要求 $i\in S$，只要求 $i\in I$。

因为 $n+1\in S$ 永远成立，所以 $N(i)$ 总是良定的。

我们可以用并查集维护这个集合，具体地讲，初始时 $\forall x\in I,p(x)=x$，那么：

1. 删除操作实现起来即 $p(x)\gets \operatorname{root}_p(x+1)$；
2. 查询操作就是返回 $\operatorname{root}_p(i)$。

我们来证明任意时刻都有 $\operatorname{root}_p(i)=N(i)$。

对删除次数进行归纳，没删除时显然成立，假设某次删除前集合为 $S$，删除了 $x$，分 $x-1,x+1$ 分别是否存活四种情况画个图讨论即可。
