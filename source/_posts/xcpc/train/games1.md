---
date: 2026-03-12 21:05:00
updated: 2026-03-13 12:30:00
title: 专题训练 - 博弈 I
katex: true
tags:
- Algo
- C++
- Games
categories:
- XCPC
description: 10 道带有 Games 标签的 CF 题目，施工中。
---

## [CF1396B] Stoned Game

首先我们要观察出来一个显然的必胜态，设 $S=\sum a_i$，如果存在一个 $a_i>\cfrac{S}{2}$，那么先手只要一直选 $a_i$ 就能够获胜了，接下来我们来看 $\forall i,a_i\le \cfrac{S}{2}$ 的情况。

由于每个人都不会想给对面留下一个 $\exists i,a_i>\cfrac{S}{2}$ 的局面（因为这样对面必胜），所以每一步必然就会尝试使得 $\forall i,a_i'\le \cfrac{S'}{2}$。

实际上也确实可以做到这一点：只要每次给最大的 $a_i$ 拿走 $1$ 即可，可以分为最大值有 $1$ 个和 $\ge 2$ 个简单讨论得出。

既然 $\forall a_i,a_i\le \cfrac{S}{2}$，所以每一步必然会有至少两个堆，所以答案只与 $S$ 的奇偶性相关。

## [CF1451D] Circle Game

由于局面是高度对称的，所以我们可以往对称轴上来思考。

观察圆内最远能到达的直线 $y=x$ 上的点 $(mk,mk)$，即 $2m^2k^2\le d^2$ 并且 $2(m+1)^2k^2>d^2$，我们来看 $((m+1)k,mk)$ 是否在圆内。

1. 若 $((m+1)k,mk)$ 不在圆内，那么对称地 $(mk,(m+1)k)$ 也不在，于是 $(mk,mk)$ 是 $P$ 态。

   那么 $((m-1)k,mk)$ 是 $N$ 态，$(mk,(m-1)k)$ 也是 $N$ 态，就能推出 $((m-1)k,(m-1)k)$ 是 $P$ 态。

   以此类推，$(0,0)$ 是 $P$ 态。

2. 若 $((m+1)k,mk)$ 在圆内，那么它不可能向上走，那能不能向右？$((m+2)^2+m^2)k^2>2(m+1)^2k^2>d^2$，所以也不能向右，则 $((m+1)k,mk)$ 是 $P$ 态，对称地 $(mk,(m+1)k)$ 也是 $P$ 态，于是 $(mk,mk)$ 是 $N$ 态。

   因为 $(mk,(m+1)k)$ 是 $P$ 态，则 $((m-1)k,(m+1)k)$ 是 $N$ 态，于是 $((m-1)k,mk)$ 是 $P$ 态，则 $((m-1)k,(m-1)k)$ 是 $N$ 态。

   以此类推，$(0,0)$ 是 $N$ 态。

## [CF2006A] Iris and Game on the Tree

显然我们可以把这个 $\texttt{01}$ 和 $\texttt{10}$ 看作这个串的 $\texttt{0}/\texttt{1}$ 切换时权值会 $+1/-1$，于是如果最后一个字符和第一个字符相同，那么这个点的权（前面提到的权值的绝对值）是 $0$，否则是 $1$，这就和中间节点无关了。

1. 如果根不是 $\texttt{?}$，那么每个人都会尽量把叶子染成自己希望的那个颜色，设未定的叶子个数是 $m$，那么答案就是既定权值 $+\lceil m/2\rceil$。

2. 如果根是 $\texttt{?}$，设未定的叶子个数是 $m$，
   + 如果既定的 $0$ 和 $1$ 数量 $x_0,x_1$ 不等，设基础权值是 $\min(x_0,x_1)$，那先手总可以用一步定根获得 $|x_0-x_1|\ge 1$ 的收益，如果选叶子的收益 $\le 1$，所以一定会先定根，即答案是 $\max(x_0,x_1)+\lfloor m/2\rfloor$
   + 如果既定的 $0$ 和 $1$ 数量 $x_0,x_1$ 相等，那么定根的收益就是 $0$ 了。如果此时某人选了一个未定的叶子，就会导致对方定根的收益为 $1$，这是我们不希望的；所以我们会选不是未定叶子也不是根的未定节点，直到必须定根，那么答案就是 $x_0+\lfloor (m+y)/2\rfloor$，其中 $y$ 是不是未定叶子也不是根的节点的未定节点数量模 $2$。

