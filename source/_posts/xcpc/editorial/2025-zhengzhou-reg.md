---
date: 2026-10-01 13:36:00
updated: 2026-10-01 16:55:00
title: "题解 - The 4th Universal Cup. Stage 7: Grand Prix of Zhengzhou"
katex: true
tags:
- Algo
- C++
categories:
- XCPC
description: 2025 CCPC 郑州站题解。
---

## M

分类讨论一下，下面下标为 $1$ base 的。

如果 $s$ 是全 $0$ 串，那么答案是 $0$；否则找到第一个 $1$ 的位置，假设这个位置是 $p$。

1. 若 $p=1,2$，当 $n$ 充分大的时候，选择 $n-1$ 位置一定会使得答案至少有 $n-p-1$ 位；

   可能比 $n-1$ 位置更优的只有 $p+1$ 位置，所以只需要把开头和结尾附近取几个位置 $O(n)$ 算一下即可。

   当 $n$ 比较小的时候，直接 $O(n^2)$ 暴力即可。

2. 若 $p\ge 3$，选择 $2\sim p-1$ 的位置都是一样的，选 $p$ 右边的情况同上。

复杂度 $O(n)$。

## B

观察到每一坐标我们切的位置都是固定的，例如 $x$ 坐标我们只能按照 $n/(p+1)$ 个一组的方式去切，并且在两组缝隙间移动并不会改变最终分组的结果。

所以我们只需要给每一个人打好它位于哪一组的标记，最后看所有组是否都真的平分了即可。

由于答案想要合法，必须有 $(p+1)(q+1)(r+1)\mid n$，因此这三个东西乘起来是不超过 $n$ 的，可以直接混合进制的方式去记录每一组。

复杂度 $O(n\log n)$，瓶颈在于排序。

## G

令 $k$ 是满足 $2^k>b$ 的最小的 $k$，那么我们可以在模 $m=\operatorname{lcm}(2^k,b)$ 意义下去做同余最短路。

从原理上来讲，设 $dp_u$ 表示到达 $x\equiv u\pmod m$ 时 $x$ 的最小值，有转移式：
$$
\begin{aligned}
dp_{u\oplus b}&\gets \min(dp_{u\oplus b}, dp_u\oplus b)\\
dp_{(u+b)\bmod m} &\gets \min(dp_{(u+b)\bmod m},dp_u+b)
\end{aligned}
$$
为什么这仍然是一个最短路的形式？因为 $dp_u\oplus b$ 实际上可以看作 $dp_u + b-2(dp_u\land b)=dp_u+b-2(u\land b)$，这是因为 $dp_u\equiv u\pmod m$，而 $2^k\mid m$，所以 $b\land dp_u=b\land u$。

即 $u$ 到 $u\oplus b$ 的这条边权并不依赖于 $dp_u$ 的具体值，所以这是一个最短路的形式。

假设我们求出了所有 $dp$ 值，那么 $dp_{c\bmod m}\le c$ 就是能到达 $c$ 的充要条件：如果 $dp_{c\bmod m}>c$，说明到达 $c\bmod m$ 这个等价类的最小值比 $c$ 大，那么 $c$ 不可达；如果 $dp_{c\bmod m}\le c$，那么我们最后再加若干倍的 $b$ 就可以到达 $c$。

现在的问题在于，这个图具有负权边。直接 SPFA 也是可以通过的，因为图形状特殊，出题人造不出卡掉 SPFA 的数据。

但是仍然可以进行 Dijkstra，只不过我们需要修改边权。观察到不会进行连续两次异或，所以我们把第一种边改成 $u\to ((u\oplus b)+b)\bmod m$，最后检查 $c\bmod m$ 和 $c\oplus b\bmod m$ 即可，这样就没有负权边了。

这是一个 $m$ 个点，$2m$ 条边的图，复杂度 $O(m\log m)$，其中 $m=O(b^2)$。

## J

根据经典结论，对 $S_n=\bigoplus_{i=1}^n i$ 有 $S_{4k-3}=1,S_{4k-2}=4k-1,S_{4k-1}=0,S_{4k}=4k$。

