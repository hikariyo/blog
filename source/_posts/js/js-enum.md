---
title: 在JS中愉快地使用枚举
date: 2022-05-21 00:00:00.0
updated: 2022-08-10 15:23:44.118

categories: 
- Learn
tags: 
- JS
---



## 背景

在JS中并没有原生枚举的实现，可以通过下面几种方法来模拟类似的操作。

## 直接使用字符串

上代码：

```js
function isWeekend(day) {
    return day === 'Sat' || day === 'Sun'
}

console.log(isWeekend('Sun'))
// true

console.log(isWeekend('Mon'))
// false


console.log(isWeekend('sun'))
// false
```

这样是非常常见的用法，包括很多类库都在这样做，但是如果哪天把字符串拼错了，就会直接返回`false`，所以说这个方法是不太合理的。

## 使用变量储存枚举值

这次进阶了一下，虽然代码啰嗦了，但是犯错概率会大大降低：

```js
const Days = Object.freeze({
    Mon: 'Mon',
    Tue: 'Tue',
    Wed: 'Wed',
    Thur: 'Thur',
    Fri: 'Fri',
    Sat: 'Sat',
    Sun: 'Sun'
})

function isWeekend(day) {
    return day === Days.Sun || day === Days.Sat
}

console.log(isWeekend(Days.Sun))
// true

console.log(isWeekend(Days.Mon))
// false
```

这里使用了`Object.freeze`，就像这个函数名一样，把对象**冰冻**起来，下面的代码会解释这些：

```js
const obj = Object.freeze({
    foo: 1
})

obj.foo = 'bar'

console.log(obj.foo === 1)
// true
```

尽管更改并不会报错，但是也不会生效。

## 使用数字

这也是老生常谈的内容了，好多语言在没有枚举类型的时候都喜欢这么干：

```js
const Days = Object.freeze({
    Mon: 0,
    Tue: 1,
    Wed: 2,
    Thur: 3,
    Fri: 4,
    Sat: 5,
    Sun: 6
})

function isWeekend(day) {
    return day === Days.Sun || day === Days.Sat
}

console.log(isWeekend(Days.Sun))
// true

console.log(isWeekend(Days.Mon))
// false
```

## 使用Symbol类型

虽然说用变量把枚举值储存起来了，不过只要别人愿意，他完全可以这样做：

```js
// 使用字符串时
isWeekend('Sun')
// 使用数字时
isWeekend(0)
```

那我们属于是白封装了，所以说`Symbol`类型就是针对这种情况的：

```js
const Days = Object.freeze({
    Mon: Symbol('Mon'),
    Tue: Symbol('Tue'),
    Wed: Symbol('Wed'),
    Thur: Symbol('Thur'),
    Fri: Symbol('Fri'),
    Sat: Symbol('Sat'),
    Sun: Symbol('Sun')
})

function isWeekend(day) {
    return day === Days.Sat || day === Days.Sun
}

console.log(isWeekend(Days.Sun))
// true

console.log(isWeekend(Days.Mon))
// false

console.log(isWeekend(Symbol('Sun')))
// false
```

得益于`Symbol`对象的唯一性，我们可以达到必须让别人使用我们定义的变量这一目的。

话虽如此，但是用`Symbol`类型有一个缺点，那就是它不可以被`JSON.stringify`识别：

```js
console.log(JSON.stringify({ day: Days.Sun }))
// {}
```

并且也不能通过`JSON.parse`获取到，所以这种方法仅适用于不和后台交互时使用。

## JS定义枚举集合时的优化

可以尝试下列几种方法，只需要写出来枚举的名字，通过几个数组的API进行赋值操作。

但是由于是动态执行的，效率相对来说会降低，不过这通常是不足一提的。

### 数字类型

用`index`当成枚举值：

```js
const Days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat'].reduce((pre, cur, index) => {
    return { ...pre, [cur]: index }
}, {})

```

也可以先映射再合并：

```js
const Days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat']
    .map((item, index) => ({ [item]: index }))
    .reduce((pre, cur) => ({ ...pre, ...cur }))
```

上面使用展开运算符的操作都可以用`Object.assign`来代替，这里不做多余的演示了。

### 字符串类型

和数字类型定义时差不多：

```js
const Days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat'].reduce((pre, cur) => {
    return { ...pre, [cur]: cur }
}, {})
```

换汤不换药，和上面的数字类型定义时类似：

```js
const Days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat']
    .map(item => ({ [item]: item }))
    .reduce((pre, cur) => ({ ...pre, ...cur }))
```

如果你想让`Days`的值为`Symbol`的话，我相信你如果能轻松地看到这里，应该知道怎么更改，就把类似`[item]: item`改成`[item]: Symbol(item)`即可。