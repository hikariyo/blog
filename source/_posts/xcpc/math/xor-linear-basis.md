---
date: 2026-05-05 21:08:00
updated: 2026-05-06 10:17:00
title: 数学 - 异或线性基
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 从向量空间视角整理异或线性基的常用定义、性质和证明。
---

## 符号约定

下文固定正整数 $B$，一般取 $60$，同时定义 $I_B=\{0,1,\dots,2^B-1\}$。

对于 $x\in I_B$，定义取位函数 $\operatorname{bit}_i(x)=\lfloor x/2^i\rfloor\bmod 2$。

对于 $x\in I_B,x\neq 0$，定义最高位 $h(x)=\max\{i\mid \operatorname{bit}_i(x)=1\}$。

对于 $a_1,a_2,\dots,a_n\in I_B$，定义张成空间 $\operatorname{span}\{a_1,a_2,\dots,a_n\}=\left\{\bigoplus_{i\in S} a_i\mid S\subseteq \{1,2,\dots,n\}\right\}$。

对于 $b_1,b_2,\dots,b_m\in I_B$，称它们线性无关，当且仅当对任意 $S\subseteq\{1,2,\dots,m\}$，若 $\bigoplus_{i\in S} b_i=0$，则 $S=\varnothing$，这里约定空集的异或和为 $0$。

若 $b_1,b_2,\dots,b_m\in I_B$，若它们线性无关，且张成空间 $L$，则称它们是 $L$ 的一组基。

若 $x\in I_B$，$L$ 是一个张成的空间，则称 $x\oplus L=\{x\oplus \ell\mid \ell\in L\}$ 为 $x$ 关于 $L$ 的陪集。

## 贪心线性基

贪心线性基用 $b_0,b_1,\dots,b_{B-1}$ 表示，若 $b_i=0$ 表示 $i$ 位置没有向量，若 $b_i\neq 0$ 要求 $h(b_i)=i$，并称 $i$ 为一个主元位。

### 构建

插入算法如下：

```cpp
#define B 60
using ull = unsigned long long;
ull b[B];

bool insert(ull x) {
    for (int i = B-1; i >= 0; i--) {
        if (x >> i & 1) {
            if (!b[i]) {
                b[i] = x;
                return true;
            }
            x ^= b[i];
        }
    }
    return false;
}
```

下面证明插入 $a_1,a_2,\dots,a_n$ 后，令 $L_n=\operatorname{span}\{a_1,a_2,\dots,a_n\}$，那么 $\{b_i\mid b_i\neq 0\}$ 是 $L_n$ 的一组基：先证 $\{b_i\mid b_i\neq 0\}$ 线性无关，然后再证 $L_n^B=\operatorname{span}\{ b_i\mid b_i\neq 0\}$ 与 $L_n$ 相等。

**证明**：

1. 首先证明 $\{b_i\mid b_i\neq 0\}$ 线性无关，令 $T=\{i\mid b_i\neq 0\}$

   设 $S\subseteq T$ 且 $S\neq \varnothing$，设 $j=\max S$。

   那么对任意 $i\in S$ 且 $i\neq j$，那么 $h(b_i)=i<j$。

   于是 $\operatorname{bit}_j(b_i)=0$，那么 $\operatorname{bit}_j\left(\bigoplus_{i\in S} b_i\right)=\bigoplus_{i\in S}\operatorname{bit}_j(b_i)=1$，所以 $\bigoplus_{i\in S} b_i\neq 0$。

   所以原命题成立。

