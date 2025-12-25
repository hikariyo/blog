---
date: 2025-12-25 15:40:00
title: 字符串 - SAM
katex: true
tags:
- Algo
- C++
- String
categories:
- XCPC
description: 后缀自动机的应用。
---

## SAM

这个自动机的原理出了名的难学懂，所以我放弃理解 `insert` 的原理了，只是学习了 SAM 提供的接口该怎么用，这里给出我整理的一个最精简的模板，具体的原理请移步各大博客。

```cpp
const int N = 2e6 + 10, Sigma = 26;

namespace SAM {
    int last, sz;
    int tot;
    int trans[N][Sigma];

    int len[N], fa[N];

    void init() {
        last = sz = 1;
        tot = 0;
    }

    void insert(int x) {
        int np = ++sz;
        int p = last, q, nq;
        last = np;

        // set attributes of np
        
        len[np] = len[p] + 1;
        while (p && !trans[p][x]) trans[p][x] = np, p = fa[p];
        if (!p) return fa[np] = 1, void();
        q = trans[p][x];
        if (len[q] == len[p] + 1) return fa[np] = q, void();
        nq = ++ sz, len[nq] = len[p] + 1;
        fa[nq] = fa[q], fa[q] = fa[np] = nq;
        memcpy(trans[nq], trans[q], sizeof(trans[q]));
        while (p && trans[p][x] == q) trans[p][x] = nq, p = fa[p];
    }
}
```

## [SDOI2016] 生成魔咒

这个题目要求的是每次操作结束后新增了多少个与前面不同的子串个数。

所以我们直接爬 $\text{last}$ 的祖先，并且一路打标记，直到被标记过为止。

这是因为如果遇到了第一个打标记的，也就是之前出现过，那么它的祖先是这个已经出现过的串的子串，所以就停止了。

本题的字符集比较大，可以用 `map` 来维护转移数组，那么复杂度是 $O(n\log n)$。

