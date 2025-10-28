---
date: 2025-10-08 09:30:00
title: 数学 - 位运算卷积与 FWT
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 记录 FWT 的原理以及在具体题目中的应用。
---

## 原理

对于长度为 $2^n$ 的数组 $A_i,B_i$ 的卷积定义如下：
$$
C_k=\sum_{i\operatorname{op}j=k}A_iB_j
$$
这里的 $\operatorname{op}$ 可以是与、或、异或。

### 与卷积

我们想找到一个可逆变换 $F$，使得下面式子成立（无说明下面的加法和乘法都是逐位进行）：
$$
F(C)=F(A)F(B)
$$
并且我们希望他是线性的，即：
$$
F(A+B)=F(A)+F(B)
$$
这样就可以快速的计算了。

我们根据最高位是 $0/1$ 将数组 $A$ 分为 $A_0,A_1$，同理 $B,C$ 也是。

那么看原来的式子，由于只有 $1\operatorname{and}1=1$，其它的结果都是 $0$，因此：
$$
\begin{aligned}
C_{0k}&=\sum_{i\operatorname{and}j=k}A_{0i}B_{0j}+A_{0i}B_{1j}+A_{1i}B_{0j}\\
C_{1k}&=\sum_{i\operatorname{and}j=k}A_{1i}B_{1j}
\end{aligned}
$$
那么，对这两个式子用 $F$ 进行变换：
$$
\begin{aligned}
F(C_0)&=F(A_0)F(B_0)+F(A_0)F(B_1)+F(A_1)F(B_0)\\
F(C_1)&=F(A_1)F(B_1)
\end{aligned}
$$
两式相加可以得到：
$$
F(C_0+C_1)=F(A_0+A_1)F(B_0+B_1)
$$
所以，我们可以这样构造 $F(C)$：
$$
F(C)=\Big(F(C_0)+F(C_1),F(C_1)\Big)
$$
就可以使得前半部分和后半部分都满足：
$$
F(C)=F(A)F(B)
$$
那么，我们还需要研究一下这个变换的逆变换，这里上标 `*` 表示当前这个数组是变换后的数组：
$$
F^{-1}(C^*)=\Big(F^{-1}(C_0^*)-F^{-1}(C_1^*),F^{-1}(C_1^*)\Big)
$$
所以我们构造出了一种快速求与卷积的方式，即先进行 $F$ 变换，然后逐位相乘，最后通过逆变换 $F^{-1}$ 复原。

### 或卷积

同样的分类方法，我们可以将所有的数组分成两部分，然后：
$$
\begin{aligned}
C_{0k}&=\sum_{i\operatorname{or}j=k}A_{0i}B_{0j}\\
C_{1k}&=\sum_{i\operatorname{or}j=k}A_{0i}B_{1j}+A_{1i}B_{0j}+A_{1i}B_{1j}
\end{aligned}
$$
设满足要求的变换是 $G$，那么两边取 $G$ 可以得到：
$$
\begin{aligned}
G(C_0)&=G(A_0)G(B_0)\\
G(C_1)&=G(A_0)G(B_1)+G(A_1)G(B_0)+G(A_1)G(B_1)
\end{aligned}
$$
同上，可以构造出这样的变换：
$$
G(C)=\Big(G(C_0),G(C_0)+G(C_1)\Big)
$$
逆变换是：
$$
G^{-1}(C^*)=\Big(G^{-1}(C_0^*),G^{-1}(C_1^*)-G^{-1}(C_0^*)\Big)
$$

### 异或卷积

