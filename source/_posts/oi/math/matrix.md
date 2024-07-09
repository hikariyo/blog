---
date: 2023-12-03 17:40:00
title: 数学 - 矩阵
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
description: 矩阵乘法是可以用来描述线性递推的一种技术，并且矩阵乘法具有结合律，因此我们可以进行快速幂操作，从而快速地求出知道递推数列某一项的值。除此之外，矩阵也可以描述一些变量之间的线性关系，观察到相互影响的线性关系的变量就可以考虑能否使用矩阵进行优化。
---

## 简介

矩阵乘法是可以用来描述线性递推的一种技术，并且矩阵乘法具有结合律，因此我们可以进行快速幂操作，从而快速地求出知道递推数列某一项的值。除此之外，矩阵也可以描述一些变量之间的线性关系，观察到相互影响的线性关系的变量就可以考虑能否使用矩阵进行优化。

对于矩阵乘法定义如下：

$$
A=BC,A_{ij}=\sum_{k=1}^n B_{ik}C_{kj}
$$

不难看出需要满足 $B$ 的列数等于 $C$ 的行数，在 OI 里转移矩阵一般都是方阵，所以对这个不是很关心。

可以证明矩阵乘法满足结合率，即：
$$
(AB)C=A(BC)\\
\begin{aligned}
L_{ij}&=\sum_{l=1}^n(\sum_{k=1}^n A_{ik}B_{kl})C_{lj}\\
&=\sum_{l=1}^n(\sum_{k=1}^n A_{ik}B_{kl}C_{lj})\\
R_{ij}&=\sum_{k=1}^nA_{ik}(\sum_{l=1}^nB_{kl}C_{lj})\\
&=\sum_{k=1}^n(\sum_{l=1}^n A_{ik}B_{kl}C_{lj})\\
&=L_{ij}
\end{aligned}
$$

本质上是加法具有交换律、乘法对加法具有分配律的体现，因此还有另一种矩阵乘法的变形，把乘法变成加法，把加法变成取 $\min /\max$，叫广义矩阵乘法，本质上还是因为 $\min/\max$ 具有交换律，加法对它们具有结合律的原因。

## 矩阵快速幂

> 给定 $n\times n$ 矩阵 $A$，求 $A^k$，每个元素对 $10^9+7$ 取模。
>
> 特别地，定义 $A^0=E$ 即单位矩阵。
>
> 题目链接：[P3390](https://www.luogu.com.cn/problem/P3390)。

快速复习：$A\times B$ 对应元素 $c_{i,j}$ 是 $A$ 的第 $i$ 行向量与 $B$ 的第 $j$ 列向量内积。

```cpp
#include <cstring>
#include <vector>
#include <cstdio>
using namespace std;

typedef long long LL;
const int N = 110, mod = 1e9+7;

struct Matrix {
    LL data[N][N];
    int rows, cols;
    
    Matrix(int r, int c) {
        rows = r, cols = c;
        memset(data, 0, sizeof data);
    }

    void build() {
        for (int i = 1; i <= rows; i++) {
            data[i][i] = 1;
        }
    }

    LL* operator[](int x) {
        return data[x];
    }
};

Matrix operator*(Matrix a, Matrix b) {
    Matrix res(a.rows, b.cols);
    for (int i = 1; i <= a.rows; i++) {
        for (int j = 1; j <= b.cols; j++) {
            // a[i][~] dot b[~][j]
            for (int k = 1; k <= a.cols; k++) {
                res[i][j] = (res[i][j] + a[i][k]*b[k][j]) % mod;
            }
        }
    }
    return res;
}

Matrix qmi(Matrix a, LL k) {
    Matrix res(a.rows, a.cols);
    res.build();
    while (k) {
        if (k & 1) res = res * a;
        a = a * a;
        k >>= 1;
    }
    return res;
}

int main() {
    LL n, k;
    scanf("%lld%lld", &n, &k);
    Matrix a(n, n);
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            scanf("%lld", &a[i][j]);
        }
    }
    Matrix b = qmi(a, k);
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            printf("%lld ", b[i][j]);
        }
        puts("");
    }
    return 0;
}
```

利用矩阵快速幂可以求解递推的问题。

## 斐波那契公约数

> 给定 $n,m\le 10^9$，求 $\gcd(f_n,f_m)$，对 $10^8$ 取模。
>
> 题目链接：[P1306](https://www.luogu.com.cn/problem/P1306)。

首先证明一下一个重要定理：

$$
\gcd(f_n,f_m)=f_{\gcd(n,m)}
$$

为了证明它，需要几个引理。

### 引理1

斐波那契相邻项互质，即：
$$
\gcd(f_n,f_{n+1})=1
$$
证明：
$$
\gcd(f_n,f_{n+1})=\gcd(f_n,f_n+f_{n-1})=\gcd(f_{n-1},f_n)=\gcd(f_1,f_2)=1
$$

### 引理2

斐波那契下标和之间的关系：
$$
f_nf_m+f_{n-1}f_{m-1}=f_{n+m-1}
$$
注意到 $n,m$ 顺序是无关紧要的，所以固定 $n$，用归纳法证明。

当 $m=1$ 时，原式为 $f_n=f_n$，显然成立。

当 $m=2$ 时，原式为 $f_nf_2+f_{n-1}f_1=f_{n+1}$，成立。

设当 $m\le k$ 时成立，推导 $m=k+1$ 的情况：
$$
\begin{aligned}
f_nf_{k+1}+f_{n-1}f_k&=f_n(f_k+f_{k-1})+f_{n-1}(f_{k-2}+f_{k-1})\\
&=f_nf_k+f_nf_{k-1}+f_{n-1}f_{k-2}+f_{n-1}f_{k-1}\\
&=(f_nf_k+f_{n-1}f_{k-1})+(f_nf_{k-1}+f_{n-1}f_{k-2})\\
&=f_{n+k-1}+f_{n+k-2}\\
&=f_{n+k}
\end{aligned}
$$
得证。

### 定理

若证明出这个式子，即可证明原式：
$$
\gcd(f_n,f_m)=\gcd(f_{n-m},f_m)\\
$$
套用引理 2 可以得到：
$$
\begin{aligned}
\gcd(f_{n},f_m)&=\gcd(f_{n-m+1}f_{m}+f_{n-m}f_{m-1},f_m)\\
&=\gcd(f_{n-m}f_{m-1},f_m)\\
&=\gcd(f_{n-m},f_m)
\end{aligned}
$$
最后一步是由引理 1 相邻项互质得到的，于是证明出来了这个定理。

### 递推

我们可以把 $f_{i+2}=f_{i+1}+f_i$ 写成矩阵的形式。
$$
\begin{bmatrix}
f_{i+2}\\
f_{i+1}
\end{bmatrix}
=
\begin{bmatrix}
1&1\\
1&0
\end{bmatrix}
\begin{bmatrix}
f_{i+1}\\
f_{i}
\end{bmatrix}
$$
因此：
$$
\begin{bmatrix}
f_n\\
f_{n-1}
\end{bmatrix}
=
\begin{bmatrix}
1&1\\
1&0
\end{bmatrix}^{n-1}
\begin{bmatrix}
f_{1}\\
f_{0}
\end{bmatrix}
$$
所以我们可以在 $O(\log n)$ 的复杂度内求出一项。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int P = 1e8;

struct Matrix {
    int dat[2][2];

    Matrix() {
        memset(dat, 0, sizeof dat);
    }

    Matrix operator*(const Matrix& mat) const {
        Matrix res;

        for (int k = 0; k < 2; k++)
            for (int i = 0; i < 2; i++)
                for (int j = 0; j < 2; j++)
                    res.dat[i][j] = (res.dat[i][j] + 1LL * dat[i][k] * mat.dat[k][j]) % P;
        
        return res;
    }
};

Matrix qmi(Matrix a, int k) {
    Matrix res;
    res.dat[0][0] = res.dat[1][1] = 1;

    while (k) {
        if (k & 1) res = res * a;
        a = a * a;
        k >>= 1;
    }

    return res;
}

int gcd(int a, int b) {
    if (!b) return a;
    return gcd(b, a%b);
}

int main() {
    int n, m;
    scanf("%d%d", &n, &m);

    Matrix mat;
    mat.dat[0][0] = mat.dat[0][1] = mat.dat[1][0] = 1;
    printf("%d\n", qmi(mat, gcd(n, m)-1).dat[0][0]);
    return 0;
}
```

## [HNOI2008] GT 考试

> 阿申准备报名参加 GT 考试，准考证号为 n 位数 X1X2⋯Xn，他不希望准考证号上出现不吉利的数字。
>
> 他的不吉利数字 A1A2⋯Am 有 m 位，不出现是指 X1X2⋯Xn 中没有恰好一段等于 A1A2⋯Am，A1 和 X1 可以为 0。
>
> 题目链接：[P3193](https://www.luogu.com.cn/problem/P3193)，[AcWing 1305](https://www.acwing.com/problem/content/1307/)。

+ 状态表示 $f(i,j)$：

    + 集合：长度为 $i$，与不吉利串匹配长度为 $j$ 的所有串的集合。
    + 属性：方案数。

+ 状态转移：
    $$
    \left \{\begin{aligned}
    f(i+1,k_0)&=a_{00}f(i,j_0)+a_{01}f(i,j_1)+\cdots\\
    f(i+1,k_1)&=a_{10}f(i,j_0)+a_{11}f(i,j_1)+\cdots\\
    \end{aligned}\right .
    $$
    其中 $a_{ij}$ 表示从 $j_n \to k_m$ 有几种方式，观察到这里是线性方程组，并且 $a$ 的取值与 $i$ 无关，因此可以用矩阵的形式写：
    $$
    F_{i+1}=F_iA\\
    F_n=F_0A^n
    $$

+ 初始化：$f(0,0)=1$

枚举所有状态 $j$，再枚举下一位字符，用 KMP 算法求出 $k$，然后 $a_{jk}$ 自增；

当计算 $F_{i+1}$ 时，就有 $f(i+1,k)=a_{jk}f(i,j)+\cdots$ 这样就达到了目的。

```cpp
#include <cstdio>
#include <cstring>
using namespace std;

const int M = 25;
int n, m, mod, ne[M], a[M][M];
char str[M];

void mul(int res[][M], int a[][M], int b[][M]) {
    static int t[M][M];
    memset(t, 0, sizeof t);
    
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < m; j++) {
            for (int k = 0; k < m; k++) {
                t[i][j] = (t[i][j] + a[i][k] * b[k][j]) % mod;
            }
        }
    }
    
    memcpy(res, t, sizeof t);
}

