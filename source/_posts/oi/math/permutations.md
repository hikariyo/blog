---
date: 2023-08-02 12:11:00
title: 数学 - 组合计数
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 递推

> 给定 n 组询问，每组询问给定两个整数 a，b，请你输出 $C_b^a \bmod (10^9+7)$ 的值。
>
> **数据范围**
>
> $1\le n \le 10000$
>
> $1\le b \le a \le 2000$
>
> 题目链接：[AcWing 885](https://www.acwing.com/problem/content/887/)。

复杂度：$O(n^2)$

由加法原理可以得到：
$$
\binom{a}b=\binom{a-1}b+\binom{a-1}{b-1}
$$
当 $n,m$ 不大，但询问次数多时，使用递推的方式求组合数。

```cpp
#include <iostream>
using namespace std;

const int N = 2001, mod = 1e9+7;
int C[N][N];

int main() {
    C[0][0] = 1;
    for (int i = 1; i < N; i++) {
        C[i][0] = 1;
        for (int j = 1; j <= i; j++)
            C[i][j] = (C[i-1][j-1] + C[i-1][j]) % mod;
    }
    int n;
    scanf("%d", &n);
    while (n--) {
        int a, b;
        scanf("%d%d", &a, &b);
        printf("%d\n", C[a][b]);
    }
    return 0;
}
```

## 逆元

> 给定 n 组询问，每组询问给定两个整数 a，b，请你输出 $C_b^a \bmod (10^9+7)$ 的值。
>
> **数据范围**
>
> $1\le n \le 10000$
>
> $1\le b \le a \le 10^5$
>
> 题目链接：[AcWing 886](https://www.acwing.com/problem/content/888/)。

复杂度：预处理 $O(n\log n)$，查询 $O(1)$

由定义式与同余性质得到：
$$
\binom{a}b=\frac{a!}{(a-b)!b!}\\
\binom{a}b\equiv a![(a-b)!]^{-1}(b!)^{-1} (\text{mod }p)
$$
预处理数据范围内的每个数与逆元的阶乘 $\text{mod }p$ 的结果就可以了，这里用费马小定理，或者拓展欧几里得算法都可以求逆元。

这里逆元也是一个积性函数，可以简单证明：
$$
\begin{aligned}
aa^{-1}&\equiv 1(\text{mod }p)\\
bb^{-1}&\equiv 1(\text{mod }p)\\
\Rightarrow ab(a^{-1}b^{-1})&\equiv 1(\text{mod }p)\\
\therefore (ab)^{-1}&\equiv a^{-1}b^{-1}(\text{mod }p)
\end{aligned}
$$
因此就可以递推求范围内的阶乘与逆元阶乘了。

```cpp
#include <cstdio>
using namespace std;

typedef long long LL;
const int mod = 1e9+7, N = 100010;

LL fact[N], infact[N];

LL qmi(LL a, LL k) {
    LL res = 1;
    while (k) {
        if (k & 1) res = (res * a) % mod;
        a = (a * a) % mod;
        k >>= 1;
    }
    return res;
}

int main() {
    fact[0] = infact[0] = 1;
    for (int i = 1; i < N; i++) {
        fact[i] = fact[i-1]*i % mod;
        infact[i] = infact[i-1]*qmi(i, mod-2) % mod;
    }
    int n;
    scanf("%d", &n);
    while (n--) {
        int a, b;
        scanf("%d%d", &a, &b);
        printf("%lld\n", ((fact[a]*infact[a-b])%mod*infact[b]) % mod);
    }
    return 0;
}
```

## Lucas 定理

> 给定 n 组询问，每组询问给定三个整数 a,b,p，其中 p 是质数，请你输出 $C_a^b \bmod p$ 的值。
>
> **数据范围**
>
> $1\le n\le 20$
>
> $1\le b \le a \le 10^{18}$
>
> $1\le p\le 10^5$
>
> 题目链接：[AcWing 887](https://www.acwing.com/problem/content/889/)。

复杂度：$O(f(p)+g(p)\log_p n)$，其中 $f(p)$ 是预处理复杂度，$g(p)$ 是单次查询复杂度。

定理：求比较大的组合数在模 $p$ 意义下的值：
$$
\binom{a}b\equiv \binom{a\bmod p}{b\bmod p}\binom{\lfloor a/p \rfloor }{\lfloor b/p \rfloor}\pmod p
$$
证明（以下步骤省略 $(\text{mod }p)$，）：
$$
(1+x)^p=\binom {p}0 1+\binom {p}1 x+\binom {p}2 x^2+\cdots+\binom {p}p x^p
$$
由于 $p$ 是质数，所以有 $\binom {p}n\equiv 0,n\in(0,p)$

因此：
$$
\begin{aligned}
(1+x)^p&\equiv 1+x^p\\
(1+x)^{p^2}&\equiv (1+x^p)^p\equiv1+x^{p^2}\\
(1+x)^{p^k}&\equiv 1+x^{p^k}
\end{aligned}
$$

这里用二项式定理证明：
$$
\begin{aligned}
\text{Let }n=k_1p&+r_1,m=k_2p+r_2\\
(1+x)^n&\equiv \sum_{i=0}^n\binom {n}ix^i\\
(1+x)^n&\equiv (1+x^p)^{k_1}(1+x)^{r_1}\\
&\equiv (\sum_{i=0}^{k_1}\binom {k_1}ix^{pi})(\sum_{j=0}^{r_1}\binom {r_1}jx^j)
\end{aligned}
$$

分别观察两个括号里面的 $x^m=x^{k_2p}x^{r_2}$，系数分别是 $\binom {n}m\equiv \binom {k_1}{k_2}\binom {r_1}{r_2}$。

所以，可以只通过计算 $p$ 以内的组合数就可以用 Lucas 定理算出一个很大的组合数了。

```cpp
#include <iostream>
using namespace std;

typedef long long LL;

int p;

LL qmi(LL a, LL k) {
    LL res = 1;
    while (k) {
        if (k & 1) res = (res * a) % p;
        a = (a * a) % p;
        k >>= 1;
    }
    return res;
}

LL C(LL a, LL b) {
    LL res = 1;
    for (LL i = 1, j = a; i <= b; i++, j--) {
        res = res * j % p;
        res = res * qmi(i, p-2) % p;
    }
    return res;
}

LL lucas(LL a, LL b) {
    if (a < p && b < p) return C(a, b);
    
    return lucas(a/p, b/p)*C(a%p, b%p) % p;
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        LL a, b;
        scanf("%lld%lld%d", &a, &b, &p);
        printf("%lld\n", lucas(a, b));
    }
    return 0;
}
```



## 高精度

> 输入 a,b，求 $C_a^b$ 的值。注意结果可能很大，需要使用高精度计算。
>
> 题目链接：[AcWing 888](https://www.acwing.com/problem/content/890/)。

对 $C_a^b$ 分解质因数，再用高精度乘把质因数全部求积，这样效率比单纯用定义来做用高精度除要快。

对于质因数 $p$ 在 $n!$ 中的次数，之前的文章提到过，它是这个式子：
$$
\text{cnt}(p)=\sum_{i=1}^{\lfloor \log_p n\rfloor}\lfloor \frac{n}{p^i}\rfloor
$$
所以把分子的质因数分解出来，然后减掉分母质因数的次数就是最终分解的答案。

```cpp
#include <cstdio>
#include <vector>
using namespace std;

const int N = 5010;
int primes[N], cnt;
int sum[N];
bool st[N];

void init(int n) {
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cnt++] = i;
        for (int j = 0; primes[j] <= n/i; j++) {
            st[primes[j]*i] = true;
            if (i % primes[j] == 0) break;
        }
    }
}

vector<int> mul(vector<int>& a, int b) {
    vector<int> c;
    int t = 0;
    for (int i = 0; i < a.size() || t; i++) {
        if (i < a.size()) t += a[i] * b;
        c.push_back(t%10);
        t /= 10;
    }
    while (c.size() && c.back() == 0) c.pop_back();
    return c;
}

int factorize(int n, int p) {
    int res = 0;
    while (n) {
        res += n/p;
        n /= p;
    }
    return res;
}

int main() {
    int a, b;
    scanf("%d%d", &a, &b);
    init(max(a, b));
    vector<int> res;
    res.push_back(1);
    for (int i = 0; i < cnt; i++) {
        int p = primes[i];
        int s = factorize(a, p) - factorize(a-b, p) - factorize(b, p);
        while (s--) res = mul(res, p);
    }
    for (int i = res.size()-1; i >= 0; i--) {
        printf("%d", res[i]);
    }
    puts("");
    return 0;
}
```

## [CQOI2014] 数三角形

> 给定一个 $N\times M$ 的网格，请计算三点都在格点上的三角形共有多少个。注意三角形的三点不能共线。
>
> 题目链接：[AcWing 1310](https://www.acwing.com/problem/content/1312/)，[P3166](https://www.luogu.com.cn/problem/P3166)。

显然，对三个点所有的选法是 $\binom{(n+1)(m+1)}3$

然后根据容斥，排除掉不合法的方案数。

+ 三点成水平线：$(m+1)\binom{n+1}3$

+ 三点成竖直线：$(n+1)\binom{m+1}3$

+ 三点成斜率为正的斜线：

    这个比较难搞，方法是先枚举 $\Delta x,\Delta y$ 看整点的个数，给斜率约分到最简，列出点斜式：
    $$
    l:y=\frac{\Delta y/\gcd(\Delta x, \Delta y)}{\Delta x/\gcd(\Delta x,\Delta y)}(x-x_1)+y_1\\
    $$
    如果整点 $(m,n)$ 在直线 $l$ 上且在 $(x_1,y_1),(x_2,y_2)$ 之间，那么一定有：
    $$
    n=\frac{\Delta y/\gcd(\Delta x, \Delta y)}{\Delta x/\gcd(\Delta x, \Delta y)}(m-x_1)+y_1\\
    \Rightarrow \frac{\Delta x}{\gcd(\Delta x, \Delta y)}|(m-x_1)
    $$
    考虑到 $m$ 的取值范围，在下式中 $t$ 的解的数量就是 $m$ 的数量：
    $$
    m=t\frac{x_2-x_1}{\gcd(\Delta x, \Delta y)}+x_1,t\in \N^*,m\in[x_1,x_2]
    $$
    带入解得：
    $$
    0\le t\le \gcd(\Delta x,\Delta y)
    $$
    去掉首尾两段点，在枚举到的两点之间的整点个数就是：
    $$
    \gcd(\Delta x,\Delta y)-1
    $$
    综合考虑，在 $m\times n$ 的网格中，可以放下 $(m-\Delta x+1)(n-\Delta y+1)$ 个这样的直角三角形：考虑当 $\Delta x=m$ 时是有一个解的，所以这里加上了一。再用乘法原理乘上这个推导出来的个数就得到了斜率为正时的三点共线个数。

+ 三点成斜率为负的斜线：与正的对称，直接乘 2 就可以。

```cpp
#include <cstdio>
using namespace std;

typedef long long LL;
int n, m;

LL C(int n) {
    return (LL)n*(n-1)*(n-2)/6;
}

int gcd(int a, int b) {
    return b ? gcd(b, a%b) : a;
}

int main() {
    scanf("%d%d", &n, &m);
    LL res = C((m+1)*(n+1))-(m+1)*C(n+1)-(n+1)*C(m+1);
    for (int dx = 1; dx <= n; dx++) {
        for (int dy = 1; dy <= m; dy++) {
            res -= 2ll*(n-dx+1)*(m-dy+1)*(gcd(dx, dy)-1);
        }
    }
    printf("%lld\n", res);
    return 0;
}
```

## 好三元组

> 平面中有一个圆，圆心坐标为 (0,0)，周长为 C。
>
> 圆周上有 C 个整数位置点，编号 0∼C−1，其中：
>
> - 0 号点：圆上的最右点。
> - 1∼C−1 号点：从 0 号点到自身位置的**逆时针**弧长恰好等于自身编号。
>
> 给定一个长度为 N 的整数序列 P1,P2,…,PN（0≤Pi≤C−1）。
>
> 请你计算，一共有多少个不同的整数三元组 (a,b,c) 同时满足：
>
> 1. 1≤a<b<c≤N
> 2. 原点 (0,0) 严格位于以点 Pa,Pb,Pc 为顶点的三角形内部。注意，三角形边上的点不在三角形内部。
>
> 题目链接：[AcWing 5183](https://www.acwing.com/problem/content/5186/)。

显然需要用 $O(n)$ 或者 $O(n\log n)$ 的算法来解决这道题目，再看求的是方案数，所以考虑是组合计数问题。

联想到之前做过的 CQOI2014 那个数三角形的问题，这里也是用求补集的思想，首先一共有 $C_n^3$ 种选择，然后减掉不合法的选择就可以了。

枚举这 $n$ 个点，那么选中点 $i$ 的情况下不合法的方案数是：
$$
x=\sum_{j=i+1}^{i+\frac{c}{2}} cnt_{j},y=cnt_i\\
t=C_y^1C_x^2+C_y^2C_x^1+C_y^3
$$
设所有不合法的 $C_a^b=0$，但是如果就这么做下去会出现问题，因为有的情况是被重复计算的，就是在枚举到在同一直径上的两个点 $i,j$ 时，会把相同的选法选两次，同时这只会发生在 $c$ 为偶数的情况，因为 $c$ 为奇数时不会有两个点在同一直径上，所以最后把这种情况加上即可。

为了处理前缀和方便，这里把所有的编号都加一。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 2000010;
int S[N];
int n, c;

LL C(LL x, int k) {
    if (k == 1) return x;
    else if (k == 2) return x * (x-1) / 2;
    else if (k == 3) return x * (x-1) * (x-2) / 6;
    else return -1;
}

int main() {
    scanf("%d%d", &n, &c);
    for (int i = 0; i < n; i++) {
        int p; scanf("%d", &p);
        ++p;
        S[p]++, S[p+c]++;
    }

    for (int i = 1; i <= 2*c; i++) S[i] += S[i-1];
    
    LL res = C(n, 3);

    // 枚举每个位置
    for (int i = 1; i <= c; i++) {
        int y = S[i] - S[i-1];
        // i+1 ~ i + c/2
        int x = S[i+c/2] - S[i];
        res -= C(y, 1)*C(x, 2) + C(y, 2)*C(x, 1) + C(y, 3);
    }
    
    // 直径被重复计算了
    if (c % 2 == 0) {
        for (int i = 1; i <= c/2; i++) {
            int y = S[i] - S[i-1];
            int x = S[i+c/2] - S[i+c/2-1];
            res += C(y, 1)*C(x, 2) + C(y, 2)*C(x, 1);
        }
    }
    
    printf("%lld\n", res);
    return 0;
}
```

## 序列统计

> 给定三个整数 N,L,R，统计长度在 1 到 N 之间，元素大小都在 L 到 R 之间的单调不降序列的数量。
>
> 输出答案对 `1e6+3`  取模的结果。
>
> 题目链接：[AcWing 1312](https://www.acwing.com/problem/content/1314/)，[LOJ 10235](https://loj.ac/p/10235)。

题目问的是相对大小关系，因此可以先映射为 $[0,R-L]$ 间的数。

设我们要找的序列长度是 $K$，这些数分别是 $a_1,a_2,\cdots,a_k$，它们是满足单调不下降的。

1. 用差分的思想，对这个数列构造一个差分数列 $x_1,x_2,\cdots,x_k$，由于是单调不下降，因此有 $x_i\ge 0$，并且 $\sum x_i\le R-L$

    令 $y_i=x_i+1\gt 0$，有 $\sum y_i\le R-L+K$，将问题转化成了一共有 $R-L+K$ 个小球，放到 $k$ 个盒子中，每个盒子至少放一个球，问整数解的个数。

    注意这里是不等式，所以最后一个位置可以插板。

    我们需要在 $R-L+K$ 个小球中插板，首项元素前面不可以放，其余位置都能放，于是这里的整数解个数就是 $\binom{R-L+K}K$，枚举所有的 $K$ 累项求和就是最终要求的答案。

2. 构造 $b_i=a_i+i-1$，这样就使得数列 $b_1,b_2,\cdots,b_k$ 严格单调上升，此时 $b_k \le R-L+K-1$，那就是从 $R-L+K$ 个数中选出 $K$ 个数，就是 $\binom{R-L+K}K$

两种方法都能求出来对于每个 $K$ 的方案数，核心思想是将一个非严格单调性的问题转化成严格单调性问题，接下来要做的就是对这些方案数累项求和。设 $M=R-L$，那么要求的式子可以如下转化：
$$
\begin{aligned}
\sum_{K=1}^N \binom {M+K}K&=\binom{M+1}1+\binom{M+2}2+\cdots+\binom{M+N}N\\
&=\binom{M+1}M+\binom{M+2}M+\cdots+\binom{M+N}M\\
&=\binom{M+1}{M+1}+\binom{M+1}M+\binom{M+2}M+\cdots +\binom{M+N}M-1\\
&=\binom{M+2}{M+1}+\binom{M+2}M+\binom{M+3}M+\cdots +\binom{M+N}M-1\\
&=\binom{M+N+1}{M+1}-1
\end{aligned}
$$
我清晰的记得我做过高中数学中的一个题目，是一个选择题，大概也是这样子用递推公式求和，所以这个题目的这一步骤难度也只是高中数学的难度而已。

用 Lucas 定理求这个比较大的组合数。

```cpp
#include <iostream>
using namespace std;

typedef long long LL;
const int mod = 1e6+3;

LL inv(LL a) {
    int k = mod-2;
    LL res = 1;
    while (k) {
        if (k & 1) res = res * a % mod;
        a = a * a % mod;
        k >>= 1;
    }
    return res;
}

LL C(int a, int b) {
    LL up = 1, down = 1;
    for (int i = 1, j = a; i <= b; i++, j--) {
        up = up * j % mod;
        down = down * i % mod;
    }
    return up * inv(down) % mod;
}

LL lucas(LL a, LL b) {
    if (a < mod && b < mod) return C(a, b);
    return lucas(a/mod, b/mod) * C(a%mod, b%mod) % mod;
}

int main() {
    int T, n, l, r;
    scanf("%d", &T);
    while (T--) {
        scanf("%d%d%d", &n, &l, &r);
        printf("%d\n", ((lucas(r-l+n+1, r-l+1)-1)+mod)%mod);
    }
    return 0;
}
```

这里不要把 `inv` 写到求组合数的循环里面了，会大大加大复杂度，统一到循环结束后再乘。

## 卡特兰数

> 给定 n 个 0 和 n 个 1，它们将按照某种顺序排成长度为 2n 的序列，求它们能排列成的所有序列中，能够满足任意前缀序列中 0 的个数都不少于 1 的个数的序列有多少个。
>
> 输出的答案对 `1e9+7` 取模。
>
> 题目链接：[AcWing 889](https://www.acwing.com/problem/content/891/)。

这种进行一个操作前必须先进行另外一个操作的方案数，叫做卡特兰数。类似于一个栈，栈中必须有元素才能弹出，这里 0 就是入栈，1 就是出栈。

或者，也可以看作从格点 $(0,0)$ 到 $(n,n)$ 不越过对角线 $y=x$ 的所有路径的方案数，一个 0 相当于向上走，1 相当于向右走。

如果一个路径超过了对角线，将其关于直线 $y=x+1$ 对称后终点 $(n,n)$ 就对称到了 $(n-1,n+1)$，这只是一个简单的解析几何问题，这里不再说明如何得出的这个点。

+ $(0,0) \to (n,n)$ 的所有方案数是 $\binom{2n}n$

+ $(0,0)\to(n-1,n+1)$ 的方案数是 $\binom{2n}{n-1}$

求一个补集就是所有的方案数：
$$
\begin{aligned}
\binom{2n}n-\binom{2n}{n-1}&=\frac{(2n)!}{n!n!}-\frac{(2n)!}{(n-1)!(n+1)!}\\
&=\frac{(2n)!(n+1)-(2n)!n}{(n+1)!n!}\\
&=\frac{(2n)!}{(n+1)!n!}\\
&=\frac{1}{n+1}\binom{2n}n
\end{aligned}
$$
这个数被称为卡特兰数。

```cpp
#include <cstdio>
using namespace std;

typedef long long LL;
const int mod = 1e9+7;

LL inv(LL a) {
    LL res = 1, k = mod-2;
    while (k) {
        if (k & 1) res = res * a % mod;
        a = a * a % mod;
        k >>= 1;
    }
    return res;
}

LL C(LL a, LL b) {
    LL res = 1;
    for (LL i = 1, j = a; i <= b; i++, j--) {
        res = res * j % mod;
        res = res * inv(i) % mod;
    }
    return res;
}


int main() {
    int n;
    scanf("%d", &n);
    printf("%lld\n", C(2*n, n) * inv(n+1) % mod);
    return 0;
}
```

## 网格

> 某城市的街道呈网格状，左下角坐标为 A(0,0)，右上角坐标为 B(n,m))，其中 n≥m。
>
> 现在从 A(0,0) 点出发，只能沿着街道向正右方或者正上方行走，且不能经过图示中直线左上方的点，即任何途径的点 (x,y) 都要满足 x≥y，请问在这些前提下，到达 B(n,m) 有多少种走法。
>
> 题目链接：[AcWing 1315](https://www.acwing.com/problem/content/1317/)，[LOJ 10238](https://loj.ac/p/10238)。

本题与卡特兰数思路基本相同，如果越界将路径沿 $y=x+1$ 对称，这样 $(n,m)$ 会被对称到 $(m-1,n+1)$，要求的结果就是：
$$
\binom{n+m}m-\binom{n+m}{m-1}
$$
这需要高精度，参考本文上面的高精度组合数计算，就是找到所有的质因数之后累项求积。

```cpp
#include <cstdio>
#include <vector>
using namespace std;

// n+m 最大 10000
const int N = 10010;
int primes[N], cnt;
bool st[N];
int n, m;

void getprimes(int n) {
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cnt++] = i;
        for (int j = 0; primes[j] <= n/i; j++) {
            st[primes[j] * i] = true;
            if (i % primes[j] == 0) break;
        }
    }
}

int factorize(int n, int p) {
    int res = 0;
    while (n) {
        res += n/p;
        n /= p;
    }
    return res;
}

vector<int> mul(const vector<int>& a, int b) {
    vector<int> res;
    int t = 0;
    for (int i = 0; i < a.size() || t; i++) {
        if (i < a.size()) t += a[i]*b;
        res.push_back(t % 10);
        t /= 10;
    }
    return res;
}

vector<int> sub(const vector<int>& a, const vector<int> &b) {
    vector<int> res;
    for (int i = 0, t = 0; i < a.size(); i++) {
        t = a[i] - t;
        if (i < b.size()) t -= b[i];
        res.push_back((t+10)%10);
        if (t < 0) t = 1;
        else t = 0;
    }
    while (res.size() && res.back() == 0) res.pop_back();
    return res;
}

vector<int> C(int a, int b) {
    vector<int> res;
    res.push_back(1);
    for (int i = 0; i < cnt; i++) {
        int p = primes[i];
        int s = factorize(a, p) - factorize(a-b, p) - factorize(b, p);
        while (s--) res = mul(res, p);
    }
    return res;
}

int main() {
    getprimes(N-1);
    scanf("%d%d", &n, &m);
    vector<int> res = sub(C(n+m, m), C(n+m, m-1));
    for (int i = res.size() - 1; i >= 0; i--)
        printf("%d", res[i]);
    puts("");
    return 0;
}
```

## [HNOI2009] 有趣的数列

> 我们称一个长度为 2n 的数列是有趣的，当且仅当该数列满足以下三个条件：
>
> 1. 它是从 1 到 2n 共 2n 个整数的一个排列 $\{a_i\}$；
> 2. 所有的奇数项满足 $a_1<a_3<\cdots<a_{2n-1}$ ，所有的偶数项满足 $a_2<a_4<\cdots<a_{2n}$；
> 3. 任意相邻的两项 $a_{2i-1}$ 与 $a_{2i}$ $(1≤i≤n)$ 满足奇数项小于偶数项，即：$a_{2i−1}<a_{2i}$。
>
> 任务是：对于给定的 n，请求出有多少个不同的长度为 2n 的有趣的数列。
>
> 因为最后的答案可能很大，所以只要求输出答案 mod P 的值。
>
> 题目链接：[AcWing 1316](https://www.acwing.com/problem/content/1318/)，[P3200](https://www.luogu.com.cn/problem/P3200)。

在数列 $1,2,3,4,5,\cdots,2n$ 中升序选择的时候，每时每刻都要保证奇数项的个数不少于偶数项的个数。假设前 5 个数中奇数项个数为 2，偶数项为 3，这会导致：$1,2,3,4,\_,5$，不合法。

所以答案就是卡特兰数，这里用分解质因数的方法求解。
```cpp
#include <cstdio>
using namespace std;

typedef long long LL;
const int N = 2e6+10;
int n, mod;
int primes[N], cnt;
bool st[N];

void getprimes(int n) {
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cnt++] = i;
        for (int j = 0; primes[j] <= n/i; j++) {
            st[primes[j]*i] = true;
            if (i % primes[j] == 0) break;
        }
    }
}

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = (LL)res * a % mod;
        a = (LL)a * a % mod;
        k >>= 1;
    }
    return res;
}

