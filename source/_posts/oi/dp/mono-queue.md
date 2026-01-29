---
date: 2023-08-16 16:02:00
updated: 2023-08-16 16:02:00
title: 动态规划 - 单调队列优化
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 最大连续子序列和

> 输入一个长度为 n 的整数序列，从中找出一段长度不超过 m 的连续子序列，使得子序列中所有数的和最大。
>
> **注意：** 子序列的长度至少是 1。
>
> 题目链接：[AcWing 135](https://www.acwing.com/problem/content/137/)。

本题可以用线段树做，但是既然是 DP 就从 DP 角度去思考。

+ 状态表示 $f(i)$：以 $i$ 结尾长度为 $m$ 的连续子序列和的最大值。

+ 状态计算：
    $$
    f(i)=\max_{j=i-m}^{i-1}(s_i-s_j)=s_i-\min_{j=i-m}^{i-1}s_j
    $$
    对于后面就是一个单调队列问题。

但是这里队列中存的元素长度不超过 $m$，但是枚举到 $i$ 时需要的是上一轮的最值，所以我们提前加入元素 $0$，并且 $s_0=0$，这样就可以把区间错开了。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 300010;
typedef long long LL;
LL s[N];
int n, m, q[N], hh = 0, tt = -1;

int main() {
    scanf("%d%d", &n, &m);
    
    LL res = -1e18;
    q[++tt] = 0;
    for (int i = 1; i <= n; i++) {
        scanf("%lld", &s[i]);
        s[i] += s[i-1];
        
        // 这里不能用 tt-hh+1 判断, 原因在下文中给出
        while (hh <= tt && i - q[hh] > m) hh++;
        // 这句话也不能放到 while 循环上面, 因为此时队列中元素个数可能不满足要求
        res = max(res, s[i] - s[q[hh]]);
        while (hh <= tt && s[q[tt]] >= s[i]) tt--;
        q[++tt] = i;
    }
    
    printf("%lld\n", res);
    return 0;
}
```

如果在注释处使用 $tt-hh+1$，是错误的，因为我们单调队列中需要保证单调递增，即使某元素已经应该划出窗口，队列中的元素个数可能依然小于 $m$，此时就不对了。

当执行那句代码时，队头元素下标最大是 $i-1$，因此这个判断条件相当于 $(i-1)-q[hh]+1$

## 旅行问题

> John 打算驾驶一辆汽车周游一个环形公路。
>
> 公路上总共有 n 个车站，每站都有若干升汽油（有的站可能油量为零），每升油可以让汽车行驶一千米。
>
> John 必须从某个车站出发，一直按顺时针（或逆时针）方向走遍所有的车站，并回到起点。
>
> 在一开始的时候，汽车内油量为零，John 每到一个车站就把该站所有的油都带上（起点站亦是如此），行驶过程中不能出现没有油的情况。
>
> 任务：判断以每个车站为起点能否按条件成功周游一周。
>
> 题目链接：[AcWing 1088](https://www.acwing.com/problem/content/1090/)。

首先需要破环成链，仿照区间 DP 的思想，其实感觉这题和 DP 也没什么关系。

其次，考虑如何进行转化。

+ 如果能从 $i\to i$ 顺时针环行一周，设每个点的权值 $w(i)$ 是当前点增加的距离和减少距离之差，那么如果：
    $$
    \forall d\in[i,i+n-1), \sum_{j=i}^{d}w(j)\ge 0
    $$
    就证明了 $i$ 是可以顺时针环行一周的。

+ 如果能从 $i\to i$ 逆时针环行一周，同样每个点的权值是当前点增加的距离和到下一个点需要的距离之差，那么如果：
    $$
    \forall d\in(i,i+n-1],\sum_{j=d}^{i+n-1} w(j)\ge 0
    $$
    就证明了 $i$ 是可以逆时针环形一周的。

这就转化成了前缀和（或后缀和）的区间最小值问题，用单调队列做。

关于逆时针的部分，建议画图辅助理解，我不想画了，占服务器空间。

```cpp
#include <cstdio>
using namespace std;

typedef long long LL;
const int N = 2000010;
LL s[N], q[N];
int p[N/2], d[N/2], n;
bool valid[N/2];

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%d%d", &p[i], &d[i]);
    
    // 顺时针
    for (int i = 1; i <= n; i++) s[i] = s[i+n] = p[i] - d[i];
    for (int i = 1; i <= 2*n; i++) s[i] += s[i-1];
    
    int tt = -1, hh = 0;
    for (int i = 1; i <= 2*n; i++) {
        while (hh <= tt && i - q[hh] >= n) hh++;
        while (hh <= tt && s[q[tt]] >= s[i]) tt--;
        q[++tt] = i;
        
        if (i >= n && s[q[hh]] - s[i-n] >= 0) valid[i-n+1] = true; 
    }
    
    // 逆时针
    d[0] = d[n];
    for (int i = 1; i <= n; i++) s[i] = s[i+n] = p[i] - d[i-1];
    for (int i = 2*n; i; i--) s[i] += s[i+1];
    tt = -1, hh = 0;
    for (int i = 2*n; i; i--) {
        while (hh <= tt && q[hh] - i >= n) hh++;
        while (hh <= tt && s[q[tt]] >= s[i]) tt--;
        q[++tt] = i;
        if (i <= n+1 && s[q[hh]] - s[i+n] >= 0) valid[i-1] = true;
    }
    
    
    for (int i = 1; i <= n; i++)
        if (valid[i]) puts("TAK");
        else puts("NIE");
    return 0;
}
```

## 烽火传递

> 在某两个城市之间有 n 座烽火台，每个烽火台发出信号都有一定的代价。
>
> 为了使情报准确传递，在连续 m 个烽火台中至少要有一个发出信号。
>
> 现在输入 n,m 和每个烽火台的代价，请计算在两城市之间准确传递情报所需花费的总代价最少为多少。
>
> 题目链接：[AcWing 1089](https://www.acwing.com/problem/content/1091/)。

终于是一个需要状态转移的 DP 题了。

+ 状态表示 $f(i)$：选择第 $i$ 个的所有方案中最优方案的总代价。

+ 状态转移方程：
    $$
    f(i)=w(i)+\min_{j=i-m}^{i-1} f(j)
    $$

+ 初始化：$f(1)=w(1),q[0]=f(0)=0$，因为计算 $f(1)$ 的时候需要用 $0+w(1)$
+ 答案：$\min_{j=n-m+1}^{n} f(j)$

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 200010;
int n, m;
int w[N], f[N], q[N];

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &w[i]);
    
    int tt = -1, hh = 0;
    q[++tt] = 0;
    for (int i = 1; i <= n; i++) {
        while (hh <= tt && i - q[hh] > m) hh++;
        f[i] = f[q[hh]] + w[i];
        while (hh <= tt && f[q[tt]] >= f[i]) tt--;
        q[++tt] = i;
    }
    
    int res = 1e9;
    for (int i = n-m+1; i <= n; i++)
        res = min(res, f[i]);
    printf("%d\n", res);
    return 0;
}
```

