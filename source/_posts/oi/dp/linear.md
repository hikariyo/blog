---
date: 2023-06-18 14:17:03
updated: 2023-08-19 13:54:00
title: 动态规划 - 线性
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 简介

本文包含数字三角形问题、最长上升子序列问题的几道基础例题，更多内容参考本站其它相关文章。

然后把一些不好分类的题也写在这里了。

## 数字三角形

> 给定一个如下图所示的数字三角形，从顶部出发，在每一结点可以选择移动至其左下方的结点或移动至其右下方的结点，一直走到底层，要求找出一条路径，使路径上的数字的和最大。
>
> ```text
>         7
>       3   8
>     8   1   0
>   2   7   4   4
> 4   5   2   6   5
> ```

分析：对于任意一个点 `i, j` 它的最优解应该是左上方的最优解和右上方的最优解取最大值，然后加上当前节点。接下来分析这个数字三角形的存储结构：

```
7
3 8
8 1 0 
2 7 4 4
4 5 2 6 5
```

对于第 `i` 行第 `j` 列，它的左上方是 `i-1, j-1`，右上方是 `i-1, j` 所以可以写出下面的代码，用 `a` 表示三角形。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 510, M = N*N/2, INF=1e9;
int f[N][N], a[N][N];
int n;

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= i; j++) {
            cin >> a[i][j];
        }
    }
    // 初始化边界
    for (int i = 0; i <= n; i++) {
        // 我们要取 i-1, j-1 和 i-1, j 所以每行都要保证前后边界是 -INF
        for (int j = 0; j <= i+1; j++) {
            f[i][j] = -INF;
        }
    }
    
    // 这里初始化 1, 1 然后从第二行开始求解
    f[1][1] = a[1][1];
    for (int i = 2; i <= n; i++) {
        for (int j = 1; j <= i; j++) {
            f[i][j] = max(f[i-1][j-1], f[i-1][j]) + a[i][j];
        }
    }
    
    // 最后一行的最大值
    int res = -INF;
    for (int j = 1; j <= n; j++) res = max(res, f[n][j]);
    cout << res << endl;

    return 0;
}
```

也可以从底向上做：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 510, M = N*N/2, INF=1e9;
int f[N][N], a[N][N];
int n;

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= i; j++) {
            cin >> a[i][j];
        }
    }
    // 初始化边界
    for (int i = 0; i <= n; i++) {
        // 我们要取 i-1, j-1 和 i-1, j 所以每行都要保证前后边界是 -INF
        for (int j = 0; j <= i+1; j++) {
            f[i][j] = -INF;
        }
    }
    
    // 初始化最后一行 自底向上
    for (int j = 1; j <= n; j++) f[n][j] = a[n][j];
    for (int i = n-1; i >= 1; i--) {
        for (int j = 1; j <= i; j++) {
            f[i][j] = max(f[i+1][j], f[i+1][j+1]) + a[i][j];
        }
    }
    
    cout << f[1][1] << endl;

    return 0;
}
```

## 最长上升子序列

> 给定一个长度为 N 的数列，求数值严格单调递增的子序列的长度最长是多少。

我一开始写了好几个版本的错误思路写出来的错误答案：

1. 子序列并不是连续子序列，所以不能用滑动窗口。

2. 如果当前数字比前面的 `k` 个数字大，那么它的长度**不等同于** `k`。


应当是如果当前数比前面的某个数大，那么构成的子序列等于那个数构成的子序列长度加上当前的一个数的长度。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010, INF=1e9+1;
int a[N], n, f[N];

