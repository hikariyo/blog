---
date: 2023-08-16 16:10:00
updated: 2023-08-16 16:10:00
title: 动态规划 - 斜率优化
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 任务安排

> 有 N 个任务排成一个序列在一台机器上等待执行，它们的顺序不得改变。
>
> 机器会把这 N 个任务分成若干批，每一批包含连续的若干个任务。
>
> 从时刻 0 开始，任务被分批加工，执行第 i 个任务所需的时间是 Ti。
>
> 另外，在每批任务开始前，机器需要 S 的启动时间，故执行一批任务所需的时间是启动时间 S 加上每个任务所需时间之和。
>
> 一个任务执行后，将在机器中稍作等待，直至该批任务全部执行完毕。
>
> 也就是说，同一批任务将在同一时刻完成。
>
> 每个任务的费用是它的完成时刻乘以一个费用系数 Ci。
>
> 请为机器规划一个分组方案，使得总费用最小。
>
> 题目链接：[AcWing 300](https://www.acwing.com/problem/content/302/)，[AcWing 301](https://www.acwing.com/problem/content/303/)，[AcWing 302](https://www.acwing.com/problem/content/304/)。

### 暴力

暂时先不进行优化，这里分析状态转移方程。

+ 状态表示 $f(i)$：前 $i$ 个任务所需要的最小花费。

+ 状态转移方程：

    首先考虑时间间隔 $s$，它是具有**后效性**的，而我们显然无法很方便地处理它，所以应该把它直接算到当前的结果中：考虑在第 $i$ 任务执行前需要时间间隔，那么所有后续的任务都会受到影响，它们受到的总共影响是：
    $$
    s\sum_{j=i}^n c_j
    $$
    令 $j=0\sim i-1$ 是前一组的末尾，也就是我们当前组是 $j+1\sim i$，那么可以写出转移方程：
    $$
    f(i)=\min_{j=0}^{i-1}\{ f(j)+(\sum_{k=1}^i t_k)(\sum_{k=j+1}^i c_k)+s\sum_{k=j+1}^n c_k \}
    $$
    
+ 初始化：$f(0)=0, f(x)=\inf$

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 5050;
int n, s;
LL T[N], C[N], f[N];

int main() {
    scanf("%d%d", &n, &s);
    for (int i = 1; i <= n; i++) {
        scanf("%lld%lld", &T[i], &C[i]);
        T[i] += T[i-1];
        C[i] += C[i-1];
    }
    memset(f, 0x3f, sizeof f);
    f[0] = 0;
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < i; j++) {
            f[i] = min(f[i], f[j] + T[i]*(C[i]-C[j]) + s*(C[n]-C[j]));
        }
    }
    printf("%lld\n", f[n]);
    return 0;
}
```

### 斜率优化 I

接下来考虑进行优化，对状态转移方程进行变形，这里省略前面的 min 了，同时用大写表示对应元素的前缀和。
$$
\begin{aligned}
f(i)&=f(j)+(\sum_{k=1}^i t_k)(\sum_{k=j+1}^i c_k)+s\sum_{k=j+1}^n c_k\\
&=f(j)+T_i(C_i-C_j)+s(C_n-C_j)\\
&=f(j)-(T_i+s)C_j+T_iC_i+sC_n
\end{aligned}
$$
令 $y=f(j),x=C_j$ 可以整理出下列方程：
$$
y=(T_i+s)x+f(i)-T_iC_i-sC_n
$$
对于每一个 $i$，斜率是大于零的常量（第二题）$x=C_j$ 是单调递增的，斜率当 $i$ 变大时也是单调递增的。而 $j$ 是严格小于 $i$ 的所有数字，因此用单调队列维护即可。

+ 在查询时，直接把凸包上斜率小于当前直线的边删掉。
+ 在插入时，在队尾把在凸包内部的点删掉。

这里注意比较斜率时尽可能保证分子分母为正数。

```cpp
#include <cstdio>
using namespace std;

typedef long long LL;
const int N = 300010;
int q[N], n, s;
LL f[N], T[N], C[N];