2. 其次证明 $L_{n}^B$ 与 $L_n$ 相等。用数学归纳法证明：对任意 $k$，记 $L_k=\operatorname{span}\{a_1,a_2,\dots,a_k\}$ 且 $L_k^B$ 是插入完前 $k$ 个数后的 $\operatorname{span}\{b_i\mid b_i\neq 0\}$，首先当 $k=0$ 时显然成立。

     假设插入完前 $k-1$ 个数后成立，即 $L_{k-1}=L_{k-1}^B$。

     插入 $a_k$ 时，当前 $x$ 初始为 $a_k$。由于每个已有的 $b_i$ 都属于 $L_{k-1}$，每次执行 $x\gets x\oplus b_i$ 后，当前 $x$ 仍属于陪集 $a_k\oplus L_{k-1}$。

     + 若插入失败，则最终 $x=0$，所以 $0\in a_k\oplus L_{k-1}$，从而 $a_k\in L_{k-1}$。

         此时 $L_k^B=L_{k-1}^B$。根据定义，有 $L_{k-1}\subseteq L_k$；根据 $a_k\in L_{k-1}$，有 $L_{k}\subseteq L_{k-1}$，因此 $L_k=L_{k-1}$。

         由归纳假设得到 $L_k^B=L_k$。

     + 若在第 $i$ 位插入当前 $x$，则更高位已经被消去，且当前第 $i$ 位为 $1$，所以 $h(x)=i$。

         又因为 $x\in a_k\oplus L_{k-1}$，存在 $\ell\in L_{k-1}$ 使得 $x=a_k\oplus \ell$。

         这里 $x=a_k\oplus \ell$，$a_k\in L_k$ 且 $\ell\in L_{k-1}\subseteq L_k$，所以 $x\in L_k$；

         同时 $a_k=x\oplus \ell$，因为 $x\in L_k^B$ 且 $\ell \in L_{k-1}^B\subseteq L_k^B$，所以 $a_k\in L_k^B$。

         因为 $L_{k-1}^B=L_{k-1}\subseteq L_k$，且 $x\in L_k$，所以 $L_k^B\subseteq L_k$；

         同时 $L_{k-1}=L_{k-1}^B\subseteq L_k^B$，且 $a_k\in L_k^B$，所以 $L_{k}\subseteq L_{k}^B$。
         
         因此 $L_k^B=L_k$。

3. 结合线性无关性，$\{b_i\mid b_i\neq 0\}$ 是 $L_n$ 的一组基。

### 规约

对于由线性基 $\{b_i\mid b_i\neq 0\}$ 张成的空间 $L$，定义规约函数 $R:I_B\to I_B$：从高到低枚举位 $i$，若 $\operatorname{bit}_i(x)=1$ 且 $b_i\neq 0$，那么令 $x\gets x\oplus b_i$，最终值为 $R(x)$，代码如下：

```cpp
ull reduce_min(ull x) {
    for (int i = B-1; i >= 0; i--) {
        if (x >> i & 1) x ^= b[i]; // 实际上可以不判 b[i] != 0, 不影响答案
    }
    return x;
}
```

下证 $R(x)$ 是陪集 $x\oplus L$ 中的最小元素：

**证明**：

1. 设 $R(x)=x\oplus \ell_1$，其中 $\ell_1\in L$，那么任取 $y\in x\oplus L$，存在一个 $\ell_2\in L$ 使得 $y=x\oplus \ell_2$。

   代换 $x$，得到 $y=R(x)\oplus \ell_1\oplus \ell_2$。

   根据空间的性质，有 $\ell=\ell_1\oplus \ell_2\in L$，即 $y=R(x)\oplus \ell$。

2. 令 $P=\{i\mid b_i\neq 0\}$，设 $\ell=\bigoplus_{i\in S} b_i$，其中 $S\subseteq P$。

   根据规约函数的定义，$\forall i\in P,\operatorname{bit}_i(R(x))=0$。

   若 $S=\varnothing$，那么 $y=R(x)$；

   若 $S\neq \varnothing$，取 $j=\max S$，那么 $\operatorname{bit}_j(R(x)\oplus \ell)=1$。

   同时，对于 $k>j$，因为 $\forall i\in S,i\le j<k$，有 $\operatorname{bit}_k(b_i)=0$，那么 $\operatorname{bit}_k(R(x)\oplus \ell)=\operatorname{bit}_k(R(x))$。

   综上所述，就得到了 $y>R(x)$。

3. 因此，$\forall y\in x\oplus L,y\ge R(x)$。

同理，如果规约函数 $R':I_B\to I_B$ 的映射规则改为：从高到低枚举位 $i$，若 $\operatorname{bit}_i(x)=0$ 且 $b_i\neq 0$，那么令 $x\gets x\oplus b_i$，最终值为 $R'(x)$，那么 $R'(x)$ 是陪集 $x\oplus L$ 中的最大元素。

---

上面的证明实际上说明了：若某个元素属于该陪集，且所有主元位为 $0$，则它就是该陪集最小值。那么我们可以根据这个得到下面的定理：$\forall x,y\in I_B,R(x\oplus y)=R(x)\oplus R(y)$。

**证明**：