int qmi(int k) {
    // 用 static 默认初始化 0
    static int f[M][M];
    f[0][0] = 1;
    while (k) {
        if (k & 1) mul(f, f, a);
        mul(a, a, a);
        k >>= 1;
    }
    int res = 0;
    for (int i = 0; i < m; i++) res = (res + f[0][i]) % mod;
    return res;
}

int main() {
    scanf("%d%d%d", &n, &m, &mod);
    scanf("%s", str+1);
    for (int i = 2, j = 0; i <= m; i++) {
        while (j && str[j+1] != str[i]) j = ne[j];
        if (str[j+1] == str[i]) j++;
        ne[i] = j;
    }
    for (int j = 0; j < m; j++) {
        for (char c = '0'; c <= '9'; c++) {
            int k = j;
            while (k && str[k+1] != c) k = ne[k];
            if (str[k+1] == c) k++;
            // f[j] -> f[k]
            if (k < m) a[j][k]++;
        }
    }
    printf("%d\n", qmi(n));
    return 0;
}
```

## [SDOI2017] 序列计数

> Alice 想要得到一个长度为 $n$ 的序列，序列中的数都是不超过 $m$ 的正整数，而且这 $n$ 个数的和是 $p$ 的倍数。
>
> Alice 还希望，这 $n$ 个数中，至少有一个数是质数。
>
> Alice 想知道，有多少个序列满足她的要求。
>
> **输入格式**
>
> 一行三个数，$n,m,p$。
>
> **输出格式**
>
> 一行一个数，满足 Alice 的要求的序列数量，答案对 $20170408$ 取模。
>
> 对 $100\%$ 的数据，$1\leq n \leq 10^9,1\leq m \leq 2\times 10^7,1\leq p\leq 100$。
>
> 题目链接：[P3702](https://www.luogu.com.cn/problem/P3702)。

有一个很典的设计这种 “数位 DP” 的套路，就是设 $f(i,j)$ 为前 $i$ 个数之和余数为 $j$ 的所有方案数，当然这个题不是数位 DP。

如果没有要求一个数是质数，我们可以推出转移方程，枚举前面的数字之和模 $p$ 余 $k$，加上当前位 $a_i$ 模 $p$ 余 $j$，所以我们可以解方程 $k+a_i \equiv j$ 得到 $a_i\equiv j-k$，那么想法就是统计一下 $c_x$ 表示 $1\sim m$ 中模 $p$ 等于 $x$ 的数字的数量。
$$
f(i,j)= \sum_{k=0}^{p-1} f(i-1,k) c_{j-k}
$$
这是一个线性递推式，所以可以用矩阵乘法优化，展开写几项就能列出来了。
$$
f_0=\begin{bmatrix}1&0&0&\cdots&0\end{bmatrix}\\
f_{i,j} = \sum_{k=0}^{p-1} f_{i-1,k}c_{j-k}\\
f_i=f_{i-1}\begin{bmatrix}
c_{0}& c_{1}&\cdots& c_{p-1}\\
c_{-1}& c_{0}&\cdots& c_{p-2}\\
c_{-2}&c_{-1}&\cdots& c_{p-3}\\
\vdots&\vdots&\ddots& c_1\\
c_{-(p-1)}&c_{-(p-2)}&\cdots& c_0\\
\end{bmatrix}
$$
当然下标为负数的加上一个 $p$ 就行了。

如果需要去掉不合法的方案，也就是整个数列中不存在质数，只需要把 $c_x$ 动下手脚就可以，把对应位置质数都剪掉，然后再用矩阵快速幂求一遍得出结果。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <bitset>
#include <vector>
using namespace std;

typedef long long LL;
const int N = 110, M = 20000010, P = 20170408;
int n, m, p, c[N];
bitset<M> st;
vector<int> primes;

struct Mat {
    LL m[N][N];

    Mat() {
        memset(m, 0, sizeof m);
    }

    Mat operator*(const Mat& t) const {
        Mat res;

        for (int k = 0; k < p; k++)
            for (int i = 0; i < p; i++)
                for (int j = 0; j < p; j++)
                    res.m[i][j] = (res.m[i][j] + m[i][k] * t.m[k][j]) % P;
        return res;
    }
};

struct Vec {
    LL m[N];

    Vec() {
        memset(m, 0, sizeof m);
    }

    Vec operator*(const Mat& t) const {
        Vec res;
        for (int k = 0; k < p; k++)
            for (int j = 0; j < p; j++)
                res.m[j] = (res.m[j] + m[k] * t.m[k][j]) % P;
        return res;
    }
};

void init(int n) {
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes.emplace_back(i);
        for (int j = 0; primes[j] <= n/i; j++) {
            st[primes[j]*i] = 1;
            if (primes[j] % i == 0) break;
        }
    }
}

Mat qmi(Mat a, int k) {
    Mat res;
    for (int i = 0; i < p; i++) res.m[i][i] = 1;
    while (k) {
        if (k & 1) res = res * a;
        a = a * a;
        k >>= 1;
    }
    return res;
}

Vec solve() {
    Mat G;
    for (int i = 0; i < p; i++)
        for (int j = 0; j < p; j++)
            G.m[i][j] = c[(-i+j+p) % p];
    Vec f;
    f.m[0] = 1;
    return f * qmi(G, n);
}

int main() {
    scanf("%d%d%d", &n, &m, &p);
    init(m);
    for (int i = 1; i <= m; i++) c[i % p]++;
    Vec f = solve();
    for (int i: primes) c[i % p]--;
    Vec g = solve();
    return !printf("%lld\n", (f.m[0] - g.m[0] + P) % P);
}
```

