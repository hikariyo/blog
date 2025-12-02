---
date: 2025-12-01 21:47:00
update: 2025-12-02 17:54:00
title: 字符串 - ACAM
katex: true
tags:
- Algo
- C++
- String
categories:
- XCPC
description: AC 自动机。
---

## Fail

> 我们把给定的若干模式串插入到字典树 $T$ 中。
>
> 对于某个节点 $u$ 代表的前缀 $S$，类比 KMP 中的 $\text{next}$ 数组，令 $\text{fail}(u)$ 指向最长的、在 $T$ 中的后缀 $S'$ 对应的节点。

类比于 KMP 的思路，对 $\text{fail}(u)$ 扣掉 $u$ 后，一定也在 $fa$ 的 $\text{fail}$ 链上。由于深度为 $1$ 的节点的 $\text{fail}$ 一定是 $0$，那么进行一个 BFS：

```python
push-depth1-nodes()
while q.notempty():
    u = q.pop()
    for ch = 'a' to 'z':
        j = u
        while not tr[j][ch].exist():
            j = fail[j]
        if tr[j][ch].exist():
            j = tr[j][ch]
        fail[tr[u][ch]] = j
```

然而，通常情况下我们不会这么写。我们定义 $u=\text{trans}(fa,ch)$：

1. 如果字典树 $T$ 中存在 $(fa,ch)$ 这条边，那么 $\text{trans}(fa,ch)=T(fa,ch)$，同时令 $\text{fail}(u)=\text{trans}(\text{fail}(fa),ch)$；
2. 如果字典树 $T$ 不存在 $(fa,ch)$ 这条边，那么 $\text{trans}(fa,ch)=\text{trans}(\text{fail}(fa),ch)$。

那么对应到 Cpp 的代码就是：

```cpp
void build() {
    queue<int> q;
    for (int i = 0; i < 26; i++) if (tr[0][i]) q.push(tr[0][i]);
    while (q.size()) {
        int fa = q.front();
        for (int i = 0; i < 26; i++) {
            int& u = tr[fa][i];
            if (!u) u = tr[fail[fa]][i];
            else {
                fail[u] = tr[fail[fa]][i];
                q.push(u);
            }
        }
    }
}
```

## Luogu P5357

初始时令当前节点 $u=0$，意思是空串；每添加一个字符 $ch$，我们就尝试在 $T$ 上走 $(u,ch)$ 这条边：

1. 如果 $T(u,ch)$ 存在，那么直接走下去；
2. 如果 $T(u,ch)$ 不存在，我们需要尝试从 $\text{fail}$ 链上走 $ch$ 边。

由上面的定义，我们知道，这个操作就等价于 $u\gets \text{trans}(u,ch)$。

接下来该怎么做？因为目前的节点代表一个 $S$ 的后缀，这个后缀需要满足：

1. 它是某些模式串的前缀，即它在字典树上；
2. 它是满足上面条件的最长的那个。

这意味着，以当前位置结尾的、所有可能出现的串都是这个后缀的子串，并且由于它们以当前位置结尾，所以它们在 $\text{fail}$ 链上，因此把当前节点沿着 $\text{fail}$ 链贡献上去即可。

看上去可能需要树剖？我们不妨反过来思考，把一个节点向所有祖先贡献，最后算答案，这个过程等价于每个节点对子树中所有可能的贡献求一个和，所以最终 DFS 一次求子树贡献和就可以做出这道题目了。

