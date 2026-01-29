---
date: 2023-07-02 10:14:47
updated: 2023-07-04 14:28:22
title: 动态规划 - 最长上升子序列
katex: true
tags:
- Algo
- C++
- DP
categories:
- OI
---

## 模板

最长上升子序列问题有 DP 和 贪心 两种解法，见[前文](https://blog.hikariyo.net/posts/oi/dp/linear/)，这里再写一遍思路。

+ 表示：`f[i]` 是以第 `i` 元素结尾的子序列。
+ 存储：这些子序列中的长度最大值。
+ 状态转移方程：`f[i] = max(f[i], f[j]+1)`，其中 `j` 是 `0~i` 中能与第 `i` 元素构成上升子序列的所有元素下标。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010;
int f[N], a[N];

int main() {
	int n;
	cin >> n;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
        f[i] = 1;
	}
	
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < i; j++) {
			if (a[j] < a[i]) f[i] = max(f[i], f[j]+1);
		}
	}
	
	cout << *max_element(f, f+n) << endl;
	return 0;
}
```

贪心做法分析如下：

+ 开数组 `q[N]`，其中 `q[i]` 表示长度为 `i` 的上升子序列中末尾的最小元素。
+ 对元素 `x` 进行二分查找 `q[j]`，其中 `q[j] < x <= q[j+1]`，这里数组 `q` 一定是单调自增的。
+ 覆盖 `q[j+1] = x`。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1e5+10;
int q[N], a[N];

int main() {
	int n;
	cin >> n;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		q[i] = -2e9;
	}
	
	int len = 0;
	for (int i = 0; i < n; i++) {
		int l = 0, r = len, x = a[i];
		while (l < r) {
			int mid = l+r >> 1;
			if (q[mid] >= x) r=mid;
			else l=mid+1;
		}
		q[l] = x;
		len = max(len, l+1); 
	}
	cout << len << endl;
	return 0;
}
```

这里通过二分找到的坐标 `l` 是上文中提到的 `j+1`，推理一下，当 `j+1 = 0` 的时候这个长度应当是 `1`，所以更新的时候用 `l+1`。可以用 `lower_bound` 实现二分：

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1e5+10;
int q[N], a[N];

int main() {
	int n;
	cin >> n;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		q[i] = -2e9;
	}
	
	int len = 0;
	for (int i = 0; i < n; i++) {
		int j = lower_bound(q, q+len, a[i]) - q;
		q[j] = a[i];
		len = max(len, j+1); 
	}
	cout << len << endl;
	return 0;
}
```

这里给出 `lower_bound` 的定义：

> ```cpp
> // until C++20
> template<class ForwardIt, class T>
> ForwardIt lower_bound(ForwardIt first, ForwardIt last, const T& value);
> 
> template<class ForwardIt, class T, class Compare>
> ForwardIt lower_bound(ForwardIt first, ForwardIt last,
>                        const T& value, Compare comp);
> ```
>
> Returns an iterator pointing to the **first** element in the range `[first, last)` that does not satisfy element < value (or comp(element, value)), (i.e. greater or equal to), or **last** if no such element is found.
>
> From https://en.cppreference.com/w/cpp/algorithm/lower_bound

`upper_bound` 与它基本上是一样的，唯一的不同点在于：

+ `lower_bound` 返回大于等于给定元素的首个位置。
+ `upper_bound` 返回大于给定元素的首个位置。

它的返回值是一个指针或者迭代器，如果传进去的是数组返回的就是 `int*`，将其与数组首元素指针相减，返回 `ptrdiff_t`（这里用 `int` 足够，具体参考 [cppreference](https://en.cppreference.com/w/cpp/types/ptrdiff_t) 给出的说明）就可以得到数组中的索引。

## 合唱队形

> N 位同学站成一排，音乐老师要请其中的 (N−K) 位同学出列，使得剩下的 K 位同学排成合唱队形。     
>
> 合唱队形是指这样的一种队形：设 K 位同学从左到右依次编号为 1，2…，K，他们的身高分别为 T1，T2，…，TK，  则他们的身高满足 `T1<...<Ti>Ti+1>...>Tk(1 <= i <= k)`。
>
> 你的任务是，已知所有 N 位同学的身高，计算最少需要几位同学出列，可以使得剩下的同学排成合唱队形。
>
> 原题：[AcWing 482](https://www.acwing.com/problem/content/484/)。

思路是分别求出头部和尾部开始的最大递增子序列，找看加起来的最大值，然后用总数减去这个最大值就是最小出列人数。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 110;
int a[N], f[N], g[N];

int main() {
	ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
	int n;
	cin >> n;
	for (int i = 0; i < n; i++) {
		cin >> a[i];
		f[i] = g[i] = 1;
	}
	
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < i; j++) {
			if (a[j] < a[i]) f[i] = max(f[i], f[j]+1);
		}
	}
	
	for (int i = n-1; i >= 0; i--) {
		for (int j = n-1; j > i; j--) {
			if (a[j] < a[i]) g[i] = max(g[i], g[j]+1);
		}
	}
	int ans = 0;
	for (int i = 0; i < n; i++) ans = max(ans, f[i] + g[i] - 1);  // 有一个人重复
	cout << n-ans << endl;
	return 0;
}
```

