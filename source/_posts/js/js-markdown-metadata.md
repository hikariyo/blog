---
title: 借助函数柯里化实现读取Markdown元数据
date: 2022-05-14 00:00:00.0
updated: 2022-12-10 19:16:23.385

categories: 
- Dev
tags: 
- JS
---

## 定义

> 柯里化（英语：Currying） 是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。
>
> 来源: [维基百科](https://zh.wikipedia.org/wiki/%E6%9F%AF%E9%87%8C%E5%8C%96)

说白了就是返回函数的函数，其实在开发过程中多半已经用过了，只是之前没有去了解过这个东西叫什么。

我认为，他还可以是被看为一种轻量级的面向对象。

## 示例

定义一个`getAdder`函数，返回一个`adder`：

```js
function getAdder(n) {
    return m => n + m
}

const adder = getAdder(3)

console.log(adder(4))
// 7
```

这便是一种柯里化的应用了。

## 实际问题

我在开发过程中遇到过这种需求，也就是你现在看到的页面，它是用markdown写成的，并且文件头部有一些元信息，即`metadata`，本质是一堆键值对，用来描述这个文件。比如当前文件的头部就是这样的：

```markdown
date: 2022-5-14
category: Learn
tags: JS TS
intro: 返回函数的函数，可以认为是轻量级面向对象
id: 11
```

我把这个结构抽象一下，改成下面这样：

```markdown
foo: abc
bar: def
```

我们的需求就是读取出来这些键值对。

## 解决方案

```js
function parser(metadata) {
    return key => {
        const prefix = key + ':'
        const data = metadata.find(s => s.startsWith(prefix))

        return data.substring(prefix.length).trim()
    }
}

const parse = parser([
    'foo: abc',
    'bar: def'
])

console.log(parse('foo'))
// abc

console.log(parse('bar'))
// def
```

可以看出，这里的`parser`函数就是保存了`metadata`这一变量内容并且重复使用。

针对于`parser`函数的执行效率，可以进行以下优化：

```js
function parser(metadata) {
    const map = new Map()
    for (const data of metadata) {
        const [key, value] = data.split(/\s*:\s*/)
        map.set(key, value)
    }
    return key => map.get(key)
}
```

这里提前把所有的键值对都保存下来了，到执行的时候只需要获取一下即可。

那么回到开头说的，他可以被看成一种轻量级的面向对象，所以说完全可以用到面向对象的方法实现这个功能：

```js
class Parser {
    metadata = new Map()

    constructor(metadata) {
        for (const data of metadata) {
            const [key, value] = data.split(/\s*:\s*/)
            this.metadata.set(key, value)
        }
    }

    parse(key) {
        return this.metadata.get(key)
    }
}

const metadata = [
    'foo: abc',
    'bar: def'
]

const parser = new Parser(metadata)

console.log(parser.parse('foo'))
// abc

console.log(parser.parse('bar'))
// def
```

但是实际应用中还是通过柯里化的方法更简洁明了，这也正体现这JS这门语言的魅力: 提供多种解决手段，我们总能从中找到较优的解决方案。