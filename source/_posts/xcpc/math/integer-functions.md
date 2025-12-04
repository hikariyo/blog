---
date: 2025-12-04 22:04:00
title: 数学 - 整值函数
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 《具体数学》第三章整值函数在竞赛视角下的阅读笔记。
---

## 关键概念性质

### 不等式去掉取整符号

对于实数 $x$ 和整数 $n$，有些时候我们可以去掉取整符号：

1. $x<n\Leftrightarrow \lfloor x\rfloor <n$；
2. $x>n\Leftrightarrow \lceil x\rceil>n$；
3. $x\le n\Leftrightarrow \lceil x\rceil\le n$；
4. $x\ge n\Leftrightarrow \lfloor x\rfloor\ge n$。

这些不等式的证明，我们只需要考虑 $x$ 在 $n-1,n,n+1$ 这些点之间的情况即可，容易验证。

### 函数自变量加取整符号

若 $f(x)$ 是连续并且单调递增的，并且如果 $f(x)\in \mathbb Z$ 可以推出 $x\in \mathbb Z$，那么我们就有：

1. $\lfloor f(x)\rfloor=\lfloor f(\lfloor x\rfloor)\rfloor$；
2. $\lceil f(x)\rceil=\lceil f(\lceil x\rceil)\rceil$。

如果 $x$ 是整数，那么这个等式显然成立，接下来我们考虑 $x$ 不是整数。

我们首先证明第一条，由于 $\lfloor x\rfloor< x$，那么 $f(\lfloor x\rfloor)\le f(x)$，于是我们有 $\lfloor f(\lfloor x\rfloor)\rfloor\le \lfloor f(x)\rfloor$。

假设 $\lfloor f(\lfloor x\rfloor)\rfloor <\lfloor f(x)\rfloor$，由不等式的性质，我们可以得到 $f(\lfloor x\rfloor)<\lfloor f(x)\rfloor$。

由于 $f(x)\in \mathbb Z$ 可以推出 $x\in \mathbb Z$，所以 $f(x)\neq \lfloor f(x)\rfloor$，所以 $f(x)>\lfloor f(x)\rfloor$。

因此我们得到了 $f(\lfloor x\rfloor)$ 与 $f(x)$ 之间夹了一个整数，由于 $f(x)$ 是连续的，所以一定存在一个 $\lfloor x\rfloor <y<x$ 使得 $f(y)=\lfloor f(x)\rfloor$。

此时可以推出 $y$ 是整数，然而 $\lfloor x\rfloor$ 与 $x$ 之间并不存在整数，所以假设不成立，因此 $\lfloor f(\lfloor x\rfloor)\rfloor =\lfloor f(x)\rfloor$。

第二条的证明过程同上。

### 区间整数点数量

对于下面三种类型的区间，如果有 $\alpha\le \beta$，我们可以用给不等式加取整符号的方法，求出区间内的整数点数量。

1. 对区间 $[\alpha,\beta]$ 来说，$\alpha\le n\le \beta$ 会有 $\lceil\alpha\rceil \le n\le\lfloor\beta\rfloor$，于是数量是 $\lfloor\beta\rfloor-\lceil\alpha\rceil+1$；
2. 对区间 $[\alpha,\beta)$ 来说，$\alpha\le n\lt \beta$ 会有 $\lceil\alpha\rceil\le n<\lceil\beta\rceil$，于是数量是 $\lceil\beta\rceil-\lceil\alpha\rceil$；
3. 对区间 $(\alpha,\beta]$ 来说，$\alpha<n\le \beta$ 会有 $\lfloor\alpha\rfloor<n\le \lfloor\beta\rfloor$，于是数量是 $\lfloor\beta\rfloor-\lfloor\alpha\rfloor$；

对这三种情况，如果对应区间之内确实至少存在一个整数，那么由于这个不等式转化是充要的，所以它给出的答案一定也是正确的，所以我们只需要确定，当对应区间不存在整数时，公式给出的答案是不是 $0$。

