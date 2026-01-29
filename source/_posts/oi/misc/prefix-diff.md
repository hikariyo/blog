---
date: 2023-06-13 15:00:00
updated: 2023-07-25 22:11:00
title: 杂项 - 前缀和 差分
katex: true
tags:
- Algo
- C++
categories:
- OI
description: 前缀和用来求一个区间内的和，差分用来对一个区间加减常数的操作减少复杂度，利用了高中数学中学过的数列前N项和的性质。
---

## 前缀和

其实就是数列前 $n$ 项和，可以用来快速求一段数字的和。对于数列 $\{a_n\}$，有下面的式子：
$$
\begin{aligned}
S_0&=0\\
S_{n}&=\sum_{i=0}^na_i\\
&=S_{n-1}+a_n
\end{aligned}
$$
虽然在高中数学的范畴内或许不可以写 $S_0$，但是我们可以定义 $S_0=0$ 来避免判断边界情况。由上式可以推导出：
$$
\begin{aligned}
S_{p..q} &= \sum_{i=p}^qa_i\\
&=S_q-S_p+a_p\\
&=S_q-(S_p-a_p)\\
&=S_q-S_{p-1}
\end{aligned}
$$
因为直接写 $S_{p-1}$ 有些不太直观，这里就多写了两步。

## 求范围和

> 输入一个长度为 $n \in [1, 10^5]$ 的整数序列。
>
> 接下来再输入 $m \in [1, 10^5]$ 个询问，每个询问输入一对 $l, r \in [1, n]$。
>
> 对于每个询问，输出原序列中从第 $l$ 个数到第 $r$ 个数的和。

暴力解法显然可以行得通，但是我们利用前缀和可以大幅度优化。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1e5+10;
int n, m;
int a[N], S[N];

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", a+i);
    for (int i = 1; i <= n; i++) S[n] = a[n] + S[n-1];
    
    int l, r;
    while (m--) {
        scanf("%d%d", &l, &r);
        printf("%d\n", S[r] - S[l-1]);
    }
    return 0;
}
```

1. 全局数组默认初始化为 0，满足 $S_0=0$。
2. 这里下标不从 0 开始，多开辟一点空间可以省去很多边界问题。

## 求子矩阵和

> 输入一个 $n \in [1, 1000]$ 行 $m \in [1, 1000]$ 列的整数矩阵，再输入 $q \in [1, 2\times10^5]$ 个询问，每个询问包含四个整数 $x_1,y_1,x_2,y_2$，表示一个子矩阵的左上角坐标和右下角坐标，对于每个询问输出子矩阵中所有数的和。

可以仿照前缀和来实现一个二维的"前缀和"。对于矩阵 $\bold A$ 中的元素 $a_{ij}$ 可以定义矩阵 $\bold S$ 满足其中元素 $s_{ij}$ 是从 $a_{11} \to a_{ij}$ 的所有元素之和。可以得到 $s_{ij}=a_{ij}+s_{i-1,j}+s_{i,j-1}-s_{i-1,j-1}$

那么可以求由 $x_1,y_1$ 与 $x_2,y_2$ 构成的矩形中所有元素的和为 $s[x_2][y_2]-s[x_1-1][y_2]-s[x_2][y_1-1]+s[x_1-1][y_1-1]$，这里就用数组的符号了，方便观察，如果实在难理解的话画个图就看明白了。

```cpp
#include <bits/stdc++.h>
using namespace std;

int n, m, q;

const int N = 1010;
int a[N][N], S[N][N];


int main() {
    scanf("%d%d%d", &n, &m, &q);
    for (int i = 1; i <= n; i++) for (int j = 1; j <= m; j++) scanf("%d", &a[i][j]);
    for (int i = 1; i <= n; i++) for (int j = 1; j <= m; j++) S[i][j] = a[i][j] + S[i-1][j] + S[i][j-1] - S[i-1][j-1];

    int x1, y1, x2, y2;
    while (q--) {
        scanf("%d%d%d%d", &x1, &y1, &x2, &y2);
        printf("%d\n", S[x2][y2] - S[x1-1][y2] - S[x2][y1-1] + S[x1-1][y1-1]);
    }
    return 0;
}
```

## 差分

> 输入一个长度为 $n \in [1, 10^5]$ 的整数序列。
>
> 接下来输入 $m \in [1, 10^5]$ 个操作，每个操作包含三个整数 $l, r \in [0, n], c \in [-10^3, 10^3]$，表示将序列中 $[l,r]$ 之间的每个数加上 $c$。
>
> 请你输出进行完所有操作后的序列。

差分就是对数列 $\{a_n\}$ 构造 $\{b_n\}$ 使得前者是后者的前缀和数列，与求前缀和的操作互为逆运算。

对数列 $\{a_n\}$ 的区间 $[l, r]$ 加一个常数 $c$ 等价于 $b_l+c, b_{r+1}-c$，这样就使一个 $O(n)$ 的操作降到 $O(1)$，只需要在最后遍历一遍。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1e5+10;
int n, m;
int a[N], b[N];

int main() {
	scanf("%d%d", &n, &m);
	for (int i = 1; i <= n; i++) scanf("%d", a+i);
	for (int i = 1; i <= n; i++) b[i] = a[i] - a[i-1];
	int l, r, c;
	while (m--) {
		scanf("%d%d%d", &l, &r, &c);
		b[l] += c;
		b[r+1] -= c;
	}
	for (int i = 1; i <= n; i++) {
		b[i] = b[i] + b[i-1];
		printf("%d ", b[i]);
	}
	return 0;
}
```

