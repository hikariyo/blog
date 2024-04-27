---
title: 从实例中学习reduce方法
date: 2022-05-25 12:43:37.0
updated: 2022-12-10 19:41:24.338

categories: 
- Learn
tags: 
- JS
---

## 需求

如下：
```js
['Red', 'White', 'Yellow']
// 转为
{ Red: 'Red', White: 'White', Yellow: 'Yellow' }
```
接下我来用几个方法去实现这个功能。

## 传统方法

自然是没有for循环解决不了的问题：

```js
const Colors = ['Red', 'White', 'Yellow']

const ObjColors = {}

for (const color of Colors) {
    ObjColors[color] = color
}

console.log(ObjColors)
// { Red: 'Red', White: 'White', Yellow: 'Yellow' }
```
这个方法固然没错，但是for循环已经有被滥用的趋势了，如果说在可以使用函数式API的情况下，我们最好尝试着使用它们。

## map与reduce

当我不理解`reduce`函数的功能时，我先想到了这个方法：

+ 先把每一项转化成`{ item: 'item' }`的对象，
+ 然后通过展开运算符合并它们。

```js
const Colors = ['Red', 'White', 'Yellow']
    .map(item => ({ [item]: item }))
    .reduce((pre, cur) => ({ ...pre, ...cur }))

console.log(Colors)
// { Red: 'Red', White: 'White', Yellow: 'Yellow' }
```

比一开始的代码更加清晰了，但是感觉`map`这一步是否有点多余? 接下来的纯`reduce`就可以解决这个问题。

## reduce

相信很多人和我一样，在印象中`reduce`函数就是个用来求和的，比如：

```js
const arr = [1, 2, 3]

console.log(arr.reduce((a, b) => a + b))
// 6
```

但它能做的不只这些，我们可以给它传第二个参数`initialValue`，看一下运行过程：

```js
const arr = [1, 2, 3]

console.log('No initial value')
let result = arr.reduce((pre, cur) => {
    console.log(pre, cur)
    return pre + cur
})
console.log('Result is', result)

console.log('\nWith initial value')
result = arr.reduce((pre, cur) => {
    console.log(pre, cur)
    return pre + cur
}, 0)
console.log('Result is', result)
```
运行结果是：
```js
No initial value
1 2
3 3
Result is 6

With initial value
0 1
1 2
3 3
Result is 6
```

通过观察可以知道，如果说不传入初始值，第一次调用是`callback(arr[0], arr[1])`。

而传入初始值，第一次调用就是`callback(initialValue, arr[0])`。

通过这个特性，我们可以通过纯粹的`reduce`方法实现我们一开始的需求：

```js
const Colors = ['Red', 'White', 'Yellow']
    .reduce((pre, cur) => ({ ...pre, [cur]: cur }), {})

console.log(Colors)
// { Red: 'Red', White: 'White', Yellow: 'Yellow' }
```

这是前面有一篇文章里面的需求，就是要把这些字符串存起来当成枚举使用。