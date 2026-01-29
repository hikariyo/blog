---
date: 2023-07-14 15:05:05
updated: 2023-07-14 15:05:05
title: 搜索 - A*
katex: true
tags:
- Algo
- C++
- Search
categories:
- OI
---

## 简介

A* 算法用来解决状态数量非常庞大的问题，如前面遇到的八数码这种每一步变换都是不确定的这类问题，它与堆优化版的 Dijkstra 算法长得很像。

### 流程

1. 用小根堆存储所有节点，排序依据是这个节点到原点的距离 + 到终点的估计距离。
2. 从队列中取出第一个节点，如果这是终点就找到了最短距离。
3. 将当前节点的所有**没有遍历过**或者**能缩短现有路径长度**的子节点入队。
4. 重复上述流程。

### 证明

设估计当前点到终点距离用到的估价函数为 $f^*(x)$，到终点的真实最短距离为 $f(x)$，到起点的距离是 $g(x)$ 估价函数需要满足 $f^*(x)\le f(x)$。

用反证法证明找到的一定是最短路。设找到的 $d_0$ 不是最短路 $d_s$，那么：
$$
d_0\gt d_s
$$
找到的路径 $d_0$ 中一定与最短路中的某点 $u$ 有重合（起码起点要重合），那么：
$$
g(u)+f^*(u)\le g(u)+f(u) =d_s<d_0
$$
也就是从优先队列中先取出了一个比 $g(u)+f^*(u)$ 更大的元素，因为这是小根堆，所以发生矛盾，因此找到的一定就是最短路。

### 注意

除终点以外任何一个点第一次出队的时候与原点的距离都**不一定是最小值**。

```text
o → o → o
↑       ↓
s → o → x → o → o
```

考虑这样的情况，其中 `s` 是起点，因为估价函数只要比真实值小就可以，那么完全有可能从上面扩展到 `x` 点，而此时 `x` 与原点的距离**不是最小值**。

由此也可以看出 A* 算法中每个点并不一定只被拓展一次，所以与 Dijkstra 算法还是不同的。

### 判重对比

+ 朴素 BFS 算法在入队时判重，原因是在一个点首次被拓展到的时候它与原点的距离一定最小。
+ 朴素 Dijkstra 虽然没用到队列，但它也是一个点首次遇到的时候判重，原因同上。
+ 堆优化的 Dijkstra 算法在出队时判重，原因是只有当一个点出队时才能说明它与原点的距离最小。
+ A* 不可以判重，原因是出队的时候也不能判断某个点的距离是否最小。

## 八数码

