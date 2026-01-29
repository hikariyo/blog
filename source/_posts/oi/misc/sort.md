---
date: 2023-09-08 09:11:00
updated: 2023-09-08 09:11:00
title: 杂项 - 排序
katex: true
tags:
- Algo
- C++
categories:
- OI
---

## 归并排序求逆序对

由于我们得到的 $q[l\sim mid],q[mid+1\sim r]$ 都是升序排列的，因此只要出现 $q[i]>q[j]$ 就说明 $i\sim mid$ 都与 $q[j]$ 构成逆序。

```cpp
#include <cstdio>
using namespace std;

typedef long long ll;

const int N = 1e6+10;
int q[N], tmp[N];

ll mergesort(int l, int r) {
    if (l >= r) return 0;
    
    int mid = l + r >> 1;
    ll res = mergesort(l, mid) + mergesort(mid+1, r);
    
    int i = l, j = mid+1;
    int k = 0;
    while (i <= mid && j <= r) {
        if (q[i] <= q[j]) tmp[k++] = q[i++];
        else {
            tmp[k++] = q[j++];
            res += mid-i+1;
        }
    }
    
    while (i <= mid) tmp[k++] = q[i++];
    while (j <= r) tmp[k++] = q[j++];
    
    int t1 = l, t2 = 0;
    while (t1 <= r) q[t1++] = tmp[t2++];
    
    return res;
}

int main() {
    int n;
    scanf("%d", &n);
    for (int i = 0; i < n; i++) scanf("%d", q+i);
    printf("%lld\n", mergesort(0, n-1));
    return 0;
}
```

## Eastern Exhibition

> 大意：给出二维平面上 n 个点，找出能使得所有点到它的曼哈顿距离之和最小的点的个数。
>
> 找出的点可以不在这 n 个点中。
>
> 题目链接：[CF1486B](https://codeforces.com/problemset/problem/1486/B)。

这个问题和下面要说的环形均分有关，所以先放上来，都是对于下面这个式子的最小值进行讨论：
$$
|a_1-x|+|a_2-x|+\cdots+|a_n-x|
$$
首先给数组排序，分 $n$ 的奇偶讨论。

+ 奇数， $x=a_{(n+1)/2}$ 时原式最小。

    如果不选 $a_{(n+1)/2}$ 的话，答案都会多出来 $k|x-a_{(n+1)/2}|$，这里的 $k$ 取决于存在几个重复的 $a_{(n+1)/2}$

+ 偶数，$x=a_{n/2}\sim a_{n/2+1}$ 之间任意一点都取最小。

    证明同上。

注意这里是不需要去重的，证明过程没有说两个点不能重合。

回到本题，由于 $x,y$ 坐标相互独立，所以分别求然后用乘法原理算出最终结果。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 1010;
int n, x[N], y[N];

int main() {
    int T; scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        for (int i = 0; i < n; i++) scanf("%d%d", &x[i], &y[i]);
        sort(x, x+n), sort(y, y+n);

        int nx, ny;
        if (n & 1) nx = ny = 1;
        else nx = x[n/2] - x[n/2-1] + 1, ny = y[n/2] - y[n/2-1] + 1;

        printf("%lld\n", (long long)nx*ny);
    }
    return 0;
}
```



## [NOIP2002] 均分纸牌

> 有 $N$ 堆纸牌，编号分别为 $1,2,\ldots,N$。每堆上有若干张，但纸牌总数必为 $N$ 的倍数。可以在任一堆上取若干张纸牌，然后移动。
>
> 移牌规则为：在编号为 $1$ 堆上取的纸牌，只能移到编号为 $2$ 的堆上；在编号为 $N$ 的堆上取的纸牌，只能移到编号为 $N-1$ 的堆上；其他堆上取的纸牌，可以移到相邻左边或右边的堆上。
>
> 现在要求找出一种移动方法，用最少的移动次数使每堆上纸牌数都一样多。
>
> 题目链接：[P1031](https://www.luogu.com.cn/problem/P1031)。

第一堆只能从第二堆转移，完成后第一堆就不会被再动了，然后第二堆只能从第三堆转移，等等。

如果某堆本来就是平均数，就不用动了。

```cpp
#include <cstdio>
#include <cmath>
using namespace std;

const int N = 110;
int n, a[N];

