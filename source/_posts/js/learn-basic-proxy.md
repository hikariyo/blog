---
title: 学习基础Proxy
date: 2022-05-24 13:28:42.0
updated: 2022-12-10 19:39:56.845
categories: 
- Learn
tags: 
- JS
---

## 背景

> 本文是我先学习[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)之后写下的，但是实现功能的代码都是我自己根据学习所得知识写出来的，不会照搬。

`Proxy` 是 `ES6` 为我们新增的对象，和它的名字一样，它起到一个**代理**作用，可以拦截对对象的操作，比如获取数据，设置属性等。


## Proxy

下面是从MDN摘过来的，但是释义是我自己写的：

```js
const p = new Proxy(target, handler)
```

+ `target`

  被`Proxy`代理的对象，或者叫原始对象。

+ `handler`

  定义了各种函数属性的对象，拦截捕捉对`target`对象的各种行为。

## handler

这里只说一些表层的方法，像捕捉`Object.getPrototypeOf`的`handler.getPrototype`这种较为底层的方法就不再进行说明了，因为那些大概是只有开发类库的时候才会用到。

所有的`Reflect.xxx`都相当于对应操作的默认行为，因为我们的目的是**拦截操作**，而非**改变这个操作的行为**，比如`set`就是设置属性，**不能把属性删除了**，所以在每个拦截函数中都调用了对应在`Reflect`对象上的API。

## handler.has(target, key)

拦截`in`操作符。

### 参数

+ `target`

  原始对象。

+ `key`

  被检测属性的键。

+ 返回值

  布尔值，表示存在与否。

### 示例

下面示例的作用是过滤掉对私有属性的检测，即以下划线开头的属性：

```js
const target = {
    foo: 1,
    _bar: 2,
}

const p = new Proxy(target, {
    has(target, key) {
        if (key[0] === '_') {
            return false
        }
        return Reflect.has(target, key)
    }
})

console.log('foo' in p)
// true

console.log('_bar' in p)
// false
```

## handler.get(target, key, receiver)

拦截获取属性的操作。

### 参数

+ `target` 

  原始对象。

+ `key`

  被获取属性的键。

+ `receiver`

  `Proxy`对象或者从原型链上获取时的对象，下方有代码解释。