int factorize(int n, int p) {
    int cnt = 0;
    while (n) {
        cnt += n / p;
        n /= p;
    }
    return cnt;
}

int C(int a, int b) {
    int res = 1;
    for (int i = 0; i < cnt; i++) {
        int p = primes[i];
        int s = factorize(a, p) - factorize(a-b, p) - factorize(b, p);
        res = (LL)res * qmi(p, s) % mod;
    }
    return res;
}

int main() {
    scanf("%d%d", &n, &mod);
    getprimes(2*n);
    printf("%d\n", (C(2*n, n) - C(2*n, n-1) + mod) % mod);
    return 0;
}
```

## 二进制

> 给定一个长度为 N 的二进制串（01 串）以及一个正整数 K。
>
> 按照从左到右的顺序，依次遍历给定二进制串的 N−K+1 个长度为 K 的子串，并计算每个遍历子串的各位数字之和。
>
> 将这 N−K+1 个子串数字和按照子串的遍历顺序进行排列，得到的序列就是给定二进制串的 K-子串数字和序列。
>
> 注意，所有子串数字和均用十进制表示。
>
> 例如，当 K=4 时，二进制串 `110010` 的 K-子串数字和序列为 `2 2 1`，分析过程如下：
>
> - 依次遍历 `110010` 的所有长度为 4 的子串： `1100`、`1001`、`0010`。
> - 计算每个遍历子串的各位数字之和：1+1+0+0=2、1+0+0+1=2、0+0+1+0=1。
> - 将所有子串数字和按顺序排列，最终得到 `2 2 1`。
>
> 现在，给定 N,K 以及一个 K-子串数字和序列，请你计算一共有多少个不同的长度为 N 的二进制串可以得到该 𝙺-K-子串数字和序列。
>
> **数据保证：至少存在一个长度为 N 的二进制串可以得到该 K-子串数字和序列。**
>
> 由于结果可能很大，你只需要输出对 `1e6+3` 取模后的结果。
>
> 题目链接：[AcWing 5170](https://www.acwing.com/problem/content/5173/)。

设第一个区间和为 $S_1$，第二个为 $S_2$，那么一定有如下关系：

+ $S_1=S_2$：第一个区间开头等于第二个区间末尾。
+ $S_1=S_2+1$：第一个区间开头为 1，第二个区间末尾为 0。
+ $S_1+1=S_2$：第一个区间开头为 0，第二个区间末尾为 1。

如果它们相差大过 $1$ 是无解的，但题目保证有解这里就不用考虑了。

当第一个区间确定的时候，后面区间也一定确定，这里用数学归纳法：设第 $i$ 个区间区间确定，接下来 $S_i,S_{i+1}$ 一定属于上面三种情况之一，都是可以确定出来的，所以只需要确定第一个区间。

这里把符合第一个条件的所有值都加到同一个集合中，用并查集维护它们的取值，最后看第一个区间中有多少数字能被确定，设 $c_0,c_1$ 分别为能确定的 $0,1$ 数，这个区间一共有 $S_1$ 个 $1$、$k$ 个空位，所以结果就是：
$$
\binom{k-c_0-c_1}{S_1-c_1}
$$
这里可以用 Lucas 定理或者最基础的逆元来求。

```cpp
#include <cstdio>
#include <cstdlib>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 1000010, MOD = 1e6+3;
int n, k;
int S[N], p[N], v[N];

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

