---
date: 2025-12-05 23:22:00
title: 数学 - 数论基础
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 竞赛视角下阅读《具体数学》第四章：数论。
---

## 关键概念性质

### 裴蜀定理

对于 $p\perp q$，我们总能找到 $a,b$ 满足 $ap+bq=1$。

由于 $\text{exgcd}$ 的过程就一定能找到一组特解 $a_0,b_0$，接下来我们来求通解。

假设 $a=a_0+k$ 是一个解，那么我们就有 $b=\cfrac{1-(a_0+k)p}{q}$。

带入 $a_0p+b_0q=1$，有 $b=\cfrac{b_0q-kp}{q}=b_0-\cfrac{kp}{q}$。

由于 $b$ 是整数，所以 $q\mid (kp)$；又因为 $p\perp q$，所以 $q\mid k$。

因此 $\begin{cases}a=a_0+k'q\\b=b_0-k'p\end{cases}$ 是不定方程 $ap+bq=1$ 的通解，其中 $k'$ 为任意整数。

### 乘法逆元

若 $a\perp m$，那么我们能在 $0\sim m-1$ 中找到唯一的 $b$ 满足 $ab\equiv 1\pmod m$。

证明：由于方程 $ab-km=1$ 一定能解出 $b=b_0+km$，所以在 $0\sim m-1$ 中有唯一的 $b$ 满足条件。

### 唯一分解定理

对于任意数 $n=p_1p_2\dots p_m$ 的表示方式是唯一的，其中 $p_i$ 是素数。

证明：当 $n=1,2$ 时，分解是唯一的。设 $n\le k$ 时分解都是唯一的，当 $n=k+1$ 时，设 $n=p_1\dots p_m=q_1\dots q_k$，其中 $p_1\le \dots\le p_m$ 并且 $q_1\le \dots\le q_k$。

1. 如果 $p_1=q_1$，那么把 $p_1$ 除掉，$n$ 一定会严格减小，根据归纳假设就有 $n$ 的表示唯一；

2. 如果 $p_1\neq q_1$，不失一般性，设 $p_1<q_1$，那么我们能找到一组 $a,b$ 满足 $ap_1+bq_1=1$，那么：

    $$
    q_2\dots q_k=(ap_1+bq_1)q_2\dots q_k=ap_1q_2\dots q_k+bq_1\dots q_k
    $$
    由于 $n=q_1\dots q_k$，所以 $p_1\mid bq_1\dots q_k$，又因为 $p_1\mid ap_1q_2\dots q_k$，所以 $p_1\mid q_2\dots q_k$。

    然而 $q_2\dots q_k$ 严格小于 $n$，根据归纳假设，它的表示形式唯一；但是 $p_1<q_2$ 又是 $q_2\dots q_k$ 的一个因子，所以我们能推出 $q_2\dots q_k$ 存在另一个含有 $p_1$ 的表示形式，产生矛盾，因此 $p_1=q_1$。

### 互质

若 $(a,b)=1$，称 $a\perp b$。设 $a=\prod p_i^{\alpha_i},b=\prod p_i^{\beta_i}$，其中 $p_i$ 取遍所有质数，$\alpha_i,\beta_i\ge 0$。

如果 $a\perp b$，则对所有 $i$ 有 $\min\{\alpha_i,\beta_i\}=0$，又因为它们都是非负整数，所以这等价于 $\alpha_i\beta_i=0$。

推论：如果 $m\perp n$，则 $a\perp m$ 且 $a\perp n$ 等价于 $a\perp mn$。设 $a$ 对应的指数是 $\alpha_i$，$m$ 对应的是 $\beta_i$，$n$ 对应的是 $\gamma_i$。

1. 充分性：$\alpha_i\beta_i=\alpha_i\gamma_i=0$，则 $\alpha_i(\beta_i+\gamma_i)=0$；
2. 必要性：$\alpha_i\beta_i+\alpha_i\gamma_i=0$，又因为这两项都是非负的，所以它们必须都等于 $0$。

### 序列 kn mod m 的性质

证明：$0,n,2n,\dots,(m-1)n$ 在模 $m$ 的情况下是 $0,d,2d,\dots,m-d$ 这 $m/d$ 个数以某种顺序重复 $d$ 次，其中 $d=(m,n)$。

首先，我们有 $kn\bmod m=kn-\lfloor\cfrac{kn}{m}\rfloor m=d(k\cfrac{n}{d}-\lfloor\cfrac{kn/d}{m/d}\rfloor\cfrac{m}{d})$。

令 $n'=n/d,m'=m/d$，有 $d(kn'-\lfloor\cfrac{kn'}{m'}\rfloor m')=d(kn'\bmod m')$。

所以我们证 $kn'\bmod m'$ 是 $0,1,2,\dots,m'-1$ 以某种顺序重复 $d$ 次。

