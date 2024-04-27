---
title: 通过EventBus实现发布与订阅模式
date: 2022-05-30 12:48:27.0
updated: 2022-12-10 19:33:40.309

categories: 
- Dev
tags: 
- JS
---

## 源码

> 来自半年后的说明：我把代码转放到 gists 里了，当时代码风格受 Python 影响较大，这里就不再更改了，毕竟大家本地都有自己的格式化工具。

点击[这里](https://gist.github.com/hikariyo/84226810d77b8a88b1f1ef94d0fc7ea3)前往Github获取本文源码。

## 背景

> 后注: 发布-订阅模式属于设计模式中的**行为型模式**，基本上和**观察者模式**相同，至于具体定义存在争议，这里不进行讨论。

在`Vue`中，我们经常通过**全局事件总线**进行简单的组件间通信，那么究其原理其实并不难，本文就来着手实现一个这样的功能。

## 实现

代码不是很难，直接贴出来，后文会解释这些代码的作用：

```js
class EventBus {
    callbacks = {}

    on(event, callback) {
        // 第一处标记，下面会提到
        const callbacks = this.callbacks[event] ||= new Set()
        callbacks.add(callback)
        return this
    }

    once(event, callback) {
        const actual = (...args) => {
            callback(...args)
            this.remove(event, actual)
        }
        this.on(event, actual)        
        return this
    }

    emit(event, ...args) {
        this.callbacks[event]?.forEach(callback => {
            callback(...args)
        })
        return this
    }

    remove(event, callback = undefined) {
        if (callback) {
            this.callbacks[event]?.delete(callback)
        } else {
            delete this.callbacks[event]
        }
        return this
    }
}
```

首先观察到所有函数都返回了`this`，由此我们可以进行**链式调用**。下面逐一解释：

+ `on(event, callback)`

  我们把传入的`callback`添加到`this.callbacks[event]`这个集合中，为了以后触发事件的时候被调用。

  注意到我们在第一处标记，通过`||=`**短路赋值**给`this.callbacks[event]`一个空集合，并且把这个赋值表达式的结果给一个**局部变量**。并且由于集合是引用类型，之后我们就可以通过操作这个局部变量来**影响到原数据**了。

+ `once(event, callback)`

  使用`once`添加的回调函数只会被调用一次，方法体中我们把传入的函数包装了一层。这个函数被调用后会通过`this.remove(event, actual)`删除本身，达到只调用一次的效果。


+ `emit(event, ...args)`

  触发指定的事件，并且传入参数，方法体内部通过`forEach`遍历所有的回调函数。

+ `reomve(event, callback = undefined)`

  删除指定的事件。如果没有指定是哪一个回调，就把整个事件对应的回调函数都删除掉。


## 使用

通过下列代码测试这个简易框架的正确性：

```js
const bus = new EventBus()

bus.on('foo', () => {
    console.log('foo 1')
}).on('foo', () => {
    console.log('foo 2')
}).emit('foo')
// foo 1
// foo 2

bus.remove('foo')
    .emit('foo')
// it will do nothing


bus.on('bar', n => {
    console.log('bar:', n)
}).once('bar', n => {
    console.log('bar once:', n)
}).emit('bar', 3)
// bar: 3
// bar once: 3

bus.emit('bar', 3)
// bar: 3
```

读者可以到控制台试验一下，看看是否是预期效果。