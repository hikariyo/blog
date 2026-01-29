---
date: 2023-06-17 22:04:13
updated: 2023-07-04 23:21:14
title: 动态规划 - 背包问题
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 01背包问题

> 有 N 件物品和一个容量是 V 的背包。每件物品只能使用一次。第 i 件物品的体积是 vi，价值是 wi。
>
> 求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大，输出最大价值。
>
> 原题：[AcWing 2](https://www.acwing.com/problem/content/2/)。

动态规划最入门的问题，我们开辟一个二维数组 `f[i][j]`，它的含义是在前 `i` 个物品中在体积最大为 `j` 的基础上的最优解。

可以得到**状态转移方程**：`f[i][j] = max(f[i-1][j], f[i-1][j-v[i]] + w[i])`，它的意思是在 `i, j` 的位置的最优解等同于不放 `i` 和放 `i` 两种情况得到的最大值。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010;
int f[N][N];
int w[N], v[N];
int n, m;

int main() {
    cin >> n >> m;
    // 这里是第一个易错点，背包问题中下标以 1 开始方便计算
    for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            if (j >= v[i])
                f[i][j] = max(f[i-1][j], f[i-1][j-v[i]] + w[i]);  // 状态转移方程
            else f[i][j] = f[i-1][j];
        }
    }
    cout << f[n][m] << endl;
    return 0;
}
```

观察到每次对 `i` 循环时实际上只需要用到 `i-1` 时产生的数据，即**上一轮循环产生的数据**，所以这个二维数组可以被优化为一维数组。

```cpp
int f[N];

// ...

for (int i = 1; i <= n; i++) {
    // 这是有问题的 下面说明
    for (int j = 1; j <= m; j++) {
        if (j >= v[i]) f[j] = max(f[j], f[j-v[i]] + w[i]);
    }
}
```

我们把所有关于 `i` 的层级删掉，把 `f[j]` 就当作上一次循环留下的结果，这也正是我们这个问题产生的原因：`j` 从左向右遍历的话会把**上一次循环留下的结果覆盖**，如果从右向左就不覆盖前面的内容了。

```cpp
for (int i = 1; i <= n; i++) {
    for (int j = m; j >= v[i]; j--) {
        f[j] = max(f[j], f[j-v[i]] + w[i]);
    }
}
```

这样就正确了，顺便我们可以控制循环范围来去掉那个 `if` 语句。

## 完全背包问题

> 有 N 种物品和一个容量是 V 的背包，每种物品都有**无限件**可用。
>
> 第 i 种物品的体积是 vi，价值是 wi。求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。输出最大价值。
>
> 原题：[AcWing 3](https://www.acwing.com/problem/content/3/)。

与 01背包问题 类似，可以有状态转移方程：
$$
f(i,j)=\max \{f(i-1,j),f(i-1,j-v_i)+w_i,f(i-1,j-2v_i)+2w_i\cdots\}
$$
可以写出下面的代码：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010;
int f[N][N], v[N], w[N];
int n, m;

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            // 注意这里是 <= j 因为是对当前的容量计算的最优解
            for (int k = 0; k*v[i] <= j; k++) {
                // 这里左边要写 f[i][j] 而不是 f[i-1][j]
                // 这是因为当 k=0 时右边就是 f[i-1][j] 所以这个情况是会被算法考虑进去的
                f[i][j] = max(f[i][j], f[i-1][j-v[i]*k] + w[i]*k);
            }
        }
    }
    cout << f[n][m] << endl;
    return 0;
}
```

但是三重循环会超时，所以考虑优化，不难得到下式：
$$
f(i,j-v_i)=\max\{f(i-1, j-v_i),f(i-1, j-2v_i)+w_i,f(i-1,j-3v_i)+2w_i\cdots\}
$$
可以看到它和 $f(i, j)$ 的第二项（包含）后面的所有项只差了一个 $w_i$，所以我们可以做这个优化：

```cpp
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= m; j++) {
        if (j >= v[i])
            // 和 01背包 唯一的区别就是这里从 [i, j-v[i]] 转移
            // 而 01背包 从 [i-1, j-v[i]] 转移
            f[i][j] = max(f[i-1][j], f[i][j-v[i]] + w[i]);
        else
            f[i][j] = f[i-1][j];
    }
}
```

