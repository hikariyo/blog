---
date: 2025-12-01 17:59:00
title: 字符串 - KMP
katex: true
tags:
- Algo
- C++
- String
categories:
- XCPC
description: KMP 与 Border 理论。
---

## Border

> 设 $S$ 为字符串，用 $S[l,r]$ 表示 $S_lS_{l+1}\cdots S_r$ 组成的子串，用 $|S|$ 表示它的长度。
>
> 如果对整数 $p$ 满足 $S[1,p]=S[|S|-p+1,|S|]$ 那么称 $p$ 为 $S$ 的一个 $\text{Border}$。

特殊地，$p=0$ 和 $p=|S|$ 都是 $S$ 的 $\text{Border}$。

例如，$S=\texttt{abbacabb}$，那么 $p=8,3,0$ 是它的所有 $\text{Border}$。

> 如果整数 $q$ 对所有 $1\le i,i+q\le |S|$ 有 $S_i=S_{i+q}$ 恒成立，则称 $q$ 是 $S$ 的一个周期。

注意这里周期并不一定整除 $|S|$，例如 $S=\texttt{abcabcab}$，那么 $q=3$ 是一个周期。

如果我们把 $\text{Border}$ 的定义改成这种形式，会是：

> 如果整数 $p$ 对所有 $1\le i,i+|S|-p\le |S|$ 有 $S_i=S_{i+|S|-p}$ 恒成立，则称 $p$ 是 $S$ 的一个 $\text{Border}$。

因此每个 $p$ 对应一个 $|S|-p$ 是字符串的周期。

KMP 算法就是求出每一个前缀 $S[1,i]$ 的最大的小于 $i$ 的 $\text{Border}$。

注意，这里的 $\text{Border}$ 既可以指长度，也可以指对应的那个字符串，根据语境确定是哪个意思。

## KMP

我们可以发现，对于前缀 $S[1,i]$ 的所有 $\text{Border}$，扣掉最后一个字母后，一定是 $S[1,i-1]$ 的 $\text{Border}$。

所以我们可以从大到小遍历 $S[1,i-1]$ 的所有 $\text{Border}$，验证加一个字母后是不是 $S[1,i]$ 的 $\text{Border}$，如果是的话就求出来了。

定义 $\text{next}(i)$ 表示 $S[1,i]$ 的最大的小于 $i$ 的 $\text{Border}$，有这样一个结论：对于 $S[1,i]$ 的所有长度小于 $i$ 的 $\text{Border}$，都可以通过跳 $\text{next}(\text{next}(\cdots\text{next}(i)))$ 获得到。

证明：如果 $p$ 是 $S[1,i]$ 的 $\text{Border}$，$q(q>p)$ 也是 $S[1,i]$ 的 $\text{Border}$，那么 $p$ 是 $q$ 的 $\text{Border}$。因此将 $S[1,i]$ 的所有 $\text{Border}$ 排序，不妨设为 $i>p_1>p_2>\dots>0$ ，那么显然有 $p_k$ 是 $p_{k-1}$ 的最大 $\text{Border}$，因此可以通过跳 $\text{next}$ 获得到所有 $\text{Border}$。

因此我们的代码写出来应当像是这个样子：

```python
n = len(s)
nxt[1] = 0
for i = 2 to n:
    nxt[i] = nxt[i-1]
    while nxt[i] != 0 and s[i] != s[nxt[i] + 1]:
        nxt[i] = nxt[nxt[i]]
    if s[i] == s[nxt[i] + 1]:
        nxt[i]++
```

这样复杂度看起来像是 $O(n^2)$ 的？其实不然，需要势能分析，我在这里尽量简单的说。

我们设势能为 $\Phi=\text{next}(i-1)$，那么每一次 `while` 循环的执行次数不会超过 $\Phi$。

这是显然的，因为跳 $\text{next}(i)$ 是一个严格递减的过程。

我们会发现，`while` 循环每执行一次，就会使得势能至少减 $1$，而外层每一轮 `for` 循环至多只会给势能增加 $1$。

因此 `while` 循环的总执行次数不会超过 $n$，所以这个算法是 $O(n)$ 的。

## [模板] KMP

这是一个 KMP 算法的经典应用：字符串匹配。

我们的目标是在模板串 $S$ 中找到模式串 $T$ 的出现次数。

假设模板串当前匹配到 $i$，对应的模式串上一次匹配到 $j$（可能为 $0$），那么我们尝试把 $j$ 向下移动一个位置。

1. 如果成功，那么无事发生。

2. 如果失败，我们需要向右平移模式串 $T$，对应的 $j$ 应当减少。

   平移到什么程度为止呢？应当是第一次前缀恰好等于后缀的时候，其实就是 $\text{Border}$。

   如果失败就接着跳，一直到 $j=0$ 或者匹配上为止。

3. 特别地，如果把 $j$ 移动到了 $T$ 的末尾，我们需要尝试进行下一次匹配，令 $j\gets \text{next}(j)$ 即可。

复杂度同样是势能分析法，设势能为 $\text{next}(j)$，`while` 循环每执行一次至少会把势能减少 $1$，并且匹配成功后 $\Phi\gets \text{next}(j+1)$ 至多会把势能增加 $1$，因此执行次数不超过 $|S|$。

因此复杂度为 $O(|S|+|T|)$。

[AC Code](https://gist.github.com/hikariyo/d83573485b31f18606397ef057041102)。

## [USACO03FALL] Milking Grid（数据加强版）

显然，我们需要对所有横向的字符串找到它们公共的周期，纵向也要找到公共周期。

由于周期唯一对应一个 $\text{Border}$，我们又知道 $\text{Border}$ 的数量是线性的，所以开一个桶计它们的出现次数即可取交集。

因此复杂度为 $O(RC)$。

[AC Code](https://gist.github.com/hikariyo/08f283fdc45479ef31b2c939c96b74fc)。