那么 $x\oplus (x+1)\oplus (x+2)\oplus (x+3)=S_{x+3}\oplus S_{x-1}$，那么当 $x$ 为偶数时，这个值为 $0$；$x$ 为奇数时，这个值为 $(4k-1)\oplus (4k+3)$ 或者 $4k\oplus (4k+4)$，总之不等于 $0$。

而 $c_{i,j}\oplus c_{i,j+1}\oplus c_{i+1,j}\oplus c_{i+1,j+1}$ 代入定义后应该为 $0$，因此 $x$ 一定是偶数。

既然 $x$ 为偶数，那么 $x+1=x\oplus 1$，$x+3=(x+2)\oplus 1$，所以 $b_j\oplus b_{j+1}=1$ 即可满足两列间的关系。

在选择这样的 $b_j$ 的前提下，我们只需要满足 $c_{i+1,j}=c_{i,j}+2$ 即可，那么 $c_{i+1,j}\oplus c_{i,j}=a_i\oplus a_{i+1}$，那么我们可以根据 $a_i\oplus a_{i+1}=x\oplus (x+2)$ 推断出 $x$ 应该是什么样的。

具体地说，由于 $x$ 是偶数，设 $x$ 的二进制从低到高位是 $x_0x_1x_2\dots x_px_{p+1}$，其中 $x_0=0,x_1=x_2=\dots=x_p=1,x_{p+1}=0$，即它从第 $1$ 位开始有 $p$ 个 $1$，那么 $x\oplus (x+2)$ 就是 $2^{p+2}-2$，其中 $p\ge 0$。

那么我们根据 $a_i\oplus a_{i+1}$ 可以得到 $p$，此时就只剩下要求 $x$ 的 $0\sim p+1$ 位符合我们的要求。

只需要查询符合要求的 $b_j$ 的数量即可，我们可以用哈希表对 $b$ 的每一种前缀二进制位开桶。

复杂度 $O(n\log V)$。

## I

设 $A_i=\{g(p_i)\}$，令 $c_g$ 表示 $g(p)=g$ 的 $p$ 的数量，即求：
$$
\begin{aligned}
\mathbb E\left[\left|\bigcup_{i=1}^k A_i \right|\right]&=\sum_{\varnothing\neq T\subseteq [k]} (-1)^{|T|-1}\mathbb E\left[\left|\bigcap_{i\in T} A_i \right|\right]\\
&=\sum_{\varnothing\neq T\subseteq [k]}(-1)^{|T|-1} \Pr(A_{T_1}=A_{T_2}=\cdots=A_{T_{|T|}})\\
&=\sum_{\varnothing\neq T\subseteq [k]}(-1)^{|T|-1} \frac{1}{(n!)^{|T|}}\sum_{g} c_g^{|T|}\\
&=\sum_{t=1}^k \binom{k}{t}(-1)^{t-1} \frac{1}{(n!)^{t}}\sum_{g} c_g^t
\end{aligned}
$$
记 $s_{t,i}$ 表示长度为 $n=i$ 时，$s_{t,i}=\sum_g c_g^t$。

打表程序如下：

```cpp
vector<int> get_g(vector<int> p) {
  int mx = -1;
  vector<int> ret;
  for (int x: p) {
    if (x > mx) ret.push_back(x);
    mx = max(x, mx);
  }
  sort(ret.begin(), ret.end());
  return ret;
}

void solve() {
  int n;
  cin >> n;
  vector<int> p(n);
  iota(p.begin(), p.end(), 1);
  
  map<vector<int>, int> c;
  do {
    auto g = get_g(p);
    c[g]++;
  } while (next_permutation(p.begin(), p.end()));

  cerr << "n=" << n << "\n";
  for (auto [_, cg]: c) {
    cerr << cg << " ";
  }
  cerr << endl;
}
```

这里贴出部分输出：

