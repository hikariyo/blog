---
date: 2023-12-24 06:52:00
title: 数据结构 - Link Cut Tree
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 简介

LCT 利用 Splay 对树进行虚实链剖分，同一实链上的结点处在同一个 Splay 中，排序的关键字是深度。因此 Splay 的中序遍历就是树的从上到下的有序的树链。虚边是只记录父结点，父结点不记录子结点。介绍一下几个重要函数。

### isroot(x)

判断 $x$ 是否为根，只需要判断 $fa(x)$ 是否含有 $x$ 的指针即可。

```cpp
bool isroot(int x) {
    return ch[fa[x]][0] != x && ch[fa[x]][1] != x;
}
```

### rotate(x)

与普通 Splay 不同的地方在于它需要先判断是否为根。

```cpp
void rotate(int x) {
    int y = fa[x], z = fa[y];
    int k = ch[y][1] == x;
    if (!isroot(y)) ch[z][ch[z][1] == y] = x;
    fa[x] = z;
    ch[y][k] = ch[x][k^1], fa[ch[x][k^1]] = y;
    ch[x][k^1] = y, fa[y] = x;
    pushup(y), pushup(x);
}
```

### splay(x)

进行 Splay 操作之前需要把一路上所有结点懒标记下放，操作之后 $x$ 是 Splay 的根。

```cpp
void splay(int x) {
    static int stk[N];
    int tp = 0, u = x;
    stk[++tp] = u;
    while (!isroot(u)) stk[++tp] = u = fa[u];
    while (tp) pushdown(stk[tp--]);

    while (!isroot(x)) {
        int y = fa[x], z = fa[y];
        if (!isroot(y)) {
            if ((ch[y][1] == x) ^ (ch[z][1] == y)) rotate(x);
            else rotate(y);
        }
        rotate(x);
    }
}
```

### access(x)

打通 $x$ 所在树中根到 $x$ 的实链，同时把 $x$ 下面都变成虚边，操作之后 $x$ 是 Splay 的根。

```cpp
void access(int x) {
    int z = x;
    for (int y = 0; x; y = x, x = fa[x]) {
        splay(x);
        ch[x][1] = y;
        pushup(x);
    }
    splay(z);
}
```

### makeroot(x)

将 $x$ 设为当前树上的根，当 `access(x)` 后，$x$ 所在 Splay 中最大的就是 $x$ 了，所以翻转一下，操作之后 $x$ 是 Splay 的根。

```cpp
void makeroot(int x) {
    access(x);
    pushrev(x);
}
```

### findroot(x)

其实就是找前驱，操作之后树的根是 Splay 的根。

```cpp
int findroot(int x) {
    access(x);
    while (ch[x][0]) pushdown(x), x = ch[x][0];
    splay(x);
    return x;
}
```

### split(x, y)

抽离出 $x,y$ 这条链，操作之后 $y$ 是 Splay 的根。

```cpp
void split(int x, int y) {
    makeroot(x);
    access(y);
}
```

### link(x,y)

加边，这里如果 $x,y$ 不在同一个树中，建立 $x\to y$ 的虚边。

```cpp
void link(int x, int y) {
    makeroot(x);
    if (findroot(y) != x) fa[x] = y;
}
```

### cut(x,y)

删边，首先 $x$ 是 $y$ 所在树的根，其次 $fa(y)$ 直接等于 $x$ 并且 $y$ 没有左子树，即没有比 $y$ 更接近 $x$ 的结点时才删掉。

```cpp
void cut(int x, int y) {
    makeroot(x);
    if (findroot(y) == x && fa[y] == x && !ch[y][0]) {
        ch[x][1] = fa[y] = 0;
        pushup(x);
    }
}
```

## 模板

