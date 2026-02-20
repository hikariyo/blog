---
date: 2026-02-20 14:54:00
updated: 2026-02-20 14:54:00
title: 数学 - FWT
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: FWT 公式推导与板子。
---

## 简介

FWT 是用来求位运算卷积的工具，对于长度为 $2^n$ 的数组 $A,B,C$，它们的元素类型通常是 $\mathbb Z_p$ 或 $\mathbb Z$，若：
$$
\forall 0\le k\le 2^n-1, C_k=\sum_{i=0}^{2^n-1}\sum_{j=0}^{2^n-1} A_iB_j[i\circ j=k]
$$
其中 $[-]$ 是艾弗森括号，那么称 $C$ 是 $A,B$ 关于 $\circ$ 运算的卷积，记作 $C=A*B$。

我们希望找到一个可逆的变换 $\mathcal F$，使得：
$$
\mathcal F(C)=\mathcal F(A)\cdot \mathcal F(B)
$$
其中 $\cdot$ 是逐元素的乘法。

其次，我们也希望这是一个线性变换，即：
$$
\mathcal F(kA+lB)=\mathcal kF(A)+\mathcal lF(B)
$$
其中 $k,l$ 是任意常数，加法是逐元素加法，乘法是数量乘法。

## 变换

下面的问题就是，怎么找到这样的变换？

设 $c_{i,j}$ 是 $A_j$ 对 $\mathcal F_i(A)$ 的贡献系数，我们可以先构造出这样的一种变换：
$$
\mathcal F_i(A)=\sum_{j=0}^{2^n-1} c_{i,j} A_j
$$
其中 $c_{i,j}$ 满足这样的性质：

1. 设 $i$ 的二进制表示是 $i_{n-1}\cdots i_1i_0$，$j$ 的二进制表示是 $j_{n-1}\cdots j_1j_0$，那么 $c_{i,j}=\prod_{k=0}^{n-1} c_{i_k,j_k}$。
2. 对于任意的 $i,j$，$c_{i,j}\in \{-1,0,1\}$。

为什么这么设？因为这是一个易于处理的假设，因为只要知道了 $c_{0/1,0/1}=\pm1/0$，就能求出所有的 $c_{i,j}$，而前者一共只有 $3^4$ 种可能性。

根据我们的需求，展开 $\mathcal F_i(C)=\mathcal F_i(A)\mathcal F_i(B)$，可以得到：
$$
\begin{aligned}
\mathcal F_i(C)&=\sum_{p=0}^{2^n-1} C_pc_{i,p}\\
&=\sum_{p=0}^{2^n-1}\sum_{j=0}^{2^n-1}\sum_{k=0}^{2^n-1} A_jB_k c_{i,p}[j\circ k=p]\\
&=\sum_{j=0}^{2^n-1}\sum_{k=0}^{2^n-1} A_jB_k \left(\sum_{p=0}^{2^n-1}c_{i,p}[j\circ k=p]\right)\\
&=\sum_{j=0}^{2^n-1}\sum_{k=0}^{2^n-1} A_jB_k c_{i,j\circ k}\\
\mathcal F_i(A)\mathcal F_i(B)&=\sum_{j=0}^{2^n-1}\sum_{k=0}^{2^n-1} A_jB_kc_{i,j}c_{i,k}
\end{aligned}
$$
因此 $c_{i,j}c_{i,k}=c_{i,j\circ k}$ 是满足 $\mathcal F(C)=\mathcal F(A)\cdot \mathcal F(B)$ 的充要条件。

当我们在寻找逆变换之前，我们来看如何快速地求出 $\mathcal F_i(A)$。

由于这和二进制相关，所以我们按照 $0\sim 2^{n-1}-1$ 和 $2^{n-1}\sim 2^n-1$ 分组，设 $i$ 在二进制从低到高的第 $n$ 位是 $i_0$，那么：
$$
\begin{aligned}
\mathcal F_i(A)&=\sum_{j=0}^{2^{n-1}-1}c_{i,j}A_j+\sum_{j=2^{n-1}}^{2^n-1}c_{i,j}A_j\\
&=c_{i_0,0}\left(\sum_{j=0}^{2^{n-1}-1} c_{i-2^{n-1}i_0,j} A_{j}\right)+c_{i_0,1}\left(\sum_{j=0}^{2^{n-1}-1}c_{i-2^{n-1}i_0,j}A_{j+2^{n-1}}\right)\\
&=c_{i_0,0}\mathcal F_{i-2^{n-1}i_0}(A^L)+c_{i_0,1}\mathcal F_{i-2^{n-1}i_0} (A^R)
\end{aligned}
$$
其中 $\mathcal F(A^L)$ 和 $\mathcal F(A^{R})$ 代表将 $A_0\sim A_{2^{n-1}-1}$ 和 $A_{2^{n-1}}\sim A_{2^n-1}$ 看作两个独立的数组解出的子问题的答案。

