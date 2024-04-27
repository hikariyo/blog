---
date: 2023-11-04 22:21:00
title: 杂项 - 挂分合集
katex: true
tags:
- Algo
- C++
categories:
- OI
description: 整理一下遇到的各种各样奇葩的问题，和一些令人智熄的迷惑操作。
---

## 动态扩容 vector 导致地址变化

来源于主席树的[提交记录](https://www.luogu.com.cn/record/131336952)，观察下面的 `13, 14` 行。

```cpp
vector<Node> tr = {Node{0, 0, 0}};

int create() {
    int idx = tr.size();
    tr.push_back(Node{0, 0, 0});
    return idx;
}

int build(int L, int R) {
    int p = create();
    if (L == R) return p;
    int mid = (L+R) >> 1;
    tr[p].ls = build(L, mid);
    tr[p].rs = build(mid+1, R);
    return p;
}
```

这个 RE 的原因是，在 `build` 的过程中会导致 `tr` 进行扩容，然后 `tr[p].ls` 的地址就发生了变化。应该是 GCC 先把左边的地址取出来然后再调用的函数，这就导致了问题。

对于这个问题，我去 [Compiler Explorer](https://godbolt.org/) 看了一下，对于下面这个类似的代码：

```cpp
#include <vector>
std::vector<int> a;

int foo() {
    a[0] = foo();
    return 0;
}
```

在赋值这一行，NOI Linux 支持的 GCC 9.3 编译出来（已将 ABI 标识符转化为源程序标识符）的是：

```asm
mov     esi, 0
mov     edi, OFFSET FLAT:a
call    std::vector<int, std::allocator<int> >::operator[](unsigned long)
mov     rbx, rax
call    foo()
mov     DWORD PTR [rbx], eax
```

在 GCC 11.4 编译出来的是：

```asm
call    foo()
mov     ebx, eax
mov     esi, 0
mov     edi, OFFSET FLAT:a
call    std::vector<int, std::allocator<int> >::operator[](unsigned long)
mov     DWORD PTR [rax], ebx
```

虽然我看不懂汇编，但是可以看出前者先把 `a[0]` 的地址取出来之后再调用的函数，后者是先调用函数，显然我们的问题就出自这里。

解决方法就是，可以先调一下 `reserve` 预分配空间，但是还不如直接开个静态数组方便。

## 矩阵初始化用未读入的全局变量

在初始化矩阵的时候，我写过这个东西：

```cpp
struct Matrix {
    LL dat[N][N];
    
    Matrix(int n) {
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++)
                dat[i][j] = INF;
    }
} f;
```

问题在于实例化 `f` 的时候 `n` 压根就没读入，然后本地一直没过样例，所以就没在提交记录里。

## FHQ-Treap 分裂找前驱

来自这个[提交记录](https://www.luogu.com.cn/record/132718333)，找前驱的时候直接按照 `v` 分裂了，实际上应该是 `v-1`。

```cpp
int pre(int v) {
    int x, y;
    split(root, v, x, y);
    int p = kth(x, tr[x].sz);
    root = merge(x, y);
    return p;
}
```

## 快速幂底数爆 LL

来自某次模拟赛。

```cpp
LL qmi(LL a, LL k, LL p) {
    LL res = 1;
    while (k) {
        if (k & 1) res = res * a % p;
        a = a * a % p;
        k >>= 1;
    }
    return res;
}
```

不正常的是传进来的 `a` 可能比 `p` 大，需要先取模一次。

## N 方双指针

建珂朵莉树的时候双指针处理连续同一元素的段，然后忘了更新 `i=j-1` 导致变成了$O(n^2)$ 的双指针，在这个[提交记录](https://www.luogu.com.cn/record/133324413)。

```cpp
void build() {
    for (int i = 1; i <= n; i++) {
        int j = i+1;
        while (j <= n && a[j] == a[i]) j++;
        // i ~ j-1
        tr.insert({i, j-1, a[i]});
    }
}
```

这个题也犯过上面快速幂爆 LL 的错误，不过是在另一个提交记录中，懒得找了。

## 隐式多测不清空

在这个[提交记录](https://www.luogu.com.cn/record/133046528)中，我考虑到换根重新处理倍增数组的时候，如果不需要处理可以直接跳过，这相当于常数级优化，然后就挂了 55pts。

```cpp
void calc_f(int u, int s) {
    if (f[u][0] == s) return;
    f[u][0] = s;
    for (int k = 1; k < K; k++) f[u][k] = f[f[u][k-1]][k-1];
}
```

原因就是在多测时后几次重新覆盖倍增数组的时候，就会被错误的优化掉。

## int char 不分

出自 [Trie 模板题提交记录](https://www.luogu.com.cn/record/133374863)。

```cpp
for (char ch = 'a'; ch <= 'z'; ch++) mp[ch] = ch - 'a';
for (char ch = 'A'; ch <= 'Z'; ch++) mp[ch] = ch - 'A' + 26;
for (char ch = '0'; ch <= 9; ch++) mp[ch] = ch - '0' + 52;
```

## 弄混最优性和可行性问题

题目是类似求一个完全背包能凑出的所有方案中小于某个数最大的值，但是我上来就把值给模了一个 $\text{lcm}$，当然这在可行性问题里面是一个降复杂度的正确方法，但是题目是最优性问题。

## 看错数据范围导致爆 LL

李超树需要维护的范围是 $0\sim 10^6$，我看另一个数据范围 $0\sim 2\times 10^{11}$ 被带歪了，然后斜率的范围是 $-2\times 10^{11}\sim 0$，然后就爆 `long long` 了：[提交记录](https://www.luogu.com.cn/record/135065575)。

## 次小值

最小值初始化成 $a_0$，然后遍历元素的时候还是从 $0\sim n-1$ 遍历，如果最小值就是 $a_0$ 的话这会导致求出来的次小值同样是 $a_0$。来自 NOIP 2023（然而 CCF 给我放过去了，但这并不能改变我 NOIP 爆炸的事实）。

```cpp
string MIN = B[0], SMIN = "{";
int MINP = 0;
for (int i = 0; i < n; i++) {
    if (B[i] <= MIN) SMIN = MIN, MIN = B[i], MINP = i;
    else if (B[i] <= SMIN) SMIN = B[i];
}
```

正确做法是从 $1\sim n-1$ 遍历。

## 错误选择 INF

来自[提交记录](https://www.luogu.com.cn/record/136239386)，用 `0x3f3f3f3f` 当 `INF`，然后输入的最大值是 $2^{31}-1$，遗憾离场。

## 动态 MST 不判一开始是否连通

[BZOJ2238](https://darkbzoj.cc/problem/2238)，可能存在一开始不存在 MST 的情况。

