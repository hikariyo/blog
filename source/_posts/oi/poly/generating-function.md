---
date: 2024-07-16 20:14:00
title: 多项式 - 生成函数
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
description: 生成函数是用来描述一个有穷或无穷序列的工具，我们只关心它的系数而不关心自变量的具体取值，在一系列计数问题中有很大的作用。
---

## 定义

普通生成函数 OGF 和指数生成函数 EGF 的形式幂级数定义为：
$$
\begin{aligned}
F(x)&=\sum_{i\ge 0} a_ix^i\\
F(x)&=\sum_{i\ge 0}\frac{a_i}{i!}x^i
\end{aligned}
$$
这里形式幂级数的意思是不关心自变量的取值而关心每一项的系数，可以是有穷的，也可以是无穷的。

## 卷积

对于 OGF 的卷积（分别为 $a_i,b_i$ 的 OGF）结果为：
$$
\begin{aligned}
F(x)G(x)&=\left(\sum_{i\ge 0} a_ix^i\right)\left(\sum_{j\ge 0}b_j x^j\right)\\
&=\sum_{i\ge 0}\sum_{j\ge 0} a_ib_jx^{i+j}\\
&=\sum_{k\ge 0} \left(\sum_{i=0}^k a_ib_{k-i}\right)x^k
\end{aligned}
$$
即结果为 $c_k=\sum_{i=0}^k a_ib_{k-i}$​​ 的 OGF。

对于 EGF 的卷积（分别为 $a_i,b_i$ 的 EGF）结果为：
$$
\begin{aligned}
F(x)G(x)&=\left(\sum_{i\ge 0}\frac{a_i}{i!}x^i\right)\left(\sum_{j\ge 0}\frac{b_j}{j!}x^j\right)\\
&=\sum_{k\ge 0}\left(\sum_{i=0}^k\binom{k}{i}a_ib_{k-i}\right)\frac{x^k}{k!}
\end{aligned}
$$
即结果为 $c_k=\sum_{i=0}^k\binom{k}{i}a_ib_{k-i}$ 的 EGF。

## 组合数恒等式

正式开始运用 OGF 前，需要了解一个组合数的恒等式。

由组合意义可知 $\binom{n}{m}=\binom{n-1}{m}+\binom{n-1}{m-1}$，不断地展开后面一项可以得到：
$$
\begin{aligned}
\binom{n}{m}&=\binom{n-1}{m}+\binom{n-2}{m-1}+\cdots+\binom{n-m-1}{0}\\
\binom{n}{m}&=\sum_{i=0}^m\binom{n-i-1}{m-i}
\end{aligned}
$$
一个比较常见的形式是：
$$
\binom{n+k}{n}=\sum_{i=0}^m\binom{n+k-1-i}{k-i}=\sum_{i=0}^m\binom{n+i-1}{i}
$$
这个恒等式用在求证 $\frac{1}{(1-x)^{n+1}}$ 的系数，也可以用广义二项式定理求。

## 封闭形式

下面给出几个常见数列 OGF 的封闭形式：

1. 对于 $a_i=1$ 的 OGF 为：
   $$
   \begin{aligned}
   F(x)&=\sum_{i\ge 0} x^i\\
   F(x)&=xF(x)+1\\
   F(x)&=\frac{1}{1-x}
   \end{aligned}
   $$
   注意这里并不关心级数是否收敛，因为我们关心的是系数。仅仅是认为 $\frac{1}{G(x)}$ 意义是乘以 $G(x)$ 结果为 $1$ 的多项式。

2. 对于 $a_i=i+1$ 的 OGF 为：
   $$
   \begin{aligned}
   F(x)&=\sum_{i\ge 0} (i+1)x^i\\
   &=\sum_{i\ge 1} ix^{i-1}\\
   &=\sum_{i\ge 0} (x^i)'\\
   &=\left(\frac{1}{1-x}\right)'\\
   &=\frac{1}{(1-x)^2}
   \end{aligned}
   $$

3. 对于 $a_i=\binom{n}{i}$ 的 OGF 为：
   $$
   \begin{aligned}
   F(x)&=\sum_{i\ge 0}\binom{n}{i} x^i\\
   &=(1+x)^n
   \end{aligned}
   $$

