---
title: JS设计模式之职责链模式
date: 2022-06-02 17:36:55.0
updated: 2022-12-10 20:07:44.608
categories: 
- Learn
tags: 
- JS
---

## 意图

> 使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连城一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。
>
> ——《设计模式：可复用面向对象软件的基础》中文版第167页

尤其是我们在处理不同类型的表单时，这个设计模式就能很好的派上用场。

## 示例

假设我们给一个商场写逻辑，需求是：

+ 已经支付了500元定金的用户享7折优惠。
+ 已经支付了100元定金的用户享9折优惠。
+ 未支付定金的普通用户不享受优惠。

除外，如果说库存不够，用户也是买不到的，考虑实现方法。

有两个思路，我可以定义一个函数把N个函数串起来，也可以借助`Promise`进行不断`next`链式调用，先演示串起来的方法：

```js
const NEXT = Symbol('next')

function order500(type, paid, left) {
    if (type === '500' && paid && left > 0) {
        return '30% off'
    }
    return NEXT
}

function order100(type, paid, left) {
    if (type === '100' && paid && left > 0) {
        return '10% off'
    }
    return NEXT
}

function orderNormal(type, paid, left) {
    if (type === 'normal' && paid && left > 0) {
        return 'no discount'
    }
    return NEXT
}

Function.prototype.next = function(f) {
    return (...args) => {
        const res = this.apply(undefined, args)
        if (res === NEXT) {
            return f.apply(undefined, args)
        }
        return res
    }
}

const order = order500.next(order100).next(orderNormal).next(() => {
    return 'you cannot buy it'
})

console.log(order('500', true, 3))
// 30% off

console.log(order('100', true, 2))
// 10% off

console.log(order('normal', true, 1))
// no discount

console.log(order('normal', true, 0))
console.log(order('normal', false, 5))
// you cannot buy it
```

我们给`Function`拓展了一个`next`方法，如果调用这个函数本身返回了`NEXT`，那就调用下一个函数，否则就把得到的结果返回来。

我们在一串`next`末尾又跟上了一个函数，如果说所有的逻辑都走不通，就会调用它返回`'you cannot buy it'`告诉用户买不了。

接下来我们通过`Promise`的方法实现这个功能：

```js
function order(type, paid, left) {
    return new Promise((next, success) => {
        if (type === '500' && paid && left > 0) {
            success('30% off')
        } else {
            next()
        }
    }).then(() => new Promise((next, success) => {
        if (type === '100' && paid && left > 0) {
            success('10% off')
        } else {
            next()
        }
    })).then(() => new Promise((next, success) => {
        if (type === 'normal' && paid && left > 0) {
            success('no discount')
        } else {
            next()
        }
    })).then(() => new Promise((_, success) => {
        success('you cannot buy it')
    })).catch((msg) => {
        console.log(msg)
    })
}

order('500', true, 3)
// 30% off

order('100', true, 2)
// 10% off

order('normal', true, 1)
// no discount

order('normal', true, 0)
order('normal', false, 5)
// you cannot buy it
```

但是如果要用`Promise`就该考虑异步的问题，因为`Promise.prototype.then`会执行异步代码而非同步。还是要根据实际情况来选择。

## 参考

[设计模式：可复用面向对象软件的基础](https://book.douban.com/subject/34262305/)

[JavaScript 设计模式之职责链模式](https://juejin.cn/post/6998826663643447326)

[JS职责链模式（责任链模式）](https://blog.csdn.net/qq_41176306/article/details/113993763)