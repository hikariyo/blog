---
date: 2023-09-04 19:00:00
title: 图论 - 2 SAT
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 简介

SAT(Satisfiability) 问题是求使得形如 $x_i\lor x_j\lor \neg x_k$ 的多个条件式同时成立的一组取值，当每个条件的变量个数大于 2 时已经被证明无法用多项式的复杂度求解，但是如果只有 2 个可以在线性时间内求出可行解。

## 模板

> 给定 $n$ 个未赋值的布尔变量 $x_1 \sim x_n$，再给 $m$ 个条件，每个条件形如 $x_i =1,0 \lor x_j=1,0$，现在求一个合法解满足所有条件。
>
> 题目链接：[AcWing 2402](https://www.acwing.com/problem/content/2404/)，[P4782](https://www.luogu.com.cn/problem/P4782)。

在数理逻辑中，命题 $a\to b$ 的真值表如下：

| $a$  | $b$  | $a\to b$ |
| ---- | ---- | -------- |
| 0  | 0  | 1      |
| 0  | 1  | 1      |
| 1  | 0  | 0      |
| 1  | 1  | 1      |

所以 $a \to b$ 等价于 $\neg a\lor b$，反过来看也就是 $a \lor b$ 等价于 $\neg a\to b$ 也等价于 $\neg b\to a$，我们可以把 $\to$ 看作有向图的边，然后用 Tarjan 算法缩点。

判断是否产生矛盾的方法是看 $x,\neg x$ 是否在同一个强连通分量中，如果在说明无解。

对于每一个命题 $x$，都比较 $x,\neg x$ 强连通分量的拓扑序，选择靠后的那个，原因如下：

+ 若 $a,b$ 处在同一强连通分量中，那么 $\neg a,\neg b$ 也一定处在同一强连通分量中。

+ 若 $a$ 的拓扑序更靠前，设存在 $p\to a, \neg a\to \neg p$，用整值函数 $\varphi(x)$ 表示拓扑序，这里参考电势，我不知道是否存在一个更常用的符号用来表示拓扑序。数值小表示靠前，那么：
    $$
    \begin{aligned}
    \varphi(a)&\lt \varphi(\neg a)\\
    \varphi(p)&\le\varphi(a)\\
    \varphi(\neg p)&\ge\varphi(\neg a)\\
    \therefore \varphi(p)&<\varphi(\neg p)
    \end{aligned}
    $$
    
    此时选择靠后 $\neg a$ 的是一定可以构造出合法解的。
    
+  若 $a$ 的拓扑序更靠前，设存在边 $a\to q, \neg q\to \neg a$，首先假设 $a,q$ 不在同一强连通分量中，由于不存在矛盾：
    $$
    \begin{aligned}
    \varphi(q)&=\varphi(a)+1\\
    \varphi(\neg q)&=\varphi(\neg a)-1\\
    \because \varphi(q)&\neq \varphi(\neg q)\\
    \therefore \varphi(a)&\neq \varphi(\neg a)-2\\
    \end{aligned}
    $$
    如果 $\varphi(a)=\varphi(\neg a)-1$，那么 $q$ 必然与 $a$ 处在同一强连通分量中，题设就不成立了，所以：
    $$
    \begin{aligned}
    \varphi(a)&\le \varphi(\neg a)-3\\
    \varphi(a)+1 &\le \varphi(\neg a)-2\\
    \varphi(q)&\lt \varphi(\neg q)
    \end{aligned}
    $$
    此时也是选择靠后的仍然是合法解。

所以我们就选拓扑序靠后的，再加上 Tarjan 算法求出来的强连通分量编号与拓扑序存在逆相关的关系，所以我们就能得到求解 2-SAT 问题的代码了：

```cpp
#include <cstdio>
#include <cstring>
#include <stack>
#include <algorithm>
using namespace std;

const int N = 2000010, M = N;
int h[N], e[M], ne[M], idx;
int id[N], dfn[N], low[N], timestamp, scc_cnt;
stack<int> stk;
bool instk[N];

int n, m;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void tarjan(int u) {
    dfn[u] = low[u] = ++timestamp;
    instk[u] = true;
    stk.push(u);
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!dfn[j]) {
            tarjan(j);
            low[u] = min(low[u], low[j]);
        }
        else if (instk[j]) {
            low[u] = min(low[u], dfn[j]);
        }
    }
    
    if (dfn[u] == low[u]) {
        int y;
        ++scc_cnt;
        do {
            y = stk.top(); stk.pop();
            id[y] = scc_cnt;
            instk[y] = false;
        } while (y != u);
    }
}

int get(int v, int t) {
    return 2*v + t;
}

bool check() {
    for (int i = 1; i <= n; i++)
        if (id[get(i, 0)] == id[get(i, 1)]) return false;
    
    return true;
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d", &n, &m);
    while (m--) {
        int i, a, j, b;
        scanf("%d%d%d%d", &i, &a, &j, &b);
        add(get(i, a^1), get(j, b));
        add(get(j, b^1), get(i, a));
    }
    
    for (int i = 2; i <= 2*n+1; i++)
        if (!dfn[i]) tarjan(i);
    
    if (!check()) puts("IMPOSSIBLE");
    else {
        puts("POSSIBLE");
        for (int i = 1; i <= n; i++) {
            // xi=0 的点拓扑序靠后
            if (id[get(i, 0)] < id[get(i, 1)]) printf("0 ");
            else printf("1 ");
        }
        puts("");
    }
    return 0;
}
```



