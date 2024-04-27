---
title: JS设计模式之原型模式
date: 2022-06-02 15:05:52.0
updated: 2022-12-10 20:10:45.248
categories: 
- Learn
tags: 
- JS
---

## 意图

> 用原型示例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。
>
> ——《设计模式：可复用面向对象软件的基础》中文版第89页

所以这里说的原型类似于对象的蓝图。但是，由于JS的灵活性，这个设计模式对于JS来说不是很重要。

> 因为在像C++这样的静态语言中，类不是对象，并且运行时只能得到很少或者得不到任何类型信息，所以Prototype（原型模式）特别有用。而在Smalltalk或Objective C这样的语言中Prototype就不是那么重要了。
>
> ——《设计模式：可复用面向对象软件的基础》中文版第92页

所以这里只做一个学习，我们只需要知道这个设计模式对于JS来说不是很重要即可。

## 示例

下面以几个拥有相同接口的形状为例：

```js
class Rectangle {
    constructor(width, height) {
        this.width = width
        this.height = height
    }

    draw() {
        console.log(`Drawing a rectangle, size: ${this.width}x${this.height}`)
    }

    clone() {
        return new Rectangle(this.width, this.height)
    }
}

class Circle {
    constructor(radius) {
        this.radius = radius
    }

    draw() {
        console.log(`Drawing a circle, radius: ${this.radius}`)
    }

    clone() {
        return new Circle(this.radius)
    }
}

class ShapeFactory {
    constructor(prototypeShape) {
        this.prototypeShape = prototypeShape
    }

    create() {
        return this.prototypeShape.clone()
    }
}

function draw(factory) {
    factory.create().draw()
}

const factory1 = new ShapeFactory(new Rectangle(3, 5))
const factory2 = new ShapeFactory(new Circle(5))

draw(factory1)
// Drawing a reactangle, size: 3x5

draw(factory2)
// Drawing a circle, radius: 5
```

在JS里这样写属实是有点大可不必了，但是对于一些静态类型的语言来说，这个方法可以实现动态创建不同对象的功能。

注意到，这里也使用了工厂模式：这个工厂以传入的对象为原型创建新的对象。

## 参考

[设计模式：可复用面向对象软件的基础](https://book.douban.com/subject/34262305/)