int main() {
    scanf("%d%d", &n, &s);
    for (int i = 1; i <= n; i++) {
        scanf("%lld%lld", &T[i], &C[i]);
        C[i] += C[i-1], T[i] += T[i-1];
    }
    int hh = 0, tt = 0;
    for (int i = 1; i <= n; i++) {
        while (hh < tt && f[q[hh+1]] - f[q[hh]] <= 
               (T[i] + s) * (C[q[hh+1]] - C[q[hh]])) hh++;
        int j = q[hh];
        f[i] = f[j] - (T[i] + s) * C[j] + T[i]*C[i] + s*C[n];
        while (hh < tt && (f[q[tt]] - f[q[tt-1]]) * (C[i] - C[q[tt]]) >=
               (f[i] - f[q[tt]]) * (C[q[tt]] - C[q[tt-1]])) tt--;
        q[++tt] = i;
    }
    printf("%lld\n", f[n]);
    return 0;
}
```

### 斜率优化 II

如果无法保证斜率是单调的，但是此时横坐标仍然是单调增的，那么：

+ 查询时二分查找比当前斜率大的第一个数，类似 `upper_bound` 的操作。
+ 插入时直接把队尾在凸包内的点删掉。

然后要关注斜率 $k=\Delta y/\Delta x$ 我们只需要保证 $\Delta x$ 是正的，两边交叉相乘之后不等号方向不变。

```cpp
#include <cstdio>
#include <cctype>
using namespace std;

typedef __int128 LL;
const int N = 300010;
int q[N], n, s;
LL f[N], T[N], C[N];

void read(LL& x) {
    x = 0;
    int sgn = 1;
    char ch = getchar();
    while (!isdigit(ch)) {
        if (ch == '-') sgn = -1;
        ch = getchar();
    }
    while (isdigit(ch)) {
        x = x * 10 + (ch ^ 48);
        ch = getchar();
    }
    x *= sgn;
}

void _write(LL x) {
    if (x < 0) x = -x, putchar('-');
    if (!x) return;
    _write(x/10);
    putchar('0' + x%10);
}

void write(LL x) {
    if (!x) putchar('0');
    else _write(x);
}

int main() {
    scanf("%d%d", &n, &s);
    for (int i = 1; i <= n; i++) {
        read(T[i]), read(C[i]);
        C[i] += C[i-1], T[i] += T[i-1];
    }
    int hh = 0, tt = 0;
    for (int i = 1; i <= n; i++) {
        int l = hh, r = tt;
        while (l < r) {
            int m = l+r >> 1;
            // k(m, m+1)
            if (f[q[m+1]] - f[q[m]] > (T[i] + s) * (C[q[m+1]] - C[q[m]])) r = m;
            else l = m+1;
        }
        int j = q[r];
        f[i] = f[j] - (T[i] + s) * C[j] + T[i]*C[i] + s*C[n];
        while (hh < tt && (f[q[tt]] - f[q[tt-1]]) * (C[i] - C[q[tt]]) >=
               (f[i] - f[q[tt]]) * (C[q[tt]] - C[q[tt-1]])) tt--;
        q[++tt] = i;
    }
    write(f[n]);
    return 0;
}
```

## [CF311B] Cats Transport

> 小 S 是农场主，他养了 M 只猫，雇了 P 位饲养员。
>
> 农场中有一条笔直的路，路边有 N 座山，从 1 到 N 编号。
>
> 第 i 座山与第 i−1 座山之间的距离为 Di。
>
> 饲养员都住在 1 号山。
>
> 有一天，猫出去玩。
>
> 第 i 只猫去 Hi 号山玩，玩到时刻 Ti 停止，然后在原地等饲养员来接。
>
> 饲养员们必须回收所有的猫。
>
> 每个饲养员沿着路从 1 号山走到 N 号山，把各座山上已经在等待的猫全部接走。
>
> 饲养员在路上行走需要时间，速度为 1 米/单位时间。
>
> 饲养员在每座山上接猫的时间可以忽略，可以携带的猫的数量为无穷大。
>
> 例如有两座相距为 1 的山，一只猫在 2 号山玩，玩到时刻 3 开始等待。
>
> 如果饲养员从 1 号山在时刻 2 或 3 出发，那么他可以接到猫，猫的等待时间为 0 或 1。
>
> 而如果他于时刻 1 出发，那么他将于时刻 2 经过 2 号山，不能接到当时仍在玩的猫。
>
> 你的任务是规划每个饲养员从 1 号山出发的时间，使得所有猫等待时间的总和尽量小。
>
> 饲养员出发的时间可以为负。
>
> 题目链接：[CF311B](https://codeforces.com/contest/311/problem/B)，[AcWing 303](https://www.acwing.com/problem/content/305/)。

首先用 $D_i$ 表示原含义的前缀和，即 $1\sim i$ 的距离。

那么对于去第 $i$ 只小猫，如果从 $t_0$ 时刻出发恰好接到，那么有 $t_0+D(H_i)=T_i$ 即 $t_0=T_i-D(H_i)$，令 $A_i=T_i-D(H_i)$，那么由 $t_0$ 时刻出发需要让小猫的等待时间就是 $t_0-A_i$，这样一来我们就不关心它到底去哪里了，只关心何时出发即可。

按照 $A_i$ 升序排好，有如下序列：
$$
A_1,A_2,A_3,A_4,\cdots,A_n
$$
若 $t_0=A_3$，那么等待时间就是 $A_3-A_3+A_3-A_2+A_3-A_1=3A_3-\sum_{i=1}^3 A_i$

拓展到区间 $[L,R]$ 需要的花费是：
$$
w(L,R)=(R-L+1)A_R-\sum_{i=L}^R A_i
$$
因而定义状态 $f(j,i)$ 表示前 $i$ 只小猫在 $j$ 个饲养员的情况下的最小花费，这个状态定义的顺序是由于我们计算第 $j$ 层状态需要依赖第 $j-1$ 层状态，把依赖想要放到前面比较利于程序运行的缓存机制。
$$
f(j,i)=\min_{k=0}^{i-1}\{ f(j-1,k) + w(k+1,i) \}
$$

好，首先先打一遍暴力。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 100010, P = 110;
int n, m, p;
LL D[N], A[N], S[N], f[P][N];

int main() {
    scanf("%d%d%d", &n, &m, &p);
    for (int i = 2; i <= n; i++) {
        scanf("%lld", &D[i]);
        D[i] += D[i-1];
    }
    
    for (int i = 1, h, t; i <= m; i++) {
        scanf("%d%d", &h, &t);
        A[i] = t - D[h];
    }
    
    sort(A+1, A+1+m);
    for (int i = 1; i <= m; i++) S[i] = S[i-1] + A[i];
    
    memset(f, 0x3f, sizeof f);
    for (int j = 0; j <= p; j++) f[j][0] = 0;
    for (int j = 1; j <= p; j++) {
        for (int i = 1; i <= m; i++) {
            for (int k = 0; k < i; k++) {
                f[j][i] = min(f[j][i], f[j-1][k] + A[i]*(i-k)-S[i]+S[k]);
            }
        }
    }
    
    printf("%lld\n", f[p][m]);
    
    return 0;
}
```

