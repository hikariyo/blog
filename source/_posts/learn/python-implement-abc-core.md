---
title: 自己实现 abc 模块的核心功能
date: 2023-01-23 17:12:45.127
updated: 2023-01-24 11:14:27.671

categories: 
- Learn
tags: 
- Dev
---

## 简介

通过 `abc` 这个模块，我们可以在 Python 中使用抽象类，定义抽象方法、抽象属性。其本质是利用**元类**来检查是否有未实现的抽象方法，从而阻止抽象类的实例化，也就达到了目的。

所以说它只是一个辅助的**检查手段**，就像 Java 里的 `@Overrides` 类似（严格来讲 Java 的这个是编译期检查），你写不写都是可以达到你的目的，但是用了之后能防止你因为忘记实现或者写错方法名而导致的错误。

```python
class A:
    def foo(self):
        pass


class B(A):
    def foo(self):
        print('foo')
```

这么写当然是可以的，但是这就需要我们**人工检查**在子类中是否存在未实现的抽象方法，否则运行时就会出 bug，而且当程序结构足够复杂的时候**排查比较困难**。所以最好还是要使用标准库 `abc` 为我们提供的这个功能。不仅是运行时有检查，IDE 也会帮你检查是否已经实现了所有抽象方法。

```python
from abc import abstractmethod, ABC


class A(ABC):
    @abstractmethod
    def foo(self):
        pass


class B(A):
    def foo(self):
        print('foo')
```

扯多了，回到我们的正题上来，即如何实现这样一个元类。

## 标记抽象方法

如果一个方法是抽象方法，那么它的 `__isabstractmethod__` 属性会被标记为 `True`。因为是我们自己实现，所以用其他的名字也未尝不可，但是会失去 Python 内部的支持，比如 `property` 对象。

通过 CPython 3.10.9 源码 `Objects/descrobject.c:1773`，如下：

```c
static PyObject *
property_get___isabstractmethod__(propertyobject *prop, void *closure)
{
    int res = _PyObject_IsAbstract(prop->prop_get);
    if (res == -1) {
        return NULL;
    }
    else if (res) {
        Py_RETURN_TRUE;
    }

    res = _PyObject_IsAbstract(prop->prop_set);
    if (res == -1) {
        return NULL;
    }
    else if (res) {
        Py_RETURN_TRUE;
    }

    res = _PyObject_IsAbstract(prop->prop_del);
    if (res == -1) {
        return NULL;
    }
    else if (res) {
        Py_RETURN_TRUE;
    }
    Py_RETURN_FALSE;
}
```

这是 `property` 对象的 `__isabstractmethod__` 属性，可以看到当 `fget`，`fset`，`fdel` 任意一个的 `__isabstractmethod__` 为 `True` 时，这个 `property` 的 `__isabstractmethod__` 也是 `True`。所以为了得到内置库的支持，还是用 `__isabstractmethod__` 而不是别的名字。


为了简便，本文涉及代码不再写类型。~~才不是因为我懒！~~

所以代码如下：

```python
def abstractmethod(func):
    func.__isabstractmethod__ = True
    return func


def isabstractmethod(func):
    return getattr(func, '__isabstractmethod__', False)
```

如果你就是不想用 `__isabstractmethod__`，想用别的，比如 `__taffy__`，你可以这么做：

```python
class taffyproperty(property):
    __taffy__ = True


def taffymethod(func):
    func.__taffy__ = True
    return func


def istaffymethod(func):
    return getattr(func, '__taffy__', False)
```

这样也能达到目的，但本文使用 `__isabstractmethod__` 来获取内置库的支持。

## 抽象元类

接下来实现本文核心 `ABCMeta`：

```python
class ABCMeta(type):
    def __new__(mcs, type_name, bases, attrs):
        attrs['__abstractmethods__'] = frozenset(
            get_abstract_methods(bases, attrs)
        )
        return super().__new__(mcs, type_name, bases, attrs)

    def __call__(cls, *args, **kwargs):
        # noinspection PyUnresolvedReferences
        if abstracts := ', '.join(cls.__abstractmethods__):
            raise TypeError(f'Can\'t instantiate abstract class {cls.__name__} '
                            f'with abstract methods {abstracts}')
        return super().__call__(*args, **kwargs)


def get_abstract_methods(bases, attrs):
    for base in bases:
        for name in getattr(base, '__abstractmethods__', []):
            # 如果当前类重写了这个方法
            if name in attrs and not isabstractmethod(attrs[name]):
                continue
            yield name

    for name, attr in attrs.items():
        if isabstractmethod(attr):
            yield name


class ABC(metaclass=ABCMeta):
    pass
```

当类实例化的时候，再检查是否已经实现了抽象方法。如果有抽象方法没实现，就抛出一个 `TypeError`。

这里 `ABC` 只是方便定义抽象类，直接继承于 `ABC` 即可，不用写 `metaclass=ABCMeta`。

## 使用

如下：

```python
class MyABC(ABC):
    @abstractmethod
    def foo(self):
        raise NotImplementedError()

    @abstractmethod
    def bar(self):
        raise NotImplementedError()


class MyABC2(ABC):
    @property
    @abstractmethod
    def prop(self):
        raise NotImplementedError()


class MySubClass(MyABC, MyABC2):
    def foo(self):
        print('foo')

    def bar(self):
        print('bar')

    @property
    def prop(self):
        return 'prop'


if __name__ == '__main__':
    obj = MySubClass()
    obj.foo()
    obj.bar()
    print(obj.prop)
```

可以删除任意一个 `MySubClass` 实现的方法或属性，当实例化的时候就会立即报错，而不是等到调用的时候再抛出我们自己指定的 `NotImplementedError`。