具体的证明过程如下：

1. 不妨设根为 $1$，叶子颜色为 $0$ 的有 $L_0$ 个，同理 $L_1,L_{?}$，再设 $I_?$ 是中间的未定节点数量。

   设 $V_I(L_0,L_?,I_?)$ 为 Iris 先手时最终得分，$V_D$ 同理，那么状态转移：
   $$
   V_I(L_0,L_?,I_?)=\max \begin{cases}
   V_D(L_0+1,L_?-1,I_?)\quad &L_?\ge 1\\
   V_D(L_0,L_?,I_?-1)\quad &I_?\ge 1\\
   \end{cases}
   $$
   同理，$V_D$ 的转移如下：
   $$
   V_D(L_0,L_?,I_?)=\min\begin{cases}
   V_I(L_0,L_?-1,I_?)\quad &L_?\ge 1\\
   V_I(L_0,L_?,I_?-1)\quad &I_?\ge 1
   \end{cases}
   $$
   我们来用数学归纳法证明 $L_?+I_?$ 为任意自然数时，都有 $V_I(L_0,L_?,I_?)=L_0+\lceil L_?/2\rceil$，$V_D(L_0,L_?,I_?)=L_0+\lfloor L_?/2\rfloor$。

   当 $L_?+I_?=0$ 时，显然成立，接下来我们进行归纳步：

   对于 $V_I$，我们有：
   $$
   \begin{aligned}
   V_I(L_0,L_?,I_?)&=\max\left\{L_0+1+\left\lfloor\frac{L_?-1}{2}\right\rfloor,L_0+\left\lfloor\frac{L_?}{2}\right\rfloor\right\}\\
   &=L_0+\left\lceil\frac{L_?}{2}\right\rceil
   \end{aligned}
   $$
   对于 $V_D$，我们有：
   $$
   \begin{aligned}
   V_D(L_0,L_?,I_?)&=\min\left\{ L_0+\left\lceil
   \frac{L_?-1}{2}\right\rceil,L_0+\left\lceil\frac{L_?}{2}\right\rceil\right\}\\
   &=L_0+\left\lceil\frac{L_?-1}{2}\right\rceil\\
   &=L_0+\left\lfloor\frac{L_?}{2}\right\rfloor
   \end{aligned}
   $$
   证毕。

2. 如果根没有被定，那么我们的状态就应当包含 $L_0,L_1,L_?,I_?$ 了，因为我们总是可以选择下一步是否定根，定根之后就有直接的公式可以计算，所以是否定根可以给我们一个下界。

   说起来比较抽象，写成公式的话，就是：
   $$
   V_I(L_0,L_1,L_?,I_?)=\max\begin{cases}
   V_D(L_0,L_?,I_?)=L_0+\lfloor L_?/2\rfloor\\
   V_D(L_1,L_?,I_?)=L_1+\lfloor L_?/2\rfloor\\
   V_D(L_0,L_1,L_?,I_?-1)\\
   V_D(L_0+1,L_1,L_?-1,I_?)\\
   V_D(L_0,L_1+1,L_?-1,I_?)
   \end{cases}
   $$
   同理，对于 $V_D$，我们有：
   $$
   V_D(L_0,L_1,L_?,I_?)=\min\begin{cases}
   V_I(L_0,L_?,I_?)=L_0+\lceil L_?/2\rceil\\
   V_I(L_1,L_?,I_?)=L_1+\lceil L_?/2\rceil\\
   V_I(L_0,L_1,L_?,I_?-1)\\
   V_I(L_0+1,L_1,L_?-1,I_?)\\
   V_I(L_0,L_1+1,L_?-1,I_?)
   \end{cases}
   $$
   也就是说，起码可以列出不等式：
   $$
   V_D(L_0,L_1,L_?,I_?)\le \min(L_0,L_1)+\lceil L_?/2\rceil\\
   V_I(L_0,L_1,L_?,I_?)\ge \max(L_0,L_1)+\lfloor L_?/2\rfloor
   $$

   1. 当 $L_0\neq L_1$ 时，我们总可以证明直接定根是更优的，即，$I$ 做出别的选择不会比直接定根更优，因为：
      $$
      \max\{\text{other choice}\}\le \max\begin{cases}
      
      \min(L_0,L_1)+\lceil\frac{L_?}{2}\rceil\\
      \min(L_0+1,L_1)+\lceil\frac{L_?-1}{2}\rceil\\
      \min(L_0,L_1+1)+\lceil\frac{L_?-1}{2}\rceil\\
      \end{cases}
      $$
      简单讨论就可以知道不会更优，所以一定会直接定根，即 $V_I(L_0,L_1,L_?,I_?)=\max(L_0,L_1)+\lfloor L_?/2\rfloor$，同理 $V_D(L_0,L_1,L_?,I_?)=\min(L_0,L_1)+\lceil L_?/2\rceil$

   2. 当 $L_0=L_1=L$ 时，不难发现如果使用了上面五行情况的最后两行，对面是可以直接套用我们得出的公式的，写出来化简后会发现前两行的值和后两行的值完全相等，所以此时的状态只剩下了 $I_?$ 一种，即：
      $$
      \begin{aligned}
      V_I(I_?)&=\max\{L+\lfloor L_?/2\rfloor,V_D(I_?-1)\}\\
      V_D(I_?)&=\min\{L+\lceil L_?/2\rceil,V_I(I_?-1)\}
      \end{aligned}
      $$
      关键点其实就是要看我们能不能逼迫 $V_D$ 给出 $L+\lceil L_?/2\rceil$，容易发现这只与 $I_?$ 的奇偶性有关，证明可以用数学归纳法，所以双方能拿 $I_?$ 会尽可能地拿 $I_?$。