若 $kn'\equiv jn'\pmod{m'}$，由于 $m'\perp n'$，所以 $k\equiv j\pmod{m'}$。

因此当 $k=0\sim m'-1$ 时，它们是互不相同的；又因为模 $m'$ 的情况下只有 $m'$ 个数字互不相同，所以它们就是 $0,1,\dots,m'-1$ 的某种排列。

并且，当 $k\equiv j\pmod {m'}$ 时，有 $kn'\equiv jn'\pmod{m'}$，所以它是重复出现的。

### 欧拉定理

欧拉函数 $\varphi(m)=\sum_{k=1}^{m}[k\perp m]$，即与 $m$ 互质的数的个数。

欧拉定理的内容是：对于 $a\perp m$，有 $a^{\varphi(m)}\equiv 1\pmod m$，下面我们来证明。

设 $k_1,k_2,\dots,k_{\varphi(m)}$ 是在 $1\sim m$ 中与 $m$ 互质的所有数，和证明序列 $kn'\bmod m'$ 是 $0,1,2,\dots,m'-1$ 以某种顺序重复出现一样，我们可以证明：$ak_1\bmod m,ak_2\bmod m,\dots,ak_{\varphi(m)}\bmod m$ 是 $k_1,k_2,\dots,k_{\varphi(m)}$ 以某种顺序的排列。

若 $ak_i\equiv ak_j\pmod m$，由于 $a\perp m$，所以 $k_i\equiv k_j\pmod m$，所以只能有 $i=j$，这证明了 $ak_i\bmod m$ 两两不相等。

由于 $m\perp a$ 且 $m\perp k_i$，于是 $ak_i\perp m$，所以序列 $ak_i\bmod m$ 是 $\varphi(m)$ 个两两不相等的、与 $m$ 互质的、$1\sim m$ 之间的数，于是这两个序列只是顺序不一样。

那么，我们把这两个序列分别相乘，就有 $a^{\varphi(m)}\prod_{i=1}^{\varphi(m)} k_i\equiv \prod_{i=1}^{\varphi(m)}k_i\pmod m$。

由于 $k_i\perp m$，有 $\prod k_i\perp m$，于是 $a^{\varphi(m)}\equiv 1\pmod m$。

### 积性函数

设 $m_1\perp m_2$ 且 $m=m_1m_2$，如果有 $f(m)=f(m_1)f(m_2)$，那么称 $f$ 是积性函数。

若 $f(m)=\sum_{d\mid m}g(d)$，那么 $f$ 是积性函数等价于 $g$ 是积性函数。

1. 必要性：若 $g$ 是积性函数，得到：
   $$
   \begin{aligned}
   f(m_1m_2)&=\sum_{d\mid m_1m_2}g(d)\\
   &=\sum_{d_1\mid m_1}\sum_{d_2\mid m_2}g(d_1d_2)\\
   &=\left(\sum_{d_1\mid m_1}g(d_1)\right)\left(\sum_{d_2\mid m_2}g(d_2)\right)\\
   &=f(m_1)f(m_2)
   \end{aligned}
   $$
   
   这里解释一下，由于 $m_1\perp m_2$，那么枚举 $d\mid m_1m_2$ 等价于枚举 $d_1\mid m_1,d_2\mid m_2$ 其中 $d=d_1d_2$。

   为什么？我们可以通过证明分别枚举不重不漏来证明这两个方式等价。
   
   1. 唯一性：不失一般性，设 $d_1\neq d_1'$，$d_2$ 与 $d_2'$ 的关系任意，那么一定有 $d_1d_2\neq d_1'd_2'$。考虑等式两边的因式分解，由于 $d_1\neq d_1'$ 并且 $d_2$ 与 $d_2'$ 不含有 $m_1$ 的质因子，所以不可能使得等式成立。
   2.  完整性：对于每个 $d$，它总有来自于 $m_1$ 的部分和 $m_2$ 的部分，因此是不可能被漏掉的。
   
