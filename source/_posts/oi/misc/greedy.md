---
date: 2023-06-19 21:46:21
title: 杂项 - 贪心及相关例题
katex: true
tags:
- Algo
- C++
categories:
- OI
---

## 区间选点

> 给定 N 个闭区间 `[ai,bi]`，请你在数轴上选择尽量少的点，使得每个区间内至少包含一个选出的点。
>
> 输出选择的点的最小数量。位于区间端点上的点也算作区间内。
>
> 题目链接：[AcWing 905](https://www.acwing.com/problem/content/907/)。

先把区间按照右端点排序，然后每个区间尽量选择右端点，如果选过的点包含了当前区间则跳过，因为这样选择之后一定会选中 M 个**不相互重叠**的区间，想覆盖这些区间的最小点数一定是 M：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

typedef pair<int, int> pii;
const int N = 1e5+10;

pii ranges[N];
int points[N], cnt, n;

int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        // 按照右端点排序 所以右端点在 first
        cin >> ranges[i].second >> ranges[i].first;
    }
    sort(ranges, ranges+n);
    
    points[0] = -2e9;
    for (int i = 0; i < n; i++) {
        int l = ranges[i].second, r = ranges[i].first;
        if (l > points[cnt]) {
            points[++cnt] = r;
        }
    }
    
    cout << cnt << endl;
    return 0;
}
```

## 最大不相交区间数量

> 给定 N 个闭区间 `[ai,bi]`，请你在数轴上选择若干区间，使得选中的区间之间互不相交（包括端点）。
>
> 输出可选取区间的最大数量。

本题代码和上一题一模一样。

分析：每次不断选择右端点，最后选出来的一定是一组 M 个互不相交的区间，可以得到 `M <= Ans`，接下来需要用反证法证明 `Ans <= M`：

假设 `Ans > M`，那么在已经选出来 `M` 个由小到大排列的区间的情况下，这些区间的中间不可能存在一个没被选中的并且与所有区间都不相交的区间；那么这个多出来的区间一定在这些区间的左侧或右侧，但是如果存在这种情况 M 一定会包含这些区间，所以假设不成立，`Ans <= M`。

## 区间分组

> 给定 N 个闭区间 `[ai, bi]`，请你将这些区间分成若干组，使得每组内部的区间两两之间（包括端点）没有交集，并使得组数尽可能小。输出最小组数。
>
> [原题链接](https://www.acwing.com/problem/content/908/)

做法如下：

1. 将所有区间按左端点升序排序。
2. 从前往后处理每个区间，判断是否能在当前区间放在某个组中 `left > group.right`，如果有就放进去，没有就新建一个组。

证明：当新建第 `cnt` 个组时，说明这个组一定与前 `cnt-1` 个所有组都相交，而我们按照左端点升序排序，说明当前区间的左端点一定大于所有区间的左端点，同时又由于它无法放到任何一组中，那么它的左端点也小于任何一个右端点，因此这 `cnt` 个组中存在公共点，所以这 `cnt` 个组一定没有更少的划分方式。

对于所有组的右端点，这里用小根堆存储，每次与最小的作比较，如果它比最小的右端点都小证明当前区间无法放到任何一个组中。

```cpp
#include <iostream>
#include <algorithm>
#include <queue>
using namespace std;

const int N = 1e5+10;
typedef pair<int, int> pii;
pii ranges[N];

