---
title: 更优雅的延时
date: 2022-04-16 00:00:00.0
updated: 2022-12-10 19:08:05.239
categories: 
- Learn
tags: 
- JS
---

## 需求

> 看到我八个月前写的东西，感叹时间过得真快啊，那时候我还在研究 `Promise`, `async`, `await` 这些玩意。

在JS里，如果有这样一个需求: 一段代码执行完后等待1秒再执行下一段。

听上去很简单，对吧?

## 常规思路

相信第一反应往往是这样写的：

```js
console.log("Doing A")
setTimeout(() => {
    console.log("Doing B")
}, 1000)
```

看上去也没什么，而且它确实解决了这个需求。那么问题来了，我需要的不是延时做两件事，而是延时做N件事，那又怎么写呢？

```js
console.log("Doing A")
setTimeout(() => {
    console.log("Doing B")
    setTimeout(() => {
    	console.log("Doing C")
	}, 1000)
}, 1000)
```

也达到目的了，不过这样下去的话代码会越来越丑，陷入**回调地狱**。

## 解决方案

可以使用ES6提供的`async`，`await`来优雅的解决这些问题。

首先定义一个`sleep`函数，它返回一个`Promise`对象：

```js
function sleep(ms) {
    return new Promise(resolve => {
        setTimeout(resolve, ms)
    })
}
```

那么接下来只要在需要的地方加上`await`调用这个函数，就可以达到延时的效果了：

```js
(async() => {
    console.log("Doing A")
    await sleep(1000)
    console.log("Doing B")
    await sleep(1000)
    console.log("Doing C")
})()
```

是不是比直接用`setTimeout`优雅多了呢?