---
date: 2023-07-21 12:27:01
title: 图论 - SPFA 负环
katex: true
tags:
- Algo
- C++
- Graph
categories:
- OI
---

## SPFA 负环模板

> 给定一个 n 个点 m 条边的有向图，图中可能存在重边和自环， **边权可能为负数**。
>
> 请你判断图中是否存在负权回路。

建立一个虚拟原点，到所有点的距离都是 0，因此需要在初始化时把所有点都加入队列中，并且不用初始化距离。

由于 SPFA 算法只会把找到路径更小的节点加入到队列中，所以根据抽屉原理只要遍历一个节点的路径大于等于所有节点的数量，那么这条路径一定存在环，并且由于是 SPFA 算法找出来的环，所以它一定是负环。

```cpp
#include <bitset>
#include <cstring>
#include <iostream>
#include <queue>
using namespace std;

const int N = 2010, M=1e4+10;
// cnt 代表从原点到第i个节点经过的路程长度
int e[M], ne[M], h[N], idx, dist[N], w[M], cnt[N];
bool st[N];
queue<int> q;
int n, m;

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

bool spfa() {
    // 不需要初始化距离 因为这题不求绝对距离
    
    // 这里需要把所有点都添加到队列中
    // 因为负环可能处于从原点不可达的位置
    for (int i = 1; i <= n; i++) q.push(i), st[i] = true;
    
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = 0;
        for (int i = h[t]; i != -1; i = ne[i]) {
            int j = e[i];
            if (dist[j] > dist[t] + w[i]) {
                dist[j] = dist[t] + w[i];
                cnt[j] = cnt[t] + 1;
                if (cnt[j] >= n) return true;
                if (!st[j]) { st[j] = 1; q.push(j); }
            }
        }
    }
    return false;
}

int main() {
    memset(h, -1, sizeof h);
    cin >> n >> m;
    int a, b, c;
    while (m--) {
        cin >> a >> b >> c;
        add(a, b, c);
    }
    if (spfa()) cout << "Yes";
    else cout << "No";
    return 0;
}
```

## 01 分数规划 + 负环

