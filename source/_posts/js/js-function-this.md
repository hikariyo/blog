---
title: JS的函数和this
date: 2022-05-03 00:00:00.0
updated: 2022-08-10 15:28:26.527

categories: 
- Learn
tags: 
- JS
- Python
---



## 背景

我没有系统性的从头开始学过一遍JS，全凭之前学的其它语言，尤其是Python，直接上手操作了，所以留了不少的坑。

虽然对我来说是一个坑，但我觉得本文更适合放到学习的分类里。

给出下列代码。这里是对原问题的一个抽象，只是把问题的核心单独摘出来了：

```js
function each(arr, action) {
    for (const item of arr) {
        action(item)
    }
}

const arr1 = [1, 2, 3]
const arr2 = []
each(arr1, item => arr2.push(item))

console.log(arr2)
// [ 1, 2, 3 ]
```

这个函数它的功能就是遍历一遍数组，然后把每个元素传给`action`参数。到目前为止是没有问题的。

## 问题复现

我这里自作聪明，把上面的第9行改了一下，简化代码：

```js
each(arr1, arr2.push)
```

然后就顺利的报错了，这里给出错误信息：

```
        action(item)
        ^
TypeError: Cannot convert undefined or null to object
```

这里是node的报错，当时在浏览器里内容不太一样，但也提到了`undefined`相关的内容。

## 原理

原理很简单，就是`action(item)`这样调用的时候，传给`action`的`this`是`undefined`。

新建一个文件，写入下列代码足以验证。这里的`foo`是`obj`对象的实例方法，`es6`语法：

```js
'use strict'

function call(func) {
    func()
}


const obj = {
    foo() {
        console.log(this)
    }
}


obj.foo()
// { foo: [Function: foo] }


call(obj.foo)
// undefined
```

这里要是严格模式，否则`this`会指向`global`(node)或者`window`(浏览器)。

这里扯一下Python，同样的代码它运行出来结果就会不一样了。如果你对Python没有兴趣可以略过这里的对比：

```python
def call(f):
    f()

class Foo:
    def bar(self):
        print(self)

obj = Foo()

obj.bar()
## <__main__.Foo object at 0x0000017E14126E10>

call(obj.bar)
## <__main__.Foo object at 0x0000017E14126E10>
```

这里的原因就是，当我们使用`obj.bar`这样获得方法的话，Python会给我们把`obj`和第一个参数`self`绑定上。

## 解决方案

我们需要进行一个操作，如果也想像Python那样： `Function.prototype.bind()`

>  `bind()` 方法创建一个新的函数，在 `bind()` 被调用时，这个新函数的 `this` 被指定为 `bind()` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。
>
> 来源: [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

所以说我们把上面的JS测试代码改成这样，就可以了：

```js
call(obj.foo.bind(obj))
// { foo: [Function: foo] }
```

但是回到我们实际应用场景，还是这样写更好点：

```js
call(() => obj.foo())
// { foo: [Function: foo] }
```

或许还有更多的方法。至于如何选择，就仁者见仁、智者见智了。