int qmi(int a, int k, int p) {
    int res = 1;
    while (k) {
        if (k & 1) res = (LL)res * a % p;
        a = (LL)a * a % p;
        k >>= 1;
    }
    return res;
}

int C(int a, int b) {
    // a!/(b!)(a-b)!
    int fa = 1, fb = 1, fab = 1;
    for (int i = 2; i <= a; i++) fa = (LL)fa * i % MOD;
    for (int i = 2; i <= b; i++) fb = (LL)fb * i % MOD;
    for (int i = 2; i <= a-b; i++) fab = (LL)fab * i % MOD;
    return (LL)fa*qmi(fb, MOD-2, MOD)*qmi(fab, MOD-2, MOD) % MOD;
}

int main() {
    scanf("%d%d", &n, &k);
    for (int i = 1; i <= n; i++)
        p[i] = i, v[i] = -1;
    
    for (int i = 1; i <= n-k+1; i++)
        scanf("%d", &S[i]);
    
    for (int i = 1; i <= n-k; i++) {
        // j: 下一个区间末尾
        int j = i+k;
        if (S[i] == S[i+1]) p[find(i)] = find(j);
    }
    
    // 先合并所有集合然后再赋值
    for (int i = 1; i <= n-k; i++) {
        int j = i+k;
        if (S[i] == S[i+1]+1) v[find(i)] = 1, v[find(j)] = 0;
        else if (S[i]+1 == S[i+1]) v[find(i)] = 0, v[find(j)] = 1;
        else if (S[i] != S[i+1]) exit(-1);
    }
    
    int c0 = 0, c1 = 0;
    for (int i = 1; i <= k; i++) {
        if (v[find(i)] == 1) c1++;
        if (v[find(i)] == 0) c0++;
    }
    
    printf("%d\n", C(k-c0-c1, S[1]-c1));
    return 0;
}
```

 ## 子序列方差之和

> 给定 $n$ 个元素的序列 $a$，求所有非空子序列方差之和，对 $10^9+7$ 取模，$1\le n,a_i\le 10^6$

不知道有无原题，某校赛题目。

首先方差大家都知道怎么求。
$$
\sigma = \frac{1}{n}\sum_{i=1}^n a_i^2 - (\frac{1}{n}\sum_{i=1}^n a_i)^2
$$

对于前面的式子，我们可以统计每一个 $a_i$ 出现在长度为 $x$ 的子序列中出现了几次，相当于 $n-1$ 个剩下的数里选 $x-1$ 个组合成 $x$ 长度的子序列。
$$
(\sum_{x=1}^n \frac{\binom{n-1}{x-1}}{x})(\sum_{i=1}^n a_i^2)
$$
对于后面的式子，我们可以拆开看。
$$
\frac{1}{n^2}(\sum_{i=1}^n a_i)(\sum_{i=1}^n a_i)
$$
这两个和式可以自由组合，它们出现在长度为 $x$ 的子序列中的次数就是，当两个数相同时，是剩下 $n-1$ 个数里选 $x-1$ 个数；当两个数不同时，在剩下 $n-2$ 个数里边选 $x-2$ 个数。
$$
(\sum_{x=1}^n\frac{\binom{n-1}{x-1}}{x^2})(\sum_{i=1}^n a_i^2)+(\sum_{x=2}^n\frac{\binom{n-2}{x-2}}{x^2})(\sum_{i=1}^n\sum_{j=1,j\ne i}^na_ia_j)\\
=(\sum_{x=1}^n\frac{\binom{n-1}{x-1}}{x^2})(\sum_{i=1}^n a_i^2)+(\sum_{x=2}^n\frac{\binom{n-2}{x-2}}{x^2})[(\sum_{i=1}^n a_i)^2-\sum_{i=1}^n a_i^2]\\
$$
都是相互独立的，所以可以 $O(n)$ 求解。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int P = 1e9+7, N = 1e6+10;
#define int long long
int n, a[N], fact[N], infact[N];

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = res * a % P;
        a = a * a % P;
        k >>= 1;
    }
    return res;
}


int C(int n, int m) {
    return fact[n] * infact[m] % P * infact[n-m] % P;
}


signed main() {
    scanf("%lld", &n);
    for (int i = 1; i <= n; i++) scanf("%lld", &a[i]);

    infact[0] = fact[0] = 1;
    for (int i = 1; i <= n; i++) fact[i] = fact[i-1] * i % P;
    infact[n] = qmi(fact[n], P-2);
    for (int i = n-1; i; i--) infact[i] = infact[i+1] * (i+1) % P;

    int k1 = 0, k2 = 0, k3 = 0;
    int squaresum = 0, sum = 0;
    for (int i = 1; i <= n; i++) {
        k1 = (k1 + C(n-1, i-1) * qmi(i, P-2) % P) % P;
        k2 = (k2 + C(n-1, i-1) * qmi(i, 2*P-4) % P) % P;
        if (i > 1) k3 = (k3 + C(n-2, i-2) * qmi(i, 2*P-4) % P) % P;
        squaresum = (squaresum + a[i]*a[i] % P) % P;
        sum = (sum + a[i]) % P;
    }

    int A = (k1 * squaresum % P - k2 * squaresum % P - k3 * (sum * sum % P - squaresum) % P) % P;
    printf("%lld\n", (A+P)%P);
    return 0;
}
```

