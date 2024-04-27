---
date: 2023-12-13 20:41:00
title: 数学 - 快速沃尔什变换
katex: true
description: 通过分治算法，将多项式在 O(nlog(n)) 的复杂度内进行变换，然后 O(n) 计算，最后进行一次 O(nlog(n)) 的逆变换进行求出位运算卷积。
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 位运算卷积

回顾 FFT，它要求的其实是：
$$
C_k=\sum_{i+j=k}A_iB_j
$$
将加号换成位运算符就是 FWT 了。

## 或卷积

首先看较为简单的或，我们这里把二进制数看作集合：
$$
C_k=\sum_{i\cup j=k}A_iB_j
$$
将多项式看作向量，我们需要找到一个线性映射 $\text{FWT}(X)$ 满足：
$$
\text{FWT}(C)_k=\text{FWT}(A)_k\text{FWT}(B)_k
$$
并且这个线性映射需要做到可逆，这里直接给出式子了：
$$
\text{FWT}(A)_k=\sum_{i\subset k}A_i
$$
因此有：
$$
\begin{aligned}
\text{FWT}(A)_k\text{FWT}(B)_k&=\sum_{i\subset k}A_i\sum_{j\subset k}B_j\\
&=\sum_{(i\cup j)\subset k}A_iB_j\\
&=\sum_{x\subset k}\sum_{(i\cup j)=x} A_iB_j\\
&=\text{FWT}(C)_k
\end{aligned}
$$
我们考虑如何进行 $\text{FWT}$ 变换，假设多项式共有有 $2^n$ 项，前半部分用 $A_0$ 表示，后边部分用 $A_1$ 表示，考虑分治：
$$
\text{FWT}(A)=\text{merge}(\text{FWT}(A_0),\text{FWT}(A_0)+\text{FWT}(A_1))
$$
由于后半部分每个对应位置都相当于前半部分对应位置的二进制位最前面添上一个 $1$，所以后半部分就可以直接向量相加，补充上缺失的部分。$\text{merge}$ 就是把两个向量首尾相接拼上。

因为有这个式子，所以我们进行逆变换的时候，直接这样：
$$
\text{IFWT}(A)=\text{merge}(\text{IFWT}(A_0),\text{IFWT}(A_1)-\text{IFWT}(A_0))
$$
这个是因为 $\text{FWT}$ 是线性变换，于是 $\text{FWT}(A_0)+\text{FWT}(A_1)=\text{FWT}(A_0+A_1)$，所以在逆变换的时候可以直接减掉。

## 和卷积

类似地，定义一下卷积：
$$
C_k=\sum_{i\cap j=k}A_iB_j
$$
然后定义一下 $\text{FWT}$ 这个变换：
$$
\text{FWT}(A)_k=\sum_{i\supset k}A_i
$$
因此有：
$$
\begin{aligned}
\text{FWT}(A)_k\text{FWT}(B)_k
&=\sum_{i\supset k} A_i\sum_{j\supset k}B_j\\
&=\sum_{(i\cap j)\supset k}A_iB_j\\
&=\sum_{x\supset k}\sum_{i\cap j=x}A_iB_j\\
&=\text{FWT}(C)_k
\end{aligned}
$$
然后看如何分治去求，下标 $k$ 如果包含在前半部分的某个下 $i$ 里面，它也一定包含在后半部分的里边；反之就不成立了，所以可以这样写：
$$
\begin{aligned}
\text{FWT}(A)&=\text{merge}(\text{FWT}(A_0)+\text{FWT}(A_1),\text{FWT}(A_1))\\
\text{IFWT}(A)&=\text{merge}(\text{IFWT}(A_0)-\text{IFWT}(A_1),\text{IFWT}(A_1))
\end{aligned}
$$

## 异或卷积

处理起来不太一样，主要用到了 $\text{popcount}$ 的性质。
$$
\text{popcount}(i\& k)+ \text{popcount}(j\& k)\equiv \text{popcount}((i\oplus j)\&k)\pmod 2
$$
分每一位去看：

+ 若 $k$ 的这一位为 $0$，那么最终是不产生贡献的。
+ 若 $k$ 的这一位为 $1$，并且 $i,j$ 这一位相同，左边的贡献为 $2$，右边的贡献为 $0$，是同余的。
+ 若 $k$ 的这一位为 $1$，并且 $i,j$ 这一位不同，左边的贡献为 $1$，右边的贡献为 $1$，是同余的。