> 给定一张 L 个点、P 条边的有向图，每个点都有一个权值 f[i]，每条边都有一个权值 t[i]。
>
> 求图中的一个环，使“环上各点的权值之和”除以“环上各边的权值之和”最大。
>
> 输出这个最大值。
>
> **注意**：数据保证至少存在一个环。
>
> 原题链接：[AcWing 361](https://www.acwing.com/problem/content//363/)。

这种求一个分数之比的最大值的 01分数规划 问题，通常使用二分解决。
$$
\begin{aligned}
p \lt \frac{\sum f_i}{\sum t_i}\\
p\sum t_i-\sum f_i&\lt 0\\
\sum (pt_i-f_i) &\lt 0
\end{aligned}
$$
换成大于号虽然这个式子同样成立，但是如果换成大于号，就是：
$$
p\gt \frac{\sum f_i}{\sum t_i}\\
\sum(f_i-pt_i)\lt 0
$$
这样求出来的就是最小值，因为每次二分判断的都是是否大于答案；而如果每次二分判断都是是否小于答案，求出来的就是最大值：因为图中可能有多个环。

根据题目给的数据范围，答案区间应当是 $(0, 1000]$，就用这个当做边界二分即可。

```cpp
#include <iostream>
#include <cstring>
#include <queue>
using namespace std;

const int N = 1010, M = 5010;
int h[N], e[M], ne[M], w[M], f[N], idx;
int n, m;
int cnt[N];
bool st[N];
double dist[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

bool spfa(double p) {
    queue<int> q;
    for (int i = 1; i <= n; i++)
        cnt[i] = 0, dist[i] = 0, st[i] = true, q.push(i);
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            double wi = p*w[i]-f[t];
            if (dist[j] > dist[t] + wi) {
                dist[j] = dist[t] + wi;
                cnt[j] = cnt[t] + 1;
                if (cnt[j] >= n) return true;
                if (!st[j]) st[j] = true, q.push(j);
            }
        }
    }
    return false;
}

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> f[i];
    memset(h, -1, sizeof h);
    while (m--) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c);
    }
    double l = 0, r = 1000;
    while (r - l > 1e-3) {
        double mid = (l+r) / 2;
        if (spfa(mid)) l = mid;
        else r = mid;
    }
    printf("%.2lf", l);
    return 0;
}
```

## 单词环

> 我们有 n 个字符串，每个字符串都是由 a∼z 的小写英文字母组成的。
>
> 如果字符串 A 的结尾两个字符刚好与字符串 B 的开头两个字符相匹配，那么我们称 A 与 B 能够相连（注意：A 能与 B 相连不代表 B 能与 A 相连）。
>
> 我们希望从给定的字符串中找出一些，使得它们首尾相连形成一个环串（一个串首尾相连也算），我们想要使这个环串的平均长度最大。
>
> 如下例：
>
> ```
> ababc
> bckjaca
> caahoynaab
> ```
>
> 第一个串能与第二个串相连，第二个串能与第三个串相连，第三个串能与第一个串相连，我们按照此顺序相连，便形成了一个环串，长度为 5+7+10=22（重复部分算两次），总共使用了 3 个串，所以平均长度是 $\cfrac{22}{3}=7.33$
>
> 原题链接：[AcWing 1165](https://www.acwing.com/problem/content/1167/)。

依然是一个 01 分数规划问题，和上题类似，还是推一下公式：
$$
p\lt \frac{\sum f_i}{\sum 1}\\
\sum(p-f_i)\lt 0
$$
但建图方式比较奇特，如果直接照搬题意硬建是不行的，下面是我一开始的做法（超时）：

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
#include <string>
#include <unordered_map>
#include <vector>
#include <queue>
using namespace std;

const int N = 1e5+10;

int n;
// 长度和后缀
vector<pair<int, string>> str;
// 前缀为 xx 的所有字符串编号
unordered_map<string, vector<int>> prefix;
bool st[N];
double dist[N];
int cnt[N];

bool spfa(double p) {
    queue<int> q;
    for (int i = 0; i < n; i++){
        q.push(i), st[i] = dist[i] = cnt[i] = 0;
    }
    
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        for (int j : prefix[str[t].second]) {
            double wi = p - (double)str[t].first;
            if (dist[j] > dist[t] + wi) {
                dist[j] = dist[t] + wi;
                cnt[j] = cnt[t] + 1;
                if (cnt[j] >= n) return true;
                if (!st[j]) st[j] = true, q.push(j);
            }
        }
    }
    return false;
}

int main() {
	char s[1001];
    while (scanf("%d", &n), n) {
        str.clear();
        prefix.clear();
        int sum = 0;
        for (int i = 0; i < n; i++) {
            scanf("%s", s);
            int len = strlen(s);
            str.push_back({len, s + len - 2});
            s[2] = 0;
            prefix[s].push_back(i);
            sum += len;
        }
        
        double l = 0, r = (double)sum / n;
        bool noans = true;
        while (r - l > 1e-3) {
            double mid = (l+r) / 2;
            if (spfa(mid)) l=mid, noans = false;
            else r=mid;
        }
        if (noans) puts("No solution");
        else printf("%.2lf\n", l);
    }
    return 0;
}
```

正确方法就是对每个字符串的前缀和后缀连一条边，边长为字符串长度，然后二分答案找负环。因为每个字符串只有前缀和后缀是有效的，所以这种做法是等价的。

但是，本题找负环的过程还很有可能 TLE，这里认为如果加入队列的次数大于 $3n$ 就存在负环（我这里保险用了 $5n$）

```cpp
#include <iostream>
#include <cstring>
#include <queue>
using namespace std;

const int N = 26*26+10, M = 1e5+10;

int n, m;
int h[N], e[M], ne[M], w[M], cnt[N], idx;
bool st[N];
int id[30][30];
double dist[N];
bool used[N];

void add(int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}

bool spfa(double p) {
    queue<int> q;
    memset(dist, 0, sizeof dist);
    memset(cnt, 0, sizeof cnt);
    memset(st, 0, sizeof st);
    
    for (int i = 0; i < n; i++) {
        if (used[i]) {
            q.push(i);
            st[i] = true;
        }
    }
    
    int times = 0;
    while (q.size()) {
        int t = q.front(); q.pop();
        st[t] = false;
        if (++times >= 5*n) return true;
        
        for (int i = h[t]; ~i; i = ne[i]) {
            int j = e[i];
            double wi = p - w[i];
            if (dist[j] > dist[t] + wi) {
                dist[j] = dist[t] + wi;
                cnt[j] = cnt[t] + 1;
                if (cnt[j] >= n) return true;
                if (!st[j]) st[j] = true, q.push(j);
            }
        }
    }
    return false;
}

int main() {
    char s[1001];
    // 打个表先
    for (char a = 'a'; a <= 'z'; a++)
        for (char b = 'a'; b <= 'z'; b++)
            id[a-'a'][b-'a'] = ++n;
    
    while (scanf("%d", &m), m) {
        memset(h, -1, sizeof h);
        memset(used, 0, sizeof used);
        idx = 0;
        for (int i = 0; i < m; i++) {
            scanf("%s", s);
            int len = strlen(s);
            int a = id[s[0]-'a'][s[1]-'a'], b = id[s[len-2]-'a'][s[len-1]-'a'];
            add(a, b, len);
            used[a] = used[b] = true;
        }
        // 答案应该是 0 ~ 1000
        double l = 0, r = 1001;
        bool noans = true;
        while (r - l > 1e-3) {
            double mid = (l+r) / 2;
            if (spfa(mid)) l=mid, noans = false;
            else r=mid;
        }
        if (noans) puts("No solution");
        else printf("%.2lf\n", l);
    }
    return 0;
}
```

或者也可以用栈代替队列，但是这比我们上面的方法慢得多。

