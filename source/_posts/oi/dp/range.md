---
date: 2023-08-12 17:30:00
title: 动态规划 - 区间 DP
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 石子合并

> 设有 N 堆石子排成一排，其编号为 1,2,3,…,N
>
> 每堆石子有一定的质量，可以用一个整数来描述，现在要将这 N 堆石子合并成为一堆。
>
> 每次只能合并相邻的两堆，合并的代价为这两堆石子的质量之和，合并后与这两堆石子相邻的石子将和新堆相邻，合并时由于选择的顺序不同，合并的总代价也不相同。
>
> 找出一种合理的方法，使总的代价最小，输出最小代价。
>
> 题目链接：[AcWing 282](https://www.acwing.com/problem/content/284/)。

+ 状态表示 $f(L,R)$：
    + 表示：区间 $[L,R]$ 中的所有方案的集合。
    + 存储：最小代价。
+ 状态转移：$f(L,R)=\min\{f(L,K)+f(K+1,R)+\sum_{i=L}^R w_i\}$
+ 目标状态：$f(1,n)$
+ 初始化：$f(x,x)=0,f(x,y)=\inf,x\ne y$

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 310;
int n, f[N][N], s[N];

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        int v; scanf("%d", &v);
        s[i] = s[i-1] + v;
    }
    for (int len = 2; len <= n; len++) {
        for (int l = 1; l+len-1 <= n; l++) {
            int r = l+len-1;
            f[l][r] = 1e9;
            for (int k = l; k <= r-1; k++) {
                f[l][r] = min(f[l][r], f[l][k] + f[k+1][r] + s[r] - s[l-1]);
            }
        }
    }
    printf("%d\n", f[1][n]);
    return 0;
}
```

## [NOI1995] 环形石子合并

> 在一个圆形操场的四周摆放 $N$ 堆石子，现要将石子有次序地合并成一堆，规定每次只能选相邻的 $2$ 堆合并成新的一堆，并将新的一堆的石子数，记为该次合并的得分。
>
> 试设计出一个算法,计算出将 $N$ 堆石子合并成 $1$ 堆的最小得分和最大得分。
>
> **输入格式**
>
> 数据的第 $1$ 行是正整数 $N$，表示有 $N$ 堆石子。
>
> 第 $2$ 行有 $N$ 个整数，第 $i$ 个整数 $a_i$ 表示第 $i$ 堆石子的个数。
>
> **输出格式**
>
> 输出共 $2$ 行，第 $1$ 行为最小得分，第 $2$ 行为最大得分。
>
> **数据范围**
>
> $1\leq N\leq 100$，$0\leq a_i\leq 20$。
>
> 题目链接：[P1880](https://www.luogu.com.cn/problem/P1880)。

和上题做法基本一样，这里把长度为 n 的数列复制一遍为 2n 的数列，然后枚举长度为 n 的区间，就可以把一个环拆成链了。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 210;
int f[N][N], g[N][N], s[N], n;

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &s[i]), s[i+n] = s[i];
    for (int i = 1; i <= 2*n; i++) s[i] += s[i-1];

    for (int len = 2; len <= n; len++) {
        for (int l = 1, r; (r = l+len-1) <= 2*n; l++) {
            f[l][r] = 0x3f3f3f3f;
            for (int k = l; k <= r-1; k++) {
                f[l][r] = min(f[l][r], f[l][k] + f[k+1][r] + s[r] - s[l-1]);
                g[l][r] = max(g[l][r], g[l][k] + g[k+1][r] + s[r] - s[l-1]);
            }
        }
    }

    int res1 = 0x3f3f3f3f, res2 = 0;
    for (int i = 1; i <= n; i++) {
        res1 = min(res1, f[i][i+n-1]);
        res2 = max(res2, g[i][i+n-1]);
    }
    return !printf("%d\n%d\n", res1, res2);
}
```

## 凸多边形划分

