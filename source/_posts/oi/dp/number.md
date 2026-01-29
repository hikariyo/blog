---
date: 2023-08-15 19:57:00
updated: 2023-08-15 19:57:00
title: 动态规划 - 数位 DP
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 简介

数位 DP 是求一个区间内满足某个性质的数的个数，这个性质往往和每一位数字有关。

## 度的数量

> 求给定区间 [X,Y] 中满足下列条件的整数个数：这个数恰好等于 K 个互不相等的 B 的整数次幂之和。
>
> 例如，设 X=15,Y=20,K=2,B=2，则有且仅有下列三个数满足题意：
>
> $17=2^4+2^0$
>
> $18=2^4+2^1$
>
> $20=2^4+2^2$
>
> 题目链接：[AcWing 1081](https://www.acwing.com/problem/content/1083/)。

求数 $L\sim R$ 在 $B$ 进制下 $1$ 的个数为 $K$ 的数的数量，转化为 $dp(R)-dp(L-1)$，其中 $dp(x)$ 是 $0\sim x$ 中满足要求的数的数量。

对于 $B$ 进制下数 $x$ 在第 $n$ 位上的数 $a_n$，这里把每一位看成树的结点，延伸出去两条边，左边一条是可以直接计算出所有结果，右边一条是需要看下一位判断。

下面分几种情况：

+ $a_n>1$，对于这剩下的 $n$ 位直接就是有 $\binom{n}{k-last}$ 种选择了，之后没有右子树，直接 `break` 掉就好。
+ $a_n=1$，当前位置只能选 $1,0$，选择 $0$ 是左边，结果是 $\binom{n-1}{k-last}$；选择 $1$ 是右边，无法直接计算，把 $last+1$ 然后继续执行。
+ $a_n=0$，只能选 $0$，默认走右边。

```cpp
#include <cstdio>
#include <vector>
using namespace std;

const int N = 35;
int C[N][N];
int L, R, K, B;

void init() {
    // 初始化 C(0, 0) 是必要的, 递推 C(1, x) 的时候需要用到这一层
    C[0][0] = 1;
    for (int i = 1; i < N; i++) {
        C[i][0] = 1;
        for (int j = 1; j <= i; j++)
            C[i][j] = C[i-1][j] + C[i-1][j-1];
    }
}

int dp(int n) {
    // 本题这句话加不加无所谓 
    if (n == 0) return 0;
    
    vector<int> nums;
    while (n) nums.push_back(n % B), n /= B;
    
    // 从最高位向下
    int res = 0, last = 0;
    for (int i = nums.size() - 1; i >= 0; i--) {
        int x = nums[i];
        
        if (x > 1) {
            // 没有右子树, 直接计算出结果
            res += C[i+1][K-last];
            break;
        }
        else if (x > 0) {
            // 选 0 是左子树
            res += C[i][K-last];
            
            // 选 1 是右子树, 当前无法计算结果
            last++;
            if (last > K) break;  // 不存在右子树
        }
        
        
        // 如果 last > K 代表它下面就不存在任何方案了, 早在之前就 break 了
        // 如果 last < K 代表原数中 1 的个数不足 K 个, 这不是合法方案
        // 如果 last = K 代表原数中 1 的个数恰为 K 个, n 这个数本身并没有被统计过, 这里加上它
        if (i == 0 && last == K) res++;
    }
    
    return res;
}

int main() {
    init();
    scanf("%d%d%d%d", &L, &R, &K, &B);
    printf("%d\n", dp(R) - dp(L-1));
    return 0;
}
```

## Windy 数

> Windy 定义了一种 Windy 数：不含前导零且相邻两个数字之差至少为 2 的正整数被称为Windy 数。
>
> 在 A 和 B 之间，包括 A 和 B，总共有多少个 Windy 数？
>
> 题目链接：[AcWing 1083](https://www.acwing.com/problem/content/1085/)，[P2657](https://www.luogu.com.cn/problem/P2657)。

本题对于前导零的处理是比较烦人的，首先还是初始化的时候需要一步 DP。

+ 状态表示 $f(i,j)$：长度为 $i$，最高位为 $j$ 数字中符合要求的所有数字的数量，包含前导零。
+ 状态转移方程：$f(i,j)=\sum_{|k-j|\ge 2} f(i-1,k)$
+ 初始化：$\forall j\in[0,9],f(1,j)=1$

在这个 DP 中无需考虑前导零，因为在接下来的数位 DP 中需要处理这个问题。

```cpp
#include <cstdio>
#include <cmath>
using namespace std;

const int N = 15;
int f[N][N], nums[N], cnt;

void init() {
    for (int j = 0; j <= 9; j++) f[1][j] = 1;
    for (int i = 2; i < N; i++)
        for (int j = 0; j <= 9; j++)
            for (int k = 0; k <= 9; k++)
                if (abs(k-j) >= 2)
                    f[i][j] += f[i-1][k];
}

int dp(int n) {
    // 本函数不认为 0 是一个合法的数字
    if (n == 0) return 0;
    
    cnt = 0;
    while (n) nums[cnt++] = n % 10, n /= 10;
    
    int res = 0, last = -2;
    for (int i = cnt-1; i >= 0; i--) {
        int x = nums[i];
        
        // 如果是最高位从 1 开始枚举, 避免前导零
        for (int j = (cnt-1 == i); j < x; j++)
            if (abs(j-last) >= 2) res += f[i+1][j];
        
        // 如果 x 不满足不代表着比当前要小的数不满
        // 因此判断右子树的存在性应当放在循环后面
        if (abs(x - last) < 2) break;
        last = x;
        // 加上 n 这个数本身
        if (i == 0) res++;
    }
    
    // 处理最高位不是 1 的情况
    for (int i = 1; i < cnt; i++)
        for (int j = 1; j <= 9; j++)
            res += f[i][j];
    
    return res;
}

int main() {
    init();
    int l, r; scanf("%d%d", &l, &r);
    printf("%d\n", dp(r) - dp(l-1));
    return 0;
}
```

## 数字游戏I

> 某人命名了一种不降数，这种数字必须满足从左到右各位数字呈非下降关系，如 123，446。
>
> 现在大家决定玩一个游戏，指定一个整数闭区间 [a,b]，问这个区间内有多少个不降数。
>
> 题目链接：[AcWing 1082](https://www.acwing.com/problem/content/description/1084/)。

同样是把数字按位处理，每位看作一个结点，左边是可以直接算出结果的，右边是需要继续处理的。

对于第 $n$ 位 $a_n$，枚举所有小于它且大于上一位的数 $j$，加上最高位为 $j$，长度为 $n$ 的所有不降数 $f(j,n)$。

关于不降数的个数，这个还是用 DP 来求。

+ 状态表示 $f(j,n)$：最高位为 $j$，长度为 $n$ 的所有数的个数。

+ 状态转移方程：下一位应该比当前位更大。
    $$
    f(j,n)=\sum_{k=j}^{9} f(k,n-1)
    $$

+ 初始化：$\forall j\in[0,9],f(j,1)=1$

```cpp
#include <cstdio>
using namespace std;

const int N = 15;
int f[10][N];
int nums[N], cnt;

void init() {
    for (int j = 0; j <= 9; j++) f[j][1] = 1;
    for (int n = 2; n < N; n++)
        for (int j = 0; j <= 9; j++)
            for (int k = j; k <= 9; k++)
                f[j][n] += f[k][n-1];
}

int dp(int n) {
    if (n == 0) return 1;
    
    cnt = 0;
    while (n) nums[cnt++] = n % 10, n /= 10;
    
    int res = 0, last = 0;
    for (int i = cnt-1; i >= 0; i--) {
        int x = nums[i];
        if (x < last) break;
        for (int j = last; j < x; j++)
            res += f[j][i+1];
        last = x;
        if (!i) res++;
    }
    return res;
}

int main() {
    init();
    int l, r;
    while (~scanf("%d%d", &l, &r)) 
        printf("%d\n", dp(r) - dp(l-1));
    return 0;
}
```

## 数字游戏 II

> 某人又命名了一种取模数，这种数字必须满足各位数字之和 $\text{mod } N$ 为 0。
>
> 指定一个整数闭区间 $[a, b]$，问这个区间内有多少个取模数。
>
> 题目链接：[AcWing 1084](https://www.acwing.com/problem/content/1086/)。

如果固定最高位 $x$，且枚举下一位 $j$，那么这个数字是 $xj\cdots$，设后面几位的数位之和为 $S$，那么此时有 $x+j+S \equiv0(\text{mod }N)$，即 $S\equiv -x-j(\text{mod }N)$，于是我们应该知道的是当最高位为 $j$，且数位之和模 $N$ 为某个值的所有数字数量。

+ 状态表示 $f(i,j,m)$：共有 $i$ 位，最高位是 $j$，数位之和模 $N$ 为 $m$ 的所有数的个数。
+ 状态转移方程：$f(i,j,m)=\sum_{k=0}^9f(i-1,k,(m-j)\bmod N)$
+ 初始化：$\forall j,m,f(1,j,m)=[j\text { mod }N =m]$

```cpp
#include <cstdio>
#include <cstring>
using namespace std;

const int N = 15, M = 100;
int f[N][N][M], p;
int nums[N], cnt;

void init() {
    for (int j = 0; j <= 9; j++)
        f[1][j][j % p]++;
    
    for (int i = 2; i < N; i++)
        for (int m = 0; m < p; m++)
            for (int j = 0; j <= 9; j++)
                for (int k = 0; k <= 9; k++)
                    f[i][j][m] += f[i-1][k][((m-j)%p+p)%p];
}

int dp(int n) {
    if (n == 0) return 1;
    
    cnt = 0;
    while (n) nums[cnt++] = n % 10, n /= 10;
    int res = 0, last = 0;
    for (int i = cnt-1; i >= 0; i--) {
        int x = nums[i];
        for (int j = 0; j < x; j++)
            res += f[i+1][j][((-last)%p+p)%p];
        // last 代表前缀数位之和
        last += x;
        if (i == 0 && last % p == 0) res++;
    }
    return res;
}

int main() {
    int l, r;
    while (~scanf("%d%d%d", &l, &r, &p)) {
        memset(f, 0, sizeof f);
        init();
        printf("%d\n", dp(r) - dp(l-1));
    }
    return 0;
}
```

## 不要62

> 给定区间 $[n,m]$ 求不包含连续 62 或 4 的数字的数量。
>
> 题目链接：[AcWing 1085](https://www.acwing.com/problem/content/1087/)。

同样是预处理出来 $f(i,j)$，表示长度为 $i$ 最高位为 $j$ 的符合条件的所有数。

```cpp
#include <cstdio>
using namespace std;

const int N = 15;
int f[N][N], nums[N], cnt;

void init() {
    for (int j = 0; j <= 9; j++) f[1][j] = 1;
    f[1][4] = 0;
    
    for (int i = 2; i < N; i++)
        for (int j = 0; j <= 9; j++)
            for (int k = 0; k <= 9; k++) {
                if (j == 4 || k == 4) continue;
                if (j == 6 && k == 2) continue;
                f[i][j] += f[i-1][k];
            }
}

int dp(int n) {
    if (n == 0) return 1;
    
    cnt = 0;
    while (n) nums[cnt++] = n % 10, n /= 10;
    
    int res = 0, last = 0;
    for (int i = cnt-1; i >= 0; i--) {
        int x = nums[i];
        
        for (int j = 0; j < x; j++) {
            if (last == 6 && j == 2) continue;
            if (j == 4) continue;
            res += f[i+1][j];
        }
        
        if (x == 4) break;
        if (last == 6 && x == 2) break;
        if (i == 0) res++;
        last = x;
    }
    
    return res;
}

int main() {
    init();
    int l, r;
    while (scanf("%d%d", &l, &r), l || r) {
        printf("%d\n", dp(r) - dp(l-1));
    }
    return 0;
}
```

## 计数问题

> 给定两个整数 a 和 b，求 a 和 b 之间的所有数字中 0∼9 的出现次数。
>
> 例如，a=1024，b=1032，则 a 和 b 之间共有 99 个数如下：
>
> ```
> 1024 1025 1026 1027 1028 1029 1030 1031 1032
> ```
>
> 其中 `0` 出现 10 次，`1` 出现 10 次，`2` 出现 7 次，`3` 出现 3 次等等…
>
> 题目链接：[AcWing 338](https://www.acwing.com/problem/content/340/)。

预处理出来 $f(i,j,k)$ 表示长度为 $i$ 最高位是 $j$ 中 $k$ 出现的次数，那么转移方程就是：
$$
f(i,j,k)=\left\{\begin{aligned}
&\sum_{u=0}^9 f(i-1,u,k),j\ne k\\
&\sum_{u=0}^9 f(i-1,u,k)+10^{i-1},j=k\\
\end{aligned}\right.
$$
当 $j=k$，例如当 $i=4$ 时 $j000\sim j999$ 这总共 $10^3$ 个数也是需要选上的。

初始化照样还是 $\forall j\in[0,9], f(1,j,j)=1$

对于一个数字的每一位，设当前枚举到 $a_i$ 这里分三种情况考虑，以统计 $334$ 中 $3$ 的个数为例。

1. 不考虑前导零。

    1. 对于最高位 $x_2=3$，枚举 $j\in[1,2]$，累加 $f(3,j,3)$ 到答案中，这包括了 $100\sim 299$ 全部数字中包含 $k$ 的个数。

        其次，发现 $x_2=k$，累加 $300\sim 333$ 这些数字的个数（共有 $34=334\bmod 100$ 个），并将计数器自增。注意这里*只看最高位*包含 $k$ 的数量，并且不考虑 $334$ 本身。

    2. 对于接下来 $x_1=3$，枚举 $j\in[0,2]$，累加 $f(2,j,3)$ 到答案中，这包括了 $300\sim 329$ 中*后两位* 包含 $k$ 的数量。

        其次，发现 $x_1=k$，累加 $330\sim 333$（共有 $4=334\bmod 10$ 个）这些数字的个数，这里*只看第二位*包含 $k$ 的数量。

    3. 对于接下来 $x_0=4$，枚举 $j\in[0,3]$，累加 $f(1,j,3)$ 到答案中，这包括了 $330\sim 333$ 中*后一位*包含 $k$ 的数量。

    4. 发现到了末尾，考虑 $334$ 这个数字本身，把计数器累加到答案中。

2. 考虑前导零。

    1. 枚举长度 $i\in[1,2]$，最高位数 $j\in[1,9]$，累加 $f(i,j,3)$ 到答案中。

        它们代表 $1\sim 9,10\sim 99$ 中 $k$ 的个数。

3. 因此，我们在不考虑前导零时，枚举出了 $100\sim 334$ 中所有 $k$ 的个数，接下来在考虑前导零时，枚举出了 $1\sim 99$ 中 $k$ 的个数，因此答案是正确的。

接下来进行抽象化的描述。

1. 对于 $a_i,i\in[0,n-1]$ 也就是 $i-1$ 位，不考虑其为最高位时的情况，首先将所有以 $0\sim a_i-1$ 开头，长度为 $i+1$ 的数字中 $k$ 的个数累加到答案中，这是左子树。

2. 其次考虑以 $a_i$ 开头，如果 $a_i\ne k$ 直接进入右子树。

3. 若 $a_i=k$，需要计算第 $i-1$ 位是 $a_i$ 到 $(n-1)$ 的所有数中第 $i-1$ 位为 $k$ 的数字的个数，也就是上文中的 $300\sim 333,330\sim 333$ 这就是 $n \bmod 10^i$ ，接下来进入右子树，统计 $a_{i-1}$ 位的信息。

    这是因为进入 $i-2$ 层后它会帮我们统计出以 $a_i$ 为第 $i-1$ 位，在 $i-2$ 位后面 $k$ 的数量，所以不能再这个分支中把它处理掉，应该扔给右子树处理。

4. 如果发现走到头了，就把 $n$ 这个数字本身含有 $k$ 的数量加上。

5. 处理完这些后，把所有含前导零的合法方案累加上。

```cpp
#include <algorithm>
#include <cstdio>
#include <vector>
#include <cmath>
using namespace std;

const int N = 15;
int f[N][N][N], nums[N], idx;

void init() {
    for (int j = 0; j <= 9; j++) {
        f[1][j][j] = 1;
    }
    
    for (int k = 0; k <= 9; k++) {
        for (int i = 2; i < N; i++) {
            for (int j = 0; j <= 9; j++) {
                for (int u = 0; u <= 9; u++) {
                    f[i][j][k] += f[i-1][u][k];
                }
                // 当前位后面可以 j000~j999 一共 pow10(i-1) 种选择
                if (j == k) f[i][j][k] += pow(10, i-1);
            }
        }
    }
}

int dp(int n, int k) {
    // 这个 dp 函数本身不包括前导 0，因此单个的 0 是永远不会被算上的, 比如 dp(10, 0) 返回 1
    // 所以不用特殊处理
    if (n == 0) return 0;
    
    idx = 0;
    while (n) nums[idx++] = n % 10, n /= 10;
    
    int res = 0, cnt = 0;
    for (int i = idx - 1; i >= 0; i--) {
        int x = nums[i];
        
        // 特判首位
        for (int j = (idx - 1 == i); j < x; j++)
            res += f[i+1][j][k];
        
        
        if (x == k) {
            cnt++;
            int p = 0;
            for (int z = i-1; z >= 0; z--)
                p = p * 10 + nums[z];
            res += p;
        }
        
        if (i == 0) res += cnt;
    }
    
    // 加上前导零的情况
    for (int i = 0; i < idx; i++)
        for (int j = 1; j <= 9; j++)
            res += f[i][j][k];
    
    return res;
}

int main() {
    init();
    int l, r;
    while (scanf("%d%d", &l, &r), l || r) {
        if (l > r) swap(l, r);
        for (int k = 0; k <= 9; k++)
            printf("%d ", dp(r, k) - dp(l-1, k));
        puts("");
    }
    return 0;
}
```

## 数数（待补充）

本题是不要 62 的拓展，使用 AC 自动机 + 数位 DP，不过这里先跳了，回来再补。
