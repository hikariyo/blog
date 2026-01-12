---
title: 学习 - Python 实现动态的工厂模式
date: 2023-01-22 15:23:27.246
updated: 2023-03-17 23:51:48.769

categories: 
- Learn
tags: 
- Python
---

本文讨论的主要是，如何把工厂模式生产的产品，即抽象类的子类信息**动态地保存到一个表中**，而不是直接简单粗暴地使用 `if-else` 来判断。可能是 Java 给我的影响比较大，我还比较喜欢用抽象类抽象方法这些东西，至于好坏应该由实际情况来决定，这里就不详细展开讨论了。

那么本文所使用的示例来自于我一个分析 QQ 聊天记录文件的项目，因为它对于群聊的聊天记录和私聊的聊天记录**格式不同**，需要分别处理，所以就到了本文所讨论的工厂模式。

## Hook

通过 `__init_subclass__` 来实现注册的功能。

```python
from typing import Type
from abc import ABC, abstractmethod


class Shape(ABC):
    _shapes: dict[str, Type['Shape']] = {}

    def __init_subclass__(cls, variety: str) -> None:
        super().__init_subclass__()
        cls._shapes[variety] = cls

    @classmethod
    def make(cls, variety: str) -> 'Shape':
        return cls._shapes[variety]()

    @abstractmethod
    def draw(self):
        raise NotImplementedError()


class Circle(Shape, variety='circle'):
    def draw(self):
        print('Drawing a circle')


class Rectangle(Shape, variety='rectangle'):
    def draw(self):
        print('Drawing a rectangle')


if __name__ == '__main__':
    Shape.make('rectangle').draw()
    Shape.make('circle').draw()
```

## 装饰器

这里**不是介绍装饰器**，只是通过装饰器的形式来保存子类信息。我这里省略了 `Parser` 中的其它方法，只保留抽象方法。

```python
from abc import ABC, abstractmethod
from typing import Type, Callable


class Parser(ABC):
    @abstractmethod
    def extract_id(self, line: str) -> str:
        raise NotImplementedError()

    @abstractmethod
    def get_display_name(self, extracted_id: str) -> str:
        raise NotImplementedError()


class ParserFactory:
    def __init__(self) -> None:
        self.parsers: dict[str, Type[Parser]] = {}

    def register(self, parser_name: str) -> Callable[[Type[Parser]], Type[Parser]]:
        def wrapper(parser: Type[Parser]) -> Type[Parser]:
            self.parsers[parser_name] = parser
            return parser
        return wrapper

    def get_instance(self, name: str) -> Parser:
        if parser := self.parsers.get(name):
            return parser()
        raise NameError(f'unknown parser name: {name}')


factory = ParserFactory()


@factory.register('group')
class GroupParser(Parser):
    def extract_id(self, line: str) -> str:
        return 'GROUP ID'

    def get_display_name(self, extracted_id: str) -> str:
        return f'GROUP NAME {extracted_id}'


@factory.register('private')
class PrivateParser(Parser):
    def extract_id(self, line: str) -> str:
        return 'PRIVATE ID'

    def get_display_name(self, extracted_id: str) -> str:
        return f'PRIVATE NAME {extracted_id}'


def process(parser_name: str) -> None:
    parser = factory.get_instance(parser_name)
    print(parser.extract_id('ABC'))
    print(parser.get_display_name('DEF'))


if __name__ == '__main__':
    process('private')
    process('group')
```

这种方式比较直观，但需要额外创建一个 `factory` 全局变量。如果你非常讨厌全局变量，我还有另一种方式。

```python
from typing import Type


class ParserFactory:
    parsers: dict[str, Type[Parser]] = {}

    def __init__(self, name: str) -> None:
        self.name = name

    def __call__(self, parser: Type[Parser]) -> Type[Parser]:
        # 如果对使用 self 访问类属性感到不舒服，你可以
        # 用 self.__class__.parsers 或者 ParserFactory.parsers
        self.parsers[self.name] = parser
        return parser

    @classmethod
    def get_instance(cls, name: str) -> Parser:
        if parser := cls.parsers.get(name):
            return parser()
        raise NameError(f'unknown parser name: {name}')


@ParserFactory('group')
class GroupParser(Parser):
    def extract_id(self, line: str) -> str:
        return 'GROUP ID'

    def get_display_name(self, extracted_id: str) -> str:
        return f'GROUP NAME {extracted_id}'


@ParserFactory('private')
class PrivateParser(Parser):
    def extract_id(self, line: str) -> str:
        return 'PRIVATE ID'

    def get_display_name(self, extracted_id: str) -> str:
        return f'PRIVATE NAME {extracted_id}'


def process(parser_name: str) -> None:
    parser = ParserFactory.get_instance(parser_name)
    print(parser.extract_id('ABC'))
    print(parser.get_display_name('DEF'))
```

