---
date: 2023-07-24 17:36:36
title: 图论 - 欧拉路径和欧拉回路
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 判定

所有边都需要连通，其次：

1. 对于无向图
    + 存在欧拉路径的充要条件：度数为奇数的点只能有0或2个。
    + 存在欧拉回路的充要条件：不能有度数为奇数的点。
2. 对于有向图
    + 存在欧拉路径的充要条件：所有点入度等于出度，或者有两个点分别满足 `din = dout+1, dout=din+1`。
    + 存在欧拉回路的充要条件：所有点入度等于出度。

## 欧拉回路

> 给定一张图，请你找出欧拉回路，即在图中找一个环使得每条边都在环上出现恰好一次。
>
> **输入格式**
>
> 第一行包含一个整数 $t\in\{1,2\}$，如果 t=1，表示所给图为**无向**图；如果 t=2，表示所给图为**有向**图。
>
> 第二行包含两个整数 n,m，表示图的结点数和边数。
>
> 接下来 m 行中，第 i 行两个整数 vi,ui，表示第 i 条边（从 1 开始编号）。
>
> - 如果 t=1 则表示 vi 到 ui 有一条无向边。
> - 如果 t=2 则表示 vi 到 ui 有一条有向边。
>
> 图中可能有重边也可能有自环。
>
> 点的编号从 1 到 n。
>
> **输出格式**
>
> 如果无法一笔画出欧拉回路，则输出一行：NO。
>
> 否则，输出一行：YES，接下来一行输出 **任意一组** 合法方案即可。
>
> - 如果 t=1，输出 m 个整数 p1,p2,…,pm。令 $e=|p_i|$，那么 e 表示经过的第 i 条边的编号。如果 pi 为正数表示从 $v_e$ 走到 $u_e$，否则表示从 ue 走到 ve。
> - 如果 t=2，输出 m 个整数 p1,p2,…,pm。其中 pi 表示经过的第 i 条边的编号。
>
> 题目链接：[AcWing 1184](https://www.acwing.com/problem/content/1186/)。

每次遍历的一条边的时候直接把这条边删除，如果是无向图也把反向边删除。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int N = 1e5+10, M = 4e5+10;
int type, n, m;
int h[N], e[M], ne[M], din[N], dout[N], idx;
bool used[M];
int ans[N*2], cnt;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

bool check() {
    for (int i = 1; i <= n; i++)
        if (type == 1) {
            if (din[i] + dout[i] & 1)
                return false;
        } else if (din[i] != dout[i]) {
            return false;
        }
    return true;
}

void dfs(int u) {
    // 表头是不断往下移动的，而 ne 是不变的
    // 如果写 i=ne[i] 遇到自环就被卡了
    for (int i = h[u]; ~i; i = h[u]) {
        h[u] = ne[i];
        
        if (used[i]) {
            continue;
        }
        
        used[i] = true;  // 这句不加也行 强迫症
        if (type == 1) used[i^1] = true;
        
        dfs(e[i]);
        
        if (type == 1) {
            // (0, 1) -> 1
            // (2, 3) -> 2
            // (4, 5) -> 3
            int t = i/2 + 1;
            if (i & 1) t = -t;
            ans[cnt++] = t;
        } else ans[cnt++] = i+1;
    }
}

int main() {
    cin >> type >> n >> m;
    memset(h, -1, sizeof h);
    for (int i = 0; i < m; i++) {
        int a, b;
        cin >> a >> b;
        add(a, b);
        if (type == 1) add(b, a);
        dout[a]++, din[b]++;
    }
    
    if (!check()) {
        cout << "NO\n";
        return 0;
    }
    
    // 随便找一个能向下走的点开始 因为每个点不一定都用上了
    for (int i = 1; i <= n; i++)
        if (~h[i]) {
            dfs(i);
            break;
        }
    
    if (cnt < m) {
        cout << "NO\n";
        return 0;
    }
    
    cout << "YES\n";
    for (int i = cnt-1; i >= 0; i--) cout << ans[i] << ' ';
    cout << '\n';
    return 0;
}
```

DFS 逆序记录的原因在于，观察下面这种中间带环的图：

![](/img/2023/07/acw1184.jpg)

答案显然是 `1 2 3 4 5 6 7` 或者 `1 5 6 3 4 2 7`，按照 DFS 序逆序添加，如下：

```text
1
- 2
-- 7
- 5
-- 6
--- 3
---- 4
```

得到的序列是 `7 2 4 3 6 5 1`，倒过来就是一个合法解。

## 单词游戏

> 有 N 个盘子，每个盘子上写着一个仅由小写字母组成的英文单词。
> 你需要给这些盘子安排一个合适的顺序，使得相邻两个盘子中，前一个盘子上单词的末字母等于后一个盘子上单词的首字母。
>
> 请你编写一个程序，判断是否能达到这一要求。
>
> 题目链接：[AcWing 1185](https://www.acwing.com/problem/content/1187/)。

每个单词从首字母到末字母连边，判断是否存在欧拉路径。

```cpp
#include <cstdio>
#include <cstring>
using namespace std;

const int N = 26;
int din[N], dout[N], g[N][N], cnt;
char s[1010];

void dfs(int u) {
    for (int i = 0; i < N; i++) {
        if (g[u][i]) {
            g[u][i]--;
            dfs(i);
            cnt++;
        }
    }
}

bool solve() {
    memset(din, 0, sizeof din);
    memset(dout, 0, sizeof dout);
    memset(g, 0, sizeof g);
    cnt = 0;
    int n;
    scanf("%d", &n);
    for (int i = 0; i < n; i++) {
        scanf("%s", s);
        int a = s[0]-'a', b = s[strlen(s)-1]-'a';
        din[b]++, dout[a]++;
        g[a][b]++;
    }
    
    int start = 0, end = 0, t = 0;
    for (int i = 0; i < N; i++)
        if (din[i] != dout[i]) {
            if (din[i] == dout[i] + 1) end++;
            else if (dout[i] = din[i] + 1) start++, t = i;
            else return false;
        }
    
    if (!(start == 1 && end == 1 || start == 0 && end == 0))
        return false;
    
    dfs(t);
    if (cnt < n) return false;
    return true;
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) puts(solve() ? "Ordering is possible." : "The door cannot be opened.");
    return 0;
}
```

或者也可以用并查集判断连通性，这里不再演示。