[AC Code](https://gist.github.com/hikariyo/32c9f3a35a06d65cd17fc9fcb9f985b2)。

## [HAOI2016] 找相同字符

我们对 $s_1$ 建一个 SAM，然后对 $s_2$ 的每一个位置 $i$，我们都求出一个最长的 $l$ 使得 $s_2[i-l+1, i]$ 是 $s_1$ 的子串，下面用 $s$ 代指 $s_2$。

求的方式是，假设 $i-1$ 对应的节点是 $u$，初始值为空状态 $1$，那么我们尝试走 $\text{trans}(u,s_i)$。

这个原因类比 KMP 的过程：以 $i$ 结尾的、最长的、出现在 SAM 中的子串，扣掉最后一个字符，一定是以 $i-1$ 位置结尾的、出现在 SAM 中的子串。

由于我们已经知道了 $i-1$ 结尾的、最长的、出现 SAM 中的子串，我们只需要不断尝试这么转移，如果失败了就跳 $\text{fa}$ 链，这样就一定能求出以 $i$ 结尾的、最长的、出现在 SAM 中的子串了。

这样的复杂度可以类比 KMP 的势能分析，每一次成功的转移势能至多加一，然后每一次跳 $\text{fa}$ 链势能至少减一，所以复杂度是 $O(n)$ 的。

求出这个子串代表的节点之后，我们需要累加 $\text{fa}$ 链上的 $\text{cnt}(u)\times [\text{len}(u)-\text{len}(\text{fa}(u))]$，当然对于当前匹配到的节点不一定是 $\text{len}(u)$。

[AC Code](https://gist.github.com/hikariyo/931bcdc9b677f7ac6bea4371edf4979d)。

## [AHOI2013] 差异

显然这个式子可以拆成两个部分 $\sum \text{len}(T_i)+\text{len}(T_j)$ 和 $-2\sum\text{lcp}(T_i,T_j)$，对于前一部分显然是有公式的，当然懒得推，写一个前缀和，$O(n)$ 去做也完全可以，关键在于后一部分。

我一开始的做法是，倒着建立 SAM，枚举 $T_i$ 的所有前缀（也就是 $\text{fa}$ 链），然后看有多少个节点能对答案产生贡献。由于我对我的假做法耿耿于怀，这里详细说一下。

如果要这么做的话，考虑每一个节点 $u$ 以及 $v=\text{fa}(u)$，显然 $u=\text{last}$ 时 $u$ 不可能作为答案，所以我们可以枚举 $\text{fa}$ 链上的 $u$ 时，累加 $v$ 的贡献：$\sum_{u,v=\text{fa}(u)} \text{len}(v)\times c(v)$，下面我们考虑 $c(v)$ 即 $v$ 作为答案的次数怎么计算。

我们知道 SAM 支持求出每一个节点对应的 $\text{endpos}$，即它的所有出现位置。这里的 $c(v)$ 要求 $v$ 是 $\text{lcp}(T_i,T_j)$，于是只有当 $v$ 第一次出现时，它才是 $\text{lcp}(T_i,T_j)$。

又有 SAM 的性质保证 $\text{endpos}(u)\subset \text{endpos}(v)$，那么 $v$ 第一次出现的次数就应当是 $c(v)=|\text{endpos}(v)-\text{endpos}(u)|=|\text{endpos}(v)|-|\text{endpos}(u)|$。

于是累加的就是 $\sum_u \text{len}(\text{fa}(u))|\text{endpos}(\text{fa}(u))|-\sum_u \text{len}(\text{fa}(u))|\text{endpos}(u)|$，我们分别维护这两部分。

这个怎么在线地处理？因为每加入一个新的前缀，就会在后缀链接树上给一个点到根的路径，每一个点加上对应的 $\text{len}(\text{fa}(u))$。

这个相当于每一个点有一个权 $k_i$，每次给数组 $a$ 区间 $+1$，并且求 $\sum k_ia_i$。

这是可以树剖线段树维护的，具体地说，在线段树上处理出每一个区间对应的 $\sum k_i$，然后区间加的时候可以根据这个 $\sum k_i$ 正确的处理懒标记和当前节点的 $\text{sum}$。

然而，遗憾的是，这么精彩（并不）的做法复杂度是 $O(n\log^2n)$，是无法通过本题的。

[TLE Code](https://gist.github.com/hikariyo/ed728e56ec66d5a567047df43b86b910)。

正解应当是建完整个 SAM 后，由于任意两个点 $(i,j)$ 的公共前缀对应的节点应当是它们在后缀链接树上的公共祖先，所以 $\text{lcp}$ 对应的自然就是 $\text{lca}$，所以我们枚举每一个节点能作为多少个 $(i,j)$ 的 $\text{lca}$ 即可，这样的复杂度是 $O(n)$ 的，高下立判。

[AC Code](https://gist.github.com/hikariyo/24fa635f21ad1ba479f645cdf467ea56)。

## [TJOI2015] 弦论

当相同子串算作一个时，SAM 上每一个点代表的数量就是 $\text{len}(u)-\text{len}(\text{fa}(u))$；算多个时，就再乘上出现次数，记这个数量为 $f(u)$。

然后，我们考虑 SAM 上转移的 DAG。

考虑每一个节点，如果 $k\le f(u)$，那么就找到了，否则 $k\gets k-f(u)$。

然后，我们枚举接下来的字符 $c=\texttt{a}\sim \texttt{z}$，如果走这条边无论如何也无法凑出第 $k$ 小，意味着 $k>\sum_{v\text{ after } \text{trans}(u,c)} f(v)$，这个值是可以预处理出来的，令它为 $g(v)$，那么就 $k\gets k-g(v)$；如果走这条边能凑出来自然就有 $k\le g(v)$，那么递归处理下去即可，复杂度 $O(n)$。

[AC Code](https://gist.github.com/hikariyo/8a5f805c6019ef57a3a94f1ef79bdfd5)。

## [TJOI2019] 甲苯先生和大中锋的字符串

我们遍历 SAM 上每一个 $|\text{endpos}(u)|=k$ 的节点，那么这个节点代表的字符串就出现了 $k$ 次。

这个节点的长度是 $\text{len}(\text{fa}(u))+1\sim \text{len}(u)$，我们用一个差分数组区间 $+1$ 即可，最后统计一下哪种长度出现的次数最多，复杂度 $O(n)$。

[AC Code](https://gist.github.com/hikariyo/a6c72161beca9a29918ac6eeee01e3e8)。

## [BJOI2020] 封印

由于 $t$ 没有限制，所以我们对 $t$ 建 SAM，然后匹配 $s$。

对每一个点 $s_i$，我们都能找到最长的、以 $i$ 结尾的、出现在 $t$ 中的子串，记 $L_i$ 为这样的长度，那么 $j=i-L_i+1\sim i$ 都满足 $s[j,i]$ 是公共子串。

对于每一个区间 $l\sim r$，$i$ 对答案的贡献有两种可能性：

1. $i-L_i+1\le l$，那么答案就是 $i-l+1$；
2. $i-L_i+1\ge l$，那么答案就是 $L_i$。

我们可以把 $i-L_i+1$ 当作下标，建一个可持久化权值线段树，分别维护 $i$ 和 $L_i$。

对于 $l\sim r$ 的查询，查 $r$ 版本的线段树，对 $\le l$ 我们取 $(\max i)-l+1$，对 $\ge l$ 我们取 $\max L_i$。

无需担心 $<l$ 的点，因为这些点是第一种情况，并且 $i-l+1\le 0$，不影响答案。

这样复杂度是 $O(n\log n)$。

[AC Code](https://gist.github.com/hikariyo/7bdec29f4daebcea395d00155597dba4)。

## [CTSC2012] 熟悉的文章

设 $f_n$ 表示 $A[1,n]$ 满足所有公共子串长度 $\ge L$ 的公共子串长度之和的最大值。

根据定义，如果 $L$ 可行，那么 $\le L$ 的值都可行，所以这是有二分性的，所以我们就检查是否有 $f_n\ge \cfrac{9}{10} n$ 即可。

设 $l_n$ 表示 $A_n$ 结尾的、最长的、出现在模式串中的子串长度。

对于这种没有什么钦定的我不会思考，所以我令 $g_n$ 表示钦定最后一个串以 $A_n$ 结尾，就有 $f_n$ 为 $g_n$ 的前缀最大值，于是得到转移方程：
$$
g_n=\max_{L\le l\le l_n}\{\max_{j\le n-l+1}\{g_j\}+l\}=\max_{n-l_n\le j\le n-L}\{f_j-j+i\}
$$
又由 $f_n=\max\{f_{n-1},g_n\}$，所以我们不需要真的建出 $g$ 这个数组，只需要求 $g$ 的每一项用来更新 $f$ 即可。

对于每一次二分检查，$L$ 是固定的，那么我们就关心 $n-l_n$ 的性质。

观察 $l_n$ 的建立过程，它是尝试从 $l_{n-1}+1$ 继承过来的，所以天然地就有 $l_n\le l_{n-1}+1$，于是就有 $n-l_n\ge n-1-l_{n-1}$，即 $n-l_n$ 非严格单调递增。

所以这是一个只会向前的滑动窗口，可以单调队列维护，这样复杂度就是 $O(n\log n)$。

[AC Code](https://gist.github.com/hikariyo/0c0b6e5c09b5b230122d0ccba0f0423d)。

## [HEOI2016/TJOI2016] 字符串

对于这种有前缀的问题，我们首先反向建立 SAM。

然而直接跳父亲是没什么前途的，因为对于每一个节点代表的一类字符串，它们要求在 $s[a,b]$ 中的开头出现位置的范围都不一样。

然后，就是这个题目最重要的部分了，需要观察出答案具有二分性。

二分答案 $m$，然后我们可以倍增找到 $s_c$ 开头、长度为 $m$ 的前缀在的那个节点。

现在我们检查这个节点代表的字符串的开头是否在 $s[a,b-m+1]$ 中出现了。

于是我们需要得到节点的所有 $\text{endpos}$，这是可以线段树合并处理出来的。

因此查询这个节点对应的线段树在 $[a,b-m+1]$ 中是否有 $\text{endpos}$ 即可。

这样的复杂度是 $O(n\log n+m\log^2n)$，其中 $O(n\log n)$ 是线段树合并的复杂度。

[AC Code](https://gist.github.com/hikariyo/e1acba5ebee5bd3db43dc25a7439273c)。

## [USACO17DEC] Standing Out from the Herd P

把这些所有的串都插入 SAM 中，中间加一个特殊字符间隔开。

然后，我们枚举整个串的本质不同的子串，看这个子串能够被谁贡献得到。

也就是说，我们维护 $\text{endpos}$ 的同时也维护 $\text{endpos}$ 中这些位置属于哪一个字符串。

然而我们并不需要获取到这些所有字符串的信息，我们只关心是不是恰好只有一个，所以用一个 `int` 来维护即可。

然后，对于这所有的 $\text{endpos}$ 中，我们任取一个即可，这里代码实现方便取了最右边的。

然后我们结合这个出现位置，以及字符串的长度，取到正确的 $\text{len}$。

这是因为这个节点代表的某些字符串有可能包含了那个用来分隔的特殊字符，需要把它去掉。

这样的复杂度是 $O(n)$。

[AC Code](https://gist.github.com/hikariyo/ccca889046f26f8a27ab2de837d2e074)。

## [CF666E] Forensic Examination

我们先把 $s$ 和 $t_1\sim t_n$ 都插入 SAM，中间用特殊字符隔开。

每次回答先倍增跳到 $s[pl,pr]$ 对应节点，然后再看这个节点被 $t_l\sim t_r$ 贡献了多少次，这一步可以线段树合并来维护。

这样单次询问的复杂度是 $O(\log |s|+\log m)$。

[AC Code](https://gist.github.com/hikariyo/f2c064bf0d045cf818dcfa68c2aa38b6)。