int main() {
    int n;
    cin >> n;
    for (int i = 0; i < n; i++) cin >> ranges[i].first >> ranges[i].second;
    sort(ranges, ranges + n);
    
    priority_queue<int, vector<int>, greater<int>> heap;
    for (int i = 0; i < n; i++) {
        int l = ranges[i].first, r = ranges[i].second;
        if (heap.empty() || heap.top() >= l) {
            // 不能放在任何一个组中
            heap.push(r);
        } else {
            heap.pop();
            heap.push(r);
        }
    }
    
    cout << heap.size() << endl;
    return 0;
}
```

## 区间覆盖

> 给定 N 个闭区间 [ai, bi] 以及一个线段区间 [s,t]，请你选择尽量少的区间，将指定线段区间完全覆盖。
>
> 输出最少区间数，如果无法完全覆盖则输出 -1。
>
> [原题链接](https://www.acwing.com/problem/content/909/)

思路如下：

1. 将所有区间按左端点升序排序。
2. 从前向后以此枚举每个区间，找到一个能覆盖掉 start 的右端点最大的区间，将 start 替换成右端点。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1e5+10;
typedef pair<int, int> pii;
pii ranges[N];
int n;

bool solve_point(int p) {
    for (int i = 0; i < n; i++) {
        if (ranges[i].first <= p && p <= ranges[i].second) return true;
    }
    return false;
}

int main() {
    int st, ed;
    cin >> st >> ed >> n;
    for (int i = 0; i < n; i++) cin >> ranges[i].first >> ranges[i].second;
    sort(ranges, ranges+n);
    
    // 特判：当线段其实为一个点时
    if (st == ed) {
        if (solve_point(st)) cout << 1;
        else cout << -1;
        return 0;
    }
    
    int cnt = 0, i = 0;
    while (st < ed) {
        int cur_st = st;
        // 当前端点一定要与 st 做比较 因为 cur_st 会在循环过程中被修改
        while (ranges[i].first <= st && i < n) {
            cur_st = max(cur_st, ranges[i].second);
            i++;
        }
        // 如果当前循环并没有更新 start 意味着找不到能覆盖掉 start 的区间了
        if (cur_st == st) break;
        st = cur_st;
        // 找到一个能覆盖 start 且右端点最大的 计数加一
        cnt++;
    }
    
    if (st < ed) cout << -1;
    else cout << cnt;

    return 0;
}
```

也可以换一种思路。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1e5+10;
typedef pair<int, int> pii;
pii ranges[N];

int main() {
    int st, ed, n;
    cin >> st >> ed >> n;
    for (int i = 0; i < n; i++) cin >> ranges[i].first >> ranges[i].second;
    sort(ranges, ranges+n);
    
    int cnt = 0;
    bool success = false;
    for (int i = 0; i < n; i++) {
        // 从 i 开始找到能覆盖 st 的最大右端点
        int j = i, r = -2e9;
        while (j < n && ranges[j].first <= st) {
            r = max(r, ranges[j].second);
            j++;
        }
        
        if (r < st) {
            cnt = -1;
            break;
        }
      
        cnt++;
        
        st = r;
        i = j-1;
        
        // 更新后的 st 如果等于 ed 也是可以的
        // 这时满足了这个线段被覆盖
        if (st >= ed) {
            success = true;
            break;
        }
    }
    
    if (success) cout << cnt;
    else cout << -1;
    
    return 0;
}
```

## 听音乐

> 小 P 正在听音乐，现在他的播放列表里有 $n$ 首音乐，编号为 $1,2,\dots,n$，每个音乐有两个属性 $a_i,b_i$。
>
> 重复听一首音乐会让小 P 感到厌倦，所以当他第 $k$ 次听到音乐 $i$ 时，他得到的愉悦度为 $\max(a_i-(k-1)b_i,0)$。
>
> 小 P 可以控制播放器，具体的在最开始时播放器正准备播放音乐 $1$，假设目前正准备播放音乐 $i$，则每次小 P 可以选择：
>
> - 切换到音乐 $i-1$（特别地若 $i=1$ 则切换到 $n$）
> - 切换到音乐 $i+1$（特别地若 $i=n$ 则切换到 $1$）
> - 播放当前音乐。
>
> 小 P 只能作 $m$ 次选择，你需要告诉他这种情况下他能得到的愉悦度最大是多少。
>
> $1\le n\le 200, 1\le m\le 1000, 0\le b_i\le a_i\le 5000$

首先不难发现如果能走遍整个区间，它是不需要再走第二遍的，否则答案一定不优。

那么可以拆环成链，规定从 $n+1$ 开始向左右走，先 $O(n^2)$ 枚举边界，然后我们先用 $L+R+\min(L,R)$ 走遍左右区间，这里 $L,R$ 表示的是从 $n+1$ 走的距离。剩下的开始插入播放操作。

考虑到值域小，直接用桶插入 $a_i,a_i-b_i,a_i-2b_i,\cdots$，一定会先选大的再选小的，并且不会超过 $m$，因此复杂度大概是 $O(n^2(m+V))$。
```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 410;
int n, m, tim[N], a[N], b[N], ts;
int cnt[5010];
bool st[N];