1. 对区间 $[\alpha,\beta]$ 来说，$\alpha=n+\varepsilon_1,\beta=n+\varepsilon_2,0\lt\varepsilon_1\le\varepsilon_2\lt1$；
2. 对区间 $[\alpha,\beta)$ 来说，$\alpha=n+\varepsilon_1,\beta=n+\varepsilon_2,0\lt \varepsilon_1\le\varepsilon_2\le 1$；
3. 对区间 $(\alpha,\beta]$ 来说，$\alpha=n+\varepsilon_1,\beta=n+\varepsilon_2,0\le \varepsilon_1\le\varepsilon_2\lt 1$；

这些情况就是所有答案应当为 $0$ 的情况，可以验证对应公式给出的都是 $0$，因此这个公式是正确的。

### 推论

1. 对于 $f(x)=\sqrt x$ 符合要求，有 $\lfloor\sqrt{x}\rfloor=\lfloor \sqrt{\lfloor x\rfloor}\rfloor$。

2. 对于 $f(x)=\cfrac{x+m}{n}$ 符合要求，有 $\lfloor\cfrac{\lfloor x\rfloor +m}{n}\rfloor=\lfloor\cfrac{x+m}{n}\rfloor$。

   特别地，令 $m=0$，我们就有 $\lfloor\cfrac{x}{kn}\rfloor=\lfloor\frac{\lfloor x/k\rfloor}{n}\rfloor$。

## 例题

### 例1

求 $\sum_{n=1}^N[\lfloor\sqrt[3]{n}\rfloor\mid n]$。

设 $K=\lfloor\sqrt[3]{N}\rfloor$，枚举 $k=\lfloor\sqrt[3]{n}\rfloor$，当 $k<K$ 时，符合条件的 $n\le N$；当 $k=K$ 时，符合条件的 $n$ 可能会超过 $N$。

因此，我们有：
$$
\begin{aligned}
\sum_{n=1}^N [\lfloor\sqrt[3]{n}\rfloor\mid n]
&=\sum_{n=1}^{K^3-1}[\lfloor\sqrt[3]{n}\rfloor\mid n]+\sum_{n=K^3}^N [K\mid n]\\
&=\sum_{n,k}[k=\lfloor\sqrt[3]{n}\rfloor][k\mid n][1\le n<K^3]+\sum_{n,m}[n=Km][K^3\le n\le N]\\
&=\sum_{n,k,m}[k\le \sqrt[3]{n}<k+1][n=km][1\le n<K^3]+\sum_{m}[K^3\le Km\le N]\\
&=\sum_{n,k,m}[k\le \sqrt[3]{n}<k+1][n=km][1\le k<K]+\lfloor\frac{N}{K}\rfloor-K^2+1\\
&=\sum_{k,m}[k^3\le km<(k+1)^3][1\le k<K]+\lfloor\frac{N}{K}\rfloor-K^2+1\\
&=\sum_{k,m}[k^2\le m<\frac{(k+1)^3}{k}][1\le k<K]+\lfloor\frac{N}{K}\rfloor-K^2+1\\
&=\sum_{k=1}^{K-1}(3k+4)+\lfloor\frac{N}{K}\rfloor-K^2+1\\
&=\frac{7+3K+1}{2}(K-1)+\lfloor\frac{N}{K}\rfloor-K^2+1\\
&=\lfloor\frac{N}{K}\rfloor+\frac{1}{2}K^2+\frac{5}{2}K-3
\end{aligned}
$$
注意，中间把 $1\le n<K^3$ 换成 $1\le k<K$ 这一步比较关键，这是因为我们固定 $k$ 后遍历符合条件的 $n$，仍然可以遍历到 $[1,K^3)$ 之间的所有 $n$。

其次，我们写 $\sum_{n,k,m}$ 的意思是 $n,k,m$ 分别独立地遍历所有非负整数。

### 例2

求 $\sum_{1\le k<n}\lfloor\sqrt{k}\rfloor$。

