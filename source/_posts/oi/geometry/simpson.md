---
date: 2024-01-27 22:51:00
update: 2024-01-28 04:57:00
title: 计算几何 - 自适应辛普森积分法
katex: true
description: 辛普森积分法是一种求数值积分的方法，用二次函数拟合被积函数达到较好的效果，还可以通过分治自适应地保证精度。本文也比较详细地记录了辛普森积分法误差的推导过程。
tags:
- Algo
- C++
- Geometry
categories:
- OI
---

## 简介

辛普森积分法是一种求数值积分的方法，用二次函数拟合被积函数达到较好的效果，还可以通过分治自适应地保证精度。本文也比较详细地记录了辛普森积分法误差的推导过程。

## 原理

在 $[l,r]$ 区间内用二次函数 $P(x)=ax^2+bx+c$ 来代替 $f(x)$，取 $l,r,\frac{l+r}{2}$ 三个点确定二次函数的参数：
$$
\begin{aligned}
\int_l^rP(x)\mathrm{d}x&=\int_l^r(ax^2+bx+c)\mathrm{d}x\\
&=\frac{a}{3}(r^3-l^3)+\frac{b}{2}(r^2-l^2)+c(r-l)\\
&=\frac{r-l}{6}[2a(r^2+l^2+lr)+3b(r+l)+6c]\\
&=\frac{r-l}{6}[ar^2+br+c+al^2+bl+c+a(r^2+l^2+2lr)+2b(r+l)+4c]\\
&=\frac{r-l}{6}[f(r)+f(l)+4f(\frac{r+l}{2})]
\end{aligned}
$$
下面用 $x_0,x_1,x_2$ 代表这三个点，并且 $x_0<x_1<x_2$。

### 牛顿插值法

首先给出差商的定义，$0$ 阶差商的定义是 $f[x_0]=f(x_0)$，对 $k(k\ge 1)$ 阶差商的定义是：
$$
f[x_0,\dots,x_k]=\frac{f[x_0,\dots,x_{k-1}]-f[x_1,\dots,x_{k}]}{x_0-x_k}
$$
把定义的式子变形可以写出下面的这些式子：
$$
\begin{aligned}
f[x]&=f[x_0]+f[x,x_0](x-x_0)\\
f[x,x_0]&=f[x_0,x_1]+f[x,x_0,x_1](x-x_1)\\
f[x,x_0,x_1]&=f[x_0,x_1,x_2]+f[x,x_0,x_1,x_2](x-x_2)\\
f[x,x_0,\dots,x_k]&=f[x_0,\dots,x_k]+f[x,\dots,x_{k+1}](x-x_{k+1})
\end{aligned}
$$
定义 $\omega_n(x)=\prod_{i=0}^{n-1} (x-x_i)$，当插入三个点时，有：
$$
f(x)=f[x_0]+f[x_0,x_1]\omega_1(x)+f[x_0,x_1,x_2]\omega_2(x)+f[x,x_0,x_1,x_2]\omega_3(x)=P(x)+R(x)
$$

其中 $P(x)$ 为前三项组成的插值多项式，$R(x)$ 为插值余项。

### 插值余项

用 $P(x)$ 代表前面的多项式，那么 $R(x)$ 余项为：
$$
R(x)=f[x,x_0,x_1,x_2]\omega_3(x)
$$
下面求参数，构造函数 $\varphi(t)$ 至少有 $x,x_0,x_1,x_2$ 四个零点：
$$
\varphi(t)=f(t)-P(t)-f[x,x_0,x_1,x_2]\omega_3(t)
$$
根据罗尔定理，$\varphi'(t)$ 至少有三个零点，推导若干次得到 $\varphi^{(3)}(t)$ 至少有一个零点 $\xi(x)$，于是：
$$
\varphi^{(3)}(\xi(x))=f^{(3)}(\xi(x))-3!f[x,x_0,x_1,x_2]=0
$$
得到 $R(x)$ 的表达式：
$$
R(x)=f[x,x_0,x_1,x_2]\omega_3(x)=\frac{f^{(3)}(\xi(x))}{3!}\omega_3(x)
$$
上面的步骤可以推广到 $k+1$ 阶的情况：
$$
f[x,x_0,\dots,x_k]=\frac{f^{(k+1)}(\xi(x))}{(k+1)!}
$$
求余项积分的时候会用到。

### 插值余项积分