## [ATDP] Walk

> 给一张有向简单图的邻接矩阵，求长度为 $K$ 的路径条数，答案对 $10^9+7$ 取模。
>
> 题目链接：[AT DP R](https://atcoder.jp/contests/dp/tasks/dp_r)。

设 $f(k,i,j)$ 是 $i\to j$ 长度为 $k$ 的路径条数。
$$
f(k,i,j)=\sum_{p\in V} f(k-1,i,p)\times f(1,p,j)
$$
这里 $f(1,i,j)$ 就是题目输入的邻接矩阵，乘上它的意思是要求 $p\to j$ 存在边。

所以就是一个矩阵加速转移的问题。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 50, P = 1e9+7;
int n, G[N][N];
LL k;

void mul(int A[N][N], int B[N][N]) {
    static int C[N][N];

    memset(C, 0, sizeof C);
    for (int k = 0; k < n; k++)
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                C[i][j] = (C[i][j] + (LL)A[i][k] * B[k][j]) % P;
    memcpy(A, C, sizeof C);
}

void qmi(LL k) {
    static int res[N][N];
    memset(res, 0, sizeof res);
    for (int i = 0; i < n; i++) res[i][i] = 1;

    while (k) {
        if (k & 1) mul(res, G);
        mul(G, G);
        k >>= 1;
    }

    memcpy(G, res, sizeof G);
}

int main() {
    scanf("%d%lld", &n, &k);
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            scanf("%d", &G[i][j]);
    
    qmi(k);
    int res = 0;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            res = (res + G[i][j]) % P;

    return !printf("%d\n", res);
}
```



## [USACO07NOV] Cow Relays G

> 给定一张 $T$ 条边的无向连通图，求从 $S$ 到 $E$ 经过 $N$ 条边的最短路长度。
>
> **输入格式**
>
> 第一行四个正整数 $N,T,S,E$ ，意义如题面所示。
>
> 接下来 $T$ 行每行三个正整数 $w,u,v$ ，分别表示路径的长度，起点和终点。
>
> **输出格式**
>
> 一行一个整数表示图中从 $S$ 到 $E$ 经过 $N$ 条边的最短路长度。
>
> **数据范围**
>
> 对于所有的数据，保证 $1\le N\le 10^6$，$2\le T\le 100$。
>
> 所有的边保证 $1\le u,v\le 1000$，$1\le w\le 1000$。
>
> 题目链接：[P2886](https://www.luogu.com.cn/problem/P2886)。

可以使用文章开头说到的广义矩阵乘法解决此问题，先用 DP 式子推一下：
$$
f(k,i,j)=\min_{p\in V} \{f(k-1,i,p)+f(1,p,j)\}
$$
这符合矩阵乘法的式子，所以用矩阵乘法加速。

观察到虽然点的范围很大，但是边数很小，因此先离散化一下，由于并不需要保证点编号的相对顺序大小就直接开个数组离散化即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 210, M = 1100;
const LL INF = 0x3f3f3f3f3f3f3f3fLL;
int k, n, m, S, T;
int p[M];

struct Matrix {
    LL dat[N][N];

    Matrix() {
        memset(dat, 0x3f, sizeof dat);
    }

    Matrix operator*(const Matrix& t) {
        Matrix res;
        for (int k = 1; k <= n; k++)
            for (int i = 1; i <= n; i++)
                for (int j = 1; j <= n; j++)
                    res.dat[i][j] = min(res.dat[i][j], dat[i][k] + t.dat[k][j]);
        return res;
    }
} G;

Matrix qmi(Matrix a, int k) {
    Matrix res;
    for (int i = 1; i <= n; i++) res.dat[i][i] = 0;
    while (k) {
        if (k & 1) res = res * a;
        a = a * a;
        k >>= 1;
    }
    return res;
}

int main() {
    scanf("%d%d%d%d", &k, &m, &S, &T);
    for (int i = 0, a, b, w; i < m; i++) {
        scanf("%d%d%d", &w, &a, &b);
        if (!p[a]) p[a] = ++n;
        if (!p[b]) p[b] = ++n;
		G.dat[p[a]][p[b]] = G.dat[p[b]][p[a]] = min(G.dat[p[a]][p[b]], 1LL*w);
	}
    G = qmi(G, k);
    return !printf("%lld\n", G.dat[p[S]][p[T]]);
}
```

## [NOI2020] 美食家

> 坐落在 Bzeroth 大陆上的精灵王国击退地灾军团的入侵后，经过十余年的休养生息，重新成为了一片欣欣向荣的乐土，吸引着八方游客。小 W 是一位游历过世界各地的著名美食家，现在也慕名来到了精灵王国。
>
> 精灵王国共有 $n$ 座城市，城市从 $1$ 到 $n$ 编号，其中城市 $i$ 的美食能为小 W 提供 $c_i$ 的愉悦值。精灵王国的城市通过 $m$ 条**单向道路**连接，道路从 $1$ 到 $m$ 编号，其中道路 $i$ 的起点为城市 $u_i$ ，终点为城市 $v_i$，沿它通行需要花费 $w_i$ 天。也就是说，若小 W 在第 $d$ 天从城市 $u_i$ 沿道路 $i$ 通行，那么他会在第 $d + w_i$ 天到达城市 $v_i$。
>
> 小 W 计划在精灵王国进行一场为期 $T$ 天的旅行，更具体地：他会在第 $0$ 天从城市 $1$ 出发，经过 $T$ 天的旅行，最终在**恰好第 $T$ 天**回到城市 $1$ 结束旅行。由于小 W 是一位美食家，每当他到达一座城市时（包括第 $0$ 天和第 $T$ 天的城市 $1$），他都会品尝该城市的美食并获得其所提供的愉悦值，若小 W 多次到达同一座城市，他将**获得多次愉悦值**。注意旅行途中小 W **不能在任何城市停留**，即当他到达一座城市且还未结束旅行时，他当天必须立即从该城市出发前往其他城市。
>
> 此外，精灵王国会在**不同**的时间举办 $k$ 次美食节。具体来说，第 $i$ 次美食节将于第 $t_i$ 天在城市 $x_i$ 举办，若小 W 第 $t_i$ 天时恰好在城市 $x_i$，那么他在品尝城市 $x_i$ 的美食时会**额外得到** $y_i$ 的愉悦值。现在小 W 想请作为精灵王国接待使者的你帮他算出，他在旅行中能获得的愉悦值之和的**最大值**。
>
> **输入格式**
>
> 第一行四个整数 $n, m, T, k$，依次表示城市数、道路条数、旅行天数与美食节次数。
>
> 第二行 $n$ 个整数 $c_i$，表示每座城市的美食所能提供的愉悦值。接下来 $m$ 行每行三个整数 $u_i, v_i, w_i$，依次表示每条道路的起点、终点与通行天数。
>
> 最后 $k$ 行每行三个整数 $t_i, x_i, y_i$，依次表示每次美食节的举办时间、举办城市与提供的额外愉悦值。
>
> 本题中数据保证：
> - 对所有 $1 \leq i \leq m$，有 $u_i\neq v_i$。但数据中可能存在路线重复的单向道路，即可能存在 $1 \leq i < j \leq m，使得 u_i = u_j, v_i = v_j$。
> - 对每座城市都满足：至少存在一条以该该城市为起点的单向道路。
> - 每次美食节的举办时间 $t_i$ 互不相同。
>
> **输出格式**
>
> 仅一行一个整数，表示小 W 通过旅行能获得的愉悦值之和的最大值。
>
> **若小 W 无法在第 $T$ 天回到城市 $1$，则输出 $-1$**。
>
> 对于所有测试点：
>
> $1 \leq n \leq 50$，$n \leq m \leq 501$，$0 \leq k \leq 200$，$1 \leq t_i \leq T \leq 10^9$。
>
> $1 \leq w_i \leq 5$，$1 \leq c_i \leq 52501$，$1 \leq u_i, v_i, x_i \leq n$，$1 \leq y_i \leq 10^9$。
>
> 题目链接：[P6772](https://www.luogu.com.cn/problem/P6772)。

简单地说，求总边长为 $T$ 的路径中点权和的最大值。注意到边权的长度都很小，所以可以把所有边都拆成长度为 1 的边，这样会导致点数变成 $5n$，新增点的点权为 0。然后可以用 Floyd 矩阵解决必须经过 $T$ 条边的最短路。

很难看出 DP 的转移方程，设 $f_{i,j}$ 为第 $i$ 天在 $j$ 个点获得的最大值：

$$
f_{i,j}=\max_k \{f_{i-1,k}+w_{k,j}\}+[t_j=i]y_i
$$

先不管 $y$ 这个东西，我们写出 $f_0$ 这个向量，由于只能从一号点出发， 所以其它点都是负无穷：
$$
f_0=[c_1,-\inf,\cdots,-\inf]
$$
看右边的转移，可以断定是 Flyod 矩阵的形式，所以考虑到矩阵快速幂优化，可以拿到 $k=0$ 的分数。
$$
f_n= f_0 \times G^T
$$
考虑 $k\ne 0$ 的情况，在每个活动之间直接通过矩阵乘法转移，在相隔为 $\Delta t$ 的两个活动间通过矩阵乘法转移：
$$
f_{t_1}=f_{t_0} \times G^{\Delta t}
$$
在第 $i$ 天的时候给 $f_{i,j}\leftarrow f_{i,j} + [t_j=i]y_i$ 即可。

如果你真的这么做了，就会像我一样拿到 60pts 的成绩，原因就是复杂度不对，这样硬算矩阵快速幂的复杂度是 $O((5n)^3k\log T)$ 比较极限，但是矩阵和向量相乘的单次复杂度是 $O((5n)^2)$，考虑预处理矩阵的二的指数次幂，然后二进制分解每个 $\Delta t$，这样做的复杂度是 $O((5n)^3\log T+(5n)^2k\log T)$，题目给了两秒，能过。

```cpp
#include <cmath>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 260, K = 32;
const LL INF = 1e18;
int n, m, T, k, c[N], tot;

struct Mat {
    LL m[N][N];
    
    Mat() {
        for (int i = 0; i < N; i++)
            for (int j = 0; j < N; j++)
                m[i][j] = -INF;
    }

    Mat operator*(const Mat& t) const {
        Mat res;
        for (int k = 1; k <= tot; k++)
            for (int i = 1; i <= tot; i++)
                for (int j = 1; j <= tot; j++)
                    res.m[i][j] = max(res.m[i][j], m[i][k] + t.m[k][j]);
        return res;
    }
} G, Gn[K];

struct Vec {
    LL m[N];
    Vec() {
        for (int i = 0; i < N; i++) m[i] = -INF;
    }

    Vec operator*(const Mat& t) const {
        Vec res;
        for (int k = 1; k <= tot; k++)
            for (int j = 1; j <= tot; j++)
                res.m[j] = max(res.m[j], m[k] + t.m[k][j]);
        return res;
    }
} f;

struct Event {
    int t, x, y;
    bool operator<(const Event& e) const {
        return t < e.t;
    }
} e[N];

int get(int u, int k) {
    return 5*u+k;
}

int main() {
    scanf("%d%d%d%d", &n, &m, &T, &k);
    tot = 5*n+5;

    for (int i = 1; i <= n; i++) scanf("%d", &c[i]);
    f.m[get(1, 0)] = c[1];

    for (int i = 0, u, v, w; i < m; i++) {
        scanf("%d%d%d", &u, &v, &w);

        for (int j = 0; j < w-1; j++) G.m[get(u, j)][get(u, j+1)] = 0;
        G.m[get(u, w-1)][get(v, 0)] = c[v];
    }

    Gn[0] = G;
    for (int i = 1; i < K; i++) Gn[i] = Gn[i-1] * Gn[i-1];

    for (int i = 0, t, x, y; i < k; i++) {
        scanf("%d%d%d", &t, &x, &y);
        e[i] = {t, x, y};
    }
    e[k++] = {0, 1, 0};
    e[k++] = {T, 1, 0};
    sort(e, e+k);
    for (int i = 1; i < k; i++) {
        int dt = e[i].t - e[i-1].t;
        for (int j = 0; j < K; j++)
            if (dt >> j & 1) f = f * Gn[j];
        f.m[get(e[i].x, 0)] += e[i].y;
    }
    return !printf("%lld\n", f.m[get(1, 0)] < 0 ? -1 : f.m[get(1, 0)]);
}
```

## [CSPS2022] 数据传输

> 给定 $n$ 个点 $n-1$ 条边的树和每个点的点权，$q$ 个询问，每次给定一个 $a_i,b_i$，每次可以跳不超过 $k$ 条边，问 $a_i\sim b_i$ 上经过点的最小点权和。
>
> 数据范围：$1\le n,q \le 2\times 10^5, 1\le k\le 3,1\le a_i,b_i\le n$。

### k=1

每次询问都是求一条链上的点权和，倍增维护或者树剖即可。

### k=2

假设我们已经把路径转化成一条长度为 $m$ 的链了，那么可以列出 $dp$ 式子：
$$
dp(i)=w(i)+\min\{dp(i-1),dp(i-2)\}
$$
由于跳到链外后再跳回链内一定会多一个点的点权，所以

但是我们用树剖会得到 $O(\log n)$ 条不完整的链，所以考虑广义矩阵加速：
$$
\begin{bmatrix}
dp(i-1)&dp(i-2)
\end{bmatrix}
\times
\begin{bmatrix}
w(i)&0\\
w(i)&+\infin
\end{bmatrix}
=
\begin{bmatrix}dp(i)&dp(i-1)\end{bmatrix}
$$
树剖维护矩阵之积即可，注意矩阵没有交换律，所以同时维护两个方向的矩阵之积。

### k=3

反正考场上写不太出来正解，这里就整理一下各位 DALAO 们题解的思路了。

这时可能跳到链外了，但是链外只有可能跳到一个儿子上去，如果往外跳的深度大于一，一定可以直接一步跳过去，所以只有可能跳到儿子上。

设 $dp(i)$ 是到当前点的最小点权和，$dp(i-1)$ 是距离 $i$ 为 $1$ 的最小点权和，所以每个点在第一次计算的时候是刚好到这个点，第二次计算的时候就可能是到下一个点然后跳出去的值了。

设 $o(u)$ 表示 $u$ 的儿子中最小的点权值。
$$
\begin{bmatrix}
dp(i-1)&dp(i-2)&dp(i-3)
\end{bmatrix}\times
\begin{bmatrix}
w(i)&0&+\infin\\
w(i)&o(i)&0\\
w(i)&+\infin&+\infin\\
\end{bmatrix}
=
\begin{bmatrix}
dp(i)&dp(i-1)&dp(i-2)
\end{bmatrix}
$$

### 结果

设第一个转移矩阵是 $A_1$，那么有：
$$
\begin{bmatrix}dp(0)&dp(-1)&dp(-2)\end{bmatrix} A_1=\begin{bmatrix}dp(1)&dp(0)&dp(-1)\end{bmatrix}
$$
然后我们想让 $dp(2)$ 只能从 $dp(1)$ 转移，所以设 $dp(0)=dp(-1)=+\infin,dp(-2)=0$，这样如果转移矩阵之积为 $A$，那么答案就是 $A_{k1}$，即 $A[k-1][0]$。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 200010, M = 400010;
const LL INF = 1e18;
int n, q, k;
int w[N], dep[N], h[N], idx = 2, ts;
LL o[N];
int dfnw[N], dfnid[N], sz[N], dfn[N], son[N], top[N], fa[N];

struct Edge {
    int to, nxt;
} e[M];

void add(int a, int b) {
    e[idx] = {b, h[a]}, h[a] = idx++;
}

void matmul(LL out[3][3], const LL A[3][3], const LL B[3][3]) {
    for (int i = 0; i < k; i++)
        for (int j = 0; j < k; j++)
            out[i][j] = INF;
    
    for (int p = 0; p < k; p++)
        for (int i = 0; i < k; i++)
            for (int j = 0; j < k; j++)
                out[i][j] = min(out[i][j], A[i][p] + B[p][j]);
}

struct Matrix {
    LL dat[3][3];

    void unit() {
        dat[0][0] = dat[1][1] = dat[2][2] = 0;
        dat[0][1] = dat[0][2] = dat[1][0] = dat[1][2] = dat[2][0] = dat[2][1] = INF;
    }

    void init(int v, int p) {
        dat[0][0] = dat[1][0] = dat[2][0] = v;
        dat[0][1] = dat[1][2] = 0;
        dat[0][2] = dat[1][1] = dat[2][1] = dat[2][2] = INF;

        if (k == 3) dat[1][1] = o[p];
    }

    Matrix operator*(const Matrix& m) const {
        Matrix res;
        matmul(res.dat, dat, m.dat);
        return res;
    }
};

struct Node {
    int l, r;
    Matrix mat, imat;
} tr[N<<2];

void pushup(int u) {
    tr[u].mat = tr[u<<1].mat * tr[u<<1|1].mat;
    tr[u].imat = tr[u<<1|1].imat * tr[u<<1].imat;
}

void build(int u, int l, int r) {
    tr[u].l = l, tr[u].r = r;
    if (l == r) return tr[u].mat.init(dfnw[l], dfnid[l]), tr[u].imat.init(dfnw[l], dfnid[l]), void();
    int mid = (l+r) >> 1;
    build(u<<1, l, mid), build(u<<1|1, mid+1, r);
    pushup(u);
}

Matrix query(int u, int l, int r, bool inv) {
    if (l <= tr[u].l && tr[u].r <= r) return inv ? tr[u].imat : tr[u].mat;

    int mid = (tr[u].l + tr[u].r) >> 1;
    if (r < mid+1) return query(u<<1, l, r, inv);
    if (l > mid) return query(u<<1|1, l, r, inv);
    if (inv) return query(u<<1|1, l, r, inv) * query(u<<1, l, r, inv);
    return query(u<<1, l, r, inv) * query(u<<1|1, l, r, inv);
}

int lca(int a, int b) {
    while (top[a] != top[b]) {
        if (dep[top[a]] < dep[top[b]]) swap(a, b);
        a = fa[top[a]];
    }
    return dep[a] > dep[b] ? b : a;
}

Matrix query_path(int x, int y) {
    int u = lca(x, y);
    
    Matrix res;
    res.unit();
    while (top[x] != top[u]) {
        res = res * query(1, dfn[top[x]], dfn[x], true);
        x = fa[top[x]];
    }
    res = res * query(1, dfn[u], dfn[x], true);

    vector<Matrix> mats;
    while (top[y] != top[u]) {
        mats.push_back(query(1, dfn[top[y]], dfn[y], false));
        y = fa[top[y]];
    }
    if (y != u) mats.push_back(query(1, dfn[son[u]], dfn[y], false));

    for (int i = mats.size()-1; i >= 0; i--) res = res * mats[i];
    return res;
}

void dfs1(int u, int father) {
    fa[u] = father, dep[u] = dep[father] + 1, sz[u] = 1;
    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == father) continue;
        dfs1(to, u);
        sz[u] += sz[to];
        if (sz[son[u]] < sz[to]) son[u] = to;
    }
}

