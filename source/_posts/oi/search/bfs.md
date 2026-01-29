---
date: 2023-07-12 10:11:12
updated: 2023-07-14 14:34:12
title: 搜索 - BFS
katex: true
tags:
- Algo
- C++
- Search
categories:
- OI
---

## 简介

BFS(Broad First Search) 是一种逐层遍历的搜索模式，用队列储存待遍历的点（因此不会爆栈），应用于查找最短路径等场景，一般格式如下：

```cpp
q.push(init);
while (q.size()) {
    int t = q.front(); q.pop();
    for (xxx) if (xxx) q.push(xxx);
}
```

若是寻找连通块，用道 Flood-fill 算法，那么大概格式如下：

```cpp
for (int i = 0; i < n; i++)
    for (int j = 0; j < m; j++)
        if (!st[i][j]) bfs(i, j);
```

下面以几道例题记录 BFS 使用方式。


## Flood-fill 算法

### 池塘计数

> 给 N * M 池塘，部分土地被水淹没，现在用一个字符矩阵来表示他的土地。
>
> 每个单元格内，如果包含雨水，则用”W”表示，如果不含雨水，则用”.”表示。
>
> 求这片土地中含多少片池塘。
> 每组相连的积水单元格集合可以看作是一片池塘。
>
> 每个单元格视为与其上、下、左、右、左上、右上、左下、右下八个邻近单元格相连。
>
> 请你输出共有多少片池塘，即矩阵中共有多少片相连的”W”块。
>
> [原题链接](https://www.acwing.com/problem/content/1099/)。

这里就不是从头开始 BFS 了，而是对每个点 BFS，当然前提是这个点没有被遍历过。

```cpp
#include <iostream>
#include <algorithm>
#include <queue>
using namespace std;

#define x first
#define y second
typedef pair<int, int> pii;

const int N = 1010;
int n, m, dx[] = {0, 1, 1, 1, 0, -1, -1, -1}, dy[] = {1, 1, 0, -1, -1, -1, 0, 1};
char g[N][N];
bool st[N][N];
queue<pii> q;

void bfs(int x, int y) {
    st[x][y] = true;
    q.push({x, y});
    while (q.size()) {
        pii p = q.front(); q.pop();
        for (int i = 0; i < 8; i++) {
            pii ne = {p.x+dx[i], p.y+dy[i]};
            if (0 <= ne.x && ne.x < n && 0 <= ne.y && ne.y < m && g[ne.x][ne.y] == 'W' && !st[ne.x][ne.y]) {
                q.push(ne);
                // 这里提前标记，不要在上面弹队列的时候再标记，否则复杂度飙升
                st[ne.x][ne.y] = true;
            }
        }
    }
}

int main() {
    cin >> n >> m;
    for (int i = 0; i < n; i++) cin >> g[i];
    
    int cnt = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (!st[i][j] && g[i][j] == 'W') bfs(i, j), cnt++;
        }
    }
    cout << cnt;
    return 0;
}
```

### 房间划分

> 请你编写一个程序，计算城堡一共有多少房间，最大的房间有多大。
>
> 城堡被分割成 m*n 个方格区域，每个方格区域可以有0~4面墙。
>
> 每个方块中墙的特征由数字 P 来描述，我们用1表示西墙，2表示北墙，4表示东墙，8表示南墙，P 为该方块包含墙的数字之和。例如，如果一个方块的 P 为3，则 3 = 1 + 2，该方块包含西墙和北墙。
>
> [原题链接](https://www.acwing.com/problem/content/1100/)。

现在进行 BFS 的时候关心的就是墙的存在与否了，显然给出的是一个二进制的形式，下面是各个方向向量与掩码，注意我们这里坐标系是 x 轴竖直向下，y 轴水平向左。
$$
\begin{aligned}
\vec{w}&=(0, -1),\text{mask }w=1_2\\
\vec{n}&=(-1, 0),\text{mask }n=10_2\\
\vec{e}&=(0, 1),\text{mask }e=100_2\\
\vec{s}&=(1, 0),\text{mask }s=1000_2\\
\end{aligned}
$$

```cpp
#include <iostream>
#include <algorithm>
#include <queue>
using namespace std;

typedef pair<int, int> pii;
const int N = 55;
int g[N][N], m, n, dx[] = {0, -1, 0, 1}, dy[] = {-1, 0, 1, 0}, mask[] = {1, 1<<1, 1<<2, 1<<3};
bool st[N][N];
queue<pii> q;

int bfs(int i, int j) {
    st[i][j] = true;
    int cnt = 0;
    q.push({i, j});
    while (q.size()) {
        pii p = q.front(); q.pop();
        cnt++;
        for (int i = 0; i < 4; i++) {
            int px = p.first, py = p.second;
            int x = px+dx[i], y=py+dy[i];
            // 这里如果到达边界数据保证是一定有墙的
            // 因此只需要判断当前格子的这个方向有没有墙就行
            if (!st[x][y] && (mask[i] & g[px][py]) == 0) {
                st[x][y] = true;
                q.push({x, y});
            }
        }
    }
    return cnt;
}

int main() {
    cin >> m >> n;
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            cin >> g[i][j];
    
    int area = 0, cnt = 0;
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (!st[i][j]) area = max(area, bfs(i, j)), cnt++;
    
    cout << cnt << endl << area << endl;
    
    return 0;
}
```

### 山峰和山谷

> FGD小朋友特别喜欢爬山，在爬山的时候他就在研究山峰和山谷。
>
> 为了能够对旅程有一个安排，他想知道山峰和山谷的数量。
>
> 给定一个地图，为FGD想要旅行的区域，地图被分为 n*n 的网格，每个格子 (i,j) 的高度 w(i,j) 是给定的。
>
> 若两个格子有公共顶点，那么它们就是相邻的格子，如与 (i,j)
>
> 我们定义一个格子的集合 S 为山峰（山谷）当且仅当：
>
> 1. S 的所有格子都有相同的高度。
> 2. S 的所有格子都连通。
> 3. 对于 s 属于 S，与 s 相邻的 s′ 不属于 S，都有 ws>ws'（山峰），或者 ws<ws'（山谷）。
> 4. 如果周围不存在相邻区域，则同时将其视为山峰和山谷。
>
> 你的任务是，对于给定的地图，求出山峰和山谷的数量，如果所有格子都有相同的高度，那么整个地图即是山峰，又是山谷。
>
> [原题链接](https://www.acwing.com/problem/content/1108/)。

在进行 BFS 搜索的同时，如果遍历到的点不属于当前连通块，那么判断一下它与当前点的关系。

```cpp
#include <queue>
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010;

#define x first
#define y second
typedef pair<int, int> pii;

queue<pii> q;
bool st[N][N];
int h[N][N], n;

void bfs(int ix, int iy, bool& hi, bool& lo) {
    q.push({ix, iy});
    st[ix][iy] = true;
    while (q.size()) {
        pii p = q.front(); q.pop();
        for (int i = p.x-1; i <= p.x+1; i++) {
            for (int j = p.y-1; j <= p.y+1; j++) {
                if (i < 0 || i >= n || j < 0 || j >= n)
                    continue;
                if (h[i][j] != h[p.x][p.y]) {
                    if (h[i][j] < h[p.x][p.y]) lo = true;
                    else hi = true;
                } else if (!st[i][j]) {
                    st[i][j] = true;
                    q.push({i, j});
                }
            }
        }
    }
}

int main() {
    cin >> n;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            cin >> h[i][j];
    
    int peak = 0, valley = 0;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            if (!st[i][j]) {
                bool hi = false, lo = false;
                bfs(i, j, hi, lo);
                // 没有比它高的就是山峰
                if (!hi) peak++;
                // 没有比它矮的就是山谷
                if (!lo) valley++;
            }
    
    cout << peak << ' ' << valley << endl;
    return 0;
}
```


## 最短路

### 武士风度的牛

> 一头牛有一个独一无二的超能力，在农场里像 Knight 一样地跳（就是我们熟悉的象棋中马的走法）。
>
> 虽然这头神奇的牛不能跳到树上和石头上，但是它可以在牧场上随意跳，我们把牧场用一个 x，y 的坐标图来表示。
>
> 这头神奇的牛像其它牛一样喜欢吃草，给你一张地图，上面标注了 The Knight 的开始位置，树、灌木、石头以及其它障碍的位置，除此之外还有一捆草。
>
> 现在你的任务是，确定 The Knight 要想吃到草，至少需要跳多少次。
>
> The Knight 的位置用 `K` 来标记，障碍的位置用 `*` 来标记，草的位置用 `H` 来标记。
>
> [原题链接](https://www.acwing.com/problem/content/190/)。

开一个 `dist` 数组记录长度就可以。

```cpp
#include <iostream>
#include <algorithm>
#include <queue>
using namespace std;

#define x first
#define y second
typedef pair<int, int> pii;
const int N = 1010;

pii init, target;
int n, m, dist[N][N], dx[] = {2, 2, 1, 1, -2, -2, -1, -1}, dy[] = {1, -1, 2, -2, 1, -1, 2, -2};
bool g[N][N], st[N][N];
queue<pii> q;

void bfs() {
    st[init.x][init.y] = true;
    q.push(init);
    while (q.size()) {
        pii p = q.front(); q.pop();
        if (p == target)
            break;
        for (int i = 0; i < 8; i++) {
            int x = p.x+dx[i], y = p.y+dy[i];
            // 这里 n 和 m 一定要看清楚
            if (x < 0 || x >= n || y < 0 || y >= m || st[x][y] || g[x][y])
                continue;
            st[x][y] = true;
            dist[x][y] = dist[p.x][p.y]+1;
            q.push({x, y});
        }
    }
}

int main() {
    cin >> m >> n;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++) {
            char ch; cin >> ch;
            if (ch == '*') g[i][j] = true;
            else if (ch == 'K') init = {i, j};
            else if (ch == 'H') target = {i, j};
        }
    
    bfs();
    cout << dist[target.x][target.y];
    
    return 0;
}
```

### 图中点的层次

> 给定一个 n 个点 m 条边的有向图，图中可能存在重边和自环。
>
> 所有边的长度都是 1，点的编号为 1∼n。
>
> 请你求出 1 号点到 n 号点的最短距离，如果从 1 号点无法走到 n 号点，输出 −1。
>
> [原题链接](https://www.acwing.com/problem/content/849/)。

这需要额外开一个数组记录每个点与上个点的距离。

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <bitset>
using namespace std;

const int N = 1e5+10;
int n, m;
int h[N], e[N], ne[N], idx, q[N], d[N];
bitset<N> st;

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

int bfs() {
    int tt = -1, hh = 0;
    q[++tt] = 1;
    memset(d, -1, sizeof d);
    d[1] = 0;
    while (hh <= tt) {
        int t = q[hh++];
        st[t] = 1;
        for (int i = h[t]; i != -1; i = ne[i]) {
            int j = e[i];
            if (!st[j] && d[j] == -1)  {
                d[j] = d[t] + 1;
                q[++tt] = j;
            }
        }
    }
    return d[n];
}

int main() {
    // 不要忘记初始化树!!
    memset(h, -1, sizeof h);
    cin >> n >> m;
    int a, b;
    for (int i = 0; i < m; i++) {
        cin >> a >> b;
        add(a, b);
    }
    cout << bfs() << endl;
    return 0;
}
```


### 走迷宫

> 给定一个 n×m 的二维整数数组，用来表示一个迷宫，数组中只包含 0 或 1，其中 0 表示可以走的路，1 表示不可通过的墙壁。
>
> 最初，有一个人位于左上角 (1,1) 处，已知该人每次可以向上、下、左、右任意一个方向移动一个位置。
>
> 请问，该人从左上角移动至右下角 (n,m) 处，至少需要移动多少次。
>
> 数据保证 (1,1) 处和 (n,m) 处的数字为 0，且一定至少存在一条通路。
>
> [原题链接](https://www.acwing.com/problem/content/846/)。

需要额外记录的是到达每个点经过路径的距离，这里用 `d` 表示：

```cpp
#include <iostream>
#include <cstring>
using namespace std;

typedef pair<int, int> pii;
const int N = 110;
int g[N][N], d[N][N];
int n, m;
pii q[N*N];

int bfs() {
    int hh = 0, tt = -1;
    q[++tt] = {0, 0};
    d[0][0] = 0;
    // 四个方向向量
    int dx[] = {-1, 0, 1, 0}, dy[] = {0, 1, 0, -1};
    while (hh <= tt) {
        pii t = q[hh++];
        for (int i = 0; i < 4; i++) {
            int x = t.first + dx[i], y = t.second + dy[i];
            // 确保 x, y 在边界内并且没被访问过而且可以通行
            if (x >= 0 && x < n && y >= 0 && y < m && d[x][y] == -1 && g[x][y] == 0) {
                d[x][y] = d[t.first][t.second] + 1;
                q[++tt] = {x, y};
            }
        }
    }
    
    return d[n-1][m-1];
}

int main() {
    cin >> n >> m;
    for (int i = 0; i < n; i++) for (int j = 0; j < m; j++) cin >> g[i][j];
    // 提前把距离全部初始化为 -1 
    memset(d, -1, sizeof d);
    cout << bfs() << endl;
    return 0;
}
```

### 矩阵距离

> 给定一个 N 行 M 列的 0101 矩阵 A，$A[i,j]$ 与 $A[k,l]$ 之间的曼哈顿距离定义为：
> $$
> \text{dist}(A[i,j],A[k,l])=|i-k|+|j-l|
> $$
> 输出一个 N 行 M 列的整数矩阵 B，其中：
> $$
> B[i,j]=\text{min}_{x\in[1,n],y\in[1,m],A[x,y]=1} \text{dist}(A[i,j],A[x,y])
> $$
> [原题链接](https://www.acwing.com/problem/content/description/175/)。

也就是求出所有点到矩阵中值为 1 的点的曼哈顿距离，这是一个多源最短路问题，可以设出一个虚拟原点，这个虚拟原点到所有点为 1 的距离为 0，然后求出所有点到这个虚拟原点的最短距离。

```cpp
#include <iostream>
#include <algorithm>
#include <queue>
#include <cstring>
using namespace std;

#define x first
#define y second
typedef pair<int, int> pii;
const int N = 1010;

int n, m, dist[N][N], dx[] = {-1, 0, 1, 0}, dy[] = {0, 1, 0, -1};
bool g[N][N];
queue<pii> q;

void bfs() {
    while (q.size()) {
        pii p = q.front(); q.pop();
        for (int i = 0; i < 4; i++) {
            int x = p.x+dx[i], y = p.y+dy[i];
            if (x < 0 || x >= n || y < 0 || y >= m || g[x][y] || dist[x][y] != -1)
                continue;
            dist[x][y] = dist[p.x][p.y]+1;
            q.push({x, y});
        }
    }
}

int main() {
    cin >> n >> m;
    memset(dist, -1, sizeof dist);
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            char ch; cin >> ch;
            g[i][j] = ch == '1';
            if (g[i][j]) {
                dist[i][j] = 0;
                q.push({i, j});
            }
        }
    }
    
    bfs();
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cout << dist[i][j] << ' ';
        }
        cout << '\n';
    }
    return 0;
}
```

### 魔板

> Rubik 先生在发明了风靡全球的魔方之后，又发明了它的二维版本——魔板。
>
> 这是一张有 8 个大小相同的格子的魔板：
>
> ```
> 1 2 3 4
> 8 7 6 5
> ```
>
> 我们知道魔板的每一个方格都有一种颜色，这 8 种颜色用前 8 个正整数来表示。
>
> 可以用颜色的序列来表示一种魔板状态，规定从魔板的左上角开始，沿顺时针方向依次取出整数，构成一个颜色序列。
>
> 对于上图的魔板状态，我们用序列 (1,2,3,4,5,6,7,8) 来表示，这是基本状态。
>
> 这里提供三种基本操作，分别用大写字母 A，B，C 来表示（可以通过这些操作改变魔板的状态）：
>
> A：交换上下两行；
>
> B：将最右边的一列插入到最左边；
>
> C：魔板中央对的4个数作顺时针旋转。
>
> 对于每种可能的状态，这三种基本操作都可以使用。
>
> 你要编程计算用最少的基本操作完成基本状态到特殊状态的转换，输出字典序最小的基本操作序列。
>
> **注意**：数据保证一定有解。
>
> [原题链接](https://www.acwing.com/problem/content/1109/)。

本题类似八数码，只不过这里的转换有点恶心人。

```cpp
#include <iostream>
#include <string>
#include <queue>
#include <unordered_map>
using namespace std;

string init = "12348765", target;
queue<string> q;
// 记录方案的同时记录转移类型
unordered_map<string, pair<char, string>> pre;

void bfs() {
    q.push(init);
    while (q.size()) {
        string p = q.front(); q.pop();
        
        if (p == target) break;
        
        string A = p;
        for (int i = 0; i <= 3; i++) swap(A[i], A[i+4]);
        if (pre.count(A) == 0) {
            pre[A] = {'A', p};
            q.push(A);
        }
        
        string B = p;
        for (int i = 1; i <= 3; i++) B[i] = p[i-1], B[i+4] = p[i+3];
        B[0] = p[3], B[4] = p[7];
        if (pre.count(B) == 0) {
            pre[B] = {'B', p};
            q.push(B);
        }
        
        string C = p;
        C[1] = p[5], C[2] = p[1], C[5] = p[6], C[6] = p[2];
        if (pre.count(C) == 0) {
            pre[C] = {'C', p};
            q.push(C);
        }
    }
}

int main() {
    for (int i = 0; i < 8; i++) {
        char ch; cin >> ch;
        target += ch;
    }
    swap(target[4], target[7]), swap(target[5], target[6]);
    bfs();
    vector<char> ans;
    string t = target;
    while (t != init) {
        auto& p = pre[t];
        ans.push_back(p.first);
        t = p.second;
    }
    cout << ans.size() << endl;
    while (ans.size()) {
        cout << ans.back();
        ans.pop_back();
    }
    return 0;
}
```

### 电路维修

> 电路板的整体结构是一个 R 行 C 列的网格（R,C≤500），如下所示：
>
> ![](/img/2023/07/acw175.png)
>
> 每个格点都是电线的接点，每个格子都包含一个电子元件。
>
> 电子元件的主要部分是一个可旋转的、连接一条对角线上的两个接点的短电缆。
>
> 在旋转之后，它就可以连接另一条对角线的两个接点。
>
> **注意**：只能走斜向的线段，水平和竖直线段不能走。
>
> 输入输出等格式参见[原题链接](https://www.acwing.com/problem/content/177/)。

#### Dijkstra

本题仍为最短路问题，不过这里边权可以为 0 或 1，如果能不更改线路直接走过去就是 0，需要更改就是 1，于是这里用堆优化 Dijkstra 算法来求解这个问题。而这里由于边权只有 1 和 0，并不需要真的用到堆，用一个双端队列手动维护就可以。

回忆 Dijkstra 算法的流程：

1. 将所有距离初始化成正无穷。
2. 取出**距离原点最近**且**未访问**的节点，用它更新它的子节点到原点的距离。如果之前已经被访问过，那么之前更新的距离势必比现在更短或者和现在一样。
3. 如果这个子节点的距离被减小，那么将子节点加入队列中。
4. 重复上述操作。

#### 方向向量

接下来考虑方向向量的定义，注意我们的 x 轴是从原点竖直向下，y 轴从原点水平向右。

![](/img/2023/07/acw175-vector.jpg)

对于任意点 $P(x,y)$，它对应的方格是它右下角的方格，设点的方向向量为 $\vec{p}$，格子的方向向量是 $\vec{b}$，那么从右上顺时针转动可以写出这四个方向向量：
$$
\begin{aligned}
&\vec{p_1} = (-1,1),\vec{b_1}=(-1,0)\\
&\vec{p_2}=(1,1),\vec{b_2}=(0,0)\\
&\vec{p_3}=(1,-1),\vec{b_3}=(0,-1)\\
&\vec{p_4}=(-1,-1),\vec{b_4}=(-1,-1)
\end{aligned}
$$
对每个方向的 0 费用状态也可以轻松得到，这里就不再展开说了。

```cpp
#include <iostream>
#include <algorithm>
#include <queue>
#include <cstring>
using namespace std;

#define x first
#define y second
typedef pair<int, int> pii;
const int N = 510;

char g[N][N];
int dx[] = {-1, 1, 1, -1}, dy[] = {1, 1, -1, -1};
int bx[] = {-1, 0, 0, -1}, by[] = {0, 0, -1, -1};
// zero cost
char zc[] = {'/', '\\', '/', '\\'};
bool st[N][N];
int dist[N][N], n, m;

int dj() {
    // 执行完本函数 q 不一定是非空的，因此这里就把 q 申请为局部变量
    deque<pii> q;
    q.push_back({0, 0});
    memset(dist, 0x3f, sizeof dist);
    memset(st, 0, sizeof st);
    dist[0][0] = 0;
    while (q.size()) {
        pii p = q.front(); q.pop_front();
        if (p.x == n && p.y == m) return dist[n][m];
        
        if (st[p.x][p.y]) continue;
        st[p.x][p.y] = true;

        for (int i = 0; i < 4; i++) {
            int x = p.x+dx[i], y = p.y+dy[i], w = 1;
            // 注意 m 和 n。。。
            if (x < 0 || x > n || y < 0 || y > m) continue;
            if (zc[i] == g[p.x+bx[i]][p.y+by[i]]) w = 0;
            int d = dist[p.x][p.y]+w;
            // 这里大于和大于等于都可以
            if (dist[x][y] > d) {
                dist[x][y] = d;
                if (w == 0) q.push_front({x, y});
                else q.push_back({x, y});
            }
        }
    }
    return -1;
}

int main() {
    int T; cin >> T;
    while (T--) {
        // 格子坐标 0 ~ n-1, 0 ~ m-1
        // 点坐标  0 ~ n, 0 ~ m
        cin >> n >> m;
        for (int i = 0; i < n; i++) cin >> g[i];
        int res = dj();
        if (res == -1) cout << "NO SOLUTION\n";
        else cout << res << '\n';
    }
    
    return 0;
}
```

### 正方形泳池

> 给定一个 N×N 的方格矩阵。
>
> 左上角方格坐标为 (1,1)，右下角方格坐标为 (N,N)。
>
> 有 T 个方格内有树，这些方格的具体坐标已知。
>
> 我们希望建立一个正方形的泳池。
>
> 你的任务是找到一个尽可能大的**正方形**子矩阵，要求子矩阵内没有包含树的方格。
>
> 输出满足条件的子矩阵的最大可能边长。
>
> 题目链接：[AcWing 5180](https://www.acwing.com/problem/content/5183/)。

本题只是一个枚举的题，我不知道放哪里合适，暂时放这里。

数据范围的特点是 $T$ 很小，$N$ 很大，所以我们枚举每个树的坐标，这里是一个贪心的分析，一个最大的正方形的某条边或者顶点上一定连着一个树，如果四条边或者四个顶点都能继续向外扩张，那么它必定不是最大的正方形。

问题又来了，怎么处理边界？可以考虑在地图外面的四个顶点放四棵树，这样就能保证整个程序逻辑正常的情况下也可以正确的处理边界了。

我们暴力枚举两颗横坐标不同的树，设这个正方形与左边这颗树 $p$ 相连，那么它的【上边界的上一行 `up`】就必须满足 $up\ge y(p)$，【下边界的下一行 `down`】就必须满足 $down\le y(p)$，我们计算的时候取 `up-down-1` 为高度。

对于纵坐标，我们将所有点沿着 $y=x$ 对称然后重新做一遍即可。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

#define x first
#define y second
typedef pair<int, int> PII;
const int M = 110;
int n, m;
PII p[M];

int solve() {
    sort(p, p+m);
    int res = 0;
    for (int i = 0; i < m; i++) {
        int up = n+1, down = 0;
        for (int j = i+1; j < m; j++) {
            if (p[j].x == p[i].x) continue;
            
            // 这里也可以优化一条
            // if (p[j].x - p[i].x > up - down) break;
            // 因为我们这里是关心横坐标对边长的限制 纵坐标应该是对称之后该关心的
            // 但这只能算是常数优化, 不加也无所谓
            
            res = max(res, min(p[j].x - p[i].x - 1, up - down - 1));
            if (p[j].y >= p[i].y) up = min(up, p[j].y);
            if (p[j].y <= p[i].y) down = max(down, p[j].y);
        }
    }
    return res;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 0; i < m; i++) scanf("%d%d", &p[i].x, &p[i].y);
    p[m++] = {0, 0};
    p[m++] = {0, n+1};
    p[m++] = {n+1, 0};
    p[m++] = {n+1, n+1};
    int res = solve();
    for (int i = 0; i < m; i++) swap(p[i].x, p[i].y);
    printf("%d\n", max(res, solve()));
    return 0;
}

```

### 八数码

> 在一个 3x3 的网格中，1∼8 这 8 个数字和一个 `x` 恰好不重不漏地分布在这 3x3 的网格中。
>
> 例如：
>
> ```
> 1 2 3
> x 4 6
> 7 5 8
> ```
>
> 在游戏过程中，可以把 `x` 与其上、下、左、右四个方向之一的数字交换（如果存在）。
>
> 我们的目的是通过交换，使得网格变为如下排列（称为正确排列）：
>
> ```
> 1 2 3
> 4 5 6
> 7 8 x
> ```
>
> 例如，示例中图形就可以通过让 `x` 先后与右、下、右三个方向的数字交换成功得到正确排列。
>
> 交换过程如下：
>
> ```
> 1 2 3   1 2 3   1 2 3   1 2 3
> x 4 6   4 x 6   4 5 6   4 5 6
> 7 5 8   7 5 8   7 x 8   7 8 x
> ```
>
> 现在，给你一个初始网格，请你求出得到正确排列至少需要进行多少次交换。
>
> [原题链接](https://www.acwing.com/problem/content/847/)。

本题最大难点在于 **状态的存储**。由于这个序列无法方便地用数字表示，所以考虑用字符串来表示一种序列。

```cpp
#include <iostream>
#include <string>
#include <queue>
#include <unordered_map>
using namespace std;

string init, target = "12345678x";
unordered_map<string, int> d;
queue<string> q;

int bfs() {
    q.push(init);
    d[init] = 0;
    while (q.size()) {
        string t = q.front(); q.pop();
        int dist = d[t];
        if (t == target) return dist;
        
        // string::find 返回下标
        int pos = t.find('x');
        int x = pos / 3, y = pos % 3;
        int dx[] = {-1, 0, 1, 0}, dy[] = {0, 1, 0, -1};
        for (int i = 0; i < 4; i++) {
            int nx = x+dx[i], ny = y+dy[i];
            
            if (nx >= 0 && nx < 3 && ny >= 0 && ny < 3) {
                swap(t[nx*3+ny], t[pos]);
                if (d.count(t) == 0) {
                    d[t] = dist+1;
                    // 将 t 这个状态添加到队列中继续向下寻找
                    q.push(t);
                }
                // 不要忘记复原这个字符串
                swap(t[nx*3+ny], t[pos]);
            }
        }
    }
    return -1;
}

int main() {
    char c;
    for (int i = 0; i < 9; i++) {
        cin >> c;
        init += c;
    }
    cout << bfs() << endl;
    return 0;
}
```

### 字串变换

> 已知有两个字串 A, B 及一组字串变换的规则（至多 6 个规则）:
>
> A1 -> B1
>
> A2 -> B2
>
> …
>
> 规则的含义为：在 A 中的子串 A1 可以变换为 B1、A2 可以变换为 B2...。
>
> 例如：A＝`abcd` B＝`xyz`
>
> 变换规则为：
>
> `abc` -> `xu` `ud` -> `y` `y` -> `yz`
>
> 则此时，A 可以经过一系列的变换变为 B，其变换的过程为：
>
> `abcd` -> `xud` -> `xy` -> `xyz`
>
> 共进行了三次变换，使得 A 变换为 B。
>
> 注意，一次变换只能变换一个子串，例如 A＝`aa` B＝`bb`
>
> 变换规则为：
>
> `a` -> `b`
>
> 此时，不能将两个 `a` 在一步中全部转换为 `b`，而应当分两步完成。
>
> [原题链接](https://www.acwing.com/problem/content/192/)。

本题要用到双向广搜，如果单向进行搜索的话复杂度太高：假设每个规则只能有一个位置进行变换，那么至多有 $6^n$ 种方案，这显然会导致 MLE 或 TLE，因此这里使用双向广搜的方式优化，即从起点到终点的搜索同时也从终点到起点搜索，相遇时就找到了答案。

```cpp
#include <iostream>
#include <algorithm>
#include <unordered_map>
#include <string>
#include <queue>
using namespace std;

typedef unordered_map<string, int> msi;
const int N = 6;

string A, B, a[N], b[N];
queue<string> qa, qb;
msi da, db;
int n;

int extend(queue<string>& q, msi& da, msi& db, string a[], string b[]) {
    string p = q.front(); q.pop();
    // 循环规则 -> 循环位置
    for (int r = 0; r < n; r++) {
        for (int i = 0; i < p.size(); i++) {
            if (p.substr(i, a[r].size()) == a[r]) {
                string state = p.substr(0, i) + b[r] + p.substr(i+a[r].size());
                if (da.count(state))
                    continue;
                
                da[state] = da[p] + 1;
                if (db.count(state))
                    return db[state] + da[state];
                
                q.push(state);
             }
        }
    }
    
    return -1;
}

int bfs() {
    if (A == B) return 0;

    qa.push(A), qb.push(B);
    da[A] = db[B] = 0;
    while (qa.size() && qb.size()) {
        int step;
        // 状态少的优先
        if (qa.size() <= qb.size()) step = extend(qa, da, db, a, b);
        else step = extend(qb, db, da, b, a);

        if (step != -1) return step;
    }
    return -1;
}

int main() {
    cin >> A >> B;
    while (cin >> a[n] >> b[n]) n++;
    int step = bfs();
    if (step == -1) cout << "NO ANSWER!";
    else cout << step;

    return 0;
}
```

> 
