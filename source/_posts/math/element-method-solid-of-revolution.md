---
title: 微积分 - 推导旋转体相关公式
date: 2023-05-06 14:54:36.853
updated: 2023-05-06 17:06:06.27
katex: true
categories:
- Learn
tags:
- Calculus
- Math
---

## 弧微分

这里简要给出过程，虽然不是特别严谨，但理解起来容易就好。

![](/img/2023/05/arc-derivative.jpg)

对于上图可以得出：

$$
\begin{aligned}
\frac{\Delta s}{\Delta x} &= \frac{|\overgroup{AB}|}{\Delta x}\\
&=\sqrt{\frac{|\overgroup{AB}|^2}{|AB|^2}\cdot\frac{|AB|^2}{(\Delta x)^2}}\\
&=\sqrt{\frac{|\overgroup{AB}|^2}{|AB|^2}\cdot\frac{(\Delta x)^2+(\Delta y)^2}{(\Delta x)^2}}\\
&=\sqrt{\frac{|\overgroup{AB}|^2}{|AB|^2}\cdot[1+(\frac{\Delta y}{\Delta x})^2]}\\
\end{aligned}
$$

令 $\Delta x \to 0$ 此时 $|\overgroup{AB}|$ 与 $|AB|$ 之比应当为 1，那么简单整理后可得到**弧微分公式**：