## [CF559C] Gerald and Giant Chess

> 给定 $h\times w$ 的棋盘，其中有 $n$ 个格子 $(x_i,y_i)$ 不能走，求从 $(1,1)$ 走到 $(h, w)$ 的方案数，答案对 $10^9+7$ 取模。
>
> 数据范围：$1\le h, w\le 10^5, 1\le n\le 2000, 1\le x \le h, 1\le y\le w$。
>
> 题目链接：[CF559C](http://codeforces.com/problemset/problem/559/C)。

设 $dp(i)$ 表示走到第 $i$ 个不能走的格子的方案数，起点为 $S=(1,1)$ 终点为 $T=(h,w)$，然后假设 $slv(A,B)$ 可以求出来在没有限制的情况下 $A\to B$ 的所有方案。走到 $i$ 需要满足不经过它左上方的不能走的格子，即：
$$
dp(i)=sln(S,P_i)-\sum_{P_j\le P_i} dp(j)
$$
这里的 $P_j\le P_i$ 意思是 $P_j$ 在 $P_i$ 左上角。

接下来考虑 $sln(A,B)$ 怎么写，因为只能向右或向下走，所以一共走 $\Delta x+\Delta y$ 步，取 $\Delta x$ 步向右走即可。
$$
sln(A,B)=\binom{\Delta x+\Delta y}{\Delta x}
$$
这里组合数的参数需要的数值范围是 $2\times 10^5$，所以用预处理阶乘和逆元的方式求组合数。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 2010, M = 200010, P = 1e9+7;
int n, h, w;
int fact[M], infact[M];
int dp[N];

struct Point {
    int x, y;

    bool operator<(const Point& p) const {
        if (x != p.x) return x < p.x;
        return y < p.y;
    }
} p[N];

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = 1LL * res * a % P;
        a = 1LL * a * a % P;
        k >>= 1;
    }
    return res;
}