同样地设 $a=\lfloor\sqrt{n}\rfloor$，那么：
$$
\begin{aligned}
\sum_{1\le k<n}\lfloor\sqrt{k}\rfloor&=\sum_{m,k}m[m=\lfloor\sqrt{k}\rfloor][k<a^2]+\sum_{a^2\le k<n}\lfloor\sqrt{k}\rfloor\\
&=\sum_{m,k}m[m\le\sqrt{k}<m+1][k<a^2]+(n-a^2)a\\
&=\sum_{m,k}m[m^2\le k<(m+1)^2][m<a]+(n-a^2)a\\
&=\sum_{m}m(2m+1)[m<a]+(n-a^2)a\\
&=\sum_{m}(2m^{\underline{2}}+3m^{\underline{1}})[m<a]+(n-a^2)a\\
&=\frac{2}{3}a^{\underline{3}}+\frac{3}{2}a^{\underline{2}}+(n-a^2)a\\
&=na-\frac{1}{3}a^3-\frac{1}{2}a^2-\frac{1}{6}a
\end{aligned}
$$
其中 $x^{\underline{n}}$ 代表 $n$ 次下降幂，我们有 $x^{\underline{n}}=\cfrac{(x+1)^{\underline{n+1}}-x^{\underline{n+1}}}{n+1}$，这个形式及其有利于求和。

### 例3

证明：$n=\sum_{k=0}^{m-1}\lceil\cfrac{n-k}{m}\rceil$，当 $n\ge m$ 且 $n,m$ 为正整数。

不妨做带余除法 $n=qm+r$，对 $k$ 分类讨论：

1. 当 $k\le r-1$ 时，有 $\lceil\cfrac{n-k}{m}\rceil=q+1$；
2. 当 $r\le k\le m-1$ 时，我们有 $(q-1)m+1\le qm+r-m+1\le qm+r-k\le qm$，因此 $\lceil\cfrac{n-k}{m}\rceil=q$。

所以，右边的和式就是 $(q+1)r+q(m-r)=qm+r=n$。

根据 $\lceil\cfrac{n}{m}\rceil=\lfloor\cfrac{n+m-1}{m}\rfloor$，可以得到 $n=\sum_{k=0}^{m-1}\lfloor\cfrac{n+k}{m}\rfloor$。

我们令 $n=\lfloor mx\rfloor$，可以得到 $\lfloor mx\rfloor=\sum_{k=0}^{m-1}\lfloor\cfrac{\lfloor mx\rfloor +k}{m}\rfloor=\sum_{k=0}^{m-1}\lfloor x+\cfrac{k}{m}\rfloor$。

### 例4

求 $\sum_{1<k<2^{2^n}}\cfrac{1}{2^{\lfloor \lg k\rfloor}4^{\lfloor\lg\lg k\rfloor}}$。

注意这里的 $\lg$ 底数是 $2$。

由于 $\lg x\in\mathbb Z$ 可以推出 $x\in\mathbb Z$，因此 $\lfloor\lg x\rfloor=\lfloor \lg \lfloor x\rfloor\rfloor$。
$$
\begin{aligned}
\sum_{1<k<2^{2^n}}\cfrac{1}{2^{\lfloor \lg k\rfloor}4^{\lfloor\lg\lg k\rfloor}}&=\sum_{1<k<2^{2^n}}\cfrac{1}{2^{\lfloor \lg k\rfloor}4^{\lfloor\lg\lfloor\lg k\rfloor\rfloor}}\\
&=\sum_{k,m}\frac{1}{2^m 4^{\lfloor \lg m\rfloor}}[m=\lfloor\lg k\rfloor][1<k<2^{2^n}]\\
&=\sum_{k,m}\frac{1}{2^m4^{\lfloor\lg m\rfloor}}[m\le \lg k<m+1][1\le m<2^n]\\
&=\sum_{k,m}\frac{1}{2^m4^{\lfloor\lg m\rfloor}}[2^m\le k<2^{m+1}][1\le m<2^n]\\
&=\sum_{m}\frac{2^{m+1}-2^m}{2^m4^{\lfloor\lg m\rfloor}}[1\le m<2^n]\\
&=\sum_{m}\frac{1}{4^{\lfloor\lg m\rfloor}}[1\le m<2^n]\\
&=\sum_{m,p}\frac{1}{4^p}[p=\lfloor\lg m\rfloor][1\le m<2^n]\\
&=\sum_{m,p}\frac{1}{4^p}[2^p\le m<2^{p+1}][0\le p<n]\\
&=\sum_{p}\frac{2^{p+1}-2^p}{4^p}[0\le p<n]\\
&=\sum_{p}\frac{1}{2^p}[0\le p<n]\\
&=2-\frac{1}{2^{n-1}}
\end{aligned}
$$
