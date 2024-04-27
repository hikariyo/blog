---
title: JS迭代器和生成器
date: 2022-05-21 00:00:00.0
updated: 2022-12-10 19:45:28.533

categories: 
- Learn
tags: 
- JS
---

## 背景

遍历数组的时候，我相信大多数人已经用上ES6的`for...of`语法了：

```js
const arr = [1, 2, 3]

for (const item of arr) {
    console.log(item)
}
// 1 2 3
```

那么我们能不能让自己创建的对象也支持`for...of`呢? 答案自然是可以的，这么好用的语法不给一个自定义支持简直天理难容。

## 引入

在Python中，有一个好用的`range`对象，它可以让我们这样写代码：
```python
for i in range(5):
    print(i)
## 0 1 2 3 4

print(list(range(5)))
## [0, 1, 2, 3, 4]
```

我们这里要借助迭代器和生成器实现的就是这个东西，实际上`range`在Python中也是通过迭代器的方式实现的。

## 同步迭代器

既然这么说了肯定有异步的迭代器，不过不着急，我们等会再讨论异步的问题，先上代码：

```js
function range(end) {
    let start = 0

    const iterator = {
        next() {
            if (start === end) {
                return { done: true }
            }
            return { value: start++, done: false }
        }
    }

    return {
        [Symbol.iterator]: () => iterator
    }
}

for (const i of range(5)) {
    console.log(i)
}
// 0 1 2 3 4
```

我们定义了一个`range`函数，它返回一个**可迭代对象**。

详细的内容请访问[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols)查阅，我这里做一个总结：
+ 可迭代对象需要有一个`Symbol.iterator`函数属性，返回迭代器对象。
+ 迭代器对象需要有一个`next`函数属性，返回迭代结果。
+ 迭代结果需要有`value`属性和`done`属性，这个看名字就知道什么意思了。

所以说，根据这些，我们可以想到一个简化代码的思路，就是让`range`返回的对象本身也是一个迭代器，当调用`Symbol.iterator`时返回`this`：

```js
function range(end) {
    let start = 0
    return {
        next() {
            if (start === end) {
                return { done: true }
            }
            return { value: start++, done: false }
        },

        [Symbol.iterator]() {
            return this
        }
    }
}

for (const i of range(5)) {
    console.log(i)
}
// 0 1 2 3 4
```

已经优化了很多了，但是这样还是不够简练，于是引出我们下文要说的内容。

## 同步生成器

我们可以发现只是实现一个简单的数字范围生成器就需要10多行代码，这显然是有点麻烦的，于是es6为我们提供了一个东西，它就是**生成器**：

```js
function* range(end) {
    for (let i = 0; i < end; i++) {
        yield i
    }
}

for (const i of range(5)) {
    console.log(i)
}
// 0 1 2 3 4
```

现在代码简单多了，然后我们看一下`range`函数返回了什么东西：

```js
const r = range(5)
console.log(r)
// Object [Generator] {}

const itor = r[Symbol.iterator]()
console.log(r === itor)
// true
```

和我们自己实现的时候类似，它调用`Symbol.iterator`时返回的是`this`。

## 相关语法

下面给出与迭代器和生成器有关的语法。

### for...of语法

遍历元素值：
```js
const arr = [1, 2]

for (const i of arr) {
    console.log(i)
}
// 1 2
```

看到这里我们也可以大致模拟一下`for...of`内部的执行原理了：

```js
function forOf(obj, action) {
    const itor = obj[Symbol.iterator]()

    while (true) {
        const res = itor.next()

        if (res.done) {
            break
        }

        action(res.value)
    }
}

forOf([1, 2, 3], console.log)
// 1 2 3
```

### for await...of语法

在开发过程中，AJAX请求返回的`Promise`对象是很常见的。

我们用`setTimeout`模拟异步请求，有下列代码：

```js
function asyncValue(value) {
    return new Promise(resolve => {
        setTimeout(() => resolve(value), value * 100)
    })
}


(async() => {
    const arr = [2, 4, 5].map(asyncValue)

    for await (const i of arr) {
        console.log(i)
    }
    // 2 4 5
})()
```
用`for await...of`语法可以来遍历一个`Promise`对象组成的数组。

### 展开语法
不管是迭代器生成器，都支持es6的展开语法：

```js
function* values() {
    yield 1
    yield 2
    yield 3
}

console.log([...values()])
// [ 1, 2, 3 ]
```

不做过多的演示了，只需知道展开语法也利用了迭代器生成器即可。


## 异步迭代器

接下来，我们继续回到`range`对象上，上面的相关语法只是起一个过渡作用。

给对象一个`Symbol.asyncIterator`属性，支持它被`for await...of`迭代：

```js
function asyncValue(value) {
    return new Promise(resolve => {
        setTimeout(() => resolve(value), value * 100)
    })
}

function asyncRange(end) {
    let start = 0

    return {
        async next() {
            if (start === end) {
                return { done: true }
            }

            const value = await asyncValue(start++)
            return { value, done: false }
        },

        [Symbol.asyncIterator]() {
            return this
        }
    }
}

(async() => {
    for await (const i of asyncRange(5)) {
        console.log(i)
    }
    // 0 1 2 3 4
})()
```

## 异步生成器

就是加一个`async`关键字：

```js
function asyncValue(value) {
    return new Promise(resolve => {
        setTimeout(() => resolve(value), value * 100)
    })
}

async function* asyncRange(end) {
    for (let i = 0; i < end; i++) {
        yield await asyncValue(i)
    }
}

(async() => {
    for await (const i of asyncRange(5)) {
        console.log(i)
    }
    // 0 1 2 3 4
})()
```