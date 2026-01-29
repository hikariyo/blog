---
date: 2023-07-16 11:38:22
updated: 2023-07-16 23:59:22
title: 搜索 - DFS
katex: true
tags:
- Algo
- C++
- Search
categories:
- OI
---

## 简介

DFS(Deep First Search)是遍历方式的一种，它可以用来求排列组合方式的这种遍历问题，实现起来一般使用递归。

## 剪枝

剪枝策略：

1. 优化搜索顺序

    应该首先搜索分支较少的节点，使得它可以被剪掉尽可能多的节点。

2. 排除等效冗余

    如果不考虑顺序，尽量用组合的方式搜索以减少冗余。

3. 可行性剪枝

4. 最优性剪枝

5. 记忆化搜索（DP）

### N皇后

> n 皇后问题是指将 n 个皇后放在 n×n 的国际象棋棋盘上，使得皇后不能相互攻击到，即任意两个皇后都不能处于同一行、同一列或同一斜线上。
>
> 现在给定整数 n，请你输出所有的满足条件的棋子摆法，`.` 代表棋盘 `Q` 代表皇后。
>
> 详细要求参考 [原题链接](https://www.acwing.com/problem/content/1623/)。

记录的内容由上一问的 `visited` 变成了三个内容：逐行遍历，同一列是否放过，同一正对角线是否放过，同一反对角线是否放过。因此，并不需要知道到底是哪个位置放了皇后，只要同一个对角线上放过就可以判断这个位置不可以放，所以用对角线的**截距加上偏移量**表示这个对角线，避免负数。

对于正对角线 $y=-x+b$ 和反对角线 $y=x+b$ （这里沿用矩阵中正反对角线的定义），我们只需要知道一个 `x, y` 坐标就能求出来这个截距，表示这个对角线。特殊地，对于反对角线，让截距加上 `n` 来确保它是正数。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 20;
bitset<N> dg, udg, cols;
char b[N][N];
int n;

// u代表当前行
void dfs(int u) {
    if (u == n) {
        for (int i = 0; i < n; i++) puts(b[i]);
        puts("");
        return;
    }
    
    // 遍历 n 列
    for (int i = 0; i < n; i++)
        if (!cols[i] && !dg[u+i] && !udg[u-i+n]) {
            b[u][i] = 'Q';
            cols[i] = dg[u+i] = udg[u-i+n] = 1;
            dfs(u+1);
            cols[i] = dg[u+i] = udg[u-i+n] = 0;
            b[u][i] = '.';
        }
}

int main() {
    scanf("%d", &n);
    for (int i = 0; i < n; i++) for (int j = 0; j < n; j++) b[i][j] = '.';
    dfs(0);
    return 0;
}
```

### 互质组

> 给定 n 个正整数，将它们分组，使得每组中任意两个数互质。
>
> 至少要分成多少个组？
>
> 详细要求参考 [原题链接](https://www.acwing.com/problem/content/1120/)。

这里搜索策略就是将能添加到当前组中的元素添加到当前组中，如果不能添加再加一个新的组。

这个可以简单证明：如果一个数字既可以添加到当前组，也可以添加到新的组，那么它添加到当前组并不改变最优解中的组的个数，因此可以这样优化。

```cpp
#include <iostream>
using namespace std;

const int N = 11;
int n, group[N][N], a[N], ans;
bool st[N];

int gcd(int a, int b) {
    return a ? gcd(b%a, a) : b;
}

bool check(int g, int gc, int x) {
    for (int i = 0; i < gc; i++) {
        if (gcd(group[g][i], x) != 1) return false;
    }
    return true;
}

// 当前组 组中数字的数量 遍历过数字的数量 开始位置
void dfs(int g, int gc, int nc, int start) {
    // 最优性剪枝
    if (g >= ans) return;
    if (nc == n) {
        ans = g;
        return;
    }
    
    bool new_group = true;
    // 这里从 start 开始就是排除等效冗余的思想
    for (int i = start; i < n; i++) {
        if (!st[i] && check(g, gc, a[i])) {
            new_group = false;
            st[i] = true;
            group[g][gc] = a[i];
            dfs(g, gc+1, nc+1, i+1);
            group[g][gc] = 0;
            st[i] = false;
        }
    }
    
    if (new_group) dfs(g+1, 0, nc, 0);
}

int main() {
    cin >> n;
    ans = n;
    for (int i = 0; i < n; i++) cin >> a[i];
    dfs(1, 0, 0, 0);
    cout << ans;
    return 0;
}
```

上面的做法是对每个组放哪个数字进行选择，下面对每个数字放到不同组进行选择，显然后者的分支在剪枝的情况下要少一些，因此运行效率更快。

```cpp
#include <iostream>
using namespace std;

const int N = 11;
int n, a[N], ans;
int group[N][N], cnt[N];

int gcd(int a, int b) {
    return a ? gcd(b%a, a) : b;
}

bool check(int g, int x) {
    for (int i = 0; i < cnt[g]; i++) {
        if (gcd(group[g][i], x) != 1) return false;
    }
    return true;
}

void dfs(int u, int k) {
    if (k >= ans) return;
    if (u == n) {
        ans = k;
        return;
    }
    
    for (int i = 0; i < k; i++) {
        if (check(i, a[u])) {
            group[i][cnt[i]++] = a[u];
            dfs(u+1, k);
            cnt[i]--;
        }
    }
    
    group[k][cnt[k]++] = a[u];
    dfs(u+1,k+1);
    cnt[k]--;
}

int main() {
    cin >> n;
    ans = n;
    for(int i = 0; i < n; i++) cin >> a[i];
    dfs(0, 0);
    cout << ans << endl;
    return 0;
}
```

### 小猫爬山

>索道上的缆车最大承重量为 W，而 N 只小猫的重量分别是 C1、C2……CN。
>
>当然，每辆缆车上的小猫的重量之和不能超过 W。
>
>求最少用多少辆缆车可以把所有小猫送下山。
>
>详细要求参考 [原题链接](https://www.acwing.com/problem/content/167/)。

枚举每只猫，都有 2 种分支，分别是将其放到现有的车中或新开一辆。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 20;
int car[N], n, m, w[N], ans;

void dfs(int u, int k) {
    if (k >= ans) return;
    if (u == n) {
        ans = k;
        return;
    }
    
    for (int i = 0; i < k; i++) {
        if (w[u] + car[i] <= m) {
            car[i] += w[u];
            dfs(u+1, k);
            car[i] -= w[u];
        }
    }
    
    // 新开一个组放进去
    car[k] = w[u];
    dfs(u+1, k+1);
    car[k] = 0;
}

int main() {
    cin >> n >> m;
    ans = n;
    for (int i = 0; i < n; i++) cin >> w[i];
    // 优化搜索顺序
    sort(w, w+n, greater<int>());
    dfs(0, 0);
    cout << ans;
    return 0;
}
```

### 数独

> 参考 [原题链接](https://www.acwing.com/problem/content/168/)。

本题除了剪枝优化还有一些位运算优化策略。

+ 搜索顺序优化。先搜索能填数字少的点，然后拓展到多的点。
+ 可行性剪枝。
+ 用二进制表示一行中能填的数字，这样每个点能填的数就是当前行 & 当前列 & 九宫格。
+ 用 `lowbit` 获取状态中可以填的数字对应的二进制位，然后对取出来的数取 `log2` 就是这个数字，这一步可以打表。

```cpp
#include <iostream>
using namespace std;

const int N = 9, M = 1 << N;
int logtwo[M], ones[M], block[3][3], row[N], col[N];
char str[N*N+1];

void init() {
    // 所有状态都是九个数字都能放
    for (int i = 0; i < N; i++) row[i] = col[i] = M-1;
    for (int i = 0; i < 3; i++)
        for (int j = 0; j < 3; j++)
            block[i][j] = M-1;
}

void put(int x, int y, int t, bool clear) {
    str[x*N+y] = clear ? '.' : t+'1';
    int v = 1 << t;
    if (clear) v = -v;

    // 不可以再放 t
    row[x] -= v;
    col[y] -= v;
    block[x/3][y/3] -= v;
}

#define state(x, y) (row[x] & col[y] & block[x/3][y/3])
#define lowbit(x) (x & -x)

bool dfs(int n) {
    if (n == 0) return true;

    // 寻找能放数字最少的点
    int minv = 10, x = -1, y = -1;
    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++)
            if (str[i*9+j] == '.' && ones[state(i, j)] < minv) {
                minv = ones[state(i, j)];
                x = i, y = j;
            }
    
    // 去掉最后一位 lowbit 用 i -= lowbit(i)
    for (int i = state(x, y); i; i -= lowbit(i)) {
        int j = logtwo[lowbit(i)];
        put(x, y, j, false);
        if (dfs(n-1))
            return true;
        put(x, y, j, true);
    }

    return false;
}

int main() {
    // 打表
    for (int i = 0; i < N; i++) logtwo[1 << i] = i;
    for (int i = 0; i < M; i++)
        for (int j = i; j; j -= lowbit(j))
            ones[i]++;
    
    while (cin >> str, str[0] != 'e') {
        init();

        int cnt = 0;
        for (int i = 0; i < N; i++)
            for (int j = 0; j < N; j++)
                if (str[i*9+j] == '.') cnt++;
                else put(i, j, str[i*9+j]-'1', false);
        
        dfs(cnt);
        cout << str << '\n';
    }

    return 0;
}
```

### 生日蛋糕

> 要制作一个体积为 Nπ 的 M 层生日蛋糕，每层都是一个圆柱体。
>
> 设从下往上数第 i 层蛋糕是半径为 Ri，高度为 Hi 的圆柱。
>
> 要求每层的半径和高度都严格大于上一层。
>
> 由于要在蛋糕上抹奶油，为尽可能节约经费，我们希望蛋糕外表面（最下一层的下底面除外）的面积 Q 最小。
>
> 令 Q=Sπ ，请编程对给出的 N 和 M，找出蛋糕的制作方案（适当的 Ri 和 Hi 的值），使 S 最小。
>
> 除 Q 外，以上所有数据皆为正整数。
>
> 更多详细要求参考 [原题链接](https://www.acwing.com/problem/content/170/)。

设当前为第 $u$ 层，前面不包含第 $u$ 层的体积，表面积分别是 $v, s$ 剪枝策略如下：

1. 优化搜索顺序：

    1. 自底向上搜索，使得体积先大后小。
    2. 先枚举半径，再枚举高度，从大到小枚举，因为半径对体积的影响是二次方。

2. 对于 $R_u$ 可以由体积和半径本身的要求推出它的范围。
    $$
    \begin{aligned}
    n-v&\ge R_u^2H_u\ge R_u^2\\
    u&\le R_u \lt R_{u+1}\\
    \therefore u&\le R_u \le \min\{R_{u+1}-1, \sqrt{n-v}\}
    \end{aligned}
    $$

3. 对于 $H_u$ 也是由上面的公式推导。
    $$
    \begin{aligned}
    n-v&\ge R_u^2H_u\ge H_u\\
    u&\le H_u \lt H_{u+1}\\
    \therefore u &\le H_u \le \min\{H_{u+1}-1, \frac{n-v}{R_u^2}\}
    \end{aligned}
    $$

4. 可以预处理出每层的最小体积 $\text{minv}(u)$ 和表面积 $\text{mins}(u)$。
    $$
    \begin{aligned}
    \text{minv}(u) &= u^3+\text{minv}(u-1)\\
    \text{mins}(u) &= 2u^2+\text{mins}(u-1)\\
    \end{aligned}
    $$
    意思就是第 $u$ 层最小半径和高度都为 $u$，可以用以下两不等式剪枝。
    $$
    \begin{aligned}
    v+\text{minv}(u) &\le n\\
    s+\text{mins}(u) &\lt \text{ans}\\
    \end{aligned}
    $$
    第一个是可行性剪枝，第二个是最优性剪枝。

5. 这个剪枝较困难，考虑 $1\sim u$ 的表面积和体积 $S=\text{ans}-s, V=n-v$ 本身满足何不等关系。
    $$
    \begin{aligned}
    S&=\sum_{i=1}^u 2R_iH_i\\
    V&=\sum_{i=1}^u R_i^2H_i\\
    \therefore R_{u+1} S &= 2\sum_{i=1}^uR_iH_iR_{u+1} \\
    &\gt 2\sum_{i=1}^uR_i^2H_i\\
    &=2V\\
    \therefore \text{ans} &\gt s+\frac{2(n-v)}{R_{u+1}}
    \end{aligned}
    $$
    两边乘 $R_u$ 理论也是可行，但实际上你在本层 DFS 时不知道 $R_u$ 是多少，但是知道 $R_{u+1}$ 的值，因此这里两边乘 $R_{u+1}$，当然你也可以在枚举方案的时候看 $R_u$ 是否可行。

```cpp
#include <iostream>
#include <algorithm>
#include <cmath>
using namespace std;

const int M = 21;
int n, m, R[M], H[M], minv[M], mins[M];
int ans = 2e9;

void dfs(int u, int s, int v) {
    if (v + minv[u] > n) return;
    if (s + mins[u] >= ans) return;
    if (s + 2*(n-v)/R[u+1] >= ans) return;
    
    if (u == 0) {
        // 体积必须刚好用完
        if (v == n) ans = s;
        return;
    }
    
    for (int r = min(R[u+1]-1, (int)sqrt(n-v)); r >= u; r--) {
        // 如果想用 Ru 剪枝 就在这里
        // if (ans < s + 2*(n-v)/r) continue;
        for (int h = min(H[u+1]-1, (n-v) / r / r); h >= u; h--) {
            R[u] = r, H[u] = h;
            int extra = 0;
            if (u == m) extra = r*r;
            dfs(u-1, s+extra+2*r*h, v+r*r*h);
            R[u] = H[u] = 0;
        }
    }
}


int main() {
    cin >> n >> m;
    
    for (int i = 1; i <= m; i++) {
        minv[i] = i*i*i + minv[i-1];
        mins[i] = 2*i*i + mins[i-1];
    }
    R[m+1] = H[m+1] = 2e9;
    
    dfs(m, 0, 0);
    if (ans == 2e9) ans = 0;
    cout << ans << '\n';
    
    return 0;
}
```

### 斗地主

> NOIP 2015，题目过长，不抄。
>
> 链接：[LOJ 2422](https://loj.ac/p/2422)。

一道不错的模拟 + 搜索题。

按照直觉来说，应该是先把多的出了，然后出散的。

然后对于枚举完多的之后（顺子，三带，四带这些），下面就是本题优化的关键：我们并不需要去 DFS 出散牌，反正上面该搜的都搜完了，这里直接把剩下的所有牌当成散牌出掉，只需要一个循环就可以。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 20;
int cnt[N];
int T, n, ans = 23;

#define DFS dfs(u+1); if (u >= ans) return

void dfs(int u) {
    if (u >= ans) return;

    // 先搜单顺
    int k = 0;
    for (int i = 3; i <= 14; i++) {
        if (cnt[i] == 0) k = 0;
        else k++;

        if (k >= 5) {
            for (int j = i; j >= i-k+1; j--) cnt[j]--;
            DFS;
            for (int j = i; j >= i-k+1; j--) cnt[j]++;
        }
    }

    // 再搜双顺
    k = 0;
    for (int i = 3; i <= 14; i++) {
        if (cnt[i] <= 1) k = 0;
        else k++;

        if (k >= 3) {
            for (int j = i; j >= i-k+1; j--) cnt[j] -= 2;
            DFS;
            for (int j = i; j >= i-k+1; j--) cnt[j] += 2;
        }
    }

    // 搜三顺
    k = 0;
    for (int i = 3; i <= 14; i++) {
        if (cnt[i] <= 2) k = 0;
        else k++;

        if (k >= 2) {
            for (int j = i; j >= i-k+1; j--) cnt[j] -= 3;
            DFS;
            for (int j = i; j >= i-k+1; j--) cnt[j] += 3;
        }
    }

    // 三带
    for (int i = 2; i <= 14; i++) {
        if (cnt[i] < 3) continue;
        
        // 带一对
        for (int j = 2; j <= 14; j++) {
            if (j == i || cnt[j] < 2) continue;

            cnt[i] -= 3;
            cnt[j] -= 2;
            DFS;
            cnt[j] += 2;
            cnt[i] += 3;
        }

        // 带单张
        for (int j = 2; j <= 16; j++) {
            if (j == i || cnt[j] < 1) continue;

            cnt[i] -= 3;
            cnt[j] --;
            DFS;
            cnt[j] ++;
            cnt[i] += 3;
        }
    }   

    // 四带
    for (int i = 2; i <= 14; i++) {
        if (cnt[i] < 4) continue;

        // 带两对
        for (int j = 2; j <= 14; j++) {
            if (j == i || cnt[j] < 2) continue;
            for (int k = 2; k <= 14; k++) {
                if (k == i || k == j || cnt[k] < 2) continue;

                cnt[i] -= 4;
                cnt[k] -= 2;
                cnt[j] -= 2;

                DFS;

                cnt[j] += 2;
                cnt[k] += 2;
                cnt[i] += 4;
            }
        }

        // 带一对
        for (int j = 2; j <= 14; j++) {
            if (j == i || cnt[j] < 2) continue;
            cnt[i] -= 4;
            cnt[j] -= 2;
            DFS;
            cnt[j] += 2;
            cnt[i] += 4;
        }

        // 带两单张
        for (int j = 2; j <= 16; j++) {
            if (cnt[j] < 1 || j == i) continue;
            for (int k = 2; k <= 16; k++) {
                if (k == i || k == j || cnt[k] < 1) continue;
                cnt[j]--;
                cnt[k]--;
                cnt[i] -= 4;
                DFS;
                cnt[k]++;
                cnt[j]++;
                cnt[i] += 4;
            }
        }
    }

    // JOKER
    if (cnt[15] && cnt[16]) {
        cnt[15]--;
        cnt[16]--;
        DFS;
        cnt[15]++;
        cnt[16]++;
    }

    // 散牌
    for (int i = 2; i <= 16; i++) if (cnt[i]) u++;
    ans = min(ans, u);
}

int main() {
    scanf("%d%d", &T, &n);
    while (T--) {
        ans = 25;
        memset(cnt, 0, sizeof cnt);
        for (int i = 0; i < n; i++) {
            int a, b;
            scanf("%d%d", &a, &b);
            // a: 2 ~ 16
            if (a == 1) a = 14;
            if (a == 0 && b == 1) a = 15;
            if (a == 0 && b == 2) a = 16;
            cnt[a]++;
        }
        dfs(0);
        printf("%d\n", ans);
    }
    return 0;
}
```

### 靶形数独

> 题目需要配图食用，这里不抄了。
>
> 链接：[P1074](https://www.luogu.com.cn/problem/P1074)。

和普通数独类似，搜索顺序还是从能填的少的开始搜，不过这里要考虑多解的情况。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 9;
int ones[1<<N], num[1<<N];
int ans = -1;
int g[N][N], row[N], col[N], block[3][3];

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

#define lowbit(x) ((x)&(-x))

int countones(int x) {
    int res = 0;
    while (x) {
        res++;
        x -= lowbit(x);
    }
    return res;
}

void put(int x, int y, int c, bool unset=false) {
    int mask = 1 << (c-1);
    if (unset) g[x][y] = 0;
    else g[x][y] = c, mask *= -1;

    row[x] += mask;
    col[y] += mask;
    block[x/3][y/3] += mask;
}

int state(int x, int y) {
    return row[x] & col[y] & block[x/3][y/3];
}

void dfs() {
    // 能填的数量最小的
    int x = -1, y = -1;
    for (int i = 0; i < 9; i++)
        for (int j = 0; j < 9; j++)
            if (!g[i][j] && (x == -1 || ones[state(i, j)] < ones[state(x, y)]))
                x = i, y = j;
    
    if (x == -1) {
        int t = 0;
        for (int i = 0; i < 9; i++)
            for (int j = 0; j < 9; j++)
                t += score[i][j] * g[i][j];
        ans = max(ans, t);
        return;
    }
    
    int s = state(x, y);
    for (int i = 9; i >= 1; i--) {
        if (s >> (i-1) & 1) {
            put(x, y,i);
            dfs();
            put(x, y, i, true);
        }
    }
}

int main() {
    for (int i = 0; i < 1 << 9; i++)
        ones[i] = countones(i);
    for (int i = 0; i < 9; i++)
        num[1<<i] = i+1;
    // init
    for (int i = 0; i < 9; i++)
        row[i] = col[i] = block[i%3][i/3] = (1 << 9) - 1;

    for (int i = 0; i < 9; i++)
        for (int j = 0; j < 9; j++) {
            int v;
            scanf("%d", &v);
            if (v) put(i, j, v);
        }
    dfs();
    printf("%d\n", ans);
    return 0;
}
```

### CCC单词搜索

> 给定一个 R×C 的大写字母矩阵。
>
> 请你在其中寻找目标单词 W。
>
> 已知，目标单词 W 由若干个**不同的**大写字母构成。
>
> 目标单词可以遵循以下两种规则，出现在矩阵的水平、垂直或斜 45 度线段中：
>
> - 单词出现在一条线段上。
> - 单词出现在两条相互垂直且存在公共端点的线段上。也就是说，单词首先出现在某线段上，直到某个字母后，转向 90 度，其余部分出现在另一条线段上。
>
> 请你计算，目标单词在给定矩阵中一共出现了多少次。
>
> 题目链接：[AcWing 5168](https://www.acwing.com/problem/content/5168/)。

设索引从向量 $(0,-1)$ 开始为 $0$，一个向量顺时针旋转 $\pi /2$ 和 $3\pi /2$ 后的向量，索引有如下特点，例如：$1\to 3,7; 2\to 4,0$

换成二进制：$001_2 \to 011_2, 111_2; 010_2 \to 100_2,000_2$

一个是只改变第二位，一个是第二位和最高位都改变，所以分别异或 $2$ 和 $4$ 即可，这里可以通过 `for` 循环写到一起。

```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 110;
char g[N][N], s[10];
int n, m, L;

int ans = 0;

int dx[8] = {-1, -1, 0, 1, 1, 1, 0, -1};
int dy[8] = {0, 1, 1, 1, 0, -1, -1, -1};


void dfs(int x, int y, int u, int d, int cnt) {
    if (x < 0 || x >= n || y < 0 || y >= m || g[x][y] != s[u]) return;
    if (u == L-1) {
        ans++;
        return;
    }
    
    dfs(x+dx[d], y+dy[d], u+1, d, cnt);
    if (u && !cnt)
        for (int i = 0; i < 2; i++)
            dfs(x, y, u, d^2^(i<<2), cnt+1);
    
}

int main() {
    cin >> s >> n >> m;
    L = strlen(s);
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cin >> g[i][j];
        }
    }
    
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            for (int k = 0; k < 8; k++)
                dfs(i, j, 0, k, 0);
    
    printf("%d\n", ans);
    return 0;
}
```

## 迭代加深

迭代加深是规定一个深度，在 DFS 过程中不超过这个深度的情况下求解，这样能在不用走太深的情况下找到最小值，并且它远小于 BFS 的空间复杂度。

### 加成序列

> 满足如下条件的序列 X（序列中元素被标号为 1、2、3…m）被称为“加成序列”：
>
> 1. X[1]=1
> 2. X[m]=n
> 3. X[1]<X[2]<…<X[m−1]<X[m]
> 4. 对于每个 k（2≤k≤m）都存在两个整数 i 和 j （1≤i,j≤k−1 i 和 j 可相等），使得 X[k]=X[i]+X[j]。
>
> 你的任务是：给定一个整数 n，找出符合上述条件的长度 m 最小的“加成序列”。
>
> 如果有多个满足要求的答案，只需要找出任意一个可行解。
>
> 详细要求参考 [原题链接](https://www.acwing.com/problem/content/172/)。

这里也有几个剪枝策略。

1. 可行性剪枝：两个数的和必须小于等于 n，并且要大于序列的末尾数字。
2. 排除等效冗余：如果两个数的和之前已经枚举过了，那么就没有必要枚举第二遍了。

```cpp
#include <iostream>
using namespace std;

const int N = 110;
int n, a[N];

bool dfs(int u, int depth) {
    // 可行性剪枝
    if (u == depth) return a[u-1] == n;
    
    // 排除等效冗余
    bool st[N] = {0};
    for (int i = u-1; i >= 0; i--) {
        for (int j = i; j >= 0; j--) {
            int s = a[i]+a[j];
            // 可行剪枝 & 冗余剪枝
            if (s > n || st[s] || s <= a[u-1]) continue;
            a[u] = s;
            if (dfs(u+1, depth)) return true;
            a[u] = 0;
        }
    }
    return false;
}

int main() {
    a[0] = 1;
    while (cin >> n, n) {
        int depth = 1;
        while (!dfs(1, depth)) depth++;
        
        for (int i = 0; i < depth; i++) cout << a[i] << ' ';
        cout << '\n';
    }
    return 0;
}
```

### 木棒

> 乔治拿来一组等长的木棒，将它们随机地砍断，使得每一节木棍的长度都不超过 50 个长度单位。
>
> 然后他又想把这些木棍恢复到为裁截前的状态，但忘记了初始时有多少木棒以及木棒的初始长度。
>
> 请你设计一个程序，帮助乔治计算木棒的可能最小长度。
>
> 每一节木棍的长度都用大于零的整数表示。
>
> 详细要求请参考 [原题链接](https://www.acwing.com/problem/content/169/)。

本题考验剪枝策略和迭代加深。

1. 可行性剪枝，木棒的长度一定能整除所有木棍长度之和。

2. 优化搜索顺序，从大到小搜索所有木棍。

3. 排除等效冗余：

    1. 按照组合的方式枚举，这里规定每组中木棍的下标单调递增。

    2. 如果当前木棍加到当前组中失败，则略过后续与它等长的木棍。

        这里用一步假设法，设木棍 $s_i$ 无法加到当前组中，如果 $s_{i+1}=s_i$ 且能找到一个合法方案，首先根据 2 和 3-1，$s_i$ 一定在后面的组中，此时交换 $s_i,s_{i+1}$ 会得到合法方案，与它无法加到当前组中这个前提矛盾，于是 3-2 是正确的。

    3. 如果木棍 $s_0$ 放在一组开头失败，那么这整个方案一定失败。

        同样是假设法，如果后续有方案成功，此时将 $s_0$ 交换到开头，这说明让 $s_0$ 开头同样能找到合法解，与前提条件矛盾。

        但是如果不是开头就不能这么说明，因为若不是开头就无法确定这跟木棍的位置能否通过简单的交换就换到指定位置，也就没办法这么轻松的证明出来剪枝策略可行。

    4. 如果木棍 $s_0$ 放在一组结尾失败，那么整个方案一定失败。

        如果有其它方案，那么还是可以把 $s_0$ 和原来失败的位置进行交换，得到可行方案，这与前提条件矛盾。

3-x 中的这些剪枝我基本上是想不出来，只能说总结经验了。

```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 65;
int n, w[N], sum;
bool st[N];

// 大棒数量 当前木棒长度 开始枚举坐标
bool dfs(int u, int s, int start, int length) {
    // 如果数量从 0 开始，那么这里就是 u * length
    // 因为 0 ~ u-1 一共有 u 个木棒
    // 但是这里我选择 u = 1 开始，这样就要乘当前木棒的长度了
    if (u * s == sum) return true;
    
    // 开新棒
    if (s == length) return dfs(u+1, 0, 0, length);
    
    // 优化搜索顺序
    for (int i = start; i < n; i++) {
        if (st[i]) continue;
        // 可行性剪枝
        if (s + w[i] > length) continue;
        
        st[i] = true;
        if (dfs(u, s+w[i], i+1, length))
            return true;
        st[i] = false;
        
        // 3-2
        int j = i;
        while (j < n && w[j] == w[i]) j++;
        i = j-1;
        
        // 3-3
        if (s == 0) return false;
        
        // 3-4
        if (s + w[i] == length) return false;
    }
    
    return false;
}

int main() {
    while (cin >> n, n) {
        sum = 0;
        memset(st, 0, sizeof st);
        for (int i = 0; i < n; i++) {
            cin >> w[i];
            sum += w[i];
        }
        // 优化搜索顺序
        sort(w, w+n, greater<int>());
        for (int length = 1; length <= sum; length++) {
            // 可行性剪枝
            if (sum % length == 0 && dfs(1, 0, 0, length)) {
                cout << length << endl;
                break;
            }
        }
    }
    return 0;
}
```


## 双向 DFS

### 送礼物

> 达达帮翰翰给女生送礼物，翰翰一共准备了 N 个礼物，其中第 i 个礼物的重量是 G[i]。
>
> 达达的力气很大，他一次可以搬动重量之和不超过 W 的任意多个物品。
>
> 达达希望一次搬掉尽量重的一些物品，请你告诉达达在他的力气范围内一次性能搬动的最大重量是多少。
>
> 详细要求请参考 [原题链接](https://www.acwing.com/problem/content/description/173/)。

本题看起来是一个背包问题，但如果用背包需要 O(NW) 的复杂度，这里 N 很小而 W 很大，因此是过不了的。这里用双向 DFS 的思想。

1. 先搜索前 K 个数，将这些数所有的组合都枚举出来
2. 然后再看后 N-K 个数，设前后两组数的和分别为 $S_1, S_2$ 先找出某组数之和 $S_2$
3. 然后二分查找前面的 $S_1 \le W-S_2$
4. 由于 $S_2=0$ 是合法的情况，那么整个流程也是不会漏解的

分析复杂度，找一个合适的 K 值。

1. 搜索前 K 个数并排序是 $2^K+2^K\log 2^K=2^K(1+K)$

2. 搜索后 N-K 个数并二分查找前 K 个数的组合是 $2^{N-K}\cdot \log 2^K=2^{N-K}K$
3. 均衡时间复杂度，显然 $K=\cfrac{N}{2}$ 比较合适。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 46;
int n, m, k, w[N];
int ans, weights[1 << (N/2)], cnt;

void dfs1(int u, int s) {
    if (u == k) {
        weights[cnt++] = s;
        return;
    }
    
    // 防止溢出, w[u]+s <= m
    if (w[u] <= m-s) dfs1(u+1, s+w[u]);
    dfs1(u+1, s);
}

void dfs2(int u, int s) {
    if (u == n) {
        int l = 0, r = cnt-1;
        while (l < r) {
            int mid = l+r+1 >> 1;
            if (weights[mid] <= m-s) l = mid;
            else r = mid-1;
        }
        ans = max(ans, weights[l]+s);
        return;
    }
    if (w[u] <= m-s) dfs2(u+1, s+w[u]);
    dfs2(u+1, s);
}

int main() {
    cin >> m >> n;
    k = n >> 1;
    for (int i = 0; i < n; i++) cin >> w[i];
    sort(w, w+n, greater<int>());
    
    dfs1(0, 0);
    sort(weights, weights+cnt);
    cnt = unique(weights, weights+cnt) - weights;
    
    dfs2(k, 0);
    cout << ans << endl;
    
    return 0;
}
```

## IDA*

迭代加深的过程中加一个额外剪枝，如果当前层数+预估层数已经超过最深层数就剪掉。

### 排书

> 给定 n 本书，编号为 1∼n。
>
> 在初始状态下，书是任意排列的。
>
> 在每一次操作中，可以抽取其中连续的一段，再把这段插入到其他某个位置。
>
> 我们的目标状态是把书按照 1∼n 的顺序依次排列。
>
> 求最少需要多少次操作。
>
> 详细要求请参考 [原题链接](https://www.acwing.com/problem/content/182/)。

首先证明一个取整的公式：
$$
\begin{aligned}
\lceil \frac{a}{b} \rceil &= \lfloor \frac{a+b-1}{b} \rfloor,\text{where }a=kb+r\\
\text{LHS}&=\left\{ \begin{aligned}
&k+1,r\ne 0\\
&k,r=0
\end{aligned}\right .\\
\text{RHS} &= \lfloor k + 1 + \frac{r-1}{b} \rfloor\\
&=\left\{ \begin{aligned}
&k+1,r\ne 0\\
&k,r=0
\end{aligned} \right .\\
&=\text{LHS}
\end{aligned}
$$
每次抽出一段插入到另一段数的后面，会改变三个位置的后继：

```text
___o[___o]__o____
```

设共有 $\text{total}$ 个后继错误的数，最好情况下需要 $\lceil \cfrac{\text{total}}{3} \rceil=\lfloor \cfrac{\text{total}+2}{3} \rfloor$ 次操作，用这个当作估价函数。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int N = 15;
int n, bak[5][N], q[N];

int f() {
    int total = 0;
    for (int i = 1; i < n; i++) {
        if (q[i] != q[i-1]+1) total++;
    }
    return (total + 2) / 3;
}

bool dfs(int u, int depth) {
    // IDA*: 可行性剪枝
    if (f() + u > depth) return false;
    if (f() == 0) return true;
    
    for (int len = 1; len <= n; len++) {
        for (int l = 0; l+len-1 < n; l++) {
            int r = l+len-1;  // r-l+1 = len
            
            for (int k = r+1; k < n; k++) {
                memcpy(bak[u], q, sizeof q);
                // bak[u][r+1~k] -> q[l~?]
                // bak[u][l~r] -> q[?~k]
                int x = l, y = r+1;
                while (y <= k) q[x++] = bak[u][y++];
                y = l;
                while (y <= r) q[x++] = bak[u][y++];
                if (dfs(u+1, depth)) return true;
                memcpy(q, bak[u], sizeof q);
            }
        }
    }
    
    return false;
}

int main() {
    int T; cin >> T;
    while (T--) {
        cin >> n;
        for (int i = 0; i < n; i++) cin >> q[i];
        int depth = 0;
        while (depth < 5 && !dfs(0, depth)) depth++;
        if (depth == 5) cout << "5 or more\n";
        else cout << depth << '\n';
    }
    return 0;
}
```

### 回转游戏

> 有一个 `#` 形的棋盘，上面有 1,2,3 三种数字各 8 个。
>
> 给定 8 种操作，分别为图中的 A∼H。这些操作会按照图中字母和箭头所指明的方向，把一条长为 7 的序列循环移动 1 个单位。
>
> 例如下图最左边的 `#` 形棋盘执行操作 A 后，会变为下图中间的 `#` 形棋盘，再执行操作 C 后会变成下图最右边的 `#` 形棋盘。
>
> 给定一个初始状态，请使用最少的操作次数，使 `#` 形棋盘最中间的 8 个格子里的数字相同。
>
> ```text
>          A   B
>          ^   ^
>          1   1
>          1   1
> H <- 3 2 3 2 3 1 3 -> C
>          2   2
> G <- 3 1 2 2 2 3 1 -> D
>          2   1
>          3   3
>          V   V
>          F   E
> ```
>
> 具体图像和详细要求等参考 [原题链接](https://www.acwing.com/problem/content/183/)，这里就不把图片也复制过来了。

我做这道题被卡住的地方是下面几点。

1. 一次操作只能使中间的一个数字发生改变，所以估价函数是 `8 - cnt` 其中 `cnt` 是数目最多的数字的个数。
2. 只有三个数字 1 2 3 如果认为是九个数字那么做就会浪费很多空间。
3. AF，BE，CH，DG 操作是互逆的，恢复现场只需要执行一遍互逆操作就可以。
4. 出于最优性剪枝，两个互逆操作不可以先后连续进行，可以直接剪掉。
5. 执行某个操作时注意覆盖顺序，不要用更新后的数据去更新原数据，这点体现在 `op` 数组的定义上。

实测加个氧气优化直接加速五倍，太厉害辣！

```cpp
/*
字母后面的数字表示它的编号，如 A 的编号为 0
        A0    B1
         0     1
         2     3
H7 4  5  6  7  8  9  10 C2
         11    12
G6 13 14 15 16 17 18 19 D3
         20    21
         22    23
         F5    E4
*/

#pragma GCC optimize(2)
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 24;
int opposite[8] = {5, 4, 7, 6, 1, 0, 3, 2};
int mid[8] = {6, 7, 8, 11, 12, 15, 16, 17};
int op[8][7] = {
    {0, 2, 6, 11, 15, 20, 22},
    {1, 3, 8, 12, 17, 21, 23},
    {10, 9, 8, 7, 6, 5, 4},
    {19, 18, 17, 16, 15, 14, 13},
    {23, 21, 17, 12, 8, 3, 1},
    {22, 20, 15, 11, 6, 2, 0},
    {13, 14, 15, 16, 17, 18, 19},
    {4, 5, 6, 7, 8, 9, 10},
};
int a[N], ans[100], cnt;

int f() {
    int cntn[4] = {0}, maxv = -1;
    for (int i = 0; i < 8; i++) {
        int n = a[mid[i]];
        cntn[n]++;
        maxv = max(maxv, cntn[n]);
    }
    return 8-maxv;
}

void move(int sln[]) {
    int head = a[sln[0]];
    for (int i = 0; i < 6; i++)
        a[sln[i]] = a[sln[i+1]];
    a[sln[6]] = head;
}

bool dfs(int u, int depth, int last) {
    if (f() == 0) return true;
    if (u + f() > depth) return false;
    
    for (int i = 0; i < 8; i++) {
        if (opposite[i] == last) continue;
        move(op[i]);
        ans[cnt++] = i;
        if (dfs(u+1, depth, i)) return true;
        ans[cnt--] = 0;
        move(op[opposite[i]]);
    }
    
    return false;
}

int main() {
    while (cin >> a[0], a[0]) {
        cnt = 0;
        for (int i = 1; i < N; i++) cin >> a[i];
        
        int depth = 0;
        while (!dfs(0, depth, -1)) depth++;
        
        if (depth == 0) printf("No moves needed");
        else for (int i = 0; i < cnt; i++) printf("%c", ans[i]+'A');

        printf("\n%d\n", a[mid[0]]);
    }
    return 0;
}
```

### 骑士精神

> 题目是图，这里不弄过来了。
>
> 链接：[P2324](https://www.luogu.com.cn/problem/P2324)。

首先估价函数应该是不同的点的个数，就是对于每个点 $(x_0,y_0)$ 上如果不同就加一，但是这样做有一个问题，就是如果只需要走一步，但是由于空格的原因我们计算出来的值是 2，这不符合要求，所以要把算出来的值减 1，然后与 0 取一个 max，因为如果两个一模一样算出来的也是 0

估价函数可以比真实值小，但是不能比真实值大。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 10;
char g[N][N];
char dest[N][N] = {
    "11111",
    "01111",
    "00*11",
    "00001",
    "00000",
};
int dx[8] = {-1, -1, -2, -2, 1, 1, 2, 2};
int dy[8] = {2, -2, 1, -1, 2, -2, 1, -1};
int lim;

int f() {
    int res = 0;
    for (int i = 0; i < 5; i++) {
        for (int j = 0; j < 5; j++) {
            if (g[i][j] != dest[i][j]) res++;
        }
    }
    return max(res-1, 0);
}

bool dfs(int u, int x, int y) {
    int v = f();
    if (v == 0) return true;
    if (u == lim || v + u > lim) return false;

    for (int i = 0; i < 8; i++) {
        int nx = x+dx[i], ny = y+dy[i];
        if (nx < 0 || nx >= 5 || ny < 0 || ny >= 5) continue;
        swap(g[x][y], g[nx][ny]);
        if (dfs(u+1, nx, ny)) return true;
        swap(g[x][y], g[nx][ny]);
    }
    return false;
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        int x = -1, y = -1;
        for (int i = 0; i < 5; i++) {
            scanf("%s", g[i]);
            for (int j = 0; j < 5; j++) {
                if (g[i][j] == '*') x = i, y = j;
            }
        }

        bool found = false;
        for (lim = 0; lim <= 15; lim++)
            if (dfs(0, x, y)) {
                found = true;
                printf("%d\n", lim);
                break;
            }
        
        if (!found) puts("-1");
    }
    return 0;
}
```



## 记忆化搜索

### 滑雪

> 给定一个 R 行 C 列的矩阵，表示一个矩形网格滑雪场。
>
> 矩阵中第 i 行第 j 列的点表示滑雪场的第 i 行第 j 列区域的高度。
>
> 一个人从滑雪场中的某个区域内出发，每次可以向上下左右任意一个方向滑动一个单位距离。
>
> 当然，一个人能够滑动到某相邻区域的前提是该区域的高度低于自己目前所在区域的高度。
>
> 下面给出一个矩阵作为例子：
>
> ```
> 1  2  3  4 5
> 16 17 18 19 6
> 15 24 25 20 7
> 14 23 22 21 8
> 13 12 11 10 9
> ```
>
> 在给定矩阵中，一条可行的滑行轨迹为 24−17−2−1。
>
> 在给定矩阵中，最长的滑行轨迹为 25−24−23−…−3−2−1，沿途共经过 25 个区域。
>
> 现在给定你一个二维矩阵表示滑雪场各区域的高度，请你找出在该滑雪场中能够完成的最长滑雪轨迹，并输出其长度(可经过最大区域数)。
>
> [原题链接](https://www.acwing.com/problem/content/903/)

这题可以用 `dfs` 暴搜解决，代码如下：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 310;
int r, c;
int a[N][N];

int dfs(int x, int y) {
    if (x < 1 || x > r || y < 1 || y > c) {
        return 0;
    }
    
    int res = 0;
    int dx[] = {-1, 0, 1, 0}, dy[] = {0, 1, 0, -1};
    for (int i = 0; i < 4; i++) {
        int nx = x + dx[i], ny = y + dy[i];
        if (a[nx][ny] < a[x][y]) res = max(dfs(nx, ny), res);
    }
    return res + 1;
}

int main() {
    cin >> r >> c;
    for (int i = 1; i <= r; i++)
        for (int j = 1; j <= c; j++)
            cin >> a[i][j];
    
    int ans = 0;
    for (int i = 1; i <= r; i++)
        for (int j = 1; j <= c; j++)
            ans = max(ans, dfs(i, j));
    
    cout << ans << endl;
    return 0;
}
```

但这会超时，所以要优化一下，用记忆化搜索，就是在计算的过程中把结果保存下来，简单改动一下 `dfs` 函数即可。

```cpp
int f[N][N];

int dfs(int x, int y) {
    if (x < 1 || x > r || y < 1 || y > c) {
        return 0;
    }
    
    if (f[x][y]) return f[x][y];
    
    int res = 0;
    int dx[] = {-1, 0, 1, 0}, dy[] = {0, 1, 0, -1};
    for (int i = 0; i < 4; i++) {
        int nx = x + dx[i], ny = y + dy[i];
        if (a[nx][ny] < a[x][y]) res = max(dfs(nx, ny), res);
    }
    
    f[x][y] = res + 1;
    return res + 1;
}
```

### 没有上司的舞会

> Ural 大学有 N 名职员，编号为 1∼N。他们的关系就像一棵以校长为根的树，父节点就是子节点的直接上司。每个职员有一个快乐指数，用整数 Hi 给出，其中 1<=i<=N。
>
> 现在要召开一场周年庆宴会，不过，没有职员愿意和直接上司一起参会。在满足这个条件的前提下，主办方希望邀请一部分职员参会，使得所有参会职员的快乐指数总和最大，求这个最大值。
>
> [原题链接](https://www.acwing.com/problem/content/287/)

题干中已提示需要用树来储存信息，由于要遍历树所以用邻接表存储。

+ 状态表示 $f(u,j)$：
  + 含义：`j` 是布尔值，表示第 `u` 节点选择或者不选得到的数的集合。
  + 存储：这些数的最大值。
+ 状态转移，其中 $s_i$ 表示 `u` 节点的某个子节点：
  + 不选当前节点：$f(u,0)=\sum \max\{f(s_i,1), f(s_i,0)\}$
  + 选当前节点，此时子节点就一定不能选了：$f(u,1)=\sum f(s_i,0)$

本题还有一个小难点，就是没有指定根节点的情况下如何找到根节点，这里的方法其实很简单，开一个布尔数组 `hasfather` 查看谁没有父节点就可以。对于每个状态，都使用 DFS 遍历子节点的状态。

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 6010;
int h[N], e[N], ne[N], idx, happy[N], f[N][2];
bool hasfather[N];
int n;

void add(int p, int c) {
    e[idx] = c, ne[idx] = h[p], h[p] = idx++;
}

int dfs(int u, bool choose) {
    if (f[u][choose]) return f[u][choose];
    
    int res = choose ? happy[u] : 0;
    for (int i = h[u]; i != -1; i = ne[i]) {
        int j = e[i];
        if (choose) {
            res += dfs(j, false);
        } else {
            res += max(dfs(j, false), dfs(j, true));
        }
    }
    f[u][choose] = res;
    return res;	
}

int main() {
    cin.tie(0), cout.tie(0), ios::sync_with_stdio(false);
    cin >> n;
    memset(h, -1, sizeof h);
    for (int i = 1; i <= n; i++) cin >> happy[i];
    for (int i = 0; i < n-1; i++) {
        int p, c;
        cin >> c >> p;
        add(p, c);
        hasfather[c] = true;
    }
    
    int root = -1;
    // 这里要从 1~n，因为遍历的是每个节点
    for (int i = 1; i <= n; i++) {
        if (!hasfather[i]) {
            root = i;
            break;
        }
    }
    
    cout << max(dfs(root, true), dfs(root, false)) << '\n';
    return 0;
}
```

也可以进一步优化，把结果储存在 `f[u][j]` 中而 `dfs` 本身没有返回值，如下：

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 6010;
int h[N], e[N], ne[N], idx, happy[N], f[N][2];
bool hasfather[N];
int n;

void add(int p, int c) {
    e[idx] = c, ne[idx] = h[p], h[p] = idx++;
}

void dfs(int u) {
    f[u][0] = 0;
    f[u][1] = happy[u];

    for (int i = h[u]; i != -1; i = ne[i]) {
        int j = e[i];
        dfs(j);
        f[u][0] += max(f[j][0], f[j][1]);
        f[u][1] += f[j][0];
    }
}

int main() {
    cin.tie(0), cout.tie(0), ios::sync_with_stdio(false);
    cin >> n;
    memset(h, -1, sizeof h);
    for (int i = 1; i <= n; i++) cin >> happy[i];
    for (int i = 0; i < n-1; i++) {
        int p, c;
        cin >> c >> p;
        add(p, c);
        hasfather[c] = true;
    }
    
    int root = -1;
    // 这里要从 1~n，因为遍历的是每个节点
    for (int i = 1; i <= n; i++) {
        if (!hasfather[i]) {
            root = i;
            break;
        }
    }
    dfs(root);
    cout << max(f[root][0], f[root][1]) << '\n';
    return 0;
}
```