## 友好城市

> 某国有一条横贯东西的大河，河有笔直的南北两岸，岸上各有位置各不相同的N个城市。
>
> 北岸的每个城市有且仅有一个友好城市在南岸，而且不同城市的友好城市不相同。每对友好城市都向政府申请在河上开辟一条直线航道连接两个城市，但是由于河上雾太大，政府决定避免任意两条航道交叉，以避免事故。编程帮助政府做出一些批准和拒绝申请的决定，使得在保证任意两条航线不相交的情况下，被批准的申请尽量多。
>
> 原题：[AcWing 1012](https://www.acwing.com/problem/content/1014/)

分析：先针对一条岸边排序，然后如果是一个合法的没有交叉的解，那么这些航线所对应的另一岸的城市序号一定是递增的，所以这还是一个最长上升子序列问题。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

#define fi first
#define se second

const int N = 5010;
typedef pair<int, int> pii;
pii a[N];
int f[N];

int main() {
	ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
	int n;
	cin >> n;
	for (int i = 1; i <= n; i++) {
		cin >> a[i].fi >> a[i].se;
		f[i] = 1;
	}
    // important: sort a[1] ~ a[n]
	sort(a+1, a+1+n);
	int ans = 0;
	for (int i = 1; i <= n; i++) {
		for (int j = 1; j < i; j++) {
			if (a[j].se < a[i].se) {
				f[i] = max(f[i], f[j]+1);
			}
		}
		ans = max(ans, f[i]);
	}
	cout << ans << endl;
	return 0;
}
```

## 拦截导弹

> 某国为了防御敌国的导弹袭击，发展出一种导弹拦截系统。但是这种导弹拦截系统有一个缺陷：虽然它的第一发炮弹能够到达任意的高度，但是以后每一发炮弹都不能高于前一发的高度。某天，雷达捕捉到敌国的导弹来袭。由于该系统还在试用阶段，所以只有一套系统，因此有可能不能拦截所有的导弹。
>
> 输入导弹依次飞来的高度（雷达给出的高度数据是不大于30000的正整数，导弹数不超过1000），计算这套系统最多能拦截多少导弹，如果要拦截所有导弹最少要配备多少套这种导弹拦截系统。
>
> 原题：[AcWing 1010](https://www.acwing.com/problem/content/1012/)。

这题有两问，第一问是比较简单的求最大**不上升子序列**问题，第二问是求**不上升子序列**的最少个数，下面分析第二问。

对于任意一个元素 `a[i]`，存在两种处理方式：

1. 新建一个序列，让它成为第一个元素。
2. 添加到现有序列中，这需要使该序列的最后一个元素 `>=` 当前元素 `a[i]`。

可以创建一个数组 `g[N]`，对于其中元素 `g[i]` 的含义是第 `i` 个子序列的**末尾元素**。这里贪心的思路是找到一个**大于当前元素**的所有末尾元素中**最小**的 `g[j]`，然后将当前元素添加到第 `j` 序列，如果找不到能添加进去的现有序列，就创建一个新的序列。

这里可以证明出 `g` 数组一定是单调递增的，反证法：假设插入了 `g[j+k] = a[i]` 后导致 `g[j] > g[j+k]`，那么会有 `a[i] < g[j]`，那么也就不会插到 `g[j+k]` 的位置了，所以矛盾，证出数组 `g` 单调递增。

如果最优解不是按照贪心算法来的话，那么由于此方案并没有尽可能给下个元素提供更优的条件，所以是劣于贪心解的，有 **最优解 >= 贪心解**；同时，贪心解一定是一个合法的解，所以 **贪心解 >= 最优解**，于是贪心解就是最优的。这里先给出第二问的代码，下面会补全。

```cpp
int cnt = 0;
for (int i = 0; i < n; i++) {
    int j = 0;
    while (j < cnt && g[j] < a[i]) j++;
    // 创建新序列
    if (j == cnt) cnt++;
    g[j] = a[i];
}
```

由于数组 `g` 是单调递增的，所以可用二分查找优化。

```cpp
int cnt = 0;
for (int i = 0; i < n; i++) {
    int j = lower_bound(g, g+cnt, a[i]) - g;
    if (j == cnt) cnt++;
    g[j] = a[i];
}
```

可以发现这和求最大上升子序列一模一样，这不是巧合，是 Dilworth 定理，但我看不懂所以这里就先放了。下面给出全部代码。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 1010;
int f[N], g[N];

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int a;
    int len = 0, cnt = 0;
    while (cin >> a) {
        int k = upper_bound(f, f+len, a, greater<int>()) - f;
        if (k == len) len++;
        f[k] = a;

        int j = lower_bound(g, g+cnt, a) - g;
        if (j == cnt) cnt++;
        g[j] = a;
    }
    cout << len << endl << cnt << endl;

    return 0;
}
```

这里求**不上升子序列**的时候反而用到了 `upper_bound`，原因在于这里计算的是元素数量，而重复的元素是需要被重复计算的，如果用 `lower_bound` 会导致重复元素被舍掉。

注：`f` 里面保存的是最长不上升子序列的值。

由于 `f` 一定非严格单调递减，所以这里又传入了比较函数 `greater<int>` 来找末尾大于当前元素的所有子序列中末尾最小的那个。

对于 `while(cin >> a)` 这句，参考以下文档：

> ```cpp
> explicit operator bool() const;  // since C++11
> ```
>
> Checks whether the stream has no errors.
>
> From https://en.cppreference.com/w/cpp/io/basic_ios/operator_bool

此外，`operator >>` 返回的是 `cin` 本身，所以它可以直接扔到 `while` 语句中。

## 导弹防御系统

> 为了对抗附近恶意国家的威胁，R 国更新了他们的导弹防御系统。
>
> 一套防御系统的导弹拦截高度要么一直 **严格单调** 上升要么一直 **严格单调** 下降。
>
> 例如，一套系统先后拦截了高度为 3 和高度为 4 的两发导弹，那么接下来该系统就只能拦截高度大于 4 的导弹。
>
> 给定即将袭来的一系列导弹的高度，请你求出至少需要多少套防御系统，就可以将它们全部击落。
>
> 原题：[AcWing 187](https://www.acwing.com/problem/content/189/)。

这题对每个元素都有两种放置方法，分别是放到不上升子序列中和不下降子序列中，所以用爆搜解决。

```cpp
#include <iostream>
#include <algorithm>
using namespace std;

const int N = 55;
int a[N], up[N], down[N], n, ans;

void dfs(int u, int sup, int sdown) {
    // 舍掉劣于当前最优解的所有子树
    if (sup + sdown >= ans) return;
    
    if (u == n) {
        ans = sup + sdown;
        return;
    }
    
    // 当前元素加到上升子序列中
    {
        int k = 0;
        while (k < sup && up[k] > a[u]) k++;
        int t = up[k];
        up[k] = a[u];
        // 新建序列
        if (k == sup) dfs(u+1, sup+1, sdown);
        else dfs(u+1, sup, sdown);
        up[k] = t;
    }
    
    // 加到下降子序列中
    {
        int k = 0;
        while (k < sdown && down[k] < a[u]) k++;
        int t = down[k];
        down[k] = a[u];
        if (k == sdown) dfs(u+1, sup, sdown+1);
        else dfs(u+1, sup, sdown);
        down[k] = t;
    }
}

int main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    while (cin >> n, n) {
        for (int i = 0; i < n; i++) cin >> a[i];
        // 最坏情况
        ans = n;
        dfs(0, 0, 0);
        cout << ans << endl;
    }
    return 0;
}
```

这地方要是用 `upper_bound` 反而是负优化。

## 最长公共上升子序列

> 对于两个数列 A 和 B，如果它们都包含一段位置不一定连续的数，且数值是严格递增的，那么称这一段数是两个数列的公共上升子序列，而所有的公共上升子序列中最长的就是最长公共上升子序列了。
>
> 原题：[AcWing 272](https://www.acwing.com/problem/content/274/)。

类比最长上升子序列的 DP 做法，那里的 `f[i]` 表示以 `a[i]` 结尾的最长上升子序列，对本题分析如下：

+ 状态表示 `f[i, j]`：
    + 集合：所有在 `A[1 ~ i]` 与 `B[1 ~ j]` 中以 `B[j]` 结尾的公共上升子序列的集合。
    + 存储：最大值。
+ 状态转移：
    + 不包含 `A[i]`：`f[i-1][j]`。
    + 包含 `A[i]`，此时 `A[i] == B[j]`，去掉 `B[j]` 同时也就去掉了 `A[i]` 后的所有子序列最大值加一，注意这里包含只有 `B[j]` 一个元素的情况：`max(1, f[i-1][k]+1, ...),k=1~j`。

```cpp
int ans = 0;
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
        // 不包含 a[i]
        f[i][j] = f[i-1][j];

        // 包含 a[i]
        if (a[i] == b[j]) {
            int maxv = 1;
            // 去掉 b[j] 后的所有子序列的最大值 + 1
            for (int k = 1; k < j; k++)
                if (b[k] < b[j])
                    maxv = max(maxv, f[i-1][k] + 1);

            f[i][j] = max(f[i][j], maxv);
        }

        ans = max(ans, f[i][j]);
    }
}
```

这有三重循环，观察最内层循环，`b[k] < b[j]` 等价于 `b[k] < a[i]`，而 `k=1~j` 再循环 `j=1~n` 时可以同步进行，所以等价变换后的代码如下：

```cpp
int ans = 0;
for (int i = 1; i <= n; i++) {
    int maxv = 1;
    for (int j = 1; j <= n; j++) {
        f[i][j] = f[i-1][j];
        if (b[j] < a[i]) maxv = max(maxv, f[i-1][j]+1);
        if (a[i] == b[j]) f[i][j] = max(f[i][j], maxv);
        ans = max(ans, f[i][j]);
    }
}
```

更进一步地，发现计算 `f[i]` 层时仅依赖 `f[i-1]` 层，因此可以删掉第一维。

```cpp
int ans = 0;
for (int i = 1; i <= n; i++) {
    int maxv = 1;
    for (int j = 1; j <= n; j++) {
        if (b[j] < a[i]) maxv = max(maxv, f[j]+1);
        if (a[i] == b[j]) f[j] = max(f[j], maxv);
        ans = max(ans, f[j]);
    }
}
```

