---
date: 2023-08-10 16:42:00
title: 数据结构 - AC 自动机
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
---

## 搜索关键词

> 给定 n 个长度不超过 50 的由小写英文字母组成的单词，以及一篇长为 m 的文章。
>
> 请问，其中有多少个单词在文章中出现了。
>
> 注意：每个单词不论在文章中出现多少次，仅累计 1 次。
>
> 题目链接：[AcWing 1282](https://www.acwing.com/problem/content/1284/)。

类似 KMP 算法中的 next 数组，AC 自动机是在一个 Trie 树上进行匹配，模板串由一个字符串变为一个字符串集，这样一来 next 表示的含义就是当前结点对应后缀能回退到的最大前缀。

以类似数学归纳法的思想，假设 $next$ 数组是完全正确的，建立 AC 自动机的时候，对于其中的任意一个结点 $t$ 及其子结点 $p$，对于边 $i=(t,p)$：

+ 当子结点 $p$ 存在时，有 $next(p)=next(t).i$，如果不存在自然为 0 表示根节点。
+ 当子结点 $p$ 不存在时，直接令 $p=next(t).i$，这样在原串匹配的时候就不需要手动做回退操作了。

这样就构成了一个 Tire 图。

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
using namespace std;

const int N = 10010, M = 1000010, S = 55;
int tr[N*S][26], cnt[N*S], ne[N*S], idx;
char str[M];

void insert() {
    int p = 0;
    for (int i = 0; str[i]; i++) {
        int t = str[i] - 'a';
        if (!tr[p][t]) tr[p][t] = ++idx;
        p = tr[p][t];
    }
    cnt[p]++;
}

void build() {
    queue<int> q;
    for (int i = 0; i < 26; i++) {
        if (tr[0][i]) q.push(tr[0][i]);
    }
    
    while (q.size()) {
        int t = q.front(); q.pop();
        for (int i = 0; i < 26; i++) {
            int p = tr[t][i];
            if (!p) tr[t][i] = tr[ne[t]][i];
            else {
                ne[p] = tr[ne[t]][i];
                q.push(p);
            }
        }
    }
}

int main() {
    int T, n;
    scanf("%d", &T);
    while (T--) {
        memset(tr, 0, sizeof tr);
        memset(cnt, 0, sizeof cnt);
        memset(ne, 0, sizeof ne);
        idx = 0;
        
        scanf("%d", &n);
        while (n--) {
            scanf("%s", str);
            insert();
        }
        
        build();
        scanf("%s", str);
        int res = 0;
        for (int i = 0, j = 0; str[i]; i++) {
            int t = str[i] - 'a';
            j = tr[j][t];
            for (int p = j; p; p = ne[p]) {
                res += cnt[p];
                cnt[p] = 0;
            }
        }
        printf("%d\n", res);
    }
    return 0;
}
```

## 修复 DNA

> DNA看作是一个由’A’, ‘G’ , ‘C’ , ‘T’构成的字符串。
>
> 修复技术就是通过改变字符串中的一些字符，从而消除字符串中包含的致病片段。
>
> 例如，我们可以通过改变两个字符，将DNA片段”AAGCAG”变为”AGGCAC”，从而使得DNA片段中不再包含致病片段”AAG”，”AGC”，”CAG”，以达到修复该DNA片段的目的。
>
> 需注意，被修复的DNA片段中，仍然只能包含字符’A’, ‘G’ , ‘C’ , ‘T’。
>
> 请你帮助生物学家修复给定的DNA片段，并且修复过程中改变的字符数量要尽可能的少。
>
> 题目链接：[AcWing 1053](https://www.acwing.com/problem/content/1055/)。

本题是 AC 自动机 + 状态机 DP

+ 状态表示 $f(i,j)$：

    + 含义：前 $i$ 个字符中，在 AC 自动机里走到第 $j$ 个结点的所有方案的集合。
    + 存储：最小修改次数的值。

+ 状态转移方程，当 $j\to p$ 是可行的时候：
    $$
    f(i,p)=\min\{f(i-1,j)+\text{cost}\}
    $$
    上面的 cost 是枚举将下一个字符改成 ATCG 中的某个字符时，是否发生了改变。

    如果修改后原字符串不合法，即含给定字符串集，就跳过。

+ 初始化：$f(0,0)=0, f(i,j)=\inf$

+ 答案：$\min_{j=0}^{total} f(n,j)$

由于只有四个代表碱基的字符，这里做一个简单的离散化，将其映射到 [0, 1, 2, 3] 节省空间。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>
using namespace std;

const int INF = 0x3f3f3f3f, N = 1500;
char str[N];
int tr[N][4], ne[N], f[N][N], n, idx;
bool st[N];

// 我写的时候这里返回了 1, 2, 3, 4 导致数组越界然后出来奇怪结果
int get(char c) {
    switch (c) {
        case 'A': return 0;
        case 'T': return 1;
        case 'G': return 2;
        default: return 3;
    }
}

void insert() {
    int p = 0;
    for (int i = 0; str[i]; i++) {
        int t = get(str[i]);
        if (!tr[p][t]) tr[p][t] = ++idx;
        p = tr[p][t];
    }
    st[p] = true;
}

void build() {
    queue<int> q;
    for (int i = 0; i < 4; i++)
        if (tr[0][i]) q.push(tr[0][i]);
    
    while (q.size()) {
        int t = q.front(); q.pop();
        for (int i = 0; i < 4; i++) {
            int p = tr[t][i];
            if (!p) tr[t][i] = tr[ne[t]][i];
            else ne[p] = tr[ne[t]][i], q.push(p);
        }
    }
}

bool check(int p) {
    for (; p; p = ne[p]) {
        if (st[p]) return false;
    }
    return true;
}

int solve() {
    memset(f, 0x3f, sizeof f);
    memset(tr, 0, sizeof tr);
    memset(ne, 0, sizeof ne);
    memset(st, 0, sizeof st);
    idx = 0;
    f[0][0] = 0;
    while (n--) {
        scanf("%s", str);
        insert();
    }
    build();
    scanf("%s", str+1);
    n = strlen(str+1);
    for (int i = 1; i <= n; i++)
        for (int j = 0; j <= idx; j++)
            for (int k = 0; k < 4; k++) {
                int cost = get(str[i]) != k;
                int p = tr[j][k];
                if (check(p)) f[i][p] = min(f[i][p], f[i-1][j] + cost);
            }
    
    int res = INF;
    for (int j = 0; j <= idx; j++) res = min(res, f[n][j]);
    if (res == INF) return -1;
    return res;
}

int main() {
    int T = 0;
    while (scanf("%d", &n), n)
        printf("Case %d: %d\n", ++T, solve());
    return 0;
}
```

然而，这种做法并不是最优解，还能继续优化，在 `check` 函数上。

当构造 AC 自动机的时候，就让 `st[p]` 关注到 `ne[p]`，如果 `st[ne[p]]` 是 `true` 也把 `st[p]` 置为 `true`，这个从语义上理解是，如果某个后缀所匹配到的前缀是一个目标字符串，那么就把那个前缀的标记也打到后缀的这个结点上，比如 `GAACT` 和 `AAC`，在 Trie 树的 `GAACT` 的 `C` 结点上也打上标记，就不用检测是否存在目标字符串的时候进行一个迭代操作了。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>
using namespace std;

const int N = 1500, INF = 0x3f3f3f3f;
int n, tr[N][4], ne[N], f[N][N], idx;
bool st[N];
char str[N];

int get(char c) {
    switch (c) {
        case 'A': return 0;
        case 'T': return 1;
        case 'G': return 2;
        default: return 3;
    }
}

void insert() {
    int p = 0;
    for (int i = 0; str[i]; i++) {
        int t = get(str[i]);
        if (!tr[p][t]) tr[p][t] = ++idx;
        p = tr[p][t];
    }
    st[p] = true;
}

void build() {
    queue<int> q;
    for (int i = 0; i < 4; i++)
        if (tr[0][i]) q.push(tr[0][i]);
    
    while (q.size()) {
        int t = q.front(); q.pop();
        for (int i = 0; i < 4; i++) {
            int p = tr[t][i];
            if (!p) tr[t][i] = tr[ne[t]][i];
            else {
                ne[p] = tr[ne[t]][i];
                q.push(p);
                st[p] |= st[ne[p]];
            }
        }
    }
}

int solve() {
    memset(tr, 0, sizeof tr);
    memset(ne, 0, sizeof ne);
    memset(st, 0, sizeof st);
    memset(f, 0x3f, sizeof f);
    idx = 0;
    f[0][0] = 0;
    
    while (n--) {
        scanf("%s", str);
        insert();
    }
    build();
    scanf("%s", str+1);
    n = strlen(str+1);
    for (int i = 1; i <= n; i++)
        for (int j = 0; j <= idx; j++)
            for (int k = 0; k < 4; k++) {
                int cost = get(str[i]) != k;
                int p = tr[j][k];
                if (!st[p]) f[i][p] = min(f[i][p], f[i-1][j] + cost);
            }
    
    int res = INF;
    for (int j = 0; j <= idx; j++) res = min(res, f[n][j]);
    if (res == INF) res = -1;
    return res;
}

int main() {
    int T = 0;
    while (scanf("%d", &n), n)
        printf("Case %d: %d\n", ++T, solve());
    return 0;
}
```

## 单词

> 某人读论文，一篇论文是由许多单词组成的。
>
> 但他发现一个单词会在论文中出现很多次，现在他想知道每个单词分别在论文中出现多少次。
>
> **输入格式**
>
> 第一行一个整数 N，表示有多少个单词。
>
> 接下来 N 行每行一个单词，单词中只包含小写字母。
>
> **输出格式**
>
> 输出 N 个整数，每个整数占一行，第 i 行的数字表示第 i 个单词在文章中出现了多少次。
>
> 题目链接：[AcWing 1285](https://www.acwing.com/problem/content/1287/)，[P3966](https://www.luogu.com.cn/problem/P3966)。

将一个单词出现的次数，联系 AC 自动机的 next 定义，单词出现的次数可以转化成单词在所有前缀的后缀中出现的次数，比如 `abcbc` 和 `bc`，单词 `bc` 在前缀 `abc, abcbc, bc` 的后缀中出现，所以就会出现 3 次。

因此只要按照拓扑序累加一遍就行。

```cpp
#include <cstdio>
#include <cstring>
#include <queue>
using namespace std;

const int N = 1e6+10;
int tr[N][26], cnt[N], ne[N], q[N], id[210], n, idx;
char str[N];

void insert(int x) {
    int p = 0;
    for (int i = 0; str[i]; i++) {
        int t = str[i] - 'a';
        if (!tr[p][t]) tr[p][t] = ++idx;
        p = tr[p][t];
        cnt[p]++;
    }
    id[x] = p;
}

void build() {
    int hh = 0, tt = -1;
    for (int i = 0; i < 26; i++)
        if (tr[0][i]) q[++tt] = tr[0][i];
    
    while (hh <= tt) {
        int t = q[hh++];
        for (int i = 0; i < 26; i++) {
            int& p = tr[t][i];
            if (!p) p = tr[ne[t]][i];
            else ne[p] = tr[ne[t]][i], q[++tt] = p;
        }
    }
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%s", str);
        insert(i);
    }
    build();
    
    // 所有除了 0 的节点都要加进队列 一共有 idx+1 个
    for (int i = idx-1; i >= 0; i--) {
        int t = q[i];
        cnt[ne[t]] += cnt[t];
    }
    
    for (int i = 1; i <= n; i++)
        printf("%d\n", cnt[id[i]]);
    
    return 0;
}
```

$$
|x|=\begin{cases}
x& x>0\\
0& x=0\\
-x& x<0
\end{cases}
$$

