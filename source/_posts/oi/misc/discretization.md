---
date: 2023-06-13 22:50:13
title: 杂项 - 离散化
katex: true
tags:
- Algo
- C++
categories:
- OI
description: 离散化的目的是将数值大且数量少的一组数据映射为一组较小的数据，比如 [-10^9, 10^9] 范围的数据映射到 [0, 10^5]，从而更方便地用数组储存一些数据。在实战中，通常将这组数据存储到 vector 中之后排序并去重，然后用下标表示某个数字，即将数字映射为下标。
---

## 离散化

离散化的目的是将**数值大且数量少**的一组数据映射为一组较小的数据，比如将 $[-10^9, 10^9]$ 范围的数据映射到 $[0, 10^5]$，从而更方便地用数组储存一些数据。

在实战中，通常将这组数据存储到 `vector` 中之后排序并去重，然后用下标表示某个数字，即将数字映射为下标：

```cpp
// requires #include <algorithm>
sort(nums.begin(), nums.end());
nums.erase(unique(nums.begin(), nums.end()), nums.end())
```

关于 `unique` 函数，可以参考 `cppreference` 上给出的文档，为了不改变其原意，这里贴出英文原文：

>Eliminates all except the first element from every consecutive group of equivalent elements from the range [`first`, `last`) and returns a past-the-end iterator for the new *logical* end of the range.
>
>From https://en.cppreference.com/w/cpp/algorithm/unique

简单意译一下：在 $\rm [first, last)$ 的区间中删掉所有连续相等的元素组，但保留这些组的第一个元素，并且返回新序列的 `past-the-end` 迭代器（相当于 `items.end()` 取到的迭代器）。因此，我们需要先**排序**再**去重**并且还要**删掉多余元素**。

## 例 区间和

> 假定有一个无限长的数轴（长度在 $[-10^9, 10^9]$ 间），数轴上每个坐标上的数都是 0。现在，我们首先进行 $n \in [1, 10^5]$ 次操作，每次操作将某一位置 $x \in [-10^9, 10^9]$ 上的数加 $c \in [-10^4, 10^4]$。接下来，进行 $m \in [1, 10^5]$ 次询问，每个询问包含两个整数 $l, r \in [-10^9, 10^9]$，你需要求出在区间 $[l, r]$ 之间的所有数的和。
>
> 输入格式：第一行包含两个整数 $n$ 和 $m$。接下来 $n$ 行，每行包含两个整数 $x$ 和 $c$。再接下来 $m$ 行，每行包含两个整数 $l$ 和 $r$。

我们需要对所有输入的下标进行离散化处理，所以用数组 `indexes` 保存这些下标，最多会有 $n+2m \le 3\times10^5$ 个不同的下标，所以开辟几个数组来储存数据并且用前缀和求区间和即可。

但是我们在输入这些操作时并不知道下标具体的离散化结果是多少，所以要暂存起来，用 `adds` 和 `sums` 两个 `vector` 把这些请求都存起来。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 3e5+10;
vector<int> indexes;
vector<pair<int, int>> adds, sums;

int a[N], S[N];
int n, m;

int find(int x) {
    int l = 0, r = indexes.size() - 1;
    while (l < r) {
        int mid = l+r >> 1;
        if (indexes[mid] >= x) r = mid;
        else l = mid+1;
    }
    // !important: 因为使用前缀和，所以这里下标从1开始计算
    return r+1;
}

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 0; i < n; i++) {
        int x, c;
        scanf("%d%d", &x, &c);
        adds.push_back({x, c});
        indexes.push_back(x);
    }
    for (int i = 0; i < m; i++) {
        int l, r;
        scanf("%d%d", &l, &r);
        sums.push_back({l, r});
        indexes.push_back(l);
        indexes.push_back(r);
    }
    
    // 去重
    sort(indexes.begin(), indexes.end());
    indexes.erase(unique(indexes.begin(), indexes.end()), indexes.end());
    
    // 将所有加的操作应用到真正的数组上
    for (auto add: adds) {
        int x = find(add.first);
        a[x] += add.second;
    }
    
    // 计算前缀和
    for (int i = 1; i <= indexes.size(); i++) S[i] = S[i-1] + a[i];
    
    // 处理区间和
    for (auto sum: sums) {
        int l = find(sum.first), r = find(sum.second);
        printf("%d\n", S[r] - S[l-1]);
    }
    return 0;
}
```