[AC Code](https://gist.github.com/hikariyo/fc6e7653df1b8b82ca2ca384f6b673de)。

## Nowcoder 14612

从构建过程可以看出，ACAM 是一个离线数据结构，并不支持插入这种操作。

所以我们需要离线下来，把所有的串全部插入后，通过 “解锁” 某些串的贡献来完成类似于在线的操作。

这个题目只要求我们求出所有模式串的出现次数，所以我们考虑使用某些数据结构来维护。

当我们进行查询的操作时，跳转到某个节点，统计的是 $\text{fail}$ 链上的模式串个数，所以每解锁一个字符串相当于给子树 $+1$，这启发我们使用 `dfs` 序来进行统一操作。

具体地说，维护一个树状数组支持区间 $+1$ 单点查询即可。

[AC Code](https://gist.github.com/hikariyo/f9d17332bf14085a8a4acde0ec7d6292)。

## Luogu P14363

听说今年 CSP-S 出了一个 ACAM 的题目，来看看怎么回事。

我在第一时间就有了一个暴力的做法：用 $s_{i,1}$ 建立 ACAM，并在每个串结尾的位置记录下 $s_{i,2}$。对于每个询问，我们把 $t_1$ 放到 ACAM 上匹配，对每个位置暴力跳 $\text{fail}$ 判断修改后是否能和 $t_2$ 相同。

复杂度高的原因是我们不能暴力跳 $\text{fail}$，因此我们需要尝试记录 $\text{fail}$ 链上的信息。

这就引出了一个问题，如何判断当前节点替换 $t_1$ 后是否等于 $t_2$？

1. 当前位置 $t_1,t_2$ 的后缀相等，这一步可以通过 $O(|t|)$ 记录最长公共后缀的位置，每一次 $O(1)$ 判断。
2. 当前位置替换后 $t_1,t_2$ 的前缀 $p_1,p_2$ 相等。

下面是关键的一步：我们把替换操作看成一个向量加减法操作。

具体地说，我们希望 $p_1-s_{i,1}+s_{i,2}=p_2$，这里 $s_{i,1}$ 和 $s_{i,2}$ 前面补 $0$ 直到与前缀长度相等。

为什么这样操作？因为这样可以把询问和 $\text{fail}$ 树上的数据独立出来，即判断有多少个 $s_{i,1}-s_{i,2}=p_1-p_2$。

到了这一步，就可以使用哈希了，但是这个哈希不能被前缀 $0$ 的长度影响，因为我们实际上并不关心这些向量的前缀 $0$ 部分。

尝试使用普通的字符串哈希 $f(S)=\sum_{i=1}^{|S|} S_iB^{|S|-i}$，我们首先需要保证不取模之前这个哈希是不会碰撞的，即 $|S|=|T|$ 时有：
$$
\sum_{i=1}^{|S|}S_iB^{|S|-i}=\sum_{i=1}^{|T|}T_iB^{|T|-i}\Leftrightarrow \forall i=1\sim |S|,S_i=T_i
$$
这个等价于：
$$
\sum_{i=1}^na_iB^{n-i}=0\Leftrightarrow\forall i=1\sim n, a_i=0
$$
由于我们哈希的对象 $-25\le a_i\le 25$，我思考出了一个必要条件：非零最高位决定符号就能做到这一点。

想做到这一点，需要对所有的 $n$ 都满足：
$$
B^{n-1}>25(B^0+B^1+\cdots+B^{n-2})=25\frac{B^{n-1}-1}{B-1}
$$
化简一下可以得到 $B-1>\frac{B^{n-1}-1}{B^{n-1}}25\approx 25$，所以正常取 $B=131$ 就可以了。

因此我们的做法就是在 $\text{fail}$ 树上建主席树，存下来每一个 $f(s_{i,1}-s_{i,2})$，然后查询 $f(p_1-p_2)$ 的计数即可。

[AC Code](https://gist.github.com/hikariyo/d3ff774cb85e8e5913d3824f97cd9075)。

## Luogu P7456

看到这个题目，就立马有一个朴素的 DP 想法，设 $dp_i$ 表示前缀 $s[1,i]$ 凑出来的最小单词数，那么转移有：
$$
dp_i=\min_{j=i-|a_k|}^{i-1} dp_j+1
$$
这里需要满足 $s[1,i]$ 以 $a_k$ 为后缀，于是你就会发现，我们需要的只是最长的满足条件的 $a_k$。

因为你可以观察这个式子，更短的 $a_k$ 对答案的贡献一定会被最长的算进去。

而 ACAM 就可以求出来最长的那个长度。

这道题目剩下的部分就是一个线段树优化 DP 了。

[AC Code](https://gist.github.com/hikariyo/2f80aadab53ac41c08b880a4833b8d1e)。

## Luogu P4052

相当于，在 ACAM 上走 $m$ 步并且不走到 $\text{fail}$ 链上有标记的点的方案数。

设 $dp(u,k)$ 表示节点 $u$ 走 $k$ 步的方案数，那么由能走到的节点 $to$ 的 $dp(to,k-1)$ 转移过来即可。

[AC Code](https://gist.github.com/hikariyo/828f483b3ef7387b1776d330d97ba53c)。

## HDU 2457

设 $dp(i,u)$ 设添加第 $i$ 个字符后，在 ACAM 上匹配到以 $u$ 结尾的最小次数。

当我们添加一个字符时，需要对每个节点 $u$ 都尝试沿着 $\texttt{A,T,G,C}$ 向下走。

形式化地，$dp(i+1,to)\gets dp(i,u)+0/1$，这里 $0/1$ 表示走的边是否等于原始串对应位置的字符。

发现只会由 $i$ 向 $i+1$ 推，所以可以滚动数组优化掉这一维。

[AC Code](https://gist.github.com/hikariyo/c95dc4fd8c32a8b879abe7d3d696c718)。