> 给定一个具有 N 个顶点的凸多边形，将顶点从 1 至 N 标号，每个顶点的权值都是一个小于 `1e9` 的正整数。
>
> 将这个凸多边形划分成 N−2 个互不相交的三角形，对于每个三角形，其三个顶点的权值相乘都可得到一个权值乘积，试求所有三角形的顶点权值乘积之和至少为多少。
>
> 题目链接：[AcWing 1069](https://www.acwing.com/problem/content/1071/)。

由于这些三角形互不相交，那么可以用区间 DP 解，将 $[L,R]$ 区间以点 $K$ 划分成三个子区间。

+ 状态表示 $f(L,R)$：
    + 含义：顶点 $L\sim R$ 中所有划分方式的集合。
    + 存储：权值最小值。
+ 状态转移：$\forall K\in (L,R), f(L,R)=\min\{f(L,K)+f(K,R)+a_La_Ka_R\}$
+ 初始化：默认为 0，然后当枚举到 $L,R$ 时先令 $f(L,R)=\inf$
+ 答案：$\forall L\in[1,N],\min\{f(L,L+N-1)\}$

答案较大，使用 `__int128` 存储，这个要手写输出。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

typedef __int128 LL;
const int N = 110;
int n;
LL f[N][N];
int a[N];

void _write(LL n) {
    if (n < 0) putchar('-'), n = -n;
    if (!n) return;
    _write(n/10);
    putchar('0' + (n%10));
}

void write(LL n) {
    if (!n) putchar('0');
    else _write(n);
    puts("");
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &a[i]);
        a[i+n] = a[i];
    }
    
    for (int len = 3; len <= n; len++) {
        for (int l = 1, r; (r = l+len-1) <= 2*n; l++) {
            f[l][r] = 1e35;
            for (int k = l+1; k <= r-1; k++) {
                f[l][r] = min(f[l][r], f[l][k] + f[k][r] + (LL)a[l]*a[k]*a[r]);
            }
        }
    }
    
    LL res = 1e35;
    for (int i = 1; i <= n; i++)
        res = min(res, f[i][i+n-1]);
    write(res);
    return 0;
}
```

## 括号匹配


> 对于括号串 $S$，记 $|S|$ 为 $S$ 的长度，记 $f(S)$ 为其 $2^{|S|}$ 个子序列中合法括号序列的个数，求：
> $$
> (\sum_{l=1}^{|S|}\sum_{r=l}^{|S|}(l\oplus r\oplus f(S[l,r])))\bmod998244353
> $$
> 
> 数据范围：$1\le |S|\le 500$

如果不管异或，那么这是个区间 DP，用 $\texttt{L},\texttt{R}$ 表示左括号和右括号。

$$
f(l,r)=\begin{cases}
f(l+1,r)\quad S_{l}=\texttt{R}\\
f(l+1,r)+\sum_{S_k=\texttt{R}} f(l+1,k-1)f(k+1,r)\quad S_{l}=\texttt{L}
\end{cases}
$$

考虑到异或的性质，由于 $l,r$ 都是较小的数，所以取一个足够大的 $2$ 的幂作为模数，低位就足够 $l,r$ 异或了。

因此我们可以得到等式：
$$
x\oplus z=x-y+(y\oplus z)
$$
其中 $y$ 是 $x$ 模一个较大的 $2$ 的整次幂，所以我们用两个模数分别求 $f(l,r)$ 即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int P = 998244353, Q = 1 << 20, N = 510;
int f[N][N], g[N][N], n;
char s[N];

signed main() {
    cin >> (s+1);
    n = strlen(s+1);

    for (int i = 1; i <= n+5; i++)
        for (int j = 1; j <= n+5; j++)
            f[i][j] = g[i][j] = 1;

    for (int len = 2; len <= n; len++) {
        for (int l = 1, r; (r = l+len-1) <= n; l++) {
            f[l][r] = f[l+1][r], g[l][r] = g[l+1][r];
            if (s[l] != '(') continue;
            for (int k = l+1; k <= r; k++) {
                if (s[k] == ')') {
                    f[l][r] = (f[l][r] + f[l+1][k-1] * f[k+1][r]) % P;
                    g[l][r] = (g[l][r] + g[l+1][k-1] * g[k+1][r]) % Q;
                }
            }
        }
    }

    int res = 0;
    for (int l = 1; l <= n; l++)
        for (int r = l; r <= n; r++)
            res = (res + f[l][r] - g[l][r] + P + (g[l][r] ^ l ^ r)) % P;
    cout << res << '\n';

    return 0;
}
```

