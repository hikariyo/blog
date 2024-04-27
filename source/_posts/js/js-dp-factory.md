---
title: JS设计模式之工厂模式
date: 2022-06-01 18:54:45.0
updated: 2022-12-10 20:09:53.223
categories: 
- Learn
tags: 
- JS
---

## 工厂方法模式

> 定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法使一个类的**实例化延迟到子类**。
>
> ——《设计模式：可复用面向对象软件的基础》中文版第81页

在我理解中，所谓工厂方法，是指我们通过**调用已知的接口，获得未知的对象，做出预期的行为**。工厂方法为我们提供这一对象。试想一下，如果我们有将当前网页分享给QQ或者微信的需求，可以通过以下代码实现：

```js
class Sharer {
    constructor({msg, link}) {
        this.msg = msg
        this.link = link
    }

    share() {}
}

class WeChatSharer extends Sharer {
    share() {
        console.log(`Sharing ${this.msg} from ${this.link} via WeChat`)
    }
}

class QQSharer extends Sharer {
    share() {
        console.log(`Sharing ${this.msg} from ${this.link} via QQ`)
    }
}

class SharerFactory {
    create(options) {}
}

class QQSharerFactory extends SharerFactory {
    create(options) {
        return new QQSharer(options)
    }
}

class WeChatSharerFactory extends SharerFactory {
    create(options) {
        return new WeChatSharer(options)
    }
}

function share(factory, options) {
    factory.create(options).share()
}

share(new QQSharerFactory(), {
    msg: 'msg1',
    link: 'https://example.com'
})
// Sharing msg1 from https://example.com via QQ

share(new WeChatSharerFactory(), {
    msg: 'msg2',
    link: 'https://baidu.com'
})
// Sharing msg2 from https://baidu.com via WeChat
```

其实JS中大可不必这样做，直接传构造函数为参数就可以，因为JS中函数是一等公民。这里列出代码只是用于学习这个模式。

## 抽象工厂模式

> 提供一个接口以创建一系列相关或相互依赖的对象，而无需指定它们具体的类。
>
> ——《设计模式：可复用面向对象软件的基础》

说人话，就是工厂方法只有一个方法，抽象工厂有多个方法。许多文章喜欢以不同的操作系统匹配不同的外观举例，但是我们既然都用上了跑在浏览器里的JS，那就尽可能不考虑跨平台的问题。

所以我想到了移动端和桌面端UI不同，这或许是一个应用抽象工厂模式的良好切入点。以下列代码为例：

```js
class UIFactory {
    createNavbar() {}

    createContent() {}
}

class MobileUIFactory extends UIFactory {
    createNavbar() {
        return { position: 'top' }
    }

    createContent() {
        return { direction: 'vertical' }
    }
}

class DesktopUIFactory extends UIFactory {
    createNavbar() {
        return { position: 'left' }
    }

    createContent() {
        return { direction: 'horizontal' }
    }
}


function getFactory() {
    if (window.innerWidth < 1000) {
        return new MobileUIFactory()
    }
    return new DesktopUIFactory()
}

const factory = getFactory()
factory.createContent()
factory.createNavbar()
```

当然这些代码没有什么实际作用，但是这就是抽象工厂模式的思想：创建一系列相关的对象。

## 可拓展的工厂

其实在 *Learning JavaScript Design Patterns* 中所实现的抽象工厂模式，就是一个可拓展的工厂。如果按照《设计模式：可复用面向对象软件的基础》中的定义来说，它是不正确的。但我们不探讨者是否正确，我们也来实现一个可拓展的工厂。

### 通过Map实现

其实是通过对象，但是为了与前面所说的对象做区分，这里用Map映射这个说法。代码如下，还是实现一个分享的功能：

```js
class Sharer {
    constructor({msg, link}) {
        this.msg = msg
        this.link = link
    }

    share() {}
}

class WeChatSharer extends Sharer {
    share() {
        console.log(`Sharing ${this.msg} from ${this.link} via WeChat`)
    }
}

class QQSharer extends Sharer {
    share() {
        console.log(`Sharing ${this.msg} from ${this.link} via QQ`)
    }
}

class Factory {
    types = {}

    register(type, cls) {
        this.types[type] = cls
    }

    create(type, options) {
        const cls = this.types[type]
        return cls ? new cls(options) : undefined
    }
}

const factory = new Factory()
factory.register('qq', QQSharer)
factory.register('wechat', WeChatSharer)

factory.create('qq', {
    msg: 'msg1',
    link: 'https://example.com'
}).share()
// Sharing msg1 from https://example.com via QQ

factory.create('wechat', {
    msg: 'msg2',
    link: 'https://baidu.com'
}).share()
// Sharing msg2 from https://baidu.com via WeChat
```

我们通过给`register`方法传递一个构造函数（用`class`关键字定义的）来注册一个类，之后就可以通过`create`方法来创建它。

### 通过重写方法实现

通过继承，先检查是否有自己拓展的类型，之后再通过`super`调用到父类的方法，代码如下：

```js
class Sharer {
    constructor({msg, link}) {
        this.msg = msg
        this.link = link
    }

    share() {}
}

class WeChatSharer extends Sharer {
    share() {
        console.log(`Sharing ${this.msg} from ${this.link} via WeChat`)
    }
}

class QQSharer extends Sharer {
    share() {
        console.log(`Sharing ${this.msg} from ${this.link} via QQ`)
    }
}

class Factory {
    create(type, options) {
        if (type === 'qq') {
            return new QQSharer(options)
        }
    }
}

class ExpandedFactory extends Factory {
    create(type, options) {
        if (type === 'wechat') {
            return new WeChatSharer(options)
        }
        return super.create(type, options)
    }
}

const factory = new ExpandedFactory()

factory.create('qq', {
    msg: 'msg1',
    link: 'https://example.com'
}).share()
// Sharing msg1 from https://example.com via QQ

factory.create('wechat', {
    msg: 'msg2',
    link: 'https://baidu.com'
}).share()
// Sharing msg2 from https://baidu.com via WeChat
```

我个人推荐通过Map实现这个需求，因为这样写的不好之处在于它想要拓展就需要重新定义一个类，然后去重写父类的方法，等等。

## 总结

下面来自于 *Learning JavaScript Design Patterns*，我手动翻译过来的。

### 什么时候需要使用

工厂模式在下列几种情况是很有用的：

+ 当创建对象或初始化的过程很复杂时。
+ 当我们需要根据当前环境创建不同的对象时。
+ 当我们操作很多拥有共同属性的小对象时。
+ 当我们利用duck typing时，它可以很方便的用来解耦。

### 什么时候不要使用

+ 由于JS的动态类型，运用工厂方法可能会导致复杂的类型问题。如果你没有提供一个统一的接口，推荐直接使用`new`创建对象（TypeScript完美解决）。
+ 由于对象的创建被隐藏起来了，所以进行单元测试的时候比较困难。

## 参考

[设计模式：可复用面向对象软件的基础](https://book.douban.com/subject/34262305/)

[Learning JavaScript Design Patterns -- The Factory Pattern](https://www.patterns.dev/posts/classic-design-patterns/#factorypatternjavascript)