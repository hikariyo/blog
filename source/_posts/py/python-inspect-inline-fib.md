---
title: Python + inspect 一行实现递归 fib 函数
date: 2022-10-04 04:33:37.813
updated: 2022-10-04 04:40:13.194

categories: 
- Dev
tags: 
- Python
---

## 背景

有个裙友要看看用 `lambda` 能不能在一行里定义出来 `fib` 函数，并且不要那个根号五的数学公式，于是就有了这篇文章。

## 介绍

`inspect` 库可以帮助我们拿到 Python 上下文的各种信息，自然也包括了当前正在运行的函数。配合 `eval` 可以达到我们的目的。

## 实现

### 原始方法

虽然大家都知道，但还是放上来做一个对比。

```py
def fib(n):
    if n in (1, 2):
        return 1
    return fib(n-1) + fib(n-2)

## 8
print(fib(6))
```

### 通过 inspect 获取当前函数

我目前只知道 `codeobject` 配合 `eval` 来执行的方法，如下：

```py
import inspect
def fib():
    if n in (1, 2):
        return 1
    return eval(inspect.currentframe().f_code, {'n': n-1, 'inspect': inspect}) + \
        eval(inspect.currentframe().f_code, {'n': n-2, 'inspect': inspect})

## 8
print(eval(fib.__code__, {'n': 6, 'inspect': inspect}))
```

注意，此时我们就通过给 `eval` 的第二个参数指定函数运行时的环境（即globals），这样我们就可以在函数内部直接通过变量名访问了。

### 用 lambda 写成一行

还用了 `__import__` 来导包，是真正的一行捏：

```py
result = (lambda number:
    eval(
        (lambda: 1 if n in (1, 2) else
            eval(f().f_code, {'n': n-1, 'f': f}) +
            eval(f().f_code, {'n': n-2, 'f': f})
        ).__code__, 
        {'n': number, 'f': __import__('inspect').currentframe}
    )
)(6)

## 8
print(result)
```

注意不要改我们 `lambda number` 后边的参数名，如果也改成 `n` 的话，`eval` 会报错，下面是报错信息：

```
Traceback (most recent call last):
  File "test.py", line 1, in <module>
    result = (
  File "test.py", line 3, in <lambda>
    eval((lambda: 1 if n in (1, 2)
TypeError: code object passed to eval() may not contain free variables
```

因为此时内层函数用的 `n` 就类似 Lua 的 `upvalue`，或者 `nonlocal`，总之就是闭包里面的那个概念，它无法访问到就报错了。

## 总结

不觉得这很酷吗？很符合我对 Python 的想象，科技并带着趣味。😎
