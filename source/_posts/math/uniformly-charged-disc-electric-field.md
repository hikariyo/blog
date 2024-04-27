---
date: 2023-07-08 00:39:00
update: 2023-07-13 22:49:22
title: 微积分 - 探究电荷均匀分布的圆盘在中轴线产生的场强
katex: true
tags:
- Math
- Calculus
categories:
- Learn
---

## 简介

本文讨论电荷均匀分布的圆盘，在过盘心且垂直于盘面的轴线上所产生的场强，在中学物理中可以视为集中于盘心的点电荷的原理。

为了分析这个问题，需要先考虑圆环在轴线上产生的场强。

偷个懒，直接归到数学里面了。

## 圆环

> 正电荷 $q$ 均匀分布在半径为 $R$ 的圆环上，计算过环心且垂直于环面轴线上任意一点 $P$ 的场强。
>
> 图源高等教育出版社物理学第七版，下同。

<img src="/img/2023/07/physics-5-10.jpg" style="zoom:66%;" />

这道题在中学物理中就有涉及（起码我的物理老师讲过），这里重新审视一下这道题目。

在环上取电荷元 $\mathrm{d}q$，其对 $P$ 产生的场强大小 $\mathrm{d}E$ 为：
$$
\mathrm{d}E=\frac{1}{4\pi \varepsilon_0}\frac{\mathrm{d}q}{r^2}
$$
根据对称性，在圆环上任意选一个点，都能选出根据环心对称后的一点，它们对 $P$ 产生的场强在垂直 $x$ 轴的分量互相抵消，即 $\int_L \mathrm{d}\vec{E_{\perp}}=\vec{0}$ 因此整个圆环对 $P$ 产生的场强方向沿 $x$ 轴正方向。

因此，在 $P$ 点的场强是由在 $x$ 轴上的分量叠加形成的，且 $\mathrm{d}E_x=\mathrm{d}E\cos\theta$，那么 $P$ 点的场强为：
$$
E=\int_L\mathrm{d}E_x=\frac{\cos\theta}{4\pi \varepsilon_0r^2}\int_L\mathrm{d}q=\frac{xq}{4\pi\varepsilon_0(x^2+R^2)^{\frac{3}{2}}}
$$
这里只是把 $\cos\theta$ 和 $r$ 用 $x, R$ 表示了而已，接下来分析圆盘。

## 圆盘

> 半径为 $R$，电荷均匀分布的薄圆盘的电荷面密度为 $\sigma$，求过盘心且垂直盘面的轴线上任意一点 $P$ 的场强。

<img src="/img/2023/07/physics-5-12.jpg" style="zoom:66%;" />

圆盘可以看作无数个圆环叠加形成的。取一个半径为 $r$ 宽度为 $\mathrm{d}r$ 的圆环，它的面积是 $2\pi r\mathrm{d}r$，这个面积元可以这么算：
$$
\mathrm{d}S=\pi(r+\mathrm{d}r)^2-\pi r^2=2\pi r\mathrm{d}r
$$
把 $(\mathrm{d}r)^2$ 忽略不计，也可以把这个圆环拉直看成长为 $2\pi r$，宽为 $\mathrm{d}r$ 的矩形。于是这个圆环对 $P$ 产生的场强为：
$$
\begin{aligned}
\mathrm{d}E&=\frac{x\mathrm{d}q}{4\pi\varepsilon_0(x^2+r^2)^{\frac{3}{2}}}\\
&=\frac{xr\sigma \mathrm{d}r}{2\varepsilon_0(x^2+r^2)^\frac{3}{2}}
\end{aligned}
$$
所有圆环对 $P$ 产生的场强方向都相同，那么圆盘在 $P$ 产生的场强为：
$$
\begin{aligned}
E&=\int_0^R\frac{xr\sigma \mathrm{d}r}{2\varepsilon_0(x^2+r^2)^\frac{3}{2}}\\
&=\frac{x\sigma}{2\varepsilon_0}\int_0^R\frac{r\mathrm{d}r}{(x^2+r^2)^\frac{3}{2}}\\
&=\frac{x\sigma}{2\varepsilon_0}\int_0^R\frac{\mathrm{d}(x^2+r^2)}{2(x^2+r^2)^\frac{3}{2}}\\
&=\frac{x\sigma}{2\varepsilon_0}(-\frac{1}{\sqrt{x^2+r^2}})\bigg|_0^R\\
&=\frac{\sigma}{2\varepsilon_0}(1-\frac{x}{\sqrt{x^2+R^2}})
\end{aligned}
$$
接下来考虑近似处理的方法。

## 近似处理

+ 当 $x \ll R$ 时，圆盘可以认为是无限大，那么 $E=\cfrac{\sigma}{2\varepsilon_0}$，这里就不讨论方向了，主要看下面的情况。

+ 当 $x\gg R$ 时，并不能像上面那样直接令 $x=0$ 的形式令 $R=0$，那么圆盘半径为 $0$ 求出来的场强自然也为 $0$，不符合我们的要求。这里用上面图中标的角 $\theta$ 进行推演：
    $$
    \begin{aligned}
    \sigma&=\frac{q}{\pi R^2}=\frac{q}{\pi x^2\tan^2\theta}\\
    E&=\frac{\sigma}{2\varepsilon_0}(1-\cos\theta)\\
    &=\frac{q}{2\pi\varepsilon_0 x^2}\frac{\cos^2\theta(1-\cos\theta)}{\sin^2\theta}\\
    &=\frac{q}{2\pi\varepsilon_0 x^2}\frac{\cos^2\theta}{1+\cos\theta}\\
    \end{aligned}
    $$
    由于 $x\gg R$，那么令 $\theta\to0$，得到：
    $$
    E=\frac{1}{4\pi\varepsilon_0}\frac{q}{x^2}
    $$
    所以此时能把圆盘看作集中于圆心的点电荷，能用点电荷的场强计算式。

