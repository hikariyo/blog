---
title: 数学 - Jensen 不等式的数学归纳法证明
date: 2024-05-04 18:28:00
katex: true
categories:
- Learn
description: 学习多元均值不等式的时候发现可以用 Jensen 不等式来证明，于是又学习了一下其证明方法，本文做一个简要记录。
tags:
- Math
---

## 凸函数

若 $f(x)$ 为区间 $I$ 上的上凸函数，则对于任意 $x_1,x_2\in I$ 且 $\lambda \in[0,1]$ 有：
$$
f(\lambda x_1+(1-\lambda)x_2)\ge \lambda f(x_1)+(1-\lambda)f(x_2)
$$
下面证明若在区间 $I$ 上有 $f''(x)\le 0$，则 $f(x)$ 在 $I$​ 上为上凸函数。

当 $x_1=x_2$ 或 $\lambda =0,1$ 时显然成立。

当 $x_1\ne x_2$ 且 $\lambda\in(0,1)$ 时，不失一般性，令 $x_1<x_2$ 且 $a=\lambda x_1+(1-\lambda)x_2$ 即证：
$$
\lambda[f(a)-f(x_1)]\ge (1-\lambda)[f(x_2)-f(a)]
$$
根据拉格朗日中值定理 $\exist\xi_1\in(x_1,a),\xi_2\in(a,x_2)$ 将得到的式子代入原不等式后得到：
$$
\lambda f'(\xi_1)(a-x_1)\ge (1-\lambda)f'(\xi_2)(x_2-a)
$$
即：
$$
\lambda(1-\lambda)(x_2-x_1)f'(\xi_1)\ge (1-\lambda)\lambda (x_2-x_1)f'(\xi_2)
$$
根据 $f''(x)\le 0$，有 $f'(\xi_1)\ge f'(\xi_2)$，得证。

## Jensen 不等式

若 $f(x)$ 为区间 $I$ 上的上凸函数，则对于任意 $x_i\in I$ 满足 $\sum_{i=1}^n \lambda_i = 1,\lambda_i\in [0,1]$ 有：
$$
f(\sum_{i=1}^n\lambda_ix_i)\ge \sum_{i=1}^n\lambda_if(x_i)
$$
取等条件为 $x_1=x_2=\dots=x_n$ 即平均值的函数值大于等于函数值的平均值，对于下凸函数不等号方向相反。这里就不讨论 $f(x)$ 是一次函数或者常函数这种特殊情况了。

当 $n=1,2$ 时根据定义显然成立，设 $n=k$ 时成立，则 $n=k+1$ 时有：
$$
f(\sum_{i=1}^n\lambda_ix_i)=f(\sum_{i=1}^k\lambda_i x_i+\lambda_{k+1}x_{k+1})
$$
利用 $n=2$ 时的情况：
$$
\begin{aligned}
f(\sum_{i=1}^k\lambda_i x_i+\lambda_{k+1}x_{k+1})&=f[(1-\lambda_{k+1})\sum_{i=1}^k\frac{\lambda_i}{1-\lambda_{k+1}}x_i+\lambda_{k+1}x_{k+1}]\\
&\ge (1-\lambda_{k+1})f(\sum_{i=1}^k\frac{\lambda_i}{1-\lambda_{k+1}}x_i)+\lambda_{k+1}f(x_{k+1})\\
\end{aligned}
$$
取等条件为 $x_{k+1}=\sum_{i=1}^k\frac{\lambda_i}{1-\lambda_{k+1}}x_i$；易知 $\sum_{i=1}^k \lambda_i=1-\lambda_{k+1}$，则 $\sum_{i=1}^k\frac{\lambda _i}{1-\lambda_{k+1}}=1$，根据归纳假设有：
$$
\begin{aligned}
(1-\lambda_{k+1})f(\sum_{i=1}^k\frac{\lambda_i}{1-\lambda_{k+1}}x_i)+\lambda_{k+1}f(x_{k+1})&\ge (1-\lambda_{k+1})\sum_{i=1}^k\frac{\lambda _i}{1-\lambda_{k+1}}f(x_i)+\lambda_{k+1}f(x_{k+1})\\
&=\sum_{i=1}^{k+1}\lambda_if(x_i)
\end{aligned}
$$
取等条件为 $x_1=x_2=\dots=x_k$，结合上一步的取等条件，因此当 $x_1=x_2=\dots=x_{k+1}$​​ 时两不等号可以同时取等。

这里我没有证明也不知道取等条件是否唯一，只是阐述一下等号经过两次操作后也是可以取到的。

## 均值不等式

对于 $n$ 元均值不等式 $(x_i> 0)$ 有：
$$
\frac{1}{n}\sum_{i=1}^nx_i\ge(\prod_{i=1}^n x_i)^{\frac{1}{n}}
$$
两边取对数，有：
$$
\ln(\frac{1}{n}\sum_{i=1}^nx_i)\ge\frac{1}{n}\sum_{i=1}^n\ln x_i
$$
易得 $f(x)=\ln x$ 为区间 $(0,+\infin)$ 的上凸函数，取 $\lambda_i=\frac{1}{n}$ 即可用 Jensen 不等式证明。