```text
n=1
1 
n=2
1 1 
n=3
1 1 2 2 
n=4
1 1 2 2 3 3 6 6 
n=5
1 1 2 2 3 3 6 6 4 4 8 8 12 12 24 24 
n=6
1 1 2 2 3 3 6 6 4 4 8 8 12 12 24 24 5 5 10 10 15 15 30 30 20 20 40 40 60 60 120 120 
```

可以观察出，当 $n\gets i+1$ 时，相当于所有 $c_g$ 乘 $i$ 拼到原本的 $c_g$ 后面，那么 $s_{t,i+1}=(1+i^t)s_{t,i}$。

复杂度 $O(nk)$。

## D

简单引入一下本题需要的结论。

假设 $A,B$ 是一对直径端点，定义 $\operatorname{ecc}(x)$ 表示点 $x$ 到最远点的距离，那么 $\operatorname{ecc}(x)=\max(d(A,x),d(B,x))$，这里不对这个结论证明。

现在我们关心 $\arg\min_x \operatorname{ecc}(x)$，设以 $x$ 为根时 $u=\operatorname{lca}(A,B)$，那么 $d(A,x)=d(x,u)+d(u,A)$ 且 $d(B,x)=d(x,u)+d(u,B)$，于是 $\operatorname{ecc}(x)=d(x,u)+\max(d(u,A),d(u,B))$。

想要让 $\operatorname{ecc}(x)$ 最小，一定有 $d(x,u)=0$，即 $A,B$ 经过 $x$；那么又因为 $d(u,A)+d(u,B)=D$，其中 $D$ 为直径，所以：

1. 当 $D$ 为偶数时，$\arg\min_x \operatorname{ecc}(x)$ 唯一处于 $AB$ 中点；
2. 当 $D$ 为奇数时，$\arg\min_x \operatorname{ecc}(x)$ 唯二处于 $AB$ 中间边的两侧。

假设我们选定了 $A,B$，那么这个点是唯一/唯二确定的。此时我们再选择别的直径，可以得到同样的结论，但是点本身已经被确定了，所以这些直径交于一点或一边。

1. 当 $D$ 为偶数时，我们以这个唯一点 $u$ 为根，先一遍 DFS 找到所有直径端点，假设有 $t$ 个，那么我们给这些直径端点分配 $1\sim t$ 就能逼迫程序选择 $t$ 作为起点并且到达 $u$，因此前半部分答案就是 $t,t+1,\dots, t+D/2$。

   此时从 $u$ 出发，我们只需要枚举最终走的所有可能情况就行了，并且对答案进行一个动态的更新。

   具体地说，由于程序一定会选择存在直径端点的子节点走，我们想逼程序走这里，设 $c=$ 当前节点存在直径端点的子节点的数量，并且之前已经分配了 $x$ 个值，那么就给你想逼迫的节点分配一个 $x+c$ 即可。

   注意如果已经走到叶子，那么此时就不是 $x+c$ 而是 $c$，这是因为我们一开始就给直径端点分配了 $1\sim t$。

   在进行枚举的过程中，我们需要动态更新最优的字典序。设当前位置为 $p$，当它优于之前的最优值时，我们需要给 $p+1\sim D$ 重新赋为无穷，这样做是因为字典序的定义；当它与之前的最优值相等时，我们不进行操作；当他劣于时，我们直接剪枝。

   这里只需要用一个树状数组即可做到，赋值为无穷的操作我们只需要给 $p+1\sim D$ 区间加一个 `INF` 就可以做到。

2. 当 $D$ 为奇数时，基本和偶数的情况完全相同，只不过需要两边都枚举一遍。

复杂度 $O(n\log n)$。

## K

我这个题目一开始想打表，但是打表打出来的 $p$ 没有任何用处，结果是一个构造 + 交换论证法的贪心。

那怎么贪心呢？设当前放了若干个左括号，我们来看当前位置 $i$ 选择右括号还是选择左括号。

1. 如果选择左括号，设当前栈顶是 $(j,w,q)$，表示上一个左括号的位置，它要求的右括号位置的值以及那个右括号最右边的下标。

   那么当前的合法位置就是 $<q$ 且与 $i$ 奇偶性不同的下标，我们会选择合法位置中最小值 $v$，如果有多个最小值则选择最右边的，设这个位置是 $p$。

   如果真的这样选择了，那么我们会在栈中压入一个 $(i,v,p)$。

   那么这样选择后，当前位置最终会是 $v$。

