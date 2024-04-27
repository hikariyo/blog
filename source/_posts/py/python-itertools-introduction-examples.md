---
title: Python itertools 简单介绍和运用例
date: 2023-01-21 17:46:23.261
updated: 2023-02-13 12:52:42.514
categories: 
- Learn
tags: 
- Python
---

最近写 Python 比较多，不可避免地要处理一堆可迭代对象，发现 Python 对于**迭代器/生成器**的支持相较于其它语言来说是更为丰富的，所以简单记录一下 `itertools` 这个内置包中几个常见的函数。

文末附一个实例，是一个关于扫雷的算法，用到了文中提到的一些函数。注意，我知道它们并**不是真正的函数**，而是以类的形式定义。为了方便起见，本文就把它们当成函数看代。

## 介绍

使用**迭代器/生成器**的好处是节省无意义的内存消耗，就像 Java 里的 `Stream API` 一样（当然也有很多不同，这里不再展开讨论）。假设我们最终要对一个 `iterable` 进行**求和**或者其它 `reduce` 操作，那是没有必要为此创建一堆数组的。

反例如 JS 中 `Array` 上的 `map`，`filter` 等操作，它们都会创建一个新的数组，但是这点的性能其实不值一提，只是**锦上添花**而已。就像上文说的，只是节省**无意义**的内存消耗。

扯多了，回到我们的 `itertools` 上面来。

| 名称                                                         | 说明                                                         | 示例                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| `count(start=0, step=1)`                                     | 参数可以为 `int` 或 `float`，类似于 `range` 但它是无限的。   | `count(10, 2) --> 10 12 14 ...`                  |
| `cycle(p)`                                                   | 重复遍历一个可迭代对象。                                     | `cycle('ABC') --> A B C A ... `                  |
| `repeat(elem, times=None)`                                   | 重复同一个元素，如果不指定次数 `times` 就是无限重复。        | `repeat(10, 3) --> 10 10 10`                     |
| `chain(p, q, ...)`                                           | 把所有的参数连接成一个可迭代对象。                           | `chain('ABC', 'DEF') -> A B C D E F`             |
| `chain.from_iterable(p)`                                     | 相当于把参数解包传给 `chain` 函数。                          | `chain(['ABC', 'DEF']) --> A B C D E F`          |
| `comparess(data, selectors)`                                 | 以 `selectors` 是否为 `truthy value`（这是 JS 的说法，就是它们是否被当作 `true`）来筛选数据。 | `compress('ABC', [0, 1, 1]) --> B C`             |
| `product(p, q, ..., repeat=1)`                               | 相当于嵌套 `for` 循环。                                      | `product('AB', 'CD') -->  AC, AD, BC, BD`        |
| `pairwise(iterable)`                                         | 成对迭代一个可迭代对象。                                     | `pairwise('ABCDE') --> AB BC CD DE`              |
| `takewhile(predicate, iterable)`                             | 直到 `predicate(i)` 返回一个 `falsy value`。                 | `takewhile(lambda x: x<5, [1, 4, 6, 1]) -> 1 4`  |
| `dropwhile(predicate, iterable)`                             | 当 `predicate(i)` 返回一个 `falsy value` 时开始。            | `dropwhile(lambda x: x<5, [1, 4, 6, 1]) --> 6 1` |
| `islice(iterable, stop), islice(iterable, start, stop, step=1)` | 针对一个可迭代对象的切片操作。                               | 见下文                                           |
| `tee(iterable, n=2)`                                         | 将一个可迭代对象分裂成多个，不是线程安全的。                 | 见下文                                           |

实际上还有很多，可以参考 [Python 官方文档](https://docs.python.org/3.10/library/itertools.html)，这里只是挑了几个我比较喜欢用的。

## 示例

下面详细说一下其中几个，举几个例子。

+ `repeat` 可以搭配 `map` 一起使用，看一下代码就明白怎么用了。

  ```python
  >>> list(map(lambda n: pow(n, 3), range(5)))
  [0, 1, 8, 27, 64]
  >>> list(map(pow, range(5), repeat(3)))
  [0, 1, 8, 27, 64]
  ```

  因为 `map` 在任何一个可迭代对象结束之后就会结束，所以我们直接让它无限重复 `3` 作为 `pow` 的第二个参数，就达到了我们求立方的目的。

+ `cycle` 与 `islice` 一起使用，重复从一个可迭代对象中取出特定个元素。

  ```python
  >>> ''.join(islice(cycle('ABCD'), 7))
  'ABCDABC'
  ```

+ 本来我还想说 `count` 和 `islice` 一起用，但我又想到这不就是 `range` 么，不说了喵。

+ `tee` 的实现原理挺有意思的，我这里把官方文档里的等价代码粘贴过来。

  ```python
  def tee(iterable, n=2):
      it = iter(iterable)
      deques = [collections.deque() for i in range(n)]
      def gen(mydeque):
          while True:
              if not mydeque:             # when the local deque is empty
                  try:
                      newval = next(it)   # fetch a new value and
                  except StopIteration:
                      return
                  for d in deques:        # load it to all the deques
                      d.append(newval)
              yield mydeque.popleft()
      return tuple(gen(d) for d in deques)
  ```

  它使用了先进先出的队列来保存迭代器返回的数据，实现还是比较巧妙的，下面贴上使用方法。

  ```python
  >>> a, b = tee(range(10))
  >>> for i in a:
  ...     print(i)
  ...
  >>> for j in b:
  ...     print(j)
  ```
  
  但其实当数据比较大的时候，它还是会消耗很多内存，所以慎重使用。


+ `product` 可以展平嵌套 `for` 循环，以下两种写法结果相同。我这里就不把输出粘贴过来了，有点长。

  ```python
  >>> for x in range(2):
  ...    for y in range(3):
  ...        print(x, y)
  ...
  >>> for x, y in product(range(2), range(3)):
  ...    print(x, y)
  ```


+ `dropwhile` 可以忽略前几个不符条件的元素，`takewhile` 可以只保留前几个符合条件的元素。用处还是挺大的。

+ 下面是计算扫雷游戏中每个格子周围有几个雷的算法。

  ```python
  for x, y in product(range(width), range(height)):
      self[x, y].count = sum(
          self[x + dx, y + dy].is_mine
          for dx, dy in product((-1, 0, 1), repeat=2)
      )
  ```

  我只把和本节相关的代码贴了出来，下面是一些解释。

  + 这是一个实例方法，它所在的类实现了`__getitem__`，当坐标无效时会返回一个所有的属性都是 `0` 或 `False` 的对象，避免进行判空。
  + 格子对象的 `is_mine` 是一个布尔值，它可以被当成 `int` 参与运算。

  + 这段代码也会把当前格子是否是雷给计算上，它对游戏是无关紧要的，因为当这个格子是雷的时候，计数也不会被显示出来。

就这样，白白喵。