+ 返回值

  任意，但当通过`Object.defineProperty`给原始对象赋值时，存在一定约束，详情请见[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy/get#%E7%BA%A6%E6%9D%9F)，因为本文只说基础应用。

### 示例

下面示例的作用是禁止获取私有属性，即以下划线开头的属性：

```js
const target = {
    foo: 1,
    _bar: 2,
}

const p = new Proxy(target, {
    get(target, key, receiver) {
        if (key[0] === '_') {
            return undefined
        }

        return Reflect.get(target, key, receiver)
    }
})


console.log(p.foo)
// 1

console.log(p._bar)
// undefined
```

此外，我们再探究一下`receiver`，这里的`receiver`和下方函数的`receiver`逻辑相同：

```js
let obj

const p = new Proxy({}, {
    get(target, key, receiver) {
        if (receiver === obj) {
            console.log('receiver is obj')
        } else if (receiver === p) {
            console.log('receiver is p')
        }
        return undefined
    }
})

obj = Object.create(p)

p.foo
// receiver is p

obj.foo
// receiver is obj
```

当我们访问`p`时, `p`这个代理本身被传入`receiver`; 当我们访问`obj.foo`时, 这时继承`p`的`obj`被传入`receiver`。

## handler.set(target, key, value, receiver)

拦截设置属性的操作。

### 参数

+ `target`

  原始对象。

+ `key`

  被设置属性的键。

+ `value`

  被设置属性的值。

+ `receiver`

  与`get`中的`receiver`相同。

+ 返回值

  布尔值，表示操作成功与否。

### 示例

下面示例的作用是禁止设置私有属性：

```js
'use strict'

const target = {
    foo: 1,
    _bar: 2
}

const p = new Proxy(target, {
    set(target, key, value, receiver) {
        if (key[0] === '_') {
            return false
        }
        return Reflect.set(target, key, value, receiver)
    }
})

p.foo = 10

p._bar = 20
// TypeError: 'set' on proxy: trap returned falsish for property '_bar'
```

只有当严格模式下，即`'use strict'`，返回`false`时才会引发`TypeError`，但无论什么模式，这里的`_bar`都不会被成功赋值。

## handler.deleteProperty(target, key)

拦截`delete`操作符。

### 参数

+ `target`

  原始对象。

+ `key`

  被删除属性的值。

+ 返回值

  布尔值，表示操作成功与否。

  但如果在`target`上的某属性是用`Object.defineProperty`定义为不可配置，那么该属性无法被删除，其实这个地方不需要我们太留意，因为`Reflect.deleteProperty`不会让这种操作成功进行。

### 示例

下面示例的作用是禁止删除私有属性：

```js
'use strict'

const target = {
    foo: 1,
    _bar: 2,
}

const p = new Proxy(target, {
    deleteProperty(target, key) {
        if (key[0] === '_') {
            return false
        }
        return Reflect.deleteProperty(target, key)
    }
})

delete p.foo

delete p._bar
// TypeError: 'deleteProperty' on proxy: trap returned falsish for property '_bar'
```

与`set`相同，这里要开启严格模式才会报错。

## handler.apply(target, thisArg, args)

拦截调用操作。

注意，`target`本身应该就是一个函数，如果不是在调用时会直接抛出一个`TypeError`。

### 参数

+ `target`

  原始**函数**，注意它应该是一个函数。

+ `thisArg`

  原来调用`target`时`this`应该是谁，它就是谁。

+ `args`

  传进来的参数，是一个数组。

+ 返回值

  任意。

### 示例

下面示例的作用是把原函数返回值都取反：

```js
function sum(a, b) {
    return a + b
}

const p = new Proxy(sum, {
    apply(target, thisArg, args) {
        return -Reflect.apply(target, thisArg, args)
    }
})

console.log(sum(1, 2))
// 3

console.log(p(1, 2))
// -3
```

## handler.construct(target, args, newTarget)

拦截`new`操作符。

注意这里的`target`本身就应该是一个构造函数或类，它可以被`new`调用，否则会直接抛出`TypeError`。

### 参数

+ `target`

  原始构造函数。

+ `args`

  `new`时传入的参数。

+ `newTarget`

  类似`set`和`get`的`receiver`，下面有解释。

+ 返回值

  必须是一个对象，即构造出的对象。

### 示例

下面示例的作用是把所有传入的参数都乘10：

```js
class Point {
    constructor(x, y) {
        this.x = x
        this.y = y
    }
}

const PointProxy = new Proxy(Point, {
    construct(target, args, newTarget) {
        args = args.map(n => n * 10)
        return Reflect.construct(target, args, newTarget)
    }
})
```

关于`newTarget`到底是谁，下面做一个探究：

```js
class Foo {}

const FooProxy = new Proxy(Foo, {
    construct(target, args, newTarget) {
        console.log(newTarget)
        return Reflect.construct(target, args, newTarget)
    }
})

class Bar extends FooProxy {}

new FooProxy()
// [class Foo]

new Bar()
// [class Bar extends Foo]
```

也就是说你`new`的是谁，这里的`newTarget`就是谁。

## Proxy.revocable(target, handler)

参数与`new Proxy`时相同，但会返回一个`proxy`对象和`revoke`函数：

```js
const { proxy, revoke } = Proxy.revocable(target, handler)
```

这里的`revoke`可以让`proxy`不可用，通常应用于只允许用户访问`proxy`对象且访问结束后就要销毁，从而不让用户把`proxy`对象保存下来以后再次访问。

### 示例

```js
const { proxy, revoke } = Proxy.revocable({}, {})

proxy.foo = 1
console.log(proxy.foo)
// 1

revoke()
console.log(proxy.foo)
// TypeError: Cannot perform 'get' on a proxy that has been revoked
```