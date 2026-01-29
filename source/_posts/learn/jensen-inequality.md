---
title: 学习 - Jensen 不等式
date: 2024-05-04 18:28:00
updated: 2024-06-03 23:36:00
katex: true
categories:
- Learn
description: Jensen 不等式阐明均值的函数值与函数值的均值之间的不等关系，本文用数学归纳法结合凸函数的性质来进行证明并阐述取等条件。
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

当 $x_1\ne x_2$ 且 $\lambda\in(0,1)$ 时，不失一般性，令 $x_1<x_2$ 且 $a=\lambda x_1+(1-\lambda)x_2$，注意到：
$$
\begin{cases}
a-x_1=(\lambda -1)(x_1-x_2)>0\\
a-x_2=\lambda(x_1-x_2)<0
\end{cases}
$$
将 $a$ 代入定义式，即证：
$$
\lambda(f(a)-f(x_1))\ge (1-\lambda)(f(x_2)-f(a))
$$
根据拉格朗日中值定理 $\exist\xi_1\in(x_1,a),\xi_2\in(a,x_2)$ 得到：
$$
\begin{aligned}
f(a)-f(x_1)&=(a-x_1)f'(\xi_1)=(1-\lambda)(x_2-x_1)f'(\xi_1)\\
f(x_2)-f(a)&=(x_2-a)f'(\xi_2)=\lambda (x_2-x_1)f'(\xi_2)
\end{aligned}
$$
代入原不等式后得到：
$$
\lambda(1-\lambda)(x_2-x_1)f'(\xi_1)\ge (1-\lambda)\lambda (x_2-x_1)f'(\xi_2)
$$
根据 $f''(x)\le 0$，有 $f'(\xi_1)\ge f'(\xi_2)$，得证。由上面的过程可以知道，若 $f''(x)$ 不恒为 $0$，且 $\lambda \not\in\{0,1\}$，则仅当 $x_1=x_2$ 时等号才能成立。

## Jensen 不等式

若 $f(x)$ 为区间 $I$ 上的上凸函数，则对于任意 $x_i\in I$ 满足 $\sum_{i=1}^n \lambda_i = 1,\lambda_i\in [0,1]$ 有：
$$
f\left(\sum_{i=1}^n\lambda_ix_i\right)\ge \sum_{i=1}^n\lambda_if(x_i)
$$
取等条件为 $x_1=x_2=\dots=x_n$ 即平均值的函数值大于等于函数值的平均值，对于下凸函数不等号方向相反。

当 $n=1,2$ 时根据定义显然成立，设 $n=k$ 时成立，则 $n=k+1$ 时有：
$$
f\left(\sum_{i=1}^n\lambda_ix_i\right)=f\left(\sum_{i=1}^k\lambda_i x_i+\lambda_{k+1}x_{k+1}\right)
$$
利用 $n=2$ 时的情况：
$$
\begin{aligned}
f\left(\sum_{i=1}^k\lambda_i x_i+\lambda_{k+1}x_{k+1}\right)&=f\left((1-\lambda_{k+1})\sum_{i=1}^k\frac{\lambda_i}{1-\lambda_{k+1}}x_i+\lambda_{k+1}x_{k+1}\right)\\
&\ge (1-\lambda_{k+1})f\left(\sum_{i=1}^k\frac{\lambda_i}{1-\lambda_{k+1}}x_i\right)+\lambda_{k+1}f(x_{k+1})\\
\end{aligned}
$$
取等条件为 $x_{k+1}=\sum_{i=1}^k\frac{\lambda_i}{1-\lambda_{k+1}}x_i$；易知 $\sum_{i=1}^k \lambda_i=1-\lambda_{k+1}$，则 $\sum_{i=1}^k\frac{\lambda _i}{1-\lambda_{k+1}}=1$，根据归纳假设有：
$$
\begin{aligned}
(1-\lambda_{k+1})f\left(\sum_{i=1}^k\frac{\lambda_i}{1-\lambda_{k+1}}x_i\right)+\lambda_{k+1}f(x_{k+1})&\ge (1-\lambda_{k+1})\sum_{i=1}^k\frac{\lambda _i}{1-\lambda_{k+1}}f(x_i)+\lambda_{k+1}f(x_{k+1})\\
&=\sum_{i=1}^{k+1}\lambda_if(x_i)
\end{aligned}
$$
取等条件为 $x_1=x_2=\dots=x_k$，结合上一步的取等条件，因此当 $x_1=x_2=\dots=x_{k+1}$​​ 时两不等号可以同时取等。