## 绿色通道

> 高二数学《绿色通道》总共有 n 道题目要抄，编号 1,2,…,n，抄第 i 题要花 ai 分钟。
>
> 小 Y 决定只用不超过 t 分钟抄这个，因此必然有空着的题。
>
> 每道题要么不写，要么抄完，不能写一半。
>
> 下标连续的一些空题称为一个空题段，它的长度就是所包含的题目数。
>
> 这样应付自然会引起马老师的愤怒，最长的空题段越长，马老师越生气。
>
> 现在，小 Y 想知道他在这 t 分钟内写哪些题，才能够尽量减轻马老师的怒火。
>
> 由于小 Y 很聪明，你只要告诉他最长的空题段至少有多长就可以了，不需输出方案。
>
> 题目链接：[AcWing 1090](https://www.acwing.com/problem/content/1092/)。

本题需要二分答案 + 上题判断。

我们最多能连续跳过 $m'$ 个数字，因此在 $m=m'+1$ 个连续的数字中必须选择一个。

状态依然是 $f(i)$，表示在选择 $i$ 的情况下前面的最佳方案。

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;

const int N = 5e4+10;
int n, t;
int w[N], q[N], f[N];

bool check(int m) {
    ++m;
    
    int tt = 0, hh = 0;
    for (int i = 1; i <= n; i++) {
        while (hh <= tt && i - q[hh] > m) hh++;
        f[i] = w[i] + f[q[hh]];
        while (hh <= tt && f[q[tt]] >= f[i]) tt--;
        q[++tt] = i;
    }
    
    for (int i = n-m+1; i <= n; i++) 
        if (f[i] <= t) return true;
    return false;
}

int main() {
    scanf("%d%d", &n, &t);
    for (int i = 1; i <= n; i++) scanf("%d", &w[i]);
    int l = 0, r = n;
    while (l < r) {
        int mid = l + r >> 1;
        if (check(mid)) r = mid;
        else l = mid+1;
    }
    printf("%d\n", r);
    return 0;
}
```

