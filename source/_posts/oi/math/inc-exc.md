---
date: 2023-08-03 23:16:00
title: 数学 - 容斥原理
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 公式

公式中 $\cap_{i=1}^{s}A_i$ 表示任意选 $s$ 个集合交起来。
$$
|\cup_{i=1}^nA_i|=\sum_{s=1}^n (-1)^{s-1} |\cap_{i=1}^{s}A_i|
$$
下面推导复杂度。从 $n$ 个集合中任选 $s$ 个集合的选法一共有：
$$
C_n^1+C_n^2+\cdots+C_n^n
$$
根据二项式定理求这些组合数之和：
$$
(1+x)^n= C_n^0+C_n^1x+C_n^2x^2+\cdots+C_n^nx^n
$$
令 $x=1$ 就得到选法的总数量是：
$$
\sum_{i=1}^n C_n^i=2^n-1
$$
因此复杂度是 $O(2^n)$ 级别的。

## 能被整除的数

> 给定一个整数 n 和 m 个不同的质数 p1,p2,…,pm。
>
> 请你求出 1∼n 中能被 p1,p2,…,pm 中的至少一个数整除的整数有多少个。
>
> 题目链接：[AcWing 890](https://www.acwing.com/problem/content/892/)。

求出不能被所有数整除的数字的个数，然后用总数减去这个数。

```cpp
#include <iostream>
using namespace std;

const int M = 20;
int n, m, p[M];

int main() {
    cin >> n >> m;
    for (int i = 0; i < m; i++) cin >> p[i];
    
    int res = 0;
    for (int i = 0; i < 1 << m; i++) {
        int sgn = 1;
        long long factor = 1;
        for (int j = 0; j < m; j++) {
            if (i >> j & 1) {
                factor *= p[j];
                if (factor > n) break;
                sgn = -sgn;
            }
        }
        res += n / factor * sgn;
    }
    printf("%d\n", n - res);
    return 0;
}
```



## Devu 和花

> Devu 有 N 个盒子，第 i 个盒子中有 Ai 枝花。同一个盒子内的花颜色相同，不同盒子内的花颜色不同。
>
> Devu 要从这些盒子中选出 M 枝花组成一束，求共有多少种方案。若两束花每种颜色的花的数量都相同，则认为这两束花是相同的方案。求方案数对 `1e9+7` 取模后输出。
>
> 题目链接：[AcWing 214](https://www.acwing.com/problem/content/216/)，[CF451E](https://codeforces.com/problemset/problem/451/E)。

本题就是一个容斥，只不过涉及组合计数的知识。

如果盒子中没有花的限制，那么这可以用隔板法解决，假设第 $i$ 个盒子中取 $x_i$ 个花。
$$
x_1+x_2+\cdots+x_n = m,\forall x_i\ge 0
$$
这时可以用一种经典的转化：
$$
y_1+y_2+\cdots+y_n=m+n,\forall y_i=x_i+1\gt 0
$$
这时就可以用隔板法了，方案数是 $\binom {m+n-1}{n-1}$，到目前为止都很简单。

接下来用补集的思想，求出所有不满足要求的方案，然后用总方案数减去它们。设 $S_i$ 是不满足第 $i$ 个盒子的方案集合，也就是需要使得 $x_i \ge A_i+1$；

对于每一组 $\cap S_i$，先取出 $\sum (A_i+1)$ 个花，剩下按照原问题求解，也就是变成了有 $m-\sum(A_i+1)$ 朵花，每个花数量不限，求方案数，这个是 $\binom{m+n-1-\sum(A_i+1)}{n-1}$ 方案数。

因此最终的解就是：
$$
\binom{m+n-1}{n-1}-|\cup_{i=1}^n S_i|=\binom{m+n-1}{n-1}-\sum_{s=1}^n(-1)^{s-1}|\cap S_i|
$$
看一下复杂度，每次求组合数的复杂度和所有组合数的数量在一起是 $O(n2^n)$，数据范围内可以过；至于所有方案这里采用二进制枚举的方式来处理。

```cpp
#include <iostream>
using namespace std;

typedef long long LL;
const int N = 25, mod = 1e9+7;
int n, b;
LL m, down, q[N];

int qmi(int a, int k) {
    int res = 1;
    while (k) {
        if (k & 1) res = (LL)res * a % mod;
        a = (LL)a * a % mod;
        k >>= 1;
    }
    return res;
}

LL C(LL a) {
    if (a < b) return 0;
    LL up = 1;
    // 大数先模后乘 否则溢出
    for (LL i = a; i >= a-b+1; i--) {
        up = i % mod * up % mod;
    }
    return up * down % mod;
}

int main() {
    cin >> n >> m;
    // 预处理出组合数的 b 和分母中的 b!^{-1}
    b = n-1;
    down = 1;
    for (int i = 2; i <= b; i++)
        down = down * i % mod;
    down = qmi(down, mod-2);
    
    for (int i = 0; i < n; i++)
        cin >> q[i];
    
    LL res = 0;
    // C(m+n-1, n-1) 也合并到同一个循环中, 当状态全为 0 的时候
    for (int i = 0; i < 1 << n; i++) {
        LL a = m + n - 1;
        int sgn = 1;
        for (int j = 0; j < n; j++) {
            if (i >> j & 1) {
                sgn *= -1;
                a -= q[j] + 1;
            }
        }
        res = (res + C(a) * sgn) % mod;
    }
    
    cout << (res + mod) % mod << endl;
    return 0;
}
```

## 破译密码

> 对于给定的整数 a,b 和 d，有多少正整数对 x,y，满足 x≤a，y≤b，并且 gcd(x,y)=d。
>
> 题目链接：[AcWing 215](https://www.acwing.com/problem/content/217/)。

首先等价变形为：
$$
\gcd(x,y)=1,x\le \lfloor\frac{a}{d}\rfloor,y\le \lfloor\frac{b}{d}\rfloor
$$
也就是找到范围内所有互质的数的个数，这就可以用容斥做了。所有两个数的组合是 $\lfloor a/d\rfloor\lfloor b/d\rfloor$，减去 $2,3,5,\cdots$ 的倍数的组合，然后加上 $2\times 3,3\times 5,2\times 5$ 的组合，等等。

所以最终的答案就会是：
$$
a'=\lfloor\frac{a}{d}\rfloor,b'=\lfloor\frac{b}{d}\rfloor\\
\sum_{i=1}^{\min(a',b')} \mu(i)\lfloor \frac{a'}{i}\rfloor \lfloor \frac{b'}{i}\rfloor
$$
其中 $\mu(x)$ 表示莫比乌斯函数，它的取值是这样：
$$
x=\prod_{i=1}^k p_i^{c_i}\\
\mu(x)=\left \{
\begin{aligned}
&0,c_i\ge 2\\
&1,x=1\\
&(-1)^k,x\ge 2\\
\end{aligned}
\right.
$$
它可以在线筛的同时求出来。但直接两重 `for` 会超时。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 50010;
int mu[N], primes[N], cnt;
bool st[N];

void init(int n) {
    mu[1] = 1;
    for (int i = 2; i <= n; i++) {
        if (!st[i]) {
            primes[cnt++] = i;
            mu[i] = -1;
        }
        for (int j = 0; primes[j] <= n/i; j++) {
            st[i*primes[j]] = true;
            if (i % primes[j] == 0) {
                break;
            }
            mu[i*primes[j]] = -mu[i];
        }
    }
}

int main() {
    int T, a, b, d;
    init(N-1);
    scanf("%d", &T);
    while (T--) {
        scanf("%d%d%d", &a, &b, &d);
        a /= d, b /= d;
        LL res = 0;
        for (int i = 1; i <= a && i <= b; i++) {
            res += (a/i)*(b/i)*mu[i];
        }
        printf("%lld\n", res);
    }
    return 0;
}
```

有很多 `a/i, b/i` 的数值是一样的，这个大概是 $\sqrt n$ 的级别，因此可以用这个原理求和时分块。

$$
\lfloor \frac{n}{x}\rfloor=\lfloor \frac{n}{x+d}\rfloor\\
$$
设这个整除结果为 $k$，那么有：
$$
n=kx+r_1=k(x+d)+r_2\\
r_2=r_1-kd
$$
可以看出，$d$ 取最大值的时候 $r_2$ 最小，那么也就是右边等于 $r_1\bmod k$，即：
$$
\begin{aligned}
d&=\lfloor\frac{r_1}{k}\rfloor\\
&=\lfloor\frac{n-kx}{k}\rfloor\\
&=\lfloor \frac{n}{k}\rfloor-x\\
x+d&=\lfloor\frac{n}{k}\rfloor=\lfloor\frac{n}{\lfloor n/x\rfloor}\rfloor
\end{aligned}
$$
就是整除两次就跳转到当前块的右边界了。

```cpp
int smu[N];

int main() {
    int T, a, b, d;
    init(N-1);
    for (int i = 1; i < N; i++)
        smu[i] = smu[i-1] + mu[i];
    
    scanf("%d", &T);
    while (T--) {
        scanf("%d%d%d", &a, &b, &d);
        a /= d, b /= d;
        LL res = 0;
        for (int i = 1, j = -1; i <= a && i <= b; i = j+1) {
            j = min(a / (a / i), b / (b / i));
            // 因为在 i~j 段中 (a/i)*(b/i) 结果相同
            // 把这两项提出来里面只剩莫比乌斯函数的求和了
            res += (LL)(a/i)*(b/i)*(smu[j]-smu[i-1]);
        }
        printf("%lld\n", res);
    }
    return 0;
}
```

