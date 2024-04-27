---
date: 2023-08-06 16:21:00
title: 数据结构 - Splay
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
---

## 简介

借助左旋和右旋操作，在保证 BST 性质的情况下尽可能地减少树的高度。

当结点 $x \to y \to z$ 呈直线形的时候，$rotate(y), rotate(x)$；呈折线形时，$rotate(x),rotate(x)$。可以五个结点分情况看一下，这样能保证树的高度尽可能小。

## 势能法摊还分析

### 原理

在难以求出一个数据结构真实的复杂度时，可以使用势能的概念辅助进行计算，设 $c_i$ 是每次的真实复杂度，$\hat{c_i}$ 是均摊复杂度，那么有：
$$
\hat{c_i}=c_i+\Delta\varphi_i
$$
由此可以推出：
$$
\sum \hat{c_i}=\sum c_i+\varphi_n-\varphi_0
$$
我们只需要找到一个合适的势能函数，就可以用 $\sum \hat{c_i}$ 当作 $\sum c_i$ 的一个上界，从而进行复杂度的证明。

### 势能函数

为了行文方便，用 $v$ 表示某个节点，$T$ 表示全树，$|v|$ 表示结点对应的子树大小，定义势能函数如下：
$$
\varphi_i=\sum_{v\in T} \log|v|
$$
实际意义是 $\log|v|$ 表示 $v$ 子树的期望树高。

### 单旋

设 $x,y,z$ 后者为前者的父结点，对 $x$ 进行单旋的时候，有：
$$
\Delta\varphi_i=\log|x'|+\log|y'|-\log|x|-\log|y|
$$
显然有 $|x'|=|y|$ 且 $|x'|\ge|y'|$ 因此可以得到：
$$
\Delta\varphi_i=\log|y'|-\log|x|\le \log|x'|-\log|x|
$$