2. 充分性：若 $f$ 是积性函数。当 $m=1$ 时，根据定义 $f(1)=g(1)$，于是 $f(1)=f^2(1)$ 能推出 $g(1)=g^2(1)$。

   设 $m\le k$ 时都成立 $g(m_1m_2)=g(m_1)g(m_2)$，那么当 $m=k+1$ 时，由 $f(m)=f(m_1)f(m_2)$ 得到：
   $$
   \begin{aligned}
   \sum_{d\mid m}g(d)&=\left(\sum_{d_1\mid m_1}g(d_1)\right)\left(\sum_{d_2\mid m_2} g(d_2)\right)\\
   &=\sum_{d_1d_2\mid m_1m_2} g(d_1)g(d_2)
   \end{aligned}
   $$

   只要不是 $d_1=m_1$ 且 $d_2=m_2$，就能根据归纳假设，得到 $g(d_1)g(d_2)=g(d_1d_2)$。

   为什么？因为只要存在任意一个 $d_i<m_i$，就有 $d_1d_2<m_1m_2$，等价于 $d_1d_2\le m-1$，这就落到了归纳假设的范围内。
   $$
   \begin{aligned}
   \sum_{d\mid m}g(d)&=\sum_{d_1d_2\mid m_1m_2} g(d_1)g(d_2)\\
   &=\left(\sum_{d_1d_2\mid m_1m_2}g(d_1d_2)\right)-g(m_1m_2)+g(m_1)g(m_2)\\
   &=\sum_{d\mid m}g(d)-g(m_1m_2)+g(m_1)g(m_2)
   \end{aligned}
   $$
   移项可得 $g(m_1m_2)=g(m_1)g(m_2)$。

### 欧拉函数

关于欧拉函数有这样一个等式：$\sum_{d\mid n}\varphi(d)=n$。

这里给出书上的证明方法：对序列 $\cfrac{0}{n},\cfrac{1}{n},\dots,\cfrac{n-1}{n}$，它们化简到最简分母时，分母一定是 $n$ 的因子。

我们假设 $d\mid n$ 是化简之后的一类分母，可以证明，对于所有 $k\perp d$ 且 $k<d$，$\cfrac{k}{d}$ 都会出现在原序列中，同样是证明不重和不漏。

1. 唯一性：设 $\cfrac{k_1}{d_1}$ 和 $\cfrac{k_2}{d_2}$ 满足上述要求，首先有 $k<d$ 因此通分后分子也小于分母，所以它们一定会出现在序列中，下面证它们不相等。

   1. 若 $d_1=d_2$，那么枚举不同地 $k$ 保证了 $k_1\neq k_2$，就有 $k_1\neq k_2$，于是 $\cfrac{k_1}{d_1}\neq \cfrac{k_2}{d_2}$。

   2. 若 $d_1\neq d_2$，当 $k_1d_2=k_2d_1$ 时，有 $k_1=\cfrac{k_2d_1}{d_2}$。由于 $k_2\perp d_2$，且 $k_1$ 为整数，有 $d_2\mid d_1$。

      设 $d_1=md_2$，由于 $d_1\neq d_2$ 所以 $m>1$，那么 $k_1=mk_2$，就有 $(k_1,d_1)=m\neq 1$，与题设矛盾，于是不成立。

2. 完全性：对于任意一个 $\cfrac{k}{n}$，它通分之后分母 $d$ 一定是 $n$ 的因子，并且分子与分母互质，并且分子小于分母。

因此，我们可以枚举每一种分母 $d$，它们代表的分数会出现 $\varphi(d)$ 次，所以 $\sum_{d\mid n}\varphi(d)=n$。

由于 $f(n)=\sum_{d\mid n}\varphi(d)=n$ 是积性函数，所以 $\varphi(n)$ 是积性函数。

那么我们只需要求 $\varphi(p^k)$，其中 $p$ 是质数，就可以求出 $\varphi(n)$ 的表达式。

对于 $1,2,\dots,p^k$ 中，不与 $p^k$ 互质的数都是 $p$ 的倍数，所以 $\varphi(p^k)=p^k-p^{k-1}=p^{k}(1-p^{-1})$，所以：$\varphi(n)=n\prod(1-p_i^{-1})$。

### 莫比乌斯函数

书上给出了一种定义方式：$\mu(d)$ 是满足 $\sum_{d\mid n}\mu(d)=[n=1]$ 的函数，接下来我们来求它的解析式。

容易验证 $f(n)=[n=1]$ 是积性函数，所以 $\mu(n)$ 也是积性函数。

那么我们只需要求 $\mu(p^k)$，其中 $p$ 是质数，就可以求出 $\mu(n)$ 的表达式。

根据定义，我们有 $\mu(1)+\mu(p)+\mu(p^2)+\cdots+\mu(p^k)=[n=1]$。

1. 当 $k=0$ 时，有 $\mu(1)=1$；
2. 当 $k=1$ 时，有 $\mu(p)=-1$；
3. 当 $k\ge 2$ 时，有 $\mu(p^2)+\cdots+\mu(p^k)=0$，分别取 $k=2,3,\dots$ 可以得知它们都是 $0$。

因此 $\mu(n)=\begin{cases}(-1)^m\quad \forall p_i^2\not\mid n\\0\quad \exist p_i^2\mid n\end{cases}$，其中 $m$ 是 $n$ 的质因子数量，$p_i$ 是 $n$ 的质因子。

### 莫比乌斯反演

式 $g(m)=\sum_{d\mid m}f(d)$ 等价于 $f(m)=\sum_{d\mid m}\mu(d)g(\cfrac{m}{d})$。