2. 如果选择右括号，设当前栈顶是 $(j,w,q)$，那么需要满足 $a_i=w$，$i\le q$，且 $j,i$ 奇偶性不同。

   那么这样选择后，当前位置最终会是 $a_j$。

3. 如果两种选择放到当前的值相等，我们会选择右括号，这样会让未来的约束更宽松；否则，我们选择更小的那个值。

主要的问题是，这样做真的能闭合掉所有左括号吗？答案是肯定的，因为栈顶会对你当前尝试放左括号进行很强的约束，你不可能到达它要求的位置之后还没有闭合掉这个左括号。并且我们每一个选择都做到了让未来最宽松。

复杂度 $O(n\log n)$。

## H

设 $\mathbb E[X_i]$ 为后缀 $i\sim n$ 的期望得分，$S$ 为 $c$ 的后缀和，那么容易写出转移：
$$
\begin{aligned}
\mathbb E[X_i]&=\sum_{\omega} \Pr(\omega)\max_{k=0}^{\operatorname{lcp}(a,b)}\left\{S_i-S_{i+k}+\mathbb E[S_{i+k}-X_{i+k}]\right\}\\
&=\sum_{\omega} \Pr(\omega)\max_{k=0}^{\operatorname{lcp}(a,b)}\left\{S_i-\mathbb E[X_{i+k}]\right\}\\
&=S_i-\sum_{\omega} \Pr(\omega)\min_{k=0}^{\operatorname{lcp}(a,b)}\mathbb E[X_{i+k}]\\
\end{aligned}
$$
简化记号，记 $f_i=\mathbb E[X_i]$。这个转移最难处理的地方在于 $f_i$ 自己依赖自己。

我们倒序进行 DP，此时 $f_{i+1}\sim f_n$ 的值都是已知的，这相当于解一个关于 $f_i$ 的方程。

由于 $f_i$ 增大时，左式严格增大，右式非严格减小，所以一定是有唯一解的。

可以维护一个单调栈记录后缀的前缀最小值，枚举当前的值在单调栈的哪一个位置，解出来后判断它是否真的在这个位置即可。

此时还有一个问题，我们如何进行快速求值？即我们需要知道到达单调栈的一个位置时，它对应的概率。

设 $p_i=\cfrac{c_{i,a_i}}{n-i+1}$，其中 $c_{i,a_i}$ 表示 $i\sim n$ 后缀中 $a_i$ 出现的次数，那么 $\operatorname{lcp}(a,b)\ge k$ 的概率就是 $\prod_{j=0}^{k-1} p_{i+j}$。

假设当前 $i$ 作为最小值一直管辖到 $k$，并且 $V_k=\sum_{\omega} \Pr(\omega)\min_p \mathbb E[X_{k+p}]$，那么就有：
$$
f_i=S_i-\left(V_k\prod_{j=i}^{k-1} p_{j}+f_i\left(1-\prod_{j=i}^{k-1}p_{j}\right)\right)
$$
整理得到：
$$
f_i=\frac{S_i-V_k\prod_{j=i}^{k-1} p_j}{2-\prod_{j=i}^{k-1} p_j}
$$
我们只需要在单调栈中记下当前位置到栈中下一个位置左闭右开的 $\prod p$ 以及 $V$ 即可，注意这里千万不要记录后缀积，精度丢失很严重。

复杂度 $O(n)$。

## C

设 $dp(u,x,y)$ 表示与 $u$ 子树中选择了 $x$ 个关键点，钦定 $u$ 是一个关键点，这些关键点中有 $y$ 个与 $u$ 相连。

定义 $s_u$ 表示 $u$ 子树的大小，拓展这个状态：

+ $y=0$ 时，认为 $u$ 不是关键点，并且此时 $dp(u,x,0)=\binom{s_u-1}{x}$；

