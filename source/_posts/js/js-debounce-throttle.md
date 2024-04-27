---
title: JS防抖与节流
date: 2022-06-07 23:18:23.0
updated: 2022-12-10 19:32:01.836
categories: 
- Learn
tags: 
- JS
---

## 源码

前往[Codepen](https://codepen.io/kifuan/pen/vYdzpPz)在线体验。

## 介绍

**防抖**和**节流**是两个JS中的概念，它们被广泛应用于被频繁触发的事件中，如搜索框在输入时会弹出候选列表：如果每次输入都发送一个AJAX请求来获取数据，那么后台就要被刷爆了。所以，这里就引出了本文要介绍的概念。

### 防抖

**在 `x` 秒内**，无论调用多少次这个函数，它只会在**最后一次调用**的 `x` 秒后被真正执行。

> 在参考文章里举了这样一个例子：
>
> 一个小孩向妈妈要蛋糕，他的妈妈被弄烦了。所以她告诉小孩说，只有他能保持安静一个小时，才能得到蛋糕。中途任意要一次就重新开始计算。

我只是照着原文，摘了重点部分翻译过来放在上面。

### 节流

**在`x`秒内**，无论调用多少次这个函数，它**只会被执行一次**。

> 在参考文章里举了这样一个例子：
> 
> 还是那个小孩要蛋糕，但这次他的妈妈允许他无限制地要。尽管如此，不论他要多少次，只能在一小时内获取到一块蛋糕。如果他不再要了，自然也就不给了。

也是照着原文，摘了重点部分翻译过来放在上面。

## 实现

虽然这个概念是比较有用的，但是原生JS并没有给我们提供一个接口。无妨，借助`setTimeout`可以轻松实现。

### 防抖

我们使用了JS里强大的闭包：

```js
function debounce(fn, delay) {
    let timeout = undefined

    return function() {
        clearTimeout(timeout)
        timeout = setTimeout(() => {
            timeout = undefined
            fn.apply(this, arguments)
        }, delay)
    }
}
```

可以看出，这个返回的函数无论被调用多少次，第一件事就是清除掉原有的定时器。如果到时间了，就执行函数`fn`；如果被清除掉了，那就不执行。

### 节流

节流可以使用`setTimeout`实现，也可以借助`Date`判断调用时间。

#### 使用setTimeout

贴代码：

```js
function throttle(fn, delay) {
    let timeout = undefined

    return function() {
        if (timeout !== undefined) {
            return
        }
        timeout = setTimeout(() => {
            timeout = undefined
            fn.apply(this, arguments)
        }, delay)
    }
}
```

当定时器存在时，调用返回的函数不会做任何事。直到定时器被执行，`timeout`被重新设置成了`undefined`，才能进行下一轮操作。

#### 使用Date

贴上代码：

```js
function throttleDate(fn, delay) {
    let prev = Date.now()

    return function() {
        const now = Date.now()

        if (now - prev < delay) {
            return
        }
        
        prev = now
        fn.apply(this, arguments)
    }
}
```

如果当前时间`now`和上一次的时间`prev`的差比预期时间`delay`小，就什么也不做。否则，就执行函数`fn`，并且重置上一次的时间`prev`。

## 测试

写了这么多逻辑，没有测试自然是不合适的。这里就写个简单的网页来测试了：

```html
<button id="b1">Debounce: <span>0</span></button>
<br><br>
<button id="b2">Throttle: <span>0</span></button>
<br><br>
<button id="b3">Throttle(Date): <span>0</span></button>
```

加上点简单粗暴的样式：

```css
* {
    font-size: 1.2rem;
}

button {
    width: 200px;
}
```

当然，还要把这些按钮的事件都绑定上：

```js
function bind(id, wrapper) {
    document.getElementById(id).onclick = wrapper(function() {
        this.querySelector('span').innerText++
    }, 500)
}

bind('b1', debounce)
bind('b2', throttle)
bind('b3', throttleDate)
```

这里的`bind`函数只是做了把指定`id`的元素绑定上一个回调函数，执行时会使得它子节点中的`span`元素自增。

同时，这个回调函数会被我们传入的`wrapper`包装起来，也就是`debounce`、`throttle`或`throttleDate`，并且延时都是`500ms`，也就是`0.5s`。

可以点击右侧的目录回到文章开头给源码的地方，到Codepen里实时预览最终效果。

## 应用

1. 可以给按钮的`onclick`事件进行**节流**，用于**防止用户频繁点击按钮**。
2. 可以给窗口的`resize`事件进行**防抖**，当最终**重新调整大小后**，再重新渲染页面。
3. 可以给输入框的`keyup`，`keydown`等事件进行**防抖**，当用户**停止输入一段时间后**弹出提示。
4. 其实输入框的事件进行**节流**也可以，用于**实时但不要太频繁**地根据用户的输入弹出提示。
5. 当**NodeJS**需要频繁更新文件到硬盘里的时候，进行**防抖**处理，这样只有在**操作停止的一段时间后**才会更新到硬盘里，有效**减少IO操作**。

## 参考

[Debouncing and Throttling in JavaScript](https://www.telerik.com/blogs/debouncing-and-throttling-in-javascript)