struct PII {
    int first, second;
    bool operator<(const PII& p) const {
        return first < p.first;
    }
};

void read(int& t) {
    t = 0;
    char ch = getchar();
    while (!isdigit(ch)) ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch ^ 48), ch = getchar();
}

void addbucket(int p) {
    int real = (p-1)%n+1;
    if (st[real]) return;
    st[real] = true;
    int now = a[p];
    int times = 0;
    while (now > 0 && times < m) {
        cnt[now]++;
        times++;
        now -= b[p];
    }
}

int main() {
    freopen("music.in", "r", stdin);
    freopen("music.out", "w", stdout);
    read(n), read(m);
    for (int i = 1; i <= n; i++) read(a[i]), a[i+n] = a[i];
    for (int i = 1; i <= n; i++) read(b[i]), b[i+n] = b[i];

    int ans = 0;
    for (int l = 1; l <= n+1; l++) {
        memset(cnt, 0, sizeof cnt);
        memset(st, 0, sizeof st);
        for (int j = l; j <= n+1; j++) addbucket(j);

        for (int r = n+1; r <= 2*n; r++) {
            int A = n+1-l, B = r-n-1, times = m - (A + B + min(A, B));
            if (times <= 0) continue;
            addbucket(r);
            int res = 0;
            for (int k = 5000; k > 0 && times > 0; k--) {
                int c = min(times, cnt[k]);
                res += c * k;
                times -= c;
            }
            ans = max(ans, res);
        }
    }

    printf("%d\n", ans);

    return 0;
}
```

## [NOIP2004] 合并果子

> 这里大致抽象一下：有 n 堆果子，每次从这里面任意挑出 2 堆合并，每堆果子的数量不同，合并它们需要的体力等于数量，求合并成 1 堆消耗的最小体力。
>
> [原题链接](https://www.acwing.com/problem/content/150/)

这是一个 Huffman 树的问题，解法是每次挑最小的两堆合并，证明：

对于合并 `n` 堆果子的方法 `f(n) = f(n-1) + a + b`，其中 `a, b` 代表当前堆中最小的两堆，`f(n-1)` 代表除去 `a, b` 后的方法，因此每一次取最优解就能达到全局的最优解。

```cpp
#include <queue>
#include <iostream>
using namespace std;

