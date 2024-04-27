---
date: 2023-12-03 22:52:00
title: 微积分 - 由最短时间原理推导折射定律
katex: true
tags:
- Math
- Calculus
categories:
- Learn
description: 从最短时间原理出发，用导数值为零列方程，推导出折射定律中折射率等于速度之比的公式。
---

## 简介

高中物理课本上提到过折射定律，但是没有推导过程，即下面的这个式子：
$$
\frac{n_2}{n_1}=\frac{\sin\theta_1}{\sin\theta_2}=\frac{v_1}{v_2}
$$
这里由最短时间原理入手推导一下这个等式，原理就是求个导就结束了。

## 构造模型

这里偷个懒，直接在纸上画了：

![Model](/img/2023/12/refraction.jpg)

可以容易列出时间的等式：
$$
t=\frac{\sqrt{a^2+x^2}}{v_1}+\frac{\sqrt{b^2+(1-x)^2}}{v_2}
$$
根据最短时间原理，$t$ 一定取得最小值，那么就有此处导数为 $0$，即：
$$
\frac{\mathrm{d}t}{\mathrm{d}x}=\frac{x}{v_1\sqrt{a^2+x^2}}-\frac{1-x}{v_2\sqrt{b^2+(1-x)^2}}=0
$$
可以看出：
$$
\begin{aligned}
\sin\theta_1&=\frac{x}{\sqrt{a^2+x^2}}\\
\sin\theta_2&=\frac{1-x}{\sqrt{b^2+(1-x)^2}}
\end{aligned}
$$
于是就有最开始的式子：
$$
\frac{\sin\theta_1}{v_1}=\frac{\sin\theta_2}{v_2}
$$
推导结束。