int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        cin >> a[i];
        f[i] = 1;
    }

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (a[i] > a[j]) f[i] = max(f[j]+1, f[i]);
        }
    }

    cout << *max_element(f, f+n) << endl;
    
    return 0;
}
```

最后一步是求最大值，使用了 `algorithm` 里的 `max_element`，传入头部指针和 `past-the-end` 指针就可以，更详细的内容参考 [cppreference](https://en.cppreference.com/w/cpp/algorithm/max_element) 上的介绍。

可以优化，分析如果有数组 `q[i]` 储存最长子序列长度为 `i` 的**末尾元素最小值**，用反证法可以证明 `q` 一定是单调递增的，所以每次用二分查找比某个数更小的末尾元素，然后把这个数加到这个序列中，就可以让这个序列最长。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;
const int N = 1e5+10;
int q[N], a[N], n;

int main() {
    cin.tie(0), cout.tie(0), ios::sync_with_stdio(false);
    
    cin >> n;
    for (int i = 0; i < n; i++) cin >> a[n];
    
    int len = 0;
    for (int i = 0; i < n; i++) {
        // 二分查找末尾元素比 a[i] 小的最长子序列长度
        int l = 0, r = len;
        while (l < r) {
            // 因为有 l = mid 防止死循环 这里向上取整
            int mid = l + r + 1 >> 1;
            if (q[mid] < a[i]) l=mid;
            else r = mid-1;
        }
        // 此时 q[l] 就存着那个最长的子序列，l 是长度
        // 这时找到的数字一定比原来 l+1 的要小，因为如果它比 q[l+1] 更大 二分就不会找到这个坐标
        q[l+1] = a[i];
        len = max(len, l+1)
    }
    cout << len << endl;
    return 0;
}
```

## 最长公共子序列

> 给定两个长度分别为 N 和 M 的字符串 A 和 B，求既是 A 的子序列又是 B 的子序列的字符串长度最长是多少。

分析如下：

+ 状态表示： `f[i, j]`
  + 表示：所以在 A 序列中前 i 个和在 B 序列中前 j 个出现的所有子序列
  + 存储：这些子序列的最大值

+ 状态转移：包含四部分，不选 `i` 不选 `j`，不选 `i` 选 `j`，选 `i` 不选 `j`，选 `i` 和 `j`。由于第一种情况被第二和第三种情况包含了，所以代码不写。当两个都选时，需要判断这两个字母是否相等，否则不可以都选。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010;
char a[N], b[N];
int n, m;
int f[N][N];

int main() {
    scanf("%d%d%s%s", &n, &m, a+1, b+1);
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            f[i][j] = max(f[i-1][j], f[i][j-1]);
            if (a[i] == b[j]) f[i][j] = max(f[i][j], f[i-1][j-1]+1);
        }
    }
    printf("%d\n", f[n][m]);
    return 0;
}
```

## 最短编辑距离

> 给定两个字符串 A 和 B，现在要将 A 经过若干操作变为 B，可进行的操作有：
>
> 1. 删除–将字符串 A 中的某个字符删除。
> 2. 插入–在字符串 A 的某个位置插入某个字符。
> 3. 替换–将字符串 A 中的某个字符替换为另一个字符。
>
> 现在请你求出，将 A 变为 B 至少需要进行多少次操作。

分析如下：

1. 状态表示 `f[i][j]`：
   + 表示：字符串 `A` 的前 `i` 个变化到字符串 `B` 的前 `j` 个需要的**所有操作的集合**。
   + 存储：这些操作的最小值。
2. 状态转移，重点是要**先让某些条件符合**，然后进行下一步操作：
   + 删除：先让  `A` 的前 `i-1` 个和 `B` 的前 `j` 个相同，然后删掉 `A[i]`，所以它需要的步数是 `f[i-1][j]+1`。
   + 增加：先让  `A` 的前 `i` 个和 `B` 的前 `j-1` 个相同，然后增 `B[j]` 在 `A[i-1]` 之后，所以它需要的步数是 `f[i][j-1]+1`。
   + 修改：先让 `A` 的前 `i-1` 个和 `B` 的前 `j-1` 个相同，所以它需要的步数是 `f[i-1][j-1] + 1 or 0`，这里后面的 `1 or 0` 取决于 `a[i]` 和 `b[j]` 是否相同。

注意，我们需要先初始化，让所有 `f[0][j] = j` 和 `f[i][0] = i`，这样做的意义是 `A` 的前 `0` 个变化到 `B` 的前 `j` 个只能添加 `j` 个字母，而 `A` 的前 `i` 个变化到 `B` 的前 `0` 个只能删除 `j` 个字母。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 1010;
char a[N], b[N];
int f[N][N];
int n, m;

int main() {
    scanf("%d%s%d%s", &n, a+1, &m, b+1);
    
    for (int i = 1; i <= n; i++) f[i][0] = i;
    for (int j = 1; j <= m; j++) f[0][j] = j;
    
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            // 添加和删除需要的步数
            f[i][j] = min(f[i-1][j]+1,f[i][j-1]+1);
            // 改变需要的步数
            if (a[i] == b[j]) f[i][j] = min(f[i][j], f[i-1][j-1]);
            else f[i][j] = min(f[i][j], f[i-1][j-1] + 1);
        }
    }
    
    printf("%d\n", f[n][m]);
    
    return 0;
}
```