int main() {
    int n;
    cin >> n;
    priority_queue<int, vector<int>, greater<int>> heap;
    while (n--) {
        int a; cin >> a; heap.push(a);
    }
    
    int res = 0;
    while (heap.size() > 1) {
        int a = heap.top(); heap.pop();
        int b = heap.top(); heap.pop();
        res += a+b;
        heap.push(a+b);
    }
    // 最后一堆不需要合并
    cout << res << endl;
    return 0;
}
```

## [CF638C] Road Improvement

> 给定一棵有 N 个节点的树，你可以使用**两支相邻节点的队伍**来修筑它们之间的道路 且 **每支队伍一天只能工作一次**。问最少需要多少天把所有路修完。输出最短时间和具体方案。
>
> 题目链接：[CF638C](https://codeforces.com/problemset/problem/638/C)。

最短时间是不难求的，就是结点的最大度数。我们可以构造答案，这样做：

+ 以最大度数为 size 建立这么多个集合。
+ 将同一个点出发的不同边放到不同的集合中，并且避开来边放的集合。

其实也不用记录度数，直接看每次用了多少个集合，然后取一个最大值就行了。

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
#include <vector>
using namespace std;

const int N = 200010, M = 400010;
int h[N], e[M], ne[M], idx;
int id[M];
int n, sz;
vector<int> g[N];

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u, int from) {
    int nowidx = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (i == (from ^ 1)) continue;
        if (from != -1 && nowidx == id[from]) nowidx++;
        id[i] = id[i^1] = nowidx;
        g[nowidx].push_back(min(i, i^1) / 2 + 1);
        nowidx++;
        dfs(j, i);
    }

    sz = max(sz, nowidx);
}

int main() {
    memset(h, -1, sizeof h);
    memset(id, -1, sizeof id);
    scanf("%d", &n);
    for (int i = 0; i < n-1; i++) {
        int a, b;
        scanf("%d%d", &a, &b);
        add(a, b), add(b, a);
    }
    dfs(1, -1);
    printf("%d\n", sz);
    for (int i = 0; i < sz; i++) {
        printf("%lu ", g[i].size());
        for (int edge: g[i]) printf("%d ", edge);
        puts("");
    }
    return 0;
}
```

## [NOIP2018] 赛道修建