也可以不手动构造差分数组，如下：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1e5+10;
int n, m;
int a[N], b[N];

void insert(int l, int r, int c) {
    b[l] += c;
    b[r+1] -= c;
}

int main() {
	scanf("%d%d", &n, &m);
	for (int i = 1; i <= n; i++) scanf("%d", a+i);
	for (int i = 1; i <= n; i++) insert(i, i, a[i]);
	int l, r, c;
	while (m--) {
		scanf("%d%d%d", &l, &r, &c);
        insert(l, r, c);
	}
	for (int i = 1; i <= n; i++) {
		b[i] = b[i] + b[i-1];
		printf("%d ", b[i]);
	}
	return 0;
}
```

完全可以使用 `insert` 来插入原始数据。

## 矩阵差分

同样，对于一个矩阵的差分矩阵，想要在 $(x_1, y_1) \to (x_2, y_2)$ 的范围内加常数 $c$，可以使差分矩阵 
$$
\begin{aligned}
&b[x_1][y_1]+c\\
&b[x_2+1][y_1]-c\\
&b[x_1][y_2+1]-c\\
&b[x_2+1][y_2+1]+c
\end{aligned}
$$
这里手动构造差分矩阵就很困难了，所以用插入元素的方式来构造。例题要求与一维差分相似，这里就不抄过来了。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1010;

int a[N][N];
int n, m, q;

void insert(int x1, int y1, int x2, int y2, int c) {
    a[x1][y1] += c;
    a[x2+1][y1] -= c;
    a[x1][y2+1] -= c;
    a[x2+1][y2+1] += c;
}

int main() {
    scanf("%d%d%d", &n, &m, &q);
    for (int i = 1; i <= n; i++ ) for (int j = 1; j <= m; j++) {
        int el;
        scanf("%d", &el);
        insert(i, j, i, j, el);
    }
    
    int x1, x2, y1, y2, c;
    
    while (q--) {
        scanf("%d%d%d%d%d", &x1, &y1, &x2, &y2, &c);
        insert(x1, y1, x2, y2, c);
    }
    
    for (int i = 1; i <= n; i++){
        for (int j = 1; j <= m; j++) {
            // 注意这里是 += 因为需要包含自身
            a[i][j] += a[i-1][j] + a[i][j-1] - a[i-1][j-1];
            printf("%d ", a[i][j]);
        }
        printf("\n");
    }
    
    return 0;
}
```

我们只需要用一个数组就足够了，最后再把这个差分数组求前缀和然后输出即可。

## 区间修改求区间和

> 给定一个长度为 N 的数列 A，以及 M 条指令，每条指令可能是以下两种之一：
>
> 1. `C l r d`，表示把 A[l],A[l+1],…,A[r] 都加上 d。
> 2. `Q l r`，表示询问数列中第 l∼r 个数的和。
>
> 对于每个询问，输出一个整数表示答案。
>
> 题目链接：[AcWing 243](https://www.acwing.com/problem/content/description/244/)。

这里用树状数组维护，可以自行前往 OI-WIKI 了解。

对于输入数列 $\{a_n\}$ 的差分数列 $\{b_n\}$，有 $S_n=\sum_{j=1}^n a_j=\sum_{j=1}^n \sum_{i=1}^j b_i$，展开来写：
$$
S_n = \text{sum} \begin{pmatrix}
b_1 & \\
\vdots & \ddots \\
b_1 & \cdots & b_n
\end{pmatrix}
$$
可以把这个三角补全，也就是：
$$
S_n= (\sum_{i=1}^n b_i)(n+1) -\sum_{i=1}^n(i\cdot b_i)
$$


多维护数列 $\{nb_n\}$ 的前缀和就可以了。

```cpp
#include <iostream>
using namespace std;

typedef long long LL;
const int N = 1e5+10;
// b[i]    i*b[i]
LL tr1[N], tr2[N];
int a[N], n, m;

inline int lowbit(int x) {
    return x & -x;
}

inline void add(LL tr[], int p, LL c) {
    for (int i = p; i < N; i += lowbit(i)) {
        tr[i] += c;
    }
}

inline LL query0(LL tr[], int p) {
    LL res = 0;
    for (int i = p; i; i -= lowbit(i)) {
        res += tr[i];
    }
    return res;
}

inline LL query(int p) {
    return query0(tr1, p) * (p+1) - query0(tr2, p);
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", a+i);
    for (int i = 1; i <= n; i++) {
        LL b = a[i]-a[i-1];
        add(tr1, i, b), add(tr2, i, i*b);
    }
    char op[2];
    int l, r, d;
    while (m--) {
        scanf("%s%d%d", op, &l, &r);
        if (*op == 'Q') printf("%lld\n", query(r)-query(l-1));
        else {
            scanf("%d", &d);
            add(tr1, l, d), add(tr2, l, l*(LL)d);
            add(tr1, r+1, -d), add(tr2, r+1, -(r+1)*(LL)d);
        }
    }
    
    return 0;
}
```

