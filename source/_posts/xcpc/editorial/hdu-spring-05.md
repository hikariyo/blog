---
date: 2026-05-12 12:57:00
updated: 2026-05-13 00:19:00
title: 题解 - 2026 杭电春季联赛 (5)
katex: true
tags:
- Algo
- C++
categories:
- XCPC
description: 2026 杭电春季联赛第五场题解。
---

## 1005

边 $(i,j)$ 存在意味着 $i\oplus j=2^{k}$，其中 $k$ 是一个非负整数。

移项可得 $j=i\oplus 2^k$，也就是说每一条边只能改变一个二进制位。

要把 $1$ 通过改变二进制位的方式达到 $n$，至少需要更改 $\operatorname{popcount}(n\oplus 1)$ 个二进制位。

又因为构造出长度为 $\operatorname{popcount}(n\oplus 1)$ 的路径使 $1$ 到达 $n$ 是可以简单做到的，因此答案就是 $\operatorname{popcount}(n\oplus 1)$。

## 1003

对于每一个 $p_i$，有 $0\le p_i\le 2^t-1$，于是 $0\le S\le n(2^t-1)$。

问题就是，在这个区间内的 $S$，是否总能被取到？

答案是肯定的，考虑令 $S$ 对 $2^t-1$ 进行带余除法，商为 $q$，余数为 $r$。

那么只需要放 $q$ 个 $2^t-1$，$1$ 个 $r$ 即可；如果 $r=0$ 则不需要放 $r$。

## 1008

这个题目我的第一想法是，设 $dp_{i,j}$ 表示第 $i$ 天以原价购买，上一次以原价购买是第 $j$ 天，所需要的最小花费。

但是发现进行转移时，枚举上一次原价购买的日期，其实并不关注上一次购买的再上一次。

所以修改状态，设 $dp_i$ 表示第 $i$ 天以原价购买所需要的最小花费。

枚举上一次原价购买的日期是 $j$，那么转移方程如下：
$$
\begin{aligned}
dp_i&=\min_{i-k\le j\le i-1}\{dp_j+\frac{S_{i-1}-S_j}{2} +a_i\}\\
&=\frac{S_{i-1}}{2} + a_i+\min_{i-k\le j\le i-1} \{dp_j-\frac{S_j}{2}\}
\end{aligned}
$$
第一天必须是原价购买，为了方便我们令 $a_{n+1}=0$ 并且让第 $n+1$ 天也是原价购买，那么这就是一个线段树优化 DP 了。

## 1004

化简公式：
$$
\begin{aligned}
\sum_{l=1}^n\sum_{r=l}^n \left\lfloor \log_2{\frac{r}{l}}\right\rfloor
&=\sum_{k=1}^{30}\sum_{l=1}^n\sum_{r=l}^nk\left[k=\left\lfloor \log_2{\frac{r}{l}}\right\rfloor\right]\\
&=\sum_{k=1}^{30}\sum_{l=1}^n\sum_{r=l}^n  k\left[l2^k\le r <l2^{k+1}\right]\\
&= \sum_{k=1}^{30}\sum_{l=1}^n k(\min\{n,l2^{k+1}-1\} - l2^k + 1)[l2^k\le n]\\
&= \sum_{k=1}^{30}\sum_{l=1}^{\lfloor n/2^k\rfloor} k(\min\{n,l2^{k+1}-1\} - l2^k + 1)\\
\end{aligned}
$$
当 $l\le \lfloor(n+1)/2^{k+1}\rfloor$ 时，最内层的求和是 $\sum_l k(l 2^k)=k2^k\sum_l l$；反之，最内层的求和是 $\sum_l k(n-l2^k+1)=k\sum_{l} (n+1-l2^k)$，分类讨论即可。

## 1006

注意到询问的 $k$ 很小，最大只有 $7$，所以如果这 $1000\times 1000$ 个网格中，如果某一个网格被 $\ge 8$ 个马占有，那么它永远是合法的。

否则，我们用二维异或前缀和记录下这个网格的马的状态，这里用异或哈希把 $\mathbb F_2^{n}\to \mathbb F_2^{64}$ 存储。