> 简单翻译一下：给定一棵带边权无根树，试找到 M 条不相交的路径，使得这些路径长度的最小值尽可能大，输出最小的赛道长度的最大值。
>
> 题目链接：[P5021](https://www.luogu.com.cn/problem/P5021)。

首先肯定是二分答案然后验证是否可行。

考虑题目给出的菊花图的情况，我们可以想到，如果一条边本身就大于我们二分出来的答案，直接记录即可，否则需要将其与另一条边进行配对，这里肯定是让与它匹配的边权尽可能小。

然后，我们考虑一般情况。对于倒数第二层结点，我们需要在它伸出来的边里进行尽可能多的匹配，然后返回一个最大的没用过的边权接到来边上。

如果最大边权被匹配过，是否会使得答案更劣？答案是不会，因为返回上去最多只能增加一个贡献，而在本层匹配掉也是一个贡献，因此答案不会因为这个变得更劣。

这里 STL 不开 O2 会超时，主要是 “找到最小的能匹配一条边的另一条边” 这个操作需要用 `lower_bound`。

```cpp
#include <cstdio>
#include <cstring>
#include <set>
#include <algorithm>
using namespace std;

const int N = 50010, M = 100010;
int h[N], e[M], ne[M], w[M], idx;
int n, m;
int cnt, len;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

// 返回当前结点出发的没用过的最大边权
int dfs(int u, int fa) {
    multiset<int> s;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == fa) continue;
        int v = w[i] + dfs(j, u);
        if (v >= len) cnt++;
        else s.insert(v);
    }
    
    int maxv = 0;
    while (s.size()) {
        auto a = s.begin();
        auto b = s.lower_bound(len - *a);
        if (b == a) b++;
        if (b == s.end()) {
            maxv = max(maxv, *a);
            s.erase(a);
        }
        else {
            cnt++;
            s.erase(a), s.erase(b);
        }
    }
    return maxv;
}

bool check(int mid) {
    len = mid, cnt = 0;
    dfs(1, -1);
    return cnt >= m;
}

int main() {
    memset(h, -1, sizeof h);
    scanf("%d%d", &n, &m);
    int l = 0, r = 0;
    for (int i = 0; i < n-1; i++) {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c), add(b, a, c);
        r += c;
    }
    while (l < r) {
        int mid = (l + r + 1) >> 1;
        if (check(mid)) l = mid;
        else r = mid-1;
    }
    printf("%d\n", r);
    return 0;
}
```

## [NOIP2012] 国王游戏

>  恰逢 H 国国庆，国王邀请 n 位大臣来玩一个有奖游戏。首先，他让每个大臣在左、右手上面分别写下一个整数，国王自己也在左、右手上各写一个整数。然后，让这 n 位大臣排成一排，国王站在队伍的最前面。排好队后，所有的大臣都会获得国王奖赏的若干金币，每位大臣获得的金币数分别是：排在该大臣前面的所有人的左手上的数的乘积除以他自己右手上的数，然后向下取整得到的结果。
>
> 国王不希望某一个大臣获得特别多的奖赏，所以他想请你帮他重新安排一下队伍的顺序，使得获得奖赏最多的大臣，所获奖赏尽可能的少。注意，国王的位置始终在队伍的最前面。
>
> 题目链接：[P1080](https://www.luogu.com.cn/problem/P1080)。

重要性质：**交换两个人的位置后前后所有人的值不会改变**，因此这是一个排序贪心。

设前面所有人的乘积为 $T$，然后有 $a_i,b_i,a_{i+1},b_{i+1}$，那么当：
$$
\max\{T/b_i,Ta_i/b_{i+1}\} \le \max\{T/b_{i+1},Ta_{i+1}/b_i\}
$$
满足要求，两边变形得到：
$$
\max\{b_{i+1},a_ib_i\} \le \max\{b_i,a_{i+1}b_{i+1}\}
$$

当左边取 $b_{i+1}$ 时，由于 $b_{i+1} \le a_{i+1}b_{i+1}$ 恒成立，因此无需考虑此情况。

当左边取 $a_ib_i$ 时，由于 $a_ib_i\ge b_i$ 恒成立，因此此时右边应该取 $a_{i+1}b_{i+1}$ 并且如果成立有 $a_ib_i\le a_{i+1}b_{i+1}$，也就有 $a_{i+1}{i+1}\ge b_i$ 成立。

综上所述，这个条件可以转化为：
$$
a_ib_i\le a_{i+1}b_{i+1}
$$
然而你写上面含有 $\max$ 的也完全没有问题，本题还要用高精度，复习一下。

```cpp
#include <cstdio>
#include <algorithm>
#include <vector>
using namespace std;

const int N = 10010;

vector<int> mul(vector<int>& a, int b) {
    int t = 0;
    vector<int> res;
    for (int i = 0; i < a.size() || t; i++) {
        if (i < a.size()) t = a[i] * b + t;
        res.push_back(t % 10);
        t /= 10;
    }
    while (res.size() > 1 && res.back() == 0) res.pop_back();
    return res;
}

vector<int> div(vector<int>& a, int b) {
    vector<int> res;
    int r = 0;
    for (int i = a.size() - 1; i >= 0; i--) {
        r = r * 10 + a[i];
        res.push_back(r / b);
        r %= b;
    }
    reverse(res.begin(), res.end());
    while (res.size() > 1 && res.back() == 0) res.pop_back();
    return res;
}

vector<int> build(int a) {
    vector<int> res;
    for (int t = a; t; t /= 10) {
        res.push_back(t % 10);
    }
    return res;
}

bool greater0(vector<int>& a, vector<int>& b) {
    if (a.size() > b.size()) return true;
    if (a.size() < b.size()) return false;

    for (int i = a.size() - 1; i >= 0; i--) {
        if (a[i] > b[i]) return true;
        else if (a[i] < b[i]) return false;
    }
    return false;
}

int n;
struct Pers {
    int a, b;

    bool operator<(const Pers& p) const {
        return a*b < p.a*p.b;
    }
} p[N];

int main() {
    scanf("%d", &n);
    for (int i = 0; i <= n; i++) {
        scanf("%d%d", &p[i].a, &p[i].b);
    }
    sort(p+1, p+1+n);

    vector<int> T = build(p[0].a);
    vector<int> ans = {0};
    for (int i = 1; i <= n; i++) {
        vector<int> cur = div(T, p[i].b);
        if (greater0(cur, ans)) ans = cur;
        T = mul(T, p[i].a);
    }
    for (int i = ans.size() - 1; i >= 0; i--) printf("%d", ans[i]);
    puts("");
    return 0;
}
```

## [HAOI2008] 糖果传递

