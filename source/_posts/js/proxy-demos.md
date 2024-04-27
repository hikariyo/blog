---
title: 用Proxy写几个Demo
date: 2022-05-24 15:20:35.0
updated: 2022-12-10 19:36:25.626
categories: 
- Learn
tags: 
- JS
---

## 背景

本文是根据阮一峰大大的文章写几个小Demo，这里的代码都是我自己看了一遍需求敲出来的，不存在直接复制黏贴的情况，请放心。

## 数组负数索引

众多语言都支持数组的索引为负数，就像这段Python代码一样：

```python
arr = [1, 2, 3]
print(arr[-1])
## 3
```

我们可以通过`Proxy`让JS也支持这一操作：

```js
function getPositiveKey(key, length) {
    const index = Number(key)
    if (index < 0) {
        key = String(index + length)
    }
    return key
}

function negativeArray(arr) {
    return new Proxy(arr, {
        get(target, key) {
            key = getPositiveKey(key, target.length)
            return Reflect.get(target, key)
        },
        set(target, key, value) {
            key = getPositiveKey(key, target.length)
            return Reflect.set(target, key, value)
        }
    })
}

const arr = negativeArray([5, 6, 7])

console.log(arr[-1])
// 7

console.log(arr[1])
// 6
```

注意，尽管我们用`arr[-1]`，但是传进去的`key`还是会被转为字符串，所以需要一步转换。

同时，我们在`getPositiveKey`中，为了符合`Reflect.get`函数的参数类型要求，把计算出来的索引也转换成了字符串，不过这里是无所谓的，因为传给`Reflect`后它会自己进行一个转换。

## 数字链式调用

下面代码的目的是：

+ 如果获取的是`value`，就返回数字本身。
+ 如果获取的东西在`Math`中存在，就调用它，并且把结果再包装成我们的代理对象。
+ 都不存在就返回一个`undefined`，因为我这里是一个小Demo，就不做太多的处理了。

```js
function proxy(n) {
    return new Proxy({ n }, {
        get(target, key) {
            if (key === 'value') {
                return target.n
            }

            if (key in Math) {
                return proxy(Math[key](target.n))
            }
            
            return undefined
        }
    })
}

console.log(proxy(64).sqrt.log2.value)
// 3
```

上述操作就是先把64开根，结果是8; 再取以2为底的对数，结果是3。

注意到这里把数字包了一层对象，因为`Proxy`是不支持代理基本类型的。

## DOM生成器

此Demo的[Codepen地址](https://codepen.io/kifuan/pen/rNJGwZP)，可以在线体验一下。

每次都写`document.createElement`然后`dom.appendChild`实在是太煎熬了，所以有了这样一个例子：

```js
const dom = new Proxy({}, {
    get(_, tag) {
        return (...children) => {
            const el = document.createElement(tag)
            children.forEach(child => {
                if (typeof child === 'string') {
                    child = document.createTextNode(child)
                }
    
                if (child instanceof Node) {
                    // 所有DOM都继承了Node
                    el.appendChild(child)
                } else {
	                // 如果child不是Node就认为它是属性对象
                    Object.entries(child).forEach(([key, value]) => {
                        el.setAttribute(key, value)
                    })
                }
            })

            return el
        }
    }
})

const root = document.getElementById('root')
const el = dom.p(
    dom.span('Hello! '),
    'This is ',
    dom.a('my website', { href: 'https://kifuan.top', target: '_blank' }),
)
root.appendChild(el)
```

别忘了在HTML里带上一个：

```html
<div id="root"></div>
```

## 数据双向绑定

`Vue`里面最香的就是这玩意，为表单数据获取验证省出了巨大时间，那么在这里通过`Proxy`的知识我们也可以实现一个简单的双向绑定。


### 外部链接

如果只是把这些链接写在开头很容易被忽略掉，所以我这里单独摘出来一个板块让下面这些**更醒目**：

+ **视频** 我录了一个视频从头开始实现这个功能，过程中有说到怎么去思考这个问题的实现方法，**点[这里](https://www.bilibili.com/video/BV1iY4y1B7mW)去B站看**。

+ **Codepen** 本Demo的[Codepen地址](https://codepen.io/kifuan/pen/WNMZEJm)，可以在线体验。


### 需求

只要我们这么写

```html
<input type="text" data-value="foo" data-update="foo">
<p data-value="foo"></p>
```

当上方输入的时候下方就会实时更新，也就是说：

+ `data-value`指的是当数据更新时**被实时更新**。
+ `data-update`指的是数据会**被这个元素更新**。

### 实现

Talk is cheap. Show me the code.
```js
'use strict'

function bindUpdates(setProp) {
    return Array.from(document.querySelectorAll('*[data-update]'))
        .reduce((pre, cur) => {
            const key = cur.dataset.update
            cur.addEventListener('keyup', () => {
                setProp(key, cur.value)
            })
            // 一个key只会被一个DOM更新
            return { ...pre, [key]: cur }
        }, {})
}

function bindValues() {
    return [...document.querySelectorAll('*[data-value]')]
        .reduce((pre, cur) => {
            const key = cur.dataset.value
            // 一个key会被多个DOM读取
            pre[key] ||= []
            pre[key].push(cur)
            return pre
        }, {})
}

function bind() {
    const values = bindValues()

    const updates = bindUpdates((key, value) => {
        data[key] = value
    })

    const data = new Proxy({}, {
        set(_, key, value) {
            values[key].forEach(el => {
                if (el.value !== undefined) {
                    el.value = value
                } else {
                    el.innerText = value
                }
            })
            // 表示设置成功
            return true
        },

        get(_, key) {
            return updates[key].value
        }
    })

    return data
}

const data = bind()
```

之后在HTML里这么写：

```html
<h1>双向绑定</h1>
<input type="text" data-value="account" data-update="account" placeholder="账号"><br><br>
<input type="text" data-value="password" data-update="password" placeholder="密码">
<p>账号: <span data-value="account"></span></p>
<p>密码: <span data-value="password"></span></p>

<button onclick="data.account = ''">清空账号</button>
<button onclick="data.password = ''">清空密码</button>
```

就可以达到预期效果了，想要预览可以到Codepen里面查看，在这里[再放一个链接](https://codepen.io/kifuan/pen/WNMZEJm)。