---
title: 箭头函数和对象
date: 2022-05-05 00:00:00.0
updated: 2022-08-10 15:03:44
description: 箭头函数的函数体被解析成了对象，而标签又是一个合法的语法，所以产生了这个问题。
categories: 
- Learn
tags: 
- JS
---

有一个函数很短，比如这样：

```js
function getData() {
    return { data: 1 }
}

console.log(getData())
// { data: 1 }
```

想用ES6箭头函数语法简化一下，于是写成这样：

```js
const getData = () => { data: 1 }

console.log(getData())
// undefined
```

怎么会是呢？

## 解析

根本原因在于这一段：

```js
const getData = () => { data: 1 }
```

如果用传统`function`来写，和下面是等价的：

```js
function getData() {
    data: 1
}
```

那么还有一个问题，第二行这个玩意居然是合法语句？

是的，它定义了一个标签，一般套在循环结构外面，比如下面的：

```js
let i, j

outer:
for (i = 0; i < 10; i++) {
    for (j = 0; j < 10; j++) {
        if (j == 2) {
            break outer
        }
    }
}

console.log(i, j)
// 0 2
```

果然和Java一个模子里刻出来的，这个语法一模一样。

## 解决方法

到这里就很简单了，把第一行改一下即可。

```js
const getData = () => ({ data: 1 })
```

不过这么多括号着实有点丑。