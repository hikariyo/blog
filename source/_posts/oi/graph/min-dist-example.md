---
date: 2023-07-18 22:04:00
updated: 2023-12-29 02:13:00
title: 图论 - 最短路例题
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## 单源最短路

### 道路与航线

> 农夫约翰正在一个新的销售区域对他的牛奶销售方案进行调查。
>
> 他想把牛奶送到 T 个城镇，编号为 1∼T。
>
> 这些城镇之间通过 R 条道路 (编号为 1 到 R) 和 P 条航线 (编号为 1 到 P) 连接。
>
> 每条道路 i 或者航线 i 连接城镇 Ai 到 Bi，花费为 Ci。
>
> 对于道路，花费 Ci 为正数；然而航线的花费很神奇，花费 Ci 可能是负数。
>
> 道路是双向的，可以从 Ai 到 Bi，也可以从 Bi 到 Ai，花费都是 Ci。
>
> 然而航线与之不同，只可以从 Ai 到 Bi。
>
> 数据保证如果有一条航线可以从 Ai 到 Bi，那么保证不可能通过一些道路和航线从 Bi 回到 Ai。
>
> 他需要运送奶牛到每一个城镇，想找到从发送中心城镇 S 把奶牛送到每个城镇的最便宜的方案。
>
> 原题链接：[AcWing 342](https://www.acwing.com/problem/content/344/)。

#### 算法1 Topsort(DAG) + 堆优 Dijkstra

本题 SPFA 可以直接无脑做，但是会被卡，然后研究一下题目本身的性质。

对于样例数据，我画了张图。

![](/img/2023/07/acw342.jpg)

航班都是单向，道路是双向，那么由道路连接起来的这些连通块可以看作一个抽象的点，内部用 Dijkstra 求最短路，然后对这些抽象出来的点用拓扑序求一个最小值就行了。

```cpp
#include <iostream>
#include <cstring>
#include <queue>
using namespace std;

typedef pair<int, int> pii;
const int N = 25010, M = 50010*3, INF = 0x3f3f3f3f;
int h[N], e[M], ne[M], w[M], idx;
int n, m1, m2, S, dist[N];
int id[N], din[N], bcnt = 1;
bool st[N];
vector<int> block[N];
queue<int> q;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

// Flood-fill
bool dfs(int u, int bid) {
    if (id[u]) return false;
    id[u] = bid;
    block[bid].push_back(u);
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (!id[j]) dfs(j, bid);
    }
    return true;
}

void dijkstra(int bid) {
    priority_queue<pii, vector<pii>, greater<pii>> heap;
    for (int t : block[bid]) {
        heap.push({dist[t], t});
    }
    while (heap.size()) {
        pii p = heap.top(); heap.pop();
        int t = p.second;
        if (st[t]) continue;
        st[t] = true;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                if (id[j] == id[t]) heap.push({dist[j], j});
            }
            
            if (id[j] != id[t] && -- din[id[j]] == 0) q.push(id[j]);
        }
    }
}

void topsort() {
    for (int i = 1; i < bcnt; i++) if (din[i] == 0) q.push(i);
    memset(dist, 0x3f, sizeof dist);
    dist[S] = 0;
    while (q.size()) {
        int t = q.front(); q.pop();
        dijkstra(t);
    }
}

int main() {
    cin >> n >> m1 >> m2 >> S;
    memset(h, -1, sizeof h);
    while (m1--) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c), add(b, a, c);
    }
    for (int i = 1; i <= n; i++) if (dfs(i, bcnt)) bcnt++;
    while (m2--) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c);
        din[id[b]]++;
    }
    
    topsort();
    for (int i = 1; i <= n; i++) {
        if (dist[i] > INF / 2) cout << "NO PATH\n";
        else cout << dist[i] << '\n';
    }
    return 0;
}
```

#### 算法2 SPFA(TLE)

对这种有向无环图（DAG）可以直接用拓扑序求解最小值，也可以用 SPFA 求连通块之间的最短路，但是这会 TLE。

```cpp
void dijkstra(int bid) {
    // 这句没加坑了我5个多小时
    memset(st, 0, sizeof st);
    priority_queue<pii, vector<pii>, greater<pii>> heap;
    for (int p : block[bid]) {
        heap.push({dist[p], p});
    }

    while (heap.size()) {
        pii p = heap.top(); heap.pop();
        int t = p.second;
        if (st[t]) continue;
        st[t] = true;

        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                if (id[j] == id[t]) heap.push({dist[j], j});
                // SPFA
                else if (!blockst[id[j]]) {
                    q.push(id[j]);
                    blockst[id[j]] = true;
                }
            }
        }
    }
}

void spfa() {
    memset(dist, 0x3f, sizeof dist);
    q.push(id[S]);
    dist[S] = 0;
    while (q.size()) {
        int t = q.front(); q.pop();
        blockst[t] = false;
        dijkstra(t);
    }
}
```

所以对于 DAG 还是直接用拓扑序线性复杂度就可以求最短距离。

#### 算法3 SPFA 最简(TLE)

其实根本不用算法2 里面那么搞，直接建完图跑 SPFA 就可以过绝大部分测试点了，这里不再演示。

### 通信线路

> 在郊区有 N 座通信基站，P 条 **双向** 电缆，第 i 条电缆连接基站 Ai 和 Bi。
>
> 特别地，1 号基站是通信公司的总站，N 号基站位于一座农场中。
>
> 现在，农场主希望对通信线路进行升级，其中升级第 i 条电缆需要花费 Li。
>
> 电话公司正在举行优惠活动。
>
> 农产主可以指定一条从 1 号基站到 N 号基站的路径，并指定路径上不超过 K 条电缆，由电话公司免费提供升级服务。
>
> 农场主只需要支付在该路径上剩余的电缆中，升级价格最贵的那条电缆的花费即可。
>
> 求至少用多少钱可以完成升级。
>
> 原题链接：[AcWing 342](https://www.acwing.com/problem/content/342/)。

本题求所有路径中第 $k+1$ 大的边长的最小值。

对于某个边长数值 $x$，如果它不是答案，所有路径中都一定有超过 $k$ 条边比它大，即 $p: \forall \text{path}, \text{count}(w(\text{edge})>x)>k$；如果它是答案，就是 $\neg p:\exist \text{path}, \text{count}(\text{edge} > x) \le k$

对于这个任意性和存在性，只需比较最小值与 $k$ 的关系，这个最小值就用 Dijkstra 求解，边长大于 $x$ 的边权为 1 否则为 0，这样用一个双端队列就可以模拟单调队列了。

这里二分查找答案的时候需要枚举 $[0, 10^6]$ 区间中的数字，因为 0 也是一个合法解，它代表存在一条路径的长度是 $\text{len} \le k$ 的。

```cpp
#include <iostream>
#include <queue>
#include <cstring>
using namespace std;

const int N = 1010, M = 20010;
int h[N], e[M], ne[M], w[M], idx;
int n, m, k, dist[N];
bool st[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

bool check(int x, bool& unreachable) {
    deque<int> q;
    q.push_back(1);
    memset(dist, 0x3f, sizeof dist);
    memset(st, 0, sizeof st);
    dist[1] = 0;
    while (q.size()) {
        int t = q.front(); q.pop_front();
        if (st[t]) continue;
        st[t] = true;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i], v = w[i] > x;
            if (dist[j] > dist[t] + v) {
                dist[j] = dist[t] + v;
                if (v) q.push_back(j);
                else q.push_front(j);
            }
        }
    }
    if (dist[n] == 0x3f3f3f3f) unreachable = true;
    return dist[n] <= k;
}

int main() {
    cin >> n >> m >> k;
    memset(h, -1, sizeof h);
    while (m--) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c), add(b, a, c);
    }
    int l = 0, r = 1e6;
    bool unreachable = false;
    while (l < r && !unreachable) {
        int mid = l+r >> 1;
        if (check(mid, unreachable)) r = mid;
        else l = mid+1;
    }
    if (unreachable) r = -1;
    cout << r << endl;
    return 0;
}
```

### 昂贵的聘礼

> 年轻的探险家来到了一个印第安部落里。
>
> 在那里他和酋长的女儿相爱了，于是便向酋长去求亲。
>
> 酋长要他用 10000 个金币作为聘礼才答应把女儿嫁给他。
>
> 探险家拿不出这么多金币，便请求酋长降低要求。
>
> 酋长说：”嗯，如果你能够替我弄到大祭司的皮袄，我可以只要 8000 金币。如果你能够弄来他的水晶球，那么只要 5000 金币就行了。”
>
> 探险家就跑到大祭司那里，向他要求皮袄或水晶球，大祭司要他用金币来换，或者替他弄来其他的东西，他可以降低价格。
>
> 探险家于是又跑到其他地方，其他人也提出了类似的要求，或者直接用金币换，或者找到其他东西就可以降低价格。
>
> 不过探险家没必要用多样东西去换一样东西，因为不会得到更低的价格。
>
> 探险家现在很需要你的帮忙，让他用最少的金币娶到自己的心上人。
>
> 另外他要告诉你的是，在这个部落里，等级观念十分森严。
>
> 地位差距超过一定限制的两个人之间不会进行任何形式的直接接触，包括交易。
>
> 他是一个外来人，所以可以不受这些限制。
>
> 但是如果他和某个地位较低的人进行了交易，地位较高的的人不会再和他交易，他们认为这样等于是间接接触，反过来也一样。
>
> 因此你需要在考虑所有的情况以后给他提供一个最好的方案。
>
> 为了方便起见，我们把所有的物品从 1 开始进行编号，酋长的允诺也看作一个物品，并且编号总是 1。
>
> 每个物品都有对应的价格 P，主人的地位等级 L，以及一系列的替代品 Ti 和该替代品所对应的”优惠” Vi。
>
> 如果两人地位等级差距超过了 M，就不能”间接交易”。
>
> 你必须根据这些数据来计算出探险家最少需要多少金币才能娶到酋长的女儿。
>
> 原题链接：[AcWing 903](https://www.acwing.com/problem/content/905/)。

#### 算法1 SPFA 求最大减少价格

我自己的做法是用边长存储减少的价格。

假设某物品价格为 $p_0=2000$，替代品价格为 $p_1=500$，替代价格为 $p_2=1000$，那么如果走替代品这条路价格就变成了 $p_2+p_1=1500$，那减少的价格就是 $p_0-p_1-p_2$，边长存的就是这个数据。

一开始我存的就是一般为正数的 $p_0-p_1-p_2$，然后用 Dijkstra 来求最大值，但是有些数据过不去，原因就是存在负权边，即有些替代品买了反而会让价格更贵，于是就改用 SPFA 然后把边长取相反数求最小值，最后找到最多能减多少价格。

对酋长所在等级区间范围做多遍 SPFA 求每次价格综合起来取最小值。

要做 $m$ 遍 SPFA, 一般 $O(mn)$，而边数能达到 $n^2$，最坏 $O(mn^3)$  达到 1e8，就算有意卡的话也勉强能过。

```cpp
#include <iostream>
#include <algorithm>
#include <queue>
#include <cstring>
using namespace std;

const int N = 105, M = N*N;
int m, n, p[N], l[N], x[N];
struct Obj { int idx, p; } obj[N][N];
int h[N], e[M], ne[M], w[M], idx;
int dist[N];
bool st[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

// 有负权 用 SPFA
int spfa(int lo, int hi) {
    memset(dist, 0x3f, sizeof dist);
    memset(st, 0, sizeof st);
    dist[1] = 0;
    queue<int> q;
    q.push(1);
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i] && l[j] <= hi && l[j] >= lo) {
                dist[j] = dist[t] + w[i];
                if (!st[j]) {
                    q.push(j);
                    st[j] = true;
                }
            }
        }
    }
    return *min_element(dist+1, dist+n+1);
}

int main() {
    cin >> m >> n;
    memset(h, -1, sizeof h);
    for (int i = 1; i <= n; i++) {
        cin >> p[i] >> l[i] >> x[i];
        for (int j = 0; j < x[i]; j++) cin >> obj[i][j].idx >> obj[i][j].p;
    }
    
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < x[i]; j++) {
            add(i, obj[i][j].idx, -(p[i] - obj[i][j].p - p[obj[i][j].idx]));
        }
    }
    int ans = 1e9;
    for (int lo = l[1]-m; lo <= l[1]; lo++) {
        ans = min(ans, p[1] + spfa(lo, lo+m));
    }
    cout << ans << endl;
    return 0;
}
```

#### 算法2 堆优 Dijkstra 反向建图求最小价格

还可以反向建图，每条边的含义是买下这个物品还需要加多少钱买下替代品是当前物品的那个物品，这么做需要建一个虚拟原点，链接所有物品，边权就是物品的价格，然后用 Dijkstra 求虚拟原点到终点的最短路即可，如下：

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <queue>
using namespace std;

typedef pair<int, int> pii;
const int N = 110, M = N*N;
int p[N], l[N], x[N], m, n;
struct Obj {
    int idx, p;
} obj[N][N];
int h[N], e[M], ne[M], w[M], dist[N], idx;
bool st[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

int dijkstra(int lo, int hi) {
    memset(st, 0, sizeof st);
    memset(dist, 0x3f, sizeof dist);
    dist[0] = 0;
    priority_queue<pii, vector<pii>, greater<pii>> heap;
    heap.push({0, 0});
    while (heap.size()) {
        pii p = heap.top(); heap.pop();
        int t = p.second;
        if (st[t]) continue;
        st[t] = true;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i] && l[j] >= lo && l[j] <= hi) {
                dist[j] = dist[t] + w[i];
                heap.push({dist[j], j});
            }
        }
    }
    
    return dist[1];
}

int main() {
    cin >> m >> n;
    
    memset(h, -1, sizeof h);
    for (int i = 1; i <= n; i++) {
        cin >> p[i] >> l[i] >> x[i];
        for (int j = 0; j < x[i]; j++) {
            cin >> obj[i][j].idx >> obj[i][j].p;
        }
    }
    
    for (int i = 1; i <= n; i++) {
        add(0, i, p[i]);
        for (int j = 0; j < x[i]; j++) {
            add(obj[i][j].idx, i, obj[i][j].p);
        }
    }
    
    int ans = 2e9;
    for (int lo = l[1] - m; lo <= l[1]; lo++) {
        ans = min(ans, dijkstra(lo, lo+m));
    }
    cout << ans << endl;
    return 0;
}
```

### [CQOI2005] 新年好

> 重庆城里有 n 个车站，m 条 **双向** 公路连接其中的某些车站。
>
> 每两个车站最多用一条公路连接，从任何一个车站出发都可以经过一条或者多条公路到达其他车站，但不同的路径需要花费的时间可能不同。
>
> 在一条路径上花费的时间等于路径上所有公路需要的时间之和。
>
> 佳佳的家在车站 1，他有五个亲戚，分别住在车站 a, b, c, d, e。
>
> 过年了，他需要从自己的家出发，拜访每个亲戚（顺序任意），给他们送去节日的祝福。
>
> 怎样走，才需要最少的时间？
>
> 原题链接：[AcWing 1135](https://www.acwing.com/problem/content/1137/)。

对车站 1 和 a, b, c, d, e 用堆优化 Dijkstra 分别求一遍单源最短路，然后暴搜所有方案，求一个最小值。

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <queue>
using namespace std;

typedef pair<int, int> pii;
const int N = 5e4+10, M = 2e5+10;
int h[N], e[M], ne[M], w[M], idx, n, m;
int dist[6][N], source[6], ans = 2e9;
bool st[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void dijkstra(int dist[], int S) {
    memset(dist, 0x3f, sizeof(int) * N);
    dist[S] = 0;
    priority_queue<pii, vector<pii>, greater<pii>> heap;
    heap.push({0, S});
    while (heap.size()) {
        pii p = heap.top(); heap.pop();
        int t = p.second;
        if (st[t]) continue;
        st[t] = true;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                heap.push({dist[j], j});
            }
        }
    }
    memset(st, 0, sizeof st);
}

void dfs(int u, int s, int cnt) {
    if (cnt >= 5) {
        ans = min(ans, s);
        return;
    }
    
    for (int i = 1; i <= 5; i++) {
        if (!st[i]) {
            st[i] = true;
            dfs(i, s+dist[u][source[i]], cnt+1);
            st[i] = false;
        }
    }
}

int main() {
    cin >> n >> m;
    memset(h, -1, sizeof h);
    source[0] = 1;
    for (int i = 1; i <= 5; i++) cin >> source[i];
    while (m--) {
        int a, b, c; cin >> a >> b >> c;
        add(a, b, c), add(b, a, c);
    }
    for (int i = 0; i <= 5; i++) dijkstra(dist[i], source[i]);
    dfs(0, 0, 0);
    cout << ans << endl;
    return 0;
}
```

### 拯救大兵瑞恩(状压)

> 迷宫的外形是一个长方形，其南北方向被划分为 N 行，东西方向被划分为 M 列， 于是整个迷宫被划分为 N×M 个单元。
>
> 每一个单元的位置可用一个有序数对 (单元的行号, 单元的列号) 来表示。
>
> 南北或东西方向相邻的 2 个单元之间可能互通，也可能有一扇锁着的门，或者是一堵不可逾越的墙。
>
> **注意：** 门可以从两个方向穿过，即可以看成一条无向边。
>
> 迷宫中有一些单元存放着钥匙，同一个单元可能存放 **多把钥匙**，并且所有的门被分成 P 类，打开同一类的门的钥匙相同，不同类门的钥匙不同。
>
> 大兵瑞恩被关押在迷宫的东南角，即 (N,M) 单元里，并已经昏迷。
>
> 迷宫只有一个入口，在西北角。
>
> 也就是说，麦克可以直接进入 (1,1) 单元。
>
> 另外，麦克从一个单元移动到另一个相邻单元的时间为 1，拿取所在单元的钥匙的时间以及用钥匙开门的时间可忽略不计。
>
> 试设计一个算法，帮助麦克以最快的方式到达瑞恩所在单元，营救大兵瑞恩。
>
> 原题链接：[AcWing 1131](https://www.acwing.com/problem/content/1133/)。

可以从分层图的角度来思考，拿了对应的钥匙后图中的对应门就有一条边，否则是没有边的。由于钥匙最多有 10 个，所以如果真的去建图要建 $2^{10}$ 层，显然不太现实，所以策略就是用 状压DP 的方式去处理每把钥匙。
$$
\left\{ \begin{aligned}
&\text{dist}(x+\Delta x,y+\Delta y,s)=\text{dist}(x,y,s)+1\\
&\text{dist}(x,y,s \leftarrow k_{x,y})=\text{dist}(x,y,s)\\
\end{aligned}\right .
$$
这里最短路仍然用堆优 Dijkstra 做，但由于边权只有 0 和 1 所以可以用双端队列模拟堆；对于一个点可以把它看成一个 11 进制数字的两位，题目限制是 1 ~ 10 所以用 11 进制。

```cpp
#include <iostream>
#include <deque>
#include <cstring>
using namespace std;

#define pos first
#define stat second
#define pp(a, b) a*11+b
typedef pair<int, int> pii;
const int N = 115, M = 1 << 11;
int n, m, p, dist[M][N], g[N][N], key[N];
bool st[M][N];
int dx[] = {0, 1, 0, -1}, dy[] = {1, 0, -1, 0};

int dijkstra() {
    deque<pii> q;
    int src = pp(1, 1);
    q.push_back({src, key[src]});
    memset(dist, 0x3f, sizeof dist);
    dist[key[src]][src] = 0;
    while (q.size()) {
        pii t = q.front(); q.pop_front();
        if (st[t.stat][t.pos]) continue;
        st[t.stat][t.pos] = true;
        int tx = t.pos/11, ty = t.pos%11;
        if (tx == n && ty == m) return dist[t.stat][t.pos];
        
        // 捡起来当前钥匙
        if (key[t.pos] && dist[t.stat|key[t.pos]][t.pos] > dist[t.stat][t.pos]) {
            dist[t.stat|key[t.pos]][t.pos] = dist[t.stat][t.pos];
            q.push_front({t.pos, t.stat | key[t.pos]});
        }
        
        for (int i = 0; i < 4; i++) {
            int x = tx+dx[i], y = ty+dy[i], pos = pp(x, y);
            if (x <= 0 || x > n || y <= 0 || y > m) continue;
            int wall = g[t.pos][pos];
            if (wall == -1) continue;
            // 没有对应的钥匙
            if (wall && !(t.stat >> wall & 1)) continue;
            if (dist[t.stat][pos] > dist[t.stat][t.pos] + 1) {
                dist[t.stat][pos] = dist[t.stat][t.pos] + 1;
                q.push_back({pos, t.stat});
            }
        }
    }
    return -1;
}

int main() {
    cin >> n >> m >> p;
    int k; cin >> k; while (k--) {
        int x1, x2, y1, y2, type;
        // 读入顺序搞错卡了我半小时
        cin >> x1 >> y1 >> x2 >> y2 >> type;
        // 不可通过的墙设为 -1
        if (type == 0) type = -1;
        g[pp(x1, y1)][pp(x2, y2)] = g[pp(x2, y2)][pp(x1, y1)] = type;
    }
    int s; cin >> s; while (s--) {
        int x, y, z;
        cin >> x >> y >> z;
        // 一个格子可能有多个钥匙
        key[pp(x, y)] |= 1 << z;
    }
    cout << dijkstra() << endl;
    return 0;
}
```

### [GXOI/GZOI2019] 旅行者

> J 国有 $n$ 座城市，这些城市之间通过 $m$ 条单向道路相连，已知每条道路的长度。
>
> 一次，居住在 J 国的 Rainbow 邀请 Vani 来作客。不过，作为一名资深的旅行者，Vani 只对 J 国的 $k$ 座历史悠久、自然风景独特的城市感兴趣。  为了提升旅行的体验，Vani 想要知道他感兴趣的城市之间「两两最短路」的最小值（即在他感兴趣的城市中，最近的一对的最短距离）。
>
> 也许下面的剧情你已经猜到了—— Vani 这几天还要忙着去其他地方游山玩水，就请你帮他解决这个问题吧。
>
> **输入格式**
>
> 每个测试点包含多组数据，第一行是一个整数 $T$，表示数据组数。注意各组数据之间是互相独立的。
>
> 对于每组数据，第一行包含三个正整数 $n,m,k$，表示 J 国的 $n$ 座城市（从 $1 \sim n$ 编号），$m$ 条道路，Vani 感兴趣的城市的个数 $k$。
>
> 接下来 $m$ 行，每行包括 $3$ 个正整数 $x,y,z$，表示从第 $x$ 号城市到第 $y$ 号城市有一条长度为 $z$ 的单向道路。注意 $x,y$ 可能相等，一对 $x,y$ 也可能重复出现。
>
> 接下来一行包括 $k$ 个正整数，表示 Vani 感兴趣的城市的编号。
>
> **输出格式**
>
> 输出文件应包含 $T$ 行，对于每组数据，输出一个整数表示 $k$ 座城市之间两两最短路的最小值。
>
> **数据范围**
>
> $2 \le k \le n$，$1 \le x,y \le n$，$1 \le z \le 2 \times 10^9$，$T \leq 5$，$1\le n\le 10^5$，$1\le m\le 5\times 10^5$
>
> 题目链接：[P5304](https://www.luogu.com.cn/problem/P5304)。

如果要求集合 $A$ 中的点到集合 $B$ 中点的最小值，这个很简单，建立一个虚拟原点和终点，分别连接边权为 $0$ 的边即可。

但现在是同一个集合内的点，我们可以随机划分两个集合，两个点分配到同一个集合的概率应当是 $\frac{1}{2}$，然后每个测试点都最多有 $5$ 组测试数据，假设我们要随机 $k$ 次，保证正确率大于 $99.9\%$，可以解方程：
$$
5(\frac{1}{2})^k \le 0.1\% \Rightarrow k \ge \log_2 5000\approx 12
$$
给他循环 $15$ 次，基本上不可能错了，由于用了大量 STL 所以比较缺氧。

```cpp
#include <queue>
#include <vector>
#include <random>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

typedef pair<int, int> PII;
typedef long long LL;
const int N = 100010, M = 600010;
const LL INF = 1e18;
int n, m, k;
int h[N], e[M], ne[M], w[M], id[N], idx;
mt19937 rng(20231004);
bool st[N];
LL dist[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void dijkstra() {
    priority_queue<PII, vector<PII>, greater<PII>> heap;
    for (int i = 0; i <= n; i++) dist[i] = INF, st[i] = false;
    for (int i = 0; i < k/2; i++) dist[id[i]] = 0, heap.push({0, id[i]});

    while (heap.size()) {
        int t = heap.top().second; heap.pop();
        if (st[t]) continue;
        st[t] = true;

        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];

            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                heap.push({dist[j], j});
            }
        }
    }
}

LL solve() {
    scanf("%d%d%d", &n, &m, &k);
    idx = 0;
    for (int i = 0; i <= n; i++) h[i] = -1;

    for (int i = 0, a, b, c; i < m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c);
    }
    for (int i = 0; i < k; i++)
        scanf("%d", &id[i]);

    LL res = INF;
    for (int i = 0; i < 15; i++) {
        shuffle(id, id+k, rng);
        dijkstra();
        for (int j = k/2; j < k; j++) res = min(res, dist[id[j]]);
    }
    return res;
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) printf("%lld\n", solve());
    return 0;
}
```

### [JLOI2011] 飞行路线

> Alice 和 Bob 现在要乘飞机旅行，他们选择了一家相对便宜的航空公司。该航空公司一共在 $n$ 个城市设有业务，设这些城市分别标记为 $0$ 到 $n-1$，一共有 $m$ 种航线，每种航线连接两个城市，并且航线有一定的价格。
>
> Alice 和 Bob 现在要从一个城市沿着航线到达另一个城市，途中可以进行转机。航空公司对他们这次旅行也推出优惠，他们可以免费在最多 $k$ 种航线上搭乘飞机。那么 Alice 和 Bob 这次出行最少花费多少？
>
> **输入格式**
>
> 第一行三个整数 $n,m,k$，分别表示城市数，航线数和免费乘坐次数。
>
> 接下来一行两个整数 $s,t$，分别表示他们出行的起点城市编号和终点城市编号。
>
> 接下来 $m$ 行，每行三个整数 $a,b,c$，表示存在一种航线，能从城市 $a$ 到达城市 $b$，或从城市 $b$ 到达城市 $a$，价格为 $c$。
>
> **输出格式**
>
> 输出一行一个整数，为最少花费。
>
> 对于 $100\%$ 的数据，$2 \le n \le 10^4$，$1 \le m \le 5\times 10^4$，$0 \le k \le 10$，$0\le s,t,a,b < n$，$a\ne b$，$0\le c\le 10^3$。
>
> 题目链接：[P4568](https://www.luogu.com.cn/problem/P4568)。

这是一个分层图问题，建立 $k+1$ 层图，然后在第 $j$ 层向第 $j+1$ 层图的相邻点连一条边权为 $0$ 的边。

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;
typedef pair<int, int> PII;
const int N = 110010, M = 2200010;
int n, m, k, S, T;
int h[N], e[M], ne[M], w[M], idx;
LL dist[N];
bool st[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void dijkstra() {
    priority_queue<PII, vector<PII>, greater<PII>> heap;
    memset(dist, 0x3f, sizeof dist);

    dist[S] = 0, heap.push({0, S});
    while (heap.size()) {
        int t = heap.top().second; heap.pop();
        if (st[t]) continue;
        st[t] = true;

        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                heap.push({dist[j], j});
            }
        }
    }
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d%d%d%d", &n, &m, &k, &S, &T);
    for (int i = 0, a, b, c; i < m; i++) {
        scanf("%d%d%d", &a, &b, &c);
        for (int j = 0; j <= k; j++)
            add(n*j+a, n*j+b, c), add(n*j+b, n*j+a, c);

        for (int j = 0; j <= k-1; j++)
            add(n*j+a, n*(j+1)+b, 0), add(n*j+b, n*(j+1)+a, 0);
    }

    for (int j = 0; j <= k-1; j++)
        add(n*j+T, n*(j+1)+T, 0);

    dijkstra();
    return !printf("%lld\n", dist[k*n+T]);
}
```

### [CF1915G] Bicycles
> 给定 $n$ 点 $m$ 边的图，每个点有一个自行车，参数为 $s_i$，使用 $j$ 个自行车会给边长乘 $s_j$，问 $1\sim n$ 的最短路，保证一定存在。$1\le n,m,s_i\le 10^3$。
> 题目链接：[CF1915G](https://codeforces.com/contest/1915/problem/G)。

首先，最短路一定长成这样子：

$$
d=s_{i_1} w_{j_1}+s_{i_2} w_{j_2}+\dots+s_{i_k}w_{j_k}
$$

其中，$s$ 会满足：

$$
s_{i_1}\ge s_{i_2}\ge \dots \ge s_{i_k}
$$

这是因为如果能用 $s_i$ 代表以后都可以使用，所以一定是非严格递减的，因此可以使用分层图。

建立 $n$ 张图，第 $i$ 张图代表当前最小值为 $s_i$，每到一个点 $j$ 都有一个连向第 $rs_{s_j}$ 张图的 $0$ 边权边，这里 $rs_i$ 为 $s_i$ 的逆映射。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
#define eb emplace_back
const int N = 1000010;
typedef pair<int, int> PII;
vector<PII> G[N];
int n, m, s[N], rs[N], L[N], R[N], W[N], dis[N];
bool st[N];

void dijkstra(int S) {
    for (int i = 1; i <= n*n; i++) dis[i] = 1e18, st[i] = false;
    priority_queue<PII> heap;
    heap.push({0, S});
    dis[S] = 0;
    while (heap.size()) {
        int t = heap.top().second; heap.pop();
        if (st[t]) continue;
        st[t] = true;
        for (const auto& [w, to]: G[t]) {
            if (dis[to] > dis[t] + w) {
                dis[to] = dis[t] + w;
                heap.push({-dis[to], to});
            }
        }
    }
}

int solve() {
    cin >> n >> m;
    for (int i = 1; i <= m; i++) cin >> L[i] >> R[i] >> W[i];
    for (int i = 1; i <= n; i++) cin >> s[i], rs[s[i]] = i;
    for (int i = 1; i <= n*n; i++) vector<PII>().swap(G[i]);

    for (int i = 1; i <= m; i++) {
        int a = L[i], b = R[i], w = W[i];
        for (int j = 1; j <= n; j++) {
            int u = (j-1) * n + a, v = (j-1) * n + b;
            G[u].eb(w*s[j], v);
            G[v].eb(w*s[j], u);
        }
    }

    // i: current graph
    for (int i = 1; i <= n; i++) {
        // j: current node
        for (int j = 1; j <= n; j++) {
            int to = rs[s[j]];
            if (to == i) continue;
            G[(i-1) * n + j].eb(0, (to-1) * n + j);
        }
    }

    dijkstra(1);

    int ans = 1e18;
    for (int i = 1; i <= n; i++) ans = min(ans, dis[(i-1) * n + n]);
    return ans;
}

signed main() {
    int t;
    cin >> t;
    while (t--) cout << solve() << '\n';
    return 0;
}
```

## 多源最短路

### 牛的旅行

> 农民John的农场里有很多牧区，有的路径连接一些特定的牧区。
>
> 一片所有连通的牧区称为一个牧场。
>
> 但是就目前而言，你能看到至少有两个牧区不连通。
>
> 现在，John想在农场里添加一条路径（注意，恰好一条）。
>
> 一个牧场的直径就是牧场中最远的两个牧区的距离（本题中所提到的所有距离指的都是最短的距离）。
>
> John将会在两个牧场中各选一个牧区，然后用一条路径连起来，使得连通后这个新的更大的牧场有最小的直径。
>
> 注意，如果两条路径中途相交，我们不认为它们是连通的。
>
> 只有两条路径在同一个牧区相交，我们才认为它们是连通的。
>
> 现在请你编程找出一条连接两个不同牧场的路径，使得连上这条路径后，所有牧场（生成的新牧场和原有牧场）中直径最大的牧场的直径尽可能小。
>
> 输出这个直径最小可能值。
>
> 原题链接：[AcWing 1125](https://www.acwing.com/problem/content/1127/)。

对于一个连通块内部，直径指的就是所有两点距离最小值的最大值，这里用 $d_i$ 表示某个连通块的直径，$d(v)$ 表示从某点出发的直径。如果答案在两个连通块见通过了新连边，那么应该是下面的式子。
$$
\text{ans}=\max \left \{\begin{aligned}
&\max (d_1, d_2)\\
&d(v_1)+\text{dist}(v_1,v_2)+d(v_2)
\end{aligned}\right .
$$
所以本题先用 Floyd 在连通块内部求一下最小值，然后算 $d$ 即从每个点出发的直径应该是多少，最后综合求一个答案。

```cpp
#include <iostream>
#include <cmath>
using namespace std;

#define x first
#define y second
typedef pair<int, int> pii;
const int N = 155;
const double INF = 1e20;
int n;
char f[N][N];
double g[N][N], d[N];
pii p[N];

double dist(pii p, pii q) {
    int dx = p.x-q.x, dy = p.y-q.y;
    return sqrt(dx*dx+dy*dy);
}

int main() {
    cin >> n;
    for (int i = 0; i < n; i++) cin >> p[i].x >> p[i].y;
    for (int i = 0; i < n; i++) cin >> f[i];
    
    // 建图
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            if (i != j) {
                if (f[i][j] == '1') g[i][j] = dist(p[i], p[j]);
                else g[i][j] = INF;
            }
    
    // Floyd
    for (int k = 0; k < n; k++)
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                g[i][j] = min(g[i][j], g[i][k]+g[k][j]);
    
    // 直径
    double res1 = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++)
            if (g[i][j] < INF)
                d[i] = max(d[i], g[i][j]);
        res1 = max(res1, d[i]);
    }
    
    double res2 = INF;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            if (g[i][j] >= INF)
                res2 = min(res2, d[i] + d[j] + dist(p[i], p[j]));
    
    return !printf("%.6lf\n", max(res1, res2));
}
```

### 排序(传递闭包)

> 给定 n 个变量和 m 个不等式。其中 n 小于等于 26，变量分别用前 n 的大写英文字母表示。
>
> 不等式之间具有传递性，即若 A>B 且 B>C 则 A>C。
>
> 请从前往后遍历每对关系，每次遍历时判断：
>
> - 如果能够确定全部关系且无矛盾，则结束循环，输出确定的次序；
> - 如果发生矛盾，则结束循环，输出有矛盾；
> - 如果循环结束时没有发生上述两种情况，则输出无定解。
>
> 原题链接：[AcWing 343](https://www.acwing.com/problem/content/345/)。

传递背包与并查集类似，如果 a -> b 并且 b -> c 那么就建 a -> c 的边，但并查集只能确定两元素是否在同一集合中，并不知道它们之间的关系，所以这种问题还是用传递闭包的方式做。

一个模板可能长成这样：

```cpp
void floyd() {
    for (int k = 1; k <= n; k++)
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++)
                if (g[i][k] && g[k][j]) g[i][j] = 1;
}
```

本题判断不等式是否成立的条件为：

1. 如果 $g(i,i)=1$ 那么一定有矛盾。根据算法如果存在 $i\ne j$ 且 $g(i,j)=g(j,i)=1$ 那么一定会使 $g(i,i)=g(j,j)=1$，所以这个情况已经被考虑过了。
2. 如果 $\exist i,j \in V,i\ne j,g(i,j)=g(j,i)= 0$ 就意味着至少存在一对关系不确定。
3. 如果上面都不成立就可以确定所有大小关系了。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int N = 26;
int n, m;
bool g[N][N], st[N];

void floyd() {
    for (int k = 0; k < n; k++)
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                if (g[i][k] && g[k][j]) g[i][j] = 1;
}

// 0: 无法确定
// 1: 已经确定
// 2: 出现矛盾
int check() {
    for (int i = 0; i < n; i++)
        if (g[i][i])
            return 2;
    
    // 由对称性可以 j 循环到 i-1 为止
    for (int i = 0; i < n; i++)
        for (int j = 0; j < i; j++)
            if (g[i][j] == 0 && g[j][i] == 0)
                return 0;
    
    return 1;
}

char getmin() {
    for (int i = 0; i < n; i++) {
        if (st[i]) continue;
        
        bool has_lower = false;
        for (int j = 0; j < n; j++)
            if (!st[j] && g[j][i]) has_lower = true;
        if (has_lower) continue;
        
        st[i] = true;
        return i+'A';
    }
    // unreachable code
    return 'N';
}

int main() {
    while (cin >> n >> m, n || m) {
        int t = 0, type = 0;
        memset(g, 0, sizeof g);
        memset(st, 0, sizeof st);
        for (int i = 1; i <= m; i++) {
            char a, _, b;
            cin >> a >> _ >> b;
            // 不要遇到别的 type 直接 break 这会导致接下来的读入问题
            if (type == 0) {
                // 这真的很容易写成 a-'A', b-'B', 巨坑
                g[a-'A'][b-'A'] = 1;
                floyd();
                type = check();
                t = i;
            }
        }
        if (type == 0) puts("Sorted sequence cannot be determined.");
        else if (type == 2) printf("Inconsistency found after %d relations.\n", t);
        else {
            printf("Sorted sequence determined after %d relations: ", t);
            for (int i = 0; i < n; i++) printf("%c", getmin());
            puts(".");
        }
    }
    return 0;
}
```

## 最小环

> 给定一张无向图，求图中一个至少包含 3 个点的环，环上的节点不重复，并且环上的边的长度之和最小。
>
> 该问题称为无向图的最小环问题。
>
> 你需要输出最小环的方案，若最小环不唯一，输出任意一个均可。
>
> 原题链接：[AcWing 344](https://www.acwing.com/problem/content/346/)。

当 Floyd 算法求最短路时，设点 $i,j<k$，从 $i\to j$ 在前 $k$ 个点中（不包含 $k$）的最短路中所有点一定都不经过 $k$，这是算法的原理保证的。
$$
L(i,j,k)=w(i,k)+w(k,j)+\delta(i,j)
$$

```cpp
#include <iostream>
#include <cstring>
#include <vector>
using namespace std;

const int N = 110;
int n, m;
int g[N][N], d[N][N], pos[N][N];
vector<int> sln;

void get_mid(int i, int j) {
    // 没有中间的点
    if (pos[i][j] == 0) return;
    
    int k = pos[i][j];
    get_mid(i, k);
    sln.push_back(k);
    get_mid(k, j);
}

int main() {
    cin >> n >> m;
    memset(g, 0x3f, sizeof g);
    for (int i = 1; i <= n; i++) {
        g[i][i] = 0;
    }
    for (int i = 1; i <= m; i++) {
        int a, b, c;
        cin >> a >> b >> c;
        g[a][b] = g[b][a] = min(g[a][b], c);
    }
    memcpy(d, g, sizeof d);
    
    int res = 0x3f3f3f3f;
    for (int k = 1; k <= n; k++) {
        for (int i = 1; i < k; i++)
            for (int j = 1; j < i; j++)
                // 三个 0x3f3f3f3f 可能爆 int
                if ((long long)d[i][j] + g[i][k] + g[k][j] < res) {
                    res = d[i][j] + g[i][k] + g[k][j];
                    sln.clear();
                    sln.push_back(k);
                    sln.push_back(i);
                    get_mid(i, j);
                    sln.push_back(j);
                }
        
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++)
                if (d[i][j] > d[i][k] + d[k][j]) {
                    d[i][j] = d[i][k] + d[k][j];
                    // i -> k -> j
                    pos[i][j] = k;
                }
    }
    
    if (sln.size()) for (int s : sln) cout << s << ' ';
    else cout << "No solution.";
    
    return 0;
}
```