$$
\mathrm{d}s=\sqrt{1+y'^2}\mathrm{d}x
$$

## 圆锥

先来一个最简单的试试手。

![](/img/2023/05/cone.jpg)

### 体积

将其分割为无限多个圆柱，分别求出体积，然后用定积分可以求得公式：

$$
\begin{aligned}
\mathrm{d}V&=\pi y^2\mathrm{d}x\\
&=\pi\frac{R^2}{h^2}x^2\mathrm{d}x \\
V&=\int_0^h\pi\frac{R^2}{h^2}x^2\mathrm{d}x\\
&=\pi\frac{R^2}{h^2}\cdot\frac{1}{3}x^3\bigg|_0^h\\
&=\frac{1}{3}\pi R^2h
\end{aligned}
$$

### 侧面积

对于侧面积，这里注意要使用 $\mathrm{d}s$ 作为圆柱的高，而不能使用 $\mathrm{d}x$，如果用 $\mathrm{d}x$ 最后积出的结果是一个圆柱而不是圆锥：

$$
\begin{aligned}
\mathrm{d}S&=2\pi y\mathrm{d}s\\
&=2\pi \frac{R}{h} \sqrt{1+(\frac{R}{h})^2}x\mathrm{d}x\\
&=2\pi \frac{R}{h^2}lxdx\\
S&=\int_0^h2\pi \frac{R}{h^2}lxdx\\
&=2\pi\frac{R}{h^2}l\cdot \frac{1}{2}x^2\bigg|_0^h\\
&=\pi Rl
\end{aligned}
$$

## 圆台

和圆锥的推导过程类似。

![](/img/2023/05/truncated-cone.jpg)

### 体积

$$
\begin{aligned}
\mathrm{d}V&=\pi y^2\mathrm{d}x\\
&=\pi[\frac{(R-r)^2}{h^2}x^2+\frac{2(R-r)r}{h}x+r^2]\mathrm{d}x \\
&=\pi(\frac{R^2-2Rr+r^2}{h^2}x^2+\frac{2Rr-2r^2}{h}x+r^2)dx\\

V&=\int_0^h\pi(\frac{R^2-2Rr+r^2}{h^2}x^2+\frac{2Rr-2r^2}{h}x+r^2)dx\\
&=\pi(\frac{R^2-2Rr+r^2}{3h^2}x^3+\frac{Rr-r^2}{h}x^2+r^2x)\bigg|_0^h\\
&=\pi h(\frac{R^2-2Rr+r^2}{3}+Rr-r^2+r^2)\\
&=\frac{1}{3}\pi h(R^2+Rr+r^2)
\end{aligned}
$$

### 侧面积

同样地，要使用弧微分：

$$
\begin{aligned}
\mathrm{d}S&=2\pi y\rm{d}s\\
&=2\pi(\frac{R-r}{h}x+r)\sqrt{1+(\frac{R-r}{h})^2}dx\\
&=2\pi\frac{l}{h}(\frac{R-r}{h}x+r)dx\\
S&=\int_0^h2\pi\frac{l}{h}(\frac{R-r}{h}x+r)dx\\
&=2\pi\frac{l}{h}(\frac{R-r}{2h}x^2+rx)\bigg|_0^h\\
&=\pi l(R+r)
\end{aligned}
$$

## 球

这里给出两种方法，可以用参数方程最终对 $\theta$ 积分，也可以对 $x$ 积分，权当练习。

![](/img/2023/05/circle.jpg)

写出圆的参数方程：

$$
\left\{\begin{array}{lr}
x=R\cos\theta\\
y=R\sin\theta
\end{array}
\right.
$$

由于 $x \in [-R, R]$ 则 $\theta \in [0, \pi]$ 且由参数方程求导法则有：

$$
y'=\frac{\mathrm{d}y}{\mathrm{d}x}=\frac{\frac{\mathrm{d}y}{\mathrm{d}\theta}}{\frac{\mathrm{d}x}{\mathrm{d}\theta}}=-\frac{1}{\tan\theta}
$$

### 体积

可以不用参数方程，直接求：

$$
\begin{aligned}
\mathrm{d}V&=\pi y^2\mathrm{d}x\\
&=\pi(R^2-x^2)\mathrm{d}x\\
V&=\int_{-R}^R\pi(R^2-x^2)\mathrm{d}x\\
&=\pi(R^2x-\frac{1}{3}x^3)\bigg|_{-R}^R\\
&=\frac{4}{3}\pi R^3
\end{aligned}
$$

也可以用参数方程求，这里用到了**换元积分法**，而且对 $\theta$ 积分时应当是从 $\pi$ 到 $0$，对应 $x$ 从 $-R$ 到 $R$：

$$
\begin{aligned}
\mathrm{d}V&=\pi y^2\mathrm{d}x\\
&=-\pi R^3\sin^3\theta\mathrm{d}\theta\\
V&=\int_\pi^0(-\pi R^3\sin^3\theta)\mathrm{d}\theta\\
&=\pi R^3\int_0^\pi\sin^3\theta\mathrm{d}\theta\\
&=\pi R^3\int_0^\pi \sin\theta(1-\cos^2\theta)\mathrm{d}\theta\\
&=\pi R^3\int_0^\pi(\cos^2\theta-1)\mathrm{d}(\cos \theta)\\
&=\pi R^3(\frac{1}{3}\cos^3\theta-\cos\theta)\bigg|_0^\pi\\
&=\frac{4}{3}\pi R^3
\end{aligned}
$$

### 表面积

虽然第一个方法不对 $\theta$ 积分，但还是借用了一下参数方程列关系式：

$$
\begin{aligned}
\mathrm{d}S&=2\pi y\mathrm{d}s\\
&=2\pi R\sin\theta\sqrt{1+\frac{1}{\tan^2\theta}}\mathrm{d}x\\
&=2\pi R\mathrm{d}x\\
S&=\int_{-R}^R2\pi R\mathrm{d}x\\
&=4\pi R^2
\end{aligned}
$$

参数方程，也是从 $\pi$ 到 $0$ 积：

$$
\begin{aligned}
\mathrm{d}S&=2\pi y\mathrm{d}s\\
&=2\pi R\sin\theta\sqrt{1+\frac{1}{\tan^2\theta}}\mathrm{d}x\\
&=-2\pi R^2\sin\theta\mathrm{d}\theta\\
S&=\int_{\pi}^{0}(-2\pi R^2\sin\theta)\mathrm{d}\theta\\
&=2\pi R^2\int_{0}^{\pi}\sin\theta\mathrm{d}\theta\\
&=-2\pi R^2\cos\theta\bigg|_0^{\pi}\\
&=4\pi R^2
\end{aligned}
$$