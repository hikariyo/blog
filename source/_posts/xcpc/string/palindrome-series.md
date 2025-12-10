---
date: 2025-12-09 00:00:00
title: 字符串 - Palindrome Series
katex: true
tags:
- Algo
- C++
- String
categories:
- XCPC
description: Palindrome Series 及其应用
---

## Border Series

先说结论，把 $S$ 的所有 $\text{Border}$ 从大到小排列，能划分为连续的 $O(\log |S|)$ 个等差数列。

设 $S$ 最大的 $\text{Border}$ 为 $\pi(S)$，那么：

1. $|\pi(S)|\le \cfrac{|S|}{2}$，由 $\text{Border}$ 理论可知，现在变成了规模为 $\cfrac{|S|}{2}$ 的子问题。

2. $|\pi(S)|> \cfrac{|S|}{2}$，那么 $S$ 的最小周期是 $|S|-|\pi(S)|<\cfrac{|S|}{2}$，我们设 $D=S[1,|S|-|\pi(S)|]$，那么 $S=D^kT$，其中 $k\ge 2$， $T$ 为 $D$ 的前缀（可能为空）。

   根据上面的定义，有 $\pi(D^kT)=D^{k-1}T$，下面我们证明 $\pi(D^{k-1}T)=D^{k-2}T$。

   考虑反证法，如果 $|\pi(D^{k-1}T)|>|D^{k-2}T|$，说明 $|D^{k-1}T|-|\pi(D^{k-1}T)|<|D|$，这说明 $D^{k-1}T$ 中存在一个更小的周期，而这个循环节必然也是 $D^kT$ 的周期，就与 $\pi(D^kT)=D^{k-1}T$ 矛盾了。

   又因为 $D^{k-2}T$ 是 $D^{k-1}T$ 的 $\text{Border}$，那么没有比它更大的就说明 $\pi(D^{k-1}T)=D^{k-2}T$。

   所以一路递推下去，最终一定会跳到 $T$，这一路上 $S'-\pi(S')=|D|$ 是等差数列，并且 $|T|<|D|<\cfrac{|S|}{2}$，变成了规模为 $\cfrac{S}{2}$ 的子问题。

所以，把 $S$ 的所有 $\text{Border}$ 从大到小排列，能划分为连续的 $O(\log |S|)$ 个等差数列。

## Palindrome Series