int C(int n, int m) {
    return 1LL * fact[n] * infact[m] % P * infact[n-m] % P;
}

int sln(int X1, int Y1, int X2, int Y2) {
    return C(X2-X1+Y2-Y1, X2-X1);
}

int main() {
    scanf("%d%d%d", &h, &w, &n);

    fact[0] = infact[0] = 1;
    for (int i = 1; i <= h+w; i++) fact[i] = 1LL * fact[i-1] * i % P;
    infact[h+w] = qmi(fact[h+w], P-2);
    for (int i = h+w-1; i >= 1; i--) infact[i] = 1LL * infact[i+1] * (i+1) % P;

    for (int i = 1; i <= n; i++) scanf("%d%d", &p[i].x, &p[i].y);
    p[++n] = {h, w};
    sort(p+1, p+1+n);

    for (int i = 1; i <= n; i++) {
        dp[i] = sln(1, 1, p[i].x, p[i].y);
        for (int j = 1; j < i; j++) {
            if (p[j].y > p[i].y) continue;
            dp[i] = (dp[i] - 1LL * dp[j] * sln(p[j].x, p[j].y, p[i].x, p[i].y)) % P;
        }
    }
    if (dp[n] < 0) dp[n] += P;
    return !printf("%d\n", dp[n]);
}
```