仿照 01背包 还可以把 `f[i][j]` 优化成一维。

```cpp
int f[N];

for (int i = 1; i <= n; i++) {
    for (int j = v[i]; j <= m; j++) {
        f[j] = max(f[j], f[j-v[i]] + w[i]);
    }
}
```

每次循环时 `f[]` 都保存着上一次的结果，而我们根据上一次的结果更新这一次的结果时，`max` 右边必须与当前同一层，所以没有 01背包 需要逆序遍历的问题。

## 多重背包问题

> 有 N 种物品和一个容量是 V 的背包，第 i 种物品最多有 si 件，每件体积是 vi，价值是 wi。
>
> 求解将哪些物品装入背包，可使物品体积总和不超过背包容量，且价值总和最大。输出最大价值。
>
> 原题：[AcWing 4](https://www.acwing.com/problem/content/4/)，[AcWing 5](https://www.acwing.com/problem/content/5/)，[AcWing 6](https://www.acwing.com/problem/content/6/)。

### 朴素做法

相较于完全背包问题，这里仅仅是多了一个数量的限制，所以从最朴素的做法看起，我们只需要在遍历 `k` 的时候加一个限制条件即可。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 110;
int f[N][N], v[N], w[N], s[N];
int n, m;

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> v[i] >> w[i] >> s[i];
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            for (int k = 0; k <= s[i] && v[i]*k <= j; k++) {
                f[i][j] = max(f[i][j], f[i-1][j-v[i]*k] + w[i]*k);
            }
        }
    }
    cout << f[n][m] << endl;
    return 0;
}
```

### 二进制优化

把每件数量为 $s$ 商品拆成 $1, 2, 4, \cdots, 2^k(2^k\le s)$ 件，由于它们都可以表示为首位为 $1$ 后面是若干 $0$ 的二进制，所以它们一定能组合出 $[1,2^k]$ 的任意数字。再加上 $s-2^k$ 可以组合出 $[1, s]$ 的任意数字。

之后就可以把这些商品选择或者不选，转化成 01背包问题。

这里要注意，对于题目所给的 $N$ 个商品，每个商品都可以拆成最多 $\log s$ 件，所以这里的数组要开到 $N\log s$ 大小。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

// N<=1000, s<=2000
const int N = 15000;
int n, m;
int f[N], w[N], v[N];

int main() {
    cin >> n >> m;
    int cnt = 0;
    for (int i = 1; i <= n; i++) {
        int a, b, s;
        cin >> a >> b >> s;
        int k = 1;
        for (int k = 1; k <= s; k *= 2) {
            // 把 k 个物品放在一起
            cnt++;
            v[cnt] = k*a;
            w[cnt] = k*b;
            // 这 k 个物品分完了，从总物品中减去
            s -= k;
        }
        if (s > 0) {
            cnt++;
            v[cnt] = s*a;
            w[cnt] = s*b;
        }
    }
    // 这里更新成 cnt 个物品
    n = cnt;
    for (int i = 1; i <= n; ++i) {
        for (int j = m; j >= v[i]; --j) {
            f[j] = max(f[j], f[j-v[i]] + w[i]);
        }
    }
    cout << f[m] << endl;
    return 0;
}
```

### 单调队列

前文中提到过二进制优化方式，但它的复杂度是 $O(nv\log s)$，通过单调队列的滑动窗口优化可以优化到 $O(nv)$。

+ 状态表示 $f(i,j)$：

    + 元素：放置第  $1\to i$ 个物品，容积为 $j$ 时的所有方案。
    + 存储：这些方案的最大值。

