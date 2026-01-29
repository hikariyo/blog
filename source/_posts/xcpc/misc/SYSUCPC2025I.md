---
date: 2025-11-02 20:10:00
updated: 2025-11-02 20:10:00
title: "[SYSUCPC 2025 I] Sum"
katex: true
tags:
- Algo
- C++
- Math
categories:
- XCPC
description: 数论中常有的小范围暴力、大范围枚举的思想。
---

## 题目

[Gym 106114 I](https://codeforces.com/gym/106114/problem/I)。

## 思路

对于这一类数论的题目，经常会有针对数据范围的分治。

不难发现，二进制下 $n$ 最多有 $\log_2n\le 40$ 个 $1$，所以答案一定是 $\le 40$ 的。

具体地说，我们考虑分解之后的位数。

如果位数是 $1$ 位，要求最高位不为 $0$，那么此时答案就是 $n$，根据上面的结论我们知道 $n$ 在 $>40$ 的时候一定不是答案；

如果两位，$n=c_1r+c_2\le r^2-1$；

如果三位，$n=c_1r^2+c_2r+c_3\le r^3-1$；

随着位数的增加，$r$ 的最小值会不断减小。

我们知道答案 $\le 40$，所以可以枚举三个 $c_1,c_2,c_3$，通过求根公式求出是否有这样的 $r$，复杂度是 $O(\log^3 n)$。

这样的话，我们成功处理了 $r>\sqrt[3]{n}$ 的部分。对于 $r\le \sqrt[3]{n}$ 的部分，直接暴力，复杂度是 $O(\sqrt[3]{n}\log n)$。

所以总复杂度为 $O(t\log^3n+t\sqrt[3]{n}\log n)$。题解给的枚举两位 $O(t\log^2n+t\sqrt{n}\log n)$ 我没有通过，不知道是不是常数太大。

## 实现

实现上来说，我们可以让 $n\le 10^5$ 和 $R\le \sqrt[3]{n}$ 都直接暴力，保证 $R$ 和 $n$ 都充分大，然后根据上面的思路进行。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define int long long
int bruteforce(int n, int R) {
    int ans = 1e9;
    for (int r = 2; r <= R; r++) {
        int x = n;
        int sum = 0;
        while (x) {
            sum += x % r;
            x /= r;
        }
        ans = min(ans, sum);
    }
    return ans;
}

int solve(int n, int R) {
    if (n <= 1e5) return bruteforce(n, R);
    
    int m = powl(n, 1.0l / 3.0l);
    if (R <= m) return bruteforce(n, R);
    int ans = bruteforce(n, m);
    for (int c1 = 0; c1 <= ans; c1++) {
        for (int c2 = 0; c1+c2 <= ans; c2++) {
            for (int c3 = 0; c1+c2+c3 <= ans; c3++) {
                if (c1 == 0 && c2 == 0) break;
                if (c1 == 0) {
                    if ((n - c3) % c2) continue;
                    int r = (n - c3) / c2;
                    if (r <= m || r > R || c1 >= r || c2 >= r || c3 >= r) continue;
                    ans = min(ans, c1+c2+c3);
                }
                else {
                    int delta = c2 * c2 - 4 * c1 * (c3 - n);
                    if (delta < 0) continue;
                    int k = sqrtl(delta);
                    if (k * k != delta) continue;
                    if (-c2 + k < 0 || (-c2 + k) % (2 * c1)) continue;
                    int r = (-c2 + k) / (2 * c1);
                    if (r <= m || r > R || c1 >= r || c2 >= r || c3 >= r) continue;
                    ans = min(ans, c1+c2+c3);
                }
            }
        }
    }
    return ans;
}

signed main() {
    ios::sync_with_stdio(false); cin.tie(0); cout.tie(0);
    int T;
    cin >> T;
    while (T--) {
        int n, R;
        cin >> n >> R;
        cout << solve(n, R) << '\n';
    }
    return 0;
}
```

