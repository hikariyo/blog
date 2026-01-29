---
date: 2026-01-29 23:11:00
updated: 2026-01-29 23:11:00
title: 数学 - 异或线性基
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 在有限域 F2 上线性空间的视角下学习异或线性基。
---

## 代数结构

这一部分介绍群、环和域这三种特殊的代数结构，本文主要内容是先讨论域 $F$ 上的线性空间的一般性质，然后代入 $F=\mathbb{F}_2$ 讨论异或线性基特殊的性质。

首先给出一堆定义。

1. 设 $A,B$ 是两个集合，我们把集合
   $$
   A\times B=\{(a,b)\mid a\in A,b\in B\}
   $$
   叫做 $A$ 和 $B$ 的**直积**。

2. 设 $A$ 是集合，从 $A\times A$ 到 $A$ 的映射
   $$
   f:A\times A\to A
   $$
   叫做集合 $A$ 上的一个（二元）**运算**。

3. 运算 $\circ$ 满足**结合律**，是指
   $$
   a\circ(b\circ c) = (a\circ b)\circ c(\forall a,b,c\in A)
   $$

4. 运算 $\circ$ 满足**交换律**，是指
   $$
   a\circ b=b\circ a(\forall a,b\in A)
   $$

5. 集合 $G$ 和 $G$ 上的二元运算 $\circ$ 构成一个**群**，如果

   1. 运算 $\circ$ 满足结合律；
   2. 存在幺元素 $1\in G$，使得对每个 $x\in G$，$x1=1x=x$；
   3. 对每个元素 $x\in G$，都存在 $y\in G$，$xy=yx=1$。

   如果二元运算 $\circ$ 还具有交换律，称 $G$ 为**阿贝尔群**。

   如果仅满足第一个条件，称 $G$ 为**半群**。

   如果满足前两个条件，称 $G$ 为**含幺半群**。

6. 集合 $R$ 和 $R$ 上两个二元运算组成的代数结构 $(R,+,\circ)$ 构成一个**环**，如果

   1. $(R,+)$ 是阿贝尔群，这个加法群的幺元素记作 $0$，叫做环 $R$ 的零元素；对每个元素 $a\in R$ 的加法逆元记作 $-a$；对于正整数 $n$，记 $n$ 个 $a$ 相加为 $na$；
   2. $(R,\circ)$ 是半群，对于正整数 $n$ 和每个元素 $a\in R$，记 $n$ 个 $a$ 相乘为 $a^n$；如果 $a$ 有逆元，将它的逆元记作 $a^{-1}$；
   3. 加法和乘法满足分配律，即对任意的 $a,b,c\in R$，有 $a(b+c)=ab+ac, (b+c)a=ba+ca$。

   如果 $R$ 上的乘法具有交换律，那么称 $R$ 为**交换环**；

   如果存在元素 $1\in R$，使得 $\forall a\in R, 1a=a1=a$，那么称 $R$ 为**含幺环**，元素 $1$ 叫做 $R$ 的**幺元素**。

7. 设 $R$ 为含幺交换环，并且 $0\neq 1$，并且每个非零元素均有乘法逆，那么称 $R$ 为**域**。

下面我们来根据这些定义证明一些简单的性质。

1. 含幺半群 $G$ 中的幺元素是唯一的。

   证：设 $1,1'\in G$ 均是幺元素，那么 $11'=1=1'$。

