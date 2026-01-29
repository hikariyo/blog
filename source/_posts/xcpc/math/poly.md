---
date: 2025-12-22 18:14:00
updated: 2025-12-22 18:14:00
title: 数学 - 多项式
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 使用了多项式加速计算的题目。
---

## [ZJOI2014] 力

首先转化一下式子：
$$
E_k=\sum_{i=1}^{k-1}\frac{q_i}{(k-i)^2}-\sum_{i=k+1}^n \frac{q_i}{(k-i)^2}=E_{1k}+E_{2k}
$$
可以令 $q_0=0$，就能推出 $E_{1k}$ 的卷积形式：
$$
E_{1k}=\sum_{i=0}^{k-1}\frac{q_i}{(k-i)^2}=\sum_{i=0}^{k-1} f_ig_{k-1-i}
$$
其中 $f_i=q_i,g_i=\cfrac{1}{(i+1)^2}$。

由于卷积的指标是从 $0$ 开始的，所以对于 $E_{2k}$，需要令 $j=n-i+1$，那么：
$$
E_{2k}=\sum_{j=1}^{n-k}\frac{q_{n-j+1}}{(n-k-j+1)^2}=\sum_{j=0}^{n-k}f_jg_{n-k-j}
$$
其中 $f_i=q_{n-i+1},g_i=\cfrac{1}{(i+1)^2}$，并且 $q_{n+1}=0$。

复杂度主要是一次 FFT，为 $O(n\log n)$。

[AC Code](https://gist.github.com/hikariyo/6d17dbd93d1989ea543fe3873394fa0e)。

## [AHOI2017/HNOI2017] 礼物

首先，给 $y$ 整体 $+c$ 对贡献的影响等价于对 $x$ 整体 $-c$，所以我们可以把这个条件转化成 $x$ 整体加一个整数。

把这个式子拆一下：
$$
\sum_{i=1}^n(x_i+c-y_i)^2=nc^2+(2\sum_{i=1}^n x_i-2\sum_{i=1}^n y_i)c+\sum_{i=1}^n x_i^2+\sum_{i=1}^ny_i^2-2\sum_{i=1}^n x_iy_i
$$
可以看出，这是一个关于 $c$ 的二次函数，并且二次项、一次项系数与排列顺序无关。

现在我们要求常数项的最小值，转化为求 $\sum x_iy_i$ 的最大值。

如果把 $y_1\sim y_n$ 复制一遍到 $y_{n+1}\sim y_{2n}$，会发现每一种排列都对应着 $x_1\sim x_n$ 与 $y_{d}\sim y_{n+d}$ 对应相乘再相加。

手玩一下就可以知道如果令 $f=\{0,x_1,x_2,\dots,x_n,0,0,\dots\}$，$g=\{y_n,y_{n-1},\dots,y_1,y_n,y_{n-1},\dots,y_1\}$，然后 $h=f*g$ 那么 $h_n\sim h_{2n-1}$ 就是我们要求的答案了。

复杂度主要是一次 NTT，为 $O(n\log n)$。

[AC Code](https://gist.github.com/hikariyo/15ac53033baa53672a913c25b8f89508)。

## [CF755G] PolandBall and Many Other Balls

状态转移方程比较简单：$f_{n,k}=f_{n-1,k}+f_{n-1,k-1}+f_{n-2,k-1}$。

如果我们把 $f_n$ 看成一个最高 $k$ 次的多项式，那么就有 $f_n=f_{n-1}+xf_{n-1}+xf_{n-2}$。

这可以写成 $\begin{bmatrix}f_n\\f_{n-1}\end{bmatrix}=\begin{bmatrix}x+1&x\\1&0\end{bmatrix}\begin{bmatrix}f_{n-1}\\f_{n-2}\end{bmatrix}$。

显然地，$f_0=1$，$f_1=1+x$，这些多项式的次数不超过 $k$，所以复杂度是 $O(k\log k\log n)$。

[AC Code](https://gist.github.com/hikariyo/8a00212904ab4ba074ced0d7284772d3)。

## [TJOI2017] DNA

下面设 $S$ 为短串，$T$ 为长串。

我们需要对每一个位置求出失配数量，考虑到本题目字符集小，所以逐个考虑所有字符。

设当前字符是 $1$，非当前字符是 $0$，那么我们希望 $S_i=1$ 时 $T_i=1$，$S_i=0$ 时 $T_i$ 无所谓，所以可以写出 $\sum S_i(1-T_i)$ 就是每一段的失配数量。

把每个字符的失配数量加起来就是总的失配数量。

这个和式同样可以把 $S$ 串翻转过来，并在后面补 $0$ 到与 $T$ 一样长，那么每一个和就对应它们卷积的一个位置。

复杂度为单次 $O(|\Sigma||T|\log |T|)$，其中 $\Sigma$ 是字符集。

[AC Code](https://gist.github.com/hikariyo/1067a64a0bd9406ca374e45224995017)。

## [CF528D] Fuzzy Search

和上面的题目一样，可以逐个考虑所有字符。

这里的前后 $k$ 个字符，可以用一个差分数组来把每个字符前后 $k$ 个位置都打上标记。

复杂度为 $O(|\Sigma|n\log n)$，其中 $\Sigma$ 是字符集。

[AC Code](https://gist.github.com/hikariyo/99398be8ebcb8f855345c36c853ff912)。
