---
date: 2023-07-11 15:21:10
title: 动态规划 - 状态机模型
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 简介

在 DP 数组中多加一维用来存储状态，进行状态转移的时候也与记录的状态相关。

## 股票买卖 IV

> 给定一个长度为 N 的数组，数组中的第 i 个数字表示一个给定股票在第 i 天的价格。
>
> 设计一个算法来计算你所能获取的最大利润，你最多可以完成 k 笔交易。
>
> 注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。一次买入卖出合为一笔交易。
>
> [原题链接](https://www.acwing.com/problem/content/1059/)。

+ 状态表示 $f(i,j,s)$：

    + 含义：在前 $i$ 天交易次数为 $j$ 且状态为 $s$ 时的利润，其中 $s=0$ 时表示不持股， $s=1$ 时表示持股。
    + 存储：最大值。

+ 状态转移方程：

    + 不持当前股：可能由上一次持股卖出后转移过来，也可能从上一次不持股转移过来，写出来就是：
        $$
        f(i,j,0)=\max\{f(i-1,j,1)+w, f(i-1,j,0)\}
        $$

    + 持有当前股：可能由上一次不持股买入后转移过来，也可能从上一次持股转移过来，写出来是：
        $$
        f(i,j,1)=\max\{f(i-1,j-1,0)-w, f(i-1,j,1)\}
        $$

+ 初始化：对任意前 $i$ 物品，没有交易的时候必定没有利润也无法买进股票，其它都是负无穷。
    $$
    f(i,0,0)=0
    $$

+ 复杂度：循环物品和交易数，故复杂度为 $O(nm)$。

+ 答案：从 $f(n,j,0),j\in[1,m]$ 中取最大值。

```cpp
#include <cstring>
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1e5+10, M = 110;
int f[N][M][2];

int main() {
    int n, m;
    cin >> n >> m;
    
    memset(f, -0x3f, sizeof f);
    for (int i = 0; i <= n; i++) f[i][0][0] = 0;
    
    for (int i = 1; i <= n; i++) {
        int w;
        cin >> w;
        for (int j = 1; j <= m; j++) {
            f[i][j][0] = max(f[i-1][j][1]+w, f[i-1][j][0]);
            f[i][j][1] = max(f[i-1][j-1][0]-w, f[i-1][j][1]);
        }
    }
    
    int ans = 0;
    for (int j = 1; j <= m; j++) ans = max(ans, f[n][j][0]);
    cout << ans;
    
    return 0;
}
```

## 股票买卖 V

> 给定一个长度为 N 的数组，数组中的第 i 个数字表示一个给定股票在第 i 天的价格。
>
> 设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:
>
> - 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
> - 卖出股票后，你无法在第二天买入股票 (即冷冻期为 11 天)。
>
> [原题链接](https://www.acwing.com/problem/content/1060/)。

+ 状态表示 $f(i, s)$：

    + 含义：前 $i$ 个股票状态为 $s$ 的时候所有方案，其中 $s=0$ 表示持有股票，$s=1$ 表示卖出股票的第一天（冷冻期），$s=2$ 表示卖出股票包括第二天以后的天。
    + 存储：最大利润。

+ 状态转移：

    + 持有股票 $s=0$：可以由上一次冷冻期后买入，也可以从上一次持股直接转移：
        $$
        f(i,0)=\max\{f(i-1,2)-w, f(i-1,0)\}
        $$

    + 冷冻期 $s=1$：只可以从上一次持股卖出后进入：
        $$
        f(i,1)=f(i-1,0)+w
        $$

    + 解冻后 $s=2$：可以从冷冻期进入，也可以从上一次没买时进入：
        $$
        f(i,2)=\max\{f(i-1,1),f(i-1,2)\}
        $$

+ 答案：可以从冷冻期取得，也可以从解冻后取得，因此为 $\max\{f(n,1),f(n,2)\}$。

+ 初始化：第 0 股一定不能买入，$f(0,1)=f(0,2)=0$，其它均为负无穷。

+ 复杂度：只遍历股票个数，复杂度为 $O(n)$。

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 1e5+10;
int f[N][3];

int main() {
    int n;
    cin >> n;
    memset(f, -0x3f, sizeof f);
    f[0][1] = f[0][2] = 0;
    
    for (int i = 1; i <= n; i++) {
        int w;
        cin >> w;
        f[i][0] = max(f[i-1][2]-w, f[i-1][0]);
        f[i][1] = f[i-1][0]+w;
        f[i][2] = max(f[i-1][1], f[i-1][2]);
    }
    
    cout << max(f[i][1], f[i][2]);
    return 0;
}
```

## 设计密码

> 你现在需要设计一个密码 S，S 需要满足：
>
> - S 的长度是 N；
> - S 只包含小写英文字母；
> - S 不包含子串 T；
>
> 例如：abc 和 abcde 是 abcde 的子串，abd 不是 abcde 的子串。
>
> 请问共有多少种不同的密码满足要求？
>
> 由于答案会非常大，请输出答案模 `1e9+7` 的余数。
>
> [原题](https://www.acwing.com/problem/content/1054/)。

本题需要先用 KMP 算法求出 $\text{next}$ 数组，然后从小到大循环密码长度，循环 `a~z` 的字符，如果跳转到 $\text{next}$ 数组末尾就意味着当前密码中存在给定子串，这时就不符合条件了。

+ 状态表示 $f(i, j)$：
    + 含义：长度为 $i$ 的密码，在 $\text{next}$ 数组中最终跳转到 $j$ 的所有方案。
    + 存储：方案数。
+ 状态转移方程：$f(i,u)=f(i,u)+f(i-1,j)$，其中 $j\in [0,m)$；$u$ 为某个字母在 $j$ 的基础上对某个字母求 $\text{next}$ 的最终跳转。
+ 初始化：$f(0, 0)=1$，意思是长度为 0 跳转到 0 的方案是一个空字符串，方案数为 1。

```cpp
#include <cstring>
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 55, mod = 1e9+7;
char s[N];
int ne[N], f[N][N], n, m;

int main() {
    cin >> n >> s+1;
    m = strlen(s+1);
    for (int i = 2, j = 0; i < m; i++) {
        while (j && s[j+1] != s[i]) j = ne[j];
        if (s[j+1] == s[i]) j++;
        ne[i] = j;
    }
    
    f[0][0] = 1;
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < m; j++) {
            for (char k = 'a'; k <= 'z'; k++) {
                int u = j;
                while (u && s[u+1] != k) u = ne[u];
                if (s[u+1] == k) u++;
                if (u < m) f[i][u] = (f[i][u] + f[i-1][j]) % mod;
            }
        }
    }
    
    int res = 0;
    for (int j = 0; j < m; j++) res = (res + f[n][j]) % mod;
    cout << res;
    return 0;
}
```