void dfs2(int u, int t) {
    top[u] = t, dfn[u] = ++ts, dfnw[ts] = w[u], dfnid[ts] = u;
    if (!son[u]) return;
    dfs2(son[u], t);

    for (int i = h[u]; i; i = e[i].nxt) {
        int to = e[i].to;
        if (to == fa[u] || to == son[u]) continue;
        dfs2(to, to);
    }
}

int main() {
    scanf("%d%d%d", &n, &q, &k);
    for (int i = 1; i <= n; i++) scanf("%d", &w[i]);
    for (int i = 1, a, b; i < n; i++) {
        scanf("%d%d", &a, &b);
        add(a, b), add(b, a);
    }

    if (k == 3) {
        for (int u = 1; u <= n; u++) {
            o[u] = INF;
            for (int i = h[u]; i; i = e[i].nxt) {
                int to = e[i].to;
                o[u] = min(o[u], 1LL * w[to]);
            }
        }
    }
    
    dfs1(1, 0), dfs2(1, 1), build(1, 1, n);

    while (q--) {
        int x, y;
        scanf("%d%d", &x, &y);
        printf("%lld\n", query_path(x, y).dat[k-1][0]);
    }
    return 0;
}

```

## [NOI Online #1] 魔法
> 给定 $n$ 点 $m$ 边的有向图，可以使用 $k$ 次魔法，使得当前经过的边权变为相反数（只在这一次经过有效），求 $1$ 到 $n$ 的最短路。
>
> 对于全部的测试点，保证：$1 \leq n \leq 100$，$1 \leq m \leq 2500$，$0 \leq k \leq 10^6$，$1 \leq u_i, v_i \leq n$，$1 \leq t_i \leq 10^9$，给出的图无重边和自环，且至少存在一条能从 $1$ 到达 $n$ 。
>
> 题目链接：[P6190](https://www.luogu.com.cn/problem/P6190)。

### 分层图 + SPFA 75pts

这个没啥思维含量，数组开大点就行，我开预估空间的 $2\sim 3$ 倍。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
typedef pair<LL, int> PII;
const int N = 200010, M = 6000010;
int h[N], n, m, k, idx = 2;
LL dist[N];
bitset<N> st;

struct Edge {
    int to, nxt, w;
} e[M];

void add(int a, int b, int c) {
    e[idx] = {b, h[a], c}, h[a] = idx++;
}

int get(int v, int lev) {
    return lev * n + v;
}

void SPFA() {
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    queue<int> q;
    q.push(1);

    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int i = h[t]; i; i = e[i].nxt) {
            int to = e[i].to;
            if (dist[to] > dist[t] + e[i].w) {
                dist[to] = dist[t] + e[i].w;
                if (!st[to]) q.push(to), st[to] = true ;
            }
        }
    }
}

int main() {
    scanf("%d%d%d", &n, &m, &k);
    if (k > 1000) return !printf("-114514\n");

    for (int i = 1, a, b, c; i <= m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        for (int lev = 0; lev <= k; lev++) {
            add(get(a, lev), get(b, lev), c);
            if (lev+1 <= k) add(get(a, lev), get(b, lev+1), -c);
        }
    }

    for (int lev = 0; lev < k; lev++) {
        add(get(n, lev), get(n, lev+1), 0);
    }

    SPFA();

    return !printf("%lld\n", dist[get(n, k)]);
}
```