看到同余于 $2$，想到一定会有：
$$
(-1)^{\text{LHS}}=(-1)^{\text{RHS}}
$$
所以定义线性变换：
$$
\text{FWT}(A)_k=\sum_{i}(-1)^{\text{popcount}(i\&k)} A_i
$$
因此有：
$$
\begin{aligned}
\text{FWT}(A)_k\text{FWT}(B)_k&=\sum_{i,j}(-1)^{\text{popcount}(i\&k)}A_i(-1)^{\text{popcount}(j\& k)}B_j\\
&=\sum_{i,j}(-1)^{\text{popcount}((i\oplus j)\&k)}A_iB_j\\
&=\text{FWT}(C)_k
\end{aligned}\\
$$
这和前面两个不同的地方在于每一项都是对整个向量所有值进行加权求和，所以最后根据加法交换律就不用在乎这一项到底是啥，只要前面的系数对了就行。

同样可以进行分治去求，由于前半部分的 $k$ 的最高位为 $0$，所以 $i$ 是没有贡献到的，后半部分贡献到了：
$$
\text{FWT}(A)=\text{merge}(\text{FWT}(A_0)+\text{FWT}(A_1),\text{FWT}(A_0)-\text{FWT}(A_1))\\
\text{IFWT}(A)=\text{merge}(\frac{\text{IFWT}(A_0)+\text{IFWT}(A_1)}{2},\frac{\text{IFWT}(A_0)-\text{IFWT}(A_1)}{2})
$$
接下来就是代码实现了。

## 代码

来自 [P4717](https://www.luogu.com.cn/problem/P4717) 的模板题，如果写个自动取模的 `mint` 可能会好看一点，我这里就不写了。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 18, INV2 = 499122177, P = 998244353;
int n, a[1<<N], b[1<<N], A[1<<N], B[1<<N], C[1<<N];

void OR(int A[], int inv) {
    for (int len = 2, half = 1; len <= 1 << n; len <<= 1, half <<= 1) {
        for (int i = 0; i < 1 << n; i += len) {
            for (int j = 0; j < half; j++) {
                // i: block start, j: block pointer
                A[i+j+half] = (A[i+j+half] + A[i+j] * inv) % P;
                if (A[i+j+half] < 0) A[i+j+half] += P;
            }
        }
    }
}

void AND(int A[], int inv) {
    for (int len = 2, half = 1; len <= 1 << n; len <<= 1, half <<= 1) {
        for (int i = 0; i < 1 << n; i += len) {
            for (int j = 0; j < half; j++) {
                // i: block start, j: block pointer
                A[i+j] = (A[i+j] + A[i+j+half] * inv) % P;
                if (A[i+j] < 0) A[i+j] += P;
            }
        }
    }
}

void XOR(int A[]) {
    for (int len = 2, half = 1; len <= 1 << n; len <<= 1, half <<= 1)
        for (int i = 0; i < 1 << n; i += len)
            for (int j = 0; j < half; j++) {
                int A0 = A[i+j], A1 = A[i+j+half];
                A[i+j] = (A0 + A1) % P;
                A[i+j+half] = (A0 - A1) % P;
                if (A[i+j+half] < 0) A[i+j+half] += P;
            }
}

void IXOR(int A[]) {
    for (int len = 2, half = 1; len <= 1 << n; len <<= 1, half <<= 1)
        for (int i = 0; i < 1 << n; i += len)
            for (int j = 0; j < half; j++) {
                int A0 = A[i+j], A1 = A[i+j+half];
                A[i+j] = 1LL * (A0 + A1) * INV2 % P;
                A[i+j+half] = 1LL * (A0 - A1) * INV2 % P;
                if (A[i+j+half] < 0) A[i+j+half] += P;
            }
}

void cpy() {
    for (int i = 0; i < 1 << n; i++) A[i] = a[i];
    for (int i = 0; i < 1 << n; i++) B[i] = b[i];
}

void mul() {
    for (int i = 0; i < 1 << n; i++) C[i] = 1LL * A[i] * B[i] % P;
}

void out() {
    for (int i = 0; i < 1 << n; i++) printf("%d ", C[i]);
    puts("");
}

int main() {
    scanf("%d", &n);
    for (int i = 0; i < 1 << n; i++) scanf("%d", &a[i]);
    for (int i = 0; i < 1 << n; i++) scanf("%d", &b[i]);

    cpy();
    OR(A, 1), OR(B, 1);
    mul();
    OR(C, -1);
    out();

    cpy();
    AND(A, 1), AND(B, 1);
    mul();
    AND(C, -1);
    out();

    cpy();
    XOR(A), XOR(B);
    mul();
    IXOR(C);
    out();

    return 0;
}
```