1. 充分性：当 $g(m)=\sum_{d\mid m}f(d)$ 时，有：
   $$
   \begin{aligned}
   \sum_{d\mid m}\mu(d)g(\frac{m}{d})&=\sum_{d\mid m}\mu(\frac{m}{d})g(d)\\
   &=\sum_{d\mid m}\sum_{k\mid d}\mu(\frac{m}{d})f(k)\\
   &=\sum_{k\mid m}\sum_{d\mid\frac{m}{k}}\mu(\frac{m}{kd})f(k)\\
   \end{aligned}
   $$
   注意，这一步的证明在例题2。
   $$
   \begin{aligned}
   \sum_{d\mid m}\mu(d)g(\frac{m}{d})&=\sum_{k\mid m}f(k)\left(\sum_{d\mid\frac{m}{k}}\mu(\frac{m}{kd})\right)\\
   &=\sum_{k\mid m}f(k)\left(\sum_{d\mid\frac{m}{k}}\mu(d)\right)\\
   &=\sum_{k\mid m}f(k)[\cfrac{m}{k}=1]\\
   &=f(m)
   \end{aligned}
   $$

2. 必要性：当 $f(m)=\sum_{d\mid m}\mu(d)g(\cfrac{m}{d})$ 时，有：
   $$
   \begin{aligned}
   \sum_{d\mid m}f(d)&=\sum_{d\mid m}\sum_{k\mid d}\mu(k)g(\frac{d}{k})\\
   &=\sum_{d\mid m}\sum_{k\mid d}\mu(\frac{d}{k})g(k)\\
   &=\sum_{k\mid m}\sum_{l\mid \frac{m}{k}}\mu(l)g(k)\\
   &=\sum_{k\mid m}g(k)\left(\sum_{l\mid \frac{m}{k}}\mu(l)\right)\\
   &=\sum_{k\mid m}g(k)[\frac{m}{k}=1]\\
   &=g(m)
   \end{aligned}
   $$

## 例题

### 例1

求 $\sum_{0\le k<m}\lfloor\cfrac{nk+x}{m}\rfloor$，其中 $m>0$，$n$ 是整数，$x$ 是实数。

我们先把 $nk/m$ 的整数部分移到外面去，即：
$$
\begin{aligned}
\sum_{0\le k<m}\lfloor\frac{nk+x}{m}\rfloor&=\sum_{0\le k<m}\lfloor\frac{kn\bmod m+x}{m}\rfloor+\sum_{0\le k<m}\frac{kn-kn\bmod m}{m}\\
&=\sum_{0\le k<m}\lfloor\frac{kn\bmod m+x}{m}\rfloor+\sum_{0\le k<m}\frac{kn}{m}-\sum_{0\le k<m}\frac{kn\bmod m}{m}\\
&=\sum_{0\le k<m}\lfloor\frac{kn\bmod m+x}{m}\rfloor+\frac{n(m-1)}{2}-\frac{d(0+1+\dots+(\cfrac{m}{d}-1))\times d}{m}\\
&=\sum_{0\le k<m}\lfloor\frac{kn\bmod m+x}{m}\rfloor+\frac{n(m-1)}{2}-\frac{m-d}{2}\\
&=d\sum_{i=0}^{m/d-1}\lfloor\frac{x+di}{m}\rfloor+\frac{n(m-1)}{2}-\frac{m-d}{2}\\
&=d\sum_{i=0}^{m/d-1}\lfloor\frac{x/d}{m/d}+\frac{i}{m/d}\rfloor+\frac{n(m-1)}{2}-\frac{m-d}{2}\\
&=d\lfloor\frac{x}{d}\rfloor+\frac{n(m-1)}{2}-\frac{m-d}{2}\\
\end{aligned}
$$

### 例2

证明：$\sum_{d\mid m}\sum_{k\mid d}a_{d,k}=\sum_{k\mid m}\sum_{l\mid\frac{m}{k}} a_{kl,k}$。

首先处理左式：
$$
\begin{aligned}
\sum_{d\mid m}\sum_{k\mid d}a_{d,k}&=\sum_{d,k}\sum_{j,l}a_{d,k}[m=jd][d=kl]\\
&=\sum_{k,j,l}a_{kl,k}[m=jkl]
\end{aligned}
$$


然后是右式：
$$
\begin{aligned}
\sum_{k\mid m}\sum_{l\mid \frac{m}{k}}a_{kl,k}&=\sum_{k,l}\sum_{j,n}a_{kl,k}[m=jk][\frac{m}{k}=nl]\\
&=\sum_{k,l}\sum_{j,n}a_{kl,k}[m=jk][j=nl]\\
&=\sum_{k,n,l}a_{kl,k}[m=nlk]\\
\end{aligned}
$$
我们发现除了求和的下标字母不同，这两个式子是相同的。