处理每一个询问时，我们 $O(2^k)$ 枚举每一个子集，用哈希表 $O(1)$ 查出有多少网格是被当前这个子集的马占有。

复杂度 $O(n+q 2^k)$。

## 1002

这个题目代码不是很难写，但是想要证明出来这么做是对的还是稍显麻烦。

定义 $G_x$ 由 $\le x$ 的顶点和原有的边组成的子图，那么 $\operatorname{ans}(u)\le x$ 当且仅当在 $G_x$ 中存在一条 $1\leadsto u$ 的路径。

进行一个增量的遍历，代码如下：

```cpp
priority_queue<int,vector<int>,greater<int>> q;
q.push(1);

auto resume = [&](int allow) {
    while (q.size()) {
        int u = q.top();
        if (u > allow) return;
        q.pop();
        if (vis[u]) continue;
        vis[u] = allow;
        for (int to: G[u]) {
            if (vis[to]) continue;
            q.push(to);
        }
    }
};

for (int i = 1; i <= n; i++) {
    resume(i);
}
```

可以证明这能求解出正确的答案，设 $R_x$ 表示 $G_x$ 中能从 $1$ 到达的点的集合，$A_x$ 表示执行 `resume(1), resume(2), ..., resume(x)` 后 `vis` 不为 $0$ 的集合，我们可以证明 $R_x=A_x$。

+ 当 $x=1$ 时，显然 $R_x=A_x=\{1\}$。

+ 假设 $x=x_0$ 时结论成立，当 $x=x_0+1$ 时，由于遍历直走了 $\le x_0+1$ 的点，所以 $A_x\subseteq R_x$ 一定是成立的；

  反之，假设存在 $v\in R_x\setminus A_x$，那么就在 $G_x$ 中存在一条路径 $v_1,\dots,v_k$ 使得 $v_1=1,v_k=v, \forall i,v_i\le x$。

  设 $j$ 是最小的、在这个路径中满足 $v_j\not\in A_x$ 的整数，由于 $v_1\in A_x$，所以 $j\ge 2$。

  由于 $v_{j-1}\in A_x$，那么 $v_j$ 应当在目前的小根堆中，又因为 $v_j\le x_0+1$，所以它应该被正常弹出小根堆并打上标记，这就产生了矛盾，于是 $R_x\setminus A_x=\varnothing$，即 $R_x=A_x$。

所有点都在自己对应最小的 $R_x$ 中被标记上了答案，所以这个做法是正确的。

## 1001

本题全局去看比较难做，我们希望能够将这个问题划为子问题后类似递推进行求解。

一个合理的想法是，假设当前有 $k$ 枚为正，我们看经过若干次操作之后第一次有 $k-1$ 枚为正是什么情形。

自然能够导出下面的分类讨论：

1. 如果 $s_k$ 为正，那么 $1$ 次操作后 $k-1$ 枚为正；

2. 如果 $s_k$ 为反，假设 $p$ 是满足 $p>k$ 且 $s_p$ 为正的第一个位置，

   模拟这个过程可以看出，经过 $2(p-k)+1$ 次操作后，第一次有 $k-1$ 枚为正。

3. 如果 $s_k$ 为反，并且没有 $p$ 满足 $p>k$ 且 $s_p$ 为正，这种情况是不可能的，因为这样的话，正的数量 $<k$，不可能等于 $k$。

设 $p_1,p_2,\dots,p_k$ 表示 $s_k$ 为正的那些下标，那么答案就是 $\sum_{i=1}^k (2(p_i-i)+1)$，$O(n)$ 预处理即可 $O(1)$ 回答询问。

## 1007

本题使用 ODT，假设 $c_i$ 表示 $i$ 次操作后区间数量，$k_i$ 表示第 $i$ 次操作访问的那些即将被覆盖的区间数量，那么我们有 $c_{i}\le c_{i-1}+2-k_i$。

移项可得 $\sum k_i\le c_n-c_0+2q$，而右边是一个 $O(n+q)$ 的级别，因此遍历的均摊复杂度是正确的。

