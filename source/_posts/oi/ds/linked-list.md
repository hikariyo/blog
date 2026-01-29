---
date: 2023-06-14 11:55:31
updated: 2023-06-14 11:55:31
title: 数据结构 - 静态单双链表
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
description: 在 C++ 中用数组来模拟单双链表，这样的方式也被称为静态链表，下面以两例题记录 C++ 中的链表应该如何定义和使用。
---

## 静态链表

在 C++ 中用数组来模拟单双链表，这样的方式也被称为**静态链表**，下面以两例题记录 C++ 中的链表应该如何定义和使用。

## 单链表

> 题目太长就只把输入输出格式贴过来了。
>
> 第一行包含整数 $M \in [1, 10^5]$，表示操作次数。接下来 $M$ 行，每行包含一个操作命令，操作命令可能为以下几种：
>
> 1. `H x`，表示向链表头插入一个数 $x$。
> 2. `D k`，表示删除第 $k$ 个插入的数后面的数（当 $k$ 为 0 时，表示删除头结点）。
> 3. `I k x`，表示在第 $k$ 个插入的数后面插入一个数 $x$（此操作中 $k$ 均大于 0）。

定义 `ne` 数组作为对应元素的下一个索引，`e` 代表每个元素的值。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1e5+10;
int head = -1, e[N], ne[N];
int p = 0;

void inserthead(int x) {
    e[p] = x;
    ne[p] = head;
    head = p++;
}

void insert(int k, int x) {
    e[p] = x;
    // p.next = k.next
    ne[p] = ne[k];
    // k.next = p
    ne[k] = p++;
}

void remove(int k) {
    // head = head.next
    if (k == -1) head = ne[head];
    // k.next = k.next.next
    else ne[k] = ne[ne[k]];
}

int main() {
    int m;
    char op[2];
    int k, x;
    scanf("%d", &m);
    while (m--) {
        scanf("%s", &op);
        switch (*op) {
            case 'H': scanf("%d", &x); inserthead(x); break;
            case 'D': scanf("%d", &k); remove(k-1); break;
            case 'I': scanf("%d%d", &k, &x); insert(k-1, x); break;
        }
    }
    for (int index = head; index != -1; index = ne[index]) {
        printf("%d ", e[index]);
    }
    return 0;
}
```

### 分析

1. `scanf` 输入字符时最好用字符串，即使是单个字符。
2. 我们输入的 `k` 指的是第 `k` 个插入的元素，与下标 $0, 1, 2, \cdots$ 相比较 `k` 是  $1, 2, 3, \cdots$ 所以 `k-1` 是对应位置。
3. 一定要先在纸上画图之后再写，不要过度相信自己想象的能力。

## 双链表

> 第一行包含整数 $M \in [1, 10^5]$，表示操作次数。接下来 $M$ 行，每行包含一个操作命令，操作命令可能为以下几种：
>
> 1. `L x`，表示在链表的最左端插入数 $x$。
> 2. `R x`，表示在链表的最右端插入数 $x$。
> 3. `D k`，表示将第 $k$ 个插入的数删除。
> 4. `IL k x`，表示在第 $k$ 个插入的数左侧插入一个数。
> 5. `IR k x`，表示在第 $k$ 个插入的数右侧插入一个数。

定义 `e` 代表每个值，`r` 代表右边索引，`l` 代表左边索引。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 1e5+10;
int e[N], r[N], l[N], idx;

void init() {
    l[1] = 0;
    r[0] = 1;
    idx = 2;
}

void insert(int k, int x) {
    e[idx] = x;
    l[idx] = k;
    r[idx] = r[k];
    // k.right.left = idx
    l[r[k]] = idx;
    // k.right = idx
    r[k] = idx++;
}

void remove(int k) {
    // k.left.right = k.right
    r[l[k]] = r[k];
    // k.right.left = k.left
    l[r[k]] = l[k];
}

#define GET(var) scanf("%d", &var)

int main() {
    char op[3];
    int m, k, x;
    GET(m);
    init();
    while (m--) {
        scanf("%s", op);
        if (*op == 'L') {
            GET(x); insert(0, x);
        }
        else if (*op == 'R') {
            GET(x); insert(l[1], x);
        }
        else if (*op == 'D') {
            GET(k); remove(k+1);
        }
        else if (!strcmp(op, "IL")) {
            GET(k); GET(x); insert(l[k+1], x);
        }
        else if (!strcmp(op, "IR")) {
            GET(k); GET(x); insert(k+1, x);
        }
    }
    for (int i = r[0]; i != 1; i = r[i]) {
        printf("%d ", e[i]);
    }
    return 0;
}
```

### 分析

1. 这里用下标为 `0` 的元素储存左端点，为 `1` 的元素储存右端点，这样就可以复用 `l, r` 两个数组了。

2. 下标序列是 $2, 3, 4, \cdots$ 而 `k` 是 $1, 2, 3, \cdots$ 所以操作的数是 `k+1`。

3. 那两个 `strcmp` 可以直接判断 `op[1]` 是否为 `L` 或 `R`，但这里为了直观就留在那里了。

4. 其实我们对于类似 `remove` 函数中复杂的数组操作可以写成下面的样子（这里只是提一下，如果能绕过来还是用普通的写法比较好）：

   ```cpp
   void remove(int k) {
       // k.left.right = k.right
       k[l][r] = k[r];
       // k.right.left = k.left
       k[r][l] = k[l];
   }
   ```

   有了一股 OOP 的味道，但是这个确实是合法的，因为 C++ 中的下标运算符原理是 `a[b]` 等价于 `*(a+b)`，那么 `r[l[k]]` 是 `*(r + (*(l + k)))`，反过来 `k[l][r]` 是 `*(*(k + l) + r)`，所以真的是等价的。这里贴出 `cppreference` 中的解释：

   > The built-in subscript expression E1[E2] is exactly identical to the expression `*(E1 + E2) `
   >
   > From https://en.cppreference.com/w/cpp/language/operator_member_access#Built-in_subscript_operator