### 矩阵快速幂 100pts

首先用 Flyod 求出最短路，用 $f_0(i,j)$ 表示。

枚举每条边，转移一下：
$$
f_1(i,j)=\min\{f_0(i,j),f_0(i,a)+f(b,j)-w(a,b)\}
$$
然后看看如果用了两条边：
$$
f_2(i,j)=\min_{k\in V}\{f_1(i,k)+f_1(k,j)\}
$$
所以可以看成矩阵乘法，$f_k=f_1^{k}$。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
const int N = 110, M = 2510;
const LL INF = 1e18;
int n, m, k;
LL g[N][N];

struct Edge {
    int a, b, w;
} edge[M];

struct Matrix {
    LL dat[N][N];

    Matrix() {
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++)
                dat[i][j] = INF;
    }

    void unit() {
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++)
                dat[i][j] = i == j ? 0 : INF;
    }
};

Matrix operator*(const Matrix& a, const Matrix& b) {
    Matrix res;
    
    for (int k = 1; k <= n; k++)
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++)
                res.dat[i][j] = min(res.dat[i][j], a.dat[i][k] + b.dat[k][j]);

    return res;
}

Matrix qmi(Matrix a, int k) {
    Matrix res;
    res.unit();
    while (k) {
        if (k & 1) res = res * a;
        a = a * a;
        k >>= 1;
    }
    return res;
}   

