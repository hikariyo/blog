---
date: 2023-08-21 14:50:00
updated: 2023-08-21 14:50:00
title: 数据结构 - Dancing Links
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
---

## 简介

Dancing Links 是用来解决精确覆盖问题和重复覆盖问题的一个数据结构，它的优势在于一旦问题被转化成精确覆盖或重复覆盖问题，那么就不需要自己想剪枝策略了，直接用 DLX 的剪枝就能达到一个优秀的效果。

DLX 使用十字双向循环链表存储，当然是用数组模拟的链表。

每列对应一个限制条件，每行对应一个选择方案，精确覆盖问题就是满足每个限制条件的同时每个限制条件只被一个选择方案满足，而重复覆盖问题就是一个限制条件可被多个选法满足。

## 精确覆盖问题

> 给定一个 N×M 的 0/1 矩阵 A，找到一个行的集合，使得这些行中，每一列都有且仅有一个数字 1。
>
> 题目链接：[AcWing 1067](https://www.acwing.com/problem/content/1069/)，[P4929](https://www.luogu.com.cn/problem/P4929)。

精确覆盖问题中的删除操作需要使得一个点对应的列被删除掉、当前行对应的所有列被删掉。

当删除一列的时候，只更改首行哨兵结点的左右指针即可。要让这个列不被重复选中，因此需要把一列对应的所有行都删掉。只需要更改上下指针即可，无需更改左右指针，因为别的行是不包括当前这一列的。

删除某列中某行不代表被删除的行对应的所有列都会被删除，因此需要更改上下指针。

恢复每一行的时候使用每个结点的左右指针遍历，恢复每个点的上下指针。恢复列的时候只需要把哨兵结点更改正确即可。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 6000;
int r[N], l[N], d[N], u[N], row[N], col[N], s[N], idx;
int ans[N], top;
int n, m;

void init() {
    for (int i = 0; i <= m; i++) {
        l[i] = i-1, r[i] = i+1;
        u[i] = d[i] = i;
    }
    l[0] = m, r[m] = 0;
    idx = m+1;
}

void add(int& hh, int& tt, int x, int y) {
    s[y]++, col[idx] = y, row[idx] = x;
    u[idx] = y, d[idx] = d[y], u[d[y]] = idx, d[y] = idx;
    l[idx] = hh, r[idx] = tt, r[hh] = l[tt] = idx;
    tt = idx++;
}

void remove(int p) {
    r[l[p]] = r[p], l[r[p]] = l[p];
    for (int i = d[p]; i != p; i = d[i]) {
        for (int j = r[i]; j != i; j = r[j]) {
            s[col[j]]--;
            u[d[j]] = u[j], d[u[j]] = d[j];
        }
    }
}

void resume(int p) {
    for (int i = u[p]; i != p; i = u[i]) {
        for (int j = l[i]; j != i; j = l[j]) {
            u[d[j]] = d[u[j]] = j;
            s[col[j]]++;
        }
    }
    r[l[p]] = l[r[p]] = p;
}

bool dfs() {
    if (!r[0]) return true;
    int p = r[0];
    for (int i = r[0]; i; i = r[i]) {
        if (s[i] < s[p]) p = i;
    }
    
    remove(p);
    for (int i = d[p]; i != p; i = d[i]) {
        ans[top++] = row[i];
        for (int j = r[i]; j != i; j = r[j]) remove(col[j]);
        if (dfs()) return true;
        for (int j = l[i]; j != i; j = l[j]) resume(col[j]);
        top--;
    }
    resume(p);
    
    return false;
}

int main() {
    scanf("%d%d", &n, &m);
    init();
    for (int i = 1; i <= n; i++) {
        int hh = idx, tt = idx;
        for (int j = 1; j <= m; j++) {
            int v; scanf("%d" ,&v);
            if (v) add(hh, tt, i, j);
        }
    }
    
    if (!dfs()) puts("No Solution!");
    else {
        for (int i = 0; i < top; i++)
            printf("%d ", ans[i]);
        puts("");
    }
    
    return 0;
}
```

## 重复覆盖问题

> 给定一个 N×M 的 0/1 矩阵 A，找到一个行的集合，使得这些行中，每一列至少包含一个数字 1。
>
> 题目链接：[AcWing 2713](https://www.acwing.com/problem/content/2715/)。

重复覆盖问题中的删除操作需要使得一个点对应的列被删除掉、当前行对应的所有列被删掉。

当删除一列的时候，这个列是可以重复选中的，因此需要修改每个点的左右指针（包含哨兵结点），因为其它行也可能包含当前列。

因为包行当前行的所有列都被删了，所以这一行就不会被遍历到了。

恢复当前列的时候使用上下指针遍历，恢复每个结点的左右指针。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 10010;
int u[N], l[N], r[N], d[N], row[N], col[N], s[N], idx;
int ans[N];
int n, m;
bool st[N];

void init() {
    for (int i = 0; i <= m; i++) {
        l[i] = i-1, r[i] = i+1;
        d[i] = u[i] = i;
    }
    l[0] = m, r[m] = 0;
    idx = m+1;
}

void add(int& hh, int& tt, int x, int y) {
    s[y]++, row[idx] = x, col[idx] = y;
    l[tt] = r[hh] = idx, l[idx] = hh, r[idx] = tt;
    u[d[y]] = idx, u[idx] = y, d[idx] = d[y], d[y] = idx;
    tt = idx++;
}

void remove(int p) {
    for (int i = d[p]; i != p; i = d[i]) {
        l[r[i]] = l[i];
        r[l[i]] = r[i];
    }
}

void resume(int p) {
    for (int i = u[p]; i != p; i = u[i]) {
        l[r[i]] = r[l[i]] = i;
    }
}

int h() {
    memset(st, 0, sizeof st);
    int cnt = 0;
    for (int i = r[0]; i; i = r[i]) {
        if (st[i]) continue;
        cnt++;
        st[i] = true;
        for (int j = d[i]; j != i; j = d[j])
            for (int k = r[j]; k != j; k = r[k])
                st[col[k]] = true;
    }
    return cnt;
}

bool dfs(int u, int depth) {
    if (u + h() > depth) return false;
    if (!r[0]) return true;
    
    int p = r[0];
    for (int i = r[0]; i; i = r[i])
        if (s[i] < s[p]) p = i;
    
    for (int i = d[p]; i != p; i = d[i]) {
        ans[u] = row[i];
        remove(i);
        for (int j = r[i]; j != i; j = r[j]) remove(j);
        if (dfs(u+1, depth)) return true;
        for (int j = l[i]; j != i; j = l[j]) resume(j);
        resume(i);
    }
    return false;
}

int main() {
    scanf("%d%d", &n, &m);
    init();
    for (int i = 1; i <= n; i++) {
        int hh = idx, tt = idx;
        for (int j = 1; j <= m; j++) {
            int v; scanf("%d", &v);
            if (v) add(hh, tt, i, j);
        }
    }
    
    int depth = 0;
    while (!dfs(0, depth)) depth++;
    printf("%d\n", depth);
    for (int i = 0; i < depth; i++) {
        printf("%d ", ans[i]);
    }
    puts("");
    return 0;
}
```

## 数独

> 填数独，没啥好说的。
>
> 题目链接：[P1784](https://www.luogu.com.cn/problem/P1784)。

本题可以用位运算优化的 DFS 解，这里转化成精确覆盖问题用 DLX 求解。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 3000, M = 800;
const int m = 81*4;
int g[N][N];
int d[N], l[N], r[N], u[N], s[N], row[N], col[N], idx;
int ans[N], top;

struct Operation {
    int x, y, p;
} op[M];

void init() {
    for (int i = 0; i <= m; i++) {
        l[i] = i-1, r[i] = i+1;
        u[i] = d[i] = i;
    }
    l[0] = m, r[m] = 0;
    idx = m+1;
}

void add(int& hh, int& tt, int x, int y) {
    row[idx] = x, col[idx] = y, s[y]++;
    r[hh] = l[tt] = idx, r[idx] = tt, l[idx] = hh;
    u[d[y]] = idx, u[idx] = y, d[idx] = d[y], d[y] = idx;
    tt = idx++;
}

void remove(int p) {
    l[r[p]] = l[p], r[l[p]] = r[p];
    for (int i = d[p]; i != p; i = d[i]) {
        for (int j = r[i]; j != i; j = r[j]) {
            u[d[j]] = u[j], d[u[j]] = d[j];
            s[col[j]]--;
        }
    }
}

void resume(int p) {
    for (int i = u[p]; i != p; i = u[i]) {
        for (int j = l[i]; j != i; j = l[j]) {
            u[d[j]] = d[u[j]] = j;
            s[col[j]]++;
        }
    }
    l[r[p]] = r[l[p]] = p;
}

bool dfs() {
    if (!r[0]) return true;
    
    int p = r[0];
    for (int i = r[0]; i; i = r[i]) {
        if (s[i] < s[p]) p = i;
    }
    remove(p);
    for (int i = d[p]; i != p; i = d[i]) {
        ans[top++] = row[i];
        for (int j = r[i]; j != i; j = r[j]) remove(col[j]);
        if (dfs()) return true;
        for (int j = l[i]; j != i; j = l[j]) resume(col[j]);
        top--;
    }
    resume(p);
    return false;
}

int main() {
    init();
    
    for (int i = 0; i < 9; i++)
        for (int j = 0; j < 9; j++)
            scanf("%d", &g[i][j]);
    
    for (int i = 0, n = 1; i < 9; i++)
        for (int j = 0; j < 9; j++) {
            int a = 0, b = 8;
            if (g[i][j]) a = b = g[i][j] - 1;
            
            for (int k = a; k <= b; k++, n++) {
                int hh = idx, tt = idx;
                op[n] = {i, j, k+1};
                add(hh, tt, n, i*9 + j + 1);
                add(hh, tt, n, 81*1 + i*9 + k + 1);
                add(hh, tt, n, 81*2 + j*9 + k + 1);
                add(hh, tt, n, 81*3 + (i/3*3 + j/3)*9 + k + 1);
            }
        }
    
    dfs();
    
    for (int i = 0; i < top; i++) {
        Operation t = op[ans[i]];
        g[t.x][t.y] = t.p;
    }
    
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++)
            printf("%d ", g[i][j]);
        puts("");
    }
    
    return 0;
}
```

## [NOIP2009] 靶形数独

> 小城和小华都是热爱数学的好学生，最近，他们不约而同地迷上了数独游戏，好胜的他们想用数独来一比高低。但普通的数独对他们来说都过于简单了，于是他们向 Z 博士请教， Z 博士拿出了他最近发明的「靶形数独」，作为这两个孩子比试的题目。
>
> 靶形数独的方格同普通数独一样，在 9 格宽 9 格高 的大九宫格中有 9 个 3 格宽 3 格高 的小九宫格（用粗黑色线隔开的）。在这个大九宫格中，有一些数字是已知的，根据这些数字，利用逻辑推理，在其他的空格上填入 1 到 9 的数字。每个数字在每个小九宫格内不能 重复出现，每个数字在每行、每列也不能重复出现。但靶形数独有一点和普通数独不同，即 每一个方格都有一个分值，而且如同一个靶子一样，离中心越近则分值越高。
>
> （图略，参考下面给出的题目链接）
>
> 上图具体的分值分布是：里面一格（黄色区域）为 10 分，黄色区域外面的一圈（红色区域）每个格子为 9 分，再外面一圈（蓝色区域）每个格子为 8 分，蓝色区域外面一圈（棕色区域）每个格子为 7 分，外面一圈（白色区域）每个格子为 6 分，如上图所示。
>
> 比赛的要求是：每个人必须完成一个给定的数独（每个给定数独可能有不同的填法），而且要争取更高的总分数。而这个总分数即每个方格上的分值和完成这个数独时填在相应格上的数字的乘积的总和。
>
> 如图，在以下的这个已经填完数字的靶形数独游戏中，总分数为 2829 。游戏规定，将以总分数的高低决出胜负。
>
> 题目链接：[LOJ 2591](https://loj.ac/p/2591)，[P1074](https://www.luogu.com.cn/problem/P1074)。

直接套用上题代码框架来做。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 3000, M = 800;
const int m = 81*4;
int g[N][N];
int d[N], l[N], r[N], u[N], s[N], row[N], col[N], idx;
int ans[N], top;
int maxv = -1;

int score[9][9] = {
    {6, 6, 6, 6, 6, 6, 6, 6, 6},
    {6, 7, 7, 7, 7, 7, 7, 7, 6},
    {6, 7, 8, 8, 8, 8, 8, 7, 6},
    {6, 7, 8, 9, 9, 9, 8, 7, 6},
    {6, 7, 8, 9, 10,9, 8, 7, 6},
    {6, 7, 8, 9, 9, 9, 8, 7, 6},
    {6, 7, 8, 8, 8, 8, 8, 7, 6},
    {6, 7, 7, 7, 7, 7, 7, 7, 6},
    {6, 6, 6, 6, 6, 6, 6, 6, 6},
};

struct Operation {
    int x, y, p;
} op[M];

void init() {
    for (int i = 0; i <= m; i++) {
        l[i] = i-1, r[i] = i+1;
        u[i] = d[i] = i;
    }
    l[0] = m, r[m] = 0;
    idx = m+1;
}

void add(int& hh, int& tt, int x, int y) {
    row[idx] = x, col[idx] = y, s[y]++;
    r[hh] = l[tt] = idx, r[idx] = tt, l[idx] = hh;
    u[d[y]] = idx, u[idx] = y, d[idx] = d[y], d[y] = idx;
    tt = idx++;
}

void remove(int p) {
    l[r[p]] = l[p], r[l[p]] = r[p];
    for (int i = d[p]; i != p; i = d[i]) {
        for (int j = r[i]; j != i; j = r[j]) {
            u[d[j]] = u[j], d[u[j]] = d[j];
            s[col[j]]--;
        }
    }
}

void resume(int p) {
    for (int i = u[p]; i != p; i = u[i]) {
        for (int j = l[i]; j != i; j = l[j]) {
            u[d[j]] = d[u[j]] = j;
            s[col[j]]++;
        }
    }
    l[r[p]] = r[l[p]] = p;
}

void dfs() {
    if (!r[0]) {
        int res = 0;
        for (int i = 0; i < top; i++) {
            Operation t = op[ans[i]];
            res += score[t.x][t.y] * t.p;
        }
        
        maxv = max(maxv, res);
        return;
    }
    
    int p = r[0];
    for (int i = r[0]; i; i = r[i]) {
        if (s[i] < s[p]) p = i;
    }
    remove(p);
    for (int i = d[p]; i != p; i = d[i]) {
        ans[top++] = row[i];
        for (int j = r[i]; j != i; j = r[j]) remove(col[j]);
        dfs();
        for (int j = l[i]; j != i; j = l[j]) resume(col[j]);
        top--;
    }
    resume(p);
}

int main() {
    init();
    
    for (int i = 0; i < 9; i++)
        for (int j = 0; j < 9; j++)
            scanf("%d", &g[i][j]);
    
    for (int i = 0, n = 1; i < 9; i++)
        for (int j = 0; j < 9; j++) {
            int a = 0, b = 8;
            if (g[i][j]) a = b = g[i][j] - 1;
            
            for (int k = a; k <= b; k++, n++) {
                int hh = idx, tt = idx;
                op[n] = {i, j, k+1};
                add(hh, tt, n, i*9 + j + 1);
                add(hh, tt, n, 81*1 + i*9 + k + 1);
                add(hh, tt, n, 81*2 + j*9 + k + 1);
                add(hh, tt, n, 81*3 + (i/3*3 + j/3)*9 + k + 1);
            }
        }
    
    dfs();
    
    printf("%d\n", maxv);
    
    return 0;
}
```