这里有：
$$
\hat{c_i}\le 1+3(\log|x'|-\log|x|)
$$

这里的 $3$ 主要是为了和下边对应起来。

### 双旋

同上，设 $x,y,z$ 三个结点，对 $x$ 单旋两次，可以得到：
$$
\Delta\varphi_i=\log|x'|+\log|y'|+\log|z'|-\log|x|-\log|y|-\log|z|
$$
有 $|x'|=|z|,|y|\ge|x|,|y'|\le|x'|$，那么：
$$
\Delta\varphi_i\le\log|x'|+\log|z'|-2\log|x|
$$
由于旋转两次，均摊复杂度为：
$$
\hat{c_i}=2+\Delta\varphi_i
$$
有一个神秘的操作可以把常数 $2$ 消掉，如下：
$$
\begin{aligned}
\log|x|+\log|z'|-2\log|x'|&=\log\frac{|x||z'|}{|x'|^2}\\
&\le\log\frac{|x||z'|}{(|x|+|z'|)^2}\\
&\le-2
\end{aligned}
$$
我们移项消掉 $\log|z'|$ 可以得到：
$$
\Delta\varphi_i\le\log|x'|+(-2+2\log|x'|-\log|x|)-2\log|x|\le-2+3(\log|x'|-\log|x|)
$$
可得：
$$
\hat{c_i}\le3(\log|x'|-\log|x|)
$$

### 综上

进行 $n$ 次 `splay` 操作需要的总复杂度为：
$$
\sum\hat{c_i}\le 3n(\log|x_n|-\log|x|)+1
$$
后面的 $1$ 表示最后一次单旋，于是可以得到复杂度为 $O(n\log n)$。

## 文艺平衡树

> 维护一个 $n$ 项有序数列，序列中第 $i$ 项初始为 $i$，输入 $m$ 行 $l,r$ 表示把区间 $[l,r]$ 翻转。
>
> 输出进行 $m$ 次变换后的序列。
>
> 题目链接：[AcWing 2437](https://www.acwing.com/problem/content/2439/)，[P3391](https://www.luogu.com.cn/problem/P3391)。

左旋和右旋可以合并到一个函数中写，在现在的 Splay 代码中大量存在着用布尔表达式判断是哪个子结点的情况，这样可以少写很多 if 语句。

```cpp
#include <cstdio>
#include <utility>
using namespace std;

const int N = 1e5+10;

struct Node {
    int s[2], p;
    // flag: 是否翻转
    int flag, size, v;
    
    void init(int _v, int _p) {
        v = _v, p = _p;
        size = 1;
    }
} tr[N];

int root, idx, n, m;

void pushdown(int u) {
    if (tr[u].flag) {
        swap(tr[u].s[0], tr[u].s[1]);
        tr[tr[u].s[0]].flag ^= 1;
        tr[tr[u].s[1]].flag ^= 1;
        tr[u].flag = 0;
    }
}

void pushup(int u) {
    tr[u].size = tr[tr[u].s[0]].size + tr[tr[u].s[1]].size + 1;
}

void rotate(int x) {
    int y = tr[x].p, z = tr[y].p;
    // k=0: x 是左儿子
    // k=1: x 是右儿子
    int k = tr[y].s[1] == x;
    // 更新 z-y 为 z-x
    tr[z].s[tr[z].s[1] == y] = x, tr[x].p = z;
    // 更新 x-B 为 y-B
    tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].p = y;
    // 更新 y-x 为 x-y
    tr[x].s[k ^ 1] = y, tr[y].p = x;
    // 先 pushup 下面的 y, 再 pushup 上面的 x
    pushup(y), pushup(x);
}

void splay(int x, int k) {
    while (tr[x].p != k) {
        int y = tr[x].p, z = tr[y].p;
        if (z != k)
            if ((tr[y].s[0] == x) ^ (tr[z].s[0] == y)) rotate(x);
            else rotate(y);
        rotate(x);
    }
    if (k == 0) root = x;
}

void output(int u) {
    pushdown(u);
    if (tr[u].s[0]) output(tr[u].s[0]);
    if (tr[u].v > 0 && tr[u].v <= n) printf("%d ", tr[u].v);
    if (tr[u].s[1]) output(tr[u].s[1]);
}

// 中序遍历的第 k 个数
int get_k(int k) {
    int u = root;
    while (u) {
        pushdown(u);
        if (tr[tr[u].s[0]].size >= k) u = tr[u].s[0];
        else if (tr[tr[u].s[0]].size + 1 == k) return u;
        else {
            k -= tr[tr[u].s[0]].size + 1;
            u = tr[u].s[1];
        }
    }
    return -1;
}

void insert(int v) {
    int u = root, p = 0;
    while (u) p = u, u = tr[u].s[v > tr[u].v];
    u = ++idx;
    tr[u].init(v, p);
    if (p) tr[p].s[v > tr[p].v] = u;
    // 插入之后把它转到根节点上
    splay(u, 0);
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 0; i <= n+1; i++) insert(i);
    while (m--) {
        int l, r;
        scanf("%d%d", &l, &r);
        l = get_k(l), r = get_k(r+2);
        splay(l, 0), splay(r, l);
        // 找到满足 l < v < r 的子树
        tr[tr[r].s[0]].flag ^= 1;
    }
    output(root);
    return 0;
}
```

## 普通平衡树

> 您需要写一种数据结构（可参考题目标题），来维护一些数，其中需要提供以下操作：
>
> 1. 插入 x 数；
> 2. 删除 x 数（若有多个相同的数，只删除一个）；
> 3. 查询 x 数的排名（若有多个相同的数，输出最小的排名）；
> 4. 查询排名为 x 的数；
> 5. 求 x 的前驱（前驱定义为小于 x，且最大的数）；
> 6. 求 x 的后继（后继定义为大于 x，且最小的数）。
>
> 题目链接：[AcWing 253](https://www.acwing.com/problem/content/255/)，[P3369](https://www.luogu.com.cn/problem/P3369)。

这题比上面的文艺平衡树反而更难写，删除一个数实在是很困难的。

```cpp
#include <cstdio>
#include <cstdlib>
using namespace std;

const int N = 1e5+10, INF = 1e9;

struct Node {
    int s[2], p, v;
    int size, cnt;

    void init(int _p, int _v) {
        s[0] = s[1] = 0;
        p = _p, v = _v;
        size = cnt = 1;
    }
} tr[N];

int root, idx, n;

void pushup(int x) {
    tr[x].size = tr[tr[x].s[0]].size + tr[tr[x].s[1]].size + tr[x].cnt;
}

void rotate(int x) {
    int y = tr[x].p, z = tr[y].p;
    int k = tr[y].s[1] == x;
    tr[z].s[tr[z].s[1] == y] = x, tr[x].p = z;
    tr[y].s[k] = tr[x].s[k^1], tr[tr[x].s[k^1]].p = y;
    tr[x].s[k^1] = y, tr[y].p = x;
    pushup(y), pushup(x);
}

void splay(int x, int k) {
    while (tr[x].p != k) {
        int y = tr[x].p, z = tr[y].p;
        if (z != k) {
            if ((tr[y].s[0] == x) ^ (tr[z].s[0] == y)) rotate(x);
            else rotate(y);
        }
        rotate(x);
    }

    if (k == 0) root = x;
}

int get_rank(int v) {
    int u = root, rk = 0;
    while (u) {
        if (v < tr[u].v) u = tr[u].s[0];
        else {
            // 左子树全部比 v 小
            rk += tr[tr[u].s[0]].size;
            if (v == tr[u].v) {
                splay(u, 0);
                return rk + 1;
            }
            // 当前结点也比 v 小
            rk += tr[u].cnt;
            u = tr[u].s[1];
        }
    }
    exit(-1);
}

// 这个函数比较容易写错
int get_kth(int k) {
    int u = root;
    while (u) {
        // 左子树中包含了第 k 大数
        if (tr[tr[u].s[0]].size >= k) u = tr[u].s[0];
        // 当前结点包含了第 k 大数
        else if (tr[tr[u].s[0]].size + tr[u].cnt >= k) return tr[u].v;
        else {
            // 把右儿子当成根继续找
            k -= tr[tr[u].s[0]].size + tr[u].cnt;
            u = tr[u].s[1];
        }
    }
    exit(-1);
}

void insert(int v) {
    int u = root, p = 0;
    while (u && tr[u].v != v) p = u, u = tr[u].s[v > tr[u].v];
    if (u) tr[u].cnt++;
    else {
        u = ++idx;
        tr[u].init(p, v);
        if (p) tr[p].s[v > tr[p].v] = u;
    }
    splay(u, 0);
}

void del(int v) {
    // 转到根
    get_rank(v);
    if (tr[root].cnt > 1) tr[root].cnt--;
    else {
        // 由于存在两个哨兵 因此前驱和后继是一定存在的
        int l = tr[root].s[0], r = tr[root].s[1];
        while (tr[l].s[1]) l = tr[l].s[1];
        while (tr[r].s[0]) r = tr[r].s[0];
        // 传统操作
        splay(l, 0), splay(r, l);
        tr[r].s[0] = 0;
        pushup(r), pushup(l);
    }
}

int pre(int v) {
    // 防止不存在
    insert(v);
    // 把查询的结点转到根
    get_rank(v);

    int l = tr[root].s[0];
    while (tr[l].s[1]) l = tr[l].s[1];

    del(v);
    return tr[l].v;
}

int suf(int v) {
    insert(v);
    get_rank(v);

    int r = tr[root].s[1];
    while (tr[r].s[0]) r = tr[r].s[0];
    del(v);

    return tr[r].v;
}

int main() {
    scanf("%d", &n);
    insert(-INF), insert(INF);
    while (n--) {
        int opt, x;
        scanf("%d%d", &opt, &x);
        if (opt == 1) insert(x);
        else if (opt == 2) del(x);
        else if (opt == 3) printf("%d\n", get_rank(x)-1);
        else if (opt == 4) printf("%d\n", get_kth(x+1));
        else if (opt == 5) printf("%d\n", pre(x));
        else printf("%d\n", suf(x));
    }
    return 0;
}
```

## [HNOI2002] 营业额统计

> 给出一个 n 个数的数列，定义 $f_i=\min_{1\le j\lt i}|a_i-a_j|$，其中 $f_1=a_1$，求 $\sum f_i$
>
> 题目链接：[LOJ 10143](https://loj.ac/p/10143)，[AcWing 265](https://www.acwing.com/problem/content/267/)。

本题坑点在于数列中有重复元素。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 35000;

struct Node {
    int s[2], p, v;
    int cnt;
    
    void init(int _p, int _v) {
        p = _p, v = _v;
        s[0] = s[1] = 0;
        cnt = 1;
    }
} tr[N];

int root, idx;

void rotate(int x) {
    int y = tr[x].p, z = tr[y].p;
    int k = tr[y].s[1] == x;
    tr[z].s[tr[z].s[1] == y] = x, tr[x].p = z;
    tr[y].s[k] = tr[x].s[k^1], tr[tr[x].s[k^1]].p = y;
    tr[x].s[k^1] = y, tr[y].p = x;
}

void splay(int x, int k) {
    while (tr[x].p != k) {
        int y = tr[x].p, z = tr[y].p;
        if (z != k) {
            if ((tr[y].s[0] == x) ^ (tr[z].s[0] == y)) rotate(x);
            else rotate(y);
        }
        rotate(x);
    }
    if (k == 0) root = x;
}

void insert(int v) {
    int u = root, p = 0;
    while (u && tr[u].v != v) p = u, u = tr[u].s[v > tr[u].v];
    if (u) tr[u].cnt++;
    else {
        u = ++idx;
        tr[u].init(p, v);
        if (p) tr[p].s[v > tr[p].v] = u;
    }
    splay(u, 0);
}

int pre() {
    int l = tr[root].s[0];
    if (!l) return 1e9;
    while (tr[l].s[1]) l = tr[l].s[1];
    return tr[l].v;
}

int suf() {
    int r = tr[root].s[1];
    if (!r) return 1e9;
    while (tr[r].s[0]) r = tr[r].s[0];
    return tr[r].v;
}

int main() {
    int n;
    scanf("%d", &n);
    
    int res;
    scanf("%d", &res);
    insert(res);
    n--;
    
    while (n--) {
        int v;
        scanf("%d", &v);
        insert(v);
        if (tr[root].cnt > 1) continue;
        int now = min(abs(pre() - v), abs(suf() - v));
        res += now;
    }
    printf("%d\n", res);
    return 0;
}
```

## [NOI2004]  郁闷的出纳员

> 题干过长，这里不粘过来了。
>
> **输入格式**
>
> 第一行有两个非负整数 n 和 min，n 表示下面有多少条命令，min 表示工资下界。
>
> 接下来的 n 行，每行表示一条命令，命令可以是以下四种之一：
>
> 1. `I` 命令，格式为 `I_k`，表示新建一个工资档案，初始工资为 k。如果某员工的初始工资低于工资下界，他将立刻离开公司。
> 2. `A` 命令，格式为 `A_k`，表示把每位员工的工资加上 k。
> 3. `S` 命令，格式为 `S_k`，表示把每位员工的工资扣除 k。
> 4. `F` 命令，格式为 `F_k`，表示查询第 k 多的工资。
>
> `_`（下划线）表示一个空格，`I` 命令、`A` 命令、`S` 命令中的 k 是一个非负整数，`F` 命令中的 k 是一个正整数。
>
> 在初始时，可以认为公司里一个员工也没有。
>
> **输出格式**
>
> 输出行数为 F 命令的条数加一。对于每条 F 命令，你的程序要输出一行，仅包含一个整数，为当前工资第 k 多的员工所拿的工资数，如果 k 大于目前员工的数目，则输出 −1。
>
> 输出文件的最后一行包含一个整数，为离开公司的员工的总数。
>
> 注意，如果某个员工的初始工资低于最低工资标准，那么将不计入最后的答案内。
>
> 题目链接：[AcWing 950](https://www.acwing.com/problem/content/952/)，[P1486](https://www.luogu.com.cn/problem/P1486)。

本题不改变相对大小关系，因此可以让 Splay 保序。

```cpp
#include <cstdio>
using namespace std;

const int N = 1e5+10, INF = 1e9;
struct Node {
    int s[2], p, v, size;
    void init(int _p, int _v) {
        p = _p, v = _v;
        size = 1;
    }
} tr[N];
int n, m, root, delta, idx;

void pushup(int u) {
    tr[u].size = tr[tr[u].s[0]].size + tr[tr[u].s[1]].size + 1;
}

void rotate(int x) {
    int y = tr[x].p, z = tr[y].p;
    int k = tr[y].s[1] == x;
    tr[z].s[tr[z].s[1] == y] = x, tr[x].p = z;
    tr[y].s[k] = tr[x].s[k^1], tr[tr[x].s[k^1]].p = y;
    tr[x].s[k^1] = y, tr[y].p = x;
    pushup(y), pushup(x);
}

void splay(int x, int k) {
    while (tr[x].p != k) {
        int y = tr[x].p, z = tr[y].p;
        if (z != k) {
            if ((tr[y].s[0] == x) ^ (tr[z].s[0] == y)) rotate(x);
            else rotate(y);
        }
        rotate(x);
    }
    // splay 需要修改 root
    if (k == 0) root = x;
}

int insert(int v) {
    int u = root, p = 0;
    while (u) p = u, u = tr[u].s[v > tr[u].v];
    u = ++idx;
    tr[u].init(p, v);
    if (p) tr[p].s[v > tr[p].v] = u;
    splay(u, 0);
    return u;
}

// 大于等于 v 的第一个结点
int get(int v) {
    int u = root, res = -1;
    while (u) {
        if (tr[u].v >= v) res = u, u = tr[u].s[0];
        else u = tr[u].s[1];
    }
    return res;
}

int get_kth(int k) {
    int u = root;
    while (u) {
        if (tr[tr[u].s[0]].size >= k) u = tr[u].s[0];
        else if (tr[tr[u].s[0]].size + 1 == k) return tr[u].v + delta;
        else {
            k -= tr[tr[u].s[0]].size + 1;
            u = tr[u].s[1];
        }
    }
    return -1;
}

int main() {
    scanf("%d%d", &n, &m);
    int L = insert(-INF), total = 0;
    insert(INF);
    while (n--) {
        char op[2];
        int k;
        scanf("%s%d", op, &k);
        if (*op == 'I') {
            if (k < m) continue;
            k -= delta;
            insert(k);
            total++;
        }
        else if (*op == 'A') delta += k;
        else if (*op == 'S') {
            delta -= k;
            int R = get(m-delta);
            splay(R, 0), splay(L, R);
            tr[L].s[1] = 0;
            pushup(L), pushup(R);
        }
        else {
            if (tr[root].size - 2 < k) puts("-1");
            else printf("%d\n", get_kth(tr[root].size - k));
        }
    }

    printf("%d\n", total - (tr[root].size - 2));
    return 0;
}
```

## [HNOI2012] 永无乡

> 永无乡包含 n 座岛，编号从 1 到 n ，每座岛都有自己的独一无二的重要度，按照重要度可以将这 n 座岛排名，名次用 1 到 n 来表示。
>
> 某些岛之间由巨大的桥连接，通过桥可以从一个岛到达另一个岛。
>
> 如果从岛 a 出发经过若干座（含 0 座）桥可以到达岛 b，则称岛 a 和岛 b 是连通的。
>
> 现在有两种操作：
>
> -  `B x y` 表示在岛 x 与岛 y 之间修建一座新桥。
> -  `Q x k` 表示询问当前与岛 x 连通的所有岛中第 k 重要的是哪座岛，即所有与岛 x 连通的岛中重要度排名第 k 小的岛是哪座，请你输出那个岛的编号，如果不存在输出 -1。
>
> 题目链接：[AcWing 1063](https://www.acwing.com/problem/content/1065/)，[P3224](https://www.luogu.com.cn/problem/P3224)。

首先应该开多少数组大小，最坏情况下需要把所有岛都合并，合并时把小的合并到大的里面去，这样能尽可能避免浪费空间，于是我们注意到在最坏的情况下会将两个长度为 $n/2$ 的连通块合并，而得到这两个连通块的方式最坏情况下也是 $n/4$ 的连通块合并，这样递归下去。

可以写出下面的函数帮助我们得到一个需要的空间：
$$
f(x)=\left\{\begin{aligned}
&\frac{3}{2}x+f(\frac{1}{2}x),x>1\\
&0,x\le1
\end{aligned}\right.
$$
用 matplotlib 画了一下，惊奇地发现这是一条直线，于是继续展开，然后小小放缩了一下：
$$
\begin{aligned}
f(x)&=\frac{3}{2}x+\frac{3}{2}(\frac{1}{2}x)+\cdots+\frac{3}{2}(\frac{1}{2^n}x)\\
&=\frac{3}{2}x(1+\frac{1}{2}+\cdots+\frac{1}{2^n})\\
&=3x(1-\frac{1}{2^{n+1}})\\
&\lt 3x
\end{aligned}
$$
所以这可以近似的认为是直线 $y=3x$，根据本题数据，保险点开 `350000` 就够了。

```cpp
#include <cstdio>
#include <utility>
using namespace std;

const int M = 350000, N = 100010;

struct Node {
    int s[2], v, id, p;
    int size;
    void init(int _v, int _id, int _p) {
        v = _v, id = _id, p = _p;
        size = 1;
    }
} tr[M];
int root[N], p[N], n, m, idx;

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

void pushup(int u) {
    tr[u].size = tr[tr[u].s[0]].size + tr[tr[u].s[1]].size + 1;
}

void rotate(int x) {
    int y = tr[x].p, z = tr[y].p;
    int k = tr[y].s[1] == x;
    tr[z].s[tr[z].s[1] == y] = x, tr[x].p = z;
    tr[y].s[k] = tr[x].s[k^1], tr[tr[x].s[k^1]].p = y;
    tr[x].s[k^1] = y, tr[y].p = x;
    pushup(y), pushup(x);
}

void splay(int x, int k, int rid) {
    while (tr[x].p != k) {
        int y = tr[x].p, z = tr[y].p;
        if (z != k) {
            if ((tr[y].s[0] == x) ^ (tr[z].s[0] == y)) rotate(x);
            else rotate(y);
        }
        rotate(x);
    }
    if (k == 0) root[rid] = x;
}

void insert(int v, int id, int rid) {
    int u = root[rid], p = 0;
    while (u) p = u, u = tr[u].s[v > tr[u].v];
    u = ++idx;
    tr[u].init(v, id, p);
    if (p) tr[p].s[v > tr[p].v] = u;
    splay(u, 0, rid);
}

void dfs(int u, int rid) {
    if (tr[u].s[0]) dfs(tr[u].s[0], rid);
    if (tr[u].s[1]) dfs(tr[u].s[1], rid);
    insert(tr[u].v, tr[u].id, rid);
}

int get_kth(int rid, int k) {
    int u = root[rid];
    while (u) {
        if (tr[tr[u].s[0]].size >= k) u = tr[u].s[0];
        else if (tr[tr[u].s[0]].size + 1 == k) return tr[u].id;
        else {
            k -= tr[tr[u].s[0]].size + 1;
            u = tr[u].s[1];
        }
    }
    return -1;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) {
        p[i] = root[i] = i;
        int v;
        scanf("%d", &v);
        tr[root[i]].init(v, i, 0);
    }
    idx = n;
    while (m--) {
        int a, b;
        scanf("%d%d", &a, &b);
        a = find(a), b = find(b);
        if (a != b) {
            // root[a], root[b] 会在 splay 的时候变化
            if (tr[root[a]].size > tr[root[b]].size) swap(a, b);
            dfs(root[a], b);
            p[a] = b;
        }
    }
    scanf("%d", &m);
    while (m--) {
        char op[2];
        int x, y;
        scanf("%s%d%d", op, &x, &y);
        if (*op == 'B') {
            x = find(x), y = find(y);
            if (x != y) {
                if (tr[root[x]].size > tr[root[y]].size) swap(x, y);
                dfs(root[x], y);
                p[x] = y;
            }
        } else {
            x = find(x);
            if (tr[root[x]].size < y) puts("-1");
            else printf("%d\n", get_kth(x, y));
        }
    }
    return 0;
}
```

## [NOI2005] 维护数列

> 请写一个程序，支持以下 6 种操作。
>
> | 编号 | 名称         | 格式                                      | 说明                                                         |
> | ---- | ------------ | ----------------------------------------- | ------------------------------------------------------------ |
> | 1    | 插入         | $INSERT$ $pos$ $tot$ $c_1,\cdots,c_{tot}$ | 在当前数列的第 $pos$ 个数字后插入 $tot$ 个数字，若在数列首插入，则 $pos =0$ |
> | 2    | 删除         | $DELETE$ $pos$ $tit$                      | 从当前数列的第 $pos$ 个数字开始连续删除 $tot$ 个数字         |
> | 3    | 修改         | $MAKE\text{-}SAME$ $pos$ $tot$ $c$        | 从当前数列的第 $pos$ 个数字开始的连续 $tot$ 个数字统一修改为 $c$ |
> | 4    | 翻转         | $REVERSE$ $pos$ $tot$                     | 取出从当前数列的第 $pos$ 个数字开始的 $tot$ 个数字，翻转后放入原来的位置 |
> | 5    | 求和         | $GET\text{-}SUM$ $pos$ $tot$              | 计算从当前数列的第 $pos$ 个数字开始的 $tot$ 个数字的和并输出 |
> | 6    | 求最大子列和 | $MAX\text{-}SUM$                          | 求出当前数列中和最大的一段子列，并输出最大和                 |
>
> 题目链接：[AcWing 955](https://www.acwing.com/problem/content/957/)，[P2042](https://www.luogu.com.cn/problem/P2042)。

当需要进行 Splay 操作之前，都会进行 `get_kth`，此时 `pushdown` 已经进行过了，因此 Splay 操作是不需要 `pushdown` 的。

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 5000010;
const int INF = 1e9;

struct Node {
    int s[2], p, v;
    int size;
    int sum, lmax, rmax, tmax;
    int rev, same;
    
    void init(int _v, int _p) {
        // 整个序列最大值不可以为 0，必须包含起码一个元素
        v = sum = tmax = _v, p = _p;
        // 前缀最大值 后缀最大值 可以为 0
        lmax = rmax = max(_v, 0);
        size = 1;
    }
    
    void makesame(int c) {
        v = c;
        same = 1;
        sum = size * c;
        if (c > 0) tmax = rmax = lmax = c * size;
        else tmax = c, lmax = rmax = 0;
    }
    
    void reverse() {
        rev ^= 1;
        swap(s[0], s[1]);
        swap(lmax, rmax);
    }
} tr[N];
int root, idx, n, m;

void pushup(int _u) {
    Node& u = tr[_u], &l = tr[u.s[0]], &r = tr[u.s[1]];
    u.size = l.size + r.size + 1;
    u.sum = l.sum + r.sum + u.v;
    u.lmax = max(l.lmax, l.sum + u.v + r.lmax);
    u.rmax = max(r.rmax, r.sum + u.v + l.rmax);
    // 这里就解释了前缀最大值和后缀最大值可以为零的原因，这里更新 tmax 时需要加上 u.v
    u.tmax = max(l.rmax + u.v + r.lmax, max(l.tmax, r.tmax));
}

void pushdown(int u) {
    if (tr[u].rev) {
        tr[tr[u].s[0]].reverse();
        tr[tr[u].s[1]].reverse();
        tr[u].rev = 0;
    }
    
    if (tr[u].same) {
        tr[tr[u].s[0]].makesame(tr[u].v);
        tr[tr[u].s[1]].makesame(tr[u].v);
        tr[u].same = 0;
    }
}

void rotate(int x) {
    int y = tr[x].p, z = tr[y].p;
    int k = tr[y].s[1] == x;
    tr[z].s[tr[z].s[1] == y] = x, tr[x].p = z;
    tr[y].s[k] = tr[x].s[k^1], tr[tr[x].s[k^1]].p = y;
    tr[x].s[k^1] = y, tr[y].p = x;
    pushup(y), pushup(x);
}

void splay(int u, int k) {
    while (tr[u].p != k) {
        int y = tr[u].p, z = tr[y].p;
        if (z != k) {
            if ((tr[y].s[0] == u) ^ (tr[z].s[0] == y)) rotate(u);
            else rotate(y);
        }
        rotate(u);
    }
    if (k == 0) root = u;
}

int get_kth(int k) {
    int u = root;
    while (u) {
        pushdown(u);
        if (tr[tr[u].s[0]].size >= k) u = tr[u].s[0];
        else if (tr[tr[u].s[0]].size + 1 == k) return u;
        else {
            k -= tr[tr[u].s[0]].size + 1;
            u = tr[u].s[1];
        }
    }
    return -1;
}

int build(int v[], int l, int r, int p) {
    int mid = (l + r) >> 1;
    int u = ++idx;
    tr[u].init(v[mid], p);
    if (l < mid) tr[u].s[0] = build(v, l, mid-1, u);
    if (mid < r) tr[u].s[1] = build(v, mid+1, r, u);
    pushup(u);
    return u;
}

int buf[N];
char op[20];

int main() {
    scanf("%d%d", &n, &m);
    buf[0] = buf[n+1] = -INF;
    tr[0].v = tr[0].tmax = -INF;
    for (int i = 1; i <= n; i++) scanf("%d", &buf[i]);
    
    root = build(buf, 0, n+1, 0);
    int pos = -1, tot = -1, c = -1;
    while (m--) {
        scanf("%s", op);
        if (!strcmp(op, "INSERT")) {
            scanf("%d%d", &pos, &tot);
            for (int i = 1; i <= tot; i++) scanf("%d", &buf[i]);
            int L = get_kth(pos+1), R = get_kth(pos+2);
            splay(L, 0), splay(R, L);
            tr[R].s[0] = build(buf, 1, tot, R);
            pushup(R), pushup(L);
        }
        else if (!strcmp(op, "DELETE")) {
            scanf("%d%d", &pos, &tot);
            int L = get_kth(pos), R = get_kth(pos+tot+1);
            splay(L, 0), splay(R, L);
            tr[R].s[0] = 0;
            pushup(R), pushup(L);
        }
        else if (!strcmp(op, "MAKE-SAME")) {
            scanf("%d%d%d", &pos, &tot, &c);
            int L = get_kth(pos), R = get_kth(pos+tot+1);
            splay(L, 0), splay(R, L);
            tr[tr[R].s[0]].makesame(c);
            pushup(R), pushup(L);
        }
        else if (!strcmp(op, "REVERSE")) {
            scanf("%d%d", &pos, &tot);
            int L = get_kth(pos), R = get_kth(pos+tot+1);
            splay(L, 0), splay(R, L);
            tr[tr[R].s[0]].reverse();
            pushup(R), pushup(L);
        }
        else if (!strcmp(op, "GET-SUM")) {
            scanf("%d%d", &pos, &tot);
            int L = get_kth(pos), R = get_kth(pos+tot+1);
            splay(L, 0), splay(R, L);
            printf("%d\n", tr[tr[R].s[0]].sum);
        }
        else printf("%d\n", tr[root].tmax);
    }
    return 0;
}

```

题目保证树中元素数量小于 $5\times 10^5$，那么实际上可以缩小十倍的内存，这就需要我们进行一个内存回收的操作，但代价是删除时需要 DFS 一遍，多了很多时间，所以如果空间不够用才考虑内存回收。

我一般静态变量初始化为 0 都不写，这个时候就成功地把我坑死了。

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
#include <vector>
using namespace std;

const int N = 500010;
const int INF = 1e9;

struct Node {
    int s[2], p, v;
    int size;
    int sum, lmax, rmax, tmax;
    int rev, same;
    
    void init(int _v, int _p) {
        // 整个序列最大值不可以为 0，必须包含起码一个元素
        v = sum = tmax = _v, p = _p;
        // 前缀最大值 后缀最大值 可以为 0
        lmax = rmax = max(_v, 0);
        size = 1;
        // 我一般初始化为 0 都不写，被坑死了
        rev = same = 0;
        s[0] = s[1] = 0;
    }
    
    void makesame(int c) {
        v = c;
        same = 1;
        sum = size * c;
        if (c > 0) tmax = rmax = lmax = c * size;
        else tmax = c, lmax = rmax = 0;
    }
    
    void reverse() {
        rev ^= 1;
        swap(s[0], s[1]);
        swap(lmax, rmax);
    }
} tr[N];
int root, idx, n, m;

void pushup(int _u) {
    Node& u = tr[_u], &l = tr[u.s[0]], &r = tr[u.s[1]];
    u.size = l.size + r.size + 1;
    u.sum = l.sum + r.sum + u.v;
    u.lmax = max(l.lmax, l.sum + u.v + r.lmax);
    u.rmax = max(r.rmax, r.sum + u.v + l.rmax);
    // 这里就解释了前缀最大值和后缀最大值可以为零的原因，这里更新 tmax 时需要加上 u.v
    u.tmax = max(l.rmax + u.v + r.lmax, max(l.tmax, r.tmax));
}

void pushdown(int u) {
    if (tr[u].rev) {
        tr[tr[u].s[0]].reverse();
        tr[tr[u].s[1]].reverse();
        tr[u].rev = 0;
    }
    
    if (tr[u].same) {
        tr[tr[u].s[0]].makesame(tr[u].v);
        tr[tr[u].s[1]].makesame(tr[u].v);
        tr[u].same = 0;
    }
}

void rotate(int x) {
    int y = tr[x].p, z = tr[y].p;
    int k = tr[y].s[1] == x;
    tr[z].s[tr[z].s[1] == y] = x, tr[x].p = z;
    tr[y].s[k] = tr[x].s[k^1], tr[tr[x].s[k^1]].p = y;
    tr[x].s[k^1] = y, tr[y].p = x;
    pushup(y), pushup(x);
}

void splay(int u, int k) {
    while (tr[u].p != k) {
        int y = tr[u].p, z = tr[y].p;
        if (z != k) {
            if ((tr[y].s[0] == u) ^ (tr[z].s[0] == y)) rotate(u);
            else rotate(y);
        }
        rotate(u);
    }
    if (k == 0) root = u;
}

int get_kth(int k) {
    int u = root;
    while (u) {
        pushdown(u);
        if (tr[tr[u].s[0]].size >= k) u = tr[u].s[0];
        else if (tr[tr[u].s[0]].size + 1 == k) return u;
        else {
            k -= tr[tr[u].s[0]].size + 1;
            u = tr[u].s[1];
        }
    }
    return -1;
}

vector<int> ava;

int build(int v[], int l, int r, int p) {
    int mid = (l + r) >> 1;
    int u = ava.back(); ava.pop_back();
    tr[u].init(v[mid], p);
    if (l < mid) tr[u].s[0] = build(v, l, mid-1, u);
    if (mid < r) tr[u].s[1] = build(v, mid+1, r, u);
    pushup(u);
    return u;
}

void dfs(int u) {
    if (tr[u].s[0]) dfs(tr[u].s[0]);
    if (tr[u].s[1]) dfs(tr[u].s[1]);
    ava.push_back(u);
}


int buf[(int)4e6+10];
char op[20];

int main() {
    scanf("%d%d", &n, &m);
    buf[0] = buf[n+1] = -INF;
    tr[0].v = tr[0].tmax = -INF;
    for (int i = 1; i < N; i++) ava.push_back(i);
    for (int i = 1; i <= n; i++) scanf("%d", &buf[i]);
    
    root = build(buf, 0, n+1, 0);
    int pos = -1, tot = -1, c = -1;
    while (m--) {
        scanf("%s", op);
        if (!strcmp(op, "INSERT")) {
            scanf("%d%d", &pos, &tot);
            for (int i = 0; i < tot; i++) scanf("%d", &buf[i]);
            int L = get_kth(pos+1), R = get_kth(pos+2);
            splay(L, 0), splay(R, L);
            tr[R].s[0] = build(buf, 0, tot-1, R);
            pushup(R), pushup(L);
        }
        else if (!strcmp(op, "DELETE")) {
            scanf("%d%d", &pos, &tot);
            int L = get_kth(pos), R = get_kth(pos+tot+1);
            splay(L, 0), splay(R, L);
            dfs(tr[R].s[0]);
            tr[R].s[0] = 0;
            pushup(R), pushup(L);
        }
        else if (!strcmp(op, "MAKE-SAME")) {
            scanf("%d%d%d", &pos, &tot, &c);
            int L = get_kth(pos), R = get_kth(pos+tot+1);
            splay(L, 0), splay(R, L);
            tr[tr[R].s[0]].makesame(c);
            pushup(R), pushup(L);
        }
        else if (!strcmp(op, "REVERSE")) {
            scanf("%d%d", &pos, &tot);
            int L = get_kth(pos), R = get_kth(pos+tot+1);
            splay(L, 0), splay(R, L);
            tr[tr[R].s[0]].reverse();
            pushup(R), pushup(L);
        }
        else if (!strcmp(op, "GET-SUM")) {
            scanf("%d%d", &pos, &tot);
            int L = get_kth(pos), R = get_kth(pos+tot+1);
            splay(L, 0), splay(R, L);
            printf("%d\n", tr[tr[R].s[0]].sum);
        }
        else printf("%d\n", tr[root].tmax);
    }
    return 0;
}
```

