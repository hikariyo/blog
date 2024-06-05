---
title: 数学 - 幂平均不等式
date: 2024-06-03 23:37:00
katex: true
categories:
- Learn
description: 幂平均不等式是常见的均值不等式的拓展，本文利用 Jensen 不等式来证明幂平均不等式。
tags:
- Math
---

## 幂平均

若 $p$ 为非零实数，对于正实数 $x_1,\dots,x_n$ 的 $p$ 次幂平均为：
$$
M_p(x_1,\dots,x_n)=\left(\frac{1}{n}\sum_{i=1}^n x_i^p\right)^{\frac{1}{p}}
$$
特别地，给出 $M_0(x_1,\dots,x_n)$ 的定义，中间用一步洛必达即可：


$$
\begin{aligned}
M_0(x_1,\dots,x_n)&=\lim_{p\to 0}M_p(x_1,\dots,x_n)\\
&=\lim_{p\to 0}\left(\frac{1}{n}\sum_{i=1}^nx_i^p\right)^{\frac{1}{p}}\\
&=\lim_{p\to 0}\exp\left(\frac{\ln(\frac{1}{n}\sum_{i=1}^nx_i^p)}{p}\right)\\
&=\lim_{p\to 0}\exp\left(\frac{\frac{1}{n}\sum_{i=1}^n x_i^p\ln x_i}{\frac{1}{n}\sum_{j=1}^n x_j^p}\right)\\
&=\exp\left(\frac{1}{n}\sum_{i=1}^n\ln x_i\right)\\
&=\left(\prod_{i=1}^n x_i\right)^{\frac{1}{n}}
\end{aligned}
$$
同时，给出 $M_{+\infin}(x_1,\dots,x_n)$ 的定义，设 $x_k=\max\{x_1\dots,x_n\}$ 有：
$$
\begin{aligned}
M_{+\infin}(x_1,\dots,x_n)&=\lim_{p\to +\infin} M_p(x_1,\dots,x_n)\\
&=\lim_{p\to+\infin}\left(\frac{1}{n}\sum_{i=1}^n x_i^p\right)^{\frac{1}{p}}\\
&=\lim_{p\to +\infin}\left(\frac{x_k^p}{n}\sum_{i=1}^n \left(\frac{x_i}{x_k}\right)^p\right)^{\frac{1}{p}}\\
&=x_k=\max\{x_1,\dots,x_n\}
\end{aligned}
$$
类似地，有 $M_{-\infin}(x_1,\dots,x_n)=\min\{x_1,\dots,x_n\}$。

## 幂平均不等式

对于实数 $p,q$ 若 $p<q$ 则有：
$$
M_p(x_1,\dots,x_n)\le M_q(x_1,\dots,x_n)
$$
下面分步骤证明这个不等式。

### 步骤 1

先证与 $0$ 相关的不等式，若 $p$ 为正实数，有：
$$
M_{-p}(x_1,\dots,x_n)\le M_0(x_1,\dots,x_n)\le M_{p}(x_1,\dots,x_n)
$$
即证明：
$$
\left(\frac{1}{n}\sum_{i=1}^n x_i^{-p}\right)^{-\frac{1}{p}}\le \left(\prod_{i=1}^n x_i\right)^{\frac{1}{n}}\le \left(\frac{1}{n}\sum_{i=1}^n x_i^p\right)^{\frac{1}{p}}
$$
根据 Jensen 不等式，有：
$$
\frac{1}{n}\sum_{i=1}^n \ln x_i\le \ln\left(\frac{1}{n}\sum_{i=1}^n x_i\right)
$$
两边取指数，可以得到：
$$
\prod_{i=1}^n x_i^{\frac{1}{n}} \le \frac{1}{n}\sum_{i=1}^n x_i
$$
考虑 $x_i\gets x_i^p$，得到：
$$
\prod_{i=1}^n x_i^{\frac{p}{n}} \le \frac{1}{n}\sum_{i=1}^n x_i^p
$$
两边取 $\frac{1}{p}$ 次幂，有：
$$
\left(\prod_{i=1}^n x_i\right)^{\frac{1}{n}}\le \left(\frac{1}{n}\sum_{i=1}^n x_i^p\right)^{\frac{1}{p}}
$$
当且仅当 $x_1=\dots=x_n$ 取等，于是原式得证。对于 $-p$ 同理，于是 $M_p(x_1,\dots,x_n)$ 与 $M_0(x_1,\dots,x_n)$ 的关系得证。

### 步骤 2

对于实数 $0<p<q$，要证明幂平均不等式成立即证明：
$$
\left(\frac{1}{n}\sum_{i=1}^n x_i^p\right)^{\frac{1}{p}}\le \left(\frac{1}{n}\sum_{i=1}^n x_i^q\right)^{\frac{1}{q}}
$$
两边取 $q$ 次幂，即证明：
$$
\left(\frac{1}{n}\sum_{i=1}^n x_i^p\right)^{\frac{q}{p}}\le \frac{1}{n}\sum_{i=1}^n x_i^q
$$
考虑函数 $f(x)=x^{\frac{q}{p}}$，有 $f''(x)=\frac{q}{p}(\frac{q}{p}-1)x^{\frac{q}{p}-2}>0$，故 $f(x)$ 是下凸的，利用 Jensen 不等式可以得到：
$$
f\left(\frac{1}{n}\sum_{i=1}^n x_i^p\right)\le \frac{1}{n}\sum_{i=1}^n f(x_i^p)=\frac{1}{n}\sum_{i=1}^n x_i^q
$$
当且仅当 $x_1=\dots=x_n$ 取等，于是原式得证。

### 步骤 3

对于实数 $-p<-q<0$，证明：
$$
\left(\frac{1}{n}\sum_{i=1}^n \frac{1}{x_i^p}\right)^{-\frac{1}{p}}\le \left(\frac{1}{n}\sum_{i=1}^n \frac{1}{x_i^q}\right)^{-\frac{1}{q}}
$$
两边取倒数，得到：
$$
\left(\frac{1}{n}\sum_{i=1}^n (\frac{1}{x_i})^p\right)^{\frac{1}{p}}\ge \left(\frac{1}{n}\sum_{i=1}^n (\frac{1}{x_i})^q\right)^{\frac{1}{q}}
$$
利用上面已经证明过的 $0<q<p$ 的结论得证。

### 步骤 4

对于实数 $p<0<q$，以 $M_0(x_1,\dots,x_n)$ 为中间值同样可以证明幂平均不等式。