2. 群 $G$ 中每个元素的逆元是唯一的。

   证：任取 $x\in G$，若 $y,y'\in G$ 均是 $x$ 的逆元，则有 $y'=y'1=y'(xy)=(y'x)y=y$。

3. 环 $R$ 中任意元 $a$ 满足 $0a=a0=0$。

   证：$0a=(0+0)a=0a+0a$，则 $0a=0$，同理 $a0=0$。

4. 环 $R$ 中任意元素 $a,b$ 满足 $(-a)b=a(-b)=-(ab)$，$(-a)(-b)=ab$。

   证：$ab+(-a)b=(a-a)b=0$，$a(-b)+ab=a(-b+b)=0$，证明了第一个式子；$(-a)(-b)=-(a(-b))=-(-(ab))=ab$，证明了第二个式子。

5. 环 $R$ 中任意元素 $a_i,b_j$ 满足 $\left(\sum_{i=1}^n a_i\right)\left(\sum_{j=1}^m b_j\right)=\sum_{i=1}^n\sum_{j=1}^m a_ib_j$。

   证：利用数学归纳法，可以将分配律推广成
   $$
   \left(\sum_{i=1}^n a_i\right) x= \sum_{i=1}^n a_ix,y\left(\sum_{j=1}^m b_j\right)=\sum_{j=1}^m yb_j
   $$
   利用这个结论即可证明原式。

6. 环 $R$ 中任意元素 $a,b$ 与正整数 $n$ 满足 $(na)b=a(nb)=n(ab)$。

   证：$(na)b=(\sum_{i=1}^n a)b=\sum_{i=1}^n ab=n(ab)$，$a(nb)=a(\sum_{i=1}^n b)=\sum_{i=1}^n ab=n(ab)$。

## 线性空间

我们学习的线性空间一般是域 $(\mathbb{C},+,\times)$ 上的线性空间，下面我们来讨论一下一般的域 $F$ 下的线性空间。

### 定义

设 $V$ 是一个非空集合，$F$ 是一个域。如果有 $V\times V\to V$ 的**加法**（将 $(\alpha,\beta)\mapsto \gamma$ 记作 $\alpha+\beta=\gamma$）与 $F\times V\to V$ 的**纯量乘法**（将 $(k,\alpha)\mapsto \delta$ 记作 $k\alpha=\delta$），并且满足下述 $8$ 条运算法则：$\forall \alpha,\beta,\gamma\in V,\forall k,l\in F$ 有：

1. $\alpha+\beta=\beta+\alpha$；
2. $(\alpha+\beta)+\gamma=\alpha+(\beta+\gamma)$；
3. $V$ 中有一个元素，记作 $0$，使得 $\alpha+0=0$，具有该性质的元叫做 $V$ 的**零元**；
4. 对于 $\alpha\in V$，存在 $\beta\in V$，使得 $\alpha+\beta=0$，具有该性质的元叫做 $\alpha$ 的**负元**；
5. $1\alpha=\alpha$，其中 $1$ 是 $F$ 的幺元；
6. $(kl)\alpha=k(l\alpha)$；
7. $(k+l)\alpha=k\alpha+l\alpha$；
8. $k(\alpha+\beta)=k\alpha+k\beta$。

称 $V$ 是域 $F$ 上的一个**线性空间**。

域 $F$ 上所有 $n$ 元有序组组成的集合 $F^n$，对于有序组的加法（对应分量相加）与纯量乘法（把 $F$ 的元素 $k$ 乘每一个分量），成为域 $F$ 上的一个线性空间，称它为域 $F$ 上的 $n$ 维**向量空间**。

下面我们来根据 $8$ 条运算法则证明一些简单的性质。

1. $V$ 中零元是唯一的。

   证：设 $0_1,0_2$ 是 $V$ 中两个零元，则 $0_1+0_2=0_1=0_2$。

2. $V$ 中每个元素 $\alpha$ 的负元是唯一的。

   证：设 $\beta_1,\beta_2$ 是 $\alpha$ 的负元，则 $\beta_1=\beta_1+(\alpha+\beta_2)=(\beta_1+\alpha)+\beta_2=\beta_2$，今后记 $\alpha$ 的负元为 $-\alpha$，并定义减法 $\alpha-\beta:=\alpha+(-\beta)$。

3. 对于 $F$ 中的零元 $0$，$0\alpha=0,\forall \alpha\in V$，注意等号右边的 $0$ 是 $V$ 中的零元。

   证：$0\alpha=(0+0)\alpha=0\alpha+0\alpha$，则 $0\alpha=0$。

4. 对于 $0\in V$，$k0=0,\forall k\in F$。

   证：$k0=k(0+0)=k0+k0$，则 $k0=0$。

5. 如果 $k\alpha=0,k\in F,\alpha\in V$，那么 $k=0$ 或 $\alpha=0$。

   证：假设 $k\neq 0$，则 $\alpha=1\alpha=k^{-1}(k\alpha)=0$，于是 $k\neq 0$ 和 $\alpha\neq 0$ 不能同时成立。

6. 对于 $F$ 中的幺元 $1$，$(-1)\alpha=-\alpha,\forall \alpha\in V$。

   证：$\alpha+(-1)\alpha=1\alpha+(-1)\alpha=(1-1)\alpha=0\alpha=0$，于是 $(-1)\alpha=-\alpha$。

### 线性相关与线性无关

设 $V$ 是域 $F$ 上的一个线性空间，则称向量组 $\alpha_1,\alpha_2,\dots,\alpha_s$ **线性无关**，当且仅当 $k_1\alpha_1+k_2\alpha_2+\dots+k_s\alpha_s=0$ 可以推出 $k_1=k_2=\dots=k_s=0$；反之称为**线性相关**。

如果 $V$ 中的一个向量 $\beta$ 可以由向量组 $\alpha_1,\alpha_2,\dots,\alpha_s$ 线性表示，称 $\beta$ 可以由该向量组**线性表出**。

下面我们证明两个定理。

1. 设向量组 $\beta_1,\beta_2,\dots,\beta_r$ 中每一个向量都可以由向量组 $\alpha_1,\alpha_2,\dots,\alpha_s$ 线性表出，如果 $r>s$，那么 $\beta_1,\beta_2,\dots,\beta_r$ 线性相关。

   证：将 $\beta$ 由 $\alpha$ 线性表出，列 $\sum_{i=1}^rx_i\beta_i=0$，将每一个 $\alpha$ 的系数提取出来令其为 $0$，此时有 $s$ 个方程，$r$ 个未知数。由于这是一个齐次线性方程组，那么根据线性方程组的知识，$s<r$ 可以推出一定有非零解，于是 $\beta$ 线性相关。

2. 设向量组 $\alpha_1,\alpha_2,\dots,\alpha_s$ 线性无关，则 $\forall k\in F,\forall 1\le i\neq j\le s$，有 $\alpha_1,\alpha_2,\dots,\alpha_i+k\alpha_j,\dots,\alpha_s$ 线性无关。

   证：设 $k_1\alpha_1+k_2\alpha_2+\dots+(k_i\alpha_i+k_ik\alpha_j)+\dots+k_s\alpha_s=0$，可以推出 $k_1=0,k_2=0,\dots,k_i=0,k_j+k_ik=0,\dots,k_s=0$，推出 $k_i=k_j=0$。易证反之亦成立。

3. 设向量组 $\alpha_1,\alpha_2,\dots,\alpha_s$ 线性无关，若 $\beta$ 能被向量组 $\alpha$ 线性表出，则表出方式唯一。

   证：设有两种表出方式，移项容易解得这两种表出方式的系数是相等的。

下文中我们将主要讨论向量空间 $F^n$ 上的性质。

### 向量空间的子空间

$F^n$ 的一个非空子集 $U$ 如果满足：

1. 若 $\alpha,\beta\in U$，则 $\alpha+\beta\in U$；
2. 若 $\alpha\in U$，$k\in K$，则 $k\alpha\in U$。

那么称 $U$ 是 $F^n$ 的**子空间**。

容易验证，$F^n$ 中向量组 $\alpha_1,\alpha_2,\dots,\alpha_s$ 的所有线性组合组成的集合 $W$ 是 $F^n$ 的一个子空间，将其称为该向量组**张成的子空间**，记作 $\left<\alpha_1,\alpha_2,\dots,\alpha_s\right>$。

### 向量空间及其子空间的基

设 $U$ 是 $F^n$ 的一个子空间，如果 $\alpha_1,\alpha_2,\dots,\alpha_r\in U$ 并且满足：

1. $\alpha_1,\alpha_2,\dots,\alpha_r$ 线性无关；
2. $U$ 中每一个向量都可以由 $\alpha_1,\alpha_2,\dots,\alpha_r$ 线性表出；

那么称 $\alpha_1,\alpha_2,\dots,\alpha_r$ 是 $U$ 的一个基。

显然，$\varepsilon_1=(1,0,\dots,0),\varepsilon_2=(0,1,\dots,0),\dots,\varepsilon_n=(0,0,\dots,1)$ 是 $F^n$ 的一个基，称它为**标准基**。

下面我们证明几个定理。

1. $F^n$ 的任一非零子空间 $U$ 都有一个基。

   这个定理的证明是构造性的。

   先取 $0\neq \alpha_1 \in U$，若 $\left<\alpha_1\right>=U$，那么 $\alpha_1$ 是一组基。否则，取 $\alpha_2\not\in\left<\alpha_1\right>$，若 $\left<\alpha_1,\alpha_2\right>=U$ 那么 $\alpha_1,\alpha_2$ 是一组基。

   这样执行下去，每一个向量都不能被前面的向量线性表出，据此容易证明得到的向量组是线性无关的。

   并且这样最多会执行 $n$ 步。

   反证法：假设执行了 $n+1$ 步，因为标准基的大小为 $n$，那么 $n+1$ 个向量可以被标准基表出，根据"线性相关与线性无关"下的定理 1，这 $n+1$ 个向量线性相关。又因为前 $n$ 个向量线性无关，容易得到 $\alpha_{n+1}$ 一定可以被前面的 $n$ 个向量表出，这违背了取向量的方法。

2. $F^n$ 的非零子空间 $U$ 的任意两个基所含向量个数相等。

   任意两个基都是线性无关，根据"线性相关与线性无关"下的定理 1 的逆否命题，可以得到这两个基所含向量个数相等。将基所含向量的个数记为 $\dim U$。

3. 设 $\dim U=r$，则 $U$ 中任意 $r+1$ 个向量都线性无关。

   任取一组基 $\alpha_1,\alpha_2,\dots,\alpha_r$，再任取 $r+1$ 个向量 $\beta_1,\beta_2,\dots,\beta_{r+1}$。由于向量组 $\beta$ 可以被向量组 $\alpha$ 线性表出，且 $r+1>r$，推出向量组 $\beta$ 线性相关。

4. 设 $\dim U=r$，则 $U$ 中任意 $r$ 个线性无关的向量都是 $U$ 的一个基。

   任取 $r$ 个线性无关的向量，再取一个向量 $\alpha$，根据定理 3 可以推出 $\alpha$ 可以被之前取的 $r$ 个向量线性表出，推出任取 $r$ 个线性无关的向量都构成 $U$ 的一个基。

因此，如果给定了 $s$ 个向量 $\alpha_1,\alpha_2,\dots,\alpha_s$，并且 $U=\left<\alpha_1,\alpha_2,\dots,\alpha_s\right>$，我们只需要类比定理 1 的构造性证明，就能构造出一个基。

## 异或线性基

算法竞赛中，我们可以将 `uint64` 的异或运算看成 $\mathbb{F}_2^{64}$ 的加法运算，于是异或线形基就指 $\mathbb{F}_2$ 上的向量空间的某一子空间的基，其中 $\mathbb{F}_2=(\mathbb{Z}_2,+,\times)$。

接下来介绍线性基的贪心构造法，下面设 $0\neq \alpha_1,\alpha_2,\dots,\alpha_n\in \mathbb{F}_2^{64}$，类比"向量空间及其子空间的基"下定理 1 的构造方式构造出一组线性基。

首先，在线性基 $B$ 中加入 $\alpha_1$，并记录下它的最高位。

对于每一个 $\alpha_i$，我们自高向低遍历每一位。如果当前位在 $B$ 中有对应的向量 $\beta$，那么 $\alpha_i\gets \alpha_i+\beta$，即把当前位消掉；否则，若 $\alpha_i\neq 0$，将 $\alpha_i$ 加入 $B$，并记录它的最高位。

由"线性相关与线性无关"下的定理 2 知，我们的操作不会改变 $\alpha_i$ 与 $B$ 中向量的线性相关性，并且这样构造能够保证每一个位置至多只会有一个向量。

这样的构造方式能够保证构造出的 $B$ 中每一个 $\alpha_i$ 都不能被 $\alpha_j,j<i$ 线性表示，也能够保证不在 $B$ 中的每一个 $\alpha_i$ 都能够被 $\alpha_j,j<i$ 线性表示。因此 $B$ 是一个线性无关的向量组，并且每一个 $\alpha_i$ 都能够被 $B$ 线性表出，因此 $B$ 是 $\left<\alpha_1,\alpha_2,\dots,\alpha_n\right>$ 的一个基。

设 $B$ 中的元素数量为 $|B|$，那么这个基能表示出 $2^{|B|}$ 个互不相同的向量，实际问题中关于能否表出 $0$ 我们应当特殊讨论。

## [HDU 3949] XOR

本题求第 $k$ 小，我们先贪心构造出一个线性基 $B$，然后从高位向低位考虑 $B$ 中的向量。对于每一个向量，设最高位比它的最高位低的向量有 $m$ 个。

由线性表出的唯一性，我们知道，当前向量选或不选都分别对应着 $2^m$ 个互不相同的向量。

因此，如果选择使答案的当前位为 $1$，那么就舍掉了 $2^m$ 个当前位为 $0$ 的向量，这些向量都一定比当前的答案要小。

所以按照这种贪心方式就能够正确地求出第 $k$ 小。

[AC Code](https://gist.github.com/hikariyo/0f202a2df9af9ddcdce1b2a24e4f0b68)。

## [ICPC2025 Shanghai G] Gemcrate

首先，如果 $m$ 是一个可行的答案，分两种情况讨论：

1. 如果 $m$ 是奇数，那么我把 $B_1,B_2,\dots,B_m$ 全部异或起来，答案一定不劣；
2. 如果 $m$ 是偶数，那么我把 $B_1,B_2,\dots,B_m$ 任意分成两组，答案一定不劣。

所以我们只需要考虑分一组和两组，一组的情况是显然的，下面讨论两组。

统计所有位置出现次数，如果这一位能够出现在答案中，那么它一定是两组均有奇数个，所以我们把出现次数是奇数的位直接抹掉。

然后把处理之后的数据加入线性基，从大到小贪心即可。

[AC Code](https://gist.github.com/hikariyo/de93f6f2c86f65fb9d95371975ee374b)。

## [JSCPC2025 B] Integer Generator

本题加大力度地考查了位运算的性质，不过我们从抽象代数的角度去思考是比较自然的。

首先，位运算关于每一个位独立，我们可以认为运算是域 $\mathbb{F}_2$ 下的运算，进而推广到整数。

那么，容易看出 $\texttt{and}$ 就是乘法，$\texttt{xor}$ 就是加法，$\texttt{or}$ 类似于乘法，具体地：
$$
\texttt{or}:\mathbb{F}_2\times \mathbb{F_2}\to \mathbb{F}_2,\quad (x,y)\mapsto 1-(1-x)(1-y)=x+y-xy=x+y+xy
$$
推广到整数，为了与一般的运算区别，这里使用 $\oplus,\otimes$ 代替 $\texttt{and},\texttt{xor}$，我们有 $x\operatorname{or}y=x\oplus y\oplus (x\otimes y)$。

类似地，自然也有分配律 $(x\oplus y)\otimes z=(x\otimes z)\oplus (y\otimes z)$。

因此，本题可以认为只存在按位与和异或运算即可，并且根据分配律，我们可以认为最终的表示方式总是异或运算最后进行。

首先对给定的 $n$ 个数字求出一个线性基 $B$，假设存在一种表出 $x$ 的方式，我们把其中的 $a_i$ 都替换成线性基中的元素，它恰好是形如 $c_1\oplus c_2\oplus \dots\oplus c_m$ 的形式，其中 $c_i$ 是某些线性基中的某些元素按位与。

进一步地，如果这个线性基满足 $\forall x,y\in B$，都有 $x\otimes y$ 能被 $B$ 表出，即对按位与运算封闭，那么上面说的 $c_i$ 就是线性基的元素。

根据题目的数据范围，这个线性基张成的空间是 $\mathbb{F}_2^{30}$ 的一个子空间，线性基至多只会有 $30$ 个元素。

也就是说，我们可以暴力地对 $B$ 进行扩张，设 $V$ 是值域，直接 $O(\log^2 V)$ 地枚举每一对元素，然后 $O(\log V)$ 检查是否能被 $B$ 表出。如果都能被表出，那么已经封闭；如果存在不能被表出的元素，那么至少会有一个不能被表出的元素，于是这种循环至多会执行 $O(\log V)$ 次。即扩张的复杂度是 $O(\log^4 V)$。

我们只需要扩张之后构造出一个方案即可，所以这里用一步高斯消元解异或方程组，是 $O(\log^3V/\omega)$ 的，因此最终的复杂度是 $O(n\log V+\log^4 V)$。

[AC Code](https://gist.github.com/hikariyo/e486b236d810e97f17dfcbd19bae01c7)。
