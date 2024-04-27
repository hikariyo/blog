---
title: 关于JS执行顺序
date: 2022-05-21 00:00:00.0
updated: 2022-12-10 19:34:25.106

categories: 
- Learn
tags: 
- JS
---

## 背景

众所周知，JS是单线程语言，但它支持异步操作，其核心机制就是JS引擎的**事件循环**。

先上一段代码：

```js
console.log(1)

setTimeout(() => {
    console.log(2)
})

new Promise(resolve => {
    console.log(3)
    resolve()
}).then(() => {
    console.log(4)
})

console.log(5)

// 1 3 5 4 2
```

背后的原因就是事件循环中的**宏任务**与**微任务**。

## 原理

总的来说，流程图如下：

![流程图](/img/2022/08/execution-order.jpg)

1. `Promise`中的代码块是立即执行的。

   下列代码可以证明：

   ```js
   console.log(1)
   
   new Promise(() => {
       console.log(2)
   })
   
   console.log(3)
   
   // 1 2 3
   ```

   这部分很简单了，从上到下同步执行。

2. `Promise`后的`then`传入的方法是**微任务**。

   下面代码为[V8引擎源码](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/8.4-lkgr/src/objects/promise.tq#72)，注意它是用V8内部语言`Torque`编写，我们只需要看它是继承了`Microtask`即可知它是一个微任务，无需在意更多细节：

   ```typescript
   @generateCppClass
   extern class PromiseResolveThenableJobTask extends Microtask {
     context: Context;
     promise_to_resolve: JSPromise;
     thenable: JSReceiver;
     then: JSReceiver;
   }
   ```

   所以说它会在最外层代码执行完后再去执行。

3. `setTimeout`或者`setInterval`都是**宏任务**。

   我这里实在是没找到源码哪里表明了这个东西，于是直接在`NodeJS`里换个方式证明一下。

   在`NodeJS`中，`process.nextTick`可以设置一个微任务，使用下列代码测试：

   ```js
   setTimeout(() => {
       console.log(1)
   })
   
   process.nextTick(() => {
       console.log(2)
   })
   
   process.nextTick(() => {
       console.log(3)
   })
   
   console.log(4)
   
   // 4 2 3 1
   ```

   我们把`setTimeout`放在最开始，而且不管设置了几次`nextTick`，`setTimeout`里的函数体总是最后执行的，由此可见它的确是一个宏任务。

## 更复杂一点

不管我的`Promise`怎么组合，怎么套，由于`setTimeout`设置的是宏任务，所以它始终在这些微任务都执行完成之后才会运行：

```js
setTimeout(() => {
    console.log(1)
})

Promise.resolve().then(() => {
    console.log(2)

    return Promise.resolve()
}).then(() => {
    console.log(3)

    Promise.resolve().then(() => {
        console.log(4)
    })
})

console.log(5)
// 5 2 3 4 1
```

可以看出，2 3 4全都是微任务，而5是最外层同步执行的代码，1是由`setTimeout`设置的下一个宏任务。

## 总结

再回到开头的程序：

```js
console.log(1)

setTimeout(() => {
    console.log(2)
})

new Promise(resolve => {
    console.log(3)
    resolve()
}).then(() => {
    console.log(4)
})

console.log(5)

// 1 3 5 4 2
```

因为1，3，5都是同步执行的，所以它们按顺序排列; 2是宏任务，会放到下一次事件循环时执行; 4是微任务，在首次运行时就把它添加到了微任务队列中，所以在下一次事件循环之前就会被执行。

通过这样的事件循环，使得单线程的JS也可以拥有异步的能力，使得如AJAX请求这样费时间的操作可以被安排到后面来执行，不影响页面的加载和渲染。