那么既然支持进行遍历操作，这个问题就极其简单了，只需要写一个支持区间 $+x$ 和区间 $\times 0$ 的线段树即可，重分配时直接遍历这些即将被重分配的区间，把线段树上目前维护的值获取出来即可。

## 1009

显然路径长度不超过 $k\le 6$，这是由鸽巢原理决定的。

那么一条路径可以被分成两部分，每一部分的点数都不超过 $3$。

给点随机二染色，可以在 $O(n)$ 的复杂度求出每一个颜色中，起点为 $u$、长度 $\le 3$、$a$ 之和模 $k$ 余 $x$、边权和最小的半条路径，然后我们枚举跨颜色的边即可。

一次成功的概率是 $\frac{1}{2^5}$，那么执行 $500$ 次错误的概率是 $(1-2^{-5})^{500}=1.28\times 10^{-7}$，足够低。

## 1010

根据欧拉函数的定义，有：
$$
\varphi(ni)=\frac{\varphi(n)\varphi(i)\gcd (n,i)}{\varphi(\gcd (n,i))}
$$
推式子：
$$
\begin{aligned}
S(n,m)&=\sum_{i=1}^m \varphi(ni)\\
&=\sum_{g=1}^n \sum_{i=1}^m\frac{\varphi(n)\varphi(i)g}{\varphi(g)}[g=\gcd(n,i)]\\
&=\sum_{g\mid n} \sum_{i=1}^{m/g}\frac{\varphi(n)\varphi(ig)g}{\varphi(g)}\left[1=\gcd\left(\frac{n}{g},i\right)\right]\\
&=\sum_{g\mid n} \sum_{i=1}^{m/g}\frac{\varphi(n)\varphi(ig)g}{\varphi(g)}\sum_{d\mid i,d\mid n/g} \mu(d)\\
&=\sum_{d=1} ^n \sum_{g\mid n} \sum_{i=1}^{m/dg} \mu(d)\frac{\varphi(n)\varphi(idg)g}{\varphi(g)}[dg\mid n]\\
&=\sum_{d=1} ^n \sum_{g\mid n} \sum_{i=1}^{m/x}\sum_{x\mid n} \mu(d)\frac{\varphi(n)\varphi(idg)g}{\varphi(g)}[dg=x]\\
&=\varphi(n)\sum_{x\mid n} \sum_{d\mid x} \sum_{i=1}^{m/x}\mu(d)\frac{\varphi(ix)\frac{x}{d}}{\varphi(\frac{x}{d})}\\
&=\varphi(n)\sum_{x\mid n} \sum_{i=1}^{m/x} \varphi(ix) \sum_{d\mid x} \mu(d)\frac{\frac{x}{d}}{\varphi(\frac{x}{d})}\\
\end{aligned}
$$
令 $F(x)=\sum_{d\mid x} \mu (d)\frac{x/d}{\varphi(x/d)}$，容易验证这是一个积性函数，那么 $F(p)=\frac{1}{p-1}$，$F(1)=1$，$F(p^k)=0(k\ge 2)$。

因此只需枚举 $\mu(x)\neq 0$ 的 $n$ 的因子即可，继续化简：
$$
\begin{aligned}
S(n,m)&=\varphi(n)\sum_{x\mid n,\mu(x)\neq 0} F(x) \sum_{i=1}^{m/x} \varphi(ix)\\
&=\varphi(n)\sum_{x\mid n,\mu(x)\neq 0} F(x) S\left(x, \left\lfloor\frac{m}{x}\right\rfloor\right)
\end{aligned}
$$
如果用 $\omega(n)$ 表示 $n$ 的质因子个数，进行记忆化，那么 $S$ 的第一个参数至多有 $O(2^{\omega(n)})$ 种，第二个参数至多有 $O(\sqrt m)$ 种。

当 $n=1$ 时，直接调用杜教筛进行求和，复杂度 $O(m^{\frac{2}{3}})$；当 $n>1$ 时，枚举 $n$ 的合法因子 $x$ 递归求解，因为是枚举子集，所以大概的复杂度上界是 $O(3^{\omega(n)}\sqrt m)$，实际上由于数论的条件难以满足，要比这个小得多。