int main() {
    scanf("%d%d%d", &n, &m, &k);

    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= n; j++)
            g[i][j] = i == j ? 0 : INF;

    for (int i = 1, a, b, c; i <= m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        edge[i] = {a, b, c};
        g[a][b] = min(g[a][b], 1LL * c);
    }

    for (int p = 1; p <= n; p++)
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++)
                g[i][j] = min(g[i][j], g[i][p] + g[p][j]);
    
    if (k == 0) printf("%lld\n", g[1][n]);
    else {
        Matrix f;

        for (int p = 1; p <= m; p++) {
            for (int i = 1; i <= n; i++) {
                for (int j = 1; j <= n; j++) {
                    int a = edge[p].a, b = edge[p].b, w = edge[p].w;
                    f.dat[i][j] = min(f.dat[i][j], min(g[i][j], g[i][a] - w + g[b][j]));
                }
            }
        }

        printf("%lld\n", qmi(f, k).dat[1][n]);
    }
    return 0;
}
```

## [RC-03] 记忆

> 有一个括号串 $S$，一开始 $S$ 中只包含一对括号（即初始的 $S$ 为 `()`），接下来有 $n$ 个操作，操作分为三种：
>
> 1. 在当前 $S$ 的末尾加一对括号（即 $S$ 变为 `S()`）；
>
> 2. 在当前 $S$ 的最外面加一对括号（即 $S$ 变为 `(S)`）；
>
> 3. 取消第 $x$ 个操作，即去除第 $x$ 个操作造成过的**一切影响**（例如，如果第 $x$ 个操作也是取消操作，且取消了第 $y$ 个操作，那么当前操作的实质就是恢复了第 $y$ 个操作的作用效果）。
>
> 每次操作后，你需要输出 $S$ 的能够括号匹配的非空子串（子串要求连续）个数。
>
> 一个括号串能够括号匹配，当且仅当其左右括号数量相等，且任意一个前缀中左括号数量不少于右括号数量。
>
> 
> 对于全部数据：$1\leq n\leq 2\times 10^5$，$op\in \{1,2,3\}$，$1\leq x\leq n$，一个操作在形式上最多只会被取消一次（即所有 $x$ 互不相同）。
> 
> 题目链接：[P6864](https://www.luogu.com.cn/problem/P6864)。

如果用 $x$ 表示答案，$y$ 表示极大的最外层括号，那么我们可以看出每个操作对这些数字的影响：

1. 在末尾加括号，使得 $x\leftarrow x + y + 1,y\leftarrow y+1$
2. 在外层包括号，使得 $x\leftarrow x + 1, y \leftarrow 1$

取消操作等会再说，可以看出现在这个可以用矩阵解决，设 $\begin{bmatrix}x\\y\\1\end{bmatrix}$ 是需要维护的向量，那么转移矩阵分别是 $\begin{bmatrix}1&1&1\\0&1&1\\0&0&1\end{bmatrix}$ 和 $\begin{bmatrix}1&0&1\\0&0&1\\0&0&1\end{bmatrix}$。

取消一个操作就是把某个操作换成单位阵，再恢复的时候就是把原先的矩阵再换上去，总体使用一个线段树维护矩阵积即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define Mid ((L+R) >> 1)
const int N = 200010;
int n;

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

struct Matrix {
    int dat[3][3];
    Matrix() {
        memset(dat, 0, sizeof dat);
        dat[0][0] = dat[1][1] = dat[2][2] = 1;
    }

    Matrix(initializer_list<int> lst) {
        assert(lst.size() == 9);
        int cnt = 0;
        for (int v: lst) dat[cnt/3][cnt%3] = v, cnt++;
    }

    Matrix operator*(const Matrix& mat) const {
        Matrix res;
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                res.dat[i][j] = dat[i][0]*mat.dat[0][j] + dat[i][1]*mat.dat[1][j] + dat[i][2]*mat.dat[2][j];
        return res;
    }
};

Matrix mat[N], prod[N<<2];
int rf[N];

void pushup(int u) {
    prod[u] = prod[u<<1] * prod[u<<1|1];
}

void modify(int u, int p, const Matrix& mat, Matrix& raw, int L, int R) {
    if (L == R) return raw = prod[u], prod[u] = mat, void();
    if (p <= Mid) modify(u<<1, p, mat, raw, L, Mid);
    else modify(u<<1|1, p, mat, raw, Mid+1, R);
    pushup(u);
}

signed main() {
    read(n);
    for (int i = 1; i <= n; i++) rf[i] = i;
    for (int i = 1, opt, x; i <= n; i++) {
        read(opt);
        if (opt == 1) modify(1, i, {
            1, 1, 1,
            0, 1, 1,
            0, 0, 1,
        }, mat[i], 1, n);
        else if (opt == 2) modify(1, i, {
            1, 0, 1,
            0, 0, 1,
            0, 0, 1,
        }, mat[i], 1, n);
        else {
            rf[i] = rf[read(x), x];
            Matrix cge = mat[rf[i]];
            modify(1, rf[i], cge, mat[rf[i]], 1, n);
        }
        printf("%lld\n", prod[1].dat[0][0] + prod[1].dat[0][1] + prod[1].dat[0][2]);
    }

    return 0;
}
```

