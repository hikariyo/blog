---
title: 原生JS实现哈希路由
date: 2022-05-25 17:19:22.0
updated: 2022-12-10 19:56:43.321

categories: 
- Learn
tags: 
- JS
---

## 源码

使用[Codepen在线体验](https://codepen.io/kifuan/pen/RwQjrzM)和查看源码。

## 页面代码

在我们用前端框架的时候，经常用到路由技术，就是在地址栏确实发生了变化但是页面没有刷新，那么本文就介绍通过更改哈希的方式实现这样一种路由，下面是页面代码：

```html
<h1>Hash Router</h1>
<div>
    <a href="#/">Home</a>
    |
    <a href="#/about">About</a>
    |
    <a href="#/page">Page</a>
</div>
<div id="view"></div>
```

我们的目的是，当点击这些a标签的时候，下面id为`view`的`div`内容能够发生指定的变化; 同样，当第一次加载页面的时候，也需要根据地址判断用户想要访问哪个页面。

那么在正式开始之前，我把样式先给出来：

```css
* {
    text-align: center;
}
```

哈，就是这么简单粗暴。

## 定义路由对象

为了更具有普遍性，我定义这里的`component`属性为一个异步函数，它可以是一个AJAX请求，这里使用`setTimeout`模拟一个请求：

```js
const routes = {
    '/': {
        title: 'Home',
        component: async () => '<p>This is home page</p>'
    },
    '/about': {
        title: 'About',
        component: async () => '<p>This is about page</p>'
    },
    '/page': {
        title: 'Page',
        component: () => new Promise(resolve => {
            setTimeout(() => {
                resolve('<p>This is a page delayed by 500ms</p>')
            }, 500)
        })
    }
}
```

那么接下来我们就该根据地址的不同来切换这几个页面了。

## 监听Hash变化

通过`window.onhashchange`或者`window.addEventListener('hashchange', () => {})`进行监听，这里选择第一种方式：

```js
window.onhashchange = function() {
    const path = location.hash.slice(1)
    const route = routes[path]

    document.title = route.title
    route.component().then(html => {
        const view = document.getElementById('view')
        view.innerHTML = html
    })
}
```

注意，直接用`location.hash`得到的**通常**是以井号开头的字符串，比如：

 ![当访问example.com/#/about时](/img/2022/08/hashrouter-img1.png)

但是如果说这个路径压根没有什么井号字符，它就会获得一个空字符串：

 ![直接访问根路径](/img/2022/08/hashrouter-img2.png)

所以说为了避免这种情况，我们需要在页面加载时检测一下`location.hash`：

```js
window.onload = function() {
    location.hash ||= '#/'
    window.onhashchange()
}
```

除此之外，这里还手动触发了一下`onhashchange`，然后就可以在页面刚加载出来，就根据路径的不同更换指定的页面。

末尾再给出一个[Codepen地址](https://codepen.io/kifuan/pen/RwQjrzM)。