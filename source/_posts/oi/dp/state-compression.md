---
date: 2023-06-19 12:20:13
update: 2023-09-25 19:14:00
title: 动态规划 - 状态压缩
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 分割棋盘

> 求把 N x M 的棋盘分成若干个 1 x 2 的长方形有几种方案。
>
> 题目链接：[AcWing 291](https://www.acwing.com/problem/content/293/)。

把某种状态以一个二进制数表示出来就是状态压缩，这里用一个二进制数储存在 `n` 行中有哪些长方形伸了出来，这里设 `n` 行 `m` 列，分析如下：

+ 状态表示 `f[i, j]`：
  + 含义：第 `i` 列中从 `i-1` 列中伸出来了状态为 `j` 的横着放的长方形的方案。由于剩下的位置只有竖着放长方形这一种方案，由**乘法原理**可以知道 `f[i, j]` 表示的就是在 `0 ~ i-1` 列中的所有摆放方案。
  + 表示：所有这样放的方案数。
+ 状态转移：`f[i, j] = sum(f[i-1, k], ...)`，当 `j` 和 `k` 满足不叠放、留下的空位可以容纳竖着放的长方形时即可。合法性检验方式：`(k & j) == 0` 并且 `k | j` 留下的 0 中不能有奇数个连续，否则放不下竖着的长方形。
+ 初始状态：`f[0, 0] = 1`，表示第 `-1` 列到第 `0` 中只能伸出来 `0` 个，故只有这一种方案，所以是 `1`。
+ 输出：`f[m, 0]`，表示 `0~m-1` 列中的所有方案数。

观察状态转移方程，计算 `f[i, j]` 时所依赖的 `f[i-1, k]` 已经被计算出来，所以先遍历列 `i` 再遍历状态 `j`。

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 12, M = 1 << N;
long long f[N][M];
bool valid[M];

int main() {
    int n, m;
    while (cin >> n >> m, n || m) {
        memset(f, 0, sizeof f);
        // f[0][j] 当 j != 0 时是不存在的 因为-1列不可能向 0 列伸出来任何长方形
        f[0][0] = 1;
        
        // 预处理 valid 数组使奇数个 0 为非法
        for (int i = 0; i < (1 << n); i++) {
            valid[i] = true;
            int cnt = 0;
            for (int j = 0; j < n; j++) {
                // 位运算最好全都加上括号 因为位运算的优先级比较低
                if (((i >> j) & 1) == 0) cnt++;
                else {
                    // cnt 为奇数
                    if (cnt & 1) {
                        valid[i] = false;
                        break;
                    }
                    // 清空 cnt
                    cnt = 0;
                }
            }
            if (cnt & 1) valid[i] = false;
        }
        
        // 这里遍历的是列
        for (int i = 1; i <= m; i++) {
            // 里面遍历的是每一列的方案
            for (int j = 0; j < (1 << n); j++) {
                // 对任意一个状态 j 枚举出所有满足条件的状态
                for (int k = 0; k < (1 << n); k++) {
                    if ((j & k) == 0 && valid[j | k]) {
                        // 当 j 和 k 满足条件时 就是第 i-1 列中的一种放法
                        // 所以 i-1 列中的所有放法就是对这些求和
                        // 注意 f[i][j] 表示的是 0 ~ i-1 列中的方法 j 表示的是伸出来的长方形的状态
                        f[i][j] += f[i-1][k];
                    }
                }
            }
        }
        
        // 0 ~ m-1 列伸出来 0 个的放法就是答案
        cout << f[m][0] << endl;
    }
    return 0;
}
```

观察到最内层循环的判断条件和 `i` 无关，因此可以进一步优化，将每个方案都预处理出来。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <cstring>
using namespace std;

const int N = 12, M = 1 << 12;
vector<int> state[M];
bool valid[M];
long long f[N][M];

int main() {
    int n, m;
    while (cin >> n >> m, n || m) {
        memset(f, 0, sizeof f);
        f[0][0] = 1;
        
        for (int i = 0; i < (1 << n); i++) {
            valid[i] = true;
            int cnt = 0;
            for (int j = 0; j < n; j++) {
                if (((i >> j) & 1) == 0) cnt++;
                else if (cnt & 1) {
                    valid[i] = false;
                    break;
                }
                else cnt = 0;
            }
            
            if (cnt & 1) valid[i] = false;
        }
        
        for (int j = 0; j < (1 << n); j++) {
            state[j].clear();
            for (int k = 0; k < (1 << n); k++) {
                if ((j & k) == 0 && valid[j | k]) {
                    state[j].push_back(k);
                }
            }
        }
        
        for (int i = 1; i <= m; i++) {
            for (int j = 0; j < (1 << n); j++) {
                for (int k : state[j]) f[i][j] += f[i-1][k];
            }
        }
        
        cout << f[m][0] << endl;
    }

    return 0;
}
```

## [SCOI2005] 小国王

> 在 n*n 的棋盘上放 k 个国王，国王可攻击相邻的 8 个格子，求使它们无法互相攻击的方案总数。
>
> 题目链接：[AcWing 1064](https://www.acwing.com/problem/content/1066/)。

DP 分析如下：

+ 状态表示 $f(i,j,s)$：

    + 含义：在前 $i$ 行放 $j$ 个国王且第 $i$ 行状态为 $s$ 的所有方案。
    + 存储：方案数。

+ 状态转移：要在当前行以 $s_2$ 状态摆放，上一行状态为 $s_1$，那么方程就是：
    $$
    f(i,j,s_2) = \sum f(i-1, j-\text{cnt } s_2, s_1)
    $$
    方案数一般都要 `long long`。

+ 初始化：第 0 行摆 0 个显然也是一种方案，$f(0,0,0)=1$

+ 复杂度：遍历所有行，所有摆放个数，所有方案，所以是 $O(nm\cdot 2^n)$

+ 状态合法性检验：一行不能摆两个相邻的国王，所以 $\text{check } s = s \& (s\gg 1) = 0$

+ 状态合法转移检验：两行相同的位置不能摆放，相邻的位置也不能摆放，因此 $s_1 \& s_2 =0,\text{check } s_1|s_2 = 0$

先预处理出来所有合法状态以及他们能转移到哪些状态。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

const int N = 11, M = 1 << N;
long long f[N][N*N][M];
int n, m, cnt[M];
vector<int> ne[M];
vector<int> states;

bool check(int s) {
    return (s & (s >> 1)) == 0;
}

int count(int s) {
    int res = 0;
    for (int i = 0; i < n; i ++) {
        res += s >> i & 1;
    }
    return res;
}

int main() {
    cin >> n >> m;
    for (int s = 0; s < 1 << n; s++) {
        if (check(s)) {
            cnt[s] = count(s);
            states.push_back(s);
        }
    }
    
    for (int s1 : states) {
        for (int s2 : states) {
            if ((s1 & s2) == 0 && check(s1 | s2)) {
                ne[s1].push_back(s2);
            }
        }
    }
    
    f[0][0][0] = 1;
    for (int i = 1; i <= n; i++) {
        // 从 0 开始统计方案数
        for (int j = 0; j <= m; j++) {
            for (int s1 : states) {
                for (int s2 : ne[s1]) {
                    if (j >= cnt[s2]) f[i][j][s2] += f[i-1][j-cnt[s2]][s1];
                }
            }
        }
    }
    
    long long ans = 0;
    for (int s : states) ans += f[n][m][s];
    cout << ans;

    return 0;
}
```

## 最短 Hamilton 路径

> 给定一张 n 个点的带权无向图，点从 0∼n−1 标号，求起点 0 到终点 n−1 的最短 Hamilton 路径。
>
> Hamilton 路径的定义是从 0 到 n−1 不重不漏地经过每个点恰好一次。[原题链接](https://www.acwing.com/problem/content/description/93/)。

本题不能用 Dijkstra 求最短路或者用 Prim 求最小生成树，因为这个题目要求是必须经过所有点。

+ 状态表示 $f(i,j)$：
  + 含义：从 `0~j` 路径表示为 $i$ 的所有路径的集合。
  + 存储：这些路径的最小值。
+ 状态转移：$f(i,j)=\min\{f(i,j),f(i\text{ remove }j, k)+w_k\}$
+ 初始化：$f(i,j)=+\infin, f(1,0)=0$：从初始点到第0个点经过的路径为第0个点，也就是表示为 $1_2$ 的最短路径为0，就是一步都没走，其它都是无穷大。

这里和上一题不一样的地方在于在外层存状态，内层存点，这是由于 DP 问题都是由递推得来，我们要保证一个状态计算时**所依赖的状态已经被计算出来**。观察状态转移方程， `i remove j` 一定比 `i` 小，而当遍历 `f[i][j]` 的时候 `f[i remove j]` 这一层一定已经被算出来了，所以用这样的顺序遍历。

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 21, M = 1 << N;
int f[M][N], w[N][N];

int main() {
    int n;
    cin >> n;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            cin >> w[i][j];

    memset(f, 0x3f, sizeof f);
    f[1][0] = 0;
    for (int i = 0; i < (1 << n); i++) {
        for (int j = 0; j < n; j++) {
            // j in path i
            if ((i >> j) & 1) {
                for (int k = 0; k < n; k++) {
                    // k in (i remove j)
                    if (((i - (1 << j)) >> k) & 1) {
                        f[i][j] = min(f[i][j], f[i - (1 << j)][k] + w[k][j]);
                    }
                }
            }
        }
    }
    
    // 注意这里我们从0开始遍历，所以最终答案在 n-1 而不是 n
    cout << f[(1 << n)-1][n-1] << endl;
    
    return 0;
}
```

这些位运算比较容易做错，还多注意。

## [NOI2001] 炮兵阵地

> 司令部的将军们打算在 $N×M,N\le 100,M\le 10$ 的网格地图上部署他们的炮兵部队。
>
> 一个 N×M 的地图由 N 行 M 列组成，地图的每一格可能是山地（用 `H` 表示），也可能是平原（用 `P` 表示）。
>
> 在每一格平原地形上最多可以布置一支炮兵部队（山地上不能够部署炮兵部队）。
>
> 每个炮兵部队能够攻击到的区域：沿横向左右各两格，沿纵向上下各两格。
>
> 求整个地图中最多放多少个炮兵部队。
>
> 题目链接：[AcWing 292](https://www.acwing.com/problem/content/294/)，[P2704](https://www.luogu.com.cn/problem/P2704)。

本题空间需要使用滚动数组优化。

+ 状态表示 $f(i,j,k)$：
    + 含义：第 $i$ 列状态为 $j$，前一列状态为 $k$ 的所有摆放方案。
    + 存储：最大值。
+ 状态转移：$f(i,j,k)=\max\{ f(i-1,k,u)+\text{cnt}(j)\}$，其中 $u\to k,k\to j$ 都是可以转移的。
+ 转移合法性检验：$u\&k=0,k\&j=0$ 且 $u,k,j$ 本身合法，也就是同一行中不存在相邻和相隔为 1 的摆放。
+ 初始化：全是 0，因为最大值初始化就是 0
+ 复杂度：$O(n2^{3m})$，其中枚举 $u,k,j$ 都需要 $O(2^m)$，虽然看着很大但是有相当一部分状态都是不合法的，它们根本不会被枚举到，因此是能过的。
+ 答案：$f(n+2,0,0)$

```cpp
#include <cstdio>
#include <vector>
using namespace std;

const int N = 105, M = 10;
int f[2][1<<M][1<<M], g[N], cnt[1<<M], n, m;
char buf[M+1];
vector<int> states, ne[1<<M];

bool check(int x) {
    return !(x&(x>>1) | x&(x>>2));
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 0; i < 1<<m; i++) {
        if (check(i)) states.push_back(i);
        // 用 lowbit 卡个常先
        for (int j = i; j; j -= j&-j) {
            cnt[i]++;
        }
    }
    
    // 提前构造好 减少枚举次数
    for (int i: states) {
        for (int j: states) {
            if (!(i & j)) ne[i].push_back(j);
        }
    }
    
    // g[i] 是掩码, 如果 (g[i] & state) != 0 意味着这个状态不合法
    for (int i = 1; i <= n; i++) {
        scanf("%s", buf);
        for (int j = 0; j < m; j++)
            if (buf[j] == 'H')
                g[i] |= 1<<j;
    }
    
    for (int i = 1; i <= n+2; i++) {
        for (int u: states) {
            if (i >= 2 && g[i-2] & u) continue;
            for (int k: ne[u]) {
                if (g[i-1] & k) continue;
                for (int j: ne[k]) {
                    if (g[i] & j) continue;
                    // 当前行和前前行也不能有在同一列上的
                    if (!(j & u))
                        f[i&1][j][k] = max(f[i&1][j][k], f[i-1&1][k][u] + cnt[j]);
                }
            }
        }
    }
    
    printf("%d\n", f[n+2&1][0][0]);
    return 0;
}
```

## [NOIP2017] 宝藏

> 参与考古挖掘的小明得到了一份藏宝图，藏宝图上标出了 $n$ 个深埋在地下的宝藏屋， 也给出了这 $n$ 个宝藏屋之间可供开发的 $m$ 条道路和它们的长度。
>
> 小明决心亲自前往挖掘所有宝藏屋中的宝藏。但是，每个宝藏屋距离地面都很远，也就是说，从地面打通一条到某个宝藏屋的道路是很困难的，而开发宝藏屋之间的道路则相对容易很多。
>
> 小明的决心感动了考古挖掘的赞助商，赞助商决定免费赞助他打通一条从地面到某个宝藏屋的通道，通往哪个宝藏屋则由小明来决定。
>
> 在此基础上，小明还需要考虑如何开凿宝藏屋之间的道路。已经开凿出的道路可以 任意通行不消耗代价。每开凿出一条新道路，小明就会与考古队一起挖掘出由该条道路所能到达的宝藏屋的宝藏。另外，小明不想开发无用道路，即两个已经被挖掘过的宝藏屋之间的道路无需再开发。
>
> 新开发一条道路的代价是 $\mathrm{L} \times \mathrm{K}$。其中 $L$ 代表这条道路的长度，$K$ 代表从赞助商帮你打通的宝藏屋到这条道路起点的宝藏屋所经过的宝藏屋的数量（包括赞助商帮你打通的宝藏屋和这条道路起点的宝藏屋） 。
>
> 请你编写程序为小明选定由赞助商打通的宝藏屋和之后开凿的道路，使得工程总代价最小，并输出这个最小值。
>
> **输入格式**
>
> 第一行两个用空格分离的正整数 $n(12),m(1000)$，代表宝藏屋的个数和道路数。
>
> 接下来 $m$ 行，每行三个用空格分离的正整数，分别是由一条道路连接的两个宝藏屋的编号（编号为 $1-n$），和这条道路的长度 $v(5\times 10^5)$。
>
> **输出格式**
>
> 一个正整数，表示最小的总代价。
>
> 题目链接：[P3959](https://www.luogu.com.cn/problem/P3959)，[AcWing 529](https://www.acwing.com/problem/content/531/)。

考虑到是选了一个根后逐层推进，所以看出这个通道是一棵树的结构，每个新边的权都是树的高度 $h$ 乘以边长 $w$。

如果我们暴力地解决，需要存当前选中的所有节点以及最后一层结点，然后枚举新的一层的结点，这样是肯定不能过的，复杂度是 $O((2^n)^3)=O(2^{36})$。

如果我们把问题换成这个边可以连接到已经存在的所有结点，这样状态就好设置了，直接 $f(s,h)$ 即可。

容易看出，设原问题的所有方案集合为 $A$，新问题的集合为 $B$，有 $A\sub B$。假设在某个方案中，结点 $h(a)=3,h(b)=1$ 连接了 $a,b$，那么把 $a$ 及其子树都放到第二层是更优的，所以 $\min A=\min B$，最终答案是正确的。

如果当前已经选的点集为 $S$，那么我们要求一下 $S$ 的补集 $F$，然后枚举 $F$ 的所有子集，对于 $\forall f\in F$，找到 $s\in S$ 使得 $w(f,s)$ 最小，然后把代价加上层数乘边权。

枚举 $F$ 的所有子集（不包括 $\varnothing$），我们可以这么写：

```cpp
for (int s = F; s; s = (s-1)&F);
```

下面推导一下对于最后两个 $1$ 能枚举到：
$$
\begin{aligned}
F_0=F&=\cdots10100\\
F_1=(F_0-1) \& F&=\cdots 10000\\
F_2=(F_1-1) \& F&=\cdots 00100\\
F_3=(F_2-1) \& F&=\cdots 00000
\end{aligned}
$$
开始归纳，假设对于最后 $n$ 个 $1$ 可以枚举出 $2^n$ 种方案，证明最后 $n+1$ 个 $1$ 可以枚举出 $2^{n+1}$ 种方案：
$$
\begin{aligned}
F_0=F=&\cdots 1001001010\\
F_1=(F_0-1)\&F=&\cdots 1001001000\\
&\cdots\\
F_i=(F_{i-1}-1)\&F=&\cdots 0001001010\\
F_{i+1}=(F_i-1)\&F=&\cdots 0001001000
\end{aligned}
$$
一开头有 $2^n$ 个，零开头有 $2^n$ 个，所以有 $2^{n+1}$ 个，所以能枚举完所有子集。

枚举一个长度为 $n$ 集合的所有子集（长度为 $k$）的所有子集（长度为 $t$）的复杂度是：
$$
\sum_{k=0}^n C_n^k (\sum_{t=0}^k C_k^t)=\sum_{k=0}^n C_n^k 2^k=3^n
$$
这里就是连续用了两遍二项式定理。实际意义是，我们从长度为 $n$ 的全集中枚举出起点的所有集合 $S$，然后枚举出互补的集合 $F$，对集合 $F$ 枚举出所有子集：所以相当于枚举子集 $F$ 再枚举 $F$ 的所有子集，所以复杂度是 $O(3^n)$

首先我们可以写出来一个没有任何优化的代码。

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
#include <algorithm>
using namespace std;

const int N = 15, INF = 0x3f3f3f3f;
int n, m;
int f[N][1<<12], g[N][N];

int main() {
    scanf("%d%d", &n, &m);
    
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < 1 << n; j++) f[i][j] = INF;
        for (int j = 0; j < n; j++) g[i][j] = INF;
    }
    
    // 第 0 层任选一个点花费为 0
    for (int i = 0; i < n; i++) f[0][1<<i] = g[i][i] = 0;
    for (int i = 0, a, b, c; i < m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        --a, --b;
        g[a][b] = g[b][a] = min(g[a][b], c);
    }
    
    for (int dep = 1; dep < n; dep++) {
        for (int s = 1; s < 1 << n; s++) {
            int full = ~s & ((1 << n) - 1);
            for (int ne = full; ne; ne = (ne-1)&full) {
                int dist = 0;
                for (int j = 0; j < n; j++) {
                    if (ne >> j & 1) {
                        // 求出 j 到集合 s 的所有点中的最近距离
                        int d = INF;
                        for (int k = 0; k < n; k++)
                            if (s >> k & 1)
                                d = min(d, g[k][j]);
                        if (d == INF) goto end;
                        dist += d;
                    }
                }
                
                f[dep][s|ne] = min(f[dep][s|ne], f[dep-1][s] + dist * dep);
                end: ;
            }
        }
    }
    
    int res = INF;
    for (int i = 0; i < n; i++) res = min(res, f[i][(1<<n)-1]);
    return !printf("%d\n", res);
}

```

看一下复杂度，$O(n^33^n)=O(9\times 10^8)$，因为中间有一个小剪枝，判断最小值为 `INF` 时会跳过一个 $O(n^2)$ 的复杂度，所以洛谷被我开 $O_2$ 卡过去了。

但是这个复杂度是可以抽出来一层循环，我们把最内层求一个点到一个集合的最近距离操作抽离出来，就能优化到 $O(n^2 3^n)=O(7.6\times 10^7)$，还有一个小剪枝，所以是比较稳的。

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
#include <algorithm>
using namespace std;

const int N = 15, INF = 0x3f3f3f3f;
int n, m;
int f[N][1<<12], g[N][N], d[N][1<<12];

int main() {
    scanf("%d%d", &n, &m);
    
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < 1 << n; j++) f[i][j] = d[i][j] = INF;
        for (int j = 0; j < n; j++) g[i][j] = INF;
    }
    
    for (int i = 0; i < n; i++) f[0][1<<i] = g[i][i] = 0;
    for (int i = 0, a, b, c; i < m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        --a, --b;
        g[a][b] = g[b][a] = min(g[a][b], c);
    }
    
    for (int i = 0; i < n; i++)
        for (int s = 0; s < 1 << n; s++)
            for (int j = 0; j < n; j++)
                if (s >> j & 1) d[i][s] = min(d[i][s], g[i][j]);
    
    for (int dep = 1; dep < n; dep++) {
        for (int s = 1; s < 1 << n; s++) {
            int full = ~s & ((1 << n) - 1);
            for (int ne = full; ne; ne = (ne-1)&full) {
                int dist = 0;
                for (int j = 0; j < n; j++) {
                    if (ne >> j & 1) {
                        if (d[j][s] == INF) goto end;
                        dist += d[j][s];
                    }
                }
                
                f[dep][s|ne] = min(f[dep][s|ne], f[dep-1][s] + dist * dep);
                end: ;
            }
        }
    }
    
    int res = INF;
    for (int i = 0; i < n; i++) res = min(res, f[i][(1<<n)-1]);
    return !printf("%d\n", res);
}
```

然而，实测下来枚举全集 $U$ 然后枚举子集 $S$ 再求补集 $F$（用异或）是比较快的，原因是上面的方法强制 $U$ 为全选了，导致存在很多重复枚举的状态。

```cpp
#include <cstdio>
#include <cstring>
#include <vector>
#include <algorithm>
using namespace std;

const int N = 15, INF = 0x3f3f3f3f;
int n, m;
int f[N][1<<12], g[N][1<<12], d[N][N];

int main() {
    scanf("%d%d", &n, &m);
    
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < 1 << n; j++) f[i][j] = g[i][j] = INF;
        for (int j = 0; j < n; j++) d[i][j] = INF;
    }
    for (int i = 0; i < n; i++) f[0][1<<i] = d[i][i] = 0;
    for (int i = 0, a, b, c; i < m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        --a, --b;
        d[a][b] = d[b][a] = min(d[a][b], c);
    }
    
    for (int i = 0; i < n; i++)
        for (int j = 0; j < 1 << n; j++)
            for (int k = 0; k < n; k++)
                if (j >> k & 1) g[i][j] = min(g[i][j], d[i][k]);
    
    for (int dep = 1; dep < n; dep++) {
        for (int U = 1; U < 1 << n; U++) {
            for (int S = (U-1)&U; S; S = (S-1)&U) {
                int dist = 0;
                int F = U ^ S;
                
                for (int k = 0; k < n; k++) {
                    if (S >> k & 1) {
                        if (g[k][F] == INF) goto end;
                        dist += g[k][F];
                    }
                }
                
                f[dep][U] = min(f[dep][U], f[dep-1][F] + dist * dep);
                end: ;
            }
        }
    }
    
    int res = INF;
    for (int i = 0; i < n; i++) res = min(res, f[i][(1<<n)-1]);
    return !printf("%d\n", res);
}
```

## 集合（三进制状态）

> 给定一个可重集 $S$，选出 $S$ 最大的一个子集，使得其中任意三个数相加不被 9 整除，输出最大子集大小，$n\le 10^5,S_i\le 10^9$。

观察到我们只需要知道 $S_i \bmod 9$ 的值，具体是多少并不重要，所以考虑 $S_i\lt 9$ 的做法。

看到状态这么小，除了爆搜就是状压，爆搜显然不太现实，所以考虑状压。

注意到每个数选 $0,1,2$ 个带来的后果都不一样，但是选 $2$ 个和 $2$ 个以上只和这个数本身有关系，若 $3a\equiv 0\pmod 9$，那么最多只选 $2$ 个，否则是能选多少就选多少。

因此，我们用三进制数表示状态，那么已知状态 $S$，并且 $a\not\in S$，可以转移：
$$
f(S)\rightarrow \begin{cases}
f(S\cup \{a\})\quad \forall b,c\in S,a+b+c\not\equiv 0\pmod 9\\
f(S\cup \{a,a\})\quad \forall b,c\in S, a+b+c\not\equiv 0,2a+b\not\equiv 0\pmod 9\\
\end{cases}
$$
若 $a\in S$ 并且只有一个，也可以转移：
$$
f(S)\to f(S\cup \{a\}),\forall b,c\in S,a+b+c\not\equiv 0\pmod 9
$$
注意，这里是允许 $b,c= a$ 的，而既然 $a\in S$，那么之前转移的时候就已经满足第一个条件了，所以实际上相当于 $2a+b\not \equiv 0\pmod 9$，而这个转移已经被上面包含了，所以从 $1$ 转移到 $2$ 是没必要的。

除此之外，我们在进行所有操作之前，都要检查当前状态是否合法，如果状态都不合法就更无从谈起转移的问题了。

```cpp
#include <bits/stdc++.h>
using namespace std;

int n, cnt[9], f[20010];
int p3[10] = {1, 3, 9, 27, 81, 243, 729, 2187, 6561, 19683};

inline int get(int s, int p) {
    return s % p3[p+1] / p3[p];
}

inline bool check1(int s, int x) {
    for (int i = 0; i < 9; i++) {
        if (!get(s, i)) continue;
        for (int j = 0; j < 9; j++) {
            if (!get(s, j)) continue;
            if ((i + j + x) % 9 == 0) return false;
        }
    }
    return true;
}

inline bool check2(int s, int x) {
    for (int i = 0; i < 9; i++) {
        if (!get(s, i)) continue;
        if ((2*x + i) % 9 == 0) return false;
    }
    return true;
}

int main() {
    cin >> n;
    for (int i = 1, a; i <= n; i++) {
        cin >> a;
        cnt[a % 9]++;
    }

    int ans = 0;
    for (int s = 0; s < p3[9]; s++) {
        bool valid = true;
        for (int i = 0; i < 9; i++) {
            if (cnt[i] < get(s, i)) {
                valid = false;
                break;
            }
        }
        if (!valid) continue;

        for (int i = 0; i < 9; i++) {
            if (get(s, i)) continue;
            if (check1(s, i) && cnt[i]) {
                f[s+p3[i]] = max(f[s+p3[i]], f[s]+1);
                if (check2(s, i) && cnt[i] >= 2) {
                    if (3*i % 9 == 0) f[s+2*p3[i]] = max(f[s+2*p3[i]], f[s]+2);
                    else f[s+2*p3[i]] = max(f[s+2*p3[i]], f[s]+cnt[i]);
                }
            }
        }
        ans = max(ans, f[s]);
    }

    cout << ans << '\n';

    return 0;
}
```