4. 对于 $a_i=\binom{n+i}{n}$ 的 OGF 为：
   $$
   \begin{aligned}
   F(x)&=\sum_{i\ge 0}\binom{n+i}{n}x^i\\
   &=\frac{1}{(1-x)^{n+1}}
   \end{aligned}
   $$
   用归纳法证明，当 $n=0$ 时 $a_i=1$ 成立，当 $n>0$ 时：
   $$
   \begin{aligned}
   F(x)&=\frac{1}{1-x}\frac{1}{(1-x)^n}\\
   &=\left(\sum_{i\ge 0} x^i\right)\left(\sum_{j\ge 0}\binom{n+j-1}{n-1} x^j\right)\\
   &=\sum_{i\ge 0}\sum_{j\ge 0}\binom{n+j-1}{n-1}x^{i+j}\\
   &=\sum_{k\ge 0}\left(\sum_{j=0}^k\binom{n+j-1}{j}\right) x^k\\
   &=\sum_{k\ge 0}\binom{n+k}{n}x^k
   \end{aligned}
   $$
   得证，也就是说 $[x^i]\frac{1}{(1-x)^{n+1}}=\binom{n+i}{n}$​​。

对于 EGF 常见的没有那么多：

1. 对于 $a_i=1$ 的 EGF 为：
   $$
   F(x)=\sum_{i\ge 0} \frac{x^i}{i!}=e^x
   $$

2. 对于 $a_i=[2\mid i]$ 的 EGF 为：
   $$
   F(x)=\sum_{i\ge 0}\frac{x^{2i}}{(2i)!}=\frac{e^x+e^{-x}}{2}
   $$

3. 对于 $a_i=[2\mid (i+1)]$ 的 EGF 为：
   $$
   F(x)=\sum_{i\ge 0}\frac{x^{2i+1}}{(2i+1)!}=\frac{e^x-e^{-x}}{2}
   $$

利用这些封闭形式对推式子化简帮助很大。

## [BZOJ3028] 食物

