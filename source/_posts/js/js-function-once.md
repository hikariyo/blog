---
title: JS实现只能调用一次的函数
date: 2022-06-12 18:36:54.0
updated: 2022-12-10 20:03:03.036

categories: 
- Learn
tags: 
- JS
---

## 源码

> 来自半年后的说明：我把代码转放到 gists 里了，当时代码风格受 Python 影响较大，这里就不再更改了，毕竟大家本地都有自己的格式化工具。

前往[Github](https://gist.github.com/hikariyo/d8ee700982a1b421c68eee14e2bc1a9e)获取本文源码。

## 目标

我们的想法是一个函数只有第一次调用的时候有效，如下：

```js
function foo() {
    console.log('Hello world')
}

foo()
foo()
foo()
```

毋庸置疑，它会打印出三个`Hello world`。这篇文章就是来看看通过哪个方式能让他只打印一个。

## 第一想法

如果它是在一个对象里面，这个事就好办了，如下：

```js
const obj = {
    hello() {
        console.log('Hello from obj')
        this.hello = () => {}
    }
}

obj.hello()
obj.hello()
```

无论后面调用多少次，因为我们已经让`this.hello`变成了一个空函数，所以都不会再次打印了。但是，我们的目标是没有`obj.`这个前缀。或许你可能会想到用`bind`解决这个问题：

```js
const obj = {
    hello() {
        console.log('Hello from obj')
        this.hello = () => {}
    }
}

const hello = obj.hello.bind(obj)
hello()
hello()
```

但是，你可以自己试一下，这是没用的。因为在这个函数里面设置的`obj.hello`已经和我们赋值出去的`const hello`没有关系了。

## 最终方案

还是要请出我们的老朋友`Proxy`来解决这个问题：

```js
function once(f) {
    let called = false

    function apply(target, thisArg, args) {
        if (called) {
            return
        }
        called = true
        return Reflect.apply(target, thisArg, args)
    }

    return new Proxy(f, { apply })
}
```

通过一个布尔变量`called`来保存这个函数是否已经被调用，那么之后我们就可以这样写：

```js
const foo = once(() => {
    console.log('Hello from foo')
})

foo()
foo()
foo()
```

无论调用多少遍只会执行第一次，达成目的。