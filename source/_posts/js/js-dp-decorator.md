---
title: JS设计模式之装饰器模式
date: 2022-06-02 16:30:17.0
updated: 2022-12-10 20:08:10.874

categories: 
- Learn
tags: 
- JS
---

## 意图

> 动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活。
>
> ——《设计模式：可复用面向对象软件的基础》中文版第132页

在Java中这样的设计可太多了，比如进行IO操作时的`XxxReader`，那么这里就模仿它进行一个应用。

## 示例

如下：

```js
class NumberReader {
    constructor(start) {
        this.pos = start
    }

    read() {
        return this.pos++
    }
}

class DoubleReader {
    constructor(reader) {
        this.reader = reader
    }

    read() {
        return this.reader.read() * 2
    }
}

class CharReader {
    constructor(reader) {
        this.reader = reader
    }

    read() {
        const code = 'a'.charCodeAt(0) + this.reader.read()
        return String.fromCharCode(code)
    }
}

class UpperCharReader {
    constructor(reader) {
        this.reader = reader
    }

    read() {
        return this.reader.read().toUpperCase()
    }
}

const reader = new UpperCharReader(
    new CharReader(
        new DoubleReader(
            new NumberReader(5)
        )
    )
)

console.log(reader.read())
// K

console.log(reader.read())
// M
```

我们来分析一下执行过程：

1. `NumberReader.prototype.read()`被调用，返回`5`
2. `DoubleReader.prototype.read()`被调用，返回`5*2=10`
3. `CharReader.prototype.read()`被调用，返回字母`'a'`后第`10`个字母`'k'`
4. `UpperCharReader.prototype.read()`被调用，返回`'k'.toUpperCase()`即`'K'`

第二个`reader.read()`同理。这就是一种经典的应用了，不过这是Java里常见的用法，JS里我还不太清楚。

## 参考

[设计模式：可复用面向对象软件的基础](https://book.douban.com/subject/34262305/)