int main() {
    int avg = 0;
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]), avg += a[i];
    avg /= n;
    
    int res = 0;
    for (int i = 1; i <= n-1; i++)
        if (a[i] != avg) a[i+1] -= avg - a[i], res++;
    printf("%d\n", res);
    return 0;
}
```

## [HAOI2008] 糖果传递

> 有 $n$ 个小朋友坐成一圈，每人有 $a_i$ 个糖果。每人只能给左右两人传递糖果。每人每次传递一个糖果代价为 $1$。求使所有人获得均等糖果的最小代价。
>
> 题目链接：[P2512](https://www.luogu.com.cn/problem/P2512)。

假设 $x_i$ 是 $a_i \to a_{i+1}$ 的糖果数量，它可以是负的，其中 $x_n$ 表示 $a_n \to a_1$，设 $\overline a$ 是糖果的平均值。

我们要求的是下面这个式子的最小值：
$$
|x_1|+|x_2|+\cdots+|x_n|
$$
由形如 $a_i -x_i + x_{i-1}=\overline a$ 的方程移项后列出下面的方程组：
$$
\left\{\begin{aligned}
&x_n-x_{n-1}=a_n-\overline a\\
&x_{n-1}-x_{n-2}=a_{n-1}-\overline a\\
&\cdots\\
&x_3-x_2=a_3 -\overline a\\
&x_2-x_1=a_2- \overline a\\
&x_1-x_n=a_1 - \overline a
\end{aligned}\right.
$$
这个时候想要转化为类似上道题绝对值不等式的式子，可以从这个方程组入手。
$$
\left\{\begin{aligned}
&x_1=x_1\\
&x_2=x_1-(\overline a-a_2)\\
&x_3=x_1-(2\overline a-a_2-a_3)\\
\end{aligned}\right.
$$
然后就可以推导出下面的通项公式：
$$
x_n=x_1-[(n-1)\overline a -\sum_{i=2}^na_i]
$$
所以构造出后面的数组然后选出中位数直接求即可。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 1000010;
int n;
LL s[N], b[N];

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%lld", &s[i]), s[i] += s[i-1];
    LL avg = a[n] / n;
    
    for (int i = 1; i <= n; i++) b[i] = (i-1) * avg - (s[i] - s[1]);
    sort(b+1, b+1+n);
    
    LL base = b[n/2+1], res = 0;
    for (int i = 1; i <= n; i++) res += abs(b[i] - base);
    printf("%lld\n", res);
    
    return 0;
}
```

## 七夕祭

> 给 $N\times M$ 的网格和 $T$ 个点，求能否通过交换相邻两个点（第一行和最后一行，第一列和最后一列也算相邻）使得给出的 $T$ 个点在各行中一样多，各列中一样多。
>
> 如果通过调整只能使得各行一样多，输出 row；
>
> 如果只能使各列一样多，输出 column；
>
> 如果均不能满足，输出 impossible。
>
> 如果输出的字符串不是 impossible， 接下来输出最小交换次数，与字符串之间用一个空格隔开。
>
> 题目链接：[AcWing 107](https://www.acwing.com/problem/content/107/)。

可以看出左右交换只影响列之间的相对关系，上下交换只影响行之间的相对关系，所以横纵做两遍环形均分纸牌即可。

可以用 `nth_element` 求一个区间第 $k$ 大数，它的效果与快排类似，就是先按照快排任选一个位置作为基准，然后部分排序，之后如果 $k$ 在左边就在左边递归处理，否则在右边递归处理，它的最常用签名如下。

```cpp
template<class RandomIt>
void nth_element(RandomIt first, RandomIt nth, RandomIt last);
```

接下来是代码。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 100010;
int n, m, t;
int row[N], col[N];

int solve(int a[], int n) {
    static int s[N], b[N];
    for (int i = 1; i <= n; i++) s[i] = s[i-1] + a[i];
    int avg = t / n ;
    
    for (int i = 1; i <= n; i++) b[i] = (i-1) * avg - (s[i] - s[1]);
    nth_element(b+1, b+n/2+1, b+1+n);
    int base = b[n/2+1], res = 0;
    
    for (int i = 1; i <= n; i++) res += abs(base - b[i]);
    return res;
}

int main() {
    scanf("%d%d%d", &n, &m, &t);
    for (int i = 0; i < t; i++) {
        int x, y;
        scanf("%d%d", &x, &y);
        row[x]++, col[y]++;
    }
    if (t % n && t % m) puts("impossible");
    else {
        long long res = 0;
        if (t % n == 0 && t % m == 0) printf("both");
        else if (t % n == 0) printf("row");
        else printf("column");
        
        if (t % m == 0) res += solve(col, m);
        if (t % n == 0) res += solve(row, n);
        printf(" %lld\n", res);
    }
    return 0;
}
```

## 中位数

> 给定一个长度为 $N$ 的非负整数序列 $A$，对于前奇数项求中位数。
>
> 题目链接：[P1168](https://www.luogu.com.cn/problem/P1168)。

题目本身并不难，这里主要是介绍对顶堆的思想。

对于小于等于当前中位数的数字，将其放在下面的堆中，否则放在上面的堆中。之后，我们需要保证 `size(down) = size(up)` 或者 `size(down) = size(up)+1`， 这样在长度为奇数时下面堆的堆顶就是中位数了。

```cpp
#include <queue>
#include <cstdio>
using namespace std;

priority_queue<int> down;
priority_queue<int, vector<int>, greater<int>> up;
int n;

void insert(int v) {
    if (down.size() == 0 || v < down.top()) down.push(v);
    else up.push(v);

    if (down.size() < up.size()) {
        down.push(up.top());
        up.pop();
    }
    else if (down.size() > up.size() + 1) {
        up.push(down.top());
        down.pop();
    }
}

int get() {
    return down.top();
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        int v; scanf("%d", &v);
        insert(v);
        if (i & 1) printf("%d\n", get());
    }
    return 0;
}
```

对于这种动态维护有序序列，当然可以用平衡树做了。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 100010;
struct Node {
    int s[2], p, v;
    int size, cnt;

    void init(int p0, int v0) {
        p = p0, v = v0;
        s[0] = s[1] = 0;
        size = cnt = 1;
    }
} tr[N];
int n, root, idx;

void pushup(int u) {
    tr[u].size = tr[tr[u].s[0]].size + tr[tr[u].s[1]].size + tr[u].cnt;
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

void insert(int v) {
    int u = root, p = 0;
    while (u && tr[u].v != v) {
        p = u, u = tr[u].s[v > tr[u].v];
    }
    if (u) tr[u].cnt++;
    else {
        u = ++idx;
        tr[u].init(p, v);
        if (p) tr[p].s[v > tr[p].v] = u;
    }
    splay(u, 0);
}

int get() {
    int k = (tr[root].size+1) / 2, u = root;

    while (u) {
        if (k <= tr[tr[u].s[0]].size) u = tr[u].s[0];
        else if (k <= tr[tr[u].s[0]].size + tr[u].cnt) {
            splay(u, 0);
            return tr[u].v;
        }
        else k -= tr[tr[u].s[0]].size + tr[u].cnt, u = tr[u].s[1];
    }

    return -1;
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        int v; scanf("%d", &v);
        insert(v);
        if (i & 1) {
            printf("%d\n", get());
        }
    }
    return 0;
}
```