## [NOIP2003] 加分二叉树

> 设一个 n 个节点的二叉树 tree 的中序遍历为 $(1,2,3,\cdots,n)$，其中数字 $1,2,3,\cdots,n$ 为节点编号。每个节点都有一个分数（均为正整数），记第 i 个节点的分数为 $d_i$，tree 及它的每个子树都有一个加分，任一棵子树 subtree（也包含 tree 本身）的加分计算方法如下：
>
> 记 subtree 的左子树加分为 l，右子树加分为 r，subtree 的根的分数为 a，则 subtree 的加分为：
> $$
> l\times r+a
> $$
> 若某个子树为空，规定其加分为 1，叶子的加分就是叶节点本身的分数。不考虑它的空子树。
>
> 试求一棵符合中序遍历为 $(1,2,3,\cdots,n)$ 且加分最高的二叉树 tree。
>
> 要求输出：
>
> 1. tree 的最高加分；
> 2. tree 字典序最小的前序遍历。
>
> 题目链接：[AcWing 479](https://www.acwing.com/problem/content/481/)，[LOJ 10158](https://loj.ac/p/10158)。

题目本身不怎么难，还是区间 DP 的思路。

+ 状态表示 $f(L,R),g(L,R)$：

    + 含义：$f(L,R)$ 表示中序遍历编号为 $L\sim R$ 中的所有树，$g(L,R)$ 表示这些树的根结点。
    + 存储：$f(L,R)$ 表示最大加分，$g(L,R)$ 表示与之对应的根结点。

+ 状态转移方程：
    $$
    \forall K\in[L,R],f(L,R)=\max\left\{\begin{aligned}
    &f(L,K-1)\times f(K+1,R)+w_K,K\ne L,K\ne R\\
    &f(K+1,R)+w_K,K=L,K\ne R\\
    &f(L,K-1)+w_K,K\ne L,K=R\\
    &w_K,K=L,K=R
    \end{aligned}\right.
    $$
    从左向右遍历 $K$，当状态发生变化就令 $g(L,R)=K$，这样就能保证是最小字典序。

+ 初始化：全是 0 即可。

+ 答案：$f(1,N)$ 与递归输出 $g$ 里面的结点。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 35;
int n, f[N][N], a[N];
int g[N][N];

void dfs(int l, int r) {
    if (l > r) return;
    if (!g[l][r]) return;
    
    printf("%d ", g[l][r]);
    dfs(l, g[l][r]-1);
    dfs(g[l][r]+1, r);
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++)
        scanf("%d", &a[i]);
    
    for (int len = 1; len <= n; len++) {
        for (int l = 1, r; (r = l+len-1) <= n; l++) {
            for (int k = l; k <= r; k++) {
                int now;
                if (k != l && k != r)
                    now = f[l][k-1]*f[k+1][r] + a[k];
                else if (k == l && k != r)
                    now = f[k+1][r] + a[k];
                else if (k != l && k == r)
                    now = f[l][k-1] + a[k];
                else
                    now = a[k];
                
                if (now > f[l][r]) {
                    f[l][r] = now;
                    g[l][r] = k;
                }
            }
        }
    }
    
    printf("%d\n", f[1][n]);
    dfs(1, n);
    
    return 0;
}
```

## [NOI1999] 棋盘分割

> 将一个 $8\times 8$ 的棋盘进行如下分割：将原棋盘割下一块矩形棋盘并使剩下部分也是矩形，再将剩下的部分继续如此分割，这样割了 (n−1) 次后，连同最后剩下的矩形棋盘共有 n 块矩形棋盘。(每次切割都只能沿着棋盘格子的边进行)。
>
> 原棋盘上每一格有一个分值，一块矩形棋盘的总分为其所含各格分值之和。
>
> 现在需要把棋盘按上述规则分割成 n 块矩形棋盘，并使各矩形棋盘总分的均方差最小。
>
> 均方差 $\sigma = \sqrt{[\sum_{i=1}^n(x_i-\overline x)^2]/n}$，平均数 $\overline x=(\sum_{i=1}^nx_i)/n$，其中 $x_i$ 为第 $i$ 块棋盘上的权值之和。
>
> 求出均方差最小值。
>
> 题目链接：[AcWing 321](https://www.acwing.com/problem/content/323/)。

+ 状态表示 $f(x_1,y_1,x_2,y_2,k)$：子矩阵 $(x_1,y_1)\to (x_2,y_2)$ 中切了 $k$ 块的最小 $\sum_{i=1}^n(x_i-\overline x)^2$，不做除法和开根运算。
+ 状态转移：枚举出所有横切和纵切的方案，然后每次保留一边，另一边用前缀和计算出来均方差加上，公式比较难写，直接看代码。
+ 答案：$\sqrt{f(1,1,8,8,n)/n}$
+ 初始化：所有都初始化成 $-1$，我一开始初始化成 $\inf$ 结果 TLE，下文解释。

```cpp
#include <cstdio>
#include <cmath>
#include <algorithm>
using namespace std;

// cmath 里面有个 y1
#define y1 yy1
const int N = 16, M = 9;
const double INF = 1e20;
int s[M][M], n;
double f[M][M][M][M][N], avg;

double get(int x1, int y1, int x2, int y2) {
    double sum = s[x2][y2] + s[x1-1][y1-1] - s[x1-1][y2] - s[x2][y1-1] - avg;
    return sum*sum;
}

double dp(int x1, int y1, int x2, int y2, int k) {
    double& v = f[x1][y1][x2][y2][k];
    if (v >= 0) return v;
    
    if (k == 1) return v = get(x1, y1, x2, y2);
    
    // 横切
    v = INF;
    for (int i = x1; i <= x2-1; i++) {
        v = min(v, dp(x1, y1, i, y2, k-1) + get(i+1, y1, x2, y2));
        v = min(v, dp(i+1, y1, x2, y2, k-1) + get(x1, y1, i, y2));
    }

    // 纵切
    for (int i = y1; i <= y2-1; i++) {
        v = min(v, dp(x1, y1, x2, i, k-1) + get(x1, i+1, x2, y2));
        v = min(v, dp(x1, i+1, x2, y2, k-1) + get(x1, y1, x2, i));
    }
    
    return v;
}


int main() {
    scanf("%d", &n);
    
    for (int i = 1; i <= 8; i++)
        for (int j = 1; j <= 8; j++)
            for (int k = 1; k <= 8; k++)
                for (int p = 1; p <= 8; p++)
                    for (int q = 1; q <= n; q++)
                        f[i][j][k][p][q] = -1;

    for (int i = 1; i <= 8; i++)
        for (int j = 1; j <= 8; j++) {
            scanf("%d", &s[i][j]);
            s[i][j] += s[i-1][j] + s[i][j-1] - s[i-1][j-1];
        }

    avg = (double)s[8][8] / n;
    
    printf("%.3lf\n", sqrt(dp(1, 1, 8, 8, n) / n));
    return 0;
}
```

这里为什么不可以初始化成 $\inf$，其原因是如果一个状态不合法，不论是初始化成什么，它都应该被计算成 $\inf$，那么如果初始化就是 $\inf$，就无法判断这个状态是没计算过还是计算过了但是不合法，就失去了记忆化搜索的**记忆**性质，所以会 TLE。

## [NOIP2007] 矩阵取数游戏

> 对于一个给定的 $n \times m$ 的矩阵，矩阵中的每个元素 $a_{i,j}$ 均为非负整数。游戏规则如下：
>
> 1. 每次取数时须从每行各取走一个元素，共 $n$ 个。经过 $m$ 次后取完矩阵内所有元素；
> 2. 每次取走的各个元素只能是该元素所在行的行首或行尾；
> 3. 每次取数都有一个得分值，为每行取数的得分之和，每行取数的得分 = 被取走的元素值 $\times 2^i$，其中 $i$ 表示第 $i$ 次取数（从 $1$ 开始编号）；
> 4. 游戏结束总得分为 $m$ 次取数得分之和。
>
> 求出取数后的最大得分。
>
> 对于 $100\%$ 的数据，满足 $1\le n,m\le 80$，$0\le a_{i,j}\le1000$。
>
> 题目链接：[P1005](https://www.luogu.com.cn/problem/P1005)，[AcWing 492](https://www.acwing.com/problem/content/494/)。

如果不用区间 DP 很难写，比如 $f(i,0/1)$ 表示第 $i$ 次从前或后取，那如何转移也是一个难题。

+ 状态表示 $f(l,r)$ 从 $[l,r]$ 的最大分值。
+ 状态转移 $f(l,r)=\max\{f(l+1,r)+2^{m-len+1}\times a_l,f(l,r-1)+2^{m-len+1}\times a_r\}$

根据范围可以算出来需要 $80\times80\times1000\times2^{80}<2^{127}$，可以用 `__int128`。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

typedef __int128 LL;
const int N = 90;
int n, m, a[N];
LL f[N][N], pow2[N];

void write(LL t) {
    if (t < 0) putchar('-'), t = -t;
    if (t < 9) return putchar('0' + t), void();
    write(t / 10);
    putchar('0' + t % 10);
}

LL qmi(LL a, LL k) {
    LL res = 1;
    while (k) {
        if (k & 1) res *= a;
        a *= a;
        k >>= 1;
    }
    return res;
}

LL solveline() {
    for (int i = 1; i <= m; i++) scanf("%d", &a[i]);
    
    for (int len = 1; len <= m; len++) {
        for (int l = 1, r; (r = l+len-1) <= m; l++) {
            if (len == 1) f[l][r] = (LL)pow2[m-len+1] * a[l];
            else f[l][r] = max(f[l+1][r] + (LL)a[l] * pow2[m-len+1], f[l][r-1] + (LL)a[r] * pow2[m-len+1]);
        }
    }
    return f[1][m];
}

int main() {
    scanf("%d%d", &n, &m);
    // 打个表先
    for (int i = 1; i <= m; i++) pow2[i] = qmi(2, i);
    
    LL res = 0;
    for (int i = 1; i <= n; i++) res += solveline();
    write(res);
    return 0;
}
```

## [CSP-S2021] 括号序列

> 定义“超级括号序列”是由字符 `(`、`)`、`*` 组成的字符串，并且对于某个给定的常数 $k$
>
> 1. `()`、`(S)` 均是符合规范的超级括号序列，其中 `S` 表示任意一个仅由**不超过** $k$ **个**字符 `*` 组成的非空字符串（以下两条规则中的 `S` 均为此含义）；
> 2. 如果字符串 `A` 和 `B` 均为符合规范的超级括号序列，那么字符串 `AB`、`ASB` 均为符合规范的超级括号序列，其中 `AB` 表示把字符串 `A` 和字符串 `B` 拼接在一起形成的字符串；
> 3. 如果字符串 `A` 为符合规范的超级括号序列，那么字符串 `(A)`、`(SA)`、`(AS)` 均为符合规范的超级括号序列。
> 4. 所有符合规范的超级括号序列均可通过上述 3 条规则得到。
>
> 例如，若 $k = 3$，则字符串 `((**()*(*))*)(***)` 是符合规范的超级括号序列，但字符串 `*()`、`(*()*)`、`((**))*)`、`(****(*))` 均不是。特别地，空字符串也不被视为符合规范的超级括号序列。
>
> 现在给出一个长度为 $n$ 的超级括号序列，其中有一些位置的字符已经确定，另外一些位置的字符尚未确定（用 `?` 表示）。小 w 希望能计算出：有多少种将所有尚未确定的字符一一确定的方法，使得得到的字符串是一个符合规范的超级括号序列？
>
> **输入格式**
>
> 第一行，两个正整数 $n, k$。
>
> 第二行，一个长度为 $n$ 且仅由 `(`、`)`、`*`、`?` 构成的字符串 $S$。
>
> **输出格式**
>
> 输出一个非负整数表示答案对 ${10}^9 + 7$ 取模的结果。
>
> 对于 $100 \%$ 的数据，$1 \le k \le n \le 500$。

可以用区间 DP 和状态机模型解决，看到上面对应的语法规则我们可以定义出下列状态，DP 储存的值是方案数：

+ 状态 `0` 表示 $(),(S),(AB),(SA),(AS)$，即所有合法的单元。
+ 状态 `1` 表示 $(\cdots)\cdots(\cdots)$，即两个并列的合法单元。
+ 状态 `2` 表示 $S$，即星号。
+ 状态 `3` 表示 $SA$，即合法的状态前面加上星号。
+ 状态 `4` 表示 $AS$，即合法的状态后面加上星号。

接下来关心如何转移。

+ 状态 `0`，只要满足 `a[L]='(', a[R]=')'` 就可以转移。
    $$
    f(L,R,0)=\sum_{i=0}^4 f(L+1,R-1,i)
    $$

+ 状态 `1`，枚举一个分界线，然后通过别的状态转移。
    $$
    f(L,R,1)=\sum_{j=L}^{R-1} [f(L,j,0)\times f(j+1,R,0/1/3)]
    $$
    但是这里要特判一下当长度为 2 时，$f(L,L+1)=1$。

+ 状态 `2`，当 `a[R]='*'` 时可以从自身转移，同时注意如果大于 $k$ 是不合法的。
    $$
    f(L,R,2)=\begin{cases}
    f(L,R-1,2)\quad len\le m\\
    0\quad len > m
    \end{cases}
    $$

+ 状态 `3` 和状态 `4`，同样是枚举分界线。
    $$
    f(L,R,3)=\sum_{j=L}^{R-1}[f(L,j,2)\times f(j+1,R,0/1)]\\
    f(L,R,4)=\sum_{j=L}^{R-1}[f(L,j,0/1)\times f(j+1,R,2)]
    $$

+ 最终答案就是 $f(1,n,0)+f(1,n,1)$

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 510, P = 1e9+7;
int n, m, f[N][N][5];
char s[N];

bool eq(char a, char b) {
    return a == b || a == '?' || b == '?';
}

int main() {
    scanf("%d%d%s", &n, &m, s+1);
    
    for (int len = 1; len <= n; len++) {
        for (int l = 1, r; (r = l+len-1) <= n; l++) {
            if (len == 1) {
                if (eq(s[l], '*')) f[l][r][2] = 1;
                continue;
            }
            
            if (len <= m && eq(s[r], '*')) f[l][r][2] = f[l][r-1][2];
            
            if (eq(s[l], '(') && eq(s[r], ')')) {
                if (len == 2) f[l][r][0] = 1;
                for (int k = 0; k <= 4; k++) f[l][r][0] = (f[l][r][0] + f[l+1][r-1][k]) % P;
            }
            
            for (int j = l; j < r; j++) {
                f[l][r][1] = (f[l][r][1] + f[l][j][0] * (f[j+1][r][0] + (LL)f[j+1][r][1] + f[j+1][r][3])) % P;
                f[l][r][3] = (f[l][r][3] + f[l][j][2] * (LL)(f[j+1][r][0] + f[j+1][r][1])) % P;
                f[l][r][4] = (f[l][r][4] + (LL)(f[l][j][0] + f[l][j][1]) * f[j+1][r][2]) % P;
            }
        }
    }
    printf("%d\n", (f[1][n][0] + f[1][n][1]) % P);
    return 0;
}
```

## [SDCPC2024 M] 回文多边形

> 按逆时针给出 $n$ 个点构成一个凸多边形，每个点有一个权值 $v_i$。问权值构成回文序列的子点集中构成的子多边形面积最大是多少，输出这个值 $\times 2$，可以证明这一定是一个整数。
>
> 数据范围：$3\le n\le 500,|x_i|,1\le v_i\le 10^9,|x_i|,|y_i|\le 10^9$​。

答案与权值无关，首先离散化。

设 $f(l,r)$ 代表 $l,r$ 逆时针组成的凸包的最大面积，满足 $v_l=v_r$，那么有这样一个转移方程：
$$
f(l,r)=\max_{l<l_1\le r_1<r}^{v_{l_1}=v_{r_1}}\{f(l_1,r_1)+S_{ll_1r_1}+S_{lr_1r}\}
$$
直接枚举是 $O(n^4)$，但是如果 $l_1,r_1$ 在区间 $(l,r)$ 内都不是最靠两边的同等权值的点，那么选上两边的一定更优。所以两个点一定有一个是最靠边上的，如果用二分确定这个点，那么复杂度就是 $O(n^3\log n)$​，可以通过。

由于是环状的，所以需要将序列复制一遍接到尾部，保证能取到答案。

面积用向量外积算一下就好，最大的面积显然是 $-10^9\sim 10^9$ 乘 $2$ 也不会超过 $8\times 10^{18}$，可以用 `long long`。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define eb emplace_back
const int N = 1010;
int T, n, id[N], x[N], y[N], f[N][N];
vector<int> nums, points[N];

int S(int a, int b, int c) {
    int dx1 = x[a] - x[b], dy1 = y[a] - y[b];
    int dx2 = x[a] - x[c], dy2 = y[a] - y[c];
    return abs(dx1 * dy2 - dx2 * dy1);
}

int getr(int R, int p) {
    int l = 0, r = points[id[p]].size() - 1;
    while (l < r) {
        int mid = (l+r+1) >> 1;
        if (points[id[p]][mid] <= R) l = mid;
        else r = mid - 1;
    }
    return points[id[p]][r];
}

int getl(int L, int p) {
    int l = 0, r = points[id[p]].size() - 1;
    while (l < r) {
        int mid = (l+r) >> 1;
        if (points[id[p]][mid] >= L) r = mid;
        else l = mid + 1;
    }
    return points[id[p]][r];
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    cin >> T;
    while (T--) {
        nums.clear();
        cin >> n;
        for (int i = 1; i <= n; i++) {
            cin >> id[i];
            nums.eb(id[i]);
            points[i].clear();
        }
        sort(nums.begin(), nums.end());
        nums.erase(unique(nums.begin(), nums.end()), nums.end());
        for (int i = 1; i <= n; i++) {
            id[i] = lower_bound(nums.begin(), nums.end(), id[i]) - nums.begin() + 1;
            points[id[i]].eb(i);
        }

        for (int i = 1; i <= n; i++) {
            cin >> x[i] >> y[i];
            x[i+n] = x[i], y[i+n] = y[i], id[i+n] = id[i];
            points[id[i]].eb(i+n);
        }

        int ans = 0;
        for (int len = 3; len <= n; len++) {
            for (int l = 1, r; (r = l + len - 1) <= 2*n; l++) {
                if ((id[l] != id[r])) continue;

                f[l][r] = 0;
                for (int l1 = l+1; l1 < r; l1++) {
                    int r1 = getr(r-1, l1);
                    f[l][r] = max(f[l][r], f[l1][r1] + S(l, l1, r) + S(l1, r1, r));
                }

                for (int r1 = r-1; r1 > l; r1--) {
                    int l1 = getl(l+1, r1);
                    f[l][r] = max(f[l][r], f[l1][r1] + S(l, l1, r) + S(l1, r1, r));
                }
                ans = max(ans, f[l][r]);    
            }
        }

        cout << ans << '\n';
    }
    return 0;
}
```