1. $R(x\oplus y)$ 是 $(x\oplus y)\oplus L$ 中的最小元素。

   又因为存在 $\ell_x,\ell_y\in L$，使得 $R(x)=x\oplus \ell_x,R(y)=y\oplus \ell_y$。

   那么 $R(x)\oplus R(y)=x\oplus y\oplus (\ell_x\oplus \ell_y)\in (x\oplus y)\oplus L$。

   所以 $R(x\oplus y)\le R(x)\oplus R(y)$。

2. 又因为对任意主元位 $i$，都有 $\operatorname{bit}_i(R(x))=0$ 且 $\operatorname{bit}_i(R(y))=0$，那么 $\operatorname{bit}_i(R(x)\oplus R(y))=0$。

   根据前文证明最小值的方法，就能得到 $\forall z\in (x\oplus y)\oplus L$，有 $R(x)\oplus R(y)\le z$。

   所以 $R(x)\oplus R(y)\le R(x\oplus y)$。
3. 因此 $R(x)\oplus R(y)=R(x\oplus y)$。

## 标准化

当需要比较两个空间是否相同时，我们需要对贪心得到的线性基进行标准化。

对应到线性代数的概念，贪心线性基实际上得到的是一个行阶梯矩阵，这里所谓标准化就是把这个行阶梯矩阵进一步化为行最简矩阵。

具体地讲，对 $i=0,1,\dots,B-1$，若 $b_i\neq 0$，则从大到小枚举 $j$，其中 $j<i$，若 $\operatorname{bit}_j(b_i)=1$ 且 $b_j\neq 0$，令 $b_i\gets b_i\oplus b_j$。

标准化后的贪心线性基张成空间和原空间相等：任意一次 $b_i\gets b_i\oplus b_j$ 后，因为这两组基可以互相表示，那么张成空间相等。

注意，如果只是说“标准化基”，并不要求它是从贪心线性基得到，只要求下面的事情：

+ 设 $P$ 是主元位集合，那么 $\forall i,j \in P$，有 $\operatorname{bit}_j(b_i)=[i=j]$ 且 $h(b_i)=i$，其中 $[-]$ 是艾佛森括号。

还可以证明，两个空间相等当且仅当标准化基相等，形式化地，若 $b_0,b_1,\dots,b_{B-1}$ 和 $c_0,c_1,\dots,c_{B-1}$ 都是标准化基，令

$$
P_b=\{i\mid b_i\neq 0\},\qquad P_c=\{i\mid c_i\neq 0\}
$$

$$
L_b=\operatorname{span} \{b_i\mid i\in P_b\},\qquad L_c=\operatorname{span}\{c_i\mid i\in P_c\}
$$

那么
$$
L_b=L_c\Leftrightarrow \forall i\in\{0,1,\dots,B-1\}, b_i=c_i
$$
**证明**：

充分性显然，只证必要性。

我们希望证明 $P_b=P_c$，这里可以借助取最高位函数 $h(x)$ 来做到这一点。

具体地讲，令 $H_b=\{h(x)\mid x\in L_b, x\neq 0\}$，我们可以证明 $H_b=P_b$：

任取 $h(x)\in H_b$，那么存在非空集合 $S\subseteq P_b$ 使得 $x=\bigoplus_{i\in S} b_i$，那么 $h(x)=\max S\in P_b$；

反之，任取 $i\in P_b$，都有 $b_i\in L_b$ 使得 $h(b_i)=i$，即 $i\in H_b$。

所以 $H_b=P_b$，同理 $H_c=P_c$，又因为 $L_b=L_c$，所以 $H_b=H_c$，就得到了 $P_b=P_c$。

现在任取 $p\in P_b$，我们希望证明 $b_p=c_p$。考虑这是一个标准化基，那么
$$
\forall q\in P_b,\operatorname{bit}_q(b_p)=\operatorname{bit}_q(c_p)=\begin{cases}
1,\quad p=q\\
0,\quad p\neq q
\end{cases}
$$
令 $z=b_p\oplus c_p$，根据 $L_b=L_c$ 和张成空间的封闭性，我们有 $z\in L_b$。

若 $z\neq 0$，那么 $h(z)\in H_b=P_b$，又因为 $\forall q\in P_b,\operatorname{bit}_q(z)=0$，矛盾，于是 $z=0$。

即对于所有的 $p\in P_b$，都有 $b_p=c_p$；对不在 $P_b$ 中的下标，由定义有 $b_p=c_p=0$。