## [THUSCH2017] 大魔法师
> 大魔法师小 L 制作了 $n$ 个魔力水晶球，每个水晶球有水、火、土三个属性的能量值。小 L 把这 $n$ 个水晶球在地上从前向后排成一行，然后开始今天的魔法表演。
>
> 我们用 $A_i,B_i,C_i$ 分别表示从前向后第 $i$ 个水晶球（下标从 $1$ 开始）的水、火、土的能量值。
>
> 小 L 计划施展 $m$ 次魔法。每次，他会选择一个区间 $[l,r]$，然后施展以下 $3$ 大类、$7$ 种魔法之一：
>
> 1. 魔力激发：令区间里每个水晶球中**特定属性**的能量爆发，从而使另一个**特定属性**的能量增强。具体来说，有以下三种可能的表现形式：
>
> 	- 火元素激发水元素能量：令 $A_i=A_i+B_i$。
> 	- 土元素激发火元素能量：令 $B_i=B_i+C_i$。
> 	- 水元素激发土元素能量：令 $C_i=C_i+A_i$。
>
>     **需要注意的是，增强一种属性的能量并不会改变另一种属性的能量，例如 $A_i=A_i+B_i$ 并不会使 $B_i$ 增加或减少。**
>
> 2. 魔力增强：小 L 挥舞法杖，消耗自身 $v$ 点法力值，来改变区间里每个水晶球的**特定属性**的能量。具体来说，有以下三种可能的表现形式：
>
> 	- 火元素能量定值增强：令 $A_i=A_i+v$。
> 	- 水元素能量翻倍增强：令 $B_i=B_i\times v$。
> 	- 土元素能量吸收融合：令 $C_i=v$。
> 3. 魔力释放：小 L 将区间里所有水晶球的能量聚集在一起，融合成一个新的水晶球，然后送给场外观众。生成的水晶球每种属性的能量值等于区间内所有水晶球对应能量值的代数和。**需要注意的是，魔力释放的过程不会真正改变区间内水晶球的能量。**
>
> 值得一提的是，小 L 制造和融合的水晶球的原材料都是定制版的 OI 工厂水晶，所以这些水晶球有一个能量阈值 $998244353$。当水晶球中某种属性的能量值大于等于这个阈值时，能量值会自动对阈值取模，从而避免水晶球爆炸。
>
> 小 W 为小 L（唯一的）观众，围观了整个表演，并且收到了小 L 在表演中融合的每个水晶球。小 W 想知道，这些水晶球蕴涵的三种属性的能量值分别是多少。
>
> $100\%$ 的数据，$n,m\le2.5\times 10^5,0\le A_i,B_i,C_i,v<998244353$
>
> 题目链接：[P7453](https://www.luogu.com.cn/problem/P7453)。

大模拟。放这个题想说的地方就是矩阵乘法的循环如果展开写可以加速非常多。

我们设每个点初始值都用一个向量 $\begin{bmatrix}A_i\\B_i\\C_i\\1\end{bmatrix}$ 来表示，多一个 $1$ 的原因是存在赋值和加常数的操作，需要额外用一个维度来支持。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define Mid ((L+R) >> 1)
const int P = 998244353, N = 250010;
int n, m, A[N], B[N], C[N];

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
}

struct Matrix {
    int dat[4][4];

    Matrix() {
        memset(dat, 0, sizeof dat);
    }

    Matrix(initializer_list<int> lst) {
        assert(lst.size() == 16);
        int cnt = 0;
        for (int v: lst) {
            dat[cnt/4][cnt%4] = v;
            cnt++;
        }
    }

    Matrix operator*(const Matrix& mat) const {
        Matrix res;
        res.dat[0][0] = (dat[0][0] * mat.dat[0][0] + dat[0][1] * mat.dat[1][0] + dat[0][2] * mat.dat[2][0] + dat[0][3] * mat.dat[3][0]) % P;
        res.dat[0][1] = (dat[0][0] * mat.dat[0][1] + dat[0][1] * mat.dat[1][1] + dat[0][2] * mat.dat[2][1] + dat[0][3] * mat.dat[3][1]) % P;
        res.dat[0][2] = (dat[0][0] * mat.dat[0][2] + dat[0][1] * mat.dat[1][2] + dat[0][2] * mat.dat[2][2] + dat[0][3] * mat.dat[3][2]) % P;
        res.dat[0][3] = (dat[0][0] * mat.dat[0][3] + dat[0][1] * mat.dat[1][3] + dat[0][2] * mat.dat[2][3] + dat[0][3] * mat.dat[3][3]) % P;
        res.dat[1][0] = (dat[1][0] * mat.dat[0][0] + dat[1][1] * mat.dat[1][0] + dat[1][2] * mat.dat[2][0] + dat[1][3] * mat.dat[3][0]) % P;
        res.dat[1][1] = (dat[1][0] * mat.dat[0][1] + dat[1][1] * mat.dat[1][1] + dat[1][2] * mat.dat[2][1] + dat[1][3] * mat.dat[3][1]) % P;
        res.dat[1][2] = (dat[1][0] * mat.dat[0][2] + dat[1][1] * mat.dat[1][2] + dat[1][2] * mat.dat[2][2] + dat[1][3] * mat.dat[3][2]) % P;
        res.dat[1][3] = (dat[1][0] * mat.dat[0][3] + dat[1][1] * mat.dat[1][3] + dat[1][2] * mat.dat[2][3] + dat[1][3] * mat.dat[3][3]) % P;
        res.dat[2][0] = (dat[2][0] * mat.dat[0][0] + dat[2][1] * mat.dat[1][0] + dat[2][2] * mat.dat[2][0] + dat[2][3] * mat.dat[3][0]) % P;
        res.dat[2][1] = (dat[2][0] * mat.dat[0][1] + dat[2][1] * mat.dat[1][1] + dat[2][2] * mat.dat[2][1] + dat[2][3] * mat.dat[3][1]) % P;
        res.dat[2][2] = (dat[2][0] * mat.dat[0][2] + dat[2][1] * mat.dat[1][2] + dat[2][2] * mat.dat[2][2] + dat[2][3] * mat.dat[3][2]) % P;
        res.dat[2][3] = (dat[2][0] * mat.dat[0][3] + dat[2][1] * mat.dat[1][3] + dat[2][2] * mat.dat[2][3] + dat[2][3] * mat.dat[3][3]) % P;
        res.dat[3][0] = (dat[3][0] * mat.dat[0][0] + dat[3][1] * mat.dat[1][0] + dat[3][2] * mat.dat[2][0] + dat[3][3] * mat.dat[3][0]) % P;
        res.dat[3][1] = (dat[3][0] * mat.dat[0][1] + dat[3][1] * mat.dat[1][1] + dat[3][2] * mat.dat[2][1] + dat[3][3] * mat.dat[3][1]) % P;
        res.dat[3][2] = (dat[3][0] * mat.dat[0][2] + dat[3][1] * mat.dat[1][2] + dat[3][2] * mat.dat[2][2] + dat[3][3] * mat.dat[3][2]) % P;
        res.dat[3][3] = (dat[3][0] * mat.dat[0][3] + dat[3][1] * mat.dat[1][3] + dat[3][2] * mat.dat[2][3] + dat[3][3] * mat.dat[3][3]) % P;
        return res;
    }
};

struct Vec {
    int dat[4];

    Vec() {
        memset(dat, 0, sizeof dat);
    }

    Vec(initializer_list<int> lst) {
        assert(lst.size() == 4);
        int i = 0;
        for (int v: lst) dat[i++] = v;
    }

    Vec operator+(const Vec& v) const {
        Vec res;
        for (int i = 0; i < 4; i++) res.dat[i] = (dat[i] + v.dat[i]) % P;
        return res;
    }
    
    Vec operator*(const Matrix& mat) const {
        return {
            (dat[0] * mat.dat[0][0] + dat[1] * mat.dat[1][0] + dat[2] * mat.dat[2][0] + dat[3] * mat.dat[3][0]) % P,
            (dat[0] * mat.dat[0][1] + dat[1] * mat.dat[1][1] + dat[2] * mat.dat[2][1] + dat[3] * mat.dat[3][1]) % P,
            (dat[0] * mat.dat[0][2] + dat[1] * mat.dat[1][2] + dat[2] * mat.dat[2][2] + dat[3] * mat.dat[3][2]) % P,
            (dat[0] * mat.dat[0][3] + dat[1] * mat.dat[1][3] + dat[2] * mat.dat[2][3] + dat[3] * mat.dat[3][3]) % P,
        };
    }
};

