---
date: 2023-07-08 00:39:00
title: 中学物理的一些边角料
katex: true
tags:
- Math
- Calculus
categories:
- Learn
---

## 简介

如标题，本文主要记录中学物理中的一些不那么重要但是又确实会用到的边角料知识。有些需要微积分，有些不需要。

## 测电阻实验的电流表内外接误差分析

设待测电阻为 $R$，电流表电压表内阻为 $R_A,R_V$ 并且 $R_A\ll R\ll R_V$，下面分析为什么将 $R$ 与 $\sqrt{R_AR_V}$ 比较。

### 内接法

对于内接法，应当电流是准确，电压包含了电流表的电压，所以：
$$
R'=\frac{U+U_A}{I}=R+R_A
$$
因此相对误差为：
$$
\delta_1=\frac{R_A}{R}
$$

### 外接法

对于外接法，应当是电压准确，电流包含了电压表的电流，所以：
$$
\begin{aligned}
R'&=\frac{U}{I+I_V}\\
&=\frac{1}{\frac{I}{U}+\frac{I_V}{U}}\\
&=\frac{R_VR}{R_V+R}
\end{aligned}
$$
因此相对误差为：
$$
\delta_2=\frac{|R-R'|}{R}=\frac{R}{R_V+R}
$$

### 比较

做差比较即可：
$$
\delta_1-\delta_2=\frac{R_AR_V+R_AR-R^2}{R(R_V+R)}
$$
认为 $R_AR\approx 0$，这样 $\delta_1=\delta_2$ 的符号取决于 $R_AR_V-R^2$ 的符号，所以当 $R>\sqrt{R_AR_V}$ 时，$\delta_1<\delta_2$，使用内接法，反之亦然。

并且，显然有内接时这个系统误差导致测量值偏大，外接偏小。

## 圆盘电荷的近似处理

### 圆环

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

### 圆盘

> 半径为 $R$，电荷均匀分布的薄圆盘的电荷面密度为 $\sigma$，求过盘心且垂直盘面的轴线上任意一点 $P$ 的场强。

<img src="/img/2023/07/physics-5-12.jpg" style="zoom:66%;" />

圆盘可以看作无数个圆环叠加形成的。取一个半径为 $r$ 宽度为 $\mathrm{d}r$ 的圆环，它的面积是 $2\pi r\mathrm{d}r$，这个面积元可以这么算：
$$
\Delta S=\pi(r+\Delta r)^2-\pi r^2=2\pi r\Delta r+\pi (\Delta r)^2
$$
把 $(\Delta r)^2$ 忽略不计，也可以把这个圆环拉直看成长为 $2\pi r$，宽为 $\mathrm{d}r$ 的矩形。就有 $\mathrm{d}S=2\pi r\mathrm{d}r$，于是这个圆环对 $P$ 产生的场强为：
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

## 折射前后的光速变化

对于折射率为 $n_1,n_2$ 两介质中的光速，满足 $v_1n_1=v_2n_2$，特别地，对于 $n_1=1$ 即真空时 $v_1=c$，那么 $v_2=c/n_2$。

只要认为 “光会沿着时间花费时间最短的路径” 传播，即最短时间原理，那么就可以推导出这一公式。

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
于是化简得到：
$$
\frac{\sin\theta_1}{v_1}=\frac{\sin\theta_2}{v_2}
$$
根据折射率的定义：
$$
n_1\sin\theta_1=n_2\sin\theta_2
$$
联立就可以推导出 $n_1v_1=n_2v_2$ 了。
