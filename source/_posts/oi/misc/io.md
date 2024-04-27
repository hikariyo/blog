---
date: 2023-12-12 23:44:00
update: 2024-01-13 01:04:00
title: 杂项 - IO 优化
katex: true
tags:
- Algo
- C++
categories:
- OI
---

## 简介

IO 优化通常对于程序的常数提升并不是很大，但是在数据量比较大，大概 $10^6$ 级别的时候就要考虑加上 IO 优化了，如果数据量不大反而可能导致负优化。

基本原理是 C++ 标准库对于格式处理的比较多，而我们想要的一般来说只是输入和输出整数，所以自己实现起来常数会略有提升。

## READ / WRITE

输入的原理是从输入流中读取字符，过滤出非整数字符并记录是否含有负号，然后处理一段连续的整数，最后将符号加上；输出的原理是将给定数字的所有数位压到栈中，然后逆序输出：

```cpp
template<typename T>
inline void read(T& t) {
    t = 0;
    bool neg = 0;
    char ch = getchar();
    while (!isdigit(ch)) neg |= ch == '-', ch = getchar();
    while (isdigit(ch)) t = t * 10 + ch - '0', ch = getchar();
    if (neg) t = -t;
}

template<typename T>
inline void write(T t, char sep = '\n') {
    static char stk[64];
    int tp = 0;
    if (t < 0) putchar('-'), t = -t;
    do stk[++tp] = t % 10 + '0'; while (t /= 10);
    while (tp) putchar(stk[tp--]);
    putchar(sep);
}
```

## FREAD / FWRITE

可以用 `fread/fwrite` 进一步优化读写字符的效率，但是很多选手对 `fread/fwrite` 大力压行劝退很多想学习使用的萌新（包括我），这里先介绍一下这两个函数的原型：

```cpp
size_t fread(void* buffer, size_t size, size_t count, FILE* stream);
size_t fwrite(const void* buffer, size_t size, size_t count, FILE* stream);
```

它们都返回成功处理的 `count`，对于读入单个字符，这里用 `p, q` 两个指针分别记录当前缓冲区没有处理的第一个字符和缓冲区的结尾，这里用到了 `fread` 的返回值，其中 `S` 是一般为 $2^{16}$ 或 $2^{20}$ 的常量。

对于输出单个字符，先把它输出到缓冲区中，当缓冲区满的时候就调用 `fwrite` 写掉，并且重置记录当前位置的指针。

为了让程序结束的时候自动调用 `flush`，可以用一个空的全局对象，然后在析构函数中调用 `flush`，这样程序结束时就会自动调用这个全局对象的析构函数，就把缓冲区剩余的内容输出了。

```cpp
const int S = 1 << 20;
char R[S], *R1, *R2;
char W[S], *W1 = W, *W2 = W + S;

void flush() {
    W1 -= fwrite(W, 1, W1 - W, stdout);
}

char gc() {
    if (R1 == R2) R2 = (R1 = R) + fread(R, 1, S, stdin);
    return R1 == R2 ? EOF : *R1++;
}

void pc(char ch) {
    *W1++ = ch;
    if (W1 == W2) flush();
}

struct F { ~F() { flush(); } } _F;
```

如果使用了自定义的字节输入输出，那么在 Shell 运行程序的时候要手动输入 Ctrl-D 来表示输入结束，在 Windows 下应该是 Ctrl-Z 表示输入结束。

## 模板

加上 `template` 能够支持 `int, long long, __int128`，通过可变参数也可以直接 `read(a, b, c)`。

```cpp
namespace IO {
    const int S = 1 << 20;
    char R[S], *R1, *R2;
    char W[S], *W1 = W, *W2 = W + S;

    inline void flush() {
        W1 -= fwrite(W, 1, W1 - W, stdout);
    }

    inline char gc() {
        if (R1 == R2) R2 = (R1 = R) + fread(R, 1, S, stdin);
        return R1 == R2 ? EOF : *R1++;
    }

    inline void pc(char ch) {
        *W1++ = ch;
        if (W1 == W2) flush();
    }

    struct F { ~F() { flush(); } } _F;

    inline void read() {}

    template<typename T>
    inline void read(T& t) {
        t = 0;
        bool neg = 0;
        char ch = gc();
        while (!isdigit(ch)) neg |= ch == '-', ch = gc();
        while (isdigit(ch)) t = t * 10 + ch - '0', ch = gc();
        if (neg) t = -t;
    }

    template<typename T, typename... Args>
    inline void read(T& t, Args&... args) {
        read(t);
        read(args...);
    }

    template<typename T>
    inline void write(T t, char sep = '\n') {
        static char stk[64];
        int tp = 0;
        if (t < 0) pc('-'), t = -t;
        do stk[++tp] = t % 10 + '0'; while (t /= 10);
        while (tp) pc(stk[tp--]);
        pc(sep);
    }
}
using IO::read;
using IO::write;
```

下面是针对字符读入与字符串的输出：

```cpp
template<>
inline void read(char& ch) {
    while (isspace(ch = gc()));
}

template<>
inline void write(const char* s, char sep) {
    while (*s) pc(*s++);
    pc(sep);
}
```

如果涉及到读入到文件末尾，可以改变一下 `read` 的返回值：

```cpp
inline bool read() { return true; }

template<typename T>
inline bool read(T& t) {
    t = 0;
    bool neg = 0;
    char ch = gc();
    while (!isdigit(ch) && ch != EOF) neg |= ch == '-', ch = gc();
    if (ch == EOF) return false;
    while (isdigit(ch)) t = t * 10 + ch - '0', ch = gc();
    if (neg) t = -t;
    return true;
}

template<typename T, typename... Args>
inline bool read(T& t, Args&... args) {
    return read(t) && read(args...);
}
```

可以通过 [Snippet Generator](https://snippet-generator.app/) 给 VSCode 添加 Snippet，这里就不多说了。