分类方法同上：
$$
\begin{aligned}
C_{0k}&=\sum_{i\oplus j=k}A_{0i}B_{0j}+A_{1i}B_{1j}\\
C_{1k}&=\sum_{i\oplus j=k}A_{0i}B_{1j}+A_{1i}B_{0j}
\end{aligned}
$$
设满足要求的变换是 $H$，那么可以得到：
$$
H(C_0)=H(A_0)H(B_0)+H(A_1)H(B_1)\\
H(C_1)=H(A_1)H(B_0)+H(A_0)H(B_1)
$$
两式相加相减：
$$
H(C_0+C_1)=H(A_0+A_1)H(B_0+B_1)\\
H(C_0-C_1)=H(A_0-A_1)H(B_0-B_1)
$$
构造变换：
$$
H(C)=\Big(H(C_0)+H(C_1),H(C_0)-H(C_1)\Big)
$$
因此逆变换为：
$$
H^{-1}(C^*)=\frac{1}{2}\Big(H^{-1}(C_0^*)+H^{-1}(C_1^*),H^{-1}(C_0^*)-H^{-1}(C_1^*)\Big)
$$
对于上面的所有变换，最终数组长度为 $1$ 时，保留其为那个数字本身即可，因为这三个卷积对于只有一个数字的情况都是普通的乘法。

## Binary Table

> 给定 $n\le 20$ 行 $m\le 10^5$ 列表格，每个元素是 $0/1$，每次操作选择一行或一列翻转，问经过若干次操作后表格中最少有多少个 $1$。

首先，翻转等价于 $\oplus 1$。而异或具有交换率，所以我们可以认为先进行行操作，最后统一进行列操作。

将第 $i$ 列看作一个二进制数 $v_i$，当行操作结束时，相当于给每一列选择了一个 $0\le x\le 2^n-1$ 使得 $v_i\gets v_i\oplus x$。

由于最后进行列操作，所以每一列的贡献 $f_v=\min\{\operatorname{popcount}v,n-\operatorname{popcount}v\}$。

枚举所有的 $x$，最后的结果就是：
$$
\min_{x=0}^{2^n-1}\left\{\sum_{i=1}^m f_{v_i\oplus x}\right\}
$$
一个常见的套路是将枚举每一个 $v_i$ 转化为枚举 $v_i$ 的个数，设 $g_v=\sum_{i=1}^m [v_i=v]$，那么目标式转化为：
$$
\min_{x=0}^{2^n-1}\left\{\sum_{v=0}^{2^n-1} f_{v\oplus x}g_v\right\}=\min_{x=0}^{2^n-1}\left\{\sum_{i\oplus j=x} f_ig_j\right\}
$$
因此求一个 $f$ 与 $g$ 的异或卷积求参数的最小值就是答案。

## Little Pony and Elements of Harmony

> 给定 $m\le 20,t\le 10^{18},p\le 10^9$，令 $n=2^m$，给定长度为 $n$ 的数组 $e_0[-]$ 和长度为 $m+1$ 的数组 $b_0\sim b_m$，定义 $e_i$ 与 $e_{i-1}$ 的递推式：
> $$
> e_i[u]=\sum_{v}e_{i-1}[v]b[\operatorname{popcount}(u\oplus v)]
> $$
> 求 $e_t[-]\bmod p$。注意 $p$ 不保证是一个质数。

显然，我们令 $f[x]=b[\operatorname{popcount} x]$，就会有：
$$
\begin{aligned}
e_i[u]&=\sum_{v}e_{i-1}[v]f[u\oplus v]\\
&=\sum_{x\oplus y=u} e_{i-1}[x]f[y]
\end{aligned}
$$
那么 $e_i=e_{i-1} * f$，则 $e_t=e_0 *f^t$。两边取一个 FWT 就会得到：
$$
F(e_t)=F(e_0)[F(f)^t]
$$
这里的次方是逐位求次方。

还有一个问题，就是 $p$ 不一定是一个质数，那么 $2$ 的逆元就不一定存在。这里修改一下逆变换的过程，设答案数组是 $a$，如果逆变换过程中不进行除 $2$ 的操作，那么最终得到的数组就会是 $2^m a_i$。只需要将模数预先乘 $2^m$，设我们得到的结果数组是 $x$，就会有：
$$
\begin{aligned}
2^m{a_i}&\equiv x_i \pmod { 2^mp}\\
a_i&\equiv \frac{x_i}{2^m}\pmod {p}
\end{aligned}
$$
这里 $2^m p$ 比较大，乘法需要用到 `int128`。