## [POJ2893] M × N Puzzle

> 给定 $M\times N$ 数码，判断是否有解。
>
> 题目链接：[POJ2893](http://poj.org/problem?id=2893)。

之前学习过八数码问题，我们知道这个问题和逆序对的数量有关，给定的最终状态的逆序对数量为 $0$，在 $M\times N$ 数码中，也是按照相似的套路分析。

+ 空格的左右移动并不会改变逆序对的数量。
+ 空格的上下移动会改变 $N-1$ 个逆序对的数量。

如果 $N$ 为奇数，那么如何移动都不会改变序列的奇偶性，只需要统计逆序对的数量判断是否为偶数即可。

如果 $N$ 为偶数，设空格的起始坐标为 $(x,y)$ 并且是从 $(0, 0)$ 开始的坐标，那么按照二维数组的坐标轴，应该要移动 $m-x-1$ 列，也就是改变 $m-x-1$ 次奇偶性。那么如果 $m-x-1$ 是奇数，初始逆序对数量就该是奇数，反之亦然。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 1000010;
int n, m, p, x, y, a[N];

LL count(int l, int r) {
    static int tmp[N];
    if (l >= r) return 0;

    int mid = (l+r) >> 1, k = 0;
    LL res = count(l, mid) + count(mid+1, r);

    int i = l, j = mid+1;
    while (i <= mid && j <= r) {
        if (a[i] <= a[j]) tmp[k++] = a[i++];
        else tmp[k++] = a[j++], res += mid-i+1;
    }
    while (i <= mid) tmp[k++] = a[i++];
    while (j <= r) tmp[k++] = a[j++], res += mid-i+1;

    for (k = 0, i = l; i <= r; i++, k++) a[i] = tmp[k];
    return res;
}

bool check() {
    if (n & 1) return count(0, p-1) % 2 == 0;
    // 同余方程移项
    else return (count(0, p-1) - (m - x - 1)) % 2 == 0;
}

int main() {
    while (scanf("%d%d", &m, &n), m || n) {
        p = 0;
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++) {
                scanf("%d", &a[p]);
                if (a[p]) p++;
                else x = i, y = j;
            }
        printf("%s\n", check() ? "YES" : "NO");
    }
    return 0;
}
```

