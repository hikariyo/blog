---
title: JS设计模式之单例模式
date: 2022-06-02 13:46:46.0
updated: 2022-12-10 20:11:27.809

categories: 
- Learn
tags: 
- JS
---

## 意图

> 保证一个类仅有一个实例，并提供一个访问它的全局访问点。
>
> ——《设计模式：可复用面向对象软件的基础》中文版第96页

这个概念有些类似于全局变量，它确保所有引用都访问到唯一的实例对象，这可以节省掉创建对象的开销。但是，相应地，在懒加载模式下，它也带来了检查是否已经创建对象的开销。所以还是要根据情况灵活选择使用与否。

## 示例

有两种实现方法，下面都列出。或许还有更多，这里就不深究了。

### 更改构造函数的返回值

在JS中，构造函数是可以有返回值的：当返回一个对象时，它就会被作为`new`操作的结果；当返回一个基本类型（`number`，`string`等）时，这个返回值是无效的。下列代码可以说明这一观点：

```js
class Foo {
    constructor() {
        return 1
    }
}

class Bar {
    constructor() {
        return { baz: 1 }
    }
}

console.log(new Foo() === 1)
// false

console.log(new Bar().baz === 1)
// true
```

借助这个功能，我们可以写出下列代码：

```js
const data = []

class Singleton {
    static instance = undefined

    constructor() {
        data.push(this)
        if (Singleton.instance === undefined) {
            Singleton.instance = this
        }
        return Singleton.instance
    }
}

const obj1 = new Singleton()
const obj2 = new Singleton()

console.log(obj1 === obj2)
// true

console.log(data[0] === data[1])
// false
```

虽然两次`new`的返回值相同，但实际上我们创造了两个对象，所以这不是一个最佳的方案。

### 通过静态方法

由于上面的方法会创造出多个对象，所以就有了下面这个方式：

```js
class Singleton {
    static instance = undefined

    static getInstance() {
        if (this.instance === undefined) {
            this.instance = new Singleton()
        }
        return this.instance
    }
}

const obj1 = Singleton.getInstance()
const obj2 = Singleton.getInstance()

console.log(obj1 === obj2)
// true
```

注意，这里`getInstance`方法中引用的`this`指向`Singleton`这个类，因为JS类的本质还是一个对象，而我们通过`Singleton.getInstance()`这种方式调用，就相当于把`this`与`Singleton`绑定。

## 参考

[设计模式：可复用面向对象软件的基础](https://book.douban.com/subject/34262305/)

[Learning JavaScript Design Patterns -- The Singleton Pattern](https://www.patterns.dev/posts/classic-design-patterns/#singletonpatternjavascript)