+ $x=0$ 时，认为这颗子树什么都不选，并且此时 $dp(u,0,0)=1$，这是符合 $y=0$ 时的定义的。
+ $y>x$ 时，$dp(u,x,y)=0$。

这主要是方便进行转移。设 $u$ 有 $k$ 个子节点 $v_1,v_2,\dots,v_k$，那么当 $x>0,y>0$ 时：
$$
dp(u,x,y)=\sum_{a_1+\cdots+a_k=x-1}\sum_{b_1+\cdots+b_k=y-1} \prod_{p=1}^k dp(v_p,a_p,b_p)
$$
看上去这是一个二重的树上背包，硬做应当是 $O(n^4)$ 的，无法通过。

观察如果我们知道了 $dp(u,x,y)$，如何求出答案。对于一个 $p_i=j$ 的限制，我们计算答案 $A$ 时，先枚举选了几个关键点，再枚举相连的数量，再给子树中关键点分配 $1\sim j-1$ 的数值，非关键点分配 $j+1\sim n$ 的数值，子树外边进行一个全排列即可：
$$
\begin{aligned}
A&=\sum_{x=1}^{s_i} \sum_{y=1}^{x} dp(i,x,y)q_y\binom{j-1}{x-1}\binom{n-j}{s_i-x}(x-1)!(s_i-x)!(n-s_i)!\\
&=\sum_{x=1}^{s_i} \binom{j-1}{x-1}\binom{n-j}{s_i-x}(x-1)!(s_i-x)!(n-s_i)!\left(\sum_{y=1}^{x}dp(i,x,y)q_y\right)
\end{aligned}
$$
设 $dp(u,x)$ 维护第三维的多项式 $dp(u,x)=\sum_{y} dp(i,x,y)z^y$，这里我们维护 $z=1\sim n$ 的点值，那么当维护点值 $z_0$ 的时候，转移就可以简写为：
$$
dp(u,x)=\sum_{a_1+\cdots+a_k=x-1} z_0\times \prod_{p=1}^k \left(\binom{s_u-1}{a_p}+dp(v_p,a_p)\right)
$$
我们枚举每一个 $z_0$，然后做一个 $O(n^2)$ 的树上背包即可获取每一个 $u,x$ 在 $z_0$ 处的点值。

注意我们的点值本身不包括 $y=0$ 的项，这是为了后面计算答案的时候方便。

那么我们现在枚举一个 $u,x$ 时，相当于知道了：
$$
\begin{bmatrix}
1^1& 1^2& \cdots & 1^n\\
2^1& 2^2& \cdots & 2^n\\
\vdots & \vdots & \cdots & \vdots\\
n^1&n^2&\cdots& n^n
\end{bmatrix}

\begin{bmatrix}dp(u,x,1)\\dp(u,x,2)\\\vdots\\dp(u,x,n)\end{bmatrix}=

\begin{bmatrix}
dp(u,x)_{z=1}\\
dp(u,x)_{z=2}\\
\vdots\\
dp(u,x)_{z=n}
\end{bmatrix}
$$
我们想要的值是 $\sum_{y=1}^n dp(i,x,y) q_y$，即：
$$
\begin{bmatrix}q_1&q_2&\cdots&q_n\end{bmatrix}
\begin{bmatrix}dp(u,x,1)\\dp(u,x,2)\\\vdots\\dp(u,x,n)\end{bmatrix}
$$
所以我们只需要知道：
$$
\begin{bmatrix}q_1&q_2&\cdots&q_n\end{bmatrix}\begin{bmatrix}
1^1& 1^2& \cdots & 1^n\\
2^1& 2^2& \cdots & 2^n\\
\vdots & \vdots & \cdots & \vdots\\
n^1&n^2&\cdots& n^n
\end{bmatrix}^{-1}
$$
即可，直接矩阵求逆即可，这是一个范德蒙德矩阵，它的行列式一定不为 $0$，所以逆一定存在。

求出这个向量后，我们每次 $O(n)$ 就可以求出每一个答案。

复杂度 $O(n^3)$。
