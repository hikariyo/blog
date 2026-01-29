---
date: 2023-07-05 15:12:12
updated: 2023-07-05 15:12:12
title: 动态规划 - 背包方案
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 背包问题求具体方案

> 有 N 件物品和一个容量是 V 的背包。每件物品只能使用一次。
>
> 第 i 件物品的体积是 vi，价值是 wi。
>
> 求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。
>
> 输出 **字典序最小的方案**。这里的字典序是指：所选物品的编号所构成的序列。物品的编号范围是 1…N。
>
> 链接：[AcWing 12](https://www.acwing.com/problem/content/12/)。

当判断当前物品 `i` 是否取时，判断下列是否成立即可：
$$
j \ge v_i \and f(i,j)=f(i-1,j-v_i)+w_i
$$
但是，我们要让字典序最小，因此这里不能从前向后求，这样我们只能从后向前推出每个取出的物品，而方案不一定唯一，所以需要**从后向前求解**，这样就可以保证从前向后判断每个物品是否要取了，因此上面的式子需要更正为：
$$
j \ge v_i \and f(i,j)=f(i+1,j-v_i)+w_i
$$

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010;
int f[N][N], w[N], v[N];
int n, m;

int main() {
    cin >> n >> m;

    // 注意读入因为顺序原因不可以放到循环内部
    for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];
    
    for (int i = n; i >= 1; i--) {
        for (int j = 1; j <= m; j++) {
            if (j >= v[i]) f[i][j] = max(f[i+1][j], f[i+1][j-v[i]]+w[i]);
            else f[i][j] = f[i+1][j];
        }
    }
    
    int j = m;
    for (int i = 1; i <= n; i++) {
        // 当前是从 f(i+1, j-vi) 转移过来 因此可以取走 i
        if (j >= v[i] && f[i][j] == f[i+1][j-v[i]] + w[i]) {
            cout << i << ' ';
            j -= v[i];
        }
    }
    return 0;
}
```

## 背包问题求方案数

> 有 N 件物品和一个容量是 V 的背包。每件物品只能使用一次。
>
> 第 i 件物品的体积是 vi，价值是 wi。
>
> 求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。
>
> 输出 **最优选法的方案数**。注意答案可能很大，请输出答案模 $10^9+7$ 的结果。
>
> 链接：[AcWing 11](https://www.acwing.com/problem/content/description/11/)。

二维朴素版：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010, mod=1e9+7;
int f[N][N], g[N][N];

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int n, m;
    cin >> n >> m;
    for (int i = 0; i <= n; i++) g[i][0] = 1;
    for (int j = 0; j <= m; j++) g[0][j] = 1;
    
    for (int i = 1; i <= n; i++) {
        int v, w;
        cin >> v >> w;
        for (int j = 1; j <= m; j++) {
            int maxv = f[i-1][j];
            if (j >= v) maxv = max(maxv, f[i-1][j-v]+w);
            if (maxv == f[i-1][j]) g[i][j] = g[i-1][j] % mod;
            if (maxv == f[i-1][j-v]+w) g[i][j] = (g[i][j] + g[i-1][j-v]) % mod;
            f[i][j] = maxv;
        }
    }
    cout << g[n][m];
    return 0;
}
```

优化成一维：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010, mod=1e9+7;
int f[N], g[N];

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int n, m;
    cin >> n >> m;
    // g(0, j) = 1
    for (int j = 0; j <= m; j++) g[j] = 1;
    
    for (int i = 1; i <= n; i++) {
        int v, w;
        cin >> v >> w;
        for (int j = m; j >= v; j--) {
            if (f[j-v]+w > f[j]) {
                f[j] = f[j-v]+w;
                g[j] = g[j-v];
            } else if (f[j-v]+w == f[j]) {
                // f[j] = f[j];
                g[j] = (g[j-v] + g[j]) % mod;
            } /*else {
                g[j] = g[j];
                f[j] = f[j];
            }*/
        }
    }
    cout << g[m];
    return 0;
}
```

这里推荐大家就写 `if`, `else if`，不要像二维那样写，要不就会有稀奇古怪的问题。