> 给定 $n$ 个点以及每个点的权值，要你处理接下来的 $m$ 个操作。  
> 操作有四种，操作从 $0$ 到 $3$ 编号。点从 $1$ 到 $n$ 编号。
>
>
> - `0 x y` 代表询问从 $x$ 到 $y$ 的路径上的点的权值的 $\text{xor}$ 和。保证 $x$ 到 $y$ 是联通的。
> - `1 x y` 代表连接 $x$ 到 $y$，若 $x$ 到 $y$ 已经联通则无需连接。
>
> 数据范围：
>
> - $1 \leq n \leq 10^5$，$1 \leq m \leq 3 \times 10^5$，$1 \leq a_i \leq 10^9$。
> - 对于操作 $0, 1, 2$，保证 $1 \leq x, y \leq n$。
> - 对于操作 $3$，保证 $1 \leq x \leq n$，$1 \leq y \leq 10^9$。
>
> 题目链接：[P3690](https://www.luogu.com.cn/problem/P3690)。

板子，上面都说了。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;
int n, m, fa[N], ch[N][2], sum[N], val[N], rev[N];

bool isroot(int x) {
    return ch[fa[x]][0] != x && ch[fa[x]][1] != x;
}

void pushup(int x) {
    sum[x] = sum[ch[x][0]] ^ val[x] ^ sum[ch[x][1]];
}

void pushrev(int x) {
    rev[x] ^= 1;
    swap(ch[x][0], ch[x][1]);
}

void pushdown(int x) {
    if (!rev[x]) return;
    pushrev(ch[x][0]), pushrev(ch[x][1]);
    rev[x] = 0;
}

void rotate(int x) {
    int y = fa[x], z = fa[y];
    int k = ch[y][1] == x;
    if (!isroot(y)) ch[z][ch[z][1] == y] = x;
    fa[x] = z;
    ch[y][k] = ch[x][k^1], fa[ch[x][k^1]] = y;
    ch[x][k^1] = y, fa[y] = x;
    pushup(y), pushup(x);
}

void splay(int x) {
    static int stk[N];
    int tp = 0, u = x;
    stk[++tp] = u;
    while (!isroot(u)) stk[++tp] = u = fa[u];
    while (tp) pushdown(stk[tp--]);

    while (!isroot(x)) {
        int y = fa[x], z = fa[y];
        if (!isroot(y)) {
            if ((ch[y][1] == x) ^ (ch[z][1] == y)) rotate(x);
            else rotate(y);
        }
        rotate(x);
    }
}

void access(int x) {
    int z = x;
    for (int y = 0; x; y = x, x = fa[x]) {
        splay(x);
        ch[x][1] = y;
        pushup(x);
    }
    splay(z);
}

void makeroot(int x) {
    access(x);
    pushrev(x);
}

int findroot(int x) {
    access(x);
    while (ch[x][0]) pushdown(x), x = ch[x][0];
    splay(x);
    return x;
}

void split(int x, int y) {
    makeroot(x);
    access(y);
}

void link(int x, int y) {
    makeroot(x);
    if (findroot(y) != x) fa[x] = y;
}

void cut(int x, int y) {
    makeroot(x);
    if (findroot(y) == x && fa[y] == x && !ch[y][0]) {
        ch[x][1] = fa[y] = 0;
        pushup(x);
    }
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &val[i]);
    for (int i = 1, opt, x, y; i <= m; i++) {
        scanf("%d%d%d", &opt, &x, &y);
        if (opt == 0) {
            split(x, y);
            printf("%d\n", sum[y]);
        }
        else if (opt == 1) link(x, y);
        else if (opt == 2) cut(x, y);
        else {
            splay(x);
            val[x] = y;
            pushup(x);
        }
    }
    return 0;
}
```

## [THUWC2017] 在美妙的数学王国中畅游

> 学渣小R 被大学的数学课程虐得生活不能自理，微积分的成绩曾是他在教室里上的课的最低分。然而他的某位陈姓室友却能轻松地在数学考试中得到满分。为了提升自己的数学课成绩，有一天晚上（在他睡觉的时候），他来到了数学王国。
>
> 数学王国中，每个人的智商可以用一个属于 $[0,1]$ 的实数表示。数学王国中有 $n$ 个城市，编号从 $0 \sim n-1$，这些城市由若干座魔法桥连接。每个城市的中心都有一个魔法球，每个魔法球中藏有一道数学题。
>
> 每个人在做完这道数学题之后都会得到一个在 $[0,1]$ 内的分数。一道题可以用一个函数 $f(x)$ 表示。若一个人的智商为 $x$，则他做完这道数学题之后会得到 $f(x)$ 分。有三种形式：
>
> -  正弦函数 $f(x)=\sin(a x + b)\ (a \in [0,1], b \in [0,\pi],a+b\in[0,\pi])$   
> -  指数函数 $f(x)=\exp(ax+b)\ (a\in [-1,1], b\in [-2,0], a+b\in [-2,0])$   
> -  一次函数 $f(x) = ax + b\ (a\in [-1,1],b\in[0,1],a+b\in [0,1])$    
>
> 数学王国中的魔法桥会发生变化，有时会有一座魔法桥消失，有时会有一座魔法桥出现。但在任意时刻，只存在至多一条连接任意两个城市的简单路径（即所有城市形成一个森林）。在初始情况下，数学王国中不存在任何的魔法桥。
>
> 数学王国的国王拉格朗日很乐意传授小R 数学知识，但前提是小R 要先回答国王的问题。这些问题具有相同的形式，即一个智商为 $x$ 的人从城市 $u$ 旅行到城市 $v$（即经过 $u \to v$ 这条路径上的所有城市，包括 $u$ 和 $v$）且做了所有城市内的数学题后，他所有得分的总和是多少。
>
> **输入格式**
>
> 第一行两个正整数 $n,m$ 和一个字符串 $type$。表示数学王国中共有 $n$ 座城市，发生了 $m$ 个事件，该数据的类型为 $type$。
>
> $type$ 字符串是为了能让大家更方便地获得部分分，你可能不需要用到这个输入。其具体含义在 **【限制与约定】** 中有解释。
>
> 接下来 $n$ 行，第 $i$ 行表示初始情况下编号为 $i$ 的城市的魔法球中的函数。一个魔法用一个整数 $f$ 表示函数的类型，两个实数 $a,b$ 表示函数的参数，若
>
> - $f=1$，则函数为 $f(x)=\sin(ax+b)$
> - $f=2$，则函数为 $f(x)=\exp(ax+b)$
> - $f=3$，则函数为 $f(x)=ax+b$
>
> 接下来 $m$ 行，每行描述一个事件，事件分为四类。
>
> - `appear u v` 表示数学王国中出现了一条连接 $u$ 和 $v$ 这两座城市的魔法，保证连接前 $u,v$ 这两座城市不能互相到达。
>
> - `disappear u v` 表示数学王国中连接 $u,v$ 这两座城市的魔法桥消失了，保证这座魔法桥是存在的。
>
> - `magic c f a b` 表示城市 $c$ 的魔法球中的魔法变成了类型为 $f$，参数为 $a,b$ 的函数
>
> - `travel u v x` 表示询问一个智商为 $x$ 的人从城市 $u$ 旅行到城市 $v$ 后，他得分的总和是多少。若无法从 $u$ 到达 $v$，则输出一行一个字符串 `unreachable`。
>
> **输出格式**
>
> 对于每个询问，输出一行实数，表示得分的总和。
>
>
> 对于 $100\%$ 的数据，$1\le n\le 10^5, 1\le m \le 2 \times 10^5$ 。
>
> 【小R 教你学数学】
>
> 若函数 $f(x)$ 的 $n$ 阶导数在 $[a,b]$ 区间内连续，则对 $f(x)$ 在 $x_0 \ (x_0\in[a,b])$ 处使用 $n$ 次拉格朗日中值定理可以得到带拉格朗日余项的泰勒展开式
>
> $$f(x)=\sum_{k=0}^{n-1}\frac{f^{(k)}(x_0)(x-x_0)^k}{k!}+\frac{f^{(n)}(\xi)(x-x_0)^n}{n!},x\in[a,b]$$
>
> 其中，当 $x>x_0$ 时，$\xi\in[x_0,x]$。当 $x<x_0$ 时，$\xi\in[x,x_0]$。
>
> $f^{(n)}$ 表示函数 $f$ 的 $n$ 阶导数。
>
> 题目链接：[P4546](https://www.luogu.com.cn/problem/P4546)。

这个题面挺唬人的，其实就是板子。把题目给的泰勒公式写一下，发现有：

$$
\begin{aligned}
\sin(ax+b)&=\sum_{k=0}^{\infin}\frac{f^{(k)}(0)}{k!}x^k=\sum_{k=0}^{\infin}\frac{a^k\sin^{(k)}b}{k!}x^k\\
\exp(ax+b)&=\sum_{k=0}^{\infin}\frac{f^{(k)}(0)}{k!}x^k=\sum_{k=0}^{\infin}\frac{a^ke^b}{k!}x^k\\
ax+b&=ax+b
\end{aligned}
$$
我不知道 $\sin^{(k)}$ 这样写是否规范，大家懂这是 $k$ 阶导数就行。

然后要求精度是 $10^{-7}$，用 $20$ 项稳一下，所以就是维护一堆多项式的和。$4s$ 对于 $20n\log n$ 的 LCT 应该是可以过的。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 100010, K = 20;
int n, m, fa[N], ch[N][2], rev[N];

struct Poly {
    double c[K];

    Poly operator+(const Poly& p) {
        Poly res;
        for (int i = 0; i < K; i++) res.c[i] = c[i] + p.c[i];
        return res;
    }
    
    double calc(double x) {
        double x0 = 1, res = 0;
        for (int i = 0; i < K; i++) res += x0 * c[i], x0 *= x;
        return res;
    }

    void init(int f, double a0, double b0) {
        double infrac = 1, a = 1, b = 1;
        if (f == 1) {
            double extra[4] = {sin(b0), cos(b0), -sin(b0), -cos(b0)};
            for (int i = 0; i < K; i++) {
                c[i] = a * extra[i%4] * infrac;
                a *= a0;
                infrac /= i+1;
            }
        }
        else if (f == 2) {
            double expb = exp(b0);
            for (int i = 0; i < K; i++) {
                c[i] = a * expb * infrac;
                a *= a0;
                infrac /= i+1;
            }
        }
        else {
            for (int i = 0; i < K; i++) c[i] = 0;
            c[0] = b0, c[1] = a0;
        }
        return;
    }
} sum[N], val[N];

bool isroot(int x) {
    return ch[fa[x]][0] != x && ch[fa[x]][1] != x;
}

void pushup(int x) {
    sum[x] = sum[ch[x][0]] + val[x] + sum[ch[x][1]];
}

void pushrev(int x) {
    rev[x] ^= 1;
    swap(ch[x][0], ch[x][1]);
}

void pushdown(int x) {
    if (!rev[x]) return;
    pushrev(ch[x][0]), pushrev(ch[x][1]);
    rev[x] = 0;
}

void rotate(int x) {
    int y = fa[x], z = fa[y];
    int k = ch[y][1] == x;
    if (!isroot(y)) ch[z][ch[z][1] == y] = x;
    fa[x] = z;
    ch[y][k] = ch[x][k^1], fa[ch[x][k^1]] = y;
    ch[x][k^1] = y, fa[y] = x;
    pushup(y), pushup(x);
}

void splay(int x) {
    static int stk[N];
    int tp = 0, u = x;
    stk[++tp] = u;
    while (!isroot(u)) stk[++tp] = u = fa[u];
    while (tp) pushdown(stk[tp--]);

    while (!isroot(x)) {
        int y = fa[x], z = fa[y];
        if (!isroot(y)) {
            if ((ch[y][1] == x) ^ (ch[z][1] == y)) rotate(x);
            else rotate(y);
        }
        rotate(x);
    }
}

void access(int x) {
    int z = x;
    for (int y = 0; x; y = x, x = fa[x]) {
        splay(x);
        ch[x][1] = y;
        pushup(x);
    }
    splay(z);
}

void makeroot(int x) {
    access(x);
    pushrev(x);
}

int findroot(int x) {
    access(x);
    while (ch[x][0]) pushdown(x), x = ch[x][0];
    splay(x);
    return x;
}

void link(int x, int y) {
    makeroot(x);
    if (findroot(y) != x) fa[x] = y;
}

void cut(int x, int y) {
    makeroot(x);
    if (findroot(y) == x && fa[y] == x && !ch[y][0]) {
        ch[x][1] = fa[y] = 0;
        pushup(x);
    }
}

int main() {
    static char opt[10];
    double a, b;
    scanf("%d%d%*s", &n, &m);
    for (int i = 1; i <= n; i++) {
        int f;
        scanf("%d%lf%lf", &f, &a, &b);
        val[i].init(f, a, b);
    }
    for (int i = 1, x, y; i <= m; i++) {
        scanf("%s%d%d", opt, &x, &y);
        if (*opt == 'a') link(++x, ++y);
        else if (*opt == 'd') cut(++x, ++y);
        else if (*opt == 'm') {
            splay(++x);
            scanf("%lf%lf", &a, &b);
            val[x].init(y, a, b);
            pushup(x);
        }
        else {
            scanf("%lf", &a);
            ++x, ++y;
            makeroot(x);
            if (findroot(y) == x) {
                access(y);
                printf("%.8e\n", sum[y].calc(a));
            }
            else puts("unreachable");
        }
    }
    return 0;
}
```

