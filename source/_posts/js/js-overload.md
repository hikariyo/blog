---
title: JS实现函数重载
date: 2022-06-01 14:37:23.0
updated: 2022-12-10 19:52:41.431

categories: 
- Dev
tags: 
- JS
---

## 需求

假设我们有这样一个数据（来源于参考文章中的*浅谈JavaScript函数重载*，链接在文章尾部给出）：

```js
const users = {
    values: ["Dean Edwards", "Alex Russell", "Dean Tom"]
}
```

要定义一个`find`方法，使得它可以在下列三种情况工作：

+ 当不传入参数时，它返回整个数组。
+ 当传入一个参数时，它返回以这个参数为名（即*First Name*）的数组。
+ 当传入两个参数时，它返回以第一个参数为名，第二个参数为姓（即*Last Name*）的数组。
+ 不符合上述条件，返回`undefined`，也就是不予处理。

接下来通过普通方法和函数重载的方法实现这一需求。

## 普通方法

考虑到根据`arguments`对象判断参数长度，写出以下代码：

```js
const users = {
    values: ["Dean Edwards", "Alex Russell", "Dean Tom"]
}

users.find = function() {
    switch (arguments.length) {
        case 0:
            return this.values
        case 1:
            return this.values.filter(name => name.startsWith(arguments[0]))
        case 2:
            return this.values.filter(name => name.startsWith(arguments[0])
                && name.endsWith(arguments[1]))
    }
}

console.log(users.find())
// [ 'Dean Edwards', 'Alex Russell', 'Dean Tom' ]

console.log(users.find('Dean'))
// [ 'Dean Edwards', 'Dean Tom' ]

console.log(users.find('Dean', 'Tom'))
// [ 'Dean Tom' ]

console.log(users.find('Dean', 'Tom', 'A'))
// undefined
```

确实可以正确运行，不过通常来说使用`switch case`都不是一个好主意：再添加逻辑的时候就比较混乱了，所以考虑下面通过函数重载的实现。

## 函数重载

通过两种方式实现函数重载，不使用闭包和使用闭包。

### 逐次添加

这是绝大多数文章实现函数重载的方法：

```js
function addMethod(obj, name, func) {
    const old = obj[name]
    obj[name] = function() {
        if (arguments.length === func.length) {
            return func.apply(this, arguments)
        } else if (typeof old === 'function') {
            return old.apply(this, arguments)
        }
    }
}
```

其中第6行使用`typeof old === 'function'`而不是`old !== undefined`的原因，我认为是它担心原来的`obj[name]`是一个数字、字符串或其他类型，使用`typeof`判断能使这个`addMethod`方法更加健壮。

接下来给出测试代码：

```js
const users = {
    values: ["Dean Edwards", "Alex Russell", "Dean Tom"]
}

addMethod(users, 'find', function() {
    return this.values
})

addMethod(users, 'find', function(firstName) {
    return this.values.filter(name => name.startsWith(firstName))
})

addMethod(users, 'find', function(firstName, lastName) {
    return this.find(firstName).filter(name => name.endsWith(lastName))
})

console.log(users.find())
// [ 'Dean Edwards', 'Alex Russell', 'Dean Tom' ]

console.log(users.find('Dean'))
// [ 'Dean Edwards', 'Dean Tom' ]

console.log(users.find('Dean', 'Tom'))
// [ 'Dean Tom' ]

console.log(users.find('Dean', 'Tom', 'A'))
// undefined
```

### 一次添加

除了逐个把函数使用闭包的方法添加到对象中，还可以通过下列代码实现一次添加全部的待重载函数：

```js
function overload(...funcs) {
    const map = funcs.reduce((pre, cur) => {
        pre.set(cur.length, cur)
        return pre
    }, new Map())

    return function() {
        return map.get(arguments.length)?.apply(this, arguments)
    }
}
```

可以看出，它接受了一个函数的数组，并以参数长度作为键、函数本身作为值创建了一个映射。当调用时，根据传入参数的不同，调用指定的函数。

下面是测试代码：

```js
const users = {
    values: ["Dean Edwards", "Alex Russell", "Dean Tom"]
}

function findAll() {
    return this.values
}

function findFirstName(firstName) {
    return this.values.filter(name => name.startsWith(firstName))
}

function findFullName(firstName, lastName) {
    return this.find(firstName).filter(name => name.endsWith(lastName))
}

users.find = overload(findAll, findFirstName, findFullName)

console.log(users.find())
// [ 'Dean Edwards', 'Alex Russell', 'Dean Tom' ]

console.log(users.find('Dean'))
// [ 'Dean Edwards', 'Dean Tom' ]

console.log(users.find('Dean', 'Tom'))
// [ 'Dean Tom' ]

console.log(users.find('Dean', 'Tom', 'A'))
// undefined
```

至于孰优孰劣，还请根据具体情况进行选择。

## 参考

+ [JavaScript Method Overloading](https://johnresig.com/blog/javascript-method-overloading/)
+ [浅谈JavaScript函数重载](https://www.cnblogs.com/yugege/p/5539020.html)
+ [JavaScript中的函数重载](https://juejin.cn/post/6844903636154187790)