## [CF1404B] Tree Tag

首先，我们来证明一个树上的结论：$\min_u(\max _vd(u,v))=\lceil D/2\rceil$，其中 $D=\max_u(\max_v d(u,v))$。

为什么？我们首先来构造出一个 $\max_v d(u,v)=\lceil D/2\rceil$ 的情况，接下来我们证明不能比它更小。

1. 令 $a,b$ 是某个直径的两个端点，我们只需要 $a,b$ 的中点作为 $u$，即可使得 $\max_v d(u,v)=\lceil D/2\rceil$，于是 $\min_u \max_v d(u,v)\le \lceil D/2\rceil$。

2. 如果存在一个 $u$，使得 $\max_v d(u,v)<\lceil D/2\rceil$，那么我们考虑任意一个直径的两个端点 $(a,b)$，会有 $d(u,a)+d(u,b)\le 2\lceil D/2\rceil-2$，根据三角不等式，我们有 $d(a,b)\le d(u,a)+d(u,b)\le 2\lceil D/2\rceil-2$。

   + 如果 $D$ 是偶数，那么 $d(a,b)\le 2\cfrac{D}{2}-2=D-2$，矛盾；
   + 如果 $D$ 是奇数，那么 $d(a,b)\le 2\cfrac{D+1}{2}-2=D-1$，矛盾。

   因此这样的 $u$ 不可能存在，所以 $\min_u \max_v d(u,v)=\lceil D/2\rceil$。

然后我们来看这道题目，考虑如下几种特殊的情况：

1. 如果一开始 $d(a,b)\le d_a$，那么一步就可以抓住；

2. 如果存在一个点 $u$，使得 $\max_v d(u,v)\le d_a$，那么只要把 $a$ 移动到 $u$ 就可以抓住，根据前文的讨论，这等价于 $\min_u\max_v d(u,v)=\lceil D/2\rceil\le d_a$；

3. 否则，无论 $a$ 跳到哪里，总会存在一个点 $v$ 使得 $d(u,v)>d_a$，即 $a$ 够不到，此时只要能保证 $b$ 总是能跳到这样的点上，就永远抓不住 $b$。

   怎么判断 $b$ 总是能够跳到这样的点上？我们不妨把每次的 $a$ 看作根，那么最坏的情况下，$b$ 需要先跳到根，再跳到这样的节点上，那么这一次的跳跃需要的 $d_b>2d_a$。

   如果满足 $d_b>2d_a$，我们就总能逃脱 $a$ 的攻击范围；否则，每一次以 $a$ 为根建树时，$b$ 都不能跳到上面 $d_a$ 层中去，我们只需要让 $a$ 步步紧逼 $b$ 总能把 $b$ 逼到它不得不向根跳的情况。