这肯定是 TLE 的，接下来考虑进行斜率优化。
$$
\begin{aligned}
f(j,i)&=f(j-1,k) + w(k+1,i)\\
&=f(j-1,k)+A_i(i-k)-(S_i-S_k)\\
&=f(j-1,k)+S_k-A_ik+A_ii-S_i
\end{aligned}
$$
直接令 $x=k,y=f(j-1,k)+S_k$ 就有：
$$
y=A_ix+f(j,i)-A_ii+S_i
$$
由于 $A_i$ 是升序的，因此斜率是不断增加的，并且 $x$ 也是不断递增的，所以用单调队列维护。

+ 当查找元素时，删除所有凸包上斜率小于等于当前直线的直线。
+ 当插入元素时，删除所有凸包上斜率大于等于当前点和倒数第一个点连线斜率的点。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 100010, P = 110;
int n, m, p;
LL D[N], A[N], S[N], f[P][N];
int q[N];

int main() {
    scanf("%d%d%d", &n, &m, &p);
    for (int i = 2; i <= n; i++) {
        scanf("%lld", &D[i]);
        D[i] += D[i-1];
    }
    
    for (int i = 1; i <= m; i++) {
        LL h, t;
        scanf("%lld%lld", &h, &t);
        A[i] = t - D[h];
    }
    
    sort(A+1, A+1+m);
    for (int i = 1; i <= m; i++) S[i] = S[i-1] + A[i];
    
    memset(f, 0x3f, sizeof f);
    for (int j = 0; j <= p; j++) f[j][0] = 0;
    
    for (int j = 1; j <= p; j++) {
        int tt = 0, hh = 0;
        q[0] = 0;
        for (int i = 1; i <= m; i++) {
            #define y(k) (f[j-1][k] + S[k])
            while (hh < tt && y(q[hh+1]) - y(q[hh]) <= A[i]*(q[hh+1]-q[hh])) hh++;
            int k = q[hh];
            f[j][i] = f[j-1][k] - A[i]*k + A[i] * i -S[i] + S[k];
            // 虽然这样写比较丑 但是不容易括号出错
            while (hh < tt && 
                (y(q[tt]) - y(q[tt-1])) * 
                (i - q[tt]) >= 
                (y(i)-y(q[tt])) * 
                (q[tt] - q[tt-1])
            ) tt--;

            q[ ++ tt] = i;
        }
    }

    printf("%lld\n", f[p][m]);
    
    return 0;
}
```

## 一般化

如果斜率不满足单调性质，可以使用李超线段树解决，本站也有关于李超树的文章。