+ 状态转移：

    + 对于任意的 $f(i, j)$ 为简化书写设 $f_j=f(i-1, j)$ 都有：
        $$
        f(i,j)=\max\{f_j,f_{j-v}+w,f_{j-2v}+2w\cdots,f_{j-sv}+sw\}
        $$
        仿照完全背包问题的思路：
        $$
        f(i,j-v)=\max \{f_{j-v},f_{j-2v}+w,f_{j-3v}+2w\cdots,f_{j-(s+1)v}+sw\}
        $$
        将这些排成一列，设 $r=j\bmod v$，有：
        $$
        \begin{aligned}
        f_r,f_{r+v},f_{r+2v},\cdots,f_{j-(s-1)v},[f_{j-sv},f_{j-(s+1)v}\cdots,f_{j-2v},f_{j-v},f_j] &\to f(i,j)\\
        f_r,f_{r+v},f_{r+2v},\cdots,[f_{j-(s-1)v},f_{j-sv},f_{j-(s+1)v}\cdots,f_{j-2v},f_{j-v}],f_j &\to f(i,j-v)\\
        f_r,f_{r+v},f_{r+2v},\cdot[\cdot\cdot,f_{j-(s-1)v},f_{j-sv},f_{j-(s+1)v}\cdots,f_{j-2v}],f_{j-v},f_j &\to f(i,j-2v)\\
        \end{aligned}
        $$
        并且对于 $f(i, j)$ 对应的任一个 $f_{j-kv}$ 的权值都为 $kw$。

+ 单调队列：储存的元素依然为下标。

    + 对于任意一个元素 $q$，计算其权值时只是一个简单的一元一次方程：
        $$
        j-kv=q\Rightarrow k=(j-q)/v
        $$

    + 当计算队头 $q_0=r+kv$ 是否需要滑出时，令 $j=r+mv$，思路类似：
        $$
        \begin{aligned}
        m=(j-r)/v,k=(q_0-r)/v\\
        m-k\gt s\Rightarrow j-q_0>sv
        \end{aligned}
        $$

+ 复杂度：对于每组余数相同的容积，都通过单调队列求滑动窗口中的最值，加上外层的枚举每个物品，这个时间复杂度就是 $O(nv)$。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010, M = 20010;

int n, m;
// q: 对每组除以 v 余数相同的容积都有可能加入队列 因此它的数量级与背包容积相同
// f: 第一维是物品 第二维是容积
int q[M], f[N][M];

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        int v, w, s;
        cin >> v >> w >> s;
        for (int r = 0; r < v; r++) {
            int tt = -1, hh = 0;
            for (int j = r; j <= m; j += v) {
                // 滑出窗口的元素出队
                if (hh <= tt && j - q[hh] > s * v) hh++;
                // 不满足单调递减的元素出队
                while (hh <= tt && f[i-1][j] >= f[i-1][q[tt]] + (j - q[tt]) / v * w) tt--;
                q[++tt] = j;
                // 队头为最大值
                f[i][j] = f[i-1][q[hh]] + (j - q[hh]) / v * w;
            }
        }
    }

    cout << f[n][m];

    return 0;
}
```

可以用滚动数组优化空间复杂度，方法就是所有第一维都 `&1` 然后定义换成 `f[2][M]` 即可，这里不再演示。

## 混合背包问题

> 有 N 种物品和一个容量是 V 的背包。
>
> 物品一共有三类：
>
> - 第一类物品只能用1次（01背包）；
> - 第二类物品可以用无限次（完全背包）；
> - 第三类物品最多只能用 si 次（多重背包）；
>
> 每种体积是 vi，价值是 wi。
>
> 求解将哪些物品装入背包，可使物品体积总和不超过背包容量，且价值总和最大。输出最大价值。
>
> 链接：[AcWing 7](https://www.acwing.com/problem/content/7/)。

### 单调队列

当成多重背包的特殊情况做就行，这里用滑动窗口优化，定义一个宏 `val` 方便操作。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010;
int f[N][N], q[N];

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        int v, w, s;
        cin >> v >> w >> s;
        if (s == -1) s = 1;
        else if (s == 0) s = N;

        for (int r = 0; r < v; r++) {
            int tt = -1, hh = 0;
            for (int j = r; j <= m; j += v) {
                #define val(x) (f[i-1][x] + (j-x)/v*w)
                if (hh <= tt && j - q[hh] > s*v) hh++;
                while (hh <= tt && val(q[tt]) <= val(j)) tt--;
                q[++tt] = j;
                f[i][j] = val(q[hh]);
            }
        }
    }
    cout << f[n][m];

    return 0;
}
```

### 二进制