## SUDOKU

> 请你将一个 16×16 的数独填写完整，使得每行、每列、每个 4×4 十六宫格内字母 A∼P 均恰好出现一次。
>
> 保证每个输入只有唯一解决方案。
>
> 题目链接：[AcWing 171](https://www.acwing.com/problem/content/171/)，[SP1110(格式不同)](https://www.spoj.com/problems/SUDOKU/)，[SP1110(Luogu)](https://www.luogu.com.cn/problem/SP1110)。

九宫格数独加强 PLUS PLUS 版本，但是实际上用 DLX 套上就直接出来了，和上面那个橙题的代码一样（当然这是因为我用 DLX 解了那个橙题 hhh）洛谷给这题评了个黑，但我觉得也就是紫。

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 20000, m = 1024;

char g[20][20];
int d[N], u[N], l[N], r[N], row[N], col[N], s[N], idx;
int ans[N], top;

struct Operation {
    int x, y;
    char ch;
} op[N];

void init() {
    memset(s, 0, sizeof s);
    for (int i = 0; i <= m; i++) {
        l[i] = i-1, r[i] = i+1;
        u[i] = d[i] = i;
    }
    l[0] = m, r[m] = 0;
    idx = m+1;
    top = 0;
}

void add(int &hh, int& tt, int x, int y) {
    row[idx] = x, col[idx] = y, s[y]++;
    u[d[y]] = idx, d[idx] = d[y], d[y] = idx, u[idx] = y;
    l[idx] = hh, r[idx] = tt, r[hh] = l[tt] = idx;
    tt = idx++;
}

void build() {
    for (int i = 0, n = 1; i < 16; i++)
        for (int j = 0; j < 16; j++, n++) {
            int a = 0, b = 15;
            if (g[i][j] != '-') a = b = g[i][j]-'A';
            for (int k = a; k <= b; k++, n++) {
                // n: 16*16*16*4 < 20000
                int hh = idx, tt = idx;
                op[n] = {i, j, k+'A'};
                add(hh, tt, n, i*16+j+1);
                add(hh, tt, n, 256*1 + i*16+k+1);
                add(hh, tt, n, 256*2 + j*16+k+1);
                add(hh, tt, n, 256*3 + (i/4*4 + j/4)*16+k+1);
            }
        }
}

void remove(int p) {
    l[r[p]] = l[p], r[l[p]] = r[p];
    for (int i = d[p]; i != p; i = d[i])
        for (int j = r[i]; j != i; j = r[j])
            u[d[j]] = u[j], d[u[j]] = d[j], s[col[j]]--;
}

void resume(int p) {
    for (int i = u[p]; i != p; i = u[i])
        for (int j = l[i]; j != i; j = l[j])
            u[d[j]] = d[u[j]] = j, s[col[j]]++;
    l[r[p]] = r[l[p]] = p;
}

bool dfs() {
    if (!r[0]) return true;
    int p = r[0];
    for (int i = r[0]; i; i = r[i])
        if (s[i] < s[p]) p = i;
    
    remove(p);
    for (int i = d[p]; i != p; i = d[i]) {
        ans[top++] = row[i];
        for (int j = r[i]; j != i; j = r[j]) remove(col[j]);
        if (dfs()) return true;
        for (int j = l[i]; j != i; j = l[j]) resume(col[j]);
        top--;
    }
    resume(p);
    return false;
}

void output() {
    for (int i = 0; i < top; i++) {
        Operation t = op[ans[i]];
        g[t.x][t.y] = t.ch;
    }
    for (int i = 0; i < 16; i++) puts(g[i]);
    puts("");
}

int main() {
    while (~scanf("%s", g[0])) {
        for (int i = 1; i < 16; i++) scanf("%s", g[i]);
        init();
        build();
        dfs();
        output();
    }
    return 0;
}
```



