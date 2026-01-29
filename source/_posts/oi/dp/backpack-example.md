---
date: 2023-07-05 17:48:23
updated: 2023-07-05 17:48:23
title: 动态规划 - 背包例题
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 机器分配

> 总公司拥有 M 台 **相同** 的高效设备，准备分给下属的 N 个分公司。
>
> 各分公司若获得这些设备，可以为国家提供一定的盈利。盈利与分配的设备数量有关。
>
> 问：如何分配这M台设备才能使国家得到的盈利最大？求出最大盈利值。
>
> 分配原则：每个公司有权获得任意数目的设备，但总台数不超过设备数 M。
>
> 链接：[AcWing 1013](https://www.acwing.com/problem/content/1015/)。

本题考查多重背包问题与求方案数，代码如下：

```cpp
#include <iostream>
#include <algorithm>
#include <stack>
using namespace std;

const int N = 20;
int f[N], w[N][N];
int g[N][N];

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            cin >> w[i][j];
    
    for (int i = 1; i <= n; i++)
        for (int j = m; j >= 1; j--)
            for (int k = 0; k <= m && j-k >= 0; k++)
                if (f[j] < f[j-k] + w[i][k]) {
                    f[j] = f[j-k]+w[i][k];
                    g[i][j] = k;
                }
    

    cout << f[m] << endl;
    int j = m;
    stack<int> stk;
    for (int i = n; i >= 1; i--) {
        int k = g[i][j];
        stk.push(k);
        // f(i, j) 从 f(i-1, j-k) 转移过来
        j -= k;
    }
    for (int i = 1; i <= n; i++) {
        cout << i << ' ' << stk.top() << endl;
        stk.pop();
    }
    return 0;
}
```

求解方案的时候只能倒序，因此这里先把具体数量存到栈中，然后输出。

也可以先求出方案然后逆推转移过程，如下：

```cpp
#include <iostream>
#include <algorithm>
#include <stack>
using namespace std;

const int N = 20;
int f[N][N], w[N][N];
int res[N];

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            cin >> w[i][j];
    
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            for (int k = 0; k <= m && j-k >= 0; k++)
                f[i][j] = max(f[i][j], f[i-1][j-k]+w[i][k]);
    
    int j = m;
    for (int i = n; i; i--)
        for (int k = 0; k <= m && j-k >= 0; k++)
            if (f[i][j] == f[i-1][j-k] + w[i][k]) {
                res[i] = k;
                j -= k;
                // 如果在当前层中找到的方案，它的设备数量不应该减少，
                // 因此需要直接 break 找上一层中的方案。
                break;
            }

    cout << f[n][m] << endl;
    for (int i = 1; i <= n; i++) {
        cout << i << ' ' << res[i] << endl;
    }

    return 0;
}
```

## 整数的划分方案数

> 一个正整数 n 可以表示成若干个正整数之和，形如：n=n1+n2+...+nk, 其中 n1>=n2>=...>=nk, k>=1，我们将这样的一种表示称为正整数 n 的一种划分。现在给定一个正整数 n，请你求出 n 共有多少种不同的划分方法。
>
> 由于答案可能很大，输出结果请对 `1e9+7` 取模。

这可以看作一个**完全背包问题**和**背包问题中的求方案数**。

容量 `n` 的背包和容量 `1~n` 的物品，每个都不限次数的拿取，问**方案数**。注意这里是方案数而不是最大值最小值，所以它的状态表示略有差异，分析：

+ 状态表示 `f(i, j)`：对 `1~i` 的数字，容量为 `j` 的背包选择的方案数。
+ 状态转移：`f(i, j) = f(i-1, j) + f(i-1, j-i) + f(i-1, j-2i) + ... + f(i-1, j-ki)`，意思是从 `1~i-1` 的数字中选 `k` 个 `i` 的方案数累加起来。

+ 初始化：`f(i, 0) = 1` 因为一个都不选也是一种方案。

最原始的写法如下：

```cpp
#include <iostream>
using namespace std;

const int N = 1010, mod = 1e9+7;
int f[N][N];

int main() {
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) f[i][0] = 1;
    
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            for (int k = 0; k*i <= j; k++) {
                // 这里一定注意加括号，取模优先级比加法高，我被这个坑了快一个小时
                f[i][j] = (f[i][j] + f[i-1][j-k*i]) % mod;
            }
        }
    }
    
    cout << f[n][n] << endl;
    return 0;
}
```

做到这个地步显然不太够，从状态转移方程入手，可以发现 `f(i, j-i) = f(i-1, j-1) + f(i-1, j-2i) + ... + f(i-1, j-ki)`，因此的到最终的**状态转移方程**： `f(i, j) = f(i-1, j) + f(i, j-i)`，当然要求 `j >= i`。

所以，优化后的代码如下：

```cpp
#include <iostream>
using namespace std;

const int N = 1010, mod = 1e9+7;
int f[N][N];

int main() {
    int n;
    cin >> n;
    for (int i = 0; i <= n; i++) f[i][0] = 1;
    
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            if (j >= i) f[i][j] = (f[i-1][j] + f[i][j-i]) % mod;
            else f[i][j] = f[i-1][j] % mod;
        }
    }
    
    cout << f[n][n] << endl;
    return 0;
}
```

还可以进一步优化，去掉 `f` 的第一个维度，得到最终版本的代码：

```cpp
#include <iostream>
using namespace std;

const int N = 1010, mod = 1e9+7;
int f[N];

int main() {
    int n;
    cin >> n;

    f[0] = 1;
    for (int i = 1; i <= n; i++) {
        for (int j = i; j <= n; j++) {
            f[j] = (f[j] + f[j-i]) % mod;
        }
    }
    
    cout << f[n] << endl;
    return 0;
}
```

## 货币系统

> 给你一个n种面值的货币系统，求组成面值为m的货币有多少种方案。
>
> 链接：[AcWing 1021](https://www.acwing.com/problem/content/1023/)。

完全背包问题，一维的时候从小到大循环。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 20, M = 3010;

long long f[M];

int main() {
    int n, m;
    cin >> n >> m;
    // f(0, 0) 什么都不选也是一种方案
    f[0] = 1;
    for (int i = 1; i <= n; i++) {
        int a;
        cin >> a;
        for (int j = a; j <= m; j++) {
            f[j] += f[j-a];
        }
    }
    cout << f[m] << endl;
    
    return 0;
}
```

## [NOIP2018] 货币系统

> 某国度中共有 n 种不同面额的货币，第 i 种货币的面额为 a[i]，你可以假设每一种货币都有无穷多张。
>
> 为了方便，我们把货币种数为 n、面额数组为 a[1..n] 的货币系统记作 (n,a)。 
>
> 在一个完善的货币系统中，每一个非负整数的金额 x 都应该可以被表示出，即对每一个非负整数 x，都存在 n 个非负整数 t[i] 满足 a[i]*t[i] 的和为 x。
>
> 然而，在网友的国度中，货币系统可能是不完善的，即可能存在金额 x 不能被该货币系统表示出。例如在货币系统 n=3,a=[2,5,9] 中，金额 1,3 就无法被表示出来。 
>
> 两个货币系统 (n,a) 和 (m,b) 是等价的，当且仅当对于任意非负整数 x，它要么均可以被两个货币系统表出，要么不能被其中任何一个表示出。 
>
> 他们希望找到一个货币系统 (m,b)，满足 (m,b) 与原来的货币系统 (n,a) 等价，且 m 尽可能的小，输出最小的 m。
>
> 链接：[AcWing 532](https://www.acwing.com/problem/content/534/)。

分析出下面的性质：

1. 集合 $A$ 中任意一个元素 $a_i$ 都应该能被集合 $B$ 中的任意几个元素以 $\sum w_jb_j$ 的形式表示。此性质保证 $A$ 能表达出来的所有数字 $B$ 也能表达。

2. 集合 $B$ 中任意元素 $b_i$ 不可被**除其本身外**的任何一组元素表示。如果不满足此性质一定**不是最优解**，但满足此性质**不一定是最优解**。

3. 集合 $B$ 满足 $B \sube A$，下面证明：

    引理：集合 $B$ 中的任意一个元素 $b_i$ 也能被集合 $A$ 中的任意几个元素以 $\sum w_ja_j$ 的形式表示。此性质保证 $A$ 不能表达的数字 $B$ 也不能表达。

    假设存在 $b_0 \notin A$，那么由引理它一定可以由 $A$ 中元素表示，而 $A$ 中元素可以用 $B$ 中元素表示，所以 $b_0$ 可以用 $B$ 中元素表示，此时不是最优解。

所以重点就是看集合 $A$ 中的某个数能否被其它数表示，这里就用到的完全背包求方案数了，如果能被表示一定不选，不能表示就一定选。由于 $a_i$ 一定只能被比它更小的数表示，所以这里先排序再判断方案数是否为 0。

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int M = 25010, N = 110;
int a[N], f[M];

int solve() {
    int n, res = 0;
    cin >> n;
    for (int i = 1; i <= n; i++) cin >> a[i];
    sort(a+1, a+1+n);
    
    memset(f, 0, sizeof f);
    f[0] = 1;
    
    for (int i = 1; i <= n; i++) {
        if (!f[a[i]]) res++;
        for (int j = a[i]; j <= a[n]; j++) {
            f[j] += f[j-a[i]];
        }
    }
    
    return res;
}

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int T; cin >> T;
    while (T--) {
        cout << solve() << endl;
    }
    return 0;
}
```

## [NOIP2006] 金明的预算方案

> 他把想买的物品分为两类：主件与附件，附件是从属于某个主件的，下表就是一些主件与附件的例子：
>
> |  主件  |      附件      |
> | :----: | :------------: |
> |  电脑  | 打印机，扫描仪 |
> |  书柜  |      图书      |
> |  书桌  |   台灯，文具   |
> | 工作椅 |       无       |
>
> 如果要买归类为附件的物品，必须先买该附件所属的主件。每个主件可以有 $0$ 个、$1$ 个或 $2$ 个附件。每个附件对应一个主件，附件不再有从属于自己的附件。金明想买的东西很多，肯定会超过妈妈限定的 $n$ 元。于是，他把每件物品规定了一个重要度，分为 $5$ 等：用整数 $1 \sim 5$ 表示，第 $5$ 等最重要。他还从因特网上查到了每件物品的价格（都是 $10$ 元的整数倍）。他希望在不超过 $n$ 元的前提下，使每件物品的价格与重要度的乘积的总和最大。
>
> 设第 $j$ 件物品的价格为 $v_j$，重要度为$w_j$，共选中了 $k$ 件物品，编号依次为 $j_1,j_2,\dots,j_k$，则所求的总和为：
>
> $v_{j_1} \times w_{j_1}+v_{j_2} \times w_{j_2}+ \dots +v_{j_k} \times w_{j_k}$。
>
> 请你帮助金明设计一个满足要求的购物单。
>
> 输入第一行有两个整数，分别表示总钱数 $n$ 和希望购买的物品个数 $m$。
>
> 第 $2$ 到第 $(m + 1)$ 行，每行三个整数，第 $(i + 1)$ 行的整数 $v_i$，$p_i$，$q_i$ 分别表示第 $i$ 件物品的价格、重要度以及它对应的的主件。如果 $q_i=0$，表示该物品本身是主件。
>
> 输出一行一个整数表示答案。
>
> 对于全部的测试点，保证 $1 \leq n \leq 3.2 \times 10^4$，$1 \leq m \leq 60$，$0 \leq v_i \leq 10^4$，$1 \leq p_i \leq 5$，$0 \leq q_i \leq m$，答案不超过 $2 \times 10^5$。

这个题虽然可以算是有依赖的背包问题，但是用树形 DP 妥妥超时，它有 60 节点 32000 体积，由于题目中说的主从关系较为简单，每个树的深度最大为 1，那么可以用二进制枚举然后当成分组背包问题求解。

+ 状态表示 $f(i,j)$：

    + 集合：前 $i$ 物品中体积不超过 $j$ 的所有方案。
    + 存储：最大值。

+ 状态转移方程：
    $$
    f(i,j)=\max\{f(i-1,j),f(i-1,j-v_i)+w_i\}
    $$
    其中 $v_i, w_i$ 表示同一组中的物品。

+ 复杂度：分组背包问题的复杂度其中组数 $s=2^k$，其中 $k$ 表示子节点数量，因此最终的复杂度是 $O(nm\cdot2^k)$，这里子节点数量最大为 2，比树形 DP 快了很多。

分组背包循环顺序：物品 -> 体积 -> 决策。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int M = 32010, N = 70;
int n, m;
int f[M];

struct Item {
    int v, w, q;
    vector<int> s;
} items[N];

int main() {
    cin >> m >> n;
    for (int i = 1, v, p, q; i <= n; i++) {
        Item& item = items[i];
        cin >> v >> p >> q;
        item.v = v, item.w = v * p, item.q = q;
        if (q) items[q].s.emplace_back(i);
    }
    
    
    for (int i = 1; i <= n; i++) {
        if (items[i].s.empty() && items[i].q) continue;
        for (int j = m; j; j--) {
            for (int state = 0; state < 1 << items[i].s.size(); state++) {
                int v = items[i].v, w = items[i].w;
                for (int k = 0; k < items[i].s.size(); k++) {
                    if (state >> k & 1) v += items[items[i].s[k]].v, w += items[items[i].s[k]].w;
                }
                
                if (j >= v) f[j] = max(f[j], f[j-v]+w);
            }
        }
    }
    
    cout << f[m] << '\n';
    return 0;
}
```

## [NOIP2014] 飞扬的小鸟

> 游戏界面是一个长为 $n$，高为 $m$ 的二维平面，其中有 $k$ 个管道（忽略管道的宽度）。 
>
> 小鸟始终在游戏界面内移动。小鸟从游戏界面最左边任意整数高度位置出发，到达游戏界面最右边时，游戏完成。
>
> 小鸟每个单位时间沿横坐标方向右移的距离为 $1$，竖直移动的距离由玩家控制。如果点击屏幕，小鸟就会上升一定高度 $x$，每个单位时间可以点击多次，效果叠加；如果不点击屏幕，小鸟就会下降一定高度 $y$。小鸟位于横坐标方向不同位置时，上升的高度 $x$ 和下降的高度 $y$ 可能互不相同。
>
> 小鸟高度等于 $0$ 或者小鸟碰到管道时，游戏失败。小鸟高度为 $m$ 时，无法再上升。
>
> 现在,请你判断是否可以完成游戏。如果可以，输出最少点击屏幕数；否则，输出小鸟最多可以通过多少个管道缝隙。
>
> 输入第 $1$ 行有 $3$ 个整数 $n, m, k$，分别表示游戏界面的长度，高度和水管的数量，每两个整数之间用一个空格隔开；
>
> 接下来的 $n$ 行，每行 $2$ 个用一个空格隔开的整数 $x$ 和 $y$，依次表示在横坐标位置 $0 \sim n-1$ 上玩家点击屏幕后，小鸟在下一位置上升的高度 $x$，以及在这个位置上玩家不点击屏幕时，小鸟在下一位置下降的高度 $y$。
>
> 接下来 $k$ 行，每行 $3$ 个整数 $p,l,h$，每两个整数之间用一个空格隔开。每行表示一个管道，其中 $p$ 表示管道的横坐标，$l$ 表示此管道缝隙的下边沿高度，$h$ 表示管道缝隙上边沿的高度（输入数据保证 $p$ 各不相同，但不保证按照大小顺序给出）。
>
> 输出共两行。
>
> 第一行，包含一个整数，如果可以成功完成游戏，则输出 $1$，否则输出 $0$。
>
> 第二行，包含一个整数，如果第一行为 $1$，则输出成功完成游戏需要最少点击屏幕数，否则，输出小鸟最多可以通过多少个管道缝隙。
>
> 对于 $100\%$ 数据：$5 \leq n \leq 10000$，$5 \leq m \leq 1000$，$0 \leq k < n$，$0 < x,y < m$，$0 < p < n$，$0 \leq l < h \leq m$， $l + 1 < h$。
>
> 题目链接：[P1941](https://www.luogu.com.cn/problem/P1941)，[AcWing 512](https://www.acwing.com/problem/content/514/)。

这里是用背包优化 DP 转移方程。

+ 状态表示 $f(i,j)$：到达坐标 $(i,j)$ 需要的最小操作步数。

+ 状态转移方程，不考虑边界情况：
    $$
    f(i,j)=\min\begin{cases}
    f(i-1,j+Y_{i-1})\\
    f(i-1,j-kX_{i-1})+k
    \end{cases}
    $$
    对于跳到顶部，这样转移，我一开始没有考虑多次点击的情况，也就是大括号下面的那个情况，获得了 60pts 的好成绩：
    $$
    f(i,m)=\min\begin{cases}
    f(i-1,m)+1\\
    f(i-1,j)+\lceil\frac{m-j}{X_{i-1}}\rceil
    \end{cases}
    $$

+ 初始化：$f(0,1\sim m)=0,f(i,j)=+\infin$

然后先写出一个洛谷给了 80pts 的代码：

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 10010, M = 1010, INF = 0x3f3f3f3f;
int n, m, k, X[N], Y[N], L[N], H[N];
int f[N][M];

void solve(int& type, int& res) {
    memset(f, 0x3f, sizeof f);
    for (int j = 1; j <= m; j++) f[0][j] = 0;
    
    int passed = 0;
    for (int i = 1; i <= n; i++) {
        // 可以跳到上边界
        if (H[i] > m) {
            f[i][m] = min(f[i][m], f[i-1][m]+1);
            for (int j = 1; j < m; j++)
                f[i][m] = min(f[i][m], f[i-1][j] + (m-j+X[i-1]-1)/X[i-1]);
        }
        
        bool success = false;
        for (int j = L[i]+1; j <= m && j < H[i]; j++) {
            // 跳上去
            for (int k = 1; k <= j/X[i-1]; k++)
                f[i][j] = min(f[i][j], f[i-1][j-k*X[i-1]]+k);
            
            // 降落下来
            if (j+Y[i-1] <= m)
                f[i][j] = min(f[i][j], f[i-1][j+Y[i-1]]);
            
            // 如果存在合法方案就可以成功通过当前点
            if (f[i][j] < INF) success = true;
        }
        
        // 通过了一个管道
        if (success && H[i] <= m) passed++;
        
        if (!success) return type = 0, res = passed, void();
        
    }
    
    res = INF;
    for (int j = 1; j <= m; j++) res = min(res, f[n][j]);
    type = 1;
}

int main() {
    memset(H, 0x3f, sizeof H);

    scanf("%d%d%d", &n, &m, &k);
    for (int i = 0; i < n; i++) scanf("%d%d", &X[i], &Y[i]);
    for (int i = 0, p; i < k; i++) scanf("%d", &p), scanf("%d%d", &L[p], &H[p]);
    
    int type, res;
    solve(type, res);
    return !printf("%d\n%d\n", type, res);
}
```



这里的性能瓶颈在于不考虑边界情况的转移中，我们需要在内部再遍历一遍 $k$ 的所有合法值，下面考虑专门建立一个 DP 用来优化这个地方。

观察 $f(i,j-X)$ 的值：
$$
\begin{aligned}
f(i,j)&=\min\begin{cases}
f(i-1,j+Y)\\
f(i-1,j-X)+1,f(i-1,j-2X)+2,f(i-1,j-3X)+3,\cdots
\end{cases}
\\
f(i,j-X)&=\min\begin{cases}
f(i-1,j+Y-X)\\
f(i-1,j-2X)+1,f(i-1,j-3X)+2,f(i-1,j-4X)+3,\cdots
\end{cases}
\end{aligned}
$$
如果我们把下面的看成 $g(i,j)$，那么有：
$$
g(i,j)=\min\{g(i,j-X)+1,f(i-1,j-X)+1\}
$$
这里注意我们要让 $j$ 枚举完 $1\sim m$ 的所有数字，所以需要把判断条件放在循环体内部。

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 10010, M = 1010, INF = 0x3f3f3f3f;
int n, m, k, X[N], Y[N], L[N], H[N];
int f[N][M], g[M];

void solve(int& type, int& res) {
    memset(f, 0x3f, sizeof f);
    for (int j = 1; j <= m; j++) f[0][j] = 0;
    
    int passed = 0;
    for (int i = 1; i <= n; i++) {
        // 可以跳到上边界
        if (H[i] > m) {
            f[i][m] = min(f[i][m], f[i-1][m]+1);
            for (int j = 1; j < m; j++)
                f[i][m] = min(f[i][m], f[i-1][j] + (m-j+X[i-1]-1)/X[i-1]);
        }
        
        bool success = false;
        memset(g, 0x3f, sizeof g);
        for (int j = 1; j <= m; j++) {
            if (j >= X[i-1]) g[j] = min(f[i-1][j-X[i-1]]+1, g[j-X[i-1]]+1);

            if (L[i] < j && j < H[i]) {
                // 上升
                f[i][j] = min(f[i][j], g[j]);
                // 降落
                if (j+Y[i-1] <= m) f[i][j] = min(f[i][j], f[i-1][j+Y[i-1]]);
            }
            
            // 如果存在合法方案就可以成功通过当前点
            if (f[i][j] < INF) success = true;
        }
        
        // 通过了一个管道
        if (success && H[i] <= m) passed++;
        
        if (!success) return type = 0, res = passed, void();
        
    }
    
    res = INF;
    for (int j = 1; j <= m; j++) res = min(res, f[n][j]);
    type = 1;
}

int main() {
    memset(H, 0x3f, sizeof H);

    scanf("%d%d%d", &n, &m, &k);
    for (int i = 0; i < n; i++) scanf("%d%d", &X[i], &Y[i]);
    for (int i = 0, p; i < k; i++) scanf("%d", &p), scanf("%d%d", &L[p], &H[p]);
    
    int type, res;
    solve(type, res);
    return !printf("%d\n%d\n", type, res);
}
```

## 能量石

> 太长了，这里直接给链接：[AcWing 734](https://www.acwing.com/problem/content/736/)。

### 贪心分析

在最优解中，任取两相邻点 $i$ 与 $i+1$，所获得的能量 $E_1$ 是：
$$
E_1=E_i+E_{i+1}-S_iL_{i+1}
$$
而如果先吃 $i+1$ 再吃 $i$，那么获得的能量 $E_2$ 是：
$$
E_2=E_i+E_{i+1}-S_{i+1}L_i
$$
因为是最优解，所以 $E_1\gt E_2$，那么可以得出
$$
S_iL_{i+1}<S_{i+1}L_i
$$
递推可以证明每个相邻的元素都满足这个条件。这个式子也是有实际意义的，由于 $L$ 是流失速度，两边同除 $S_iS_{i+1}$ 就是流失加速度。

### DP 分析

+ 状态表示 $f(i, j)$：

    + 集合：前 $i$ 个石头中恰好用时为 $j$ 的所有方案。

        注意这里的 $j$ 含义是时间恰好为 $j$，因为每个状态都是从上一个**刚结束的时刻**转移过来的。

        那么最优解是 $\max\{f(n, k)\}, k\in[1, m]$，初始化时所有状态均为 $-\inf$，除了 $f(i,0)=0,i\in [1,n]$，因为时间为 0 时的最优解一定是 0。

    + 存储：最大值。

+  状态转移：

    假设当前时刻为 $j$，如果已经吃了石头 $E_i$ 那么含有的能量为：
    $$
    E_0=E_i-(j-S)L_i
    $$
    这里的时刻 $j$ 不能是开始吃某个石头的时刻，而是结束的时刻，这样的话可以保证计算出这个状态**一定可以被转移**，如果不这样的话就无从得知是否能转移了。

    因此状态转移方程为：

$$
f(i,j)=\max\{f(i-1,j), f(i-1,j-S)+E-(j-S)L\}
$$

### 代码

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int M = 10010, N = 110;
struct Stone {
    int s, e, l;
    bool operator<(const Stone& a) const {
        return s*a.l < a.s*l;
    }
} stones[N];
int f[M];

int main() {
    int T;
    cin >> T;
    for (int C = 1; C <= T; C++) {
        int n, m = 0;
        cin >> n;
        for (int i = 0; i < n; i++) {
            int s, e, l;
            cin >> s >> e >> l;
            stones[i] = {s, e, l};
            m += s;
        }
        sort(stones, stones+n);
        
        memset(f, -0x3f, sizeof f);
        f[0] = 0;
        
        for (int i = 0; i < n; i++) {
            int s = stones[i].s, l = stones[i].l, e = stones[i].e;
            for (int j = m; j >= s; j--)
                f[j] = max(f[j], f[j-s]+e-(j-s)*l);
        }
        
        int res = 0;
        for (int i = 0; i <= m; i++) res = max(res, f[i]);
        printf("Case #%d: %d\n", C, res);
    }
    return 0;
}
```