## [NOIP2014] 子矩阵

> 给出如下定义：
>
> 1. 子矩阵：从一个矩阵当中选取某些行和某些列交叉位置所组成的新矩阵（保持行与列的相对顺序）被称为原矩阵的一个子矩阵。
>
>     例如，下面图中选取第 2,4 行和第 2,4,5 列交叉位置的元素得到一个 2×3 的子矩阵如下所示：
>     
>     ```text
>     9 3 3 3 9
>     9 4 8 7 4
>     1 7 4 6 6
>     6 8 5 6 9
>     7 4 5 6 1
>     
>     4 7 4
>     8 6 9
>     ```
>
> 2. 相邻的元素：矩阵中的某个元素与其上下左右四个元素（如果存在的话）是相邻的。
> 3. 矩阵的分值：矩阵中每一对相邻元素之差的绝对值之和。
>
> 本题任务：给定一个 n 行 m 列的正整数矩阵，请你从这个矩阵中选出一个 r 行 c 列的子矩阵，使得这个子矩阵的分值最小，并输出这个分值。
>
> 题目链接：[P2258](https://www.luogu.com.cn/problem/P2258)。

如果直接暴力枚举出 R 行 C 列，需要的时间复杂度会是：
$$
O(\binom{n}r \binom{m}c rc)
$$
这是相当不能接受的，但是如果只有一个地方枚举还是可以接受的。

于是想到枚举行然后在这些行中选择 $c$ 列，这里就可以用 DP 做了。

由于状态转移是和选中哪一列有关的，所以这里定义状态的时候需要注意到这一点，因此：

+ 状态表示 $f(i,j)$：前 $i$ 列中选择第 $i$ 列且一共选 $j$ 列的所有方案。

+ 状态转移方程：
    $$
    f(i,j)=\min_{k\le i-1}\{f(k,j-1) + w(i)+\text{cost}(i,k)\}
    $$

+ 初始化：$f(i,0)=0,f(i,1)=w(i),f(i,j)=\inf$
+ 答案：$\min_{i=c}^m f(i,c)$

其中 $\text{cost}(i,k)$ 指的是第 $i,k$ 列之间的分值，$w(i)$ 指的是第 $i$ 列的分支。

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 20;
int a[N][N], n, m, r, c;
int rows[N], w[N];
int f[N][N], pre[N][N];
int ans = 2e9;

int cost(int x, int y) {
    int res = 0;
    for (int j = 1; j <= r; j++)
        res += abs(a[rows[j]][x] - a[rows[j]][y]);
    return res;
}

void dfs(int u, int cnt) {
    if (u > n) return;
    rows[cnt] = u;
    if (cnt < r) {
        for (int i = u+1; i <= n; i++) dfs(i, cnt+1);
        return;
    }

    // 预处理列分值
    memset(w, 0, sizeof w);
    for (int j = 1; j <= m; j++) {
        for (int i = 2; i <= r; i++) {
            w[j] += abs(a[rows[i]][j] - a[rows[i-1]][j]);
        }
    }
    
    // 选择 C 列
    memset(f, 0x3f, sizeof f);
    f[0][0] = 0;
    for (int i = 1; i <= m; i++) {
        f[i][0] = 0, f[i][1] = w[i];
        for (int j = 2; j <= c; j++) {
            for (int k = 1; k < i; k++) {
                f[i][j] = min(f[i][j], f[k][j-1] + cost(k, i) + w[i]);
            }
        }
    }
    for (int i = c; i <= m; i++) ans = min(ans, f[i][c]);
}

int main() {
    scanf("%d%d%d%d", &n, &m, &r, &c);
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            scanf("%d", &a[i][j]);
    
    for (int i = 1; i <= n; i++) dfs(i, 1);

    printf("%d\n", ans);
    return 0;
}
```

## [NOIP2005] 过河

> 在河上有一座独木桥，一只青蛙想沿着独木桥从河的一侧跳到另一侧。在桥上有一些石子，青蛙很讨厌踩在这些石子上。由于桥的长度和青蛙一次跳过的距离都是正整数，我们可以把独木桥上青蛙可能到达的点看成数轴上的一串整点：$0,1,\cdots,L$（其中 $L$ 是桥的长度）。坐标为 $0$ 的点表示桥的起点，坐标为 $L$ 的点表示桥的终点。青蛙从桥的起点开始，不停的向终点方向跳跃。一次跳跃的距离是 $S$ 到 $T$ 之间的任意正整数（包括 $S,T$）。当青蛙跳到或跳过坐标为 $L$ 的点时，就算青蛙已经跳出了独木桥。
>
> 题目给出独木桥的长度 $L$，青蛙跳跃的距离范围 $S,T$，桥上石子的位置。你的任务是确定青蛙要想过河，最少需要踩到的石子数。
>
> $1\le L \le 10^9$，$1\le S\le T\le10$，$1\le M\le100$。
>
> 题目链接：[P1052](https://www.luogu.com.cn/problem/P1052)。

不考虑数据范围是很简单的：
$$
f(i)=\min_{S\le i-j\le T} f(j)+[\text{stone}(i)=1]
$$
但是数据范围很大，石头的数量却很少，所以可以考虑将一个很大的区间等效成一个比较小的区间。

考虑两个相邻的石头 $s_i,s_j$ 之间不会有任何石头，由 $p,q$ 互质时，存在这样一个定理，在 NOIP2017 T2 中考过：
$$
\forall x\ge (p-1)(q-1), x=ap+bq,a,b\in \N
$$
这个定理稍后证明，于是当 $T=10$ 时，大于等于 $(T-1)(T-2)=72$ 的所有步数都能表示出来，$w_i$ 的右边和 $w_j$ 的左边各延出来 $T$，预估一下当区间长度大于$100$ 时与 $100$ 是等价的。

要证明上面这个定理，首先证明 $(p-1)(q-1)-1$ 不能通过 $ap+bq$ 表示：
$$
\begin{aligned}
&\text{Let }ap+bq=(p-1)(q-1)-1\\
&(a+1)p+(b+1)q=pq\\
&p|pq, q|pq,\gcd(p,q)=1\\
\Rightarrow &p|(b+1)q,q|(a+1)p\\
&p|(b+1),q|(a+1)\\
&p\le (b+1),q\le (a+1)\\
\Rightarrow &(b+1)q\ge pq,(a+1)p\ge pq\\
\Rightarrow &2pq = pq,pq=0
\end{aligned}
$$
矛盾，所以这个数字无法被 $ap+bq$ 表示。

接下来证明 $pq-p-q+d,d\in \N^*$ 可以被表示，不妨设 $p\lt q$，然后设带余除法的形式 $d=kp+r,r\in[0,p)$

由裴蜀定理知存在 $xp+yq=r,x,y\in\N^*$，设 $x\in(0,q)$
$$
yq=r-xp>-pq\Rightarrow y>-p\\
yq=r-xp<0
$$
所以 $y\in(-p,0)$，因此：
$$
\begin{aligned}
&pq-p-q+d\\
&=pq-p-q+kp+xp+yq\\
&=(y+p-1)q+(x-1+k)p\\
\end{aligned}
$$
我们知道 $y+p-1\ge 0,x-1+k\ge 0$，证毕。

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 110, M = 11000;
int L, S, T, n, p[N], stone[N];
int f[M], s[M];

int main() {
    scanf("%d%d%d%d", &L, &S, &T, &n);
    for (int i = 1; i <= n; i++)
        scanf("%d", &p[i]);
    
    // S = T 只能表达 S 的倍数, 无法保证可以表达出 72 以上的任何一个数
    if (S == T) {
        int res = 0;
        for (int i = 1; i <= n; i++) {
            if (p[i] % S == 0) res++;
        }
        printf("%d\n", res);
        return 0;
    }
    
    sort(p+1, p+1+n);
    
    int m = 0;
    for (int i = 1; i <= n; i++) {
        stone[i] = stone[i-1] + min(p[i] - p[i-1], 100);
        m += stone[i] - stone[i-1];
        s[stone[i]] |= 1;
    }
    
    if (L - m > 10) L = m+10;
    memset(f, 0x3f, sizeof f);
    f[0] = 0;
    for (int i = 0; i < L; i++) {
        for (int j = i+S; j <= i+T; j++) f[j] = min(f[j], f[i]+s[j]);
    }
    int ans = 1e9;
    for (int i = L; i <= L+10; i++) ans = min(ans, f[i]);
    printf("%d\n", ans);
    return 0;
}
```

## [NOIP2015] 子串

> 有两个仅包含小写英文字母的字符串 A 和 B。
>
> 现在要从字符串 A 中取出 k 个互不重叠的非空子串，然后把这 k 个子串按照其在字符串 A 中出现的顺序依次连接起来得到一个新的字符串，请问有多少种方案可以使得这个新串与字符串 B 相等？答案对 `1e9+7` 取模。
>
> 注意：子串取出的位置不同也认为是不同的方案。
>
> 题目链接：[AcWing 520](https://www.acwing.com/problem/content/522/)，[P2679](https://www.luogu.com.cn/problem/P2679)。

+ 状态表示 $f(i,j,k,0/1)$：$A[1\sim i]$ 中抽出 $k$ 个子串拼接成 $B[1\sim j]$ 的所有方案，最后表示选或不选 $A_i$。

+ 初始化：$f(i,0,0,0)=1$，都只有一种方案就是什么都不选。

+ 状态转移：

    当前不选的很简单，就是 $i-1$ 和 $j$ 选或者不选就可以不重不漏的划分了：
    $$
    f(i,j,k,0)=f(i-1,j,k,0)+f(i-1,j,k,1)
    $$
    当前选的话，如果 $A_i\ne B_j$ 那么就没有方案，就是零，否则就是：
    $$
    f(i,j,k,1)=f(i-1,j-1,k,1)+f(i-1,j-1,k-1,1)+f(i-1,j-1,k-1,0)
    $$
    如果放进 $i-1$ 对应的组，此时 $k$ 不变，需要把 $A_{i-1}$ 选上，如果不选 $A_{i-1}$ 这个组就不是连续的了，状态转移就错了。如果放进新的组，那么 $A_{i-1}$ 选不选都无所谓，它需要拼接到 $B_{j-1}$ 结尾的串。

+ 优化：明显可以用滚动数组，空间合格了。

```cpp
#include <cstdio>
using namespace std;

const int N = 1010, M = 210, MOD = 1e9+7;
char a[N], b[N];
int n, m, K;
int f[2][N][M][2];

int main() {
    scanf("%d%d%d%s%s", &n, &m, &K, a+1, b+1);
    
    f[0][0][0][0] = 1;
    for (int i = 1; i <= n; i++) {
        int I = i & 1;
        f[I][0][0][0] = 1;
        for (int j = 1; j <= m; j++) {
            for (int k = 1; k <= K; k++) {
                f[I][j][k][0] = (f[I^1][j][k][0] + f[I^1][j][k][1]) % MOD;
                if (a[i] != b[j]) f[I][j][k][1] = 0;
                else f[I][j][k][1] = (f[I^1][j-1][k][1] + (f[I^1][j-1][k-1][0] + f[I^1][j-1][k-1][1]) % MOD) % MOD;
            }
        }
    }
    
    printf("%d\n", (f[n&1][m][K][0] + f[n&1][m][K][1]) % MOD);
    return 0;
}
```

## 陪审团

> 法官先随机挑选 N 个人（编号 1,2…,N）作为陪审团的候选人，然后再从这 N 个人中按照下列方法选出 M 人组成陪审团。
>
> 首先，参与诉讼的控方和辩方会给所有候选人打分，分值在 00 到 2020 之间。
>
> 第 i 个人的得分分别记为 `p[i]` 和 `d[i]`。
>
> 为了公平起见，法官选出的 M 个人必须满足：辩方总分 D 和控方总分 P 的差的绝对值 |D−P| 最小。
>
> 如果选择方法不唯一，那么再从中选择辨控双方总分之和 D+P 最大的方案。
>
> 求最终的陪审团获得的辩方总分 D、控方总分 P，以及陪审团人选的编号。
>
> 题目链接：[POJ 1015](http://poj.org/problem?id=1015)，[AcWing 280](https://www.acwing.com/problem/content/282/)。

因为本题需要用到分数之差，这也提示我们在 DP 状态中加上一维的差。

+ 状态表示 $f(i,j,k)$：前 $i$ 人中选 $j$ 个且差为 $k$ 的方案中的总分之和最大值。

+ 初始化：$f(i,0,0)=0,f(i,j,k)=-\infin$

    从前 $i$ 人中一个也不选的最大值一定是确定的 $0$

+ 状态转移方程：当选择第 $i$ 个人时，从前 $i-1$ 选 $j-1$ 转移过来。
    $$
    f(i,j,k)=\max\left\{\begin{aligned}
    &f(i-1,j,k)\\
    &f(i-1,j-1,k-(p_i-d_i))+p_i+d_i
    \end{aligned}\right.
    $$

+ 还原方案：倒序 $i=n,j=m$ 每次比较判断是从哪个状态转移过来的即可。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 210, M = 810, base = 400;
int n, m;
int p[N], d[N];
int f[N][25][M], ans[25];


int main() {
    int C = 1;
    while (scanf("%d%d", &n, &m), n || m) {
        for (int i = 1; i <= n; i++) scanf("%d%d", &p[i], &d[i]);
        memset(f, -0x3f, sizeof f);
        for (int i = 0; i <= n; i++) f[i][0][base] = 0;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                for (int k = 0; k < M; k++) {
                    f[i][j][k] = f[i-1][j][k];
                    int t = k - (p[i] - d[i]);
                    if (t < 0 || t >= M) continue;
                    f[i][j][k] = max(f[i][j][k], f[i-1][j-1][t] + p[i] + d[i]);
                }
            }
        }
        
        int v = 0;
        while (f[n][m][base+v] < 0 && f[n][m][base-v] < 0) v++;
        if (f[n][m][base-v] > f[n][m][base+v]) v = -v;
        
        int cnt = 0, i = n, j = m, k = base+v;
        while (i) {
            // 没选 i
            if (f[i][j][k] == f[i-1][j][k]) i--;
            else {
                ans[cnt++] = i;
                k -= p[i] - d[i];
                i--, j--;
            }
        }
        
        int sp = 0, sd = 0;
        for (int i = 0; i < cnt; i++) sp += p[ans[i]], sd += d[ans[i]];
        
        printf("Jury #%d\n", C++);
        printf("Best jury has value %d for prosecution and value %d for defence:\n", sp, sd);
        for (int i = cnt-1; i >= 0; i--) printf(" %d", ans[i]);
        puts("\n");
    }
    return 0;
}
```

## [CSP-S2019] Emiya 家今天的饭

> Emiya 是个擅长做菜的高中生，他共掌握 $n$ 种**烹饪方法**，且会使用 $m$ 种**主要食材**做菜。为了方便叙述，我们对烹饪方法从 $1 \sim n$ 编号，对主要食材从 $1 \sim m$ 编号。
>
> Emiya 做的每道菜都将使用**恰好一种**烹饪方法与**恰好一种**主要食材。更具体地，Emiya 会做 $a_{i,j}$ 道不同的使用烹饪方法 $i$ 和主要食材 $j$ 的菜（$1 \leq i \leq n$、$1 \leq j \leq m$），这也意味着 Emiya 总共会做 $\sum\limits_{i=1}^{n} \sum\limits_{j=1}^{m} a_{i,j}$ 道不同的菜。
>
> Emiya 今天要准备一桌饭招待 Yazid 和 Rin 这对好朋友，然而三个人对菜的搭配有不同的要求，更具体地，对于一种包含 $k$ 道菜的搭配方案而言：
> - Emiya 不会让大家饿肚子，所以将做**至少一道菜**，即 $k \geq 1$
> - Rin 希望品尝不同烹饪方法做出的菜，因此她要求每道菜的**烹饪方法互不相同**
> - Yazid 不希望品尝太多同一食材做出的菜，因此他要求每种**主要食材**至多在**一半**的菜（即 $\lfloor \frac{k}{2} \rfloor$ 道菜）中被使用
>
> 这些要求难不倒 Emiya，但他想知道共有多少种不同的符合要求的搭配方案。两种方案不同，当且仅当存在至少一道菜在一种方案中出现，而不在另一种方案中出现。
>
> Emiya 找到了你，请你帮他计算，你只需要告诉他符合所有要求的搭配方案数对质数 $998,244,353$ 取模的结果。
>
> 题目链接：[P5664](https://www.luogu.com.cn/problem/P5664)，[AcWing 1155](https://www.acwing.com/problem/content/1157/)。

### 暴力 64pts

看到前 16 个测试点都有 $m\le 3$，所以可以考虑先暴力拿到这些分。

+ 状态表示 $f(i,x,y,z)$：在前 $i$ 个烹饪方法中 $x,y,z$ 分别表示某个食材选了几个。

+ 状态转移：当前烹饪方法要么不做，要么在 $x,y,z$ 里挑一个做。
    $$
    \begin{aligned}
    f(i,x,y,z)=&f(i-1,x,y,z)+f(i-1,x-1,y,z)\cdot a_{i,1}+\\
    &f(i-1,x,y-1,z)\cdot a_{i,2}+f(i-1,x,y,z-1)\cdot a_{i,3}
    \end{aligned}
    $$

+ 答案：$\sum f(n,X,Y,Z)$，满足 $X+Y+Z> 0$ 且 $\max\{X,Y,Z\}\le (X+Y+Z)/2$

+ 初始化：$f(i,0,0,0)=1$

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 45, M = 4, P = 998244353;
int n, m, f[N][N][N][N];
int a[N][M];

inline int add(int a, int b) {
    return (a+b) % P;
}


inline int mul(int a, int b) {
    return (LL)a*b % P;
}

int main() {
    scanf("%d%d", &n, &m);
    
    // 考场上可以随便输出一个随机数, 这里防止消耗我 RP 就不加了
    if (m > 3) {
        puts("-1");
        return 0;
    }
    
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            scanf("%d", &a[i][j]);
    
    f[0][0][0][0] = 1;
    for (int i = 1; i <= n; i++) {
        f[i][0][0][0] = 1;
        
        for (int x = 0; x <= i; x++)
            for (int y = 0; y <= i; y++)
                for (int z = 0; z <= i; z++) {
                    f[i][x][y][z] = f[i-1][x][y][z];
                    
                    if (x) f[i][x][y][z] = add(f[i][x][y][z], mul(f[i-1][x-1][y][z], a[i][1]));
                    if (y) f[i][x][y][z] = add(f[i][x][y][z], mul(f[i-1][x][y-1][z], a[i][2]));
                    if (z) f[i][x][y][z] = add(f[i][x][y][z], mul(f[i-1][x][y][z-1], a[i][3]));
                }
    }
    
    int res = 0;
    for (int x = 0; x <= n; x++)
        for (int y = 0; y <= n; y++)
            for (int z = 0; z <= n; z++)
                if (x+y+z > 0 && max({x, y, z}) <= (x+y+z) / 2) {
                    res = add(res, f[n][x][y][z]);
                }
    printf("%d\n", res);
    return 0;
}
```

### 容斥+差值状态 100pts

如果只满足前两个条件，这个问题还是比较简单的，先做一遍 DP 求出来所有的方案。

+ 状态表示 $f(i,j)$：前 $i$ 个烹饪方法做 $j$ 道菜的方案数（种类）。

+ 初始化：$f(i,0)=1$

+ 状态转移：第 $i$ 种烹饪方法一共可以做当前列之和种菜。
    $$
    f(i,j)=f(i-1,j)+f(i-1,j-1)\cdot (a_{i,1}+a_{i,2}+\cdots+a_{i,m})
    $$
    用到乘法原理。

这一步可以先对每列求和进行优化。

然后，枚举每个食材，对每个食材做一遍 DP，求出用 $n$ 种烹饪方法做 $i=1,2,\cdots,n$ 个菜时第 $k$ 种食材用了多于 $\lfloor i/2\rfloor$ 的方案数。

这里为什么不统计满足要求的，而是统计不满足要求的？对于这个问题，我们考虑如果多种食材满足要求的方案，每次都会加上，那么这个容斥就更麻烦了，所以求不满足要求的所有方案用总方案数减去更优。

+ 状态表示，对于 $k$ 食材：$g(i,j)$：前 $i$ 个烹饪方法中 $k$ 食材的数量减去别的食材的数量为 $j$ 的所有方案数。

+ 初始化：$g(0,0) = 1$，这里不能再整 $g(i,0)=1$ 了，因为 $j$ 表示的是差值。

+ 状态转移：分为不选 $i$ 烹饪方式，选 $i$ 烹饪方式的 $k$ 菜，选 $i$ 烹饪方式不选 $k$ 菜。
    $$
    g(i,j)=g(i-1,j)+g(i-1,j-1)\cdot a_{i,k}+g(i-1,j+1)\cdot(s_i-a_{i,k})
    $$
    当选 $i$ 时，前一种会使得 $j$ 加一，因此从 $j-1$ 转移过来；后一种会使得 $j$ 减一，因此从 $j+1$ 转移过来。

+ 贡献：$\sum_{j=1}^n g(n,j)$，只要 $k$ 减去别的食材大于零就是非法的方案。

有负数，加一个 $N$ 作为偏移量。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 110, M = 2010, P = 998244353;
int n, m;
int a[N][M], s[N], f[N][N], g[N][2*N];

#define add(a, b) (((a)+(b))%P)
#define sub(a, b) (((a)-(b))%P)
#define mul(a, b) ((LL)(a)*(b) % P)

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++) {
            scanf("%d", &a[i][j]);
            s[i] = add(s[i], a[i][j]);
        }
    
    f[0][0] = 1;
    for (int i = 1; i <= n; i++) {
        f[i][0] = 1;
        for (int j = 1; j <= n; j++) {
            f[i][j] = add(f[i-1][j], mul(f[i-1][j-1], s[i]));
        }
    }
    
    int total = 0;
    for (int i = 1; i <= n; i++) total = add(total, f[n][i]);
    for (int k = 1; k <= m; k++) {
        memset(g, 0, sizeof g);
        g[0][N] = 1;
        for (int i = 1; i <= n; i++)
            for (int j = 0; j+1 <= 2*N; j++) {
                int v = 0;
                if (j) v = mul(g[i-1][j-1], a[i][k]);
                g[i][j] = add(g[i-1][j], add(v, mul(g[i-1][j+1], sub(s[i], a[i][k]))));
            }
        
        for (int j = 1; j <= n; j++) total = sub(total, g[n][j+N]);
    }
    printf("%d\n", (total + P) % P);
    return 0;
}
```