Vec sum[N<<2];
Matrix mul[N<<2];

void update(int u, const Matrix& mat) {
    mul[u] = mul[u] * mat;
    sum[u] = sum[u] * mat;
}

void pushdown(int u) {
    update(u<<1, mul[u]);
    update(u<<1|1, mul[u]);
    mul[u] = {
        1, 0, 0, 0,
        0, 1, 0, 0,
        0, 0, 1, 0,
        0, 0, 0, 1,
    };
}

void pushup(int u) {
    sum[u] = sum[u<<1] + sum[u<<1|1];
}

void build(int u, int L, int R) {
    mul[u] = {
        1, 0, 0, 0,
        0, 1, 0, 0,
        0, 0, 1, 0,
        0, 0, 0, 1,
    };

    if (L == R) return sum[u] = {A[L], B[L], C[L], 1}, void();
    build(u<<1, L, Mid), build(u<<1|1, Mid+1, R);
    pushup(u);
}

Vec query(int u, int l, int r, int L, int R) {
    if (l <= L && R <= r) return sum[u];
    pushdown(u);
    Vec res;
    if (l <= Mid) res = query(u<<1, l, r, L, Mid);
    if (Mid+1 <= r) res = res + query(u<<1|1, l, r, Mid+1, R);
    return res;
}

void modify(int u, int l, int r, const Matrix& mat, int L, int R) {
    if (l <= L && R <= r) return update(u, mat);
    pushdown(u);
    if (l <= Mid) modify(u<<1, l, r, mat, L, Mid);
    if (Mid+1 <= r) modify(u<<1|1, l, r, mat, Mid+1, R);
    pushup(u);
}

signed main() {
    read(n);
    for (int i = 1; i <= n; i++) read(A[i]), read(B[i]), read(C[i]);
    build(1, 1, n);
    read(m);
    for (int i = 1, opt, l, r, v; i <= m; i++) {
        read(opt), read(l), read(r);
        if (opt == 1) modify(1, l, r, {
            1, 0, 0, 0,
            1, 1, 0, 0,
            0, 0, 1, 0,
            0, 0, 0, 1,
        }, 1, n);
        else if (opt == 2) modify(1, l, r, {
            1, 0, 0, 0,
            0, 1, 0, 0,
            0, 1, 1, 0,
            0, 0, 0, 1,
        }, 1, n);
        else if (opt == 3) modify(1, l, r, {
            1, 0, 1, 0,
            0, 1, 0, 0,
            0, 0, 1, 0,
            0, 0, 0, 1,
        }, 1, n);
        else if (opt == 4) read(v), modify(1, l, r, {
            1, 0, 0, 0,
            0, 1, 0, 0,
            0, 0, 1, 0,
            v, 0, 0, 1,
        }, 1, n);
        else if (opt == 5) read(v), modify(1, l, r, {
            1, 0, 0, 0,
            0, v, 0, 0,
            0, 0, 1, 0,
            0, 0, 0, 1,
        }, 1, n);
        else if (opt == 6) read(v), modify(1, l, r, {
            1, 0, 0, 0,
            0, 1, 0, 0,
            0, 0, 0, 0,
            0, 0, v, 1,
        }, 1, n);
        else {
            Vec v = query(1, l, r, 1, n);
            printf("%lld %lld %lld\n", v.dat[0], v.dat[1], v.dat[2]);
        }
    }

    return 0;
}
```

## [ABC305G] Banned Substrings

> 给定 $M$ 个长度不超过 $6$ 的仅由 $\texttt{a,b}$ 构成的非空字符串集合 $S=\{s_1,\dots,s_M\}$，求多少个字符串 $T$ 满足：
>
> + $|T|=N$
> + $\forall s_i\in S,s_i\not\subset T$
>
> 答案对 $998244353$  取模，其中 $1\le N\le 10^{18},1\le M\le 126$​。
>
> 题目链接：[ABC305G](https://atcoder.jp/contests/abc305/tasks/abc305_g)。

首先看作 $01$ 串，设 $dp_{i,s}$ 表示长度为 $i$ 且后缀表示为二进制数 $s$ 的字串数量，其中 $s$ 包含末尾 $5$ 个位置，于是 $s$ 的范围是 $0\sim 2^5-1$ 即 $0\sim 31$。

枚举所有 $s$，并且枚举下一位填 $0/1$，检查是否存在 $s_i\subset (s+0/1)$，这样可以构造出转移矩阵。

对于 $i\le 5$ 时，暴力求解即可。对于 $i>6$​，通过矩阵快速幂解决，下面 $s\to s'$ 表示 $s\to s'$ 的转移是否合法。
$$
\begin{bmatrix}
0\to 0&1\to 0&2\to 0&\cdots&31\to 0\\
0\to 1&1\to 1&2\to 1&\cdots&31\to 1\\
\vdots&\vdots&\vdots&\ddots&\vdots\\
0\to 31&1\to 31&2\to 31&\cdots&31\to 31
\end{bmatrix}
\begin{bmatrix}
dp_{i,0}\\
dp_{i,1}\\
dp_{i,2}\\
\vdots\\
dp_{i,31}
\end{bmatrix}
=
\begin{bmatrix}
dp_{i+1,0}\\
dp_{i+1,1}\\
dp_{i+1,2}\\
\vdots\\
dp_{i+1,31}
\end{bmatrix}
$$

具体地，对于 $s\to s'$ 的转移，需满足 $s,s'$ 均合法，且 $s$ 的后四位是 $s'$ 的前四位，即 $s'=(2s+0/1) \bmod 32$，且对应的 $2s+0/1$ 也应合法。注意到只要 $2s+0/1$ 合法那么 $s,s'$ 都是其子串，也一定合法。

初始状态是 $dp_{5,s}$ 进行矩阵快速幂后对结果向量进行求和就是最终答案。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 32, M = 130, P = 998244353;
int n, m;
string s[M];
bool valid[N<<1];

struct Matrix {
    int dat[N][N];
    Matrix() { memset(dat, 0, sizeof dat); }
    Matrix operator*(const Matrix& mat) const {
        Matrix res;
        for (int i = 0; i < N; i++)
            for (int k = 0; k < N; k++)
                for (int j = 0; j < N; j++)
                    res.dat[i][j] = (res.dat[i][j] + dat[i][k] * mat.dat[k][j]) % P;
        return res;
    }
    Matrix qmi(int k) const {
        Matrix a = *this, res;
        for (int i = 0; i < N; i++) res.dat[i][i] = 1;
        while (k) {
            if (k & 1) res = res * a;
            a = a * a;
            k >>= 1;
        }
        return res;
    }
} A;


bool chk(int a, const string& b, int sz) {
    string k;
    for (int i = 0; i < sz; i++) {
        k += 'a' + (a >> i & 1);
    }
    for (int i = 0; i < sz; i++)
        if (k.substr(i, b.size()) == b)
            return false;
    return true;
}

bool chk(int a, int sz) {
    for (int i = 1; i <= m; i++) if (!chk(a, s[i], sz)) return false;
    return true;
}

signed main() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) cin >> s[i];
    for (int i = 0; i < N; i++) {
        int x = (i & (0b1111)) << 1;
        A.dat[x][i] = chk(i << 1, 6);
        A.dat[x | 1][i] = chk(i << 1 | 1, 6);
    }
    
    if (n <= 5) {
        int cnt = 0;
        for (int i = 0; i < (1 << n); i++)
            if (chk(i, n)) cnt++;
        cout << cnt << endl;
        return 0;
    }

    Matrix K = A.qmi(n-5);
    int res = 0;
    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++)
            res = (res + K.dat[i][j] * chk(j, 5)) % P;
    cout << res << endl;
    return 0;
}
```