也可以划分成完全背包问题和多重背包问题，这里用二进制优化写一下。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010;
int f[N];

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        int v, w, s;
        cin >> v >> w >> s;
        if (s == 0) {
            // 完全背包
            for (int j = v; j <= m; j++)
                f[j] = max(f[j], f[j-v]+w);
        } else {
            // 多重背包
            if (s == -1) s = 1;
            for (int k = 1; k <= s; k <<= 1) {
                for (int j = m; j >= k*v; j--) {
                    f[j] = max(f[j], f[j-k*v]+w*k);
                }
                s -= k;
            }
            if (s) {
                for (int j = m; j >= s*v; j--) {
                    f[j] = max(f[j], f[j-s*v]+s*w);
                }
            }
        }
    }
    cout << f[m];
    
    return 0;
}
```

对于二进制优化的部分，这里就不额外开辟数组了，由于内部进行的始终是优化成 01背包问题 后的那个内层循环，所以这么写是可行的。实测在这个题的数据量下，单调队列要比二进制快一倍少一点。

## 分组背包问题

> 有 N 组物品和一个容量是 V 的背包。每组物品有若干个，**同一组内的物品最多只能选一个**。每件物品的体积是 vij，价值是 wij，其中 i 是组号，j 是组内编号。
>
> 求解将哪些物品装入背包，可使物品总体积不超过背包容量，且总价值最大。输出最大价值。
>
> 原题：[AcWing 9](https://www.acwing.com/problem/content/9/)。

循环组内编号时要考虑到本组物品不放的情况，也就是下面代码中的 `k=0~s[i]` 而不是从 1 开始。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 110;
int n, m;
int w[N][N], v[N][N], s[N], f[N];

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        cin >> s[i];
        for (int j = 1; j <= s[i]; j++) {
            cin >> v[i][j] >> w[i][j];
        }
    }
    for (int i = 1; i <= n; i++) {
        for (int j = m; j >= 1; j--) {
            for (int k = 0; k <= s[i]; k++) {
                if (j >= v[i][k]) f[j] = max(f[j], f[j-v[i][k]] + w[i][k]);
            }
        }
    }
    cout << f[m] << endl;
    return 0;
}
```

同样，对于这个可以进行优化，把 `f` 变成一维数组。

```cpp
for (int i = 1; i <= n; i++) {
    for (int j = m; j >= 1; j--) {
        for (int k = 0; k <= s[i]; k++) {
            if (j >= v[i][k]) f[j] = max(f[j], f[j-v[i][k]] + w[i][k]);
        }
    }
}
```

由于这里用了 `i-1` 层的数据，所以需要自右向左遍历。

## 二维费用的背包问题

> 有 N 件物品和一个容量是 V 的背包，背包能承受的最大重量是 M。每件物品只能用一次。体积是 vi，重量是 mi，价值是 wi。求解将哪些物品装入背包，可使物品总体积不超过背包容量，总重量不超过背包可承受的最大重量，且价值总和最大。输出最大价值。
>
> 原题：[AcWing 8](https://www.acwing.com/problem/content/8/)。

思路与 01背包问题 十分类似。
$$
f(i,j,k)=\max\{f(i-1,j,k),f(i-1,j-v_1,k-v_2)+w\}
$$
原始代码：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010, M = 110;
int f[N][M][M];

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int n, m1, m2;
    cin >> n >> m1 >> m2;
    for (int i = 1; i <= n; i++) {
        int v1, v2, w;
        cin >> v1 >> v2 >> w;
        for (int j = 1; j <= m1; j++) {
            for (int k = 1; k <= m2; k++) {
                if (j >= v1 && k >= v2) {
                    f[i][j][k] = max(f[i-1][j][k], f[i-1][j-v1][k-v2] + w);
                } else {
                    f[i][j][k] = f[i-1][j][k];
                }
            }
        }
    }
    cout << f[n][m1][m2];
    
    return 0;
}
```

可以把状态优化成二维的：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010, M = 110;
int f[M][M];

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int n, m1, m2;
    cin >> n >> m1 >> m2;
    for (int i = 1; i <= n; i++) {
        int v1, v2, w;
        cin >> v1 >> v2 >> w;
        for (int j = m1; j >= v1; j--) {
            for (int k = m2; k >= v2; k--) {
                f[j][k] = max(f[j][k], f[j-v1][k-v2] + w);
            }
        }
    }
    cout << f[m1][m2];

    return 0;
}
```

## 树形背包

参考本站关于树形 DP 的文章。