---
date: 2026-02-20 18:23:00
updated: 2026-02-20 18:23:00
title: VP - Codeforces Round 1078 (Div. 2)
katex: true
tags:
- Algo
- C++
categories:
- XCPC
description: Codeforces Round 1078 (Div. 2) 题解。
---

## [CF2194D] Table Cut

已知 $a+b$ 是定值，求 $ab$ 的最大值，那么根据均值不等式或者说二次函数的视角来看，$a$ 尽量接近 $\cfrac{a+b}{2}$ 能让 $ab$ 尽可能大。

然后，$a$ 也确实可以从 $0$ 到 $a+b$ 任意取：$0$ 指 `DDD...RRR...`，然后从最后一行开始往上，可以让 $a$ 逐个从左向右取，直到 $a=\lfloor\cfrac{a+b}{2}\rfloor$ 为止。当然上取整也行。

[AC Code](https://gist.github.com/hikariyo/5148f237394fb4756656bb67870c669b)。

## [CF2194E] The Turtle Strikes Back

本题的核心问题是如何快速求出改了一个点后的全局最长路。

对于一般的图确实不太好做，但这是网格图，并且只能向右或向下。

对于一个点 $(i,j)$，我们按照 $i+j$ 给他们分类，那么每一步要么 $i\gets i+1$，要么 $j\gets j+1$，都会使得 $i+j\gets i+j+1$。

我们只需要对每一类记录经过这一类点的最长路和次长路，然后当强制不经过点 $(i,j)$ 时，就查询 $i+j$ 这个类中不经过 $(i,j)$ 的最长路即可，这里我偷懒用了堆存储，实际上用两个变量就可以存储。

[AC Code](https://gist.github.com/hikariyo/1742ce945c81c3eba2c72c4a7d8a8c69)。

## [CF2194F1] Again Trees... (Easy Version)

我认为本题的难点在于状态设计。

对于当前的节点 $u$，切断边 $(u,v)$ 后，我们并不知道 $u$ 还和谁连着；不切断边 $(u,v)$，我们也不知道 $v$ 到底和谁连着。

但是，我们并不可能存储连通块本身的信息，所以考虑存储某些连通块的异或和。

那到底是存还与 $u$ 连着的连通块的异或和，还是存切掉的？

设 $s_u$ 表示 $u$ 的子树权值的异或和，那么已经与 $u$ 分离的部分应当是某个 $b_i$，从这个角度来看，我们应当存已经切掉的异或和，这样才能保证状态大小与 $k$ 相关，其中 $k$ 是一个比较小的数。

现在又有一个问题，就是虽然 $k$ 很小，但是 $b$ 的值域并没有限制，所以我们转而用 $2^k$ 的代价存储 $b$ 的下标出现在异或和中的情况，例如 $1001$ 表示 $b_1\oplus b_4$。

那么，每一个点的初始值就是 $dp_{u,0}=1,dp_{u,*}=0$。转移时，枚举下标的状态 $S$，然后枚举所有的边 $(u,v)$：

1. 如果不断，那么枚举 $v$ 的下标状态 $T$，有 $dp_{u,S\oplus T}\gets dp_{u,S\oplus T}+dp_{u,S}\times dp_{v,T}$；
2. 如果断开，那么枚举 $v$ 的下标状态 $T$，$v$ 切掉异或和为 $T$ 的部分后应当是某个 $b_i$，那么 $dp_{u,S\oplus T\oplus 2^{i-1}}\gets dp_{u,S\oplus T\oplus 2^{i-1}}+dp_{u,S}\times dp_{v,T}$。

统计答案时，我们枚举 $b$ 集合的所有情况检查是否合法并且累和即可，这样单次复杂度是 $O(n4^k+k2^k)$，可以通过。

[AC Code](https://gist.github.com/hikariyo/df888d1c732f641243380c29f57fb2e0)。

## [CF2194F2] Again Trees... (Hard Version)

如果我们仍然沿用下标的逻辑，那么是无法做出困难版本的，因为没什么办法处理断开之后的那个转移。

又因为 $k$ 仍然是一个比较小的数，所以这里可以把 $b_i$ 全部插入线性基中，用线性基的坐标表示切掉部分的异或和，这样就可以控制在 $2^k$ 的大小了。

对于不断开的情况，显然这里是一个异或卷积的形式，接下来看断开的情况。

首先不难发现，如果 $s_v$ 本身不能被这个线性基表出，那么这条边是不可能断开的，因为断开之后这个块必然不能表示称若干个 $b$ 的异或和；如果 $s_v$ 可以被这个线性基表出，那么我们枚举所有的 $b_i$，看这个状态是怎么转移的：
$$
\forall S,T,b_i,dp_{u,S\oplus T\oplus b_i}\gets dp_{u,S\oplus T\oplus b_i} + dp_{u,S}\times dp_{v,T}\\
\forall S,b_i,dp_{u,S\oplus s_v}\gets dp_{u,S\oplus s_v} + dp_{u,S}\times dp_{v,s_v\oplus b_i}
$$
由于我们并不能每一次都进行一遍正变换和逆变换（复杂度 $O(nk2^k)$ 太大），所以我们只能维护 $\mathcal F(dp_u)$，但是公式的推导还是在未变换之前来推的。

我们希望把这个写成卷积的形式，即 $\forall S,T,f_S\gets f_S+g_{T}\times h_{S\oplus T}$。

进一步转化式子：
$$
\forall S,b_i,dp_{u,S}\gets dp_{u,S} + dp_{u,S\oplus s_v}\times dp_{v,s_v\oplus b_i}
$$
构造 $g_{s_v}$ 满足 $g_{s_v}=\sum_{i=1}^k dp_{v,s_v\oplus b_i}$，其他位置 $T\neq s_v$ 都有 $g_T=0$，那么就有：
$$
\forall S,dp_{u,S}\gets dp_{u,S}+dp_{u,S\oplus s_v}\times g_{s_v}\\
\forall S,T,dp_{u,S}\gets dp_{u,S}+dp_{u,S\oplus T}\times g_{T}
$$
此时的更新就是 $dp_u\gets dp_u*dp_v+dp_u*g$。如果 $s_v$ 不能表出，那么令 $g=0$ 即可。

于是，枚举每一条边就是 $\mathcal F(dp_u)\gets \mathcal F(dp_u)\cdot (\mathcal F(dp_v)+\mathcal F(g))$，其中 $\cdot$ 是逐元素乘法。

还有一个问题，这个 $g_{s_v}$ 我们如何快速求得？

我们并不能获取到 $dp_v$ 原始的值，我们只能把它尝试写成某种卷积的形式，于是：
$$
\forall b_i,g_{s_v}\gets g_{s_v}+dp_{v,s_v\oplus b_i}\times e_{b_i}\\
\forall T,g_{s_v}\gets g_{s_v}+dp_{v,s_v\oplus T}\times e_T\\
\forall S,T,h_{S}\gets h_{S}+dp_{v,S\oplus T}\times e_T
$$
其中当 $T$ 为某个 $b_i$ 时，$e_T=1$，否则 $e_T=0$。

那我们可以在 $O(2^k)$ 的复杂度求出 $\mathcal F(h)$，然后在 $O(2^k)$ 复杂度求出 $h_{s_v}$ 单点的值，进而构造出整个 $\mathcal F(g)$。

统计答案的方法与简单版本相似，这样复杂度是单次 $O(n2^k+k2^k)$，有些卡常。

[AC Code](https://gist.github.com/hikariyo/04f6c5d4ec53d8e093478e1d3f47e99d)。