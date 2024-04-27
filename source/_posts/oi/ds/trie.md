---
date: 2023-06-14 20:52:27
update: 2024-01-21 01:10:00
title: 数据结构 - Trie 树
katex: true
tags:
- Algo
- C++
- DS
categories:
- OI
description: Trie 树是用来存储和查找字符串或数字等构成元素不多的数据结构，多用来匹配特定前缀。
---

## [模板] 字典树

> 给定 $n$ 个模式串 $s_1, s_2, \dots, s_n$ 和 $q$ 次询问，每次询问给定一个文本串 $t_i$，请回答 $s_1 \sim s_n$ 中有多少个字符串 $s_j$ 满足 $t_i$ 是 $s_j$ 的**前缀**。
>
> 一个字符串 $t$ 是 $s$ 的前缀当且仅当从 $s$ 的末尾删去若干个（可以为 0 个）连续的字符后与 $t$ 相同。
>
> 输入的字符串大小敏感。例如，字符串 `Fusu` 和字符串 `fusu` 不同。
>
> **输入格式**
>
> **本题单测试点内有多组测试数据**。  
>
> 输入的第一行是一个整数，表示数据组数 $T$。
>
> 对于每组数据，格式如下：
>
> 第一行是两个整数，分别表示模式串的个数 $n$ 和询问的个数 $q$。
>
> 接下来 $n$ 行，每行一个字符串，表示一个模式串。
>
> 接下来 $q$ 行，每行一个字符串，表示一次询问。
>
> **输出格式**
>
> 按照输入的顺序依次输出各测试数据的答案。对于每次询问，输出一行一个整数表示答案。
>
> **数据范围**
>
> 对于全部的测试点，保证 $1 \leq T, n, q\leq 10^5$，且输入字符串的总长度不超过 $3 \times 10^6$。输入的字符串只含大小写字母和数字，且不含空串。
>
> 题目链接：[P8306](https://www.luogu.com.cn/problem/P8306)。

字典树从根节点出发到某个点经过的所有边就是一个单词。统计某个单词作为前缀出现了几次，那我们就在加入单词的时候让经过路径上的每个点都加一，这样某个点表示的就是一个单词作为前缀出现的次数。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 3000010;

int mp[128], tr[N][62], cnt[N], idx;

void insert(const string& s) {
    int p = 0;
    for (char ch: s) {
        if (!tr[p][mp[ch]]) tr[p][mp[ch]] = ++idx;
        p = tr[p][mp[ch]];
        cnt[p]++;
    }
}

int get(const string& s) {
    int p = 0;
    for (char ch: s) {
        p = tr[p][mp[ch]];
        if (!p) return 0;
    }
    return cnt[p];
}

void solve() {
    int n, q;
    string s;

    for (int i = 0; i <= idx; i++) {
        cnt[i] = 0;
        for (int j = 0; j < 62; j++) {
            tr[i][j] = 0;
        }
    }
    idx = 0;

    cin >> n >> q;
    for (int i = 1; i <= n; i++) {
        cin >> s;
        insert(s);
    }
    for (int i = 1; i <= q; i++) {
        cin >> s;
        cout << get(s) << '\n';
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0), cout.tie(0);
    for (char ch = 'a'; ch <= 'z'; ch++) mp[ch] = ch - 'a';
    for (char ch = 'A'; ch <= 'Z'; ch++) mp[ch] = ch - 'A' + 26;
    for (char ch = '0'; ch <= '9'; ch++) mp[ch] = ch - '0' + 52;
    int T;
    cin >> T;
    while (T--) solve();

    return 0;
}
```

## [模板] 压位 Trie

> 实现一个数据结构，$n$ 次操作：
>
> 1. 插入 $x$。
> 2. 删除 $x$，若 $x$ 不存在则不进行。
> 3. 求 $x$ 的前驱，不存在返回 $0$。
> 4. 求 $x$ 的后继，不存在返回 $0$。
>
> 数据范围：$1\le n\le 3\times 10^7,0\le x\lt 2^{30}$，时限 $3s$。
>
> 题目链接：[U156719](https://www.luogu.com.cn/problem/U156719)。

### 存储方式

和 0/1 Trie 不同，这里要用 Trie 实现一个支持动态求前驱后继的数据结构，存储方式是从低位到高位存储有利于快速找，下文详细说明。目前流传的存储方式是：$A_i(x,y)$ 表示 $(x2^k+y)2^{ki}$ 是否存在，是一个布尔值，因此可以把 $y$ 用 $64$ 位无符号数压掉。

因此取 $k=6$ 即 $2^k=64$，可以利用位运算 $O(1)$ 求某些值。

### 函数

首先介绍两个内置函数：

+ `__builtin_clzll(x)` 用于统计二进制中前导零的数量，所以它是用来求最高位 $1$ 的函数。
+ `__builtin_ctzll(x)` 用于统计二进制中结尾零的数量，所以它是用来求最低位 $1$ 的函数。

这里我们要的是对数值，同时也是为了统一调用的函数类型，所以没用 `lowbit` 求最低位 $1$，这两个函数是用来求前驱后继的。

下面除法都代表整除，为了阅读方便就不写向下取整的符号了。

+ `ins(x)` 插入 $x$，原理是从最低的 $2^k$ 个二进制位开始，首先令 $A_0(x/2^k,x\bmod 2^k)=1$，然后 $x\leftarrow x/2^k$，重复以上步骤，直到 $A_i(x/2^k,x\bmod 2^k)$ 本身就为 $1$ 或者 $i$ 到达 $\log V$ 为止。插入可以不剪枝，每次都直接 $i=0\sim \log V$，但是删除必须剪，才能保证算法正确。
+ `del(x)` 删除 $x$，原理是从最低的 $2^k$ 个二进制位开始，首先令 $A_0(x/2^k,x\bmod 2^k)=0$，然后 $x\leftarrow x/2^k$，重复以上步骤，直到 $\exist j\ne x\bmod 2^k,A_i(x,j)=1$ 这时就不能继续向上删除了，否则就会让 $j$ 及与其有关的数字也被删掉。

+ `pre(x)` 查询 $x$ 前驱，根据贪心的思路，肯定是从低位开始找是否存在比 $x$ 更小的数字，因此可以只保留 $x\bmod 2^k$ 这个二进制位以后的所有二进制位。

  令 $s$ 为后面的这些二进制位：若 $s$ 为空继续递归查找，否则就把它的最高位拿出来，与 $x/2^k$ 进行拼接。

  之后回溯到低位，这里是用循环的方式，那么就是跳出循环时 $A_i$ 已经被处理完了，接下来就是处理 $A_{i-1}\sim A_0$，将已经拼出来的数字扔进 $A$ 中查找，最后找到的就是前驱。

  如果不存在前驱就是循环到尽头也没有找到一个更小的数字，判断一下如果循环是正常结束的直接返回无效值即可。

+ `suc(x)` 查询 $x$ 后继，基本和前驱一样。

### 复杂度

复杂度是 $O(n\log_{2^k}V)$，当 $k=6$ 时复杂度是 $O(n\log_{64}V)$，与 $O(n)$ 相差无几。

用 $G=\log_{64} V$ 调控 Trie 支持的数据范围，因为 $A_0$ 的大小是 $2^{6G-6}$，所以 Trie 支持的最大数字是 $2^{6G}-1$，占用的空间略大于 $2^{6G-6}$。

当 $G=4$ 时，数据范围 $10^7$，空间为 $2MB$；当 $G=5$ 时，数据范围 $10^9$，空间为 $130MB$。

```cpp
typedef unsigned long long ull;
#define lg2highbit(x) (63 - __builtin_clzll(x))
#define lg2lowbit(x) (__builtin_ctzll(x))
const int G = 5, INVALID = 0;

struct Trie {
    ull* A[G];

    Trie() {
        int sz = 1;
        // 注意这行后面加个括号，表示进行初始化，如果不加我也没碰到问题，但是加上比较保险。
        for (int i = G-1; i >= 0; i--, sz <<= 6) A[i] = new ull[sz]();
    }

    void ins(int x) {
        for (int i = 0; i < G; i++, x >>= 6) {
            ull mask = 1ull << (x & 63);
            if (A[i][x >> 6] & mask) return;
            A[i][x >> 6] |= mask;
        }
    }

    void del(int x) {
        for (int i = 0; i < G; i++, x >>= 6) {
            if (A[i][x >> 6] &= ~(1ull << (x & 63))) return;
        }
    }

    int pre(int x) {
        int i, res;
        for (i = 0; i < G; i++, x >>= 6) {
            ull s = A[i][x >> 6] & ((1ull << (x & 63)) - 1);
            if (!s) continue;
            res = x & ~63 | lg2highbit(s);
            break;
        }
        if (i >= G) return INVALID;
        for (; i; i--) res = res << 6 | lg2highbit(A[i-1][res]);
        return res;
    }

    int suc(int x) {
        int i, res;
        for (i = 0; i < G; i++, x >>= 6) {
            ull s = A[i][x >> 6] & (~((1ull << (x & 63)) - 1) << 1);
            if (!s) continue;
            res = x & ~63 | lg2lowbit(s);
            break;
        }
        if (i >= G) return INVALID;
        for (; i; i--) res = res << 6 | lg2lowbit(A[i-1][res]);
        return res;
    }

    ~Trie() {
        for (int i = 0; i < G; i++) delete[] A[i];
    }
};
```

## [HNOI2002] 营业额统计

> 给定 $n\lt 2^{15}$ 元素的数列 $a$，对于每个 $|a_i|\le 10^6$ 它的权值是：
>
> 1. 若 $i=1$，则权值 $w_i=a_i$。
> 2. 若 $i\ne 1$，则权值 $w_i=\min_{j=1}^{i-1} |a_j-a_i|$。
>
> 求出 $\sum_{i=1}^n w_i$，保证答案小于 $2^{31}$。
>
> 题目链接：[P2234](https://www.luogu.com.cn/problem/P2234)。

这就是求前驱后继，对于负数加上一个偏移量使其变成非负数，由于求的是相对大小所以答案不变，一开始我的快读没支持负数所以挂了好几次。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int OFFSET = 1e6, G = 4, INF = 0x3f3f3f3f;
typedef unsigned long long ull;
#define lghb(x) (63 - __builtin_clzll(x))
#define lglb(x) __builtin_ctzll(x)

inline int read() {
    int t = 0;
    int neg = 0;
    char ch = getchar();
    while (!isdigit(ch)) neg |= ch == '-', ch = getchar();
    while (isdigit(ch)) t = t*10 + (ch^48), ch = getchar();
    return neg ? -t : t;
}

struct Trie {
    ull* A[G];

    Trie() {
        int sz = 1;
        for (int i = G-1; i >= 0; i--, sz <<= 6) A[i] = new ull[sz]();
    }

    void ins(int x) {
        for (int i = 0; i < G; i++, x >>= 6) {
            ull mask = 1ull << (x & 63);
            if (A[i][x >> 6] & mask) return;
            A[i][x >> 6] |= mask;
        }
    }

    bool has(int x) {
        return A[0][x >> 6] & (1ull << (x & 63));
    }

    int pre(int x) {
        int i, res;
        for (i = 0; i < G; i++, x >>= 6) {
            ull s = A[i][x >> 6] & ((1ull << (x & 63)) - 1);
            if (!s) continue;
            res = x & ~63 | lghb(s);
            break;
        }
        if (i >= G) return -INF;
        for (; i; i--) res = res << 6 | lghb(A[i-1][res]);
        return res;
    }

    int suc(int x) {
        int i, res;
        for (i = 0; i < G; i++, x >>= 6) {
            ull s = A[i][x >> 6] & ~((1ull << (x & 63)) - 1) << 1;
            if (!s) continue;
            res = x & ~63 | lglb(s);
            break;
        }
        if (i >= G) return INF;
        for (; i; i--) res = res << 6 | lglb(A[i-1][res]);
        return res;
    }

    ~Trie() {
        for (int i = 0; i < G; i++) delete[] A[i];
    }
} trie;

int main() {
    int n = read(), res = 0;
    for (int i = 1; i <= n; i++) {
        int a = read() + OFFSET;
        if (i == 1) res += a - OFFSET;
        else if (!trie.has(a)) res += min(a - trie.pre(a), trie.suc(a) - a);
        trie.ins(a);
    }
    return !printf("%d\n", res);
}
```

## [CF1895D] XOR Construction

> 给 $n-1$ 个数 $a_1,\cdots,a_{n-1}$，构造 $b_1,\cdots,b_n$ 满足：
>
> + $b_1,\cdots,b_n$ 是 $0\sim n-1$ 的一个排列。
> + 对于 $1\le i\lt n,b_i\oplus b_{i+1}=a_i$。
>
> 数据保证有解，$2\le n\le 2\times 10^5,0\le a_i\le 2n$。
>
> 题目链接：[CF1895D](https://codeforces.com/contest/1895/problem/D)。

首先我们推一下柿子。
$$
b_1\oplus b_2=a_1\\
b_2\oplus b_3=a_2\\
b_3\oplus b_4=a_3\\
$$
我们知道异或是可以移项的，因为满足自反性，即 $x\oplus x=0$，那么：
$$
b_2=a_1\oplus b_1\\
b_3=a_2\oplus b_2\\
b_4=a_3\oplus b_3
$$
继续展开我们可以推出最终的式子：
$$
b_i=(\bigoplus _{j=1}^{i-1} a_j)\oplus b_1
$$
所以我们枚举 $b_1$ 的值，看一下 $\max b_i$ 是否小于 $n$，判断方式就是将异或前缀和全部加入一个 01 Trie 中，然后传入 $b_1$ 求一个最大值。因为 $b_i=S_{i-1}\oplus b_1$，而题目保证有解，一定有 $S_i$ 互不相同，因此只要最大值小于 $n$，那么就是合法解。

然后看一下数据范围，$a_i$ 最大是 $4\times 10^5$，有 $19$ 个二进制位，那么有 $2^{19}=524288$ 条边。

求最大值的过程就是一个贪心的过程，对于一个更高的二进制位，显然是能取 $1$ 就取 $1$，因为有等比数列求和公式：
$$
\sum_{i=0}^n 2^i=2^{n+1}-1<2^{n+1}
$$
就算你后面全都取 $1$，也不如当前位取 $1$ 更大。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 210000, M = 530000;
int tr[M][2], s[N], n, idx;

void insert(int a) {
    int p = 0;
    for (int i = 18; i >= 0; i--) {
        int b = a >> i & 1;
        if (!tr[p][b]) tr[p][b] = ++idx;
        p = tr[p][b];
    }
}

int get(int k) {
    int p = 0, res = 0;
    for (int i = 18; i >= 0; i--) {
        int b = k >> i & 1;
        if (tr[p][b^1]) res |= 1 << i, p = tr[p][b^1];
        else p = tr[p][b];
    }
    return res;
}

int main() {
    scanf("%d", &n);

    for (int i = 1; i < n; i++) {
        scanf("%d", &s[i]);
        s[i] ^= s[i-1];
        insert(s[i]);
    }

    for (int b1 = 0; b1 < n; b1++) {
        if (get(b1) < n) {
            printf("%d ", b1);
            for (int i = 1; i < n; i++) printf("%d ", b1 ^ s[i]);
            break;
        }
    }

    return 0;
}
```

## [CF665E] Beautiful Subarrays

> 给定 $n$ 个元素的数组 $a$ 以及 $k$，问多少个 $[l,r]$ 满足 $\bigoplus_{i=l}^r a_i\ge k$，$1\le n\le10^6$，$1\le k,a_i\le 10^9$。
>
> 题目链接：[CF665E](https://codeforces.com/problemset/problem/665/E)。

首先通过前缀和转化为存在多少个 $[l,r]$ 满足 $S_{l-1}\oplus S_r\ge k$，也就是给定一个数字，问一堆数字异或它后大于某个数的有多少。

将 $S_{0}\sim S_{r-1}$ 都插入 0/1 Trie 中，然后从高到低逐位查找，如果当前位 $i$ 可以比 $k_i$ 大（其中 $k_i$ 指 $k$ 的第 $i$ 二进制位）那么就把更大的所有数都统计答案，否则就递归处理，类似于线段树二分。

众所周知二元组的数量级是 $O(n^2)$ 的，所以需要 `long long`。关于写法方面，我觉得循环写法容易把变量整乱，递归写法更不容易出错，上面那个题是我很久之前写的（写现在这段文字的时候是 2024/01/21 ）所以风格不一样。

```cpp
#include <bits/stdc++.h>
using namespace std;

namespace IO {}
using IO::read;
using IO::write;

const int N = 1000010, M = 30000010;
int n, k, s[N], tr[M][2], sz[M], idx = 1;

void ins(int a, int p, int i = 30) {
    sz[p]++;
    if (i == -1) return;
    int b = a >> i & 1;
    if (!tr[p][b]) tr[p][b] = ++idx;
    ins(a, tr[p][b], i-1);
}

int query(int a, int p, int i = 30) {
    if (i == -1) return sz[p];
    int b = a >> i & 1, t = k >> i & 1, res = 0;
    if (t) return query(a, tr[p][b^1], i-1);
    return sz[tr[p][b^1]] + query(a, tr[p][b], i-1);
}

int main() {
    read(n, k);
    ins(0, 1);

    long long res = 0;
    for (int i = 1; i <= n; i++) {
        read(s[i]), s[i] ^= s[i-1];
        res += query(s[i], 1);
        ins(s[i], 1);
    }
    return write(res), 0;
}
```