满足 $i-2^{n-1}i_0=m$（$m$ 是某个常数）的 $i$ 有且仅有两个 $i_L,i_R$，它们的 $i_0$ 对应着 $0$ 和 $1$。

也就是说：
$$
\begin{aligned}
\mathcal F_{i_L}(A)&=c_{0,0} \mathcal F_m(A^L)+c_{0,1}\mathcal F_m(A^R)\\
\mathcal F_{i_R}(A)&=c_{1,0}\mathcal F_m(A^L)+c_{1,1}\mathcal F_m(A^R)
\end{aligned}
$$
如果矩阵 $\begin{bmatrix}c_{0,0}&c_{0,1}\\c_{1,0}&c_{1,1}\end{bmatrix}$ 可逆，那么这个变换自然就可逆。

+ 当 $\circ$ 是按位与运算时，$c_{0,0},c_{0,1},c_{1,0},c_{1,1}=1,1,0,1$ 是满足要求的；
+ 当 $\circ$ 是按位或运算时，$1,0,1,1$ 是满足要求的；
+ 当 $\circ$ 是按位异或运算时，$1,1,1,-1$ 是满足要求的。

## 逆变换

接下来我们看，具体如何进行逆变换。

对于变换 $\mathcal F$，我们想找到一个 $\mathcal G$ 满足：
$$
\mathcal G_i(\mathcal F(A))= A_i
$$
设 $\mathcal G$ 对应的贡献系数是 $d_{i,j}$，那么展开可以得到：
$$
\begin{aligned}
\mathcal G_i(\mathcal F(A))&=\sum_{j=0}^{2^n-1} d_{i,j}\mathcal F_j(A)\\
&=\sum_{j=0}^{2^n-1} d_{i,j}\left(\sum_{k=0}^{2^n-1} c_{j,k} A_k\right)\\
&=\sum_{k=0}^{2^n-1} A_k\left(\sum_{j=0}^{2^n-1} d_{i,j}c_{j,k}\right)
\end{aligned}
$$
于是我们希望 $\sum_{j=0}^{2^n-1} d_{i,j}c_{j,k}=[i=k]$，这显然是一个矩阵的形式，于是这个式子成立的充要条件是 $d$ 的矩阵与 $c$ 的矩阵之积是单位阵 $I$。

进一步地，如果下面的式子成立：
$$
\begin{bmatrix}c_{0,0}&c_{0,1}\\c_{1,0}&c_{1,1}\end{bmatrix} \begin{bmatrix}d_{0,0}&d_{0,1}\\d_{1,0}&d_{1,1}\end{bmatrix}=I_2
$$
令左边的矩阵是 $F$，右边的矩阵是 $G$，就会有：
$$
\begin{bmatrix}
c_{0,0}&c_{0,1}&c_{0,2}&c_{0,3}\\
c_{1,0}&c_{1,1}&c_{1,2}&c_{1,3}\\
c_{2,0}&c_{2,1}&c_{2,2}&c_{2,3}\\
c_{3,0}&c_{3,1}&c_{3,2}&c_{3,3}\\
\end{bmatrix}=
\begin{bmatrix}
c_{0,0} F&c_{0,1}F\\
c_{1,0} F&c_{1,1}F
\end{bmatrix},

\begin{bmatrix}
d_{0,0}&d_{0,1}&d_{0,2}&d_{0,3}\\
d_{1,0}&d_{1,1}&d_{1,2}&d_{1,3}\\
d_{2,0}&d_{2,1}&d_{2,2}&d_{2,3}\\
d_{3,0}&d_{3,1}&d_{3,2}&d_{3,3}\\
\end{bmatrix}=
\begin{bmatrix}
d_{0,0} G&d_{0,1}G\\
d_{1,0} G&d_{1,1}G
\end{bmatrix}
$$
于是，这两个矩阵按照分块矩阵的乘法乘起来也是单位阵 $I_4$，以此类推，最大的那个 $F,G$ 乘起来就是 $I_{2^n}$。

