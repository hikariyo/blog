---
title: JS函数无限柯里化
date: 2022-05-24 20:55:07.0
updated: 2022-12-10 20:04:20.541

categories: 
- Learn
tags: 
- JS
- Algo
---

## 不用箭头函数的实现
网上看到很多用箭头函数的版本，在看不懂的时候非常的眼花。

所以在这里我选择先用纯粹的`function`配合`arguments`分析完原理，再过渡到轻量级的箭头函数。下面给出最原始的实现：

```js
function curry(f, ...savedArgs) {
    return function() {
        const totalArgs = [...savedArgs, ...arguments]
        if (totalArgs.length >= f.length) {
            return f(...totalArgs)
        }
        return curry(f, ...totalArgs)
    }
}
```

步骤如下：

1. 我们在定义时就做了一个手脚，那就是留了一个可变长参数`savedArgs`

2. 第3行定义了一个`totalArgs`总参数数组，它包含着外层保存下来的`savedArgs`以及这个函数本身的`arguments`

3. 第4行判断了总参数`totalArgs`与原始函数的长度`f.length`，如果参数数量足够，那就直接调用原始函数`f`并且返回结果
4. 第7行就是如果说参数还不够，那就把总参数`totalArgs`一并传给`curry`包装起来，等待下一次调用

那么我们可以试一下好不好用，读者也可以打开控制台试一试：

```js
const add = (a, b, c) => a + b + c


curry(add)(1)(2)(3)
curry(add)(1, 2)(3)
curry(add)(1)(2, 3)
curry(add)(1, 2, 3)
```

以下结果全都是6，符合我们的要求。

## 箭头函数轻量级实现

众所周知，箭头函数是一种轻量级的函数，它不像`function`那样会有冗余的字段。

那么分析完原理之后就较为简单了：

```js
const curry = (f, ...outer) => {
    return (...inner) => {
        if (outer.length + inner.length >= f.length) {
            return f(...outer, ...inner)
        }
        return curry(f, ...outer, ...inner)
    }
}
```

我觉得这已经够好了，但是如果你说还能压缩的话也对，它可以写成一行：

```js
const curry = (f, ...outer) => (...inner) => outer.length + inner.length >= f.length ? f(...outer, ...inner) : curry(f, ...outer, ...inner)
```

但是我觉得正常人类是看不大懂这玩意的，**不推荐！**