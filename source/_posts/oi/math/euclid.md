---
date: 2023-07-29 21:39:00
update: 2023-08-31 00:16:00
title: 数学 - 欧几里得算法
katex: true
tags:
- Algo
- C++
- Math
categories:
- OI
---

## 欧几里得算法

辗转相除法，设 $a\ge b$ 原理是：
$$
\gcd(a,b)=\gcd(b,a-b)
$$
这个原因是 $\gcd(a,b)\mid a,\gcd(a,b)\mid b$，所以相减同样满足 $\gcd(a,b)\mid (a-b)$。

我们知道取模实际上是减法，所以同理可得：
$$
\gcd(a,b)=\gcd(b,a\bmod b)
$$
实现起来就是：

```cpp
int gcd(int a, int b) {
    if (!b) return a;
    return gcd(b, a % b);
}
```

## 拓展欧几里得算法

裴蜀定理：对于任意正整数 $a,b$ 一定存在非零整数 $x,y$ 使 $ax+by=\gcd(a,b)$

如果已知：
$$
by+(a\bmod b)x=d\\
$$
则能递推出：
$$
\begin{aligned}
by+(a-\lfloor\frac{a}{b}\rfloor b)x&=d\\
ax+b(y-\lfloor\frac{a}{b}\rfloor x)&=d\\
\end{aligned}
$$
然后就写出 `exgcd` 算法求一组解了。

```cpp
int exgcd(int a, int b, int& x, int& y) {
    if (b == 0) return x = 1, y = 0, a;
    int d = exgcd(b, a%b, y, x);
    y -= a/b * x;
    return d;
}
```

由于 $ax_0+by_0=d$，那么满足条件的所有 $x,y$ 一定也满足：
$$
a(x+\frac{b}{d})+b(y-\frac{a}{d})=d
$$
也就有通解如下：
$$
\left \{ \begin{aligned}
x&=x_0+k\frac{b}{d}\\
y&=y_0-k\frac{a}{d}\\
k&\in \Z
\end{aligned}\right .
$$
除以 $d$ 后这个数仍为整数。