注意，由 $2\times 2$ 到 $4\times 4$ 的过程中，以 $c_{0,0}$ 为例，它其实发生了一个微妙的变化：原来 $2\times 2$ 的矩阵中，这个 $0/1$ 是长度为 $1$ 的二进制数，扩张之后是长度为 $2$ 的二进制数。这在当前讨论的与或异或卷积的体现不是很明显，但是在下面的一般化的公式中这就有区别了。

下面是洛谷模板题的代码：

```cpp
void OR(int A[], int n) {
    for (int len = 2, half = 1; len <= 1 << n; len <<= 1, half <<= 1) {
        for (int i = 0; i < 1 << n; i += len) {
            for (int j = 0; j < half; j++) {
                // i: block start, j: block pointer
                A[i+j+half] = (A[i+j+half] + A[i+j]) % P;
            }
        }
    }
}

void IOR(int A[], int n) {
        for (int len = 2, half = 1; len <= 1 << n; len <<= 1, half <<= 1) {
        for (int i = 0; i < 1 << n; i += len) {
            for (int j = 0; j < half; j++) {
                A[i+j+half] = (A[i+j+half] - A[i+j] + P) % P;
            }
        }
    }
}

void AND(int A[], int n) {
    for (int len = 2, half = 1; len <= 1 << n; len <<= 1, half <<= 1) {
        for (int i = 0; i < 1 << n; i += len) {
            for (int j = 0; j < half; j++) {
                A[i+j] = (A[i+j] + A[i+j+half]) % P;
            }
        }
    }
}

void IAND(int A[], int n) {
    for (int len = 2, half = 1; len <= 1 << n; len <<= 1, half <<= 1) {
        for (int i = 0; i < 1 << n; i += len) {
            for (int j = 0; j < half; j++) {
                A[i+j] = (A[i+j] - A[i+j+half] + P) % P;
            }
        }
    }
}

void XOR(int A[], int n) {
    for (int len = 2, half = 1; len <= 1 << n; len <<= 1, half <<= 1)
        for (int i = 0; i < 1 << n; i += len)
            for (int j = 0; j < half; j++) {
                int A0 = A[i+j], A1 = A[i+j+half];
                A[i+j] = (A0 + A1) % P;
                A[i+j+half] = (A0 - A1) % P;
                if (A[i+j+half] < 0) A[i+j+half] += P;
            }
}

void IXOR(int A[], int n) {
    for (int len = 2, half = 1; len <= 1 << n; len <<= 1, half <<= 1)
        for (int i = 0; i < 1 << n; i += len)
            for (int j = 0; j < half; j++) {
                int A0 = A[i+j], A1 = A[i+j+half];
                A[i+j] = 1ll * (A0 + A1) * INV2 % P;
                A[i+j+half] = 1ll * (A0 - A1) * INV2 % P;
                if (A[i+j+half] < 0) A[i+j+half] += P;
            }
}
```

## 一般化

