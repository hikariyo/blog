---
date: 2023-06-14 10:30:43
updated: 2023-06-14 10:30:43
title: 数据结构 - 堆
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
---

## 简介

堆是一种**完全二叉树**（从上到下，从左到右排列）这意味着它可以被储存到一个数组中，并且对任意一个节点 $i$ 都有左子节点为 $2i$，右子节点为 $2i+1$，和线段树类似。

当插入元素时，我们需要**下滤**操作来维持堆的单调性，具体操作流程如下：

1. 将当前节点和两个子节点作比较，如果满足单调性不做任何调整。
2. 如果不满足将较小的子节点与父节点交换，使较小的子节点在上面。
3. 递归执行将放下去的大数继续下滤，直到满足单调性或者没有子节点为止。

由于操作次数最多为树的高度，所以这个操作的复杂度是 $O(\log n)$。

## 模板

在 `down` 的时候首先取出来当前结点较小的儿子，然后将其与当前结点元素比较，如果满足性质就停止，否则就下放，并且继续处理它被下放到的节点。

在 `up` 的时候只需要比较当前结点和其父结点是否满足性质即可。

当然可以用 `priority_queue`，不过这里只是给出一个手写模板。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1000010;
int hp[N], tp;

void down(int u) {
    while (1) {
        int a = u << 1, b = u << 1 | 1;
        if (a > tp) break;
        if (a <= tp && b <= tp && hp[a] > hp[b]) swap(a, b);
        if (hp[a] < hp[u]) swap(hp[a], hp[u]);
        else break;
        u = a;
    }
}

void up(int u) {
    while (1) {
        int a = u >> 1;
        if (!a || hp[a] < hp[u]) break;
        swap(hp[a], hp[u]);
        u = a;
    }
}

int main() {
    int n, op, x;
    scanf("%d", &n);
    while (n--) {
        scanf("%d", &op);
        if (op == 1) scanf("%d", &x), hp[++tp] = x, up(tp);
        else if (op == 2) printf("%d\n", hp[1]);
        else swap(hp[1], hp[tp--]), down(1);
    }
    return 0;
}
```

## 可删堆

由于 `multiset` 的常数太大了，当我们需要维护一个支持求最值和加入删除元素的数据结构的时候，用可删堆往往更优。虽然 `priority_queue` 是不支持元素删除的，但是我们可以定义两个堆 $v,d$ 存储集合中的所有元素和被删除的元素，

```cpp
struct RemovableHeap {
    priority_queue<int, vector<int>, greater<int>> v, d;
    void remove(int x) {
        d.push(x);
    }

    int top() {
        while (d.size() && v.top() == d.top()) v.pop(), d.pop();
        return v.top();
    }

    void push(int x) {
        v.push(x);
    }

    int size() {
        return v.size() - d.size();
    }
};
```

