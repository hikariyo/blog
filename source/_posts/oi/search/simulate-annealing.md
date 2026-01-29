---
date: 2023-08-18 09:35:00
updated: 2023-08-18 09:35:00
title: 搜索 - 模拟退火
katex: true
tags:
- Algo
- C++
- Search
categories:
- OI
---

## 简介

模拟退火是基于随机化的求多峰函数最值的算法，它虽然能以较高的概率求得最值，但是并不是 `100%`（那也比纯暴力拿个 10, 20 分要好）

一个题如果能够满足是要求一个区间最值，并且函数是具有连续性的，也就是当自变量变化小时函数值变化也很小，这样就可以用模拟退火去做。

## 费马点

> 在二维平面上有 n 个点，第 i 个点的坐标为 $(x_i,y_i)$。
>
> 请你找出一个点，使得该点到这 n 个点的距离之和最小。
>
> 该点可以选择在平面中的任意位置，甚至与这 n 个点的位置重合。
>
> 题目链接：[AcWing 3167](https://www.acwing.com/problem/content/3170/)。

$x_i, y_i$ 的范围是 10000，因此定义初始温度是 10000，然后令其以指数趋势衰减，每次乘一个系数 $0.99$ 之类的，然后定义下界，本题要求保留整数因此是 $10^{-4}$ 即可，每次根据温度随机一个点，将函数值看作 “能量”，我们希望选择该解的概率会是：

+ 当能量变化越大时选择的概率越小。
+ 当温度越高时选择的概率越大。

模拟退火算法是用的指数函数的模型定义这个概率的，但是为什么以 $e$ 为底数我也不清楚，可能是计算机算起来方便吧，毕竟其他所有数字的指数函数可以转化成它，只是在指数位置上差了一个常数（$a^x=\exp(x\ln a)$）
$$
\Delta E=E_1-E_0\\
P=\min\{1,\exp(-\Delta E/T)\}
$$
一般使用卡时的方法确定退火次数，当运行 `0.8` 后就结束。

```cpp
#include <cstdio>
#include <ctime>
#include <cstdlib>
#include <cstring>
#include <cmath>
#include <algorithm>
using namespace std;

const int N = 1010;
struct Object { int x, y, m; };
int n;
Object objs[N];

namespace ans {
    double x = 0, y = 0, energy = 1e20;
}

double calc(double x, double y) {
    double res = 0;
    for (int i = 0; i < n; i++) {
        double dx = objs[i].x - x, dy = objs[i].y - y;
        res += objs[i].m * sqrt(dx*dx + dy*dy);
    }
    if (res < ans::energy) ans::energy = res, ans::x = x, ans::y = y;
    return res;
}

double random(double l, double r) {
    return (double)rand() / RAND_MAX * (r-l) + l;
}

void SA() {
    double x = ans::x, y = ans::y;
    for (double t = 1e4; t > 1e-10; t *= 0.998) {
        double nx = x + random(-t, t), ny = y + random(-t, t);
        double de = calc(nx, ny) - calc(x, y);
        if (exp(-de / t) > random(0, 1)) x = nx, y = ny;
    }
}

int main() {
    srand(20230818);
    scanf("%d", &n);
    for (int i = 0; i < n; i++) {
        int x, y, m;
        scanf("%d%d%d", &x, &y, &m);
        objs[i] = {x, y, m};
        ans::x += x, ans::y += y;
    }
    ans::x /= n, ans::y /= n;
    calc(ans::x, ans::y);

    while ((double)clock() / CLOCKS_PER_SEC < 0.8) SA();

    
    if (fabs(ans::x) < 1e-4) ans::x = 0;
    if (fabs(ans::y) < 1e-4) ans::y = 0;
    printf("%.3lf %.3lf\n", ans::x, ans::y);
    return 0;
}
```

## 均分数据

> 已知 N 个正整数：A1、A2、……、An。
>
> 今要将它们分成 M 组，使得各组数据的数值和最平均，即各组的均方差最小。
>
> 均方差公式如下：
> $$
> \sigma =\sqrt{\frac{\sum_{i=1}^m (x_i-\overline{x})^2}{m}},\overline{x}=\frac{\sum_{i=1}^m x_i}{m}
> $$
> 其中 $x_i$ 代表第 $i$ 组数据的数值和。

首先，在退火的过程中我们需要随机挑两个数交换位置，然后进行概率跳转。对于如何将一个数划分成 $m$ 组，这里当然也可以随机搞，但是可以做一步贪心，每次将数字放到数值和（即 $x_i$）最小的一组中，这样尽可能使方差小。

本题要求精度两位小数，所以温度的下限调低一点。

```cpp
#include <cstdlib>
#include <cstdio>
#include <ctime>
#include <cmath>
#include <algorithm>
#include <cstring>
using namespace std;

const int N = 25, M = 10;
int x[M], w[N], n, m;
double ans = 1e9, avg = 0;

double calc() {
    memset(x, 0, sizeof x);
    for (int i = 0; i < n; i++) {
        // 选出当前最小的坑
        int t = 0;
        for (int j = 0; j < m; j++)
            if (x[j] < x[t]) t = j;
        x[t] += w[i];
    }
    
    // 求方差
    double res = 0;
    for (int i = 0; i < m; i++)
        res += (x[i] - avg) * (x[i] - avg);
    res /= m;
    ans = min(res, ans);
    return res;
}

void SA() {
    random_shuffle(w, w+n);
    for (int t = 1e4; t > 1e-6; t *= 0.99) {
        int a = rand() % n, b = rand() % n;
        double pre = calc();
        swap(w[a], w[b]);
        double now = calc(), delta = now - pre;
        // 如果概率跳转没成功就换回来
        if (exp(-delta / t) < (double)rand() / RAND_MAX) swap(w[a], w[b]);
    }
}

int main() {
    srand(20200614);
    scanf("%d%d", &n, &m);
    for (int i = 0; i < n; i++) {
        scanf("%d", &w[i]);
        avg += w[i];
    }
    avg /= m;
    
    while ((double)clock() / CLOCKS_PER_SEC < 0.8) SA();
    
    // 防止 -0
    if (fabs(ans) < 1e-3) puts("0.00");
    else printf("%.2lf\n", sqrt(ans));
    
    return 0;
}
```

## 球形空间产生器

> 有一个球形空间产生器能够在 n 维空间中产生一个坚硬的球体。
>
> 现在，你被困在了这个 n 维球体中，你只知道球面上 n+1 个点的坐标，你需要以最快的速度确定这个 n 维球体的球心坐标，以便于摧毁这个球形空间产生器。
>
> **注意：** 数据保证有唯一解。
>
> 题目链接：[AcWing 207](https://www.acwing.com/problem/content/209/)，[P4035](https://www.luogu.com.cn/problem/P4035)。

高斯消元是可以求出来唯一解，但是它难写啊。

这题用模拟退火的话判断是否更优的依据就是当前点到各个点距离的方差更小。

```cpp
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <ctime>
#include <cmath>
#include <algorithm>
using namespace std;

const int N = 12;
int n;

double points[N][N];
double ans[N], cur[N], ansv = 1e9;


double get_dist(double a[], double b[]) {
    double res = 0;
    for (int i = 0; i < n; i++) {
        double delta = a[i] - b[i];
        res += delta * delta;
    }
    // 开根很影响精度 这里不开了
    return res;
}

double random(double l, double r) {
    return (double)rand() / RAND_MAX * (r-l) + l;
}

double calc(double p[]) {
    static double dist[N];
    double avg = 0, res = 0;
    for (int i = 0; i <= n; i++) {
        dist[i] = get_dist(p, points[i]);
        avg += dist[i];
    }
    avg /= n + 1;
    
    for (int i = 0; i <= n; i++)
        res += (dist[i] - avg) * (dist[i] - avg);
    // res /= n + 1;
    // 方差除不除元素数量不影响相对大小
    if (res < ansv) memcpy(ans, p, sizeof ans), ansv = res;
    return res;
}

void SA() {
    static double np[N];
    for (double t = 2e4; t > 1e-8; t *= 0.999) {
        for (int i = 0; i < n; i++)
            np[i] = random(cur[i] - t, cur[i] + t);
        double delta = calc(np) - calc(cur);
        if (exp(-delta / t) > random(0, 1)) memcpy(cur, np, sizeof cur);
    }
}

int main() {
    scanf("%d", &n);
    for (int i = 0; i <= n; i++) {
        for (int j = 0; j < n; j++) {
            scanf("%lf", &points[i][j]);
            cur[i] += points[i][j];
        }
        cur[i] /= n + 1;
    }
    
    while ((double)clock() / CLOCKS_PER_SEC < 0.8) SA();
    
    for (int i = 0; i < n; i++)
        if (fabs(ans[i]) < 1e-4) printf("0.000 ");
        else printf("%.3lf ", ans[i]);
    return 0;
}
```

## 平衡点

> 有 n 个重物，每个重物系在一条足够长的绳子上。
>
> 每条绳子自上而下穿过桌面上的洞，然后系在一起。图中 x 处就是公共的绳结。假设绳子是完全弹性的（即不会造成能量损失），桌子足够高（重物不会垂到地上），且忽略所有的摩擦，求绳结 x 最终平衡于何处。
>
> 题目链接：[P1337](https://www.luogu.com.cn/problem/P1337)。

首先稳定是指重力势能最小的时候，所以这是个典型的连续多峰函数，可以模拟退火。

设绳长为 $L_i$，桌子高度为 $H$，当前点与每个洞口的距离是 $d_i$，肯定有 $h=H-(L_i-d_i)$，那么：
$$
\begin{aligned}
E_p&=mgh\\
\sum E_p&=\sum m_ig(H-L_i+d_i)\\
&=\sum m_igL_i-\sum m_igd_i\\
&=C_0+C_1\sum m_id_i
\end{aligned}
$$
我们只需要比较出来相对大小，所以直接用 $\sum m_id_i$ 当作重力势能即可。

注意初始温度不要设置的太高，容易乱跳然后挑不出来。

```cpp
#include <cstdio>
#include <ctime>
#include <cstdlib>
#include <cstring>
#include <cmath>
#include <algorithm>
using namespace std;

const int N = 1010;
struct Object { int x, y, m; };
int n;
Object objs[N];

namespace ans {
    double x = 0, y = 0, energy = 1e20;
}

double calc(double x, double y) {
    double res = 0;
    for (int i = 0; i < n; i++) {
        double dx = objs[i].x - x, dy = objs[i].y - y;
        res += objs[i].m * sqrt(dx*dx + dy*dy);
    }
    if (res < ans::energy) ans::energy = res, ans::x = x, ans::y = y;
    return res;
}

double random(double l, double r) {
    return (double)rand() / RAND_MAX * (r-l) + l;
}

void SA() {
    double x = ans::x, y = ans::y;
    for (double t = 1e4; t > 1e-10; t *= 0.998) {
        double nx = x + random(-t, t), ny = y + random(-t, t);
        double de = calc(nx, ny) - calc(x, y);
        if (exp(-de / t) > random(0, 1)) x = nx, y = ny;
    }
}

int main() {
    srand(20230818);
    scanf("%d", &n);
    for (int i = 0; i < n; i++) {
        int x, y, m;
        scanf("%d%d%d", &x, &y, &m);
        objs[i] = {x, y, m};
        ans::x += x, ans::y += y;
    }
    ans::x /= n, ans::y /= n;
    calc(ans::x, ans::y);

    while ((double)clock() / CLOCKS_PER_SEC < 0.8) SA();

    
    if (fabs(ans::x) < 1e-4) ans::x = 0;
    if (fabs(ans::y) < 1e-4) ans::y = 0;
    printf("%.3lf %.3lf\n", ans::x, ans::y);
    return 0;
}
```

## Haywire

> Farmer John有 N 只奶牛,($4≤N≤12$,其中 N 是偶数).
>
> 他们建立了一套原生的系统，使得奶牛与他的朋友可以通过由干草保护的线路来进行对话交流.
>
> 每一头奶牛在这个牧场中正好有3个朋友，并且他们必须把自己安排在一排干草堆中.
>
> 一条长 L 的线路要占用刚好 L 堆干草来保护线路。
>
> 比如说，如果有两头奶牛分别在草堆4与草堆7中，并且他们是朋友关系，那么我们就需要用3堆干草来建造线路，使他们之间能够联系.
>
> 假设每一对作为朋友的奶牛都必须用一条单独的线来连接，并且我们可以随便地改变奶牛的位置，请计算出我们建造线路所需要的最少的干草堆.
>
> 题目链接：[P2210](https://www.luogu.com.cn/problem/P2210)。

本题是求一个最优排列，并且交换两个位置后函数值的变化不会太大，所以适合使用模拟退火。

由于点很少，用一个 01 邻接矩阵存图就可以了，计算的时候就看每个结点所连接的边在数组中跨度多长就行，这里可以先建一个数组，值对应索引，方便进行计算。

```cpp
#include <algorithm>
#include <cstdio>
#include <cstdlib>
#include <cmath>
#include <cstring>
#include <ctime>
using namespace std;

const int N = 15;
int q[N], n, ans = 1e9;
int refl[N];
bool st[N][N], g[N][N];

double calc() {
    memset(st, 0, sizeof st);
    // 构造反映射
    for (int i = 1; i <= n; i++) refl[q[i]] = i;

    int res = 0;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            if (!g[i][j] || st[i][j]) continue;
            int cost = abs(refl[i] - refl[j]);
            res += cost;
            st[i][j] = st[j][i] = true;
        }
    }
    ans = min(res, ans);
    return res;
}

void SA() {
    random_shuffle(q+1, q+1+n);
    for (double t = 1e8; t >= 1e-8; t *= 0.998) {
        int a = rand() % n + 1, b = rand() % n + 1;
        double pre = calc();
        swap(q[a], q[b]);
        double now = calc(), dt = now - pre;
        if (exp(dt / t) < (double)rand() / RAND_MAX) swap(q[a], q[b]);
    }
}

int main() {
    srand(20230818);
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        g[i][a] = g[i][b] = g[i][c] = true;
    }
    for (int i = 1; i <= n; i++) q[i] = i;

    while ((double)clock() / CLOCKS_PER_SEC < 0.8) SA();
    printf("%d\n", ans);

    return 0; 
}
```

## 分金币

> 现在有 n 枚金币，它们可能会有不同的价值，第 i 枚金币的价值为 vi。
>
> 现在要把它们分成两部分，要求这两部分金币数目之差不超过 1，问这样分成的两部分金币的价值之差最小是多少？
>
> 题目链接：[P3878](https://www.luogu.com.cn/problem/P3878)。

题目想告诉我们直接 $\lfloor n/2 \rfloor$ 为分界线就可以，然后交换两个位置也不会对答案造成很大的影响，因此还可以用模拟退火。

```cpp
#include <cstdlib>
#include <cstdio>
#include <ctime>
#include <cmath>
#include <algorithm>
using namespace std;

typedef long long LL;
const int N = 35;
int n, w[N];
LL ans;

int calc() {
    int mid = n/2;
    // 0~mid-1 mid~n-1
    LL l = 0, r = 0;
    for (int i = 0; i < mid; i++) l += w[i];
    for (int i = mid; i < n; i++) r += w[i];
    LL res = abs(l-r);
    ans = min(res, ans);
    return res;
}

void SA() {
    random_shuffle(w, w+n);
    for (double t = 1e4; t > 1e-4; t *= 0.99) {
        int a = rand() % n, b = rand() % n;
        int pre = calc();
        swap(w[a], w[b]);
        int now = calc(), dt = now - pre;
        if (exp((double)-dt / t) < (double)rand() / RAND_MAX) swap(w[a], w[b]);
    }
}

#define now ((double)clock() / CLOCKS_PER_SEC)

int main() {
    srand(20230818);
    int T;
    scanf("%d", &T);
    // Time per case
    double tpc = 0.8 / T;
    while (T--) {
        double start = now;
        scanf("%d", &n);
        ans = 1e18;
        for (int i = 0; i < n; i++) scanf("%d", &w[i]);
        while (now - start < tpc) SA();
        printf("%lld\n", ans);
    }
    return 0;
}
```



