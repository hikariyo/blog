---
date: 2023-06-20 20:56:43
title: 动态规划 - 数字三角形
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 简介

数字三角形类问题的特点是当前状态可以由上一行计算出的状态得出，在之前写的一篇[文章](https://blog.hikariyo.net/posts/oi/dp/linear/)中记录过，本文记录两道基于数字三角形模型的实际问题。

## 方格取数

> 设有 N×N 的方格图，我们在其中的某些方格中填入正整数，而其它的方格中则放入数字0。
>
> 某人从图中的左上角 A 出发，可以向下行走，也可以向右行走，直到到达右下角的 B 点。
>
> 在走过的路上，他可以取走方格中的数（取走后的方格中将变为数字0）。
>
> 此人从 A 点到 B 点共走了两次，试找出两条这样的路径，使得取得的数字和为最大。
>
> [原题链接 AcWing 1027](https://www.acwing.com/problem/content/1029/)，这里就不把图也贴过来了，大家可以去 AcWing 看。

本题并不能分别两次取最值，两次分别取最大值并不等同于最终的最大值，如下：

```text
1 0 1
2 2 2
0 1 1
```

如果分别走两次求最大值，是不能把数全部取完的，但实际上我们是取掉所有的数，所以单纯用二维的状态表示解决这个问题是不可能的，需要让这两个点同时移动，状态分析如下：

+ 状态表示 $f(i_1, j_1, i_2, j_2)$：

  + 含义：从原点出发，同时到达 $(i_1, j_1)$ 和 $(i_2, j_2)$ 所有路线的集合。
  + 存储：这些路线中取到点数的最大值。
  + 补充：因为这两个点是同时移动的，所以满足 $i_1+j_1=i_2+j_2$。

+ 状态转移方程：
  $$
  \begin{aligned}
  f(i_1,j_1,i_2,j_2) &= \max\{\\
  f(i_1-1,&j_1,i_2-1,j_2),\\
  f(i_1-1,&j_1,i_2,j_2-1),\\
  f(i_1,j_1-1&,i_2-1,j_2),\\
  f(i_1,j_1-1&,i_2,j_2-1),\\
  \} +w(i_1,j_1) &+ w(i_2,j_2)
  \end{aligned}
  $$
  当 $(i_1,j_1)$ 和 $(i_2,j_2)$ 是同一个点时不可以累加，这里需要特判一下。

+ 优化：既然 $i_1+j_1=i_2+j_2=k$，把它们的和 $k$ 存起来，这样只用三个值就可以表示出所有状态了。对于任何 $f(i_1,j_1,i_2,j_2)=f(k,i_1,i_2)$，同样可以得到 $f(i_1-1,j_1,i_2-1,j_2)=f(k-1,i_1-1,i_2-1)$。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 15;
int w[N][N], f[N*2][N][N];

int main() {
    int n, a, b, c; cin >> n;
    while (cin >> a >> b >> c, a || b || c) {
        w[a][b] = c;
    }
    
    for (int k = 2; k <= n*2; k++) {
        for (int i1 = 1; i1 <= n && k-i1 >= 1; i1++) {
            for (int i2 = 1; i2 <= n && k-i2 >= 1; i2++) {
                int j1 = k-i1, j2 = k-i2;
                // 两点相同
                if (i1 == i2) {
                    f[k][i1][i2] = max({
                        f[k-1][i1-1][i2-1],
                        f[k-1][i1][i2],
                        f[k-1][i1-1][i2],
                        f[k-1][i1][i2-1],
                    }) + w[i1][j1];
                } else {
                    f[k][i1][i2] = max({
                        f[k-1][i1-1][i2-1],
                        f[k-1][i1][i2],
                        f[k-1][i1-1][i2],
                        f[k-1][i1][i2-1],
                    }) + w[i1][j1] + w[i2][j2];
                }
            }
        }
    }
    
    cout << f[2*n][n][n] << endl;
    
    return 0;
}
```

观察到每次 `k` 的计算只用了上一次，所以可以用滚动数组优化；对于两种不同的情况存在相同代码，可以分离出来，所以优化后代码如下：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 15;
int w[N][N], f[2][N][N];

int main() {
    int n, a, b, c; cin >> n;
    while (cin >> a >> b >> c, a || b || c) {
        w[a][b] = c;
    }
    
    for (int k = 2; k <= n*2; k++) {
        for (int i1 = 1; i1 <= n && k-i1 >= 1; i1++) {
            for (int i2 = 1; i2 <= n && k-i2 >= 1; i2++) {
                int j1 = k-i1, j2 = k-i2;
                f[k&1][i1][i2] =  max({
                    f[k-1&1][i1-1][i2-1],
                    f[k-1&1][i1][i2],
                    f[k-1&1][i1-1][i2],
                    f[k-1&1][i1][i2-1],
                }) + w[i1][j1];
                
                if (i1 != i2) f[k&1][i1][i2] += w[i2][j2];
            }
        }
    }
    
    cout << f[2*n&1][n][n] << endl;
    
    return 0;
}
```

## 传纸条

> 题目太长这里只放链接：[AcWing 275. 传纸条](https://www.acwing.com/problem/content/277/)

这题和方格取数代码基本一样，虽然这题两步不能相交，但是由于我们要求的是一个**最大值**，而一个格取两遍的时候第二遍它可以看作 0，由于每个格子都是非负数，那么此时绕道走一定**不会使方案变得更差**，所以相不相交无所谓。这里直接写用滚动数组优化过的版本了：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 55;
int f[2][N][N], w[N][N];

int main() {
    int n, m;
    cin >> m >> n;
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            cin >> w[i][j];
   
    for (int k = 2; k <= m+n; k++) {
        for (int i1 = 1; i1 <= m && k-i1 >= 1; i1++) {
            for (int i2 = 1; i2 <= m && k-i2 >= 1; i2++) {
                int j1 = k-i1, j2 = k-i2;
                int t = w[i1][j1];
                if (i1 != i2) t += w[i2][j2];
                f[k&1][i1][i2] = max({
                    f[k-1&1][i1-1][i2],
                    f[k-1&1][i1-1][i2-1],
                    f[k-1&1][i1][i2],
                    f[k-1&1][i1][i2-1],
                }) + t;
            }
        }
    }
    
    cout << f[n+m&1][m][m] << endl;
    
    return 0;
}
```

补充：此题也可以用费用流。

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
#include <algorithm>
using namespace std;

const int N = 5010, M = 20010;
int n, m;
int h[N], e[M], ne[M], f[M], w[M], idx;
int pre[N], incf[N], dist[N];
int S, T;
bool st[N];

int get(int x, int y, int p) {
    return 2*(x*m+y)+p;
}

void add0(int a, int b, int c, int d) {
    e[idx] = b, ne[idx] = h[a], f[idx] = c, w[idx] = d, h[a] = idx++;
}

void add(int a, int b, int c, int d) {
    add0(a, b, c, d), add0(b, a, 0, -d);
}

bool SPFA() {
    queue<int> q;
    memset(dist, -0x3f, sizeof dist);
    memset(incf, -0x3f, sizeof incf);
    dist[S] = 0, q.push(S), incf[S] = 0x3f3f3f3f;
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (f[i] && dist[j] < dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                incf[j] = min(incf[t], f[i]);
                pre[j] = i;
                if (!st[j]) st[j] = true, q.push(j);
            }
        }
    }
    return incf[T] > 0;
}

int EK() {
    int cost = 0;
    while (SPFA()) {
        cost += incf[T] * dist[T];
        for (int i = T; i != S; i = e[pre[i] ^ 1]) {
            f[pre[i]] -= incf[T], f[pre[i] ^ 1] += incf[T];
        }
    }
    return cost;
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d", &n, &m);
    S = get(0, 0, 1), T = get(n-1, m-1, 0);
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++) {
            int c; scanf("%d", &c);
            add(get(i, j, 0), get(i, j, 1), 1, c);
            if (i+1 < n) add(get(i, j, 1), get(i+1, j, 0), 1, 0);
            if (j+1 < m) add(get(i, j, 1), get(i, j+1, 0), 1, 0);
        }
    printf("%d\n", EK());
    return 0;
}
```