这里给出一道题目：[Belarusian State University](https://qoj.ac/contest/536/problem/1083)。

对于任意给定真值表的二进制位独立的运算 $\circ$，我们还能导出上面的结论吗？答案是否定的。

例如，如果 $j\circ k\equiv 1$（恒等于 $1$），那么可以推出 $c_{i,j}\equiv 1$，此时的矩阵显然就是不可逆的了。

此时我们退而求其次，可以尝试找三种变换 $\mathcal F,\mathcal G,\mathcal H$，满足：
$$
\mathcal H(A*B)=\mathcal F(A)\cdot \mathcal G(B)
$$
其中 $\mathcal H$ 是可逆的变换，$\mathcal F,\mathcal G$ 可以不可逆。

设 $\mathcal H$ 对应的系数是 $h_{0/1,0/1}$，由上一节的推导同理可得 $h_{i,j\circ k}=f_{i,j}g_{i,k}$。

由于运算一共有 $2^4$ 种，然后 $f,g,h$ 的取值一共有 $(3^4)^3$ 种，这是可以暴力寻找的。

下面是打表程序：

```cpp
#include <bits/stdc++.h>
using namespace std;

int pow3[5] = {
    1, 3, 9, 27, 81,
};

int calc(int rule, int a, int b) {
    return rule >> ((a << 1) | b) & 1;
}

bool validate(int rule, int f, int g, int h) {
    for (int i = 0; i < 2; i++) {
        for (int j = 0; j < 2; j++) {
            for (int k = 0; k < 2; k++) {
                // f(i, j) g(i, k) = h(i, j op k)
                int ij = (i << 1) | j;
                int ik = (i << 1) | k;
                int ijk = (i << 1) | calc(rule, j, k);
                if (
                    (f / pow3[ij] % 3 - 1) * (g / pow3[ik] % 3 - 1) !=
                    (h / pow3[ijk] % 3 - 1)
                ) {
                    return false;
                }
            }
        }
    }
    return true;
}

array<tuple<int,int,int>, 16> init() {
    array<tuple<int,int,int>, 16> res{};
    for (int rule = 0; rule < 16; rule++) {
        for (int h = 0; h < pow3[4]; h++) {
            // validate whether h is inversable
            int h00 = h % 3 - 1;
            int h01 = h / 3 % 3 - 1;
            int h10 = h / 9 % 3 - 1;
            int h11 = h / 27 % 3 - 1;
            if (h00 * h11 - h01 * h10 == 0) continue;

            for (int f = 0; f < pow3[4]; f++) {
                for (int g = 0; g < pow3[4]; g++) {
                    if (validate(rule, f, g, h)) {
                        res[rule] = {f, g, h};
                    }
                }
            }
        }
    }
    return res;
}

signed main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    auto tab = init();
    for (auto [f, g, h]: tab) {
        cout << '{' << f << ',' << g << ',' << h << "},";
    }
    return 0;
}
```

用这个打出来的表，就可以来做自定义运算的 FWT 了。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
tuple<int,int,int> fgh[] = {{80,76,79},{77,77,79},{77,79,79},{77,80,79},{79,77,79},{80,77,79},{78,74,78},{79,79,77},{79,79,79},{78,78,78},{80,79,79},{79,77,77},{79,80,79},{77,79,77},{77,77,77},{80,80,79}};
int rule[20];
int A[1 << 18], B[1 << 18];

template<size_t F>
void FWT(int n, int A[]) {
    for (int len = 2, half = 1, k = 1; len <= 1 << n; len <<= 1, half <<= 1, k++) {
        int f = get<F>(fgh[rule[k]]);
        int f00 = f % 3 - 1;
        int f01 = f / 3 % 3 - 1;
        int f10 = f / 9 % 3 - 1;
        int f11 = f / 27 % 3 - 1;
        for (int i = 0; i < 1 << n; i += len) {
            for (int j = 0; j < half; j++) {
                int A0 = A[i+j], A1 = A[i+j+half];
                A[i+j] = f00 * A0 + f01 * A1;
                A[i+j+half] = f10 * A0 + f11 * A1;
            }
        }
    }
}

void IFWT(int n, int A[]) {
    for (int len = 2, half = 1, k = 1; len <= 1 << n; len <<= 1, half <<= 1, k++) {
        int h = get<2>(fgh[rule[k]]);
        int h00 = h % 3 - 1;
        int h01 = h / 3 % 3 - 1;
        int h10 = h / 9 % 3 - 1;
        int h11 = h / 27 % 3 - 1;

        int div = h00 * h11 - h01 * h10;
        int r00 = h11, r01 = -h01, r10 = -h10, r11 = h00;

        for (int i = 0; i < 1 << n; i += len) {
            for (int j = 0; j < half; j++) {
                int A0 = A[i+j], A1 = A[i+j+half];
                A[i+j] = r00 * A0 + r01 * A1;
                A[i+j+half] = r10 * A0 + r11 * A1;
                assert(A[i+j] % div == 0);
                A[i+j] /= div;
                assert(A[i+j+half] % div == 0);
                A[i+j+half] /= div;
            }
        }
    }
}

signed main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    int n;
    cin >> n;
    for (int k = 1; k <= n; k++) {
        string s;
        cin >> s;
        for (int i = 0; i < 2; i++) {
            for (int j = 0; j < 2; j++) {
                rule[k] ^= (s[(i << 1) | j] - '0') << ((i << 1) | j);
            }
        }
    }
    for (int i = 0; i < 1 << n; i++) cin >> A[i];
    FWT<0>(n, A);
    for (int i = 0; i < 1 << n; i++) cin >> B[i];
    FWT<1>(n, B);

    for (int i = 0; i < 1 << n; i++) A[i] = A[i] * B[i];
    IFWT(n, A);
    for (int i = 0; i < 1 << n; i++) {
        cout << A[i] << " \n"[i == (1 << n) - 1];
    }

    return 0;
}
```