> 认为每种食物都是以『个』为单位，只要总数加起来是 $n$ 就算一种方案。对于给出的 $n$，计算出方案数，并对 $10007$ 取模，其中 $1\leq n\leq 10^{500}$，每种食物的限制如下：
> 1. 偶数个
> 2. $0$ 个或 $1$ 个
> 3. $0$ 个，$1$ 个或 $2$ 个
> 4. 奇数个
> 5. $4$ 的倍数个
> 6. $0$ 个，$1$ 个，$2$ 个或 $3$ 个
> 7. 不超过一个
> 8. $3$ 的倍数个
>
> 题目链接：[P10780](https://www.luogu.com.cn/problem/P10780)。

如果能写出所有项的 OGF，由于两个 OGF 的卷积意义为 $[x^n]F(x)G(x)=\sum_{i=0}^n a_ib_{n-i}$，所以只要求出 $[x^n]\prod F_i(x)$ 即可。

1. $\sum_{i\ge 0} x^{2i}=\frac{1}{1-x^2}$
2. $1+x$
3. $1+x+x^2=\frac{1-x^3}{1-x}$
4. $\sum_{i\ge0} x^{2i+1}=\frac{x}{1-x^2}$
5. $\sum_{i\ge 0} x^{4i}=\frac{1}{1-x^4}$​
6. $1+x+x^2+x^3=\frac{1-x^4}{1-x}$
7. $1+x$
8. $\sum_{i\ge 0}x^{3i}=\frac{1}{1-x^3}$

全都乘一起的结果是 $F(x)=\frac{x}{(1-x)^4}$，先前证明过：
$$
F(x)=x\sum_{i\ge 0}\binom{i+3}{3}x^{i}=\sum_{i\ge 1}\binom{i+2}{3} x^i
$$
因此最终的答案是 $[x^n]F(x)=\binom{n+2}{3}=\frac{n(n+1)(n+2)}{6}$。

```python
n = int(input()) % 10007
print(n * (n+1) * (n+2) * 1668 % 10007)
```

## [CEOI2004] Sweets

> 给 $n$ 罐糖，每罐有 $m_i$ 个糖果，每罐中糖果相同，不同罐糖果不同，问吃掉至少 $a$ 至多 $b$ 个的方案数，答案对 $2004$ 取模。
>
> 数据范围：$1\le n\le 10,0\le a\le b\le 10^7,0\le m_i\le 10^6$。
>
> 题目链接：[P6078](https://www.luogu.com.cn/problem/P6078)。

显然地，对于 $i$ 罐中的 OGF 为 $F_i(x)=\sum_{j=0}^{m_i} x^j=\frac{1-x^{m_i+1}}{1-x}$，于是答案的 OGF 是：
$$
\begin{aligned}
F(x)&=\prod_{i=1}^n F_i(x)\\
&=\frac{1}{(1-x)^n}\prod_{i=1}^n(1-x^{m_i+1})\\
&=\left(\sum_{i\ge 0}\binom{n-1+i}{i}x^i\right)\left(\prod_{i=1}^n(1-x^{m_i+1})\right)
\end{aligned}
$$
注意到 $n$ 的范围是 $10$，所以直接暴力求解右边的 OGF $G(x)$，复杂度为 $O(2^n)$；然后枚举 $k$ 为 $[x^k]G(x)$，这样它贡献的答案会是：
$$
[x^k]G(x)\left(\sum_{i=a-k}^{b-k}\binom{n-1+i}{i}\right)=[x^k]G(x)\left(\binom{n+b-k}{n}-\binom{n+a-k-1}{n}\right)
$$
还有一个问题，模数非质数，怎么求组合数？注意到我们需要求的形式是 $\binom{n+x}{n}$，不妨展开看一下：
$$
\binom{n+x}{n}=\frac{(n+x)!}{n!x!}=\frac{(n+x)^{\underline{n}}}{n!}
$$
上面的式子可以 $O(n)$ 算出来，设 $A=(n+x)^{\underline{n}}$，对 $2004n!$ 做带余除法，有 $A=2004n!k_1+r$，注意到一定有 $n!\mid A$，因此 $n!\mid r$，并且：
$$
\begin{aligned}
{A}&\equiv r\pmod{2004n!}\\
\frac{A}{n!}&=2004k_1+\frac{r}{n!}\\
\frac{r}{n!}&\equiv \binom{n+x}{n}\pmod {2004}
\end{aligned}
$$
由于枚举 $k$ 时最多有 $2^n$ 个非零系数，对它们算组合数有一个 $O(n)$ 的复杂度，但是相较于 $O(b)$ 的枚举还是可以忽略的，所以复杂度是 $O(2^n+b)$​。

这里先用 `map` 算，最后将暴力算出来的多项式存在 `int16_t F[N]` 里，为了加速后面的枚举对 `F` 的访问速度。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int N = 1e7+10, P = 2004;

int n, a, b, facn = 1;
int16_t F[N];

int C(int x) {
    int mod = P * facn, r = 1;
    for (int i = 0; i < n; i++) r = r * (n + x - i) % mod;
    return r / facn;
}

void mul(map<int, int>& f, int m) {
    // f *= 1 - x^{m+1}
    map<int, int> f0 = f;
    for (auto& [p, a]: f0) f[p+m+1] = (f[p+m+1] - a) % P;
}

int solve(int k) {
    if (F[k] == 0) return 0;
    if (k >= a) return F[k] * C(b-k) % P;
    return F[k] * (C(b-k) - C(a-k-1)) % P;
}

signed main() {
    map<int, int> f = {{0, 1}};
    cin >> n >> a >> b;
    for (int i = 1, m; i <= n; i++) cin >> m, mul(f, m), facn = facn * i;
    for (const auto& [p, a]: f) F[p] = (a + P) % P;
    int ans = 0;
    for (int k = 0; k <= b; k++) ans = (ans + solve(k)) % P;
    if (ans < 0) ans += P;
    cout << ans << endl;
    return 0;
}
```

## [国家集训队] 整数的 lqp 拆分

> 设 $f_0=0,f_1=1,f_n=f_{n-1}+f_{n-2}$，给定 $n$，求整数 $n$ 的所有拆分权值之和，定义拆分 $\sum_{i} a_i=n$ 的权值为 $\prod f_{a_{i}}$，其中 $1\le n\le 10^{10000}$，答案对 $10^9+7$ 取模。
>
> 题目链接：[P4451](https://www.luogu.com.cn/problem/P4451)。

定义 $g_n$ 为 $n$ 的答案，枚举 $g_{n-i}$ 的答案乘 $f_i$ 就是当前答案。
$$
g_n=\sum_{i=0}^n g_{n-i}f_i
$$
当然，这里需要令 $g_0=1$，满足当新开一个数字时权值为 $f_n$。

设 $f_i,g_i$ 的 OGF 分别为 $F(x),G(x)$，有：
$$
G(x)=F(x)G(x)+1
$$
原因是 $[x^0]F(x)=0$，因此 $[x^0]F(x)G(x)=0$，所以补上 $[x^0]G(x)=g_0=1$。

接下来推导 $F(x)$ 的封闭形式：
$$
\begin{aligned}
F(x)&=\sum_{i\ge 0} f_ix^i\\
xF(x)&=\sum_{i\ge 1} f_{i-1}x^i\\
x^2F(x)&=\sum_{i\ge 2} f_{i-2} x^i\\
(1-x-x^2)F(x)&=f_0x^0+(f_1-f_0)x^1\\
F(x)&=\frac{x}{1-x-x^2}
\end{aligned}
$$
代回原式子可以得到：
$$
\begin{aligned}
G(x)&=\frac{1}{1-F(x)}\\
&=\frac{1-x-x^2}{1-2x-x^2}\\
&=1+\frac{x}{1-2x-x^2}
\end{aligned}
$$
所以现在就是要求 $[x^n]\frac{x}{1-2x-x^2}$，注意到这也是一个类似斐波那契的递推数列的 OGF，它应该满足 $g_0=0,g_1=1,g_n=2g_{n-1}+g_{n-2}$，所以直接写一个矩阵快速幂就好了。

```python
P = int(1e9+7)
n = int(input()) - 1
res = [[1, 0], [0, 1]]
a = [[2, 1], [1, 0]]

def mul(A, B):
    res = [[0, 0], [0, 0]]
    for k in range(2):
        for i in range(2):
            for j in range(2):
                res[i][j] = (res[i][j] + A[i][k] * B[k][j]) % P
    return res

while n:
    if n & 1: res = mul(res, a)
    a = mul(a, a)
    n >>= 1

print(res[0][0])
```

## [集训队作业2013] 城市规划

> 求 $n$ 个点简单有标号无项连通图数量，答案对 $1004535809(479\times 2^{21}+1)$ 取模，其中 $1\le n\le 130000$。
>
> 题目链接：[P4841](https://www.luogu.com.cn/problem/P4841)。

设 $g_n$ 表示 $n$ 个点的无向图数量，那么 $g_n=2^{\binom{n}{2}}$，可以枚举 $1$ 的连通块大小为 $i$，有：
$$
g_n=\sum_{i=1}^n\binom{n-1}{i-1}f_ig_{n-i}
$$
展开，并且两边除以 $(n-1)!$ 有：
$$
\frac{2^{\binom{n}{2}}}{(n-1)!}=\sum_{i=1}^n\frac{f_x}{(i-1)!} \frac{2^{\binom{n-x}{2}}}{(n-x)!}
$$
定义 $f_i,g_i=2^{\binom{i}{2}}$ 的 EGF 为 $F(x),G(x)$，那么有：
$$
[x^{n-1}]G'(x)=\sum_{i=1}^n [x^{i-1}] F'(x)[x^{n-i}]G(x)
$$
即：
$$
G'(x)=F'(x)G(x)
$$
所以要求的 $F(x)$ 为：
$$
F(x)=\int \frac{G'(x)}{G(x)}\mathrm{d}x=\ln G(x)+C
$$
由于 $n\ge 1$，所以常数 $C$ 的取值无所谓。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
constexpr int N = 310000, P = 1004535809;
int fact[N], infact[N];

constexpr int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

struct Poly { /* 省略 */ } G;

signed main() {
    int n;
    cin >> n;
    fact[0] = infact[0] = 1;
    for (int i = 1; i <= n; i++) fact[i] = (fact[i-1] * i) % P;
    infact[n] = qmi(fact[n], P-2);
    for (int i = n-1; i; i--) infact[i] = (infact[i+1] * (i+1)) % P;

    G.resize(n);
    for (int i = 0; i <= n; i++) G[i] = qmi(2, i * (i-1) / 2) * infact[i] % P;
    cout << G.ln(0)[n] * fact[n] % P << endl;
    return 0;
}
```

## 付公主的背包

> 有 $n$ 件商品，每件体积 $v_i$，都有无限件，给定 $m$ 问对于 $s=1\sim m$ 恰好装 $s$ 体积的方案数，答案对 $998244353$ 取模。其中 $1\le n,m\le 10^5,1\le v_i\le m$。
>
> 题目链接：[P4389](https://www.luogu.com.cn/problem/P4389)。

对于每个物品的 OGF 显然为 $F_i(x)=\sum_{j\ge 0} x^{v_ij}=\frac{1}{1-x^{v_i}}$，答案的 OGF 是 $F(x)=\prod_{i=1}^n F_i(x)$，但显然不能进行 $n$ 次 $O(n\log n)$​ 的多项式乘法。

前置知识，$\ln(1+x)$ 的泰勒展开：
$$
\begin{aligned}
\ln(1+x)&=\sum_{i=1}^{+\infin}\frac{(-1)^{i+1}}{i} x^i\\
\ln(1-x)&=\sum_{i=1}^{+\infin}-\frac{1}{i}x^i
\end{aligned}
$$
因此对其进行恒等变换：
$$
\begin{aligned}
F(x)&=\exp(\ln F(x))\\
&=\exp\left(\sum_{i=1}^n \ln F_i(x)\right)\\
&=\exp\left(\sum_{i=1}^n \ln \frac{1}{1-x^{v_i}}\right)\\
&=\exp\left(\sum_{i=1}^n \sum_{j\ge1} \frac{x^{v_ij}}{j}\right)\\
\end{aligned}
$$
因为最后要求的是 $[x^s]F(x)$ 其中 $s=1\sim m$，所以对于内部只需要枚举每个 $v_i$ 的值进行贡献，复杂度是 $O(\sum \frac{m}{i})=O(m\log m)$，然后进行 $\exp$ 复杂度也应当是 $O(m\log m)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int P = 998244353;

constexpr int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

struct Poly { /* 省略 */ };

const int N = 100010;
int n, m, cnt[N];

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    cin >> n >> m;
    Poly f;
    f.resize(m);
    for (int i = 1, v; i <= n; i++) cin >> v, cnt[v]++;
    for (int v = 1; v <= m; v++) {
        for (int j = v; j <= m; j += v) {
            f[j] = (f[j] + cnt[v] * qmi(j / v, P-2)) % P;
        }
    }
    f = f.exp();
    for (int i = 1; i <= m; i++) cout << f[i] << '\n';
    return 0;
}
```

## [CF438E] The Child and Binary Tree

> 给定长度为 $n$ 的互异正整数序列 $c_i$，二叉树有点权，一个树的权值为所有点权之和，所有点权都等于某个 $c_i$，对于 $1\le s\le m$，统计权值为 $s$ 的二叉树个数，答案对 $998244353$ 取模，其中 $1\le n,m,c_i\le 10^5$​。
>
> 题目链接：[CF438E](https://codeforces.com/problemset/problem/438/E)。

计数 DP，设 $f_n$ 表示 $n$ 的树的个数，那么枚举根的权值为 $i$，有：
$$
f_n=\begin{cases}
1\quad n=0\\
\sum_{i=1}^n [i\in C] \sum_{j=0}^{n-i} f_jf_{n-i-j}
\end{cases}
$$
设 $g_i=[i\in C]$，那么可以看出这是一个卷积的形式，设生成函数分别为 $F(x),G(x)$，那么有：
$$
F(x)=G(x)F^2(x)+1
$$
这里 $+1$ 是因为 $G(x)$ 常数项为 $0$，这个二元一次方程可以解出：
$$
F(x)=\frac{1\pm \sqrt{1-4G(x)}}{2G(x)}=\frac{2}{1\mp \sqrt{1-4G(x)}}
$$
由于 $G(0)=0$，所以只能取 $+$，因为如果取 $-$ 这个式子无法在 $0$ 处泰勒展开，也就不能写成 $\sum c_ix^i$ 的形式，那么 $F(x)=\frac{2}{1+\sqrt{1-4G(x)}}$。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
const int P = 998244353;

constexpr int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}

struct Poly { /* 省略 */ };

const int N = 100010;
int c[N];
Poly G;

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    int n, m;
    cin >> n >> m;

    G.resize(m);
    for (int i = 1; i <= n; i++) {
        cin >> c[i];
        if (c[i] <= m) G[c[i]] = 1;
    }

    for (int i = 0; i <= m; i++) G[i] = G[i] * (P - 4) % P;
    G[0]++;
    G = G.sqrt();
    G[0]++;
    G = G.inv();
    for (int i = 0; i <= m; i++) G[i] = G[i] * 2 % P;
    for (int i = 1; i <= m; i++) cout << G[i] << '\n';

    return 0;
}
```

