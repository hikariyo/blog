---
date: 2023-06-15 15:10:05
title: 数据结构 - Hash
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
---

## 简介

散列函数将大范围的数字映射为小范围的数字，但不保证单调性，即多个值可能映射为同一个值，这种情况叫做**哈希碰撞**。对于整数 $M$ 来说，一般求 hash 值直接取模 $M\bmod N$ 作为映射到 $[0, N-1]$ 的 hash 值，这里的 $N$ 最好是一个质数才保证碰撞发生概率小，证明如下（$M, N$ 均为正数且 $m,n$ 为它们的约数）：
$$
\begin{aligned}
M&=qN+r,m=qn+\frac{r}{k}\\
\Rightarrow m\bmod n&=\frac{r}{k}\in\{0,1,2,\cdots,m-1\}\\
\Rightarrow r&\in\{0,k,2k,\cdots,k(m-1)\}
\end{aligned}
$$
因此，要使得 $M\bmod N$ 能尽可能取到多的数字，让 $N$ 是质数是最好的选择，这样可以保证最小公因数为 $1$。

## 拉链法

> 维护一个集合，支持如下几种操作：
>
> 1. `I x`，插入一个数 $x$；
> 2. `Q x`，询问数 $x$ 是否在集合中出现过；
>
> 现在要进行 $N \in [1, 10^5]$ 次操作，对于每个询问操作输出对应的结果。

有两种方法实现散列表，开放地址法和拉链法，先看下拉链法的实现：

```cpp
#include <bits/stdc++.h>

const int N = 1e5+3;
int h[N], e[N], ne[N], idx;

int hash(int x) {
    return (x % N + N) % N;
}

void insert(int x) {
    int k = hash(x);
    e[idx] = x; ne[idx]=h[k];
    h[k] = idx++;
}

bool find(int x) {
    for (int i = h[hash(x)]; i != -1; i = ne[i]) if (e[i] == x) return true;
    return false;
}

int main() {
    memset(h, -1, sizeof h);
    char op[2];
    int n, x;
    scanf("%d", &n);
    while (n--) {
        scanf("%s%d", op, &x);
        if (*op == 'I') insert(x);
        else puts(find(x) ? "Yes" : "No");
    }
    return 0;
}
```

由于输入的数可能为负数，所以采取 `(x%N+N)%N` 的方法获取到正的余数。

对于插入的步骤，用一张图解释：

这是一个头插法，好处是时间复杂度为 $O(1)$，如果尾插的话要遍历一遍就不是常数复杂度了。

## 开放地址法

用单个数组存储，冲突就下移一位，这里的 `0x3f3f3f3f` 看情况可以调整成更大或更小的数。这里数组长度的经验值取到 $2$ ~ $3$ 倍能降低碰撞率。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 2e5+3, null = 0x3f3f3f3f;
int h[N];

int find(int x) {
    int k = (x % N + N) % N;
    while (h[k] != null && h[k] != x) {
        ++k;
        if (k == N) k = 0;
    }
    return k;
}

int main() {
    int n, x; char op[2];
    memset(h, 0x3f, sizeof h);
    scanf("%d", &n);
    while (n--) {
        scanf("%s%d", op, &x);
        int k = find(x);
        if (*op == 'I') h[k] = x;
        else puts(h[k] == x ? "Yes" : "No");
    }
    return 0;
}
```

`memset` 函数按照字节初始化，所以要让 `null` 的每个字节都相等。实在想不起来可以 `for` 循环赋值。

这里的 `find` 寻找的是 `x` 所在位置或者是可插入的空闲位置，与拉链法里的 `find` 不同。

## 字符串哈希

> 给定一个长度为 $n \in [1, 10^5]$ 的字符串，再给定 $m \in [1, 10^5]$ 个询问，每个询问包含四个整数 $l1,r1,l2,r2$，请你判断 $[l1,r1]$ 和 $[l2,r2]$ 这两个区间所包含的字符串子串是否完全相同。字符串中只包含大小写英文字母和数字。

可快速判断字符串是否相等，原理是将字符串看作一个 $P$ 进制的数字，然后对 $Q$ 取模求哈希值。一般来说，$P=131, 13331, Q=2^{64}$，这样取的原因是 `131` 是 `128` 后第一个质数，`13331` 也是质数；$Q$ 取这个数的原因是 `unsigned long long` 自然溢出就等同于对 $2^{64}$ 取模。

在计算时，仿照前缀和的形式，将字符串从第  $1$ 位到第 $i$ 位的子区间求哈希，方法是左边为高位，右边为低位，每次将上一次求的结果乘 $P$ 加上当前位置的数值就是所要得到的结果。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef unsigned long long ull;
const int P = 131, N = 1e5+10;
// p[i] 表示 p 的 i 次方
ull p[N], h[N];
char s[N];

ull hashrange(int l, int r) {
    return h[r] - h[l-1] * p[r-l+1];
}

int main() {
    int n, m, l1, r1, l2, r2;
    scanf("%d%d%s", &n, &m, s+1);

    p[0] = 1;
    for (int i = 1; i <= n; i++) {
        p[i] = p[i-1] * P;
        h[i] = h[i-1] * P + s[i];
    }

    while (m--) {
        scanf("%d%d%d%d", &l1, &r1, &l2, &r2);
        puts(hashrange(l1, r1) == hashrange(l2, r2) ? "Yes" : "No");
    }

    return 0;
}
```

`hashrange` 函数的原理是，将 $[1, l-1]$ 与 $[1, r]$ 区间的值对齐后，让 $[1, r]$ 把这个前缀减掉，就是我们要求得的区间内哈希值。由于左边是高位，右边是低位，所以 $[1, l-1]$ 这个区间比另一个区间要少的位数，就是 $[l, r]$ 这个区间的长度，即 $r-l+1$，因此计算式是 `h[r] - h[l-1] * p[r-l+1]`

注意，这里一定不要用 `pow` 函数求 `p` 的几次方，会溢出（特殊处理后应该是可以用的，但是这里就不多介绍了）。

理论上来说，这是有可能发生哈希碰撞的，但概率极低，一般可忽略不计。