当然，也可以定义一个类方法 `register` 使代码**语义更清晰**，这里不再演示。

## 元类

~~我超，原！~~

这里有一个坑，就是 `ParserMeta` 需要继承自 `abc.ABCMeta`，因为 `Parser` 是继承自 `abc.ABC` 的抽象类，下文有详细解释。

```python
from abc import ABCMeta
from typing import Type, cast


class ParserMeta(ABCMeta):
    parsers: dict[str, Type[Parser]] = {}

    def __new__(mcs, name, bases, attrs) -> Type[Parser]:
        assert Parser in bases
        assert '__parser_name__' in attrs

        t = cast(Type[Parser], super().__new__(mcs, name, bases, attrs))
        mcs.parsers[attrs['__parser_name__']] = t
        return t

    @classmethod
    def get_instance(mcs, name: str) -> Parser:
        if parser := mcs.parsers.get(name):
            return parser()
        raise NameError(f'unknown parser name: {name}')


class GroupParser(Parser, metaclass=ParserMeta):
    __parser_name__ = 'group'

    def extract_id(self, line: str) -> str:
        return 'GROUP ID'

    def get_display_name(self, extracted_id: str) -> str:
        return f'GROUP NAME {extracted_id}'


class PrivateParser(Parser, metaclass=ParserMeta):
    __parser_name__ = 'private'

    def extract_id(self, line: str) -> str:
        return 'PRIVATE ID'

    def get_display_name(self, extracted_id: str) -> str:
        return f'PRIVATE NAME {extracted_id}'


def process(parser_name: str) -> None:
    parser = ParserMeta.get_instance(parser_name)
    print(parser.extract_id('ABC'))
    print(parser.get_display_name('DEF'))
```

如果直接让 `ParserMeta` 继承 `type`，那么会喜提下方的报错信息：

```bash
TypeError: metaclass conflict: the metaclass of a derived class 
must be a (non-strict) subclass of the metaclasses of all its bases
```

其实这句话我只看明白了大概意思，就是需要保证元类之间的继承关系，需要使 `ParserMeta` 继承与 `abc.ABCMeta` 即可解决。

具体的分析如下：首先，元类的本质是默认创建类的时候会调用 `type`，而指定元类后就调用指定的那个类。由于以下的继承关系（比较简单就不用专门的作图软件了，省略 `PrivateParser`，它和图示一样）：

```bash
abc.ABC <- abc.ABCMeta
  |
Parser
  |
GroupParser <- ParserMeta
```

如果 `ParserMeta` 不继承于 `abc.ABCMeta`，创建 `GroupParser` 时就无法判断使用 `abc.ABCMeta` 还是 `ParserMeta`，所以导致了上文中的报错。

## 题外话：依赖注入

其实本文内容和依赖注入没什么关系，但还是比较想提一下。

依赖注入，就是入参传一个 `interface`，结束。

哈哈，其实没这么简单，大家可以参考[维基百科](https://en.wikipedia.org/wiki/Dependency_injection)给出的解释，就是 `Spring` 那一套，只不过注入的时候需要手动指定。

本文中给的情景就是一个很好的解释依赖注入的例子。如果 `Parser` 类中有很多具体方法和抽象方法，那可以把这些抽象方法单独提取出来一个 `interface`，然后把 `Parser` 变成具体类，构造时接受这个 `interface` 为参数，通过这个 `interace` 来提取 `id` 之类的信息。

但是，由于 Python 中不存在 `interface` 这个概念，实际上我这个 `Parser` 类中方法也不是太多，如果硬要使用依赖注入反而会让代码更复杂，所以最终我就没用。