> 在一个 3*3 的网格中，1∼8 这 8 个数字和一个 `x` 恰好不重不漏地分布在这 3×3 的网格中。
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
> 例如，示例中图形就可以通过让 `X` 先后与右、下、右三个方向的数字交换成功得到正确排列。
>
> 交换过程如下：
>
> ```
> 1 2 3   1 2 3   1 2 3   1 2 3
> x 4 6   4 x 6   4 5 6   4 5 6
> 7 5 8   7 5 8   7 x 8   7 8 x
> ```
>
> 把 `X` 与上下左右方向数字交换的行动记录为 `u`、`d`、`l`、`r`。
>
> 现在，给你一个初始网格，请你通过最少的移动次数，得到正确排列。
>
> 输入输出等格式请参考 [原题链接](https://www.acwing.com/problem/content/description/181/)。

### 可达性分析

将九宫格按顺序依次排成一行，包含空格，统计其中逆序对的数量，有以下定理。

定理：逆序对数量奇偶性相同的两个状态相互可达，反之亦然。

要证明这个定理，需要先用几个引理。

+ 引理1：一次合法的变换不会改变状态逆序对数量的奇偶性。

    1. 如果在行内变换，那显然是不会改变状态表示的，此时奇偶性不变。
    2. 如果在行间变换，相当于一个数依次与相邻的两个数交换位置，那么可能逆序对数量 +2, -2, 或不变。

    因此，一次合法的变换并不会改变逆序对数量的奇偶性。

+ 引理2：合法变换是可逆的。如果能从 A -> B 那么也一定能从 B -> A，每一次交换反着来就可以，显然成立。

+ 引理3：所有奇状态都可以转化成 `x 2 1 3 4 5 6 7 8`，所有偶状态都可以转换为 `x 1 2 3 4 5 6 7 8`。

    对于任意 `a b c x` 都可以把 `a, b, c ` 移动到 `x` 空位上并且变换仅限这四个位置，下面用 `l,r,u,d` 分别代表上，下，左，右记录空格与数字的交换顺序。

    + a： `u`，移动后是 `x b c a`。

    + b：`urrdluldrrulld`，移动后是 `x c a b`。
    + c：`urrdluldrru`，移动后是 `a b x c`。

    同样，还可以证明对于任意 `a b c d x` 可以把 `a, b, c, d` 移动到 `x` 空位上并且仅限这五个位置，下面的移动方法不一定唯一，都是用计算机找出来的路径。

    + a：`uldrurdllurdrull`，移动后是 `x c b d a`。
    + b：`u`，移动后是 `a x c d b`。
    + c：`lurdldrruuldrdllu`，移动后是 `b a d x c`。
    + d：`l`，移动后是 `a b c x d`。

    先把 8 移动到倒数第二行，然后用第一个方法换下来，然后将它移到最后一个格子；然后将 7 换到倒数第二行，用第二个方法换下来；然后是 6，依次换下来，因此引理 3 成立。

    这里可能不太严谨，但是思路是利用这些方法将数字依次交换下来。

由引理 3 得到奇偶状态都可以转化成两个目标状态，而由引理 2 知转换是可逆的，于是充分性得证。

由引理 1 就可以推出来如果两个状态相互可达，那么它们的逆序对数量奇偶性一定相同，于是必要性得证。

### A* 分析

这里的估价函数 $f^*(x)$ 就可以取各点到它应该在的位置的**曼哈顿距离之和**，最好情况下我们可以依次将这些数字移动到它们的位置，因此可以保证 $f^*(x) \le f(x)$，这是 A* 能找到最短路的必要条件。

当这个状态是无解的时候，A* 算法会把所有状况都枚举一遍，这样效率是很低的，所以用上面的性质先判断是否有解然后再用 A* 搜索最短路径。

```cpp
#include <iostream>
#include <string>
#include <queue>
#include <algorithm>
#include <unordered_map>
using namespace std;

typedef pair<int, string> pis;
string start, target = "12345678x";
priority_queue<pis, vector<pis>, greater<pis>> q;
unordered_map<string, int> dist;
unordered_map<string, pair<char, string>> pre;
int dx[] = {-1, 0, 1, 0}, dy[] = {0, -1, 0, 1};
char way[] = "uldr";

int f(string& s) {
    int res = 0;
    for (int i = 0; i < 9; i++) {
        if (s[i] == 'x') continue;
        int n = s[i] - '1';
        res += abs(n/3-i/3) + abs(n%3-i%3);
    }
    return res;
}

string astar() {
    q.push({f(start), start});
    dist[start] = 0;
    while (q.size()) {
        pis p = q.top(); q.pop();
        string cur = p.second;
        if (cur == target) break;
        
        int idx = cur.find('x'), tx = idx/3, ty = idx%3;
        for (int i = 0; i < 4; i++) {
            int x = tx+dx[i], y = ty+dy[i];
            if (x < 0 || x >= 3 || y < 0 || y >= 3) continue;
            string child = cur;
            swap(child[x*3+y], child[tx*3+ty]);
            if (dist.count(child) == 0 || dist[child] > dist[cur] + 1) {
                dist[child] = dist[cur]+1;
                pre[child] = {way[i], cur};
                q.push({dist[child] + f(child), child}); 
            }
        }
    }
    
    string ans, t = target;
    while (t != start) {
        auto p = pre[t];
        t = p.second;
        ans += p.first;
    }
    reverse(ans.begin(), ans.end());
    return ans;
}

int main() {
    for (int i = 0; i < 9; i++) {
        char ch; cin >> ch;
        start += ch;
    }
    // 逆序对数量
    int cnt = 0;
    for (int i = 0; i < 9; i++) {
        for (int j = i+1; j < 9; j++) {
            if (start[j] == 'x' || start[i] == 'x') continue;
            if (start[j] < start[i]) cnt++;
        }
    }
    if (cnt & 1) cout << "unsolvable";
    else cout << astar();
    
    return 0;
}
```

## 第 K 短路

> 给定一张 N 个点（编号 1,2…N），M 条边的有向图，求从起点 S 到终点 T 的第 K 短路的长度，路径允许重复经过点或边。
>
> **注意：** 每条最短路中至少要包含一条边。
>
> 输入输出等格式见 [原题链接](https://www.acwing.com/problem/content/180/)。

本题的估价函数就用反向图中终点到各点的最短距离即可，也就是我们需要建一个反向图，跑一遍 Dijkstra 求最短距离。

注意，当终点与起点重合时，最短路是一条边都不走，这不符合题意，因此这时要 `K++`。

类比证明 A* 算法正确性时的证明方法，这里同样可以证明出第 K 次弹出的终点就是第 K 短路。

```cpp
#include <iostream>
#include <algorithm>
#include <queue>
#include <cstring>
using namespace std;

// 建反向图 边数比它给的乘2
typedef pair<int, int> pii;
typedef pair<int, pii> piii;
const int N = 1010, M = 2e4+10;
int n, m, S, T, K;
int h[N], rh[N], e[M], ne[M], w[M], idx;
int dist[N];

void add(int h[], int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

void dj() {
    memset(dist, 0x3f, sizeof dist);
    dist[T] = 0;
    priority_queue<pii, vector<pii>, greater<pii>> heap;
    heap.push({0, T});
    while (heap.size()) {
        pii _t = heap.top(); heap.pop();
        int t = _t.second;
        // 这里是反向图！！
        for (int i = rh[t]; ~i; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                heap.push({dist[j], j});
            }
        }
    }
}

int astar() {
    priority_queue<piii, vector<piii>, greater<piii>> heap;
    // 估价距离 到原点距离 点
    // 因为它不是单纯最短路 所以需要记录一下到原点的距离
    heap.push({dist[S], {0, S}});
    int cnt = 0;
    while (heap.size()) {
        piii _t = heap.top(); heap.pop();
        int t = _t.second.second, distance = _t.second.first;
        
        if (t == T) cnt++;
        if (cnt == K) return distance;
        
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            // 跳过不可达到终点的节点
            if (dist[j] == 0x3f3f3f3f) continue;
            heap.push({distance + w[i] + dist[j], {distance + w[i], j}});
        }
    }
    return -1;
}

int main() {
    cin >> n >> m;
    memset(h, -1, sizeof h);
    memset(rh, -1, sizeof rh);
    for (int i = 0; i < m; i++) {
        int a, b, c;
        cin >> a >> b >> c;
        add(h, a, b, c);
        add(rh, b, a, c);
    }
    cin >> S >> T >> K;
    if (S == T) K++;
    dj();
    cout << astar();
    return 0;
}
```

