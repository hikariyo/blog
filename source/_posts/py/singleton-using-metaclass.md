---
title: 以元类实现单例
date: 2023-01-30 09:18:34.25
updated: 2023-01-30 09:18:34.25
description: 借助元类可以控制类在实例化时的行为，进而达到单例的目的。在 Python 中类本身就是 type 的对象，而元类就是继承了 type 的类，仅此而已。
categories: 
- Learn
tags: 
- Python
---

## 实现

极简化解释：在 Python 中类本身就是 type 的对象，而元类就是继承了 type 的类，仅此而已。

其实就是在初始化前检查一下当前类中是否存在一个实例，存在就返回。

```python
class SingletonMeta(type):
    def __init__(cls, *args, **kwargs):
        super().__init__(*args, **kwargs)
        cls._singleton_instance = None

    def __call__(cls, *args, **kwargs):
        if cls._singleton_instance is None:
            cls._singleton_instance = super().__call__(*args, **kwargs)
        return cls._singleton_instance
```

如果你想以继承的方式使这个类成为单例，可以再添加一个工具类。

```python
class Singleton(metaclass=SingletonMeta):
    __slots__ = ()
```

下面那句只是为了节省一点内存，可以用 `sys.getsizeof` 来验证这一点，这里不再赘述。如果你愿意的话，直接 `pass` 也可以。