参考文章：[Border Series在回文自动机上的运用](https://www.cnblogs.com/Parsnip/p/12426971.html)、[字符串科技：Palindrome Series](https://zhuanlan.zhihu.com/p/92874690)。

对于一个回文串 $S$ 来说，它的所有 $\text{Border}$ 一定也是回文串：设 $p$ 是一个 $\text{Border}$，我们有 $1\le i\le p$ 时，$S_i=S_{i+|S|-p}$；又因为对所有 $1\le i\le |S|$ 有 $S_i=S_{|S|-i+1}$，那么 $S_i=S_{p-i+1}$，即 $\text{Border}$ 是回文串。

反过来看，结尾在 $S$ 末尾的回文子串也一定是 $S$ 的 $\text{Border}$，证明过程同上。

所以回文自动机任意节点的失配链可以被划分成连续的 $O(\log |S|)$ 个等差数列。

怎么维护？我们在自动机上维护节点 $x$ 的如下信息：

+ $\text{link}(x)$ 表示失配链上的父节点，其实就是 $\text{fail}(x)$，只不过在学习一个文章的时候他用的 $\text{link}$，和下面的 $\text{slink}$ 统一。
+ $\text{len}(x)$ 表示节点 $x$ 对应的回文子串长。
+ $\text{dif}(x)$ 表示 $\text{len}(x)-\text{len}(\text{link}(x))$。
+ $\text{slink}(x)$ 表示失配链上第一个满足 $\text{dif}(y)\neq \text{dif}(\text{link}(y))$ 的 $\text{link}(y)$，根据 $\text{Border Series}$，跳 $\text{slink}$ 链的复杂度为 $O(\log|S|)$。

下面这张图可以表示一个节点 $x$ 的失配链的状态。

![Fail Chain](/img/2025/12/ps1.jpeg)

下面我们会用 $\text{slink}$ 链表示 $x\sim \text{slink}(x)$，其中不包含 $\text{slink}(x)$ 的所有节点。可以直观的看出，它们的 $\text{dif}$ 值都是一样的，接下来我们证明两个结论。

1. 一条 $\text{slink}$ 链上非链底的所有节点，若最后一次出现位置为 $p$，则倒数第二次出现位置为 $p-d$，其中 $x$ 是这条链上的任意节点，$d$ 是这条链上的 $\text{dif}$ 值。

   直观地看，就是这两个绿串的上一次出现的位置是紫串位置。

   ![Conclusion 1](img/2025/12/ps2.jpeg)

   设 $x$ 和 $y=\text{link}(x)$ 是两个在这条链上的节点，根据文章开头所说，$y$ 是 $x$ 的最大 $\text{Border}$，因此我们知道 $y$ 一定在 $p-\text{dif}(x)$ 出现了，那我们只需要证 $p-d+1\sim p-1$ 都没出现即可。

   由于 $x,y$ 的 $\text{dif}$ 值相等，那么 $2d<\text{len}(x)$ 则 $\text{len}(y)=\text{len}(x)-d>\cfrac{\text{len}(x)}{2}$。

   假设 $y$ 在 $t\in[p-d+1,p-1]$ 位置出现了，那么 $y$ 就会是 $S[t-\text{len}(y)+1,p]$ 的一个 $\text{Border}$，又因为它的长度超过了这个子串的一半，所以这个子串就是回文的。它的长度比 $\text{len}(y)$ 更长，这与 $y=\text{link}(x)$ 矛盾。

2. 设一条 $\text{slink}$ 链的链底为 $x$，若 $\text{slink}(x)\neq\text{link}(x)$，设 $y=\text{link}(x)$，$\text{pos}(y)$ 是 $y$ 最后出现的位置，那么 $y$ 在 $\text{pos}(y)-\text{dif}(y)$ 这个位置一定是某个链的链底。

   ![Conclusion 2](img/2025/12/ps3.jpeg)

   如果存在，说明上图中的 $S_z$ 串存在，并且我们可以推出 $S_z=S_x$，可以进一步推出 $S_a$ 串的存在，那么 $a$ 一定在这个 $\text{slink}$ 链中，并且在 $x$ 的下面，此时 $x$ 就不是链底了，推出矛盾。 

这两个结论的用处我们会在下面的例题中感受到。

## [ICPC2019 Shenyang] M

全网找不到原始题面，可能是这一场锅太多了，代码中给出了牛客的一个可以交的链接。

设 $f_{k,i}$ 表示前 $i$ 个字符中选了 $k$ 个回文串，答案的最大值。

转移方程显然是 $f_{k,i}=\max_{S[j,i]=S[j,i]^R}\{f_{k-1,{i-(i-j+1)}}+(i-j+1)\}$，其中 $k=2,3$；当 $k=1$ 时，$f_{1,i}=\text{len}(x)$，其中 $x$ 是 $i$ 位置对应的自动机上的节点。

观察我们上面的转移，实际上钦定了最后一个串以 $i$ 结尾，所以我们要给 $f_{k,i}$ 与 $f_{k,i-1}$ 取一个前缀最大值，保证是前 $i$ 个

但是，我们不能暴力跳失配链，拿出全 $\texttt{a}$ 串就能卡到 $O(|S|^2)$ 的复杂度。

所以我们需要一个辅助数组 $g_u$，表示 $\text{slink}$ 链底 $u$ 到顶这一段的信息。

我们在每个位置会爬 $\text{slink}$ 链更新每个链底的 $g$ 值，并且将 $g$ 贡献给 $f$。

根据文章开头证明的第一个结论，如果 $\text{slink}(u)\neq \text{link(u)}$，那么 $g_{\text{link}(u)}$ 如果被更新过，它与 $g_u$ 只差了一个 $\text{slink}(u)$ 下面那个节点对应的 $f$ 值。

根据第二条结论，$\text{link}(u)$ 在 $p-d$ 的位置一定被更新了，于是这样转移是可靠的。

具体地说，$g_{k,u}=\max_{v}\{f_{k-1,i-\text{len}(v)}+\text{len}(v)\}$，其中 $v$ 是 $u$ 的 $\text{slink}$ 链上所有节点。

当我们从 $g_{k,\text{link}(u)}$ 更新 $g_{k,u}$ 时，所有的 $\text{len}$ 都对应少了一个 $\text{dif}$。根据 $\max$ 与加法的分配律，我们可以在更新完 $g_{k,u}$ 后统一加上这一个 $\text{dif}$，具体地说：

```cpp
void update() {
    f1[tot] = max(f1[tot-1], len[last]);
    f2[tot] = f2[tot-1];
    f3[tot] = f3[tot-1];

    for (int u = last; u; u = slink[u]) {
        // 这里少加一个 dif[u], 在后面统一加上
        g2[u] = f1[tot - len[slink[u]] - dif[u]] + len[slink[u]];
        g3[u] = f2[tot - len[slink[u]] - dif[u]] + len[slink[u]];
        
        if (slink[u] != Link[u]) {
            g2[u] = max(g2[u], g2[Link[u]]);
            g3[u] = max(g3[u], g3[Link[u]]);
        }

        g2[u] += dif[u];
        g3[u] += dif[u];

        f2[tot] = max(f2[tot], g2[u]);
        f3[tot] = max(f3[tot], g3[u]);
    }
}
```

注意，我们每次都是给 $g$ 数组重新赋值。

[AC Code](https://gist.github.com/hikariyo/bf2b81acb44378a807f750c1319a5957)。

## [CF932G] Palindrome Partition

首先这个题意需要进行一步转化。

由于 $t_1=t_k,t_2=t_{k-1},\dots,t_{\frac{k}{2}}=t_{\frac{k}{2}+1}$，设 $t_1=a_1a_2\dots a_m$，$t_k=b_1b_2\dots b_m$。

那么考虑下面的拼接 $a_1b_ma_2b_{m-1}a_3b_{m-2}\dots a_{m-1}b_2a_{m}b_1$，那么 $t_1=t_k$ 等价于该拼接串是偶回文串。

因此，按照这种方式重排 $S$，问题就变成了，$S$ 划分为偶回文串的方案数。

枚举每个位置的回文后缀，我们显然可以写出一个转移方程：$f_i=\sum_{S[j,i]\text{ is valid}}f_{i-(i-j+1)}$。

但是，这个题目并不能简单地更新 $g_u$ 时判断 $\text{slink}(u)$ 下面的那个节点长度是不是偶数，因为考虑下面这种情况：
$$
\begin{aligned}
\Delta &\texttt{b}\\
&\texttt{b}\texttt{ba}
\end{aligned}\quad \Rightarrow\quad 
\begin{aligned}
\Delta &\texttt{b}\\
\Delta\texttt{b}&\texttt{b}\\
\texttt{b}&\texttt{ba}

\end{aligned}
$$
这里后面 $\texttt{bb}$ 的贡献继承自 $\texttt{b}$ 的贡献，所以我们不能在自动机的节点上做文章，而应当在 $f$ 上做文章。

注意到，奇数位置的 $f_i$ 一定是 $0$，因此每一个 $f_i$ 只能从偶数位置的 $f_j$ 贡献而来。

于是只要是非零贡献，就一定是通过偶数长度的回文串转移而来的，所以我们在 $g$ 数组中无需区份回文串的奇偶性。

[AC Code](https://gist.github.com/hikariyo/f9559dc44429addb098edb5259f3f54c)。
