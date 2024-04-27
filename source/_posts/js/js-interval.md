---
title: 使用setTimeout模拟setInterval
date: 2022-06-19 14:43:36.0
updated: 2022-12-10 19:54:04.06

categories: 
- Dev
tags: 
- JS
---

## 源码

> 来自半年后的说明：我把代码转放到 gists 里了，当时代码风格受 Python 影响较大，这里就不再更改了，毕竟大家本地都有自己的格式化工具。

前往[Github](https://gist.github.com/hikariyo/7523f05e580208398485930279f5456e)获取本文源码。

## 介绍

相信`setInterval`这个东西大火都比较熟了，这里不做关于它的介绍，而是关于本文是如何实现这一功能。

我们通过一个`Set`来保存定时器的`id`，当清除时，就把这个`id`删掉；当每一次调用时，都会检查一下当前`id`是否存储于这个`Set`中。

## 实现

如下：

```js
const interval = {
    _active: new Set(),
    _id: 0,

    set(callback, delay) {
        const id = this._id++
        this._active.add(id)

        const handler = () => {
            if (!this._active.has(id)) {
                return
            }
            setTimeout(() => {
                callback()
                handler()
            }, delay)
        }
        handler()
        return id
    },

    clear(id) {
        this._active.delete(id)
    }
}
```

我们用`_active`这个属性来保存上文提到过的`id`，其它的就没什么要说的了。

## 测试

用下面的代码测试它的正确性：

```js
let count = 0
const id = interval.set(() => {
    count++
    console.log(`count is ${count}`)
    if (count === 5) {
        interval.clear(id)
    }
}, 500)
```

一般定时器都是自己把自己`clear`掉，这里也不例外，这种需求应该还挺多的。

这篇文章写着玩的，实际用途没啥用，只是说明了`setInterval`可以基于`setTimeout`来实现。