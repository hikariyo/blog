---
title: 学习 - 基于 inspect 实现重载
date: 2023-02-04 19:36:32.019
updated: 2023-02-05 21:41:01.335
description: 通常情况下 Python 只能做到在函数体内部判断参数类型进行重载，但我们可以借助 inspect 来实现像一般静态语言那样的重载。为了简化代码，不考虑 positional only 和 keyword only 的参数。
categories: 
- Learn
tags: 
- Python
---

通常来说，Python 中的重载依赖 `typing.overload`，如下：

```python
from typing import overload, Any


@overload
def foo(a: int) -> None: ...


@overload
def foo(a: float) -> None: ...


def foo(a: Any) -> None:
    if isinstance(a, int):
        return

    if isinstance(a, float):
        return

    raise TypeError('...')
```

本文要实现的是借助 `inspect` 实现一个运行时帮助判断类型并调用指定函数的工具。

## 实现

为了简化逻辑，我们这里只支持普通的参数，也就是不支持 `positional only` 和 `keyword only` 参数。

```python
import inspect
from inspect import Parameter
from typing import Any, TypeVar, Callable, ParamSpec, Iterator


T = TypeVar('T')
P = ParamSpec('P')


def get_params(f: Callable[P, T]) -> Iterator[Parameter]:
    yield from inspect.signature(f).parameters.values()


def match(f: Callable[P, T], args: tuple[Any, ...]) -> bool:
    params = list(get_params(f))

    if len(params) != len(args):
        return False

    return all(
        isinstance(args[index], param.annotation)
        for index, param in enumerate(params)
    )


class Overload:
    def __init__(self) -> None:
        self.callables: set[Callable[..., T]] = set()

    def __call__(self, *args: Any) -> Any:
        for f in self.callables:
            if match(f, args):
                return f(*args)
        raise TypeError(f'unmatched arguments {args}')

    def register(self, f: Callable[P, T]) -> 'Overload':
        for p in get_params(f):
            if p.kind != Parameter.POSITIONAL_OR_KEYWORD:
                raise TypeError('only allow normal arguments')
            if not isinstance(p.annotation, type):
                raise TypeError('annotations must be real types')
        self.callables.add(f)  # type: ignore
        return self
```

使用：

```python
FIRST = 1
SECOND = 2
THIRD = 3


foo = Overload()


@foo.register
def foo(a: int) -> int:
    print(f'calling foo with an integer: {a}')
    return FIRST


@foo.register
def foo(a: float) -> int:
    print(f'calling foo with a float: {a}')
    return SECOND


@foo.register
def foo(a: int, b: int) -> int:
    print(f'calling foo with two integers: {a}, {b}')
    return THIRD


assert foo(1) == FIRST
assert foo(3.14) == SECOND
assert foo(1, 2) == THIRD
```

就这样。