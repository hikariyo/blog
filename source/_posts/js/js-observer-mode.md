---
title: 实现观察者模式
date: 2022-05-29 18:49:55.0
updated: 2022-12-10 19:50:46.121

categories: 
- Dev
tags: 
- JS
---

## 源码

> 来自半年后的说明：我把代码转放到 gists 里了，当时代码风格受 Python 影响较大，这里就不再更改了，毕竟大家本地都有自己的格式化工具。

点击[这里](https://gist.github.com/hikariyo/267b25e28520176ff8277e6ea2542f7b)前往Github查看本文源码。

## 实现

我们要定义一个`observer`函数，它会把我们传入的对象复制一份再进行代理，并且返回的代理对象上还有一个`observe`方法，用来添加一个观察者：


```js
function observable(obj) {
    const observers = {}
    
    // 使用Object.assign避免修改原对象
    const target = Object.assign({
        observe(key, fn) {
            observers[key] = fn
            // For method chaining
            return this
        }
    }, obj)

    const set = (target, key, value) => {
        const oldVal = Reflect.get(target, key)

        // 如果存在则调用当前属性对应的观察者回调函数，
        // 并且把它的旧值和新值传给回调函数
        observers[key]?.call(undefined, oldVal, value)
        
        return Reflect.set(target, key, value)
    }

    return new Proxy(target, { set })
}
```

其中我们使用了`Object.assign`来避免修改原对象，但是如果非要不那么做也没什么大问题，只是容易出bug。

## 使用

注意我们传给它的回调是接受两个参数，第一个表示旧值，第二个表示新值：

```js
const ob = observable({
    x: 0,
    y: 1,
}).observe('x', (oldVal, newVal) => {
    console.log(`x: ${oldVal} -> ${newVal}`)
}).observe('y', (oldVal, newVal) => {
    console.log(`y: ${oldVal} -> ${newVal}`)
})

ob.x = 10
// x: 0 -> 10

ob.y = 20
// y: 1 -> 20
```

读者可以打开控制台试一试效果。