下面定积分的计算全都是 [Wolfram Alpha](https://www.wolframalpha.com/) 完成的，这里就没必要自己去算了。

由于我们选择的 $x_1$ 是 $x_0,x_2$ 的中点，有 $h=x_2-x_1=x_1-x_0$，设 $g(x)$ 为：
$$
g(x)=\int_{x_0}^x\omega_3(t)\mathrm{d}t=\frac{1}{4}(x-x_0)^2(x-x_2)^2\\
$$
可以得到 $R(x)$ 的表达式，并对其积分：
$$
\begin{aligned}
\int_{x_0}^{x_2}R(x)\mathrm{d}x&=\int_{x_0}^{x_2}\frac{f^{(3)}(\xi(x))}{3!}g'(x)\mathrm{d}x\\
&=\left[\frac{f^{(3)}(\xi(x))}{3!}g(x)\right]_{x_0}^{x_2}-\int_{x_0}^{x_2}g(x)\mathrm{d}\frac{f^{(3)}(\xi(x))}{3!}\\
&=-\int_{x_0}^{x_2}g(x)\frac{\mathrm{d}}{\mathrm{d}x}\frac{f^{(3)}(\xi(x))}{3!}\mathrm{d}x
\end{aligned}
$$
套用差商的定义：
$$
\begin{aligned}
\frac{\mathrm{d}}{\mathrm{d}x}\left(\frac{f^{(3)}(\xi(x))}{3!}\right)&=\lim_{\Delta x\to 0}\frac{f[x+\Delta x,x_0,x_1,x_2]-f[x,x_0,x_1,x_2]}{\Delta x}\\
&=\lim_{\Delta x\to 0}f[x,x+\Delta x,x_0,x_1,x_2]\\
&=\frac{f^{(4)}(\xi(x))}{4!}
\end{aligned}
$$

导函数和原函数中的 $\xi(x)$ 应当是不同的，但是这里它并不重要，我就不再引入新符号让读者更加难以阅读了。由于 $g(x)\ge0$ 恒成立，根据积分第一中值定理，$\exist c_0\in[x_0,x_2]$ 使得：
$$
\begin{aligned}
\int_{x_0}^{x_2}R(x)\mathrm{d}x&=-\int_{x_0}^{x_2}\frac{f^{(4)}(\xi(x))}{4!}g(x)\mathrm{d}x\\
&=-\frac{f^{(4)}(c_0)}{4!}\int_{x_0}^{x_2}g(x)\mathrm{d}x\\
&=-\frac{h^5}{90}f^{(4)}(c_0)
\end{aligned}
$$

因此得到最终的式子：
$$
\int_{x_0}^{x_2}f(x)\mathrm{d}x=\int_{x_0}^{x_2}P(x)\mathrm{d}x-\frac{h^5}{90}f^{(4)}(c_0)
$$

### 误差分析

令 $S(a,b)=\int_a^bP(x)\mathrm{d}x$ 即实际运算得到的结果，那么计算的时候会算上子区间的结果，也就是说：
$$
S(x_0,x_2)-S(x_0,x_1)-S(x_1,x_2)=\frac{h^5}{90}f^{(4)}(c_0)-\frac{1}{32}\frac{h^5}{90}[f^{(4)}(c_1)+f^{(4)}(c_2)]
$$
认为 $c_0\approx c_1\approx c_2$ 就可以得到：
$$
S(x_0,x_2)-S(x_0,x_1)-S(x_1,x_2)\approx\frac{15}{16}\frac{h^5}{90}f^{(4)}(c_0)
$$
也就是说：
$$
\begin{aligned}
\int_{x_0}^{x_2}f(x)\mathrm{d}x&=S(x_0,x_1)+S(x_1,x_2)-\frac{1}{32}\frac{h^5}{90}[f^{(4)}(c_1)+f^{(4)}(c_2)]\\
&\approx S(x_0,x_1)+S(x_1,x_2)-\frac{1}{16}\frac{h^5}{90}f^{(4)}(c_0)\\
&\approx S(x_0,x_1)+S(x_1,x_2)-\frac{1}{15}[S(x_0,x_2)-S(x_0,x_1)-S(x_1,x_2)]
\end{aligned}
$$
设允许的误差为 $\epsilon$，那么进行递归调用的时候左右区间的精度都应要求 $\frac{\epsilon}{2}$，保证加起来误差不超过 $\epsilon$，判断是否足够接近的条件：
$$
|S(x_0,x_1)+S(x_1,x_2)-S(x_0,x_2)|<15\epsilon
$$
### 实现

如果用上面分析出的结论，应该是这么写：

```cpp
double calc(double l, double r) {
    return (r-l) / 6 * (f(l) + f(r) + 4*f((l+r)/2));
}

double simpson(double l, double r, double eps, double s) {
    double mid = (l+r) / 2, ls = calc(l, mid), rs = calc(mid, r);
    if (fabs(ls+rs-s) < 15*eps) return ls+rs-(s-ls-rs)/15;
    return simpson(l, mid, eps / 2, ls) + simpson(mid, r, eps / 2, rs);
}
```

实际上，由于算法竞赛对时间要求较高，所以当精度要求不是很高的时候，直接无视上面的结论，这样写一样可以：

```cpp
double calc(double l, double r) {
    return (r-l) / 6 * (f(l) + f(r) + 4*f((l+r)/2));
}

double simpson(double l, double r, double s) {
    double mid = (l+r) / 2, ls = calc(l, mid), rs = calc(mid, r);
    if (fabs(ls+rs-s) < eps) return ls+rs;
    return simpson(l, mid, ls) + simpson(mid, r, rs);
}
```

这两种模板之间的选择：

1. 如果函数的连续性较好，尤其是具有解析式的初等函数，用第一版较好。
2. 如果函数是离散的，尤其是求若干个图形的面积并，并且数据量较大时，用第二版较好。

第二版可以通过设置最少递归次数的方式来保证精度，在下面的代码中有，这里不多说了。

## 模板

> 计算积分，保证区间内分母不为零并且是收敛的：
> $$
> \int_L^R\frac{cx+d}{ax+b}\mathrm{d}x
> $$
> 数据范围：$-10\le a,b,c,d\le 10,-100\le L\lt R\le 100,R-L\ge 1$。
>
> 题目链接：[P4525](https://www.luogu.com.cn/problem/P4525)。

这里演示一下完整的模板。

```cpp
#include <bits/stdc++.h>
using namespace std;

const double eps = 1e-6;
double a, b, c, d, l, r;

double f(double x) {
    return (c*x+d)/(a*x+b);
}

double calc(double l, double r) {
    return (r-l) / 6 * (f(l) + f(r) + 4*f((l+r)/2));
}

double simpson(double l, double r, double eps, double s) {
    double mid = (l+r) / 2, ls = calc(l, mid), rs = calc(mid, r);
    if (fabs(ls+rs-s) < 15*eps) return ls+rs-(s-ls-rs)/15;
    return simpson(l, mid, eps / 2, ls) + simpson(mid, r, eps / 2, rs);
}

int main() {
    scanf("%lf%lf%lf%lf%lf%lf", &a, &b, &c, &d, &l, &r);
    printf("%.6lf\n", simpson(l, r, eps, calc(l, r)));
    return 0;
}
```

## Ellipse

> 给定椭圆 $\frac{x^2}{a^2}+\frac{y^2}{b^2}=1$，求 $[L,R]$ 之间椭圆的面积。
>
> 题目链接：[HDU1724](https://acm.hdu.edu.cn/showproblem.php?pid=1724)。

转换成解析式为 $y=b\sqrt{1-\frac{x^2}{a^2}}$ 的函数，面积就是 $2\int_L^R y\mathrm{d}x$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const double eps = 1e-6;
int T;
double a, b, l, r;

double f(double x) {
    return b*sqrt(1-x*x/a/a);
}

double calc(double l, double r) {
    return (r-l) / 6 * (f(l) + f(r) + 4*f((l+r)/2));
}

double simpson(double l, double r, double eps, double s) {
    double mid = (l+r) / 2, ls = calc(l, mid), rs = calc(mid, r);
    if (fabs(ls+rs-s) < 15*eps) return ls+rs+(s-ls-rs)/15;
    return simpson(l, mid, eps / 2, ls) + simpson(mid, r, eps / 2, rs);
}

int main() {
    scanf("%d", &T);
    while (T--) scanf("%lf%lf%lf%lf", &a, &b, &l, &r), printf("%.3lf\n", 2*simpson(l, r, eps, calc(l, r)));
    return 0;
}
```

## 圆的面积并

> 给定 $N$ 个圆，保证 $N\le 1000$，求面积并。
>
> 题目链接：[BZOJ2178](https://darkbzoj.cc/problem/2178)。

为了防止构造出卡掉取 $l,r,\frac{l+r}{2}$ 这三个点拟合曲线的恶意数据，我们需要给所有点旋转一点角度，让这种数据难以卡掉。

具体地说，就是在极坐标下 $(\rho,\theta)$ 旋转至 $(\rho,\theta+\alpha)$，其中 $\alpha$ 是一个随机的角度。反映到直角坐标系，用和角公式展开就能得到：
$$
\begin{aligned}
(x,y)&=(\rho\cos\theta,\rho\sin\theta)\\
(x',y')&=(\rho\cos(\theta+\alpha),\rho\sin(\theta+\alpha))\\
&=(\rho\cos\theta\cos\alpha-\rho\sin\theta\sin\alpha,\rho\sin\theta\cos\alpha+\rho\cos\theta\sin\alpha)\\
&=(x\cos\alpha-y\sin\alpha,x\sin\alpha+y\cos\alpha)
\end{aligned}
$$
对于 $x_0$ 处的函数值 $f(x_0)$ 代表所有圆与直线 $x=x_0$ 交线长，然后积起来就是面积了。

为了更好的使数据连续，把所有圆在 $x$ 轴上的投影进行区间合并，然后逐个这样可以防止出现一大段函数值都为 $0$ 导致的难以拟合。

调试过程中发现对于 `std::sort` 需要保证比较函数当 $a_i<a_j$ 时不能有 $a_j<a_i$ 否则就会 RE，由于浮点数比较起来比较恶心人，所以这里就直接按照左边界排序，不是字典序排序了。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define pb push_back
const double eps = 1e-6, INF = 1e8;
int n;
struct Circle {
    double x, y, r;
} c[1010];

struct Segment {
    double l, r;
    bool operator<(const Segment& seg) const {
        if (fabs(l - seg.l) < eps) return false;
        return l < seg.l;
    }
};

void merge(vector<Segment>& seg) {
    int n = seg.size(), cnt = 0;
    sort(seg.begin(), seg.end());
    for (int i = 0; i < n; i++) {
        int j = i+1;
        double l = seg[i].l, r = seg[i].r;
        while (j < n && seg[j].l < r + eps) r = max(r, seg[j].r), j++;
        seg[cnt++] = {l, r};
        i = j-1;
    }
    seg.resize(cnt);
}

double f(double x) {
    static vector<Segment> seg;
    seg.clear();
    seg.reserve(n);
    double ans = 0;
    int tp = 0;
    for (int i = 0; i < n; i++) {
        double d = fabs(c[i].x - x);
        if (d > c[i].r - eps) continue;
        double k = sqrt(c[i].r*c[i].r - d*d);
        seg.pb({c[i].y - k, c[i].y + k});
    }
    merge(seg);
    for (const auto& s: seg) ans += s.r - s.l;
    return ans;
}

double calc(double l, double r) {
    return (r-l) / 6 * (f(l) + f(r) + 4*f((l+r)/2));
}

double simpson(double l, double r, double ans) {
    double mid = (l+r) / 2, L = calc(l, mid), R = calc(mid, r);
    if (fabs(L+R-ans) < eps) return L+R;
    return simpson(l, mid, L) + simpson(mid, r, R);
}

int main() {
    double alpha = 0.514, sa = sin(alpha), ca = cos(alpha);
    scanf("%d", &n);

    vector<Segment> rgs;
    rgs.reserve(n);
    for (int i = 0, x, y, r; i < n; i++) {
        scanf("%d%d%d", &x, &y, &r);
        c[i] = {x*ca - y*sa, x*sa + y*ca, 1.*r};
        rgs.pb({c[i].x - c[i].r, c[i].x + c[i].r});
    }

    merge(rgs);
    double res = 0;
    for (const auto& rg: rgs) res += simpson(rg.l, rg.r, calc(rg.l, rg.r));
    return !printf("%.3lf\n", res);
}
```

## [CQOI2005] 三角形面积并

> 给定 $n\le 100$ 个三角形的顶点，求面积并。
>
> 题目链接：[P4406](https://www.luogu.com.cn/problem/P4406)。

已经知道是直线了，所以可以直接用一次函数近似，由于直线被 Hack 的可能性更高，所以需要强制至少进行若干次递归保证精度。那么积分公式就是：

$$
\begin{aligned}
\int_l^rf(x)\mathrm{d}x&\approx\int_l^r(kx+b)\mathrm{d}x\\
&=\frac{r-l}{2}[k(r+l)+2b]\\
&=\frac{r-l}{2}[f(l)+f(r)]
\end{aligned}
$$
为了避免直线斜率不存在，旋转一个随机角度。实际上用二次函数拟合也可以，但是在同样要求 $15$ 次递归的情况下直线更快，因为这些边本来就是直线。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 110, INF = 1e8;
const double eps = 1e-8;
double alpha, sina, cosa;
int n;

struct Point {
    double x, y;

    void init(double a, double b) {
        x = a * cosa - b * sina, y = a * sina + b * cosa;
    }
};

struct Segment {
    double l, r;
    bool operator<(const Segment& seg) const {
        if (fabs(l - seg.l) < eps) return false;
        return l < seg.l;
    }
} segs[N], rgs[N];

struct Line {
    double k, b, l, r;

    void init(Point p1, Point p2) {
        k = (p1.y - p2.y) / (p1.x - p2.x);
        b = p1.y - k * p1.x;
        l = min(p1.x, p2.x);
        r = max(p1.x, p2.x);
    }

    double calc(double x) {
        return k * x + b;
    }
};

struct Triangle {
    Point p[3];
    Line line[3];
    double l, r;

    void init(Point pnt[]) {
        l = INF, r = -INF;
        for (int i = 0; i < 3; i++) p[i] = pnt[i], l = min(l, p[i].x), r = max(r, p[i].x);
        line[0].init(p[0], p[1]);
        line[1].init(p[0], p[2]);
        line[2].init(p[1], p[2]);
    }

    Segment calc(double x) {
        double low = INF, high = -INF;
        for (int i = 0; i < 3; i++) {
            if (line[i].l - eps < x && x < line[i].r + eps) {
                double f = line[i].calc(x);
                low = min(low, f);
                high = max(high, f);
            }
        }
        return {low, high};
    }
} t[N];

double f(double x) {
    int tp = 0;
    for (int i = 1; i <= n; i++)
        if (t[i].l - eps < x && x < t[i].r + eps)
            segs[++tp] = t[i].calc(x);
    sort(segs+1, segs+1+tp);

    double ans = 0;
    for (int i = 1; i <= tp; i++) {
        int j = i+1;
        double r = segs[i].r;
        while (j <= tp && segs[j].l < r + eps) r = max(r, segs[j].r), j++;
        ans += r - segs[i].l;
        i = j-1;
    }
    return ans;
}

double calc(double l, double r) {
    return (r-l) * (f(r) + f(l)) / 2;
}

double simpson(double l, double r, double s, int steps = 15) {
    double mid = (l+r) / 2, ls = calc(l, mid), rs = calc(mid, r);
    if (steps < 0 && fabs(ls + rs - s) < eps) return ls + rs;
    return simpson(l, mid, ls, steps - 1) + simpson(mid, r, rs, steps - 1);
}

int main() {
    static Point p[3];
    alpha = 0.114514, sina = sin(alpha), cosa = cos(alpha);
    scanf("%d", &n);
    int k = 0;
    for (int i = 1; i <= n; i++) {
        double a, b;
        for (int j = 0; j < 3; j++) scanf("%lf%lf", &a, &b), p[j].init(a, b);
        t[i].init(p);
        rgs[++k] = {t[i].l, t[i].r};
    }

    sort(rgs+1, rgs+1+k);
    double ans = 0;
    for (int i = 1; i <= k; i++) {
        int j = i+1;
        double l = rgs[i].l, r = rgs[i].r;
        while (j <= k && rgs[j].l < r + eps) r = max(r, rgs[j].r), j++;
        ans += simpson(l, r, calc(l, r));
        i = j-1;
    }
    return !printf("%.2lf\n", ans);
}
```

## [NOI2005] 月下柠檬树

> 给定 $n-1$ 个轴线在同一直线上并且第 $i$ 个的上面半径与 $i+1$ 的下面半径相同的圆台和最上面一个圆锥，求它被与地面角度成 $\alpha$ 的平行光投到地面上的投影。
>
> 数据范围：$1\le n\le 500$，$0.3\le \alpha\lt \frac{\pi}{2}$，$0\lt h_i,r_i\le 100$。
>
> 题目链接：[P4207](https://www.luogu.com.cn/problem/P4207)。

洛谷有图，我不搬运过来了。本题的重点在于投影之后的图形是什么样子的，由于一个圆被平行光投影之后仍然是一个圆，所以猜测一个圆台投影之后是两个圆和两条公切线。

根据几何关系，在高度为 $h$ 的点投影距离原点的距离是 $x=h\cot\alpha$，然后半径不变，所以重点就在于求公切线。

设直线 $l:y=k(x-x_0)$ 是圆 $i,i+1$ 的公切线，分两个情况：

1. 当 $r_i>r_{i+1}$ 的时候，设 $k<0$，倾斜角的补角是 $\theta$，有 $x_0>x_{i+1}>x_i$：
   $$
   \begin{cases}
   x_0-x_i=\frac{r_i}{\sin\theta}\\
   x_0-x_{i+1}=\frac{r_{i+1}}{\sin\theta}
   \end{cases}\Rightarrow \sin\theta=\frac{r_i-r_{i+1}}{x_{i+1}-x_i}
   $$

2. 当 $r_i<r_{i+1}$ 的时候，设 $k=\tan\theta>0$，有 $x_0<x_i<x_{i+1}$：
   $$
   \begin{cases}
   x_i-x_0=\frac{r_i}{\sin\theta}\\
   x_{i+1}-x_0=\frac{r_{i+1}}{\sin\theta}\\
   \end{cases}\Rightarrow \sin\theta=\frac{r_i-r_{i+1}}{x_i-x_{i+1}}
   $$

根据 $\theta$ 求出 $k$ 后再求 $x_0$ 就能得到切线方程，并且根据切线方程求出两个切点。特别地，对于最后一个圆锥，设 $r_{n+1}=0$​ 就行了。

这里 $f(x_0)$ 的含义是对直线 $x=x_0$ 所在的圆及其所在的切线上求到的值的最大值，一开始我甚至根据这个图形的轮廓来判断它处在哪个地方，但是那样写起来码量太大了，承受不了，所以才用这个方式。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 510, INF = 1e8;
const double eps = 1e-8;
int n;
double cota, x[N], r[N];

struct Line {
    double k, x0, a, b;

    double calc(double x) {
        return k * (x - x0);
    }
} line[N];
int lines;

double geta(double c, double b) {
    return sqrt(c*c-b*b);
}

void calclines() {
    for (int i = 1; i <= n; i++) {
        if (x[i+1] - x[i] < fabs(r[i] - r[i+1])) continue;
        double sint, cost, k, x0, a, b;
        if (r[i] < r[i+1]) {
            sint = (r[i] - r[i+1]) / (x[i] - x[i+1]), cost = sqrt(1 - sint * sint);
            k = sint / cost;
            x0 = -r[i] / sint + x[i];
            a = x0 + sqrt((x[i] - x0) * (x[i] - x0) - r[i] * r[i]) * cost;
            b = x0 + sqrt((x[i+1] - x0) * (x[i+1] - x0) - r[i+1] * r[i+1]) * cost;
        }
        else {
            sint = (r[i+1] - r[i]) / (x[i] - x[i+1]), cost = sqrt(1 - sint * sint);
            k = -sint / cost;
            x0 = r[i] / sint + x[i];
            a = x0 - sqrt((x[i] - x0) * (x[i] - x0) - r[i] * r[i]) * cost;
            b = x0 - sqrt((x[i+1] - x0) * (x[i+1] - x0) - r[i+1] * r[i+1]) * cost;
        }
        line[++lines] = {k, x0, a, b};
    }
}

double f(double x1) {
    double ans = 0;
    for (int i = 1; i <= n; i++) if (fabs(x1 - x[i]) < r[i] + eps) ans = max(ans, geta(r[i], x1 - x[i]));
    for (int i = 1; i <= lines; i++) if (line[i].a - eps < x1 && x1 < line[i].b + eps) ans = max(ans, line[i].calc(x1));
    return ans;
}

double calc(double l, double r) {
    return (r-l) * (f(l) + f(r) + 4*f((l+r)/2)) / 6;
}

double simpson(double l, double r, double s) {
    double mid = (l+r) / 2, ls = calc(l, mid), rs = calc(mid, r);
    if (fabs(ls+rs-s) < eps) return ls+rs;
    return simpson(l, mid, ls) + simpson(mid, r, rs);
}

int main() {
    scanf("%d%lf", &n, &cota), cota = 1.0 / tan(cota);
    for (int i = 1; i <= n+1; i++) scanf("%lf", &x[i]), x[i] = x[i] * cota + x[i-1];
    for (int i = 1; i <= n; i++) scanf("%lf", &r[i]);
    calclines();

    ++n;
    double a = INF, b = -INF;
    for (int i = 1; i <= n; i++) a = min(a, x[i] - r[i]), b = max(b, x[i] + r[i]);
    return !printf("%.2lf\n", 2.0 * simpson(a, b, calc(a, b)));
}
```

