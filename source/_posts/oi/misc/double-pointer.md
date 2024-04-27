---
date: 2023-6-13 21:13:14
title: 杂项 - 双指针
katex: true
tags:
- Algo
- C++
categories:
- OI
---

## 双指针

双指针是在暴力解法可行的情况下进行优化，达到降低**时间复杂度**的结果。通常来说，它能将一个嵌套循环（平方复杂度）优化为一层循环（线性复杂度）。

一般地，双指针算法的模板与下面给出代码类似：

```cpp
for (int i = 0, j = 0; i < n; i++) {
    while (cond(i, j, m)) j++;
    foo();
}
```

下文均为例题。

## 最长连续不重复子序列

> 给定一个长度为 $n \in [1, 10^5]$ 的整数序列 $\{a_n\}$，满足 $a_i\in[0, 10^5]$，请找出最长的不包含重复的数的连续区间，输出它的长度。

这里用 `num` 数组当作一个哈希表，储存某个数字出现的次数。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1e5+10;
int a[N], num[N];
int n;

int main() {
    scanf("%d", &n);
    for (int i = 0; i < n; i++) scanf("%d", a+i);
    
    int ans = 0;
    for (int i = 0, j = 0; i < n; i++) {
        num[a[i]]++;
        while (j <= i && num[a[i]] > 1) {
            num[a[j]]--;
            j++;
        }
        ans = max(ans, i-j+1);
    }
    
    printf("%d\n", ans);
    return 0;
}
```

### 分析

1. `i` 代表右边界，`j` 代表左边界。
2. 每次循环进入时都将 `a[i]` 的出现次数增1，如果在 `i, j` 中间出现了重复的数字，**一定是新加入的数字重复**，故判断条件是 `num[a[i]] > 1`。
3. 如果 `i` 向右移动后 `j` 向左存在不重复的子序列，那么在 `i=i-1` 的时候 `j` 左就应该有不重复的子序列，所以 `j` 只能向右移动。

## 数组元素的目标和

> 给定两个升序排序的有序数组 $A, B$ 长度分别为 $n, m \in [1, 10^5]$，保证数组内元素各不相同，以及一个目标值 $x$。数组下标从 0 开始，求满足 $A_i+B_j=x$ 的数对 $(i, j)$，数据保证有唯一解。

可以一个指针从左边开始，另一个从右边开始，如果 $A_i+B_j < x$ 就自增左指针，否则自减右指针，直到找到符合要求的一组数对。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1e5+10;
int A[N], B[N];

int main() {
    int n, m, x;
    scanf("%d%d%d", &n, &m, &x);
    for (int i = 0; i < n; i++) scanf("%d", A+i);
    for (int i = 0; i < m; i++) scanf("%d", B+i);
    
    for (int i = 0, j=m-1; i < n; i++) {
        while (A[i] + B[j] >= x) {
            if (A[i] + B[j] == x) {
                printf("%d %d", i, j);
                return 0;
            }
            j--;
        }
    }
    return 0;
}
```

如果你不想要这么多嵌套，可以这样调整。

```cpp
for (int i = 0, j = m-1; i < n; i++) {
    while (A[i] + B[j] > x) j--;
    if (A[i] + B[j] == x) {
        printf("%d %d", i, j);
        break;
    }
}
```

